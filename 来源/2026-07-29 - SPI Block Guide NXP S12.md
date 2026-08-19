---
type: source
tags: [spi, serial, nxp, freescale, motorola, reference]
created: 2026-07-29
updated: 2026-07-29
---

# 2026-07-29 - SPI Block Guide (NXP S12)

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | SPI Block Guide V03.06 — S12 Family |
| 日期 | 2008-10 (Freescale) |
| 来源 | `参考/标准/SPI_Block_Guide_S12_NXP.pdf` |
| 发布者 | NXP Semiconductors (原 Freescale / Motorola) |
| 许可 | 免费公开发布（应用笔记） |

## 文档概述

SPI (Serial Peripheral Interface) 没有独立的正式标准——Motorola 从未发布单行本 SPI 规范。该协议定义分散在 Motorola 各系列 MCU 参考手册中。此 NXP S12 系列 SPI Block Guide 是嵌入式开发中最接近"SPI 规范"的参考文档，涵盖寄存器定义、传输格式（CPOL/CPHA）、主从模式和时序。

## 关键知识点

- **四线制**：SCK (时钟) + MOSI (主出从入) + MISO (主入从出) + SS (从机选择)
- **全双工**：主从同时发送和接收（移位寄存器环）
- **CPOL (Clock Polarity)**：空闲时 SCK 电平，0=低，1=高
- **CPHA (Clock Phase)**：0=第一沿采样，1=第二沿采样
- **四种模式**：Mode 0 (CPOL=0,CPHA=0) 最常用 → 空闲低，上升沿采样
- **主从架构**：主机产生 SCK，`/SS` 拉低选择从机
- **无标准流控**：无 ACK 机制，无内置寻址——完全依赖片选
- **速度**：实际可达数十 MHz，受限于 PCB 布线和芯片 IO

## 注意

SPI 是事实标准 (de facto standard)，并非由标准组织（如 IEEE、ISO）发布。不同芯片厂商的 SPI 实现可能有寄存器级差异，但总线时序互操作。

## 关联的 Wiki 页面

- [嵌入式软件/嵌入式固件开发流程](../知识/嵌入式软件/嵌入式固件开发流程.md) — SPI Flash 启动与固件加载
