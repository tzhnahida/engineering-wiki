---
type: concept
tags: [姿态解算, 生物力学, 运动链, EKF]
created: 2026-07-03
updated: 2026-07-15
sources:
  - "[2026-06-26 - Foxlin 鞋装惯性传感器行人追踪](../../来源/2026-06-26%20-%20Foxlin%20鞋装惯性传感器行人追踪.md)"
  - "医学 ROM 数据来自 Kapandji 关节生理学等标准参考"
---

# 医学 ROM 约束的关节节点模型

## 动机

传统惯性动捕的运动链约束只用骨骼长度不变（`|l_ij| = const`），这是一个纯几何约束——只能保证骨架不散架，不能防止关节摆出物理上不可能的姿势。

引入**医学 ROM（Range of Motion）数据**，从纯几何约束升级为**生物力学可行域约束**：

| 约束层级 | 来源 | 作用 |
|:---:|------|------|
| L1: 骨骼长度 | 身体测量 | 骨架不散架 |
| L2: 关节 ROM | 医学文献 | 姿态物理可行 |
| L3: 传感器观测 | IMU/UWB | 数据驱动 |

三个层级叠在一起，构成 **biomechanically informed state space**——EKF 的状态向量被约束在一个医学上有意义的流形上。

---

## 1. 节点定义

### 节点数据结构

```
Node {
  id: "L_ELBOW",
  parent: "L_SHOULDER",
  chain_position: 3,          // 运动链深度
  bone_length: 0.27,          // m，个人标定

  // 医学 ROM 约束
  rom: {
    flexion_extension:    [0, 145],    // 屈伸 [min, max] deg
    pronation_supination: [-80, 85],   // 旋前旋后
    carrying_angle:       [5, 15],     // 提携角（解剖位固定偏角）
  },

  // EKF 内部
  state: {
    R: SO(3),
    b_g: [3]float64,
    b_a: [3]float64,
  },
  covariance: [6][6]float64,
}
```

### 15 节点拓扑

```
          HEAD (1)
           │
   NECK ───┴─── TRUNK (2)
           │
    ┌──────┴──────┐
    │             │
 L_SHOULDER(3)  R_SHOULDER(9)
    │             │
 L_UPPER_ARM(4) R_UPPER_ARM(10)
    │             │
 L_ELBOW(5)     R_ELBOW(11)
    │             │
 L_FOREARM(6)   R_FOREARM(12)
    │             │
 L_WRIST(7)     R_WRIST(13)
    │             │
 L_HAND(8)      R_HAND(14)

 PELVIS(0) ──── 根节点
    │
 L_THIGH(15)
    │
 L_KNEE(16)
    │
 L_SHANK(17)
    │
 L_FOOT(18)
    │
 (右侧对称 R_THIGH..R_FOOT)
```

---

## 2. 医学 ROM 数据

数据来源：American Academy of Orthopaedic Surgeons (AAOS) 标准值 + **《中国成年人人体尺寸》GB/T 10000-1988** 关节活动度附注。

### 2.1 上肢

| 关节 | 自由度 | ROM | 参考来源 |
|------|------|------|------|
| **肩关节** | 屈曲 | 0–180° | AAOS |
| | 伸展 | 0–60° | AAOS |
| | 外展 | 0–180° | AAOS |
| | 内旋 | 0–90° | AAOS |
| | 外旋 | 0–90° | AAOS |
| **肘关节** | 屈曲 | 0–145° | AAOS |
| | 伸展 | 145–0° | 过伸 ≤ 10°（病理性） |
| | 前臂旋前 | 0–80° | AAOS |
| | 前臂旋后 | 0–85° | AAOS |
| **腕关节** | 屈曲 | 0–80° | AAOS |
| | 伸展 | 0–70° | AAOS |
| | 桡偏 | 0–20° | AAOS |
| | 尺偏 | 0–30° | AAOS |

### 2.2 下肢

| 关节 | 自由度 | ROM | 参考来源 |
|------|------|------|------|
| **髋关节** | 屈曲 | 0–120°（屈膝）/ 0–90°（伸膝） | AAOS |
| | 伸展 | 0–30° | AAOS |
| | 外展 | 0–45° | AAOS |
| | 内收 | 0–30° | AAOS |
| | 内旋 | 0–45° | AAOS |
| | 外旋 | 0–45° | AAOS |
| **膝关节** | 屈曲 | 0–135° | AAOS |
| | 伸展 | 135–0° | 不应过伸 |
| **踝关节** | 背屈 | 0–20° | AAOS |
| | 跖屈 | 0–50° | AAOS |
| | 内翻 | 0–35° | AAOS |
| | 外翻 | 0–15° | AAOS |

### 2.3 脊柱与头部

| 关节 | 自由度 | ROM |
|------|------|------|
| **颈椎** | 屈曲 | 0–45° |
| | 伸展 | 0–45° |
| | 侧屈 | 0–45° |
| | 旋转 | 0–80° |
| **胸腰椎** | 屈曲 | 0–80° |
| | 伸展 | 0–25° |
| | 侧屈 | 0–35° |
| | 旋转 | 0–45° |

