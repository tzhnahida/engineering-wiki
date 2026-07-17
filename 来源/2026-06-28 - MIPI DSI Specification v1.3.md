---
type: source
tags: [mipi, dsi, display, serial-protocol, standard]
created: 2026-06-28
updated: 2026-06-28
---

# MIPI DSI Specification v1.3

## 基本信息

| 属性 | 值 |
|------|-----|
| **标题** | MIPI Alliance Specification for Display Serial Interface (DSI) |
| **版本** | v1.3 (23-March-2015) |
| **通过日期** | Board Adopted 10 March 2015 |
| **组织** | MIPI Alliance, Inc. |
| **版本历史** | v1.00a (2006) → v1.01 (2008) → v1.02 (2010) → v1.1 (2012) → v1.2 (2014) → v1.3 (2015) |
| **文件** | `参考/标准/MIPI_DSI_specification_Version_1.3.pdf` |

## 概述

DSI 规范定义了主机处理器与显示模组之间的高速串行通信协议。构建在 D-PHY（或 C-PHY）物理层之上，定义了四层协议栈：PHY 层、Lane Management 层、Low Level Protocol (LLP) 层和 Application 层。

v1.3 是当前广泛使用的主流版本，新增 Sub-Link（分割链路/Deskew）和 DSC（Display Stream Compression）支持。

## 关键内容

- **协议分层**：PHY → Lane Management → LLP → Application 四层架构
- **包格式**：短包 (4 bytes: DI+Data+ECC) 和长包 (6-65541 bytes: DI+WC+Payload+Checksum)
- **Data Identifier (DI)**：VC[7:6] + DT[5:0]，编码虚拟通道和数据类型
- **Video Mode**：Non-Burst（Sync Pulse/Events）和 Burst Mode 三种传输模式
- **Command Mode**：面板侧 GRAM + TE 撕裂防护
- **虚拟通道**：最多 4 个 VC，单链路复用多外设
- **ECC 与 Checksum**：汉明码 (24,16) ECC + CRC-16 校验
- **Sub-Link**（v1.3）：多 DSI 链路组合 + Deskew 偏移补偿
- **DSC**（v1.3）：VESA Display Stream Compression 集成
- **像素格式**：RGB 5-6-5/6-6-6/8-8-8, YUV 4:2:0/4:2:2, Compressed
- **错误处理**：SoT Error、Sync Error、Contention Detection、超时机制

## 产生的知识页

- [视频显示/MIPI DSI](../视频显示/MIPI DSI.md) — DSI 协议知识页
- [视频显示/MIPI 概述](../视频显示/MIPI 概述.md) — MIPI 概述（引用 DSI）
- [视频显示/MIPI DCS](../视频显示/MIPI DCS.md) — DSI 承载 DCS 命令
- [视频显示/MIPI DBI](../视频显示/MIPI DBI.md) — DSI 的前身（DBI 并行接口）
- [视频显示/MIPI DPI](../视频显示/MIPI DPI.md) — DSI Video Mode 的原型

## 相关标准

- [2026-06-28 - MIPI D-PHY Specification v2.5](2026-06-28 - MIPI D-PHY Specification v2.5.md) — DSI 的物理层基础
- [2026-06-28 - MIPI DCS Specification v1.02](2026-06-28 - MIPI DCS Specification v1.02.md) — DSI 承载的命令集
