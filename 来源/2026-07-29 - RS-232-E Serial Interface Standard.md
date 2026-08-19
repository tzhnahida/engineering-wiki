---
type: source
tags: [uart, serial, rs-232, eia, tia, standard]
created: 2026-07-29
updated: 2026-07-29
---

# 2026-07-29 - RS-232-E Serial Interface Standard

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | EIA RS-232-E — Interface Between DTE and DCE |
| 日期 | 1991-07 (Revision E) |
| 来源 | `参考/标准/RS-232-E_Standard.pdf` |
| 发布者 | Electronic Industries Association (EIA)，现 TIA |
| 许可 | 存档副本（原标准需向 TIA 购买，~$100） |

## 文档概述

RS-232 (现 TIA-232) 是串行通信物理层最经典的标准，定义了 DTE (数据终端设备) 和 DCE (数据电路设备) 之间的电气特性、信号功能和连接器引脚。UART 协议层本身没有统一标准——RS-232 覆盖了物理层；协议层（起始位/数据位/校验/停止位）依赖各芯片实现。

> ⚠️ **注意**：RS-232 定义了 ±3V~±15V 电平（-3V~-15V = mark/1，+3V~+15V = space/0），与 MCU 的 TTL/CMOS 电平 UART (0~3.3V/5V) 不同。需 MAX232 等电平转换芯片互联。

## 关键知识点

- **信号电平**：Mark (逻辑 1) = −3V~−15V，Space (逻辑 0) = +3V~+15V
- **25-pin D-sub (DB-25)**：RS-232 原始定义；9-pin (DE-9) 由 TIA-574 补充
- **核心信号**：TXD (2/3)、RXD (3/2)、RTS (4/7)、CTS (5/8)、DTR (20/4)、DSR (6/6)、DCD (8/1)、RI (22/9)
- **流控**：RTS/CTS 硬件流控、XON/XOFF 软件流控
- **速率**：典型 ≤115,200 bps（标准定义 ≤20 kbps，但实际远超）
- **UART 帧格式**：Start Bit (1) + Data Bits (5-9) + Parity (None/Odd/Even) + Stop Bits (1/1.5/2)

## 关联的 Wiki 页面

- [嵌入式软件/MCU裸机软件分层架构](../知识/嵌入式软件/MCU裸机软件分层架构.md) — UART 驱动层
- [嵌入式软件/MCU 固件升级 IAP OTA 实战](../知识/嵌入式软件/MCU%20固件升级%20IAP%20OTA%20实战.md) — 串口 IAP 升级
