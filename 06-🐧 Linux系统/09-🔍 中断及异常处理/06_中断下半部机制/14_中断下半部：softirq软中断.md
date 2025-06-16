
现在我们需要学习一个中断处理中的重要机制：**软中断是如何工作的？**

简单来说，**软终端（softirq）** 就像餐厅的“后厨处理系统”。当服务员（硬件中断）接到客人点餐后，不会在客人面前慢慢做菜，而是快速记录订单，然后吧复杂的烹饪工作交给后厨（软中断）来处理。这样既保证了对客人的快速响应，又能完成复杂的烹饪任务。

> [!note]+ 重要笔记
> 软中断是Linux中断处理的下半部机制之一，它允许中断处理程序在开启中断的情况下执行耗时较长的处理工作，可以被硬件中断抢占，提高了系统的响应性和并发处理能力

## 1 软终端基础概念

### 1.1 什么是软中断

**软中断**是中断处理程序在开启中断（解锁，不占据）的情况下执行的部分，它可以被硬中断抢占。换句话说，软中断就像是"可以被打断的工作"，当有更紧急的硬中断到来时，软中断会暂停处理，等硬中断完成后再继续。

内核定义了一张 **软中断向量表** ，每种软中断都有唯一的编号，对应一个`softirq_action`实例
```c
// kernel/softirq.c
static struct softirq_action_softirq_vec[NR_SORTIRQS] __cacheline_aligned_in_smp;

enum{
	HI_SOFTIRQ = 0,
	TIMER_SOFTIRQ,
	NET_TX_SOFTIRQ,
	NET_RX_SOFTIRQ,
	BLOCK_SOFTIRQ,
	IRQ_POLL_SOFTIRQ,
	TASKLET_SOFTIRQ,
	SCHED_SOFTIRQ,
	HRTIMER_SOFTIRQ,
	RCU_SOFTIRQ,     /* Preferable RCU should always be the last softirq */
	NR_SOFTIRQ,
}
```
- 这个枚举定义了10种不同类型的软中断，每种软中断都有特定的用途和优先级，**编号越小优先级越高**

**softirq_action结构体**
```c
// include/linux/interrupt.h
struct softirq_action{
	void (*action)(struct softirq_action *);
};
```

> [!tip]+ 重要提示
> 
> softirq_action结构体非常简单，就是一个函数指针，用于指向具体的处理函数。这种设计保证了接口的简洁性和高效性。

### 1.2 软中断类型详解

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/42f6170470b659a35e2137c187549977.png)

如上图所示，目前内核定义了 **10种软中断** ，各种软中断的功能和优先级如下：

**高优先级软中断**：
- `HI_SOFTIRQ`：高优先级的小任务，用于紧急处理
- `TIMER_SOFTIRQ`：定时器软中断，处理定时器到期事件

**网络相关软中断**：
- `NET_TX_SOFTIRQ`：网络栈发送报文软中断
- `NET_RX_SOFTIRQ`：网络栈接收报文软中断

**存储和调度软中断**：
- `BLOCK_SOFTIRQ`：块设备软中断，处理磁盘I/O完成
- `IRQ_POLL_SOFTIRQ`：支持I/O轮询的块设备软中断
- `SCHED_SOFTIRQ`：调度软中断，用于处理器之间的负载均衡

**其他功能软中断**：
- `TASKLET_SOFTIRQ`：低优先级的小任务
- `HRTIMER_SOFTIRQ`：高精度定时器（已废弃）
- `RCU_SOFTIRQ`：RCU（读拷贝更新）软中断

> [!example]+ 示例
> 
> 网卡接收数据时，硬中断快速读取数据包到内存，然后触发NET_RX_SOFTIRQ软中断来处理复杂的协议栈处理工作，这样避免了硬中断处理时间过长。

> [!warning]+ 警告或注意
> 
> 软中断的编号形成了优先级顺序，编号小的软中断优先级高。这个优先级关系在系统负载高时会影响处理顺序。

### 1.3 软中断注册机制

`open_softirq()`函数用来注册软中断的处理函数，在软中断向量表中为指定的软中断编号设置处理函数
```c
// kernel/softirq.c
void open_softirq(int nr, void (*action)(struct softirq_action *));
```
- `nr`：软中断编号，对应前面枚举中的某个值
- `action`：软中断处理函数指针

