---
type: entity
tags: [electronics, mcu, stm32, arm, cortex-m3]
created: 2026-09-30
updated: 2026-09-30
sources: []
---

# STM32F103RCT6

> STMicroelectronics · Arm Cortex-M3 72 MHz · LQFP-64 · 256 KB Flash / 48 KB SRAM
> 本库**复用度最高的 MCU**：5 个工程使用，全部作为主板主控。

**手册**：`F:\Projects\PCB\Library\Datasheet\Chip_Microcontroller_STM32F103xCDE.PDF`
ST **DS5792 Rev 13**（2018-07），143 页，覆盖 STM32F103xC / xD / xE。

> [!note] 页码约定
> 本页 `p.N` 一律指 **PDF 页码**。该手册 PDF 页码与印刷页码一致（PDF 第 1 页标注 `1/143`）。

---

## 1. 身份选型

订购型号解码（p135, Table 75 Ordering information scheme）：

| 段 | 值 | 含义 |
|---|---|---|
| STM32 | — | Arm 32 位 MCU |
| F | — | 通用型 |
| 103 | — | 性能线 |
| **R** | 64 pins | 引脚数（R=64 / V=100 / Z=144） |
| **C** | 256 KB | Flash 容量（C=256KB / D=384KB / E=512KB） |
| **T** | LQFP | 封装（T=LQFP / H=BGA / Y=WLCSP64） |
| **6** | −40 ~ +85 °C | 工业温度档（7 = −40 ~ +105 °C） |

`p1` 核心与存储：

- 内核 Arm Cortex-M3，**72 MHz**，1.25 DMIPS/MHz（0 等待周期）
- Flash **256 KB**；SRAM **48 KB**（p11 Table 2：xC 档 = 48 KB，xD/xE 档 = 64 KB）
- 3 × 12-bit 1 µs ADC（**Rx 封装 16 通道**）、2 × 12-bit DAC、12 通道 DMA
- **51 个 GPIO**（Rx / Vx / Zx 分别为 51 / 80 / 112，p11 Table 2）
- 供电 **2.0 ~ 3.6 V**；`VBAT` 供 RTC 与后备寄存器

`p11` Table 2 外设数量（Rx 列）：

| 外设 | 数量 |
|---|---|
| 通用定时器 / 高级 / 基本 | 4 / 2 / 2 |
| SPI（可作 I2S） | 3 |
| I²C | 2 |
| USART | 5 |
| USB / CAN / SDIO | 各 1 |
| 12-bit ADC | 3 个，16 通道 |
| 12-bit DAC | 2 通道 |

> [!warning] **FSMC 在 R 封装上不可用** —— 这是选型时最容易踩的坑
> `p1` Features 列出 FSMC 与 LCD 并行接口，但那是**整个 xC/xD/xE 系列**的清单。
> `p11` Table 2 的 FSMC 行明确写：**Rx = No**，Vx = Yes，Zx = Yes
> （且 LQFP100/BGA100 的 FSMC 也受限：只有 Bank1 + Bank2，Bank1 仅支持复用式 NOR/PSRAM 且只能用 NE1）。
> 需要外挂并行 SRAM/NAND 时必须选 [STM32F103ZET6](STM32F103ZET6.md) 或 V 封装 —— 本库的 [通讯处理板](../../通讯处理板.md) 正是这么做的。

## 2. 极限工况

