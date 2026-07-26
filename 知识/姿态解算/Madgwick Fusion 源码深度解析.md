---
type: concept
tags: [姿态解算, Madgwick, AHRS, 源码分析, 梯度下降, 互补滤波]
created: 2025-07-25
updated: 2025-07-25
sources: ["[2026-06-24 - Madgwick AHRS 姿态解算滤波器](../../%E6%9D%A5%E6%BA%90/2026-06-24%20-%20Madgwick%20AHRS%20%E5%A7%BF%E6%80%81%E8%A7%A3%E7%AE%97%E6%BB%A4%E6%B3%A2%E5%99%A8.md)"]
---

# Madgwick Fusion 源码深度解析

> x-io Technologies 的 FusionAhrs C 实现（507行）。生产级代码，无显式 Jacobian，用叉积近似梯度下降。

## 核心算法：叉积替代梯度

原始 Madgwick 论文需要计算 6×4 Jacobian + 目标函数梯度。Fusion 库用一个关键简化：**叉积反馈 = ∇f 的等效方向**。

```c
// 加速度计反馈：叉积 ≈ Madgwick 目标函数的梯度方向
ahrs->halfAccelerometerFeedback = Feedback(
    FusionVectorNormalise(accelerometer),  // 测量的重力方向
    ahrs->halfGravity                      // 预测的重力方向
);
```

`Feedback(a, b) = a×b`（若夹角 >90° 则归一化，避免 180° 时叉积趋于零）。

## 四元数更新公式

```c
// 陀螺仪+反馈融合
halfGyro = 0.5 * gyro_rad;                    // 0.5 因子贯穿始终
adjusted = halfGyro + beta * (accFeed + magFeed);  // beta = 梯度下降步长

// 一阶 Euler 积分
q_new = q + q ⊗ adjusted ⊗ dt;               // 不是 q ⊗ exp()
// 然后归一化
```

**对比 VQF**：Euler 积分 vs ZOH 解析积分。Euler 在 200Hz 下可接受（角增量 <1° 时误差 <0.01%）。

## β 参数

默认 `beta = 0.5 rad/s`（设计为快速收敛，非最优）。论文推荐值：

```
beta = √(3/4) · ω̃_err ≈ 0.866 · 0.5°/s · π/180 ≈ 0.0076 rad/s
```

启动期 ramp：从 10.0 rad/s 线性降到 0.5（3 秒），确保快速初始收敛。

## 磁力计倾斜补偿

关键设计：磁力计先与重力叉积，**仅用水平分量**校正偏航。倾角（dip angle）完全忽略——这是抗磁干扰的核心。

```c
// 投影到水平面
ahrs->halfMagnetometerFeedback = Feedback(
    FusionVectorNormalise(FusionVectorCross(ahrs->halfGravity, magnetometer)),
    halfMagnetic  // 从旋转矩阵提取的预期水平磁场方向
);
```

## 迟滞拒绝机制

```
加速度计/磁力计反馈误差 > 阈值:
    trigger += 1       ← 慢累积
    trigger > timeout  → 强制恢复 → 立即重新纳入反馈
    
反馈误差 <= 阈值:
    trigger -= 9       ← 快恢复（9x 速度）
```

拒绝阈值的内部表示：`sin²(angle/2) / 4`（存储为平方值，避免 sqrt）。

## 性能

| 模式 | 浮点运算 | sqrt 调用 |
|------|---------|----------|
| IMU（无磁力计） | ~50 | 2 |
| MARG（含磁力计） | ~90 | 3 |

全 float32，无 double。适合 ESP32，每帧 <20µs。

## 三种坐标系

| 约定 | 上轴 | 前向 | HalfGravity 符号 |
|------|------|------|-----------------|
| NWU (默认) | +Z | North | +0.5 |
| ENU | +Z | East | +0.5 |
| NED (航空) | -Z | North | -0.5 |

切换仅影响旋转矩阵的提取列，核心算法不变。
