
## 1 引言

在上一章中，我们了解了**GPIO的基本概念和硬件原理**，就像认识了工具的外形和用途。现在，是时候学习如何使用这些工具了。本章将深入GPIO子系统的内部结构，学习Linux提供的编程接口，并通过实际的代码示例来掌握GPIO编程。

学习编程接口就像学习驾驶汽车。你不仅要知道方向盘、油门、刹车的位置（API函数），还要理解它们是如何协同工作的（数据结构和调用流程）。更重要的是，你需要大量的练习才能熟练驾驶（编写代码）。

本章会涉及较多的数据结构和函数接口，可能会让初学者感到有些枯燥。但请相信，这些都是GPIO编程的基石。我们会通过详细的注释和实例，让这些抽象的概念变得具体可感。

参考资料
- Linux 5.x内核文档
    - Linux-5.4\Documentation\driver-api
    - Linux-5.4\Documentation\devicetree\bindings\gpio\gpio.txt
    - Linux-5.4\drivers\gpio\gpio-74x164.c
- Linux 4.x内核文档
    - Linux-4.9.88\Documentation\gpio
    - Linux-4.9.88\Documentation\devicetree\bindings\gpio\gpio.txt
    - Linux-4.9.88\drivers\gpio\gpio-74x164.c

## 2 GPIO子系统的层次结构

### 2.1 整体架构回顾

让我们再次审视GPIO子系统的整体架构，这次我们要深入到内部实现细节：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/04f042665ea5885c27491931584c5273.png)

从这个架构图可以看出，GPIO子系统采用了经典的分层设计：

**上层接口**：提供给驱动开发者使用的`API函数`。就像餐厅的菜单，你只需要知道有哪些菜（函数），不需要知道厨师是怎么做的。

**中间层（gpiolib）**：`GPIO子系统的核心，管理所有的GPIO资源`。它就像餐厅的厨房，负责协调各种资源，完成客人的点单。

**底层驱动**：`具体的GPIO控制器驱动，由芯片厂商提供`。这就像不同的厨师，虽然做菜的手法不同，但都能做出菜单上的菜品。

### 2.2 GPIOLIB向上提供的接口（消费者-gpio设备）

GPIOLIB向上层提供了两套接口，它们的对比如下：

**基于描述符（descriptor-based） 的新接口**（推荐使用）：
- 定义在内核源码的`drivers/gpio/gpiolib.h` 目录
- 所有函数以`gpiod_`开头
- 使用`struct gpio_desc`结构体
- 支持设备资源管理（devm_前缀）
- 功能更完善，使用更安全

**基于整数的传统接口**（逐渐废弃）：
- 所有函数以`gpio_`开头
- 使用整数表示GPIO编号
- 为兼容老代码而保留
- 新项目不推荐使用

让我们看一个具体的对比表格，了解两套接口的对应关系：

| 功能描述        | 基于`描述符`的接口(新)          | 基于`整数`的接口（旧）          |
| ----------- | ---------------------- | --------------------- |
| 获得GPIO      | gpiod_get              | gpio_request          |
| 获得GPIO（带索引） | gpiod_get_index        | -                     |
| 获得GPIO数组    | gpiod_get_array        | gpio_request_array    |
| 设置为输入       | gpiod_direction_input  | gpio_direction_input  |
| 设置为输出       | gpiod_direction_output | gpio_direction_output |
| 读取值         | gpiod_get_value        | gpio_get_value        |
| 设置值         | gpiod_set_value        | gpio_set_value        |
| 释放GPIO      | gpiod_put              | gpio_free             |

> [!tip]+ 实用技巧 
> 如果你看到函数名中有`devm_`前缀，这表示"设备资源管理"（Device Resource Management）。使用这类函数申请的资源会在设备卸载时自动释放，不需要手动调用释放函数，可以有效避免资源泄漏。

### 2.3 GPIOLIB向下提供的接口（生产者-gpio控制器）

**gpiochip_add_data**：注册GPIO芯片到系统，建立**GPIO控制器与GPIO框架**的映射关系
```c
// include/linux/gpio/driver.h
int gpiochip_add_data(struct gpio_chip *chip, void *data);
```
- `chip`：指向要注册的GPIO芯片结构体，必须预先初始化chip->base字段。当chip->base为负数时，系统将动态分配GPIO编号范围；当chip->base为非负数时，使用指定的起始编号
- `data`：与该GPIO芯片关联的私有数据指针，可为NULL。通过gpiochip_get_data()可获取此数据
- **返回值**：
	- 成功：返回0，GPIO芯片成功注册到系统
	- 失败：返回负数错误码，常见情况包括chip->base无效、GPIO编号范围冲突、或系统资源不足

**调用要求**：
- 必须在gpiolib框架初始化完成后调用（core_initcall()之后）
- 在早期启动阶段调用时，chip->parent设备必须在GPIO框架的arch_initcall()之前完成注册
- 可在IRQ子系统启用前调用

**gpiochip_add_data宏**：GPIO芯片注册宏，根据LOCKDEP配置自动选择合适的实现方式
```c
#ifdef CONFIG_LOCKDEP
/* 启用LOCKDEP时：创建静态锁类键，用于锁依赖跟踪 */
#define gpiochip_add_data(chip, data) ({		\
		static struct lock_class_key lock_key;	        /* 静态锁类键 */ \
		static struct lock_class_key request_key;	/* 静态请求锁类键 */ \
		gpiochip_add_data_with_key(chip, data, &lock_key, \
					   &request_key);	        /* 调用带锁键的版本 */ \
	})
#else
/* 未启用LOCKDEP时：直接传递NULL作为锁键参数 */
#define gpiochip_add_data(chip, data) gpiochip_add_data_with_key(chip, data, NULL, NULL)
#endif
```
- **参数和返回值**：`同上`

**宏展开行为**：
- **CONFIG_LOCKDEP启用时**：创建静态锁类键（lock_key和request_key），调用gpiochip_add_data_with_key()传递锁键参数，用于内核死锁检测和锁依赖分析
- **CONFIG_LOCKDEP未启用时**：直接调用gpiochip_add_data_with_key()并传递NULL作为锁键参数，避免锁跟踪的性能开销

**设计目的**：在保持API一致性的同时，**根据内核配置自动启用或禁用锁依赖检测功能**

