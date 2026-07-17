---
type: concept
tags: [imu, ahrs, vqf, quaternion, sensor-fusion, gyroscope, bias-estimation]
created: 2026-06-26
updated: 2026-07-16
sources:
  - "[2026-07-15 - Laidig VQF 姿态解算滤波器](2026-07-15 - Laidig VQF 姿态解算滤波器.md)"
---

# VQF 姿态解算滤波器

> Versatile Quaternion-based Filter (Laidig & Seel, Information Fusion 2023)。核心创新：把姿态估计分解为**三级串联模块**——陀螺 Strapdown 积分、准惯性系倾斜校正、标量航向校正——三者之间无反馈回路，磁干扰不再污染倾斜估计。单一固定参数在 6 个公开数据集（161 trials，12.9 h）上平均 RMSE 2.9°，对比方法 5.3°–16.7°。

## 核心设计：三级串联解耦

[📷 _llm/raw/assets/papers/vqf2022/vqf_p2_fig1.jpg|560]
*Figure 1 — IMU 姿态估计的传感器融合架构总览：陀螺积分提供高频姿态，加计/磁力计提供绝对参考，VQF 的串联模块设计使三者解耦*

传统 AHRS（Madgwick、Mahony、EKF）把陀螺积分、倾斜校正、航向对齐三个任务揉在同一个反馈回路里。磁力计一旦被干扰，污染经反馈回路传染到倾斜估计——而倾斜本来只依赖加计。

VQF 的状态本身就被分解为三个独立变量的乘积：

$$
\underbrace{^{\mathcal{S}_i}_{\mathcal{E}}\mathbf{q}}_{\text{9D 估计}} = 
\underbrace{(\delta_i @ [0,0,1]^\top)}_{\text{航向校正（仅 Yaw）}} \otimes
\underbrace{^{\mathcal{N}_i}_{\mathcal{E}_i}\mathbf{q}}_{\text{倾斜校正}} \otimes
\underbrace{^{\mathcal{S}_i}_{\mathcal{N}_i}\mathbf{q}}_{\text{Strapdown 积分}}
$$

- $\mathcal{S}_i$：传感器坐标系
- $\mathcal{N}_i$：准惯性坐标系（almost-inertial frame），由纯陀螺积分定义，缓慢漂移
- $\mathcal{E}_i$：6D 全局坐标系（天向对齐，航向自由漂移）
- $\mathcal{E}$：9D ENU 全局坐标系（东-北-天）
- $\delta_i$：标量航向偏移角，$\mathcal{E}_i$ 与 $\mathcal{E}$ 之间的唯一自由度

[📷 _llm/raw/assets/papers/vqf2022/vqf_p4_fig1.jpg|620]
*Figure 3 — 传统滤波器结构（上）vs VQF 结构（下）：VQF 的串联模块消除了反馈回路，磁场干扰只影响航向校正模块*

> [!note] 关键洞察
> 把姿态分解为三级串联意味着 6D（无磁力计）和 9D（含磁力计）估计可以**同时输出**。6D 估计完全不依赖磁力计，相当于天然免疫磁干扰。这是后续偏置估计也只依赖加计信息的基础。

## 模块一：Strapdown 积分（陀螺预测）

陀螺预测是四元数的精确乘法更新——不是一阶近似：

$$
^{\mathcal{S}_i}_{\mathcal{N}_i}\mathbf{q}(t_k) = ^{\mathcal{S}_i}_{\mathcal{N}_i}\mathbf{q}(t_{k-1}) \otimes (T_s \|\boldsymbol{\omega}\| @ \boldsymbol{\omega})
$$

其中旋转轴为瞬时角速率归一化方向，旋转角为 $T_s \|\boldsymbol{\omega}\|$。这是**精确**的指数映射（单位四元数），对应刚体有限转动的闭式解——与一阶近似 $\mathbf{q} \approx \mathbf{q} + \frac{1}{2} \boldsymbol{\omega} \mathbf{q} T_s$ 的区别在于：

- **一阶近似**假设小角度，在高速转动时积分误差累积
- **VQF 的精确解**在任何角速率下都保持单位四元数范数，精度仅受采样率影响

积分误差（偏置、噪声、标度误差等）导致 $\mathcal{N}_i$ 帧缓慢漂移，可由加计/磁力计校正补偿。

## 模块二：准惯性系倾斜校正（加计校正）

这是 VQF 区别于所有传统方法最核心的创新。

