
在学习了[3_设备树中的中断信息](../02_中断基础概念/3_设备树中的中断信息.md)之后，我们知道了如何在设备树中配置中断信息。现在该是时候深入学习具体的编程接口了，这些**API函数**就像是我们与中断系统对话的"语言工具"。

简单来说，Linux为我们提供了一套完整的中断编程接口，包括中断申请、处理函数编写、中断控制等功能。掌握这些API是编写中断驱动程序的基础技能。

> [!note]+ 重要笔记
> 
> 这些API函数是Linux内核提供的标准接口，理解它们的使用方法和注意事项，对于编写稳定可靠的驱动程序至关重要。

## 1 中断申请和注销API

### 1.1 中断号的概念

每个中断都有一个**中断号**，通过中断号即可区分不同的中断。有的资料也把中断号叫做中断线，在Linux内核中使用一个int变量表示中断号。

换句话说，中断号就像每个中断的"身份证号码"，系统通过这个号码来识别和管理不同的中断源。关于如何获取中断号，我们在前面的[4 中断号管理机制](../07-🔍%20中断及异常处理/02_中断基础概念/4_中断子系统框架.md#4%20中断号管理机制)中已经详细学习过了。

### 1.2 request_irq前言

在介绍request_irq之前，我们先了解一些概念，主要是**中断线程化**，中断线程化是实时Linux项目开发的一个新特性，目的是降低中断处理对系统实时延迟的影响

request_irq()函数是比较旧的接口函数，在Linux 2.6.30内核中新增了线程化的中断注册函数`request_threaded_irq()`

> [!question]+ Linux内核已经把中断处理分成了上下半部，为什么还需要引入中断线程化机制呢？
> - 在Linux内核中，中断具有最高的优先级，只要有中断内核就会暂停目前工作，转向中断处理，等到所有的中断和软中断处理完成后才会执行**进程调度**，这个过程导致实时任务得不到及时处理
> - 中断上下文（中断处理程序、softirq、tasklet等）总是抢占进程上下文，所以中断上下文成为**优化Linux实时性的最大挑战**

> [!example]+ 示例
> - 假设 一个高优先级任务 和 一个中断同时触发
> - 内核首先执行中断处理程序，中断处理重新完成之后可能触发软中断，也可能有一些tasklet要执行或有新的中断发生，**导致**高优先级任务的延迟不可预测

> [!abstract]+ 中断线程化机制的意义
> - 把中断处理中一些繁重的任务作为内核线程来运行，让实时进程优先级高于中断线程
> - 这样，实时进程可以得到优先处理，延迟粒度小得多

### 1.3 📕request_irq函数简介

**request_irq**函数是在Linux内核中用于注册中断处理程序的函数。它用于请求一个中断号（IRQ number）并将一个中断处理程序与该中断关联起来
```c
// linux/interrupt.h
int request_irq(unsigned int irq, irq_handler_t handler, unsigned long flags, const char *name, void *dev);

static inline int __must_check request_irq(unsigned int irq, irq_handler_t handler, unsigned long flags, const char *name, void *dev){
	return request_threaded_irq(irq, handler, NULL, flags, name, dev);
}

// devm_开头的API申请的是内核“managed”的资源，一般不需要在出错处理和 remove()接口里再显式的释放
int devm_request_irq(struct device *dev, unsigned int irq, irq_handler_t handler,
                        unsigned long irqflags, const char *devname, void *dev_id);
```

**函数作用**： request_irq函数的主要功能是请求一个中断号，并将一个中断处理程序与该中断号关联起来。当中断事件发生时，与该中断号关联的中断处理程序会被调用执行。

> [!tip]+ 重要提示
> 
> request_irq函数会激活(使能)中断，所以不需要我们手动去使能中断，这个设计简化了中断使用流程。

**参数详解**：
- **irq**：请求的中断号（IRQ number），通过之前学习的[2 从设备树获取中断号](../07-🔍%20中断及异常处理/02_中断基础概念/3_获取中断号.md#2%20从设备树获取中断号)方法获得
- **handler**：指向中断处理程序的函数指针，必须符合特定的函数签名，当中断发生以后就会执行此中断处理函数
- **flags**：中断标志，用于指定中断处理程序的行为和属性，可以在文件`include/linux/interrupt.h`里面查看所有的中断标志
- **name**：中断名字，用于标识该中断，可以在`/proc/interrupts`文件中看到对应的中断名字
- **dev**：指向设备或数据结构的指针，可以在中断处理程序中使用
	- 如果将flags设置为**IRQF_SHARED**的话，**dev用来区分不同的中断**
	- 一般情况下将dev设置为设备结构体，dev会传递给中断处理函数`irq_handler_t`的第二个参数

**返回值**：
- 成功：0或正数，表示中断请求成功
- 失败：负数，表示中断请求失败，返回的负数值表示错误代码

**用于request_irq函数的触发类型标志**：
- **IRQF_TRIGGER_NONE**：无触发方式，表示中断不会被触发
- **IRQF_TRIGGER_RISING**：上升沿触发方式，表示中断在信号上升沿时触发
- **IRQF_TRIGGER_FALLING**：下降沿触发方式，表示中断在信号下降沿时触发
- **IRQF_TRIGGER_HIGH**：高电平触发方式，表示中断在信号为高电平时触发
- **IRQF_TRIGGER_LOW**：低电平触发方式，表示中断在信号为低电平时触发
- **IRQF_SHARED**：中断共享方式，表示中断可以被多个设备共享使用、
- **IRQF_ONESHOT**：单次中断，中断执行一次就结束

> [!note]+ 重要笔记
> 
> flags参数用于指定中断处理程序的行为和属性，如中断触发方式、中断共享等。可以使用位运算来组合多个属性。

> [!example]+ 示例
> 
> 按键中断通常使用IRQF_TRIGGER_FALLING（下降沿触发），因为按键按下时GPIO电平从高变低。而某些传感器可能使用IRQF_TRIGGER_HIGH（高电平触发），表示数据准备就绪。

### 1.4 request_threaded_irq函数

**request_threaded_irq**函数是Linux内核提供的一个功能强大的函数，用于请求分配一个中断，并将中断处理程序与该中断关联起来。该函数的主要作用是在系统中注册中断处理函数，以响应对应中断的发生。
```c
// 内核源码/kernel/irq/manage.c
int request_threaded_irq(unsigned int irq, irq_handler_t handler, irq_hendler_t thread_fn, unsigned long irqflags, const char *devname, void *dev_id)
```
- irq：申请的中断号
- handler：绑定的中断处理函数
- thread_fn：中断线程化处理函数
- irqflags：中断标志位，参考如下和如上
- devname：请求中断的设备名称
- dev_id：标识中断的设备id，全局唯一
	- 对于共享中断是必须要有的，因为后续释放中断时需要用到它

**共享与特殊标志**
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

### 1.5 gpio_to_irq函数

**gpio_to_irq**函数用于将GPIO引脚的编号（GPIO pin number）转换为对应的中断请求号（interrupt request number）

```c
// linux/gpio.h
unsigned int gpio_to_irq(unsigned int gpio);
```

**功能描述**： gpio_to_irq是一个用于将GPIO引脚映射到对应中断号的函数。其作用是根据指定的GPIO引脚号，获取与之关联的中断号。

**参数说明**：
- **gpio**：要映射的GPIO引脚号

**返回值**：
- 成功：返回值为该GPIO引脚所对应的中断号
- 失败：返回值为负数，表示映射失败或无效的GPIO引脚号

> [!warning]+ 警告或注意
> 
> 不是所有的GPIO都支持中断功能，使用前需要确认硬件是否支持，并检查返回值的有效性。

### 1.6 free_irq函数

**free_irq**函数用于释放之前通过request_irq函数注册的中断处理程序。它的作用是取消对中断的注册并释放相关的系统资源。

```c
// linux/interrupt.h
void free_irq(unsigned int irq, void *dev_id);
```

**功能描述**： free_irq函数用于释放之前通过request_irq函数注册的中断处理程序。它会取消对中断的注册并释放相关的系统资源，包括中断号、中断处理程序和设备标识等。

**函数特性**：
- 如果中断不是共享的，那么free_irq会删除中断处理函数并且禁止中断
- 共享中断只有在释放最后一个中断处理函数的时候才会被禁止掉

**参数说明**：
- **irq**：要释放的中断号
- **dev_id**：设备标识，用于区分不同的中断请求。通常是在request_irq函数中传递的设备特定数据指针
	- 如果中断设置为共享(IRQF_SHARED)的话，此参数用来区分具体的中断

### 1.7 中断内部状态标志详解（了解）

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

**重要标志位的实际应用**：
**IRQS_ONESHOT的处理流程**：
- 注册时：`request_irq()`中的**IRQF_ONESHOT**标志被转换为**IRQS_ONESHOT**状态
- 执行时：中断处理程序运行期间，该中断线被禁用
- 完成时：调用`irq_finalize_oneshot()`函数重新使能中断
**IRQS_PENDING的典型场景**：
- 中断发生时如果该中断线被禁用，内核会设置**IRQS_PENDING**标志
- 当中断重新使能时，内核检查此标志并处理挂起的中断
- 这确保了即使在中断禁用期间发生的中断也不会丢失

## 2 中断处理API原型

中断处理程序是在中断事件发生时自动调用的函数。它负责处理与中断相关的操作，例如读取数据、清除中断标志、更新状态等。

```c
irqreturn_t handler(int irq, void *dev_id);

//函数原型
irqreturn_t (*irq_handler_t) (int, void *)
```

**函数功能**： handler函数是一个中断服务函数，用于处理特定中断事件。它在中断事件发生时被操作系统或硬件调用，执行必要的操作来响应和处理中断请求。

**参数说明**：
- **irq**：表示中断号或中断源的标识符。它指示引发中断的硬件设备或中断控制器
- **dev_id**：
	- 是一个void类型的指针，用于传递设备特定的数据或标识符
	- 通常用于在中断处理程序中区分不同的设备或资源
	- 需要与request_irq函数的dev_id参数保持一致，用于区分共享中断的不同设备，dev_id也可以指向设备数据结构

**返回值类型**：**irqreturn_t**是一个特定类型的枚举值，用于表示中断服务函数的返回状态
```c
enum irqreturn {
    IRQ_NONE = (0 << 0),
    IRQ_HANDLED = (1 << 0),
    IRQ_WAKE_THREAD = (1 << 1),
};

typedef enum irqreturn irqreturn_t;
```

它可以有以下几种取值，告诉系统中断处理的结果
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

## 3 中断控制与资源管理

详细内容可以参考[6_禁止或开启中断(补充API)](6_禁止或开启中断(补充API).md)

## 4 总结

通过这个章节的学习，我们掌握了Linux中断编程的核心API：

> [!abstract]+ 摘要或总结
> 
> **核心技能掌握**：
> 
> - 理解了中断申请和释放的完整流程：request_irq → 中断处理 → free_irq
> - 掌握了中断处理函数的编写规范和返回值含义
> - 学会了单个中断和全局中断的控制方法
> - 了解了中断标志的配置和共享中断的处理方式
> - 认识了正确的资源管理和错误处理方法

这些API函数是中断驱动开发的基础工具，正确使用它们能够确保我们编写出稳定可靠的驱动程序。在下一章节中，我们将学习设备树中的中断信息配置，了解硬件描述与软件驱动的对接方法。

> [!question]+ 常见问题
> 
> **Q**: 什么时候应该使用共享中断？ 
> **A**: 当多个设备必须使用同一个中断线时（比如PCI设备），就需要使用IRQF_SHARED标志。每个设备的处理函数都需要检查是否是自己的设备产生的中断，如果不是则返回IRQ_NONE。