---
type: concept
tags: [嵌入式, 软件架构, 分层设计, MCU, RTOS]
created: 2026-07-21
updated: 2026-07-21
sources: ["[2026-07-21 - 嵌入式软件分层通用原则](../../%E6%9D%A5%E6%BA%90/2026-07-21%20-%20%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%88%86%E5%B1%82%E9%80%9A%E7%94%A8%E5%8E%9F%E5%88%99.md)"]
---

# MCU 裸机软件分层架构

![[mcu-六层架构.svg]]

> 📝 [在 Excalidraw 中编辑](https://excalidraw.com) — 拖入 `_llm/raw/mcu-六层架构.excalidraw`

## 一句话

**分层不是为了"目录好看"，而是通过明确职责边界和依赖方向，隔离变化传播，降低维护成本。**

## 六层架构总览

```
platform → bsp → device → service → app
component is cross-cutting
```

`component`（RingBuffer/CRC/PID/状态机框架/内存池）是横向复用层，可被所有上层调用，不依赖任何业务和硬件。

## 各层职责

| 层 | 回答的问题 | 不做什么 |
|---|---|---|
| **app** | 产品要做什么？ | 不操作 HAL、不关心 GPIO、不承担底层初始化 |
| **service** | 哪些能力可被多个业务复用？ | 不承载产品策略、不写设备语义 |
| **device** | 如何把器件变成可替换的设备接口？ | 不写业务流程、不掺杂产品逻辑 |
| **bsp** | 这块板子怎么用？ | 不写设备语义、不写业务逻辑 |
| **platform** | 项目跑在什么基础上？ | 尽量少改、不承载业务代码 |
| **component** | 什么能力可以跨项目复用？ | 不依赖任何产品/硬件 |

## 七条核心原则

### 1. 单一职责
一个模块只做一类事。不要一边读传感器，一边顺手做业务判断，再顺手发协议。

### 2. 高内聚、低耦合
模块内部紧密相关，对外暴露接口尽可能少。改内部实现不影响上层。

### 3. 接口清晰
模块之间通过稳定接口交互，禁止 `extern` 全局变量到处穿透。头文件声明 API，输入输出、生命周期、错误码明确。

### 4. 隐藏实现细节
上层只关心"是否成功"，不关心是 I2C 还是 SPI、DMA 还是轮询、有没有校验重试。

### 5. 禁止越层访问
- app 不能直接调 HAL
- service 不能直接访问寄存器
- device 不能越过 bsp 去碰 platform 细节

边界一旦失效，维护越来越难。

### 6. 可替换
- 换 MCU → app 基本不用改
- 换传感器 → 业务逻辑尽量不动
- 换板卡 → device 和 service 尽量稳定

这是分层真正起作用的检验标准。

### 7. 面向接口编程
上层依赖"能力"而非"某个具体实现细节"。依赖稳定抽象，不依赖易变实现。

## 初始化 vs 运行期调用

> [!important] 这两者不是一回事

**初始化**应由 `main` 统一编排：

```c
// Init order (top to bottom):
// 1. bsp_board_init()   — board-level hardware init
// 2. device_init_all()  — device drivers init
// 3. service_init_all() — service layer init
// 4. app_init()         — application init
// 5. app_start()        — start application
```

`app_init()` **不应**直接调用 `bsp_board_init()`。否则边界倒置——业务层负责底层初始化，整个分层就失效了。

## 新模块归属判断

按这个顺序问自己：

1. 是不是业务流程/策略/状态决策？→ **app**
2. 是不是给多个业务提供公共能力？→ **service**
3. 是不是在抽象某个具体外部设备？→ **device**
4. 是不是在封装 MCU 片上资源？→ **bsp**
5. 是不是官方库/HAL/RTOS/启动文件？→ **platform**
6. 是不是与业务和硬件都无关、可复用？→ **component**

> 先看职责，再看依赖，最后看复用性。

## 相关页面

- [PCIe 信号编码演进](../%E9%80%9A%E8%AE%AF%E7%BD%91%E7%BB%9C/PCIe%20%E4%BF%A1%E5%8F%B7%E7%BC%96%E7%A0%81%E6%BC%94%E8%BF%9B.md)
- [USB 3.0 信号完整性设计](../%E9%80%9A%E8%AE%AF%E7%BD%91%E7%BB%9C/USB%203.0%20%E4%BF%A1%E5%8F%B7%E5%AE%8C%E6%95%B4%E6%80%A7%E8%AE%BE%E8%AE%A1.md)
- [COM Express 标准](../%E9%80%9A%E8%AE%AF%E7%BD%91%E7%BB%9C/COM%20Express%20%E6%A0%87%E5%87%86.md)