> [!warning]+ 警告或注意
> 
> 同一种软中断的处理函数可以在多个处理器上同时执行，因此处理函数必须是可重入的，需要使用锁保护临界区。这与硬中断处理函数的要求类似。

## 2 软中断的触发和执行

### 2.1 软中断的触发机制

`raise_softirq`函数用来触发软中断，参数是软中断编号：
```c
void raise_softirq(unsigned int nr){
	unsigned long flags;
	local_irq_save(flags);
	raise_softirq_irqoff(nr);
	local_irq_restore(flags);
}
```
- 该函数首先保存中断状态并禁用中断
- 然后调用raise_softirq_irqoff()
- 最后恢复中断状态
- **确保了触发过程的原子性**

`raise_softirq_irqoff`函数是在已经禁止中断的情况下调用的核心函数：
```c
inline void raise_softirq_irqoff(unsigned int nr) {
    __raise_softirq_irqoff(nr);
    /*
     * If we're in an interrupt or softirq, we're done
     * (this also catches softirq-disabled code). We will
     * actually run the softirq once we return from
     * the irq or softirq.
     *
     * Otherwise we wake up ksoftirqd to make sure we
     * schedule the softirq soon.
     */
    if (!in_interrupt() && should_wake_ksoftirqd())
        wakeup_softirqd();
}
```

`__raise_softirq_irqoff` 核心处理函数
```c
void __raise_softirq_irqoff(unsigned int nr) {
    lockdep_assert_irqs_disabled();
    trace_softirq_raise(nr);
    or_softirq_pending(1UL << nr);
}
```

把宏`or_softirq_pending`展开后是：

```c
irq_stat[smp_processor_id()].softirq_pending |= (1UL << nr);
```

> [!tip]+ 重要提示
> 
> 这个函数设定本CPU上的softirq_pending的某个bit等于1，具体的bit由软中断编号决定。如果在中断上下文中，只需要设置标志位即可，中断返回时会自动处理。如果在普通进程上下文中，还需要唤醒ksoftirqd内核线程。

换句话说，触发软中断就像是"点亮警示灯"，告诉系统某种类型的软中断需要处理，系统会在合适的时机执行对应的处理函数

### 2.2 中断处理程序执行软中断

内核执行软中断的主要时机有3种
1. **中断处理程序的后半部分**：对执行时间有限制，不能超过2ms，最多执行10次
2. **软中断线程**：每个处理器有一个专门的线程用于处理软中断
3. **开启软中断时**：调用`local_bh_enable()`函数时

---

