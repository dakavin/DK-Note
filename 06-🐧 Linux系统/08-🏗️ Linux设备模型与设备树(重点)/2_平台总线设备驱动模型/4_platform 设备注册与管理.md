
在前面我们学习了Platform总线的工作原理和匹配机制，了解了Platform总线如何为简单设备提供统一的管理框架。今天我们深入学习**Platform设备的注册与管理**，这是实现硬件信息与驱动代码分离的关键一步。

简单来说，**Platform设备就是硬件信息的载体**，它告诉驱动程序"我是什么设备、我在哪里、我需要什么资源"。

> [!note]+ 核心理念 
> Platform设备的作用是"描述硬件"：把原本写死在驱动代码里的寄存器地址、中断号、GPIO配置等硬件信息，统一用标准的数据结构来描述。

## 1 Platform设备的数据结构

### 1.1 platform_device结构体详解

**platform_device**是描述Platform设备的核心数据结构：
```c
struct platform_device {
    const char *name;               // 设备名称，用于唯一表示设备
    int id;                         // 设备ID，用于区分同种设备的不同实例
    bool id_auto;                   // 表示设备ID，是否自动生成
    struct device dev;              // 继承的设备结构体，用于设备的基本管理和操作
    u32 num_resources;              // 设备资源数量
    struct resource *resource;      // 指向设备资源的指针
    const struct platform_device_id *id_entry;  // 匹配成功的ID条目，用于匹配驱动
    char *driver_override;          // 强制设备与指定驱动匹配的驱动名称
    /* 省略部分成员 */
};
```
**1. name（设备名称）**
- 必须提供的唯一标识符
- 用于总线匹配，驱动会根据这个名称来识别设备
- 命名要有意义，比如"led_device"、"button_device"等
**2. id（设备ID）**
- 用于区分同一类型的不同设备实例
- 可以设置M_为-1表示不使用ID
- 支持自动分配（PLATFORDEVID_AUTO）
**3. dev（设备结构体）**
- 继承Linux设备模型的device结构体
- **release方法必须实现**，否则编译时会报错
- 提供完整的设备管理功能

**4. resource（资源信息）**
- 指向resource结构体数组
- 描述设备需要的硬件资源
- 如内存地址、中断号、DMA通道等

> [!important]+ 重要提醒 
> **device结构体的release方法是必须实现的**，如果不实现，在移除设备时会出现"Device does not have a release() function"的错误提示。

### 1.2 resource结构体 - 硬件资源描述

**resource结构体**是硬件资源的标准描述方式：

![resource结构体图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e0f56fcd63c6c7f176682bc621d517a0.png)

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
    /* 省略部分成员 */
};
```

**资源类型相关标志位**

| 序号  | 标志位            | 说明                  |
| :-: | -------------- | ------------------- |
|  1  | IORESOURCE_IO  | 表示资源是 I/O 端口资源      |
|  2  | IORESOURCE_MEM | 表示资源是内存资源           |
|  3  | IORESOURCE_REG | 表示资源是寄存器偏移量         |
|  4  | IORESOURCE_IRQ | 表示资源是中断资源           |
|  5  | IORESOURCE_DMA | 表示资源是 DMA（直接内存访问）资源 |

**资源属性和特征相关标志位**

| 序号  | 标志位                    | 说明               |
| :-: | ---------------------- | ---------------- |
|  1  | IORESOURCE_PREFETCH    | 表示资源是无副作用的预取资源   |
|  2  | IORESOURCE_READONLY    | 表示资源是只读的         |
|  3  | IORESOURCE_CACHEABLE   | 表示资源支持缓存         |
|  4  | IORESOURCE_RANGELENGTH | 表示资源的范围长度        |
|  5  | IORESOURCE_SHADOWABLE  | 表示资源可以被影子资源替代    |
|  6  | IORESOURCE_SIZEALIGN   | 表示资源的大小表示对齐      |
|  7  | IORESOURCE_STARTALIGN  | 表示起始字段是对齐的       |
|  8  | IORESOURCE_MEM_64      | 表示资源是 64 位内存资源   |
|  9  | IORESOURCE_WINDOW      | 表示资源由桥接器转发       |
| 10  | IORESOURCE_MUXED       | 表示资源是软件复用的       |
| 11  | IORESOURCE_SYSRAM      | 表示资源是系统 RAM（修饰符） |

**其他状态和控制标志位**

| 序号  | 标志位                  | 说明            |
| :-: | -------------------- | ------------- |
|  1  | IORESOURCE_EXCLUSIVE | 表示用户空间无法映射此资源 |
|  2  | IORESOURCE_DISABLED  | 表示资源当前被禁用     |
|  3  | IORESOURCE_UNSET     | 表示尚未分配地址给资源   |
|  4  | IORESOURCE_AUTO      | 表示地址由系统自动分配   |

> [!example]+ 实际应用举例 对于一个GPIO控制器设备，可能需要这些资源：
> 
> ```c
> static struct resource gpio_resources[] = {
>     {
>         .start = 0x50000000,        // GPIO寄存器基地址
>         .end   = 0x50000FFF,        // 地址范围
>         .flags = IORESOURCE_MEM,    // 内存类型资源
>         .name  = "gpio-regs",       // 资源名称
>     },
>     {
>         .start = 32,                // 中断号
>         .end   = 32,                // 同一个中断
>         .flags = IORESOURCE_IRQ,    // 中断类型资源
>         .name  = "gpio-irq",        // 资源名称
>     },
> };
> ```


## 2 Platform设备的注册流程

### 2.1 platform_device_register注册函数

**platform_device_register**是Platform设备注册的标准接口，它的工作就像给设备"办理入户手续"

![platform_device_register函数流程](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f90eccf2c2efe53e36f89f16906bd744.png)

**函数原型**：
```c
// 内核源码/include/linux/platform_device.h
// 函数声明
extern int platform_device_register(struct platform_device *);