### 为什么在 $\mathcal{N}_i$ 帧中低通滤波有效

传统方法（Madgwick、Mahony）在**传感器坐标系 $\mathcal{S}_i$** 中处理加计——此时每次测量都混有重力 + 瞬时运动加速度，每一步都要用非线性方法分开二者。

VQF 的做法：

1. 用当前姿态四元数将加速度变换到 $\mathcal{N}_i$ 帧：$[\mathbf{a}]_{\mathcal{N}_i} = ^{\mathcal{S}_i}_{\mathcal{N}_i}\mathbf{q} \otimes \mathbf{a} \otimes ^{\mathcal{S}_i}_{\mathcal{N}_i}\mathbf{q}^{-1}$
2. 对三个分量分别做**二阶 Butterworth 低通滤波**
3. 归一化滤波后的加速度向量，计算最短旋转校正四元数

关键物理直觉：$\mathcal{N}_i$ 帧中，运动加速度（加减速）在长时间尺度上均值为零，低通滤波后剩下的是重力方向。这正是"不是猜重力，而是等运动加速度自己平均掉"。

[📷 _llm/raw/assets/papers/vqf2022/vqf_p4_fig5.jpg|420]
*Figure 5 — 原始传感器系（上）vs 准惯性系（下）中的加速度：在准惯性系中低通滤波能有效滤除瞬时运动加速度，保留重力方向*

校正四元数有闭式解（无需三角函数）：归一化后的 $[a_x, a_y, a_z]^\top$ 到 $[0,0,1]^\top$ 的最短旋转为 $q_w = \sqrt{(a_z+1)/2}$，$\mathbf{q}_{\text{corr}} = [q_w,\; a_y/2q_w,\; -a_x/2q_w,\; 0]^\top$。

截止频率与时间常数 $\tau_{\text{acc}}$ 的关系：$f_{\text{c,acc}} = \sqrt{2}/(2\pi\tau_{\text{acc}})$。该映射使二阶 Butterworth 的阶跃响应在 $t=\tau$ 处约 49.2%，与一阶指数滤波器的 63.2% 大致对齐。

- **默认 $\tau_{\text{acc}} = 3\,\text{s}$**：6 数据集平均最优
- 典型步行周期 ~1-1.5 s、跑步 ~0.6-0.8 s，3 s 的"记忆"足以让多个步态周期的加速度相互抵消
- 调大（如 10 s）更信任陀螺，适合慢速平滑运动；调小（如 0.5 s）快速跟随加计

## 模块三：标量航向校正（磁力计校正）

航向被建模为**单一标量** $\delta_i$。将 $\mathbf{m}$ 变换到 $\mathcal{E}_i$ 帧后取水平投影：$\delta_{\text{mag}} = \text{atan2}(m_x, m_y)$。一阶指数滤波器更新：

$$
\delta_i \leftarrow \delta_i + k_{\text{mag}} \cdot \text{wrapToPi}(\delta_{\text{mag}} - \delta_i), \quad
k_{\text{mag}} = 1 - \exp\left(-\frac{T_s}{\tau_{\text{mag}}}\right)
$$

默认 $\tau_{\text{mag}} = 9\,\text{s}$：足以抵抗短暂磁干扰，又足够跟踪实际航向。初始化的 $1, \frac12, \frac13, \dots$ 自适应权重实现快速收敛。9D 最终输出：$^{\mathcal{S}_i}_{\mathcal{E}}\mathbf{q} = [\cos\frac{\delta_i}{2}, 0, 0, \sin\frac{\delta_i}{2}]^\top \otimes ^{\mathcal{S}_i}_{\mathcal{E}_i}\mathbf{q}$。

> [!note] 标量化的优势
> 传统方法用完整四元数做航向校正，磁干扰映射到三个轴。VQF 把航向自由度压缩成单个标量 $\delta_i$，任何磁干扰只能影响这一个数——Roll/Pitch 完全不受牵连。

## VQF 参数总表

