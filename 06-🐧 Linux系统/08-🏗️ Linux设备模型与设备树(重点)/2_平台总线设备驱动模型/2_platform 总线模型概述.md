
在前面我们学习了Linux设备驱动模型的基础概念，了解了总线、设备、驱动、类这四大核心组件。今天我们深入学习**Platform总线**，这是嵌入式Linux系统中最重要的虚拟总线。

简单来说，**Platform总线是Linux为那些不属于传统总线的设备专门创建的虚拟总线**，它让简单的GPIO设备也能享受到现代设备模型的所有优势。

> [!note]+ 为什么需要Platform总线？ 
> 在嵌入式系统中，很多设备（如LED、按键、GPIO控制器）直接连接到CPU，没有特殊的通信时序，也不符合I2C、SPI等标准总线。为了让这些设备也能使用设备驱动模型，Linux发明了Platform总线。

## 1 概念和意义

### 1.1 概念

**Platform总线**是Linux虚构的一条总线，专门用来管理那些直接与CPU相连、不符合常见总线标准的设备控制器。

在Linux系统中，总线分为两种类型：
- **实际存在的总线**：如I2C、SPI、USB等硬件总线，有明确的通信时序和电气特性
- **虚拟存在的总线**：Platform总线，由软件抽象出来的管理机制

![Platform总线概念图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bb0982477f28be70af3049a752e0e46f.png)

换句话说，**Platform总线是为了让那些"无家可归"的简单设备也能有个统一的管理方式**。

### 1.2 意义

在嵌入式SoC系统中，**集成的独立外设控制器**和**挂接在SoC内存空间的外设**无法依附于传统总线，所以需要Platform总线来管理它们

**Platform总线解决的问题**：
- 为简单设备提供统一的管理框架
- 实现设备信息与驱动代码的分离
- 支持设备的热插拔和电源管理
- 提供标准化的设备驱动接口

> [!tip]+ 实际应用场景 
> 比如SoC芯片内部的GPIO控制器、定时器、串口控制器等，它们直接通过内存映射访问，不需要特殊的通信协议，就适合挂载在Platform总线上。

## 2 注册与初始化

### 2.1 platform_bus_type总线结构体

Platform总线在内核中用`platform_bus_type`结构体来描述：

```c
// drivers/base/platform.c
// 是bus_type的一种
struct bus_type platform_bus_type = {
	.name		= "platform",
	.dev_groups	= platform_dev_groups,
	.match		= platform_match,           //核心，匹配函数！！！
	.uevent		= platform_uevent,
	.dma_configure	= platform_dma_configure,
	.pm		= &platform_dev_pm_ops,
};
EXPORT_SYMBOL_GPL(platform_bus_type);
```
- **name**：总线名称为"platform"，对应`/sys/bus/platform`目录
- **match**：最重要的成员！负责判断设备和驱动是否匹配
- **uevent**：处理与Platform总线相关的事件
- **dma_configure**：配置Platform总线上的DMA功能
- **pm**：电源管理相关操作

### 2.2 Platform总线的注册过程

**内核启动时会自动注册Platform总线**，调用流程如下：
```c
start_kernel()                     // 内核入口函数 (init/main.c)
    ↓
driver_init()                      // 驱动子系统初始化
    ↓
buses_init()                       // 总线子系统初始化
    ↓
bus_register(&platform_bus_type)   // 注册平台总线
    ↓
platform_bus_init()               // 平台总线特定初始化
```

**driver_init函数** 调用platform_bus_init
```c
void __init driver_init(void)
{
    /* These are the core pieces */
    devtmpfs_init();
    devices_init();
    buses_init();           // 初始化总线子系统
    classes_init();
    firmware_init();
    hypervisor_init();
    
    /* These are also core pieces, but must come after the
     * core core pieces.
     */
    platform_bus_init();   // 初始化平台总线
}
```

**platform_bus_init函数**是总线注册的核心：
```c
int __init platform_bus_init(void)
{ 
    int error;
    early_platform_cleanup();         // 提前清理平台总线相关资源
    error = device_register(&platform_bus);  // 注册平台总线设备(总线也是设备的一种)
    if (error) {
        put_device(&platform_bus);     // 注册失败，释放平台总线设备
        return error;
    } 
    error = bus_register(&platform_bus_type);  // 注册平台总线类型
    if (error) {
        device_unregister(&platform_bus);  // 注册失败，注销平台总线设备
        return error;
    } 
    of_platform_register_reconfig_notifier();  // 注册平台重新配置的通知器
    return error;
}
```

