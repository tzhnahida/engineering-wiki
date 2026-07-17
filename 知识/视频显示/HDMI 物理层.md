---
type: concept
tags: [hdmi, physical-layer, tmds, connector, pcb-design, impedance]
created: 2026-06-26
updated: 2026-07-15
sources: ["[2026-06-26 - HDMI 1.4 Specification](../../来源/2026-06-26 - HDMI 1.4 Specification.md)"]
---

# HDMI 物理层

> HDMI 物理层定义了连接器、线缆、TMDS 差分信号、DC/AC 电气特性以及辅助信道的硬件规范。本节内容基于 HDMI 1.4b 规范。

## 1. 连接器类型

HDMI 1.4 规范定义了五种连接器类型，覆盖从消费电子到汽车应用的场景。

| 类型 | 名称 | 引脚数 | 尺寸 | 用途 |
|------|------|--------|------|------|
| Type A | Standard | 19 | 13.9 mm × 4.45 mm | 电视、显示器、投影仪 |
| Type B | Dual-Link | 29 | 21.2 mm × 4.45 mm | 超高分辨率（双链路，极少使用） |
| Type C | Mini | 19 | 10.42 mm × 2.42 mm | 便携设备（摄像机、平板） |
| Type D | Micro | 19 | 6.4 mm × 2.8 mm | 超便携（手机、RPi Zero） |
| Type E | Automotive | 19 | 带锁定机构 | 汽车应用 |

所有五种连接器均传输相同的 HDMI 信号集，占用相同的 TMDS 通道数（三数据通道 + 一时钟通道），唯 Type B 例外——它配备第二条 TMDS 链路（双链路，共六数据通道 + 双时钟通道），但实际产品极少采用。

> [!note] Type B 的消亡
> Type B 定义于 HDMI 1.0，用于支持超高分辨率双链路传输。随着单链路带宽从 1.65 Gbps（HDMI 1.0）提升至 10.2 Gbps（HDMI 1.4），已无实际需求，HDMI 2.0 之后从规范中移除。

连接器插入顺序（接插时序）：**Pin 19（HPD）最先接触**，随后是 TMDS 信号引脚，最后是 +5V 电源引脚。这一时序确保 Hot Plug Detect 先建立连接，Source 检测到 Sink 后再进行 EDID 读取和 TMDS 启动。

参见：[HDMI Type A](../../元件/连接器/HDMI Type A.md)、[HDMI Type D](../../元件/连接器/HDMI Type D.md)

## 2. Type A 引脚定义

Type A 是 HDMI 最通用的连接器，19 引脚，0.5 mm 间距，排布如下：

| 引脚 | 信号 | 引脚 | 信号 | 引脚 | 信号 |
|------|------|------|------|------|------|
| 1 | TMDS Data2+ | 8 | TMDS Data0 Shield | 15 | SCL（DDC） |
| 2 | TMDS Data2 Shield | 9 | TMDS Data0- | 16 | SDA（DDC） |
| 3 | TMDS Data2- | 10 | TMDS Clock+ | 17 | DDC/CEC Ground |
| 4 | TMDS Data1+ | 11 | TMDS Clock Shield | 18 | +5V Power |
| 5 | TMDS Data1 Shield | 12 | TMDS Clock- | 19 | Hot Plug Detect |
| 6 | TMDS Data1- | 13 | CEC | |
| 7 | TMDS Data0+ | 14 | Utility/HEAC+ | |

**信号分组：**

- **TMDS 通道** — 四个差分对（Data0-2 + Clock），每个差分对独立屏蔽，共占 12 个引脚
- **DDC 总线** — SCL/SDA（I²C），用于 EDID 读取和 HDCP 认证
- **CEC** — 消费电子控制，单线双向
- **HEAC** — HDMI Ethernet 模拟音频通道（HDMI 1.4 引入），复用 Pin 14 和 Pin 17
- **电源** — +5V（Pin 18，Source 提供）和 HPD（Pin 19，Sink 驱动）

## 3. 线缆分类

HDMI 线缆按最大 TMDS 时钟频率分为两类：

