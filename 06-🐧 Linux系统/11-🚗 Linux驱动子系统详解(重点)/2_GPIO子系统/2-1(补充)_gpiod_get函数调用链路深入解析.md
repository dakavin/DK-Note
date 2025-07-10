# Linux内核GPIO驱动gpiod_get完整分析

## 1 gpiod_get函数的核心作用

`gpiod_get`是Linux内核GPIO子系统的核心接口函数，它的主要作用是：

**核心功能**：为设备驱动程序提供统一的GPIO资源获取接口，屏蔽底层硬件差异和不同平台的配置方式。

**核心作用联系**：

- **资源定位**：通过设备树、ACPI或平台查找表定位GPIO资源
- **引脚配置**：自动调用pinctrl子系统将引脚配置为GPIO功能，处理引脚复用
- **资源申请**：确保GPIO独占使用，防止资源冲突
- **硬件抽象**：将底层硬件GPIO转换为内核统一的gpio_desc描述符
- **配置管理**：根据用户需求配置GPIO方向、初始状态等属性
- **生命周期管理**：通过引用计数确保模块和设备在GPIO使用期间不被卸载

当我们调用`gpiod_get`函数时，内核内部发生了一系列复杂的操作。就像剥洋葱一样，让我们一层层地分析这个过程，理解每个环节的作用和设计思想。

## 2 完整的调用流程

首先看看完整的调用链路：

```c
// 用户调用
struct gpio_desc *desc = gpiod_get(&pdev->dev, "reset", GPIOD_OUT_HIGH);

// 完整的调用链路
gpiod_get(&pdev->dev, "reset", GPIOD_OUT_HIGH)
  └── gpiod_get_index(&pdev->dev, "reset", 0, GPIOD_OUT_HIGH)
      ├── of_find_gpio(dev, "reset", 0, &lookupflags)       // 设备树查找
      │   └── of_get_named_gpiod_flags(dev->of_node, "reset", 0, &flags)
      ├── gpiod_find(dev, "reset", 0, &lookupflags)         // 平台查找表
      │   └── gpiochip_get_desc(chip, p->chip_hwnum)        // 获取GPIO描述符
      ├── gpiod_request(desc, "reset")                       // 申请GPIO资源
      │   └── gpiod_request_commit(desc, "reset")
      │       ├── pinctrl_gpio_request(desc_to_gpio(desc))   // 自动pinctrl配置！！！
      │       │   └── pinmux_request_gpio(pctldev, range, pin, gpio)
      │       │       └── ops->gpio_request_enable(pctldev, range, pin)  // 芯片pinctrl配置
      │       │           └── [SoC特定实现：配置引脚为GPIO模式]
      │       ├── chip->request(chip, offset)                // 调用芯片特定GPIO请求函数
      │       └── gpiod_get_direction(desc)                  // 获取GPIO方向
      └── gpiod_configure_flags(desc, "reset", lookupflags, GPIOD_OUT_HIGH)
          ├── gpiod_direction_output(desc, value)            // 配置为输出方向
          │   └── gpiod_direction_output_raw(desc, value)
          │       └── chip->direction_output(chip, offset, value)  // 芯片特定方向设置
          └── gpiod_set_value(desc, value)                   // 设置初始值
```

让我们逐个分析关键函数的实现

## 3 gpiod_get函数：实际调用

**函数作用**：GPIO获取的入口函数，获取索引为0的GPIO描述符  
**调用流程**：gpiod_get → gpiod_get_index（索引=0）

```c
// drivers/gpio/gpiolib.c
struct gpio_desc *__must_check gpiod_get(struct device *dev, const char *con_id,
					 enum gpiod_flags flags)
{
	// 调用gpiod_get_index函数，索引设为0（第一个GPIO）
	return gpiod_get_index(dev, con_id, 0, flags);
}
// 导出符号，允许模块使用此函数（GPL许可证）
EXPORT_SYMBOL_GPL(gpiod_get);
```