| 参数 | 符号 | 默认值 | 物理含义 |
|------|------|:------:|----------|
| 倾斜校正时间常数 | $\tau_{\text{acc}}$ | 3 s | 二阶 Butterworth LPF 的时间常数，越大越信任陀螺 |
| 航向校正时间常数 | $\tau_{\text{mag}}$ | 9 s | 一阶指数滤波器的时间常数，越大越信任陀螺 |
| 静止检测陀螺阈值 | — | 2°/s | 各轴 LPF 残差上限 |
| 静止检测加计阈值 | — | 0.5 m/s² | 各轴 LPF 残差上限 |
| 静止检测窗口 | — | 1.5 s | 低于阈值持续此时间才确认静止 |
| 静止检测 LPF | — | $\tau = 0.5\,\text{s}$ | 二阶 Butterworth |
| 偏置 Kalman 初值 | $\sigma_{\text{init}}$ | 0.5°/s | 初始估计不确定度 |
| 偏置运动收敛 | $\sigma_{\text{motion}}$ | 0.1°/s | 运动中非垂直轴收敛不确定度 |
| 偏置静止收敛 | $\sigma_{\text{rest}}$ | 0.03°/s | 静止时收敛不确定度 |
| 偏置遗忘时间 | $t_{\text{forget}}$ | 100 s | 无更新时偏置不确定度从 0 增长到 0.1°/s |
| 偏置限幅 | — | $\pm$2°/s | 估计值硬限幅 |
| 磁检测 LPF | — | $\tau = 0.05\,\text{s}$ | 对幅值/倾角的预处理滤波 |
| 磁正常幅值偏差 | — | 10% of $n_{\text{ref}}$ | $|n - n_{\text{ref}}| < 0.1 n_{\text{ref}}$ 视为正常 |
| 磁正常倾角偏差 | — | 10° | $|\theta - \theta_{\text{ref}}| < 10^\circ$ 视为正常 |
| 磁正常确认时间 | — | 0.5 s | 持续正常后才标记为未干扰 |
| 磁完全拒绝时长 | — | 60 s | 前 60 秒完全禁用磁校正 |
| 磁半速恢复 | — | — | 60 s 后以 $k_{\text{mag}}/2$ 更新 |
| 磁新场接受时间 | — | 20 s | 运动中磁扬均匀持续此时间后接受为新参考 |
| 新场接受运动条件 | — | $\geq$20°/s | 陀螺范数阈值，确保非静止 |

> [!tip] $\tau_{\text{mag}} > \tau_{\text{acc}}$ 的物理意义
> 航向校正时间常数（9 s）是倾斜校正（3 s）的 3 倍。这反映了磁力计的本质：地球磁场弱（25-65 µT）、易受局部干扰；而重力方向始终稳定可靠。VQF 对磁力计的信任天然低于加计。

## 扩展模块一：静止检测与偏置估计

### 静止检测判据

```
对陀螺和加计各轴信号：
  1. 二阶 Butterworth LPF（τ = 0.5 s），直接在传感器坐标系中滤波
  2. 计算当前测量值与滤波值的绝对偏差
  3. 若所有轴的偏差同时满足：
     陀螺偏差 < 2°/s  且  加计偏差 < 0.5 m/s²
     持续 ≥ 1.5 s → 判定为静止
  4. 阈值使用绝对物理单位（而非统计量），与采样率无关
```

> [!note] VQF 的静止检测只使用陀螺和加计，不使用磁力计。论文实验表明陀+加的组合已足够可靠。

### 静止中偏置估计

静止时低通滤波后的陀螺读数直接就是偏置观测量：$\mathbf{y}(t_k) = \boldsymbol{\omega}_{\text{LP}}(t_k)$，$\mathbf{C} = \mathbf{I}_{3\times 3}$。对应 $\sigma_{\text{rest}} = 0.03\degree/\text{s}$，数秒收敛。

### 运动中偏置估计的数学原理

运动中以倾斜校正的漂移量作为偏置观测（核心物理直觉：**陀螺偏置导致 $\mathcal{N}_i$ 帧缓慢漂移，这个漂移会被倾斜校正模块观测到**）。

[📷 _llm/raw/assets/papers/vqf2022/vqf_p6_fig1.jpg|560]
*Figure 6 — 偏置 Kalman 滤波器的阶跃响应：蓝色带为估计不确定度，静止更新收敛远快于运动更新*

理想稳态下，倾斜校正的旋转向量与残差偏置的关系为（详见论文 Appendix D）：

$$
\mathbf{R}(\mathbf{b} - \hat{\mathbf{b}}) = -\frac{1}{T_s} \begin{bmatrix} c_x & c_y & * \end{bmatrix}^\top
$$

其中 $\mathbf{R}$ 为 $^{\mathcal{S}_i}_{\mathcal{E}_i}\mathbf{q}$ 对应的旋转矩阵，$[c_x, c_y, 0]^\top$ 为倾斜校正旋转向量。$*$ 表示垂直方向的偏置不可观测——这也是为什么航向偏置仅靠加计无法估计。

