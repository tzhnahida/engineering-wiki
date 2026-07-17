---
type: entity
tags: [electronics, usb, type-c, controller, pd]
created: 2026-06-07
updated: 2026-07-15
---

# HUSB320
> Hynetek USB Type-C 端口控制器（CC 逻辑专用，非 PD 协议引擎）。USB-C Rev 2.1，I2C+GPIO 接口。QFN-12（1.6×1.6mm）。
> 本次改写修正：引脚映射完全重写（原版 12 个引脚全错）；撤消"PD 3.0"标记（手册仅覆盖 CC 层）。

## 1. 身份与选型

| 项目 | 内容 |
|------|------|
| 型号 | HUSB320 |
| 核心功能 | USB Type-C CC 逻辑控制器 — 检测连接方向/角色、广告电流、附件识别 |
| 关键卖点 | 超小 QFN-12（1.6×1.6mm）、28V VBUS 耐压、无电池支持、15 μA 级功耗 |
| 封装选项 | BA000：VBUS_DET 直连（0Ω），内部放电+Safe0V 检测；BA001：VBUS_DET 经 866kΩ 分压，无放电/Safe0V |
| 替代参考 | FUSB302B / STUSB4500（注意后者含 PD 协议栈，HUSB320 需外接 MCU 实现 PD） |

**并非 PD 控制器**：HUSB320 仅完成 CC 物理层检测与角色状态机，PD 协议协商需在 MCU 侧通过 I2C 读取连接结果后自行实现。

## 2. 极限工况

> [!warning] 超过即永久损坏
> CC1/CC2/VBUS_DET/ID 引脚耐压 28V DC，其余数字引脚限 6.5V。

| 参数 | 最小 | 最大 | 单位 |
|------|------|------|------|
| VDD, PORT/DEBUG_N, ADDR/ORIENT, EN_N, INT_N/OUT3 | −0.3 | 6.5 | V |
| CC1, CC2, VBUS_DET, ID | −0.3 | 28 | V |
| SDA/OUT1, SCL/OUT2 | −0.3 | 6.5 | V |
| 结温（T_J） | −40 | 125 | °C |
| ESD HBM（CC1/CC2/VBUS_DET） | — | ±4000 | V |
| ESD HBM（其它引脚） | — | ±2000 | V |
| ESD CDM | — | ±500 | V |
| 热阻 θ_JA（QFN-12） | — | 164.4 | °C/W |
| 热阻 θ_JC | — | 47.1 | °C/W |

## 3. 推荐工作条件

| 参数 | 最小 | 典型 | 最大 | 单位 |
|------|------|------|------|------|
| VDD 供电 | 2.85 | — | 5.5 | V |
| VBUS_DET 输入（分压后） | 4 | — | 22 | V |
| I2C 总线上拉电压（无独立 VDDIO 引脚） | 1.2 | — | 3.3 | V |
| 环境温度 T_A | −40 | — | 85 | °C |
| 结温 T_J | −40 | — | 125 | °C |

**UVLO**：VDD 上升阈值 2.75~2.9 V（typ 2.9 V），迟滞 80 mV。低于 UVLO 时所有引脚（除 Sink/DRP 模式 CC 外）进入 Hi-Z，CC 引脚进入 Dead Battery 钳位。

## 4. 功耗与供电特性

| 参数 | 条件 | 最小 | 典型 | 最大 | 单位 |
|------|------|------|------|------|------|
| VDD 漏电流（禁用） | VDD>UVLO 且 EN_N=High，或 VDD<UVLO−HYS | — | — | 5 | μA |
| 待机电流 | 未连接，Sink/Source/DRP 配置 | — | 15 | 30 | μA |
| 工作电流 | 已连接为 Sink 或 Source | — | 15 | 30 | μA |
| VBUS 检测上升阈值 | VBUSOK 位置位 | 3.67 | 4 | 4.4 | V |
| VBUS 检测迟滞 | VBUSOK 清零 | — | 0.7 | — | V |
| VBUS 去抖时间 | 有效 VBUSOK 标志 | 250 | 375 | 500 | μs |
| vSafe0V 下降阈值 | 判定 VBUS 已放电至安全电压 | — | 0.8 | — | V |
| vSafe0V 去抖 | 同 VBUSOK | 250 | 375 | 500 | μs |
| VBUS_DET 放电电阻 | 断开事件发生时 | — | 2 | — | kΩ |

