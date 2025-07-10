
## 1 引言

在前面的GPIO学习中，我们了解了GPIO的基本概念、编程接口和驱动实现。现在让我们探讨一个更加深入的话题：**GPIO子系统与Pinctrl子系统的深度交互机制**。

这个话题是理解现代SoC GPIO管理的关键。简单来说，`Pinctrl`子系统负责引脚的功能配置（GPIO、UART、I2C等），而`GPIO`子系统负责GPIO功能的具体操作。两者必须紧密协作才能实现用户友好的GPIO使用体验。

让我们用一个生活化的比喻来理解：如果把SoC比作一个智能办公楼，Pinctrl就像物业管理部门，负责房间的装修和功能配置（会议室、办公室、休息室等）；GPIO子系统就像办公室管理员，专门负责那些配置为办公用途的房间的日常运营。当有人要使用某个房间时，管理员会自动联系物业部门完成装修配置，用户无需关心这些复杂的协调过程。

> [!note]+ 重要概念澄清 
> `gpio`子系统是`pinctrl`的`client`用户。Pinctrl是一个软件虚拟处理的概念，它的实现本来就跟GPIO密切相关。当你使用`gpiod_get`获得GPIO引脚时，它就`偷偷地`通过Pinctrl把引脚复用为GPIO功能了。

本文将深入解析这种自动协调机制是如何实现的，以及如何在复杂应用中进行动态功能切换。

## 2 GPIO与Pinctrl强耦合设计的必要性

### 2.1 现代SoC的引脚复用挑战

现代SoC面临一个根本性矛盾：**功能需求无限增长，但引脚数量有物理限制**。为了解决这个问题，工程师们设计了引脚复用机制，让一个物理引脚可以配置成多种功能。

以RK3568为例，一个引脚可能支持多种功能：
```txt
GPIO1_A0引脚的可能功能：
├── 功能0: GPIO1_A0 (通用输入输出)
├── 功能1: I2C3_SDA_M0 (I2C3数据线)
├── 功能2: SPI1_RXD_M1 (SPI1接收数据)
└── 功能3: PWM4_M0 (PWM4输出)
```

假设某一个`gpio chip`只包括`2`个`gpio`，这两个`gpio`分别和`uart`进行功能复用。如果这两个管脚是同时控制的，要么是`gpio`，要么是`uart`，就很好处理了，按照`pinctrl subsystem`的精神，抽象出两个`function`：`gpio`和`uart`，`gpio chip`在使用`gpio`功能的时候，调用`pinctrl set state`，将它们切换为`gpio`即可。

但是，如果这两个`gpio`可以分开控制（很多硬件都是这样设计的），麻烦就出现了。每个`gpio`要单独抽象为一个`function`，因此我们可以抽象出`3`个`function`：`gpio1`、`gpio2`和`uart`。

然后考虑一下一个包含`32`个`gpio`的`chip`（一般硬件的`gpio bank`都是`32`个），如果它们都可以单独控制，则会出现`32`个`function`。而系统又不止有一个`chip`，灾难就发生了，我们的`device tree`文件将会被一坨坨的`gpio functions`撑爆！

上述情况实际上是个很现实的问题，尤其在定制化比较高的产品上`pin`脚又很多，那么很容易出现上述情况。这时候就需要一个对特殊情况的解决方案，就是现在驱动中的这种设计：`pinctrl`和`gpio`之间的耦合度很高。

### 2.2 传统独立设计的问题

假设GPIO和Pinctrl完全独立，会产生以下问题：

**配置复杂性**：用户需要手动进行两步操作
```c
// 传统方式需要用户手动协调
pinctrl_select_state(pinctrl, gpio_state);  // 步骤1：配置为GPIO功能
gpio_request(32, "my-gpio");                 // 步骤2：申请GPIO资源
```

**功能冲突风险**：无法检测到配置冲突
```c
// 用户A配置为I2C功能
pinctrl_select_state(pinctrl, i2c_state);

// 用户B尝试申请为GPIO（应该失败但可能不会被检测到）
gpio_request(32, "my-gpio");
```

