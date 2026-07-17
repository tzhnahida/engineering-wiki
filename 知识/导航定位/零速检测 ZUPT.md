---
type: concept
tags: [zupt, imu, navigation, zero-velocity, likelihood-ratio, foot-mounted, glrt, svm]
created: 2026-06-24
updated: 2026-07-15
sources:
  - "[2026-06-24 - Skog 零速检测算法评估](../../来源/2026-06-24%20-%20Skog%20零速检测算法评估.md)"
  - "[2026-06-24 - IPIN 2017 实时步态分类 ZUPT](../../来源/2026-06-24%20-%20IPIN%202017%20实时步态分类%20ZUPT.md)"
  - "[2026-06-24 - Sun 行星表面 ZUPT 多步态评估](../../来源/2026-06-24%20-%20Sun%20行星表面%20ZUPT%20多步态评估.md)"
---

# 零速检测 ZUPT

> 脚装 IMU 检测"脚着地了没"——GLRT 统计框架统一四种检测器，角速度能量检测器最简单最优，自适应阈值让混合运动误差降低 63%。

---

## 为什么需要 ZUPT：三次方误差

纯惯性积分的位置误差 $\propto t^3$。对低成本 MEMS：

$$
\sigma_p(t) \approx \frac{1}{6} \cdot b_a \cdot t^3
$$

| 时间 | 位置误差（典型 1 mg 零偏） |
|------|:---:|
| 1 秒 | 1.6 mm |
| 10 秒 | **1.6 m** |
| 60 秒 | **580 m** |

但人走路时脚周期性触地——每步有 0.1~0.3 秒完全静止。这是天然锚点。

[📷 _llm/raw/assets/papers/skog2010/skog_p7_fig1.jpg|460]
*Fig. 4 — 支撑相（stance phase）与加计/陀螺原始输出的时间对应关系：脚触地期间信号明显"安静"，这就是零速检测的物理依据*

> [Foxlin (2005)](脚装惯性导航奠基.md) 第一个提出鞋装 IMU + ZUPT 的完整方案，用"软"ZUPT（Kalman 测量更新）替代"硬"归零。Skog (2010) 把零速检测问题形式化为 GLRT。

---

## 传感器模型

IMU 输出 = 真值 + 噪声：

$$y_k = s_k(\theta) + v_k, \quad v_k \sim \mathcal{N}(0, C)$$

$$C = \begin{bmatrix} \sigma_a^2 I_3 & 0 \\ 0 & \sigma_\omega^2 I_3 \end{bmatrix}$$

两个假设：

| | H₀（运动） | H₁（静止） |
|------|------|------|
| 加计 | 任意 | $\|s^a_k\| = g$，方向恒定 |
| 陀螺 | 任意 | $s^\omega_k = 0$ |
| 未知参数 | 全部 $\{s_k\}$ | 仅重力方向 $\mathbf{u}_n$ |

---

## GLRT: Stance Hypothesis Optimal dEtector (SHOE)

由于参数未知，用广义似然比检验（GLRT）——代入 MLE 替代未知参数：

$$L_G(z_n) = \frac{p(z_n; \hat{\theta}_1, H_1)}{p(z_n; \hat{\theta}_0, H_0)} > \gamma$$

推导后得到 SHOE 检测器（一篇公式推下来，最终化简为）：

$$T(z_n) = \frac{1}{N} \sum_{k=n}^{n+N-1} \left( \frac{1}{\sigma_a^2} \left\| y_k^a - g \frac{\bar{y}_n^a}{\|\bar{y}_n^a\|} \right\|^2 + \frac{1}{\sigma_\omega^2} \|y_k^\omega\|^2 \right)$$

判据：$T(z_n) < \gamma'$ → 静止

**物理含义**：加计和陀螺测量的"偏离理想静止信号"程度，各自按噪声方差的倒数加权，窗口平均。低于阈值 → 大概率静止。

[📷 _llm/raw/assets/papers/skog2010/skog_p6_fig3.jpg|420]
*Fig. 3 — 四种检测统计量 $T(z_n)$ 随时间的输出：静止段（stance）统计量显著低于摆动段，阈值 γ 水平切割即得零速判决*

---

## 四种检测器（全部从 GLRT 退化而来）

### 1. 加速度移动方差检测器

$$T_v(z_n^a) = \frac{1}{N} \sum_{k=n}^{n+N-1} \|y_k^a - \bar{y}_n^a\|^2 < \gamma_v$$

假设：静止时加计**方向恒定**（但不约束幅值为 g）。

### 2. 加速度幅值检测器

$$T_m(z_n^a) = \frac{1}{N} \sum_{k=n}^{n+N-1} (\|y_k^a\| - g)^2 < \gamma_m$$

假设：静止时加计**幅值为 g**（但不约束方向恒定）。

### 3. 角速度能量检测器

$$T_\omega(z_n^\omega) = \frac{1}{N} \sum_{k=n}^{n+N-1} \|y_k^\omega\|^2 < \gamma_\omega$$

假设：静止时陀螺**输出为零**。这是 SHOE 在 $\sigma_a^2 \to \infty$ 时的退化。

### 性能对比

| 检测器 | 传感器 | 慢走 PSD | 正常走 PSD | 虚警率控制 |
|--------|:---:|:---:|:---:|:---:|
| **角速度能量** | 陀螺 | ⭐ 最高 | ⭐ 最高 | ⭐ |
| **SHOE** | 加计+陀螺 | ⭐ 最高 | ⭐ 最高（略优） | ⭐ |
| 加速度移动方差 | 加计 | 🟡 弱 | 🟡 弱 | 🟡 |
| 加速度幅值 | 加计 | 🔴 最弱 | 🔴 最弱 | 🔴 |

