
在学习了设备树中的中断信息配置之后，我们知道了如何获取中断号。现在该是时候深入学习具体的编程接口了，这些**API函数**就像是我们与中断系统对话的"语言工具"。

简单来说，Linux为我们提供了一套完整的中断编程接口，包括中断申请、处理函数编写、中断控制等功能。掌握这些API是编写中断驱动程序的基础技能。同时，中断处理函数的编写有着严格的规范要求，任何不当的编程都可能导致系统崩溃。

> [!note]+ 重要笔记
> 
> 这些API函数是Linux内核提供的标准接口，理解它们的使用方法和注意事项，对于编写稳定可靠的驱动程序至关重要。中断处理函数运行在特殊的系统环境中，必须遵循严格的编程规范。

## 1 中断申请和注销API

### 1.1 中断号的概念

每个中断都有一个**中断号**，通过中断号即可区分不同的中断。有的资料也把中断号叫做中断线，在Linux内核中使用一个int变量表示中断号。

换句话说，中断号就像每个中断的"身份证号码"，系统通过这个号码来识别和管理不同的中断源。关于如何获取中断号，我们在前面的章节中已经详细学习过了。

就像电话总机需要通过电话号码来区分不同的通话请求一样，Linux内核需要通过中断号来区分不同设备发出的中断请求。

### 1.2 中断线程化技术背景

在介绍request_irq之前，我们先了解一些重要概念，主要是**中断线程化**。中断线程化是实时Linux项目开发的一个新特性，目的是降低中断处理对系统实时延迟的影响。

request_irq()函数是比较旧的接口函数，在Linux 2.6.30内核中新增了线程化的中断注册函数`request_threaded_irq()`。

> [!question]+ Linux内核已经把中断处理分成了上下半部，为什么还需要引入中断线程化机制呢？
> 
> - 在Linux内核中，中断具有最高的优先级，只要有中断内核就会暂停目前工作，转向中断处理，等到所有的中断和软中断处理完成后才会执行**进程调度**，这个过程导致实时任务得不到及时处理
> - 中断上下文（中断处理程序、softirq、tasklet等）总是抢占进程上下文，所以中断上下文成为 **优化Linux实时性的最大挑战**

> [!example]+ 示例
> 
> - 假设一个高优先级任务和一个中断同时触发
> - 内核首先执行中断处理程序，中断处理重新完成之后可能触发软中断，也可能有一些tasklet要执行或有新的中断发生，**导致**高优先级任务的延迟不可预测

> [!abstract]+ 中断线程化机制的意义
> 
> - 把中断处理中一些繁重的任务作为内核线程来运行，让实时进程优先级高于中断线程
> - 这样，实时进程可以得到优先处理，延迟粒度小得多

简单来说，传统方式就像紧急事务总是打断重要会议，而线程化方式则像给紧急事务也安排了时间段，让重要会议能够按计划进行。

### 1.3 request_irq函数详解

**request_irq**函数是在Linux内核中用于注册中断处理程序的函数。它用于请求一个中断号（IRQ number）并将一个中断处理程序与该中断关联起来。
```c
// linux/interrupt.h
int request_irq(unsigned int irq, irq_handler_t handler, 
                unsigned long flags, const char *name, void *dev);

static inline int __must_check request_irq(unsigned int irq, irq_handler_t handler, 
                                          unsigned long flags, const char *name, void *dev){
    return request_threaded_irq(irq, handler, NULL, flags, name, dev);
}

// devm_开头的API申请的是内核"managed"的资源，一般不需要在出错处理和 remove()接口里再显式的释放
int devm_request_irq(struct device *dev, unsigned int irq, irq_handler_t handler,
                    unsigned long irqflags, const char *devname, void *dev_id);
```

**函数作用**： request_irq函数的主要功能是请求一个中断号，并将一个中断处理程序与该中断号关联起来。当中断事件发生时，与该中断号关联的中断处理程序会被调用执行。

> [!tip]+ 重要提示
> 
> request_irq函数会激活(使能)中断，所以不需要我们手动去使能中断，这个设计简化了中断使用流程。

**参数详解**：
- `irq`：请求的中断号（IRQ number），通过之前学习的方法获得
- `handler`：指向中断处理程序的函数指针，必须符合特定的函数签名，当中断发生以后就会执行此中断处理函数
- `flags`：中断标志，用于指定中断处理程序的行为和属性，可以在文件`include/linux/interrupt.h`里面查看所有的中断标志
- `name`：中断名字，用于标识该中断，可以在`/proc/interrupts`文件中看到对应的中断名字
- `dev`：指向设备或数据结构的指针，可以在中断处理程序中使用
    - 如果将flags设置为**IRQF_SHARED**的话，**dev用来区分不同的中断**
    - 一般情况下将dev设置为设备结构体，dev会传递给中断处理函数`irq_handler_t`的第二个参数

