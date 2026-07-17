---
type: source
tags: [imu, ahrs, quaternion, paper]
created: 2026-07-15
updated: 2026-07-15
---

# Laidig 2022 - VQF 姿态解算滤波器

> Laidig & Seel, "VQF: Highly Accurate IMU Orientation Estimation with Bias Estimation and Magnetic Disturbance Rejection", Information Fusion 91:187-204, 2023. arXiv:2203.17024 (2022)。
> PDF：`参考/论文/Laidig 2022 - VQF Highly Accurate IMU Orientation Estimation.pdf`（22 页）

## 核心贡献

1. **准惯性系低通滤波**：把加计测量旋转到几乎惯性的坐标系后再做低通滤波——瞬时运动加速度被有效滤除而重力方向保留，这是倾斜精度翻倍的关键
2. **解耦航向校正**：航向漂移用一个标量偏移校正，磁干扰不再污染倾斜（roll/pitch）
3. **6D/9D 同时输出**：解耦的状态表示天然支持有磁/无磁两种模式并行
4. **附加能力**：静止和运动中的陀螺零偏估计、磁干扰检测与拒绝、离线非因果变体

## 关键结果

- 公开数据集平均 RMSE **2.9°**，八种对比方法为 5.3°~16.7°（1.8~5 倍差距）
- **单一固定参数**跨运动类型/硬件/环境全部适用——无需按应用调参
- 开源 C++/Python/Matlab 实现，支持陀螺/加计/磁力计不同采样率

## 与其他方法的关系

- 对比 [Madgwick](../知识/姿态解算/梯度下降姿态解算.md)：同为免调参哲学，但 VQF 的准惯性系滤波从机理上区分了重力和运动加速度，而非靠固定权重折中
- 与 [MDR](../知识/姿态解算/MDR 磁畸变抑制.md) 互补：VQF 拒绝磁干扰（不用），MDR 建模磁干扰（利用）

## 服务的知识页

- [知识/姿态解算/VQF 姿态解算滤波器](../知识/姿态解算/VQF 姿态解算滤波器.md)
