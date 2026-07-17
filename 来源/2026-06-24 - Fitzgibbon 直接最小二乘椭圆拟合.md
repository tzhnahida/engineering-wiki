---
type: source
tags: [computer-vision, ellipse-fitting, least-squares, eigenvalue]
created: 2026-06-24
updated: 2026-06-30
---

# 2026-06-24 - Fitzgibbon 直接最小二乘椭圆拟合

> **论文**: Fitzgibbon, Pilu & Fisher (1999) — *Direct Least Square Fitting of Ellipses*, IEEE TPAMI
> **存放**: [参考/论文/Fitzgibbon 1999 - Direct Least Square Fitting of Ellipses.pdf](../参考/论文/Fitzgibbon 1999 - Direct Least Square Fitting of Ellipses.pdf.md)

## 概述

提出直接最小二乘椭圆拟合（DLSF）算法。通过将椭圆约束 $4ac - b^2 = 1$ 引入广义特征值问题，保证拟合结果始终为椭圆（不会退化为双曲线或抛物线）。是计算机视觉中椭圆检测的标准算法。

## 关键贡献

- **约束形式化**：$4ac - b^2 = 1$ 将椭圆拟合转化为广义特征值问题
- **唯一解保证**：证明了椭圆的特征值解的存在性和唯一性
- **避免虚假解**：无需迭代筛选，直接输出椭圆

## 关联知识

- [直接最小二乘椭圆拟合](../知识/计算机视觉/直接最小二乘椭圆拟合.md) — 知识页
- [双目标定](../知识/计算机视觉/双目标定.md) — 相机标定流程
