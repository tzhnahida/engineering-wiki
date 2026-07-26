---
type: concept
tags: [嵌入式, C语言, 编译器, GCC, 链接器, 内存布局]
created: 2026-07-22
updated: 2026-07-22
sources: ["[[2026-07-22 - GCC __attribute__ 嵌入式C编译器扩展]]"]
---

# GCC `__attribute__` 编译器扩展

## 一句话

**`__attribute__` 是 GCC 提供的编译期元信息机制，给变量、函数、类型附加底层控制能力——这些能力标准 C 不提供，但在嵌入式开发中不可或缺。**

## 为什么需要 `__attribute__`

标准 C 在以下场景无能为力：

| 需求 | 标准 C | `__attribute__` |
|------|--------|-----------------|
| 把变量放到特定内存地址（TCM） | ❌ | `section(".itcm_ram")` |
| 取消结构体填充（协议帧解析） | ❌ | `packed` |
| 提供可被覆盖的默认函数实现 | ❌ 同名函数链接报错 | `weak` |
| 在 main 之前自动执行初始化 | ❌ | `constructor` |
| 防止未引用变量被优化删除 | ❌ | `used` |

## 八大核心属性

### 1. `section` — 把变量/函数放到指定内存段

默认所有函数进 `.text`，全局变量进 `.data`/`.bss`。`section("name")` 让你自定义段名。

**场景 A：把关键数据放到 TCM（紧耦合内存）**

```c
#define ITCM_RAM  __attribute__((section(".itcm_ram")))

ITCM_RAM volatile uint32_t system_tick;
ITCM_RAM struct can_rx_buf rx_buffers[16];
```

链接脚本中定义 `.itcm_ram` 段映射到 TCM 地址。

**场景 B：构建初始化函数表（模块注册）**

```c
typedef void (*init_fn_t)(void);

#define INIT_PRIO(n)  __attribute__((section(".init_call." #n)))

void spi_init(void)  INIT_PRIO(1);
void i2c_init(void)  INIT_PRIO(2);
void uart_init(void) INIT_PRIO(3);

// 统一遍历
extern init_fn_t __init_call_start[];
extern init_fn_t __init_call_end[];

void run_init_chain(void) {
    for (init_fn_t *fn = __init_call_start; fn < __init_call_end; fn++)
        (*fn)();
}
```

> [!note] 核心思路
> 用编译期的段分配 + 链接脚本排序，替代运行时的全局数组维护。Linux 内核 `__initcall` 机制同理。

**场景 C：固件版本固化到固定 Flash 地址**

```c
__attribute__((section(".fw_version")))
const char firmware_version[] = "v2.1.0-build20260714";
```

上位机通过读取该固定地址获取固件版本。

### 2. `aligned` / `packed` — 精细控制内存对齐

**`aligned(n)`**：提升对齐要求

```c
// DMA 缓冲区 32 字节对齐（Cache Line 对齐）
uint8_t dma_buffer[1024] __attribute__((aligned(32)));

// 结构体整体 16 字节对齐
struct __attribute__((aligned(16))) matrix {
    float data[4][4];
};
```

Cortex-M7 的 Cache Line 是 32 字节，DMA 缓冲区跨越 Cache Line 边界会引发一致性问题。`aligned(32)` 是最简单的解决方案。

**`packed`**：取消填充

```c
struct __attribute__((packed)) can_frame {
    uint16_t id;      // offset 0
    uint8_t  dlc;     // offset 2
    uint8_t  data[8]; // offset 3
};  // sizeof = 11（正常对齐为 12）
```

适用于协议解析和存储格式。注意非对齐访问的性能代价。

> [!important] 两者可组合
> `struct __attribute__((packed, aligned(4))) { ... }` — 内部不填充，整体按 4 字节对齐。

### 3. `weak` — 弱符号：默认实现 + 用户可覆盖

这是嵌入式开发最实用的属性。

**问题**：标准 C 中同名函数链接报错 "multiple definition"。但框架需要"提供默认实现，允许用户覆盖"。

```c
// 弱符号：提供默认实现
__attribute__((weak)) void HAL_GPIO_Init(void) {
    // 默认：全部设为推挽输出
    GPIOA->CRL = 0x33333333;
}

// 用户代码中定义同名函数（强符号）→ 自动覆盖弱符号
void HAL_GPIO_Init(void) {
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
    GPIOA->CRL = 0x44444444;  // 自定义配置
}
```

**经典用法**：

| 场景 | 示例 |
|------|------|
| HAL 库中断回调 | `HAL_UART_RxCpltCallback` — STM32 HAL 几乎所有回调都是 weak |
| RTOS 钩子函数 | `vApplicationStackOverflowHook` — 默认死循环，用户可自定义 |
| 框架扩展点 | 给用户提供"不改库代码就能定制行为"的能力 |

> [!warning] 注意
> `weak` 必须用在函数/变量**定义**上（分配存储空间），不能用在声明（`extern`）上。

**工作原理**：编译器给弱符号打标记 → 链接器解析时强符号优先 → 同名弱符号被自动丢弃。

### 4. `constructor` / `destructor` — main 前后的自动钩子