int platform_device_register(struct platform_device *pdev);
```
- **pdev**：指向platform_device结构体的指针，包含设备的完整信息
- **返回值**：成功返回0，失败返回负数错误码

换句话说，这个函数就是设备向内核"自我介绍"的过程，告诉内核自己的身份和需要的资源。

### 2.2 platform_device_register内部实现

```c
// 内核源码/drivers/base/platform.c
int platform_device_register(struct platform_device *pdev)
{ 
    device_initialize(&pdev->dev);     // 初始化设备基础结构
    arch_setup_pdev_archdata(pdev);    // 设置架构相关数据
    return platform_device_add(pdev);  // 添加设备到系统
}
```
- **device_initialize**：设置设备的引用计数、设备类型等基本信息
- **arch_setup_pdev_archdata**：根据具体的硬件架构设置（ARM、x86等）平台相关数据
- **platform_device_add**：实际注册，将设备添加到平台总线的设备列表中，并触发总线的match

> [!tip]+ 理解要点 
> 注册过程实际上是"三步走"：先初始化、再适配架构、最后正式登记。这样的设计确保了设备能在各种环境下正确工作。

### 2.3 platform_device_add函数详解

**platform_device_add**是设备注册的核心函数，负责完成所有的注册细节：
```c
int platform_device_add(struct platform_device *pdev)
{
    u32 i;
    int ret;
    
    // 检查设备指针有效性
    if (!pdev)
        return -EINVAL;
    
    // 设置设备的父设备为platform_bus
    if (!pdev->dev.parent)
        pdev->dev.parent = &platform_bus;
    
    // 设置设备的总线为platform_bus_type
    pdev->dev.bus = &platform_bus_type;
    
    // 根据设备ID设置设备名称
    switch (pdev->id) {
    default:
        // 普通情况：设备名.ID，如led_dev.0
        dev_set_name(&pdev->dev, "%s.%d", pdev->name, pdev->id);
        break;
    case PLATFORM_DEVID_NONE:
        // 无ID：只使用设备名，如led_dev
        dev_set_name(&pdev->dev, "%s", pdev->name);
        break;
    case PLATFORM_DEVID_AUTO:
        // 自动分配ID，如led_dev.1.auto
        ret = ida_simple_get(&platform_devid_ida, 0, 0, GFP_KERNEL);
        if (ret < 0)
            goto err_out;
        pdev->id = ret;
        pdev->id_auto = true;
        dev_set_name(&pdev->dev, "%s.%d.auto", pdev->name, pdev->id);
        break;
    }
    
    // 处理设备资源
    for (i = 0; i < pdev->num_resources; i++) {
        struct resource *p, *r = &pdev->resource[i];
        
        // 如果资源名称为空，使用设备名称
        if (r->name == NULL)
            r->name = dev_name(&pdev->dev);
        
        p = r->parent;
        if (!p) {
            // 设置默认的父资源
            if (resource_type(r) == IORESOURCE_MEM)
                p = &iomem_resource;
            else if (resource_type(r) == IORESOURCE_IO)
                p = &ioport_resource;
        }
        
        // 将资源插入到资源树中
        if (p && insert_resource(p, r)) {
            dev_err(&pdev->dev, "failed to claim resource %d: %pR\n", i, r);
            ret = -EBUSY;
            goto failed;
        }
    }
    
    // 打印注册信息
    pr_debug("Registering platform device '%s'. Parent at %s\n",
             dev_name(&pdev->dev), dev_name(pdev->dev.parent));
    
    // 最终注册设备到设备子系统
    ret = device_add(&pdev->dev);
    if (ret == 0)
        return ret;
    
failed:
    // 错误处理：清理已分配的资源
    if (pdev->id_auto) {
        ida_simple_remove(&platform_devid_ida, pdev->id);
        pdev->id = PLATFORM_DEVID_AUTO;
    }
    while (i--) {
        struct resource *r = &pdev->resource[i];
        if (r->parent)
            release_resource(r);
    }
    
err_out:
    return ret;
}
```

> [!warning]+ 常见错误 
> 如果资源地址有冲突，insert_resource会失败，函数返回-EBUSY错误。这通常说明多个设备试图使用相同的硬件资源。

### 2.4 platform_device_unregister注销函数

设备使用完毕后，需要正确注销以释放资源：
```c
void platform_device_unregister(struct platform_device *pdev)
{
    platform_device_del(pdev);    // 从设备树中移除
    platform_device_put(pdev);    // 减少引用计数
}
```
- **platform_device_del**：将设备从Platform总线的设备列表中移除
- **platform_device_put**：减少设备的引用计数，当计数为零时释放设备内存

## 3 Platform设备定义实例

### 3.1 定义设备资源

让我们通过一个LED设备的完整例子来理解Platform设备的定义：
```c
#include <linux/platform_device.h>
#include <linux/ioport.h>

