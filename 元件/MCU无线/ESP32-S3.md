---
type: entity
tags: [esp32, mcu, wifi, bluetooth, xtensa, iot]
created: 2026-06-24
sources:
  - "F:/Projects/PCB/Library/Candidate/esp32-s3_datasheet_cn.pdf"
---

# ESP32-S3

> Espressif · Xtensa LX7 双核 240MHz · WiFi 4 + BLE 5.0 · 动捕主节点网关/眼追主控
> Datasheet: `F:\Projects\PCB\Library\Candidate\esp32-s3_datasheet_cn.pdf`

## 关键规格

| 参数 | 值 |
|------|----|
| CPU | Xtensa LX7 双核 240 MHz |
| SRAM | 512 KB |
| PSRAM | 可选 2/8 MB (Octal SPI) |
| Flash | 最大 16 MB (Quad SPI) |
| WiFi | 802.11 b/g/n (2.4 GHz) |
| BLE | 5.0 |
| GPIO | 45 |
| DVP | ✅ 8-bit 并行相机接口 |
| USB | OTG / Serial-JTAG |
| SPI | 4 组 |
| I²C | 2 组 |

## TSF API

```c
esp_wifi_get_tsf_time(WIFI_IF_STA); // µs 精度
```

## 项目用量

| 位置 | 数量 | 用途 |
|------|:---:|------|
| CAN 主节点网关 | 5 | Madgwick + TSF + UDP 发送（每网关管 3-4 节点） |
| 眼部推理 | 2 | OV2640 阈值分割 + 质心 |

## 相关

- [TSF WiFi 时间同步](TSF WiFi 时间同步.md) — TSF API 使用
- [梯度下降姿态解算](梯度下降姿态解算.md) — Madgwick 固件
- `F:\Projects\PCB\Library\Candidate\esp32-s3_datasheet_cn.pdf`
