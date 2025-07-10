
在前面我们学习了Platform设备的注册与管理，了解了如何用标准的数据结构来描述硬件信息。今天我们学习**Platform驱动的注册与管理**，这是实现设备控制逻辑的关键环节。

简单来说，**Platform驱动就是设备的"操作手册"**，它告诉内核如何初始化设备、如何控制设备、如何清理设备。

> [!note]+ 核心理念 
> Platform驱动的作用是"控制硬件"：获取Platform设备提供的硬件资源，然后实现具体的设备控制逻辑，为上层应用提供标准的操作接口。

## 1 Platform驱动的数据结构

### 1.1 platform_driver结构体详解

**platform_driver**是描述Platform驱动的核心数据结构：
```c
struct platform_driver {
    int (*probe)(struct platform_device *);        // 设备探测函数
    int (*remove)(struct platform_device *);       // 设备移除函数
    void (*shutdown)(struct platform_device *);    // 设备关机函数
    int (*suspend)(struct platform_device *, pm_message_t state);  // 挂起函数
    int (*resume)(struct platform_device *);       // 恢复函数
    struct device_driver driver;                   // 继承的驱动结构体
    const struct platform_device_id *id_table;     // 支持的设备列表
    bool prevent_deferred_probe;                   // 是否禁用延迟探测
};
```
- **probe**：最重要！设备匹配成功后调用，必须实现
- **remove**：设备移除时调用，必须实现
- **shutdown**：系统关机时调用，可选
- **suspend/resume**：电源管理函数，可选

- **driver（设备驱动结构体）**：继承device_driver结构体，内部的name成员用于匹配，提供驱动管理功能
- **id_table（设备兼容列表）**：列出驱动支持的所有设备，可以用于匹配，优先级高于上面
- **prevent_deferred_probe**：布尔值，控制是否允许延迟探测

> [!example]+ id_table实际应用 一个串口驱动可能支持多种串口芯片：
> 
> ```c
> static const struct platform_device_id uart_id_table[] = {
>     { "uart-16550", (unsigned long)&uart16550_data },
>     { "uart-8250",  (unsigned long)&uart8250_data },
>     { }  // 结束标记
> };
> ```

### 1.2 device_driver结构体核心成员（回顾）

**device_driver**是platform_driver继承的基础结构体：
```c
struct device_driver {
    const char *name;                               // 驱动名称
    struct bus_type *bus;                           // 所属总线
    struct module *owner;                           // 驱动拥有者
    const struct of_device_id *of_match_table;      // 设备树匹配表
    int (*probe)(struct device *dev);               // 通用探测函数
    int (*remove)(struct device *dev);              // 通用移除函数
    /* 省略部分成员 */
};
```
- **name**：驱动名称，用于与platform_device.name进行匹配
- **of_match_table**：设备树匹配表，支持设备树匹配方式
- **probe/remove**：通用的探测和移除函数（被platform_drv_probe包装）

## 2 Platform驱动的注册接口

### 2.1 platform_driver_register注册函数

**platform_driver_register**是Platform驱动注册的标准接口，就像给驱动程序"办理营业执照"。
```c
// 内核源码/include/linux/platform_device.h
#include <linux/platform_device.h>
int platform_driver_register(struct platform_driver *driver);
```
- **driver**：指向platform_driver结构体的指针，描述驱动的属性和回调函数
- **返回值**：成功返回0，失败返回负数错误码

换句话说，这个函数就是驱动向内核"注册服务"的过程，**告诉内核自己能支持哪些设备、提供什么功能**。

### 2.2 platform_driver_register注销函数

驱动使用完毕后，需要正确注销：
```c
void platform_driver_unregister(struct platform_driver *drv);
```
1. **解除设备绑定**：调用所有已绑定设备的remove函数
2. **从总线移除**：将驱动从Platform总线的驱动列表中移除
3. **清理资源**：释放驱动占用的系统资源
4. **更新文件系统**：删除/sys目录下的相关文件