**gpiochip_add_data_with_key**函数解析：带锁键的GPIO芯片注册函数
```c fold
// drivers/gpio/gpiolib.c
// @chip: 要注册的GPIO芯片
// @data: 关联的数据指针
// @lock_key: 锁类键(用于LOCKDEP)
// @request_key: 请求锁类键(用于LOCKDEP)
int gpiochip_add_data_with_key(struct gpio_chip *chip, void *data,
			       struct lock_class_key *lock_key,
			       struct lock_class_key *request_key)
{
	unsigned long	flags;
	int		status = 0;
	unsigned	i;
	int		base = chip->base;		// 获取芯片基地址
	struct gpio_device *gdev;

	/*
	 * 第一步：分配并填充内部状态容器，设置struct device
	 */
	
	/* 分配GPIO设备结构体内存 */
	gdev = kzalloc(sizeof(*gdev), GFP_KERNEL);
	if (!gdev)
		return -ENOMEM;			// 内存分配失败
	
	/* 设置设备总线类型为GPIO总线 */
	gdev->dev.bus = &gpio_bus_type;
	
	/* 建立芯片和设备的双向关联 */
	gdev->chip = chip;
	chip->gpiodev = gdev;
	
	/* 设置父设备关系和设备树节点 */
	if (chip->parent) {
		gdev->dev.parent = chip->parent;	// 设置父设备
		gdev->dev.of_node = chip->parent->of_node;  // 继承父设备的设备树节点
	}

#ifdef CONFIG_OF_GPIO
	/* 如果gpiochip有指定的设备树节点，则优先使用 */
	if (chip->of_node)
		gdev->dev.of_node = chip->of_node;	// 使用芯片指定的设备树节点
	else
		chip->of_node = gdev->dev.of_node;	// 否则使用设备的设备树节点
#endif
```


**devm_gpiochip_add_data**：设备管理版GPIO芯片注册，自动处理资源释放的GPIO控制器注册函数
```c  fold
/**
 * devm_gpiochip_add_data - 设备管理版本的GPIO芯片注册函数
 * @dev: 管理此GPIO芯片的设备
 * @chip: 要注册的GPIO芯片
 * @data: 关联的数据指针
 * 
 * 使用设备资源管理机制注册GPIO芯片，设备移除时会自动清理资源
 * 返回: 成功返回0，失败返回错误码
 */
int devm_gpiochip_add_data(struct device *dev, struct gpio_chip *chip,
			   void *data)
{
	struct gpio_chip **ptr;
	int ret;
	
	/* 分配设备资源管理内存，用于存储芯片指针 */
	ptr = devres_alloc(devm_gpio_chip_release, sizeof(*ptr),
			     GFP_KERNEL);
	if (!ptr)
		return -ENOMEM;		// 内存分配失败
	
	/* 调用实际的GPIO芯片注册函数 */
	ret = gpiochip_add_data(chip, data);
	if (ret < 0) {
		devres_free(ptr);	// 注册失败时释放已分配的资源
		return ret;
	}
	
	/* 保存芯片指针并添加到设备资源管理中 */
	*ptr = chip;
	devres_add(dev, ptr);		// 将资源添加到设备管理列表
	
	return 0;
}
/* 导出符号，允许GPL模块使用此函数 */
EXPORT_SYMBOL_GPL(devm_gpiochip_add_data);
```

**struct rockchip_pin_bank**：描述Rockchip芯片上的一个GPIO/PIN组的所有信息
```c fold
struct rockchip_pin_bank {
        struct device *dev;                     // 关联的设备指针
        void __iomem                    *reg_base;              // 寄存器基地址
        struct regmap                   *regmap_pull;           // 上下拉配置寄存器映射
        struct clk                      *clk;                   // 时钟源
        struct clk                      *db_clk;                // 去抖动时钟
        int                             irq;                    // 中断号
        u32                             saved_masks;            // 保存的中断掩码
        u32                             pin_base;               // 引脚基地址
        u8                              nr_pins;                // 引脚数量
        char                            *name;                  // 引脚组名称
        u8                              bank_num;               // 引脚组编号
        struct rockchip_iomux           iomux[4];               // IO复用配置(最多4个)
        struct rockchip_drv             drv[4];                 // 驱动强度配置(最多4个)
        enum rockchip_pin_pull_type     pull_type[4];           // 上下拉类型(最多4个)
        struct device_node              *of_node;               // 设备树节点
        struct rockchip_pinctrl         *drvdata;               // 引脚控制器数据
        struct irq_domain               *domain;                // 中断域
        struct gpio_chip                gpio_chip;              // GPIO芯片结构
        struct pinctrl_gpio_range       grange;                 // GPIO范围
        raw_spinlock_t                  slock;                  // 自旋锁，保护寄存器访问
        const struct rockchip_gpio_regs *gpio_regs;             // GPIO寄存器配置
        u32                             gpio_type;              // GPIO类型
        u32                             toggle_edge_mode;       // 边沿触发模式切换
        u32                             recalced_mask;          // 重新计算的掩码
        u32                             route_mask;             // 路由掩码
};
```

## 3 GPIO子系统的核心数据结构

在深入学习API之前，我们需要理解GPIO子系统中的三个核心数据结构。这就像学习面向对象编程，要先理解类的定义，才能更好地使用类的方法。

首先我们要理解GPIO控制器的要素，有助于理解它的驱动程序
- 一个GPIO控制器里有多少个引脚？有那些引脚？
- 需要提供函数设置引脚方向、读取和设置数值
- 需要提供函数，把引脚转换为中断

所以，按照Linux面向对象编程的事项，一个GPIO控制器必定会使用一个结构体来表示，这个结构体包含
- GPIO引脚信息
- 控制引脚相关函数
- 中断相关函数

### 3.1 gpio_device - GPIO控制器的抽象（核心）

> [!tip]+ 提示
> gpio_device不需要芯片厂商自己实现，是注册生成的

**每个GPIO控制器**在内核中用一个`gpio_device`结构体表示。这个结构体包含了该控制器的所有信息：
```c
// 内核源码/drivers/gpio/gpiolib.h
struct gpio_device {
    int                       id;           // gpio控制器的id，表示是第几个控制器
    struct device             dev;          // 内嵌的设备结构体
    struct cdev               chrdev;       // 字符设备，用于实现/dev/gpiochipX接口
    struct device             *mockdev;     // 模拟设备指针
    struct module             *owner;       // 所属模块
    struct gpio_chip          *chip;        // 指向gpio_chip结构体，包含操作函数！！！
    struct gpio_desc          *descs;       // GPIO描述符数组，每个引脚一个描述符！！！
    int                       base;         // 该控制器的GPIO起始编号
    u16                       ngpio;        // 该控制器管理的GPIO数量
    const char                *label;       // 控制器的标签名称
    void                      *data;        // 私有数据指针
    struct list_head          list;         // 用于将该结构体链接到全局链表

#ifdef CONFIG_PINCTRL
    /*
     * 如果启用了CONFIG_PINCTRL，GPIO控制器可以描述它们在SoC中
     * 服务的实际引脚范围。这些信息会被pinctrl子系统用来
     * 配置相应的引脚功能。
     */
    struct list_head pin_ranges;           // 引脚范围链表
#endif
};
```

这个结构体就像一个GPIO控制器的"身份证"，记录了它的所有信息。其中最重要的两个成员是：
- `chip`：指向具体的操作函数集合，即引脚的函数（引脚控制、中断相关）
- `descs`：GPIO描述符数组，记录每个引脚的状态，即每个引脚用一个gpio_desc表示

### 3.2 gpio_chip - GPIO操作函数集

