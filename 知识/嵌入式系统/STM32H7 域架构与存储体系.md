---
type: concept
tags: [stm32, stm32h7, arm, cortex-m7, memory-architecture, power-domain]
created: 2026-08-14
updated: 2026-08-14
sources: ["[2026-08-14 - STM32H723VE 数据手册](../../来源/2026-08-14%20-%20STM32H723VE%20数据手册.md)"]
---

# STM32H7 域架构与存储体系

STM32H7 系列（Cortex-M7 @ 最高 550 MHz）与 F1/F4/G4 最根本的架构差异不在算力，而在于一套**分域电源管理 + 分层存储体系**。理解它，才理解 H7 的性能上限为什么是 550 MHz 而不是 216 MHz 的简单翻倍，以及"代码放哪、数据放哪、DMA 怎么喂"这些工程决策的依据。

## 1. D1/D2/D3 电源域

H7 的 VCORE（内核电源，由内部 LDO 产生，典型 1.0~1.36 V 分档）被划分为三个可独立控制的电源域：

| 域 | 内容 | 特点 |
|----|------|------|
| **D1** | Cortex-M7 核、ITCM/DTCM、AXI SRAM、MDMA、DMA2D、LTDC、以太网 MAC 等高性能外设 | 可独立断电（DStandby），CPU 子系统整体关停 |
| **D2** | 大部分经典外设（USART/SPI/I2C/定时器/ADC 等）+ SRAM1/2 | 可独立断电 |
| **D3** | 系统控制（RCC/PWR/EXTI/SYSCFG）+ SRAM4 + 备份 SRAM + BDMA | **常驻不断电**——系统"心脏" |

关键推论：**D3 永不断电**，所以唤醒逻辑、低功耗状态机必须放在 D3 的 SRAM4 或备份 SRAM 里跑；D1/D2 断电后其 SRAM 内容丢失，只有备份 SRAM 保留。

### 低功耗模式的分级

H7 把低功耗做成了层级结构，每级对应"关掉多少"：

| 系统模式 | D1 | D2 | D3 | 效果 |
|----------|----|----|----|----|
| Run | DRun/DStop/DStandby | 同左 | DRun | 全速，域可按需降级 |
| Stop | DStop/DStandby | 同左 | DStop | 系统时钟停，SRAM 保持 |
| Standby | DStandby | DStandby | DStandby | 仅备份域（RTC/备份 SRAM）存活 |

- 每域内部还有 CPU 级模式：CSleep（CPU 时钟停，WFI/WFE 进入）、CStop（CPU 子系统停）
- 域降级的约束：**域内所有成员都进入低功耗，域才降级**——一个没睡的外设拖住整个域
- Stop 模式下调压器档位（SVOS3-5）决定哪些外设保留唤醒能力：SVOS3 下 UART/SPI/I2C/LPTIM 可唤醒，SVOS4/5 只能 GPIO 或异步中断唤醒
- 域模式矩阵（system vs domain）来自 DS13313 Table 2（p.27）

### VCORE 电压调节（VOS）

H7 用**电压调节换频率**，而不是 F1 时代"固定电压换功耗"：

| 档位 | VCORE 典型 | CPU 上限 | 设计意图 |
|------|-----------|----------|----------|
| VOS0 | 1.36 V | 520 MHz（+boost 550） | 极限性能 |
| VOS1 | 1.21 V | 400 MHz | 高性能 |
| VOS2 | 1.10 V | 300 MHz | 均衡 |
| VOS3 | 1.00 V | 170 MHz | 低功耗运行 |

工程要点：

1. **VOS0 用内部 LDO 时结温限制降到 105°C、VDD 下限升到 1.7 V**——高压高频档的代价。改用外部旁路供电（VCAP 直供 VCORE）才恢复 125°C/1.62 V
2. 总线时钟与 VOS 联动：fACL/fHCLK 从 275 MHz（VOS0）降到 85 MHz（VOS3）
3. VCAP 外接电容（2×2.2 µF，ESR <100 mΩ）是 LDO 稳压环路的稳定条件
4. 升降档必须先改 VOS 再升频、先降频再降 VOS——硬件上 VOS 切换有稳定时间

## 2. 存储分层：为什么 H7 有 7 种 RAM

H723 的 564 KB RAM 拆成 7 块，每块的定位不同。核心矛盾：**550 MHz 下 Flash 访问有等待，AXI 总线有仲裁延迟**——Cortex-M7 的解法是让"确定性的快内存"走专用总线。

| 存储 | 位置 | 总线 | 定位 |
|------|------|------|------|
| ITCM | 核内 64-bit 专用口 | ITCM 口 | 关键实时代码：中断 ISR、控制环路，零等待确定性 |
| DTCM | 核内 2×32-bit 双口 | DTCM 口 | 关键实时数据：栈、堆、高频访问变量；双口支持双发射并行 |
| AXI SRAM | D1 域 AXI 总线 | 64-bit AXI | 大吞吐数据，DMA 流主战场 |
| SRAM1/2 | D2 域 AHB | 32-bit AHB | 普通外设数据 |
| SRAM4 | D3 域 AHB | 32-bit AHB | 低功耗状态机、D3 常驻代码 |
| 备份 SRAM | 备份域 | — | Standby/VBAT 保留，带写保护 |

