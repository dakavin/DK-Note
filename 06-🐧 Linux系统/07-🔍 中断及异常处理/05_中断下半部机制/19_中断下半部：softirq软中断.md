
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b19e4443509badb883ad072b733223dd.png)


📢本篇章将详细介绍软中断。

## 软中断的定义

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/07212752bf384d1149ae2fa3e6f7e984.png)


软中断（softirq）是中断处理程序在开启中断的情况下执行的部分，可以被硬中断抢占内核定义了一张软中断向量表，每种软中断有一个唯一的编号，对应一个softirq_actior实例，softirq_action实例的成员action是处理函数.

```C
<kernel/softirq.cstatic struct softirq_action softirq_vec[NR_SOFTIRQS] __cacheline_aligned_in_smp;...enum{
  HI_SOFTIRQ=0,
  TIMER_SOFTIRQ,
  NET_TX_SOFTIRQ,
  NET_RX_SOFTIRQ,
  BLOCK_SOFTIRQ,
  IRQ_POLL_SOFTIRQ,
  TASKLET_SOFTIRQ,
  SCHED_SOFTIRQ,
  HRTIMER_SOFTIRQ,
  RCU_SOFTIRQ,    /* Preferable RCU should always be the last softirq */
  NR_SOFTIRQS
};
```

```C
<include/linux/interrupt.hstruct softirq_action{void        (*action)(struct softirq_action *);};
```

softirq_action该结构体非常简单就是一个函数指针，用于指向具体定义的函数。

**软中断的种类：**  目前内核定义了10种软中断，各种软中断的编号如下：

1. HI_SOFTIRQ：高优先级的小任务。
    
2. TIMER_SOFTIRQ：定时器软中断。
    
3. NET_TX_SOFTIRQ：网络栈发送报文的软中断。
    
4. NET_RX_SOFTIRQ：网络栈接收报文的软中断。
    
5. BLOCK_SOFTIRO：块设备软中断。
    
6. IRQ_POLL_SOFTIRQ：支持I/O轮询的块设备软中断。
    
7. TASKLET SOFTIRQ：低优先级的小任务。
    
8. SCHED `_SOFTIRQ`：调度软中断，用于在处理器之间负载均衡。
    
9. HRTIMER SOFTIRQ：高精度定时器，这种软中断已经被废弃，目前在中断处理程序的上半部处理高精度定时器。
    
10. RCU SOFTIRO：RCU软中断。
    

软中断的编号形成了优先级顺序，编号小的软中断优先级高。

## 注册软中断的处理函数

函数open_softirq()用来注册软中断的处理函数，在软中断向量表中为指定的软中断编号设置处理函数。

```C
<kernel/softirq.c
void open_softirq(int nr, void (*action)(struct softirq_action *))
```

同一种软中断的处理函数可以在多个处理器上同时执行，处理函数必须是可以重入的，需要使用锁保护临界区。

  

## 触发软中断


函数raise_softirq用来触发软中断，参数是软中断编号。

```C
void raise_softirq(unsigned int nr){unsigned long flags;local_irq_save(flags);raise_softirq_irqoff(nr);local_irq_restore(flags);}
```

该函数调用raise_softirq_irqoff(),raise_softirq_irqoff()是在已经禁止中断的情况下调用函数来触发软中断。

```C
inline void raise_softirq_irqoff(unsigned int nr){__raise_softirq_irqoff(nr);/*
         * If we're in an interrupt or softirq, we're done
         * (this also catches softirq-disabled code). We will
         * actually run the softirq once we return from
         * the irq or softirq.
         *
         * Otherwise we wake up ksoftirqd to make sure we
         * schedule the softirq soon.
         */if (!in_interrupt() && should_wake_ksoftirqd())wakeup_softirqd();}
```

调用__raise_softirq_irqoff函数，如下：

```C
void __raise_softirq_irqoff(unsigned int nr){lockdep_assert_irqs_disabled();trace_softirq_raise(nr);or_softirq_pending(1UL << nr);}
```

把宏or_softirq_pending展开以后是：

```C
irq_stat[smp processor_id()].softirq_pending |= (1UL << nr);
```

- raise_softirq_irqoff函数设定本CPU上的softirq_pending的某个bit等于1，具体的bit是由soft irq number（nr参数）指定的。
    