> [!tip]+ 提示
> 对于gpio_device我们不需要自己创建，但是**gpio_chip需要创建！！**，用于提供相关函数，不过一般都是由芯片原厂工程师来完成的，我们只需要学会新gpio子系统相应的API函数使用即可

`gpio_chip`结构体定义了**操作GPIO的所有函数指针**，这是GPIO控制器驱动需要实现的**核心内容**：
```c
// 内核源码/include/linux/gpio/driver.h
struct gpio_chip {
    const char                *label;       // GPIO控制器的标签
    struct gpio_device        *gpiodev;     // 关联的gpio_device
    struct device             *parent;      // 父设备
    struct module             *owner;       // 所属模块

    /* 下面是GPIO操作相关的函数指针 */
    
    /* 请求和释放GPIO */
    int (*request)(struct gpio_chip *chip, unsigned offset);
    void (*free)(struct gpio_chip *chip, unsigned offset);
    
    /* 方向控制函数 */
    int (*get_direction)(struct gpio_chip *chip, unsigned offset);
    int (*direction_input)(struct gpio_chip *chip, unsigned offset);
    int (*direction_output)(struct gpio_chip *chip, unsigned offset, int value);
    
    /* 读写GPIO值 */
    int (*get)(struct gpio_chip *chip, unsigned offset);
    void (*set)(struct gpio_chip *chip, unsigned offset, int value);
    int (*get_multiple)(struct gpio_chip *chip, unsigned long *mask, unsigned long *bits);
    void (*set_multiple)(struct gpio_chip *chip, unsigned long *mask, unsigned long *bits);
    
    /* 其他配置函数 */
    int (*set_config)(struct gpio_chip *chip, unsigned offset, unsigned long config);
    int (*to_irq)(struct gpio_chip *chip, unsigned offset);  // GPIO转中断号
    void (*dbg_show)(struct seq_file *s, struct gpio_chip *chip);
    
    int                       base;         // GPIO编号基址
    u16                       ngpio;        // GPIO数量
    const char                *const *names;// GPIO名称数组
    bool                      can_sleep;    // 操作是否可能睡眠
    
    /* ... 省略其他成员 ... */
};
```

这个结构体定义了GPIO控制器必须提供的"能力清单"。**芯片厂商在编写GPIO控制器驱动时，需要实现这些函数指针指向的具体操作**

### 3.3 gpio_desc - GPIO引脚描述符

**每个GPIO引脚**都有一个对应的`gpio_desc`结构体，记录该引脚的当前状态：
```c
// 内核源码/drivers/gpio/gpiolib.h
struct gpio_desc {
    struct gpio_device        *gdev;        // 所属的GPIO控制器
    unsigned long             flags;        // 状态标志位
    
    /* 标志位定义 */
    #define FLAG_REQUESTED    0   // 引脚已被申请
    #define FLAG_IS_OUT       1   // 配置为输出
    #define FLAG_EXPORT       2   // 已导出到sysfs
    #define FLAG_SYSFS        3   // 通过sysfs导出
    #define FLAG_ACTIVE_LOW   6   // 低电平有效
    #define FLAG_OPEN_DRAIN   7   // 开漏输出
    #define FLAG_OPEN_SOURCE  8   // 开源输出
    #define FLAG_USED_AS_IRQ  9   // 用作中断
    #define FLAG_IS_HOGGED    11  // 被占用
    #define FLAG_TRANSITORY   12  // 可能在睡眠或复位时丢失状态

    const char                *label;       // 使用者标签
    const char                *name;        // GPIO名称
};
```

这个结构体记录了GPIO引脚的"当前状态"，包括：
- 是否已被申请使用
- 当前的输入输出方向
- 是否配置为低电平有效
- 是否用作中断等

> [!note]+ 背景知识 
> 标志位使用位操作来管理，这是内核中常见的技巧。每个标志占用一个比特位，可以用`set_bit()`、`clear_bit()`、`test_bit()`等函数来操作。这种方式既节省内存，又提高了操作效率。

### 3.4 示例：GPIO控制器驱动程序

参考文章：[3 Rockchip GPIO控制器驱动实例](3_GPIO控制器驱动深入解析.md#3%20Rockchip%20GPIO控制器驱动实例)

## 4 基于描述符的GPIO接口详解

现在让我们详细学习推荐使用的基于描述符的GPIO接口。这套接口设计得更加完善，使用起来也更加安全。

### 4.1 获取GPIO描述符

在使用GPIO之前，首先需要获取GPIO描述符。这就像去图书馆借书，你需要先办理借阅手续。

内核头文件：`linux/gpio/consumer.h`

**gpiod_get**：Linux内核提供的GPIO子系统接口函数，用于获取与给定设备和连接标识符相关联的GPIO描述符。该函数的主要作用是在系统中请求分配一个GPIO引脚，并将其与特定设备关联起来。
```c
struct gpio_desc *gpiod_get(struct device *dev, const char *con_id, 
                           enum gpiod_flags flags);
```
- `dev`：指向设备结构体的指针，表示与GPIO相关联的设备
- `con_id`：连接标识符（connection identifier），用于标识所需的GPIO连接。
	- 一般可以根据设备树中 xxx-gpios属性，然后让其为xxx字符串即可
	- 表示目前占用gpio的对象的名称，可以在`cat /sys/kernel/debug/gpio` 看到
	  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/07/16327c816e9f444e6f9a0401b4a90207.png)
- `flags`：GPIO描述符的选项标志，用于指定GPIO的属性和操作模式
- **返回值**：
    - 成功：返回GPIO描述符指针
    - 失败：返回错误码

**gpiod_get_optional**：一个用于获取可选GPIO描述符的函数。其作用是与`gpiod_get`类似，但当GPIO不存在时返回`NULL`而不是错误，适用于可选的GPIO配置。
```c
struct gpio_desc *gpiod_get_optional(struct device *dev, const char *con_id,
                                    enum gpiod_flags flags);
```
- **入参**：同上
- **返回值**：
    - 成功：返回GPIO描述符指针
    - GPIO不存在：返回`NULL`
    - 失败：返回错误码

**gpiod_get_index**：一个用于获取指定索引GPIO描述符的函数。其作用是当设备树中一个属性包含多个GPIO时，根据索引获取特定的GPIO描述符。
```c
struct gpio_desc *gpiod_get_index(struct device *dev, const char *con_id, 
                                  unsigned int idx, enum gpiod_flags flags);
```
- **入参**：同上
- `idx`：GPIO索引，当一个属性包含多个GPIO时使用
- **返回值**：
    - 成功：返回GPIO描述符指针
    - 失败：返回错误码

**gpiod_get_index_optional**：获取指定索引的可选GPIO描述符，如果GPIO不存在也不会报错，适用于可选的GPIO功能
```c
// #include <linux/gpio/consumer.h>
struct gpio_desc *gpiod_get_index_optional(struct device *dev, const char *con_id, unsigned int index, enum gpiod_flags flags);
```
- **入参**：同上
- **返回值**：
    - 成功：返回有效的GPIO描述符指针
    - GPIO不存在：返回NULL（这是正常情况，不是错误）
    - 失败：返回错误指针（使用IS_ERR()检查），如-ENODEV（设备不存在）、-EINVAL（无效参数）等

