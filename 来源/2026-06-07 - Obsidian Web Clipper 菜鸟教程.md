---
type: source
tags: [clippings, obsidian, tool]
created: 2026-06-07
updated: 2026-06-07
source: "https://www.runoob.com/obsidian/obsidian-web-clipper.html"
site: 菜鸟教程
---

# Obsidian Web Clipper — 菜鸟教程

> [!info] 来源
> 来自：菜鸟教程 | [原文链接](https://www.runoob.com/obsidian/obsidian-web-clipper.html)

## 核心内容

[Obsidian Web Clipper](../元件/其他/Obsidian%20Web%20Clipper.md) 是 Obsidian 官方出品的免费开源浏览器扩展，可将任何网页一键保存为本地 Markdown 笔记。

### 两种剪藏模式

- **整页剪藏** — 自动提取正文（去广告/侧边栏）、标题、作者、日期、URL、代码块、LaTeX 公式
- **高亮剪藏** — 选中感兴趣段落，以引用块形式保存；可多次高亮后一次性提交

### 模板系统（最强功能）

支持针对不同网站类型定制保存结构：

| 模板类型 | 适用场景 |
|---------|---------|
| 文章模板 | 博客、新闻 |
| 食谱模板 | 食材列表转为待办复选框 |
| 电影/书籍模板 | 配合 Dataview 制作观影数据库 |
| 学术论文模板 | 保留代码块和 LaTeX 公式 |

### 模板变量

支持预设变量（`{{title}}`、`{{url}}`、`{{author}}` 等）、Meta 变量、CSS 选择器变量、Schema.org 变量、AI Prompt 变量（需启用 Interpreter）。

### 隐私优势

所有内容存本地 `.md` 文件，无服务器上传，可导出 JSON 迁移，无锁定风险。

### 界面一览

![runoob_1780454126644.png](../raw/assets/runoob_1780454126644.png)

## 与 LLM Wiki 的关联

Web Clipper 是[LLM Wiki方法论](../知识/LLM-AI/LLM%20Wiki方法论.md)中推荐的获取资料到 `raw/sources/` 的核心工具。通过模板自动触发功能，可以针对不同网站（如 arxiv、YouTube）使用不同的保存格式，大幅简化资料收集流程。

## See Also

- [Obsidian Web Clipper](../元件/其他/Obsidian%20Web%20Clipper.md) — 实体页
- [LLM Wiki方法论](../知识/LLM-AI/LLM%20Wiki方法论.md) — 方法论模式
- [LLM Wiki](../LLM%20Wiki.md) — 原始文章
