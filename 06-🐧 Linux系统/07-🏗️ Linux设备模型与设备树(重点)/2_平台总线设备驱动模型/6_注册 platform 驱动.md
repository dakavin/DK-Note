
## 注册 platform 驱动

**platform_driver_register** 函数用于在 Linux 内核中注册一个平台驱动程序。 下面是对该函数的详细介绍：

**函数原型：**

int platform_driver_register(struct platform_driver *driver);

**头文件：**

#include <linux/platform_device.h>

**函数作用：**

platform_driver_register 函数用于将一个平台驱动程序注册到内核中。 通过注册平台驱动程序， 内核可以识别并与特定的平台设备进行匹配， 并在需要时调用相应的回调函数。

**参数含义：**

driver： 指向 struct platform_driver 结构体的指针， 描述了要注册的平台驱动程序的属性和

回调函数 。

**返回值：**

返回一个整数值， 表示函数的执行状态。 如果注册成功， 返回 0； 如果注册失败， 返回一个负数错误码。

该函数在内核源码目录下的“/include/linux/platform_device.h” 文件中， 具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1c3ea647be1445c9cb1acfdff4fd6fd5.png)


这个宏用于简化平台驱动程序的注册过程。 它将实际的注册函数 __platform_driver_regist er 与当前模块（驱动程序） 关联起来。 宏的参数 drv 是一个指向 struct platform_driver 结构体的指针， 描述了要注册的平台驱动程序的属性和回调函数。 THIS_MODULE 是一个宏， 用于获取当前模块的指针。

而__platform_driver_register 实际定义在“/drivers/base/platform.c” 文件中， 相关定义如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f898d88d54d7a57a5f29c3ef3b5c5ca2.png)


driver_register(最後)

- 第3 行： 将指向当前模块的指针 owner 赋值给平台驱动程序的 owner 成员。 这样做是为了将当前模块与平台驱动程序关联起来， 以确保模块的生命周期和驱动程序的注册和注销相关联。
    
- 第4行： 将指向平台总线类型的指针 &platform_bus_type 赋值给平台驱动程序的 bus 成员。这样做是为了指定该驱动程序所属的总线类型为平台总线， 以便内核能够将平台设备与正确的驱动程序进行匹配。
    
- 第 5 行： 将指向平台驱动程序探测函数 platform_drv_probe 的指针赋值给平台驱动程序的 probe 成员。 这样做是为了指定当内核发现与驱动程序匹配的平台设备时， 要调用的驱动程序探测函数。
    
- 第 6 行： 将指向平台驱动程序移除函数 platform_drv_remove 的指针赋值给平台驱动程序的 remove 成员。 这样做是为了指定当内核需要从系统中移除与驱动程序匹配的平台设备时，要调用的驱动程序移除函数。
    
- 第 7 行 = platform_drv_shutdown;： 将指向平台驱动程序关机函数 platform_drv_shutdow n 的指针赋值给平台驱动程序的 shutdown 成员。 这样做是为了指定当系统关机时， 要调用的驱动程序关机函数。
    
- 第 9 行： 调用 driver_register 函数， 将平台驱动程序的 driver 成员注册到内核中。 该函数负责将驱动程序注册到相应的总线上， 并在注册成功时返回 0， 注册失败时返回一个负数错误码。
    

通过这些操作， `__platform_driver_register` 函数将平台驱动程序与内核关联起来， 并确保内核能够正确识别和调用驱动程序的各种回调函数， 以实现与平台设备的交互和管理。 函数的返回值表示注册过程的执行状态， 以便在需要时进行错误处理。

  

## probe函数

重点看看 platform 总线的 probe 函数是如何执行的：

```C
static int platform_drv_probe(struct device *_dev)
{ 
    // 将传递给驱动程序的设备指针转换为 platform_driver 结构体指针
    struct platform_driver *drv = to_platform_driver(_dev->driver);
    // 将传递给驱动程序的设备指针转换为 platform_device 结构体指针
    struct platform_device *dev = to_platform_device(_dev);
    int ret;
    // 设置设备节点的默认时钟属性
    ret = of_clk_set_defaults(_dev->of_node, false);
    if (ret < 0)
        return ret;
    // 将设备附加到电源域
    ret = dev_pm_domain_attach(_dev, true);
    if (ret)
        goto out;
    // 调用驱动程序的探测函数（probe）
    if (drv->probe) {
        ret = drv->probe(dev);
        if (ret)
            dev_pm_domain_detach(_dev, true);
    } 
    out:
    // 处理探测延迟和错误情况
    if (drv->prevent_deferred_probe && ret == -EPROBE_DEFER) {
        dev_warn(_dev, "probe deferral not supported\n");
        ret = -ENXIO;
    } 
return ret;
}
```

