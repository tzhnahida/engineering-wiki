---
type: concept
tags: [C语言, 预处理器, 宏, 条件编译, 嵌入式]
created: 2026-08-01
updated: 2026-08-01
sources: ["⚠️ 存疑：内容综合自公开语言/编译器文档，原文未入库"]
---

# C 预处理器完全指南

> C 预处理器在编译阶段 4 执行，处理所有 `#` 开头的指令。它是嵌入式 C 的编译时配置引擎——控制条件编译、常量定义、头文件管理、错误检测和断言。

## 1. 指令速查表

| 指令 | 作用 | 嵌入式场景 |
|------|------|-----------|
| `#define` | 定义宏 | 寄存器地址、常量、bit 标志 |
| `#undef` | 取消宏定义 | 重定义前的安全清理 |
| `#include` | 插入文件内容 | 头文件、CMSIS 驱动 |
| `#if` / `#elif` / `#else` / `#endif` | 条件编译 | 芯片型号选择、特性开关 |
| `#ifdef` / `#ifndef` | 测试宏是否定义 | 头文件防护、平台检测 |
| `#error` | 编译期报错 | 缺失配置、不支持的平台 |
| `#warning` | 编译期警告 | 弃用提示 |
| `#line` | 修改行号/文件名 | 代码生成工具 |
| `#pragma` | 编译器特定指令 | 对齐、优化、诊断控制 |
| `_Pragma` | `#pragma` 的宏内等价物 | X-Macro 中使用 pragma |
| `#` (null) | 空指令，无效果 | 分隔逻辑块 |

## 2. `#define` — 对象宏与函数宏

### 2.1 对象宏

```c
#define UART_BASE    0x40011000
#define LED_PIN      13
#define VERSION_STR  "v1.2.3"
```

### 2.2 函数宏

```c
#define BIT(n)       (1UL << (n))
#define ARRAY_SIZE(a) (sizeof(a) / sizeof((a)[0]))
#define MIN(a, b)    ((a) < (b) ? (a) : (b))
```

> [!warning] 宏的副作用陷阱
> `MAX(a++, b)` 中 `a` 会被递增两次。能用 `static inline` 就别写函数宏。详见 [嵌入式软件/C宏的副作用与类型安全](../../知识/嵌入式软件/C宏的副作用与类型安全.md)。

### 2.3 字符串化 (`#`) 与连接 (`##`)

| 操作符 | 作用 | 示例 |
|--------|------|------|
| `#` | 将参数转为字符串字面量 | `#x` → `"x"` |
| `##` | 将两个 token 粘合为一个 | `GPIO##port` → `GPIOA` |

```c
#define STR(x)       #x              // STR(hello) → "hello"
#define GPIO(port)   GPIO##port      // GPIO(A)  → GPIOA
```

### 2.4 可变参数宏 (C99)

```c
#define LOG(fmt, ...)  printf("[%s:%d] " fmt, __FILE__, __LINE__, ##__VA_ARGS__)
//                  GNU 扩展 ## 吃掉空参数前的逗号
```

## 3. `#if` / `#ifdef` — 条件编译

### 3.1 基本形式

```c
// 头文件防护
#ifndef _MY_HEADER_H_
#define _MY_HEADER_H_
// ...
#endif

// 基于数值的条件
#if CONFIG_UART_BAUDRATE >= 115200
    #define UART_BRR_VALUE  0x138
#elif CONFIG_UART_BAUDRATE >= 9600
    #define UART_BRR_VALUE  0x1D4C
#else
    #error "Unsupported baudrate"
#endif

// 多条件组合
#if defined(STM32F4) && !defined(USE_HAL_DRIVER)
    #error "STM32F4 requires HAL driver"
#endif
```

### 3.2 `defined` 操作符

```c
#if defined(__ARM_ARCH) && (__ARM_ARCH >= 8)
    // ARMv8+ specific
#endif
```

`defined` 是 `#if`/`#elif` 中专用的操作符，不能用在普通代码中。`#ifdef X` 等价于 `#if defined(X)`。

### 3.3 条件宏的技巧

```c
// 定义或默认值
#ifndef CONFIG_STACK_SIZE
    #define CONFIG_STACK_SIZE  2048
#endif

// 分模块定义，统一 include
// board_config.h:
#undef LED_PORT          // cleanup
#define LED_PORT GPIOB    // redefine
```

## 4. `#include` — 文件包含

| 形式 | 搜索路径 | 用途 |
|------|----------|------|
| `#include <file.h>` | 系统/编译器路径 (`-I` 指定) | CMSIS、标准库 |
| `#include "file.h"` | 当前目录 → 系统路径 | 项目头文件 |

```c
#include <stdint.h>          // 编译器提供的标准头
#include "bsp_led.h"         // 项目自己的 BSP 头
```

## 5. `#error` / `#warning` — 编译期诊断

```c
// 强制必需配置
#ifndef HSE_VALUE
    #error "HSE_VALUE must be defined in board_config.h"
#endif

// 弃用提示
#if defined(OLD_API)
    #warning "OLD_API is deprecated, use NEW_API instead"
#endif
```

## 6. `#pragma` — 编译器特定指令

### 6.1 `#pragma once`

```c
#pragma once
// 等效于 #ifndef / #define / #endif 三重防护
// 所有主流编译器支持
```

### 6.2 `#pragma pack`

```c
#pragma pack(push, 1)
struct protocol_frame {
    uint8_t  header;
    uint32_t timestamp;
    uint16_t checksum;
};
#pragma pack(pop)
// 等效于 __attribute__((packed))，但作用于整个结构体块
```

### 6.3 `_Pragma` 操作符 (C99)

