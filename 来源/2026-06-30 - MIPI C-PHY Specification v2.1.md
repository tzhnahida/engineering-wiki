---
type: source
tags: [mipi, c-phy, physical-layer, serial-interface, standard]
created: 2026-06-30
updated: 2026-06-30
---

# 2026-06-30 - MIPI C-PHY Specification v2.1

> **文档**: `MIPI_C-PHY_specification_v2-1.pdf` (272 pages)
> **来源**: MIPI Alliance, Version 2.1, 01-Apr-2021 (Board Approved: 21-Jul-2021)
> **存放**: [[参考/标准/MIPI_C-PHY_specification_v2-1.pdf]]

## 概述

MIPI C-PHY 是 MIPI 联盟定义的高速串行物理层规范，基于**三线三电平（3-Phase）符号编码**，每符号传输约 2.28 bits，专为带宽受限通道（如显示 CoG、图像传感器）设计。C-PHY 复用 D-PHY 的低功耗模式，可与 D-PHY 共用同一 IC 引脚。

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 07-Oct-2014 | 初始 Board Approved Release |
| v1.1 | 11-Feb-2016 | Board Approved Release |
| v1.2 | 28-Mar-2017 | Board Approved Release |
| v2.0 | 09-Sep-2019 | Board Approved Release |
| v2.1 | 21-Jul-2021 | Board Approved Release（本文档） |

## 关键内容

- **Ch.1** — Introduction & Scope：3-Phase 编码，2.28 bits/symbol，与 D-PHY 共存
- **Ch.4** — C-PHY Overview：六种 Wire State (±x/±y/±z)、嵌入式时钟、三差分接收器
- **Ch.5** — Architecture：Universal Lane Module (HS-TX/RX, LP-TX/RX, LP-CD, CIL)、双向/单向 Lane
- **Ch.6** — Global Operation：16-bit→7 symbol Mapping、Flip/Rotation/Polarity 编码、HS Burst 结构、ALP Mode、Escape Mode
- **Ch.7** — Fault Detection：冲突检测、序列错误检测、协议看门狗
- **Ch.8** — Interconnect：100Ω 差分 / 50Ω 单端、S-parameter 模板
- **Ch.9** — Electrical Characteristics：HS/LP 发送器接收器电参数
- **Ch.10** — Timing：时序参数表、眼图模板

## 关联知识

- [[视频显示/MIPI C-PHY]] — 知识页
- [[视频显示/MIPI D-PHY]] — 同类物理层对比
- [[视频显示/MIPI DSI]] — 上层协议（可用 C-PHY 作为 PHY）
- [[视频显示/MIPI 概述]] — MIPI 家族全景