实际实现中，对 $\mathbf{R}$ 和 $\mathbf{R}\hat{\mathbf{b}}$ 施加与加速度相同的低通滤波（$\tau = \tau_{\text{acc}}$），以匹配加计校正的时域特性。观测方程为：

$$
\mathbf{y}_k = \begin{bmatrix} -\frac{1}{T_s}a_y + r_{11}\hat{b}_x + r_{12}\hat{b}_y + r_{13}\hat{b}_z \\ \frac{1}{T_s}a_x + r_{21}\hat{b}_x + r_{22}\hat{b}_y + r_{23}\hat{b}_z \\ 0 \end{bmatrix}
$$

$\mathbf{C} = \text{LPF}(\mathbf{R})$，观测方差 $w_{\text{motion}}$ 对应 $\sigma_{\text{motion}} = 0.1\degree/\text{s}$。

### Kalman 滤波器的直觉参数化

用户不直接设协方差，而指定四个直观量：$\sigma_{\text{init}}=0.5\degree/\text{s}$（初始）、$\sigma_{\text{motion}}=0.1\degree/\text{s}$（运动收敛）、$\sigma_{\text{rest}}=0.03\degree/\text{s}$（静止收敛）、$t_{\text{forget}}=100\,\text{s}$（遗忘时间）。内部协方差自动映射：$\mathbf{V} = (0.1\degree/\text{s})^2 T_s / t_{\text{forget}} \cdot \mathbf{I}$，$w = \sigma^4/v + \sigma^2$。参数值本身即可解释为"估计精度约 n°/s"，且自动适配采样率。

## 扩展模块二：磁干扰检测与拒绝

[📷 _llm/raw/assets/papers/vqf2022/vqf_p13_fig1.jpg|560]
*Figure 14 — 磁干扰拒绝效果 (BROAD trial 37)：无拒绝时最大误差 20.0°、RMSE 6.1°；启用后降至 3.7° 和 1.8°*

### 检测判据：幅值 + 倾角双条件

VQF 不依赖于单一特征，而是联合使用磁场的**总强度**和**倾角**：

1. 计算当前磁力计读数的幅值 $n = \|\mathbf{m}\|$ 和倾角 $\theta$
2. 倾角计算：将 $\mathbf{m}$ 变换到 $\mathcal{E}_i$ 帧，取垂直分量与水平分量的反正弦
3. 对 $n$ 和 $\theta$ 做 $\tau = 0.05\,\text{s}$ 的低通滤波去噪
4. **双条件判断**（须同时满足）：

$$
|n - n_{\text{ref}}| < 0.1\, n_{\text{ref}} \quad \text{AND} \quad |\theta - \theta_{\text{ref}}| < 10^\circ
$$

5. 满足条件持续 $\geq 0.5\,\text{s}$ → 标记为"未干扰"
6. 未干扰时用 $k_{\text{ref}}$ 速率缓慢更新 $n_{\text{ref}}$ 和 $\theta_{\text{ref}}$，跟踪环境磁场的缓慢变化

### 三级拒绝策略

| 阶段 | 条件 | 行为 |
|:----:|------|------|
| 短时拒信 | 检测到干扰后 $< 60\,\text{s}$ | **完全禁用**航向校正（$\hat{\delta}_i$ 仅靠陀螺积分维持） |
| 长时降权 | $\geq 60\,\text{s}$ 持续干扰 | 以 $k_{\text{mag}}/2$ 进行半速更新，允许缓慢跟踪 |
| 恢复 | 干扰消失 | 以 $k_{\text{mag}}$ 正常更新；拒绝计数器以 2 倍速衰减（$T_{\text{reject}} \leftarrow T_{\text{reject}} - 2T_s$） |

### 新磁场接受机制

当环境磁场发生永久性改变（如换房间），算法自动适应：

- 持续追踪候选参考值 $n_{\text{cand}}, \theta_{\text{cand}}$
- 满足以下条件时接受为新参考：
  - 当前处于"干扰"状态
  - 候选值一致持续 $\geq 20\,\text{s}$
  - 期间陀螺范数 $\|\boldsymbol{\omega}\| \geq 20\degree/\text{s}$（确保不是静止时的局部畸变）

## 离线版 OflineVQF