| 参数 | 符号 | 条件 | 额定值 | 单位 | 页 |
|---|---|---|---|---|---|
| 主供电电压 | VDD − VSS | 含 VDDA 与 VDD | −0.3 ~ 4.0 | V | p43 |
| 输入电压（5V 耐受脚） | VIN | FT 引脚 | VSS − 0.3 ~ VDD + 4.0 | V | p43 |
| 输入电压（其他脚） | VIN | 非 FT | VSS − 0.3 ~ 4.0 | V | p43 |
| VDD 引脚间压差 | \|ΔVDDx\| | — | ≤ 50 | mV | p43 |
| 地引脚间压差 | \|VSSX − VSS\| | 含 VREF− | ≤ 50 | mV | p43 |
| 流入 VDD/VDDA 总电流 | IVDD | — | 150 | mA | p43 |
| 流出 VSS 总电流 | IVSS | — | 150 | mA | p43 |
| 单 I/O 灌/拉电流 | IIO | 任意 I/O 与控制脚 | ±25 | mA | p43 |
| 注入电流（5V 耐受脚） | IINJ(PIN) | — | −5 / +0 | mA | p43 |
| 注入电流（其他脚） | IINJ(PIN) | — | ±5 | mA | p43 |
| 存储温度 | TSTG | — | −65 ~ +150 | °C | p44 |
| 最大结温 | TJ | — | 150 | °C | p44 |

