
可以看到，DMA系统在dma engine注册的时候需要设备驱动提供的一套回调函数来支持

第一小节里的各个API，这些回调函数操作具体硬件，完成相关硬件的配置。我们这里可以

以MEMCPY要提供的回调函数示例说明回调函数的意义。

- device_alloc_chan_resources
    
- 分配chan的硬件资源
    
- device_free_chan_resources
    
- 释放chan的硬件资源
    
- device_prep_dma_memcpy
    
- 接收用户传入的请求，分配驱动层面的用户请求
    
- device_issue_pending
    
- 操作硬件发起具体的dma请求
    

分析现有的dma驱动，可以看到里面用了virt-dma.[ch]里提供的接口。这里也简单看下

virt-dma的使用方法。virt-dma的核心数据结构是一组链表，这组链表记录处于不同阶段

的dma请求。当用 e.g. device_prep_dma_memcpy创建一个请求后，这个请求应该挂入

desc_allocated链表，当用tx->tx_submit提交这个请求后，应该把请求挂入desc_submitted

链表，当用dma_async_issue_pending执行请求后，应该把请求挂入desc_issued链表，

当最后请求执行完成后，应该挂入desc_completed链表。virt-dma在原来的dma_chan上

封装了virt_dma_chan，在virt_dma_chan创建的时候, vchan_init为每一个vchan创建

一个tasklet，设备驱动可以在中断处理里调用 e.g. vchan_cookie_complete->tasklet_schedule

执行tasklet函数vchan_complete, 这个函数里会执行dma请求中用户设置的回调函数。