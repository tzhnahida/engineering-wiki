---
type: concept
tags: [zupt, zero-velocity, probabilistic, soft-zupt, eskf, foot-mounted, imu]
created: 2026-06-27
updated: 2026-06-27
sources:
  - "Gao & Deng, Posterior-Contact Soft-ZUPT for Foot-Mounted Inertial Navigation: Uncertainty-Aware Pseudo-Observation Modeling, Sensors 26(10):3033, 2026. DOI: 10.3390/s26103033"
---

# Soft-ZUPT · 概率零速修正

> 着地不是瞬间完成的。从"正在着地"到"完全静止"之间，ZUPT 应该给连续变化的置信度，而非 0/1 二值。北京邮电大学 2026。

## 问题

传统 ZUPT 的修正逻辑是硬的：

```
if 静止检测器说你是静止的:
    速度 = 0（硬修正，100% 信任）
else:
    不修正（0% 信任）
```

这有两个问题：

1. **Heel-strike（足跟着地瞬间）**：脚刚触地但还没踏实，速度接近零但不是零。硬修正会把剩余的小速度强行拉到零 → 引入冲击误差
2. **Toe-off（脚尖离地瞬间）**：同理，硬切会产生相反方向的冲击

## 核心思路

把 ZUPT 建模为**概率伪观测**，修正强度由两步决定：

### 第一步：接触先验（Contact Prior）

检测器输出不是 0/1，而是 **p(静止)**——一个 0 到 1 之间的概率：
- 完全静止（flat foot）：p ≈ 0.98
- 足跟着地（heel-strike）：p ≈ 0.6
- 脚尖离地（toe-off）：p ≈ 0.4
- 空中（swing）：p ≈ 0.02

### 第二步：创新后验（Innovation Posterior）

ESKF 的预测速度 v̂ 和零之间的差距 e = 0 − v̂。如果预测速度已经很小（10cm/s），说明状态估计本身就认为脚快停了 → 提高修正置信度。如果预测速度很大（1m/s），说明估计器认为在跑 → 即使检测器说静止，也降低修正。

### 融合

```
修正强度 = p(静止) × p(e 在合理范围内)
    ↓
ESKF 测量更新时，R 矩阵不是固定值，而是 1/修正强度
    → 高置信度 = 小 R = 强修正
    → 低置信度 = 大 R = 弱修正
```

## 关键数据

- 56 次足部 VICON 基准测试
- 对比对象：hard-ZUPT、robust soft-ZUPT、contact soft-ZUPT、FIBA-like
- Soft-ZUPT 在困难试次（转身/急停）的**尾部误差**（p95）显著优于所有基线
- 在单 ESKF 框架内完成，不需要额外的滤波器

## 和 AWGF-ZVD 的关系

| | AWGF-ZVD | Soft-ZUPT |
|------|---------|---------|
| 改进什么 | **检测器**：什么时候判静止 | **修正器**：怎么利用静止信息 |
| 输出 | 更好的阈值函数 T(f) | 连续的修正强度 p × innovation |
| 传统对应 | GLRT 固定阈值 | 硬 ZUPT 速度归零 |
| 可以一起用？ | ✅ 互补：AWGF 给更好的二值检测，Soft 把二值软化 |

## 对本项目的意义

相位 2 ZUPT 模块的理想组合：**AWGF-ZVD 做检测 + Soft-ZUPT 做修正**。在单脚节点上验证后，可以扩展到双脚节点（左脚+右脚交替 ZUPT → 双重观测）。

## 参见

- [自适应步频零速检测](自适应步频零速检测.md) — AWGF-ZVD 检测器增强
- [2026-06-24 - Skog 零速检测算法评估](../../来源/2026-06-24 - Skog 零速检测算法评估.md) — 传统检测器
- [误差状态卡尔曼滤波](../姿态解算/误差状态卡尔曼滤波.md) — ESKF 基础
