
## 1 引言

在前面三章中，我们已经掌握了GPIO的基本概念、编程接口和调试技巧。现在让我们深入到"引擎盖下面"，了解GPIO控制器驱动是如何实现的。

可以把GPIO控制器驱动比作汽车的发动机。用户通过方向盘、油门、刹车（GPIO API）来控制汽车，但真正让汽车动起来的是发动机（GPIO控制器驱动）。理解驱动的实现原理，不仅能让你更好地使用GPIO，还能帮你在遇到问题时快速定位根因。

虽然GPIO控制器驱动通常由芯片厂商提供，但作为Linux驱动开发者，理解其设计思想和实现方法是非常重要的。这不仅有助于调试问题，还能让你在特殊场景下进行定制化开发。

## 2 GPIO控制器驱动的核心架构

### 2.1 在系统中的位置

GPIO控制器驱动位于GPIO子系统的最底层，它的主要职责是：
```txt
用户空间应用程序
        ↓
   GPIO子系统核心层(gpiolib)  
        ↓
   GPIO控制器驱动 ← 本章重点
        ↓
   硬件寄存器操作
        ↓
   GPIO硬件控制器
```

GPIO控制器驱动的核心职责包括：
- **硬件抽象**：将复杂的硬件寄存器操作抽象成标准接口
- **资源管理**：管理GPIO引脚的分配和释放
- **功能实现**：提供GPIO的基本操作（输入、输出、电平和中断等）
- **平台适配**：适配不同芯片的硬件特性

### 2.2 核心数据结构详解

GPIO控制器驱动的核心是`gpio_chip`结构体，它定义了**所有GPIO操作的函数指针**。这就像定义了一套"标准化的工具接口"，不管底层硬件如何变化，上层都能用统一的方式操作。

```c fold
// 内核源码/include/linux/gpio/driver.h
struct gpio_chip {
    const char                *label;       // GPIO控制器的标签名称
    struct gpio_device        *gpiodev;     // 关联的gpio_device
    struct device             *parent;      // 父设备指针
    struct module             *owner;       // 所属模块

    /* GPIO基本操作函数指针 */
    
    /* GPIO请求和释放 - 生命周期管理 */
    int (*request)(struct gpio_chip *chip, unsigned offset);
    void (*free)(struct gpio_chip *chip, unsigned offset);
    
    /* GPIO方向控制 - 配置输入输出 */
    int (*get_direction)(struct gpio_chip *chip, unsigned offset);
    int (*direction_input)(struct gpio_chip *chip, unsigned offset);
    int (*direction_output)(struct gpio_chip *chip, unsigned offset, int value);
    
    /* GPIO电平操作 - 读写数据 */
    int (*get)(struct gpio_chip *chip, unsigned offset);
    void (*set)(struct gpio_chip *chip, unsigned offset, int value);
    int (*get_multiple)(struct gpio_chip *chip, unsigned long *mask, unsigned long *bits);
    void (*set_multiple)(struct gpio_chip *chip, unsigned long *mask, unsigned long *bits);
    
    /* GPIO配置功能 - 高级特性 */
    int (*set_config)(struct gpio_chip *chip, unsigned offset, unsigned long config);
    
    /* GPIO中断转换 - 中断支持 */
    int (*to_irq)(struct gpio_chip *chip, unsigned offset);
    
    /* GPIO控制器属性 */
    int                       base;         // GPIO编号基址
    u16                       ngpio;        // GPIO数量
    const char                *const *names;// GPIO名称数组
    bool                      can_sleep;    // 操作是否可能睡眠
    
    /* 中断支持相关（可选） */
#ifdef CONFIG_GPIOLIB_IRQCHIP
    struct irq_chip           *irqchip;      // 中断芯片
    struct irq_domain         *irqdomain;   // 中断域
    unsigned int              irq_base;     // 中断基址
    irq_flow_handler_t        irq_handler;  // 中断处理流程
    unsigned int              irq_default_type; // 默认中断类型
#endif
};
```

每个函数指针都有特定的作用：

**request函数**：当用户申请GPIO时调用，用于执行芯片特定的初始化
```c
// 典型实现：调用通用的pinctrl处理
int my_gpio_request(struct gpio_chip *chip, unsigned offset)
{
    return gpiochip_generic_request(chip, offset);
}
```

