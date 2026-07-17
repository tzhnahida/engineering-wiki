---
type: source
tags: [mipi, d-phy, physical-layer, standard]
created: 2026-06-28
updated: 2026-06-28
---

# MIPI D-PHY Specification v2.5

## 基本信息

| 属性 | 值 |
|------|-----|
| **标题** | MIPI Alliance Specification for D-PHY |
| **版本** | v2.5 (05-Jul-2019) |
| **通过日期** | Board Adopted 17 October 2019 |
| **组织** | MIPI Alliance, Inc. |
| **前身** | v1.0 (2009) → v1.2 (2014) → v2.0 (2016) → v2.1 (2017) |
| **文件** | `参考/标准/MIPI_D-PHY_specification_Version_2.5.pdf` |

## 概述

D-PHY 规范定义了 MIPI 联盟的源同步高速串行物理层，为 DSI（显示）和 CSI-2（摄像头）等上层协议提供数据传输服务。核心设计采用双模信号：HS（差分 LVDS，最高 4.5 Gbps/通道）和 LP（单端 CMOS，≤10 Mbps）。

v2.5 是当前最新版本，新增 8b9b Line Coding 和光互联支持。

## 关键内容

- **Lane 模块架构**：Universal Lane Module、Clock Lane、Unidirectional/Bi-directional Data Lane
- **双模信号**：HS 差分 LVDS（140-270 mV）和 LP 单端 CMOS（1.2V）
- **HS Burst 传输**：SoT (Start-of-Transmission) → Sync → Payload → EoT (End-of-Transmission)
- **Escape Mode**：LPDT、ULPS、Trigger、Remote Reset、Tearing Effect、Acknowledge 子模式
- **BTA (Bus Turnaround)**：双向 Data Lane 的方向反转机制
- **ALP Mode**（v2.0+）：替代传统 LP 的低开销差分模式
- **Deskew Calibration**：补偿多 Lane 间偏移
- **8b9b Line Coding**（v2.5 新增）
- **电气规范**：HS-TX/RX、LP-TX/RX、时序参数、眼图模板

## 产生的知识页

- [视频显示/MIPI D-PHY](../视频显示/MIPI D-PHY.md) — 物理层知识页
- [视频显示/MIPI 概述](../视频显示/MIPI 概述.md) — MIPI 概述（引用 D-PHY）
- [视频显示/MIPI DSI](../视频显示/MIPI DSI.md) — DSI 协议层（基于 D-PHY）

## 相关标准

- [2026-06-28 - MIPI DSI Specification v1.3](2026-06-28 - MIPI DSI Specification v1.3.md) — DSI 基于 D-PHY
- [2026-06-28 - MIPI DCS Specification v1.02](2026-06-28 - MIPI DCS Specification v1.02.md)
