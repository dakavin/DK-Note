DMA子系统下有一个帮助测试的测试驱动(drivers/dma/dmatest.c), 从这个测试驱动入手 我们了解到内核里的其他部分怎么使用DMA engine。配置内核，选则CONFIG_DMATEST可以 把这个模块选中，编译会生成dmatest.ko。可以参考这个文档来快速了解怎么使用dmatest.ko: [https://www.kernel.org/doc/html/v4.15/driver-api/dmaengine/dmatest.html](https://www.kernel.org/doc/html/v4.15/driver-api/dmaengine/dmatest.html).

具体上来讲，内核的其他模块使用dma engine的步骤是:


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

  

- 使用dma_request_channel先申请一个dma channel，之后的dma请求都基于这个申请 的dma channel。
    
- 调用dma_dev->device_prep_dma_memcpy(chan, dst, src, len, flag)把dma操作的参 数传给dma子系统。同时返回一个从chan申请的异步传输描述符: struct dma_async_tx_descriptor.
    
- 可以把用户的回调函数设置在上面的描述符里。通常这里的回调函数里是一个complete 函数，用来在传输完成后通知用户业务流程里的wait等待。
    
- tx->tx_submit(tx) 把请求提交。
    
- dma_submit_error 判断提交的请求是否合法。
    
- dma_async_issue_pending 触发请求真正执行。
    
- 如上，在发送请求之后，一般可以在这里wait等待，通过上面注册的回调函数在dma 执行完成后通知这里的wait。
    
- dma_async_is_tx_complete 查看请求的状态。
    
- 做完dma操作之后使用dma_release_channel释放申请的dma channel。
    

## 申请DMA channel

任何consumer（slave driver）在开始DMA传输之前，都要申请一个DMA channel。

consumer可以通过如下的API申请DMA channel：

```C++
struct dma_chan *dma_request_channel(struct device *dev, const char *name);
```

> 该接口会返回绑定在指定设备（dev）上名称为name的dma channel。dma engine的provider和consumer可以使用device tree、ACPI或者struct dma_slave_map类型的match table提供这种绑定关系

最后，申请得到的dma channel可以在不需要使用的时候通过下面的API释放掉：

```C++
void dma_release_channel(struct dma_chan *chan);
```

## 配置DMA channel的参数

  

driver申请到一个为自己使用的DMA channel之后，需要根据自身的实际情况，以及DMA controller的能力，对该channel进行一些配置。可配置的内容由struct dma_slave_config数据结构表示。driver将它们填充到一个struct dma_slave_config变量中后，可以调用如下API将这些信息告诉给DMA controller：

```C
int dmaengine_slave_config(struct dma_chan *chan, struct dma_slave_config *config)
```

  

## 获取传输描述（tx descriptor）

  

DMA传输属于异步传输，在启动传输之前，slave driver需要将此次传输的一些信息（例如src/dst的buffer、传输的方向等）提交给dma engine（本质上是dma controller driver），dma engine确认okay后，返回一个描述符（由struct dma_async_tx_descriptor抽象）。此后，slave driver就可以以该描述符为单位，控制并跟踪此次传输。

  

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

dmaengine_prep_slave_sg用于在“scatter gather buffers”列表和总线设备之间进行DMA传输，参数如下：

chan，本次传输所使用的dma channel。

sgl，要传输的“scatter gather buffers”数组的地址；

sg_len，“scatter gather buffers”数组的长度。

direction，数据传输的方向，具体可参考enum dma_data_direction （include/linux/dma-direction.h）的定义。

flags，可用于向dma controller driver传递一些额外的信息，包括（具体可参考enum dma_ctrl_flags中以DMA_PREP_开头的定义）：

DMA_PREP_INTERRUPT，告诉DMA controller driver，本次传输完成后，产生一个中断，并调用client提供的回调函数

DMA_PREP_FENCE，告诉DMA controller driver，后续的传输，依赖本次传输的结果（这样controller driver就会小心的组织多个dma传输之间的顺序）；

DMA_PREP_PQ_DISABLE_P、DMA_PREP_PQ_DISABLE_Q、DMA_PREP_CONTINUE，PQ有关的操作，TODO。

  

## 启动传输

获取传输描述符之后，client driver可以通过dmaengine_submit接口将该描述符放到传输队列上，然后调用dma_async_issue_pending接口，启动传输。

dmaengine_submit的原型如下：

```C
dma_cookie_t dmaengine_submit(struct dma_async_tx_descriptor *desc)
```

dma_async_issue_pending的原型如下：

```C
void dma_async_issue_pending(struct dma_chan *chan);
```

## 等待传输结束

`dma_async_tx_descriptor`