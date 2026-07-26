---
type: source
tags: [embedded, iot, mqtt, protobuf, serialization]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - Paho MQTT 与 nanopb 源码分析

## 基本信息

| 项目 | Paho MQTT Embedded-C | nanopb |
|------|---------------------|--------|
| **作者** | Eclipse Foundation | Petteri Aimonen |
| **语言** | C | C |
| **许可证** | EPL-2.0 | zlib |
| **仓库** | `github.com/eclipse/paho.mqtt.embedded-c` | `github.com/nanopb/nanopb` |

## 概述

Paho MQTT Embedded-C 是 Eclipse 的 MQTT 3.1.1 客户端 C 实现。nanopb 是 Google Protocol Buffers 的嵌入式 C 实现。两者配合构成 IoT 传感器到云端的标准数据链路。

## 分析产出

- [[MQTT nanobp/1. MQTT 协议与 Paho Embedded-C]]
- [[MQTT nanobp/2. nanopb 与 IoT 数据序列化]]
