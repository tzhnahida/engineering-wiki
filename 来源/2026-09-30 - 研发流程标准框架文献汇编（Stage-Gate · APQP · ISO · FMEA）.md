---
type: source
tags: [研发管理, Stage-Gate, APQP, PPAP, IATF16949, ISO13485, FMEA, IEC60812, 标准, 文献引用]
created: 2026-09-30
updated: 2026-09-30
---

# 2026-09-30 - 研发流程标准框架文献汇编（Stage-Gate · APQP · ISO · FMEA）

## 元数据

| 属性 | 值 |
|------|-----|
| 文档类型 | **文献引用汇编**（非单一文档） |
| 采集方式 | 网络检索（WebSearch）+ `curl` 探测 |
| 原始文本 | ❌ **未入库** —— 见下方「采集限制」 |
| 内容性质 | 标准 / 学术论文的书目信息 |
| 数据密度 | 仅书目级 |

> [!danger] 采集限制 —— 本页与其他来源页性质不同，使用前必须读这段
>
> **1. 全文一律未取回。** 本次采集实测：
> - `iso.org` / `asq.org` / `aiag.org` → **HTTP 403**（标准机构全挡）
> - Wiley（Cooper 论文）→ 付费墙
> - `citeseerx` → 301 跳转后无内容
>
> **2. 标准原文有版权，本就不应整篇存档。** ISO / IEC / AIAG 的标准是版权文本，不能复制进 `_llm/raw/` 再公开到 `_wiki/`。本页因此只记**书目信息**（标准号 / 条款号 / 卷期 / DOI）——这是行业惯例的引用方式。
>
> **3. 下方「内容要点」来自搜索引擎摘要，不是我自己读的原标准。** 按防幻觉规则，**全部标 B 级**，引用前须回原文核对。

## 一、Stage-Gate（阶段门）

| 项 | 值 |
|----|-----|
| 文献 | Cooper, R. G. (2008). *Perspective: The Stage-Gate® Idea-to-Launch Process—Update, What's New, and NexGen Systems* |
| 期刊 | Journal of Product Innovation Management |
| 卷期页 | **25(3), 213–232** |
| DOI | `10.1111/j.1540-5885.2008.00296.x` |
| ISSN | 0737-6782 |
| 作者单位 | McMaster University |
| 商标 | Stage-Gate® 是 Product Development Institute Inc. 的注册商标 |

**内容要点（B 级 — 来自检索摘要，未核对原文）：**

- 阶段（stage）= 跨职能团队做信息收集的工作段；门（gate）= Go/Kill 决策点
- 论文的主要目的是**纠正误解**：Stage-Gate 不是线性流程、不是刚性体系、不是职能式分阶段评审
- 提出「**gates with teeth**」（有牙齿的门）——明确 gatekeeper 及其议事规则
- 治理问题是主要失败模式：过度官僚化、把六西格玛/精益的成本削减逻辑误用到产品创新上
- NexGen 版本强调：**可伸缩**（适应不同规模项目）、**螺旋开发**、**并行执行**
- 内置问责机制：严格的产品上市后评审

> ⚠️ 以上为检索摘要转述，**B 级**。引用时应回 JPIM 原文。

## 二、APQP / PPAP（汽车业核心工具）

| 项 | 值 |
|----|-----|
| 发布机构 | **AIAG**（Automotive Industry Action Group，美国） |
| APQP 手册 | *Advanced Product Quality Planning and Control Plan*，**第 3 版 2024-03-01 生效** |
| PPAP 手册 | *Production Part Approval Process*，首版 1993 |
| 上位标准 | **IATF 16949**（将 AIAG 核心工具列为参考手册） |

**APQP 五阶段（B 级 — 检索摘要）：**

| 阶段 | 名称 | 关键产出 |
|------|------|---------|
| 1 | Plan and Define Program | 可靠性/质量目标、初版特殊特性清单、初版过程流程图 |
| 2 | Product Design and Development | **DFMEA**、DFM/DFA、设计验证、原型控制计划；（PPAP 文档在此阶段开始准备） |
| 3 | Process Design and Development | **PFMEA**、试生产控制计划、MSA 计划、过程能力研究计划 |
| 4 | Product and Process Validation | 试生产、MSA、SPC、生产控制计划、**Run-at-Rate**、**PPAP 提交** |
| 5 | Feedback, Assessment, Corrective Actions | 用生产数据与客户反馈持续改进 |

