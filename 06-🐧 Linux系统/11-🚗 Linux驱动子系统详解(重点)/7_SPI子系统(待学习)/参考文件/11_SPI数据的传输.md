
- spi_transfer
    
- spi_message


## 数据结构

`spi` 数据传输主要使用了 `spi_message` 和 `spi_transfer` 结构， 多个 `spi_transfer` 够成一个 `spi_message`.

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/754c802a99aac6b2d63baf12dbde7528.png)
上图是一个SPI子系统数据结构关系图，展示了SPI传输的层次化组织架构：

**左侧 - spi_transfer()**：

- 最基本的传输单元，包含tx_buf（发送缓冲区）等字段
- 通过transfers_list连接到传输链表中
- 多个transfer可以组合成一个完整的SPI操作

**中间 - spi_transfer_list**：

- 连接多个spi_transfer结构的链表
- 将相关的传输操作组织在一起

**右侧 - spi_message()**：

- 更高层的消息结构，包含transfers、queue等字段
- 一个message可以包含多个transfer
- 代表一次完整的SPI通信事务

**最右侧 - spi master端的链表**：

- SPI主控制器维护的消息队列
- 将多个message排队等待处理
- 实现SPI传输的调度和管理

整个架构体现了"传输->消息->队列"的三级组织结构，支持复杂的SPI通信场景，如多段传输、批量操作等。

在进行 `spi` 数据传输的时候，如果有同一时段有多个 `spi msg` 要处理，则会将要处理的 `msg` 连成一个链表，等待依次处理，该链表头一般都是包含了 `spi_master{}`的实际控制端。

### spi_transfer 结构体

文件目录：`kernel\include\linux\spi\spi.h`

```C++
struct spi_transfer {
        /* it's ok if tx_buf == rx_buf (right?)
         * for MicroWire, one buffer must be null
         * buffers must work with dma_*map_single() calls, unless
         *   spi_message.is_dma_mapped reports a pre-existing mapping
         */
        const void        *tx_buf;
        void                *rx_buf;
        unsigned        len;

        dma_addr_t        tx_dma;
        dma_addr_t        rx_dma;
        struct sg_table tx_sg;
        struct sg_table rx_sg;

        unsigned        cs_change:1;
        unsigned        tx_nbits:3;
        unsigned        rx_nbits:3;
#define        SPI_NBITS_SINGLE        0x01 /* 1bit transfer */
#define        SPI_NBITS_DUAL                0x02 /* 2bits transfer */
#define        SPI_NBITS_QUAD                0x04 /* 4bits transfer */
        u8                bits_per_word;
        u16                delay_usecs;
        u32                speed_hz;

        struct list_head transfer_list;
};
```

结构内各元素的含义如下。

- tx_buf：该缓冲区包含要写入的数据。在只读事务中它应该为NULL或保留其原样。如果需要通过直接内存访问（DMA）执行SPI事务，它应该是与DMA无关的。
    
- rx_buf：被读取数据的缓冲区（具有与tx_buf相同的属性），在只写事务中为NULL。
    
- tx_dma：spi_message.is_dma_mapped被设置为1时，这是tx_buf的DMA地址。
    
- rx_dma：这与tx_dma相同，但是用于rx_buf。
    
- len：表示rx和tx缓冲区的大小（以字节为单位），这意味着如果两者都使用，它们必须具有相同的大小。
    
- speed_hz：这会覆盖在spi_device.max_speed_hz中指定的默认速度，但仅限于当前传输。如果为0，则使用默认值（struct spi_device结构中提供）。
    
- bits_per_word：数据传输涉及一个或多个字。字是数据单位，其位长度可以根据需要而改变。这里，bits_per_word表示这次SPI传输中字的位长度。它覆盖spi_device.bits_per_word中提供的默认值。如果为0，则使用默认值（来自spi_device）。
    
- cs_change：决定这次传输完成后chip_select线的状态。
    
- delay_usecs：表示这次传输之后的延迟（以微秒为单位），在（可选）更改chip_select状态，并开始下一次传输或完成这次spi_message之前。
    

### spi_message

文件目录：`kernel\include\linux\spi\spi.h`

另外，struct spi_message以原子方式包装一个或多个SPI传输。所使用的SPI总线将被驱动程序占用，直到构成SPI消息的所有传输完成为止。SPI消息结构也在include/linux/spi/spi.h中定义：

