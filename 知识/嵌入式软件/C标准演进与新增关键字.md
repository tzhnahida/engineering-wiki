---
type: concept
tags: [C语言, 标准, C99, C11, C17, C23, 关键字, 演进]
created: 2026-08-01
updated: 2026-08-01
sources: ["⚠️ 存疑：内容综合自公开语言/编译器文档，原文未入库"]
---

# C 标准演进与新增关键字

> ISO C 从 C89/C90 到现在经历了五次重大修订。嵌入式开发长期停留在 C99 和 C11，但 C23 带来了 `constexpr`、`nullptr`、`typeof`、`bool`（真关键字）等期待已久的特性。本页追踪每个标准版本新增的关键字和嵌入式影响。

## 1. 标准版本速览

| 标准 | 年份 | 别称 | 关键字数 | 嵌入式现状 |
|------|------|------|----------|-----------|
| **C89/C90** | 1989/1990 | ANSI C / ISO C | 32 | ✅ 最兼容 |
| **C99** | 1999 | — | +5 | ✅ 嵌入式主流：`inline`, `restrict`, `_Bool` |
| **C11** | 2011 | — | +7 | ✅ 逐渐广泛：`_Atomic`, `_Generic`, `_Static_assert` |
| **C17** | 2018 | Bugfix C11 | 0 | ✅ 等同于 C11，无新特性 |
| **C23** | 2023/2024 | — | +17（新增+晋升） | 🟡 编译器支持中：`constexpr`, `nullptr`, `typeof` |

### 1.1 检测标准版本

```c
#if __STDC_VERSION__ >= 202311L
    // C23
#elif __STDC_VERSION__ >= 201710L
    // C17
#elif __STDC_VERSION__ >= 201112L
    // C11
#elif __STDC_VERSION__ >= 199901L
    // C99
#else
    // C89/C90
#endif
```

## 2. C99 新增（嵌入式最相关）

| 关键字 | 作用 | 嵌入式用法 |
|--------|------|-----------|
| `inline` | 建议编译器内联展开 | `static inline` 快速访问函数 |
| `restrict` | 指针独占别名提示 | `memcpy(void *restrict, ...)` 向量化 |
| `_Bool` | 布尔类型 (C23 改为 `bool`) | `_Bool flag = 0;` |
| `_Complex` | 复数类型 `float _Complex` | DSP/FFT 算法 |
| `_Imaginary` | 纯虚数类型 (很少用) | — |

C99 还引入了：`//` 单行注释、可变参数宏 `__VA_ARGS__`、for 循环内声明变量、变长数组 VLA、灵活数组成员 (flexible array member)。

## 3. C11 新增（现代嵌入式起点）

### 3.1 `_Atomic` — 原子类型与操作

```c
#include <stdatomic.h>

_Atomic int shared_flag;         // 硬件保证原子读写
atomic_fetch_add(&counter, 1);   // 原子自增 (LDREX/STREX)
```

> Cortex-M3+ 上，`_Atomic` 映射到 `LDREX/STREX` 排他加载/存储指令。但嵌入式实践更常用关中断实现临界区，因为原子操作不能替代互斥。

### 3.2 `_Generic` — 编译时类型分发

```c
#define sin(x) _Generic((x), \
    float:       sinf,        \
    double:      sin,         \
    long double: sinl          \
)(x)
```

> `_Generic` 实现编译时多态：根据参数类型选择不同函数。嵌入式用途有限但可用于类型安全日志宏。

### 3.3 `_Static_assert` — 编译时断言

```c
_Static_assert(sizeof(MyStruct) == 12, "MyStruct size must be 12");
_Static_assert(__alignof__(uint64_t) == 8, "uint64_t must be 8-byte aligned");
```

> C11 之前只能用 "负大小数组" 技巧实现编译时断言。`_Static_assert` 直接且语义明确。

### 3.4 `_Alignas` — 指定对齐

```c
_Alignas(32) uint8_t dma_buffer[128];  // 32 字节对齐
_Alignas(64) struct cache_line { ... };
```

> 等价于 `__attribute__((aligned(N)))` 或 C23 `alignas(N)`。

### 3.5 `_Alignof` — 查询对齐

```c
size_t a = _Alignof(double);  // → 8
```

> 等价于 `__alignof__` 或 C23 `alignof`。

### 3.6 `_Noreturn` — 函数永不返回

```c
_Noreturn void fatal_error(void) {
    while (1);
}
```

### 3.7 `_Thread_local` — 线程局部存储

```c
_Thread_local int errno;  // 每个线程独立副本
```

> 嵌入式裸机中通常只有一个执行上下文，`_Thread_local` 意义有限。

## 4. C17 (2018)

**无新关键字**。纯粹是 C11 的勘误修正版 (bugfix release)。

