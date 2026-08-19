---
type: source
tags: [embedded, shell, debug, cli, library]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - letter-shell 源码分析

> **仓库**: https://github.com/NevermindZZT/letter-shell
> **本地**: `_llm/raw/repos/letter-shell/`
> **协议**: MIT | **版本**: 3.2.4

## 概述

letter-shell 是嵌入式串口命令行 shell，C99 实现。支持 Tab 补全、命令历史、变量导出、用户认证、函数签名类型检查。通过链接器段魔法自动收集注册的命令。

## 核心设计

1. **链接器段自动注册**：`SHELL_EXPORT_CMD` 宏将命令放入 `.shellCommand` 段，`shellInit()` 时自动发现
2. **命令分发**：线性搜索命令表 → `shellExec()` 空格分割参数 → `shellRunCommand()` 分发到 main 风格或 func 风格
3. **按键匹配**：`shellHandler()` 每字符检测多字节转义序列（方向键/Tab/回车等）
4. **尾行模式**：日志输出不破坏正在键入的命令行

## 关键特性

- **Tab 补全**：前缀匹配，多匹配时显示候选并截断到最长公共前缀
- **命令历史**：循环缓冲区（默认 5 条），上下箭头导航
- **变量导出**：`SHELL_EXPORT_VAR` 将 C 变量暴露为可读写 shell 变量（PID 调参利器）
- **函数签名**：声明参数类型（`"iis"`=两个整数一个字符串），shell 执行前校验
- **用户认证**：多用户，密码保护，权限位掩码

## 已产出的知识页

- [letter-shell/1. letter-shell 概述与命令系统](../知识/嵌入式软件/letter-shell/1.%20letter-shell%20概述与命令系统.md)