## 4 gpiod_get_index函数：核心查找与配置

**函数作用**：通过多种机制查找指定索引的GPIO并完成配置  
**调用流程**：设备树查找 → ACPI查找 → 平台查找表 → 资源申请 → 标志配置

```c fold
struct gpio_desc *__must_check gpiod_get_index(struct device *dev,
					       const char *con_id,
					       unsigned int idx,
					       enum gpiod_flags flags)
{
	struct gpio_desc *desc = NULL;      // GPIO描述符指针
	int status;                         // 状态码
	enum gpio_lookup_flags lookupflags = 0;  // 查找标志
	
	/* 获取设备名，如果没有设备则使用"?" */
	const char *devname = dev ? dev_name(dev) : "?";
	
	dev_dbg(dev, "GPIO lookup for consumer %s\n", con_id);
	
	if (dev) {
		/* 使用设备树查找GPIO? */
		if (IS_ENABLED(CONFIG_OF) && dev->of_node) {
			dev_dbg(dev, "using device tree for GPIO lookup\n");
			desc = of_find_gpio(dev, con_id, idx, &lookupflags);  // 设备树查找
		} else if (ACPI_COMPANION(dev)) {
			/* 使用ACPI查找GPIO */
			dev_dbg(dev, "using ACPI for GPIO lookup\n");
			desc = acpi_find_gpio(dev, con_id, idx, &flags, &lookupflags);  // ACPI查找
		}
	}
	
	/*
	 * 如果没有使用DT或ACPI，或者查找没有返回结果
	 * 使用平台查找表作为fallback
	 */
	if (!desc || desc == ERR_PTR(-ENOENT)) {
		dev_dbg(dev, "using lookup tables for GPIO lookup\n");
		desc = gpiod_find(dev, con_id, idx, &lookupflags);  // 平台查找表查找
	}
	
	/* 如果查找失败，返回错误 */
	if (IS_ERR(desc)) {
		dev_dbg(dev, "No GPIO consumer %s found\n", con_id);
		return desc;
	}
	
	/*
	 * 如果传递了连接标签则使用它，否则尝试使用设备名作为标签
	 */
	status = gpiod_request(desc, con_id ? con_id : devname);  // 申请GPIO资源
	if (status < 0)
		return ERR_PTR(status);
	
	/* 配置GPIO标志 */
	status = gpiod_configure_flags(desc, con_id, lookupflags, flags);  // 配置GPIO属性
	if (status < 0) {
		dev_dbg(dev, "setup of GPIO %s failed\n", con_id);
		gpiod_put(desc);  // 配置失败时释放GPIO资源
		return ERR_PTR(status);
	}
	
	return desc;  // 返回配置好的GPIO描述符
}
// 导出符号，允许GPL模块使用
EXPORT_SYMBOL_GPL(gpiod_get_index);
```

## 5 平台查找机制
### 5.1 gpiod_find函数：平台查找表机制

**函数作用**：在平台GPIO查找表中搜索匹配的GPIO条目，作为设备树的备用机制  
**调用流程**：查找查找表 → 遍历条目匹配 → 查找GPIO芯片 → 获取描述符