## 5. C23 新增（下一代嵌入式 C）

### 5.1 全新关键字

| 关键字 | 作用 | 嵌入式价值 |
|--------|------|-----------|
| **`constexpr`** | 真正的编译时常量 | ⭐⭐⭐ 替代 `#define` 魔法数字 |
| **`nullptr`** | 空指针常量 | ⭐⭐⭐ 替代 `NULL` / `(void*)0` |
| **`true` / `false`** | 真正的 bool 字面量 | ⭐⭐ 不再依赖 `<stdbool.h>` |
| **`typeof`** | 查询表达式类型 | ⭐⭐ 泛型宏、类型安全 |
| **`typeof_unqual`** | 去掉限定符的 `typeof` | ⭐ |
| **`_BitInt(N)`** | 任意精度整数 `_BitInt(24)` | ⭐ 24-bit ADC 值？ |

### 5.2 晋升为真关键字（旧形式被弃用）

| C23 新名 | C11 旧名 | 头文件宏 |
|----------|----------|---------|
| `alignas` | `_Alignas` | `<stdalign.h>` → 移除 |
| `alignof` | `_Alignof` | `<stdalign.h>` → 移除 |
| `bool` | `_Bool` | `<stdbool.h>` → 移除 |
| `static_assert` | `_Static_assert` | `<assert.h>` → 移除 |
| `thread_local` | `_Thread_local` | `<threads.h>` → 移除 |
| `noreturn` | `_Noreturn` | `<stdnoreturn.h>` → 移除 |

### 5.3 嵌入式场景展望

```c
// C23: 真正的编译时常量 — 比 #define 好一万倍
constexpr size_t DMA_BUFFER_SIZE = 256;
_Alignas(32) uint8_t dma_buf[DMA_BUFFER_SIZE];

// C23: typeof — 类型安全的 max
#define MAX(a, b) ({            \
    typeof(a) _a = (a);         \
    typeof(b) _b = (b);         \
    _a > _b ? _a : _b;          \
})

// C23: nullptr — 比 NULL 类型安全
int *ptr = nullptr;  // OK
int val = nullptr;   // 编译错误 — NULL 做不到
```

## 6. 嵌入式编译器支持状态

| 特性 | GCC 14 | armclang 6.22 | IAR 9.x | 推荐 |
|------|--------|--------------|---------|------|
| C99 全部 | ✅ | ✅ | ✅ | 默认基线 |
| C11 `_Atomic` | ✅ | ✅ | ✅ | 谨慎 |
| C11 `_Generic` | ✅ | ✅ | ✅ | 可尝试 |
| C11 `_Static_assert` | ✅ | ✅ | ✅ | **强烈推荐** |
| C11 `_Alignas` / `_Alignof` | ✅ | ✅ | ✅ | 推荐 |
| C17 | ✅ | ✅ | ✅ | 安全 (等同于 C11) |
| C23 `constexpr` | ✅ (13+) | 🟡 (部分) | ❌ | 等待 |
| C23 `nullptr` | ✅ (13+) | 🟡 (部分) | ❌ | 等待 |

> **当前建议**：以 C11 为基线（`-std=c11`），善用 `_Static_assert`、`_Alignas`。C23 成熟后再逐步采用 `constexpr` 和 `nullptr`。

## 7. 快速参考

```c
// === C99 ===
inline int max(int a, int b) { return a > b ? a : b; }  // 内联函数
void copy(int *restrict dst, const int *restrict src);    // 指针独占
_Bool done = 0;                                           // 布尔 (C23 废弃)

// === C11 ===
_Atomic int flag;                             // 原子变量
_Static_assert(sizeof(x) == 4, "!");          // 编译时断言
_Alignas(32) char buf[128];                   // 对齐分配
_Alignof(double);                              // 查询对齐
#define SINE(x) _Generic((x), float: sinf, default: sin)(x) // 类型分发

// === C23 ===
constexpr int N = 1024;                        // 编译时常量
int *p = nullptr;                              // 空指针
typeof(x) y = x;                               // 类型推导
_BitInt(24) adc_value;                         // 24位整数
```

## 相关页面

- [嵌入式软件/嵌入式C关键字实战指南](嵌入式C关键字实战指南.md) — volatile/static/const/extern 实战
- [嵌入式软件/ARM编译器扩展关键字](ARM编译器扩展关键字.md) — __asm / __inline / __restrict
- [嵌入式软件/C预处理器完全指南](C预处理器完全指南.md) — #define / #if / #pragma
- [嵌入式软件/结构体内存对齐与位域](结构体内存对齐与位域.md) — _Alignas / packed / pragma pack
- [嵌入式软件/C宏的副作用与类型安全](C宏的副作用与类型安全.md) — _Generic 安全替代
