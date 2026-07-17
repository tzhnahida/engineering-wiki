---
type: entity
tags: [tool, obsidian, clipper, browser-extension]
created: 2026-06-07
updated: 2026-06-07
sources:
---
# Obsidian Web Clipper
  - [[2026-06-07 - Obsidian Web Clipper 菜鸟教程]]
# Obsidian Web Clipper

Obsidian 官方出品的免费开源浏览器扩展，可将任何网页一键保存为本地 Markdown 笔记。

## 基本信息

- **开发方**: Obsidian 官方
- **许可证**: 免费、开源、无广告
- **支持平台**: Chrome / Brave / Arc / Vivaldi / Firefox / Safari (iOS + macOS) / Edge
- **核心用途**: 网页内容剪藏 → 本地 Markdown 文件

## 主要特性

### 剪藏模式
- **整页剪藏** — 自动提取正文、元数据、代码块
- **高亮剪藏** — 选中段落保存为引用块，支持多次高亮后一次性提交

### 模板系统
支持按网站 URL 自动触发不同模板（如 youtube.com → 视频模板，arxiv.org → 论文模板），模板可自定义笔记名称、保存位置、YAML frontmatter、正文格式。

### 变量体系
- 预设变量（title, url, author, date 等）
- CSS 选择器变量（精准提取特定元素）
- Schema.org 变量（提取结构化数据）
- AI Prompt 变量（调用 AI 做摘要/翻译）

## 与 LLM Wiki 的关系

Web Clipper 是[[LLM Wiki方法论]]中推荐的关键工具，用于快速将网页资料送入 `raw/sources/` 目录，启动 Ingest 流程。

## 参见

- [[2026-06-07 - Obsidian Web Clipper 菜鸟教程]] — 来源资料
- [[LLM Wiki方法论]] — 方法论模式
