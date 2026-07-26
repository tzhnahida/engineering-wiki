---
type: concept
tags: [嵌入式系统, 调试接口, JTAG, SWD, 硬件]
created: 2026-07-24
updated: 2026-07-24
sources: ["[[2026-07-24 - JTAG SWD 调试接口图解]]"]
---

# JTAG/SWD 调试接口

JTAG 和 SWD 是嵌入式开发中最核心的调试与固件下载接口。JTAG（IEEE 1149.1）最初为芯片测试而生，后演变为通用调试标准；SWD 是 ARM 在其基础上提出的两线精简版。两者共同支撑从 IDE 到目标芯片的完整调试链路。

## 调试链路架构

```
IDE (Keil/IAR/OpenOCD) → USB → Debugger (J-Link/ST-Link/CMSIS-DAP) → JTAG/SWD → Target MCU
```

调试器的本质是 **USB 命令 → JTAG/SWD 时序** 的精确翻译器。它不执行调试逻辑，只负责在正确的时间点产生正确的电平跳变。

> ⚠️ 图片读取：原文含 6 张图解，本页信息基于文字描述整理，图片内容未经视觉确认。

## JTAG：五根信号线

标准 JTAG 接口使用四根必需信号加一根可选复位：

| 信号 | 方向 | 功能 |
|------|------|------|
| **TCK** | 输入 | 测试时钟，驱动整个调试链路的时序基准 |
| **TMS** | 输入 | 测试模式选择，通过电平变化驱动 TAP 状态机切换 |
| **TDI** | 输入 | 测试数据输入，串行数据进入目标芯片 |
| **TDO** | 输出 | 测试数据输出，串行数据从目标芯片返回 |
| **TRST** | 输入（可选） | 测试复位，将 TAP 状态机强制拉回初始状态 |

### TAP 状态机

TAP（Test Access Port）状态机是 JTAG 的核心，由 TMS 信号驱动。共有 16 个状态，关键路径包括：

- **Shift-IR / Shift-DR**: 分别在指令寄存器/数据寄存器链上移入移出数据
- **Capture-IR / Capture-DR**: 将当前寄存器值捕获到移位链
- **Update-IR / Update-DR**: 将移位完成的值锁存生效

每次 TCK 上升沿，TMS 的电平（0/1）决定下一状态。调试器通过在正确的时钟边沿置正确的 TMS 电平来遍历状态机。

### 菊花链 (Daisy Chain)

JTAG 支持多芯片菊花链：多颗芯片的 TDI→TDO 首尾相接，共用一组 TCK/TMS。调试器通过 TDI 串入指令/数据，从 TDO 串出结果，一组信号即可访问链上所有器件。

## 边界扫描 (Boundary Scan)

> ⚠️ 图片读取：图 3 边界扫描示意图未视觉验证

这是 JTAG 的"老本行"——芯片测试。每个芯片引脚旁内置一个**边界扫描单元（BSC, Boundary Scan Cell）**：

- **EXTEST 模式**: BSC 可捕获或驱动对应引脚的电平，无需物理探针
- **检测能力**: PCB 走线的开路、短路、虚焊
- **关键价值**: BGA 封装引脚藏在芯片底部，飞针都无法接触——边界扫描几乎是产线测试的唯一手段

这是 JTAG 至今不可替代的工业价值。

## SWD：两线替代五线

对于引脚紧张的 MCU，JTAG 占五个引脚是奢侈的。ARM 的 Serial Wire Debug（SWD）将其精简到两根：

| 信号 | 功能 |
|------|------|
| **SWCLK** | 串行时钟 |
| **SWDIO** | 双向串行数据（一条线实现收发） |

SWD 用两根线实现了与 JTAG 几乎等价的调试能力：暂停、单步、读写寄存器、烧录 Flash 一应俱全。省下的三个引脚对小封装芯片（如 QFN、WLCSP）至关重要。

