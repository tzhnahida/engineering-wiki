---
type: concept
tags: [ethernet, ieee-802.3, lan, mac, phy, tcpip]
created: 2026-07-29
updated: 2026-07-29
sources: ["[2026-07-29 - IEEE 802.3 Ethernet Standard](../../来源/2026-07-29%20-%20IEEE%20802.3%20Ethernet%20Standard.md)"]
---

# Ethernet 协议概述 (IEEE 802.3)

> Ethernet 是全球部署最广泛的有线局域网技术，由 IEEE 802.3 工作组标准化。从前身 Xerox PARC 的 3 Mbps 实验性网络 (1973)，到今天 400 Gbps 的数据中心互联。本页覆盖 MAC 帧格式、CSMA/CD、PHY 类型和速率演进。

## 1. 与 WiFi 对比

| 特性 | Ethernet (802.3) | WiFi (802.11) |
|------|-----------------|---------------|
| 介质 | 双绞线/光纤/背板 | 无线电 (2.4/5/6 GHz) |
| 信道接入 | CSMA/CD (半双工) / 全双工 | CSMA/CA |
| 碰撞检测 | ✅ 电压检测 | ❌ 不能同时收发 |
| 帧最大长度 | 1518/1522 (VLAN) | ~2304 |
| 确认机制 | ❌ 无 ACK | ✅ 每帧 ACK |
| 地址字段 | 2 (SA+DA) | 3-4 (取决于 To/From DS) |
| 最低延迟 | <1 µs (全双工交换机) | >100 µs (CSMA/CA + ACK) |

## 2. MAC 帧格式

### 2.1 基本帧 (IEEE 802.3)

| # | 字段 | 大小 | 说明 |
|---|------|------|------|
| 1 | **Preamble** | 7 B | 55-55-55-55-55-55-55，接收端时钟同步 |
| 2 | **SFD** | 1 B | Start Frame Delimiter = D5，标记帧开始 |
| 3 | **DA** | 6 B | 目的 MAC 地址 |
| 4 | **SA** | 6 B | 源 MAC 地址 |
| 5 | **EtherType/Length** | 2 B | ≤1500 = 802.3 长度；≥1536 = EtherType |
| 6 | **Payload** | 46-1500 B | 上层 PDU；不足 46 B 时填充 |
| 7 | **FCS** | 4 B | CRC-32，多项式 0x04C11DB7 |

> 常用 EtherType：IPv4=0x0800, ARP=0x0806, IPv6=0x86DD, VLAN=0x8100

### 2.2 VLAN 标签 (802.1Q)

插入在 SA 和 EtherType 之间：

| 字段 | 大小 | 值/说明 |
|------|------|---------|
| **TPID** | 2 B | 0x8100 (802.1Q) / 0x88A8 (802.1ad) |
| **PCP** | 3 bit | Priority Code Point (802.1p QoS) |
| **DEI** | 1 bit | Drop Eligible Indicator |
| **VID** | 12 bit | VLAN ID (1-4094) |

带 VLAN 标签的帧最大长度 1522 B。

### 2.3 最小帧长约束

- **最小帧长 = 64 B** (Preamble/SFD 不计入)
- 64 B − 6(DA) − 6(SA) − 2(EtherType) − 4(FCS) = **46 B 最小 Payload**
- 若上层数据不足 46 B，MAC 层自动填充 (Padding)

> [!note] 64 B 最小帧长是 CSMA/CD 碰撞检测的核心约束——帧太短则可能在发送完成前检测不到远端碰撞。10 Mbps Ethernet 的 slot time = 512 bit times = 64 B。

## 3. CSMA/CD (半双工)

半双工 Ethernet 使用 CSMA/CD (Carrier Sense Multiple Access with Collision Detection)：

```
1. 监听信道 (Carrier Sense)
2. 信道空闲 + 等待 IFG (96 bit times) → 开始发送
3. 边发边听 → 检测碰撞 (信号幅值异常)
4a. 无碰撞 → 发送完成
4b. 有碰撞 → 发 32-bit JAM 信号 → 退避 (Truncated Binary Exponential Backoff)
```

### 3.1 退避算法

```
k = min(n, 10)          # n = 重传次数
r = random(0, 2^k - 1)  # r 个 slot times
等待 r × 512 bit times
```

- 第 1 次重传：0-1 slots
- 第 5 次重传：0-31 slots
- 第 16 次重传：放弃，上报错误

> [!note] 全双工时代 (交换机普及后) CSMA/CD 几乎已成为历史。全双工链路使用独立的 TX/RX 差分对，无需碰撞检测。

## 4. PHY 类型与速率演进

### 4.1 双绞线 (Twisted Pair)

