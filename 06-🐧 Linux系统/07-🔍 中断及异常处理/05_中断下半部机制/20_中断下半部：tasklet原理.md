
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f82c97d575bb8760db1f18c093f56c13.png)


📢 在上半部处理紧急的事情，在上半部的处理过程中，中断是被禁止的； 在下半部处理耗时的事情，在下半部的处理过程中，中断是使能的。

  

## 什么是 tasklet



在 Linux 内核中，tasklet 是一种特殊的软中断机制，被广泛用于处理中断下文相关的任务。

它是一种常见且有效的方法，在多核处理系统上可以避免并发问题。Tasklet 绑定的函数在同一时间只能在一个 CPU 上运行，因此不会出现并发冲突。然而，需要注意的是，tasklet 绑定的函数中不能调用可能导致休眠的函数，否则可能引起内核异常。

在 `Linux` 内核中，`tasklet` 结构体的定义位于 include/linux/interrupt.h 头文件中。其原型如下：

![](https://ncn13z89mqjm.feishu.cn/space/api/box/stream/download/asynccode/?code=MTA5NDk1YTQ1OWE0NzYyM2I2ODFjYmZmMzA3MjE2YzdfMEY4Y0k4WE9jWjEwM3hZdXdxMWJkNk1ycDh1N0pKSlJfVG9rZW46VkdPRmJNVXJKb0Z0Yjl4QkZXbmNJSlJUbmhmXzE3NDk3OTg5OTA6MTc0OTgwMjU5MF9WNA)

```c
struct tasklet_struct
{
    struct tasklet_struct *next;
    unsigned long state;
    atomic_t count;
    bool use_callback;
    union {
        void (*func)(unsigned long data);
        void (*callback)(struct tasklet_struct *);
    };
    unsigned long data;
};
```

tasklet_struct 结构体包含以下成员：

- next:指向下一个tasklet的指针，用于形成链表结构，以便内核中可以同时管理多个tasklet。
    
- state:表示 tasklet 的当前状态。
    
- count:用于引用计数，用于确保 tasklet 在多个地方调度或取消调度时的正确处理。
    
- func:指向 tasklet 绑定的函数的指针，该函数将在 tasklet 执行时被调用。
    
- data:传递给 tasklet 绑定函数的参数
    

此外，为了方便，还定义了 tasklet_t 类型作为 struct tasklet_struct 的别名。这样我们可以使用 tasklet_t 来声明 tasklet 变量，而不是直接使用 struct

## tasklet 接口

---

### 静态初始化函数

在 Linux 内核中，有一个用于静态初始化 tasklet 的宏函数：DECLARE_TASKLET。这个宏函数可以帮助我们更方便地进行 tasklet 的静态初始化。 宏函数的原型如下：
```c
#define DECLARE_TASKLET(name, func, data) \
    struct tasklet_struct name = { NULL, 0, ATOMIC_INIT(0), func, data }
```
其中，name 是 tasklet 的名称，func 是 tasklet 的处理函数，data 是传递给处理函数的参数。初始化状态为使能状态。

如果 tasklet 初始化函数为非使能状态，使用以下宏定义：
```c
#define DECLARE_TASKLET_DISABLED(name, func, data) \
    struct tasklet_struct name = { NULL, 0, ATOMIC_INIT(1), func, data }
```


其中，name 是 tasklet 的名称，func 是 tasklet 的处理函数，data 是传递给处理函数的参数。初始化状态为非使能状态。

下面是一个示例，展示了如何使用 DECLARE_TASKLET 宏函数进行 tasklet 的静态初始化：
```c
#include <linux/interrupt.h>

// 定义 tasklet 处理函数
void my_tasklet_handler(unsigned long data)
{
    // Tasklet 处理逻辑
    // ...
}

// 静态初始化 tasklet
DECLARE_TASKLET(my_tasklet, my_tasklet_handler, 0);
// 驱动程序的其他代码
```

在上述示例中，my_tasklet 是 tasklet 的名称，my_tasklet_handler 是 tasklet 的处理函数，0是传递给处理函数的参数。但是需要注意的是，使用 DECLARE_TASKLET 静态初始化的 tasklet无法在运行时动态销毁，因此在不需要 tasklet 时，应该避免使用此方法。如果需要在运行时销毁 tasklet，应使用 tasklet_init 和 tasklet_kill 函数进行动态初始化和销毁，接下来我们来学习动态初始化函数。

### 动态初始化函数

在 Linux 内核中，可以使用 tasklet_init 函数对 tasklet 进行动态初始化。该函数原型为:

```C
void tasklet_init(struct tasklet_struct *t, void (*func)(unsigned long), unsigned long data);
```

其中，t 是指向 tasklet 结构体的指针，func 是 tasklet 的处理函数，data 是传递给处理函数的参数

以下是一个示例，tasklet_init 函数进行动态初始化如下所示：
```c
#include <linux/interrupt.h>

// 定义 tasklet 处理函数
void my_tasklet_handler(unsigned long data)
{
    // Tasklet 处理逻辑
    // ...
}

// 声明 tasklet 结构体
static struct tasklet_struct my_tasklet;

// 初始化 tasklet
tasklet_init(&my_tasklet, my_tasklet_handler, 0);
// 驱动程序的其他代码

```

在示例中，我们首先定义了 my_tasklet_handler 作为 tasklet 的处理函数。然后，声明了一个名为 my_tasklet 的 tasklet 结构体。接下来，通过调用 tasklet_init 函数，进行动态初始化。

通过使用 tasklet_init 函数，我们可以在运行时动态创建和初始化 tasklet。这样，我们可以根据需要灵活地管理和控制 tasklet 的生命周期。在不再需要 tasklet 时，可以使用 tasklet_kill 函数进行销毁，以释放相关资源。

  

### 关闭函数

在 Linux 内核中，可以使用 tasklet_disabled 函数来关闭一个已经初始化的 tasklet。该函数的原型如下：
```c
void tasklet_disable(struct tasklet_struct *t);
```
其中，t 是指向 tasklet 结构体的指针。

需要注意的是，关闭 tasklet 并不会销毁 tasklet 结构体，因此可以随时通过调用tasklet_enable 函数重新启用 tasklet，或者调用 tasklet_kill 函数来销毁tasklet。

### 使能函数

在 Linux 内核中，可以使用 tasklet_enable 函数来使能（启用）一个已经初始化的 tasklet。该函数的原型如下：
```c
void tasklet_enable(struct tasklet_struct *t);
```

其中，t 是指向 tasklet 结构体。

需要注意的是，使能 tasklet 并不会自动触发 tasklet 的执行，而是通过调用 tasklet_schedule函数来触发。同时，可以使用 tasklet_disable 函数来临时暂停或停止 tasklet 的执行。如果需要永久停止 tasklet 的执行并释放相关资源，则应调用 tasklet_kill 函数来销毁 taskl

### 调度函数

在 Linux 内核中，可以使用 tasklet_schedule 函数来调度（触发）一个已经初始化的 tasklet 执行。该函数的原型如下：
```c
void tasklet_schedule(struct tasklet_struct *t);
```
其中，t 是指向 tasklet 结构体的指针。

需要注意的是，调度 tasklet 只是将 tasklet 标记为需要执行，并不会立即执行 tasklet 的处理函数。实际的执行时间取决于内核的调度和处理机制。

### 销毁函数

在 Linux 内核中，可以使用 tasklet_kill 函数来销毁一个已经初始化的 tasklet，释放相关资源。该函数的原型如下：
```c
void tasklet_kill(struct tasklet_struct *t);
```

调用 tasklet_kill 函数会释放 tasklet 所占用的资源，并将 tasklet 标记为无效。因此，销毁后的 tasklet 不能再被使用。

需要注意的是，在销毁 tasklet 之前，应该确保该 tasklet 已经被停止（通过调用tasklet_disable 函数）。否则，销毁一个正在执行的 tasklet 可能导致内核崩溃或其他错误。

一旦销毁了 tasklet，如果需要再次使用 tasklet，需要重新进行初始化（通过调用 tasklet_init函数）。


## tasklet 内部机制


tasklet属于TASKLET_SOFTIRQ软件中断，入口函数为 tasklet_action，这在内核 kernel\softirq.c 中设置：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/90cc784be0739758524dcb97d814fca9.png)