> [!warning]+ 注意事项 
> 驱动注销时，必须确保所有设备都已正确移除，否则可能导致系统不稳定。建议在模块卸载前检查是否还有设备在使用该驱动。


### 2.3 platform_driver_register宏定义

实际上，**platform_driver_register**是一个宏定义：
```c
#define platform_driver_register(drv) \
    __platform_driver_register(drv, THIS_MODULE)

extern int __platform_driver_register(struct platform_driver *,
					struct module *);
```
- 简化驱动注册过程
- 自动将当前模块（THIS_MODULE）与驱动关联
- 确保模块的生命周期与驱动注册状态同步

> [!tip]+ 理解要点 
> 使用宏的好处是自动处理模块管理细节，开发者只需要关注驱动的核心逻辑，不用担心模块引用计数等复杂问题。

### 2.4 `__platform_driver_register`内部实现

Platform驱动注册的真正实现：
```c
// 内核源码/drivers/base/platform.c
int __platform_driver_register(struct platform_driver *drv, struct module *owner)
{
    drv->driver.owner = owner;                      // 设置模块拥有者
    drv->driver.bus = &platform_bus_type;           // 指定总线类型
    drv->driver.probe = platform_drv_probe;         // 设置探测函数
    drv->driver.remove = platform_drv_remove;       // 设置移除函数
    drv->driver.shutdown = platform_drv_shutdown;   // 设置关机函数
    
    return driver_register(&drv->driver);           // 注册到驱动子系统
}
```
1. **设置模块拥有者**：建立驱动和模块的关联关系，确保模块在驱动使用期间不能被卸载
2. **指定总线类型**：告诉内核这个驱动输入Platform总线，只会与Platform设备匹配
3. **设置回调函数**：将platform_driver的回调函数和device_driver的回调函数联系起来
```c
设备匹配成功时：
    ↓
struct driver.probe() 被调用
    ↓
实际调用 platform_drv_probe()  // 通用包装函数
    ↓
platform_drv_probe() 内部调用
    ↓
struct platform_driver.probe()  // 具体驱动的probe函数
```
4. **注册到驱动子系统**：将驱动添加到Platform总线的驱动列表，并触发触发总线的匹配机制

> [!important]+ 关键理解 
> 内核会自动设置标准的回调函数（platform_drv_probe等），这些函数会在合适的时机调用我们自定义的probe、remove函数。这是一种"包装器"设计模式。

### 2.5 platform_drv_probe包装函数

