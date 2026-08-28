---
type: concept
tags: [arm, intrinsics, builtin, cmsis, acle, cortex-m, embedded]
created: 2026-08-01
updated: 2026-08-01
sources: ["⚠️ 存疑：内容综合自公开语言/编译器文档，原文未入库"]
---

# ARM 内置函数与 ACLE 指南

> ARM 编译器提供两类底层硬件访问：**builtin**（编译器内置函数，以 `__builtin_arm_` 开头）和 **ACLE intrinsic**（ARM C Language Extensions，头文件提供）。CMSIS 通过标准化宏封装两者，使代码在 GCC/armclang/IAR 间可移植。

## 1. 内置函数 vs ACLE vs CMSIS

| 层级 | 形式 | 可移植性 | 示例 |
|------|------|----------|------|
| **Compiler builtin** | `__builtin_arm_xxx()` | 编译器特定 | `__builtin_arm_clz(0x1000)` |
| **ACLE intrinsic** | `#include <arm_acle.h>` | ARM 编译器间 | `__clz(0x1000)` |
| **CMSIS** | `#include <cmsis_compiler.h>` | **所有 Cortex-M 编译器** | `__CLZ(0x1000)` |

> **嵌入式最佳实践**：永远使用 CMSIS 宏 (`__CLZ`, `__NOP`, `__WFE`)，确保代码在 GCC、armclang、IAR 间零修改。

## 2. CMSIS 核心指令

### 2.1 中断控制

```c
__disable_irq();     // CPSID i — 关全局中断 (PRIMASK=1)
__enable_irq();      // CPSIE i — 开全局中断 (PRIMASK=0)
__disable_fiq();     // CPSID f — 关快速中断 (FAULTMASK=1) (Cortex-M3+)
__enable_fiq();      // CPSIE f — 开快速中断
```

### 2.2 等待与提示

```c
__NOP();             // 空操作
__WFE();             // Wait For Event — 省电等待
__WFI();             // Wait For Interrupt — 进入睡眠
__SEV();             // Send Event — 唤醒其他核
__YIELD();           // 线程让步 (RTOS 用)
```

### 2.3 屏障与同步

```c
__DMB();             // Data Memory Barrier — 确保之前的存储完成
__DSB();             // Data Synchronization Barrier — 等所有存储完成
__ISB();             // Instruction Sync Barrier — 刷新指令流水线
__COMPILER_BARRIER();// 禁止编译器重排，不插入硬件指令
```

使用场景：

```c
// 启动 FPU — 必须在 CPACR 写入后加屏障
SCB->CPACR |= (0xF << 20);
__DSB();
__ISB();

// 更新 VTOR 后必须同步
SCB->VTOR = (uint32_t)vector_table;
__DSB();
__ISB();
```

### 2.4 位操作

```c
__CLZ(x);            // Count Leading Zeros — 用于快速 log2
__RBIT(x);           // Reverse Bits — 位反转 (FFT、CRC)
__REV(x);            // Reverse byte order (32-bit)
__REV16(x);          // Reverse halfword byte order (16-bit × 2)
__ROR(x, n);         // Rotate Right
```

```c
// CLZ 应用: 快速向上取整到 2 的幂
static inline uint32_t round_up_pow2(uint32_t x) {
    return 1UL << (32 - __CLZ(x - 1));
}
```

### 2.5 饱和运算 (Cortex-M3+)

```c
__SSAT(x, n);        // 有符号饱和到 n bits
__USAT(x, n);        // 无符号饱和到 n bits

// DSP 扩展 (Cortex-M4/M7):
__SADD8 / __SSUB8 / __SMLAD / __SMLALD / ...
```

### 2.6 系统寄存器访问

```c
// 读 CONTROL 寄存器 (PRIV 位、SPSEL 位)
uint32_t control = __get_CONTROL();

// 读 MSP / PSP
uint32_t msp = __get_MSP();
uint32_t psp = __get_PSP();
__set_PSP(new_sp);   // RTOS 任务切换时用

// 读 PRIMASK / FAULTMASK / BASEPRI
uint32_t primask = __get_PRIMASK();
__set_BASEPRI(0x80); // 屏蔽低于特定优先级的中断 (FreeRTOS 用)
```

## 3. 编译器内置函数

CMSIS 之下，armclang 和 GCC 各自有编译器级内置函数：

### 3.1 通用内置 (armclang + GCC 共性)

