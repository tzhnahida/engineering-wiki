---
type: concept
tags: [hdmi, video, deep-color, 4k, 3d, pixel-encoding]
created: 2026-06-26
updated: 2026-07-15
sources: ["[2026-06-26 - HDMI 1.4 Specification](../../来源/2026-06-26 - HDMI 1.4 Specification.md)"]
---

# HDMI 视频传输

> HDMI 视频传输定义了像素编码格式、色彩空间、色深、时序格式以及 InfoFrame 元数据的完整规范。本节内容基于 HDMI 1.4b 规范 Section 6 (Video) 及 Section 8 (InfoFrame)。

## 1. 概述 (Section 6.1)

HDMI 可传输任意视频格式的时序——这一设计原则使 HDMI 能够兼容从标清到超高清的几乎全部视频标准。Source 根据以下因素选择最终的视频输出格式：

- **源视频特性**：原始内容的分辨率、帧率、色彩格式
- **Source 转换能力**：Source 是否具备格式转换（如 RGB ↔ YCbCr、4:4:4 ↔ 4:2:2）能力
- **Sink 的 EDID 能力**：Sink 通过 [视频显示/HDMI EDID](../../视频显示/HDMI EDID.md) 声明其支持的格式列表

HDMI 定义了三种像素编码格式可供选择：

| 编码格式 | 描述 | 典型场景 |
|---------|------|---------|
| RGB 4:4:4 | 红绿蓝各 8/10/12/16 bit，无亚采样 | PC/图形内容 |
| YCbCr 4:4:4 | 亮度 + 色差全分辨率 | 专业视频 |
| YCbCr 4:2:2 | 亮度全分辨率，色差水平半采样 | 消费视频（主流） |

Source 通过 AVI InfoFrame 的 Y 字段告知 Sink 当前传输的编码格式。

## 2. 视频格式支持要求 (Section 6.2)

HDMI 规范对 Source 和 Sink 的格式支持提出了最低要求，确保不同设备间的基本互操作性。

### Source 最低要求

Source 必须至少支持以下格式之一：

- 640×480p @ 59.94/60Hz（VGA 分辨率）
- 720×480p @ 59.94/60Hz（NTSC 逐行）
- 720×576p @ 50Hz（PAL 逐行）

### Sink 最低要求

Sink 的强制格式取决于其目标市场：

| Sink 类型 | 强制支持格式 |
|-----------|------------|
| 60Hz Sink | 640×480p @ 60Hz + 720×480p @ 60Hz |
| 50Hz Sink | 640×480p @ 60Hz + 720×576p @ 50Hz |
| HDTV Sink | 额外支持 1280×720p 或 1920×1080i（根据目标市场选择 50Hz 或 60Hz 变体） |

### 跨接口一致性

HDMI 规范要求：如果 Source 在某个其他模拟或数字视频输出接口（如 VGA、DVI、DisplayPort）上支持某一视频格式（来自规范指定的列表），则它**必须**在 HDMI 上也支持同一格式。这一要求保证了跨接口的信号一致性和用户体验统一。

## 3. 像素编码要求 (Section 6.2.3)

[📷 _llm/raw/assets/standards/hdmi14/hdmi14_p124_fig1.jpg|560]
*Figure 6-1 — 默认像素编码 RGB 4:4:4 / 8bit：三通道各承载一个分量*


三种编码格式的支持要求如下表：

| 编码 | Source 要求 | Sink 要求 |
|------|------------|----------|
| RGB 4:4:4 | **必须支持** | **必须支持** |
| YCbCr 4:4:4 | 如果 Source 支持色差(CbCr)视频输出则**必须**支持 | 如果 Sink 支持色差视频输入则**必须**支持；且必须同时支持 4:2:2 |
| YCbCr 4:2:2 | 同上（与 4:4:4 绑定） | 同上（与 4:4:4 绑定） |

> [!note] 色差编码的绑定关系
> 如果设备声明支持 YCbCr 色差格式，则必须同时支持 4:4:4 和 4:2:2 两种亚采样模式。仅支持其中一种而排除另一种不符合规范。

## 4. 色深 —— Color Depth (Section 6.2.4, 6.5)

### 默认色深

所有 HDMI 设备必须支持 **24-bit 颜色**（8 bits/component，即 8bpc）。这是 HDMI 的基本色深模式，在 24-bit 下：