**返回值**：
- 成功：0或正数，表示中断请求成功
- 失败：负数，表示中断请求失败，返回的负数值表示错误代码

**常用的中断标志**：
- **IRQF_TRIGGER_NONE**：无触发方式，表示中断不会被触发
- **IRQF_TRIGGER_RISING**：上升沿触发方式，表示中断在信号上升沿时触发
- **IRQF_TRIGGER_FALLING**：下降沿触发方式，表示中断在信号下降沿时触发
- **IRQF_TRIGGER_HIGH**：高电平触发方式，表示中断在信号为高电平时触发
- **IRQF_TRIGGER_LOW**：低电平触发方式，表示中断在信号为低电平时触发
- **IRQF_SHARED**：中断共享方式，表示中断可以被多个设备共享使用
- **IRQF_ONESHOT**：单次中断，中断执行一次就结束

> [!note]+ 重要笔记
> 
> flags参数用于指定中断处理程序的行为和属性，如中断触发方式、中断共享等。可以使用位运算来组合多个属性。

### 1.4 request_threaded_irq函数

**request_threaded_irq**函数是Linux内核提供的一个功能强大的函数，用于请求分配一个中断，并将中断处理程序与该中断关联起来。该函数的主要作用是在系统中注册中断处理函数，以响应对应中断的发生。
```c
// 内核源码/kernel/irq/manage.c
int request_threaded_irq(unsigned int irq, irq_handler_t handler, 
                        irq_handler_t thread_fn, unsigned long irqflags, 
                        const char *devname, void *dev_id)
```
- `irq`：申请的中断号
- `handler`：绑定的中断处理函数（上半部）
- `thread_fn`：中断线程化处理函数（下半部）
- `irqflags`：中断标志位
- `devname`：请求中断的设备名称
- `dev_id`：标识中断的设备id，全局唯一，对于共享中断是必须要有的，因为后续释放中断时需要用到它

这个函数就像给设备安装了一个"两阶段处理系统"：第一阶段（handler）快速响应紧急情况，第二阶段（thread_fn）在合适的时机进行详细处理。

**共享与特殊标志**：
- **IRQF_SHARED**: 允许中断在多个设备间共享
- **IRQF_PROBE_SHARED**: 指示可能出现共享冲突的标志
- **IRQF_PERCPU**: 每CPU的中断
- **IRQF_NOBALANCING**: 排除中断负载均衡
- **IRQF_IRQPOLL**: 用于轮询的中断
- **IRQF_ONESHOT**: 中断处理后不重新使能
- **IRQF_NO_SUSPEND**: 在挂起期间不禁用此中断
- **IRQF_FORCE_RESUME**: 在恢复时强制启用中断，即使设置了 `IRQF_NO_SUSPEND`
- **IRQF_NO_THREAD**: 中断不可线程化
- **IRQF_EARLY_RESUME**: 在 `syscore` 期间提前恢复中断
- **IRQF_COND_SUSPEND**: 如果与 `NO_SUSPEND` 用户共享，挂起时执行中断处理

### 1.5 free_irq函数

**free_irq**函数用于释放之前通过request_irq函数注册的中断处理程序。它的作用是取消对中断的注册并释放相关的系统资源，包括中断号、中断处理程序和设备标识等
```c
// linux/interrupt.h
void free_irq(unsigned int irq, void *dev_id);
```
- `irq`：要释放的中断号
- `dev_id`：设备标识，用于区分不同的中断请求。通常是在request_irq函数中传递的设备特定数据指针
    - 如果中断设置为共享(IRQF_SHARED)的话，此参数用来区分具体的中断

**函数特性**：
- 如果中断不是共享的，那么free_irq会删除中断处理函数并且禁止中断
- 共享中断只有在释放最后一个中断处理函数的时候才会被禁止掉

换句话说，free_irq就像"退订服务"，告诉系统不再需要这个中断服务，可以释放相关资源。

### 1.6 中断内部状态标志详解（了解）

除了前面介绍的用户可配置的**IRQF_** 标志外，Linux内核还在内部使用一套**IRQS_** 状态标志来管理中断的运行状态。这些标志位于`kernel/irq/internals.h`文件中，主要用于内核内部的中断状态管理。

> [!note]+ 重要笔记
> 
> 这些标志是内核内部使用的，驱动开发者通常不需要直接操作，但了解它们有助于理解中断系统的工作机制和调试中断问题。

**内核中断状态枚举定义**：
```c
enum {
    IRQS_AUTODETECT         = 0x00000001,
    IRQS_SPURIOUS_DISABLED  = 0x00000002,
    IRQS_POLL_INPROGRESS    = 0x00000008,
    IRQS_ONESHOT            = 0x00000020,
    IRQS_REPLAY             = 0x00000040,
    IRQS_WAITING            = 0x00000080,
    IRQS_PENDING            = 0x00000200,
    IRQS_SUSPENDED          = 0x00000800,
    IRQS_TIMINGS            = 0x00001000,
};
```