**设备树爆炸**：每个可独立控制的GPIO都需要单独的function定义，设备树会变得非常臃肿。

### 2.3 Linux的强耦合解决方案

Linux内核通过**强耦合设计**巧妙地解决了这些问题：

```txt
用户申请GPIO (gpiod_get)
      ↓
GPIO子系统接收请求
      ↓
自动查找GPIO-Pin映射关系
      ↓
自动调用Pinctrl配置引脚为GPIO功能
      ↓
返回配置完成的GPIO给用户
```

这种设计实现了**三个透明**：
- **配置透明**：用户无需手动配置Pinctrl
- **冲突透明**：系统自动检测和避免功能冲突
- **复杂性透明**：将引脚管理的复杂性隐藏在内核内部

## 3 自动配置机制的技术实现

### 3.1 映射关系的建立机制

GPIO和Pinctrl之间的协作基于**映射关系**，这个关系通过数据结构和配置来实现。让我们看看核心的数据结构关系：

![pinctrl和gpio的关系](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cb96f54230b4a16a0b22085bc96cc3f2.png)

`gpio`和`pinctrl`通过`gpio_pin_range`建立映射关系，将`gpio`中的`pin`描述转换到`pinctrl`中的`pin`描述空间，从而将两者建立关联。这样`gpio`就可以不通过`pinctrl`提供的标准接口来设置`pin`的复用和配置了。

### 3.2 设备树中的映射关系配置

`gpio_pin_range`这个描述关系通过设备树文件来描述，主要有两种配置方式：

#### 3.2.1 方式一：设备树中使用gpio-ranges属性

在GPIO控制器的设备树节点中使用`gpio-ranges`关键字指定映射关系：

```dts
gpio0: gpio@fdd60000 {
    compatible = "rockchip,gpio-bank";
    reg = <0x0 0xfdd60000 0x0 0x100>;
    
    gpio-controller;
    #gpio-cells = <2>;
    
    // 核心属性：建立映射关系
    gpio-ranges = <&pinctrl 0 0 235>;
    //              ↑      ↑ ↑ ↑
    //              |      | | └── 映射的引脚数量(235个)
    //              |      | └──── Pinctrl中的起始引脚号(从0开始)  
    //              |      └────── GPIO中的起始编号(从0开始)
    //              └───────────── 对应的Pinctrl控制器
};
```

这个配置表示：`gpio0`映射到`pinctrl0`，长度为`235`个引脚。

#### 3.2.2 方式二：代码中动态建立映射

在驱动代码中可以通过API函数建立映射关系：
```c
// MTK平台的示例代码
pctl->chip = devm_kzalloc(&pdev->dev, sizeof(*pctl->chip), GFP_KERNEL);
if (!pctl->chip)
    return -ENOMEM;

*pctl->chip = mtk_gpio_chip;
pctl->chip->ngpio = pctl->devdata->npins;  // 设置GPIO数量
pctl->chip->label = dev_name(&pdev->dev);
pctl->chip->parent = &pdev->dev;
pctl->chip->base = -1;

ret = gpiochip_add_data(pctl->chip, pctl);

/* 注册GPIO到pin的映射关系 */
ret = gpiochip_add_pin_range(pctl->chip, dev_name(&pdev->dev),
                0, 0, pctl->devdata->npins);
```

**映射关系建立过程**
![映射关系建立过程|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e32831403a04bee101cf5e647b9db7f2.png)

以上过程将生成`pinctrl_dev`下由`gpio_ranges`组织成的链表。

### 3.3 完整的自动配置调用链

当用户调用`gpiod_get`时，触发的完整调用链如下：

![gpiod_get调用流程](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8753881c2f7a09d97779e3e9cfac87e4.png)

详细的调用关系：
```c
gpiod_get
  └── gpiod_get_index
      └── desc = of_find_gpio(dev, con_id, idx, &lookupflags);
          └── ret = gpiod_request(desc, con_id ? con_id : devname);
              └── ret = gpiod_request_commit(desc, label);
                  └── if (chip->request) {
                          ret = chip->request(chip, offset);
                      }
```

