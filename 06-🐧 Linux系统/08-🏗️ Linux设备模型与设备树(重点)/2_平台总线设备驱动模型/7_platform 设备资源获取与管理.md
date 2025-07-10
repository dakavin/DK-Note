
在前面我们学习了设备与驱动的匹配机制，了解了Platform总线如何智能地为设备和驱动配对。今天我们学习**Platform设备资源获取与管理**，这是驱动程序获取硬件信息并进行实际控制的关键环节。

简单来说，**资源获取就是驱动程序从设备中"提取"硬件信息的过程**，包括寄存器地址、中断号、GPIO配置等，然后将这些信息转化为驱动可以直接使用的形式。

> [!note]+ 核心理念 资源获取的本质是"硬件信息的标准化访问"：将设备中描述的原始硬件信息，转换为驱动程序可以直接使用的标准接口，实现硬件抽象。

## 1 Platform设备资源概述

### 1.1 资源的定义与分类

在Platform设备模型中，**资源（Resource）** 是对硬件信息的标准化描述，主要包括以下几类：

**1. 内存资源（IORESOURCE_MEM）**
- 设备寄存器的物理地址范围
- 设备专用内存区域
- 是最常用的资源类型

**2. 中断资源（IORESOURCE_IRQ）**
- 设备使用的中断号
- 中断触发类型和优先级信息

**3. I/O端口资源（IORESOURCE_IO）**
- 主要用于x86架构的端口映射
- 在嵌入式ARM系统中较少使用

**4. DMA资源（IORESOURCE_DMA）**
- DMA通道号
- DMA传输相关配置

### 1.2 resource结构体回顾

每个资源都用`resource`结构体来描述：
```c
struct resource {
    resource_size_t start;      // 资源起始地址/号码
    resource_size_t end;        // 资源结束地址/号码  
    const char *name;           // 资源名称
    unsigned long flags;        // 资源类型标志
    unsigned long desc;         // 资源描述信息
    struct resource *parent;    // 父资源
    struct resource *sibling;   // 兄弟资源  
    struct resource *child;     // 子资源
};
```

> [!example]+ 资源定义实例 对于一个GPIO控制器，可能定义这些资源：
> 
> ```c
> static struct resource gpio_resources[] = {
>     {
>         .start = 0x50000000,        // GPIO寄存器基地址
>         .end   = 0x50000FFF,        // 地址范围4KB
>         .flags = IORESOURCE_MEM,    // 内存类型资源
>         .name  = "gpio-regs",
>     },
>     {
>         .start = 32,                // 中断号
>         .end   = 32,                // 同一个中断
>         .flags = IORESOURCE_IRQ,    // 中断类型资源
>         .name  = "gpio-irq",
>     },
> };
> ```

## 2 内存资源获取与管理

### 2.1 platform_get_resource函数详解

**platform_get_resource**是获取Platform设备资源的核心函数：
```c
struct resource *platform_get_resource(struct platform_device *pdev,
                                        unsigned int type,
                                        unsigned int num);
```
- **pdev**：要获取资源的Platform设备指针
	- 其他资源可以在 linux/ioport.h 和 linux/irq.h 头文件找到
- **num**：资源索引号（同类型资源可能有多个）
- **返回值**：
	- 成功：指向资源resource结构体指针
	- 失败：返回NULL

### 2.2 platform_get_resource函数实现

让我们看看这个函数的具体实现：
```c
struct resource *platform_get_resource(struct platform_device *dev,
                                        unsigned int type, unsigned int num)
{
    int i;
    
    // 遍历设备的所有资源
    for (i = 0; i < dev->num_resources; i++) {
        struct resource *r = &dev->resource[i];
        
        // 检查资源类型是否匹配，如果匹配再检查索引
        if (type == resource_type(r) && num-- == 0)
            return r;
    }
    return NULL;
}
```

**函数工作原理**：
1. **从头开始搜索**：从设备资源数组的第一个元素开始
2. **类型匹配**：通过`type == resource_type(r)`判断资源类型是否匹配
3. **索引递减**：如果类型匹配，通过`num-- == 0`确定是否是要找的资源
4. **返回结果**：找到匹配的资源就返回，否则继续搜索

> [!tip]+ 理解要点 
> 函数使用了巧妙的`num--`逻辑：只有在类型匹配时才递减num，这样可以正确获取同类型的第num个资源。

### 2.3 内存资源的地址映射

获取到resource结构体后，通常需要将物理地址映射为虚拟地址才能在驱动中使用：