后处理时采用**前向-后向**零相滤波：正向 VQF → 反向 VQF → 协方差加权融合偏置 $\hat{\mathbf{b}} = (\mathbf{P}_1^{-1} + \mathbf{P}_2^{-1})^{-1}(\mathbf{P}_1^{-1}\hat{\mathbf{b}}_1 - \mathbf{P}_2^{-1}\hat{\mathbf{b}}_2)$ → 干扰状态取"与" → 重新积分 + `filtfilt` 加计 LPF。6D RMSE 从 1.12° 降至 0.88°（+20%）。

## 实验结果深度分析

### 数据集构成与评价方法

| 数据集 | trials | 运动类型 | 传感器 | 采样率 |
|:------:|:------:|----------|:------:|:------:|
| BROAD | 39 | 旋转/平移/组合/慢/快/间歇，含 6 类扰 | ADIS16448 | 200 Hz |
| Sassari | 18 | 3 种速度 × 3 种 IMU × 2 个样机 | 3 种不同型号 | 128-286 Hz |
| RepoIMU | 21 | T-Stick 运动 | 1 种 | 100 Hz |
| OxIOD | 71 | 手提/手持/口袋/跑步/慢走/推车 | 手机 IMU | 200 Hz |
| TUM VI | 6 | 室内手持（无磁力计） | VI 传感器 | 200 Hz |
| EuRoC MAV | 6 | 微飞行器（无磁力计） | ADIS16448 | 200 Hz |

**评价指标**：SOP（TAGPx）——先按数据集平均，BROAD 权重 ×5，其他数据集等权；对无磁力计数据（TUM VI、EuRoC MAV）和 RIANN 使用倾角误差代替方向误差。

**参数确定**：在 $\tau_{\text{acc}} \in [1, 10]$ s、$\tau_{\text{mag}} \in [1, 30]$ s 网格上搜索 TAGPx 最小值。为同时服务 VQF 和 BasicVQF，选取 $(3\,\text{s}, 9\,\text{s})$ 使二者误差平均值最小。

[📷 _llm/raw/assets/papers/vqf2022/vqf_p8_fig3.jpg|620]
*Figure 8 — 各算法 6D（左）与 9D（右）的加权平均 RMSE*

| 算法 | 6D RMSE | 9D RMSE | 特性 |
|:----:|:-------:|:-------:|------|
| **OflineVQF** | **0.88°** | **2.37°** | 离线前后向滤波 |
| **VQF** | **1.12°** | **2.94°** | 默认参数 |
| BasicVQF | 1.39° | 3.92° | 无偏置/磁拒绝 |
| RIANN | 1.32° | — | 神经网络，6D only |
| SEL | 2.37° | 5.34° | Seel & Ruppin |
| VAC | 2.89° | 6.72° | Valenti 互补滤波 |
| MKF | 3.81° | 8.57° | MATLAB EKF |
| MAD | 6.34° | 11.83° | Madgwick |
| MAH | 4.99° | 16.73° | Mahony |
| FKF | — | 11.49° | 快速 Kalman |
| KOK | — | 14.63° | Kok & Schon |

> RIANN 是唯一 6D 接近 VQF 的算法，但它是神经网络（黑盒）、仅在 6D 可用、训练集与评测集有重叠。VQF 是**白盒工程算法**，6D 和 9D 均最优。

### BROAD 分运动类型对比

VQF 在 7 种运动特性组和 4/6 种干扰组中超越所有对比方法的**各自最优**。仅有的例外：敲击（tapping）稍逊 SEL（加速度瞬变超出 LPF 能力）；振动（vibration）倾角误差低于 FKF 但总误差略高。

### 偏置估计性能

| 条件 | VQF 偏置降幅 | 最佳对比方法偏置降幅 |
|:----|:------------:|:-------------------:|
| BROAD 全 trials | 90% (残余 0.033°/s) | VAC 79% |
| BROAD 切去首尾静止 | 39% (残余 0.204°/s) | SEL 18% |
| Sassari | 89% (残余 0.092°/s) | VAC 39% |
| OflineVQF (BROAD) | 97% (残余 0.008°/s) | — |

VQF 的默认参数在偏置估计上**超过**其他算法经过独立优化后的性能。

### 计算量

[📷 _llm/raw/assets/papers/vqf2022/vqf_p11_fig1.jpg|560]
*Figure 11 — 各算法单步执行时间 vs 精度 (AMD Ryzen 5 3600)：VQF 在 C++ 实现中达到相同数量级的计算量与最高精度*

