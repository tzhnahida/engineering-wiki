---
type: concept
tags: [imu, bayesian, lie-group, so3, orientation, probability]
created: 2026-06-26
updated: 2026-07-16
sources:
  - "[2026-07-15 - Lee Matrix Fisher SO3 姿态估计](2026-07-15 - Lee Matrix Fisher SO3 姿态估计.md)"
---

# Matrix Fisher SO(3) 概率姿态估计

> SO(3) 上直接输出旋转矩阵概率分布的贝叶斯姿态估计框架——Taeyoung Lee (2018, arXiv 1710.03746)。两种传播方案：解析矩匹配与 Unscented 变换。核心卖点是**大初始误差下的优越收敛性**。

## 为什么需要 SO(3) 上的概率分布

传统姿态估计有两个根深蒂固的表示问题：

- **欧拉角 / 修正罗德里格斯参数** 存在奇异性，不适合大幅机动
- **四元数** 虽无奇异性，但 $S^3$ 双覆盖 $SO(3)$（$q$ 与 $-q$ 表示同一姿态），导致协方差矩阵在单位范数约束下退化

SO(3) 上的确定性观测器（如互补滤波 [Mahony 2008]）可避免表示奇异性，但它们是**确定性的**——只输出点估计，不提供姿态不确定性的概率描述。非交换调和分析 [Chirikjian 2001] 可以在 SO(3) 上定义任意概率密度，但涉及复杂酉表示积分，实时计算不可行。

**Matrix Fisher 分布在 SO(3) 上提供了一个紧凑的 9 参数指数族分布**——与 $\mathbb{R}^3$ 中的高斯分布（3 均值 + 6 协方差）参数数量相同——既避免了四元数的歧义，又避免了调和分析的计算负担。

## Matrix Fisher 分布定义

### 概率密度函数

随机旋转矩阵 $\mathbf{R} \in SO(3)$ 服从 Matrix Fisher 分布，记作 $\mathbf{R} \sim \mathcal{M}(\mathbf{F})$，其概率密度函数（相对于 SO(3) 上的归一化 Haar 测度 $\int_{SO(3)} d\mathbf{R} = 1$）为：

$$
p(\mathbf{R} \mid \mathbf{F}) = \frac{1}{c(\mathbf{F})} \exp\big(\mathrm{tr}(\mathbf{F}^\top \mathbf{R})\big)
$$

其中 $\mathbf{F} \in \mathbb{R}^{3\times 3}$ 是参数矩阵，$c(\mathbf{F})$ 是归一化常数：

$$
c(\mathbf{F}) = \int_{SO(3)} \exp\big(\mathrm{tr}(\mathbf{F}^\top \mathbf{R})\big) \, d\mathbf{R}
$$

### Proper 奇异值分解

令 $\mathbf{F}$ 的 SVD 为 $\mathbf{F} = U' S' (V')^\top$。$U', V'$ 的行列式可能为 $-1$，不保证属于 $SO(3)$。Lee 引入 **proper SVD** 解决此问题：

$$
\mathbf{F} = U S V^\top, \quad U, V \in SO(3), \quad S = \mathrm{diag}[s_1, s_2, s_3]
$$

其中 $U = U' \mathrm{diag}[1,1,\det U']$，$V$ 同理。第三个 proper 奇异值 $s_3$ 可能为负，且满足 $0 \leq |s_3| \leq s_2 \leq s_1$。

### 归一化常数的一维积分

**定理 II.1**：$c(\mathbf{F})$ 只依赖于 proper 奇异值 $S$，且可通过一维积分计算：

$$
c(S) = \int_{-1}^{1} \frac{1}{2} I_0\!\left(\tfrac{1}{2}(s_i - s_j)(1-u)\right) I_0\!\left(\tfrac{1}{2}(s_i + s_j)(1+u)\right) e^{s_k u} \, du
$$

