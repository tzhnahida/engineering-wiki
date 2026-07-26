---
type: concept
tags: [slimevr, protocol, udp, quaternion, steamvr, driver, fullbody-tracking]
created: 2026-06-24
updated: 2026-06-24
sources:
  - "https://docs.slimevr.dev"
  - "https://github.com/SlimeVR/SlimeVR-Server"
  - "https://github.com/SlimeVR/SlimeVR-Firmware"
  - "[2026-06-24 - SlimeVR 协议文档](../../%E6%9D%A5%E6%BA%90/2026-06-24%20-%20SlimeVR%20%E5%8D%8F%E8%AE%AE%E6%96%87%E6%A1%A3.md)"
---

# SlimeVR 协议

> SlimeVR 是开源全身动捕协议。ESP32 发 UDP 四元数 → SlimeVR Server 解析 → SteamVR Driver → VRChat。通过标准 UDP 协议与 SteamVR 生态对接。

---

## 三层架构

Three-layer architecture:
- ESP32 Firmware: Madgwick → quaternion, TSF timestamp, WiFi UDP send
- SlimeVR Server (Java): UDP receive → bone mapping, streaming → SteamVR Driver, GUI config/WiFi management
- SteamVR: OpenVR API → VRChat

---

## UDP 协议格式

### 搜索阶段

ESP32 上电 → 通过 WiFi 连上 AP → UDP 广播搜索包到 6969 端口，寻找 SlimeVR Server：

Search packet (ESP32 → Server, UDP 6969): [3-byte "search" header] [MAC: 6 bytes] [Board ID: 1 byte] [IMU type: 1 byte]

Server 收到后回复握手包（含分配的 tracker ID）。

### 数据包（正常运行时）

UDP 单播，端口由握手阶段协商。

Tracker Data Packet (ESP32 → Server): [Pkt Type: 1B] [Tracker ID: 1B] [Quat: 16B] [Optional fields] [Reserved]

Packet Types:
- 0x00 — Heartbeat (empty packet, keep alive)
- 0x01 — Handshake (search/handshake)
- 0x02 — (reserved)
- 0x04 — Tracker data (quaternion)
- 0x05 — (reserved)
- 0x0F — (reserved)

四元数编码：4 个 float32 (x, y, z, w)，little-endian。每包 ≈ 20-30 字节。

### 数据率

- 默认 100 Hz（可配 50-200 Hz）
- 带宽 ≈ 30 B × 100 Hz × 15 节点 = **45 kB/s**（远低于 WiFi 极限）

---

## SlimeVR Server

### 核心功能

| 功能 | 说明 |
|------|------|
| **UDP 接收** | 监听 6969，接收所有 tracker 的搜索和数据包 |
| **骨骼映射** | 将每个 tracker 的四元数映射到预定义的骨骼节点 |
| **SteamVR 桥接** | 通过 OpenVR Driver API 将骨骼姿态注入 SteamVR |
| **自动校准** | T-pose 标定、身体比例测量、重置漂移 |
| **WiFi 管理** | GUI 配置 ESP32 的 SSID/密码/信道 |

### 骨骼映射表

| Tracker ID | SteamVR Bone |
|------------|-------------|
| 0 | Head |
| 1 | Chest |
| 2 | Hips |
| 3 | Left Upper Leg |
| 4 | Right Upper Leg |
| 5 | Left Lower Leg |
| 6 | Right Lower Leg |
| 7 | Left Foot |
| 8 | Right Foot |
| 9 | Left Upper Arm |
| 10 | Right Upper Arm |
| 11 | Left Lower Arm |
| 12 | Right Lower Arm |
| 13 | Left Hand |
| 14 | Right Hand |

---

## 扩展能力

SlimeVR 协议的可扩展性允许在标准四元数包之上叠加附加数据通道：

- **附加传感器**：手指弯曲 (FlexData type 26)、温度 (Temperature type 20)
- **时间同步**：TSF (802.11 §11.1) 可与协议并行，提供 µs 级多节点对齐
- **CAN 桥接**：CAN 集群架构可通过一个 WiFi 网关节点汇聚多传感器数据

> 你的系统是 SlimeVR 的**超集**——协议兼容，但增加了时间同步、眼部数据、手指 ADC 和 STM32 头显回传。

---

## 协议扩展能力

SlimeVR 协议的扩展性：

```c
// 扩展包格式示例
typedef struct {
    uint8_t  packet_type;      // 0x04 = tracker data
    uint8_t  tracker_id;       // 由 Server 握手分配
    float    quat[4];          // x, y, z, w (little-endian)
    // 扩展字段（兼容但标准 SlimeVR 忽略）:
    uint8_t  finger_adc[5];    // 手指电位器
    uint8_t  pupil_x, pupil_y; // 瞳孔坐标
} slimevr_packet_t;
```

标准 Server 不直接支持手指 ADC 和眼部数据，可以通过自写中间层或修改骨骼求解器接入。Phase 1 可用标准 Server 处理 13 个 IMU 节点，手指在 PC 端后处理，眼部走独立通道。

SteamVR 驱动可复用 SlimeVR 开源 driver，注册为 tracked device，注入骨骼姿态。

---

## 关键词

`SlimeVR` `UDP protocol` `tracker handshake` `quaternion encoding` `SteamVR driver` `OpenVR` `full body tracking` `bone mapping`