| 算法 | 6D 单步耗时 | 9D 单步耗时 | 语言 |
|:----:|:----------:|:----------:|:----:|
| BasicVQF | 121 ns | 186 ns | C++ |
| VQF | 280 ns | 387 ns | C++ |
| OflineVQF | 741 ns | 940 ns | C++ |
| VAC | 96.3 ns | 149 ns | C++ |
| MAH | 29.3 ns | 48.7 ns | C++ |
| MAD | 40.4 ns | 59.7 ns | C++ |
| SEL | 165 ns | 277 ns | C++ |
| KOK | — | 4.81 µs | C++ |
| FKF | — | 12.6 µs | C++ |
| MKF | 225 µs | 265 µs | MATLAB |
| RIANN | 15.9 µs | — | ONNX |

## 与主流方法的本质区别

| 维度 | Madgwick (梯度下降) | Mahony (互补滤波) | EKF (误差状态) | VQF |
|:----|:------------------:|:-----------------:|:--------------:|:---:|
| **滤波架构** | 反馈回路 | 反馈回路 | 反馈回路（全状态） | **串联前馈** |
| **状态表示** | 单一四元数 | 四元数 + 偏置 | 全状态向量 | **三级分解**（Strapdown × 倾斜 × 标量航向） |
| **倾斜校正** | 传感器系梯度下降 | 传感器系 PI 补偿 | 传感器系非线性观测 | **准惯性系线性 LPF** |
| **磁力计处理** | 融合入整体梯度 | 融合入整体 PI | 作为观测量 | **解耦标量一阶滤波** |
| **磁干扰→倾角传染** | 是 | 是 | 是 | **否（结构免疫）** |
| **6D/9D 同时输出** | 否 | 否 | 否 | **是** |
| **偏置估计** | 积分器反馈 | PI 积分项 | 作为状态 | **Kalman 滤波器（静止+运动）** |
| **偏置受磁干扰影响** | 是 | 是 | 是 | **否（仅用加计观测）** |
| **参数可解释性** | 无（增益 $\beta$） | 中（$K_P, K_I$） | 差（协方差矩阵） | **好（时间常数 $\tau$，有物理含义）** |
| **参数数量** | 2 | 2 | 5-8+ | **2（$\tau_{\text{acc}}, \tau_{\text{mag}}$）** |
| **默认参数跨场景** | 差 | 差 | 差 | **好（单一参数覆盖全数据集）** |

> [!note] 反馈回路 vs 串联前馈的本质区别
> 所有传统方法都有一个公共缺陷：磁力计校正的输出会通过反馈回路修改状态，而这个修改后的状态又反过来影响下一次的加计校正。VQF 的三级串联使加计倾斜校正只接收 Strapdown 积分结果，完全无视磁力计的存在——这种**结构性免疫**不是靠调参实现的，而是由架构保证的。

## 工程实现要点

### C++ / Python API

C++：`VQF vqf(Ts); vqf.update(gyro, acc, mag, q_6d, q_9d);`
Python：`vqf = VQF(sample_period=0.01); q_6d, q_9d = vqf.update(gyr, acc, mag)`

### 采样率适配

- VQF 内部参数自动适配采样率：$\tau_{\text{acc}}$、$\tau_{\text{mag}}$ 以及 Kalman 滤波器的协方差映射均包含 $T_s$ 因子
- 实测验证范围：**100 Hz – 1600 Hz**（Cortex M4）
- **无需针对不同采样率调参**

核心运算：四元数乘法 + 3 轴二阶 IIR 滤波 + Kalman 5×3 矩阵运算。VQF 的 C++ 实现在 Cortex M4 上以 1600 Hz 通过验证。

## 开源实现

| 语言 | 链接 |
|------|------|
| C++ | [github.com/dlaidig/vqf](https://github.com/dlaidig/vqf) |
| Python | `pip install vqf` |
| MATLAB | 含在仓库中 |
| Rust | [crates.io/vqf-rs](https://crates.io/crates/vqf-rs) |

MIT 许可证。

## 参见

- [IMU姿态解算算法演进](IMU姿态解算算法演进.md) — 全景对比
- [梯度下降姿态解算](梯度下降姿态解算.md) — Madgwick 数学基础
- [误差状态卡尔曼滤波](误差状态卡尔曼滤波.md) — EKF 方法对比
- [MDR 磁畸变抑制](MDR 磁畸变抑制.md) — 磁干扰抑制专题
- [ICM-42688-P](ICM-42688-P.md)
