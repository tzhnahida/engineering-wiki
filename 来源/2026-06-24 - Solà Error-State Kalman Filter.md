---
type: source
tags: [imu, kalman-filter, quaternion, eskf, state-estimation]
created: 2026-06-24
updated: 2026-06-30
---

# 2026-06-24 - Solà Error-State Kalman Filter

> **论文**: Solà (2017) — *Quaternion Kinematics for the Error-State Kalman Filter*, arXiv:1711.02508
> **存放**: [参考/论文/Solà 2017 - Quaternion Kinematics for the Error-State Kalman Filter.pdf](参考/论文/Solà 2017 - Quaternion Kinematics for the Error-State Kalman Filter.pdf.md)

## 概述

ESKF（误差状态卡尔曼滤波）的完整数学教程。系统推导四元数运动学在 ESKF 框架下的应用：将系统状态分解为 nominal state（大信号、非线性积分）和 error state（小信号、线性化），error state 通过标准 Kalman 滤波估计后注入 nominal state 校正。是 IMU 导航和姿态估计领域的核心参考文献。

## 关键贡献

- Nominal/Error state 分离框架
- 四元数误差的线性化推导（Liouville 方程）
- ESKF 的完整递推方程（预测→观测→校正→注入）
- Phase 2 15 节点全局融合的理论基础

## 关联知识

- [误差状态卡尔曼滤波](误差状态卡尔曼滤波.md) — 知识页
- [姿态解算/统一因子图姿态推断框架](姿态解算/统一因子图姿态推断框架.md) — 因子图扩展
