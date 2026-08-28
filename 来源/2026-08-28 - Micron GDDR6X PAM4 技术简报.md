---
type: source
tags: [gddr6x, pam4, micron, 技术简报, 图形内存]
created: 2026-08-28
updated: 2026-08-28
---

# 2026-08-28 - Micron GDDR6X PAM4 技术简报

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档 | Micron Technical Brief — Doubling I/O Performance with PAM4 |
| 主题 | Micron® Innovates GDDR6X to Accelerate Graphics Memory |
| 页数 | 8 页 |
| 文件 | `参考/标准/GDDR/Micron_GDDR6X_PAM4_Tech_Brief.pdf` |

## 核心内容

### 定位 (p.1)

- GDDR6 于 2018 年随 GDDR5X 演进而来,每引脚最高 **16 Gb/s**;GDDR6X 目标 **21 Gb/s 及以上**
- GDDR6X 保持与 GDDR6 相同的数据访问粒度和外形,可低风险升级
- 与 NVIDIA 合作:首个采用 **PAM4(四电平脉冲幅度调制)** 编码信令的独立图形存储器,四个电压电平编码 **2 bit/UI**
- 系统内存带宽可达 **1 TB/s**;每事务功耗更低
- 历史:GDDR5X 支撑 GTX 1080 Ti 的成功推动了 JEDEC GDDR6 标准

### GDDR5 → GDDR6X 对比 (Table 1, p.2)

| 特性 | GDDR5 | GDDR5X | GDDR6 | GDDR6X |
|------|-------|--------|-------|--------|
| 密度 | 512Mb–8Gb | 8Gb | 8/16Gb | 8/16Gb |
| 每引脚数据率 | ≤8 Gb/s | ≤12 Gb/s | ≤16 Gb/s | 19 / 21 / >21 Gb/s |
| 通道数 | 1 | 1 | 2 | 2 |
| 访问粒度 | 32B | 64B | 2×32B | 2×32B |
| Burst 长度 | 8 | 16/8 | 16 | 8(PAM4 模式)/16(RDQS 模式) |
| 信令 | POD15/POD135 | POD135 | POD135/POD125 | **PAM4** + POD135/POD125 |
| 封装 | BGA-170, 0.8mm | BGA-190, 0.65mm | BGA-180, 0.75mm | BGA-180, 0.75mm |
| I/O 宽度 | x32/x16 | x32/x16 | 2ch ×16/8 | 2ch ×16/8 |
| CRC | CRC-8 | Modified CRC-8 | 2× CRC-8 | 2× CRC-8(半速率 jedec.org 方案) |
| VREFD | 外部/内部(每 2B) | 内部(每字节) | 内部(每引脚) | 内部每引脚,3 子接收器/引脚 |

### PAM4 原理 (p.3)

- 16 Gb/s 时单 UI 时序窗口仅 **62.5 ps**;GDDR6 用 NRZ (PAM2) 已逼近极限
- PAM4 每个符号编码 2 bit → 相同数据率下 GDDR6X 电路频率只需 GDDR6 的一半,同时降低 I/O 功耗(Fig. 2 示 16 Gb/s 下 2-bit 传输的眼图对比)

## 参见

- [GDDR 图形内存体系](../知识/硬件设计/GDDR%20图形内存体系.md) — GDDR 代际与架构知识页
