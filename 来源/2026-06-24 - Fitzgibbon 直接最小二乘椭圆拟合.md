---
type: source
tags: [computer-vision, ellipse-fitting, least-squares, eigenvalue]
created: 2026-06-24
updated: 2026-06-30
---

# 2026-06-24 - Fitzgibbon 直接最小二乘椭圆拟合

> **论文**: Fitzgibbon, Pilu & Fisher (1999) — *Direct Least Square Fitting of Ellipses*, IEEE TPAMI
> **存放**: [[参考/论文/Fitzgibbon 1999 - Direct Least Square Fitting of Ellipses.pdf]]

## 概述

提出直接最小二乘椭圆拟合（DLSF）算法。通过将椭圆约束 $4ac - b^2 = 1$ 引入广义特征值问题，保证拟合结果始终为椭圆（不会退化为双曲线或抛物线）。是计算机视觉中椭圆检测的标准算法。

## 关键贡献

- **约束形式化**：$4ac - b^2 = 1$ 将椭圆拟合转化为广义特征值问题
- **唯一解保证**：证明了椭圆的特征值解的存在性和唯一性
- **避免虚假解**：无需迭代筛选，直接输出椭圆

## 关联知识

- [直接最小二乘椭圆拟合](../%E7%9F%A5%E8%AF%86/%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%A7%86%E8%A7%89/%E7%9B%B4%E6%8E%A5%E6%9C%80%E5%B0%8F%E4%BA%8C%E4%B9%98%E6%A4%AD%E5%9C%86%E6%8B%9F%E5%90%88.md) — 知识页
- [双目标定](../%E7%9F%A5%E8%AF%86/%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%A7%86%E8%A7%89/%E5%8F%8C%E7%9B%AE%E6%A0%87%E5%AE%9A.md) — 相机标定流程