- 如果在中断上下文，我们只要set `__softirq_pending`的某个bit就OK了，在中断返回的时候自然会进行软中断的处理。但是，如果在context上下文调用这个函数的时候，我们必须要调用wakeup_softirqd函数用来唤醒本CPU上的softirqd这个内核线程。
    

这样softirq就相当于准备好了，在合适的时机将会调用softirq的处理函数。

## 执行软中断


内核执行软中断的地方如下。

1. 在中断处理程序的后半部分执行软中断，对执行时间有限制：不能超过2毫秒，并且最多执行10次。
    
2. 每个处理器有一个软中断线程，调度策略是SCHED_NORMAL，优先级是120。
    
3. 开启软中断的函数local_bh_enable()。
    

如果开启了强制中断线程化的配置宏CONFIG_IRO_FORCED_THREADING，并且在引导内核的时候指定内核参数“threadirqs”，那么所有软中断由软中断线程执行

### 中断处理程序执行软中断

在中断处理程序的后半部分，调用函数irq_exit()以退出中断上下文，处理软中断，其代码如下：

```C
<kernel/softirq.c>
void irq_exit(void)
{
  __irq_exit_rcu();
  rcu_irq_exit();
   /* must be last! */
  lockdep_hardirq_exit();
}
...
static inline void __irq_exit_rcu(void)
{
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

如果in_interrupt()为真，表示在不可屏蔽中断、硬中断或软中断上下文，或者禁止软中断。如果正在处理的硬中断没有抢占正在执行的软中断，没有禁止软中断，并且当前处理器的待处理软中断位图不是空的，那么调用函数invoke_softirq()来处理软中断。

函数invoke_softirq的代码如下：kernel/softirq.c

```C
static inline void invoke_softirq(void)
{
    /* 1. 如果软中断线程处于就绪状态或运行状态，那么让软中断线程执行软中断 */
  if (ksoftirqd_running(local_softirq_pending()))
    return;

    /* 2. 如果没有强制中断线程化，那么调用函数_do_softirq()执行软中断 */
  if (!force_irqthreads || !__this_cpu_read(ksoftirqd)) {
#ifdef CONFIG_HAVE_IRQ_EXIT_ON_IRQ_STACK
    /*
                 * We can safely execute softirq on the current stack if
                 * it is the irq stack, because it should be near empty
                 * at this stage.
                 */
    __do_softirq();
#else
    /*
                 * Otherwise, irq_exit() is called on the task stack that can
                 * be potentially deep already. So call softirq in its own stack
                 * to prevent from any overrun.
                 */
    do_softirq_own_stack();
#endif
  } else {
        /* 3. 如果强制中断线程化，那么唤醒软中断线程执行软中断*/
    wakeup_softirqd();
  }
}
```

函数_do_softirq是执行软中断的核心函数，其主要代码如下：

```C
<kernel/softirq.c>
asmlinkage __visible void __softirq_entry __do_softirq(void)
{
  unsigned long end = jiffies + MAX_SOFTIRQ_TIME;
  unsigned long old_flags = current->flags;
  int max_restart = MAX_SOFTIRQ_RESTART;
  struct softirq_action *h;
  bool in_hardirq;
  __u32 pending;
  int softirq_bit;

  current->flags &= ~PF_MEMALLOC;

    /* 1. 把局部变量pending设置为当前处理器的待处理软中断位图 */
  pending = local_softirq_pending();

    /* 2. softirq_handle_begin调用__local_bh_disable_ip 把抢占计数器的软中断计数加1 */
  softirq_handle_begin();
  in_hardirq = lockdep_softirq_start();
  account_softirq_enter(current);

restart:
  /* 3. 把当前处理器的待处理软中断位图重新设置为0 */
  set_softirq_pending(0);

    /* 4. 开启硬中断 */
  local_irq_enable();

  h = softirq_vec;

    /* 5. 从低位向高位扫描待处理软中断位图，针对每个设置了对应位的转中断编号，执行软中断的处理函数*/
  while ((softirq_bit = ffs(pending))) {
    unsigned int vec_nr;
    int prev_count;

    h += softirq_bit - 1;

    vec_nr = h - softirq_vec;
    prev_count = preempt_count();

    kstat_incr_softirqs_this_cpu(vec_nr);

    trace_softirq_entry(vec_nr);
        /* 调用action */
    h->action(h);
    trace_softirq_exit(vec_nr);
    if (unlikely(prev_count != preempt_count())) {
      pr_err("huh, entered softirq %u %s %p with preempt_count %08x, exited with %08x?\n",
             vec_nr, softirq_to_name[vec_nr], h->action,
             prev_count, preempt_count());
      preempt_count_set(prev_count);
    }
    h++;
    pending >>= softirq_bit;
  }

  if (!IS_ENABLED(CONFIG_PREEMPT_RT) &&
      __this_cpu_read(ksoftirqd) == current)
    rcu_softirq_qs();

    /* 6. 禁止硬中断 */
  local_irq_disable();

    /* 7. 如果软中断的处理函数又触发软中断，处理如下 */
  pending = local_softirq_pending();
  if (pending) {
        /* 8. 如果软中断的执行时间小于2毫秒，不需要重新调度进程，并口且软中断的执行次数没超过10，那么跳转到restart继续执行软中断 */
    if (time_before(jiffies, end) && !need_resched() &&
        --max_restart)
      goto restart;

        /* 9. 唤醒软中断线程执行软中断 */
    wakeup_softirqd();
  }

  account_softirq_exit(current);
  lockdep_softirq_end(in_hardirq);
    /* 10. 把抢占计数器的软中断计数减1 */
  softirq_handle_end();
  current_restore_flags(old_flags, PF_MEMALLOC);
}
```

上面就是软中断的调用流程。

### 软中断线程

每个处理器有一个软中断线程，名称是“ksofirqd/”后面跟着处理器编号，调度策略SCHED_NORMAL，优先级是120。软中断线程的核心函数是run_ksoftirqd()，其代码如下

```C
<kernel/softirq.c>
static void run_ksoftirqd(unsigned int cpu)
{
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
...
static struct smp_hotplug_thread softirq_threads = {
  .store                        = &ksoftirqd,
  .thread_should_run        = ksoftirqd_should_run,
  .thread_fn                = run_ksoftirqd,
  .thread_comm                = "ksoftirqd/%u",
};
static __init int spawn_ksoftirqd(void)
{
  cpuhp_setup_state_nocalls(CPUHP_SOFTIRQ_DEAD, "softirq:dead", NULL,
          takeover_tasklets);
  BUG_ON(smpboot_register_percpu_thread(&softirq_threads));

  return 0;
}
early_initcall(spawn_ksoftirqd);
...
static int __smpboot_create_thread(struct smp_hotplug_thread *ht, unsigned int cpu)
{
  struct task_struct *tsk = *per_cpu_ptr(ht->store, cpu);
  struct smpboot_thread_data *td;
    ...
  tsk = kthread_create_on_cpu(smpboot_thread_fn, td, cpu,
            ht->thread_comm);
  if (IS_ERR(tsk)) {
    kfree(td);
    return PTR_ERR(tsk);
  }
  kthread_set_per_cpu(tsk, cpu);
    ...
  return 0;
}
```

这里创建一个线程，然后线程中执行run_ksoftirqd函数，run_ksoftirqd函数里执行__do_softirq()函数。

## 抢占计数器

在介绍“禁止/开启软中断”之前，首先了解一下抢占计数器这个背景知识。

每个进程的thread_info结构体有一个抢占计数器：int preempt_count，它用来表示当前进程能不能被抢占。

抢占是指当进程在内核模式下运行的时候可以被其他进程抢占，如果优先级更高的进程处于就绪状态，强行剥夺当前进程的处理器使用权。

但是有时候进程可能在执行一些关键操作，不能被抢占，所以内核设计了抢占计数器。如果抢占计数器为0，表示可以被抢占；如果抢占计数器不为0，表示不能被抢占。当中断处理程序返回的时候，如果进程在被打断的时候正在内核模式下执行，就会检查抢占计数器是否为0。如果抢占计数器是0，可以让优先级更高的进程抢占当前进程虽然抢占计数器不为0意味着禁止抢占，但是内核进一步按照各种场景对抢占计数器的位进行了划分，

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5c43cdc4cae37f86724cbb53e19bbbfa.png)


其中第0(7位是抢占计数，第8)15位是软中断计数，第16~19位是硬中断计数第20位是不可屏蔽中断（Non Maskable Interrupt，NMI）计数。

```C
#define PREEMPT_MASK        (__IRQ_MASK(PREEMPT_BITS) << PREEMPT_SHIFT)
#define SOFTIRQ_MASK        (__IRQ_MASK(SOFTIRQ_BITS) << SOFTIRQ_SHIFT)
#define HARDIRQ_MASK        (__IRQ_MASK(HARDIRQ_BITS) << HARDIRQ_SHIFT)
#define NMI_MASK        (__IRQ_MASK(NMI_BITS)     << NMI_SHIFT)
...
#define PREEMPT_BITS        8
#define SOFTIRQ_BITS        8
#define HARDIRQ_BITS        4
#define NMI_BITS            4
```

各种场景分别利用各自的位禁止或开启抢占。

1. 普通场景（PREEMPT_MASK）：对应函数preempt_disable()和preempt_enable() 。
    
2. 软中断场景（SOFTIRO_MASK）：对应函数local_bh_disable()和local_bh_enabe() 。
    
3. 硬中断场景（HARDIRQ_MASK）：对应函数 _irq_enter()和_irq_exit()。
    
4. 不可屏蔽中断场景（NMI MASK）：对应函数nmi_enter()和nmi_exit() 。
    

反过来，我们可以通过抢占计数器的值判断当前处在什么场景：

```C
<include/linux/preempt.h>
/*
 * Macros to retrieve the current execution context:
 *
 * in_nmi()                - We're in NMI context
 * in_hardirq()                - We're in hard IRQ context
 * in_serving_softirq()        - We're in softirq context
 * in_task()                - We're in task context
 */
#define in_nmi()                (nmi_count())
#define in_hardirq()                (hardirq_count())
#define in_serving_softirq()        (softirq_count() & SOFTIRQ_OFFSET)
#define in_task()                (!(in_nmi() | in_hardirq() | in_serving_softirq()))

/*
 * The following macros are deprecated and should not be used in new code:
 * in_irq()       - Obsolete version of in_hardirq()
 * in_softirq()   - We have BH disabled, or are processing softirqs
 * in_interrupt() - We're in NMI,IRQ,SoftIRQ context or have BH disabled
 */
#define in_irq()                (hardirq_count())
#define in_softirq()                (softirq_count())
#define in_interrupt()                (irq_count())
```

- in_irq()表示硬中断场景，也就是正在执行硬中断。
    
- in_softirq()表示软中断场景，包括禁止软中断和正在执行软中断。
    
- in_interrupt()表示正在执行不可屏蔽中断、硬中断或软中断，或者禁止软中断。
    
- in_serving_softirq()表示正在执行软中断。
    
- in_nmi()表示不可屏蔽中断场景。
    
- in_task()表示普通场景，也就是进程上下文。
    

## 禁止/开启软中断


如果进程和软中断可能访问同一个对象，那么进程和软中断需要互斥，进程需要禁止软中断。禁止软中断的函数是local_bh_disable()，注意：这个函数只能禁止本处理器的软中断，不能禁止其他处理器的软中断。该函数把抢占计数器的软中断计数加2，其代码如下：

```C
static inline void local_bh_disable(void)
{
  __local_bh_disable_ip(_THIS_IP_, SOFTIRQ_DISABLE_OFFSET);
}
...
#define SOFTIRQ_DISABLE_OFFSET        (2 * SOFTIRQ_OFFSET)
```

调用__local_bh_disable_ip函数:

```C
static __always_inline void __local_bh_disable_ip(unsigned long ip, unsigned int cnt)
{
  preempt_count_add(cnt);
  barrier();
}
```

开启软中断的函数是local_bh_enable()，该函数把抢占计数器的软中断计数减2。为什么禁止软中断的函数local_bh_disable()把抢占计数器的软中断计数加2，而不是加 1呢？目的是区分禁止软中断和正在执行软中断这两种情况。执行软中断的函数__do_sofir()把抢占计数器的软中断计数加1。如果软中断计数是奇数，可以确定正在执行软中断。