配置标志`flags`可以是以下值：
- `GPIOD_ASIS`：保持现有配置
- `GPIOD_IN`：配置为输入
- `GPIOD_OUT_LOW`：配置为输出，初始输出低电平
- `GPIOD_OUT_HIGH`：配置为输出，初始输出高电平

让我们看一个实际的使用例子：
```c
/* 从设备树中获取名为"led"的GPIO，配置为输出高电平 */
struct gpio_desc *led_gpio;
led_gpio = gpiod_get(&pdev->dev, "led", GPIOD_OUT_HIGH);
if (IS_ERR(led_gpio)) {
    dev_err(&pdev->dev, "Failed to get LED GPIO\n");
    return PTR_ERR(led_gpio);
}
```

### 4.2 释放GPIO描述符

**gpiod_put**函数用于释放之前通过`gpiod_get()`或类似函数获取的GPIO描述符。该函数的主要作用是在系统中释放GPIO资源，清理相关状态。
```c
// 内核头文件 linux/gpio/consumer.h
void gpiod_put(struct gpio_desc *desc)
```
- `desc`：指向要释放的GPIO描述符的指针
- **返回值**：无返回值

### 4.3 GPIO方向和电平控制

获取到GPIO描述符后，我们就可以对GPIO进行操作了：

**gpiod_get_direction**函数用于获取GPIO的方向，即判断GPIO是输入还是输出。
```c
// 内核头文件 linux/gpio/consumer.h
int gpiod_get_direction(struct gpio_desc *desc)
```
- `desc`：指向GPIO描述符的指针
- **返回值**：
	- **成功**：获取到GPIO方向
		- **GPIO_LINE_DIRECTION_IN** (0) 表示输入
		- **GPIO_LINE_DIRECTION_OUT** (1) 表示输出。
	- **失败**：返回值为负数，表示错误码。

**gpiod_direction_input**：将指定的GPIO引脚配置为输入模式，用于读取外部信号
```c
// #include <linux/gpio/consumer.h>
int gpiod_direction_input(struct gpio_desc *desc);
```
- `desc`：GPIO描述符指针，指向要配置的GPIO引脚，不能为NULL
- **返回值**：
    - 成功：返回0
    - 失败：返回负错误码，如-EINVAL（无效参数）、-EIO（硬件错误）等

**gpiod_direction_output**：将指定的GPIO引脚配置为输出模式，并设置初始输出电平
```c
// #include <linux/gpio/consumer.h>
int gpiod_direction_output(struct gpio_desc *desc, int value);
```
- `desc`：GPIO描述符指针，指向要配置的GPIO引脚，不能为NULL
- `value`：初始输出电平值，0表示低电平，非0值表示高电平
- **返回值**：
    - 成功：返回0
    - 失败：返回负错误码，如-EINVAL（无效参数）、-EIO（硬件错误）等

**gpiod_get_value**：读取指定GPIO引脚的当前电平状态
```c
// #include <linux/gpio/consumer.h>
int gpiod_get_value(const struct gpio_desc *desc);
```
- `desc`：GPIO描述符指针，指向要读取的GPIO引脚，不能为NULL
- **返回值**：
    - 成功：0表示低电平，1表示高电平
    - 失败：负错误码（较少见），如-EINVAL（无效参数）

**gpiod_set_value**：设置指定GPIO引脚的输出电平
```c
// #include <linux/gpio/consumer.h>
void gpiod_set_value(struct gpio_desc *desc, int value);
```
- `desc`：GPIO描述符指针，指向要设置的GPIO引脚，不能为NULL，该引脚必须已配置为输出模式
- `value`：要设置的电平值，0表示低电平，非0值表示高电平
- **返回值**：
    - 该函数无返回值（void类型）
    - 注意：如果传入无效参数或GPIO未配置为输出模式，可能导致系统异常

### 4.4 GPIO描述符转换为中断编号

**gpiod_to_irq**函数用于将GPIO描述符转换为中断号。该函数用于将给定GPIO描述符所代表的GPIO转换为对应的中断号。
```c
// 内核头文件 linux/gpio/consumer.h
int gpiod_to_irq(const struct gpio_desc *desc);
```
- `desc`：指向GPIO描述符的指针
- **返回值**：
	- **成功**：将GPIO描述符转换为中断号
		- 返回值为大于等于0的中断号
	- **失败**：返回值为负数，表示错误码

### 4.5 示例：LED驱动

让我们通过一个完整的LED驱动示例来综合运用这些接口：

对应的设备树节点：
```d
mygpio {
	mygpio_ctrl: my-gpio-ctrl {
		rockchip,pins = <1 RK_PA0 RK_FUNC_GPIO &pcfg_pull_none>;
	};
};


my_gpio:gpiol_a0 {
	compatible = "mygpio";
	my-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>,<&gpio1 RK_PB0 GPIO_ACTIVE_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&mygpio_ctrl>;
};
```
- **rockchip,pins属性**
	- 1：表示引脚索引，gpio组编号，gpio控制器编号，即gpio1
	- RK_PA0：表示资源描述符，标识与该引脚相关联的物理资源，即bank组，即A组的第0个引脚
	- RK_FUNC_GPIO：引脚的复用功能，这里选择复用为GPIO
	- &pcfg_pull_none：定义引脚的电气特性，这里是引脚无上下拉
		- pcfg_pull_up：上拉
		- pcfg_pull_down：引脚下拉
		- pcfg_pull_none_drv_level_0：

驱动程序代码
```c fold
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/gpio/consumer.h>

struct gpio_desc *mygpio1;  // GPIO 描述符指针
struct gpio_desc *mygpio2;  // GPIO 描述符指针
int num;  // GPIO 编号

//平台设备初始化函数
// 平台设备初始化函数
static int my_platform_probe(struct platform_device *dev)
{
    printk("This is my_platform_probe\n");

    // 获取可选的GPIO描述符
    // 代码中使用的名称：去掉"-gpios"后缀
    mygpio1 = gpiod_get_optional(&dev->dev, "my", 0);
    if (mygpio1 == NULL) {
        printk("gpiod_get_optional error\n");
        return -1;
    }
    num = desc_to_gpio(mygpio1);
    printk("num is %d\n", num);

    // 释放GPIO描述符
    gpiod_put(mygpio1);

    // 获取指定索引的GPIO描述符
    mygpio2 = gpiod_get_index(&dev->dev, "my", 0, 0);
    if (IS_ERR(mygpio2)) {
        printk("gpiod_get_index error\n");
        return -2;
    }
    num = desc_to_gpio(mygpio2);
    printk("num is %d\n", num);

    return 0;
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}

const struct of_device_id of_match_table_id[]  = {
    {.compatible="mygpio"},
};

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
        .of_match_table =  of_match_table_id,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    gpiod_put(mygpio2);
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
```

