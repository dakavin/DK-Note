DMA引擎是开发DMA控制器驱动程序的通用内核框架。DMA的主要目标是在复制内存时减轻CPU的负担。使用通道将事务（I/O数据传输）委托给DMA引擎，DMA引擎通过其驱动程序/API提供一组可供其他设备（从设备）使用的通道。

这里将简单介绍这个（从设备）API，它只适用于从设备DMA用法。这里必须引用的头文件如下：

```C++
#include <linux/dmaengine.h>
```

从设备DMA用法很简单，它包含以下步骤：

- 分配DMA从通道。
    
- 设置从设备和控制器的特定参数。
    
- 获取事务的描述符。
    
- 提交事务。
    
- 发出挂起的请求并等待回调通知。
    

## 分配DMA从通道

使用dma_request_channel()请求通道。其原型如下：

```C++
struct dma_chan *dma_request_channel(const dma_cap_mask_t *mask, 
                          dma_filter_fn fn, void *fn_param);
```

mask是位图掩码，表示该通道必须满足的功能。使用它主要是为了指定驱动程序需要执行的传输类型：

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

dma_cap_zero()和dma_cap_set()函数用于清除掩码，设置所需的功能。例如：

```C++
dma_cap_mask my_dma_cap_mask; 
struct dma_chan *chan; 
dma_cap_zero(my_dma_cap_mask); 
dma_cap_set(DMA_MEMCPY, my_dma_cap_mask); /* 内存到内存复制 */ 
chan = dma_request_channel(my_dma_cap_mask, NULL, NULL);
```

在前面 dma_filter_fn定义为：

```C++
typedef bool (*dma_filter_fn)(struct dma_chan *chan,  
                void *filter_param);
```

如果filter_fn参数（这是可选项）为NULL，则dma_request_channel()将只返回满足功能掩码的第一个通道。否则，当掩码参数不足以指定所需的通道时，可以使用filter_fn例程作为系统中可用通道的过滤器。内核为系统中的每个空闲通道调用一次filter_fn例程。在看到合适的通道时，filter_fn应该返回DMA_ACK，它将把给定的通道标记为dma_request_channel()的返回值。

通过此接口分配的通道由调用者独占，直到调用dma_release_channel()为止：

```C++
void dma_release_channel(struct dma_chan *chan)
```

  

## 设置从设备和控制器指定参数

此步骤引入了新的数据结构struct dma_slave_config，它表示DMA从通道的运行时配置。这允许客户端为外设指定设置，如DMA方向、DMA地址、总线宽度、DMA突发长度（DMA burst length）等。

```C++
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

该结构中每个元素的含义如下。

- direction：数据进入或者离开这个从通道。其可能取值：
    

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

- src_addr：所要读取（RX）DMA从设备数据的缓冲器的物理地址（实际上是总线地址）。如果源是内存，则此元素将被忽略。dst_addr是所写入（TX） DMA从设备数据的缓冲区的物理地址（实际上是总线地址），如果源是内存，则该元素将被忽略。src_addr_width是所读取DMA数据（RX）的源寄存器的字节宽度。如果源是内存，则根据架构不同，这可能会被忽略。其有效值为1、2、4或8。因此，dst_addr_width与src_addr_width相同，但是用于目的目标（TX）。
    
- 所有总线宽度必须是以下枚举之一：
    

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

- src_maxburs：一次突发中可以发送到设备的最大字数（这里，将字作为src_addr_width成员的单位，而不是字节）。通常情况下，这是I/O外设FIFO大小的一半，不会使其溢出。这可能适用于内存源，也可能不适用。dst_maxburst与src_maxburst相同，但是用于目的目标。
    

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

## 获取事务描述符

是否还记得本节的第一步中请求DMA通道时，其返回值是struct dma_chan结构的实例。如果查看include/linux/dmaengine.h中对它的定义，就会发现它包含struct dma_device * device字段，它表示提供该通道的DMA设备（实际上是控制器）。此控制器的内核驱动程序负责（这是内核API要求DMA控制器驱动程序遵守的规则）提供一组函数来准备DMA事务，其中每个函数都对应于DMA事务类型（第1步中已经列举）。根据事务类型不同，可能只能选择专用函数。其中的一些函数如下。

- device_prep_dma_memcpy()：准备memcpy操作。
    
- device_prep_dma_sg()：准备分散/聚集memcpy操作。
    
- device_prep_dma_xor()：对于异或操作。
    
- device_prep_dma_xor_val()：准备异或验证操作。
    
- device_prep_dma_pq()：准备pq操作。
    
- device_prep_dma_pq_val()：准备pqzero_sum操作。
    
- device_prep_dma_memset()：准备memset操作。
    
- device_prep_dma_memset_sg()：用于分散列表上的memset操作。
    
- device_prep_slave_sg()：准备从设备DMA操作。
    
- device_prep_interleaved_dma()：以通用方式传输表达式。
    

  

## 提交事务


要将事务放入驱动程序的等待队列中，需要调用dmaengine_submit()。一旦准备好描述符并添加回调信息，就应该将其放在DMA引擎驱动程序等待队列中：

```C++
dma_cookie_t dmaengine_submit(struct dma_async_tx_descriptor *desc)
```

该函数返回一个cookie，可以通过其他DMA引擎检查DMA活动的进度。dmaengine_submit()不会启动DMA操作，它只是将其添加到待处理的队列中。下一步将讨论如何启动事务：

```C++
struct completion transfer_ok; 
init_completion(&transfer_ok); 
tx->callback = my_dma_callback; 
 
/* 提交DMA传输*/ 
dma_cookie_t cookie = dmaengine_submit(tx); 
 
if (dma_submit_error(cookie)) { 
    printk(KERN_ERR "%s: Failed to start DMA transfer\n", __FUNCTION__); 
    /* 处理 */ 
[...]  
}
```

  

## 发布待处理DMA请求并等待回调通知

启动事务是DMA传输设置的最后一步。在通道上调用dma_async_issue_pending()来激活通道待处理队列中的事务。如果通道空闲，则队列中的第一个事务将启动，后续事务排队等候。DMA操作完成时，队列中的下一个事务启动，并触发软中断（tasklet）。如果已经设置，则该tasklet负责调用客户端驱动程序的完成回调例程进行通知：

```C++
void dma_async_issue_pending(struct dma_chan *chan);
```

下面是一个例子：

```C++
dma_async_issue_pending(my_dma_chan); 
wait_for_completion(&transfer_ok); 
 
dma_unmap_single(my_dma_chan->device->dev, dma_src_addr, 
BUFFER_SIZE, DMA_MEM_TO_MEM); 
dma_unmap_single(my_dma_chan->device->dev, dma_src_addr, 
               BUFFER_SIZE, DMA_MEM_TO_MEM); 
 
/* 通过rx_data和tx_data虚拟地址处理缓冲区*/
```

wait_for_completion()函数将被阻塞，直到DMA回调被调用，这将更新（填写）完成变量，以恢复先前阻塞的代码。这适用于替代while(!done) msleep(SOME_TIME);方法。

```C++
static void my_dma_callback() 
{ 
    complete(transfer_ok); 
    return;  
}
```