- TMDS 时钟频率 = 像素时钟频率
- 每像素时钟周期传输 24 bits（三个 TMDS 通道 × 10 bits 编码后 = 30 bits，其中 24 bits 有效数据）

### Deep Color 模式

Deep Color 使用更高的每分量位数来减少色阶条纹（banding），提高色彩精度：

| 模式 | 每分量位数 | TMDS 时钟倍率 | 总颜色数 |
|------|-----------|--------------|---------|
| 24-bit（默认） | 8 bpc | 1× pixel clock | 16.7 M |
| 30-bit (Deep Color) | 10 bpc | 1.25× pixel clock | 1.07 B |
| 36-bit (Deep Color) | 12 bpc | 1.5× pixel clock | 68.7 B |
| 48-bit (Deep Color) | 16 bpc | 2× pixel clock | 281 T |

Deep Color 全部为可选支持，但有一个重要约束：

> **如果设备支持任何 Deep Color 模式，则**必须**支持 36-bit（12 bpc）模式。** 这一要求确保 Source 和 Sink 在 Deep Color 下至少有一个共同的 12 bpc 模式可以回退。

### 色深与 TMDS 时钟的关系

Deep Color 下 TMDS 时钟按比例提高：

- **24-bit** → TMDS clock = **1.0 ×** pixel clock
- **30-bit** → TMDS clock = **1.25 ×** pixel clock
- **36-bit** → TMDS clock = **1.5 ×** pixel clock
- **48-bit** → TMDS clock = **2.0 ×** pixel clock

例如，1080p @ 60Hz 的像素时钟为 148.5 MHz：
- 24-bit → TMDS clock = 148.5 MHz
- 36-bit → TMDS clock = 222.75 MHz
- 48-bit → TMDS clock = 297 MHz（接近 Cat 2 线缆上限）

### YCbCr 4:2:2 36-bit 的特殊处理

YCbCr 4:2:2 36-bit 模式下，由于 4:2:2 亚采样已经降低了每像素数据量（只需传输两分量而非三分量），因此**不需要启用 Deep Color 机制**。该模式下使用标准的 24-bit 传输管道，色深提升来自亚采样节省的带宽。

## 5. 像素重复 —— Pixel Repetition (Section 6.4)

像素重复是一种向后兼容技术，用于传输像素时钟低于 25 MHz 的标清格式。

### 触发条件

- **像素时钟 < 25 MHz 的格式必须使用像素重复**
- **720×480i 和 720×576i 始终需要像素重复**（无论像素时钟值）

### 实现原理

像素重复将每个像素的数据在 TMDS 链路上传输多次，使 TMDS 时钟保持在一个可接受的范围内。Source 端输出重复像素，Sink 端根据 AVI InfoFrame 指示的重复因子丢弃多余的重复数据。

### 重复因子

像素重复因子通过 AVI InfoFrame 的 `PR3:PR0` 字段指示：

| PR 值 | 重复因子 | 含义 |
|-------|---------|------|
| 0 | 1× | 不重复（正常模式） |
| 1 | 2× | 每个像素传输 2 次 |
| 2 | 3× | 每个像素传输 3 次 |
| ... | ... | ... |
| 10 | 11× | 最大重复（每个像素传输 11 次） |

> [!note] 像素重复与 TMDS 时钟
> 像素重复使 TMDS 时钟升高至 pixel clock × (重复因子 + 1)。以 720×480i 为例，其像素时钟约为 13.5 MHz，重复因子 1（2×）时 TMDS 时钟 = 27 MHz，恰好落在 TMDS 可靠工作的范围内。

## 6. 视频格式时序 —— Video Format Timings (Section 6.3)

HDMI 规范定义了两类视频格式时序：Primary（核心强制时序）和 Secondary（可选扩展时序）。

### Primary Video Format Timings（核心格式）

这些是 HDMI 的强制或广泛支持的格式：

| 格式 | 帧率 (Hz) | 像素时钟 (MHz) | 说明 |
|------|----------|---------------|------|
| 640×480p | 59.94/60 | 25.175 | VGA，通用后备格式 |
| 720×480p | 59.94/60 | 27.0 | NTSC 逐行 DVD |
| 720×576p | 50 | 27.0 | PAL 逐行 DVD |
| 720(1440)×480i | 59.94/60 | 13.5 | NTSC 隔行（需像素重复） |
| 720(1440)×576i | 50 | 13.5 | PAL 隔行（需像素重复） |
| 1280×720p | 50/60 | 74.25 | 720p HD |
| 1920×1080i | 50/60 | 74.25 | 1080i HD |

