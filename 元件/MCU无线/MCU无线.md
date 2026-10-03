---
type: index
created: 2026-06-12
updated: 2026-09-30
---

# MCU 无线

| 型号 | 内核 | 频率 | 特性 |
|------|------|------|------|
| [STM32G431](STM32G431.md) | Cortex-M4 + FPU | 170MHz | DSP, CAN-FD, HRTIM |
| [STM32F103C8T6](STM32F103C8T6.md) | Cortex-M3 | 72MHz | CAN, SPI, I²C, USB, ¥3.6；**手册已补**（DS5319，64/128KB Flash、20KB SRAM、10ch ADC） |
| [STM32F103RCT6](STM32F103RCT6.md) | Cortex-M3 | 72MHz | LQFP-64，本库复用度最高（5 个工程）；256KB Flash / 48KB SRAM，**R 封装无 FSMC** |
| [STM32F103ZET6](STM32F103ZET6.md) | Cortex-M3 | 72MHz | LQFP-144，**带 FSMC** 外扩 SRAM；512KB Flash / 64KB SRAM |
| [ESP32-S3](ESP32-S3.md) | Xtensa LX7 双核 | 240MHz | WiFi 4, BLE 5, DVP |
| [ESP32-C6](ESP32-C6.md) | RISC-V 32bit | 160MHz | WiFi 6, BLE 5, Zigbee, Thread |
| [ESP32-C3-MINI-1](ESP32-C3-MINI-1.md) | RISC-V 单核 | 160MHz | WiFi 4, BLE 5，模组形态；**⛔ -N4 档 NRND**（替代 N4X） |
| [ESP8266EX](ESP8266EX.md) | Tensilica L106 | 80MHz | WiFi 4, 经典 IoT |
| [AIR724UG](AIR724UG.md) | — | — | LTE Cat.1 蜂窝模组；FDD B1/3/5/8 + TDD B34/38/39/40/41，3.3~4.3V（典型 3.8V），**电源峰值 1A / 平均 0.7A**，**5 路 UART**，SPI Camera（可直挂 GC0310），支持 WiFi Scan/定位 |

> [!note] 手册状态（2026-09-30 更新）
> 本分类下的 STM32F103 三页、ESP32-C3-MINI-1、AIR724UG **均已补齐手册并带页码引用**。
> 仍待补的是「发射峰值电流」（[AIR724UG](AIR724UG.md)，需合宙硬件设计手册）与模组脚位（[GC9A01](../音频显示/GC9A01.md) 的 12 脚模组）。
