---
type: concept
tags: [sata, storage, ahci, protocol, interface]
created: 2026-08-03
updated: 2026-08-03
sources: []
---

# SATA 存储接口协议

> SATA (Serial ATA) 是从 PATA (Parallel ATA/IDE) 进化而来的串行存储接口。从 2003 年 SATA 1.0 (1.5 Gbps) 到 SATA 3.3 (6 Gbps)，它统治消费级存储长达 15 年。如今被 NVMe 逐步替代，但仍是 HDD 和入门 SSD 的主力接口。

## 1. 为什么从 PATA 变 SATA

| 问题 | PATA (IDE) | SATA |
|------|-----------|------|
| 信号 | 16-bit 并行单端 | **差分串行** |
| 电压 | 5V TTL | **400-600mV LVDS** |
| 线缆 | 40/80 芯扁平排线 | **7 芯细缆** |
| 最大线长 | 45cm (80芯) | **1m** |
| 带宽 | 133 MB/s (ATA-133) | **600 MB/s** (SATA 3.0) |
| 设备连接 | 主/从跳线 (Master/Slave) | **点对点** (无跳线) |
| 热插拔 | 不支持 | **支持** |

> 核心变化：**并行 → 串行**。用差分信号替代单端 TTL，消除了并行总线的 skew 问题——不再需要所有数据线同时到达。

## 2. 物理层

### 2.1 信号

SATA 使用两对差分线：

```
Host ─── TX+/TX- ─────────── RX+/RX- ─── Device
Host ←── RX+/RX- ─────────── TX+/TX- ─── Device

每方向一对差分线 → 全双工
```

| 参数 | SATA 1.0 | SATA 2.0 | SATA 3.0 |
|------|----------|----------|----------|
| 速率 | 1.5 Gbps | 3.0 Gbps | 6.0 Gbps |
| 编码 | 8b/10b | 8b/10b | 8b/10b |
| 有效带宽 | 150 MB/s | 300 MB/s | **600 MB/s** |
| 差分摆幅 | 400-600 mV | 400-600 mV | 400-600 mV |
| 差分阻抗 | 100 Ω | 100 Ω | 100 Ω |

> [!note] 8b/10b 编码开销 20%。SATA 6 Gbps 线速率 → 6G × 0.8 / 8 = 600 MB/s 有效带宽。

### 2.2 连接器

```
SATA 数据线 (7-pin):
Pin 1: GND    Pin 4: GND
Pin 2: TX+    Pin 5: RX-
Pin 3: TX-    Pin 6: RX+
               Pin 7: GND

SATA 电源线 (15-pin):
  3.3V · 5V · 12V + 预充电 + 交错接地
```

关键：电源和数据的**交错 GND**——每对信号引脚之间夹着地线，最小化串扰。

### 2.3 OOB (Out-of-Band) 信令

SATA 用 OOB 信号建立链路——这是 SATA 独有的：

```
COMRESET  : Host 发 106.7ns Burst + 320ns Idle → 6次
COMINIT   : Device 同上
COMWAKE   : 106.7ns Burst + 106.7ns Idle → 6次

OOB 序列 ≠ 数据，是纯粹的物理层握手
```

| OOB 信号 | 方向 | 含义 |
|----------|------|------|
| COMRESET | Host → Device | 强制复位 |
| COMINIT | Device → Host | 设备就绪请求 |
| COMWAKE | 双向 | 唤醒（从睡眠/省电模式） |

## 3. 链路层与传输层

### 3.1 帧结构 (FIS — Frame Information Structure)

SATA 的所有通信通过 FIS 进行：

| FIS 类型 | 用途 |
|----------|------|
| **Register FIS (27h)** | 主机→设备：发送 ATA 命令 (类似 IDE 端口写) |
| **DMA Setup FIS (41h)** | 双向：DMA 传输参数协商 |
| **Data FIS (46h)** | 双向：实际数据传输 |
| **PIO Setup FIS (5Fh)** | 设备→主机：通知主机准备好 PIO 数据 |
| **D2H Register FIS (34h)** | 设备→主机：命令完成状态 |
| **Set Device Bits FIS (A1h)** | 设备→主机：更新状态寄存器位 |

### 3.2 命令提交 (Register FIS → Data FIS 流程)

```
Host:  Register FIS (写命令 + LBA + sector count)
Device: (执行)
Device: Data FIS (数据) 或 PIO Setup FIS
Device: D2H Register FIS (状态 = OK/Error)
```

### 3.3 NCQ (Native Command Queuing)

SATA 2.0 引入的命队列——设备可重排序命令以优化寻道：