**platform_drv_probe**是最重要的包装函数，它在设备与驱动匹配成功后被调用：
```c
static int platform_drv_probe(struct device *_dev)
{ 
    // 获取platform_driver和platform_device指针
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
    
    // 调用驱动程序的探测函数（核心！）
    if (drv->probe) {
        ret = drv->probe(dev);              // 这里调用我们自定义的probe函数
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

> [!tip]+ 理解要点 
> platform_drv_probe函数是一个"适配器"，它把通用的设备模型接口转换为Platform特有的接口，让我们的probe函数能够直接操作platform_device

### 2.6 自定义probe函数的实现

**probe函数**是Platform驱动的核心，类似于应用程序的main函数：
```c
static int my_driver_probe(struct platform_device *pdev)
{
    struct resource *res;
    void __iomem *base;
    int irq;
    int ret;
    
    printk(KERN_INFO "Device %s matched with driver\n", pdev->name);
    
    // 1. 获取设备资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res) {
        dev_err(&pdev->dev, "No memory resource\n");
        return -ENODEV;
    }
    
    // 2. 映射寄存器地址
    base = devm_ioremap_resource(&pdev->dev, res);
    if (IS_ERR(base)) {
        dev_err(&pdev->dev, "Cannot map memory\n");
        return PTR_ERR(base);
    }
    
    // 3. 获取中断资源
    irq = platform_get_irq(pdev, 0);
    if (irq < 0) {
        dev_err(&pdev->dev, "No IRQ resource\n");
        return irq;
    }
    
    // 4. 初始化硬件
    // 这里进行具体的硬件初始化操作
    
    // 5. 注册字符设备或其他上层接口
    // 这里注册对应的字符设备等
    
    // 6. 保存驱动私有数据
    platform_set_drvdata(pdev, driver_private_data);
    
    dev_info(&pdev->dev, "Driver probe successfully\n");
    return 0;
}
```

**probe函数的标准工作流程**：
1. **获取设备资源**：从platform_device中提取硬件信息
2. **地址映射**：将物理地址映射为虚拟地址
3. **初始化硬件**：配置寄存器，设置硬件工作状态
4. **注册上层接口**：创建字符设备、网络设备等用户接口
5. **保存私有数据**：将驱动数据与设备关联

### 2.7 remove函数的实现

**remove函数**负责清理probe函数中分配的资源：
```c
static int my_driver_remove(struct platform_device *pdev)
{
    // 获取保存的私有数据
    struct my_driver_data *data = platform_get_drvdata(pdev);
    
    printk(KERN_INFO "Removing device %s\n", pdev->name);
    
    // 1. 注销上层接口
    // 注销字符设备等
    
    // 2. 释放中断
    // free_irq()等
    
    // 3. 取消地址映射
    // iounmap()等（如果使用devm_xxx函数，会自动释放）
    
    // 4. 清理私有数据
    // kfree()等（如果使用devm_xxx函数，会自动释放）
    
    dev_info(&pdev->dev, "Driver removed successfully\n");
    return 0;
}
```

> [!warning]+ 重要注意 
> **remove函数必须与probe函数对应**，probe中分配的每一项资源都应该在remove中释放。建议使用devm_xxx系列函数，它们会在设备移除时自动释放资源。

## 3 Platform驱动定义实例

### 3.1 完整的驱动定义示例

让我们通过一个LED驱动的完整例子来理解Platform驱动的定义：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/io.h>
#include <linux/cdev.h>
#include <linux/fs.h>

// 驱动私有数据结构
struct led_driver_data {
    void __iomem *gpio_base;        // GPIO寄存器基地址
    unsigned int gpio_pin;          // GPIO引脚号
    struct cdev cdev;               // 字符设备
    dev_t devno;                    // 设备号
};

// 字符设备操作函数（省略具体实现）
static int led_open(struct inode *inode, struct file *filp) { /* ... */ }
static int led_release(struct inode *inode, struct file *filp) { /* ... */ }
static ssize_t led_write(struct file *filp, const char __user *buf, size_t count, loff_t *ppos) { /* ... */ }

static struct file_operations led_fops = {
    .owner = THIS_MODULE,
    .open = led_open,
    .release = led_release,
    .write = led_write,
};

// probe函数实现
static int led_driver_probe(struct platform_device *pdev)
{
    struct led_driver_data *led_data;
    struct resource *res;
    int ret;
    
    dev_info(&pdev->dev, "LED driver probe started\n");
    
    // 分配驱动私有数据
    led_data = devm_kzalloc(&pdev->dev, sizeof(*led_data), GFP_KERNEL);
    if (!led_data)
        return -ENOMEM;
    
    // 获取GPIO寄存器资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res) {
        dev_err(&pdev->dev, "No memory resource\n");
        return -ENODEV;
    }
    
    // 映射寄存器地址
    led_data->gpio_base = devm_ioremap_resource(&pdev->dev, res);
    if (IS_ERR(led_data->gpio_base)) {
        dev_err(&pdev->dev, "Cannot map GPIO registers\n");
        return PTR_ERR(led_data->gpio_base);
    }
    
    // 获取GPIO引脚配置
    struct led_platform_data *pdata = dev_get_platdata(&pdev->dev);
    if (pdata) {
        led_data->gpio_pin = pdata->gpio_pin;
    } else {
        led_data->gpio_pin = 7;  // 默认值
    }
    
    // 初始化GPIO（设置为输出模式）
    // 这里进行具体的GPIO初始化
    
    // 分配设备号
    ret = alloc_chrdev_region(&led_data->devno, 0, 1, "led_device");
    if (ret) {
        dev_err(&pdev->dev, "Cannot allocate device number\n");
        return ret;
    }
    
    // 初始化并注册字符设备
    cdev_init(&led_data->cdev, &led_fops);
    led_data->cdev.owner = THIS_MODULE;
    ret = cdev_add(&led_data->cdev, led_data->devno, 1);
    if (ret) {
        dev_err(&pdev->dev, "Cannot add character device\n");
        unregister_chrdev_region(led_data->devno, 1);
        return ret;
    }
    
    // 保存私有数据
    platform_set_drvdata(pdev, led_data);
    
    dev_info(&pdev->dev, "LED driver probe completed successfully\n");
    return 0;
}

// remove函数实现
static int led_driver_remove(struct platform_device *pdev)
{
    struct led_driver_data *led_data = platform_get_drvdata(pdev);
    
    dev_info(&pdev->dev, "LED driver remove started\n");
    
    // 注销字符设备
    cdev_del(&led_data->cdev);
    unregister_chrdev_region(led_data->devno, 1);
    
    // devm_xxx函数分配的资源会自动释放
    
    dev_info(&pdev->dev, "LED driver removed successfully\n");
    return 0;
}

// 设备兼容列表
static const struct platform_device_id led_id_table[] = {
    { "led_platform", 0 },      // 支持名为led_platform的设备
    { }                          // 结束标记
};
MODULE_DEVICE_TABLE(platform, led_id_table);

// 设备树匹配表（可选）
static const struct of_device_id led_of_match[] = {
    { .compatible = "my-company,led-gpio" },
    { }
};
MODULE_DEVICE_TABLE(of, led_of_match);

// Platform驱动结构体
static struct platform_driver led_platform_driver = {
    .probe = led_driver_probe,
    .remove = led_driver_remove,
    .driver = {
        .name = "led_platform",           // 驱动名称
        .owner = THIS_MODULE,
        .of_match_table = led_of_match,   // 设备树匹配表
    },
    .id_table = led_id_table,             // 设备兼容列表
};

// 模块初始化函数
static int __init led_driver_init(void)
{
    int ret;
    
    pr_info("LED platform driver init\n");
    
    // 注册Platform驱动
    ret = platform_driver_register(&led_platform_driver);
    if (ret) {
        pr_err("Failed to register LED platform driver: %d\n", ret);
        return ret;
    }
    
    pr_info("LED platform driver registered successfully\n");
    return 0;
}

// 模块退出函数
static void __exit led_driver_exit(void)
{
    pr_info("LED platform driver exit\n");
    
    // 注销Platform驱动
    platform_driver_unregister(&led_platform_driver);
    
    pr_info("LED platform driver unregistered\n");
}

module_init(led_driver_init);
module_exit(led_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("LED Platform Driver Example");
```

### 3.2 驱动注册后的系统变化

**1. 文件系统中的体现**
- `/sys/bus/platform/drivers/led_platform/` - 驱动目录
- 如果有匹配的设备，会在该目录下看到设备链接

**2. 匹配机制触发**
- 驱动注册后，Platform总线立即检查所有已注册的设备
- 如果找到匹配的设备，立即调用probe函数
- 如果暂时没有匹配设备，等待后续设备注册

**3. 设备文件创建**
- probe函数成功执行后，会创建`/dev/led_device`等设备文件
- 用户程序可以通过这些文件操作硬件

> [!tip]+ 调试技巧 可以通过以下方式检查驱动状态：
> 
> - `cat /sys/bus/platform/drivers/led_platform/uevent` - 查看驱动事件
> - `ls /sys/bus/platform/drivers/led_platform/` - 查看绑定的设备
> - `dmesg | grep led` - 查看相关内核日志


## 4 总结

**Platform驱动是设备控制的核心实现**，通过标准化的驱动框架，我们可以专注于硬件控制逻辑，而不用担心底层的设备管理细节。

接下来我们将学习设备与驱动的匹配机制，深入了解Platform总线如何智能地为设备和驱动配对。
