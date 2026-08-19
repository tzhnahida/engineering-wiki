---
type: source
tags: [embedded, rtos, zephyr, linux-foundation, cortex-m]
created: 2026-07-26
updated: 2026-07-26
---

# 2026-07-26 - Zephyr RTOS 源码分析

> **仓库**: https://github.com/zephyrproject-rtos/zephyr
> **本地**: `_llm/raw/repos/zephyr/`
> **协议**: Apache 2.0 | **版本**: 4.4.99-dev

## 概述

Zephyr 是 Linux 基金会维护的可扩展安全 RTOS。支持 ARM/x86/RISC-V/MIPS 等 7 种架构、179 块开发板、99 类驱动、45 个子系统。内核 API 面 7,331 行。

## 核心架构

- **三级构建系统**：CMake + Kconfig + Device Tree，多阶段链接（pre0/pre1/final）
- **内核**：协作/抢占双优先级 + EDF 调度 + 原生 SMP + MPU/MMU 线程隔离
- **IPC 完备**：信号量/互斥锁/消息队列/FIFO/管道/邮箱/事件/条件变量 全部内置

## STM32 支持

- 29 个 STM32 系列、129 块板卡（Nucleo/Discovery/Eval/DevKit），覆盖 F0→H7→MP2
- HAL/LL 集成在 SoC 层，多编程器支持（CubeProgrammer/OpenOCD/J-Link）

## 关键子系统

| 子系统 | 内容 |
|--------|------|
| 蓝牙 | 完整 BLE 协议栈（Controller+Host+Mesh+Audio） |
| 网络 | 内置 TCP/IP，支持 MQTT/CoAP/HTTP/TLS |
| 文件系统 | LittleFS + FatFS + VFS 抽象 |
| USB | 设备/主机/USB-C PD |
| GUI | LVGL 集成 + 40+ 显示驱动 |
| 传感器 | 统一驱动模型，数十种传感器 |
| 电源管理 | 多状态 PM + 设备级电源域 |
| POSIX | pthread/semaphore/clock/fs/eventfd |

## 与 FreeRTOS 对比

| | Zephyr | FreeRTOS |
|--|--------|----------|
| 构建 | CMake+Kconfig+DT | 简单 Makefile |
| 驱动模型 | 统一 `struct device` (99 类) | 无 |
| 网络栈 | 内置 TCP/IP | 需外挂 LWIP |
| BLE | 内置完整协议栈 | 无 |
| POSIX | 完整子集 | 无 |
| 安全性 | MPU/MMU 隔离+syscall | 无 |
| 学习曲线 | 陡峭 | 平缓 |

## 已产出的知识页

- [Zephyr/1. Zephyr 概述与架构全景](../Zephyr/1.%20Zephyr%20概述与架构全景.md)