// 定义GPIO寄存器地址
#define GPIO0_BASE      0xfdd60000
#define GPIO0_DR        (GPIO0_BASE + 0x0004)    // 数据寄存器
#define GPIO0_DDR       (GPIO0_BASE + 0x000C)    // 方向寄存器

// 定义设备资源
static struct resource led_resources[] = {
    [0] = {
        .start = GPIO0_DR,
        .end   = GPIO0_DR + 3,
        .flags = IORESOURCE_MEM,
        .name  = "led-data-reg",
    },
    [1] = {
        .start = GPIO0_DDR,
        .end   = GPIO0_DDR + 3,
        .flags = IORESOURCE_MEM,
        .name  = "led-dir-reg",
    },
};
```

### 3.2 定义设备私有数据

除了硬件资源，设备还可能需要一些配置信息：
```c
// LED设备的配置信息
struct led_platform_data {
    unsigned int gpio_pin;          // GPIO引脚号
    unsigned int active_level;      // 有效电平（0或1）
    const char  *label;             // LED标签
};

// 具体的LED配置
static struct led_platform_data led_pdata = {
    .gpio_pin    = 7,               // 使用GPIO0_C7
    .active_level = 0,              // 低电平有效
    .label       = "sys_led",       // 系统LED
};
```

### 3.3 定义完整的Platform设备

```c
// 设备释放函数（必须定义）
static void led_device_release(struct device *dev)
{
    // 通常情况下可以为空，但必须定义
    pr_info("LED device released\n");
}

// 完整的Platform设备定义
static struct platform_device led_device = {
    .name = "led_platform",                    // 设备名称
    .id   = 0,                                 // 设备ID
    .num_resources = ARRAY_SIZE(led_resources), // 资源数量
    .resource = led_resources,                  // 资源数组
    .dev = {
        .platform_data = &led_pdata,           // 私有数据
        .release = led_device_release,          // 释放函数
    },
};
```

### 3.4 设备注册模块

```c
// 模块初始化函数
static int __init led_device_init(void)
{
    int ret;
    
    pr_info("LED platform device init\n");
    
    // 注册Platform设备
    ret = platform_device_register(&led_device);
    if (ret) {
        pr_err("Failed to register LED platform device: %d\n", ret);
        return ret;
    }
    
    pr_info("LED platform device registered successfully\n");
    return 0;
}

// 模块退出函数
static void __exit led_device_exit(void)
{
    pr_info("LED platform device exit\n");
    
    // 注销Platform设备
    platform_device_unregister(&led_device);
    
    pr_info("LED platform device unregistered\n");
}

module_init(led_device_init);
module_exit(led_device_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("LED Platform Device Example");
```

> [!tip]+ 编程技巧
> 
> 1. 使用ARRAY_SIZE宏自动计算资源数量，避免手动计数错误
> 2. 资源定义时使用数组索引[0]、[1]，方便驱动程序按索引获取
> 3. 给资源设置有意义的名称，便于调试和维护

## 4 设备注册后的系统变化

### 4.1 文件系统中的体现

设备注册成功后，在文件系统中会有相应的体现：

**1. /sys/bus/platform/devices/**
- 出现设备链接文件：`led_platform.0`
- 这是一个符号链接，指向实际的设备目录

**2. /sys/devices/platform/**
- 出现实际的设备目录：`led_platform.0/`
- 包含设备的各种属性文件

**3. 设备目录下的文件**
- `driver` → 指向匹配的驱动（如果已匹配）
- `modalias` → 设备的模块别名
- `uevent` → 设备事件信息
- 其他属性文件

### 4.2 总线匹配触发

设备注册完成后，Platform总线会立即尝试为该设备寻找合适的驱动：
1. **遍历驱动列表**：检查所有已注册的Platform驱动
2. **执行匹配函数**：调用platform_match函数进行匹配
3. **匹配成功处理**：如果找到匹配的驱动，调用驱动的probe函数
4. **等待驱动**：如果暂时没有匹配的驱动，设备会等待后续驱动的注册

> [!note]+ 理解要点 
> 设备注册不仅仅是"登记信息"，更重要的是"激活匹配机制"。一旦注册成功，总线就会积极地为设备寻找合适的驱动程序。

## 5 总结

**Platform设备是实现"一套驱动，多种硬件"的关键技术**，通过标准化的设备描述，我们可以让同一个驱动程序支持不同的硬件配置。

接下来我们将学习Platform驱动的注册与管理，了解驱动程序如何获取设备资源并控制硬件。
