---
type: source
tags: [嵌入式, RTOS, FreeRTOS, 源码分析, 内核]
created: 2026-07-24
updated: 2026-07-24
source_url: https://github.com/FreeRTOS/FreeRTOS-Kernel
version: V10.4.6
format: source-code
---

# FreeRTOS Kernel V10.4.6 源码分析

> 来源：GitHub [FreeRTOS/FreeRTOS-Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel)，Tag V10.4.6
> 本地路径：`_llm/raw/freertos-kernel/`

## 概述

对 FreeRTOS 内核源码进行 B 级（逐函数精读）分析，覆盖所有核心子系统。此前知识库中 FreeRTOS 页面均来源一篇微信公众号概述文章，本次分析将每个子系统升级为基于源码实现的深度分析。

## 分析范围

| 源文件 | 行数 | 分析产出 | 知识页面 |
|--------|------|----------|----------|
| `list.c` + `list.h` | ~300 | 循环双链表 + 哨兵机制 | [[FreeRTOS 链表实现]] |
| `tasks.c` + `task.h` | ~5443 | TCB、调度器、状态机、上下文切换 | [[FreeRTOS 任务管理与调度]] |
| `queue.c` + `queue.h` + `semphr.h` | ~3100 | 队列环形缓冲区、信号量、互斥锁、优先级继承、双锁机制 | [[FreeRTOS 队列管理]]、[[FreeRTOS 信号量与互斥锁]] |
| `timers.c` + `timers.h` | ~2500 | 命令队列协议、守护任务、双列表溢出处理 | [[FreeRTOS 软件定时器]] |
| `event_groups.c` | ~800 | 位掩码同步、无序事件链表 | [[FreeRTOS 事件组]] |
| `stream_buffer.c` + `message_buffer.h` | ~1200 | 字节流、消息边界、任务通知阻塞 | [[FreeRTOS 流缓冲区与消息缓冲区]] |
| `heap_1~5.c` | ~2000 | 五种分配策略、碎片化对比 | [[FreeRTOS 内存管理]] |
| `port.c` + `portmacro.h` (ARM_CM3/CM4F) | ~500 | BASEPRI 临界区、PendSV 汇编、栈帧初始化 | [[FreeRTOS 中断管理]] |
| `tasks.c` (通知部分) | ~500 | TCB 内嵌通知值、零对象开销 IPC | [[FreeRTOS 任务通知]] |

## 关键架构发现

1. **队列是统一内核对象** — `queue.c` 通过 `ucQueueType` 字段实现了队列、二值信号量、计数信号量、互斥锁、递归互斥锁五种原语
2. **链表是一等公民** — 就绪、延时、事件等待全部基于 `list.c` 的循环双链表，支持 O(1) 尾部插入和删除
3. **双锁机制** — `cRxLock`/`cTxLock` 三级状态（-1/0/>0）实现了任务与 ISR 之间的无锁化并发
4. **PendSV 上下文切换** — Cortex-M 上最低优先级 PendSV 确保不在 ISR 内切换上下文
5. **任务通知是最轻量 IPC** — 32 位值直接嵌在 TCB 中，比信号量快 ~45%，比队列省 RAM

## 方法论

采用 **B 级（逐函数精读）** 深度：跟踪每个关键函数的完整逻辑路径，标注分支条件、临界区边界、ISR 差异、边界情况。不逐行翻译代码，而是提取设计意图和实现技巧。
