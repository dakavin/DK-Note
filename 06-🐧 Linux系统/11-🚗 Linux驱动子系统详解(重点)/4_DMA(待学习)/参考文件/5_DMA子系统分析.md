DMA子系统是一个相对比较简单的子系统，整个框架比较薄，Linux下DMA子系统的框架图见下图蓝色部分

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/288fee0aadaadc7841eefdbe42c3e5ea.png)


Linux DMA子系统主要有有5部分组成

- **dmaengine：**是DMA子系统的核心，为DMA Device Driver提供DMA设备注册的API，为DMA调用者（Device Driver）提供屏蔽DMA设备实现细节的统一接口API
    
- **virt-dma：**为DMA子系统提供虚拟DMA channel的支持
    
- **of-dma：**为DMA子系统提供设备树描述DMA信息传入的支持
    
- **DMA Device Driver：**为DMA设备的驱动程序
    
- **dmatest：**提供dmatest模块测试DMA memcpy, memset, XOR和RAID6 P+Q操作各种长度和各种偏移量进入源和目标缓冲区