**各状态标志详细说明**：

**IRQS_AUTODETECT：自动检测状态**
- **作用**：表示中断描述符正在进行自动检测过程
- **应用场景**：系统启动时或设备热插拔时，内核自动识别中断配置

**IRQS_SPURIOUS_DISABLED：伪中断禁用**
- **作用**：标记因检测到"伪中断"而被禁用的中断
- **触发原因**：当中断频繁触发但没有对应的处理程序响应时，内核认为是伪中断并自动禁用

> [!warning]+ 警告或注意
> 
> 伪中断通常表示硬件故障或驱动程序问题，需要及时排查和修复。

**IRQS_POLL_INPROGRESS：轮询进行中**
- **作用**：表示内核正在轮询方式调用中断处理函数
- **使用场景**：当正常中断机制出现问题时，内核会切换到轮询模式保证系统稳定性

**IRQS_ONESHOT：单次执行模式**
- **作用**：表示中断处理采用"一次性"模式，处理完成前不会重新使能
- **转换关系**：由用户设置的**IRQF_ONESHOT**标志转换而来
- **典型应用**：线程化中断处理中，确保上半部处理完成后才能接收新中断

> [!example]+ 示例
> 
> 在触摸屏驱动中，为了避免触摸事件的重复处理，通常会使用IRQF_ONESHOT标志，内核会自动设置对应的IRQS_ONESHOT状态。

**IRQS_REPLAY：中断重放**
- **作用**：标记需要重新发送的中断
- **使用场景**：当中断在禁用期间发生时，系统会标记为重放，待中断重新使能后再次处理

**IRQS_WAITING：等待状态**
- **作用**：表示中断描述符处于等待某种条件的状态
- **常见情况**：等待硬件初始化完成或等待其他中断处理完毕

**IRQS_PENDING：挂起状态**
- **作用**：表示中断请求已经发生但还未被处理
- **设置时机**：在`handle_fasteoi_irq()`函数中，当中断被禁用或缺少处理程序时设置

**IRQS_SUSPENDED：暂停状态**
- **作用**：表示中断在系统挂起过程中被暂停
- **恢复时机**：系统从休眠状态恢复时会清除此标志

换句话说，这些内部状态标志就像中断系统的"健康监控指标"，帮助内核实时了解每个中断的工作状态并做出相应的调整。

## 2 中断处理函数API

**函数功能**： handler函数是一个中断服务函数，是在中断事件发生时自动调用的函数，用于处理特定中断事件。它在中断事件发生时被操作系统或硬件调用，执行必要的操作来响应和处理中断请求。
```c
irqreturn_t handler(int irq, void *dev_id);

//函数原型
irqreturn_t (*irq_handler_t) (int, void *)
```
- `irq`：表示中断号或中断源的标识符。它指示引发中断的硬件设备或中断控制器
- `dev_id`：
    - 是一个void类型的指针，用于传递设备特定的数据或标识符
    - 通常用于在中断处理程序中区分不同的设备或资源
    - 需要与request_irq函数的dev_id参数保持一致，用于区分共享中断的不同设备，dev_id也可以指向设备数据结构

简单来说，中断处理函数就像一个"紧急事务处理员"，当有紧急情况（中断）发生时，系统会立即调用这个处理员来解决问题。

**返回值类型**：**irqreturn_t**是一个特定类型的枚举值，用于表示中断服务函数的返回状态
```c
enum irqreturn {
    IRQ_NONE = (0 << 0),
    IRQ_HANDLED = (1 << 0),
    IRQ_WAKE_THREAD = (1 << 1),
};

typedef enum irqreturn irqreturn_t;
```

它可以有以下几种取值，告诉系统中断处理的结果：
- **IRQ_NONE**：表示中断服务函数未处理该中断，中断控制器可以继续处理其他中断请求
- **IRQ_HANDLED**：表示中断服务函数已成功处理该中断，中断控制器无需进一步处理
- **IRQ_WAKE_THREAD**：表示中断服务函数已收到中断，并且请求唤醒一个内核线程来继续执行进一步的处理。适用于一些需要长时间处理的中断情况

一般中断服务函数返回值使用如下形式：
```c
return IRQ_RETVAL(IRQ_HANDLED)
```

> [!tip]+ 重要提示
> 
> 在共享中断的情况下，正确的返回值非常重要。如果你的设备没有产生中断，一定要返回IRQ_NONE，让系统能够正确地调用其他设备的处理函数。

也就是说，返回值就像处理员向调度中心汇报处理结果："我处理了"、"不是我的事"、"需要进一步处理"。

## 3 中断控制接口

### 3.1 全局中断控制机制

**软件可以禁止中断**，使处理器不响应所有中断请求，但是**不可屏蔽中断（Non Maskable Interrupt，NMI）** 是个例外。

