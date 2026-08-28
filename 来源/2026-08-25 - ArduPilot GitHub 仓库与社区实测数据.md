---
type: source
tags: [无人机, 飞控, 数据实测]
created: 2026-08-25
updated: 2026-08-25
---

# 2026-08-25 - ArduPilot GitHub 仓库与社区实测数据

> [!note] 来源性质
> GitHub REST API(`gh api`)与 raw.githubusercontent.com 实测数据,采集于 2026-08-25。未 clone 仓库(约 1GB+),代码行数未精确统计。论坛数据来自 discuss.ardupilot.org 的 about.json。

## 仓库规模(GitHub API 实测)

| 指标 | 数值 |
|------|------|
| Stars | 15,736 |
| Forks | 21,273 |
| Watchers / subscribers | 15,736 / 682 |
| Open issues(不含 PR) | 1,685 |
| Open PRs | 1,464 |
| License | GPL-3.0 |
| 默认分支 | master |
| 仓库创建 | 2013-01-09(自 Google Code 迁移) |
| master 提交总数 | 73,442 |
| Git 仓库体积 | 约 635 MB(size 字段) |
| 贡献者(有 commit 计数) | 312(⚠️ 另一统计口径 1,366,含匿名贡献者等) |
| 近 30 天合入 PR | 169(日均 5.6) |
| 近 30 天新开/关闭 issue | 40 / 20(净增积压) |

## 语言构成(按字节,不含 submodule)

| 语言 | 占比 |
|------|------|
| C++ | 61.1% |
| Python | 13.9% |
| C | 9.8% |
| Objective-C | 7.4%(AP_NavEKF 设计模型生成代码) |
| Lua | 4.7% |
| 其他 | ~3.1% |

## Top 贡献者(按 commit 数)

| 排名 | ID | 提交数 |
|------|-----|--------|
| 1 | peterbarker | 14,105 |
| 2 | tridge(Andrew Tridgell,联合创始人/Plane 维护者) | 12,573 |
| 3 | rmackay9(Randy Mackay,Copter 维护者) | 8,307 |
| 4 | IamPete1 | 2,256 |
| 5 | andyp1per | 1,856 |
| 10 | priseborough(EKF 作者) | 1,196 |

## 目录结构实测

- **根目录**:6 个载具目录(ArduCopter/ArduPlane/Rover/ArduSub/Blimp/AntennaTracker)+ libraries/(154 个库,AP_ 131 个)+ Tools/(autotest、AP_Periph、AP_Bootloader、Replay、CodeStyle 等 26 条目)+ modules/(15 个 submodule:ChibiOS、mavlink、waf、gtest、DroneCAN、lwip、littlefs、Micro-XRCE-DDS 等)+ docs/benchmarks/tests。
- **发布节奏**:Copter-4.5.0(2024-04-02)→ 4.6.0(2025-05-22)→ 4.7.0(2026-07-22),约 13~14 个月一个大版本;每载具独立 tag;大版本后先密集出点版本(4.5.x 曾每月 1 个)。
- **CI**:master 现存 30 个 workflow 文件 —— 每载具独立 SITL 测试、全板卡编译矩阵(test_chibios/test_size/test_linux_sbc/esp32_build 等)、单元/覆盖率/回放/Lua/DDS 测试、pre-commit 与分支规范检查;已启用 GitHub Copilot PR 评审。
- **代码风格**:`Tools/CodeStyle/astylerc` —— style=linux、4 空格缩进、强制大括号、行尾 LF。

## 社区规模(2026-08-25)

| 指标 | 数值 |
|------|------|
| 论坛主题 / 回帖 | 58,879 / 559,897 |
| 论坛注册用户 | 43,789 |
| 论坛 30 天活跃 | 882 |
| 装机量声明 | 超 1,000,000 台(ardupilot.org 首页自述 ⚠️ 厂商口径) |
| 固件构建变体 | Copter stable 636 个 / Plane stable 304 个 |

## 数据局限

- 精确代码行数 ⚠️ 未获取(未 clone);
- issue 中位解决时长 ⚠️ 未获取;
- Actions API 返回 47 条 workflow 记录与 master 树 30 个文件有差异(已移除工作流的残留记录,推测)。

## 原文链接

- https://github.com/ArduPilot/ardupilot
- https://api.github.com/repos/ArduPilot/ardupilot
- https://api.github.com/repos/ArduPilot/ardupilot/contributors
- https://api.github.com/repos/ArduPilot/ardupilot/languages
- https://api.github.com/repos/ArduPilot/ardupilot/releases
- https://raw.githubusercontent.com/ArduPilot/ardupilot/master/.gitmodules
- https://raw.githubusercontent.com/ArduPilot/ardupilot/master/Tools/CodeStyle/astylerc
- https://raw.githubusercontent.com/ArduPilot/ardupilot/master/.github/CONTRIBUTING.md
- https://discuss.ardupilot.org/about.json
- https://firmware.ardupilot.org/Copter/stable/
- https://autotest.ardupilot.org/
- https://github.com/ArduPilot/MissionPlanner 、https://github.com/ArduPilot/pymavlink 、https://github.com/ArduPilot/MAVProxy 、https://github.com/ArduPilot/SITL_Models
