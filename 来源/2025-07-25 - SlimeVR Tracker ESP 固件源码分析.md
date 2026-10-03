---
type: source
tags: [slimevr, firmware, esp32, vqf, source-analysis]
created: 2025-07-25
updated: 2025-07-25
---

# SlimeVR Tracker ESP 固件源码分析

> SlimeVR/SlimeVR-Tracker-ESP — ESP32 开源全身动捕追踪器固件。MIT 协议。

- **仓库**: https://github.com/SlimeVR/SlimeVR-Tracker-ESP
- **平台**: ESP32 / ESP8266 / ESP32-C3
- **核心算法**: VQF（Versatile Quaternion Filter）— 解耦式三模块姿态融合
- **协议**: UDP 大端序二进制包 → SlimeVR Server → SteamVR Driver
- **支持传感器**: ICM-42688-P, ICM-20948, ICM-45686, BMI160, BMI270, BNO055, BNO080, MPU6050, MPU9250, LSM6DS3/DSO/DSR/DSV

固件采用裸机主循环架构（无 RTOS），µs 级定时控制，7 个子系统顺序调度。传感器层采用虚拟基类 + 模板多态实现 12 种 IMU 统一接口。网络层实现 UDP 广播发现 + 握手协议 + Bundle 批量传输。校准系统支持静止检测、陀螺/加速度偏置在线估计、温度补偿。

## 分析范围

| 文件/目录 | 行数 | 职责 |
|-----------|------|------|
| `src/main.cpp` | 228 | 入口 + 主循环 |
| `src/network/` | ~1200 | UDP 通信 + 握手 + 包格式 |
| `src/sensors/` | ~3500 | 传感器抽象 + 驱动 + VQF 融合 |
| `src/sensors/softfusion/` | ~2000 | SoftFusion 传感器 + ICM-42688 驱动 + 运行时校准 |