当驱动程序调用 tasklet_schedule 时，会设置 tasklet 的 state 为TASKLET_STATE_SCHED，并把它放入某个链表：

include/linux/interrupt.h

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5ee320bcb1cee0beb2019174b3ce87c1.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0e579a2b0148ee41afbfc902622e5d98.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3df7b431236779bb7fa83c47767973c0.png)


当发生硬件中断时，内核处理完硬件中断后，会处理软件中断。对于TASKLET_SOFTIRQ 软件中断，会调用 tasklet_action 函数。

执行过程还是挺简单的：从队列中找到 tasklet，进行状态判断后执行 func函数，从队列中删除 tasklet。

从这里可以看出：

① tasklet_schedule 调度 tasklet 时，其中的函数并不会立刻执行，而只是把tasklet 放入队列； ② 调用一次 tasklet_schedule，只会导致 tasklnet 的函数被执行一次； ③ 如果 tasklet 的函数尚未执行，多次调用 tasklet_schedule 也是无效的，只会放入队列一次。

我们softirq 初始化的时候 指定服务程序：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/83704115ac3ea7c493474b9e7587496a.png)


tasklet_action 函数解析如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d52bc572cf816cc038b8cc09a5e9cbc2.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/26c6c3af5dbaead3119cd662550b1f1d.png)
