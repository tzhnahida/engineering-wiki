---
type: source
tags: [无人机, 飞控, 官方文档]
created: 2026-08-25
updated: 2026-08-25
---

# 2026-08-25 - ArduPilot 官方文档与开发指南

> [!note] 来源性质
> ArduPilot 官方文档站(ardupilot.org)的开发指南(dev docs)与各载具用户文档(copter/plane/rover/sub/blimp),以及 GitHub 仓库中的 BUILD.md 等工程文件。全部为在线网页,无 PDF。抓取日期 2026-08-25。

## 覆盖范围

| 主题 | 关键页面 |
|------|----------|
| 代码组织与架构 | `learning-ardupilot-introduction`、`learning-the-ardupilot-codebase`、`apmcopter-programming-libraries` |
| 线程与调度 | `learning-ardupilot-threading` |
| EKF 估计系统 | `extended-kalman-filter`、`ekf2-estimation-system`、`common-ek3-affinity-lane-switching` |
| MAVLink 接口 | `mavlink-basics`、`mavlink-commands` |
| 构建系统 | `building-the-code` + 仓库根 `BUILD.md` |
| 历史沿革 | `common-history-of-ardupilot` |
| 许可证 | `license-gplv3` |
| SITL 仿真 | `sitl-simulator-software-in-the-loop` |
| 参数系统 | `code-overview-adding-a-new-parameter` |
| 硬件选型 | `common-autopilots`、`common-limited-firmware` |
| 载具介绍 | Copter/Plane/Rover/Sub/Blimp 各自 introduction |
| 地面站 | `common-choosing-a-ground-station`、`common-antenna-tracking` |
| Lua 脚本 | `common-lua-scripts` |

## 关键事实摘录

- **代码规模**:核心 git 树约 70 万行代码(官方文档原话)。
- **五大组成部分**:载具代码(vehicle code)、共享库(shared libraries)、AP_HAL、Tools 目录、外部支持代码(ChibiOS/DroneCAN/mavlink 以 submodule 导入)。
- **EKF2 状态模型**:24 状态 —— 姿态四元数、速度 NED、位置 NED、陀螺偏置 XYZ、陀螺尺度因子 XYZ、Z 轴加速度计偏置、地磁场 NED、机体系磁场 XYZ、风速 NE。
- **Copter 飞行模式**:25 种,其中 10 种常用。
- **Lua 引擎**:Lua 5.3.5,沙箱化,低优先级时间片执行;F4 板不支持脚本。
- **GPLv3 商业路径**:官方明确允许厂商把闭源代码放 companion computer,通过 MAVLink 与飞控通信。
- **2016 事件**:官方历史页原文 "DroneCode Platinum board members outvote Silver board members to remove GPLv3 projects including ArduPilot from DroneCode"。

## 存疑项(转述研究时的标注)

1. 线程文档中的 `ps` 输出示例为 PX4/NuttX 时代遗留,与 ChibiOS 主线不完全对应;
2. Blimp 飞行模式清单来自搜索摘要,未经逐页核对;
3. 开发页称参数类型"不支持无符号整数",与现行代码(已有 `AP_Uint8`)不符,以源码为准;
4. Linux 板以用户态进程运行(AP_HAL_Linux),故"无 Linux 用户态"的说法不成立。

## 原文链接

- https://ardupilot.org/dev/docs/learning-ardupilot-introduction.html
- https://ardupilot.org/dev/docs/learning-the-ardupilot-codebase.html
- https://ardupilot.org/dev/docs/learning-ardupilot-threading.html
- https://ardupilot.org/dev/docs/extended-kalman-filter.html
- https://ardupilot.org/dev/docs/ekf2-estimation-system.html
- https://ardupilot.org/dev/docs/common-ek3-affinity-lane-switching.html
- https://ardupilot.org/dev/docs/mavlink-basics.html
- https://ardupilot.org/dev/docs/mavlink-commands.html
- https://ardupilot.org/dev/docs/building-the-code.html
- https://ardupilot.org/dev/docs/common-history-of-ardupilot.html
- https://ardupilot.org/dev/docs/license-gplv3.html
- https://ardupilot.org/dev/docs/sitl-simulator-software-in-the-loop.html
- https://ardupilot.org/copter/docs/common-apm-navigation-extended-kalman-filter-overview.html
- https://ardupilot.org/copter/docs/flight-modes.html
- https://ardupilot.org/copter/docs/common-lua-scripts.html
- https://ardupilot.org/copter/docs/common-autopilots.html
- https://ardupilot.org/copter/docs/common-limited-firmware.html
- https://ardupilot.org/copter/docs/common-choosing-a-ground-station.html
- https://ardupilot.org/plane/docs/introduction.html
- https://ardupilot.org/sub/docs/introduction.html
- https://ardupilot.org/blimp/docs/getting-started.html
- https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md
