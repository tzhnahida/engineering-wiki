---
type: concept
tags: [language-learning, ITS, intelligent-tutoring, education-effectiveness, effect-size]
created: 2026-09-02
updated: 2026-09-02
sources: ["[2026-09-02 - VocabLock v2 设计决策的论文依据](../../来源/2026-09-02%20-%20VocabLock%20v2%20设计决策的论文依据.md)"]
---

# 智能导师系统(ITS)

> 核心问题:用 AI 当"一对一老师"替代真人教学,证据上能达到什么水平?

## Bloom 的 2σ 差距

📖 Bloom (1984) *The 2-Sigma Problem* (Educational Researcher 13(6):4-16):

- 一对一辅导的学生成绩比班级教学高 **2 个标准差(2σ)**
- 这是 ITS 的**理论上限参照**:ITS 的全部意义在于"逼近一对一辅导",而非超越

## 可测量效果:元分析

✅ Kulik & Fletcher (2016) *Effectiveness of Intelligent Tutoring Systems* (Review of Educational Research 86(1), DOI 10.3102/0034654315581420):

- 50 项对照研究的中位效应 **+0.66σ**(即 50 分位学员提升到 75 分位)
- 量级:仅为 2σ 上限的三分之一 — 说明 ITS 有效,但离"完全替代人类一对一"仍有距离

## 可靠性警示:综述揭示评测缺陷

✅ Létourneau et al. (2025) *A systematic review of AI-driven ITS in K-12 education* (npj Science of Learning, DOI 10.1038/s41539-025-00320-7):

- 28 项研究综述:ITS 总体上正效应,但**实验设计质量参差**
- 含义:单个 ITS 声称的"效果"不可直接采信;**自证效果依赖严谨评测设计** — 对照组、效应量、事前/事后测、样本量等是产品级责任

> [!important] 设计含义
> ITS 有效性 = 已知效应的工程化复现(0.66σ 量级可预期),不是未验证的创新。评测设计缺失会掩盖无效甚至有害的 AI 教学;任何"AI 出题/判题/反馈"都应以对照实验标准自审。

## 与 [测试增强学习](测试增强学习.md) 的关系

ITS 的有效性来源之一正是测试增强学习 — ITS 通过"出题-反馈"闭环复现测试效应,效应量证据与 ITS 证据互相支撑。
