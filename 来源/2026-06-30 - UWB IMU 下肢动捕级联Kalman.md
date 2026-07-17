---
type: source
tags: [uwb, imu, mocap, kalman-filter, sensor-fusion]
created: 2026-06-30
updated: 2026-06-30
---

# 2026-06-30 - UWB IMU 下肢动捕级联 Kalman

**Zihajehzadeh, Yoon, Kang & Park (2015).** *UWB-Aided Inertial Motion Capture for Lower Body 3-D Dynamic Activity and Trajectory Tracking.* IEEE Transactions on Instrumentation and Measurement, 64(12), 3577–3587. DOI: `10.1109/TIM.2015.2459532`，70 引。

## 做了什么

7 颗 MEMS IMU（骨盆+大腿+小腿+脚）+ 1 个 UWB 标签挂腰上 → **级联 Kalman 滤波器**做姿态和位置两层融合。对标准：相机参考系统。

## 核心方法

**级联 KF 两步走**：

1. **姿态 KF** — 先 tilt（陀螺+加速度计），再 yaw（陀螺+磁力计，含磁干扰检测与 ~20s 桥接）
2. **位置/速度 KF** — 取姿态消除重力分量，融合 UWB 绝对位置做 3D 轨迹

不跑全局 KF（子优但计算量可控），证明级联结构在实际应用中精度接近全局。

## 关键数据

| 指标 | 结果 |
|------|:---:|
| 腰部 3D 轨迹 | **< 5 cm** |
| 膝关节角度 | **< 2.1°** |
| 快/慢一致性 | 走路≈跳跃 |

> ⚠️ 使用商业级 UWB 系统（非 DW1000），DW1000/MK8000 需要下调预期。

## 意义

首次在 UWB+IMU 融合中做到动态活动（跳、跑）不退化。级联 KF 架构直接可复用——现代方案就是 Madgwick 替代姿态 KF，位置 KF 不变。
