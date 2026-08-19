---
type: source
tags: [i2c, iic, specification, nxp, standard, protocol]
created: 2026-07-29
updated: 2026-07-29
---

# 2026-07-29 - I²C Bus Specification UM10204

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | I²C-bus specification and user manual (UM10204) |
| 版本 | Rev. 7 (2021) |
| 来源 | `参考/标准/I2C-Bus_Specification_UM10204.pdf` |
| 发布者 | NXP Semiconductors (原 Philips Semiconductors) |
| 许可 | 免费公开发布 |

## 文档概述

I²C (Inter-Integrated Circuit) 总线规范，由 Philips 于 1982 年发明。定义了双线串行总线（SCL + SDA）的电气特性、时序、寻址、仲裁和协议层。最新 Rev. 7 版本涵盖 Standard (100 kbps)、Fast (400 kbps)、Fast+ (1 Mbps)、High-speed (3.4 Mbps) 和 Ultra Fast (5 Mbps) 五种速率模式。

## 关键知识点

- **两线制**：SCL (时钟) + SDA (数据)，开漏输出 + 上拉电阻
- **多主多从**：任意节点可发起传输，基于 SDA 线的逐位仲裁 (wired-AND)
- **7-bit / 10-bit 寻址**：7-bit 地址 112 个可用，10-bit 扩展地址
- **START / STOP 条件**：SCL 高时 SDA 下降沿 = START，上升沿 = STOP
- **ACK / NACK**：第 9 个时钟 SCL 时接收方拉低 SDA 确认
- **时钟拉伸** (Clock Stretching)：从机可拉低 SCL 延迟主机的下一时钟
- **Ultra Fast Mode**：单向推挽输出 (仅写)，最高 5 Mbps
- **电平兼容**：1.8V / 2.5V / 3.3V / 5V 均可，取决于上拉电压

## 关联的 Wiki 页面

- [嵌入式软件/MCU裸机软件分层架构](../知识/嵌入式软件/MCU裸机软件分层架构.md) — I²C 外设驱动层
- [视频显示/HDMI EDID](../知识/视频显示/HDMI%20EDID.md) — DDC 通道基于 I²C
