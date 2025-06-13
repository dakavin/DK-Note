
## 中断号

每个中断都有一个中断号，通过中断号即可区分不同的中断，有的资料也把中断号叫做中断线， 在 Linux 内核中使用一个 int 变量表示中断号。

GPIO---

## request_irq 函数

在 Linux 内核中要想使用某个中断是需要申请的， request_irq 函数用于申请中断， request_irq函数可能会导致睡眠，因此不能在中断上下文或者其他禁止睡眠的代码段中使用 request_irq 函数。 request_irq 函数会激活(使能)中断，所以不需要我们手动去使能中断， request_irq 函数原型如下：

```C++
int request_irq(unsigned int irq,
    irq_handler_t handler,
    unsigned long flags,
    const char *name,
    void *dev)
```

  

**irq**：要申请中断的中断号。

**handler**：中断处理函数，当中断发生以后就会执行此中断处理函数。

**flags**：中断标志，可以在文件 include/linux/interrupt.h 里面查看所有的中断标志。

**name**：中断名字，设置以后可以在/proc/interrupts 文件中看到对应的中断名字。

**dev**： 如果将 flags 设置为 **IRQF_SHARED** 的话， **dev 用来区分不同的中断**，一般情况下将dev 设置为设备结构体， dev 会传递给中断处理函数 irq_handler_t 的第二个参数。

**返回值：** 0 中断申请成功，其他负值 中断申请失败，如果返回-EBUSY 的话表示中断已经被申请了。

这里我们介绍几个常用的中断标志 ：

- IRQF_SHARED多个设备共享一个中断线，共享的所有中断都必须指定此标志。如果使用共享中断的话， request_irq 函数的 dev 参数就是唯一区分他们的标志。
    
- IRQF_ONESHOT 单次中断，中断执行一次就结束。
    
- IRQF_TRIGGER_NONE 无触发。
    
- IRQF_TRIGGER_RISING 上升沿触发。
    
- IRQF_TRIGGER_FALLING 下降沿触发。
    
- IRQF_TRIGGER_HIGH 高电平触发。
    
- IRQF_TRIGGER_LOW 低电平触发。
    

## **free_irq** 函数

使用中断的时候需要通过 request_irq 函数申请，使用完成以后就要通过 free_irq 函数释放掉相应的中断。如果中断不是共享的，那么 free_irq 会删除中断处理函数并且禁止中断。 free_irq函数原型如下所示：

```C++
void free_irq(unsigned int irq, void *dev_id)  
```

函数参数和返回值含义如下：

**irq**： 要释放的中断。

**dev_id**：如果中断设置为共享(IRQF_SHARED)的话，此参数用来区分具体的中断。共享中断只有在释放最后中断处理函数的时候才会被禁止掉。

**返回值**：无。

## 中断处理函数（上半部驱动）**

使用 request_irq 函数申请中断的时候需要设置中断处理函数，中断处理函数格式如下所示：

```C++
irqreturn_t (*irq_handler_t) (int, void *)  
```

第一个参数是要中断处理函数要相应的中断号。第二个参数是一个指向 void 的指针，也就是个通用指针，需要与 request_irq 函数的 dev_id 参数保持一致。用于区分共享中断的不同设备， dev_id 也可以指向设备数据结构。中断处理函数的返回值为 irqreturn_t 类型， irqreturn_t 类型定义如下所示：

```C++
enum irqreturn {

    IRQ_NONE = (0 << 0),
    IRQ_HANDLED = (1 << 0),
    IRQ_WAKE_THREAD = (1 << 1),
};  

typedef enum irqreturn irqreturn_t;  
```

可以看出 irqreturn_t 是个枚举类型，一共有三种返回值。一般中断服务函数返回值使用如下形式：

```C++
return IRQ_RETVAL(IRQ_HANDLED)  
```

## 中断使能与禁止函数

常用的中断使用和禁止函数如下所示：

```C++
void enable_irq(unsigned int irq)（CPU1 CPU2）
void disable_irq(unsigned int irq)  
```

enable_irq 和 disable_irq 用于使能和禁止指定的中断， irq 就是要禁止的中断号。 disable_irq函数要等到当前正在执行的中断处理函数执行完才返回，因此使用者需要保证不会产生新的中断，并且确保所有已经开始执行的中断处理程序已经全部退出。在这种情况下，可以使用另外一个中断禁止函数：

```C++
void disable_irq_nosync(unsigned int irq)  
```

disable_irq_nosync 函数调用以后立即返回，不会等待当前中断处理程序执行完毕。

上面三个函数都是使能或者禁止某一个中断，有时候我们需要关闭当前处理器的整个中断系统也就是关闭全局中断，这个时候可以使用如下两个函数：

  

CPU1 ----- local_irq_disable

CPU2

```C++
local_irq_enable()（多CPU）
local_irq_disable()  
```

local_irq_enable 用于使能当前处理器中断系统， **local_irq_disable 用于禁止当前处理器中断系统**。假如 A 任务调用 local_irq_disable 关闭全局中断 10S，当关闭了 2S 的时候 B 任务开始运行， B 任务也调用 local_irq_disable 关闭全局中断 3S， 3 秒以后 B 任务调用 local_irq_enable 函数将全局中断打开了。此时才过去 2+3=5 秒的时间，然后全局中断就被打开了，此时 A 任务要关闭 10S 全局中断的愿望就破灭了，然后 A 任务就“生气了”，结果很严重，可能系统都要被A 任务整崩溃。为了解决这个问题， B 任务不能直接简单粗暴的通过 local_irq_enable 函数来打开全局中断，而是将中断状态恢复到以前的状态，要考虑到别的任务的感受，此时就要用到下面两个函数：

```C++
local_irq_save(flags) //推荐
local_irq_restore(flags)
```

这两个函数是一对， local_irq_save 函数用于禁止中断，并且将中断状态保存在 flags 中。local_irq_restore 用于恢复中断，将中断到 flags 状态。