```C++
struct spi_message {
        struct list_head        transfers;

        struct spi_device        *spi;

        unsigned                is_dma_mapped:1;

        /* REVISIT:  we might want a flag affecting the behavior of the
         * last transfer ... allowing things like "read 16 bit length L"
         * immediately followed by "read L bytes".  Basically imposing
         * a specific message scheduling algorithm.
         *
         * Some controller drivers (message-at-a-time queue processing)
         * could provide that as their default scheduling algorithm.  But
         * others (with multi-message pipelines) could need a flag to
         * tell them about such special cases.
         */

        /* completion is reported through a callback */
        void                        (*complete)(void *context);
        void                        *context;
        unsigned                frame_length;
        unsigned                actual_length;
        int                        status;

        /* for optional use by whatever driver currently owns the
         * spi_message ...  between calls to spi_async and then later
         * complete(), that's the spi_master controller driver.
         */
        struct list_head        queue;
        void                        *state;
};
```

- transfers：构成消息的传输列表。稍后将会看到如何将传输添加到此列表中。
    
- is_dma_mapped：通知控制器是否使用DMA执行传输。然后，代码负责为每个传输缓冲区提供DMA和CPU虚拟地址。
    
- complete：这是事务完成时调用的回调，context是传递给回调的参数。
    
- frame_length：这将自动设置为消息中的总字节数。
    
- actual_length：这是在所有成功段中传输的字节数。
    
- status：报告传输状态。成功为0，否则为-errno。
    

## 数据发送程序分析

下图是数据发送程序的导图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9dcbdf2877e1756590e525a319ea47d9.png)


spi_sync函数用于实现同步传输，其数据传输也是调用的__spi_queued_transfer函数，只是第三个参数为false。spi_sync函数定义在drivers/spi/spi.c文件：

```C++
/**
 * spi_sync - blocking/synchronous SPI data transfers
 * @spi: device with which data will be exchanged
 * @message: describes the data transfers
 * Context: can sleep
 *
 * This call may only be used from a context that may sleep.  The sleep
 * is non-interruptible, and has no timeout.  Low-overhead controller
 * drivers may DMA directly into and out of the message buffers.
 *
 * Note that the SPI device's chip select is active during the message,
 * and then is normally disabled between messages.  Drivers for some
 * frequently-used devices may want to minimize costs of selecting a chip,
 * by leaving it selected in anticipation that the next message will go
 * to the same chip.  (That may increase power usage.)
 *
 * Also, the caller is guaranteeing that the memory associated with the
 * message will not be freed before this call returns.
 *
 * Return: zero on success, else a negative error code.
 */
int spi_sync(struct spi_device *spi, struct spi_message *message)
{
        int ret;

        mutex_lock(&spi->controller->bus_lock_mutex);
        ret = __spi_sync(spi, message);
        mutex_unlock(&spi->controller->bus_lock_mutex);

        return ret;
}
```

### 同步接口 `__spi_sync`

这里又调用了__spi_sync函数：

```C++
static int __spi_sync(struct spi_device *spi, struct spi_message *message)
{
        DECLARE_COMPLETION_ONSTACK(done);   // 声明dowe变量
        int status;
        struct spi_controller *ctlr = spi->controller;
        unsigned long flags;

        status = __spi_validate(spi, message);
        if (status != 0)
                return status;

        message->complete = spi_complete;   // 传输完成回调函数
        message->context = &done;           // 提供给complete的可选参数
        message->spi = spi;                 // 设置目标SPI从设备

        SPI_STATISTICS_INCREMENT_FIELD(&ctlr->statistics, spi_sync);
        SPI_STATISTICS_INCREMENT_FIELD(&spi->statistics, spi_sync);

        /* If we're not using the legacy transfer method then we will
         * try to transfer in the calling context so special case.
         * This code would be less tricky if we could remove the
         * support for driver implemented message queues.
         */
        if (ctlr->transfer == spi_queued_transfer) {   // 之前已经初始化，这一步会进入
                spin_lock_irqsave(&ctlr->bus_lock_spinlock, flags);

                trace_spi_message_submit(message);

                status = __spi_queued_transfer(spi, message, false);  // 第三个参数设置为了false

                spin_unlock_irqrestore(&ctlr->bus_lock_spinlock, flags);
        } else {
                status = spi_async_locked(spi, message);
        }

        if (status == 0) {                    //SPI控制器没有数据在传输，因此可以进行数据传输
                /* Push out the messages in the calling context if we
                 * can.
                 */
                if (ctlr->transfer == spi_queued_transfer) {
                        SPI_STATISTICS_INCREMENT_FIELD(&ctlr->statistics,
                                                       spi_sync_immediate);
                        SPI_STATISTICS_INCREMENT_FIELD(&spi->statistics,
                                                       spi_sync_immediate);
                        __spi_pump_messages(ctlr, false);    // 进行SPI数据传输
                }

                wait_for_completion(&done);    // 等待传输完成
                status = message->status;
        }
        message->context = NULL;
        return status;
}
```

