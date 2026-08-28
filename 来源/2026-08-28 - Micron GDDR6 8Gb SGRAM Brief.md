---
type: source
tags: [gddr6, micron, sgram, 数据手册, 图形内存]
created: 2026-08-28
updated: 2026-08-28
---

# 2026-08-28 - Micron GDDR6 8Gb SGRAM Brief

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档 | 8Gb: 2 Channels x16/x8 GDDR6 SGRAM — Features/Part Numbering Brief |
| 器件 | MT61K256M32 (256 Meg × 32) |
| 文档编号 | gddr6_sgram_8gb_brief.pdf, Rev. I 10/19, 23 页 |
| 文件 | `参考/标准/GDDR/Micron_GDDR6_8Gb_SGRAM_Brief.pdf` |

## 核心内容

### 特性列表 (p.1)

- VDD = VDDQ = 1.35V ±3% / 1.25V ±3% / 1.20V −2%/+3%;VPP = 1.8V −3%/+6%
- 数据率:12 / 14 / 16 Gb/s(每引脚)
- **2 个独立通道(x16)**,x16/x8 与伪通道 (PC) 模式在复位时配置
- 每通道命令/地址 (CA) 与数据为单端接口;CA 用差分时钟 CK_t/CK_c(每 2 通道一组);每通道一个差分数据时钟 WCK_t/WCK_c(DQ、DBI_n、EDC)
- CA 为 DDR,数据为 QDR (WCK) 或 DDR(依工作频率)
- **16n prefetch,每次阵列读/写 256 bits**
- 16 个内部 bank;4 个 bank group(tCCDL = 3tCK / 4tCK)
- 可编程 READ/WRITE latency;Write data mask 经 CA 总线(单/双字节粒度)
- DBI 与 CABI(数据/CA 总线反相)
- 输入/输出 PLL
- **CA bus training**(经 DQ/DBI_n/EDC 监视 CA 输入)、**WCK2CK 时钟训练**(经 EDC 相位信息)、读写训练(读 FIFO 深度 = 6)
- 读写传输完整性由 **CRC** 保证;可编程 CRC 读写延迟;可编程 EDC hold pattern 供 CDR
- 低功耗模式;片上温度传感器;自动刷新 32ms / 16k cycles(per-bank 与 per-2-bank 选项)
- 高速输入全部有 ODT;**POD135 / POD125 兼容伪开漏输出**
- ZQ 引脚 (120Ω) 自动校准;数据输入用内部 VREF + **DFE**,接收特性逐引脚可编程
- 180-ball BGA 封装;符合 IEEE 1149.1 边界扫描;TC = 0°C ~ +95°C

### 型号解析 (p.2)

`MT 61 K 256M32 JE -16 :A`:61 = GDDR6 SGRAM;K = 1.35V;JE = 180-ball FBGA 12.0×14.0mm;-16 = 16 Gb/s;A = Rev A。FBGA 丝印为缩写,需用 Micron 官网解码器转换。

## 参见

- [GDDR 图形内存体系](../知识/硬件设计/GDDR%20图形内存体系.md) — GDDR 代际与架构知识页