```c fold
// drivers/gpio/gpiolib.c
static struct gpio_desc *gpiod_find(struct device *dev, const char *con_id,
                                    unsigned int idx,
                                    enum gpio_lookup_flags *flags)
{
    struct gpio_desc *desc = ERR_PTR(-ENOENT);  // 默认返回"未找到"错误
    struct gpiod_lookup_table *table;           // GPIO查找表指针
    struct gpiod_lookup *p;                     // 查找条目指针
    
    /* 查找设备对应的GPIO查找表 */
    table = gpiod_find_lookup_table(dev);
    if (!table)
        return desc;  // 没有找到查找表
    
    /* 遍历查找表中的所有条目 */
    for (p = &table->table[0]; p->chip_label; p++) {
        struct gpio_chip *chip;  // GPIO芯片指针
        
        /* 索引必须完全匹配 */
        if (p->idx != idx)
            continue;
        
        /* 如果查找条目有con_id，要求精确匹配 */
        if (p->con_id && (!con_id || strcmp(p->con_id, con_id)))
            continue;
        
        /* 根据芯片标签查找GPIO芯片 */
        chip = find_chip_by_name(p->chip_label);
        if (!chip) {
            /* 芯片可能稍后出现，返回延迟探测错误 */
            dev_warn(dev, "cannot find GPIO chip %s, deferring\n",
                     p->chip_label);
            return ERR_PTR(-EPROBE_DEFER);
        }
        
        /* 检查请求的GPIO引脚号是否在有效范围内 */
        if (chip->ngpio <= p->chip_hwnum) {
            dev_err(dev, "requested GPIO %u is out of range [0..%u] for chip %s\n",
                    p->chip_hwnum, chip->ngpio - 1, chip->label);
            return ERR_PTR(-EINVAL);
        }
        
        /* 获取GPIO描述符并设置标志 */
        desc = gpiochip_get_desc(chip, p->chip_hwnum);  // 转换为GPIO描述符
        *flags = p->flags;  // 设置查找标志
        return desc;
    }
    
    return desc;  // 未找到匹配条目，返回-ENOENT
}
```

这个函数展示了Linux内核的一个重要设计原则：**提供多种机制来适应不同的硬件平台**。虽然现代系统主要使用设备树，但平台查找表为老旧平台提供了兼容性支持。

### 5.2 gpiochip_get_desc函数：硬件号转换

**函数作用**：根据硬件GPIO编号从指定芯片获取对应的GPIO描述符  
**调用流程**：验证硬件编号 → 返回描述符数组中的对应项

```c
// drivers/gpio/gpiolib.c
struct gpio_desc *gpiochip_get_desc(struct gpio_chip *chip, u16 hwnum)
{
    struct gpio_device *gdev = chip->gpiodev;  // 获取GPIO设备
    
    /* 检查硬件编号是否超出芯片GPIO数量范围 */
    if (hwnum >= gdev->ngpio)
        return ERR_PTR(-EINVAL);  // 硬件编号无效
    
    /* 返回对应硬件编号的GPIO描述符 */
    return &gdev->descs[hwnum];  // 从描述符数组获取
}
```

这个函数虽然简单，但体现了一个关键的转换过程：**从芯片内部的硬件编号转换为系统中的GPIO描述符**。每个GPIO控制器内部都有自己的编号体系（0到ngpio-1），而GPIO描述符则是系统级的抽象。

## 6 gpiod_request函数：申请GPIO资源

**函数作用**：请求GPIO描述符并进行资源管理，防止模块在使用期间被卸载  
**调用流程**：获取模块引用 → 提交请求 → 增加设备引用计数

```c
// drivers/gpio/gpiolib.c
int gpiod_request(struct gpio_desc *desc, const char *label)
{
    int status = -EPROBE_DEFER;  // 默认状态为延迟探测
    struct gpio_device *gdev;   // GPIO设备指针
    
    VALIDATE_DESC(desc);  // 验证GPIO描述符有效性
    gdev = desc->gdev;    // 获取GPIO设备
    
    /* 尝试获取模块引用，防止模块在使用期间被卸载 */
    if (try_module_get(gdev->owner)) {
        /* 提交GPIO请求操作 */
        status = gpiod_request_commit(desc, label);  // 实际执行请求
        if (status < 0)
            module_put(gdev->owner);  // 请求失败时释放模块引用
        else
            get_device(&gdev->dev);   // 请求成功时增加设备引用计数
    }
    
    /* 如果有错误，打印调试信息 */
    if (status)
        gpiod_dbg(desc, "%s: status %d\n", __func__, status);
    
    return status;  // 返回操作状态
}
```