我们编写**GPIO驱动程序**时，所设置的`chip->request`函数，一般直接调用`gpiochip_generic_request`，它导致Pinctrl把引脚复用为GPIO功能：
```c
gpiochip_generic_request(struct gpio_chip *chip, unsigned offset)
  └── pinctrl_request_gpio(chip->gpiodev->base + offset)
      └── ret = pinctrl_get_device_gpio_range(gpio, &pctldev, &range); 
          // gpio是引脚的全局编号
      └── pin = gpio_to_pin(range, gpio);  // 转换为Pin编号空间
      └── ret = pinmux_request_gpio(pctldev, range, pin, gpio);
          └── ret = pin_request(pctldev, pin, owner, range);
```

关键的转换过程在`pinctrl_request_gpio`函数中：
```c
int pinctrl_request_gpio(unsigned gpio)
{
    struct pinctrl_dev *pctldev;
    struct pinctrl_gpio_range *range;
    int ret, pin;

    /* 步骤1：根据GPIO编号找到对应的Pinctrl设备和范围 */
    ret = pinctrl_get_device_gpio_range(gpio, &pctldev, &range);
    if (ret) {
        if (pinctrl_ready_for_gpio_range(gpio))
            ret = 0; /* 某些GPIO可能不需要pinctrl管理 */
        return ret;
    }

    mutex_lock(&pctldev->mutex);

    /* 步骤2：将GPIO编号转换为Pin编号 */
    pin = gpio_to_pin(range, gpio);

    /* 步骤3：请求Pin并配置为GPIO功能 */
    ret = pinmux_request_gpio(pctldev, range, pin, gpio);

    mutex_unlock(&pctldev->mutex);
    return ret;
}
```

Pinctrl子系统中的`pin_request`函数就会把引脚配置为GPIO功能：
```c
static int pin_request(struct pinctrl_dev *pctldev,
                       int pin, const char *owner,
                       struct pinctrl_gpio_range *gpio_range)
{
    const struct pinmux_ops *ops = pctldev->desc->pmxops;
    int status;
    
    /*
     * 如果没有为引脚定义request函数，我们假设默认就可以使用该引脚
     */
    if (gpio_range && ops->gpio_request_enable)
        /* 这个函数请求并启用单个GPIO引脚 */
        status = ops->gpio_request_enable(pctldev, gpio_range, pin);
    else if (ops->request)
        status = ops->request(pctldev, pin);
    else
        status = 0;
        
    return status;
}
```

芯片厂商实现的`gpio_request_enable`函数最终将硬件引脚配置为GPIO功能。以Rockchip为例：
```c
static int rockchip_gpio_request_enable(struct pinctrl_dev *pctldev,
                                       struct pinctrl_gpio_range *range,
                                       unsigned pin)
{
    struct rockchip_pinctrl *info = pinctrl_dev_get_drvdata(pctldev);
    struct rockchip_pin_bank *bank;
    int mux;

    /* 计算引脚所属的bank和在bank内的偏移 */
    bank = pin_to_bank(info, pin);
    if (!bank)
        return -EINVAL;

    /* 将引脚复用功能设置为GPIO */
    mux = RK_FUNC_GPIO;
    
    /* 写入硬件寄存器，配置引脚复用 */
    return rockchip_set_mux(bank, pin - bank->pin_base, mux);
}
```

通过这个完整的调用链，用户只需要调用一个`gpiod_get`函数，系统就会自动完成**设备树解析**、**映射关系查找**、**引脚冲突检测**、**硬件寄存器配置**等所有工作。

### 3.4 两种配置方式的选择

根据具体需求，可以选择不同的配置方式：

#### 3.4.1 使用GPIO前先设置Pinctrl（传统方式）

```dts
&{/} {  // 加入到根节点
    my_gpio: gpiol_a0 {
        compatible = "mygpio";
        my-gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_HIGH>;
        
        pinctrl-names = "default";
        pinctrl-0 = <&mygpio_ctrl>;
    };
}

&{/pinctrl} {  // 加入到pinctrl节点
    mygpio {
        mygpio_ctrl: my-gpio-ctrl {
            rockchip,pins = <1 RK_PA0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
}
```

