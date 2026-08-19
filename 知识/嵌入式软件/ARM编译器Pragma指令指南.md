---
type: concept
tags: [arm, armclang, pragma, compiler, embedded, migration]
created: 2026-08-01
updated: 2026-08-01
sources: []
---

# ARM 编译器 Pragma 指令指南

> `#pragma` 是编译器特定的扩展指令。armclang (ARM Compiler 6) 基于 LLVM/Clang，支持一套与 armcc (ARM Compiler 5) 不同的 pragma 体系。从 armcc 迁移时，多数旧 pragma 需替换。

## 1. armclang 支持的 Pragma 速查

| Pragma | 作用 | 嵌入式场景 |
|--------|------|-----------|
| `#pragma pack` / `pack(push,n)` / `pack(pop)` | 结构体对齐控制 | 协议帧解析、Flash 数据 |
| `#pragma clang section` | 指定代码/数据段 | 将 ISR 放 `.ramfunc`、变量放 `.ccmram` |
| `#pragma clang diagnostic` | 诊断控制 | 压制第三方库的警告 |
| `#pragma once` | 头文件只包含一次 | 替代 `#ifndef` 老式防护 |
| `#pragma unroll` / `unroll_completely` | 循环展开控制 | 性能优化 |
| `#pragma weak` | 弱符号定义 | 默认 ISR 实现 |

## 2. `#pragma pack` — 结构体对齐

```c
// 方法一：块级
#pragma pack(push, 1)
struct can_frame {
    uint32_t id;
    uint8_t  dlc;
    uint8_t  data[8];
};
#pragma pack(pop)

// 方法二：reset to default
#pragma pack()          // 重置到编译时默认值
```

| 形式 | 作用 |
|------|------|
| `pack(n)` | 设置最大对齐字节 (1/2/4/8) |
| `pack(push, n)` | 压栈当前对齐 + 设置新值 |
| `pack(pop)` | 弹出恢复之前对齐 |
| `pack()` | 重置为默认 |

> `#pragma pack` 与 `__attribute__((packed))` 等效，但前者作用于一个代码块内所有结构体。

## 3. `#pragma clang section` — 段控制

将代码/变量放到自定义段，配合链接脚本实现 RAM 执行、快速内存等：

```c
// 将后续函数放入 .ramfunc 段 (链接脚本映射到 RAM)
#pragma clang section text=".ramfunc"
void flash_erase_sector(uint32_t addr) { /* ... */ }
void flash_program_word(uint32_t addr, uint32_t val) { /* ... */ }
#pragma clang section text=""    // 重置回默认

// 将数据放入 CCMRAM (STM32F4 的 64KB 紧耦合 RAM)
#pragma clang section bss=".ccmram"
static uint8_t fast_buffer[4096];
#pragma clang section bss=""
```

| 段类型 | 对应 armcc `#pragma arm section` | 目标段 |
|--------|--------------------------------|--------|
| `text` | `code` | `.text` / 自定义代码段 |
| `data` | `rwdata` | `.data` (已初始化读写) |
| `bss` | `zidata` | `.bss` (零初始化) |
| `rodata` | `rodata` | `.rodata` (只读/Flash) |
| `relro` | (new) | 重定位只读段 (PIC) |

> `__attribute__((section("name")))` 优先级高于 `#pragma clang section`。

## 4. `#pragma clang diagnostic` — 诊断控制

```c
// 压制第三方库的特定警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wsign-conversion"
#include "third_party/crypto.h"
#pragma clang diagnostic pop

// 将特定警告提升为错误
#pragma clang diagnostic error "-Wunused-variable"

// 完全静默一段代码
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Weverything"
// ...
#pragma clang diagnostic pop
```

常用诊断名称：

| `-W` 名称 | 内容 |
|-----------|------|
| `-Wunused-variable` | 未使用变量 |
| `-Wimplicit-function-declaration` | 隐式函数声明 |
| `-Wsign-conversion` | 符号转换 (嵌入式高频) |
| `-Wmissing-prototypes` | 缺少函数原型 |
| `-Wpacked` | packed 属性变更对齐 |
| `-Wpadded` | 被填充对齐的结构体 |

