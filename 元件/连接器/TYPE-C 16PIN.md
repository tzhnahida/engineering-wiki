---
type: entity
tags: [electronics, connector, usb, type-c]
created: 2026-06-07
updated: 2026-06-17
sources: 
---
# TYPE-C 16PIN
# TYPE-C 16PIN

## 1. 身份选型
USB Type-C receptacle, 16-pin (non-full-featured), SMT. Standard 24-pin Type-C with 8 pins omitted (typically CC1, CC2, SBU1, SBU2, and some GND).

## 2. 极限工况
- **额定电压**: 20V (USB PD capable)
- **额定电流**: 3A (standard), 5A (if marked e-marked)
- **插入寿命**: ≥ 10,000 cycles
- **工作温度**: −30°C to +85°C

## 3. 推荐条件
- **配合**: Reversible plug — no orientation required
- **焊接**: Reflow soldering; high-temp plastic rated for lead-free profiles
- **ESD**: Recommend TVS on CC and VBUS lines

## 4. 电气特性
- **耐压**: 500V AC (terminal-to-terminal)
- **接触电阻**: ≤ 30mΩ initial
- **绝缘电阻**: ≥ 100MΩ @ 500V DC
- **电流能力**: 3A per VBUS pin (multiple VBUS pins in parallel)

## 5. 机械特性
- **尺寸**: — mm (L×W×H — typical Type-C receptacle mid-mount)
- **插入力**: 5–20N
- **拔出力**: 8–20N

## 6. 用途
USB 2.0/3.x data + power delivery, device charging, peripheral connection.

## 7. 引脚/触点
16 pins — including VBUS (×2), GND (×2), D+/D−, CC1, CC2, SBU1, SBU2 (if populated), plus shield GND. Contact material: copper alloy with selective gold plating.

## 8. 封装/安装
- **封装**: Mid-mount SMT + through-hole anchor legs (for mechanical retention)
- **安装**: SMT reflow + THT anchor soldering
- **卷装**: Tape & reel or tray
