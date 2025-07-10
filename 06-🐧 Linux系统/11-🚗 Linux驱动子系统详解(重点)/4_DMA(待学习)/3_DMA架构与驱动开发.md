
## 引言：从使用到实现

在前面的章节中，我们学习了如何使用DMA Engine API进行DMA传输。现在，让我们深入了解DMA子系统的内部架构，以及如何开发DMA控制器驱动。

理解DMA子系统架构对于：

- 调试DMA相关问题
- 优化DMA性能
- 开发新的DMA控制器驱动
- 扩展DMA功能

都是必不可少的。

## DMA子系统架构

DMA子系统是一个相对比较简单的子系统，整个框架比较薄，Linux下DMA子系统的框架图见下图蓝色部分：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/288fee0aadaadc7841eefdbe42c3e5ea.png)

### 子系统组成

Linux DMA子系统主要有5部分组成：

**1. dmaengine** - 核心框架

- 是DMA子系统的核心，为DMA Device Driver提供DMA设备注册的API
- 为DMA调用者（Device Driver）提供屏蔽DMA设备实现细节的统一接口API
- 管理所有DMA channels和devices
- 处理通道分配和资源管理

**2. virt-dma** - 虚拟DMA支持

- 为DMA子系统提供虚拟DMA channel的支持
- 简化DMA驱动开发
- 提供通用的描述符管理
- 实现软件队列管理

**3. of-dma** - 设备树支持

- 为DMA子系统提供设备树描述DMA信息传入的支持
- 解析DMA相关的设备树节点
- 建立DMA provider和consumer的绑定关系
- 支持DMA路由配置

**4. DMA Device Driver** - 具体驱动

- 为DMA设备的驱动程序
- 实现硬件相关的操作
- 向dmaengine注册设备
- 提供channel操作回调

**5. dmatest** - 测试模块

- 提供dmatest模块测试DMA memcpy, memset, XOR和RAID6 P+Q操作
- 测试各种长度和各种偏移量进入源和目标缓冲区
- 验证DMA功能的正确性
- 性能基准测试

> [!tip]+ 架构理解 DMA子系统采用分层设计：
> 
> - **应用层**：使用DMA的驱动（如网卡、音频等）
> - **框架层**：dmaengine提供统一接口
> - **驱动层**：具体DMA控制器驱动
> - **硬件层**：实际的DMA控制器

### 核心数据流

```
设备驱动 (Consumer)
    ↓ (DMA Engine API)
DMA Engine Core
    ↓ (回调函数)
DMA Controller Driver
    ↓ (硬件操作)
DMA Hardware
```

## DMA Engine驱动开发

### 驱动需要提供的回调函数

可以看到，DMA系统在dma engine注册的时候需要设备驱动提供的一套回调函数来支持第一小节里的各个API，这些回调函数操作具体硬件，完成相关硬件的配置。我们这里可以以MEMCPY要提供的回调函数示例说明回调函数的意义。

**主要回调函数及其作用：**

- **device_alloc_chan_resources**
    
    - 分配chan的硬件资源
    - 在通道被分配给consumer时调用
    - 初始化硬件相关的资源
    - 分配描述符池
- **device_free_chan_resources**
    
    - 释放chan的硬件资源
    - 在通道被释放时调用
    - 清理硬件状态
    - 释放描述符池
- **device_prep_dma_memcpy**
    
    - 接收用户传入的请求，分配驱动层面的用户请求
    - 准备内存拷贝操作的描述符
    - 配置源地址、目标地址、长度等参数
    - 返回传输描述符
- **device_issue_pending**
    
    - 操作硬件发起具体的dma请求
    - 启动硬件传输
    - 触发DMA控制器开始工作
    - 处理队列中的待处理描述符

### 基本的驱动框架

```c
/* DMA设备结构 */
struct my_dma_device {
    struct dma_device dma_dev;
    void __iomem *base;
    struct clk *clk;
    /* 硬件相关字段 */
};

/* DMA通道结构 */
struct my_dma_chan {
    struct dma_chan chan;
    void __iomem *ch_base;
    /* 通道相关字段 */
};

/* 分配通道资源 */
static int my_dma_alloc_chan_resources(struct dma_chan *chan)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    
    /* 初始化硬件通道 */
    /* 分配描述符池 */
    
    return 0;
}

/* 准备内存拷贝 */
static struct dma_async_tx_descriptor *
my_dma_prep_memcpy(struct dma_chan *chan, dma_addr_t dst, dma_addr_t src,
                   size_t len, unsigned long flags)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    struct my_dma_desc *desc;
    
    /* 分配描述符 */
    desc = my_dma_alloc_desc(mchan);
    if (!desc)
        return NULL;
    
    /* 配置传输参数 */
    desc->src_addr = src;
    desc->dst_addr = dst;
    desc->len = len;
    
    return &desc->tx;
}

/* 启动待处理的传输 */
static void my_dma_issue_pending(struct dma_chan *chan)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    
    /* 启动硬件传输 */
    writel(DMA_START, mchan->ch_base + DMA_CONTROL);
}

/* 驱动probe函数 */
static int my_dma_probe(struct platform_device *pdev)
{
    struct my_dma_device *mdev;
    struct dma_device *dma_dev;
    int ret;
    
    mdev = devm_kzalloc(&pdev->dev, sizeof(*mdev), GFP_KERNEL);
    if (!mdev)
        return -ENOMEM;
    
    /* 初始化DMA设备 */
    dma_dev = &mdev->dma_dev;
    dma_cap_set(DMA_MEMCPY, dma_dev->cap_mask);
    
    /* 设置回调函数 */
    dma_dev->device_alloc_chan_resources = my_dma_alloc_chan_resources;
    dma_dev->device_free_chan_resources = my_dma_free_chan_resources;
    dma_dev->device_prep_dma_memcpy = my_dma_prep_memcpy;
    dma_dev->device_issue_pending = my_dma_issue_pending;
    
    /* 注册DMA设备 */
    ret = dma_async_device_register(dma_dev);
    if (ret)
        return ret;
    
    return 0;
}
```

