---
type: source
tags: [computer-vision, face-mesh, deep-learning, mediapipe]
created: 2026-06-24
updated: 2026-06-30
---

# 2026-06-24 - Kartynnik MediaPipe Face Mesh

> **论文**: Kartynnik et al. (2019) — *Real-time Facial Surface Geometry from Monocular Video on Mobile GPUs*, CVPR Workshop
> **存放**: [[参考/论文/Kartynnik 2019 - Real-time Facial Surface Geometry from Monocular Video on Mobile GPUs.pdf]]

## 概述

MediaPipe Face Mesh：从单目视频实时估计 468 个 3D 面部顶点。采用 BlazeFace 轻量检测器 + 定制 ResNet 回归器，在 iPhone 上仅需 2.5ms 推理。无需深度传感器，纯 RGB 输入。IOD（瞳孔间距）MAD 误差 3.96%。

## 关键贡献

- 468 个 3D 顶点的稠密面部网格
- BlazeFace：轻量 GPU 友好的人脸检测器
- 2.5ms 推理（iPhone），适合实时应用
- 1 Euro Filter 时序平滑

## 关联

- [面部关键点检测](../%E7%9F%A5%E8%AF%86/%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%A7%86%E8%A7%89/%E9%9D%A2%E9%83%A8%E5%85%B3%E9%94%AE%E7%82%B9%E6%A3%80%E6%B5%8B.md) — 知识页（含 Attention Mesh 扩展）
- Grishchenko 2020 — Attention Mesh（精度改进）