**get/set函数**：GPIO的核心功能，读取和设置电平
```c
// 读取GPIO电平
int my_gpio_get(struct gpio_chip *chip, unsigned offset)
{
    struct my_gpio_bank *bank = gpiochip_get_data(chip);
    u32 val = readl(bank->reg_base + GPIO_DATA_REG);
    return (val >> offset) & 1;
}

// 设置GPIO电平
void my_gpio_set(struct gpio_chip *chip, unsigned offset, int value)
{
    struct my_gpio_bank *bank = gpiochip_get_data(chip);
    u32 val = readl(bank->reg_base + GPIO_DATA_REG);
    
    if (value)
        val |= (1 << offset);   // 置位
    else
        val &= ~(1 << offset);  // 清位
        
    writel(val, bank->reg_base + GPIO_DATA_REG);
}
```


## 3 Rockchip GPIO控制器驱动实例

让我们通过Rockchip RK3568的GPIO控制器驱动来理解真实的实现。这是一个产品级的驱动程序，包含了从平台驱动注册到GPIO控制器注册的完整流程

后续使用到的结构体和API可以参考：[2.3 GPIOLIB向下提供的接口（生产者-gpio控制器）](2_GPIO编程接口与实践.md#2.3%20GPIOLIB向下提供的接口（生产者-gpio控制器）)

### 3.1 设备树的定义

```c
// arch/arm64/boot/dts/rockchip/rk3568.dtsi
gpio0: gpio@fdd60000 {
    compatible = "rockchip,gpio-bank";        // 兼容性字符串，用于驱动匹配
    reg = <0x0 0xfdd60000 0x0 0x100>;        // 寄存器基地址和大小
    interrupts = <GIC_SPI 33 IRQ_TYPE_LEVEL_HIGH>;  // 中断配置
    clocks = <&pmucru PCLK_GPIO0>, <&pmucru DBCLK_GPIO0>;  // 时钟配置

    gpio-controller;                          // 标识这是一个GPIO控制器
    #gpio-cells = <2>;                       // GPIO引用需要2个参数（引脚号和标志）
    gpio-ranges = <&pinctrl 0 0 32>;         // GPIO范围映射到pinctrl
    interrupt-controller;                     // 标识这是一个中断控制器
    #interrupt-cells = <2>;                  // 中断引用需要2个参数
};
```

### 3.2 平台驱动注册与匹配机制

在深入了解GPIO操作函数之前，我们需要理解GPIO控制器驱动是如何被加载和初始化的。这个过程涉及Linux平台驱动框架的完整流程。

**平台驱动注册**：
```c
// drivers/gpio/gpio-rockchip.c
static const struct of_device_id rockchip_gpio_match[] = {
    { .compatible = "rockchip,gpio-bank", },      // 匹配设备树compatible
    { .compatible = "rockchip,rk3188-gpio-bank0" }, // 向后兼容性支持
    { },  // 数组结束标记
};
MODULE_DEVICE_TABLE(of, rockchip_gpio_match);

static struct platform_driver rockchip_gpio_driver = {
    .probe = rockchip_gpio_probe,        // 设备发现时的处理函数
    .remove = rockchip_gpio_remove,      // 设备移除时的处理函数
    .driver = {
        .name = "rockchip-gpio",         // 驱动名称
        .of_match_table = rockchip_gpio_match,  // 设备匹配表
    },
};

// 驱动注册：在内核启动早期阶段调用
static int __init rockchip_gpio_init(void)
{
    return platform_driver_register(&rockchip_gpio_driver);
}
postcore_initcall(rockchip_gpio_init);  // 比普通驱动更早加载
```

**设备树与驱动匹配过程**：
1. **内核启动时**：`postcore_initcall`确保GPIO驱动在系统早期就被注册
2. **设备树解析时**：内核发现compatible为"rockchip,gpio-bank"的节点
3. **驱动匹配时**：平台驱动框架将设备与驱动进行匹配
4. **probe调用时**：匹配成功后调用`rockchip_gpio_probe`函数

这里`postcore_initcall`的使用很重要，因为GPIO控制器是基础设施，其他驱动可能依赖它，所以需要早期加载

### 3.3 rockchip_gpio_probe函数详解

**rockchip_gpio_probe**函数是驱动初始化的核心，它建立了**从平台驱动到GPIO控制器**的完整桥梁：
- 本质：就是把DT上的gpio bank节点中的所有内容，先转换为对应的pdev设备，然后用pdev的资源构建rockchip_pin_bank结构体
```c fold
// drivers/gpio/gpio-rockchip.c（简化版本）
static int rockchip_gpio_probe(struct platform_device *pdev)
{
    struct device *dev = &pdev->dev;
    struct device_node *np = dev->of_node;
    struct rockchip_pin_bank *bank;
    struct resource *res;
    int ret;

    /* 第一步：分配并初始化GPIO bank结构 */
    bank = devm_kzalloc(dev, sizeof(*bank), GFP_KERNEL);
    if (!bank)
        return -ENOMEM;

    /* 第二步：解析设备树获取硬件信息 */
    bank->dev = dev;
    bank->of_node = np;
    
    /* 获取寄存器基地址 */
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    bank->reg_base = devm_ioremap_resource(dev, res);
    if (IS_ERR(bank->reg_base))
        return PTR_ERR(bank->reg_base);

    /* 获取中断号 */
    bank->irq = platform_get_irq(pdev, 0);
    if (bank->irq < 0)
        return bank->irq;

    /* 获取时钟资源 */
    bank->clk = devm_clk_get(dev, "pclk");
    if (!IS_ERR(bank->clk))
        clk_prepare_enable(bank->clk);

    /* 第三步：解析GPIO特定属性 */
    ret = of_property_read_u32(np, "gpio,nr-gpios", &bank->nr_pins);
    if (ret) {
        dev_err(dev, "failed to get gpio,nr-gpios\n");
        return ret;
    }

    /* 第四步：设置GPIO寄存器配置 */
    bank->gpio_regs = &rk3568_gpio_regs;  // 芯片特定的寄存器布局

    /* 第五步：初始化自旋锁 */
    raw_spin_lock_init(&bank->slock);

    /* 第六步：关键调用 - 注册GPIO控制器 */
    ret = rockchip_gpiolib_register(bank);
    if (ret) {
        dev_err(dev, "Failed to register gpio controller\n");
        return ret;
    }

    /* 第七步：保存私有数据供后续使用 */
    platform_set_drvdata(pdev, bank);

    dev_info(dev, "GPIO controller registered successfully\n");
    return 0;
}
```

这个函数展示了典型的平台驱动probe函数的实现模式：**资源获取 → 硬件初始化 → 功能注册 → 状态保存**。

### 3.4 rockchip_gpiolib_register函数：连接的关键

**rockchip_gpiolib_register**函数是连接平台驱动和GPIO子系统的关键桥梁。这个函数将hardware-specific的GPIO控制器转换为Linux GPIO子系统可以识别的标准接口：
- 本质：将bank的内容给标准的GPIO子系统定义的结构体 gpio_chip
```c fold
// drivers/gpio/gpio-rockchip.c
static int rockchip_gpiolib_register(struct rockchip_pin_bank *bank)
{
    struct gpio_chip *gc;
    int ret;

    /* 第一步：准备gpio_chip结构 */
    bank->gpio_chip = rockchip_gpiolib_chip;  // 复制标准操作集
    
    gc = &bank->gpio_chip;
    gc->base = bank->pin_base;               // GPIO编号起始值
    gc->ngpio = bank->nr_pins;               // GPIO引脚数量
    gc->label = bank->name;                  // GPIO控制器名称
    gc->parent = bank->dev;                  // 父设备指针

#ifdef CONFIG_OF_GPIO
    gc->of_node = of_node_get(bank->of_node);  // 关联设备树节点
#endif

    /* 第二步：关键调用 - 注册到GPIO子系统 */
    ret = gpiochip_add_data(gc, bank);
    if (ret) {
        dev_err(bank->dev, "failed to add gpiochip %s, %d\n", gc->label, ret);
        return ret;
    }

    /* 第三步：处理GPIO引脚范围映射（与Pinctrl的关联） */
    if (!of_property_read_bool(bank->of_node, "gpio-ranges")) {
        struct device_node *pctlnp = of_get_parent(bank->of_node);
        struct pinctrl_dev *pctldev = NULL;

        if (!pctlnp)
            return -ENODATA;

        pctldev = of_pinctrl_get(pctlnp);
        if (!pctldev)
            return -ENODEV;

        /* 手动添加GPIO到pinctrl的映射关系 */
        ret = gpiochip_add_pin_range(gc, dev_name(pctldev->dev), 
                                     0, gc->base, gc->ngpio);
        if (ret) {
            dev_err(bank->dev, "Failed to add pin range\n");
            goto fail;
        }
    }

    /* 第四步：注册中断支持 */
    ret = rockchip_interrupts_register(bank);
    if (ret) {
        dev_err(bank->dev, "failed to register interrupt, %d\n", ret);
        goto fail;
    }

    return 0;

fail:
    gpiochip_remove(&bank->gpio_chip);
    return ret;
}
```

### 3.5 完整的初始化流程图

现在我们可以描绘出完整的初始化流程：
```txt
系统启动
    ↓
postcore_initcall(rockchip_gpio_init)
    ↓
platform_driver_register(&rockchip_gpio_driver)
    ↓
内核发现设备树中compatible="rockchip,gpio-bank"的节点
    ↓
平台驱动框架调用rockchip_gpio_probe()：获得对应的pdev，将pdev资源给rockchip_pin_bank
    ↓
调用rockchip_gpiolib_register()：将rockchip_pin_bank资源给gpio_chip
    ↓
调用gpiochip_add_data() ：把gpio_chip注册到GPIO子系统
    ↓
建立与Pinctrl的映射关系
    ↓
注册中断支持
    ↓
GPIO控制器可以正常使用
```

这个流程解释了为什么用户在调用`gpiod_get`时能够成功获取GPIO：因为在系统启动早期，所有的GPIO控制器都已经通过这个流程注册到了GPIO子系统中。

### 3.6 GPIO操作函数集定义

在GPIO控制器成功注册后，用户就可以通过标准的GPIO API来操作硬件了。Rockchip定义了完整的GPIO操作函数集：
```c
// drivers/gpio/gpio-rockchip.c
static const struct gpio_chip rockchip_gpiolib_chip = {
    .request = gpiochip_generic_request,         // 使用通用的请求处理
    .free = gpiochip_generic_free,               // 使用通用的释放处理
    .set = rockchip_gpio_set,                    // 自定义的设置函数
    .get = rockchip_gpio_get,                    // 自定义的读取函数
    .get_direction = rockchip_gpio_get_direction, // 获取方向
    .direction_input = rockchip_gpio_direction_input,   // 设置输入
    .direction_output = rockchip_gpio_direction_output, // 设置输出
    .set_config = rockchip_gpio_set_config,      // 配置功能（如去抖等）
    .to_irq = rockchip_gpio_to_irq,             // GPIO转中断号
    .owner = THIS_MODULE,                        // 模块所有者
};
```

注意这里有些函数使用了通用实现（如`gpiochip_generic_request`），有些使用了自定义实现。这体现了Linux内核的设计哲学：**在通用性和特异性之间找到平衡**。

**rockchip_gpio_set**：设置GPIO输出值的实现
```c
static void rockchip_gpio_set(struct gpio_chip *gc, unsigned int offset, int value)
{
    struct rockchip_pin_bank *bank = gpiochip_get_data(gc);  // 获取私有数据
    unsigned long flags;

    // 使用自旋锁保护寄存器访问，确保原子性
    raw_spin_lock_irqsave(&bank->slock, flags);
    
    // 调用寄存器写入函数，设置指定位的值
    rockchip_gpio_writel_bit(bank, offset, value, bank->gpio_regs->port_dr);
    
    raw_spin_unlock_irqrestore(&bank->slock, flags);
}
```

**rockchip_gpio_get**：读取GPIO输入值的实现
```c
static int rockchip_gpio_get(struct gpio_chip *gc, unsigned int offset)
{
    struct rockchip_pin_bank *bank = gpiochip_get_data(gc);
    u32 data;

    // 从外部端口寄存器读取所有GPIO的状态
    data = readl(bank->reg_base + bank->gpio_regs->ext_port);
    
    // 提取指定offset位的值
    data >>= offset;
    data &= 1;      // 只保留最低位

    return data;    // 返回0或1
}
```

这些实现的设计细节值得注意：
- **使用自旋锁**：保证在多核系统中的并发安全
- **禁用中断**：在操作期间禁用中断，确保操作的原子性
- **读取外部端口寄存器**：反映引脚的实际电平状态，而不仅仅是输出寄存器的值

通过这个完整的分析，我们现在明白了从`platform_driver_register`到`rockchip_gpiolib_register`的完整链条：平台驱动框架负责设备发现和匹配，probe函数负责硬件初始化，而`rockchip_gpiolib_register`则负责将硬件抽象为Linux GPIO子系统可以使用的标准接口。

## 4 本章小结

通过本章的学习，我们深入了解了GPIO控制器驱动的实现原理。虽然大部分开发者不需要编写GPIO控制器驱动（通常由芯片厂商提供），但理解其工作原理对于以下方面非常有帮助：

**问题诊断**：当GPIO不工作时，能够从驱动层面分析可能的原因。比如理解为什么有时会返回-EPROBE_DEFER错误，这通常意味着依赖的Pinctrl驱动还未加载完成。

**定制开发**：在特殊场景下，可能需要对GPIO驱动进行定制化修改，理解标准实现是定制的基础。

下一章我们将学习GPIO与Pinctrl子系统的深度交互机制，这是理解现代SoC引脚管理的关键知识。