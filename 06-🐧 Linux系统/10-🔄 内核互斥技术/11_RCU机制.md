
在前面学习的同步机制中，读写锁已经是针对读多写少场景的优化了。但是，有没有可能做到**读操作完全无锁**呢？答案就是**RCU**(Read-Copy-Update)机制——Linux内核中最先进的同步技术之一。

## 1 什么是RCU

**RCU**的全称是**Read-Copy-Update**，它是Linux内核中一种重要的同步机制，专门针对读多写少的场景进行了`极致优化`。

> [!note]+ 通俗解释
> 
> RCU就像是图书馆的读书规则：
> 
> - 读者可以随时进入，不需要任何许可（读操作无锁）
> - 当需要更换书籍时，管理员会准备新书放在一边
> - 等所有正在读旧书的人都离开后，再把旧书收走
> - 这样读者永远不会被打断，也能读到一致的内容

传统的锁机制存在一些问题：
1. **原子操作开销**：自旋锁、读写锁都使用原子操作指令，多CPU争用会导致高速缓存一致性问题
2. **读写互斥**：读写信号量不允许读者和写者同时存在
3. **性能瓶颈**：在读操作占主导的场景下，传统锁机制成为性能瓶颈

> [!success]+ RCU的目标
> 
> RCU要实现的目标是：
> 
> - **读者线程没有同步开销**，甚至可以忽略不计
> - **不需要额外的锁**、原子操作指令和内存屏障指令
> - **写者线程承担同步任务**，等待所有读者线程完成后才销毁旧数据

## 2 RCU的核心概念

RCU机制的`核心原理`可以概括为：**记录所有指向共享数据的指针的使用者，当要修改共享数据时，首先创建一个副本，在副本中修改。所有读者线程离开读者临界区之后，指针指向修改后的副本，并且删除旧数据。**

**关键概念**
- **静止状态(Quiescent State)**：当一个CPU没有在RCU读者临界区内执行时，该CPU就处于静止状态。
- **宽限期 (Grace Period)**：从RCU写者开始修改数据到所有CPU都经历过至少一次静止状态的时间段。在宽限期结束后，可以安全地释放旧数据。

> [!info]+ 概念关系
> 
> - 只有当所有CPU都经历过静止状态，宽限期才算结束
> - 宽限期结束意味着没有读者在使用旧数据，可以安全回收
> - 新的读者只会看到新数据，不会访问即将回收的旧数据

## 3 RCU提供API

RCU为读者和写者提供了不同的接口：

**1、读者接口**

| 函数                     | 参数               | 返回值    | 描述             |
| ---------------------- | ---------------- | ------ | -------------- |
| `rcu_read_lock()`      | 无                | `void` | 进入RCU读者临界区     |
| `rcu_read_unlock()`    | 无                | `void` | 离开RCU读者临界区     |
| `rcu_dereference(ptr)` | `ptr`: 受RCU保护的指针 | 指针值    | 安全地解引用RCU保护的指针 |

**2、写者接口**

| 函数                             | 参数                                 | 返回值    | 描述              |
| ------------------------------ | ---------------------------------- | ------ | --------------- |
| `rcu_assign_pointer(ptr, val)` | `ptr`: 指针变量<br>`val`: 新值           | `void` | 发布更新后的数据指针      |
| `synchronize_rcu()`            | 无                                  | `void` | 同步等待所有现存的读访问完成  |
| `call_rcu(head, func)`         | `head`: rcu_head指针<br>`func`: 回调函数 | `void` | 注册回调函数，宽限期结束后执行 |
**3、内存管理接口**

| 函数                       | 参数                                    | 返回值    | 描述            |
| ------------------------ | ------------------------------------- | ------ | ------------- |
| `kfree_rcu(ptr, field)`  | `ptr`: 要释放的指针<br>`field`: rcu_head字段名 | `void` | 延迟释放内存        |
| `kvfree_rcu(ptr, field)` | `ptr`: 要释放的指针<br>`field`: rcu_head字段名 | `void` | 延迟释放vmalloc内存 |

**4、读者侧使用**
```c
rcu_read_lock();                    // 进入RCU读者临界区
ptr = rcu_dereference(global_ptr);  // 安全获取指针
if (ptr) {
    // 使用ptr指向的数据
    use_data(ptr);
}
rcu_read_unlock();                  // 离开RCU读者临界区
```