## virt-dma的使用

分析现有的dma驱动，可以看到里面用了virt-dma.[ch]里提供的接口。这里也简单看下virt-dma的使用方法。

### virt-dma核心概念

virt-dma的核心数据结构是一组链表，这组链表记录处于不同阶段的dma请求：

1. **desc_allocated** - 已分配的描述符
2. **desc_submitted** - 已提交的描述符
3. **desc_issued** - 已发布的描述符
4. **desc_completed** - 已完成的描述符

### 描述符的生命周期

当用e.g. device_prep_dma_memcpy创建一个请求后，这个请求应该挂入desc_allocated链表。

当用tx->tx_submit提交这个请求后，应该把请求挂入desc_submitted链表。

当用dma_async_issue_pending执行请求后，应该把请求挂入desc_issued链表。

当最后请求执行完成后，应该挂入desc_completed链表。

> [!tip]+ 状态流转
> 
> ```
> 分配(allocated) → 提交(submitted) → 发布(issued) → 完成(completed)
>      ↓                ↓                 ↓              ↓
> prep_dma_xxx      tx_submit      issue_pending    中断处理
> ```

### virt-dma提供的便利

virt-dma在原来的dma_chan上封装了virt_dma_chan，在virt_dma_chan创建的时候，vchan_init为每一个vchan创建一个tasklet，设备驱动可以在中断处理里调用e.g. vchan_cookie_complete->tasklet_schedule执行tasklet函数vchan_complete，这个函数里会执行dma请求中用户设置的回调函数。

### 使用virt-dma的示例

```c
#include <linux/dmaengine.h>
#include "virt-dma.h"

/* 虚拟DMA通道 */
struct my_dma_chan {
    struct virt_dma_chan vchan;
    /* 硬件相关字段 */
};

/* 虚拟DMA描述符 */
struct my_dma_desc {
    struct virt_dma_desc vdesc;
    dma_addr_t src_addr;
    dma_addr_t dst_addr;
    size_t len;
};

/* 描述符释放函数 */
static void my_dma_desc_free(struct virt_dma_desc *vdesc)
{
    struct my_dma_desc *desc = to_my_dma_desc(vdesc);
    kfree(desc);
}

/* 准备传输 */
static struct dma_async_tx_descriptor *
my_dma_prep_memcpy(struct dma_chan *chan, dma_addr_t dst, dma_addr_t src,
                   size_t len, unsigned long flags)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    struct my_dma_desc *desc;
    
    desc = kzalloc(sizeof(*desc), GFP_NOWAIT);
    if (!desc)
        return NULL;
    
    desc->src_addr = src;
    desc->dst_addr = dst;
    desc->len = len;
    
    /* 使用virt-dma初始化描述符 */
    return vchan_tx_prep(&mchan->vchan, &desc->vdesc, flags);
}

/* 启动传输 */
static void my_dma_issue_pending(struct dma_chan *chan)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    unsigned long flags;
    
    spin_lock_irqsave(&mchan->vchan.lock, flags);
    if (vchan_issue_pending(&mchan->vchan))
        my_dma_start_transfer(mchan);
    spin_unlock_irqrestore(&mchan->vchan.lock, flags);
}

/* 中断处理 */
static irqreturn_t my_dma_interrupt(int irq, void *dev_id)
{
    struct my_dma_chan *mchan = dev_id;
    struct my_dma_desc *desc;
    
    /* 读取并清除中断状态 */
    
    /* 获取当前描述符 */
    desc = mchan->current_desc;
    if (desc) {
        /* 标记完成并调度tasklet */
        vchan_cookie_complete(&desc->vdesc);
    }
    
    /* 尝试启动下一个传输 */
    my_dma_start_next_transfer(mchan);
    
    return IRQ_HANDLED;
}
```

> [!note]+ virt-dma优势
> 
> 1. **简化开发**：自动管理描述符的生命周期
> 2. **统一接口**：提供标准的链表操作
> 3. **异步处理**：自动创建tasklet处理完成回调
> 4. **错误处理**：提供完善的错误处理机制