**典型功耗约 67.5 μW（4.5V × 15 μA）**，适合电池供电设备。

## 5. 通信/接口特性

### 5.1 I2C 模式

| 参数 | 值 |
|------|-----|
| 从机地址 | 42H（ADDR 接 GND） 或 62H（ADDR 接 VDD） |
| 时钟频率 | 标准/快速模式，最高 400 kHz |
| 总线电压 | 兼容 1.2V / 1.8V / 3.3V 电平 |
| 写操作 | START + SlaveAddr(W) + RegAddr + Data + STOP |
| 读操作 | START + SlaveAddr(W) + RegAddr + (Re)START + SlaveAddr(R) + Data + NAK + STOP |

> [!warning] 地址易错点：手册的 42H/62H 是含 R/W 位的 8 位地址（bit 格式 `01 ADDR 0001 R/W`），对应 **7 位地址 0x21 / 0x31**。多数 MCU HAL 使用 7 位地址，直接填 0x42/0x62 会通信失败。

> [!warning] 上电陷阱：CONTROL1[3]（ENABLE）默认 0b——I2C 模式下状态机停在 I2CDisable，MCU 初始化必须写 CONTROL1[3]=1b 才开始 Type-C 检测。另 EN_N 拉低后需等待 tEN（最长 100 ms）I2C 才可访问。

### 5.2 GPIO 模式

当 ADDR/ORIENT 浮空上电时进入 GPIO 模式，三引脚编码输出连接状态（下表为 BA000/BA001 出货配置，即"禁用 ADDR/PORT 输出模式"，对应手册 Table 12）：

| ID | OUT1 | OUT2 | 含义 |
|----|------|------|------|
| Hi-Z | Hi-Z | Hi-Z | 未连接 |
| Hi-Z | Hi-Z | Low | Sink — Default Rp（80 μA） |
| Hi-Z | Low | Hi-Z | Sink — 1.5 A Rp（180 μA） |
| Hi-Z | Low | Low | Sink — 3 A Rp（330 μA） |
| Low | Hi-Z | Low | Source — Default Rp |
| Low | Low | Hi-Z | Source — 1.5 A Rp |
| Low | Low | Low | Source — 3 A Rp |

OUT3 标示音频附件，ORIENT 标示 CC 方向，DEBUG_N 标示调试配件（后两者在出货配置下禁用输出，仅作输入采样）。

### 5.3 CC 电气参数

| 参数 | 最小 | 典型 | 最大 | 单位 |
|------|------|------|------|------|
| 内部 Rd（Sink 下拉） | 4.6 | 5.1 | 5.6 | kΩ |
| Default Rp 电流（Source） | 64 | 80 | 96 | μA |
| 1.5 A Rp 电流 | 166 | 180 | 194 | μA |
| 3 A Rp 电流 | 304 | 330 | 356 | μA |
| 禁用时 CC 阻抗 | 1 | — | — | MΩ |

## 6. 核心功能

![[_llm/raw/assets/datasheets/husb320/husb320_p11_fig1.jpg|560]]
*Figure 4 — 功能方框图*

### 6.1 CC 检测原理（Rp / Rd / Ra 组合解码）

USB-C 连接器的两端通过 CC 引脚上的终止电阻组合确定角色与电流能力：

| 本端角色 | 本端终止 | 对端终止 → 判定 |
|---------|----------|----------------|
| **Source** | Rp 电流源（80/180/330 μA） | Rd（5.1 kΩ）→ 连接为 Source；Ra（1 kΩ）→ eMarker 线缆；开路 → 未连接 |
| **Sink** | Rd（5.1 kΩ） | Rp 电流源 → 连接为 Sink，电流大小由 Rp 值反映；Ra → 音频/调试配件 |
| **DRP** | 交替 Rp / Rd（见 6.2） | 检测到对端 Rd → 本端成 Source；检测到 Rp → 本端成 Sink |

**电流广告三档**（Sink 端由 CC 电压判决，阈值为手册 typ 值）：

