---
type: concept
tags: [嵌入式, C语言, 宏, 预处理器, X-Macro, 代码生成]
created: 2026-07-23
updated: 2026-07-23
sources: ["[2026-07-23 - C语言宏高级写法](../../%E6%9D%A5%E6%BA%90/2026-07-23%20-%20C%E8%AF%AD%E8%A8%80%E5%AE%8F%E9%AB%98%E7%BA%A7%E5%86%99%E6%B3%95.md)"]
---

# C 语言宏高级技巧

## 一句话

**宏不止 `#define PI 3.14`——`do{}while(0)` 是安全外套，`##` 是标识符拼图，可变参数是零开销日志，X-Macro 是数据驱动的代码生成引擎。**

## 四个高阶技巧

### 1. `do{}while(0)` — 宏的安全外套

**问题**：多条语句的宏直接展开会引入"悬空 else"bug：

```c
#define SAFE_PRINT(msg) \
    if (uart_busy)       \
        printf("busy\n");\
    printf(msg)

// if (cond) SAFE_PRINT("hello"); else ...
// 展开后 else 挂到内部 if 上 → bug
```

**`{}` 方案为什么不行**：

```c
#define BAD_MACRO() { func1(); func2(); }

if (cond)
    BAD_MACRO();   // 展开: { func1(); func2(); };
else                //        ↑ 多了一个分号! else 悬空
    other();        // 编译错误
```

**正确方案**：

```c
#define SAFE_PRINT(msg)            \
    do {                           \
        if (uart_busy)             \
            printf("busy\n");      \
        else                       \
            printf(msg);           \
    } while (0)

// if (cond) SAFE_PRINT("hello"); else ...
// ✅ 完美编译。末尾分号是语句的自然结束符
```

> [!tip] 为什么是 `while(0)` 而不是 `while(1)`？
> `while(0)` 保证循环体只执行一次，`break` 语句在宏内部可用于提前退出。

**实战：断言宏**：

```c
#define ASSERT(cond, msg)                              \
    do {                                               \
        if (!(cond)) {                                 \
            printf("ASSERT: %s in %s:%d\n",            \
                   msg, __FILE__, __LINE__);            \
            hard_fault_handler();                      \
        }                                              \
    } while (0)
```

### 2. `#` 与 `##` — 字符串化与拼接

**`#x` — 参数变字符串**：

```c
#define TO_STRING(x) #x
printf("%s\n", TO_STRING(GPIOA));  // 输出 "GPIOA"
```

**实战：寄存器名+值自动打印**：

```c
#define REG_DUMP(reg)                                   \
    do {                                                \
        printf("%s = 0x%08lX\n",                        \
               #reg, (unsigned long)(reg));             \
    } while (0)

REG_DUMP(USART1->SR);   // 输出: USART1->SR = 0x000000C0
REG_DUMP(TIM2->CNT);    // 输出: TIM2->CNT = 0x00000452
```

**`##` — 拼接标识符**：

```c
#define MAKE_REG(name, num) name ## num
MAKE_REG(GPIO, A)   // → GPIOA
MAKE_REG(GPIO, B)   // → GPIOB
```

**实战：自动生成中断向量表名称**：

```c
#define IRQ_HANDLER(irq_num) \
    void MAKE_REG(USART, irq_num)##_IRQHandler(void)

IRQ_HANDLER(1) {
    uint8_t data = USART1->DR;  // 自动变成 USART1_IRQHandler
}
```

### 3. 可变参数宏 — 灵活的日志系统

**基本用法**：

```c
#define LOG(fmt, ...) \
    printf("[LOG] " fmt "\n", ##__VA_ARGS__)

LOG("temp = %d°C", temperature);  // [LOG] temp = 28°C
```

> [!warning] 零参数时的尾部逗号
> `LOG("init OK")` → `printf("[LOG] init OK\n", )` — 多了一个逗号。GCC 的 `##__VA_ARGS__` 扩展：如果 `__VA_ARGS__` 为空，自动删除前面的逗号。

**实战：编译时裁剪的等级日志**：

```c
#define LOG_LEVEL_NONE  0
#define LOG_LEVEL_ERR   1
#define LOG_LEVEL_WARN  2
#define LOG_LEVEL_INFO  3

#ifndef LOG_LEVEL
#define LOG_LEVEL LOG_LEVEL_INFO
#endif

#define LOG_ERR(fmt, ...)                              \
    do { if (LOG_LEVEL >= LOG_LEVEL_ERR)               \
        printf("[ERR] " fmt "\n", ##__VA_ARGS__);      \
    } while (0)

#define LOG_WARN(fmt, ...)                             \
    do { if (LOG_LEVEL >= LOG_LEVEL_WARN)              \
        printf("[WARN] " fmt "\n", ##__VA_ARGS__);     \
    } while (0)

#define LOG_INFO(fmt, ...)                             \
    do { if (LOG_LEVEL >= LOG_LEVEL_INFO)              \
        printf("[INFO] " fmt "\n", ##__VA_ARGS__);     \
    } while (0)
```

> [!tip] 零运行时开销
> 设 `LOG_LEVEL=LOG_LEVEL_WARN`，所有 `LOG_INFO()` 的 `if` 条件恒假，编译器优化器直接删除——**零运行时开销**。

