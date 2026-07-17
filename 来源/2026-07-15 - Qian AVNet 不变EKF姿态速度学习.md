---
type: source
tags: [imu, deep-learning, kalman-filter, invariant-ekf, sensor-fusion, attitude-estimation, paper]
created: 2026-07-15
updated: 2026-07-15
---

# 2026-07-15 - Qian AVNet 不变EKF姿态速度学习

**标题**: AVNet: Learning Attitude and Velocity for Vehicular Dead Reckoning Using Smartphone by Adapting an Invariant EKF
**作者**: Long Qian, Xinchuang Lin, Xiaoguang Niu, Qihai Huang, Leilei Li, Guangyi Guo, Zexin Wang, Ruizhi Chen
**期刊**: Satellite Navigation, 2025, 6:15. DOI: 10.1186/s43020-025-00168-7. 开放获取 (CC BY 4.0).
**机构**: 武汉大学 LIESMARS + 重庆大学
**PDF**: [参考/论文/Qian 2025 - AVNet Attitude Velocity IEKF.pdf](../参考/论文/Qian 2025 - AVNet Attitude Velocity IEKF.pdf.md)

## 核心贡献

1. **DMDVDR 框架**：提出数据+模型双驱动车载航位推算（Data- and Model- Driven Vehicle Dead Reckoning），将深度学习伪观测与 InEKF 模型驱动滤波融合
2. **AVNet 网络**：CNN-GRU 混合架构从单 IMU 同时学习姿态和速度伪观测，替代车载物理传感器（里程计、转角传感器）
3. **协方差适配器**：CNN 适配器根据 IMU 信号质量实时动态调节 InEKF 的过程噪声 Q 和测量噪声 N——以相对平移误差为损失函数间接优化
4. **首次将 3D 学习姿态引入 InEKF**：此前工作仅使用速度约束，AVNet 首次加入 DDATT 与 DDODO + DDNHC 形成 6-DOF 几何约束

## 关键内容

### 方法
- AVNet 输入 200 帧 IMU 原始数据（200 x 6），输出频率 1 Hz；姿态输出为四元数变化量（3-DOF），速度输出为前进方向速度（1-DOF）
- 协方差适配器输入窗口 20 帧，输出频率 200 Hz；动态范围 beta=3（各协方差可在默认值 10^3 倍到 10^{-3} 倍间调节）
- InEKF 状态嵌入 SE2(3) 李群，采用右不变误差定义，利用群仿射性质实现轨迹无关的误差动力学
- 速度测量模型包含杆臂补偿（传感器到车辆中心的偏移）和外参估计（传感器到车体的安装姿态）
- 非完整性约束（DDNHC）：横向和垂向速度为零

### 实验
- 传感器：华为 Mate 30（STM LSM6DSM），刚性固定于测试车
- 真值：NovAtel SPAN + ISA-100C IMU（厘米级精度）
- 场景：地面停车场（11 条序列，矩形 + S 形轨迹，时速 5/10/15 km/h）+ GSDC 隧道数据
- 对比方法：AI-IMU（仅 NHC 约束）、DeepOdo（+DDODO）、DeepOri（+DDATT）
- 结果：Proposed 相对水平平移误差 0.29%-0.58%，对比方法为 4-66%；隧道 55s GNSS 中断漂移 0.64%

### 局限性
- 智能手机刚性固定安装假设（未测试手持/口袋场景）
- 单一手机型号（华为 Mate 30）
- 数据集规模有限（11 序列）
- 垂直方向误差大于水平，部分源于固定重力假设

## 与本 wiki 的关系

- 知识页：[AVNet 不变扩展卡尔曼姿态](../知识/姿态解算/AVNet 不变扩展卡尔曼姿态.md)
- 关联概念：[迭代不变扩展卡尔曼滤波](../知识/姿态解算/迭代不变扩展卡尔曼滤波.md)、[误差状态卡尔曼滤波](../知识/姿态解算/误差状态卡尔曼滤波.md)、[DO IONet Transformer直接姿态](../知识/姿态解算/DO IONet Transformer直接姿态.md)、[IMU姿态解算算法演进](../知识/姿态解算/IMU姿态解算算法演进.md)
- 方法论集合：深度学习 + 不变 Kalman 滤波混合方法