| Rp 设定 | Sink 端 CC 电压区间（typ） | 对应 VBUS 电流能力 |
|---------|--------------------------|-------------------|
| Default（80 μA） | 0.2 ~ 0.66 V | 默认 USB（500 mA / 900 mA） |
| 1.5 A（180 μA） | 0.66 ~ 1.23 V | 1.5 A |
| 3 A（330 μA） | 1.23 ~ 2.11 V | 3 A |

BC_LVL 变化去抖时间由 USER_CFG[1:0] 配置（0.5/3/12/18 ms，默认 12 ms）。

> [!warning] 广告电流仅表示端口能力，实际电流受线缆、电源和 PD 协商约束。HUSB320 不执行 PD 协商，电流限制需 MCU 或独立充电 IC 实施。

### 6.2 角色状态机

HUSB320 实现完整的 USB-C 端口角色状态机，五种模式通过 PORTROLE 寄存器（0x03）或硬件引脚设定。

| 模式 | PORTROLE[2:0] | 行为 |
|------|--------------|------|
| **Source Only** | 001b | 持续发送 Rp，等待 Rd 接入后进入 Attached.SRC |
| **Sink Only** | 010b | 持续下拉 Rd，检测到 Rp 后进入 Attached.SNK |
| **DRP** | 100b（最高位优先） | CC 引脚交替上拉 Rp 和下拉 Rd，周期 60~90 ms（可配）；侦听到对端终止后锁定角色 |
| **Try.SRC** | DRP + TRY[5:4]=10b | DRP 基础上优先尝试作为 Source：检测到 Rd 时提前锁定 Source 角色 |
| **Try.SNK** | DRP + TRY[5:4]=01b | DRP 基础上优先尝试作为 Sink：检测到 Rp 时提前锁定 Sink 角色 |

**迁移逻辑要点**：
- 角色切换（通过 I2C 改写 PORTROLE）会触发 ErrorRecovery 状态，完成序列后进入新角色
- DRP 占空比通过 CONTROL[5:4]（地址 0x04）配置：60/40、50/50、40/60、30/70（SNK%/SRC%）
- 支持切换占空比动态调整（RESET[4] 位，每 500 ms 翻转）
- tTryCCDebounce：10~20 ms 窗口，Try.* 模式判定窗口

### 6.3 无电池（Dead Battery）模式

当 VDD < UVLO 阈值（约 2.75 V）时，HUSB320 进入 Dead Battery 状态：

- CC1/CC2 引脚对地向 Rd 方向钳位，不主动上拉
- **若对端 Source 通过 Rp 供电**，CC 线电压被钳位在 VSNKDB 范围：
  - Default Rp（80 μA）：0.25~1.5 V
  - 1.5 A Rp（180 μA）：0.45~1.5 V
  - 3 A Rp（330 μA）：0.85~2.18 V
- 此钳位电压足以让对端 Source 判定连接有效，从而在 VBUS 上建立电源
- 系统 VDD 上电 → UVLO 释放 → HUSB320 初始化 → 进入正常 Sink 操作
- 除 EN_N 检测外，Dead Battery 期间无其它功能运行

### 6.4 VBUS_DET 分压监测

VBUS_DET 引脚支持两种配置（由订购型号决定）：

- **BA000（直连 0Ω）**：VBUS_DET 直接连到 VBUS。片内比较器直接检测 VBUS 电压，生成 VBUSOK 和 vSafe0V 标志。断开时通过内部 2 kΩ 电阻放电。
- **BA001（866kΩ 分压）**：VBUS 经外部 866 kΩ 串联电阻分压后送入 VBUS_DET。**仅提供 VBUSOK 检测**；vSafe0V 被强制为 1（始终安全），内部放电通路禁用。断开后 VBUS 需外接泄放电阻。

> [!warning] 选型时注意：BA001 无法感知 VBUS 放电完成，不适合需要 vSafe0V 信号的应用（如 PD 协议要求的 VBUS 安全放电）。

**AUTOSNK 自动切换**：当 VDD 跌落到设定阈值（3.0~3.3 V，由 CONTROL1[6:5] 选择）时，AUTOSNK 可将端口临时切换至 Sink 角色请求 VBUS 供电，VDD 恢复后自动切回原角色。

### 6.5 附件检测

HUSB320 通过 CC 线 Ra 终止识别两种附件：