**PPAP 要点（B 级）：**
- 目的：确认供方**理解**客户工程设计与规格，且其过程在**实际生产节拍下**有能力持续满足要求
- 试生产要求：**连续 300 件**以上、**1–8 生产小时**，使用**生产用工装/过程/设备/材料**，在标称节拍下（run at rate）
- 默认提交等级 **Level 3**
- 审批状态三档：**APPROVED / INTERIM APPROVAL / REJECTED**
- 记录保留：零件在役期 + 1 个日历年
- 触发重新提交的情形：新零件、纠正不合格、设计/工艺/材料变更、**生产地点变更、停产超过 12 个月**

> IATF 16949 与 APQP 的挂接点：APQP 支撑 **IATF 16949 §8.3.2.1（设计与开发策划）**。

## 三、ISO 9001 / ISO 13485 设计条款

| 标准 | 条款 | 名称 |
|------|------|------|
| ISO 9001:2015 | **8.3** | Design and development of products and services |
| ISO 13485:2016 | **7.3** | Design and development（子条款 7.3.1–7.3.10） |

- ISO 13485 §7.3 的**子条款号与条款名**已在 [2026-09-30 - 设计控制与追溯（MedDeviceGuide）](2026-09-30%20-%20设计控制与追溯（MedDeviceGuide）.md) 逐条核对（A 级）
- ISO 9001 §8.3 的**覆盖范围**已在 [2026-09-30 - ISO 9001 条款 8.3（Connect981）](2026-09-30%20-%20ISO%209001%20条款%208.3（Connect981）.md) 逐条核对（A 级）
- ✅ **已补齐**：ISO 9001 §8.3.1–8.3.6 的子条款号、条目数与各条目主题，见 [2026-09-30 - ISO 9001 设计开发子条款 8.3.1-8.3.6](2026-09-30%20-%20ISO%209001%20设计开发子条款%208.3.1-8.3.6.md)（来源为 ISO/TS 9002 官方应用指南 + Oxebridge 英文解读，**A 级**）

## 四、FMEA

| 项 | 值 |
|----|-----|
| 国际标准 | **IEC 60812:2018** *Failure modes and effects analysis (FMEA and FMECA)*（首版 1985） |
| 汽车业手册 | **AIAG & VDA FMEA Handbook**，2019-06 第 1 版 |
| 取代 | AIAG FMEA 第 4 版（2008）、VDA4（2012） |
| 相关 | SAE J1739:2021；ISO 26262（功能安全） |
| 中国对应 | GB/T 7826（新稿 20256218-T-339，2025-10 公开征求意见） |

**要点（B 级 — 检索摘要）：**
- AIAG-VDA 手册将 FMEA 分为 **DFMEA**（设计）/ **PFMEA**（过程）/ **FMEA-MSR**（监测与系统响应，机电产品用）
- 采用**七步法**：策划与准备（5T：Team/InTent/Time/Tasks/Tools）→ 结构分析 → 功能分析 → 失效分析 → 风险分析 → 优化 → 结果文件化
- 关键变化：**用 AP（Action Priority，高/中/低）取代 RPN**（严重度×发生率×探测度）——因为不同 S/O/D 组合可能算出相同 RPN 而实际风险不同
- 手册是**参考手册，非强制标准**；除非客户特定要求（CSR）或企业自选，不强制采用
- 起源：1949 年美国军用标准 **MIL-P-1629**

## 采集完整度

| 框架 | 书目信息 | 全文 | 内容要点 | 等级 |
|------|---------|------|---------|------|
| Stage-Gate (Cooper 2008) | ✅ | ❌ 付费墙 | 检索摘要 | B |
| APQP / PPAP (AIAG) | ✅ | ❌ 403 | 检索摘要 | B |
| ISO 9001 §8.3 | ✅ | ❌ 版权 | 另页核对 | A（见 Connect981 页） |
| ISO 9001 §8.3.1–8.3.6 | ✅ | ❌ 版权 | **另页核对** | A（见「ISO 9001 设计开发子条款」页） |
| ISO 13485 §7.3 | ✅ | ❌ 版权 | 另页核对 | A（见 MedDeviceGuide 页） |
| IEC 60812 / AIAG-VDA FMEA | ✅ | ❌ 403 | 检索摘要 | B |

⚠️ **B 级条目在用于设计决策前，必须回标准/论文原文核对。**