编译该驱动的Makefile：
```makefile
# 如果是在内核源码树外编译
#设置平台架构
export ARCH=arm64
#交叉编译器前缀
export CROSS_COMPILE=aarch64-linux-gnu-
#platform_device.c对应.o文件的名称。名称要保持一致。
obj-m += gpio_api.o 
#内核源码所在虚拟机ubuntu的实际路径
KDIR :=/kernel    
PWD ?= $(shell pwd)
all:
    make -C $(KDIR) M=$(PWD) modules    #make操作
clean:
    make -C $(KDIR) M=$(PWD) clean    #make clean操作
```

## 5 基于整数的传统GPIO接口

虽然不推荐在新项目中使用，但了解传统接口仍然很重要，因为你可能需要维护使用这些接口的老代码。

源文件路径：`#include <linux/gpio.h>`

### 5.1 主要函数介绍

**1、GPIO申请与释放**

**gpio_request**：申请指定编号的GPIO资源，获取GPIO的独占访问权
```c
// #include <linux/gpio.h>
int gpio_request(unsigned gpio, const char *label);
```
- `gpio`：要申请的GPIO编号，必须是系统中有效的GPIO号
- `label`：标识字符串，用于标记GPIO的用途，便于调试和管理，不能为NULL
	- 类似于新接口中的入参 `con_id`
- **返回值**：
    - 成功：返回0
    - 失败：返回负错误码，如-EBUSY（GPIO已被占用）、-EINVAL（无效GPIO号）等

**gpio_free**：释放之前申请的GPIO资源，允许其他模块使用该GPIO
```c
// #include <linux/gpio.h>
void gpio_free(unsigned gpio);
```
- `gpio`：要释放的GPIO编号，必须是之前通过gpio_request申请的有效GPIO号
- **返回值**：
    - 该函数无返回值（void类型）
    - 注意：释放未申请的GPIO可能导致系统异常

**2、GPIO方向配置**

**gpio_direction_input**：将指定GPIO配置为输入模式
```c
// #include <linux/gpio.h>
int gpio_direction_input(unsigned gpio);
```
- `gpio`：要配置的GPIO编号，必须是已申请的有效GPIO号
- **返回值**：
    - 成功：返回0
    - 失败：返回负错误码，如-EINVAL（无效GPIO号）、-EIO（硬件错误）等

**gpio_direction_output**：将指定GPIO配置为输出模式，并设置初始输出值
```c
// #include <linux/gpio.h>
int gpio_direction_output(unsigned gpio, int value);
```
- `gpio`：要配置的GPIO编号，必须是已申请的有效GPIO号
- `value`：初始输出电平值，0表示低电平，非0值表示高电平
- **返回值**：
    - 成功：返回0
    - 失败：返回负错误码，如-EINVAL（无效GPIO号）、-EIO（硬件错误）等

**3、GPIO值读写（原子上下文）**

**gpio_get_value**：在原子上下文中读取GPIO的当前电平状态
```c
// #include <linux/gpio.h>
int gpio_get_value(unsigned gpio);
```
- `gpio`：要读取的GPIO编号，必须是已申请的有效GPIO号
- **返回值**：
    - 成功：0表示低电平，1表示高电平
    - 失败：负错误码（较少见），如-EINVAL（无效GPIO号）

**gpio_set_value**：在原子上下文中设置GPIO的输出电平
```c
// #include <linux/gpio.h>
void gpio_set_value(unsigned gpio, int value);
```
- `gpio`：要设置的GPIO编号，必须是已申请且配置为输出的有效GPIO号
- `value`：要设置的电平值，0表示低电平，非0值表示高电平
- **返回值**：
    - 该函数无返回值（void类型）
    - 注意：在原子上下文中调用，不能睡眠

**4、GPIO值读写（非原子上下文，可睡眠）**

**gpio_get_value_cansleep**：在非原子上下文中读取GPIO电平，允许睡眠操作
```c
// #include <linux/gpio.h>
static int gpio_get_value_cansleep(unsigned gpio);
```
- `gpio`：要读取的GPIO编号，必须是已申请的有效GPIO号
- **返回值**：
    - 成功：0表示低电平，1表示高电平
    - 失败：负错误码，如-EINVAL（无效GPIO号）

**gpio_set_value_cansleep**：在非原子上下文中设置GPIO电平，允许睡眠操作
```c
// #include <linux/gpio.h>
void gpio_set_value_cansleep(unsigned gpio, int value);
```
- `gpio`：要设置的GPIO编号，必须是已申请且配置为输出的有效GPIO号
- `value`：要设置的电平值，0表示低电平，非0值表示高电平
- **返回值**：
    - 该函数无返回值（void类型）
    - 注意：适用于可能涉及I2C、SPI等总线操作的GPIO控制器

**5、GPIO转IRQ**

**gpio_to_irq**：将GPIO编号转换为对应的中断号
```c
// #include <linux/gpio.h>
int gpio_to_irq(unsigned gpio);
```
- `gpio`：要转换的GPIO编号，必须是支持中断功能的有效GPIO号
- **返回值**：
    - 成功：返回对应的中断号（正整数）
    - 失败：返回负错误码，如-EINVAL（GPIO不支持中断）、-ENODEV（设备不存在）等

**6、检验GPIO是否有效**

**gpio_is_valid**：检查指定的GPIO编号是否在系统中有效
```c
// #include <linux/gpio.h>
bool gpio_is_valid(int gpio);
```
- `gpio`：要检查的GPIO编号，可以是任意整数值
- **返回值**：
    - true：GPIO编号有效，可以在系统中使用
    - false：GPIO编号无效，不能使用该编号

### 5.2 示例：传统接口的使用