换句话说，就像给房间安装了总闸开关一样，我们可以一次性关闭所有电器，但是紧急安全设备（如烟雾报警器）仍然会工作。NMI就是那个"烟雾报警器"，即使关闭了总开关，它依然能够工作。

> [!warning]+ 警告或注意
> 
> 禁用中断是一个强大但危险的操作，使用时必须谨慎，确保禁用时间尽可能短，否则会影响系统的实时性和稳定性。

**禁止中断的接口**：
- `local_irq_save(flags)`：首先把中断状态保存在参数flags中，然后禁止中断
- `local_irq_disable()`：直接禁止本处理器的中断

> [!tip]+ 重要提示
> 
> 这两个接口只能禁止**本处理器**的中断，不能禁止其他处理器的中断。在多核系统中，每个处理器都有独立的中断控制

**开启中断的接口**：
- `local_irq_enable()`：直接开启本处理器的中断
- `local_irq_restore(flags)`：恢复本处理器的中断状态

> **警告注意**  
> `local_irq_disable()`和`local_irq_enable()`不能嵌套使用，但`local_irq_save(flags)`和`local_irq_restore(flags)`可以嵌套使用。

**ARM64架构禁用中断的实现**：
```c
// local_irq_disable() -> raw_local_irq_disable() -> arch_local_irq_disable()

// arch/arm64/include/asm/irqflags.h
static inline void arch_local_irq_disable(void)
{
    asm volatile(
        "msr    daifset, #2    // arch_local_irq_disable"
        :
        :
        : "memory");
}
```
**代码说明**：这段汇编代码的作用是把处理器状态的**中断掩码位设置成1**，从此以后处理器不会响应中断请求。`daifset`指令用于设置处理器状态寄存器中的特定位。

**ARM64架构开启中断的实现**：
```c
// local_irq_enable() -> raw_local_irq_enable() -> arch_local_irq_enable()

// arch/arm64/include/asm/irqflags.h
static inline void arch_local_irq_enable(void)
{
    asm volatile(
        "msr    daifclr, #2    // arch_local_irq_enable"
        :
        :
        : "memory");
}
```
**代码说明**：这段代码把处理器状态的**中断掩码位设置成0**，重新启用中断响应。`daifclr`指令用于清除处理器状态寄存器中的特定位。

### 3.2 嵌套使用的重要性

为什么推荐使用save/restore组合呢？让我们通过一个实际场景来理解：

> [!example]+  问题场景示例
> **函数A**调用`local_irq_disable()`关闭中断，然后调用**函数B**  
> **函数B**也需要关闭中断，于是也调用`local_irq_disable()`  
> **函数B**执行完毕，调用`local_irq_enable()`开启中断  
> 此时**函数A**还没执行完，但中断已经被提前开启了，这可能导致竞态条件
> 
> 如果使用save/restore组合，每个函数都会记住进入时的中断状态，退出时恢复到正确的状态。

简单来说，save/restore组合就像"状态记忆器"，确保每个函数都能正确恢复到调用前的状态，而不会互相干扰。

### 3.3 单个中断控制机制

除了全局中断控制，**软件还可以禁止某个外围设备的中断**，这时中断控制器不会把该设备发送的中断转发给处理器。

简单来说，这就像给每个电器单独安装开关，我们可以选择性地关闭某些设备，而让其他设备正常工作。这种精细化控制在实际开发中非常有用。

**单个中断禁用函数**：
```c
void disable_irq(unsigned int irq);
```
- **irq**：Linux中断号，通过之前学习的方法获得
- **作用**：禁用指定的中断源，该中断不再被转发给处理器

**使用场景**：
- 临时关闭某个设备的中断响应
- 在设备初始化或配置期间避免中断干扰
- 调试时隔离特定的中断源

> [!warning]+ 警告或注意
> 
> disable_irq函数要等到当前正在执行的中断处理函数执行完才返回，因此使用者需要保证不会产生新的中断，并且确保所有已经开始执行的中断处理程序已经全部退出。

在某些情况下，我们不想等待当前中断处理完成，可以使用另外一个中断禁止函数，**disable_irq_nosync**函数调用以后立即返回，不会等待当前中断处理程序执行完毕。
```c
void disable_irq_nosync(unsigned int irq)
```

**disable_irq_nosync**函数调用以后立即返回，不会等待当前中断处理程序执行完毕。

**单个中断启用函数**：
```c
void enable_irq(unsigned int irq);
```
- **irq**：Linux中断号
- **作用**：启用指定的中断源，恢复中断转发

**实际应用示例**：
```c
// 临时禁用串口中断进行配置
disable_irq(uart_irq);

// 配置串口硬件寄存器
configure_uart_registers();

// 重新启用串口中断
enable_irq(uart_irq);
```

**代码说明**：在配置硬件寄存器期间，我们临时禁用相关中断，避免配置过程中的意外中断干扰，配置完成后再重新启用。

