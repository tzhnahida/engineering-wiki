---
type: concept
tags: [electronics, methodology, EDA, AI-validation]
created: 2026-08-13
updated: 2026-08-13
sources: ["[2026-08-13 - OrCAD 网表导出与解析](../../来源/2026-08-13%20-%20OrCAD%20网表导出与解析.md)"]
---

# 原理图 AI 校验方法论

用 AI（Claude Code）对 OrCAD Capture 原理图做辅助校验的完整方法。

## 核心问题：数据从哪来

OrCAD 的 `.dsn` 文件是 OLE2 复合文档格式，逆向解析可提取：

| 数据 | DSN 直接解析 | 网表导出 |
|------|:---:|:---:|
| 版本/许可证 | ✅ | — |
| 网络名称 | ✅ | ✅ |
| 元件实例 | ✅ | ✅ |
| 页面结构 | ✅ | ✅ |
| MCU 引脚名 | ✅ | ✅ |
| **引脚到网络连接** | ❌ | ✅ |

> [!note] 关键发现
> DSN 二进制格式**不存储**引脚到网络的连接关系。连接是 OrCAD 数据库引擎
> 运行时通过走线坐标与引脚坐标的空间匹配计算的，不落盘。
> 因此 pin-to-net 数据只能通过 OrCAD 官方导出（netlist）获得。

## 数据获取流程

```mermaid
flowchart LR
    A[OrCAD Capture] -->|Tools → Create Netlist<br/>PCB Editor 格式| B[pstxnet.dat<br/>pstchip.dat<br/>pstxprt.dat]
    B --> C[Python 解析器<br/>pst_parser.py]
    C --> D[结构化 JSON<br/>元件/网络/引脚连接]
    D --> E[AI 深度校验]
```

## 网表格式（EXPANDEDNETLIST）

```
NET_NAME
'USB3_OTG0_DM'                    ← 网络名
NODE_NAME	U17 A7                ← 元件位号 + 引脚号
'DN1':;                           ← 引脚名
```

差分对属性通过 `DIFFERENTIAL_PAIR='...'` 记录，可用于完整性检查。

## 校验维度

### 结构校验（自动）

| 检查项 | 检测方法 |
|--------|----------|
| 重复位号 | 同一 ref 出现多个元件 |
| 单引脚网络 | net 的引脚列表长度 = 1 |
| 引脚跨网络短路 | 同一 `ref.pin` 出现在多个 net |
| 未连接元件 | 元件不在任何 net 的引脚列表 |
| 差分对完整性 | DIFFERENTIAL_PAIR 属性应成对 |
| 电源轨缺失 | 期望的 VCC/GND 网络是否存在于网表 |

### 语义校验（需要 AI 知识）

| 检查项 | 方法 |
|--------|------|
| 电源轨与芯片 datasheet 匹配 | SoC 需要的 VDD_CPU/GPU/NPU 是否都有 |
| NC 标记与连线矛盾 | OrCAD 导出日志的 ORCAP-36038 警告 |
| DDR4 引脚类别完整性 | DQ 数、GND 数、时钟对是否对称 |
| 自动编号网络审计 | N 开头的网络是否应命名 |

### 深度校验（需要外部参考）

- 与数据手册引脚定义表逐脚对比
- 电源上电时序网络分组
- EMC 滤波/去耦电容分布

## 工具链位置

- 解析器: `_llm/mcp-servers/orcad-reader/pst_parser.py`
- DSN 直接解析: `_llm/mcp-servers/orcad-reader/orcad_parser.py`
- MCP Server: `_llm/mcp-servers/orcad-reader/orcad_reader_server.py`（15 工具）
- 网表存放: `F:/Projects/PCB/_ai_netlist/`

## 已知限制

- OrCAD Capture 命令行 Tcl 自动化受限：脚本在设计加载前执行、对象句柄不可字符串化。可靠的自动化路径是用户手动执行一次 `Tools → Create Netlist` 导出。
- DSN 的 Cache 流含引脚坐标数据，理论上可通过空间匹配重建连接，但精度未验证。