在中断处理程序的后半部分，调用`irq_exit()`函数退出中断上下文时会处理软中断，可以参考[5 中断上下文管理](../04_深入理解/11_内核异常处理过程.md#5%20中断上下文管理)
```c
// kernel/softirq.c
// kernel/softirq.c
void irq_exit(void) {
    __irq_exit_rcu();
    rcu_irq_exit();
    /* must be last! */
    lockdep_hardirq_exit();
}

static inline void __irq_exit_rcu(void) {
#ifndef __ARCH_IRQ_EXIT_IRQS_DISABLED
    local_irq_disable();
#else
    lockdep_assert_irqs_disabled();
#endif
    account_hardirq_exit(current);
    preempt_count_sub(HARDIRQ_OFFSET);
    if (!in_interrupt() && local_softirq_pending())
        invoke_softirq();

    tick_irq_exit();
}
```
**代码说明**：如果当前不在中断上下文中，且有待处理的软中断，就调用invoke_softirq()来处理软中断。

其中`invoke_softirq`函数是软中断执行的调度函数
```c
static inline void invoke_softirq(void) {
    /* 1. 如果软中断线程处于就绪或运行状态，让软中断线程执行 */
    if (ksoftirqd_running(local_softirq_pending()))
        return;

    /* 2. 如果没有强制中断线程化，调用__do_softirq()执行软中断 */
    if (!force_irqthreads || !__this_cpu_read(ksoftirqd)) {
#ifdef CONFIG_HAVE_IRQ_EXIT_ON_IRQ_STACK
        __do_softirq();
#else
        do_softirq_own_stack();
#endif
    } else {
        /* 3. 如果强制中断线程化，唤醒软中断线程执行软中断 */
        wakeup_softirqd();
    }
}
```

其中`__do_softirq()`函数是执行软中断的核心函数
```c
asmlinkage __visible void __softirq_entry __do_softirq(void) {
    unsigned long end = jiffies + MAX_SOFTIRQ_TIME;
    unsigned long old_flags = current->flags;
    int max_restart = MAX_SOFTIRQ_RESTART;
    struct softirq_action *h;
    bool in_hardirq;
    __u32 pending;
    int softirq_bit;

    current->flags &= ~PF_MEMALLOC;

    /* 1. 获取当前处理器的待处理软中断位图 */
    pending = local_softirq_pending();

    /* 2. 增加软中断计数，禁止抢占 */
    softirq_handle_begin();
    in_hardirq = lockdep_softirq_start();
    account_softirq_enter(current);

restart:
    /* 3. 清除待处理软中断位图 */
    set_softirq_pending(0);

    /* 4. 开启硬中断 */
    local_irq_enable();

    h = softirq_vec;

    /* 5. 从低位向高位扫描，执行每个待处理的软中断 */
    while ((softirq_bit = ffs(pending))) {
        unsigned int vec_nr;
        int prev_count;

        h += softirq_bit - 1;
        vec_nr = h - softirq_vec;
        prev_count = preempt_count();

        kstat_incr_softirqs_this_cpu(vec_nr);
        trace_softirq_entry(vec_nr);
        /* 调用软中断处理函数 */
        h->action(h);
        trace_softirq_exit(vec_nr);

        h++;
        pending >>= softirq_bit;
    }

    /* 6. 禁止硬中断 */
    local_irq_disable();

    /* 7. 检查是否有新的软中断被触发 */
    pending = local_softirq_pending();
    if (pending) {
        /* 8. 如果执行时间未超时且重启次数未超限，继续处理 */
        if (time_before(jiffies, end) && !need_resched() && --max_restart)
            goto restart;

        /* 9. 否则唤醒软中断线程处理 */
        wakeup_softirqd();
    }

    account_softirq_exit(current);
    lockdep_softirq_end(in_hardirq);
    /* 10. 恢复抢占计数 */
    softirq_handle_end();
    current_restore_flags(old_flags, PF_MEMALLOC);
}
```

> [!example]+ 示例
> 
> 这个处理流程就像流水线作业：先关闭"新订单接收"（清除标志位），然后"开门营业"（开启硬中断），按优先级处理所有待处理任务，如果处理过程中又来了新任务且时间充足就继续处理，否则交给专门的工作线程处理。

### 2.3 软中断线程机制

每个处理器都有一个 **软中断线程**，名称是 `ksoftirqd/` 后面跟着处理器编号，使用SCHED_NORMAL调度策略，优先级是120

**软中断线程的核心函数run_ksoftirqd**：
```c
static void run_ksoftirqd(unsigned int cpu) {
    ksoftirqd_run_begin();
    if (local_softirq_pending()) {
        /*
         * We can safely run softirq on inline stack, as we are not deep
         * in the task stack here.
         */
        __do_softirq();
        ksoftirqd_run_end();
        cond_resched();
        return;
    }
    ksoftirqd_run_end();
}
```

**软中断线程的创建**
```c
static struct smp_hotplug_thread softirq_threads = {
    .store                  = &ksoftirqd,
    .thread_should_run      = ksoftirqd_should_run,
    .thread_fn              = run_ksoftirqd,
    .thread_comm            = "ksoftirqd/%u",
};

static __init int spawn_ksoftirqd(void) {
    cpuhp_setup_state_nocalls(CPUHP_SOFTIRQ_DEAD, "softirq:dead", NULL, takeover_tasklets);
    BUG_ON(smpboot_register_percpu_thread(&softirq_threads));
    return 0;
}
early_initcall(spawn_ksoftirqd);
```

> [!tip]+ 重要提示
> 
> 软中断线程的设计保证了即使在系统负载很高的情况下，软中断也能得到及时处理，避免了硬中断处理时间过长的问题。

## 3 抢占计数器与软中断控制

### 3.1 抢占计数器原理

在介绍软中断的控制机制之前，我们需要理解抢占计数器这个重要概念。

每个进程的thread_info结构体都有一个抢占计数器：`int preempt_count`，它用来表示当前进程不能被抢占。换句话说，抢占计数器就是进程的“免打扰模式开关”

**抢占机制的基本原理**
- 抢占是指当前进程在内核模式下运行时可以被其他优先级更高的进程强行剥夺处理器使用权
- 如果抢占计数器为0，表示可以被抢占
- 如果抢占计数器不为0，表示能被抢占

**抢占计数器的位字段划分**：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5c43cdc4cae37f86724cbb53e19bbbfa.png)
```c
#define PREEMPT_MASK    (__IRQ_MASK(PREEMPT_BITS) << PREEMPT_SHIFT)
#define SOFTIRQ_MASK    (__IRQ_MASK(SOFTIRQ_BITS) << SOFTIRQ_SHIFT)
#define HARDIRQ_MASK    (__IRQ_MASK(HARDIRQ_BITS) << HARDIRQ_SHIFT)
#define NMI_MASK        (__IRQ_MASK(NMI_BITS)     << NMI_SHIFT)

#define PREEMPT_BITS    8
#define SOFTIRQ_BITS    8
#define HARDIRQ_BITS    4
#define NMI_BITS        4
```

