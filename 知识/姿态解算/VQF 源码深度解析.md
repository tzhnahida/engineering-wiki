---
type: concept
tags: [姿态解算, VQF, 源码分析, 卡尔曼滤波, 陀螺偏置, 磁干扰]
created: 2025-07-25
updated: 2025-07-25
sources: ["[2025-07-25 - SlimeVR Tracker ESP 固件源码分析](../../%E6%9D%A5%E6%BA%90/2025-07-25%20-%20SlimeVR%20Tracker%20ESP%20%E5%9B%BA%E4%BB%B6%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)", "[2026-07-15 - Laidig VQF 姿态解算滤波器](../../%E6%9D%A5%E6%BA%90/2026-07-15%20-%20Laidig%20VQF%20%E5%A7%BF%E6%80%81%E8%A7%A3%E7%AE%97%E6%BB%A4%E6%B3%A2%E5%99%A8.md)"]
---

# VQF 源码深度解析

> 基于 SlimeVR 内置的 VQF C++ 库（`lib/vqf/`，~2000 行）。单精度 float，每实例 ~750 字节 RAM。

## 三模块级联架构

VQF 不输出"一个四元数"，而是维护三个独立状态再用四元数乘法组合：

| 状态 | 符号 | 更新方式 | 含义 |
|------|------|----------|------|
| `gyrQuat` | q_SI | 捷联积分 | 纯陀螺仪姿态（漂移） |
| `accQuat` | q_IE | 倾角校正（仅横滚/俯仰） | 用加速度计消除倾角误差 |
| `delta` | δ | 标量航向差 | 用磁力计消除偏航误差 |

输出组合：`q_6D = q_IE ⊗ q_SI`、`q_9D = [cos(δ/2),0,0,sin(δ/2)] ⊗ q_6D`

## 核心算法逐函数

### updateGyr(gyr, dt)

```
1. 静止检测（陀螺仪部分）：二阶巴特沃斯 LPF → ||gyr - gyr_lp||² < restThGyr²
   + 额外检查 |gyr_lp[i]| < biasClip（防匀速大角速率误判）
2. 偏置移除：gyrNoBias = gyr - bias
3. 捷联积分：q_new = q ⊗ [cos(θ/2), sin(θ/2)·ω/|ω|]
   其中 θ = |gyrNoBias|·dt
```

### updateAcc(acc) — 最重的方法（~500 FLOPs）

```
1. 静止检测（加速度计部分）：LPF → ||acc - acc_lp||² < restThAcc²
   两条件都满足且持续 restMinT=2.59s → restDetected = true

2. 倾角校正：
   将 LPF 后的加速度转到惯性系，构造绕水平轴的旋转 q_c
   使得 q_c 将测得的重力方向旋转到 [0,0,1]
   关键恒等式：q_w = √((a_z+1)/2), q_x = 0.5·a_y/q_w, q_y = -0.5·a_x/q_w, q_z = 0
   accQuat = q_c ⊗ accQuat（纯水平校正，不改变偏航）

3. 偏置估计（3×3 卡尔曼滤波器）：
   静止模式：直接测量 bias = gyr_lp
   运动模式：从倾角校正隐含的角速率 + 垂直偏置零假设推导
   协方差膨胀 V = (0.1°/s)²·Ts / T_forget → T_forget=137s 内方差从 0 → (0.1°/s)²
```

### updateMag(mag) — 磁干扰拒绝

```
1. 将磁力计转到 E_i 系
2. 计算磁场范数 ||m|| 和倾角 dip = -asin(m_z/||m||)
3. 与参考值比较：|norm-refNorm| < 10% AND |dip-refDip| < 10° → 未受干扰
   受干扰时：k=0（完全拒绝航向更新），最长 60s
   超时后：k = k_normal/2（部分恢复）
4. 航向更新：δ += k · atan2(m_x, m_y)
```

## SlimeVR 调优参数 vs 论文默认

| 参数 | 论文默认 | SlimeVR 值 | 效果 |
|------|---------|-----------|------|
| tauAcc | 3.0s | **4.34s** | 更激进的低通滤波=更稳但延迟更大 |
| biasClip | 2.0°/s | **5.0°/s** | 允许更大偏置范围 |
| restThGyr | 2.0°/s | **1.40°/s** | 更敏感的静止检测 |
| restMinT | 1.5s | **2.59s** | 需要更长的静止确认 |

## ESP32 性能

| 操作 | 浮点运算 | ESP32 @240MHz 耗时 |
|------|---------|-------------------|
| updateGyr | ~100 | <5µs |
| updateAcc（含偏置估计） | ~500 | ~15µs |
| updateMag（含干扰拒绝） | ~200 | ~8µs |
| **每帧总计** | ~800 | **<30µs** |

200Hz ODR 下每帧 5ms 预算，VQF 占用 <1%。

## 与 Madgwick 的源码级对比

| | Madgwick (Fusion) | VQF |
|---|---|---|
| 偏置估计 | 无内置 | 3×3 Kalman，静止+运动双模式 |
| 磁干扰处理 | 叉积阈值+迟滞计数器 | 范数/倾角双参考+自适应接受 |
| 积分方式 | Euler 一阶 `q += 0.5·q·ω·dt` | ZOH 解析积分 `q ⊗ exp(ω·dt/2)` |
| 代码行数 | ~500 C | ~2000 C++ |
| RAM | ~200 B | ~750 B |
