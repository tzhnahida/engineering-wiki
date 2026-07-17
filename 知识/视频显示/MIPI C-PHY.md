---
type: concept
tags: [mipi, c-phy, physical-layer, serial-interface, three-phase-encoding]
created: 2026-06-30
updated: 2026-07-15
sources:
  - "[2026-06-30 - MIPI C-PHY Specification v2.1](2026-06-30 - MIPI C-PHY Specification v2.1.md)"
---

# MIPI C-PHY

> C-PHY 是 MIPI 联盟定义的高速串行物理层规范，基于**三线三电平（3-Phase）符号编码**，每符号传输 ~2.28 bits。相比 [D-PHY](视频显示/MIPI D-PHY.md) 的差分对方案，C-PHY 用三根线实现嵌入式时钟，省去独立时钟通道，引脚效率更高。C-PHY 的 LP 模式和 Escape Mode 几乎完全复用 D-PHY，可与 D-PHY 共用同一 IC 引脚实现双模器件。

## 1. 核心编码原理

[📷 _llm/raw/assets/standards/cphy21/cphy21_p27_fig7.jpg|400]
*Figure 1 — C-PHY 三线六状态（+x/-x/+y/-y/+z/-z）的电流路径与电平组合*

[📷 _llm/raw/assets/standards/cphy21/cphy21_p28_fig1.jpg|460]
*Figure 2 — 六状态转移图：每个 UI 必须换状态，5 种转移编码 log₂5≈2.32bit*


### 1.1 六种 Wire State

C-PHY 的每条 Lane 由三根线（A, B, C）组成。HS 模式下，每根线被驱动到三个电平之一：High（¾V）、Mid（½V）、Low（¼V），且三根线电平各不相同，共 6 种排列：

| Wire State | A | B | C | Rx_AB | Rx_BC | Rx_CA | 极性 |
|------------|---|---|---|-------|-------|-------|------|
| **+x** | ¾V | ¼V | ½V | 1 | 0 | 0 | 正 |
| **-x** | ¼V | ¾V | ½V | 0 | 1 | 1 | 负 |
| **+y** | ½V | ¾V | ¼V | 0 | 1 | 0 | 正 |
| **-y** | ½V | ¼V | ¾V | 1 | 0 | 1 | 负 |
| **+z** | ¼V | ½V | ¾V | 0 | 0 | 1 | 正 |
| **-z** | ¾V | ½V | ¼V | 1 | 1 | 0 | 负 |

> [!note] 关键约束
> 每个 symbol 边界上 Wire State **必须改变**（不允许停留在同一状态）。这保证了每个 symbol 都有跳变，时钟信息嵌入在跳变中——无需独立时钟 Lane。

### 1.2 状态转移与符号编码

从任一 Wire State 出发，有 5 种可能的转移（自身转移被禁止）。每个 symbol 用 3-bit 表示转移类型：

| Symbol [Flip, Rot, Pol] | 含义 |
|--------------------------|------|
| `1xx` | **Flip** — 同相位、反极性（+x↔-x, +y↔-y, +z↔-z），忽略 Rot/Pol |
| `000` | Rotate CCW, Polarity Same |
| `001` | Rotate CCW, Polarity Opposite |
| `010` | Rotate CW, Polarity Same |
| `011` | Rotate CW, Polarity Opposite |

- **CW/CCW 旋转**：按 x→y→z→x（顺时针）或 x→z→y→x（逆时针）改变相位
- 5 种转移 = log₂(5) ≈ **2.3219 bits/symbol** 的理论信息量

```mermaid
stateDiagram-v2
    direction LR
    state "+x" as +x
    state "-x" as -x
    state "+y" as +y
    state "-y" as -y
    state "+z" as +z
    state "-z" as -z

    +x --> +y : 010
    +x --> -y : 011
    +x --> +z : 000
    +x --> -z : 001
    +x --> -x : 1xx

    +y --> +z : 010
    +y --> -z : 011
    +y --> +x : 000
    +y --> -x : 001
    +y --> -y : 1xx

    +z --> +x : 010
    +z --> -x : 011
    +z --> +y : 000
    +z --> -y : 001
    +z --> -z : 1xx
```

### 1.3 16-bit → 7-Symbol 映射

C-PHY 通过两层编码传输数据：

```
┌──────────┐    ┌──────────────┐    ┌──────────────────┐
│ 16b Word │ → │ 7-Sym Mapper │ → │ Symbol Encoder   │ → A, B, C Lines
│ Tx_Data  │    │ (7×3=21 bit) │    │ (State Machine)  │
└──────────┘    └──────────────┘    └──────────────────┘
```

- **Mapper**：16-bit 字 → 7 个 symbol（7×3=21 bits 内部表示）
- **7 个 symbol 的 Flip bits** 将 16-bit 空间划分为 28 个区域（1×16384 + 7×4096 + 20×1024）
- **Encoder**：每个 symbol 根据**前一个 Wire State** 和当前 symbol 值，查表确定下一个 Wire State
- **总开销**：7 symbols × log₂(5) ≈ 16.25 bits 容量 → 编码 16 bits → 效率 ~98.5%
- 不需要 8b10b 等额外线路编码