**注册步骤解析**：
1. **清理资源**：清空early_platform_device_list上的所有节点
2. **注册总线设备**：调用`device_register()`将platform_bus注册到**设备子系统**
3. **注册总线类型**：调用`bus_register()`将platform_bus_type注册到**总线子系统**
4. **注册通知器**：为设备树重新配置注册通知机制

> [!warning]+ 注意 
> Platform总线的注册是在内核启动时自动完成的，开发者通常不需要手动注册总线，只需要注册设备和驱动即可

**注册成功后的系统变化**：
- 在`/sys/devices`目录下创建`/sys/devices/platform`目录
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bfa1cd9e11dff05d9bcb5a84649d499a.png)
- 在`/sys/bus`目录下创建`/sys/bus/platform`目录
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/47e09acf69541aa5f6c850b84ca3fe7b.png)
- platform目录成为所有Platform设备的父目录

## 3 设备与驱动的数据结构

### 3.1 platform_device结构体详解

**platform_device**是Platform总线上设备的标准描述，它**继承了device结构体**：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/66a874a4755e1d55aefb176d551f75e7.png)

```c
// include/linux/platform_device.h
struct platform_device {
    const char *name;           // 设备名称，用于匹配
    int id;                     // 设备ID，区分同名设备
    struct device dev;          // 继承device结构体！！
    u32 num_resources;          // 资源数量
    struct resource *resource;  // 资源数组指针
    const struct platform_device_id *id_entry;  // 匹配成功后的ID条目
    /* 省略部分成员 */
};
```
- **name**：设备名称，总线匹配的重要依据
- **id**：设备编号，支持多个同名设备共存
- **dev**：继承Linux设备模型，获得完整的设备管理功能
- **resource**：硬件资源信息，如寄存器地址、中断号等

### 3.2 platform_driver结构体详解

**platform_driver**是Platform总线上驱动的标准描述：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/886601edb0bdc3082f3e9457eadbb115.png)

```c
// include/linux/platform_device.h
struct platform_driver {
    int (*probe)(struct platform_device *);     // 设备初始化函数
    int (*remove)(struct platform_device *);    // 设备移除函数
    struct device_driver driver;                // 继承driver结构体
    const struct platform_device_id *id_table;  // 支持的设备列表
    /* 省略部分成员 */
};
```
- **probe**：最重要的函数！设备与驱动匹配成功后调用
- **remove**：设备移除时调用，做清理工作
- **driver**：继承设备模型的驱动结构体
- **id_table**：兼容设备列表，实现一驱动支持多设备

> [!important]+ 关键理解 
> probe函数是Platform驱动的核心，它类似于普通程序的main函数，是驱动开始工作的入口点。

### 3.3 resource结构体-硬件资源的描述

**resource结构体**用于描述设备的硬件资源：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7629b2d0f278b0c294d039461007d776.png)


```c
struct resource {
    resource_size_t start;      // 资源起始地址
    resource_size_t end;        // 资源结束地址
    const char *name;           // 资源名称
    unsigned long flags;        // 资源类型标志
    /* 省略部分成员 */
};
```

**常用资源类型标志 flags**：

|资源宏定义|描述|
|---|---|
|IORESOURCE_MEM|内存资源，如寄存器地址|
|IORESOURCE_IO|IO端口资源|
|IORESOURCE_IRQ|中断资源|
|IORESOURCE_DMA|DMA通道资源|

> [!example]+ 实际应用举例 
> 对于一个GPIO控制器，可能需要这些资源：
> 
> - IORESOURCE_MEM：GPIO寄存器基地址
> - IORESOURCE_IRQ：GPIO中断号
> - 这些信息都通过resource结构体数组来描述


## 4 Platform设备和驱动的注册函数

### 4.1 Platform设备注册

**注册函数**：
```c
// linux/platform_device.h
int platform_device_register(struct platform_device *pdev);
void platform_device_unregister(struct platform_device *pdev);
```

**platform_device_register内部实现**：

![platform_device_register函数](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e2338d7854b6bb8a6cb6490e0ce21b15.png)

