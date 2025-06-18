## platform 总线注册流程

内核在初始化的过程中调用 platform_bus_init()函数来初始化平台总线， 调用流程如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fbea77d84e2d41deb270b5940cb0e784.png)

我们直接分析 platform_bus_init 函数。在内核 driver/base/platform.c 文件中注册了 platform文件， platform_bus_init 函数如下所示：

```C
int __init platform_bus_init(void)
{ 
    int error;
    early_platform_cleanup(); // 提前清理平台总线相关资源
    error = device_register(&platform_bus); // 注册平台总线设备
    if (error) {
        put_device(&platform_bus); // 注册失败， 释放平台总线设备
        return error; // 返回错误代码
    } 
    error = bus_register(&platform_bus_type); // 注册平台总线类型
    if (error) {
        device_unregister(&platform_bus); // 注册失败， 注销平台总线设备
        return error; // 返回错误代码
    } 
    of_platform_register_reconfig_notifier(); // 注册平台重新配置的通知器
    return error; // 返回错误代码（如果有）
}
```

函数首先清空总线 early_platform_device_list 上的所有节点。 然后使用调用device_register(platform_bus_type) 注册平台总线设备， 将 platform_bus 结构体注册到设备子系统中。然后使用 bus_register(&platform_bus_type)函数注册平台总线类型， 将 platform_bus_type结构体注册到总线子系统中。

  

在上面的函数中使用 bus_register**(&**platform_bus_type**)**注册了平台总线。platform_bus_type结构体如下所示：

```C
struct bus_type platform_bus_type = {
    .name = "platform",
    .dev_groups = platform_dev_groups,
    .match = platform_match,
    .uevent = platform_uevent,
    .dma_configure = platform_dma_configure,
    .pm = &platform_dev_pm_ops,
};
```

述代码定义了一个名为 platform_bus_type 的 struct bus_type 结构体变量， 表示平台总线类型。

该结构体变量的成员包括：

- .name = "platform"： 指定平台总线类型的名称为"platform"。
    
- .dev_groups = platform_dev_groups： 指定设备组的指针， 用于定义与平台总线相关的设备属性组。
    
- .match = platform_match： 指定匹配函数的指针， 用于确定设备是否与平台总线兼容
    
- .uevent = platform_uevent： 指定事件处理函数的指针， 用于处理与平台总线相关的事件。
    
- .dma_configure = platform_dma_configure： 指定 DMA 配置函数的指针， 用于配置平台总线上的 DMA。
    
- .pm = &platform_dev_pm_ops： 指定与电源管理相关的操作函数的指针， 用于管理平台总线上的设备电源
    

## platform 总线注册分析


在 Linux 内核中， platform 总线是一种特殊的总线类型， 用于管理与硬件平台紧密相关的设备。 它提供了一种机制， 使得与特定硬件平台相关的设备能够在系统中得到正确的识别和初始化。

先调用 device_register 函数注册 paltform_bus 这个设备， 会在/sys/devices 目录下创建目录/sys/devices/platform， 此目录所有 platform 设备的父目录， 即所有 platform_device 设备都会在/sys/devices/platform 下创建子目录， 如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a148b0cdb3d5fba0edff4a7ad79fcb08.png)


创建好 platform bus 设备之后， 使用 platform_device_add 函数将 platform_device 结构体添加到 platform 总线中进行注册， 代码实现如下所示：

```C
int platform_device_add(struct platform_device *pdev)
{
        u32 i;
        int ret;
        // 检查输入的平台设备指针是否为空
        if (!pdev)
                return -EINVAL;
        // 如果平台设备的父设备为空， 将父设备设置为 platform_bus
        if (!pdev->dev.parent)
                pdev->dev.parent = &platform_bus;
```