### 1.4 接收端

接收端使用三组差分比较器：

| 接收器 | 比较 | 输出 |
|--------|------|------|
| **Rx_AB** | A - B | 1 if A > B |
| **Rx_BC** | B - C | 1 if B > C |
| **Rx_CA** | C - A | 1 if C > A |

3-bit 数字输出唯一标识当前 Wire State，再通过与前一个状态的差异解码出 symbol 值。

## 2. 信号模式

与 D-PHY 一样，C-PHY 使用双模信号：

| 模式 | 信号类型 | 摆幅 | 速率 | 用途 |
|------|----------|------|------|------|
| **HS** | 三线三电平差分 | ~250 mV | 0.08–6 Gsps (~13.7 Gbps) | 高速数据传输 |
| **LP** | 单端 CMOS | 0–1.2 V | ≤ 10 Mbps | 控制、Escape Mode |

> [!note] LP 模式完全复用 D-PHY
> C-PHY 的 LP 信号电平、Escape Mode 进入/退出序列、ULPS 等与 D-PHY 几乎完全相同（spec 明确说明 "reused almost completely"）。这使得双模（D-PHY/C-PHY）芯片设计更简单。

### ALP Mode（Alternate Low-Power）

v2.0+ 支持 ALP Mode，与 D-PHY v2.0 的 ALP 类似：使用差分低摆幅信号替代传统 LP 单端 CMOS，消除 LP↔HS 模式切换延迟。

## 3. HS Burst 结构

C-PHY 的 HS 传输也以 Burst 形式进行，但与 D-PHY 有显著差异：

| 阶段 | D-PHY | C-PHY |
|------|-------|-------|
| **空闲** | LP-11 (Stop) | LP-111 (Stop, 三线) |
| **进入 HS** | LP-11→LP-01→LP-00 | LP-111→LP-001→LP-000 (三线 LP-000) |
| **Preamble** | HS-0 序列 (8-bit) | t3-PREAMBLE (PREBEGIN + PROGSEQ + PREEND) |
| **Sync Word** | `00011101` | `3444443` (7 symbols) |
| **Payload** | 字节流 | 16-bit words → 7-symbol groups |
| **Post** | EoT (LP-11) | Post（全部 "4" symbol 序列）→ LP-111 |

```
LP Mode ────────╮                           ╭──── LP Mode
    LP-111 → LP-001 → LP-000 → Preamble → Sync → [Payload] → Post → LP-111
              └── HS Entry ──┘  └─── SoT ──────────┘       └ EoT ┘
```

## 4. Lane 架构

[📷 _llm/raw/assets/standards/cphy21/cphy21_p29_fig1.jpg|640]
*Figure 3 — 16bit 数据端到端传输：映射器 → 三线状态序列 → 解映射还原*

[📷 _llm/raw/assets/standards/cphy21/cphy21_p34_fig1.jpg|560]
*Figure 6 — 三 Lane PHY 配置：C-PHY 无独立时钟 Lane，时钟内嵌于状态转移*


### 4.1 Universal Lane Module

```
    ┌─────────────────────────────────┐
    │   Lane Control & Interface      │
    │   Logic (CIL)                   │
    ├─────────────────────────────────┤
    │ HS-TX │ HS-RX │ LP-TX │ LP-RX  │
    │       │       │ LP-CD │        │
    └───┬───┴───┬───┴───┬───┴────┬───┘
        A       B       C        PPI
```

| 子模块 | 功能 |
|--------|------|
| **HS-TX** | 三线三电平发送器 |
| **HS-RX** | 三差分接收器 + Symbol Decoder |
| **LP-TX** | 单端 LP 发送器（三线独立） |
| **LP-RX** | 单端 LP 接收器（施密特触发器） |
| **LP-CD** | LP 冲突检测（双向 Lane 需要） |

### 4.2 Lane 类型

| 类型 | HS 方向 | LP 方向 | 线数 |
|------|---------|---------|------|
| **Unidirectional** | Forward only | Forward（Escape 可选双向） | 3 |
| **Bi-Directional** | Forward + Reverse（half-duplex） | 双向 | 3 |

### 4.3 时钟方案

- **无独立 Clock Lane**：时钟嵌入在 symbol 跳变中
- 接收端从 symbol 跳变中恢复时钟（CDR）
- 需要一个 **Clock Multiplier Unit (PLL)** 在发送端产生高频时钟
- **多 Lane 配置**：每个 Lane 独立时钟恢复，Lane 间无需严格相位对齐

## 5. 版本演进

