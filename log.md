---
type: log
created: 2026-06-07
updated: 2026-07-15
---

## [2026-07-15] maintain | 全量：26 颗核心 IC 理解融合式中文数据手册完成

- 在试点 5 颗基础上铺开剩余 21 颗（21 个并行 agent，每颗完整读取手册全文后整页重写），全库覆盖：
  - 电源（10）：TPS54302、TPS55288、BQ25758、TPS65130、TPS65150、TLV62569、LP5907、AP3012、LM5050、SGM3752
  - 接口（5）：TJA1044、HUSB320、CH340、LT7911D、W25Q64
  - MCU/传感器（4）：STM32G431、ESP32-C6、ESP8266EX、ICM-42688-P
  - 视频/音频（4）：TC358870、PCM5102A、ICS-43434、LCD Display
  - 逻辑/分立（3）：MAX706、A4950、BSC028N06NS
- 总规模：26 页共约 6900 行，每页含工作原理详解、设计计算流程（LaTeX 公式）、寄存器/指令速查、PCB 布局要点、callout 标坑
- 系统性纠错（原页与手册不符，全部以手册为准修正并留 callout 记录），重灾区是引脚表：
  - **引脚表虚构/错位（7 页）**：HUSB320（12 脚全错+虚构 VDDIO/GPIO/PCBL）、LM5050（虚构 VIN/VS/VCC 且 MOSFET 方向画反）、A4950（整表为其他型号）、PCM5102A（虚构 VINL/VOUTR 等）、ICS-43434（6 脚错位）、CH340（SOP-16 大面积错位）、TJA1044（5/8 脚互换——照原表画板 STB 会接到总线侧）
  - **虚构功能删除**：STM32G431"HRTIM"（G474 专属）、AP3012"热关断"、TJA1044"睡眠/静默模式"、CH340"CDC-ACM 原生驱动/硬件流控"、HUSB320"PD 3.0"、BQ25758"NVDC 功率路径/充电状态机"
  - **危险参数修正**：TPS54302 EN 闲置接 VIN（EN 极限仅 7V）、LT7911D VCC33 极限 4.6→3.63V、BSC028N06NS Tj 175→150°C/EAS 130→100mJ、W25Q64 极限电压/ESD、LCD Display 整页张冠李戴（实为 HS20HS072RX 2.4" TFT ST7789T3，原按 HD44780 字符屏写）
- 全部校验通过：乱码 0、嵌图断链 0、无项目页违规链接
- 图片标注纠错：BSC028N06NS 的 p4_fig1 实为功耗降额曲线（原标 SOA），已用 PDF 矢量文字坐标核实并修正

## [2026-07-15] maintain | 试点：5 颗核心 IC 理解融合式中文数据手册

- 方式：提取数据手册全文文本 → 5 个并行 agent 深度阅读 → 在现有 8 节骨架上整页重写（非翻译，理解融合）
- TPS54302（105→284 行）：峰值电流环逐周期行为、7 步 LC 设计公式流程（参考设计验算贯通）、打嗝 OCP 细节；按 Rev C 修正 ESD/VFB/EN 阈值/热关断参数；纠错——原页"EN 闲置接 VIN"是危险建议（EN 绝对最大 7V）
- ICM-42688-P（130→398 行）：噪声密度→ARW 换算、滤波链群延迟表、FIFO 包格式、CLKIN 时间缩放（多 IMU 同步的代价与收益）、寄存器速查；纠错——-P 为单接口版本，无 AUX 路径
- TC358870（444→559 行）：4K@30 带宽核算（双 DSI 的必然性）、HDMI-Rx 五级信号检测链、音频 PLL 外部 LPF 电路、TMDS 反向电流防护三方案；修正 Tstg/球距为手册原值；原第 8/9 节 EDID 与寄存器总表全保留
- ESP32-C6（69→313 行）：HP/LP 双核低功耗架构、三模射频峰值电流表（供电按 ≥0.5A 设计）、Strapping 启动真值表、电源模式 mermaid 状态图
- W25Q64（113→369 行）：存储层级图、34 条指令速查、SR1-3 保护位组合逻辑、IO 模式带宽换算；纠错——原页极限值 -0.5V/±4000V 与手册不符（实为 -0.6V/±2000V）；标坑 Q 后缀 QE 出厂锁定
- 校验：5 页共 1923 行，乱码 0、嵌图断链 0、wikilink 全部可解析

## [2026-07-15] correction | 11 个元件页 GBK 乱码修复

- 症状：历史 GBK/UTF-8 转码损坏，中文字符变为 U+FFFD（"特性"→"特�?"、"至"→"�?"等），共约 455 处
- 波及：ESD3V3、ESD_CANFD、SMAJ/SMBJ/SMCJ、USBLC6-2SC6、CH340、Current Sense Amp、HUSB320、LT7911D（最重，142 行）、A4950
- 修复方式：上下文规则批量替换（_llm/raw/fix_mojibake.py）+ LT7911D/HUSB320 全文重写 + 逐文件人工校对
- 附带修复：损坏的粗体标记（**x*: → **x**:）、表头（最小/最大/值/页）、箭头与波浪线区间
- 校验：全库 U+FFFD 残留 0，130 处图片嵌入 0 断链

## [2026-07-15] maintain | PaddleOCR 图片管线上线 — 50 页全库插图

