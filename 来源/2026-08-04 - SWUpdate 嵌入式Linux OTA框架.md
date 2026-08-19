---
type: source
tags: [embedded, linux, ota, swupdate, yocto, buildroot, dual-copy]
created: 2026-08-04
updated: 2026-08-04
---

# 2026-08-04 - SWUpdate 嵌入式 Linux OTA 框架

## 元数据

| 属性 | 值 |
|------|-----|
| 标题 | SWUpdate：嵌入式 Linux OTA 更新框架 |
| 日期 | 2026-08-03 |
| 来源 | 微信公众号「LabHub」 |
| 参考 | [SWUpdate GitHub](https://github.com/sbabic/swupdate) (GPL-2.0, 1841 Star) |
| 格式 | 在线文章 + 官方文档 |

## 文档概述

SWUpdate 是嵌入式 Linux 最广泛使用的开源 OTA 框架，Yocto Project 和 Buildroot 官方集成组件。核心机制：双分区镜像 + 原子替换 + U-Boot bootcount 自动回退。支持更新 rootfs、内核、U-Boot、Device Tree、MCU 固件、FPGA 比特流。内置 Lua 脚本引擎和 RSA 签名 + AES-256 加密。

## 关键知识点

- **双副本 (dual-copy)**：A/B 分区 + U-Boot bootcount/bootlimit 机制——连续 N 次引导失败自动回退
- **一个 .swu 包更新一切**：cpio 压缩包含 sw-description + 镜像 + 签名
- **Lua 钩子**：pre_install() / post_install() 实现自定义更新逻辑
- **安全**：OpenSSL/mbedTLS/WolfSSL 三后端，RSA 签名 + AES-256-CBC 加密
- **增量更新**：基于 librsync，只传输差异块 (5-20MB vs 200MB 全量)
- **局限**：无内置服务端（需自建 HTTP/hawkBit）；不能用于 bare metal MCU；双分区需额外 Flash

## 创建的 Wiki 页面

- [嵌入式软件/SWUpdate 嵌入式Linux OTA](../知识/嵌入式软件/SWUpdate%20嵌入式Linux%20OTA.md)
