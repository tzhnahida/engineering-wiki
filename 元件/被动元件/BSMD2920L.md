---
type: entity
tags: [electronics, ptc, resettable-fuse, overcurrent-protection]
created: 2026-06-07
updated: 2026-06-17
sources: 
---
# BSMD2920L
# BSMD2920L

## 1. 身份选型
SMD PTC resettable fuse (PPTC), 1A hold current / 24V max, 2920 package.

## 2. 极限工况
- **最大电压**: 24V
- **最大故障电流**: — A
- **工作温度**: −40°C to +85°C
- **动作温度**: Device trips when internal temp reaches ~125°C (self-heating)

## 3. 推荐条件
- **降额**: Operating current ≤ 80% of hold current at ambient 25°C
- **散热**: Good PCB copper area for heatsinking
- **恢复**: After trip, remove power / reduce current for device to reset

## 4. 电气特性
- **保持电流 (Ihold) @ 25°C**: 1A
- **动作电流 (Itrip) @ 25°C**: — A (typ. 2× Ihold)
- **最大电压 (Vmax)**: 24V
- **初始电阻 (Rmin–Rmax)**: — mΩ
- **动作后电阻**: — × initial (increased after trip)

## 5. 机械特性
- **尺寸**: 2920 (7.4×5.1mm)
- **端子**: Ni/Sn plated

## 6. 用途
Resettable overcurrent protection — USB ports, battery circuits, general PSU protection.

## 7. 引脚/触点
2-terminal SMD.

## 8. 封装/安装
- **封装**: 2920 (7.4×5.1mm)
- **安装**: SMT reflow soldering
- **卷装**: Tape & reel