```c
__attribute__((constructor)) void before_main(void) {
    init_hardware();  // 在 main() 之前自动执行
}

__attribute__((destructor)) void after_main(void) {
    cleanup_resources();  // 在 exit() 或 main() 返回后执行
}
```

**优先级控制**（数值越小越先执行，101~65535 为用户可用）：

```c
__attribute__((constructor(101))) void early_init(void) { /* 最先 */ }
__attribute__((constructor(102))) void mid_init(void)   { /* 第二 */ }
__attribute__((constructor(103))) void late_init(void)  { /* 第三 */ }
```

> [!warning] 裸机 MCU 上慎用
> `constructor` 依赖 C 运行时初始化（`__libc_init_array()` 遍历 `.init_array` 段）。裸机 `-nostdlib` 下可能不存在。**推荐在启动文件中显式调用 `SystemInit()` + 各模块初始化函数**，更可控。

### 5. 其他高频属性

| 属性 | 用途 | 说明 |
|------|------|------|
| `used` | 防止编译器优化删除"未引用"变量/函数 | 配合 `section` 使用——函数没人直接调用但不能被删 |
| `always_inline` | 强制内联 | 中断服务程序中的关键路径优化，注意代码膨胀 |
| `naked` | 裸函数——无编译器序言/尾声 | 自定义 ISR 入口，**不能有局部变量和 C 函数调用** |
| `unused` | 消除"未使用"警告 | 调试阶段保留但暂不用的函数 |

## 综合实战：模块化初始化框架

组合 `section` + `used` + `constructor` 实现松耦合模块注册：

```c
/* module_init.h */
typedef void (*module_init_fn_t)(void);

#define MODULE_INIT(priority) \
    static void module_init_##__LINE__(void); \
    static module_init_fn_t __attribute__((used, section(".module_init." #priority))) \
        __module_init_ptr_##__LINE__ = module_init_##__LINE__; \
    static void module_init_##__LINE__(void)

/* 各模块使用 */
MODULE_INIT(1) { RCC->APB2ENR = 0xFFFFFFFF; }  // 时钟最先
MODULE_INIT(2) { USART1->CR1 = USART_CR1_UE; } // UART 其次
MODULE_INIT(3) { I2C1->CR1 = I2C_CR1_PE; }    // I2C 最后

/* main() 中统一执行 */
extern module_init_fn_t __module_init_start[];
extern module_init_fn_t __module_init_end[];

int main(void) {
    for (module_init_fn_t *fn = __module_init_start; fn < __module_init_end; fn++)
        (*fn)();
    // ...
}
```

链接脚本对应段定义：

```ld
.module_init : {
    __module_init_start = .;
    KEEP(*(.module_init.*))
    __module_init_end = .;
} > FLASH
```

**优势**：
- ✅ 松耦合——每个模块只需包含头文件，不需要手动注册
- ✅ 自动排序——优先级数字决定执行顺序
- ✅ 零运行时开销——线性遍历，无链表管理开销
- ✅ 编译时检查——函数签名不匹配在编译时报错

## 可移植性：跨编译器宏抽象

`__attribute__` 是 GCC 扩展。IAR、ARMCC 语法不同，需要宏封装：

```c
#ifdef __GNUC__
  #define WEAK        __attribute__((weak))
  #define PACKED      __attribute__((packed))
  #define SECTION(x)  __attribute__((section(x)))
#elif defined(__ICCARM__)
  #define WEAK        __weak
  #define PACKED      __packed
  #define SECTION(x)  @ x
#endif
```

推荐给每个属性起语义化名字：

```c
#define RAM_FUNC   __attribute__((section(".ram_functions"), long_call))
#define IN_ITCM    __attribute__((section(".itcm_code")))
#define ALIGN_32   __attribute__((aligned(32)))
#define WEAK       __attribute__((weak))
#define USED       __attribute__((used))
#define PACKED     __attribute__((packed))
```

## 属性选型矩阵

| 属性 | 推荐指数 | 说明 |
|------|----------|------|
| `packed` | ⭐⭐⭐ | 协议解析必备，注意非对齐代价 |
| `aligned` | ⭐⭐⭐ | DMA/Cache 场景必用 |
| `weak` | ⭐⭐⭐⭐⭐ | HAL 库回调、框架扩展——最实用的属性 |
| `section` | ⭐⭐⭐⭐ | 高端用法，构建初始化链的利器 |
| `constructor` | ⭐⭐ | 裸机 MCU 上慎用，PC 调试可用 |
| `used` | ⭐⭐⭐ | 配合 section 使用 |
| `always_inline` | ⭐⭐⭐ | 关键路径优化，注意代码膨胀 |
| `naked` | ⭐⭐ | 高级用户专用，新手容易踩坑 |

## 相关页面

- [MCU裸机软件分层架构](MCU%E8%A3%B8%E6%9C%BA%E8%BD%AF%E4%BB%B6%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84.md) — 本页的模块化初始化框架是六层架构中「初始化与运行期分离」原则的编译期实现
- [[通讯网络/CAN 总线]] — CAN 帧结构解析中 packed 的实际应用