该函数的主要逻辑如下：

1. 首先， 将传递给驱动程序的设备指针 _dev 转换为 platform_driver 结构体指针 drv， 将传递给驱动程序的设备指针 _dev 转换为 platform_device 结构体指针 dev。
    
2. 使用 of_clk_set_defaults() 函数设置设备节点的默认时钟属性。 这个函数会根据设备节点的属性信息配置设备的时钟。
    
3. 调用 dev_pm_domain_attach() 将设备附加到电源域。 这个函数会根据设备的电源管理需求，将设备与相应的电源域进行关联。
    
4. 如果驱动程序的 **probe** 函数存在， 调用它来执行设备的探测操作。 **drv->probe(dev)** 表示调用驱动程序的 **probe** 函数， 并传递 **platform_device** 结构体指针 **dev** 作为参数。 如果探测失败， 会调用 **dev_pm_domain_detach()** 分离设备的电源域。
    
5. 处理探测延迟和错误情况。 如果驱动程序设置了 prevent_deferred_probe 标志， 并且返回值为 -EPROBE_DEFER ， 则 表 示 探 测 被 延 迟 。 在 这 种 情 况 下 如 果 驱 动 程 序 设 置 了prevent_deferred_probe 标志， 并且返回值为 -EPROBE_DEFER， 则表示探测被延迟。 在这种情况下， 代码会打印一个警告信息 probe deferral not supported， 并将返回值设置为 -ENXIO， 表示设备不存在。
    

总体而言， 该函数的作用是执行平台驱动程序的探测操作， 在设备上调用驱动程序的probe 函数， 并处理探测延迟和错误情况。

## platform_driver 结构体

platform_driver 结构体是 Linux 内核中用于编写平台设备驱动程序的重要数据结构。 它提供了与平台设备驱动相关的函数和数据成员， 以便与平台设备进行交互和管理。 该结构体定义在内核的“ /include/linux/platform_device.h” 文件中， 具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8a5ef748cf9b7347f781c0829fdecde5.png)


- probe： 平台设备的探测函数指针。 当系统检测到一个平台设备与该驱动程序匹配时， 该函数将被调用以初始化和配置设备。
    
- remove： 平台设备的移除函数指针。 当平台设备从系统中移除时， 该函数将被调用以执行清理和释放资源的操作。
    
- shutdown： 平台设备的关闭函数指针。 当系统关闭时， 该函数将被调用以执行与平台设备相关的关闭操作。
    
- suspend： 平台设备的挂起函数指针。 当系统进入挂起状态时， 该函数将被调用以执行与平台设备相关的挂起操作。
    
- resume： 平台设备的恢复函数指针。 当系统从挂起状态恢复时， 该函数将被调用以执行与平台设备相关的恢复操作。
    
- driver： 包含了与设备驱动程序相关的通用数据， 它是 struct device_driver 类型的实例。 其中包括驱动程序的名称、 总线类型、 模块拥有者、 属性组数组指针等信息， **该结构体的 name参数需要与 platform_device 的.name 参数相同才能匹配成功， 从而进入 probe 函数。**
    
- id_table： 指向 struct platform_device_id 结构体数组的指针， 用于匹配平台设备和驱动程序之间的关联关系。 通过该关联关系， 可以确定哪个平台设备与该驱动程序匹配， 和.driver.name起到相同的作用， 但是优先级高于.driver.name。
    
- prevent_deferred_probe： 一个布尔值， 用于确定是否阻止延迟探测。 如果设置为 true， 则延迟探测将被禁用。
    

  

使用 struct platform_driver 结构体， 开发人员可以定义平台设备驱动程序， 并将其注册到内核中。 当系统检测到与该驱动程序匹配的平台设备时， 内核将调用相应的函数来执行设备的初始化、 配置、 操作和管理。 驱动程序可以利用提供的函数指针和通用数据与平台设备进行交互， 并提供必要的功能和服务。

  

需要注意的是， struct platform_driver 结构体继承了 struct device_driver 结构体， 因此可以直接访问 struct device_driver 中定义的成员。 这使得平台驱动程序可以利用通用的驱动程序机制， 并与其他类型的设备驱动程序共享代码和功能。