```c fold
#include <linux/init.h> 
#include <linux/module.h> 
#include <linux/kernel.h> 
#include <linux/gpio.h>        /* 遗留的基于整数的GPIO */ 
#include <linux/interrupt.h>   /* IRQ */ 
 
static unsigned int GPIO_LED_RED = 8;  //放到设备树里面
static unsigned int GPIO_BTN1 = 115; 
static unsigned int GPIO_BTN2 = 116; 
static unsigned int GPIO_LED_GREEN = 120; 
static unsigned int irq; 
 
static irq_handler_t btn1_pushed_irq_handler(unsigned int irq, 
                             void *dev_id, struct pt_regs *regs) 
{ 
    int state;  
    /*读取BTN2值并更改LED状态*/ 
    state = gpio_get_value(GPIO_BTN2); 
    gpio_set_value(GPIO_LED_RED, state); 
    gpio_set_value(GPIO_LED_GREEN, state); 
 
    pr_info("GPIO_BTN1 interrupt: Interrupt! GPIO_BTN2 state is %d)\n", state); 
    return IRQ_HANDLED; 
} 
 
static int __init helloworld_init(void) 
{ 
    int retval;  
 
    /* 
     * 可以检查GPIO在控制器上是否有效 
     * 使用gpio_is_valid()函数 
     * 例: 
     *  if (!gpio_is_valid(GPIO_LED_RED)) { 
     *       pr_infor("Invalid Red LED\n"); 
     *       return -ENODEV;  
     *  } 
     */ 
    gpio_request(GPIO_LED_GREEN, "green-led"); 
    gpio_request(GPIO_LED_RED, "red-led"); 
    gpio_request(GPIO_BTN1, "button-1"); 
    gpio_request(GPIO_BTN2, "button-2"); 
 
    /* 
     * 配置按钮GPIO作为输入 
     * 
     * 在此之后，只有当控制器具有该特性时才可以调用gpio_set_debounce() 
     * 
     * 例如，取消延迟200ms的按钮 
     * gpio_set_debounce(GPIO_BTN1, 200); 
     */ 
    gpio_direction_input(GPIO_BTN1); 
    gpio_direction_input(GPIO_BTN2); 
 
    /* 
     * 将LED GPIO设置为输出，初始值设置为0 
     */ 
    gpio_direction_output(GPIO_LED_RED, 0); 
    gpio_direction_output(GPIO_LED_GREEN, 0); 
 
    irq = gpio_to_irq(GPIO_BTN1); 
    retval = request_threaded_irq(irq, NULL,\ 
                            btn1_pushed_irq_handler, \ 
                            IRQF_TRIGGER_LOW | IRQF_ONESHOT, \ 
                            "device-name", NULL); 
 
    pr_info("Hello world!\n"); 
    return 0;  
}  
 
static void __exit hellowolrd_exit(void) 
{ 
    free_irq(irq, NULL); 
    gpio_free(GPIO_LED_RED); 
    gpio_free(GPIO_LED_GREEN); 
    gpio_free(GPIO_BTN1); 
    gpio_free(GPIO_BTN2); 
 
    pr_info("End of the world\n"); 
} 
 
module_init(hellowolrd_init); 
module_exit(hellowolrd_exit); 
 
MODULE_LICENSE("GPL");
```

### 5.3 两种接口的转换

在某些情况下，你可能需要在两种接口之间转换：

**desc_to_gpio**：将GPIO描述符转换为对应的GPIO编号
```c
// #include <linux/gpio/consumer.h>
int desc_to_gpio(const struct gpio_desc *desc);
```
- `desc`：GPIO描述符指针，指向要转换的GPIO描述符，不能为NULL
- **返回值**：
    - 成功：返回对应的GPIO编号（非负整数）
    - 失败：返回负错误码，如-EINVAL（无效描述符）、-ENODEV（描述符无效）等

**gpio_to_desc**：将GPIO编号转换为对应的GPIO描述符
```c
// #include <linux/gpio/consumer.h>
struct gpio_desc *gpio_to_desc(unsigned gpio);
```
- `gpio`：要转换的GPIO编号，必须是系统中有效的GPIO号
- **返回值**：
    - 成功：返回有效的GPIO描述符指针
    - 失败：返回NULL或错误指针，如GPIO编号无效或未注册时

## 6 对比新旧API

| **对比维度**   | **旧API（基于GPIO编号）**                                                             | **新API（基于GPIO描述符）**                                                                |
| ---------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| **参数类型**   | `unsigned int gpio`                                                            | `struct gpio_desc *desc`                                                           |
| **头文件**    | `#include <linux/gpio.h>`                                                      | `#include <linux/gpio/consumer.h>`                                                 |
| **资源申请**   | `gpio_request(gpio, label)`                                                    | `gpiod_get(dev, con_id, flags)`<br>`gpiod_get_optional(dev, con_id, flags)`        |
| **资源释放**   | 手动：`gpio_free(gpio)`                                                           | devm前缀可以自动：驱动卸载时自动释放                                                               |
| **方向配置**   | `gpio_direction_input(gpio)`<br>`gpio_direction_output(gpio, value)`           | `gpiod_direction_input(desc)`<br>`gpiod_direction_output(desc, value)`             |
| **读取值**    | 原子：`gpio_get_value(gpio)`<br>可睡眠：`gpio_get_value_cansleep(gpio)`               | 自动选择：`gpiod_get_value(desc)`<br>可睡眠：`gpiod_get_value_cansleep(desc)`               |
| **设置值**    | 原子：`gpio_set_value(gpio, value)`<br>可睡眠：`gpio_set_value_cansleep(gpio, value)` | 自动选择：`gpiod_set_value(desc, value)`<br>可睡眠：`gpiod_set_value_cansleep(desc, value)` |
| **批量操作**   | ❌ 不支持                                                                          | ✅ `gpiod_set_array()`<br>`gpiod_get_multiple()`                                    |
| **中断转换**   | `gpio_to_irq(gpio)`                                                            | `gpiod_to_irq(desc)`                                                               |
| **有效性检查**  | `gpio_is_valid(gpio)`                                                          | 内置检查，无需手动验证                                                                        |
| **相互转换**   | `gpio_to_desc(gpio)`                                                           | `desc_to_gpio(desc)`                                                               |
| **设备树集成**  | ❌ 硬编码GPIO编号<br>移植性差                                                            | ✅ 原生支持<br>`led-gpios = <&gpio1 25 0>;`                                             |
| **极性处理**   | 手动处理活跃高/低                                                                      | 自动处理，支持`GPIO_ACTIVE_LOW`                                                           |
| **可选GPIO** | 需要手动检查和处理                                                                      | `gpiod_get_optional()`优雅处理                                                         |
| **错误处理**   | 简单错误码                                                                          | 更完善的错误检查和处理                                                                        |
| **内存管理**   | 容易资源泄露                                                                         | 自动资源管理，更安全                                                                         |
| **代码复杂度**  | 需要更多样板代码                                                                       | 代码更简洁                                                                              |

## 7 多级GPIO节点的操作

在实际项目中，我们可能需要在一个设备节点下管理多个GPIO，比如**一个LED控制器**管理**多个LED灯**。Linux提供了专门的API来处理这种多级节点的情况。

### 7.1 多级节点相关函数

**fwnode_handle结构体概述**
- 源文件路径`include/linux/fwnode.h` 
```c
struct fwnode_handle {
    struct fwnode_handle *secondary;     /* 次要固件节点 */
    const struct fwnode_operations *ops; /* 操作函数指针表 */
    struct device *dev;                  /* 关联的设备指针 */
    struct list_head suppliers;         /* 生产者链表 */
    struct list_head consumers;         /* 消费者链表 */
    u8 flags;                           /* 标志位 */
};
```

**device_get_child_node_count**函数用于计算设备节点的子节点数量：
```c
// 内核头文件 linux/device.h
unsigned int device_get_child_node_count(struct device *dev);
```
- `dev`：表示要计算子节点数量的设备节点指针
- **返回值**：
	- **成功**：返回一个大于0的无符号整数，表示设备节点的子节点数量
	- **失败**：返回值为0