> [!warning]+ 警告或注意
> 
> 使用disable_irq/enable_irq时要确保配对使用，避免忘记重新启用中断导致设备失去响应能力。

### 3.4 GIC控制器的硬件实现

对于**ARM64架构的GIC控制器**，中断的启用和禁用是通过操作特定寄存器实现的：

- **启用中断**：设置分发器的寄存器**GICD_ISENABLERn**（Interrupt Set-Enable Register）
- **禁用中断**：设置分发器的寄存器**GICD_ICENABLERn**（Interrupt Clear-Enable Register）

> [!tip]+ 重要提示
> 
> 假设某个外围设备的硬件中断号是n，当这个外围设备发送中断给分发器时，只有在分发器上开启了硬件中断n，分发器才会把硬件中断n转发给处理器。

**工作流程**：
1. 外设产生中断信号发送给GIC分发器
2. 分发器检查对应的GICD_ISENABLERn寄存器位
3. 如果该位为1，则转发中断给目标处理器
4. 如果该位为0，则忽略该中断信号

也就是说，这就像一个有选择性的"过滤器"，只有通过"白名单"的中断才能进入系统进行处理。

## 4 中断函数编写规则

### 4.1 中断函数的基本限制

**无法传递自定义参数**：
- **原因**：中断函数是由硬件或内核调用的，其调用不允许传递用户自定义参数
- **特点**：函数原型固定为`irqreturn_t handler(int irq, void *dev_id)`

**无法处理返回值**：
- **原因**：中断函数的调用者（内核）只关心处理状态，无法处理复杂返回值
- **限制**：只能返回IRQ_HANDLED、IRQ_NONE等预定义状态

换句话说，中断函数就像是"紧急呼叫系统"，只能按照固定格式接收和回应，不能有个性化的对话。

> [!tip]+ 重要提示
> 
> 这些限制是为了确保中断处理的标准化和可靠性，所有的个性化数据交换都必须通过其他机制来实现。

### 4.2 数据交换的解决方案

**使用全局变量进行通信**：
```c
// 全局状态变量
volatile int button_pressed = 0;
volatile int sensor_data_ready = 0;

static irqreturn_t button_irq_handler(int irq, void *dev_id)
{
    // 通过全局变量记录事件
    button_pressed = 1;
    
    // 清除硬件中断状态
    clear_button_interrupt();
    
    return IRQ_HANDLED;
}

// 主程序中轮询检查
void main_loop(void)
{
    if (button_pressed) {
        button_pressed = 0;  // 清除标志
        handle_button_event();  // 处理按键事件
    }
}
```

**使用设备私有数据**：
```c
struct my_device {
    int status;
    int data_count;
    // 其他设备相关信息
};

static irqreturn_t device_irq_handler(int irq, void *dev_id)
{
    struct my_device *dev = (struct my_device *)dev_id;
    
    // 通过设备结构体传递信息
    dev->status = read_device_status();
    dev->data_count++;
    
    return IRQ_HANDLED;
}
```

> [!warning]+ 警告或注意
> 
> 使用全局变量时必须加上`volatile`关键字，避免编译器优化导致的数据同步问题。

### 4.3 中断函数的性能要求

**时间限制的原因**：
- 中断处理通常会屏蔽其他中断，耗时过长会影响系统实时性
- 短小的中断函数有助于提高系统整体响应能力
- 减少对正常程序执行的干扰

简单来说，中断函数就像"120急救电话"，必须快速响应、快速处理、快速结束，不能在电话里做复杂的诊断。

**减少代码逻辑**：
```c
// ❌ 错误示例：中断中处理复杂逻辑
static irqreturn_t bad_irq_handler(int irq, void *dev_id)
{
    // 复杂的数据处理（不推荐）
    for (int i = 0; i < 1000; i++) {
        process_complex_data(i);
    }
    calculate_statistics();
    update_database();
    
    return IRQ_HANDLED;
}

// ✅ 正确示例：中断中只做必要操作
static irqreturn_t good_irq_handler(int irq, void *dev_id)
{
    // 只记录事件，设置标志
    data_ready_flag = 1;
    
    // 快速读取关键数据
    current_sensor_value = read_sensor_register();
    
    // 清除中断状态
    clear_interrupt_status();
    
    return IRQ_HANDLED;
}
```

**避免使用延时函数**：
```c
// ❌ 错误：中断中不能有延时
static irqreturn_t bad_delay_handler(int irq, void *dev_id)
{
    mdelay(100);  // 绝对禁止！
    udelay(50);   // 也要避免
    msleep(10);   // 更不允许
    
    return IRQ_HANDLED;
}

// ✅ 正确：需要延时的操作放到工作队列
static irqreturn_t good_delay_handler(int irq, void *dev_id)
{
    // 立即响应中断
    disable_device_interrupt();
    
    // 将需要延时的工作交给工作队列
    schedule_work(&my_work);
    
    return IRQ_HANDLED;
}
```

