---
type: source
tags: [embedded, debug, cortex-m, hardfault, library]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - CmBacktrace 源码分析

> **仓库**: https://github.com/armink/CmBacktrace
> **本地**: `_llm/raw/repos/CmBacktrace/`
> **协议**: MIT | **版本**: 1.5.0 | **作者**: armink (朱天龙)

## 概述

CmBacktrace 是 ARM Cortex-M 系列 MCU 的自动 HardFault 诊断库。捕获故障现场的全部寄存器、解析故障原因、重建调用栈。

支持 Cortex-M0/M3/M4/M7/M33，兼容裸机/FreeRTOS/RT-Thread/uCOS/RTX5/ThreadX。

## 核心设计

1. **汇编跳板**：替换 `HardFault_Handler`，捕获 LR (EXC_RETURN) 和 SP
2. **C 诊断层**：读取异常栈帧（R0-R3/R12/LR/PC/xPSR），解析所有 SCB 故障状态寄存器
3. **调用栈重建**：扫描栈内存，通过 BL/BLX 指令特征验证候选返回地址
4. **线程感知**：通过 EXC_RETURN bit2 判断故障发生在线程还是 ISR，分别读取 PSP/MSP

## 关键文件

| 文件 | 作用 |
|------|------|
| `cm_backtrace.c` (726行) | 核心诊断逻辑 |
| `cmb_cfg.h` | 用户配置（平台、OS、语言） |
| `cmb_def.h` | 寄存器结构体、RTOS 头文件、内联汇编 |
| `cmb_fault.S` (4个编译器版本) | HardFault_Handler 跳板 |

## 已产出的知识页

- [CmBacktrace/1. CmBacktrace 概述与故障诊断原理](../知识/嵌入式软件/CmBacktrace/1.%20CmBacktrace%20概述与故障诊断原理.md)
- [CmBacktrace/2. CmBacktrace 集成与配置](../知识/嵌入式软件/CmBacktrace/2.%20CmBacktrace%20集成与配置.md)