**5、写者侧使用**
```c
// 方法1：同步等待
new_ptr = create_new_data();
rcu_assign_pointer(global_ptr, new_ptr);
synchronize_rcu();  // 等待所有读者完成
free_old_data(old_ptr);

// 方法2：异步回调
new_ptr = create_new_data();
rcu_assign_pointer(global_ptr, new_ptr);
call_rcu(&old_ptr->rcu_head, free_callback);  // 注册回调
```

## 4 RCU实战案例

让我们通过一个完整的例子来理解RCU的使用：

```c fold
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/spinlock.h>
#include <linux/rcupdate.h>
#include <linux/kthread.h>
#include <linux/delay.h>

struct foo {
    int a;
    struct rcu_head rcu;  // RCU回调所需的结构
};

static struct foo *g_ptr;

// 读者线程
static void myrcu_reader_thread(void *data)
{
    struct foo *p = NULL;

    while (1) {
        msleep(200);
        
        rcu_read_lock();                    // 进入RCU读者临界区
        p = rcu_dereference(g_ptr);         // 安全获取指针
        if (p)
            printk("%s: read a=%d\n", __func__, p->a);
        rcu_read_unlock();                  // 离开RCU读者临界区
    }
}

// RCU回调函数：用于释放旧数据
static void myrcu_del(struct rcu_head *rh)
{
    struct foo *p = container_of(rh, struct foo, rcu);
    printk("%s: freeing a=%d\n", __func__, p->a);
    kfree(p);
}

// 写者线程
static void myrcu_writer_thread(void *p)
{
    struct foo *new_ptr;
    struct foo *old_ptr;
    int value = (unsigned long)p;

    while (1) {
        msleep(400);
        
        // 创建新数据
        new_ptr = kmalloc(sizeof(struct foo), GFP_KERNEL);
        old_ptr = g_ptr;
        
        printk("%s: updating to new value %d\n", __func__, value);
        
        // 从旧数据复制并更新
        *new_ptr = *old_ptr;
        new_ptr->a = value;
        
        // 发布新指针
        rcu_assign_pointer(g_ptr, new_ptr);
        
        // 注册回调，宽限期结束后自动释放旧数据
        call_rcu(&old_ptr->rcu, myrcu_del);
        
        value++;
    }
}

static int __init my_test_init(void)
{
    struct task_struct *reader_thread;
    struct task_struct *writer_thread;
    int value = 5;

    printk("RCU test module init\n");
    
    // 初始化全局数据
    g_ptr = kzalloc(sizeof(struct foo), GFP_KERNEL);

    // 创建读者和写者线程
    reader_thread = kthread_run(myrcu_reader_thread, NULL, "rcu_reader");
    writer_thread = kthread_run(myrcu_writer_thread, 
                               (void *)(unsigned long)value, "rcu_writer");

    return 0;
}

static void __exit my_test_exit(void)
{
    printk("RCU test module exit\n");
    if (g_ptr)
        kfree(g_ptr);
}

MODULE_LICENSE("GPL");
module_init(my_test_init);
module_exit(my_test_exit);
```

> [!note]+ 案例分析
> 
> 这个例子展示了RCU的典型使用模式：
> 
> 1. **读者**：使用`rcu_read_lock/unlock`保护临界区，用`rcu_dereference`安全访问数据
> 2. **写者**：创建新数据，使用`rcu_assign_pointer`发布，用`call_rcu`延迟释放旧数据
> 3. **数据结构**：包含`rcu_head`字段用于RCU回调

## 5 RCU在链表中的应用

RCU的一个重要应用场景是链表操作，它可以显著提高遍历性能：

### 5.1 RCU链表的优势

1. **读取性能**：遍历链表时只需要`rcu_read_lock()`，允许多个线程同时读取
2. **写入并发**：允许一个线程修改链表的同时，其他线程继续遍历
3. **内存安全**：保证在读者访问期间，节点不会被提前释放

### 5.2 链表操作示例