## 驱动开发最佳实践

### 1. 资源管理

```c
/* 使用devm_*函数自动管理资源 */
static int my_dma_probe(struct platform_device *pdev)
{
    struct my_dma_device *mdev;
    struct resource *res;
    
    mdev = devm_kzalloc(&pdev->dev, sizeof(*mdev), GFP_KERNEL);
    
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    mdev->base = devm_ioremap_resource(&pdev->dev, res);
    
    mdev->clk = devm_clk_get(&pdev->dev, NULL);
    
    /* 资源会在驱动卸载时自动释放 */
}
```

### 2. 中断处理

```c
/* 中断处理要快速完成 */
static irqreturn_t my_dma_isr(int irq, void *data)
{
    struct my_dma_device *mdev = data;
    u32 status;
    
    /* 读取中断状态 */
    status = readl(mdev->base + DMA_INT_STATUS);
    
    /* 清除中断 */
    writel(status, mdev->base + DMA_INT_CLEAR);
    
    /* 调度底半部处理 */
    if (status & DMA_TRANS_COMPLETE)
        tasklet_schedule(&mdev->tasklet);
    
    return IRQ_HANDLED;
}
```

### 3. 并发控制

```c
/* 保护共享资源 */
static void my_dma_start_transfer(struct my_dma_chan *mchan)
{
    unsigned long flags;
    
    spin_lock_irqsave(&mchan->lock, flags);
    
    /* 操作硬件寄存器 */
    writel(mchan->src_addr, mchan->base + DMA_SRC_ADDR);
    writel(mchan->dst_addr, mchan->base + DMA_DST_ADDR);
    writel(mchan->len, mchan->base + DMA_LEN);
    writel(DMA_START, mchan->base + DMA_CONTROL);
    
    spin_unlock_irqrestore(&mchan->lock, flags);
}
```

### 4. 错误处理

```c
/* 完善的错误处理 */
static int my_dma_alloc_chan_resources(struct dma_chan *chan)
{
    struct my_dma_chan *mchan = to_my_dma_chan(chan);
    int ret;
    
    /* 申请中断 */
    ret = request_irq(mchan->irq, my_dma_isr, 0, "my-dma", mchan);
    if (ret) {
        dev_err(chan->device->dev, "Failed to request IRQ\n");
        return ret;
    }
    
    /* 初始化硬件 */
    ret = my_dma_hw_init(mchan);
    if (ret) {
        free_irq(mchan->irq, mchan);
        return ret;
    }
    
    return 0;
}
```

## 调试技巧

### 1. 使用debugfs

```c
/* 创建debugfs接口 */
static int my_dma_debugfs_init(struct my_dma_device *mdev)
{
    mdev->debugfs = debugfs_create_dir("my-dma", NULL);
    
    debugfs_create_u32("transfers", 0444, mdev->debugfs,
                       &mdev->transfer_count);
    
    debugfs_create_file("status", 0444, mdev->debugfs,
                        mdev, &my_dma_status_fops);
    
    return 0;
}
```

### 2. 添加跟踪点

```c
/* 在关键路径添加跟踪 */
static void my_dma_start_transfer(struct my_dma_chan *mchan)
{
    trace_printk("Starting DMA transfer: src=%pad dst=%pad len=%zu\n",
                 &mchan->src_addr, &mchan->dst_addr, mchan->len);
    
    /* 启动传输 */
}
```

### 3. 使用dmatest验证

```bash
# 加载dmatest模块
modprobe dmatest

# 设置测试参数
echo 4096 > /sys/module/dmatest/parameters/test_buf_size
echo 100 > /sys/module/dmatest/parameters/iterations
echo "my-dma-chan0" > /sys/module/dmatest/parameters/channel

# 运行测试
echo 1 > /sys/module/dmatest/parameters/run

# 查看结果
dmesg | grep dmatest
```

> [!warning]+ 常见问题
> 
> 1. **死锁**：注意中断上下文和进程上下文的锁使用
> 2. **内存泄漏**：确保描述符正确释放
> 3. **时序问题**：硬件操作要考虑时序要求
> 4. **缓存一致性**：DMA操作要处理缓存同步

## 总结

通过本章的学习，我们了解了：

1. **DMA子系统架构**：
    
    - 5个主要组成部分的职责
    - 分层设计的优势
    - 各组件的协作关系
2. **驱动开发要点**：
    
    - 必须实现的回调函数
    - virt-dma的使用方法
    - 描述符的生命周期管理
3. **最佳实践**：
    
    - 资源管理技巧
    - 并发控制方法
    - 调试和验证手段

DMA子系统虽然框架简单，但要写好一个DMA驱动需要：

- 深入理解硬件特性
- 熟悉内核编程规范
- 注意性能优化
- 做好错误处理

> [!tip]+ 学习建议
> 
> 1. 阅读现有DMA驱动代码（如pl330、dw-dmac等）
> 2. 使用dmatest充分测试你的驱动
> 3. 关注内核邮件列表中的DMA相关讨论
> 4. 实践是最好的老师，动手写一个简单的DMA驱动