#### 3.4.2 使用GPIO前不设置Pinctrl（自动配置）

在GPIO设备树中使用`gpio-ranges`来描述它们之间的联系：

解析过程：
```c
int gpiochip_add_data(struct gpio_chip *chip, void *data)
{
    status = of_gpiochip_add(chip);
    // 调用 of_gpiochip_add_pin_range(chip);
}

of_gpiochip_add_pin_range
{
    for (;; index++) {
        ret = of_parse_phandle_with_fixed_args(np, "gpio-ranges", 3,
                        index, &pinspec);

        pctldev = of_pinctrl_get(pinspec.np); // 根据gpio-ranges的第1个参数找到pctldev

        // 增加映射关系        
        /* npins != 0: linear range */
        ret = gpiochip_add_pin_range(chip,
                                     pinctrl_dev_get_devname(pctldev),
                                     pinspec.args[0],
                                     pinspec.args[1],
                                     pinspec.args[2]);
    }
}
```

> [!tip]+ 实用建议 
> 推荐使用`gpio-ranges`方式，因为它实现了真正的自动化，用户无需手动配置Pinctrl状态，系统会在GPIO申请时自动处理。

## 4 动态引脚复用切换的应用实践

在某些复杂应用中，我们需要**在运行时动态切换引脚功能**。虽然GPIO子系统提供了自动配置，但动态切换仍需要直接使用Pinctrl API

### 4.1 Pinctrl API详细介绍

**pinctrl_get**：用于获取与给定设备对象相关联的pinctrl（引脚控制器）实例，是使用pinctrl功能的第一步。
```c
// linux/pinctrl/consumer.h
struct pinctrl *pinctrl_get(struct device *dev);
```
- `dev`：指向struct device的指针，表示要获取pinctrl的设备对象
- **返回值**：
	- **成功**：返回一个指向struct pinctrl的指针，表示获取到的pinctrl实例
	- **失败**：返回错误指针（使用IS_ERR()检查），如果设备对象不支持pinctrl或获取失败


**pinctrl_put**：用于释放由pinctrl_get()函数获得的pinctrl实例，以释放相关资源，确保资源管理的完整性。
```c
// linux/pinctrl/consumer.h
void pinctrl_put(struct pinctrl *p);
```
- `p`：指向struct pinctrl的指针，表示要释放的pinctrl实例
- **无返回值**

**pinctrl_lookup_state**：用于在给定的pinctrl实例中查找指定名称的pinctrl状态，用于后续的状态切换操作。
```c
// linux/pinctrl/consumer.h
struct pinctrl_state *pinctrl_lookup_state(struct pinctrl *p, const char *name);
```
- `p`：指向pinctrl实例的指针，表示要进行状态查找的pinctrl
- `name`：指向状态名称的字符串指针，表示要查找的状态名称（对应设备树中pinctrl-names的值）
- **返回值**：
	- **成功**：返回一个指向struct pinctrl_state的指针，表示找到的pinctrl状态
	- **失败**：返回错误指针（使用IS_ERR()检查），如果未找到指定名称的状态或发生错误


**pinctrl_select_state**：用于将指定的pinctrl状态设置到硬件上，实现引脚功能的实际切换。
```c
// linux/pinctrl/consumer.h
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
```
- `p`：指向pinctrl实例的指针，表示要进行状态设置的pinctrl
- `s`：指向pinctrl_state的指针，表示要设置的目标状态
- **返回值**：
	- **成功**：返回0
	- **失败**：返回负数错误码，表示设置失败的原因

### 4.2 动态切换的使用步骤

动态切换的基本步骤是：
1. **pinctrl_get** - 获取设备的pinctrl句柄
2. **pinctrl_lookup_state** - 查找不同的配置状态
3. **pinctrl_select_state** - 切换到指定状态

### 4.3 多功能引脚的设备树配置