```c
int platform_device_register(struct platform_device *pdev)
{ 
    device_initialize(&pdev->dev);    // 初始化设备结构体
    arch_setup_pdev_archdata(pdev);   // 设置架构相关数据
    return platform_device_add(pdev); // 添加设备到系统
}
```

**platform_device_add函数**的关键步骤：

```c
int platform_device_add(struct platform_device *pdev)
{
    // 设置设备的父设备为platform_bus
    if (!pdev->dev.parent)
        pdev->dev.parent = &platform_bus;
    
    // 设置设备的总线为platform_bus_type
    pdev->dev.bus = &platform_bus_type;
    
    // 根据设备ID设置设备名称
    switch (pdev->id) {
    default:
        dev_set_name(&pdev->dev, "%s.%d", pdev->name, pdev->id);
        break;
    case PLATFORM_DEVID_NONE:
        dev_set_name(&pdev->dev, "%s", pdev->name);
        break;
    case PLATFORM_DEVID_AUTO:
        // 自动分配ID
        ret = ida_simple_get(&platform_devid_ida, 0, 0, GFP_KERNEL);
        pdev->id = ret;
        dev_set_name(&pdev->dev, "%s.%d.auto", pdev->name, pdev->id);
        break;
    }
    
    // 处理设备资源
    for (i = 0; i < pdev->num_resources; i++) {
        // 设置资源名称和父资源
        // 将资源插入到资源树中
    }
    
    // 注册设备到设备子系统
    ret = device_add(&pdev->dev);
    return ret;
}
```

> [!note]+ 注册成功的标志 
> 设备注册成功后，会在`/sys/bus/platform/devices/`目录下看到对应的设备文件。

### 4.2 Platform驱动注册

**注册函数**：
```c
#define platform_driver_register(drv) \
    __platform_driver_register(drv, THIS_MODULE)

void platform_driver_unregister(struct platform_driver *drv);
```

**__platform_driver_register内部实现**：

![__platform_driver_register函数](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f898d88d54d7a57a5f29c3ef3b5c5ca2.png)

```c
int __platform_driver_register(struct platform_driver *drv, struct module *owner)
{
    drv->driver.owner = owner;              // 设置模块拥有者
    drv->driver.bus = &platform_bus_type;   // 设置总线类型
    drv->driver.probe = platform_drv_probe; // 设置探测函数
    drv->driver.remove = platform_drv_remove; // 设置移除函数
    drv->driver.shutdown = platform_drv_shutdown; // 设置关机函数
    
    return driver_register(&drv->driver);   // 注册到驱动子系统
}
```

**platform_drv_probe函数**是关键的包装函数：

```c
static int platform_drv_probe(struct device *_dev)
{ 
    struct platform_driver *drv = to_platform_driver(_dev->driver);
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
    
    // 调用驱动程序的探测函数（重点！）
    if (drv->probe) {
        ret = drv->probe(dev);
        if (ret)
            dev_pm_domain_detach(_dev, true);
    } 
    
    return ret;
}
```

> [!important]+ 关键理解 
> 当我们调用platform_driver_register时，内核会自动设置各种回调函数，最终在匹配成功时调用我们自定义的probe函数。

## 5 总结

**Platform总线模型的核心价值**：

- **统一管理**：为简单设备提供标准化管理框架
- **分离关注点**：硬件信息与驱动逻辑完全分离
- **灵活匹配**：多种匹配方式满足不同场景需求
- **资源管理**：自动化的设备资源分配和释放

**Platform总线的工作流程**：

1. **总线注册**：内核启动时自动注册Platform总线
2. **设备注册**：定义platform_device并注册到总线
3. **驱动注册**：定义platform_driver并注册到总线
4. **自动匹配**：总线调用match函数进行匹配
5. **调用probe**：匹配成功后自动调用驱动的probe函数
6. **设备工作**：驱动获取资源，初始化设备，开始提供服务

**Platform总线是嵌入式Linux驱动开发的基础**，掌握了Platform总线模型，就掌握了现代Linux驱动开发的核心思想。

接下来我们将学习Platform设备的注册与管理，了解如何具体实现设备信息的描述和注册。

---

_建议存放目录：`Linux驱动开发/设备模型基础/03_Platform总线模型详解`_