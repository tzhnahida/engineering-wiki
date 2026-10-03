---
type: concept
tags: [嵌入式软件, Rust, async, RTOS, Embassy]
created: 2026-09-11
updated: 2026-09-11
sources: ["[2026-09-11 - Embassy异步框架（单片机那点事）](../../来源/2026-09-11%20-%20Embassy异步框架（单片机那点事）.md)"]
---

# Embassy 异步框架

Embassy（**EMB**edded **ASY**nc）是 MCU 上的 Rust async 框架：任务在**编译期**被变成状态机，运行时全程**单栈**，没有内核调度器，外设走自家 HAL。它不是「跑在 RTOS 上的库」，而是直接顶掉 RTOS 的位置——调度交给 async executor，外设驱动交给 embassy-stm32 / embassy-nrf / embassy-rp 等 HAL。2020 年开源，MIT/Apache-2.0 双许可，GitHub 9.7k+ Star（2026-08），主仓 1.6 万+ 次提交，每周仍在活跃开发。

## 生态版图

| 层 | 项目 | 覆盖 |
|----|------|------|
| HAL | embassy-stm32 | STM32 全系 |
| HAL | embassy-nrf | nRF52/53/54/91 |
| HAL | embassy-rp | RP2040/RP235x |
| HAL | TI / NXP | MSPM0、MCX-A（官方仓库） |
| HAL | esp-rs/esp-hal | ESP32（乐鑫官方维护，文档在 docs.espressif.com） |
| HAL（社区） | — | 沁恒 CH32、普冉 PY32 |
| 协议栈 | embassy-net | 以太网/TCP/UDP/DHCP |
| 协议栈 | embassy-usb | CDC 串口、HID |
| 协议栈 | trouble | BLE 主机栈 |
| 中间件 | embassy-boot | 断电安全固件升级，带试跑（try）与回滚（rollback） |
| 基础库 | embassy-time | 全局 Timer/Instant 时基，不手撕硬件定时器、不会溢出 |

> [!note] 与 FreeRTOS 的定位差异
> FreeRTOS 是抢占式内核 + 独立栈 + 显式调度；Embassy 是协作式 executor + 单栈 + 编译期状态机。RTOS 与裸机/协作式的本质区别背景见 [嵌入式软件/RTOS vs Linux 本质区别](RTOS%20vs%20Linux%20本质区别.md)。

## 核心机制1：编译期状态机（源码级）

Rust 的 `async fn` 是语法糖：编译器把它整体编成一个实现了 `Future` trait 的结构体。**每个 `.await` 点是一个状态**，跨 await 存活的局部变量被提升为该结构体的字段。

### 变换形态（⚠️ 本节基于 Rust 官方 async 模型展开——The Rust Reference / Asynchronous Programming in Rust，非本文内容）

一个 async 函数：

```rust
async fn blink(pin: AnyPin) {
    let mut led = Output::new(pin, Level::Low, OutputDrive::Standard);
    loop {
        led.set_high();
        Timer::after_millis(150).await;   // 挂起点 A
        led.set_low();
        Timer::after_millis(150).await;   // 挂起点 B
    }
}
```

编译器生成的形态（示意，实际还含 loop 去糖化与 resume 标签）：

```rust
enum BlinkFuture {
    Start { led: Output },                 // 进入函数体，未到挂起点
    AfterHigh { led: Output, timer: Timer }, // 挂起点 A 后，等待 150ms 到期
    AfterLow { led: Output, timer: Timer },  // 挂起点 B 后
    Done,
}

impl Future for BlinkFuture {
    fn poll(mut self: Pin<&mut Self>, cx: &mut Context) -> Poll<()> {
        loop {
            match &mut *self {
                Self::Start { led } => {
                    led.set_high();
                    *self = Self::AfterHigh { led: /* moved */, timer: Timer::after_millis(150) };
                }
                Self::AfterHigh { led, timer } => match timer.poll(cx) {
                    Poll::Ready(()) => {
                        led.set_low();
                        *self = Self::AfterLow { /* … */, timer: Timer::after_millis(150) };
                    }
                    Poll::Pending => return Poll::Pending,   // ← 函数“返回”，状态已保存
                },
                // AfterLow 类似，Ready 后回到 AfterHigh（loop）
                _ => Poll::Ready(()),
            }
        }
    }
}
```

