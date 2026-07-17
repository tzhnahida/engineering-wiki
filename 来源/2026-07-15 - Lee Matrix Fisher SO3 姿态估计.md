---
type: source
tags: [imu, bayesian, so3, lie-group, paper]
created: 2026-07-15
updated: 2026-07-15
---

# 2026-07-15 — Lee Matrix Fisher SO(3) 贝叶斯姿态估计

> Lee T (2018). Bayesian Attitude Estimation with the Matrix Fisher Distribution on SO(3). arXiv:1710.03746. 20 页。

**核心贡献**：在 SO(3) 旋转群上定义 Matrix Fisher 分布作为姿态的先验概率模型，构建全局贝叶斯滤波器——避免了四元数的双覆盖歧义和欧拉角的奇异性。

**关键内容**：
- Matrix Fisher 分布在 SO(3) 上的定义：$p(R|F) = \frac{1}{c(F)} \exp(\text{tr}(F^T R))$，其中 F 为 3×3 参数矩阵
- F 的奇异值决定分布集中度：奇异值越大→不确定性越小
- 两种贝叶斯框架：解析矩传播 + Unscented 变换（Sigma 点近似）
- 不需要局部坐标（欧拉角）也不需归一化约束（四元数），概率分布直接定义在流形上

**对本项目的价值**：提供姿态不确定性的数学框架，后续 Jin (2025) Information Fusion 论文将其与深度学习结合做端到端概率姿态估计。

**PDF**：`参考/论文/Lee 2018 - Bayesian Attitude Estimation Matrix Fisher SO3.pdf`