### Secondary Video Format Timings（HDMI 1.4 新增扩展格式）

HDMI 1.4 大幅扩展了支持的视频格式，主要包括：

**全高清高帧率：**

| 格式 | 帧率 (Hz) | 像素时钟 (MHz) |
|------|----------|---------------|
| 1920×1080p | 24 | 74.25 |
| 1920×1080p | 25 | 74.25 |
| 1920×1080p | 30 | 74.25 |
| 1920×1080p | 50 | 148.5 |
| 1920×1080p | 60 | 148.5 |
| 1920×1080p | 100 | 297 |
| 1920×1080p | 120 | 297 |

**720p 扩展帧率：**

| 格式 | 帧率 (Hz) | 像素时钟 (MHz) |
|------|----------|---------------|
| 1280×720p | 24/25/30 | 74.25 |
| 1280×720p | 100 | 148.5 |
| 1280×720p | 120 | 148.5 |

**Pixel-doubled SD 格式：**

将标清格式的像素时钟加倍，以消除像素重复的兼容模式。

**高帧率 480i/576i：**

| 格式 | 帧率 (Hz) |
|------|----------|
| 720×480i | 120/240 |
| 720×576i | 100/200 |

### 关于像素时钟的基础知识

- 像素时钟 = 水平总像素 × 垂直总行数 × 帧率
- 1080p @ 60Hz：2200 × 1125 × 60 ≈ 148.5 MHz（含 blanking）
- 4K×2K @ 30Hz：5500 × 2812 × 30 ≈ 297 MHz（4K 典型值）

HDMI 1.4 Category 2 线缆的最大 TMDS 时钟为 **340 MHz**，对应 TMDS 单通道 3.4 Gbps，总带宽 10.2 Gbps。

## 7. 色彩空间与色度学 (Section 6.6-6.7)

### 默认色彩空间

HDMI 规范为不同编码格式定义了默认色彩空间：

| 编码 | 默认色彩空间 | 标准 | 量化范围 |
|------|------------|------|---------|
| RGB | sRGB | IEC 61966-2-1 | Limited Range (16-235 for 8-bit) |
| YCbCr (SD) | ITU-R BT.601 | — | 16-235 for 8-bit |
| YCbCr (HD) | ITU-R BT.709 | — | 16-235 for 8-bit |

### 扩展色彩空间

通过 AVI InfoFrame 的 `EC2:EC0` 字段，Source 可以声明使用非默认的色彩空间：

| EC 值 | 色彩空间 | 参考标准 |
|-------|---------|---------|
| 000 | xvYCC601 | IEC 61966-2-4 |
| 001 | xvYCC709 | IEC 61966-2-4 |
| 010 | sYCC601 | — |
| 011 | AdobeYCC601 | — |
| 100 | AdobeRGB | IEC 61966-2-5 |

### 量化范围 (Quantization Range)

量化范围通过 AVI InfoFrame 的 `Q0:Q1`（RGB）和 `YQ0:YQ1`（YCbCr）字段控制：

| 量化范围 | 8-bit 数值范围 | 用途 |
|---------|--------------|------|
| Limited Range（有限范围） | 16–235 | 默认，适用于视频内容 |
| Full Range（全范围） | 0–255 | 适用于 PC/图形内容 |

**Limited Range** 是 HDMI 的默认量化方式，即使 RGB 编码也默认使用 16-235 范围。这是因为 HDMI 脱胎于消费电子视频接口，所有视频内容默认采用 studio swing 量化。如果需要 PC 标准的 Full Range，Source 通过 AVI InfoFrame 显式声明。

> [!warning] RGB Limited vs Full Range 的常见混淆
> 许多 PC 用户发现 HDMI 连接显示器后黑色不纯、白色不够亮，原因正是 Source 输出 RGB Limited Range，而显示器预期 RGB Full Range。这是 HDMI 与 PC 生态之间最常见的色彩兼容性问题之一。解决方案是在驱动设置中手动选择 RGB Full Range / PC Standard。

## 8. 3D 视频格式 (Section 8.2.3, Appendix H)

[📷 _llm/raw/assets/standards/hdmi14/hdmi14_p159_fig1.jpg|540]
*Figure 8-3 — 3D Frame Packing 结构：左右眼帧垂直堆叠于同一「超高帧」内传输*


> HDMI 1.4 新增功能