```c
// 安全遍历RCU保护的链表
struct list_head *pos;
struct my_struct *entry;

rcu_read_lock();
list_for_each_entry_rcu(entry, &my_list, list) {
    // 处理链表节点
    process_entry(entry);
}
rcu_read_unlock();

// 安全删除链表节点
void delete_entry(struct my_struct *entry)
{
    list_del_rcu(&entry->list);     // 从链表中移除
    call_rcu(&entry->rcu, free_entry);  // 延迟释放
}
```

## 6 RCU相比读写锁的优势

### 6.1 性能对比

|特性|读写锁|RCU|
|---|---|---|
|读操作开销|需要原子操作|几乎无开销|
|读写并发|不允许|允许|
|扩展性|随CPU数量下降|优秀|
|写操作开销|较低|较高|
|适用场景|读写比例适中|读多写少|

### 6.2 技术优势

1. **读操作无锁化**：读者不需要任何原子操作
2. **优秀的扩展性**：读性能不会随CPU数量下降
3. **读写真并发**：读者和写者可以真正并发执行
4. **低延迟**：读操作的延迟极低且可预测

> [!tip]+ 适用判断
> 
> **何时使用RCU？**
> 
> - 读操作远多于写操作（如10:1以上）
> - 对读操作的性能要求极高
> - 数据结构相对稳定（如配置信息、路由表）
> - 可以接受写操作的额外开销

## 7 RCU的实现原理简介

RCU的实现基于以下观察：
1. **读者识别**：通过`rcu_read_lock/unlock`标识读者的存在
2. **写者等待**：写者必须等待所有现有读者完成
3. **新读者引导**：新的读者会看到新数据，不会访问旧数据
4. **回收时机**：当确保没有读者访问旧数据时，才回收内存


内核使用多种机制来检测宽限期的结束：
1. **调度点检测**：进程主动调度即表示离开RCU临界区
2. **中断检测**：中断的到来表明CPU处于非RCU临界区
3. **系统调用**：用户态到内核态的切换
4. **CPU idle**：CPU进入空闲状态

## 8 使用注意事项

### 8.1 RCU的限制

1. **写操作开销高**：需要等待宽限期，开销比普通锁大
2. **内存使用增加**：需要同时保持新旧数据
3. **编程复杂性**：需要理解RCU的语义和使用规则
4. **调试困难**：RCU相关的问题较难调试

### 8.2 最佳实践

> [!warning]+ 使用规则
> 
> 1. **RCU临界区要短**：`rcu_read_lock/unlock`之间的代码要尽量短
> 2. **不能睡眠**：RCU读者临界区内不能睡眠或阻塞
> 3. **正确使用接口**：必须使用`rcu_dereference`和`rcu_assign_pointer`
> 4. **回调函数要简单**：RCU回调函数应该尽量简单，避免长时间执行

### 8.3 错误使用示例

```c
// 错误：在RCU临界区内睡眠
rcu_read_lock();
ptr = rcu_dereference(global_ptr);
msleep(100);  // 错误！不能在RCU临界区内睡眠
rcu_read_unlock();

// 错误：没有使用rcu_dereference
rcu_read_lock();
ptr = global_ptr;  // 错误！应该使用rcu_dereference
use_data(ptr);
rcu_read_unlock();
```

## 9 总结

RCU是Linux内核中最先进的同步机制之一：

### 9.1 核心优势

1. **读操作高性能**：读者几乎无同步开销
2. **良好扩展性**：性能随CPU数量线性扩展
3. **读写真并发**：读者和写者可以真正并发
4. **内存安全**：保证读者访问数据的一致性

### 9.2 适用场景

1. **读多写少**：读写比例悬殊的场景
2. **性能关键**：对读操作性能要求极高
3. **数据相对稳定**：配置信息、路由表等
4. **链表遍历**：频繁遍历的链表结构

### 9.3 使用要点

1. **理解语义**：深入理解RCU的工作原理
2. **正确使用接口**：严格按照RCU的编程规范
3. **适当的场景**：在合适的场景下使用
4. **充分测试**：RCU代码需要在多核环境下充分测试

> [!success]+ 学习价值
> 
> 掌握RCU机制有助于：
> 
> - 理解现代操作系统的先进同步技术
> - 在性能关键场景下选择最优方案
> - 设计高性能的并发数据结构
> - 深入理解内存管理和并发编程

RCU代表了同步机制设计的最高水平，它在特定场景下提供了接近理想的性能表现。