在该函数中调用了__spi_queued_transfer：

- 如果当前正在SPI数据传输中，那么将返回-ESHUTDOWN；
    
- 否者将spi_message挂到spi_controller->queue队列上；
    
- 返回0。
    

然后与异步传输不同的是：

- 异步传输是将__spi_pump_messages作为工作挂到内核工作线程，由内核工作线程来完成数据的传输；
    
- 而同步传输则是在内部调用了__spi_pump_messages方法，针对不同的情景情况而不同（只有同步传输、既有同步传输，也有异步传输）：
    
    - __spi_pump_messages函数内部可能直接调用ctrl->transfer_one_message通过SPI控制器进行数据传输；
        
    - 也有可能将__spi_pump_messages作为工作挂到内核工作线程，由内核工作线程完成数据的传输；
        
- 此外__spi_pump_messages函数传入的第二个参数不同，异步方式传入true，同步方式传入false。
    

### `__spi_pump_messages`

__spi_pump_messages用于处理SPI消息队列的，此函数检查队列中是否有SPI消息需要处理，如果需要，调用驱动程序初始化硬件并传送每个消息。

注意，它既会被kthread本身调用，也会被spi_sync函数调用；

```C++
/**
 * __spi_pump_messages - function which processes spi message queue
 * @ctlr: controller to process queue for
 * @in_kthread: true if we are in the context of the message pump thread
 *
 * This function checks if there is any spi message in the queue that
 * needs processing and if so call out to the driver to initialize hardware
 * and transfer each message.
 *
 * Note that it is called both from the kthread itself and also from
 * inside spi_sync(); the queue extraction handling at the top of the
 * function should deal with this safely.
 */
static void __spi_pump_messages(struct spi_controller *ctlr, bool in_kthread)
{
        unsigned long flags;
        bool was_busy = false;
        int ret;

        /* Lock queue */
        spin_lock_irqsave(&ctlr->queue_lock, flags);   //  互斥访问  自旋锁 + 关中断

        /* Make sure we are not already running a message */
        if (ctlr->cur_msg) {                 // 有msg正在处理，这里直接就返回了，这里直接返回会导致消息丢失么，不会的因为消息已经放入消息队列了，当正在处理的消息被处理完，                                                会调用spi_finalize_current_message函数，该函数又会重复将__spi_pump_messages作为任务挂到内核工作线程的
                spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                return;
        }

        /* If another context is idling the device then defer */
        if (ctlr->idling) {                 // 空闲状态会进入  这个值基本上始终为false
                kthread_queue_work(&ctlr->kworker, &ctlr->pump_messages);  // 将work加入到worker
                spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                return;
        }

        /* Check if the queue is idle */
        if (list_empty(&ctlr->queue) || !ctlr->running) {  // 在__spi_queue_transfer函数中会将需要传输的消息挂到消息队列；所以这个队列刚开始不为空 这个分支页不会进入；                                                               但是当消息队列消息传输完后，则会进入
                if (!ctlr->busy) {                         // SPI控制器不繁忙，进入 直接返回
                        spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                        return;
                }

                /* Only do teardown in the thread */
                if (!in_kthread) {                           // 异步参数为true 同步参数为false，同步进入
                        kthread_queue_work(&ctlr->kworker,   // 将work加入到worker
                                           &ctlr->pump_messages);
                        spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                        return;
                }

                ctlr->busy = false;    // 设置SPI控制器状态不繁忙
                ctlr->idling = true;   // 设置内核工作线程状态为空闲
                spin_unlock_irqrestore(&ctlr->queue_lock, flags);

                kfree(ctlr->dummy_rx);
                ctlr->dummy_rx = NULL;
                kfree(ctlr->dummy_tx);
                ctlr->dummy_tx = NULL;
                if (ctlr->unprepare_transfer_hardware &&
                    ctlr->unprepare_transfer_hardware(ctlr))
                        dev_err(&ctlr->dev,
                                "failed to unprepare transfer hardware\n");
                if (ctlr->auto_runtime_pm) {
                        pm_runtime_mark_last_busy(ctlr->dev.parent);
                        pm_runtime_put_autosuspend(ctlr->dev.parent);
                }
                trace_spi_controller_idle(ctlr);

                spin_lock_irqsave(&ctlr->queue_lock, flags);
                ctlr->idling = false;      // 设置内核工作线程状态为空闲
                spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                return;
        }

        /* Extract head of queue */
        ctlr->cur_msg =   // 从消息队列获取第一个spi_message
                list_first_entry(&ctlr->queue, struct spi_message, queue);

        list_del_init(&ctlr->cur_msg->queue);  // 将第一个spi_message从ctrl->queue队列删除
        if (ctlr->busy)           // 当前SPI控制器繁忙
                was_busy = true;
        else
                ctlr->busy = true;  // 将当前状态设备为繁忙
        spin_unlock_irqrestore(&ctlr->queue_lock, flags);

        mutex_lock(&ctlr->io_mutex);

        if (!was_busy && ctlr->auto_runtime_pm) {  // 电源管理相关 忽略
                ret = pm_runtime_get_sync(ctlr->dev.parent);
                if (ret < 0) {
                        pm_runtime_put_noidle(ctlr->dev.parent);
                        dev_err(&ctlr->dev, "Failed to power device: %d\n",
                                ret);
                        mutex_unlock(&ctlr->io_mutex);
                        return;
                }
        }

        if (!was_busy)  // 过去不busy则进入
                trace_spi_controller_busy(ctlr);

        if (!was_busy && ctlr->prepare_transfer_hardware) {
                ret = ctlr->prepare_transfer_hardware(ctlr);
                if (ret) {
                        dev_err(&ctlr->dev,
                                "failed to prepare transfer hardware: %d\n",
                                ret);

                        if (ctlr->auto_runtime_pm)
                                pm_runtime_put(ctlr->dev.parent);

                        ctlr->cur_msg->status = ret;
                        spi_finalize_current_message(ctlr);

                        mutex_unlock(&ctlr->io_mutex);
                        return;
                }
        }

        trace_spi_message_start(ctlr->cur_msg);

        if (ctlr->prepare_message) {
                ret = ctlr->prepare_message(ctlr, ctlr->cur_msg);
                if (ret) {
                        dev_err(&ctlr->dev, "failed to prepare message: %d\n",
                                ret);
                        ctlr->cur_msg->status = ret;
                        spi_finalize_current_message(ctlr);
                        goto out;
                }
                ctlr->cur_msg_prepared = true;
        }

        ret = spi_map_msg(ctlr, ctlr->cur_msg);
        if (ret) {
                ctlr->cur_msg->status = ret;
                spi_finalize_current_message(ctlr);
                goto out;
        }

        ret = ctlr->transfer_one_message(ctlr, ctlr->cur_msg);   // SPI数据传输
        if (ret) {
                dev_err(&ctlr->dev,
                        "failed to transfer one message from queue\n");
                goto out;
        }

out:
        mutex_unlock(&ctlr->io_mutex);

        /* Prod the scheduler in case transfer_one() was busy waiting */
        if (!ret)
                cond_resched();
}
```