要点：

- **状态即程序计数器**：`Pending` 时函数并非真的「挂起在栈上」，而是已返回；下次 `poll` 从 `match` 匹配到的状态续跑——与手写状态机等价，但由编译器生成，无手工状态表
- **跨 await 存活的局部变量**（如 `led`、`timer`）是状态结构体的字段；不跨 await 的变量（如临时计算值）不占用跨挂起存储
- 状态转移发生在内存里，无寄存器快照、无栈切换

### 为什么需要 Pin（⚠️ 本节基于官方 async 模型展开）

`poll` 签名里的 `Pin<&mut Self>` 不是装饰：状态机可能**自引用**。例：

```rust
async fn demo() {
    let a = [0u8; 64];
    let r = &a;                    // r 指向 a
    Timer::after_millis(10).await; // 挂起时 a 与 r 都存进状态结构体
    println!("{}", r[0]);          // 跨挂起解引用
}
```

`a` 与指向 `a` 的引用 `r` 同时作为状态机字段 → 自引用结构体。普通 `&mut Self` 允许 `swap`/`move`，移动后 `r` 悬垂；`Pin` 承诺「被钉住的值不再移动」，编译器据此拒绝可能移动 `Self` 的操作。裸机 async 框架绕不开 Pin，这是理解 Embassy 类型报错（`!Unpin`、`Pin<&mut dyn Future>`）的地基。

### Waker 契约（⚠️ 本节基于官方 async 模型展开）

`Context` 里的 `Waker` 是「我还没好，好了叫谁」的登记机制：

1. `poll` 返回 `Pending` 前**必须**登记 Waker（底层通常经 `RawWakerVTable` 的 clone/wake/wake_by_ref 指向中断线或事件队列）
2. 事件就绪时（定时器到期、引脚中断、DMA 完成）调用 `wake()`
3. executor 把该任务重新入队，下次 poll 从上次状态续跑
4. **漏登记 = 任务永久沉睡**——这是 async 实现里最隐蔽的 bug 类别，Rust 的 `poll` 契约靠类型系统没法防，靠实现约定

Embassy 的 `Timer::after_millis` 走 embassy-time 全局时基：到期由定时器中断路径唤醒任务；`wait_for_low()` 由 GPIO EXTI 中断唤醒——**中断只做「叫醒」这一件事，不碰调度器**。✅ 源码佐证：`Timer::poll` 只做 `embassy_time_driver::schedule_wake(self.expires_at.as_ticks(), cx.waker())`（embassy-time/src/timer.rs:306-310）——把 Waker 登记给全局 time driver，任务自己不拥有任何硬件定时器。

## 核心机制2：executor 与单栈模型

### 任务存储（✅ 已按 embassy@bd35bd7 源码核对）

- `#[embassy_executor::task]` 宏展开为一个 `static POOL: TaskPool<...>`（`embassy-executor-macros/src/macros/task.rs:211`）——任务函数 + Future 状态机**静态分配**，`spawn` 不需要堆、无 malloc 碎片
- 池内的 `TaskStorage<F>`（raw/mod.rs:221-224）`#[repr(C)]`：TaskHeader 在偏移 0，`future: UninitCell<F>` 只存**编译出来的状态机**（大小由编译器精确计算）；注释明言 "A TaskStorage must live forever"
- RAM 细节（raw/mod.rs:240-241 注释）：`poll_fn` 惰性初始化，「so that a static TaskStorage will go in .bss」——这也是实测静态 RAM 只有 872B 的结构性原因之一
- `#[embassy_executor::main]` 初始化 HAL/时钟/外设并启动 executor 主循环