```
传统: CMD1 → 完成 → CMD2 → 完成 → CMD3 → 完成
NCQ:   CMD1, CMD2, CMD3 同时下发
       → 设备按最优寻道顺序执行
       → 乱序完成, 主机按 tag 匹配
```

| 参数 | SATA NCQ | NVMe |
|------|----------|------|
| 队列深度 | **32** | 65535 |
| 队列数 | 1 | 65535 |
| 重排序 | 设备端（不透明） | 主机端（可控） |
| 适用设备 | HDD (寻道优化) | SSD (并行通道利用) |

> NCQ 对 HDD 有效（减少寻道），对 SSD 意义有限（无机械寻道）。NVMe 的多队列 + 多核才是 SSD 的正解。

## 4. AHCI 编程模型

AHCI 是 SATA 控制器的标准编程接口：

```
┌──────────────────────┐
│    OS / Driver       │
├──────────────────────┤
│    AHCI Controller   │ ← PCI BAR (MMIO 寄存器)
│    (HBA — Host Bus   │
│     Adapter)         │
├──────────┬───────────┤
│  Port 0  │  Port 1   │ ← 每个 Port 独立
│  (SATA)  │  (SATA)   │
└──────────┴───────────┘
```

AHCI 关键寄存器：

| 寄存器 | 说明 |
|--------|------|
| PxCI | Port Command Issue — 主机写 1 发命令 |
| PxSACT | SActive — NCQ 命令 slot 占用 |
| PxIS | Interrupt Status |
| PxFB/PxFBU | FIS Base Address (指向内存中的 FIS 区) |
| PxCLB/PxCLBU | Command List Base Address |

### 命令提交流程

```
1. Host: 填 Command Table (在内存中)
2. Host: 设置 Command List 指向 Command Table
3. Host: 写 PxCI → 通知控制器
4. Controller: DMA 取 Command Table → 执行
5. Controller: DMA 写完成 FIS 到内存
6. Controller: 中断
7. Host: 读 PxIS → 处理完成
```

## 5. 电力管理

| 状态 | 退出延迟 | 功耗 | 说明 |
|------|----------|------|------|
| **PHYRDY** | — | 全功率 | 正常运转 |
| **Partial** | < 10 µs | 中等 | 快速恢复——空闲时自动进入 |
| **Slumber** | < 10 ms | 低 | 深度休眠 |

> Partial 模式是 SATA 链路最常见的省电状态——空闲后自动进入，数据到达时微秒级恢复。

## 6. SATA 代际

| 版本 | 速率 | 编码 | 年份 | 关键特性 |
|------|------|------|------|----------|
| SATA 1.0 | 1.5 Gbps | 8b/10b | 2003 | 首次串行化 |
| SATA 2.0 | 3.0 Gbps | 8b/10b | 2004 | NCQ、Port Multiplier |
| SATA 3.0 | 6.0 Gbps | 8b/10b | 2009 | 等时传输、NCQ 优化 |
| SATA 3.1 | 6.0 Gbps | 8b/10b | 2011 | mSATA、USM、零功耗光驱 |
| SATA 3.2 | 6.0 Gbps | 8b/10b | 2013 | **SATA Express** (PCIe ×2 混合) |
| SATA 3.3 | 6.0 Gbps | 8b/10b | 2016 | SMR HDD 支持、Power Disable |

> SATA Express 从未真正起飞——M.2 NVMe 直接绕过了 SATA 控制器的瓶颈。

## 7. 与 NVMe 的本质对比

| | SATA AHCI | NVMe |
|------|-----------|------|
| 传输层 | AHCI (为 HDD 设计) | NVMe (为 SSD 原生) |
| 物理层 | SATA PHY (6 Gbps) | PCIe PHY (Gen5 32 GT/s ×4) |
| 最大带宽 | 600 MB/s | **16000 MB/s** |
| 队列深度 | 1Q/32cmd | **64K Q/64K cmd/Q** |
| CPU 开销 | 高（寄存器 MMIO × 多次） | 低（门铃一次 + 内存 SQ） |
| 适用 | HDD、SATA SSD | NVMe SSD、高性能存储 |

> SATA 的物理层上限 (6 Gbps) 在 2010 年前后就触达了。之后所有的"SATA 3.x"都是协议层的小修小补——没有提速空间。NVMe over PCIe 是物理带宽的维度提升。

## 相关页面

- [通讯网络/NVMe SSD 协议](NVMe%20SSD%20协议.md) — NVMe 架构、队列模型、命令体系
- [通讯网络/PCIe 信号编码演进](PCIe%20信号编码演进.md) — 8b/10b → 128b/130b → PAM4