| 类别 | 最大 TMDS 时钟 | 最高分辨率（典型） | 市场名称 |
|------|---------------|-------------------|----------|
| Category 1 | 74.25 MHz | 1080i / 720p | Standard HDMI |
| Category 2 | 340 MHz | 1080p / 4K×2K@30Hz / 3D | High Speed HDMI |

配备 HEAC 支持的线缆对应升级命名：

- Category 1 + HEAC → "Standard HDMI with Ethernet"
- Category 2 + HEAC → "High Speed HDMI with Ethernet"

**线缆适配器配置**：规范定义了以下适配器组合的电气要求——A-A、A-C、A-D、C-C、E-E、E-A、C-A、D-A 等，每种适配器须保持 100 Ω 差分阻抗连续性和屏蔽完整性。

## 4. TMDS 电气特性

[📷 _llm/raw/assets/standards/hdmi14/hdmi14_p68_fig1.jpg|540]
*Figure 4-22 — 单对 TMDS 差分线概念原理图：源端电流舵 + 接收端 3.3V 上拉 50Ω 端接*


TMDS（Transition Minimized Differential Signaling）是 HDMI 的核心物理层传输技术。所有电气参数均在两个测试点定义：TP1（Source 端）和 TP2（Sink 端）。

### 4.1 Source（TP1）直流特性

| 参数 | 条件 | 最小值 | 典型值 | 最大值 |
|------|------|--------|--------|--------|
| 单端输出摆幅 | 50 Ω to 3.3 V 终端 | 400 mV | 500 mV | 600 mV |
| 共模电压 | 相对 AVcc | -200 mV | — | +10 mV |
| AVcc 电源电压 | — | 3.135 V | 3.3 V | 3.465 V |

- 输出摆幅指差分对中每个单端信号相对 GND 的峰峰值
- 共模电压以 AVcc（3.3 V）为参考，允许略微低于 AVcc 但不得高于
- AVcc = 3.3 V ± 5%

### 4.2 Source（TP1）交流特性

| 参数 | Category 1 | Category 2 |
|------|-----------|-----------|
| 上升/下降时间（20%-80%） | 75 ps ≤ Tr/tf ≤ 0.4 Tbit | 75 ps ≤ Tr/tf ≤ 0.3 Tbit |
| 时钟占空比 | 40%-60% | 40%-60% |
| 眼图模板 | 定义于规范 | 定义于规范 |
| 数据率 ± 时钟比 | 10:1 | 10:1 |

- Tbit = 1 /（10 × TMDS_clock）。当 TMDS 时钟为 340 MHz 时，Tbit ≈ 294 ps
- 上升/下降时间控制对 EMI 和信号完整性至关重要——过陡增加 EMI，过缓恶化眼图
- 眼图模板定义了最小眼高和眼宽，Source 输出必须闭合于模板外

### 4.3 Sink（TP2）直流特性

| 参数 | 条件 | 最小值 | 最大值 |
|------|------|--------|--------|
| 单端终端阻抗 | 对 AVcc（3.3 V） | 45 Ω | 55 Ω |
| 差分输入阻抗 | — | 90 Ω | 110 Ω |
| AVcc 电压 | — | 3.135 V | 3.465 V |

- 终端阻抗精确匹配决定了信号反射系数。Sink 端的 50 Ω ± 10% 单端阻抗（100 Ω ± 10% 差分）是链路信号完整性的基础

### 4.4 Sink（TP2）交流特性

| 参数 | 最小值 |
|------|--------|
| 差分输入电压灵敏度 | ≤ 150 mVp-p |
| 眼图容差 | 比 Source 眼图模板更宽松 |

- Sink 必须能可靠检测低至 150 mV 的差分信号，而 Source 最小输出为 400 mV，留出约 250 mV 的电缆和连接器损耗预算

## 5. 信号完整性关键参数

[📷 _llm/raw/assets/standards/hdmi14/hdmi14_p74_fig2.jpg|480]
*Figure 4-30 — TP1 源端眼图模板：归一化幅度 vs 0.5×Tbit 时间窗*


**阻抗体系：**