- **音频适配器（Audio Accessory）**：CC1/CC2 均为 Ra（1 kΩ）。芯片进入 AudioAccessory 状态，OUT3 拉低指示（GPIO 模式），TYPE[0]（AUDIO）= 1。
- **调试配件（Debug Accessory）**：CC1 或 CC2 为 Ra。ORIENT 检测后可进入 DebugAccessory.SNK/SRC，TYPE[6]（DEBUGSRC）或 TYPE[5]（DEBUGSNK）置位。

音频/调试附件检测可通过 PORTROLE[3]（AUDIOACC）位关闭。

### 6.6 I2C 寄存器体系速查（核心寄存器）

| 地址 | 名称 | 类型 | 功能 | 默认 |
|------|------|------|------|------|
| 0x01 | DEVICE ID | R | VER_ID[7:4] + REV_ID[3:0] | 0x10 |
| 0x02 | DEVICE TYPE | R | 设备型号标识 | 0x03 |
| 0x03 | **PORTROLE** | R/W | 角色配置（DRP/SNK/SRC）+ Try 机制 + 音频附件使能 | 由引脚采样决定 |
| 0x04 | **CONTROL** | R/W | tDRP 周期 / 占空比 / Rp 电流选择 / 全局中断掩码 | 0x41（70ms, 60/40, 中断全掩码） |
| 0x05 | **CONTROL1** | R/W | AUTOSNK 阈值与使能 / I2C 使能 / 连接去抖时间 | 0x23（3.1V, AUTOSNK 关, 150ms） |
| 0x09 | **MANUAL** | WC+R/W | 强制状态跳转（ER/Disable/Unatt.SRC/Unatt.SNK/Force） | 0x00 |
| 0x0A | RESET | WC | 软复位 / tDRPTry 设置 / 使能占空比切换 | 0x20 |
| 0x0E | **MASK** | R/W | 中断源掩码（ATTACH/DETACH/BC_LVL/AUTOSNK/VBUS/FAULT/ORIENT） | 0x00（全不掩码） |
| 0x0F | MASK1 | R/W | Force 操作结果掩码 | 0x00 |
| 0x11 | **STATUS** | R | AUTOSNK / VSAFE0V / ORIENT[1:0] / VBUSOK / BC_LVL[1:0] / ATTACH | 0x00 |
| 0x13 | **TYPE** | R | 当前连接类型（SRC/SNK/DEBUG/AUDIO/ACTIVECABLE） | 0x00 |
| 0x14 | **INTERRUPT** | R/W | 中断标志（写 1 清零）：ATTACH/DETACH/BC_LVL/VBUS_CHG/AUTOSNK/ORIENT | 0x00 |
| 0x15 | INTERRUPT1 | R/W | I_FRC_SUCC / I_FRC_FAIL | 0x00 |
| 0x16 | USER CFG | R/W | BC_LVL 去抖 / VBUS 放电超时 / CC 断线监测 | 取决于工厂配置 |

### 6.7 MCU 对接流程（中断驱动）

推荐 I2C 模式下 MCU 对接工作流：

```
1. 初始化（EN_N 拉低后等待 tEN ≤ 100 ms）：
   └─ 配置 PORTROLE（Source/Sink/DRP）+ CONTROL（Rp 电流 / tDRP / INT_MASK=0）
      ├─ 使能 MASK 中感兴趣的中断源（至少 ATTACH + DETACH）
      └─ 写 CONTROL1[3]=1b（ENABLE）启动 Type-C 状态机

2. 连接事件（INT_N 拉低）：
   └─ MCU 读取 INTERRUPT（0x14）→ 确认 I_ATTACH
      ├─ 读取 STATUS（0x11）→ ORIENT（CC 方向）+ BC_LVL（电流等级）
      ├─ 读取 TYPE（0x13）→ SRC/SNK/AUDIO/DEBUG
      ├─ 根据结果配置系统电源路径 / USB MUX
      └─ 写 1 到 INTERRUPT 对应位 → 清中断

3. Power Role / Data Role 变更（需要时）：
   └─ 改写 PORTROLE → 自动触发 ErrorRecovery → 重新连接新角色

4. 断开事件（INT_N 拉低）：
   └─ MCU 读取 INTERRUPT → 确认 I_DETACH
      ├─ 关闭 VBUS / 断开通路
      └─ 清中断

5. 异常处理：
   ├─ I_FAULT → 检查 VBUSOK / VSAFE0V
   ├─ I_BC_LVL → 电流广告变更（如老化线缆）
   └─ I_FRC_FAIL → Force 操作未成功（对端不配合）
```