**device_get_next_child_node**函数用于遍历设备的子节点：
```c
// 内核头文件 linux/device.h
struct fwnode_handle *device_get_next_child_node(struct device *dev, 
                                                 struct fwnode_handle *child);
```
- `dev`：指向struct device的指针，表示父设备节点
- `child`：指向struct fwnode_handle的指针，表示当前子设备节点
- **返回值**：
	- **成功**：返回一个指向struct fwnode_handle的指针，表示下一个子设备节点
	- **失败**：如果没有下一个子设备节点，返回值为NULL

**fwnode_get_named_gpiod**函数通过指定节点的对象地址获取子节点中的GPIO结构描述：
```c
// 内核头文件 linux/gpio/consumer.h
struct gpio_desc *fwnode_get_named_gpiod(struct fwnode_handle *fwnode, 
                                        const char *propname, int index, 
                                        enum gpiod_flags dflags, 
                                        const char *label);
```
- `fwnode`：指向struct fwnode_handle的指针，表示要获取GPIO的节点对象地址
- `propname`：属性名，指定要获取的**GPIO的属性名称**
- `index`：索引值，用于指定要获取的GPIO在属性中的**索引**
- `dflags`：获得到**GPIO后的初始化配置**
- `label`：标签，用于**标识GPIO的描述**

**GPIO初始化标志**：
- `GPIOD_ASIS`：不进行初始化
- `GPIOD_IN`：初始化为输入模式
- `GPIOD_OUT_LOW`：初始化为输出模式，输出低电平
- `GPIOD_OUT_HIGH`：初始化为输出模式，输出高电平

### 7.2 实验：多级节点

设备树配置示例：
```dts
my_gpio:gpio1_a0 {
    compatible = "mygpio";
    
    /* 第一个子节点：控制两个LED */
    led1 {
        my-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>, 
                   <&gpio1 RK_PB1 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&mygpio_ctrl>;
    };
    
    /* 第二个子节点：控制一个LED */
    led2 {
        my-gpios = <&gpio1 RK_PB0 GPIO_ACTIVE_HIGH>;
    };
};
```

驱动代码实现：
```c fold
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/gpio/consumer.h>
#include <linux/gpio.h>

/* 设备结构体 */
struct multi_gpio_dev {
    unsigned int count;                    /* 子节点数量 */
    struct gpio_desc *led[4];             /* GPIO描述符数组 */
};

/* 平台设备初始化函数 */
static int my_platform_probe(struct platform_device *pdev)
{
    struct multi_gpio_dev *mgd;
    struct fwnode_handle *child = NULL;
    int i = 0;
    int num;
    
    printk("This is my_platform_probe\n");
    
    /* 分配设备结构体 */
    mgd = devm_kzalloc(&pdev->dev, sizeof(*mgd), GFP_KERNEL);
    if (!mgd)
        return -ENOMEM;
    
    /* 获取父设备节点的子设备节点数量 */
    mgd->count = device_get_child_node_count(&pdev->dev);
    printk("Child node count is %d\n", mgd->count);
    
    /* 遍历所有子节点 */
    for (i = 0; i < mgd->count && i < 4; i++) {
        /* 获取下一个子设备节点 */
        child = device_get_next_child_node(&pdev->dev, child);
        
        if (child) {
            /* 从子设备节点中获取名为"my-gpios"的GPIO描述符
             * 这里获取第一个GPIO（index=0）！！！！index固定了
             */
            mgd->led[i] = fwnode_get_named_gpiod(child, "my-gpios", 
                                                  0, GPIOD_OUT_LOW, "LED");
            if (IS_ERR(mgd->led[i])) {
                dev_err(&pdev->dev, "Failed to get GPIO from child %d\n", i);
                continue;
            }
            
            /* 将GPIO描述符转换为GPIO编号并打印 */
            num = desc_to_gpio(mgd->led[i]);
            printk("Child %d: GPIO number is %d\n", i, num);
            
            /* 点亮LED测试 */
            gpiod_set_value(mgd->led[i], 1);
        }
    }
    
    platform_set_drvdata(pdev, mgd);
    return 0;
}

/* 平台设备的移除函数 */
static int my_platform_remove(struct platform_device *pdev)
{
    struct multi_gpio_dev *mgd = platform_get_drvdata(pdev);
    int i;
    
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");
    
    /* 关闭所有LED */
    for (i = 0; i < mgd->count && i < 4; i++) {
        if (mgd->led[i])
            gpiod_set_value(mgd->led[i], 0);
    }
    
    return 0;
}

/* 设备树匹配表 */
static const struct of_device_id of_match_table_id[] = {
    { .compatible = "mygpio", },
    { }
};
MODULE_DEVICE_TABLE(of, of_match_table_id);

/* 定义平台驱动结构体 */
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
        .of_match_table = of_match_table_id,
    },
};

/* 模块初始化和退出 */
module_platform_driver(my_platform_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Multi-level GPIO Node Demo");
```

加载驱动后的输出结果：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dcd36fb8aef54a9639506ee07abed088.png)

从输出可以看到，驱动成功识别了2个子节点，并获取了对应的GPIO编号（32和40）。

> [!tip]+ 实用技巧 
> 使用多级节点的好处是可以更灵活地组织硬件资源。比如一个LED控制器下可以有多组LED，每组LED有不同的控制逻辑。这种设计让设备树更清晰，驱动代码更模块化。

设备树是描述硬件的标准方式，GPIO的配置自然也离不开设备树。

## 8 GPIO和设备树

### 8.1 GPIO设备树属性命名规范

|**命名格式**|**语法**|**适用场景**|**示例**|
|---|---|---|---|
|**推荐格式**|`<name>-gpios`|多个GPIO，有明确功能名称|`led-gpios`, `reset-gpios`|
|**兼容格式**|`<name>-gpio`|单个GPIO，向后兼容|`reset-gpio`, `enable-gpio`|
|**通用格式**|`gpios`|无特定功能名称的GPIO数组|`gpios`|

**设备树示例**
```dts
gpio_demo {
    /* 推荐格式：多个LED控制GPIO */
    led-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>,
                <&gpio1 RK_PB1 GPIO_ACTIVE_HIGH>;
    
    /* 兼容格式：单个复位GPIO */
    reset-gpio = <&gpio0 RK_PB0 GPIO_ACTIVE_LOW>;
    
    /* 通用格式：GPIO数组 */
    gpios = <&gpio1 2 0>, <&gpio1 5 0>;
};
```

### 8.2 GPIO设备树解析API