HDMI 1.4 首次在标准中定义了 3D 视频传输能力，支持多种 3D 格式和传输结构。

### 3D 结构类型

3D 结构类型通过 HDMI VSDB (Vendor Specific Data Block) 的 `3D_Structure` 字段指示：

| 3D 结构 | 描述 | 典型分辨率/帧率 |
|---------|------|----------------|
| Frame Packing（帧封装） | 左眼帧 + 右眼帧垂直叠加，帧率翻倍 | 1080p @ 24Hz, 720p @ 50/60Hz |
| Field Alternative | 隔行交替（左眼奇场，右眼偶场） | 1080i |
| Line Alternative | 逐行交替（左眼奇数行，右眼偶数行） | 720p |
| Side-by-Side (Full) | 左右各半，全宽 | 1080i |
| Side-by-Side (Half) | 左右各半，半宽 | 1080p @ 24Hz |
| Top-and-Bottom | 上下各半，半高 | 720p @ 50/60Hz |
| L + Depth | 左眼 + 深度图 | — |
| L + Depth + Graphics | 左眼 + 深度 + 图形 | — |

### Frame Packing 的带宽需求

Frame Packing 是质量最高的 3D 传输方式，它将左右眼两帧垂直堆叠为一帧传输，因此：

- 1080p @ 24Hz 3D Frame Packing 的 TMDS 时钟 ≈ 2 × 74.25 = **148.5 MHz**
- 720p @ 60Hz 3D Frame Packing 的 TMDS 时钟 ≈ 2 × 74.25 = **148.5 MHz**

Frame Packing 在帧之间插入垂直 blanking 间隔，用于区分左右眼。

### 3D 传输细节

- **VIC 复用**：3D 视频使用与 2D 相同的 VIC（Video Identification Code），通过 HDMI VSDB 的 3D_Structure 字段区分是 2D 还是 3D 传输
- **AVI InfoFrame VIC 字段满足**：3D_VIC = 2D_VIC + (3D_Structure 相关偏移)
- **Sink 能力声明**：Sink 通过 EDID 的 HDMI VSDB 声明其支持的 3D 格式列表

### 3D 与 Deep Color

3D 视频可以与 Deep Color 结合使用，但带宽限制意味着：
- 1080p @ 24Hz 3D Frame Packing + 36-bit Deep Color → TMDS clock = 148.5 MHz × 1.5 ≈ **222.75 MHz**
- 高帧率 3D + Deep Color 的组合受 Category 2 线缆的 340 MHz TMDS 时钟上限约束

## 9. 4K×2K 格式 (Section 8.2.3)

> HDMI 1.4 新增功能

HDMI 1.4 首次引入 4K 超高清分辨率支持，但帧率受限于 Category 2 线缆的带宽上限。

### 支持的 4K 格式

| 格式 | 帧率 | 像素时钟 | TMDS 时钟 | 备注 |
|------|------|---------|-----------|------|
| 3840×2160p | 24 Hz | 297 MHz | 297 MHz | 16:9 UHD |
| 3840×2160p | 25 Hz | 297 MHz | 297 MHz | — |
| 3840×2160p | 30 Hz | 297 MHz | 297 MHz | — |
| 4096×2160p | 24 Hz | 297 MHz | 297 MHz | 影院 4K (DCI) |

### 4K 传输的核心限制

- **最大 TMDS 时钟**：Category 2 线缆的上限为 340 MHz
- **4K @ 30Hz** 已占用 297 MHz，接近上限
- **4K @ 60Hz** 需要约 594 MHz TMDS 时钟（以 24-bit 计算），远超 Cat 2 能力
- HDMI 1.4 的 4K 仅支持 **24/25/30 Hz 低帧率**

> [!warning] HDMI 1.4 的 4K 局限
> 60Hz 4K 需要 HDMI 2.0 及以上版本。HDMI 2.0 通过将 TMDS 时钟提升至 **600 MHz**（单通道 6 Gbps，总带宽 18 Gbps）实现 4K @ 60Hz。HDMI 1.4 的 297 MHz 时钟只能覆盖 4K @ 30Hz 及以下。

### 4K 的像素时钟计算

4K×2K @ 30Hz 的详细时序参数：

- 水平总像素：~5280（含 blanking）
- 垂直总行数：~2250（含 blanking）
- 像素时钟 ≈ 5280 × 2250 × 30 ≈ 356 MHz
- 实际规范值为 **297 MHz**（使用更优化的 blanking 参数）