| 标准 | 速率 | 线对使用 | 编码 | 最远距离 |
|------|------|----------|------|----------|
| **10BASE-T** | 10 Mbps | 2 对 (Cat3) | Manchester | 100 m |
| **100BASE-TX** | 100 Mbps | 2 对 (Cat5) | 4B/5B + MLT-3 | 100 m |
| **1000BASE-T** | 1 Gbps | 4 对 (Cat5e) | PAM-5 + Trellis | 100 m |
| **2.5GBASE-T** | 2.5 Gbps | 4 对 (Cat5e) | PAM-16 | 100 m |
| **5GBASE-T** | 5 Gbps | 4 对 (Cat6) | PAM-16 | 100 m |
| **10GBASE-T** | 10 Gbps | 4 对 (Cat6a) | PAM-16 + LDPC | 100 m |

### 4.2 1000BASE-T 编码细节

1000BASE-T 是全双工千兆 Ethernet 最广泛的形式：

- **4 对双绞线**同时收发 (echo cancellation)
- **PAM-5** 调变：5 个电压电平 (-2, -1, 0, +1, +2)
- 每个符号传 2 bits → 125 Mbaud × 2 bits × 4 对 = **1 Gbps**
- **Trellis Coding** + **Viterbi Decoding** 前向纠错
- 自协商 (Auto-Negotiation)：自动检测对方支持的最高速率和双工模式

### 4.3 光纤

| 标准 | 速率 | 波长 | 介质 | 最远距离 |
|------|------|------|------|----------|
| **1000BASE-SX** | 1 Gbps | 850 nm | MMF | 550 m |
| **1000BASE-LX** | 1 Gbps | 1310 nm | SMF | 10 km |
| **10GBASE-SR** | 10 Gbps | 850 nm | MMF | 300 m |
| **10GBASE-LR** | 10 Gbps | 1310 nm | SMF | 10 km |
| **100GBASE-LR4** | 100 Gbps | 4λ WDM | SMF | 10 km |

### 4.4 MII 接口 (芯片侧)

MAC 芯片与 PHY 芯片之间通过 MII (Media Independent Interface) 通信：

| 接口 | 数据宽 | 时钟 | 速率 | 用途 |
|------|--------|------|------|------|
| **MII** | 4-bit | 25 MHz | 100 Mbps | 传统 |
| **RMII** | 2-bit | 50 MHz | 100 Mbps | 引脚减半，常用 |
| **GMII** | 8-bit | 125 MHz | 1 Gbps | 千兆 |
| **RGMII** | 4-bit DDR | 125 MHz | 1 Gbps | **最常用**：12 引脚 → MAC↔PHY 互联 |
| **SGMII** | 差分串行 | 1.25 Gbps | 1 Gbps | 串行替代 RGMII |

## 5. 嵌入式 Ethernet 实践

### 5.1 Typical MCU + PHY

```mermaid
flowchart LR
    MCU["MCU<br/>MAC (STM32/LPC/ESP32)"]
    PHY["PHY<br/>(LAN8720/DP83848)"]
    Mag["Magnetics<br/>(RJ45 带变压器)"]
    Cable["Cat5e Cable"]
    
    MCU -->|"RGMII/RMII"| PHY
    PHY -->|"MDI (差分)"| Mag
    Mag -->|"RJ45"| Cable
```

| 组件 | 功能 | 常见型号 |
|------|------|----------|
| **MCU MAC** | 帧封装、CRC、DMA、地址过滤 | STM32F4/F7/H7 ETH 外设 |
| **PHY** | 物理层：MLT-3/PAM-5、自协商、链路检测 | LAN8720A、DP83848、KSZ8081 |
| **Magnetics** | 隔离、共模抑制、阻抗匹配 1:1 | 集成 RJ45 Jack (HanRun HR911105A) |

### 5.2 LWIP 协议栈

嵌入式 TCP/IP 通常使用 LWIP。参见 [嵌入式软件/LWIP/1. LWIP 概述与架构](../嵌入式软件/LWIP/1.%20LWIP%20概述与架构.md)。

## 相关页面

- [通讯网络/WiFi 协议概述](WiFi%20协议概述.md) — 无线局域网对比
- [嵌入式软件/LWIP/1. LWIP 概述与架构](../嵌入式软件/LWIP/1.%20LWIP%20概述与架构.md) — 嵌入式 TCP/IP 协议栈
- [嵌入式软件/LWIP/4. LWIP IP 层](../嵌入式软件/LWIP/4.%20LWIP%20IP%20层.md) — IP 层实现
- [嵌入式软件/LWIP/7. LWIP netif 网络接口](../嵌入式软件/LWIP/7.%20LWIP%20netif%20网络接口.md) — netif 网络接口抽象
