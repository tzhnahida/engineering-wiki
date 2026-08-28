---
type: source
tags: [无人机, 飞控, PX4]
created: 2026-08-25
updated: 2026-08-25
---

# 2026-08-25 - PX4 官方文档与 Dronecode 生态

> [!note] 来源性质
> PX4 官方文档(docs.px4.io)、PX4-Autopilot 仓库源码文档、Dronecode Foundation 官网与年度报告、Auterion 官方页面、2016 年 ArduPilot 退出 Dronecode 的一手声明(tridge 论坛帖)与二手报道。抓取日期 2026-08-25。

## 覆盖范围

| 主题 | 关键来源 |
|------|----------|
| PX4 架构 | `concept/architecture.html`(两层:flight stack + middleware,反应式设计) |
| uORB 消息总线 | `middleware/uorb.html`(发布/订阅、多实例 topic、v1.16 消息版本化) |
| 估计器 | `advanced_config/tuning_the_ecl_ekf.html`(ECL EKF2,参数矩阵) |
| 控制分配 | `concept/control_allocation.html`(v1.13+ 取代 mixer) |
| 载具构型 | `airframes/airframe_reference.html`(MC/FW/VTOL 为主,含实验性 Airship/Autogyro/Balloon/Spacecraft/Underwater) |
| 许可证 | `contribute/licenses.html`(BSD-3 飞控栈;硬件 CC-BY-SA 3.0;文档 CC BY 4.0) |
| 治理 | Dronecode 官网、2020 Contributor Report、2022 Annual Report |

## 关键事实摘录

- **PX4 架构声明**:模块可"被快速替换,甚至在运行时替换";模块以独立 Task 或共享 Work Queue 运行;NuttX 为一级 RTOS(POSIX 兼容是模块化的基石)。
- **贡献集中度**:Auterion(Lorenz Meier 联合创办)占 PX4 代码贡献 56%(2020)/ 52.7%(2022)。
- **2016 分裂**:tridge 公开声明称 Dronecode Platinum 成员(3DR、Intel、Qualcomm)策划了"政变"——将顶级开源项目移出 TSC,并要求交出商标、账号与域名控制权;PX4 接受条件留在 Dronecode,ArduPilot 拒绝并退出。
- **2026 趋势**:PX4 Developer Summit 宣布引入付费维护者,从纯志愿维护转型。
- **安全**:CVE-2026-1579 —— PX4 未启用 MAVLink 签名时 SERIAL_CONTROL 可被未认证调用(VulDB)。
- **Skydio**:⚠️ 未见可靠来源证实其使用 PX4,分析称其飞行栈完全闭源,故不归入任一阵营。
- **NuttX 许可证**:⚠️ PX4 架构页仍称 NuttX 为 BSD,但 NuttX 已于 2019–2022 转入 Apache 基金会并改 Apache-2.0,文档表述过时。

## 原文链接

- https://docs.px4.io/main/en/concept/architecture.html
- https://docs.px4.io/main/en/middleware/uorb.html
- https://docs.px4.io/main/en/advanced_config/tuning_the_ecl_ekf.html
- https://docs.px4.io/main/en/concept/control_allocation.html
- https://docs.px4.io/main/en/airframes/airframe_reference.html
- https://docs.px4.io/main/en/contribute/licenses.html
- https://docs.px4.io/main/en/flight_controller/
- https://github.com/PX4/PX4-Autopilot/blob/main/docs/en/mavlink/message_signing.md
- https://discuss.ardupilot.org/t/ardupilot-and-dronecode/11295
- https://diydrones.com/profiles/blogs/ardopilot-and-dronecode-part-ways
- https://www.linuxfoundation.org/press/press-release/linux-foundation-and-leading-technology-companies-launch-open-source-dronecode-project
- https://www.dronecode.org/wp-content/uploads/sites/24/2021/02/2020-Contributor-Report-%E2%80%94-Dronecode-Foundation.pdf
- https://www.suasnews.com/2023/03/dronecode-annual-report-2022-auterion-powers-majority-of-contributions/
- https://dronecode.org/px4-developer-summit-2026-recap-minneapolis/
- https://auterion.com/contribution_rankings_px4_ecosystem/
- https://en.wikipedia.org/wiki/NuttX
- https://vuldb.com/zh/cve/CVE-2026-1579
- https://www.forbes.com/sites/ryanmac/2016/10/05/3d-robotics-solo-crash-chris-anderson/