## 10. AVI InfoFrame (Section 8.2.1)

AVI InfoFrame (Auxiliary Video Information InfoFrame) 是 HDMI 视频传输中最关键的数据包之一。它携带了 Source 的视频格式元数据，Sink 根据这些信息正确解码和显示视频。

### 包结构

| 字段 | 值 |
|------|-----|
| Packet Type | 0x82 |
| Version | 0x02 |
| Length | 13 bytes |

### 关键字段详解

**Y1:Y0 —— 色彩空间指示 (bits 5:4 of Data Byte 1)**

| Y 值 | 色彩空间 |
|------|---------|
| 00 | RGB |
| 01 | YCbCr 4:2:2 |
| 10 | YCbCr 4:4:4 |
| 11 | Future |

**C1:C0 —— 色度学 (bits 3:2 of Data Byte 1)**

指示与原始内容相关的色度信息（如电影胶卷、ITU 601、ITU 709 等）。

**M1:M0 —— 画面比例 (bits 1:0 of Data Byte 1)**

| M 值 | 画面比例 |
|------|---------|
| 00 | 无数据 |
| 01 | 4:3 |
| 10 | 16:9 |

**R3:R0 —— 活跃格式比例 (bits 7:4 of Data Byte 2)**

指示活跃视频区域的比例，用于源端已知而 Sink 端需要解析的比例信息。

**VIC6:VIC0 —— 视频格式识别码 (bits 6:0 of Data Byte 3)**

VIC（Video Identification Code）是 HDMI 中标识视频格式的核心编码。例如：
- VIC 1: 640×480p @ 60Hz
- VIC 4: 1280×720p @ 60Hz
- VIC 16: 1920×1080p @ 60Hz
- VIC 32: 3840×2160p @ 30Hz

> VIC 码与 CEA-861 标准定义的时序格式一一对应。完整的 VIC 码表见 HDMI 规范 Table 8 至 Table 11。

**PR3:PR0 —— 像素重复因子 (bits 3:0 of Data Byte 4)**

指示当前视频流的像素重复倍数，详见第 5 节。

**EC2:EC0 —— 扩展色度学 (bits 6:4 of Data Byte 5)**

声明扩展色彩空间，参见第 7 节 Extended Colorimetry 表。

**Q1:Q0 —— RGB 量化范围 (bits 1:0 of Data Byte 5)**

| Q 值 | 量化范围 |
|------|---------|
| 00 | 默认（由色彩空间决定） |
| 01 | Limited Range (16-235) |
| 10 | Full Range (0-255) |

**YQ1:YQ0 —— YCC 量化范围 (bits 3:2 of Data Byte 5)**

适用于 YCbCr 编码的量化范围，编码方式与 Q 字段相同。

**CN1:CN0 + ITC —— 内容类型 (bits 7:4 of Data Byte 6)**

| ITC | CN1:CN0 | 内容类型 |
|-----|---------|---------|
| 0 | — | 无内容类型 |
| 1 | 00 | Graphics（图形） |
| 1 | 01 | Photo（照片） |
| 1 | 10 | Cinema（电影） |
| 1 | 11 | Game（游戏） |

Sink 可以根据内容类型自动切换画面处理模式——例如游戏模式降低延迟、电影模式平滑插帧、照片模式保持色彩准确。

### AVI InfoFrame 的传输周期

- Source 在视频信号有效期间以**逐帧**方式发送 AVI InfoFrame
- 除非 Source 检测到 Sink 的 EDID 变化（通过 HPD 重新触发），否则仅在视频格式变化时更新 AVI InfoFrame
- Sink 应监控 AVI InfoFrame 的序列号识别更新

---

## See Also

- [视频显示/HDMI 协议概述](../../视频显示/HDMI 协议概述.md) — HDMI 协议栈（数据岛、视频周期、控制周期）
- [视频显示/HDMI 物理层](../../视频显示/HDMI 物理层.md) — 连接器、TMDS 电气特性、PCB 设计要点
- [视频显示/HDMI TMDS 编码](../../视频显示/HDMI TMDS 编码.md) — TMDS 8b10b 编码算法与 DC 平衡
- [视频显示/HDMI EDID](../../视频显示/HDMI EDID.md) — 扩展显示识别数据（EDID 结构与解析）
- [TC358870](../../元件/接口存储/TC358870.md) — HDMI 1.4 到 MIPI DSI 桥接芯片（头显项目核心器件）
