---
type: concept
tags: [electronics, power-supply, dc-dc, switching-regulator]
created: 2026-06-07
updated: 2026-07-15
sources: ["[2026-06-07 - Datasheet Collection](../../来源/2026-06-07 - Datasheet Collection.md)"]
  - SiC46x 数据手册
---

# DC-DC 降压转换器

降压转换器（Buck Converter）是一种开关稳压器，将较高的输入电压转换为较低的输出电压，效率远高于线性稳压器（LDO）。

## 基本原理

[📷 _llm/raw/assets/datasheets/tps54302/tps54302_p1_fig1.jpg|480]
*同步降压最小系统（TPS54302 首页简化原理图）：上/下管 + 电感 + 输入/输出电容 + 反馈分压*


通过高低边 MOSFET 的交替开关，配合电感和输出电容，将输入电压斩波并滤波为稳定的直流输出。占空比决定输出电压：

$$D = \frac{V_{OUT}}{V_{IN}}$$

## 关键拓扑类型

- **同步整流** — 低边用 MOSFET 替代二极管，效率更高（如 [SiC46x](../../元件/电源管理/SiC46x.md)）
- **非同步** — 低边用肖特基二极管，成本更低

## 控制方案

[📷 _llm/raw/assets/datasheets/tps54302/tps54302_p9_fig1.jpg|560]
*峰值电流模式控制实例（TPS54302 功能模块图）：电流采样 + 斜率补偿 + 误差放大器构成双环*


| 方案 | 特点 | 代表器件 |
|------|------|---------|
| 电压模式 PWM | 成熟稳定，需补偿 | — |
| 电流模式 PWM | 逐周期限流，响应快 | — |
| **Constant On-Time (COT)** | 超快瞬态响应，无需复杂补偿 | [SiC46x](../../元件/电源管理/SiC46x.md) |
| 滞回控制 | 简单，纹波大 | — |

## 关键设计参数

- **效率** = P_OUT / P_IN（SiC46x 可达 98%）
- **纹波电流** = (V_IN - V_OUT) × t_ON / L
- **开关频率**：高频 → 小体积，但开关损耗增加
- **环路补偿**：保证稳定性和瞬态响应

## See Also

- [SiC46x](../../元件/电源管理/SiC46x.md) — 实体页（Vishay microBUCK 系列）
- [电源电子/恒定导通时间控制](恒定导通时间控制.md) — COT 控制方案详解
