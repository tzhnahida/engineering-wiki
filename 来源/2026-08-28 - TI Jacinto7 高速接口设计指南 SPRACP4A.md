---
type: source
tags: [硬件设计, 设计指南, TI, 高速接口, PCB, Jacinto7, SI]
created: 2026-08-28
updated: 2026-08-28
---

# 2026-08-28 - TI Jacinto7 高速接口设计指南 SPRACP4A

## 文档信息

| 项目 | 内容 |
|------|------|
| 文档 | Jacinto7 AM6x, TDA4x, and DRA8x High-Speed Interface Design Guidelines (Application Note) |
| 编号 | SPRACP4A — DECEMBER 2019, REVISED JUNE 2024 |
| 页数 | 33 页 |
| 文件 | `参考/标准/设计指南/TI_SPRACP4A_Jacinto7_High-Speed_Interface_Design.pdf` |
| 适用器件 | J7200/J721E/J721S2/J722S/J742S2/J784S4 系(AM6x / TDA4x / DRA8x)(p.3) |

## 内容结构

### §2 通用高速接口设计指导 (p.4–14)

| 主题 | 要点 | 页 |
|------|------|-----|
| 走线阻抗 | 按协议确定单端/差分阻抗与容差(示例 50Ω ±15%);**松散耦合差分对更易控阻抗**,贴近耦合对线宽/间距变化极敏感,量产阻抗难保 | p.4 |
| 走线长度 | 尽量最短;长度要求见 §3 各协议表 | p.4 |
| 差分等长 | 组内 (intra-pair) 必须匹配,蛇形线加在**不匹配端**附近;组间 (inter-pair) 视标准而定,如 USB3.0 TX 与 RX 无需互等 | p.4 |
| 参考平面 | 高速信号走**实心 GND 平面**,**不建议参考电源平面**;跨越平面开槽/分割会迫使回流绕行 → 辐射、延迟、抖动、幅度劣化 | p.5–6 |
| 差分间距 | 见 Fig. 2-8 示例 | p.7 |
| 过孔 | 过孔不连续 → 减小 stub、背钻、反焊盘直径、均衡过孔数;注意 SMD 焊盘不连续与地平面掏空 | p.10–13 |
| 信号拐弯 | 遵循信号弯曲规则(Fig. 2-16) | p.13 |
| ESD/EMI | 流经式 (flow-through) 布线等 ESD/EMI 布局规则 | p.14 |

### §3 接口专属设计指导 (p.15–28)

USB(3.1/2.0 路由规格表 p.16–17)、DisplayPort(AC 耦合电容与路由规格 p.18–20)、PCIe(REFCLK 要求、AC 耦合、路由规格 p.21–23)、MIPI D-PHY CSI-2/DSI(p.23–25)、UFS(p.25–26)、Q/SGMII(p.27–28)。

### §4 板级仿真 (p.28–31)

板模型提取 → 模型验证 → S 参数检查 → TDR 分析 → 通道仿真(眼图/浴缸曲线)→ 结果审查。表 4-1 列出各标准的眼罩规格。

## 参见

- [高速PCB设计通用规范](../知识/硬件设计/高速PCB设计通用规范.md) — 综合高速板设计知识页