> [!tip]+ 重要提示
> 
> 中断函数的执行时间应该控制在微秒级别，如果需要更长时间的处理，应该使用下半部机制（工作队列、tasklet等）。

**禁止的操作类型**：
- **文件操作**：read()、write()、open()等
- **内存分配**：malloc()、kmalloc()等可能阻塞的分配
- **复杂运算**：大量循环、浮点运算等
- **系统调用**：任何可能导致进程切换的调用

### 4.4 数据同步和安全性

**上下文保存的重要性**：
- 中断函数可能在任何时刻打断正在运行的程序
- 必须确保被中断程序的状态不被破坏
- 现代系统通常由硬件和编译器自动处理寄存器保存

**编译器自动处理的内容**：
- CPU寄存器的保存和恢复
- 程序计数器的记录
- 栈指针的管理

换句话说，就像有人突然闯进你的办公室，你需要先把手头的工作暂停并保存好，处理完突发事件后再完全恢复到之前的状态。

**volatile关键字的使用**：
```c
// 中断和主程序共享的变量必须声明为volatile
volatile int interrupt_count = 0;
volatile bool data_ready = false;

static irqreturn_t counting_irq_handler(int irq, void *dev_id)
{
    interrupt_count++;  // 中断中修改
    data_ready = true;
    
    return IRQ_HANDLED;
}

void main_function(void)
{
    while (!data_ready) {  // 主程序中读取
        // 等待数据准备就绪
    }
    
    printf("Received %d interrupts\n", interrupt_count);
}
```

**临界区保护**：
```c
static DEFINE_SPINLOCK(data_lock);
static int shared_counter = 0;

static irqreturn_t protected_irq_handler(int irq, void *dev_id)
{
    unsigned long flags;
    
    // 进入临界区
    spin_lock_irqsave(&data_lock, flags);
    
    shared_counter++;  // 安全地修改共享数据
    
    // 退出临界区
    spin_unlock_irqrestore(&data_lock, flags);
    
    return IRQ_HANDLED;
}
```

> [!warning]+ 警告或注意
> 
> 在中断上下文中使用锁时，必须使用不会睡眠的锁（如spinlock），绝对不能使用可能导致睡眠的锁（如mutex）

### 4.5 避免死锁的策略

**安全的锁使用原则**：
- 使用spinlock而不是mutex
- 锁定时间要尽可能短
- 避免嵌套锁定
- 使用irqsave版本的锁函数

```c
// ❌ 危险：可能导致死锁
static irqreturn_t dangerous_handler(int irq, void *dev_id)
{
    mutex_lock(&some_mutex);     // 错误！可能睡眠
    // ... 处理逻辑
    mutex_unlock(&some_mutex);
    
    return IRQ_HANDLED;
}

// ✅ 安全：正确的锁使用
static irqreturn_t safe_handler(int irq, void *dev_id)
{
    unsigned long flags;
    
    spin_lock_irqsave(&data_lock, flags);
    // 快速处理共享数据
    update_shared_data();
    spin_unlock_irqrestore(&data_lock, flags);
    
    return IRQ_HANDLED;
}
```

### 4.6 标准的中断函数模板

```c
// 推荐的中断函数编写模板
static irqreturn_t standard_irq_handler(int irq, void *dev_id)
{
    struct my_device *dev = (struct my_device *)dev_id;
    u32 status;
    
    // 1. 快速读取中断状态
    status = readl(dev->base + INT_STATUS_REG);
    
    // 2. 检查是否是本设备的中断（共享中断必需）
    if (!(status & MY_DEVICE_INT_MASK)) {
        return IRQ_NONE;
    }
    
    // 3. 快速清除中断状态，避免重复触发
    writel(status, dev->base + INT_CLEAR_REG);
    
    // 4. 最小化的数据处理
    if (status & DATA_READY_INT) {
        dev->data_ready = true;
        wake_up(&dev->wait_queue);  // 唤醒等待进程
    }
    
    // 5. 如果需要复杂处理，交给下半部
    if (status & COMPLEX_PROCESS_INT) {
        schedule_work(&dev->work);
    }
    
    return IRQ_HANDLED;
}
```

这个模板展示了一个标准中断处理函数应该包含的要素：快速状态检查、中断源确认、状态清除、基本处理和下半部分调度。

## 5 API使用示例

让我们通过几个完整的示例来演示如何正确使用这些中断API：

### 5.1 简单GPIO按键中断示例

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/gpio.h>
#include <linux/interrupt.h>
#include <linux/platform_device.h>

#define BUTTON_GPIO 18
static int irq_number;
static volatile int button_press_count = 0;

// 中断处理函数
static irqreturn_t button_irq_handler(int irq, void *dev_id)
{
    // 简单计数，避免复杂操作
    button_press_count++;
    
    printk(KERN_INFO "Button pressed! Count: %d\n", button_press_count);
    
    return IRQ_HANDLED;
}