- OCR 切换：删除 deepseek-ocr skill（服务已废弃），全面改用 PaddleOCR PPStructureV3（GPU，含版面分析 + figure bbox 裁剪）
- 新管线脚本（_llm/raw/）：paddle_extract.py（PDF→OCR→裁图+manifest）、caption_fix.py（PDF 文本层回填图注）、find_fig_pages.py（图注定位选页）、page_crop.py（整页白边裁剪）、batch_papers/datasheets/standards.py（批处理驱动）
- 提取规模：11 篇论文 + 30 份数据手册 + 8 份标准，共 289 张裁剪图，按来源分目录存放 _llm/raw/assets/{papers,datasheets,standards}/，每目录带 manifest.json（页码/bbox/尺寸/图注）
- 知识页插图（23 页）：脚装惯性导航奠基、零速检测 ZUPT、误差状态卡尔曼滤波、双目标定、直接最小二乘椭圆拟合、面部关键点检测、立体视觉与关键点估计、TSF WiFi 时间同步、展频谱与EMI抑制、DC-DC降压转换器、恒定导通时间控制、轻载效率模式、HDMI 协议概述/TMDS 编码/物理层/视频传输、MIPI 概述/DSI/D-PHY/C-PHY/DCS/DBI/DPI
- 元件页插图（27 页）：TPS54302、BQ25758、TPS55288、TPS65130、TPS65150、TLV62569、LP5907、AP3012、LM5050、SGM3752、PCM5102A、ICS-43434、HUSB320、LT7911D、TC358870、TJA1044、W25Q64、CH340（无有效图）、STM32G431、ESP32-C6、ICM-42688-P、INA226 CSA、MAX706、A4950、BSC028N06NS、AO3401A、MMBT2222A、USBLC6
- 修复：零速检测 ZUPT / 双目标定 / 误差状态卡尔曼滤波 三页 sources YAML 格式破损；展频谱与EMI抑制 补齐 sources/updated 字段
- 清理：删除 7 个引用已废弃 DeepSeek OCR 服务的遗留脚本
- 校验：全库 129 处图片嵌入路径 0 断链
- 遗留问题：Madgwick 2010 论文 PDF 仅 2 页（疑为截断版，完整技术报告 32 页），梯度下降姿态解算页无图可插；ESP8266EX/CH340/LCD 数据手册无可提取图形

## [2026-06-30] ingest | UWB 室内定位论文四篇 → 知识页 + 来源摘要

- 来源 [2026-06-30 - UWB IMU 下肢动捕级联Kalman](来源/2026-06-30%20-%20UWB%20IMU%20下肢动捕级联Kalman.md)（Zihajehzadeh 2015，70 引，<5cm 腰部轨迹）
- 来源 [2026-06-30 - UWB IMU 上肢动捕 Madgwick KF 融合](来源/2026-06-30%20-%20UWB%20IMU%20上肢动捕%20Madgwick%20KF%20融合.md)（Shi 2023，24 引，Madgwick+KF，RMSE 改善 1.2cm）
- 来源 [2026-06-30 - UWB 人体遮挡 IMU 缓解](来源/2026-06-30%20-%20UWB%20人体遮挡%20IMU%20缓解.md)（De Cock 2023，14 引，HBS p75 误差降 37%）
- 来源 [2026-06-30 - UWB MIMU 人体运动分析综述](来源/2026-06-30%20-%20UWB%20MIMU%20人体运动分析综述.md)（Yogesh 2023，技术综述）
- 知识 [导航定位/UWB 室内定位与动捕融合](知识/导航定位/UWB%20室内定位与动捕融合.md) 创建：级联 KF 架构、人体遮挡缓解、EKF 观测模型、EWM550 选型
- 知识总览更新

## [2026-06-30] design | 全身动捕系统 UWB 集成 + CAN 拓扑调整

- 躯干 CAN 总线 master 从脖子移至腰部（运动链根节点）
- PCB 从 2 种增至 5 种：手背主(ADC)、脚背主(标准)、腰主(UWB)、从节点、头显
- 腰主 PCB P1 焊接 EWM550 但不写固件，P2 启用
- UWB 锚点 P2 独立一版 PCB：ESP32-C3 + EWM550 + 18650
- Phase 1 成本 ~¥1431，Phase 2 加 ¥400，总硬件 ~¥1831
- 新增创新点：ZUPT-UWB 紧耦合、分布式人体遮挡检测

## [2026-06-30] lint | 全量审计修复 — 断链/孤儿/stub/OCR清理

- log.md: 去除全部 wikilink（12+ 条违规）
- 主页.md: 修复 2 条断链、更新视频显示分类含 C-PHY
- 元件页: 修复 知识/电源保护机制→电源电子/电源保护机制（7 文件）、删除不存在页面的 wikilink（9 文件）
- 孤儿页补充入链: 零速检测 ZUPT→脚装惯性导航奠基、双目标定→椭圆拟合
- 来源摘要: 删除 9 个二进制损坏 stub、重建完整摘要
- OCR 临时文件: 删除 _llm/raw/gaoshu*/ocr_text.md（19 个中间产物）

## [2026-06-30] ingest | MIPI C-PHY Specification v2.1
## [2026-06-29] project | TC358870 Demo 评估板入库

