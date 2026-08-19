---
type: source
tags: [embedded, logging, debug, library]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - EasyLogger 源码分析

> **仓库**: https://github.com/armink/EasyLogger
> **本地**: `_llm/raw/repos/EasyLogger/`
> **协议**: MIT | **版本**: 2.2.99 | **作者**: armink (朱天龙)

## 概述

EasyLogger 是超轻量嵌入式日志库，ROM <1.6KB，RAM <0.3KB。支持六级日志、标签过滤、ANSI 彩色输出、异步环形缓冲、文件/Flash 后端插件。

## 核心设计

1. **单例架构**：全局 `EasyLogger` 结构体持有所有状态
2. **三级输出模式**：直接同步 → 缓冲批量 → 异步环形缓冲+消费线程
3. **六级过滤**：全局级别 + 全局标签子串 + 关键词 + 每标签级别覆盖
4. **干净移植接口**：仅需实现 6 个 `elog_port_*` 函数

## 输出模式

| 模式 | 宏 | 特点 |
|------|-----|------|
| 直接 | (默认) | 立即调用 `elog_port_output`，阻塞 |
| 缓冲 | `ELOG_BUF_OUTPUT_ENABLE` | 攒满一批再输出，适合 DMA |
| 异步 | `ELOG_ASYNC_OUTPUT_ENABLE` | 环形缓冲+消费任务，非阻塞 |

## 已产出的知识页

- [EasyLogger/1. EasyLogger 概述与架构](../知识/嵌入式软件/EasyLogger/1.%20EasyLogger%20概述与架构.md)
- [EasyLogger/2. EasyLogger 移植与输出模式](../知识/嵌入式软件/EasyLogger/2.%20EasyLogger%20移植与输出模式.md)
