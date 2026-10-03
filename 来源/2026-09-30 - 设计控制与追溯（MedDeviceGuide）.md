---
type: source
tags: [研发管理, 设计控制, ISO13485, FDA, 21CFR820, 追溯矩阵, DHF, 医疗器械, 风险管理]
created: 2026-09-30
updated: 2026-09-30
---

# 2026-09-30 - 设计控制与追溯（MedDeviceGuide）

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | Design Controls for Medical Devices: FDA Requirements, Process, and Implementation Guide |
| 作者 | Ran Chen（页面自述 Global MedTech Expert） |
| 发布渠道 | meddeviceguide.com |
| URL | https://meddeviceguide.com/blog/design-controls-medical-devices-guide |
| 发布日期 | 2026-03-25（页面标注 Published / Last reviewed） |
| 原始文本 | `_llm/raw/2026-09-30-企业研发流程-设计控制与追溯-MedDeviceGuide.md`（101,586 字节） |
| 内容性质 | 行业解读文章（非标准原文），引用 FDA 法规与 ISO 条款 |
| 数据密度 | 高——含条款级对照表、追溯矩阵结构、DHF/DMR/DHR 区分 |

> [!warning] 这是**二手解读**，不是标准原文
> 本页可作为「条款号 ↔ 条款名」的索引，但**不构成对 ISO 13485 / 21 CFR 820 条文的权威引用**。
> ISO 13485 与 21 CFR 820 为版权文本，未入库、不可整篇存档。引用时应回标准原文核对。

## 核心论点（A 级 — 原文逐段核对）

### 三套法规的关系

| 体系 | 载体 | 术语 |
|------|------|------|
| 美国 | FDA **21 CFR 820.30** | "design controls" |
| 国际 | **ISO 13485:2016 clause 7.3** | "design and development" |
| 欧盟 | EU MDR (Regulation 2017/745) | 无独立条款；嵌在 Annex II 技术文档要求中 |

原文结论：

> "ISO 13485 clause 7.3 is the global lingua franca for design controls."

即：以 ISO 13485 §7.3 为基线建体系，再叠加 FDA 特有项（DHF / DMR / DHR / UDI）。

### ISO 13485:2016 §7.3 条款结构（原文逐条抄录）

| 条款 | 名称 |
|------|------|
| 7.3.1 | General |
| 7.3.2 | Design and development planning |
| 7.3.3 | Design and development inputs |
| 7.3.4 | Design and development outputs |
| 7.3.5 | Design and development review |
| 7.3.6 | Design and development verification |
| 7.3.7 | Design and development validation |
| **7.3.8** | **Design and development transfer** |
| 7.3.9 | Control of design and development changes |
| 7.3.10 | Design and development files |

> 原文强调 §7.3.8 设计转移是 **ISO 9001 没有对应条款**的一节。

### 风险管理的接入点

- §7.3.3 明确要求**风险管理输出作为设计输入**，引用 **ISO 14971**
- 这是 ISO 13485 相对 21 CFR 820.30 更明确强调的部分

### 追溯矩阵

原文单列一节 `## The Traceability Matrix`，要点：

- 追溯矩阵用于串起**需求 → 设计输出 → 验证/验证证据**
- 追溯链的缺口是**最常见的审计发现**（原文："traceability is a defining feature... gaps in this chain are the most common audit finding"）
- ISO 13485 要求设计文件（§7.3.10）中，每一条输入可追溯到输出与验证证据

### 三份记录的区别

| 记录 | 含义 |
|------|------|
| **DHF** | Design History File —— 证明设计符合设计计划的记录（对应 21 CFR 820.30(j)） |
| **DMR** | Device Master Record —— 器械主记录 |
| **DHR** | Device History Record —— 器械历史记录 |

## 提取完整性

| 类别 | 请求项 | 已提取 | 带节位溯源 | 存疑 |
|------|--------|--------|-----------|------|
| ISO 13485 条款号 | 10 | 10 | 10 | 0 |
| 法规体系对照 | 3 | 3 | 3 | 0 |
| 条款内容详述 | 10 | 3 | 3 | 7 — ⚠️ 本文只给出条款名，未展开每条要求；需回标准原文 |
| 21 CFR 820.30 子条 | 1 | 1 | 1 | 0 — 仅 820.30(j) |
| 追溯矩阵结构 | 1 | 1 | 1 | 0 |

⚠️ **未找到**：ISO 13485 各条款的**详细要求文本**（本文仅给条款名）。如需条款内容，须另找来源。