这个函数体现了Linux内核严谨的资源管理机制：通过模块引用计数和设备引用计数，确保在GPIO使用期间相关的模块和设备不会被意外卸载。

## 7 gpiod_request_commit函数：核心请求实现

**函数作用**：实际执行GPIO请求的核心逻辑，包括标志设置、芯片特定请求和方向获取  
**调用流程**：设置请求标志 → 调用芯片请求函数 → 获取GPIO方向 → 错误恢复

```c fold
// drivers/gpio/gpiolib.c
static int gpiod_request_commit(struct gpio_desc *desc, const char *label)
{
    struct gpio_chip *chip = desc->gdev->chip;  // 获取GPIO芯片
    int status;           // 状态码
    unsigned long flags;  // 中断标志
    unsigned offset;      // GPIO偏移量
    
    /* 复制标签字符串，避免原字符串被释放 */
    if (label) {
        label = kstrdup_const(label, GFP_KERNEL);  // 内核内存分配复制
        if (!label)
            return -ENOMEM;  // 内存分配失败
    }
    
    spin_lock_irqsave(&gpio_lock, flags);  // 获取GPIO全局锁，禁用中断
    
    /* 原子性地测试并设置REQUESTED标志 */
    if (test_and_set_bit(FLAG_REQUESTED, &desc->flags) == 0) {
        desc_set_label(desc, label ? : "?");  // 设置GPIO标签
        status = 0;  // 设置成功
    } else {
        kfree_const(label);  // GPIO已被请求，释放标签内存
        status = -EBUSY;     // 返回忙碌错误
        goto done;
    }
    
    /* 自动设置pinctrl：将引脚配置为GPIO功能 */
    if (status == 0) {
        /* 释放自旋锁，因为pinctrl操作可能会睡眠 */
        spin_unlock_irqrestore(&gpio_lock, flags);
        status = pinctrl_gpio_request(desc_to_gpio(desc));  // 关键：自动pinctrl配置
        spin_lock_irqsave(&gpio_lock, flags);  // 重新获取锁
        
        /* 如果pinctrl设置失败，清理GPIO请求状态 */
        if (status < 0) {
            desc_set_label(desc, NULL);           // 清除标签
            kfree_const(label);                   // 释放标签内存
            clear_bit(FLAG_REQUESTED, &desc->flags);  // 清除请求标志
            goto done;
        }
    }
    
    /* 如果芯片有特定的请求函数，调用它 */
    if (chip->request) {
        /* chip->request可能会睡眠，需要释放自旋锁 */
        spin_unlock_irqrestore(&gpio_lock, flags);
        offset = gpio_chip_hwgpio(desc);  // 获取硬件GPIO偏移
        
        /* 检查GPIO线是否有效并调用芯片请求函数 */
        if (gpiochip_line_is_valid(chip, offset))
            status = chip->request(chip, offset);   // 关键调用点：芯片特定操作
        else
            status = -EINVAL;  // GPIO线无效
        
        spin_lock_irqsave(&gpio_lock, flags);  // 重新获取锁
        
        /* 如果芯片请求失败，清理已设置的状态 */
        if (status < 0) {
            desc_set_label(desc, NULL);           // 清除标签
            kfree_const(label);                   // 释放标签内存
            clear_bit(FLAG_REQUESTED, &desc->flags);  // 清除请求标志
            goto done;
        }
    }
    
    /* 如果芯片支持获取方向，读取当前GPIO方向 */
    if (chip->get_direction) {
        /* chip->get_direction可能会睡眠，需要释放自旋锁 */
        spin_unlock_irqrestore(&gpio_lock, flags);
        gpiod_get_direction(desc);  // 获取并缓存GPIO方向
        spin_lock_irqsave(&gpio_lock, flags);  // 重新获取锁
    }
    
done:
    spin_unlock_irqrestore(&gpio_lock, flags);  // 释放锁并恢复中断
    return status;  // 返回操作状态
}
```

