---
type: source
tags: [embedded, button, gpio, input, library]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - MultiButton 源码分析

> **仓库**: https://github.com/0x1abin/MultiButton
> **本地**: `_llm/raw/repos/MultiButton/`
> **协议**: MIT | **版本**: 1.1.1

## 概述

MultiButton 是嵌入式按键驱动库，一个 `multi_button.c`（322行）+ `multi_button.h`（117行）。事件驱动的状态机、非阻塞设计。

## 核心设计

- **5 态状态机**：IDLE → PRESS → RELEASE → REPEAT → LONG_HOLD
- **7 种事件**：PRESS_DOWN/UP, PRESS_REPEAT, SINGLE_CLICK, DOUBLE_CLICK, LONG_PRESS_START, LONG_PRESS_HOLD
- **数字消抖**：连续 DEBOUNCE_TICKS(默认3)次一致才接受电平变化
- **5ms 滴答驱动**：`button_ticks()` 每次调用推进所有按钮状态机
- **位域优化**：每个按钮状态字段仅占 16 bits

## 可配置参数

| 参数 | 默认 | 含义 |
|------|------|------|
| TICKS_INTERVAL | 5ms | 滴答周期 |
| DEBOUNCE_TICKS | 3 | 消抖采样次数 |
| SHORT_TICKS | 300ms | 短按超时（区分单击/长按） |
| LONG_TICKS | 1000ms | 长按触发阈值 |

## 已产出的知识页

- [MultiButton/1. MultiButton 状态机与事件系统](../知识/嵌入式软件/MultiButton/1.%20MultiButton%20状态机与事件系统.md)