**1. 手动映射方式**
```c
static int my_driver_probe(struct platform_device *pdev)
{
    struct resource *res;
    void __iomem *base;
    
    // 获取内存资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res) {
        dev_err(&pdev->dev, "No memory resource\n");
        return -ENODEV;
    }
    
    // 检查资源大小
    if (resource_size(res) < 0x1000) {
        dev_err(&pdev->dev, "Memory resource too small\n");
        return -EINVAL;
    }
    
    // 映射物理地址到虚拟地址
    base = ioremap(res->start, resource_size(res));
    if (!base) {
        dev_err(&pdev->dev, "Cannot map memory\n");
        return -ENOMEM;
    }
    
    // 使用映射后的地址操作寄存器
    writel(0x12345678, base + 0x10);
    
    // 记住在remove函数中要iounmap(base)
    return 0;
}
```

**2. 推荐的devm方式**
```c
static int my_driver_probe(struct platform_device *pdev)
{
    struct resource *res;
    void __iomem *base;
    
    // 获取内存资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res) {
        dev_err(&pdev->dev, "No memory resource\n");
        return -ENODEV;
    }
    
    // 使用devm_ioremap_resource，自动管理资源
    base = devm_ioremap_resource(&pdev->dev, res);
    if (IS_ERR(base)) {
        dev_err(&pdev->dev, "Cannot map memory\n");
        return PTR_ERR(base);
    }
    
    // 使用映射后的地址操作寄存器
    writel(0x12345678, base + 0x10);
    
    // devm_ioremap_resource会在设备移除时自动iounmap
    return 0;
}
```

> [!warning]+ 重要提醒 
> 推荐使用`devm_ioremap_resource`而不是`ioremap`，因为devm版本会在设备移除时自动释放映射，避免内存泄漏。

### 2.4 of_iomap函数 - 一步到位的映射

对于设备树的设备节点，内核提供了更简便的`of_iomap`函数：
```c
void __iomem *of_iomap(struct device_node *np, int index);
```
- **np**：设备树节点指针
- **index**：reg属性中的索引号

**使用示例**：
```c
static int my_driver_probe(struct platform_device *pdev)
{
    struct device_node *np = pdev->dev.of_node;
    void __iomem *base;
    
    // 直接映射设备树中第0个reg属性
    base = of_iomap(np, 0);
    if (!base) {
        dev_err(&pdev->dev, "Cannot map memory from device tree\n");
        return -ENOMEM;
    }
    
    // 使用映射后的地址
    writel(0x12345678, base + 0x10);
    
    // 记住要iounmap
    return 0;
}
```

### 2.5 直接访问resource数组

除了使用`platform_get_resource`函数，也可以直接访问设备的resource数组：
```c
static int my_driver_probe(struct platform_device *pdev)
{
    void __iomem *base;
    
    // 直接访问第一个资源（通常是内存资源）
    if (pdev->num_resources < 1) {
        dev_err(&pdev->dev, "No resources available\n");
        return -ENODEV;
    }
    
    struct resource *res = &pdev->resource[0];
    if (!(res->flags & IORESOURCE_MEM)) {
        dev_err(&pdev->dev, "First resource is not memory\n");
        return -EINVAL;
    }
    
    // 映射地址
    base = devm_ioremap(&pdev->dev, res->start, resource_size(res));
    if (!base) {
        dev_err(&pdev->dev, "Cannot map memory\n");
        return -ENOMEM;
    }
    
    return 0;
}
```

> [!tip]+ 使用建议 
> 虽然可以直接访问resource数组，但推荐使用`platform_get_resource`函数，因为它提供了类型检查和索引管理，代码更安全可靠。

## 3 中断资源获取与管理

### 3.1 platform_get_irq函数

对于中断资源，内核提供了专门的获取函数：
```c
int platform_get_irq(struct platform_device *pdev, unsigned int num);
```
- **pdev**：Platform设备指针
- **num**：中断资源索引号
- **返回值**：
	- 成功：返回中断号（正数）
	- 失败：返回负数错误码

### 3.2 中断资源使用示例