### 4. X-Macro — 数据驱动的代码生成

X-Macro 的核心思想：**同一份数据表，通过不同 `#define X` 生成不同代码**。

**定义数据表**（`gpio_pins.def`）：

```c
// gpio_pins.def
X(GPIO_PIN_LED,     A, 5)   // PA5 - LED
X(GPIO_PIN_BTN,     B, 3)   // PB3 - Button
X(GPIO_PIN_UART_TX, A, 9)   // PA9 - USART1 TX
X(GPIO_PIN_UART_RX, A, 10)  // PA10 - USART1 RX
#undef X
```

**一处数据，三处生成**：

```c
// 1. 生成枚举
typedef enum {
  #define X(name, port, pin) name,
  #include "gpio_pins.def"
    GPIO_PIN_COUNT
} gpio_pin_t;

// 2. 生成字符串表
static const char *gpio_pin_names[] = {
  #define X(name, port, pin) #name,
  #include "gpio_pins.def"
};

// 3. 生成初始化代码
static void gpio_init_all(void) {
  #define X(name, port, pin)                              \
    do {                                                   \
        GPIO_InitTypeDef init = {0};                       \
        init.Pin = GPIO_PIN_##pin;                         \
        init.Mode = GPIO_MODE_OUTPUT_PP;                   \
        HAL_GPIO_Init(GPIO##port, &init);                  \
    } while (0);
  #include "gpio_pins.def"
}
```

**不用头文件的写法**（单文件内使用）：

```c
#define GPIO_PIN_TABLE \
    X(GPIO_PIN_LED,     A, 5)  \
    X(GPIO_PIN_BTN,     B, 3)  \
    X(GPIO_PIN_UART_TX, A, 9)  \
    X(GPIO_PIN_UART_RX, A, 10)

typedef enum {
  #define X(name, port, pin) name,
    GPIO_PIN_TABLE
    GPIO_PIN_COUNT
} gpio_pin_t;
#undef X
```

> [!important] X-Macro 的真正价值
> 手写枚举 + 手写字符串表 + 手写初始化 → 三份信息靠人工同步，改一个引脚要改三个地方。
> X-Macro 把数据集中在**一个地方**，所有代码都从数据推导生成——**不可能出现不一致**。
> 几十上百个引脚的项目中，节省的时间是量级的。

## 终极综合：注册表风格设备驱动框架

组合全部四种技巧：

```c
// 1. do{}while(0) 确保宏安全
// 2. ## 拼接函数名
// 3. 可变参数做日志
// 4. X-Macro 做设备表

#define DEVICE_TABLE                               \
    X(DEV_USART1, USART1, 115200,   A, 9)          \
    X(DEV_USART2, USART2, 9600,     D, 5)          \
    X(DEV_I2C1,   I2C1,   400000,   B, 6)          \
    X(DEV_SPI1,   SPI1,   8000000,  A, 5)

// 生成设备枚举
typedef enum {
  #define X(name, reg, baud, port, pin) name,
    DEVICE_TABLE
    DEV_COUNT
} device_id_t;
#undef X

// 生成各设备初始化函数
#define DEV_INIT(name, reg, baud, port, pin)       \
    static void name##_init(void) {                 \
        LOG_INFO("Init %s @ %d baud", #name, baud); \
        /* 实际初始化... */                          \
    }
DEVICE_TABLE
#undef X

// 统一初始化入口
void all_devices_init(void) {
  #define X(name, reg, baud, port, pin) name##_init();
    DEVICE_TABLE
  #undef X
}
```

> 改设备只需改 `DEVICE_TABLE` 一行 → 枚举、初始化函数、统一入口全部自动更新。这就是**用编译器帮你写代码**。

## 技巧速查

| 技巧 | 语法 | 解决的问题 | 典型场景 |
|------|------|-----------|----------|
| `do{}while(0)` | `do { ... } while (0)` | 多条语句宏的安全封装 | 断言、GPIO 操作 |
| `#` 字符串化 | `#param` | 变量名→字符串 | 寄存器打印、日志 |
| `##` 拼接 | `a##b` | 标识符拼接 | 中断向量表、寄存器访问 |
| 可变参数 | `...` + `__VA_ARGS__` | 参数数量不固定 | 日志系统 |
| `##__VA_ARGS__` | GCC 扩展 | 零参数时去尾部逗号 | 日志系统 |
| X-Macro | `#define X` + `#include` | 数据驱动代码生成 | 引脚表、设备表、状态机 |

## 相关页面

- [GCC __attribute__ 编译器扩展](GCC%20__attribute__%20%E7%BC%96%E8%AF%91%E5%99%A8%E6%89%A9%E5%B1%95.md) — 同系列第 4 期，`used` + `section` 与 X-Macro 可组合使用
- [结构体内存对齐与位域](%E7%BB%93%E6%9E%84%E4%BD%93%E5%86%85%E5%AD%98%E5%AF%B9%E9%BD%90%E4%B8%8E%E4%BD%8D%E5%9F%9F.md) — 同系列第 3 期，位域可与宏拼接组合定义寄存器映射
- [MCU裸机软件分层架构](MCU%E8%A3%B8%E6%9C%BA%E8%BD%AF%E4%BB%B6%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84.md) — X-Macro 生成的设备表是 device 层初始化的编译期实现