```c
int n = __builtin_clz(0x00100000);    // CLZ 指令 (如果可用)
int p = __builtin_popcount(0xFF00);   // 硬件 popcount 或软件模拟
void *ret = __builtin_return_address(0); // 返回地址 (通常用于调试)
__builtin_trap();                       // 触发 breakpoint/trap
__builtin_unreachable();               // 标记不可达路径 (帮助优化)
```

### 3.2 ARM 专用内置

```c
// armclang: CRC32 硬件加速 (Cortex-M33/M55 +crc)
uint32_t crc = __builtin_arm_crc32w(0, *(uint32_t*)data);

// armclang: 系统寄存器读写
uint64_t midr = __builtin_arm_rsr64("midr_el1");
__builtin_arm_wsr64("sctlr_el1", value);

// 预取
__builtin_arm_prefetch(addr, 1, 0);  // 预取到 L1 数据缓存
```

## 4. ACLE 头文件

ACLE (ARM C Language Extensions) 为标准 ARM 编译器提供可移植 API：

```c
#include <arm_acle.h>        // 核心 intrinsic: __clz, __rbit, __rev, __ssat
#include <arm_neon.h>        // SIMD/Neon intrinsic (Cortex-A/M55)
#include <arm_cmse.h>        // TrustZone CMSE 安全门 (Cortex-M33/M55)
#include <arm_sve.h>         // SVE 向量扩展 (Cortex-A)
```

## 5. 平台检测宏

```c
// 检测 FPU 类型
#if (__ARM_FP & 0x04)         // bit2 = single precision
    #define HAS_FPU_SP
#endif

// 检测硬件除法
#if defined(__ARM_FEATURE_IDIV)
    // 硬件整数除法 (Cortex-M3+)
#endif

// 检测 NEON
#if defined(__ARM_NEON)
    #include <arm_neon.h>
#endif

// 检测 DSP 扩展
#if defined(__ARM_FEATURE_DSP)
    // Cortex-M4/M7/M33/M55
#endif
```

## 6. armcc → armclang 内置函数迁移

| armcc (Compiler 5) | armclang (Compiler 6) | CMSIS (可移植) |
|---------------------|-----------------------|----------------|
| `__schedule_barrier()` | `__asm volatile("":::"memory")` | `__COMPILER_BARRIER()` |
| `__breakpoint(0)` | `__builtin_arm_trap()` | `__BKPT(0)` |
| `__nop()` | `__asm volatile("NOP")` | `__NOP()` |
| `__clz(x)` | `__builtin_arm_clz(x)` | `__CLZ(x)` |
| `__rbit(x)` | `__builtin_arm_rbit(x)` | `__RBIT(x)` |
| `__rev(x)` | `__builtin_bswap32(x)` | `__REV(x)` |
| `__disable_irq()` | `__asm volatile("CPSID i")` | `__disable_irq()` |

## 7. 常见模式

### 7.1 临界区保护 (FreeRTOS 风格)

```c
static inline uint32_t critical_enter(void) {
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    __DMB();         // 确保禁用操作完成后才进入临界区
    return primask;
}

static inline void critical_exit(uint32_t primask) {
    __DMB();         // 确保临界区操作完成
    __set_PRIMASK(primask);
}
```

### 7.2 精确延时 (纳秒级)

```c
__STATIC_FORCEINLINE void delay_cycles(uint32_t cycles) {
    if (cycles == 0) return;
    // DWT 周期计数器 — Cortex-M3+
    uint32_t start = DWT->CYCCNT;
    while ((DWT->CYCCNT - start) < cycles);
}
```

### 7.3 硬件 CRC 加速

```c
// Cortex-M33 带 +crc 扩展
static uint32_t hw_crc32(const uint8_t *data, size_t len) {
    uint32_t crc = 0xFFFFFFFF;
    for (size_t i = 0; i < len; i += 4) {
        crc = __builtin_arm_crc32w(crc, *(uint32_t*)(data + i));
    }
    return ~crc;
}
```

## 相关页面

- [嵌入式软件/ARM编译器扩展关键字](../../知识/嵌入式软件/ARM编译器扩展关键字.md) — __asm / __inline / __restrict
- [嵌入式软件/ARM编译器Pragma指令指南](../../知识/嵌入式软件/ARM编译器Pragma指令指南.md) — #pragma 完全指南
- [嵌入式软件/GCC __attribute__ 编译器扩展](../../知识/嵌入式软件/GCC%20__attribute__%20编译器扩展.md) — always_inline / section / aligned
- [嵌入式软件/C预处理器完全指南](../../知识/嵌入式软件/C预处理器完全指南.md) — 平台检测宏、条件编译
