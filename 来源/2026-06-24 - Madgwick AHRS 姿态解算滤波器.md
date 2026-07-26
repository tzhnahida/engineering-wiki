---
type: source
tags: [imu, ahrs, sensor-fusion, paper]
created: 2026-06-24
updated: 2026-06-24
---

# 2026-06-24 - Madgwick AHRS 姿态解算滤波器

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | An efficient orientation filter for inertial and inertial/magnetic sensor arrays |
| 作者 | Sebastian O.H. Madgwick |
| 日期 | 2010 |
| 页数 | 32 |
| 来源 | `参考/论文/Madgwick 2010 - An efficient orientation filter for inertial and inertial magnetic sensor arrays.pdf` |
| 类型 | 学术论文 |

## 文档概述

Madgwick 提出的梯度下降四元数姿态融合算法。将姿态估计形式化为优化问题而非滤波问题，用解析 Jacobian 替代数值微分，每步仅 109 (IMU) / 277 (MARG) 次标量运算。

## 关键知识点

- 核心融合方程：q̇_est = q̇_ω − β·∇f/‖∇f‖
- 梯度下降的解析 Jacobian 推导（省去数值微分）
- β = √3/2·ω̃_err，从陀螺噪声密度直接计算，非经验参数
- 磁力计在线补偿：限制磁干扰仅影响 yaw
- 陀螺零偏漂移跟踪（MARG 模式）
- 运算量：IMU 109 ops、MARG 277 ops

## 创建的 Wiki 页面

- [[姿态解算/梯度下降姿态解算]] — 完整推导、β 物理含义、运算量解剖
