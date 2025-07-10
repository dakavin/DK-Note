
## Platform总线介绍

总线代表着同类设备需要共同遵循的工作时序，不同的总线对于物理电平的要求是不一样的，对于每个比特的电平维持宽度也是不一样的，总线上传递的命令也会有自己的格式约束。 在`Linux`系统中总线可分为两种：

- 一种是实际存在的总线（例如`I2C`、`SPI`、`USB`等总线）
    
- 另一种是**虚拟存在的总线**（`platform`总线）
    

为什么需要虚拟的platform总线？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bb0982477f28be70af3049a752e0e46f.png)


是因为在嵌入式系统里面`SoC`系统中**集成的独立的外设控制器**、挂接在`SoC`内存空间的外设等无法依附于第一种总线，所以基于这一背景，`Linux`发明了`platform`总线

## Platform总线的数据结构

### platform_bus_type 总线注册

设备和驱动需要挂载在总线上，要先指明设备和驱动是属于哪条总线的，所以设备与驱动需要注册，而总线在`Linux`系统中也是属于设备，所以总线也要注册，同时因为设备和驱动要挂载在总线上，所以总线要先注册。

文件：driver/base/platform.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3876b208eb37dd105ae09e3f29b4e15c.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/437cca48893e2c7e0e850dc0ef04c147.png)


### platform_device结构体

此结构体通过platform_bus_init函数进行注册。

此结构体通过`platform_add_devices`函数进行注册

目录：`include\linux\platform_device.h`

从中可以看出它是 device 的子类，name 用于驱动邦定，还有需要何种资源。定义 resouce 后，系统启动时完成对硬件资源的规划，硬件不会获得规划以外的资源，减少的冲突。 其中**resource结构体**表明该平台设备所需要的资源（IRQ、内存、DMA 或 IO），具体结构如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/af2d9ec0e6ba7aa4cc0ce485758afefb.png)


arch/arm64/include/asm/device.h

  

### platform_driver结构体

`platform_driver` 除了 `driver` 结构体外，还定义了一些函数，这些函数与结构体`driver_device`中的函数意义一样，只不过它们的输入参数变成了 `platform_device`。具体结构如下所示

文件：`include/linux/device.h`

linux/platform_device.h

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c673a060e68d7d570d483790b97f5f5e.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/82d3cbae5999d91c6e81dd517ab19c44.png)


- `probe`探测函数，如果驱动匹配到了目标设备，总线会自动回调`probe`函数，必须实现。
    
- `remove`释放函数，如果匹配到的设备从总线移除了，总线会自动回调remove函数，必须实现
    
- `device_driver`是`platform_driver`的父类
    
- `id_table`设备信息 ----
    

此结构体使用`platform_driver_register`注册