其中 $(i,j,k)$ 是 $(1,2,3)$ 的任意循环排列，$I_0, I_1$ 为修正 Bessel 函数。$c(S)$ 的一阶、二阶导数也有闭式表达——这些是后续矩匹配的关键。论文附录还给出了**指数缩放版本** $\bar{c}(S) = e^{-\mathrm{tr}[S]}c(S)$，避免大奇异值下的数值溢出。

### 累积分布函数

当 $\mathbf{F} = s\mathbf{I}_{3\times3}$（三个奇异值相等）时，CDF 有简明的表达式：

$$
\mathrm{Prob}[ \angle(\mathbf{R}, \mathbf{M}) \leq \theta ] = \frac{1}{\pi(I_0(2s) - I_1(2s))} \int_0^\theta e^{2s\cos\rho} (1-\cos\rho) \, d\rho
$$

[📷 _llm/raw/assets/papers/leefisher2018/fisher_p6_fig1.jpg|480]
*Fig. 1 — $M(sI_{3\times3})$ 的累积分布函数。$s$ 越大分布越集中；$s=0$ 退化为均匀分布。$s=100$ 时姿态角误差 $<10^\circ$ 的概率为 0.9。*

## 均值姿态与一阶矩

### 均值姿态

**定理 II.3**：Matrix Fisher 分布的**最大均值（max mean）** 与 **MMSE 均值** 等价：

$$
\mathcal{M}_{\max}[\mathbf{R}] = \mathcal{M}_{\mathrm{mse}}[\mathbf{R}] = U V^\top \in SO(3)
$$

即均值姿态由 $\mathbf{F}$ 的 proper SVD 正交因子直接给出。

### 一阶矩

**定理 II.2**：令 $\mathbf{Q} = U^\top \mathbf{R} V \sim \mathcal{M}(S)$，则 $\mathbf{Q}$ 的一阶矩为对角矩阵：

$$
\mathbb{E}[\mathbf{Q}_{ii}] = \frac{1}{c(S)} \frac{\partial c(S)}{\partial s_i} = \frac{\partial \log c(S)}{\partial s_i}, \quad \mathbb{E}[\mathbf{Q}_{ij}] = 0\ (i \neq j)
$$

$\mathbf{R}$ 的一阶矩为 $\mathbb{E}[\mathbf{R}] = U \, \mathbb{E}[\mathbf{Q}] \, V^\top$。注意 $\mathbb{E}[\mathbf{R}]$ 本身不一定属于 $SO(3)$，它只是一个 $\mathbb{R}^{3\times3}$ 矩阵。

> [!note] 从样本均值重构 F
> 给定样本均值 $\bar{\mathbf{R}}$ 的 proper SVD $\bar{\mathbf{R}} = U D V^\top$，可通过求解 $\frac{1}{c(S)} \frac{\partial c(S)}{\partial s_i} = d_i$ 得到 $S$，进而重构 $\mathbf{F} = U S V^\top$。这就是定理 II.6 的最大似然估计过程——也是后续滤波器中传播步骤的核心。

## 几何解释

$\mathbf{F} = U S V^\top$ 的三种成分有直观几何含义：

1. **均值姿态** $\mathbf{M} = UV^\top$
2. $U$ 的**第 $i$ 列** $U e_i$ 定义了 SO(3) 上的**第 $i$ 个主轴**
3. $s_j + s_k$（$(i,j,k)$ 循环排列）决定了绕轴 $U e_i$ 旋转方向上的**集中度**

沿第 $i$ 主轴的概率密度为：

$$
p\big(\mathbf{R}_i(\theta_i)\big) \propto \exp\big((s_j + s_k) \cos \theta_i\big)
$$

这类似于圆上的 von Mises 分布，集中参数 $\kappa = s_j + s_k$。当 $s_j + s_k$ 足够大时，$\theta_i$ 近似服从方差为 $1/(s_j + s_k)$ 的高斯分布。

[📷 _llm/raw/assets/papers/leefisher2018/fisher_p8_fig1.jpg|520]
*Fig. 3 — $U$ 和 $V$ 对分布形状的几何作用：左乘旋转矩阵 $A$ 同时改变均值姿态和主轴方向（左图）；同时调整 $U$ 和 $V$ 可仅改变主轴方向而不改变均值姿态（右图）。*