`_Pragma` 可在宏内部使用——这是 `#pragma` 做不到的：

```c
#define PACK_STRUCT _Pragma("pack(push, 1)")

PACK_STRUCT
struct my_struct { /* ... */ };
// 等价于: #pragma pack(push, 1)
```

详见 [嵌入式软件/ARM编译器Pragma指令指南](../../知识/嵌入式软件/ARM编译器Pragma指令指南.md)。

## 7. 标准预定义宏

| 宏 | 类型 | 说明 |
|----|------|------|
| `__FILE__` | `const char[]` | 当前源文件名 |
| `__LINE__` | `int` | 当前行号 |
| `__DATE__` | `const char[]` | 编译日期 `"Mmm dd yyyy"` |
| `__TIME__` | `const char[]` | 编译时间 `"hh:mm:ss"` |
| `__STDC__` | `int` (1) | 编译器遵循 ISO C |
| `__STDC_VERSION__` | `long` | C 标准版本：199409L (C94), 199901L (C99), 201112L (C11), 201710L (C17), 202311L (C23) |
| `__STDC_HOSTED__` | `int` | 1=hosted (OS), 0=freestanding (bare metal) |
| `__func__` | `const char[]` | 当前函数名 (非宏，是隐式变量) |

### 7.1 编译器标识

| 宏 | 编译器 |
|----|--------|
| `__GNUC__` | GCC 主版本号 |
| `__clang__` | Clang/armclang 已定义 |
| `__ARMCC_VERSION` | ARM Compiler 5 (armcc) 版本 |
| `__armclang_version__` | ARM Compiler 6 (armclang) 版本字符串 |
| `__TI_ARM__` | TI ARM Clang |
| `__ICCARM__` | IAR EWARM |
| `_MSC_VER` | MSVC 版本 |

### 7.2 嵌入式常用的编译器预定义

```c
// 检测编译器类型
#if defined(__GNUC__) && !defined(__clang__)
    #define COMPILER "GCC"
#elif defined(__clang__)
    #define COMPILER "clang"
#elif defined(__ICCARM__)
    #define COMPILER "IAR"
#else
    #error "Unsupported compiler"
#endif
```

### 7.3 目标平台

| 宏 | 含义 |
|----|------|
| `__arm__` | 32-bit ARM (AArch32) |
| `__aarch64__` | 64-bit ARM (AArch64) |
| `__thumb__` | Thumb 模式 |
| `__CORTEX_M` | Cortex-M 系列号 (0/0+/1/3/4/7 等) |
| `__ARM_ARCH` | 架构版本：6=ARMv6-M, 7=ARMv7-M/R, 8=ARMv8-M |
| `__ARM_FP` | FPU 位掩码 (bit1=half, bit2=single, bit3=double) |

### 7.4 调试与断言

```c
// 编译时断言 (C11 _Static_assert)
_Static_assert(sizeof(int) == 4, "int must be 32-bit");

// 运行时断言配合预定义宏
#define ASSERT(cond) \
    do { if (!(cond)) { \
        printf("ASSERT FAIL: %s, %s:%d\n", #cond, __FILE__, __LINE__); \
        while(1); \
    }} while(0)
```

## 8. 常见模式

### 8.1 X-Macro 代码生成

```c
// 定义列表
#define GPIO_PINS \
    X(LED,     GPIOA, 5) \
    X(BUTTON,  GPIOB, 0) \
    X(RELAY,   GPIOC, 13)

// 生成枚举
typedef enum {
    #define X(name, port, pin) PIN_##name,
    GPIO_PINS
    #undef X
    PIN_COUNT
} pin_id_t;

// 生成初始化数组
const struct { GPIO_TypeDef *port; uint16_t pin; } pin_config[] = {
    #define X(name, port, pin) {port, pin},
    GPIO_PINS
    #undef X
};
```

详见 [嵌入式软件/C 语言宏高级技巧](../../知识/嵌入式软件/C%20语言宏高级技巧.md)。

### 8.2 编译时配置选择

```c
// board_config.h
#define BOARD_REV     3
#define USE_DMA       1
// ...

// driver.c
#include "board_config.h"

#if BOARD_REV >= 2
    #define HAS_FAST_MODE  1
#endif

#if USE_DMA
    #include "dma_driver.h"
#endif
```

### 8.3 双用途头文件

同一个头文件 via include 路径差异化：

```c
// 第一次包含 → 定义结构体
#define GPIO_DEFINE_STRUCT
#include "gpio_defs.h"

// 第二次包含 → 生成初始化表
#undef GPIO_DEFINE_STRUCT
#define GPIO_PINS_TABLE
#include "gpio_defs.h"
```

详见 [嵌入式软件/C 语言宏高级技巧](../../知识/嵌入式软件/C%20语言宏高级技巧.md)。

## 相关页面

- [嵌入式软件/C 语言宏高级技巧](../../知识/嵌入式软件/C%20语言宏高级技巧.md) — X-Macro、代码生成、预处理器元编程
- [嵌入式软件/C宏的副作用与类型安全](../../知识/嵌入式软件/C宏的副作用与类型安全.md) — 宏陷阱、static inline 替代
- [嵌入式软件/GCC __attribute__ 编译器扩展](../../知识/嵌入式软件/GCC%20__attribute__%20编译器扩展.md) — GCC/armclang 属性系统
- [嵌入式软件/ARM编译器Pragma指令指南](../../知识/嵌入式软件/ARM编译器Pragma指令指南.md) — `#pragma` 完全指南
- [嵌入式软件/嵌入式C关键字实战指南](../../知识/嵌入式软件/嵌入式C关键字实战指南.md) — volatile/static/const/extern 全关键字