> **结论**：角速度能量 ≈ SHOE ≫ 纯加计方案。陀螺信号比加计信号**干净得多**——信号噪声比更高，且"静止=零角速度"的先验比"静止=重力方向"更确定。

[📷 _llm/raw/assets/papers/skog2010/skog_p7_fig3.jpg|400] [📷 _llm/raw/assets/papers/skog2010/skog_p8_fig1.jpg|400]
*Fig. 5 / Fig. 6 — 慢走（左）与正常步速（右）下四种 GLRT 检测器的检测概率-虚警概率曲线：角速度能量与 SHOE 几乎重合于左上角，两种纯加计检测器明显落后*

$\sigma_a = 0.02$ m/s², $\sigma_\omega = 0.1$ °/s, N=5 @ 30 Hz ($\approx$ 150 ms 窗口)。

---

## PSD: 更适合 ZUPT 的度量

传统的 PD/PFA（检测概率/虚警概率）ROC 曲线对 ZUPT 有误导——你不需要检测**每个**静止采样点，只需要每个步态周期抓到**至少一次**零速。

引入 PSD（Probability of Stationarity Detection）：

$$
\text{PSD} = \Pr(\text{静止段内至少一次 } H_1)
$$

| PFA | 慢走 PSD (SHOE) | 正常走 PSD (SHOE) |
|:---:|:---:|:---:|
| 10⁻⁴ | 0.95 | 0.80 |
| 10⁻³ | 0.99 | 0.92 |
| 10⁻² | ~1.0 | 0.98 |

> **工程策略**：把 PFA 压到 10⁻³~10⁻⁴，PSD 仍有 80-95%。宁可漏检、不能误检——一次错误零速更新对导航解的破坏远超少一次正确更新。

---

## 自适应阈值：不同动作不同 γ

ZUPT 的死穴：**走路调好的阈值，跑起步来全部失效**。

原因：跑步时着地时间短、冲击大、角速度峰值高→角速度能量分布整体上移→走路阈值太低，大量零速事件被漏检。

### IPIN 2017 方案

SVM 分类器（RBF 核），1 秒窗口（125 样本 @ 125 Hz），输入 6 轴原始 IMU 数据归一化：

[📷 _llm/raw/assets/papers/wagstaff2017/wagstaff_p1_fig3.jpg|540]
*Fig. 1 — 系统架构：SVM 依惯性数据分类动作 → 按动作切换零速检测阈值 → ZUPT-EKF 融合*

| 动作 | 分类准确率 |
|------|:---:|
| 走路 | 98.1% |
| 慢跑 | 87.1% |
| 跑步 | 87.3% |
| 冲刺 | 92.0% |
| 匍匐 | 85.6% |
| 爬梯 | 96.5% |
| **平均** | **91.1%** |
| 走/跑二分类 | **99.9%** |

### 每类动作的独立最优 γ

通过 Vicon 动捕做零速真值 → 扫描 γ → 最大化 F-score:

$$F_\beta = (1+\beta^2)\frac{P \cdot R}{\beta^2 P + R}$$

| 动作 | 最优 γ (×10⁻⁵) | 相对于走路 |
|------|:---:|:---:|
| 走路 | 0.96 | ×1 |
| **跑步** | **13.11** | **×13.7** |

[📷 _llm/raw/assets/papers/wagstaff2017/wagstaff_p5_fig3.jpg|420]
*Fig. 4 — 用 Vicon 真值扫描零速阈值：不同动作的最优 γ 相差一个数量级以上*

> 跑步阈值是走路的 **14 倍**——如果只用走路的阈值跑起步，大部分零速事件被漏检，位置误差膨胀至 128m（5 名被试均值）vs 4.3m（用跑步专用阈值）。

### 混合运动实验（5 名被试，5.9 km）

| 方案 | 最远点误差 (m) |
|------|:---:|
| 固定走路 γ | 7.35 |
| 固定跑步 γ | 3.30 |
| **自适应（SVM 切 γ）** | **2.68** |

[📷 _llm/raw/assets/papers/wagstaff2017/wagstaff_p7_fig3.jpg|440]
*Fig. 6 — 走廊实验轨迹对比：固定走路阈值（漂移最大）vs 固定跑步阈值 vs 自适应切换（最贴合真值标记点）*

> 自适应降低误差 63%（vs 走路阈值）、19%（vs 跑步阈值）。纯单动作时自适应 = 对应最优固定阈值（分类正确率 >95%）。

---

## 软 ZUPT vs 硬 ZUPT

| | 硬 ZUPT | 软 ZUPT |
|------|------|------|
| 操作 | 速度直接 = 0, 位置不动 | EKF 测量更新：$z = v_{meas} = 0$ |
| 优点 | 简单，不需 EKF | 保留不确定性，多源融合友好 |
| 缺点 | 姿势也重置（IMU 不一定是平的） | 需要建 EKF |
| 误差传播 | 零速段无误差累积 | R 矩阵表征零速置信度 |

---
## 相关页面

- [脚装惯性导航奠基](脚装惯性导航奠基.md) — Foxlin 2005：鞋装 IMU 奠基，软 ZUPT 概念，EKF 行人追踪

## 关键词

`GLRT` `Neyman-Pearson` `SHOE detector` `angular rate energy` `PSD` `F-score` `SVM motion classification` `adaptive thresholding` `soft ZUPT` `EKF measurement update`
