# Linux DMA子系统学习指南 - DMA Engine API使用

## 引言：DMA Engine是什么

在上一篇中，我们了解了DMA的硬件原理。但是，不同厂商的DMA控制器实现各不相同，如果每个驱动都直接操作硬件，会造成代码重复和移植困难。因此，Linux内核提供了**DMA Engine**框架。

DMA引擎是开发DMA控制器驱动程序的通用内核框架。DMA的主要目标是在复制内存时减轻CPU的负担。使用通道将事务（I/O数据传输）委托给DMA引擎，DMA引擎通过其驱动程序/API提供一组可供其他设备（从设备）使用的通道。

> [!tip]+ 理解要点 DMA Engine就像一个"快递公司"：
> 
> - 各种DMA控制器 = 不同的快递员
> - DMA Engine API = 统一的下单接口
> - 你只需要告诉"寄什么、从哪寄、寄到哪"，不用关心具体是哪个快递员配送

## DMA Engine使用步骤概览

DMA子系统下有一个帮助测试的测试驱动(drivers/dma/dmatest.c)，从这个测试驱动入手，我们了解到内核里的其他部分怎么使用DMA engine。配置内核，选择CONFIG_DMATEST可以把这个模块选中，编译会生成dmatest.ko。可以参考这个文档来快速了解怎么使用dmatest.ko：[https://www.kernel.org/doc/html/v4.15/driver-api/dmaengine/dmatest.html](https://www.kernel.org/doc/html/v4.15/driver-api/dmaengine/dmatest.html)。

具体上来讲，内核的其他模块使用dma engine的步骤是：

> 1）申请一个DMA channel。
> 
> 2）根据设备（slave）的特性，配置DMA channel的参数。
> 
> 3）要进行DMA传输的时候，获取一个用于识别本次传输（transaction）的描述符（descriptor）。
> 
> 4）将本次传输（transaction）提交给dma engine并启动传输。
> 
> 5）等待传输（transaction）结束。
> 
> 然后，重复3~5即可。

从设备DMA用法很简单，它包含以下步骤：

- 分配DMA从通道
- 设置从设备和控制器的特定参数
- 获取事务的描述符
- 提交事务
- 发出挂起的请求并等待回调通知

这里必须引用的头文件如下：

```C++
#include <linux/dmaengine.h>
```

## 步骤1：申请DMA channel

任何consumer（slave driver）在开始DMA传输之前，都要申请一个DMA channel。

### 基本申请方法

使用dma_request_channel()请求通道。其原型如下：

```C++
struct dma_chan *dma_request_channel(const dma_cap_mask_t *mask, 
                          dma_filter_fn fn, void *fn_param);
```

consumer可以通过如下的API申请DMA channel：

```C++
struct dma_chan *dma_request_channel(struct device *dev, const char *name);
```

> 该接口会返回绑定在指定设备（dev）上名称为name的dma channel。dma engine的provider和consumer可以使用device tree、ACPI或者struct dma_slave_map类型的match table提供这种绑定关系

### 参数说明

**mask**是位图掩码，表示该通道必须满足的功能。使用它主要是为了指定驱动程序需要执行的传输类型：

```C++
enum dma_transaction_type { 
    DMA_MEMCPY,     /* 内存到内存复制*/ 
    DMA_XOR,        /* 内存到内存XOR*/ 
    DMA_PQ,         /* 内存到内存的P+Q计算 */ 
    DMA_XOR_VAL,    /* 使用XOR进行内存缓冲区奇偶校验 */ 
    DMA_PQ_VAL,     /* 使用P+Q进行内存缓冲区奇偶校验 */ 
    DMA_INTERRUPT,  /* 该设备生成虚拟传输，虚拟传输产生中断*/ 
    DMA_SG,         /* 内存对内存散聚 */ 
    DMA_PRIVATE,    /* 通道不用于全局memcpy，通常与DMA_SLAVE一起使用 */ 
    DMA_SLAVE,      /* 内存到设备的传输*/ 
    DMA_CYCLIC,     /* 设备能够处理循环传输 */ 
    DMA_INTERLEAVE, /* 内存到内存交错传输 */ 
}
```

### 使用示例

dma_cap_zero()和dma_cap_set()函数用于清除掩码，设置所需的功能。例如：

```C++
dma_cap_mask_t my_dma_cap_mask; 
struct dma_chan *chan; 

/* 清空并设置DMA能力掩码 */
dma_cap_zero(my_dma_cap_mask); 
dma_cap_set(DMA_MEMCPY, my_dma_cap_mask); /* 内存到内存复制 */ 

/* 申请DMA通道 */
chan = dma_request_channel(my_dma_cap_mask, NULL, NULL);
if (!chan) {
    pr_err("Failed to request DMA channel\n");
    return -ENODEV;
}
```

### 过滤函数

在前面dma_filter_fn定义为：

```C++
typedef bool (*dma_filter_fn)(struct dma_chan *chan,  
                void *filter_param);
```

如果filter_fn参数（这是可选项）为NULL，则dma_request_channel()将只返回满足功能掩码的第一个通道。否则，当掩码参数不足以指定所需的通道时，可以使用filter_fn例程作为系统中可用通道的过滤器。内核为系统中的每个空闲通道调用一次filter_fn例程。在看到合适的通道时，filter_fn应该返回DMA_ACK，它将把给定的通道标记为dma_request_channel()的返回值。

> [!tip]+ 理解要点 过滤函数就像"面试官"：
> 
> - 系统把所有空闲的DMA通道（候选人）逐个介绍给你
> - 你的过滤函数检查每个通道是否符合要求
> - 看到合适的就返回true（录用），该通道就归你使用了

### 释放通道

通过此接口分配的通道由调用者独占，直到调用dma_release_channel()为止：

```C++
void dma_release_channel(struct dma_chan *chan)
```

最后，申请得到的dma channel可以在不需要使用的时候通过下面的API释放掉：

```C++
void dma_release_channel(struct dma_chan *chan);
```

## 步骤2：配置DMA channel的参数

driver申请到一个为自己使用的DMA channel之后，需要根据自身的实际情况，以及DMA controller的能力，对该channel进行一些配置。

### 配置结构体

此步骤引入了新的数据结构struct dma_slave_config，它表示DMA从通道的运行时配置。这允许客户端为外设指定设置，如DMA方向、DMA地址、总线宽度、DMA突发长度（DMA burst length）等。

可配置的内容由struct dma_slave_config数据结构表示。driver将它们填充到一个struct dma_slave_config变量中后，可以调用如下API将这些信息告诉给DMA controller：

```C
int dmaengine_slave_config(struct dma_chan *chan, 
                          struct dma_slave_config *config)
```

struct dma_slave_config结构如下所示：

```C++
/* 
 * 请参考include/linux/dmaengine.h中的完整描述 
 */ 
struct dma_slave_config { 
   enum dma_transfer_direction direction; 
   phys_addr_t src_addr; 
   phys_addr_t dst_addr; 
   enum dma_slave_buswidth src_addr_width; 
   enum dma_slave_buswidth dst_addr_width; 
   u32 src_maxburst; 
   u32 dst_maxburst; 
   [...] 
};
```

### 配置参数详解

该结构中每个元素的含义如下：

**1. direction** - 数据进入或者离开这个从通道。其可能取值：

```C++
/* DMA传输模式和方向指示器*/ 
enum dma_transfer_direction { 
    DMA_MEM_TO_MEM, /* Async/Memcpy 模式*/ 
    DMA_MEM_TO_DEV, /* 内存到设备 */ 
    DMA_DEV_TO_MEM, /* 设备到内存 */ 
    DMA_DEV_TO_DEV, /* 设备到设备 */ 
    [...] 
};
```

**2. src_addr/dst_addr** - 物理地址配置：

- `src_addr`：所要读取（RX）DMA从设备数据的缓冲器的物理地址（实际上是总线地址）。如果源是内存，则此元素将被忽略。
- `dst_addr`：所写入（TX）DMA从设备数据的缓冲区的物理地址（实际上是总线地址），如果源是内存，则该元素将被忽略。

**3. src_addr_width/dst_addr_width** - 总线宽度配置：

- `src_addr_width`：所读取DMA数据（RX）的源寄存器的字节宽度。如果源是内存，则根据架构不同，这可能会被忽略。其有效值为1、2、4或8。
- `dst_addr_width`：与src_addr_width相同，但是用于目的目标（TX）。

所有总线宽度必须是以下枚举之一：

```C++
enum dma_slave_buswidth { 
        DMA_SLAVE_BUSWIDTH_UNDEFINED = 0, 
        DMA_SLAVE_BUSWIDTH_1_BYTE = 1, 
        DMA_SLAVE_BUSWIDTH_2_BYTES = 2, 
        DMA_SLAVE_BUSWIDTH_3_BYTES = 3, 
        DMA_SLAVE_BUSWIDTH_4_BYTES = 4, 
        DMA_SLAVE_BUSWIDTH_8_BYTES = 8, 
        DMA_SLAVE_BUSWIDTH_16_BYTES = 16, 
        DMA_SLAVE_BUSWIDTH_32_BYTES = 32, 
        DMA_SLAVE_BUSWIDTH_64_BYTES = 64, 
};
```

**4. src_maxburst/dst_maxburst** - 突发传输配置：

- `src_maxburst`：一次突发中可以发送到设备的最大字数（这里，将字作为src_addr_width成员的单位，而不是字节）。通常情况下，这是I/O外设FIFO大小的一半，不会使其溢出。这可能适用于内存源，也可能不适用。
- `dst_maxburst`：与src_maxburst相同，但是用于目的目标。

> [!note]+ 参数设置技巧
> 
> - 方向(direction)：根据你的数据流向选择
> - 地址(addr)：外设寄存器的物理地址，从设备datasheet获取
> - 宽度(width)：匹配外设的数据寄存器宽度
> - 突发(burst)：通常设为外设FIFO深度的一半，避免溢出

### 配置示例

```C++
struct dma_chan *my_dma_chan; 
dma_addr_t dma_src, dma_dst; 
struct dma_slave_config my_dma_cfg = {0}; 
 
/* 没有过滤器回调，也没有过滤器参数 */ 
my_dma_chan = dma_request_channel(my_dma_cap_mask, 0, NULL); 

/* scr_addr和dst_addr被添加到这个结构中，所以可以复制 */ 
my_dma_cfg.direction = DMA_MEM_TO_MEM; 
my_dma_cfg.dst_addr_width = DMA_SLAVE_BUSWIDTH_32_BYTES; 
 
dmaengine_slave_config(my_dma_chan, &my_dma_cfg); 
 
char *rx_data, *tx_data; 
/* 没有错误检查 */ 
rx_data = kzalloc(BUFFER_SIZE, GFP_DMA); 
tx_data = kzalloc(BUFFER_SIZE, GFP_DMA); 
 
feed_data(tx_data); 
 
/* 得到DMA地址 */ 
dma_src_addr = dma_map_single(NULL, tx_data, 
                             BUFFER_SIZE, DMA_MEM_TO_MEM); 
dma_dst_addr = dma_map_single(NULL, rx_data, 
                             BUFFER_SIZE, DMA_MEM_TO_MEM);
```

上面的摘录中，调用dma_request_channel()函数来获取DMA通道的所有者芯片，在其上调用dmaengine_slave_config()来应用其配置。调用dma_map_single()映射rx和tx缓冲区，以便在DMA中使用它们。

## 步骤3：获取传输描述符（tx descriptor）

DMA传输属于异步传输，在启动传输之前，slave driver需要将此次传输的一些信息（例如src/dst的buffer、传输的方向等）提交给dma engine（本质上是dma controller driver），dma engine确认okay后，返回一个描述符（由struct dma_async_tx_descriptor抽象）。此后，slave driver就可以以该描述符为单位，控制并跟踪此次传输。

### 为什么需要描述符

是否还记得本节的第一步中请求DMA通道时，其返回值是struct dma_chan结构的实例。如果查看include/linux/dmaengine.h中对它的定义，就会发现它包含struct dma_device * device字段，它表示提供该通道的DMA设备（实际上是控制器）。此控制器的内核驱动程序负责（这是内核API要求DMA控制器驱动程序遵守的规则）提供一组函数来准备DMA事务，其中每个函数都对应于DMA事务类型（第1步中已经列举）。

> [!tip]+ 理解要点 传输描述符就像"快递单"：
> 
> - 记录了这次传输的所有信息（从哪到哪、多少数据等）
> - 可以用来追踪传输状态
> - 一个描述符对应一次DMA传输任务

### 获取描述符的API

根据传输模式的不同，slave driver可以使用下面三个API获取传输描述符：

```C
struct dma_async_tx_descriptor *dmaengine_prep_slave_sg(
        struct dma_chan *chan, struct scatterlist *sgl,
        unsigned int sg_len, enum dma_data_direction direction,
        unsigned long flags);

struct dma_async_tx_descriptor *dmaengine_prep_dma_cyclic(
        struct dma_chan *chan, dma_addr_t buf_addr, size_t buf_len,
        size_t period_len, enum dma_data_direction direction);

struct dma_async_tx_descriptor *dmaengine_prep_interleaved_dma(
        struct dma_chan *chan, struct dma_interleaved_template *xt,
        unsigned long flags);
```

### 准备函数详解

根据事务类型不同，可能只能选择专用函数。其中的一些函数如下：

- `device_prep_dma_memcpy()`：准备memcpy操作
- `device_prep_dma_sg()`：准备分散/聚集memcpy操作
- `device_prep_dma_xor()`：对于异或操作
- `device_prep_dma_xor_val()`：准备异或验证操作
- `device_prep_dma_pq()`：准备pq操作
- `device_prep_dma_pq_val()`：准备pqzero_sum操作
- `device_prep_dma_memset()`：准备memset操作
- `device_prep_dma_memset_sg()`：用于分散列表上的memset操作
- `device_prep_slave_sg()`：准备从设备DMA操作
- `device_prep_interleaved_dma()`：以通用方式传输表达式

### dmaengine_prep_slave_sg详解

dmaengine_prep_slave_sg用于在"scatter gather buffers"列表和总线设备之间进行DMA传输，参数如下：

- `chan`：本次传输所使用的dma channel
- `sgl`：要传输的"scatter gather buffers"数组的地址
- `sg_len`："scatter gather buffers"数组的长度
- `direction`：数据传输的方向，具体可参考enum dma_data_direction（include/linux/dma-direction.h）的定义
- `flags`：可用于向dma controller driver传递一些额外的信息，包括（具体可参考enum dma_ctrl_flags中以DMA_PREP_开头的定义）：
    - `DMA_PREP_INTERRUPT`：告诉DMA controller driver，本次传输完成后，产生一个中断，并调用client提供的回调函数
    - `DMA_PREP_FENCE`：告诉DMA controller driver，后续的传输，依赖本次传输的结果（这样controller driver就会小心的组织多个dma传输之间的顺序）
    - `DMA_PREP_PQ_DISABLE_P`、`DMA_PREP_PQ_DISABLE_Q`、`DMA_PREP_CONTINUE`：PQ有关的操作

### 准备描述符示例

调用dma_dev->device_prep_dma_memcpy(chan, dst, src, len, flag)把dma操作的参数传给dma子系统。同时返回一个从chan申请的异步传输描述符：struct dma_async_tx_descriptor。

```C++
/* 准备一个内存到内存的DMA传输 */
struct dma_async_tx_descriptor *tx;
struct dma_device *dma_dev = chan->device;

tx = dma_dev->device_prep_dma_memcpy(chan, dma_dst_addr, dma_src_addr,
                                     BUFFER_SIZE, DMA_PREP_INTERRUPT);
if (!tx) {
    pr_err("Failed to prepare DMA memcpy\n");
    return -ENOMEM;
}
```

## 步骤4：提交事务并启动传输

获取传输描述符之后，client driver可以通过dmaengine_submit接口将该描述符放到传输队列上，然后调用dma_async_issue_pending接口，启动传输。

### 设置回调函数

可以把用户的回调函数设置在上面的描述符里。通常这里的回调函数里是一个complete函数，用来在传输完成后通知用户业务流程里的wait等待。

```C++
static void my_dma_callback(void *data) 
{ 
    struct completion *cmp = data;
    complete(cmp); 
    return;  
}
```

### 提交传输

要将事务放入驱动程序的等待队列中，需要调用dmaengine_submit()。一旦准备好描述符并添加回调信息，就应该将其放在DMA引擎驱动程序等待队列中：

dmaengine_submit的原型如下：

```C
dma_cookie_t dmaengine_submit(struct dma_async_tx_descriptor *desc)
```

该函数返回一个cookie，可以通过其他DMA引擎检查DMA活动的进度。dmaengine_submit()不会启动DMA操作，它只是将其添加到待处理的队列中。

tx->tx_submit(tx)把请求提交，dma_submit_error判断提交的请求是否合法。

### 完整的提交示例

```C++
struct completion transfer_ok; 
init_completion(&transfer_ok); 

/* 设置回调函数和回调参数 */
tx->callback = my_dma_callback; 
tx->callback_param = &transfer_ok;
 
/* 提交DMA传输*/ 
dma_cookie_t cookie = dmaengine_submit(tx); 
 
if (dma_submit_error(cookie)) { 
    printk(KERN_ERR "%s: Failed to start DMA transfer\n", __FUNCTION__); 
    /* 处理错误 */ 
    return -EINVAL;
}
```

> [!warning]+ 注意事项 dmaengine_submit()只是把传输请求加入队列，并不会真正启动传输！就像把快递单贴好了，但还没有通知快递员来取件。

### 启动传输

启动事务是DMA传输设置的最后一步。在通道上调用dma_async_issue_pending()来激活通道待处理队列中的事务。如果通道空闲，则队列中的第一个事务将启动，后续事务排队等候。DMA操作完成时，队列中的下一个事务启动，并触发软中断（tasklet）。如果已经设置，则该tasklet负责调用客户端驱动程序的完成回调例程进行通知：

dma_async_issue_pending的原型如下：

```C
void dma_async_issue_pending(struct dma_chan *chan);
```

dma_async_issue_pending触发请求真正执行。

下面是一个例子：

```C++
/* 触发DMA传输开始 */
dma_async_issue_pending(my_dma_chan); 

/* 等待传输完成 */
wait_for_completion(&transfer_ok); 
 
/* 解除DMA映射 */
dma_unmap_single(my_dma_chan->device->dev, dma_src_addr, 
                BUFFER_SIZE, DMA_MEM_TO_MEM); 
dma_unmap_single(my_dma_chan->device->dev, dma_dst_addr, 
                BUFFER_SIZE, DMA_MEM_TO_MEM); 
 
/* 通过rx_data和tx_data虚拟地址处理缓冲区*/
```

## 步骤5：等待传输结束

如上，在发送请求之后，一般可以在这里wait等待，通过上面注册的回调函数在dma执行完成后通知这里的wait。

wait_for_completion()函数将被阻塞，直到DMA回调被调用，这将更新（填写）完成变量，以恢复先前阻塞的代码。这适用于替代while(!done) msleep(SOME_TIME);方法。

### 查询传输状态

dma_async_is_tx_complete查看请求的状态：

```C++
enum dma_status dma_async_is_tx_complete(struct dma_chan *chan,
                                        dma_cookie_t cookie,
                                        dma_cookie_t *last,
                                        dma_cookie_t *used);
```

返回值可能是：

- `DMA_COMPLETE`：传输完成
- `DMA_IN_PROGRESS`：传输进行中
- `DMA_PAUSED`：传输暂停
- `DMA_ERROR`：传输错误

### 清理资源

做完dma操作之后使用dma_release_channel释放申请的dma channel：

```C++
/* 释放DMA通道 */
dma_release_channel(my_dma_chan);

/* 释放分配的内存 */
kfree(tx_data);
kfree(rx_data);
```

## 完整使用示例

让我们通过一个完整的例子来巩固上述知识：

```C++
#include <linux/dmaengine.h>
#include <linux/completion.h>
#include <linux/slab.h>

#define BUFFER_SIZE 4096

/* DMA完成回调函数 */
static void dma_complete_callback(void *data)
{
    struct completion *cmp = data;
    complete(cmp);
}

/* DMA传输示例函数 */
int perform_dma_transfer(void)
{
    struct dma_chan *chan;
    struct dma_device *dma_dev;
    struct dma_async_tx_descriptor *tx;
    struct dma_slave_config config = {0};
    dma_cap_mask_t mask;
    dma_cookie_t cookie;
    struct completion cmp;
    char *src_buf, *dst_buf;
    dma_addr_t src_dma, dst_dma;
    int ret = 0;

    /* 1. 申请DMA通道 */
    dma_cap_zero(mask);
    dma_cap_set(DMA_MEMCPY, mask);
    chan = dma_request_channel(mask, NULL, NULL);
    if (!chan) {
        pr_err("Failed to request DMA channel\n");
        return -ENODEV;
    }

    dma_dev = chan->device;

    /* 2. 配置DMA参数（对于内存到内存传输，通常不需要） */
    config.direction = DMA_MEM_TO_MEM;
    config.src_addr_width = DMA_SLAVE_BUSWIDTH_4_BYTES;
    config.dst_addr_width = DMA_SLAVE_BUSWIDTH_4_BYTES;
    config.src_maxburst = 16;
    config.dst_maxburst = 16;

    ret = dmaengine_slave_config(chan, &config);
    if (ret) {
        pr_err("Failed to config DMA channel\n");
        goto release_channel;
    }

    /* 分配测试缓冲区 */
    src_buf = kzalloc(BUFFER_SIZE, GFP_KERNEL | GFP_DMA);
    dst_buf = kzalloc(BUFFER_SIZE, GFP_KERNEL | GFP_DMA);
    if (!src_buf || !dst_buf) {
        ret = -ENOMEM;
        goto free_buf;
    }

    /* 填充源数据 */
    memset(src_buf, 0xAA, BUFFER_SIZE);

    /* 获取DMA地址 */
    src_dma = dma_map_single(dma_dev->dev, src_buf, 
                            BUFFER_SIZE, DMA_TO_DEVICE);
    dst_dma = dma_map_single(dma_dev->dev, dst_buf,
                            BUFFER_SIZE, DMA_FROM_DEVICE);

    /* 3. 准备DMA传输描述符 */
    tx = dma_dev->device_prep_dma_memcpy(chan, dst_dma, src_dma,
                                        BUFFER_SIZE, DMA_PREP_INTERRUPT);
    if (!tx) {
        pr_err("Failed to prepare DMA transfer\n");
        ret = -ENOMEM;
        goto unmap_dma;
    }

    /* 设置完成回调 */
    init_completion(&cmp);
    tx->callback = dma_complete_callback;
    tx->callback_param = &cmp;

    /* 4. 提交并启动传输 */
    cookie = dmaengine_submit(tx);
    if (dma_submit_error(cookie)) {
        pr_err("Failed to submit DMA transfer\n");
        ret = -EINVAL;
        goto unmap_dma;
    }

    dma_async_issue_pending(chan);

    /* 5. 等待传输完成 */
    wait_for_completion(&cmp);

    /* 验证数据 */
    if (memcmp(src_buf, dst_buf, BUFFER_SIZE) == 0) {
        pr_info("DMA transfer successful!\n");
    } else {
        pr_err("DMA transfer data mismatch!\n");
        ret = -EIO;
    }

unmap_dma:
    dma_unmap_single(dma_dev->dev, src_dma, BUFFER_SIZE, DMA_TO_DEVICE);
    dma_unmap_single(dma_dev->dev, dst_dma, BUFFER_SIZE, DMA_FROM_DEVICE);

free_buf:
    kfree(src_buf);
    kfree(dst_buf);

release_channel:
    dma_release_channel(chan);

    return ret;
}
```

> [!tip]+ 调试技巧
> 
> 1. 使用`dmesg`查看DMA相关的内核日志
> 2. 检查`/sys/class/dma/`下的通道信息
> 3. 使用`dmatest`模块验证DMA功能：
>     
>     ```bash
>     modprobe dmatestecho 1 > /sys/module/dmatest/parameters/run
>     ```
>     

## 总结

通过本章的学习，我们掌握了DMA Engine API的使用方法：

1. **申请通道**：获取DMA服务的入口
2. **配置参数**：告诉DMA控制器传输细节
3. **准备描述符**：创建传输任务单
4. **提交启动**：将任务加入队列并执行
5. **等待完成**：同步等待或异步通知

整个流程就像快递服务：选择快递公司（申请通道）→ 填写快递单（配置参数）→ 打包货物（准备描述符）→ 下单发货（提交启动）→ 等待签收（等待完成）。

> [!note]+ 进阶提示 掌握了DMA Engine的使用后，下一步可以：
> 
> - 学习DMA子系统的整体架构
> - 了解如何编写DMA控制器驱动
> - 研究高级特性如scatter-gather、cyclic DMA等