函数内容不少，一步步看看：

- 如果当前正在有msg处理，即cur_msg不为NULL，则暂时不发起传输，直接返回；那你可能会问了，这里直接返回会导致需要传输的的spi_message消息丢失么，答案肯定是不会的，因为消息在__spi_queued_transfer方法执行的时候，已经放入消息队列了，当cur_msg被处理完，会调用spi_finalize_current_message函数，该函数又会重复将__spi_pump_messages作为任务挂到内核工作线程的；从而循环调用了当前函数；
    
- 如果当前没有msg处理，并且内核工作线程是空闲的，即没有工作需要执行，所以需要将_spi_pump_messages作为任务挂到内核工作线程的；并直接返回；
    
- 走到这一步说明，当前SPI控制器没有msg需要处理，并且当前内核工作线程有工作在执行：
    
    - 从 ctlr->queue 取出第一个 spi_message 赋值给 ctlr->cur_msg，标记当前状态 ctlr->busy为busy，同时记录上一个spi_message对应的状态到was_busy变量；第一次传输这个值为false，后面都是true；
        
    - 如果 was_busy 为 true 的话，调用 spi_controller 的 prepare_transfer_hardware；
        
    - 如果有 prepare_message，则调用 ctlr->prepare_message，准备传输之前的一些准备工作，SoC 下面挂接的钩子实现，这一步如果执行失败，则返回错误码到 ctlr->cur_msg->status，否则 ctlr->cur_msg_prepared = true; 设置为准备成功；
        
    - 调用到 spi_map_msg 函数，主要处理一些dma相关的内容；
        
    - 调用到 ctlr->transfer_one_message，transfer_one_message 在初始化的时候如果没有指定的话（transfer_one_message是SPI控制器驱动要实现的，主要功能是处理spi_message中的每个spi_transfer），其默认挂接了SPI核心层的 spi_transfer_one_message；
        