这个函数的设计展示了几个重要的内核编程技巧：
- **原子操作的使用**：通过`test_and_set_bit`确保GPIO申请的原子性，避免竞争条件。
- **锁的精细管理**：在调用可能睡眠的函数前释放自旋锁，防止在持有锁的情况下睡眠导致的死锁。
- **错误恢复机制**：当芯片特定的请求函数失败时，正确清理已设置的状态，确保系统状态的一致性。

## 8 GPIO与Pinctrl的自动集成机制

在现代SoC中，一个引脚通常可以配置为多种功能（GPIO、UART、SPI、I2C等）。当我们请求一个GPIO时，内核需要自动将对应的引脚从其他功能切换到GPIO功能。这个过程通过pinctrl子系统自动完成。

**Pinctrl配置的详细时序**
```c
// Pinctrl自动配置的详细调用时序
gpiod_request_commit(desc, "reset")
  │
  ├── 1. 设置GPIO请求标志
  │   └── test_and_set_bit(FLAG_REQUESTED, &desc->flags)
  │
  ├── 2. 自动pinctrl配置 [关键步骤]
  │   └── pinctrl_gpio_request(gpio_num)
  │       ├── pinctrl_get_device_gpio_range(gpio, &pctldev, &range)  // 查找pinctrl设备
  │       ├── gpio_to_pin(range, gpio)                              // GPIO号转引脚号
  │       └── pinmux_request_gpio(pctldev, range, pin, gpio)
  │           ├── pin_is_required(pctldev, pin)                     // 检查引脚占用状态
  │           ├── ops->gpio_request_enable(pctldev, range, pin)     // 配置引脚为GPIO模式
  │           │   └── [硬件寄存器操作：MUX_REG &= ~FUNC_MASK; MUX_REG |= GPIO_FUNC]
  │           └── pin_set_gpio_owner(pctldev, pin, range, gpio)     // 标记引脚为GPIO所有
  │
  ├── 3. 芯片特定GPIO配置
  │   └── chip->request(chip, offset)                               // 芯片驱动特定操作
  │
  └── 4. 获取GPIO当前状态
      └── gpiod_get_direction(desc)                                 // 读取硬件方向设置
```

### 8.1 pinctrl_gpio_request函数：自动引脚配置

**函数作用**：自动将指定的引脚配置为GPIO功能，处理引脚复用  
**调用流程**：查找pinctrl设备 → 查找引脚映射 → 应用GPIO配置

```c
// drivers/pinctrl/core.c
int pinctrl_gpio_request(unsigned gpio)
{
    struct pinctrl_dev *pctldev;     // pinctrl设备
    struct pinctrl_gpio_range *range; // GPIO范围
    int ret;                         // 返回值
    int pin;                         // 引脚号
    
    /* 查找处理该GPIO的pinctrl设备和范围 */
    ret = pinctrl_get_device_gpio_range(gpio, &pctldev, &range);
    if (ret) {
        if (pinctrl_ready_for_gpio_range(gpio))
            ret = 0; /* 某些情况下允许无pinctrl */
        return ret;
    }
    
    mutex_lock(&pctldev->mutex);  // 锁定pinctrl设备
    
    /* 将GPIO编号转换为芯片内部的引脚编号 */
    pin = gpio_to_pin(range, gpio);
    
    /* 应用GPIO功能的pinctrl设置 */
    ret = pinmux_request_gpio(pctldev, range, pin, gpio);  // 配置引脚复用
    
    mutex_unlock(&pctldev->mutex);  // 解锁
    
    return ret;
}
```

### 8.2 pinmux_request_gpio函数：引脚复用配置

**函数作用**：实际执行引脚复用配置，将引脚设置为GPIO模式  
**调用流程**：检查引脚状态 → 查找GPIO功能 → 应用配置

