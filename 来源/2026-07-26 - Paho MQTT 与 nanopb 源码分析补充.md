---
type: source
tags: [嵌入式, IoT, MQTT, nanopb, Protobuf, 源码分析]
created: 2026-07-26
updated: 2026-07-26
---

# Paho MQTT 与 nanopb 源码分析

## 基本信息

| 项目 | Paho MQTT Embedded-C | nanopb |
|------|---------------------|--------|
| **仓库** | `eclipse-paho/paho.mqtt.embedded-c` | `nanopb/nanopb` |
| **语言** | C99 | C99 |
| **许可** | EPL-1.0 / EDL-1.0 | Zlib |
| **核心文件** | MQTTPacket.h/c, MQTTClient.h/c | pb.h, pb_encode.c, pb_decode.c, pb_common.c |

## Paho MQTT 源码验证

| 声明 | 源码验证 |
|------|---------|
| MQTT 消息类型 1-14 | ✅ `enum msgTypes` 精确匹配 |
| Remaining length 可变长度编码 | ✅ `MQTTPacket_encode()` 最多 4 字节 |
| QoS 1 PUBREL 强制 QoS=1 | ✅ `MQTTSerialize_ack()` 中 `header.bits.qos = (packettype == PUBREL) ? 1 : 0` |
| "发→等→收" 模型 | ✅ `waitfor()` 阻塞等待指定包类型或超时 |
| Transport 抽象 | ✅ `Network` 结构体中的函数指针 |

### 源码修正

| 修正点 | 详情 |
|--------|------|
| `MQTTClient_create` → `MQTTClientInit` | 实际函数名（原页面使用旧名） |
| 三回调 API `MQTTClient_setCallbacks` → 不存在 | 实际使用 `MQTTSetMessageHandler()` 逐主题注册 + `defaultMessageHandler` |
| 消息处理数组上限 5 | `#define MAX_MESSAGE_HANDLERS 5` |
| `MQTTString` 双表示 | `cstring` + `lenstring`，解码时 lenstring 优先且不 null-terminated |

### 关键遗漏

- **非阻塞读取状态机**：MQTTPacket.c 中 `state=0/1/2` 三步读取（header → rem_len → payload）
- **PINGREQ 超时检测**：`ping_outstanding` 标志是唯一断连检测机制
- **主题通配符匹配**：`isTopicMatched()` 纯 C 实现 `+`/`#` 匹配

## nanopb 源码验证

| 声明 | 源码验证 |
|------|---------|
| Wire types (0=VARINT, 2=LENGTH_DELIMITED, 5=32BIT) | ✅ `pb_wire_type_t` enum |
| Varint 编码 7 数据位 + 1 连续位 | ✅ `pb_encode_varint`/`pb_decode_varint32` |
| 静态分配核心 | ✅ `PB_ATYPE_STATIC = 0x00` |
| 三种分配策略 | ✅ STATIC/CALLBACK/POINTER (ATYPE 掩码 0xC0) |

### 源码修正

| 修正点 | 详情 |
|--------|------|
| "4 个运行时文件" → 实际 7 个 | pb.h, pb_encode.c/h, pb_decode.c/h, pb_common.c/h（pb_common.c 389 行，不可省略） |
| `FT_*` 命名 → `PB_ATYPE_*` | `FT_STATIC` → `PB_ATYPE_STATIC` 等（新版本 API） |
| `ImuReading_init_zero` → `MyMessage_init_default` | 初始化宏名 |
| `PB_FIELDS()` → `PB_BIND()` | 字段绑定宏已更新 |

### 关键遗漏

- **`pb_type_t` 编码**：字节 bit 布局 — LTYPE(0-3), HTYPE(4-5), ATYPE(6-7)
- **子消息两遍编码**：SIZING pass → 测量长度 → 实际写入（`PB_OSTREAM_SIZING` 伪流）
- **Required 字段跟踪**：`pb_fields_seen_t` 位掩码（默认 64 位）
- **Proto3 默认值省略**：`pb_check_proto3_default_value()` 对所有标量零值跳过编码
- **编译选项**：`PB_ENABLE_MALLOC`, `PB_MAX_REQUIRED_FIELDS`, `PB_FIELD_32BIT` 等
- **回调机制**：`pb_callback_t` 的 `decode`/`encode` 联合体 + `arg` 指针

## 集成注意

MQTT + nanopb 组合中，`MQTTPublish()` 是**阻塞**调用。在 100Hz QoS 1 场景下每发布阻塞 ≥10ms。如需真正 100Hz，应使用 QoS 0 或异步变体。两者均为零 malloc 设计，架构兼容嵌入式环境。