这段代码实际上比较饶人，就我个人认为，它其实就是想做这么一件事：

- 从消息队列去获取spi_message，然后调用SPI控制器驱动提供的transfer_one函数进行消息的传输，传输完成后就回调spi_finalize_current_message，再次将__spi_pump_messages放到内核工作线程执行；
    
- 当然了当消息队列被输出完的话，内核工作线程就没工作可执行了，所以__spi_pump_messages函数内部又多了一些状态的判断，判断何时需要将__spi_pump_messages放到内核工作线程执行；
    

**spi_transfer_one_message**

```C++
/*
 * spi_transfer_one_message - Default implementation of transfer_one_message()
 *
 * This is a standard implementation of transfer_one_message() for
 * drivers which implement a transfer_one() operation.  It provides
 * standard handling of delays and chip select management.
 */
static int spi_transfer_one_message(struct spi_controller *ctlr,
                                    struct spi_message *msg)
{
        struct spi_transfer *xfer;
        bool keep_cs = false;
        int ret = 0;
        struct spi_statistics *statm = &ctlr->statistics;
        struct spi_statistics *stats = &msg->spi->statistics;

        spi_set_cs(msg->spi, true);   // 片选

        SPI_STATISTICS_INCREMENT_FIELD(statm, messages);
        SPI_STATISTICS_INCREMENT_FIELD(stats, messages);

        list_for_each_entry(xfer, &msg->transfers, transfer_list) {
                trace_spi_transfer_start(msg, xfer);

                spi_statistics_add_transfer_stats(statm, xfer, ctlr);
                spi_statistics_add_transfer_stats(stats, xfer, ctlr);

                if (xfer->tx_buf || xfer->rx_buf) {     // 接收缓冲区或发送缓冲区不为空
                        reinit_completion(&ctlr->xfer_completion);

                        ret = ctlr->transfer_one(ctlr, msg->spi, xfer);   // 真正的SPI数据传输 这个一般由SoC厂家提供  0传输完成
                        if (ret < 0) {    // 传输错误
                                SPI_STATISTICS_INCREMENT_FIELD(statm,
                                                               errors);
                                SPI_STATISTICS_INCREMENT_FIELD(stats,
                                                               errors);
                                dev_err(&msg->spi->dev,
                                        "SPI transfer failed: %d\n", ret);
                                goto out;
                        }

                        if (ret > 0) {   // 传输进行中
                                ret = spi_transfer_wait(ctlr, msg, xfer);   // 等待传输完成
                                if (ret < 0)
                                        msg->status = ret;
                        }
                } else {
                        if (xfer->len)
                                dev_err(&msg->spi->dev,
                                        "Bufferless transfer has length %u\n",
                                        xfer->len);
                }

                trace_spi_transfer_stop(msg, xfer);

                if (msg->status != -EINPROGRESS)
                        goto out;

                if (xfer->delay_usecs) {    // 延时
                        u16 us = xfer->delay_usecs;

                        if (us <= 10)
                                udelay(us);
                        else
                                usleep_range(us, us + DIV_ROUND_UP(us, 10));
                }

                if (xfer->cs_change) {      // 变动片选信号
                        if (list_is_last(&xfer->transfer_list,
                                         &msg->transfers)) {
                                keep_cs = true;
                        } else {
                                spi_set_cs(msg->spi, false);  // 取消片选
                                udelay(10);
                                spi_set_cs(msg->spi, true);
                        }
                }

                msg->actual_length += xfer->len;  // 实际传输长度
        }

out:
        if (ret != 0 || !keep_cs)
                spi_set_cs(msg->spi, false);    // 取消片选

        if (msg->status == -EINPROGRESS)
                msg->status = ret;

        if (msg->status && ctlr->handle_err)
                ctlr->handle_err(ctlr, msg);

        spi_res_release(ctlr, msg);

        spi_finalize_current_message(ctlr);   // 告知传输完成

        return ret;
}
```