```c
// 中断处理函数
static irqreturn_t my_interrupt_handler(int irq, void *dev_id)
{
    struct my_device_data *data = dev_id;
    
    // 处理中断
    pr_info("Interrupt %d triggered\n", irq);
    
    // 清除中断标志（根据具体硬件）
    writel(0x1, data->base + IRQ_CLEAR_REG);
    
    return IRQ_HANDLED;
}

static int my_driver_probe(struct platform_device *pdev)
{
    int irq;
    int ret;
    
    // 获取中断号
    irq = platform_get_irq(pdev, 0);
    if (irq < 0) {
        dev_err(&pdev->dev, "No IRQ resource: %d\n", irq);
        return irq;
    }
    
    dev_info(&pdev->dev, "Using IRQ %d\n", irq);
    
    // 注册中断处理函数
    ret = devm_request_irq(&pdev->dev, irq, my_interrupt_handler,
                           IRQF_TRIGGER_RISING, "my-device", driver_data);
    if (ret) {
        dev_err(&pdev->dev, "Cannot request IRQ %d: %d\n", irq, ret);
        return ret;
    }
    
    return 0;
}
```

### 3.3 中断资源的高级用法

**1. 获取多个中断**
```c
static int my_driver_probe(struct platform_device *pdev)
{
    int tx_irq, rx_irq;
    
    // 获取发送中断
    tx_irq = platform_get_irq(pdev, 0);
    if (tx_irq < 0)
        return tx_irq;
    
    // 获取接收中断  
    rx_irq = platform_get_irq(pdev, 1);
    if (rx_irq < 0)
        return rx_irq;
    
    // 分别注册中断处理函数
    devm_request_irq(&pdev->dev, tx_irq, tx_interrupt_handler,
                     0, "my-device-tx", driver_data);
    devm_request_irq(&pdev->dev, rx_irq, rx_interrupt_handler, 
                     0, "my-device-rx", driver_data);
    
    return 0;
}
```

**2. 中断资源的错误处理**
```c
static int my_driver_probe(struct platform_device *pdev)
{
    int irq;
    
    // 获取中断号，处理各种错误情况
    irq = platform_get_irq(pdev, 0);
    if (irq == -EPROBE_DEFER) {
        dev_info(&pdev->dev, "IRQ not ready, deferring probe\n");
        return -EPROBE_DEFER;
    } else if (irq < 0) {
        dev_err(&pdev->dev, "No valid IRQ resource\n");
        return irq;
    }
    
    return 0;
}
```

## 4 资源获取的最佳实践

**1. 资源检查顺序**
```c
static int check_resources(struct platform_device *pdev)
{
    // 首先检查必需的资源
    if (!platform_get_resource(pdev, IORESOURCE_MEM, 0)) {
        dev_err(&pdev->dev, "Missing required memory resource\n");
        return -ENODEV;
    }
    
    // 然后检查可选的资源
    if (platform_get_irq(pdev, 0) < 0) {
        dev_info(&pdev->dev, "No IRQ resource, some features disabled\n");
    }
    
    return 0;
}
```

**2. 使用devm_系列函数**
```c
// 推荐：使用devm_函数，自动管理资源
base = devm_ioremap_resource(&pdev->dev, res);
devm_request_irq(&pdev->dev, irq, handler, flags, name, dev_id);

// 不推荐：手动管理资源
base = ioremap(res->start, resource_size(res));
request_irq(irq, handler, flags, name, dev_id);
// 需要在remove函数中手动iounmap和free_irq
```

**3. 错误处理**
```c
static int my_probe(struct platform_device *pdev)
{
    void __iomem *base;
    int irq, ret;
    
    // 获取并检查内存资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res)
        return -ENODEV;
    
    base = devm_ioremap_resource(&pdev->dev, res);
    if (IS_ERR(base))
        return PTR_ERR(base);
    
    // 获取并检查中断资源
    irq = platform_get_irq(pdev, 0);
    if (irq == -EPROBE_DEFER)
        return -EPROBE_DEFER;   // 延迟探测
    if (irq < 0) {
        dev_err(&pdev->dev, "No valid IRQ\n");
        return irq;
    }
    
    return 0;
}
```

> [!tip]+ 调试技巧 
> 在资源获取过程中，建议添加详细的日志输出，这样可以在调试时清楚地看到资源的获取情况：
> 
> ```c
> dev_info(&pdev->dev, "Memory resource: 0x%llx-0x%llx (%llu bytes)\n",
>          (u64)res->start, (u64)res->end, (u64)resource_size(res));
> ```

## 5 总结

**资源获取是驱动程序的基础工作**，只有正确获取和管理硬件资源，驱动程序才能正常控制硬件设备。这个过程体现了Linux设备模型"分离关注点"的设计思想：设备提供资源描述，驱动负责资源使用。


