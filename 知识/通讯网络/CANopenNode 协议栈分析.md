---
type: concept
tags: [通讯, CAN, CANopen, 协议栈, 源码分析, CiA301, PDO, SDO, NMT]
created: 2025-07-25
updated: 2025-07-25
sources: ["[2025-07-25 - CANopenNode 源码分析](../../%E6%9D%A5%E6%BA%90/2025-07-25%20-%20CANopenNode%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)"]
---

# CANopenNode 协议栈分析

> CANopenNode v4.0 — CiA 301 标准 C 实现。55 个 C/H 文件，~20KB 代码，~2KB RAM。

## 架构总览

CANopenNode 是一个**事件驱动、非阻塞**协议栈，双线程模型：

- **实时线程（1ms 定时器）**：SYNC → RPDO → TPDO，硬实时
- **主线程（后台循环）**：NMT、SDO、Heartbeat、Emergency，非实时

```cpp
// setup() — 分配所有对象（~2KB 堆）
CO_new();
// 初始化 CAN 控制器
CO_CANinit();
// 初始化协议栈
CO_CANopenInit();
// 初始化 PDO
CO_CANopenInitPDO();

// loop() — 后台循环处理 NMT + SDO + HB + EM
CO_process();
/* 应用代码 */
sleep();

// 1ms 中断 — 硬实时处理
CO_process_SYNC();
CO_process_RPDO();
CO_process_TPDO();
```

## 对象字典（OD）

OD 是 CANopen 的核心——所有参数按 `index:subIndex` 编址。外部工具(如 CANopenEditor)生成 `OD.h/OD.c`。

```c
// OD 访问 API
OD_find(od, 0x1017)           // 查找 Heartbeat 时间
OD_get_u16(entry, 0, &val)    // 读 sub0
OD_set_u32(entry, 1, newVal)  // 写 sub1

// 直接内存指针（仅在 RAM 中的变量）
OD_getPtr(entry, subIdx, &len, &err);
```

## NMT 状态机

INITIALIZING (after power-on) → PRE-OPERATIONAL ↔ OPERATIONAL, with STOPPED accessible from both PRE-OP and OPERATIONAL

| 状态 | 允许 | 禁止 |
|------|------|------|
| PRE-OP | SDO, SYNC, EM, NMT | **PDO** |
| OPERATIONAL | 全部 | — |
| STOPPED | 仅 NMT | 所有数据通信 |

## PDO — 实时数据传输

PDO 是 CANopen 的高效机制——**无协议开销**，一个 CAN 帧直接映射到 OD 变量。

TPDO trigger methods:
1. Event-driven (254/255): auto-send on OD variable change
2. SYNC synchronized (1-240): send every N SYNCs
3. Application request: CO_TPDOsendRequest()

Mapping example (OD 1A00):
- 0x2001:01 → PDO byte 0-1 (sensor 1 X axis)
- 0x2001:02 → PDO byte 2-3 (sensor 1 Y axis)
- 0x2001:03 → PDO byte 4-5 (sensor 1 Z axis)

对于你的 CAN 集群：一个 TPDO 可以打包 4 个 float32（四元数）到一个 8 字节 CAN 帧。

## SDO — 参数配置

SDO 用于非实时参数访问（读/写 OD）。服务器端状态机支持三种模式：

| 模式 | 条件 | 吞吐量 |
|------|------|--------|
| 加速传输 | 数据 ≤4 字节 | 1 帧完成 |
| 分段传输 | 任意大小 | 逐帧确认 |
| 块传输 | 任意大小 | 批量传输+CRC |

```c
// 主节点通过 SDO 配置从节点
CO_SDOclientDownload(SDO_C, timeDiff, ...);
// 写入 0x2000:01 = 200 (修改 IMU 采样率)
```

## 标准 CAN-ID 分配

| 对象 | CAN-ID |
|------|--------|
| NMT | `0x000` |
| SYNC | `0x080` |
| Emergency | `0x080 + nodeId` |
| TPDO1 | `0x180 + nodeId` |
| RPDO1 | `0x200 + nodeId` |
| SDO(server) | `0x580 + nodeId` |
| SDO(client) | `0x600 + nodeId` |
| Heartbeat | `0x700 + nodeId` |

## 对你的 CAN 集群的适配

你的 5 主 + 10 从架构：

Node ID allocation:
- Waist master: 1 (NMT Master)
- Hand master L/R: 2, 3
- Foot master L/R: 4, 5
- Slaves: 10-19

PDO mapping (8 bytes/frame):
- TPDO1 (slave→master): quaternion [w,x,y,z] float32×4
- RPDO1 (master→slave): config command [cmd, param1, param2]

Startup flow:
1. All nodes power-on → PRE-OPERATIONAL
2. Waist master configures nodes via SDO
3. NMT → OPERATIONAL → PDO starts automatically

CANopenNode 只需移植 `CO_driver.c`（CAN 控制器驱动）和 `CO_driver_target.h`（平台类型定义）两个文件。
