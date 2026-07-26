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

## 参见

- [自适应步频零速检测](%E8%87%AA%E9%80%82%E5%BA%94%E6%AD%A5%E9%A2%91%E9%9B%B6%E9%80%9F%E6%A3%80%E6%B5%8B.md) — AWGF-ZVD 检测器增强
- [2026-06-24 - Skog 零速检测算法评估](../../%E6%9D%A5%E6%BA%90/2026-06-24%20-%20Skog%20%E9%9B%B6%E9%80%9F%E6%A3%80%E6%B5%8B%E7%AE%97%E6%B3%95%E8%AF%84%E4%BC%B0.md) — 传统检测器
- [误差状态卡尔曼滤波](../%E5%A7%BF%E6%80%81%E8%A7%A3%E7%AE%97/%E8%AF%AF%E5%B7%AE%E7%8A%B6%E6%80%81%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2.md) — ESKF 基础