```dts
// 定义了一个支持多种引脚功能切换的GPIO设备
my_gpio: gpio1_a0 {
    compatible = "mygpio";                    /* 设备兼容性字符串，用于驱动匹配 */
    my-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>;  /* GPIO引脚定义：
                                                   * &gpio1: 引用gpio1控制器
                                                   * RK_PA0: Rockchip PA0引脚
                                                   * GPIO_ACTIVE_HIGH: 高电平有效 */
    pinctrl-names = "mygpio_func1", "mygpio_func2";  /* 引脚控制状态名称列表：
                                                      * "mygpio_func1": 功能状态1
                                                      * "mygpio_func2": 功能状态2 */
    pinctrl-0 = <&mygpio_ctrl>;               /* 状态0对应的引脚控制配置 */
    pinctrl-1 = <&i2c3_sda>;                  /* 状态1对应的引脚控制配置(I2C3 SDA功能) */
};
```
- `pinctrl-names`表示引脚控制器配置的名称，这里有两个值，分别对应复用1和复用2
- `pinctrl-0`指定了与该配置相关联的引脚控制器句柄，这里为`&mygpio_ctrl`，表示复用为gpio功能
- `pinctrl-1`指定了与该配置相关联的引脚控制器句柄，这里为`&i2c3_sda`，表示复用为i2c3_sda功能

然后在pinctrl节点中定义具体的引脚配置：
```dts
/*
 * 引脚控制配置节点定义
 * 定义了GPIO设备的不同功能状态配置
 */

/* GPIO功能1配置组 */
mygpio_func1 {
    /* GPIO控制配置：将RK_PA0配置为GPIO功能 */
    mygpio_ctrl: my-gpio-ctrl {
        rockchip,pins = <1 RK_PA0 RK_FUNC_GPIO &pcfg_pull_none>;
        /* 引脚配置说明：
         * 1: GPIO组号(gpio1)
         * RK_PA0: 引脚编号(PA0)
         * RK_FUNC_GPIO: 功能选择(GPIO功能)
         * &pcfg_pull_none: 引脚电气配置(无上下拉) */
    };
};

/* GPIO功能2配置组 */
myggio_func2 {
    /* I2C3 SDA配置：将RK_PA0配置为I2C3数据线功能 */
    i2c3_sda: i2c3_sda {
        rockchip,pins = <1 RK_PA0 1 &pcfg_pull_none>;
        /* 引脚配置说明：
         * 1: GPIO组号(gpio1)
         * RK_PA0: 引脚编号(PA0)
         * 1: 功能选择(复用功能1，通常为I2C3_SDA)
         * &pcfg_pull_none: 引脚电气配置(无上下拉) */
    };
};
```

### 4.4 完整的动态切换驱动实现

