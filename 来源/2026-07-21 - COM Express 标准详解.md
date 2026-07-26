---
type: source
tags: [COM Express, COMe, 嵌入式, 载板设计, PCB]
created: 2026-07-21
updated: 2026-07-21
source_url: https://mp.weixin.qq.com/s/T2AJeIp-X6UdQna8boPj1A
author: 我的PCB
---

# COM Express 标准详解

> 来源：微信公众号「我的PCB」，2026-07-21 抓取

## 概述

COM Express（COMe）是 PICMG 制定的嵌入式 Computer-on-Module 标准（COM.0）。核心思想是把 CPU、内存、芯片组做成标准模块，通过底部连接器插到载板上。当前版本 R3.1，支持 PCIe Gen4 和 USB4。

## 四种板型

| 类型 | 尺寸 (mm) | 连接器 | 最大功耗 |
|---|---|---|---|
| Mini | 84×55 | 1×220pin (AB) | 68W |
| Compact | 95×95 | 2×440pin (AB+CD) | 137W |
| Basic | 125×95 | 2×440pin (AB+CD) | 137W |
| Extended | 155×110 | 2×440pin (AB+CD) | 137W |

模块板厚 2mm（不是常规 1.6mm），连接器间距 0.5mm。

## 常用 Type 对比

| 特性 | Type 6（主流） | Type 7（网络） | Type 10（Mini） |
|---|---|---|---|
| PCIe | ≤24 lane | ≤32 lane | ≤4 lane |
| 10GbE | ❌ | 4 路 | ❌ |
| 显示 | 3 DDI + LVDS | ❌ | 1 DDI + LVDS |
| 适用 | 通用嵌入式 | 网络/服务器 | 超小体积 |

## 常见坑点

1. 连接器间距 0.5mm（不是 0.8mm）
2. 安装孔 2.7mm / 2.5mm 螺丝（不是 M3）
3. 模块板厚 2mm（不是 1.6mm）
4. Type 10 ≠ Type 1（引脚不兼容）
5. 12V Pin Reclamation 必须加肖特基保护
6. 5mm 堆叠时载板顶部元件 ≤1mm
7. 上电时序：VCC_RTC → VCC_5V_SBY → VCC_12V → PWR_OK

## 与 wiki 的关联

已创建知识页：[COM Express 标准](../%E7%9F%A5%E8%AF%86/%E9%80%9A%E8%AE%AF%E7%BD%91%E7%BB%9C/COM%20Express%20%E6%A0%87%E5%87%86.md)