> 如今绝大多数 Cortex-M 芯片把 SWD 作为首选调试接口，许多调试器支持 JTAG/SWD 自动切换。

## SWO / SWV：实时跟踪

在 SWD 基础上多加一根 **SWO**（Serial Wire Output）线，就能实现低开销的实时跟踪：

- **ITM**（Instrumentation Trace Macrocell）单元把 printf 输出、事件、异常、计数器等信息，经 SWO 单向"流"给上位机 IDE
- 配合 IDE 的 **SWV**（Serial Wire Viewer）功能，可视化程序运行时行为
- **不占用串口**、几乎不打断程序运行——特别适合调试实时性强的代码（电机控制、通信协议栈等）

## 典型调试会话流程

1. Connection detection: handshake (read IDCODE)
2. Halt core
3. Set breakpoints/single-step to observe execution
4. Read/write registers and memory (diagnose issues)
5. Download firmware to on-chip Flash

无论底层走 JTAG 还是 SWD，这套"连接→暂停→观察→烧录"的流程一致。

## 板级设计要点

### 接口选择

| 场景 | 推荐 |
|------|------|
| Cortex-M 单芯片调试 | **SWD**（SWCLK + SWDIO） |
| 需要边界扫描测试 | JTAG（五线） |
| 多核/多芯片链 | JTAG（菊花链） |

### 设计检查清单

| 要点 | 说明 |
|------|------|
| **务必引 SWO** | 多一根线就能做实时 printf 跟踪，省掉一个串口 |
| **标准排针** | 10-pin（Cortex-M 常见）或 20-pin（ARM 标准），注意第 1 脚方向标记 |
| **引出 nRESET** | 硬件复位线有助于稳定连接，尤其在芯片已进入低功耗模式时 |
| **电平匹配** | 调试器与目标 I/O 电平需一致（1.8V / 3.3V）；支持电平转换的调试器（如 J-Link）可自适应 |
| **SWDIO/TMS 上拉** | 按芯片手册加合适的上拉/下拉电阻，防止信号浮空 |
| **走线** | 调试线走短、远离强干扰源（DC-DC 电感、PWM 功率走线） |
| **量产保护** | 量产阶段可使能读保护（RDP）锁定调试口，防止固件被读出；但要保留解锁/ISP 手段用于返修 |

## 调试器选型速查

| 调试器 | 协议 | 特点 |
|--------|------|------|
| J-Link | JTAG + SWD + SWO | 业界标准，高速，商用 |
| ST-Link | SWD + SWO | STM32 专用，免费 |
| CMSIS-DAP | SWD + JTAG | ARM 开源标准，低成本 |
| OpenOCD | JTAG + SWD | 开源软件，适配多种调试器硬件 |

## 提取完整性

| 类别 | 来自原文 | 说明 |
|------|----------|------|
| JTAG 五线定义 | ✅ 文字提取 | TCK/TMS/TDI/TDO/TRST 完整 |
| TAP 状态机 | ✅ 文字提取 | 16 状态，Shift/Capture/Update 路径 |
| 边界扫描 BSC | ✅ 文字提取 | EXTEST、PCB 检测、BGA 价值 |
| SWD 两线 | ✅ 文字提取 | SWCLK+SWDIO，等价 JTAG 能力 |
| SWO/SWV | ✅ 文字提取 | ITM 实时跟踪，不占串口 |
| 调试流程 | ✅ 文字提取 | 连接→握手→Halt→观察→烧录 |
| 设计检查清单 | ✅ 文字提取 | 7 条要点 |
| 调试器选型 | ✅ 文字提取 | J-Link/ST-Link/CMSIS-DAP/OpenOCD |
| 图解（图 1–6） | ⚠️ 图片读取 | 文字描述已涵盖，图片内容未视觉验证 |
| 具体时序参数 | ⚠️ 未找到 | 原文为概念性图解，未给出频率/建立时间等参数 |
