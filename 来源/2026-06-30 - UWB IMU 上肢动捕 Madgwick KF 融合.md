---
type: source
tags: [uwb, imu, mocap, madgwick, kalman-filter, upper-limb]
created: 2026-06-30
updated: 2026-06-30
---

# 2026-06-30 - UWB IMU 上肢动捕 Madgwick KF 融合

**Shi, Zhang, Li, Yuan & Zhu (2023).** *IMU/UWB Fusion Method Using a Complementary Filter and a Kalman Filter for Hybrid Upper Limb Motion Estimation.* Sensors, 23(15), 6700. DOI: `10.3390/s23156700`，24 引。北航。

## 做了什么

上肢 IMU + 4 个 UWB 锚点。**RK4 Madgwick 互补滤波器做取向 + Kalman Filter 融 UWB 位置**。对标准：光学动捕。

## 核心方法

1. **RK4 Madgwick** — 四阶 Runge-Kutta 积分提升姿态精度
2. **KF for UWB** — 四个锚点（立方分布）的三边定位值做 KF 观测
3. 两层独立 —— Madgwick 输出姿态，KF 输出位置，不互相耦合

## 关键数据

| 指标 | 结果 |
|------|:---:|
| RMSE 改善 | 融合后降低 **1.2 cm**（vs 纯 IMU） |
| 稳定性 | 多种上肢动作均可行 |

## 意义

**与 [[全身动捕与头显系统]] 方案 A 架构几乎一样**——Madgwick + KF(UWB)，分层不耦合。2023 年最新工作，直接为方案可行性背书。立方锚点分布可供参考。