下面是一个完整的支持动态功能切换的驱动示例：
```c fold
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/pinctrl/consumer.h>
#include <linux/gpio/consumer.h>
#include <linux/device.h>
#include <linux/mutex.h>

/* 功能模式枚举 */
enum pin_function_mode {
    PIN_MODE_GPIO = 0,    /* GPIO功能模式 */
    PIN_MODE_I2C,         /* I2C功能模式 */
    PIN_MODE_UART,        /* UART功能模式 */
    PIN_MODE_MAX,
};

/* 设备私有数据结构 */
struct multi_func_device {
    struct device *dev;                           /* 设备指针 */
    struct pinctrl *pinctrl;                      /* pinctrl句柄 */
    struct pinctrl_state *pin_states[PIN_MODE_MAX]; /* 各种功能状态 */
    struct gpio_desc *gpio;                       /* GPIO描述符 */
    enum pin_function_mode current_mode;          /* 当前功能模式 */
    struct mutex mode_mutex;                      /* 模式切换互斥锁 */
};

/* 模式名称数组，与枚举对应 */
static const char *mode_names[PIN_MODE_MAX] = {
    [PIN_MODE_GPIO] = "mygpio_func1",   /* 对应设备树中的pinctrl-names */
    [PIN_MODE_I2C] = "mygpio_func2", 
    [PIN_MODE_UART] = "uart",
};

/**
 * 核心的模式切换函数
 * @mfd: 多功能设备实例
 * @new_mode: 要切换到的新模式
 * @return: 0表示成功，负数表示失败
 */
static int multi_func_set_mode(struct multi_func_device *mfd, 
                               enum pin_function_mode new_mode)
{
    enum pin_function_mode old_mode;
    int ret;
    
    if (new_mode >= PIN_MODE_MAX)
        return -EINVAL;
    
    mutex_lock(&mfd->mode_mutex);
    
    old_mode = mfd->current_mode;
    if (old_mode == new_mode) {
        mutex_unlock(&mfd->mode_mutex);
        return 0;  /* 已经是目标模式，无需切换 */
    }
    
    dev_info(mfd->dev, "Switching from %s to %s mode\n", 
             mode_names[old_mode], mode_names[new_mode]);
    
    /* 步骤1：释放当前模式的资源 */
    if (old_mode == PIN_MODE_GPIO && mfd->gpio) {
        gpiod_put(mfd->gpio);         /* 释放GPIO资源 */
        mfd->gpio = NULL;
        dev_dbg(mfd->dev, "Released GPIO resource\n");
    }
    
    /* 步骤2：切换pinctrl状态 */
    ret = pinctrl_select_state(mfd->pinctrl, mfd->pin_states[new_mode]);
    if (ret) {
        dev_err(mfd->dev, "Failed to switch pinctrl state: %d\n", ret);
        goto restore_old_mode;
    }
    
    /* 步骤3：初始化新模式的资源 */
    if (new_mode == PIN_MODE_GPIO) {
        /* 获取GPIO资源，这时引脚已经被配置为GPIO功能 */
        mfd->gpio = gpiod_get(mfd->dev, "my", GPIOD_OUT_LOW);
        if (IS_ERR(mfd->gpio)) {
            ret = PTR_ERR(mfd->gpio);
            dev_err(mfd->dev, "Failed to get GPIO: %d\n", ret);
            mfd->gpio = NULL;
            goto restore_old_mode;
        }
        dev_dbg(mfd->dev, "Acquired GPIO resource\n");
    }
    
    /* 步骤4：更新当前状态 */
    mfd->current_mode = new_mode;
    dev_info(mfd->dev, "Successfully switched to %s mode\n", mode_names[new_mode]);
    
    mutex_unlock(&mfd->mode_mutex);
    return 0;

restore_old_mode:
    /* 错误恢复：尝试恢复到原来的模式 */
    dev_warn(mfd->dev, "Restoring %s mode after failure\n", mode_names[old_mode]);
    pinctrl_select_state(mfd->pinctrl, mfd->pin_states[old_mode]);
    
    mutex_unlock(&mfd->mode_mutex);
    return ret;
}

/**
 * sysfs接口：显示当前模式
 */
static ssize_t selectmux_show(struct device *dev, struct device_attribute *attr, 
                              char *buf)
{
    struct multi_func_device *mfd = dev_get_drvdata(dev);
    return sprintf(buf, "%s\n", mode_names[mfd->current_mode]);
}

/**
 * sysfs接口：设置新模式
 * 用户可以通过 echo "模式名" > selectmux 来切换模式
 */
static ssize_t selectmux_store(struct device *dev, struct device_attribute *attr,
                              const char *buf, size_t count)
{
    struct multi_func_device *mfd = dev_get_drvdata(dev);
    unsigned long select;
    int ret;
    
    /* 解析用户输入：0表示切换到I2C模式，1表示切换到GPIO模式 */
    ret = kstrtoul(buf, 10, &select);
    if (ret)
        return ret;
    
    if (select == 1) {
        ret = multi_func_set_mode(mfd, PIN_MODE_GPIO);    /* 选择GPIO模式 */
    } else if (select == 0) {
        ret = multi_func_set_mode(mfd, PIN_MODE_I2C);     /* 选择I2C模式 */
    } else {
        dev_err(dev, "Invalid mode selection. Use 0 for I2C, 1 for GPIO\n");
        return -EINVAL;
    }
    
    return ret ? ret : count;
}
static DEVICE_ATTR_RW(selectmux);    /* 定义可读写的设备属性 selectmux */

/**
 * 获取并查找所有pinctrl状态
 * @dev: 设备指针
 * @return: 0表示成功，负数表示失败
 */
static int pinctrl_get_and_lookstate(struct multi_func_device *mfd)
{
    struct device *dev = mfd->dev;
    int i, ret;
    
    /* 获取设备的pinctrl句柄 */
    mfd->pinctrl = pinctrl_get(dev);
    if (IS_ERR(mfd->pinctrl)) {
        ret = PTR_ERR(mfd->pinctrl);
        dev_err(dev, "Failed to get pinctrl: %d\n", ret);
        return ret;
    }
    
    /* 查找所有定义的pinctrl状态 */
    for (i = 0; i < PIN_MODE_MAX; i++) {
        mfd->pin_states[i] = pinctrl_lookup_state(mfd->pinctrl, mode_names[i]);
        if (IS_ERR(mfd->pin_states[i])) {
            dev_err(dev, "Failed to find %s state\n", mode_names[i]);
            return PTR_ERR(mfd->pin_states[i]);
        }
        dev_dbg(dev, "Found pinctrl state: %s\n", mode_names[i]);
    }
    
    return 0;
}

/* 平台设备初始化函数 */
static int my_platform_probe(struct platform_device *pdev)
{
    struct multi_func_device *mfd;
    struct device *dev = &pdev->dev;
    int ret;
    
    dev_info(dev, "Multi-function device probing\n");
    
    /* 分配设备私有数据结构 */
    mfd = devm_kzalloc(dev, sizeof(*mfd), GFP_KERNEL);
    if (!mfd)
        return -ENOMEM;
        
    mfd->dev = dev;
    mutex_init(&mfd->mode_mutex);
    
    /* 获取pinctrl句柄并查找所有状态 */
    ret = pinctrl_get_and_lookstate(mfd);
    if (ret)
        return ret;
    
    /* 设置初始模式为GPIO */
    ret = multi_func_set_mode(mfd, PIN_MODE_GPIO);
    if (ret) {
        dev_err(dev, "Failed to set initial GPIO mode\n");
        return ret;
    }
    
    /* 创建sysfs接口供用户空间控制 */
    ret = device_create_file(dev, &dev_attr_selectmux);
    if (ret) {
        dev_err(dev, "Failed to create sysfs interface\n");
        return ret;
    }
    
    platform_set_drvdata(pdev, mfd);
    dev_info(dev, "Multi-function device initialized in GPIO mode\n");
    
    return 0;
}

/* 平台设备移除函数 */
static int my_platform_remove(struct platform_device *pdev)
{
    struct multi_func_device *mfd = platform_get_drvdata(pdev);
    
    /* 移除sysfs接口 */
    device_remove_file(&pdev->dev, &dev_attr_selectmux);
    
    /* 释放GPIO资源（如果当前处于GPIO模式） */
    if (mfd->current_mode == PIN_MODE_GPIO && mfd->gpio) {
        gpiod_put(mfd->gpio);
    }
    
    dev_info(&pdev->dev, "Multi-function device removed\n");
    return 0;
}

/* 设备匹配表 */
static const struct of_device_id of_match_table_id[] = {
    { .compatible = "mygpio" },    /* 与设备树中compatible属性匹配 */
    { }
};
MODULE_DEVICE_TABLE(of, of_match_table_id);

/* 平台驱动结构体定义 */
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
        .of_match_table = of_match_table_id,
    },
};

/* 模块初始化函数 */
static int __init my_platform_driver_init(void)
{
    int ret;

    /* 注册平台驱动 */
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        pr_err("Failed to register platform driver: %d\n", ret);
        return ret;
    }

    pr_info("Multi-function platform driver initialized\n");
    return 0;
}

/* 模块退出函数 */
static void __exit my_platform_driver_exit(void)
{
    /* 注销平台驱动 */
    platform_driver_unregister(&my_platform_driver);
    pr_info("Multi-function platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Multi-function Pin Control Driver with GPIO/I2C switching");
MODULE_AUTHOR("Your Name");
```