| 参数 | 值 | 说明 |
|------|-----|------|
| TMDS 差分阻抗 | 100 Ω ± 10% | 每个 TMDS 对 |
| 单端阻抗 | 50 Ω ± 10% to 3.3 V | 每根走线 |
| DDC（I²C）阻抗 | 非受控 | 低速总线，无需阻抗匹配 |
| CEC | 非受控 | 低速单线总线 |

**时钟与数据率：**

- **TMDS 时钟频率** = 像素时钟频率（≤ 340 MHz for Category 2）
- **每通道数据率** = 10 × TMDS 时钟（单通道最高 3.4 Gbps）
- **总带宽** = 3 × 3.4 = 10.2 Gbps（三个数据通道合计）
- 一个 Tbit 周期传输 1 bit 数据，10 个 Tbit = 1 像素时钟周期

**编码机制概述**（参见 [视频显示/HDMI TMDS 编码](HDMI TMDS 编码.md)）：

每个 TMDS 通道在 1 个像素时钟周期内传输 10 bits 编码数据，其中包含 8 bits 视频/控制数据 + 2 bits 编码开销。编码算法将 8 bit 数据转换为最小化跳变次数的 10 bit 符号，同时保证足够的 DC 平衡。

## 6. DDC 通道

| 参数 | 值 |
|------|-----|
| 协议基础 | I²C（SCL + SDA） |
| 功能 | EDID 读取、HDCP 认证 |
| 电压 | 5 V（由 Source Pin 18 +5V 供电） |
| Sink/Repeater 电容 | ≤ 50 pF |
| 线缆电容 | ≤ 700 pF |
| 上拉电阻 | Source 端集成，通常 1.5 kΩ-2.2 kΩ to 5 V |

DDC 通道在 HDMI 连接中的作用：

1. Source 检测到 HPD 高电平
2. Source 通过 DDC 读取 Sink 的 EDID（扩展显示识别数据）
3. Source 根据 EDID 选择最佳视频格式
4. 若启用 HDCP，通过 DDC 完成认证握手
5. 认证通过后启动 TMDS 传输

电容限制（≤ 50 pF in Sink）确保 I²C 在 100 kHz 标准速率下信号完整。700 pF 线缆电容大致对应 10 米以上的优质线缆。

## 7. CEC 线

[📷 _llm/raw/assets/standards/hdmi14/hdmi14_p169_fig1.jpg|560]
*Figure 8-5 — CEC 与 DDC 的连接拓扑：CEC 单线菊花链贯穿所有设备，DDC 点对点*


CEC（Consumer Electronics Control）是一条单线双向总线，用于设备间命令控制。

| 参数 | Source | Sink | 线缆 |
|------|--------|------|------|
| 最大互联电阻 | 5 Ω | 5 Ω | 合计 |
| 断电泄漏电流 | ≤ 1.8 µA | ≤ 1.8 µA | — |
| 电容 | ≤ 150 pF | ≤ 200 pF | ≤ 700 pF |

CEC 工作在 3.3 V 逻辑电平，通过开源驱动器实现多主机总线仲裁。物理层采用线与（wired-AND）拓扑，设备可在任意时刻开始传输，通过隐式仲裁解决冲突。

## 8. HPD 线

HPD（Hot Plug Detect）是 Sink 驱动的状态指示信号。

| 状态 | 电压 | 含义 |
|------|------|------|
| 高电平 | > 2.0 V | Sink 已连接且就绪 |
| 低电平 | < 0.8 V | Sink 断开或未就绪 |

- Sink 通过上拉电阻将 HPD 拉至 +5 V（由 Source 的 Pin 18 供电）
- Source 检测 HPD 上升沿后启动 DDC 读取流程
- HPD 低电平超过 100 ms 触发 Source 重检测
- HPD 还可用于传输 HEAC 活动指示

## 9. 鲁棒性要求

HDMI 规范对物理层的可靠性提出了明确的容错要求：