```c
// drivers/pinctrl/pinmux.c
int pinmux_request_gpio(struct pinctrl_dev *pctldev,
                       struct pinctrl_gpio_range *range,
                       unsigned pin, unsigned gpio)
{
    const struct pinmux_ops *ops = pctldev->desc->pmxops;  // 复用操作函数
    const char *owner;    // 当前引脚所有者
    int ret;             // 返回值
    
    /* 检查引脚是否已被其他功能占用 */
    owner = pin_get_name(pctldev, pin);
    if (!owner) {
        dev_err(pctldev->dev, "pin %d is not registered\n", pin);
        return -EINVAL;
    }
    
    /* 检查引脚当前状态 */
    if (pin_is_required(pctldev, pin)) {
        dev_err(pctldev->dev,
                "pin %d (%s) is already used by %s; cannot claim for GPIO\n",
                pin, owner, pin_get_usage(pctldev, pin));
        return -EINVAL;
    }
    
    /* 如果芯片支持GPIO请求回调，调用它 */
    if (ops->gpio_request_enable) {
        /* 这里会自动配置引脚为GPIO功能 */
        ret = ops->gpio_request_enable(pctldev, range, pin);  // 关键：启用GPIO功能
        if (ret) {
            dev_err(pctldev->dev,
                    "GPIO request enable failed for pin %d (%s)\n",
                    pin, owner);
            return ret;
        }
    }
    
    /* 标记引脚为GPIO使用 */
    pin_set_gpio_owner(pctldev, pin, range, gpio);
    
    return 0;
}
```

### 8.3 芯片特定的GPIO使能函数

每个SoC的pinctrl驱动都会实现`gpio_request_enable`回调，用于实际配置硬件寄存器：

```c
// 示例：某SoC的GPIO使能实现
static int example_pmx_gpio_request_enable(struct pinctrl_dev *pctldev,
                                          struct pinctrl_gpio_range *range,
                                          unsigned pin)
{
    struct example_pinctrl *pc = pinctrl_dev_get_drvdata(pctldev);
    u32 reg_val;     // 寄存器值
    void __iomem *reg; // 寄存器地址
    
    /* 计算寄存器地址 */
    reg = pc->base + PIN_MUX_REG(pin);
    
    /* 读取当前寄存器值 */
    reg_val = readl(reg);
    
    /* 清除功能选择位，设置为GPIO模式 */
    reg_val &= ~PIN_MUX_FUNC_MASK;     // 清除功能选择位
    reg_val |= PIN_MUX_GPIO_FUNC;      // 设置为GPIO功能
    
    /* 写回寄存器 */
    writel(reg_val, reg);
    
    return 0;
}
```

### 8.4 自动pinctrl配置的意义

这种自动配置机制具有重要意义：
1. **透明性**：用户只需调用`gpiod_get`，无需手动配置pinctrl
2. **一致性**：确保GPIO和pinctrl状态的一致性
3. **错误防范**：防止引脚功能冲突和配置错误
4. **简化开发**：大大简化了驱动程序的开发复杂度

## 9 总结：gpiod_get的设计精髓

这种细致入微的设计正是Linux内核稳定性和可靠性的基础。`gpiod_get`函数体系展现了内核设计的几个核心理念：

1. **多层抽象**：从用户接口到硬件操作的层次化设计
2. **多路径支持**：设备树、ACPI、平台查找表的多重机制
3. **自动集成**：GPIO和pinctrl子系统的无缝集成，自动处理引脚复用
4. **资源管理**：严格的引用计数和生命周期管理
5. **错误处理**：完善的错误恢复和状态清理机制
6. **并发安全**：原子操作和锁机制保证多线程安全
7. **透明操作**：用户无需关心底层pinctrl配置，系统自动处理

特别值得注意的是GPIO与pinctrl的集成机制，这体现了Linux内核 **"让复杂的事情变简单"** 的设计哲学。用户只需要一个简单的`gpiod_get`调用，内核就会自动完成从引脚复用配置到GPIO资源管理的全部工作。

每一个看似简单的操作背后，都有着深思熟虑的设计和严谨的实现。