### 4.5 使用方法和测试

编译并加载驱动后，可以通过sysfs接口进行动态切换：
```bash
# 查看当前模式
cat /sys/devices/platform/gpio1_a0/selectmux

# 切换到I2C模式（写入0）
echo 0 > /sys/devices/platform/gpio1_a0/selectmux

# 切换到GPIO模式（写入1）
echo 1 > /sys/devices/platform/gpio1_a0/selectmux

# 通过dmesg查看切换过程的日志信息
dmesg | tail -20
```

可以通过以下命令验证引脚复用状态的变化：
```bash
# 查看引脚复用状态（以gpio1 RK_PA0为例）
cat /sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins | grep -A 2 -B 2 "pin 32"
```

![引脚状态查看示例](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b97b3829f67b860136e43d217bef86bb.png)

驱动加载后的状态变化：

![驱动加载后的状态](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/921bdc5f3b6de920da16c2a85d489536.png)

> [!warning]+ 重要提醒 
> 在进行动态切换时，必须确保没有其他驱动正在使用相同的引脚，否则可能导致功能冲突。建议在切换前检查引脚的使用状态。

## 5 实际应用场景和最佳实践

### 5.1 典型应用场景

动态引脚复用在以下场景中特别有用：
- **多协议通信接口**：同一组引脚在不同时期用作不同的通信协议（SPI、I2C、UART等）
- **功能可配置的产品**：根据用户配置或外部条件动态调整引脚功能
- **调试和测试**：在开发阶段需要频繁切换引脚功能进行测试
- **资源复用**：在引脚资源紧张的系统中最大化利用有限的引脚

