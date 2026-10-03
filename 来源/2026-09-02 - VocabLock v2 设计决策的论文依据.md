---
type: source
tags: [研究方法, 语言学习, spaced-repetition, AI教育, 设计决策]
created: 2026-09-02
updated: 2026-09-02
---

# 2026-09-02 - VocabLock v2 设计决策的论文依据

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | VocabLock v2 设计决策的论文依据 |
| 日期 | 2026-09-02 |
| 来源 | 项目侧设计备忘录（VocabLock v2：Go 后端 + Flutter 桌面，AI 出题 + SM-2 调度 + 字典 gate + 插件协议） |
| 格式 | Markdown 设计文档 |
| 内容性质 | 为既有设计决策补充可追溯的文献支撑，非系统设计文档本身 |

## 文档概述

本文档按六大设计主题整理文献依据：SM-2 调度、字典 gate（grounded generation 硬约束）、ITS 有效性、测试效应、"研究-后考"顺序、发音评分（GOP 谱系）。每条引文标注：**✅ 已核实**（scholar/crossref 检索到元数据）；**📖 经典**（教科书级引用，未逐条检索）。

## 引文清单

| # | 主题 | 引文 | 关键结论 |
|---|------|------|---------|
| 1 | 间隔重复 | ✅ Cepeda, Pashler, Vul, Wixted & Rohrer (2006) *Distributed practice in verbal recall tasks: A review and quantitative synthesis*. Psychological Bulletin 132(3):354-380. DOI 10.1037/0033-2909.132.3.354 | 839 项评估 / 317 实验的元分析；最优 ISI 约为保留间隔的 10-20%，且随保留间隔增长 |
| 2 | 调度模型 | ✅ Ye, Su & Cao (2022) *A Stochastic Shortest Path Algorithm for Optimizing Spaced Repetition Scheduling* (FSRS). KDD 2022. DOI 10.1145/3534678.3539081 | 调度建模为随机最短路问题（记忆稳定性/难度二参数），优于 SM-2 类启发式 |
| 3 | 词汇可学习性 | ✅ Settles & Meeder (2016) *A Trainable Spaced Repetition Model for Language Learning* (HLR). ACL 2016. DOI 10.18653/v1/p16-1174 | HLR 从大规模日志学习词汇"半衰期"，召回率预测误差比基线降低 45%+ |
| 4 | 教育学综述 | 📖 Kang (2016) *Spaced repetition promotes efficient and effective learning*. Policy Insights from Behavioral and Brain Sciences | 政策级综述：间隔重复是证据最强的学习技术之一 |
| 5 | 有源生成 | 📖 Lewis et al. (2020) *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS 2020 | 检索作为生成的事实锚点，抑制幻觉式编造 |
| 6 | 词汇门槛 | ✅ Nation (2006/2020) *How large a vocabulary is needed for reading and listening?*. DOI 10.26686/wgtn.12552221 | 无辅助理解需 98% 覆盖 → 英语 8,000-9,000 词族；按频级阶梯推进有量化依据 |
| 7 | 辅导差距 | 📖 Bloom (1984) *The 2-Sigma Problem*. Educational Researcher 13(6):4-16 | 一对一辅导比班级教学高 2σ；ITS 的目标是逼近这个差距 |
| 8 | ITS 元分析 | ✅ Kulik & Fletcher (2016) *Effectiveness of Intelligent Tutoring Systems*. Review of Educational Research 86(1). DOI 10.3102/0034654315581420 | 50 项对照研究中位效应 +0.66σ（50 分位 → 75 分位） |
| 9 | ITS 综述 | ✅ Létourneau et al. (2025) *A systematic review of AI-driven intelligent tutoring systems (ITS) in K-12 education*. npj Science of Learning. DOI 10.1038/s41539-025-00320-7 | 28 项研究综述：ITS 有正效应但实验设计质量参差，自证效果需要评测设计 |
| 10 | 测试效应 | ✅ Roediger & Karpicke (2006) *Test-enhanced learning: Taking memory tests improves long-term retention*. Psychological Science 17(3). DOI 10.1111/j.1467-9280.2006.01693.x | 测试（含无反馈）比等时重读的长期保留显著更优 |
| 11 | 反馈时机 | 📖 Kulik & Kulik (1988) 反馈时机元分析 | 即时反馈在保持测试中优于延迟反馈 |
| 12 | 预测试效应 | ✅ Kornell, Hays & Bjork (2009) *Unsuccessful retrieval attempts enhance subsequent learning*. JEP:LMC 35(4). DOI 10.1037/a0015729; ✅ Richland, Kornell & Kao (2009) *The pretesting effect*. JEP:Applied 15(3). DOI 10.1037/a0016496 | 失败检索本身加强后续学习；先测后学通常优于先学后测 |
| 13 | 发音评分 | ✅ Witt & Young (2000) *Phone-level pronunciation scoring and assessment for interactive language learning*. Speech Communication. DOI 10.1016/s0167-6393(99)00044-8 | GOP（音素级似然比评分）是自动发音评测的经典框架；后续 DNN 化见 ✅ Huang et al. (2017) JASA, DOI 10.1121/1.5011159 |

## 关键知识点

- 间隔重复的时机（**分布练习元分析**）：最优 ISI 是保留间隔的函数 — 调度的"参数"不是拍脑袋，而是有元分析支撑的定量规律
- **两种调度模型谱系**：SM-2 固定 EF 启发式（可解释 `1, 6, EF×…`）→ 数据驱动模型（FSRS 二参数随机最短路、HLR 词汇半衰期回归）在日志规模上更优；SM-2 是可解释基线而非终点
- **字典 gate = RAG 的硬约束版**：普通 RAG 是检索增强生成（retrieval 引导 generation）；gate 是"检索必须成功才生成入库"，把防幻觉从概率引到结构
- **有源生成 vs 无源生成**：来源越可控，幻觉越可控 — 零 AI 降级（exact 判题）保证双挂时学习闭环不破
- **ITS 效应量有据可查**：Bloom 2σ 是上限（一对一），Kulik & Fletcher 元分析中位 +0.66σ 是可达效果；但 2025 年综述显示实验设计质量参差，**评测设计本身是产品责任**
- **测试增强学习**：测试本身（含无反馈）优于重读；失败检索（pretesting）对后续学习也有促进 — "先测后学"优于"先学后测"
- **Pron 评分谱系**：GOP（Witt & Young 2000）→ DNN 迁移学习（Huang 2017）→ 现代流式端到端 ASR 的旁支

## 创建的 Wiki 页面

- [语言学习/间隔重复与记忆调度](../知识/语言学习/间隔重复与记忆调度.md) — 间隔重复元分析 · SM-2 · FSRS · HLR · 调度模型设计约束
- [语言学习/词汇量门槛与分级学习](../知识/语言学习/词汇量门槛与分级学习.md) — Nation 98% 覆盖率 · 8,000-9,000 词族 · 频级阶梯
- [LLM-AI/检索增强生成](../知识/LLM-AI/检索增强生成.md) — grounded generation · 词典 gate(RAG 硬约束变体) · 与 Wiki 方法论的对比
- [语言学习/智能导师系统 ITS](../知识/语言学习/智能导师系统%20ITS.md) — Bloom 2σ · ITS 效应量 · 评测设计责任
- [语言学习/测试增强学习](../知识/语言学习/测试增强学习.md) — 测试效应 · 预测试效应 · 反馈时机
- [语言学习/发音评分 GOP](../知识/语言学习/发音评分%20GOP.md) — GOP 框架 · DNN 化 · 发音评测谱系
