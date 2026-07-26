---
type: source
tags: [slimevr, server, java, kotlin, steamvr, skeleton, source-analysis]
created: 2025-07-25
updated: 2025-07-25
---

# SlimeVR Server 上位机源码分析

> SlimeVR/SlimeVR-Server — PC 端 Java/Kotlin 上位机。接收 Tracker UDP 数据 → 骨骼解算 → SteamVR 输出。

- **仓库**: https://github.com/SlimeVR/SlimeVR-Server
- **语言**: Kotlin + Java (Gradle 多模块)
- **子模块**: `solarxr-protocol` (共享协议), `bindings-provider/openvr` (Valve OpenVR SDK)
- **目标平台**: Windows / Linux (Android 变体)
- **GUI**: React/TypeScript (pnpm monorepo)

核心架构：单线程游戏主循环（~1000Hz）+ UDP 接收线程 + SteamVR 桥接 IO 线程。从四元数到骨骼姿态的完整管线：旋转校正（复位/漂移/滤波）→ FK 链装配 → 约束执行 → 腿部校正 → 计算输出 → SteamVR。

## 分析范围

| 模块 | 文件数 | 职责 |
|------|--------|------|
| `server/core` | ~66 Java/Kotlin | UDP 接收、Tracker 管理、骨骼解算、姿态管线 |
| `server/desktop` | ~20 Kotlin/Java | SteamVR 桥接、平台 IO（命名管道/Unix Socket） |
| `solarxr-protocol` | protobuf | 协议定义（共享给固件和 GUI） |
| `gui/` | React/TS | Web GUI |