这里我们就要发起传输了：

- 首先设置了片选cs，具体调用到了SPI控制器驱动提供的片选函数set_cs；
    
- 从spi_message 中的transfer_list逐个取出transfer，并调用SPI控制器驱动提供的transfer_one函数；
    
- 根据是否配置了需要的延时delay_usecs，来确定是否在下一次传输之前进行必要的udelay；
    
- 根据是否配置了cs_change，来判断是否需要变动片选信号cs；
    
- 累加实际传输的数据 actual_length；
    
- 调用spi_finalize_current_message告知完成传输；
    

## 初始化

消息提交给总线之前，必须先使用`void spi_message_init(struct spi_message * message)`初始化，这将结构中的每个元素清零并初始化`transfers`列表。对于要添加到消息中的每个传输，应该在该传输上调用`void spi_message_add_tail(struct spi_transfer * t，struct spi_message * m)`，把传输加入 `transfers`列表队列。完成后，有两种选择来启动事务。

- 同步：使用`int spi_sync(struct spi_device * spi，struct spi_message * message)`函数，它可能处于睡眠状态，不用在中断上下文中。此处不需要回调完成。该函数是对第二个函数（`spi_async()`）的封装。
    
- 异步：使用`spi_async()`函数，它也可以用于原子上下文，其原型为`int spi_async(struct spi_device * spi，struct spi_message *message)`。这里提供回调是很好的做法，因为它会在消息完成时执行。
    

以下是单个传输SPI消息事务示例：

```C
char tx_buf[] = { 
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 
        0xFF, 0x40, 0x00, 0x00, 0x00, 
        0x00, 0x95, 0xEF, 0xBA, 0xAD, 
        0xF0, 0x0D, 
};  
 
char rx_buf[10] = {0,}; 
int ret; 
struct spi_message single_msg; 
struct spi_transfer single_xfer; 
 
single_xfer.tx_buf = tx_buf; 
single_xfer.rx_buf = rx_buf; 
single_xfer.len    = sizeof(tx_buff); 
single_xfer.bits_per_word = 8; 
 
spi_message_init(&msg); 
spi_message_add_tail(&xfer, &msg); 
ret = spi_sync(spi, &msg);
```

现在来编写多传输消息事务：

```C
struct { 
    char buffer[10]; 
    char cmd[2]  
    int foo; 
} data; 
 
struct data my_data[3]; 
initialize_date(my_data, ARRAY_SIZE(my_data)); 
 
struct spi_transfer   multi_xfer[3]; 
struct spi_message    single_msg; 
int ret; 
 
multi_xfer[0].rx_buf = data[0].buffer; 
multi_xfer[0].len = 5; 
multi_xfer[0].cs_change = 1; 
/* 命令A */ 
multi_xfer[1].tx_buf = data[1].cmd; 
multi_xfer[1].len = 2; 
multi_xfer[1].cs_change = 1; 
/* 命令B */ 
multi_xfer[2].rx_buf = data[2].buffer; 
multi_xfer[2].len = 10; 
 
spi_message_init(single_msg); 
spi_message_add_tail(&multi_xfer[0], &single_msg); 
spi_message_add_tail(&multi_xfer[1], &single_msg); 
spi_message_add_tail(&multi_xfer[2], &single_msg); 
ret = spi_sync(spi, &single_msg);
```