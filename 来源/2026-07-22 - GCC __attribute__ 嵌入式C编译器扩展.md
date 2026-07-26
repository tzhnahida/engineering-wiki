---
type: source
tags: [嵌入式, C语言, 编译器, GCC, 链接器]
created: 2026-07-22
updated: 2026-07-22
source_url: https://mp.weixin.qq.com/s/09_4TuYmFp41cMRqm5hOsg
author: LongWay
---

# GCC `__attribute__` 嵌入式 C 编译器扩展

> 来源：微信公众号「LongWay Embedded」· 嵌入式知识体系第4期，2026-07-22 抓取

## 概述

本文系统介绍 GCC `__attribute__` 扩展在嵌入式 C 开发中的八大核心属性：`section`、`aligned`、`packed`、`weak`、`constructor`/`destructor`、`used`、`always_inline`、`naked`。最后组合 `section` + `used` + `constructor` 构建了一个模块化初始化框架。

## 核心属性速览

| 属性 | 功能 | 典型场景 |
|------|------|----------|
| `section("name")` | 变量/函数放到指定内存段 | TCM 放置、初始化函数表、固件版本固化 |
| `aligned(n)` | 提升对齐要求 | DMA 缓冲区 Cache Line 对齐、矩阵运算 |
| `packed` | 取消结构体填充 | 协议帧解析、寄存器映射 |
| `weak` | 弱符号——可被覆盖的默认实现 | HAL 库回调、RTOS 钩子函数 |
| `constructor` / `destructor` | main 前后自动执行 | 预初始化、资源清理 |
| `used` | 防编译器优化删除 | 配合 section 的初始化函数 |
| `always_inline` | 强制内联 | 中断服务程序中的关键路径 |
| `naked` | 裸函数——无编译器序言/尾声 | 自定义 ISR 入口 |

## 关键要点

1. **`weak` 是最实用的属性**——HAL 库和 RTOS 框架的可扩展性基石
2. **`section` 的能力上限取决于链接脚本**——不读 `.ld` 文件，section 只能发挥一半功力
3. **可移植性需宏抽象**——`__attribute__` 是 GCC 扩展，ARMCC、IAR 语法不同
4. **`constructor` 在裸机 MCU 上慎用**——依赖 C 运行时初始化，`-nostdlib` 下可能不存在

## 模块化初始化框架

文章第 6 节组合了 `section` + `used` + `constructor` 实现了一个带优先级的模块注册系统：

```c
#define MODULE_INIT(priority) \
    static void module_init_##__LINE__(void); \
    static module_init_fn_t __attribute__((used, section(".module_init." #priority))) \
        __module_init_ptr_##__LINE__ = module_init_##__LINE__; \
    static void module_init_##__LINE__(void)
```

每个模块只需 `MODULE_INIT(N)`，链接器自动按优先级排列，`main()` 统一遍历执行。类似 Linux 内核的 `__initcall` 机制。

## 与 wiki 的关联

已创建知识页：[[GCC __attribute__ 编译器扩展]]
