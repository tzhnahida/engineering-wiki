---
type: source
tags: [通讯, 电力线通信, 数据手册]
created: 2026-08-03
updated: 2026-08-03
---

# 2012-03-16 - QCA6410 HomePlug AV 数据手册

> Qualcomm Atheros QCA6410 HomePlug AV MAC/PHY Transceiver 数据手册。39 页，Ver. 5.0。

## 文档信息

| 项目 | 内容 |
|------|------|
| 器件 | QCA6410 |
| 厂商 | Qualcomm Atheros (Qualcomm) |
| 类型 | HomePlug AV / IEEE 1901 MAC/PHY SoC 收发器 |
| 文档版本 | Ver. 5.0, March 16, 2012 |
| 文档页数 | 39 |
| PDF | `Chip_PLC_QCA6410.PDF` |

## 核心亮点

- IEEE 1901 及 HomePlug AV 全兼容单芯片 SoC
- 集成 10/100 802.3az Ethernet PHY（嵌入式以太网 MAC+PHY）
- 集成模拟前端 (AFE)：ADC/DAC、接收放大器、发射放大器、线路驱动
- 内置 ARM926 CPU + 片上 SRAM，FW 从 SPI Flash 或以太网加载
- 集成 1.2V 开关稳压器，单 3.3V 供电即可工作
- 68-pin QFN 8×8mm 封装，支持低功耗模式（Active/Standby/Sleep/Deep Sleep）
- SPI Master 接口专用于外部 NVM Flash 固件加载
- UART 主机接口 + 4 路 GPIO
- 零交叉检测器 (Zero Cross Detector)，同步电力线 50/60Hz 周期
- 200 Mbps PHY 速率

## 关键规格

| 参数 | 值 |
|------|-----|
| 核心电压 VDD | 1.2V (1.14–1.26) |
| I/O 电压 VDDIO | 3.3V (3.13–3.46) |
| 发射功耗 | ~2100 mW |
| 接收功耗 | ~1100 mW |
| 晶振频率 | 25 MHz ±10 ppm |
| SPI Flash 时钟 | 最高 40 MHz (25ns 周期) |
| 封装 | 68-QFN 8×8 mm |
| 结温 Tj(max) | +125°C |

## 章节概览

| 章节 | 内容 |
|------|------|
| §1 | 总体描述、特性、应用、功能框图 |
| §2 | 引脚定义（68 引脚完整描述）、系统介绍、零交叉、上电配置 strap |
| §3 | 系统架构：SPI Master、UART、GPIO |
| §4 | 功能描述：电源管理（4 种功耗状态）、启动选项、串行接口 |
| §5 | 电气特性：绝对最大值、推荐工作条件、功耗、DC 阈值、晶振规格 |
| §6 | 时序约束：时钟/复位、零交叉检测、SPI Flash 时序 |
| §7 | 封装尺寸 68-QFN 8×8mm |
| §8 | 订购信息 QCA6410-AL3C |

## 参见

- [QCA6410](QCA6410.md) — 元件页
- [HomePlug AV 电力线通信](HomePlug%20AV%20电力线通信.md) — 知识页
- [Ethernet 协议概述](Ethernet%20协议概述.md) — 802.3 以太网基础
