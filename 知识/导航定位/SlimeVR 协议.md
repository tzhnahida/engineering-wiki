---
type: concept
tags: [slimevr, protocol, udp, quaternion, steamvr, driver, fullbody-tracking]
created: 2026-06-24
updated: 2026-06-24
sources:
  - "https://docs.slimevr.dev"
  - "https://github.com/SlimeVR/SlimeVR-Server"
  - "https://github.com/SlimeVR/SlimeVR-Firmware"
---

# SlimeVR 协议

> SlimeVR 是开源全身动捕协议。ESP32 发 UDP 四元数 → SlimeVR Server 解析 → SteamVR Driver → VRChat。通过标准 UDP 协议与 SteamVR 生态对接。

---

## 三层架构

```
ESP32 固件                 SlimeVR Server (Java)        SteamVR
───────                    ────────────────────         ──────
Madgwick → 四元数           UDP 收包 → 骨骼映射          OpenVR API
TSF 时间戳                  串流 → SteamVR Driver        VRChat
WiFi UDP 发送               GUI 配置/WiFi 管理
```

---

## UDP 协议格式

### 搜索阶段

ESP32 上电 → 通过 WiFi 连上 AP → UDP 广播搜索包到 6969 端口，寻找 SlimeVR Server：

```
Search packet (ESP32 → Server, UDP 6969):
┌────┬──────┬──────────┬──────────┐
│ 3B │ MAC  │ Board ID │ IMU type │
│"搜索"│ 6B  │  1B      │  1B      │
└────┴──────┴──────────┴──────────┘
```

Server 收到后回复握手包（含分配的 tracker ID）。

### 数据包（正常运行时）

UDP 单播，端口由握手阶段协商。

```
Tracker Data Packet (ESP32 → Server):
┌──────────┬─────────┬─────────┬──────────┬──────────────┐
│ Pkt Type │ Tracker │  Quat   │ Optional │   Reserved   │
│  1B      │  ID 1B  │  16B    │  fields  │              │
└──────────┴─────────┴─────────┴──────────┴──────────────┘

Packet Types:
  0x00 — Heartbeat (空包，保持连接存活)
  0x01 — Handshake (搜索/握手)
  0x02 — (保留)
  0x04 — Tracker data (四元数)
  0x05 — (保留)
  0x0F — (保留)
```

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

```
Tracker ID  →  SteamVR Bone
─────────      ─────────────
  0            Head
  1            Chest
  2            Hips
  3            Left Upper Leg
  4            Right Upper Leg
  5            Left Lower Leg
  6            Right Lower Leg
  7            Left Foot
  8            Right Foot
  9            Left Upper Arm
  10           Right Upper Arm
  11           Left Lower Arm
  12           Right Lower Arm
  13           Left Hand
  14           Right Hand
```

---

## 与你的系统对比

| | 标准 SlimeVR | 你的系统 (Phase 1) |
|------|------|------|
| **IMU 节点** | ESP8266/ESP32 | ESP32-S3 + ICM-42688-P |
| **节点数** | 5-14 | **15 追踪点** (5 CAN 主节点 + 10 CAN 从节点) |
| **姿态算法** | Madgwick/Bosch BHI | **Madgwick (自己移植，β 按 ICM-42688-P 计算)** |
| **时间同步** | 无（各节点独立） | **TSF (802.11 §11.1)** |
| **附加数据** | 仅四元数 | 四元数 + 手指 ADC 数据 |
| **头部** | 独立节点 | **STM32 头显内置（USB 回传）** |
| **眼部** | 无 | **ESP32 瞳孔坐标（<50B/帧）** |
| **输出** | SteamVR | SteamVR (Phase 1) + BVH/FBX (Phase 2) |

> 你的系统是 SlimeVR 的**超集**——协议兼容，但增加了时间同步、眼部数据、手指 ADC 和 STM32 头显回传。

---

## 你需要做的适配

### 1. ESP32 端固件

```c
// SlimeVR 兼容的 UDP 发送函数
typedef struct {
    uint8_t  packet_type;      // 0x04 = tracker data
    uint8_t  tracker_id;       // 由 Server 握手分配
    float    quat[4];          // x, y, z, w (little-endian)
    // 扩展字段（兼容但 SlimeVR 忽略）:
    uint8_t  finger_adc[5];    // 手指电位器 (仅手指节点)
    uint8_t  pupil_x, pupil_y; // 瞳孔坐标 (仅眼部节点)
} slimevr_packet_t;
```

### 2. SlimeVR Server 改造（或自写 Server）

标准 Server 不支持的：
- 手指 ADC → 需要修改骨骼求解器或自写中间层
- 眼部坐标 → 无法直接灌入 SteamVR（SteamVR 不原生支持眼追）
- TSF 时间戳 → Server 端统一归算

**策略**：Phase 1 直接用标准 SlimeVR Server（13 个 IMU 节点走标准协议），手指在 PC 端做后处理，眼部走独立通道。

### 3. SteamVR 驱动

复用 SlimeVR 开源 driver，注册为 tracked device，注入骨骼姿态。

---

## 关键词

`SlimeVR` `UDP protocol` `tracker handshake` `quaternion encoding` `SteamVR driver` `OpenVR` `full body tracking` `bone mapping`