- **TCM 与 AXI SRAM 之间有一段 192 KB 可分配区**：按 64 KB 粒度决定给 ITCM（指令）还是 AXI SRAM（数据）——启动时按应用剖分，是 H7 工程的标准动作
- 所有 RAM 带 ECC（SECDED 单纠双检）：SRAM 7 bit/32 位字，AXI-SRAM/ITCM 8 bit/64 位字；Flash 256 bit 数据 + 10 bit ECC
- 备份 SRAM 在 VBAT 供电下可跨主电源断电保留，天然适合"掉电日志、校准参数"

### Cache 是刚需，不是可选项

H723 的 CoreMark 数据（550 MHz，全外设关）：

| 执行位置 | Cache | CoreMark | µA/CoreMark |
|----------|-------|----------|-------------|
| ITCM / Flash / AXI SRAM | ON | 2778 | 52.2 |
| Flash | OFF | 923 | 107.3 |
| AXI SRAM | OFF | 1271 | 82.6 |
| SRAM1 | OFF | 790 | 122.2 |

- Cache ON 时三种介质性能完全一致——**ICache/DCache（各 32 KB）抹平了介质差异**
- Cache OFF 后 Flash 执行掉到 1/3 性能，单位算力功耗翻倍
- 推论：Flash 存代码 + Cache ON 是默认解；只有对延迟确定性有极端要求的 ISR 才放 ITCM

### DMA 体系：四条腿

- **MDMA**（D1 域，16 通道）：唯一带主 AXI 口 + 专用 AHBS 口——能直接访问 TCM，能做链表传输。定位是"内存搬运的主力"，还能在 Sleep 模式给 TCM 装代码
- **DMA1/DMA2**（D2 域，8 流，FIFO）：经典外设 DMA
- **BDMA**（D3 域）：D3 内与外设的搬运（如 LPUART）
- **DMAMUX**：请求路由器，任意外设请求可路由到任意通道，还支持事件触发合成请求

分层意图：MDMA 在 D1 与 TCM/AXI SRAM 之间搬大块，DMA1/2 在 D2 外设与 SRAM 之间搬小包，互不争抢总线。

## 3. 时钟体系

- **4 内部 RC**：HSI 64 MHz（启动默认）、HSI48（USB/RNG）、CSI 4 MHz（快速起振、低功耗唤醒）、LSI 32 kHz（IWDG）
- **2 外部**：HSE 4~48 MHz 晶体（外部源可到 50 MHz）、LSE 32.768 kHz
- **3 PLL**：PLL1 系统时钟（到 550 MHz），PLL2/PLL3 内核时钟——**音频 I2S 用独立 PLL 实现 audio class 精度**，这是 H7 相对 F4 的音频能力来源
- 总线频率由 VOS 档位决定（VOS0：AXI/AHB 275 MHz，APB 137.5 MHz）
- 通信外设双时钟域：总线接口时钟（随系统变）+ 内核时钟（外设功能时钟，独立）——改系统频率不破坏 UART 波特率

## 4. 安全与启动

- 启动源：BOOT 脚 + BOOT_ADDx 选项字节，可指向 0x0000 0000~0x3FFF FFFF 内**任意** Flash/RAM/系统 Bootloader 地址
- 系统 Bootloader（128 KB 系统 Flash）支持 USART/I2C/SPI/FDCAN/USB-DFU 烧录
- H7 系列内部安全能力分层：H723 无 Crypto/OTFDEC（纯算力型号）；需要硬件加密/在线解密的型号（H750/H753）在加密外设上扩展——**选型时安全能力是 H7 子系列间的重要分水岭**
- 存储器写保护由 RDP 读保护等级 + WRP 区域写保护 + 安全用户存储区组成

## 5. 与低阶 STM32 的对比

| 维度 | F1/F4/G4 思路 | H7 思路 |
|------|---------------|---------|
| 电源域 | 单一 VDD 域 + 模式切换 | D1/D2/D3 三域独立断电 |
| 频率调节 | 固定电压 + PLL 分频 | VOS 电压调节换频率上限 |
| 快内存 | CCM SRAM（G4，10 KB 级） | ITCM/DTCM + 可分配区（百 KB 级） |
| 缓存 | 无（靠 ART/预取） | 32 KB I$+D$，性能依赖 Cache |
| DMA | 1-2 控制器 + DMAMUX | MDMA + 2×DMA + BDMA 四层 |
| 时钟 | 1 PLL | 3 PLL（系统 + 音频 + 内核） |

一句话概括：H7 把"快、确定、可关"三件事拆到硬件结构里——TCM 管确定，Cache 管快，域管可关。

## 相关页面

- [STM32H723VE](../../元件/MCU无线/STM32H723VE.md) — H7 入门型号元件页（本页数据来源）
- [STM32G431](../../元件/MCU无线/STM32G431.md) — G4 实时控制线的对照（CCM SRAM vs TCM 的定位差异）
- [2026-08-14 - STM32H723VE 数据手册](../../来源/2026-08-14%20-%20STM32H723VE%20数据手册.md) — 来源摘要（DS13313 Rev 5）