- **任意引脚间短路不得损坏设备** — 包括信号引脚对信号引脚、信号对电源、信号对地
- **Source-Source 直连不得损坏** — 两个 Source 设备通过线缆对接时不会损坏任一方
- **Sink-Sink 直连不得损坏** — 两个 Sink 设备通过线缆对接时不会损坏任一方

这些要求在实际设计中意味着：

- 所有 TMDS 输出端必须具有短路保护
- +5V 引脚必须有过流保护
- ESD 保护器件（如 [USBLC6-2SC6](../../元件/分立器件/USBLC6-2SC6.md)）应放置在连接器近端，钳位电压需匹配 3.3 V TMDS 共模

## 10. PCB 设计要点

基于 HDMI 物理层电气特性，在实际板级设计时需重点关注以下方面：

### 10.1 TMDS 差分对走线

- **目标阻抗**：100 Ω 差分 ± 10%（配合层叠结构计算线宽线距）
- **等长匹配**：同一差分对内等长偏差 ≤ 5 mil；四对 TMDS 之间等长偏差 ≤ 10 mm（优先参考时钟通道）
- **参考平面**：TMDS 走线必须紧邻连续 GND 参考层，不得跨分割
- **过孔限制**：每个 TMDS 对最多 2 个过孔（Source → Connector），过孔引入的阻抗不连续需通过回流路径优化和背钻控制

### 10.2 常见 4 层板阻抗设计参考

| 层叠 | TMDS 线宽/线距 | 参考平面 |
|------|----------------|----------|
| 4 层（SGPG） | ~6 mil / 8 mil（JLC 7628 半固化片） | L2 GND |
| 4 层（GSPG）表面微带 | ~6 mil / 8 mil | L2 GND |
| 6 层以上带状线 | 内层微带或带状线设计 | 上下 GND |

> 实际值需根据 PCB 厂家的叠层结构计算，以上值仅作参考。

### 10.3 布局与器件放置

- **ESD 保护**：[USBLC6-2SC6](../../元件/分立器件/USBLC6-2SC6.md) 或同类 TVS 二极管，每对 TMDS 一个，布局在连接器 5 mm 以内
- **AC 耦合电容**：TMDS 差分对串联 0.1 µF 电容（HDMI 规范要求的 DC 去耦位置在 Source 端内部；若芯片手册要求在板级放置，则以 0402/0201 封装紧靠芯片引脚）
- **DDC 上拉**：SCL/SDA 上拉电阻至 5 V（非 3.3 V），电阻值根据总线电容选择（典型 1.5 kΩ-2.2 kΩ）
- **+5V 滤波**：Pin 18 需 LC 滤波（10 µH + 10 µF），注意电感额定电流 ≥ 500 mA
- **HPD 处理**：HPD 信号可能需要电压比较器或施密特触发器整形（避免噪声误触发）

### 10.4 常见设计陷阱

| 问题 | 后果 | 对策 |
|------|------|------|
| 差分对不等长 | 时序偏移，眼图闭合 | 蛇形绕线补偿 |
| 跨分割参考面 | 阻抗突变 + EMI | 保证完整 GND 层 |
| 过孔阻抗不连续 | 回损增加 | 最小化过孔数，GND 回流过孔伴行 |
| ESD 器件寄生电容过大 | 高频衰减 | 选择 Cj ≤ 0.5 pF 的 TVS |
| 连接器 100 Ω 不匹配 | 反射 | 选型时确认连接器差分阻抗 |
| 线缆过长（>15m Cat2） | 损耗过大，眼图闭合 | 使用均衡器或中继器 |

## See Also

- [视频显示/HDMI 协议概述](HDMI 协议概述.md) — HDMI 协议栈（数据岛、视频周期、控制周期）
- [视频显示/HDMI TMDS 编码](HDMI TMDS 编码.md) — TMDS 8b10b 编码算法与 DC 平衡
- [HDMI Type A](../../元件/连接器/HDMI Type A.md) — Type A 连接器实体页（含封装信息）
- [HDMI Type D](../../元件/连接器/HDMI Type D.md) — Type D Micro 连接器实体页
- [TC358870](../../元件/接口存储/TC358870.md) — HDMI 1.4 到 MIPI DSI 桥接芯片