static int __init button_init(void)
{
    int ret;
    
    // 1. 申请GPIO
    ret = gpio_request(BUTTON_GPIO, "button_gpio");
    if (ret) {
        printk(KERN_ERR "Failed to request GPIO %d\n", BUTTON_GPIO);
        return ret;
    }
    
    // 2. 设置GPIO为输入
    ret = gpio_direction_input(BUTTON_GPIO);
    if (ret) {
        printk(KERN_ERR "Failed to set GPIO direction\n");
        gpio_free(BUTTON_GPIO);
        return ret;
    }
    
    // 3. 获取中断号
    irq_number = gpio_to_irq(BUTTON_GPIO);
    if (irq_number < 0) {
        printk(KERN_ERR "Failed to get IRQ number\n");
        gpio_free(BUTTON_GPIO);
        return irq_number;
    }
    
    // 4. 注册中断
    ret = request_irq(irq_number, button_irq_handler,
                     IRQF_TRIGGER_FALLING, "button_irq", NULL);
    if (ret) {
        printk(KERN_ERR "Failed to request IRQ %d\n", irq_number);
        gpio_free(BUTTON_GPIO);
        return ret;
    }
    
    printk(KERN_INFO "Button driver loaded, IRQ: %d\n", irq_number);
    return 0;
}

static void __exit button_exit(void)
{
    free_irq(irq_number, NULL);
    gpio_free(BUTTON_GPIO);
    printk(KERN_INFO "Button driver unloaded. Total presses: %d\n", 
           button_press_count);
}