**示例**：$\mathbf{F}_c = \mathrm{diag}[25,5,1]$ 时均值姿态为 $\mathbf{I}_{3\times3}$，主轴为 $(e_1,e_2,e_3)$。$s_2+s_3 = 6$（绕 $e_1$ 轴最分散），$s_1+s_2 = 30$（绕 $e_3$ 轴最集中）。这与多元高斯中协方差矩阵的特征值分解完全对应——$U$ 的列就是 SO(3) 上的"特征向量"。

## 贝叶斯姿态估计：问题表述

姿态运动学由 SO(3) 上的 Ito 随机微分方程描述：

$$
(\mathbf{R}^\top d\mathbf{R})^\vee = \boldsymbol{\Omega}\, dt + \mathbf{H}\, d\mathbf{W}
$$

其中 $\boldsymbol{\Omega} \in \mathbb{R}^3$ 是体坐标系角速度（陀螺测量），$\mathbf{H}$ 是噪声缩放矩阵，$d\mathbf{W}$ 是 Wiener 过程增量。

时间离散化为步长 $h$，下标 $k$ 表示第 $k$ 时刻。先验分布 $\mathbf{R}_k \sim \mathcal{M}(\mathbf{F}_k)$。

## 框架一：一阶（解析矩传播）姿态估计

### 预测步骤

**定理 III.1**：若 $\mathbf{R}_k \sim \mathcal{M}(\mathbf{F}_k)$，$\boldsymbol{\Omega}(t)$ 在 $[t_k, t_{k+1}]$ 上恒定，则 $t_{k+1}$ 时 $\mathbf{R}_{k+1}$ 的一阶矩为：

$$
\mathbb{E}[\mathbf{R}_{k+1}] = \mathbb{E}[\mathbf{R}_k] \left( \mathbf{I}_{3\times3} + \frac{h}{2}\big(-\mathrm{tr}[\mathbf{G}_k] \mathbf{I}_{3\times3} + \mathbf{G}_k\big) \right) \exp(h\hat{\boldsymbol{\Omega}}_k) + \mathcal{O}(h^{1.5})
$$

其中 $\mathbf{G}_k = \mathbf{H}_k \mathbf{H}_k^\top$。推导基于 Magnus 展开（附录 B）和 Ito 引理，利用了随机微分方程在零漂移变换下的特性。

传播后的 $\mathcal{M}(\mathbf{F}_{k+1})$ 通过**一阶矩匹配**得到：
1. 计算 $\mathbb{E}[\mathbf{R}_{k+1}]$ 的 proper SVD $\to U, D, V$
2. 求解 $\frac{1}{c(S)}\frac{\partial c(S)}{\partial s_i} = d_i$（Newton 法，利用 $c$ 的二阶导数）
3. 输出 $\mathbf{F}_{k+1} = U S V^\top$

### 更新步骤——共轭性

Matrix Fisher 分布在向量观测下具有**共轭**性质，这是该方法最优雅的特性：

**定理 III.2**：先验 $\mathbf{R} \sim \mathcal{M}(\mathbf{F})$，考虑两类测量：

- **全姿态测量** $\mathbf{Z}_i \in SO(3)$：测量误差 $\mathbf{R}^\top \mathbf{Z}_i \sim \mathcal{M}(\mathbf{F}_{Z_i})$，即 $p(\mathbf{Z}_i \mid \mathbf{R}) \propto \exp(\mathrm{tr}[\mathbf{F}_{Z_i}^\top \mathbf{R}^\top \mathbf{Z}_i])$
- **方向测量** $\mathbf{z}_i \in S^2$（如重力方向、地磁方向）：似然为 von Mises-Fisher 分布 $p(\mathbf{z}_i \mid \mathbf{R}) \propto \exp(b_i \mathbf{a}_i^\top \mathbf{B}_i^\top \mathbf{R} \mathbf{z}_i)$

后验分布**仍然是 Matrix Fisher**：