> ⚠️ 上表为**应力额定值**，手册原文明确「不意味着器件可在该条件下工作」（p43）。
> ⚠️ ΣIINJ(PIN)（注入电流总和）在高容量手册 p43 的提取文本中被表尾截断，**未取到** —— 见 [#9. 提取完整性](../../#9.%20提取完整性.md)。

## 3. 推荐工作条件

`p44` Table 10 General operating conditions：

| 参数 | 符号 | 条件 | 最小 | 最大 | 单位 |
|---|---|---|---|---|---|
| 内部 AHB 时钟 | fHCLK | — | 0 | 72 | MHz |
| 内部 APB1 时钟 | fPCLK1 | — | 0 | 36 | MHz |
| 内部 APB2 时钟 | fPCLK2 | — | 0 | 72 | MHz |
| 标准工作电压 | VDD | — | 2 | 3.6 | V |
| 模拟工作电压 | VDDA | **不用 ADC** | 2 | 3.6 | V |
| 模拟工作电压 | VDDA | **用 ADC** | 2.4 | 3.6 | V |
| 后备工作电压 | VBAT | — | 1.8 | 3.6 | V |
| 功耗 | PD | TA = 85 °C（6 档） | — | **444**（LQFP64） | mW |
| 环境温度 | TA | 6 档，最大功耗 | −40 | 85 | °C |
| 环境温度 | TA | 6 档，低功耗态 | −40 | 105 | °C |
| 结温范围 | TJ | 6 档 | −40 | 105 | °C |
| 结温范围 | TJ | 7 档 | −40 | 125 | °C |

同页其他封装的 PD（TA = 85 °C）：LQFP144 = 666 mW · LQFP100 = 434 mW · LFBGA144 = 500 mW · LFBGA100 = 500 mW · WLCSP64 = 400 mW。

> [!warning] 用 ADC 时 VDDA 下限抬到 2.4 V
> 不用 ADC 时 VDDA 只需 2 V 且「必须与 VDD 同电位」；手册另注：建议 VDD 与 VDDA 同源，上电与工作中最多容忍 **300 mV** 压差（p44 脚注）。

## 4. 功耗

`p47` Table 14（**最大**值，代码在 Flash 中运行，外部时钟 + PLL）：

| 条件 | fHCLK | TA = 85 °C | TA = 105 °C | 单位 |
|---|---|---|---|---|
| 全部外设开启 | 72 MHz | **69** | 70 | mA |
| 全部外设开启 | 48 MHz | 50 | 50.5 | mA |
| 全部外设开启 | 8 MHz | 11 | 11.5 | mA |
| 全部外设关闭 | 72 MHz | 37 | 37.5 | mA |
| 全部外设关闭 | 8 MHz | 8 | 8 | mA |

`p47` Table 15（代码在 RAM 中运行）：72 MHz 全外设开启 66 / 67 mA，关闭 33 / 33.5 mA。

> ⚠️ 以上均为 **Max**（p47 脚注：guaranteed by characterization results）。
> 本页**未提取典型值** —— 高容量手册的典型功耗以图表给出（Figure），属 `⚠️ 图片读取` 范畴，未采信。
> 参考对照：中容量手册给出同条件典型值 36 mA @72 MHz（见 [STM32F103C8T6](STM32F103C8T6.md)，p47 Table 17）—— ⚠️ **不可跨手册套用**。

## 5. 封装与热阻

`p132` Table 74 Package thermal characteristics：

| 封装 | 尺寸 / 间距 | ΘJA | 单位 |
|---|---|---|---|
| **LQFP64**（本型号） | 10 × 10 mm / 0.5 mm | **45** | °C/W |
| LQFP100 | 14 × 14 mm / 0.5 mm | 46 | °C/W |
| LQFP144 | 20 × 20 mm / 0.5 mm | 30 | °C/W |
| LFBGA144 | 10 × 10 mm / 0.8 mm | 40 | °C/W |
| LFBGA100 | 10 × 10 mm / 0.8 mm | 40 | °C/W |
| WLCSP64 | — | 50 | °C/W |

手册给出的结温计算式（p132）：

```
TJmax = TAmax + (PDmax × ΘJA)
PDmax = PINTmax + PI/Om​ax
PINTmax = IDD × VDD
PI/Om​ax = Σ(VOL × IOL) + Σ((VDD − VOH) × IOH)
```

## 6. 引脚定义核对（LQFP-64）

手册 `p31` Table 5 的 LQFP64 列，与工程网表里嘉立创符号的引脚号**逐行一致**：

| LQFP64 脚 | 名称 | 手册页 | 网表实测 |
|---:|---|---|---|
| 1 | VBAT | p31 | `+3.3V`（5 板均接主电源） |
| 2 | PC13-TAMPER-RTC | p31 | 未接 / 悬空标签 |
| 3 | PC14-OSC32_IN | p31 | `OSC32_IN` |
| 4 | PC15-OSC32_OUT | p31 | `OSC32_OUT` |
| 5 | PD0-OSC_IN | p31 | `OSC_IN` |
| 6 | PD1-OSC_OUT | p31 | `OSC_OUT` |
| 7 | NRST | p31 | `NRST` |
| 8–11 | PC0 / PC1 / PC2 / PC3 | p32 | 端口用途见 §8 |
| 12 | **VSSA** | p32 | `AGND` / `GND` |
| 13 | **VDDA** | p32 | `+3.3V` |

> [!tip] 这条核对的意义
> 网表里的 `U2-12` 之所以能确定是 VSSA，是因为**嘉立创 EDA 符号的引脚编号与实物封装一致**（手册 p31/p32 逐行验证）。
> 这一点让所有项目页的引脚级结论（含 [电机板](../../电机板.md) 的 VBAT 缺失发现）都建立在实际封装引脚上，而不是符号自定义编号。

## 7. 项目用量（网表）

| 工程 | 位号 | 角色 | 该板规模 |
|---|---|---|---|
| [电池主控PCB](../../电池主控PCB.md) | U2 | 主控 | 157 元件 |
| [电池控制板](../../电池控制板.md) | U2 | 主控 | 139 元件 |
| [4G通讯模块](../../4G通讯模块.md) | U2 | 主控（配合 [AIR724UG](AIR724UG.md)） | 141 元件 |
| [IoT traffic lights](../../IoT%20traffic%20lights.md) | U27 | 主控（配合 [AIR724UG](AIR724UG.md)） | 194 元件 |
| [Wifi通讯模块](../../Wifi通讯模块.md) | U33 | 主控（配合 [ESP32-C3-MINI-1](ESP32-C3-MINI-1.md)） | 47 元件 |

**5 个工程 / 5 颗**。以下用法来自这 5 块板的网表交叉比对。

## 8. 跨工程一致的引脚用法

| 引脚 | 网络 | 外围 | 出现工程数 |
|---|---|---|---:|
| 5 | `OSC_IN` | 8 MHz 晶振 + 起振电阻 R51 + 负载电容 C48 | 5 |
| 6 | `OSC_OUT` | 同上 | 5 |
| 7 | `NRST` | C43 去耦 + R50 上拉 + 复位按键 | 5 |
| 1 | `+3.3V`（VBAT） | 直接接主电源，**未用独立纽扣电池** | 5 |
| 3 | `OSC32_IN` | 32.768 kHz RTC 晶振 Y2 + C50 | 3 |
| 4 | `OSC32_OUT` | 同上 | 3 |
| 12 | `AGND` / `GND`（VSSA） | [电池控制板](../../电池控制板.md) 中单独成网 | 1（显式） |

> [!note] 8 MHz + 32.768 kHz 双晶振是标准配置
> 5 块板无一例外都用了 8 MHz 主晶振 + 32.768 kHz RTC 晶振，起振电阻统一 R51、复位网络统一 R50 + C43
> —— 同一份最小系统原理图被复制到 5 个工程的直接证据。

各工程差异化的端口用法：

| 工程 | 引脚 → 用途（网表可证） |
|---|---|
| [电池主控PCB](../../电池主控PCB.md) | `PC1`(9) `PC2`(10) `PC3`(11) `PC4`(24) `PC5`(25) `PB1`(27) `PB10`(29) → 分别接 6 路 XH2.54-2P 端子（U22–U27） |
| [电池控制板](../../电池控制板.md) | `PC1`–`PC5` `PB1` → 6 路 XH2.54-2P 端子（U28–U33）；**与电池主控PCB 引脚完全一致** |
| [4G通讯模块](../../4G通讯模块.md) | `PA5`(21) → 经 TXS0108E 电平转换进 AIR724UG；`PB1`(27) → KEY3 |
| [IoT traffic lights](../../IoT%20traffic%20lights.md) | `PA5` 类引脚经 TXS0104E 接 AIR724UG；`PB1` → KEY3 |
| [Wifi通讯模块](../../Wifi通讯模块.md) | `PA5`(21) `PA6`(22) → ESP32-C3 `IO20/IO21`；`PA0`(14) → ESP32-C3 `IO6`；`PA9`(42) `PA10`(43) → H6 排针；`pin16` → ESP32-C3 `BOOT` |

> [!tip] [Wifi通讯模块](../../Wifi通讯模块.md) 的用法最值得注意
> `BOOT` 直连 `U33-16`，配合 `EN-ESP` 由电阻网络控制 —— **主控能自动复位并让 ESP32-C3 进入烧录模式**，省掉手动按键。

## 9. 提取完整性

| 类别 | 请求项 | 已提取 | 存疑 | 未找到 |
|---|---|---|---|---|
| 身份 / 订购解码 | 7 | 7 | 0 | 0 |
| 绝对最大额定值 | 13 | 12 | 0 | 1（ΣIINJ 被表尾截断） |
| 推荐工作条件 | 12 | 12 | 0 | 0 |
| 功耗 | 8 | 8 | 0 | 0（典型值以图给出，未提取） |
| 封装 / 热参数 | 7 | 6 | 0 | 1（封装机械尺寸未提取） |
| I/O 电气特性（VIL/VIH/VOL/VOH） | 8 | 0 | 0 | 8（未提取） |
| 时钟 / ADC / 时序 | — | 0 | — | 未提取 |

⚠️ 未提取项需按需回手册补充（该手册 143 页，本页只覆盖选型与工况所需部分）。

## 10. 参见

- [STM32F103C8T6](STM32F103C8T6.md) — 同系列 LQFP-48 中容量版（**无 FSMC**）
- [STM32F103ZET6](STM32F103ZET6.md) — 同系列 LQFP-144 版（同一份手册）
- [AIR724UG](AIR724UG.md) · [ESP32-C3-MINI-1](ESP32-C3-MINI-1.md) — 本 MCU 直接对接的无线模块
- [电池主控PCB](../../电池主控PCB.md) · [电池控制板](../../电池控制板.md) · [4G通讯模块](../../4G通讯模块.md) · [IoT traffic lights](../../IoT%20traffic%20lights.md) · [Wifi通讯模块](../../Wifi通讯模块.md) — 使用本器件的 5 个工程
- [元件总表](../../元件总表.md)