> [!note] **GPIO 模式无中断**，MCU 需轮询 OUT1/OUT2/ID 编码或监测 EN_N 引脚电平变化。

### 6.8 PCB 布局要点

- CC1/CC2 信号对噪声敏感：**走线尽量短、等长**、远离高频开关节点（DC-DC 电感、SW 节点）
- VBUS_DET 分压路径（BA001）：866 kΩ 电阻靠近 HUSB320 放置，分压节点尽少量过孔
- VDD/VDDIO 去耦电容（0.1 μF）紧贴对应引脚，回路面积最小化
- INT_N 开漏上拉电阻选 4.7~10 kΩ 至 VDDIO，避免上拉到与 SCL/SDA 不同的电压域
- QFN-12 中心散热焊盘（如有）建议接地，改善 θ_JA

## 7. 引脚与典型连线

![[_llm/raw/assets/datasheets/husb320/husb320_p4_fig1.jpg|340]]
*Figure 2 — 引脚分配（QFN-12 Top View）*

![[_llm/raw/assets/datasheets/husb320/husb320_p1_fig1.jpg|480]]
*Figure 1 — 典型应用电路*

> [!warning] **引脚表修订说明**：原版 12 个引脚编号与功能全部错位，以下为手册 Table 1 重写。

| 引脚 | 名称 | 类型 | 功能 | 闲置处理 |
|------|------|------|------|---------|
| 1 | CC1 | DIO | USB-C CC 通道 1 | — |
| 2 | CC2 | DIO | USB-C CC 通道 2 | — |
| 3 | PORT/DEBUG_N | IO | 输入：三态电设置端口角色（VDD→Source / GND→Sink / 浮空→DRP）；输出：调试配件检测（低=检出） | 900 kΩ 接 VDD/GND 或浮空 |
| 4 | VBUS_DET | AI | VBUS 电压检测输入（0Ω 直连或 866kΩ 分压），内含 2 kΩ 放电通路（BA000） | 经电阻分压来自 VBUS |
| 5 | ADDR/ORIENT | IO | 输入：三态选工作模式（VDD→I2C 62H / GND→I2C 42H / 浮空→GPIO）；输出：连接方向（低=CC1，高=CC2） | 900 kΩ 接 VDD/GND 或浮空 |
| 6 | INT_N/OUT3 | OD | I2C 模式：中断请求（低有效）；GPIO 模式：音频附件指示（低=检出） | 4.7~10 kΩ 上拉到 VDDIO |
| 7 | SDA/OUT1 | IO | I2C 模式：数据线；GPIO 模式：状态输出 1 | I2C 上拉或留空 |
| 8 | SCL/OUT2 | IO | I2C 模式：时钟线；GPIO 模式：状态输出 2 | I2C 上拉或留空 |
| 9 | ID | OD | 开漏输出：低=已连接为 Source，Hi-Z=已连接为 Sink 或未连接 | NC |
| 10 | GND | P | 地 | — |
| 11 | EN_N | AI | 使能（内部上拉至 VDD，低电平使能，高电平禁用） | 接 GND（始终使能）或 MCU GPIO 控制 |
| 12 | VDD | P | 供电 2.85~5.5 V | 0.1 μF 去耦电容 |

**典型连线**：MCU I2C → HUSB320 SDA/SCL/INT_N（中断驱动）；CC1/CC2 → USB-C 连接器（差分对等长）；VBUS_DET 分压后监测 VBUS；EN_N 接地持续使能或由 PMU 控制。

## 8. 封装

| 参数 | 值 |
|------|-----|
| 封装 | QFN-12L |
| 尺寸 | 1.6 × 1.6 mm |
| 高度 | ≈0.6 mm（手册文本未给出具体值，见 Figure 11 外形图） |
| 引脚间距 | 0.35 mm |
| 工作结温 | −40 ~ +125 °C |
| 订购 | HUSB320-BA000-QN12R / BA001-QN12R，T&R 3k |

- 接口存储 · USB Type-C 协议 · Type-C 端口控制器选型 · CC 逻辑检测