| 版本 | 日期 | 最大符号率 | 关键新特性 |
|------|------|-----------|-----------|
| **v1.0** | 2014-10 | 2.5 Gsps | 基础三电平编码、HS/LP 双模 |
| **v1.1** | 2016-02 | 2.5 Gsps | 规范完善 |
| **v1.2** | 2017-03 | 3.5 Gsps | 高速优化 |
| **v2.0** | 2019-09 | 4.5 Gsps | ALP Mode、Calibration Preamble、Fast BTA |
| **v2.1** | 2021-07 | **6.0 Gsps** | 更高符号率、Alternate/User-defined Calibration Sequence |

> 数据速率 ≈ 符号率 × 2.28（bits/symbol）。v2.1 的最高数据率约 **13.7 Gbps/Lane**。

## 6. 与 D-PHY 对比

| 参数 | D-PHY | C-PHY |
|------|-------|-------|
| **线数/Lane** | 2（差分对） | 3（三线 trio） |
| **时钟** | 独立 Clock Lane（DDR 双边沿） | **嵌入式时钟**（symbol 跳变） |
| **编码效率** | 1 bit/symbol/edge（无编码）/ 8b9b（v2.5） | **~2.28 bits/symbol** |
| **最高速率（v2.5/v2.1）** | 4.5 Gbps/Lane | **~13.7 Gbps/Lane**（6 Gsps） |
| **HS 信号摆幅** | 140–270 mV 差分 | HS 电平 ≈ 250 mV（A-B 差分 ~½V） |
| **LP 模式** | 单端 CMOS 0–1.2V | **相同**（完全复用 D-PHY LP） |
| **Escape Mode** | LPDT / ULPS / Trigger / Remote Reset / TE | LPDT / ULPS / Trigger / Reset-Trigger |
| **双向通信** | BTA（同线反转） | BTA + Fast BTA（v2.0+） |
| **线编码** | 无（v1.x）/ 8b9b（v2.5） | 7-symbol Mapper（~98.5% 效率） |
| **多 Lane** | 1–4 Data + 1 Clock | 1–N Lane（每 Lane 3 线独立） |
| **引脚共存** | — | **可与 D-PHY 共用同一 IC 引脚** |

### 引脚数对比

以 4-Lane 配置为例：

| | D-PHY (4 Data + 1 Clock) | C-PHY (4 Lane) |
|---|---|---|
| 信号线 | 5×2 = 10 线 | 4×3 = 12 线 |
| 等效带宽 | 4×4.5 = 18 Gbps | 4×13.7 = **54.8 Gbps** |
| 带宽/线 | 1.8 Gbps/线 | **4.57 Gbps/线** |

> [!note] 为什么比 D-PHY 省线？
> 虽然单 Lane 多用 1 根线，但 C-PHY 省去了 Clock Lane。在相同带宽需求下，C-PHY 需要更少的 Lane 数，总引脚数可能更优。

## 7. 协议栈位置

C-PHY 与 D-PHY 同级，为上层协议提供 PHY 服务：

```
┌─────────────────────────────────┐
│  Application (DSI / CSI-2)      │
├─────────────────────────────────┤
│  Low Level Protocol (LLP)       │
├─────────────────────────────────┤
│  Lane Management                │
├─────────────────────────────────┤
│  PHY:  D-PHY  |  C-PHY          │  ← 同一层级，互换使用
└─────────────────────────────────┘
```

DSI v1.3 和 CSI-2 v2.0+ 均可在 D-PHY 或 C-PHY 上运行。

## 8. 关键时序参数 (v2.1)

| 参数 | 说明 | 最小 | 最大 | 单位 |
|------|------|------|------|------|
| **t3-PREPARE** | TX 在 HS 前驱动 LP-000 的时间 | 38 | 95 | ns |
| **t3-TERM-EN** | RX 使能 HS 终端的时间 | — | 38 | ns |
| **tHS-EXIT** | HS Burst 后驱动 LP-111 的时间 | 100 | — | ns |
| **tLPX** | 任意 LP 状态最短持续时间 | 50 | — | ns |
| **tINIT** | 初始化 Stop 状态持续时间 | 100 | — | µs |
| **t3-PREBEGIN** | Preamble 首段长度 | 7 | 448 | UI |
| **t3-POST(TX)** | TX 发送 Post 序列长度 | 7 | 224 | UI |

## 相关页面

- [视频显示/MIPI 概述](视频显示/MIPI 概述.md) — MIPI 家族全景与 D-PHY/C-PHY 关系
- [视频显示/MIPI D-PHY](视频显示/MIPI D-PHY.md) — 差分物理层对比
- [视频显示/MIPI DSI](视频显示/MIPI DSI.md) — 显示串行接口（可使用 C-PHY）
- [视频显示/HDMI 物理层](视频显示/HDMI 物理层.md) — HDMI TMDS 物理层（同类参照）
- [2026-06-30 - MIPI C-PHY Specification v2.1](2026-06-30 - MIPI C-PHY Specification v2.1.md) — 来源摘要