module_init(button_init);
module_exit(button_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Simple GPIO Button Interrupt Driver");
```

### 5.2 线程化中断示例

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/interrupt.h>
#include <linux/gpio.h>
#include <linux/kthread.h>

#define SENSOR_GPIO 25
static int irq_number;
static volatile int data_ready = 0;

// 中断上半部：快速响应
static irqreturn_t sensor_irq_handler(int irq, void *dev_id)
{
    // 快速清除中断状态
    data_ready = 1;
    
    // 请求唤醒线程化处理
    return IRQ_WAKE_THREAD;
}

// 中断下半部：线程化处理
static irqreturn_t sensor_thread_handler(int irq, void *dev_id)
{
    // 这里可以进行复杂的数据处理
    printk(KERN_INFO "Processing sensor data in thread context\n");
    
    // 模拟复杂的数据处理
    msleep(10);  // 在线程中可以使用睡眠函数
    
    // 处理完成后重新使能中断
    data_ready = 0;
    
    return IRQ_HANDLED;
}

static int __init sensor_init(void)
{
    int ret;
    
    ret = gpio_request(SENSOR_GPIO, "sensor_gpio");
    if (ret) {
        return ret;
    }
    
    ret = gpio_direction_input(SENSOR_GPIO);
    if (ret) {
        gpio_free(SENSOR_GPIO);
        return ret;
    }
    
    irq_number = gpio_to_irq(SENSOR_GPIO);
    if (irq_number < 0) {
        gpio_free(SENSOR_GPIO);
        return irq_number;
    }
    
    // 注册线程化中断
    ret = request_threaded_irq(irq_number, 
                              sensor_irq_handler,    // 上半部
                              sensor_thread_handler, // 下半部（线程）
                              IRQF_TRIGGER_RISING | IRQF_ONESHOT,
                              "sensor_irq", NULL);
    if (ret) {
        gpio_free(SENSOR_GPIO);
        return ret;
    }
    
    printk(KERN_INFO "Threaded sensor driver loaded\n");
    return 0;
}

static void __exit sensor_exit(void)
{
    free_irq(irq_number, NULL);
    gpio_free(SENSOR_GPIO);
    printk(KERN_INFO "Threaded sensor driver unloaded\n");
}

module_init(sensor_init);
module_exit(sensor_exit);
MODULE_LICENSE("GPL");
```

### 5.3 共享中断示例

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/interrupt.h>
#include <linux/platform_device.h>

struct shared_device {
    int device_id;
    void __iomem *base;
    int irq;
};

static irqreturn_t shared_irq_handler(int irq, void *dev_id)
{
    struct shared_device *dev = (struct shared_device *)dev_id;
    u32 status;
    
    // 读取设备状态寄存器
    status = readl(dev->base + 0x10);
    
    // 检查是否是本设备的中断
    if (!(status & (1 << dev->device_id))) {
        return IRQ_NONE;  // 不是本设备的中断
    }
    
    // 清除中断状态
    writel(1 << dev->device_id, dev->base + 0x14);
    
    printk(KERN_INFO "Device %d interrupt handled\n", dev->device_id);
    
    return IRQ_HANDLED;
}

static int shared_device_probe(struct platform_device *pdev)
{
    struct shared_device *dev;
    int ret;
    
    dev = devm_kzalloc(&pdev->dev, sizeof(*dev), GFP_KERNEL);
    if (!dev)
        return -ENOMEM;
    
    // 获取设备ID
    of_property_read_u32(pdev->dev.of_node, "device-id", &dev->device_id);
    
    // 获取中断号
    dev->irq = platform_get_irq(pdev, 0);
    if (dev->irq < 0)
        return dev->irq;
    
    // 注册共享中断
    ret = request_irq(dev->irq, shared_irq_handler,
                     IRQF_SHARED, "shared_device", dev);
    if (ret) {
        dev_err(&pdev->dev, "Failed to request shared IRQ\n");
        return ret;
    }
    
    platform_set_drvdata(pdev, dev);
    
    return 0;
}

static int shared_device_remove(struct platform_device *pdev)
{
    struct shared_device *dev = platform_get_drvdata(pdev);
    
    free_irq(dev->irq, dev);  // 注意：dev_id必须匹配
    
    return 0;
}

static const struct of_device_id shared_device_of_match[] = {
    { .compatible = "example,shared-device" },
    {},
};

static struct platform_driver shared_device_driver = {
    .probe = shared_device_probe,
    .remove = shared_device_remove,
    .driver = {
        .name = "shared-device",
        .of_match_table = shared_device_of_match,
    },
};

module_platform_driver(shared_device_driver);
MODULE_LICENSE("GPL");
```

## 6 API版本演进

### 6.1 历史发展轨迹

**早期Linux（2.0-2.4时代）**：
- 简单的`request_irq()`和`free_irq()`接口
- 基本的中断处理机制
- 缺乏高级特性支持

**Linux 2.6时代的改进**：
- 引入更多中断标志位
- 改进中断处理性能
- 增加SMP支持

**Linux 2.6.30的重大更新**：
- 引入`request_threaded_irq()`函数
- 支持中断线程化处理
- 改善实时性能

**现代Linux（3.0+）的增强**：
- 引入`devm_request_irq()`管理接口
- 完善的错误处理机制
- 更好的调试支持

### 6.2 API演进的设计思想

**从简单到复杂**：
- 早期API设计简单，功能基础
- 随着硬件复杂度增加，API功能不断扩展
- 保持向后兼容性的同时增加新特性

**从性能到实时性**：
- 早期关注基本功能实现
- 后期更加注重实时性和可预测性
- 引入线程化中断解决实时性问题

**从手动到自动**：
- 早期需要手动管理资源
- 现代API提供自动资源管理（devm_系列）
- 减少资源泄漏和编程错误

### 6.3 向前兼容性

Linux内核在API演进过程中始终保持**向前兼容性**：
```c
// 传统方式仍然支持
static inline int __must_check request_irq(unsigned int irq, irq_handler_t handler, 
                                          unsigned long flags, const char *name, void *dev){
    return request_threaded_irq(irq, handler, NULL, flags, name, dev);
}
```

这种设计确保了旧代码能够在新内核上正常运行，同时新代码能够利用更先进的特性。

也就是说，API的演进就像手机系统的升级：新功能不断增加，但老应用依然可以运行。

## 7 总结

通过这个章节的学习，我们掌握了Linux中断编程的核心API和编写规范：

> [!abstract]+ 核心技能掌握
> - 理解了中断申请和释放的完整流程：request_irq → 中断处理 → free_irq
> - 掌握了中断处理函数的编写规范和返回值含义
> - 学会了单个中断和全局中断的控制方法
> - 了解了中断标志的配置和共享中断的处理方式
> - 认识了中断函数编写的严格限制和解决方案
> - 掌握了数据同步和安全编程的方法
> - 学会了避免常见编程错误的技巧
> - 通过实际示例理解了API的正确使用方法

> [!abstract]+ 关键理解要点：
> - 中断API提供了完整的中断管理功能
> - 中断处理函数有严格的编程规范和性能要求
> - 不同的应用场景需要选择合适的API接口
> - 正确的错误处理和资源管理至关重要
> - API的演进体现了Linux内核的技术发展轨迹

这些API函数和编程规范是中断驱动开发的基础工具，正确使用它们能够确保我们编写出稳定可靠的驱动程序。中断处理函数的编写规范虽然严格，但这些限制都是为了保证系统的稳定性和实时性。

在下一章节中，我们将学习更高级的中断处理机制，包括上下半部处理、软中断、tasklet和工作队列等技术，进一步提升中断处理的效率和可靠性。

> [!question]+ 常见问题
> **Q**: 什么时候应该使用共享中断？  
> **A**: 当多个设备必须使用同一个中断线时（比如PCI设备），就需要使用IRQF_SHARED标志。每个设备的处理函数都需要检查是否是自己的设备产生的中断，如果不是则返回IRQ_NONE。这种机制在中断资源有限的系统中特别有用，但会增加软件的复杂度。