> [!note] ROM 个人化
> 上述为标准成人均值。运动员、舞者的 ROM 可以超出标准范围 10–30°。系统应支持**个人 ROM 标定**——让用户做全范围关节运动，采集极值，更新约束边界。

---

## 3. 约束数学形式

### 3.1 关节角提取

给定父子节点姿态 $R_i, R_j \in SO(3)$，关节相对旋转：

$$R_{rel} = R_i^T R_j = R_{parent}^{-1} R_{child}$$

将 $R_{rel}$ 分解为关节坐标系下的欧拉角或旋转向量：

$$\theta = f(R_{rel}) \in \mathbb{R}^3$$

关节坐标轴定义依据 ISB（International Society of Biomechanics）推荐标准。

### 3.2 ROM 约束函数

每个自由度上的约束：

$$c_k(\theta) = \begin{cases}
\theta_k - \theta_k^{\max} & \text{if } \theta_k > \theta_k^{\max} \\
\theta_k^{\min} - \theta_k & \text{if } \theta_k < \theta_k^{\min} \\
0 & \text{otherwise}
\end{cases}$$

总 ROM 约束代价：

$$C_{ROM}(x) = \sum_{k \in \text{all DOFs}} w_k \cdot \max(0, c_k(\theta))^2$$

其中 $w_k$ 为各自由度权重（主要关节 DOF 权重高，如肘屈伸；次要 DOF 权重低）。

### 3.3 约束梯度（用于 EKF 更新）

$$\frac{\partial C_{ROM}}{\partial R_i} = 2 w_k \cdot c_k(\theta) \cdot \frac{\partial \theta_k}{\partial R_i}$$

其中 $\frac{\partial \theta_k}{\partial R_i}$ 可以通过 SO(3) 上的李代数导数解析计算：

$$\frac{\partial \theta}{\partial R} \approx \frac{\partial}{\partial \phi} \text{Log}(R \exp(\phi^\wedge)) \Big|_{\phi=0}$$

---

## 4. 融合数据结构

### 4.1 EKF 约束堆栈

```
Constraints = [
  L1_GEOMETRIC:  bone_length_constraint(),    // |l_ij| = const
  L2_BIOMECH:    rom_constraint(),             // θ ∈ [θ_min, θ_max]
  L3_SOFT_ROM:   rom_penalty(),                // 接近边界时渐变惩罚
]
```

### 4.2 三层约束的区别

| 约束层 | 硬度 | 用途 | 违反时 |
|------|:---:|------|------|
| L1 骨骼长度 | 硬约束 | 结构一致性 | 投影回可行域 |
| L2 ROM 硬边界 | 硬约束 | 防逆关节 | 截断 |
| L3 ROM 软边界 | 软约束 | 近极限预警 | EKF 观测量协方差增大 |

### 4.3 ROM 约束与运动链遮挡推断的联动

ROM 约束反过来也能帮助 UWB 遮挡推断：

```
if ROM_constraint_violation(node_i):
    // 该节点可能被异常外力干扰，或 IMU 误差过大
    → 增大该节点 EKF 过程噪声 Q
    → 降低该节点对相邻节点的约束传递权重
```

形成了**双向信息流**：

```
运动链姿态 → ROM 检查 → 约束更新
     ↑                      ↓
     └── 节点可靠性 ← 约束违反检测
```

---

## 5. 与传统方法的本质区别

| | 传统运动链 | Xsens / Noitom | 本方案 |
|---|---|---|---|
| 骨骼长度约束 | ✓ | ✓ | ✓ |
| 关节 ROM 硬约束 | ✗ | 部分（陀螺仪限位） | ✓ 医学数据 |
| ROM 软边界 + 渐变惩罚 | ✗ | ✗ | ✓ |
| ROM 约束反哺传感器可信度 | ✗ | ✗ | ✓ |
| 约束数据来源 | 用户测量 | 商业闭源模型 | 公开医学文献 |

---

## 6. 实现路线

### Phase 1: 静态 ROM 约束
- 提取每帧关节角
- 超限则投影回 ROM 边界
- 验证：IMU 数据 + 故意超限 → 约束应能纠正

### Phase 2: 软边界
- ROM 的 80%–100% 区间施加渐变惩罚
- 惩罚梯度进入 EKF 观测更新
- 目的：让系统"知道自己在接近极限"而非撞墙后被拉回

### Phase 3: 约束反馈
- ROM 违反 → 增大本节点 Q
- ROM 违反 → 降低对邻居的约束传递权重
- 目的：异常姿态不污染全局估计

### Phase 4: 个人 ROM 标定
- 全关节 ROM 采集流程
- 用户特异性 ROM 替换标准值
- 存储为个人模型文件

---

## 参考

- AAOS: *Joint Motion: Method of Measuring and Recording*, 1965
- ISB: *Recommendations for Standardization in the Reporting of Kinematic Data*, 2002, 2005
- GB/T 10000-1988: 中国成年人人体尺寸
- [SO(3)-EKF 与运动链约束的全身姿态解算](../../SO(3)-EKF%20与运动链约束的全身姿态解算.md) — 本文的 ROMM 约束嵌入该框架的约束层