- 0~7：抢占计数，对应函数preempt_disable和preempt_enable
- 8~15：软中断计数，对应函数local_bh_disable和local_bh_enable
- 16~19：硬中断计数，对应函数irq_enter和irq_exit
- 20：不可屏蔽中断（NMI）计数，对应函数nmi_enter和nmi_exit

**上下文判断函数**：
```c
#define in_nmi()                (nmi_count())
#define in_hardirq()            (hardirq_count())
#define in_serving_softirq()    (softirq_count() & SOFTIRQ_OFFSET)
#define in_task()               (!(in_nmi() | in_hardirq() | in_serving_softirq()))

#define in_irq()                (hardirq_count())
#define in_softirq()            (softirq_count())
#define in_interrupt()          (irq_count())
```

> [!example]+ 示例
> 
> 这些函数就像是"身份识别器"，让内核能够准确判断当前处于什么执行环境：
> 
> - in_hardirq()：正在执行硬中断
> - in_softirq()：正在执行软中断或已禁用软中断
> - in_interrupt()：正在执行任何类型的中断或已禁用软中断
> - in_task()：普通进程上下文

### 3.2 禁止和开启软中断

当进程和软中断可能访问同一个对象时，进程需要禁止软中断来实现互斥保护

**禁止软终端的函数local_bh_disable**
```c
static inline void local_bh_disable(void) {
    __local_bh_disable_ip(_THIS_IP_, SOFTIRQ_DISABLE_OFFSET);
}

#define SOFTIRQ_DISABLE_OFFSET  (2 * SOFTIRQ_OFFSET)
```

**具体实现函数**：
```c
static __always_inline void __local_bh_disable_ip(unsigned long ip, unsigned int cnt) {
    preempt_count_add(cnt);
    barrier();
}
```
**代码说明**：该函数把抢占计数器的软中断计数加2，注意这里是加2而不是加1

**开启软中断的函数local_bh_enable()**：该函数把抢占计数器的软中断计数减2。

> [!question]+ 为什么禁止软中断要加2而不是加1？
> 
> 这是为了区分"禁止软中断"和"正在执行软中断"两种情况：
> 
> - 执行软中断时：软中断计数加1（奇数）
> - 禁止软中断时：软中断计数加2（偶数）
> - 如果软中断计数是奇数，说明正在执行软中断
> - 如果软中断计数是偶数且非零，说明软中断被禁止

> [!warning]+ 警告或注意
> 
> local_bh_disable()只能禁止本处理器的软中断，不能禁止其他处理器的软中断。在多核系统中，每个处理器都有独立的软中断处理机制。

## 4 总结

通过这个章节的学习，我们深入了解了Linux软中断机制：

> [!abstract]+ 摘要或总结
> 
> **核心知识掌握**：
> 
> - 理解了软中断作为中断下半部机制的重要作用和设计思想
> - 掌握了10种软中断类型的功能和优先级关系
> - 学习了软中断的注册、触发和执行完整流程
> - 了解了软中断线程机制和抢占计数器的工作原理
> - 认识了软中断禁用和启用的控制方法

软中断机制是Linux内核实现高效中断处理的关键技术，它通过将耗时的处理工作从硬中断上下文中分离出来，既保证了硬中断的快速响应，又完成了复杂的处理任务。理解软中断机制对于编写高性能的内核代码和驱动程序至关重要。

> [!question]+ 常见问题
> 
> **Q**: 软中断和硬中断的主要区别是什么？ 
> **A**: 硬中断具有最高优先级，会禁用中断执行，不能被抢占；而软中断在开启中断的情况下执行，可以被硬中断抢占，允许更好的并发处理。硬中断用于紧急处理，软中断用于复杂的后续处理工作。