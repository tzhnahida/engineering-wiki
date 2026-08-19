---
type: concept
tags: [arm, armclang, compiler, keyword, __asm, __inline, __restrict, embedded]
created: 2026-08-01
updated: 2026-08-01
sources: []
---

# ARM 编译器扩展关键字

> armclang (ARM Compiler 6 / LLVM-based) 和 armcc (ARM Compiler 5) 都有标准 C 之外的扩展关键字。从 armcc 迁移到 armclang 时，`__irq` 和 `__forceinline` 等关键字发生了变化。本页覆盖当前 armclang 关键字体系和 CMSIS 映射。

## 1. armclang 扩展关键字

当前 armclang 官方支持的四个扩展关键字：

| 关键字 | 作用 | 嵌入式场景 |
|--------|------|-----------|
| `__asm` | 内联汇编 | `__ASM volatile("cpsid i")` 关中断 |
| `__inline` | 函数内联提示 | `__STATIC_INLINE` 快速访问函数 |
| `__restrict` | 指针独占别名 | `memcpy(void * __restrict dst, ...)` |
| `__alignof__` | 查询类型对齐 | `__alignof__(uint64_t)` → 8 |
| `__declspec` | MSVC 风格属性声明 | `__declspec(noreturn)` |

## 2. CMSIS 标准映射

CMSIS (`cmsis_armclang.h`) 定义了可移植封装：

```c
#define __ASM            __asm
#define __INLINE         __inline
#define __STATIC_INLINE  static __inline
#define __STATIC_FORCEINLINE  __attribute__((always_inline)) static __inline
#define __RESTRICT       __restrict
#define __COMPILER_BARRIER()  __ASM volatile("":::"memory")
```

| CMSIS 宏 | armclang 底层 | armcc (legacy) | 说明 |
|-----------|-------------|----------------|------|
| `__ASM` | `__asm` | `__asm` | ✅ 两个编译器一致 |
| `__INLINE` | `__inline` | `__inline` | ✅ 一致 |
| `__STATIC_INLINE` | `static __inline` | `static __inline` | ✅ 一致 |
| `__STATIC_FORCEINLINE` | `__attribute__((always_inline))` | `__forceinline` | ⚠️ armcc 用 `__forceinline` 关键字 |
| `__RESTRICT` | `__restrict` | `__restrict` | ✅ 一致 |
| `__COMPILER_BARRIER` | `__asm volatile("":::"memory")` | `__schedule_barrier()` | ⚠️ armcc 用内置函数 |

## 3. armcc → armclang 关键字迁移

### 3.1 `__irq` → 不需要了

**armcc** 时代，ISR 函数必须标记 `__irq`，编译器才会生成正确的异常返回指令（`SUBS PC, LR, #4` 或等价的 `BX LR`）：

```c
// armcc (ARM Compiler 5)
__irq void UART_Handler(void) {
    // ...
}
```

**armclang** 时代，Cortex-M 架构硬件自动完成上下文保存和恢复（压栈/出栈），ISR 写成**普通 C 函数**即可：

```c
// armclang (ARM Compiler 6) — 无需任何关键字
void UART_Handler(void) {
    // 框架自动处理 NVIC 向量表注册
}
```

核心原因：Cortex-M 的异常入口硬件自动压栈 R0-R3/R12/LR/PC/xPSR，异常返回到 `PC` 时硬件自动检测 `EXC_RETURN` 值并正确恢复。编译器不需要 `__irq` 提示。

### 3.2 `__forceinline` → `__attribute__((always_inline))`

```c
// armcc
__forceinline void critical_delay(void) { __NOP(); }

// armclang (使用 CMSIS 宏)
__STATIC_FORCEINLINE void critical_delay(void) { __NOP(); }

// armclang (直接写法)
__attribute__((always_inline)) static inline void critical_delay(void) { __NOP(); }
```

### 3.3 `__value_in_regs` → 不复存在

armcc 的 `__value_in_regs` 将 struct 返回值放在寄存器中。armclang 不直接支持，改用标准 ABI（小 struct 自动以寄存器返回）。

### 3.4 `__softfp` → `__attribute__((pcs("aapcs-vfp")))`

```c
// armcc
__softfp double my_func(float x);

// armclang
__attribute__((pcs("aapcs-vfp"))) double my_func(float x);
```

## 4. `__asm` — 内联汇编语法差异

### 4.1 armclang: Integrated Assembler (GAS-like)

```c
// 读 CONTROL 寄存器
uint32_t control;
__ASM volatile("MRS %0, CONTROL" : "=r"(control));

// 关中断 (PRIMASK)
__ASM volatile("CPSID i" : : : "memory");

// 空操作
__ASM volatile("NOP");

// 数据同步屏障
__ASM volatile("DSB" ::: "memory");
```

### 4.2 armcc: Legacy ARM Assembler (armasm)

```c
// armcc syntax — 不支持 armclang
__asm {
    MRS control, CONTROL
    CPSID i
}
```

> [!note] 迁移时需将 `__asm { }` 块语法逐个转换为 GAS 风格单行语法。

## 5. `__restrict` — 指针优化提示

告诉编译器：这个指针是**唯一**访问目标数据的方式。消除别名疑虑后，编译器可以更激进地优化加载/存储：

```c
void copy_buffer(uint8_t * __restrict dst, const uint8_t * __restrict src, size_t n) {
    for (size_t i = 0; i < n; i++)
        dst[i] = src[i];  // 编译器可以向量化这个循环
}
```

> 等价于 C99 `restrict` 关键字。在 CMSIS 中通过 `__RESTRICT` 宏使用。

## 6. `__alignof__` — 查询对齐

```c
_Static_assert(__alignof__(uint64_t) == 8, "uint64 misaligned");

// 等价于 C11 _Alignof
// 等价于 C23 alignof
```

## 7. `__declspec` — MSVC 兼容层

armclang 支持部分 `__declspec` 用于 Windows 兼容（主要在 Keil MDK 环境中）：

```c
__declspec(noreturn) void fatal_error(void);
__declspec(dllimport) int external_func(void);
```

嵌入式裸机开发中几乎不需要。

## 相关页面

- [嵌入式软件/ARM编译器Pragma指令指南](ARM编译器Pragma指令指南.md) — #pragma pack / section / diagnostic
- [嵌入式软件/ARM内置函数与ACLE指南](ARM内置函数与ACLE指南.md) — __builtin_arm_* / CMSIS intrinsics
- [嵌入式软件/GCC __attribute__ 编译器扩展](GCC%20__attribute__%20编译器扩展.md) — packed / aligned / section / weak
- [嵌入式软件/嵌入式C关键字实战指南](嵌入式C关键字实战指南.md) — volatile/static/const/extern
