---
type: source
tags: [canopen, can, protocol, ciA301, embedded, source-analysis]
created: 2025-07-25
updated: 2025-07-25
---

# CANopenNode 源码分析

> CANopenNode v4.0 — Apache 2.0 协议。CiA 301 CANopen 应用层标准 C 实现。

- **仓库**: https://github.com/CANopenNode/CANopenNode
- **标准**: CiA 301 (CANopen), 303-3 (LEDs), 304 (SRDO), 305 (LSS), 309 (gateway)
- **规模**: 55 C/H 文件, ~20KB 代码, ~2KB RAM (最小配置)

双线程非阻塞架构：1ms 实时线程处理 SYNC/PDO，后台主循环处理 NMT/SDO/Heartbeat/Emergency。只需移植 `CO_driver.c`（CAN 控制器驱动）即可适配任意 MCU。
