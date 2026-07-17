---
type: source
tags: [mipi, dcs, display, command-set, standard]
created: 2026-06-28
updated: 2026-06-28
---

# MIPI DCS Specification v1.02

## 基本信息

| 属性 | 值 |
|------|-----|
| **标题** | MIPI Alliance Specification for Display Command Set (DCS) |
| **版本** | v1.02.00 (23-July-2009) |
| **通过日期** | Board Approved 24 November 2009 |
| **组织** | MIPI Alliance, Inc. |
| **文件** | `参考/标准/MIPI_DCS_specifation_Version_1.02.pdf` |

## 概述

DCS 规范定义了控制显示模组的标准化命令集。无论底层使用 DBI（并行）还是 DSI（串行）接口，DCS 命令码都保持一致，实现了面板驱动的跨平台兼容。

## 关键内容

- **三种显示架构**：Type 1（全帧 GRAM）、Type 2（部分 GRAM + 视频流）、Type 3（无 GRAM，实时刷新）
- **电源状态机**：Sleep ↔ Normal（含 Partial/Idle 子状态）
- **关键时序**：exit_sleep_mode 后等待 5ms + 120ms；set_display_on 后等待 120ms
- **伽马曲线**：GC0 (2.2)、GC1 (1.8)、GC2 (2.5)、GC3 (Linear)
- **自诊断框架**：Register Loading、Functionality、Chip Attachment、Glass Break 检测
- **命令分类**：电源控制、像素格式、扫描方向、伽马校正、色彩反转、状态读取

## DCS 命令速查（核心）

| 命令 | 码 | 说明 |
|------|:--:|------|
| `enter_sleep_mode` | 0x10 | 进入睡眠 |
| `exit_sleep_mode` | 0x11 | 退出睡眠（延时 120ms） |
| `set_display_off` | 0x28 | 关闭显示 |
| `set_display_on` | 0x29 | 开启显示 |
| `set_pixel_format` | 0x3A | 设置像素格式 |
| `set_address_mode` | 0x36 | 扫描方向 / RGB-BGR |
| `set_column_address` | 0x2A | 列地址 |
| `set_page_address` | 0x2B | 行地址 |
| `write_memory_start` | 0x2C | 写入帧缓存 |

## 产生的知识页

- [视频显示/MIPI DCS](视频显示/MIPI DCS.md) — DCS 知识页
- [视频显示/MIPI DSI](视频显示/MIPI DSI.md) — DSI 承载 DCS 命令
- [视频显示/MIPI DBI](视频显示/MIPI DBI.md) — DBI 承载 DCS 命令

## 相关标准

- [2026-06-28 - MIPI DSI Specification v1.3](2026-06-28 - MIPI DSI Specification v1.3.md)
- [2026-06-28 - MIPI D-PHY Specification v2.5](2026-06-28 - MIPI D-PHY Specification v2.5.md)