## 5. `#pragma unroll` — 循环展开

```c
#pragma unroll
for (int i = 0; i < 4; i++) {
    buffer[i] = process(buffer[i]);
}

#pragma unroll(8)
for (int i = 0; i < 128; i++) {
    // 编译器尝试展开 8 次
}

#pragma unroll_completely
for (int i = 0; i < 3; i++) {
    coeff[i] = read_adc();
}
```

> armclang 默认 `#pragma unroll` = 完全展开；armcc 默认 = `unroll(4)`。

## 6. `#pragma weak` — 弱符号

```c
// 直接定义弱符号
#pragma weak UART_Handler
void UART_Handler(void) {
    while (1);  // 默认死循环 (可被强符号覆盖)
}

// 别名形式的弱符号 (symbol1 映射到 symbol2)
#pragma weak Default_Handler = HardFault_Handler
```

等价于 `__attribute__((weak))`，但 `#pragma weak` 可以定义**别名形式的弱符号**。

## 7. armcc → armclang Pragma 迁移表

| armcc (Compiler 5) | armclang (Compiler 6) | 说明 |
|--------------------|-----------------------|------|
| `#pragma push` / `pop` | `#pragma clang diagnostic push/pop` | 改为分段 push/pop |
| `#pragma arm` / `#pragma thumb` | `-marm` / `-mthumb` (命令行) | 不再支持文件内切换 |
| `#pragma arm section code="X"` | `#pragma clang section text="X"` | 段类型名变化 |
| `#pragma arm section rwdata="X"` | `#pragma clang section data="X"` | |
| `#pragma arm section zidata="X"` | `#pragma clang section bss="X"` | |
| `#pragma diag_suppress` | `#pragma clang diagnostic ignored` | 诊断语法重构 |
| `#pragma diag_error` | `#pragma clang diagnostic error` | |
| `#pragma inline` / `no_inline` | `__attribute__((always_inline/noinline))` | 改为属性 |
| `#pragma Onum` / `Otime` | 命令行 `-On` | 不再支持 pragma 级 |
| `#pragma softfp_linkage` | `__attribute__((pcs(...)))` | 改为属性 |
| `#pragma import(__use_no_semihosting)` | `__asm(".global __use_no_semihosting")` | 改为内联汇编 |
| `#pragma import(symbol)` | `__asm(".global symbol")` | 改为内联汇编 |

## 8. armclang 不支持的 armcc Pragma

以下 pragma 在 armclang 中**没有等效替代**，遇到底层依赖时需要重新设计：

| 已弃用 Pragma | 介绍 | 替代思路 |
|--------------|------|---------|
| `CHECK_MISRA` | MISRA 检查控制 | 用独立静态分析工具 (PC-lint, SonarQube) |
| `MUST_ITERATE` | 循环迭代次数提示 | 手动循环展开 |
| `FUNC_EXT_CALLED` | 阻止优化删除函数调用 | `__attribute__((used))` |
| `CLINK` | C 链接标记 | 不需要 — armclang 默认符合标准 |
| `DUAL_STATE` | ARM/Thumb 联编代码生成 | 命令行 `-mthumb` 全文件适用 |

## 相关页面

- [嵌入式软件/ARM编译器扩展关键字](ARM编译器扩展关键字.md) — `__asm` / `__inline` / `__restrict`
- [嵌入式软件/GCC __attribute__ 编译器扩展](GCC%20__attribute__%20编译器扩展.md) — packed / section / aligned / weak
- [嵌入式软件/C预处理器完全指南](C预处理器完全指南.md) — `#define` / `#if` / `_Pragma`
- [嵌入式软件/结构体内存对齐与位域](结构体内存对齐与位域.md) — pragma pack 的对齐机制原理