相关API参考
- [4 基于描述符的GPIO接口详解](#4%20基于描述符的GPIO接口详解)
- [5 基于整数的传统GPIO接口](#5%20基于整数的传统GPIO接口)
- [2 核心OF GPIO函数详解](../../08-🏗️%20Linux设备模型与设备树(重点)/3_设备树基础/16_OF函数获取GPIO及源码分析.md#2%20核心OF%20GPIO函数详解)

**1、新API（基于GPIO描述符）**：通过device来获取，因为device内置of_node（设备节点）
- **gpiod_get**：获取命名的GPIO描述符
	- `gpiod_get(dev, "led", GPIOD_OUT_LOW)` 查找 `led-gpios` 属性
- **gpiod_get_index**：获取指定索引的GPIO描述符
	- `gpiod_get_index(dev, "led", 1, GPIOD_OUT_LOW)` 获取 `led-gpios` 的第2个GPIO
	- `gpiod_get_index(dev, NULL, 0, GPIOD_IN)` 获取 `gpios` 的第1个GPIO
- **gpiod_get_optional**：获取可选的GPIO描述符
	- **使用场景**：适用于可选功能的GPIO，不存在时不报错
- **fwnode_get_named_gpiod**：从固件节点获取命名GPIO
	- **使用场景**：直接从子节点获取GPIO，适用于遍历设备树子节点

**2、旧API（基于GPIO编号）**
- **of_get_named_gpio**：从设备树获取命名GPIO编号
- **of_get_gpio**：获取通用gpio属性
- **of_gpio_named_count**：获取命名GPIO数量
- **of_gpio_count**：获取通用GPIO数量

**3、GPIO转中断API**
- **gpiod_to_irq**：GPIO描述符转中断号
- **gpio_to_irq**：GPIO编号转中断号


### 8.3 完整的设备树配置示例

**1、Pinctrl配置**
```dts
&pinctrl {
    gpio_demo_pins: gpio-demo-pins {
        led_pins: led-pins {
            rockchip,pins = <1 RK_PA0 RK_FUNC_GPIO &pcfg_pull_none>,
                           <1 RK_PB1 RK_FUNC_GPIO &pcfg_pull_none>;
        };
        
        button_pins: button-pins {
            rockchip,pins = <1 RK_PB2 RK_FUNC_GPIO &pcfg_pull_up>;
        };
        
        reset_pin: reset-pin {
            rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
};
```

**2、GPIO设备节点**
```dts
gpio_demo: gpio-demo {
    compatible = "my,gpio-demo";
    pinctrl-names = "default";
    pinctrl-0 = <&gpio_demo_pins>;
    
    /* 推荐格式：多个LED */
    led-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>,
                <&gpio1 RK_PB1 GPIO_ACTIVE_HIGH>;
    
    /* 兼容格式：复位GPIO */
    reset-gpio = <&gpio0 RK_PB0 GPIO_ACTIVE_LOW>;
    
    /* 按键GPIO（可选） */
    button-gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_LOW>;
    
    /* 通用GPIO数组 */
    gpios = <&gpio1 2 0>, <&gpio1 5 0>;
    
    status = "okay";
};
```

**3、GPIO中断配置**
```dts
spi_device: spi-device@0 {
    compatible = "my,spi-device";
    reg = <0>;
    spi-max-frequency = <1000000>;
    
    /* 方式1：直接指定中断 */
    interrupt-parent = <&gpio4>;
    interrupts = <29 IRQ_TYPE_LEVEL_LOW>;
    
    /* 方式2：使用GPIO属性，驱动中转换为中断 */
    irq-gpios = <&gpio4 29 GPIO_ACTIVE_LOW>;
    
    status = "okay";
};
```

### 8.4 驱动中的使用示例

**1、新API使用**
```c
/* 获取LED GPIO */
struct gpio_desc *led1 = gpiod_get_index(dev, "led", 0, GPIOD_OUT_LOW);
struct gpio_desc *led2 = gpiod_get_index(dev, "led", 1, GPIOD_OUT_LOW);

/* 获取可选的按键GPIO */
struct gpio_desc *button = gpiod_get_optional(dev, "button", GPIOD_IN);

/* 获取中断 */
int irq = gpiod_to_irq(button);
```

**2、旧API使用**
```c
struct device_node *np = dev->of_node;

/* 获取GPIO编号 */
int led_gpio = of_get_named_gpio(np, "led-gpios", 0);
int reset_gpio = of_get_named_gpio(np, "reset-gpio", 0);

/* 获取GPIO数量 */
int led_count = of_gpio_named_count(np, "led-gpios");

/* 申请和使用 */
gpio_request(led_gpio, "led");
gpio_direction_output(led_gpio, 1);
```

## 9 GPIO编程最佳实践

### 9.1 错误处理

良好的错误处理是健壮驱动的基础：
```c
struct gpio_desc *gpio;

/* 获取GPIO时的错误处理 */
gpio = devm_gpiod_get(dev, "reset", GPIOD_OUT_HIGH);
if (IS_ERR(gpio)) {
    /* 区分不同的错误类型 */
    if (PTR_ERR(gpio) == -EPROBE_DEFER) {
        /* GPIO控制器还未就绪，稍后重试 */
        return -EPROBE_DEFER;
    }
    
    /* 其他错误，打印具体信息 */
    dev_err(dev, "Failed to get reset GPIO: %ld\n", PTR_ERR(gpio));
    return PTR_ERR(gpio);
}
```

### 9.2 资源管理

推荐使用devm_系列函数自动管理资源：
```c
/* 推荐：使用devm_函数，资源自动管理 */
gpio = devm_gpiod_get(dev, "led", GPIOD_OUT_LOW);

/* 不推荐：需要手动释放 */
gpio = gpiod_get(dev, "led", GPIOD_OUT_LOW);
/* 在错误路径和remove函数中需要： */
gpiod_put(gpio);
```

### 9.3 中断处理中的GPIO操作

在中断上下文中操作GPIO需要特别注意：
```c
/* 检查GPIO操作是否可能睡眠 */
if (gpiod_cansleep(gpio)) {
    /* 不能在中断处理函数中直接使用 */
    /* 需要使用工作队列或线程化中断 */
    schedule_work(&my_work);
} else {
    /* 可以直接在中断处理函数中使用 */
    gpiod_set_value(gpio, value);
}
```

> [!warning]+ 重要提醒 
> 通过I2C或SPI扩展的GPIO控制器，其操作可能会睡眠（因为总线访问可能睡眠）。在中断处理函数中操作这类GPIO会导致系统崩溃。一定要使用`gpiod_cansleep()`检查。

## 10 本章小结

通过本章的学习，我们掌握了GPIO编程的核心知识：
1. **系统架构**：理解了GPIO子系统的分层设计和各层的职责
2. **数据结构**：熟悉了gpio_device、gpio_chip、gpio_desc三个核心结构体
3. **编程接口**：重点学习了基于描述符的新接口，了解了传统接口
4. **设备树配置**：掌握了GPIO在设备树中的配置方法
5. **最佳实践**：学会了错误处理和资源管理的技巧

GPIO编程看似简单，但要写出健壮的代码需要注意很多细节。记住以下几个要点：
- 优先使用基于描述符的新接口
- 使用devm_函数自动管理资源
- 注意GPIO操作是否可能睡眠
- 做好错误处理，特别是-EPROBE_DEFER

在下一章中，我们将学习GPIO的调试方法，包括如何使用sysfs和debugfs接口调试GPIO，以及常见问题的解决方法。这些调试技能在实际开发中非常有用，能帮助你快速定位和解决问题。