### 5.2 设计最佳实践

**互斥性保证**：确保同一时间只有一个功能模块使用特定引脚

**状态管理**：维护清晰的状态转换逻辑，包括错误恢复机制

**资源清理**：在模式切换时正确释放和重新申请相关资源

**用户接口**：提供简洁的用户接口，隐藏复杂的底层操作

**调试支持**：提供充分的日志信息和状态查询接口

## 6 本章小结

通过本章的深入学习，我们全面理解了GPIO与Pinctrl子系统交互的核心机制。这种强耦合设计巧妙地解决了现代SoC引脚管理的复杂性问题，实现了用户接口的简洁性和系统功能的完整性。

**核心设计原则**总结：
- **透明性**：GPIO子系统自动处理Pinctrl配置，用户专注业务逻辑，无需关心底层的引脚复用细节。
- **一致性**：统一的API接口屏蔽底层硬件差异，同样的代码可以在不同平台上运行。
- **安全性**：自动检测引脚冲突，避免配置错误，通过映射关系确保引脚使用的排他性。
- **灵活性**：支持动态切换以适应复杂应用需求，满足多功能产品的设计要求。

**技术实现要点**回顾：
- `gpio-ranges`属性建立了GPIO编号空间到Pinctrl引脚空间的映射关系桥梁，这是自动配置的基础。
- `gpiochip_generic_request`函数触发了自动配置机制，在用户申请GPIO时悄无声息地完成引脚功能配置。
- 完整的调用链确保了引脚功能的正确配置，从用户API到硬件寄存器的每一步都有严格的检查和处理。
- Pinctrl API提供了动态切换的控制能力，支持复杂应用场景的引脚功能管理。

**实际应用价值**：
- 理解这些交互机制对驱动开发具有重要意义。当GPIO操作出现问题时，你可以从多个层面分析：设备树配置是否正确、映射关系是否建立、Pinctrl状态是否冲突等。在需要实现复杂引脚管理功能时，可以借鉴动态切换的实现模式。
- 换句话说，这种设计哲学体现了Linux内核的优秀传统：通过精心的抽象和封装，将复杂的硬件管理变得简单易用，同时保持足够的灵活性以应对各种需求。这种设计思想在其他Linux子系统中同样适用，深入理解GPIO与Pinctrl的交互机制，不仅能够提升你的GPIO开发能力，更能为学习其他内核子系统提供有益的思路和方法。
- 掌握了这些知识，你就具备了在Linux系统中自如控制GPIO的能力，为更复杂的嵌入式系统开发打下了坚实的基础。无论是简单的LED控制还是复杂的多功能引脚管理，你都能够从容应对，并且知道如何在出现问题时进行系统性的分析和解决。