从 `F:\Projects\PCB\GitHub\Public\TC358870_Demo\` 读取 README 及配套 CLAUDE.md，写入项目页 `项目/PCB/GitHub/Public/TC358870 Demo.md`。该 Demo 板是全身动捕与头显系统头显显示子系统的前期验证平台——4 层 PCB，TC358870XBG HDMI→MIPI DSI 桥接，驱动 LS029B3SX01 1440×1440@90Hz 面板。Rev 1.0 已废弃（MIPI lane swap 烧屏），Rev 2.0 布局完成待仿真。同步更新全身动捕与头显系统页（参见链）、项目总览（修正 wikilink）、TC358870 元件页（参见链）。

## [2026-06-28] ingest | MIPI 显示接口规范族

从 Clippings 摄入 9 份 MIPI Alliance 规范 PDF（D-PHY v1.0/v2.5、DSI v1.01/v1.3、DCS v1.02、DBI v2.0、DPI v2.0、DDB v0.82、CSI-2 v2.0）。

创建知识页：
- 视频显示/MIPI 概述 — MIPI 规范家族全景，分层体系与 HDMI 对比
- 视频显示/MIPI D-PHY — 物理层：HS/LP 双模信号、Burst 传输、Escape Mode、BTA、电气规范
- 视频显示/MIPI DSI — 协议层：包格式、Video/Command Mode、虚拟通道、ECC/Checksum、DSC、Sub-Link
- 视频显示/MIPI DCS — 命令集：电源状态机、伽马曲线、自诊断、DCS 命令速查
- 视频显示/MIPI DBI — 并行总线接口：Type A/B/C（6800/8080/SPI）
- 视频显示/MIPI DPI — 并行像素接口：HSYNC/VSYNC/DE/DOTCLK

来源摘要：（5 篇）MIPI D-PHY v2.5、DSI v1.3、DCS v1.02、DBI v2.0、DPI v2.0

PDF 移至 参考/标准/。更新 知识总览（6 条目）和 主页。

## [2026-06-28] ingest | 同济高等数学第8版下册

OCR 354 页扫描版 PDF（阿里云 OCR + DeepSeek OCR），基于 5 章内容创建 26 篇数学知识页：
- 第 8 章 向量代数与空间解析几何（11 篇）
- 第 9 章 多元函数微分法及其应用（7 篇）
- 第 10 章 重积分（3 篇）
- 第 11 章 曲线积分与曲面积分（3 篇）
- 第 12 章 无穷级数（2 篇）
PDF 归档至 参考/论文/，来源摘要及知识索引见 知识/数学/数学知识总览。

# Wiki Log

## [2026-06-27] knowledge | ZUPT/EKF 升级算法知识页 + 实验待办

- 自适应步频零速检测 (AWGF-ZVD): GLRT 阈值变为步频函数，慢走/跑步自适应，Wang 2024
- Soft-ZUPT 概率零速修正: 着地→静止连续概率修正，替代硬 0/1，Gao & Deng 2026
- 迭代不变扩展卡尔曼滤波 (IterIEKF): 测量更新迭代线性化，Chauchat arXiv 2024
- 更新: 知识总览.md 姿态解算+导航定位索引，全身动捕项目 Phase 1 新增三个实验待办

## [2026-06-27] knowledge | 统一因子图姿态推断框架

- 新建: 姿态解算/统一因子图姿态推断框架 — 六篇论文的数学统一
- 形式化 15 节点动捕为 SO(3) 因子图：5 类因子 + Matrix Fisher 消息传递
- 六篇论文映射到因子图的边：不是算法拼接，是图模型的模块化替换
- 精度/性能量化：L0−L5 渐进精度表（3.7°→<0.15°），ESP32 15%负载，GPU 7ms
- 算法创新评估：MKCL+InEKF 在 Lie 群上的测量更新是唯一纯数学贡献
- 论文策略：主论文 T-ASE/TIM，可选拆出 RAL 短文

## [2026-06-26] knowledge | 六篇姿态解算算法知识页（论文细读修正）

- 下载并细读 VQF/Correntropy GD/MDR/DO IONet/AVNet 五篇全文
- VQF 姿态解算滤波器: 修正为准确的三模块串联方程，倾斜校正关键技巧（近似惯性系低通），2.9° 全数据集平均，非 ITU 数据
- 相关熵梯度下降姿态解算: 修正为 MKCL 的 MLE 框架，作者证明 IMU 噪声是重尾分布，非简单"抗野值"
- MDR 磁畸变抑制: 修正为体素数据库+IAI 自适应更新完整流程，27 小时智能手表数据
- DO IONet Transformer直接姿态: 修正为 200 帧窗口+四路输入+CNN+Transformer 完整架构
- AVNet 不变扩展卡尔曼姿态: 修正为 CNN+GRU 双分支+协方差适配器+InEKF 完整架构
- Matrix Fisher SO3概率姿态: 修正为 SO(3) 流形概率分布+贝叶斯融合框架
- 更新: IMU姿态解算算法演进 总览页 VQF 对比表

## [2026-06-26] knowledge | 六篇姿态解算算法知识页

- VQF 姿态解算滤波器: 解耦式三模块, 静态 0.011°, 磁干扰拒绝, C++ 开源
- 相关熵梯度下降姿态解算: 多核相关熵损失, 抗野值, 比 Kalman 计算量低
- MDR 磁畸变抑制: 3D 体素建模局部磁场, 比上代好 44.5%
- DO IONet Transformer 直接姿态: Transformer 直出四元数, 跳过积分漂移
- AVNet 不变扩展卡尔曼姿态: CNN+GRU + InEKF, 手机 IMU 水平误差 ~0.4%
- Matrix Fisher SO(3) 概率姿态: SO(3) 上贝叶斯融合, 最前沿数学框架
- 更新: IMU姿态解算算法演进 总览页加详页链接, 知识总览.md 索引

## [2026-06-26] knowledge | IMU 姿态解算算法演进全景

- 新建知识页: 姿态解算/IMU姿态解算算法演进 — Madgwick 到深度学习六篇论文全景
- 跨三派系对比: 经典滤波器(VQF/Correntropy/MDR) vs 深度学习(DO IONet/AVNet) vs 贝叶斯(Matrix Fisher)
- 项目路线图: Phase 1 Madgwick→VQF, Phase 2 PC端混合学习方案
- 论文创新点: 六条备选方向 + 可行性评估
- 更新: 知识总览.md 索引

## [2026-06-26] ingest | ICM-42688-P 数据手册入库

- 新建元件页: 传感器/ICM-42688-P (6 轴 MEMS, 陀螺 2.8mdps/√Hz, 加速度 70μg/√Hz)
- 重写来源摘要: 2026-06-24 - ICM-42688-P 数据手册 (原文件已损坏)
- 更新项目页: 全身动捕与头显系统器件清单 ICM-42688-P 已入库
- 关键选型理由: 噪声比 MPU6050 低 44%, 支持 SPI 24MHz, FIFO 20-bit

## [2026-06-26] correction | LS029B3SX01 背光 SGM3752 配置纠正

- LS029B3SX01: LED 拓扑确认为 2P4S, 总电流 22mA (非 8 串 11mA)
- SGM3752: 新增背光应用节, RSET=9.1Ω, VOUT≈11.6V
- 两页新增双向链接

## [2026-06-24] redesign | CAN 集群架构重构

- 动捕从独立 WiFi 节点改为 CAN 集群: 5 ESP32-S3 主节点 + 10 STM32F103C8T6 从节点
- 仅 2 种 PCB: 主节点(ESP32+IMU+CAN+WiFi) + 从节点(STM32+IMU+CAN+拨码)
- 互连线: VCC/CAN_H/CAN_L/GND 四线, 从节点 PCB 过路并联
- WiFi 节点: 17 → 7 (5 主 + 2 眼追), 碰撞问题消解
- 主节点物理位置自明(躯干/手/脚), 从节点 1-bit 拨码
- 手背主节点 5 路 ADC 直读拉线电位器, 无需独立 ADC 芯片
- QMC5883P 选型确定: 2mG, I²C, ¥5/颗
- 成本: ¥1665 → ¥1366
- 更新: 架构图、节点表、器件清单、成本表、数据流、时序图、待办

## [2026-06-24] knowledge | SlimeVR 协议

- Created 知识/SlimeVR 协议 — UDP packet format, tracker handshake, bone mapping, SteamVR bridge
- Compared to standard SlimeVR: project is superset (TSF sync, finger ADC, eye tracking, STM32 headset)
- Updated 知识总览 导航 section, project 参见
- Protocol: 0x01 handshake, 0x04 quat data, 20-30B/packet @ 100Hz → 45kB/s total

## [2026-06-24] ingest | MediaPipe Face Mesh + Attention Mesh

- Kartynnik 2019 (CVPR Workshop): 468 3D face vertices, BlazeFace→ResNet, 2.5ms iPhone, IOD MAD 3.96%
- Grishchenko 2020 (arXiv): Attention Mesh — spatial transformer submodels, lips error -12%, 16.6ms unified
- Created knowledge: 面部关键点检测 — pipeline, model variants, 1 Euro filter, Phase 2 mouth tracking
- Updated 知识总览 and project 参见
- PDFs archived to _llm/raw/sources/

## [2026-06-24] ingest | Zhang 相机标定 + Fusiello 双目校正

- Zhang 2000 (MSR-TR-98-71): checkerboard homography → B=A⁻ᵀA⁻¹ closed-form → LM refinement → k1,k2 radial
- Fusiello 2000 (MVA): new PPMs with shared R, baseline-aligned X axis → rectification T=Q_n·Q_o⁻¹
- Created knowledge: 双目标定 — OpenCV calibrateCamera→stereoCalibrate→stereoRectify full derivation
- Updated 知识总览 CV section, project 参见 links
- PDF archived to _llm/raw/sources/

## [2026-06-24] knowledge | 5篇深度知识页（对标 立体视觉与关键点估计 标准）

- Deep rewrote all knowledge pages from ingested papers — full derivations, quantitative tables, ablation analysis:
  - 梯度下降姿态解算 — Madgwick filter: fusion equation, analytic Jacobian, β physics, 109 ops breakdown, ICM-42688-P β=0.015
  - 零速检测 ZUPT — Skog+IPIN merged: GLRT framework, SHOE→4 detectors, PSD metric, SVM adaptive ×14 threshold, 63% error reduction
  - 误差状态卡尔曼滤波 — Solà ESKF: nominal/error split, 99-dim Phase 2 state, ZUPT+marker+kinematic observations
  - 直接最小二乘椭圆拟合 — Fitzgibbon B2AC: 4ac-b²=1 constraint, eigenvalue uniqueness proof, noise sensitivity table, ESP32 strategy
  - TSF WiFi 时间同步 — 802.11 §11.1: PHY compensation, ±100ppm→±20µs @100ms Beacon, ESP32 API, multi-source alignment

## [2026-06-24] knowledge | 论文知识化 + 元件规格书

- Created 5 knowledge pages from ingested papers:
  - 梯度下降姿态解算 — Madgwick filter concept
  - 零速检测 ZUPT — Skog 2010 + IPIN 2017 combined
  - 误差状态卡尔曼滤波 — Solà 2017 ESKF for Phase 2
  - 直接最小二乘椭圆拟合 — Fitzgibbon 1999 for Phase 2 pupil
  - TSF WiFi 时间同步 — 802.11-2016 TSF for ESP32
- Created ICM-42688-P component datasheet in F:\Projects\PCB\Library\Candidate\
- Updated 知识总览 with 2 new sections: 姿态解算&传感器融合, 导航&定位
- All knowledge pages cross-linked to source summaries in 来源/

## [2026-06-24] ingest | 批量摄入 6 篇论文 + 1 份数据手册

- Ingested and archived 7 source documents to _llm/raw/sources/:
  - Skog 2010 — Zero-Velocity Detection: An Algorithm Evaluation (GLRT framework, 4 detectors → angular rate energy best)
  - Wagstaff 2017 — IPIN: Real-time motion classification ZUPT (SVM 6-class, adaptive thresholds, 5.9 km)
  - Solà 2017 — Quaternion kinematics for ESKF (Phase 2 EKF理论)
  - Fitzgibbon 1999 — Direct least squares ellipse fitting (广义特征值法, Phase 2瞳孔)
  - Sun 2014 — Astronaut ZUPT multi-locomotion (行星表面实验, 1.5-2.3%误差)
  - ICM-42688-P Datasheet — 陀螺2.8mdps/√Hz, 加计65µg/√Hz, SPI 24MHz
- Created 7 source summaries in 来源/
- Updated 全身动捕与头显系统 参见 section with all new sources
- Clippings now contains only 802.11-2016 (to keep as reference) + 2 legacy .md files

## [2026-06-24] ingest | Madgwick AHRS 姿态解算滤波器

- Full paper read: 32 pages, all 7 sections + 2 code appendices
- Archived PDF: _llm/raw/sources/Madgwick 2010 - An efficient orientation filter for inertial and inertial magnetic sensor arrays.pdf
- Created source summary: 来源/2026-06-24 - Madgwick AHRS 姿态解算滤波器
- Core algorithm: gradient-descent quaternion fusion — q_dot_est = q_dot_ω - β·q_hat_∇
- Key specs: 109 (IMU) / 277 (MARG) scalar ops per update, <0.6° static RMS, works at 10 Hz
- Single parameter β = √(3/4)·ω̃_err, physically meaningful (gyro noise), not empirical
- Includes online magnetic distortion compensation + gyroscope bias drift tracking
- Directly applicable to ESP32 MadgwickAHRSupdate() in project firmware
- Updated 全身动捕与头显系统 参见 section

## [2026-06-24] maintain | 剪藏清理 + 802.11 摄入

- Removed duplicate KeyPose PDF from 剪藏 (already in _llm/raw/sources/)
- Deleted ingested clippings: KeyPose CSDN 阅读笔记 (→ 2026-06-18 source), DeepSeek.md (→ 本科项目立项方法论)
- Ingested 802.11-2016: analyzed TSF (Clause 11.1, pp1581-1599), created source summary 2026-06-24 - IEEE 802.11-2016 TSF 时间同步标准
- Updated 全身动捕与头显系统 参见 section with TSF source link
- Clippings now: 802.11-2016.pdf, Obsidian Web Clipper 菜鸟教程.md, 提示词工程完全指南.md
- Principle: 剪藏 is staging; ingested items are summarized in 来源, raw PDFs move to _llm/ only after ingestion

## [2026-06-21] update | 全身动捕与头显系统两阶段重规划

- Restructured to Phase 1 (VRChat-playable) + Phase 2 (paper-quality) roadmap
- Phase 1: 13-node IMU + dual-screen HMD + SlimeVR protocol → SteamVR, ¥1220, solo
- Phase 2: add ZUPT, BVH export, edge eye tracking, optical calibration, hybrid fingers, ¥1630 total
- Moved software stack to dedicated section after hardware
- Added experiment design, timeline (3-year gantt), and team strategy
- Created new knowledge page: 本科项目立项方法论 (from DeepSeek clipping)
- Discussion: identified 6 competitors (SlimeVR, Mesquite MoCap, TU Darmstadt, etc.), confirmed gap exists
- Accuracy projection: joint angle 3-8° pure IMU, 2-5° with optical reference

## [2026-06-07] init | Initialize LLM Wiki infrastructure

- Created directory structure: `raw/sources/`, `raw/assets/`, `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`
- Created schema document: CLAUDE.md
- Created wiki index: 主页
- Created LLM Wiki方法论 concept page
- Based on the methodology from LLM Wiki

## [2026-06-07] ingest | Obsidian Web Clipper 菜鸟教程

- Ingested source: Clippings/Obsidian Web Clipper 菜鸟教程 → 2026-06-07 - Obsidian Web Clipper 菜鸟教程
- Created entity: Obsidian Web Clipper
- Updated concept: LLM Wiki方法论 — added Recommended Sourcing Tools section
- Downloaded 6 local images to `raw/assets/`
- Updated 主页 with new entity and source entries

## [2026-06-07] ingest | Vishay SiC46x microBUCK DC-DC Datasheet

- Ingested source: raw/datasheet/Chip_DCDC_SiC46x_Family.PDF → 2026-06-07 - Vishay SiC46x microBUCK DC-DC Datasheet
- Created entity: SiC46x
- Created concept: DC-DC降压转换器
- Created concept: 恒定导通时间控制
- Updated 主页 — added 1 entity, 2 concepts, 1 source

## [2026-06-07] maintain | Naming unification on Datasheet target

- `raw/datasheet/` replaced with `raw/Datasheet.lnk` → `F:\Projects\PCB\Library\Datasheet\`
- Re-applied naming fixes on actual target directory (12 files: `.pdf`→`.PDF`, `tps`→`TPS`, `usblc6`→`USBLC6`, `chip_`→`Chip_`, `DC-DC`→`DCDC`)
- Removed duplicate `Chip_IC_CurrentSenseAmplifier.PDF`
- Updated CLAUDE.md — added path mapping note and datasheet naming convention

## [2026-06-07] ingest | Datasheet Collection (71 datasheets)

- Analyzed all 71 datasheets in `F:\Projects\PCB\Library\Datasheet\`
- Created master index: Datasheet Inventory — full listing by category
- Created batch source: 2026-06-07 - Datasheet Collection
- Created 5 category concept pages:
  - 电源管理IC — DCDC, LDO, LED driver
  - 被动元件 — R, C, L, ferrite, fuse, PTC
  - 连接器 — USB, FPC, HDMI, RF, audio, power
  - 逻辑与接口IC — MCU, memory, interface, logic
  - 分立半导体 — MOSFET, BJT, TVS, ESD
- Created 10 entity pages for key components:
  - STM32G431, ESP32-C6, ESP8266EX, CH340, W25Q64
  - TJA1044, BQ25758, TPS54302, MAX706, SN74LVC1G14, A4950
- Updated 主页 — added 11 entities, 5 concepts, 1 source, 1 reference

## [2026-06-07] ingest | Individual component pages (batch 2)

- Created **58 additional entity pages** to cover every component:
  - Power: TPS55288, TPS65130, TPS65150, AP3012, TLV62569, LP5907, LM5050, SGM3752
  - Discrete: BSC028N06NS, AO3401A, MMBT2222A, SMBJ Series, SMCJ Series, SMAJ Series, SMF Series, ESD3V3, ESD_CANFD, USBLC6-2SC6
  - Passives (13): resistors, capacitors, inductors, ferrite beads, fuses, PTC
  - Connectors (13): USB-C, FPC, wafer, HDMI, RF, audio, power, B2B
  - Crystals (3): 32.768kHz, 8MHz, 40MHz
  - ICs: AT24C256, HUSB320, LT7911D, TC358870, Current Sense Amp, SN74AHC1G08, PCM5102A
  - Audio/Display: LCD Display, ICS-43434
  - Misc: Red LED R0603, MLT-5020 Buzzer, Tactile Switch 2P, MK-12C02 Slide
- Restructured 主页 — entities grouped by category (71 total)

## [2026-06-07] maintain | 全部 Wiki 页翻译为中文

- Translated all 73 entity pages: headings → 中文, field labels → 中文, feature descriptions → 中文
- Translated 5 concept pages: 电源管理IC, 被动元件, 连接器, 逻辑与接口IC, 分立半导体
- Translated Datasheet Inventory — master listing now in Chinese
- Batch-processed 1000+ sed replacements across all datasheet-related wiki pages

## [2026-06-12] project | 便携储能电源项目总结

- Created project page: 项目 便携储能电源 — 完整的电源架构、电池配置、输出通道、保护链路和监控体系梳理
- Documented power tree topology: 3S3P (internal) + 3S8P (replaceable), dual USB PD charging, 5 regulated outputs
- Analyzed hardware watchdog chain: MAX706 → LM5050 → MOS, software-independent safety cutoff
- Identified missing datasheets: INA226, BSC014N06NS
- Reviewed PCB Library structure, added Candidate/ directory, cleaned stale Allegro files
- Analyzed BCM6135 (Vicor CM-ChiP Bus Converter) — Not Recommended for New Designs
- Updated 主页 — added Projects section

## [2026-06-12] ingest | 提示词工程完全指南

- Ingested source: Clippings/提示词工程完全指南 → 2026-06-12 - 提示词工程完全指南
- Created concept: 提示词工程 — CO-STAR 框架、五段式模板、六大策略、安全防御
- Created concept: 链式思考 CoT — CoT→ToT→GoT→Agent 演化路径
- Key insights captured:
  - Prompt 是 AGI 时代的"编程语言"
  - 如果 prompt 不起作用，微调成功的可能性就很低
  - XML 标签是结构化 prompt 的最佳实践
  - Agent = 更复杂的提示工程 + 工具调用 + 工作流
- Updated 主页 — added 2 concepts, 1 source

## [2026-06-12] maintain | 重构为用户友好的中文目录

- Restructured vault from `wiki/entities/ concepts/ sources/` to `元件/ 知识/ 来源/ 项目/ 参考/`
- Moved 73 entities → `元件/`, 10 concepts → `知识/`, 4 sources → `来源/`
- Project page → `项目/便携储能电源.md`
- `wiki/index.md` → `主页.md`, moved `log.md` to vault root
- `Datasheet Inventory.md` → `参考/元件总表.md`
- Moved Web Clipper tutorial from `raw/` to `Clippings/`
- Updated CLAUDE.md to match new structure
- Rewrote 主页 with clean Chinese layout

## [2026-06-12] maintain | 分离 LLM 与用户工作区

- Moved `raw/` → `_llm/raw/` — LLM workspace now nested in `_llm/` folder
- Human view: `项目/ 元件/ 知识/ 来源/ 参考/ 剪藏/`
- LLM view: `_llm/raw/` (sources, assets, datasheet link)
- Shared: `剪藏/` — both human and LLM access
- Updated CLAUDE.md paths

## [2026-06-12] project | 全项目扫描与总结

- Opened `F:\Projects\` — scanned C/, PCB/, Golang/, CAD/
- Created 项目总览 — master project index (11 projects)
- Created 9 project pages:
  - PCB: PD100W 3S BMS, F1C200S 核心板, RK3568 学习板, 超级电容控制器
  - Embedded: RM2026 机甲大师, STM32 标准模板
  - Go tools: PCB-CIS 管理器, AutoLogin, AliEcsSwitch, VocabLock
- Linked PD100W 3S BMS ↔ 便携储能电源
- Updated 主页 with project highlights

## [2026-06-12] maintain | 按 GitHub 分类重新整理项目总览

- Fixed 项目总览 — now organized by GitHub classification (Public/Fork/Private)
- Discovered 5 missed projects: IoT-traffic-lights, STM32-Rescue-Support-vehicle, LearnGit, STM32G431-DevBoard, TC358870_Demo
- Updated structure: C → GitHub(Public/Fork) + School + Learn; PCB → GitHub(Public/Fork) + Library; Golang → GitHub(Private) + Learn

## [2026-06-18] ingest | LS029B3SX01 数据手册 + KeyPose 论文

- Ingested LS029B3SX01 datasheet from F:\Projects\PCB\Library\Candidate
  - Created component page: LS029B3SX01 (CGS LCD, 2.89寸, 1440×1440, MIPI 双端口, 90Hz)
  - Created source summary: 2026-06-18 - LS029B3SX01 数据手册
  - CRITICAL CORRECTION: 项目页之前误标为 0.29寸/1280×720/Micro OLED
    - Actual: 2.89寸 (73.4mm diagonal), 1440×1440 (square), CG-Silicon TFT LCD
    - Updated 全身动捕与头显系统: screen spec corrected (0.29/1280/Micro OLED → 2.89/1440/CGS LCD)
- Ingested KeyPose paper (1912.02805v2, CVPR 2020) + CSDN reading notes
  - Created source summary: 2026-06-18 - KeyPose 透明物体立体关键点估计
  - Created knowledge page: 立体视觉与关键点估计
  - Linked to 全身动捕与头显系统 project (optical reference methodology)
- Updated indexes: 音频显示.md, 知识总览.md

## [2026-06-18] deep-read | KeyPose 论文全文精读

- OCR server 500 error, used full pdftotext (19 pages, 897 lines)
- Archived PDF to _llm/raw/sources/
- Enriched source summary + knowledge page with: architecture, ablation table, training tricks, data augmentation, labeling accuracy

## [2026-06-24] audit | 全身动捕与头显系统

基于知识库全部关联数据（元件手册、理论知识、来源论文、方法论）对项目进行系统性交叉审核。审核覆盖：器件选型（TC358870/LS029B3SX01/ICM-42688-P/ESP32-S3/STM32G431/OV2640）、架构（TSF同步/Madgwick-ESKF数据流/异步传感器融合/SlimeVR适配）、算法（ZUPT/EKF/椭圆拟合/光学纠偏/面部追踪）、方法论（轻启动验证/范围/团队/时间线）、成本。识别 4 个致命问题（轻启动验证违反、磁力计空缺、ESKF数据流模糊、异步数据包未处理）、5 个重要问题（WiFi版本描述错误、TSF双频策略缺失、ZUPT非步态覆盖、RANSAC遗漏、精度预期过于乐观）、7 个亮点。方法论自评 68/100。审核报告保存为独立文件（审核-全身动捕与头显系统），后融合入项目主线。
## [2026-06-24] audit-revise | 全身动捕与头显系统 — 目标校准

明确项目定位为 maker 工程（低成本 VRChat），修正评审框架。CAN 集群重构后 9 项问题中 8 项已解决，余 1 项（异步传感器融合）低风险。结论：¥1381、两种 PCB、两套固件、单人可维护，技术选型扎实，无阻塞性漏洞。审核报告已同步更新。
## [2026-06-24] merge | 审核结论融合到项目主线

审核文件已删除，结论融合到 全身动捕与头显系统.md 新增的「设计决策记录」章节。涵盖：CAN 集群决策、QMC5883P 选型、WiFi 版本说明、IP 策略、Phase 2 学术定位（79/100）、风险缓解。
## [2026-06-24] lint | 修复9项数据不一致

修复内容：创建 STM32F103C8T6 元件页、修正成本表（去重复磁力计行 + 从节点求和）、更新 ESP32-S3/ICM-42688-P/SlimeVR 知识页用量描述、主页新增活跃项目和知识分类、修正 STM32G4 待定/已入库矛盾、参见补充 Sun ZUPT 来源、移除 log.md 中 wikilink、更新 MCU 无线索引表。
## [2026-06-24] lint | 修复5项新发现问题

修复：概要图表 6 处旧架构术语→CAN 集群、Phase 1 数据流 F_DATA 悬空节点、log.md wikilink（二次确认已清除）、OV2640 补充 wikilink、frontmatter updated 日期更新。
## [2026-06-24] audit | Phase 2 学术定位重评估

第三次交叉审核后，按 本科项目立项方法论 量化打分对 Phase 2 学术定位重新评估。当前 62/100（立项依据 18/30、研究内容与方法 22/40、工作基础 22/30），潜力 83/100。三个主要降分项：创新点待 Phase 1 开发中浮现（非能力问题）、真值采集依赖导师准入（时机问题）、嘴部摄像头未列入硬件清单（小修）。核心优势：CAN 集群架构独特、全栈开源、¥1366 可复现。结论——工程底子扎实，论文思路需等 Phase 1 跑通后精准定义创新点，ablation 实验是最低成本的学术产出路径。更新了项目页 Phase 2 学术定位节。
## [2026-06-24] design | 光学参考方案深化

讨论 inside-out vs outside-in 坐标漂移问题。确认 outside-in 方案天然规避世界坐标系累积误差——摄像头固定，T-pose 标定后坐标系锁定不漂。学长反馈商业方案统一用红外+视觉 XY，确认 IR 升级路径：USB 摄像头去 IR 滤片 + 850nm IR LED 标记点，与 OV2640 眼追完全相同的处理方式，一套方案两处复用。Phase 1 可见光跑通算法，Phase 2 切红外提升稳定性。更新了项目页光学参考章节。
## [2026-06-24] correction | PCB 数量从 2 种修正为 4 种

原描述"仅 2 种 PCB"不准确。实际为 4 种：通用主节点（ESP32-S3 网关）、主手部（+5 路 ADC 拉线直读）、独立从节点（STM32 传感器采集）、头显与眼捕（STM32G431 集线器 + ESP32-S3 眼追）。修正了项目页总述、节点章节、成本表和设计决策记录共 4 处。
## [2026-06-26] concept | HDMI 物理层知识页

- Created knowledge page: HDMI 物理层 — 基于 HDMI 1.4b 规范的完整物理层梳理
  - 五种连接器类型（A/B/C/D/E）对比、Type A 19 引脚定义
  - Cat1/Cat2 线缆分类、TMDS 电气特性（Source/Sink 直流+交流参数）
  - 信号完整性关键参数：100 Ω 差分阻抗、等长匹配、布线规范
  - DDC I2C 通道、CEC 单线总线、HPD 检测协议
  - 鲁棒性要求（短路/对连接容错）
  - PCB 设计要点：4 层板阻抗计算、ESD 保护、布局约束、常见陷阱
- Updated 知识总览 — added 视频显示 category

## [2026-06-26] concept | HDMI 视频传输知识页

- Created knowledge page: HDMI 视频传输 — 基于 HDMI 1.4b 规范 Section 6 + Section 8 的视频传输层完整梳理
  - 视频格式概述、Source/Sink 最低支持要求、跨接口一致性原则
  - 三种像素编码（RGB 4:4:4、YCbCr 4:4:4、YCbCr 4:2:2）及支持要求表
  - Deep Color 模式：30/36/48-bit，TMDS 时钟倍率（1.25x/1.5x/2x），36-bit 强制要求
  - 像素重复：<25MHz 触发条件、720x480i/576i 强制、PR 字段表
  - 视频时序：Primary/Secondary 格式、像素时钟值、高帧率扩展
  - 色彩空间：sRGB/BT.601/BT.709 默认，xvYCC/sYCC/AdobeRGB 扩展，Limited vs Full Range
  - 3D 视频结构：Frame Packing、Side-by-Side、Top-and-Bottom 等 8 种结构
  - 4Kx2K 格式：3840x2160@24/25/30Hz、TMDS 时钟 297MHz 限
  - AVI InfoFrame：完整字段定义表（Y/C/M/VIC/PR/EC/Q/CN+ITC）
- Updated 知识总览 — 视频显示 section 补充 HDMI 视频传输条目

## [2026-06-26] concept | HDMI TMDS 编码知识页

- Created knowledge page: HDMI TMDS 编码 — 基于 HDMI 1.4b 规范 Section 5 的链路层编码详解
  - TMDS 链路架构：3 数据通道 + 1 时钟，Channel 0/1/2 分配
  - 三种操作模式：Video Data、Data Island、Control Period 的切换时序
  - Control Period：Preamble 前导码（8 字符）、数据类型指示
  - Video Data 8b→10b 编码：XOR/XNOR 最小跳变选择、Disparity DC 平衡算法
  - TERC4 编码：4b→10b 映射表、Data Island 抗误码特性
  - Data Island 包结构：包类型表、Header/Body/ECC 格式、InfoFrame 分类
  - Guard Band 机制：Leading/Trailing 保护带、周期边界界定
  - 编码完整时序流程：帧内 Control→Video→Control→Data Island→Control 迭代
- Updated 知识总览 — 视频显示 section 补充 HDMI TMDS 编码条目

## [2026-06-26] concept | HDMI EDID/DDC 知识页

- Created knowledge page: HDMI EDID — 基于 HDMI 1.4b 规范 Section 8 的配置和状态通道详解
  - DDC 通道：I²C 总线分配 (EDID 0xA0/HDCP 0x74/DDC-CI 0x6E)、5V 电平、100/400kHz
  - E-EDID 基础块 (128 bytes)：Header、DTD 解析、扩展块计数、校验和
  - CEA Extension 扩展块：SVD、SAD、Speaker Allocation、VSDB 数据块格式
  - HDMI VSDB 详解：CEC 物理地址、Max_TMDS_Clock、3D/HDMI 1.4 新增特性
  - InfoFrame 系统：AVI/Audio/SPD/MPEG/Vendor Specific 五种类型
  - HPD 热插拔检测：电平阈值、HPD 脉冲 (>100ms)、EDID 重读时序
  - CEC 物理地址路由：Root 0.0.0.0、Repeater 端口分配
  - 自动唇音同步：EDID 延迟字段声明、Source 音频偏移调整
  - 源选择配置流程：HPD→EDID→HDCP→分析→InfoFrame→TMDS
  - 常见问题排查表、PCB 调试要点
- Updated 知识总览 — added HDMI EDID to 视频显示 section

## [2026-06-26] ingest | HDMI 1.4 标准 PDF 知识化

- 从 `剪藏/HDMI1.4标准.pdf` (425 页官方规范) 提取全文并制作 5 页详细知识页：
  - 视频显示/HDMI 协议概述 — Source/Sink 架构、版本演进 (1.0→1.4)、HDMI 1.4 新特性 (Type D/E, ARC, HEC, 3D, 4K)
  - 视频显示/HDMI 物理层 — 五种连接器、Type A 19 引脚、Cat1/Cat2 线缆、TMDS 电气特性、PCB 设计要点
  - 视频显示/HDMI TMDS 编码 — 8b→10b 转换最小化、TERC4、三种操作周期、Data Island 包、Disparity DC 平衡
  - 视频显示/HDMI 视频传输 — 像素编码、Deep Color、像素重复、视频时序、色彩空间、3D/4K 格式、AVI InfoFrame
  - 视频显示/HDMI EDID — DDC I²C 协议、E-EDID 128-byte 结构、HDMI VSDB、InfoFrame 系统、HPD、唇音同步
- Created source summary: 来源/2026-06-26 - HDMI 1.4 Specification
- Updated HDMI Type A and Type D component pages — added source reference
- Updated 知识总览 — added 视频显示 category with 5 HDMI entries
- Updated 主页 — added 视频显示 row to knowledge table