### 单栈 vs 每任务栈（本文核心论点，源码级对比）

| | FreeRTOS | Embassy |
|--|----------|---------|
| 任务内存 | 每任务独立栈（静态数组或 heap 分配）+ TCB | 状态机结构体（只含跨 await 存活变量） |
| 切换动作 | 保存/恢复整套寄存器 + 换 SP（PendSV） | 无——poll 返回即结束，「切换」是普通函数调用 |
| 栈深 | 每任务最坏调用链之和，需人为估算 | 只有一条主栈（main 栈 + 中断栈），估算对象从 N 个减到 1 个 |
| 溢出风险 | 「拍脑袋」栈大小 → HardFault | 单栈可一次性测透；状态机字段由编译器精确计算 |

「任务栈溢出」这个 RTOS 老毛病在 Embassy 里从根上消失：**没有每任务栈这个概念了**。代价见 [#协作式语义与硬实时](../../#协作式语义与硬实时.md)。

### 调度循环（✅ 已按 embassy@bd35bd7 源码核对）

线程模式 executor 的真实主循环（`embassy-executor/src/platform/cortex_m.rs:107-117`）：

```rust
pub fn run(&'static mut self, init: impl FnOnce(Spawner)) -> ! {
    init(self.inner.spawner());
    loop {
        unsafe {
            self.inner.poll();
            crate::trace_idle();
            asm!("wfe");     // ← Cortex-M 上睡眠指令是 WFE，不是 WFI
        };
    }
}
```

- **唤醒指令修正**：文章称「进 WFI」——源码里 Cortex-M 用 **WFE + SEV 配对**（pender 的 thread-mode 分支发 `sev`，cortex_m.rs:18-21）；RISC-V 平台才是 `wfi`（riscv.rs:82）。功能等价（睡眠至事件/中断），指令因架构而异
- **低功耗白送**：无 busy-wait 空转；`wfe` 等待 SEV 或中断
- **唤醒路径**：中断 → Waker（waker.rs:11-13，`wake` 直接调 `wake_task`）→ 任务入队 → 主循环 poll；中断里不 poll、不调度
- **Poll 的细节**（raw/mod.rs:270-289）：`poll` 把 `future: UninitCell<F>` 按 `Pin::new_unchecked` 钉住再 poll；`Ready` 后 `drop_in_place` 释放任务体，并把 poll_fn 换成 `poll_exited` 空函数供后续清理——任务结构可复用

## 文章代码解剖

```rust
#[embassy_executor::task]
async fn blink(pin: Peri<'static, AnyPin>) {
    let mut led = Output::new(pin, Level::Low, OutputDrive::Standard);
    loop {
        led.set_high();
        Timer::after_millis(150).await; // 全局时基，不碰硬件定时器
        led.set_low();
        Timer::after_millis(150).await;
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_nrf::init(Default::default());
    spawner.spawn(blink(p.P0_13.into())).unwrap();

    let mut button = Input::new(p.P0_11, Pull::Up);
    loop {
        button.wait_for_low().await;  // 等按键时核心在睡觉
        info!("按下了！");
        button.wait_for_high().await;
    }
}
```

- `wait_for_low().await`：等按键不占 CPU、不占额外栈——中断一来任务**原地续跑**（状态机从挂起点恢复）
- **写起来是顺序逻辑，跑起来是事件驱动**：这是 async 模型对嵌入式最大的生产力红利——没有状态表、没有回调金字塔、没有信号量串联

## 与 FreeRTOS 实测对比（STM32F446 @180MHz）

数据源：Tweede golf 2022 对比测试（官方 README 引用），同样按键+串口应用，Embassy/Rust vs FreeRTOS/C：

| 指标 | FreeRTOS/C | Embassy/Rust | 变化 |
|------|-----------|--------------|------|
| 中断处理耗时 | 2.96 μs | 1.45 μs | 快 51% |
| 中断延迟 | 4.97 μs | 3.74 μs | 低 25% |
| 程序体积 .text | 20.7 KB | 14.3 KB | 小 31% |
| 静态 RAM .data+.bss | 5480 B | 872 B | 省 84% |
| 上下文切换（单项） | **2.01 μs** | 2.29 μs | FreeRTOS 更快 |

读法（文章两句实话，原样保留）：

1. **单看上下文切换 FreeRTOS 反而更快**（2.01 vs 2.29μs）——Embassy 赢在整体设计而非每一环
2. 这是 **2022 年 demo 级测试**，当方向参考，别直接外推到项目

### 数字背后的架构解释（⚠️ 本节为分析展开，非文章原文）

- **RAM 省 84%**：无每任务栈（demo 里 4 个任务 × 每栈数百字节 ~ 数 KB）+ 无动态 TCB/heap 池 + Rust 静态分配。872B 基本只剩全局变量
- **中断路径快 51%**：Embassy 中断只登记唤醒；FreeRTOS 中断路径有 FromISR API 的开销（队列/临界区/必要时 PendSV 挂起，见 [嵌入式软件/FreeRTOS/11. FreeRTOS 中断管理](FreeRTOS/11.%20FreeRTOS%20中断管理.md)）
- **.text 小 31%**：无内核调度器/同步原语全家桶，且 Rust 可整块裁剪未用代码（泛型单态化 + 死代码消除，此处不及 LTO 细粒度讨论）
- **上下文切换慢于 FreeRTOS**：PendSV 切换是高度优化的固定开销（寄存器+SP 交换）；poll 切换要进入 Future::poll → match 状态 → 可能多层子 Future 嵌套解引用。协作式框架的切换不是免费午餐，只是把成本从「每次抢占」摊到「每次事件」

## 协作式语义与硬实时

**同一 executor 内是协作式调度**：一个任务闷头算 10ms 不 await，其他任务干等 10ms。与 FreeRTOS 的抢占语义（见 [嵌入式软件/FreeRTOS/3. FreeRTOS 任务管理与调度](FreeRTOS/3.%20FreeRTOS%20任务管理与调度.md)）有本质差异：

| 场景 | FreeRTOS | Embassy |
|------|----------|---------|
| CPU 密集型任务阻塞其他任务 | 高优先级抢占 | 同 executor 内会饿死他人 |
| 中断快速响应 | BASEPRI 临界区 + PendSV | 中断路径极短，天然快 |
| 硬实时保证 | 优先级 + 抢占（仍需自证） | 多 executor 挂不同中断优先级（官方 multiprio 例子），协作+抢占混合，设计自己兜住 |

> [!note] multiprio 设计已按源码确认（✅）
> `InterruptExecutor`（cortex_m.rs:134-147）文档与文章描述一致：「multiple interrupt mode executors at different priorities... Higher priority tasks will preempt lower priority ones」。官方例子 `examples/nrf52840/src/bin/multiprio.rs` 的头注释直接写明了分层：低优先级 executor 跑线程模式（wfe/sev），中/高优先级各挂一个中断（pending 中断即调度），高优先级 tick 可打断一切。

## 适用边界（文章明确列出的劝退清单）

- **C 资产厚**（自研协议栈、驱动、算法库）：FFI 混编能做，但成本不低，别为尝鲜硬迁老项目
- **功能安全认证**（车规、医疗）：FreeRTOS/ThreadX/Zephyr 有成熟认证路径，Embassy 没有
- **硬实时卡得死**：协作式语义 + 自组 multiprio executor，见上节
- **团队没人写 Rust 且下月交付**：借用检查器前两周劝退，别拿排期赌
- **芯片不在官方 HAL 列表**：自己写 PAC/HAL 工作量不小，先掂量

## 上手与调试

```bash
# 1. 装工具链与烧录工具（ST-Link / J-Link / DAPLink 都认）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install probe-rs-tools

# 2. 拉仓库进对应例子目录
git clone https://github.com/embassy-rs/embassy.git
cd embassy/examples/stm32f4   # nRF 用 examples/nrf52840

# 3. 接上调试器直接跑
cargo run --release --bin blinky
```

- **Cargo.toml 芯片 feature 必须改成板子具体型号**，否则烧进去就崩（文章明确提醒）
- **stable Rust 即可**：Embassy 保证在最新 stable 编译，网上「必须 nightly」教程已过时
- **调试**：probe-rs + defmt，走 RTT 打日志；无 Keil 式全家桶，习惯要换
- **混 C**：FFI 调 C 库是常规操作（复用现成驱动），unsafe 边界自守
- **学多少 Rust**：所有权、借用、async 三件套；建议电脑上先写一两周命令行小工具再上板，直接上板容易两头懵
- **商用**：MIT/Apache-2.0 双许可，免费商用，改了不强制开源

## 数值与置信度

| 数值 | 级别 | 依据 |
|------|------|------|
| 9.7k+ Star（2026-08）、1.6 万+ 提交、2020 开源、MIT/Apache-2.0 | A | 文字（文章自查时间戳） |
| F446 四项对比（1.45/3.74μs、14.3KB、872B） | A | 文字（转引自 Tweede golf 2022 测试） |
| 上下文切换 2.01 vs 2.29μs | A | 文字 |
| 状态机变换/Pin/Waker 细节 | B | ⚠️ 基于 Rust 官方 async 模型展开，非本文内容；具体编译器实现版本间有差异 |
| 任务静态分配、无堆 | A | ✅ 源码：task.rs:211 `static POOL: TaskPool` |
| 单栈/无栈切换 | A | ✅ 源码：run() 是普通 loop+poll（cortex_m.rs:107-117），全程无 SP 切换代码 |
| Waker=任务指针、wake 直通 wake_task | A | ✅ 源码：waker.rs:5-21 |
| Timer 登记全局 driver、不占硬件定时器 | A | ✅ 源码：timer.rs:306-310 schedule_wake |
| 「进 WFI 睡觉」 | B | ⚠️ 文章表述不精确：Cortex-M 用 WFE+SEV，RISC-V 才用 WFI（riscv.rs:82） |
| multiprio 抢占设计 | A | ✅ 源码：cortex_m.rs:134-147 + 官方 multiprio 例子头注释 |

## 源码核对声明

本文机制性声明已于 2026-09-11 对照 embassy 仓库（浅克隆 HEAD `bd35bd7`，2026-09-10 合并）逐一核对，路径与行号随文标注。唯一与文章表述有出入的是睡眠指令：文章「WFI」应为 **WFE**（Cortex-M；RISC-V 为 WFI），功能等价。

## 参见

- [嵌入式软件/FreeRTOS vs Embassy 对比](FreeRTOS%20vs%20Embassy%20对比.md) — 调度/栈/中断/IPC/认证九维对比 + 选型决策图
- [嵌入式软件/FreeRTOS/1. FreeRTOS 概述与架构](FreeRTOS/1.%20FreeRTOS%20概述与架构.md) — 抢占式内核全景与源码分析入口
- [嵌入式软件/FreeRTOS/3. FreeRTOS 任务管理与调度](FreeRTOS/3.%20FreeRTOS%20任务管理与调度.md) — TCB、PendSV 上下文切换、抢占语义
- [嵌入式软件/FreeRTOS/11. FreeRTOS 中断管理](FreeRTOS/11.%20FreeRTOS%20中断管理.md) — 中断路径开销的结构性来源
- [嵌入式软件/RTOS vs Linux 本质区别](RTOS%20vs%20Linux%20本质区别.md) — 协作式/抢占式/内核态的宏观分野
- [嵌入式软件/MCU裸机软件分层架构](MCU裸机软件分层架构.md) — 裸机状态机/事件驱动与 async 模型的对照
- [嵌入式软件/嵌入式固件开发流程](嵌入式固件开发流程.md) — 技术选型（RTOS vs async）落在开发流程的位置