$$
\mathbf{R} \mid (\mathbf{Z}_i, \mathbf{z}_j) \sim \mathcal{M}\!\left(\mathbf{F} + \sum_{i=1}^{N_Z} \mathbf{Z}_i \mathbf{F}_{\!Z_i}^\top + \sum_{j=1}^{N_z} b_j \mathbf{B}_j \mathbf{a}_j \mathbf{z}_j^\top\right)
$$

> [!note] 闭环共轭的工程意义
> 更新步骤是**闭式的、精确的**——无需线性化、无需迭代、无需近似。加速度计、磁力计、星敏感器等不同类型的传感器可以任意组合，各自贡献一个加法项到 $\mathbf{F}$ 参数中。这与 EKF 的测量线性化形成鲜明对比。

### 完整算法（原文 Table I）

每个时间步执行：传播（矩匹配一步）$\to$ 若有测量则累加 $\mathbf{F}$ 参数 $\to$ 循环。没有重采样、没有粒子退化、没有 Jacobian 计算。

## 框架二：Unscented 姿态估计

### Sigma 点选取

受 UKF 启发，Lee 提出了 Matrix Fisher 分布在 SO(3) 上的 Unscented 变换。

**定义 IV.1**：给定 $\mathcal{M}(\mathbf{F})$ 的 proper SVD $\mathbf{F} = U S V^\top$，选取 **7 个 sigma 点**：

$$
\Sigma_{\mathbf{R}}(\mathbf{F}) = \{\mathbf{R}_0 = UV^\top\} \cup \{\mathbf{R}_i(\theta_i), \mathbf{R}_i(-\theta_i)\}_{i=1,2,3}
$$

其中 $\mathbf{R}_i(\theta_i) = U \exp(\theta_i \hat{e}_i) V^\top$ 是绕主轴 $U e_i$ 旋转 $\theta_i$。旋转角 $\theta_i$ 由集中度参数 $s_j + s_k$ 决定——分为 $s_j+s_k \geq 1$ 和 $< 1$ 两段，保证 σ 点从均匀散布到高度集中平滑过渡（公式 59a-b）。参数 $\sigma \in (0,1)$ 控制 sigma 点的散布程度。

权重 $\{w_0, w_1, w_2, w_3\}$ 的设计保证加权一阶矩 $\sum \tilde{w} \tilde{\mathbf{R}}$ 与 $\mathbb{E}[\mathbf{R}]$ 精确匹配（定理 IV.1）。

[📷 _llm/raw/assets/papers/leefisher2018/fisher_p12_fig2.jpg|520]
*Fig. 4 — 不同 Matrix Fisher 分布（$F_a = 5I$，$F_b = 20I$，$F_c = \mathrm{diag}[25,5,1]$）的 sigma 点体轴可视化。下图为 $\cos \theta_i$ 随 $s_j+s_k$ 的变化曲线。*

### Unscented 传播（原文 Table II）

1. 从 $\mathcal{M}(\mathbf{F}_k)$ 计算 7 个 sigma 点及权重
2. 每个 sigma 点沿无噪声运动学传播：$\mathbf{R}_{k+1}^i = \mathbf{R}_k^i \exp(h\hat{\boldsymbol{\Omega}}_k)$
3. 加权均值 $\to$ 噪声补偿修正（同公式 52）$\to$ proper SVD $\to$ 求解 $S$ $\to$ $\mathbf{F}_{k+1}$

更新步骤与一阶框架完全相同（定理 III.2 的闭式后验），**不需要引入新的 Unscented 更新规则**。

## 数值实验

论文以 **3D 摆**的复杂不规则机动为测试场景（初始角速度 $4.14 \times [1,1,1]$ rad/s），姿态传感器 10 Hz、陀螺 50 Hz，单次姿态测量误差均值 $10.45^\circ$。

### Case I：大初始估计误差 180°

初始估计为绕第一体轴旋转 $180^\circ$ 的错误姿态，且估计器对该错误**过度自信**（$s=100$）：

