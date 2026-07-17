<p align="center">
  <img src="https://img.shields.io/badge/Obsidian-7C3AED?style=for-the-badge&logo=obsidian&logoColor=white" alt="Obsidian">
  <img src="https://img.shields.io/badge/notes-150+-blue?style=for-the-badge" alt="Notes">
  <img src="https://img.shields.io/badge/status-growing-success?style=for-the-badge" alt="Status">
</p>

<h1 align="center">⚡ Engineering Wiki</h1>

<p align="center">
  <b>个人工程技术知识库</b> · 基于 Obsidian 双链笔记 · 持续更新<br>
  涵盖嵌入式硬件、姿态解算、通讯协议、计算机视觉等工程领域
</p>

---

## 🗺️ 知识领域

| 领域 | 内容 |
|:-----|:-----|
| ⚡ **电源电子** | DC-DC 拓扑 · COT 控制 · EMI 抑制 · 环路补偿 · 保护机制 |
| 🧭 **姿态解算** | Madgwick · VQF · ESKF · 因子图融合 · 矩阵 Fisher |
| 📡 **通讯协议** | CAN 总线 · HDMI TMDS · TSF 时间同步 · MIPI DSI/CSI/D-PHY/C-PHY |
| 👁️ **计算机视觉** | 立体视觉 · 双目标定 · 椭圆拟合 · 面部关键点 |
| 📍 **导航定位** | ZUPT 零速修正 · UWB 室内定位 · SlimeVR 协议 · 行人惯性导航 |
| 🎥 **视频显示** | HDMI 协议栈 · EDID · MIPI 显示接口族 |
| 🔩 **元器件** | 数据手册摘要 · 选型笔记 · IC 参数速查 |
| 📐 **数学基础** | 高等数学 · 线性代数 · 概率论 · 最优化 |
| 🔬 **研究方法** | 文献检索 · 项目立项 · 实验设计 |

---

## 📂 仓库结构

```
engineering-wiki/
├── 主页.md          ← 知识库入口，分类目录
├── log.md           ← 更新日志
├── CLAUDE.md        ← LLM 协作规范
├── 知识/             ← 概念/知识页，按学科分目录
│   ├── 电源电子/
│   ├── 姿态解算/
│   ├── 导航定位/
│   ├── 视频显示/
│   ├── 通讯网络/
│   ├── 计算机视觉/
│   ├── 数学/
│   └── 研究方法/
├── 元件/             ← 元器件页（IC、传感器、连接器）
└── 来源/             ← 文献/数据手册摘要页
```

---

## 🔗 设计原则

- **知识是纯净的** — 概念页不引用项目上下文，保证可迁移复用
- **来源必须标注** — 每个概念页的 `sources` 关联到原始文献
- **双链是核心** — Wiki-links 交叉引用，形成知识网络而非孤岛
- **持续复合增长** — 每次摄取、查询、审查都让 wiki 更丰富

---

## 🛠️ 关于

本仓库是私有全量 vault 的公开镜像，筛选去除了项目设计文档、PDF 参考文件和个人配置。原始 vault 通过 [Obsidian](https://obsidian.md) + [Obsidian Git](https://github.com/denolehov/obsidian-git) 自动备份。

欢迎通过 [Issues](https://github.com/tzhnahida/engineering-wiki/issues) 讨论或纠错。

---

<p align="center">
  <sub>Built with ❤️ using Obsidian · MIT License</sub>
</p>
