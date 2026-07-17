---
type: entity
tags: [electronics, transistor, npn, general-purpose]
created: 2026-06-07
updated: 2026-07-15
---

# MMBT2222A

## 1. 身份选型

NPN 通用小信号三极管，SOT-23 封装。两种主要变体：

| 参数 | L (MMBT2222AL) | AL (MMBT2222A) |
|------|---------------|----------------|
| VCEO | 30V | 40V |
| VCBO | 60V | 75V |
| IC | 600mA | 600mA |
| hFE | 100–300 | 100–300 |

汽车级型号为 SMMBT2222AL (AEC-Q101)。广泛应用于开关、信号放大、逻辑电平转换。

## 2. 极限工况

| 参数 | L | AL |
|------|---|---|
| VCEO | 30V | 40V |
| VCBO | 60V | 75V |
| VEBO | 5V | 6V |
| IC | 600mA |
| 总功耗 | 350mW (SOT-23) |
| 工作/存储温度 | -55 – +150°C |

## 3. 推荐工作条件

- **IC**: ≤ 500mA（留余量）
- **VCE**: ≤ VCEO - 5V
- **基极驱动**: 确保 IC / hFE_min × 2~3 的 IB
- **温度范围**: -55 – +150°C

## 4. 功耗热特性

- **PD (最大)**: 350mW @25°C
- **RthJA**: 约 357°C/W (SOT-23)
- **降额**: 2.8mW/°C 高于 25°C

## 5. IO/电气特性

| 参数 | 条件 | L 典型 | AL 典型 |
|------|------|--------|---------|
| hFE | VCE=10V, IC=150mA | 100–300 | 100–300 |
| VCE(sat) | IC=150mA, IB=15mA | 0.3V | 0.3V |
| VBE(sat) | IC=150mA, IB=15mA | 0.6–1.2V | 0.6–1.2V |
| ICBO | VCB=50V | 50nA | 50nA |
| fT | VCE=20V, IC=20mA | 300MHz | 300MHz |

## 6. 核心功能

![[_llm/raw/assets/datasheets/mmbt2222a/mmbt2222a_p3_fig6.jpg|620]]
*Figure 3 — 直流电流增益 hFE 随 IC 变化曲线*


- **通用开关**: 典型的 NPN 开关管，饱和压降低 (< 0.3V)。
- **信号放大**: hFE 100–300 线性区，适用于小信号放大。
- **逻辑电平转换**: MCU 3.3V → 5V/12V 转换接口。
- **互补对**: 可与 MMBT2907A (PNP) 配对使用。

## 7. 引脚

SOT-23 标准引脚：

| 引脚 | 名称 | 功能 |
|------|------|------|
| 1 | B (Base) | 基极 |
| 2 | E (Emitter) | 发射极 |
| 3 | C (Collector) | 集电极 |

## 8. 封装

- **SOT-23**: 2.92×1.30mm，3 引脚
- 汽车级：SMMBT2222AL (AEC-Q101)