[📷 _llm/raw/assets/papers/leefisher2018/fisher_p14_fig3.jpg|520]
*Fig. 6 — Case I 估计误差对比：一阶（红线）、Unscented（蓝线）。初始误差 $180^\circ$，三次姿态测量后（$t=0.3$s）降至 $4^\circ$ 以下。*

对比结果（原文 Table III）：

| 方法 | 稳态误差 | $s_2+s_3$ | $s_3+s_1$ | $s_1+s_2$ |
|------|---------|-----------|-----------|-----------|
| 一阶矩匹配 | $6.32^\circ$ | 95.89 | 119.78 | 124.88 |
| Unscented | $6.32^\circ$ | 96.14 | 119.70 | 124.98 |
| **MEKF** | **$10.18^\circ$** | — | — | — |

> [!note] 核心洞察
> 尽管初始误差高达 $180^\circ$，Matrix Fisher 估计器仅需 **3 次姿态测量**（0.3 秒）就将误差收敛到 $4^\circ$ 以下。MEKF 的稳态误差 $10.18^\circ$ 明显更大——这是因为大初始误差下四元数协方差退化导致线性化失效。**真正困难的不只是"大误差"，而是"错得自信"——算法必须同时修正错误估计和错误的置信度。**

### Case II：大初始不确定性

初始 $\mathbf{F}(0) = \mathbf{0}_{3\times3}$，即 SO(3) 上的均匀分布（完全未知姿态）。Unscented 滤波器在此场景下**明显优于**一阶滤波器（一阶 $8.70^\circ$，Unscented $7.91^\circ$），因为 sigma 点更好地捕获了均匀分布的多模态特性。

## 与 MEKF 的理论对比

| 维度 | MEKF | Matrix Fisher（一阶） | Matrix Fisher（Unscented） |
|------|------|---------------------|--------------------------|
| 表示空间 | 四元数 $S^3$（双覆盖 + 范数约束） | SO(3)（无约束） | SO(3)（无约束） |
| 不确定性描述 | 3×3 协方差 | 9 参数 $\mathbf{F}$ 矩阵 | 7 个 sigma 点 |
| 初始化条件 | 需小误差假设 | 任意误差 | 任意误差 |
| 测量更新 | 线性化（EKF）或 sigma 点（UKF） | 共轭闭式 | 共轭闭式 |
| 大误差收敛 | 可能发散或缓慢 | 快速收敛（已验证 $180^\circ$） | 快速收敛 |

论文的核心结论：**在 SO(3) 上全局定义的概率分布使得估计器自然适应大误差场景——这是 MEKF 和 UKF 基于局部线性化/局部 sigma 点方法做不到的。**

## Jin 2025：数据驱动的深度学习扩展

> 以下内容对应 Jin 等人 2025 年发表于 *Information Fusion* 的论文。**库内暂无该 PDF。**

Jin 2025 将 Lee 2018 的贝叶斯框架与深度学习结合：
- 用神经网络直接输出 Matrix Fisher 参数 $\mathbf{F}$（9 维）
- 训练损失为负对数似然 $\mathcal{L} = -\log p(\mathbf{R} \mid \mathbf{F})$
- 网络输出 $\mathbf{F}$ 后，均值姿态 $M = UV^\top$ 和集中度 $S$ 同时解码
- 支持端到端的多传感器数据融合

该路线将 Matrix Fisher 从**纯贝叶斯估计**扩展到**数据驱动的姿态回归**，但失去了 Lee 2018 中严格的共轭贝叶斯更新框架——后验不再有闭式形式。

## 参见

- [误差状态卡尔曼滤波](误差状态卡尔曼滤波.md) — ESKF 的误差状态表示与 Matrix Fisher 的全局分布对比
- [AVNet 不变扩展卡尔曼姿态](AVNet 不变扩展卡尔曼姿态.md) — InEKF 也在 SO(3) 上工作，但以不变观测驱动而非显式概率分布
- [IMU姿态解算算法演进](IMU姿态解算算法演进.md) — 姿态估计算法全景图
