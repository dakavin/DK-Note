
在嵌入式Linux开发中，**GPIO**（General Purpose Input/Output）是最常用的硬件资源之一。而**设备树**为GPIO的描述和管理提供了标准化的方法。本文将深入介绍如何通过OF函数在设备树中获取和操作GPIO资源。

## 1 GPIO子系统与设备树概述

### 1.1 内核配置与头文件

**必要的内核配置：**
```makefile
CONFIG_OF_GPIO=y
```

**必要的头文件：**
```c
#include <linux/of_gpio.h>
#include <linux/gpio.h>
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a23fbdf216f202095f91f5094f7db47a.png)

_上图展示了OF GPIO相关的头文件包含关系和CONFIG_OF_GPIO宏的定义_

### 1.2 设备树中GPIO的描述方式

在设备树中，GPIO通常通过以下方式描述：
```dts
gpio_node {
    compatible = "manufacturer,device-name";
    gpio-controller;
    #gpio-cells = <2>;
    
    // GPIO属性描述
    reset-gpios = <&gpio1 10 GPIO_ACTIVE_LOW>;
    power-gpios = <&gpio2 5 GPIO_ACTIVE_HIGH>;
    status-gpios = <&gpio1 15 GPIO_ACTIVE_HIGH>;
};
```
- `gpio-controller`：声明该节点是GPIO控制器
- `#gpio-cells`：指定引用该控制器时需要的参数个数
- `xxx-gpios`：具体的GPIO引脚描述

## 2 核心OF GPIO函数详解

### 2.1 获取指定GPIO

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca171133dc7d7b22c852a7a034c3f78c.png)

这是获取设备树GPIO信息的核心函数：
```c
// drivers/gpio/gpiolib-of.c
int of_get_named_gpio_flags(struct device_node *np, const char *propname, int index, enum of_gpio_flags *flags)
```
- **np**：设备节点指针
- **propname**：GPIO属性名称（如"reset-gpios"）
- **index**：GPIO索引（当属性包含多个GPIO时）
- **flags**：输出参数，返回GPIO标志（极性、触发方式等）
- **返回值：**
	- 成功：返回GPIO编号（正数）
	- 失败：返回负数错误码

### 2.2 函数实现源码分析

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/830ddb08d48e12f06dbbed41873d3a4c.png)

**源码工作流程：**
1. **解析设备树属性**：提取GPIO控制器引用和参数
2. **查找GPIO控制器**：根据phandle找到对应的GPIO chip
3. **转换为系统GPIO号**：将控制器内部编号转换为全局GPIO号
4. **提取标志信息**：解析GPIO的极性和其他属性

> [!tip]+ 简化版本函数 
> 如果不需要获取GPIO标志，可以使用简化版本：
> 
> ```c
> static inline int of_get_named_gpio(struct device_node *np, const char *propname, int index)
> {
>     return of_get_named_gpio_flags(np, propname, index, NULL);
> }
> ```

### 2.3 其他常用OF GPIO函数

#### 2.3.1 统计GPIO数量

```c
static inline int of_gpio_count(struct device_node *np, const char *propname)
```
**功能：** 统计指定节点中包含的GPIO数量

#### 2.3.2 统计命名GPIO数量

```c
int of_gpio_named_count(struct device_node *np, const char *propname)
```
**功能：** 更精确地统计指定属性的GPIO数量，处理更复杂的情况

## 3 设备树GPIO配置实例

### 3.1 设备树节点配置

以下是一个实际的设备树GPIO配置示例：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ef62d76276c1fc761732d034ec64c481.png)

_上图显示了设备树中touchscreen设备的GPIO配置，包括reset和irq引脚的定义_

```dts
// 示例设备树节点
touchscreen@40 {
    compatible = "silead,gsl1680";
    reg = <0x40>;
    
    // GPIO配置
    power-gpios = <&gpio1 10 GPIO_ACTIVE_HIGH>;
    reset-gpios = <&gpio1 11 GPIO_ACTIVE_LOW>;
    irq-gpios = <&gpio2 15 GPIO_ACTIVE_HIGH>;
    
    // 其他属性
    touchscreen-size-x = <1024>;
    touchscreen-size-y = <600>;
    status = "okay";
};
```

**配置要点：**
- 必须设置 `status = "okay"` 才能激活设备
- GPIO引用格式：`<&控制器 引脚号 标志>`
- 标志可以是 `GPIO_ACTIVE_HIGH` 或 `GPIO_ACTIVE_LOW`

### 3.2 驱动程序实现

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d7f88315796efd6693d146005f3c850e.png)
_上图展示了在touchscreen驱动中如何解析和使用设备树中的GPIO配置_

以下是一个完整的驱动程序实现示例：
```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/of_gpio.h>
#include <linux/gpio.h>
#include <linux/delay.h>

struct my_gpio_data {
    int reset_gpio;
    int power_gpio;
    int irq_gpio;
    enum of_gpio_flags reset_flags;
    enum of_gpio_flags power_flags;
    enum of_gpio_flags irq_flags;
};

static int my_gpio_probe(struct platform_device *pdev)
{
    struct device_node *np = pdev->dev.of_node;
    struct my_gpio_data *gpio_data;
    int ret;

    // 分配设备数据结构
    gpio_data = devm_kzalloc(&pdev->dev, sizeof(*gpio_data), GFP_KERNEL);
    if (!gpio_data)
        return -ENOMEM;

    // 获取reset GPIO
    gpio_data->reset_gpio = of_get_named_gpio_flags(np, "reset-gpios", 0, &gpio_data->reset_flags);
    if (!gpio_is_valid(gpio_data->reset_gpio)) {
        dev_err(&pdev->dev, "Failed to get reset GPIO\n");
        return -EINVAL;
    }

    // 获取power GPIO
    gpio_data->power_gpio = of_get_named_gpio_flags(np, "power-gpios", 0, &gpio_data->power_flags);
    if (!gpio_is_valid(gpio_data->power_gpio)) {
        dev_err(&pdev->dev, "Failed to get power GPIO\n");
        return -EINVAL;
    }

    // 获取irq GPIO
    gpio_data->irq_gpio = of_get_named_gpio_flags(np, "irq-gpios", 0, &gpio_data->irq_flags);
    if (!gpio_is_valid(gpio_data->irq_gpio)) {
        dev_warn(&pdev->dev, "No IRQ GPIO specified\n");
        gpio_data->irq_gpio = -1;
    }

    // 申请GPIO资源
    ret = devm_gpio_request_one(&pdev->dev, gpio_data->reset_gpio, 
                                GPIOF_OUT_INIT_LOW, "reset-gpio");
    if (ret) {
        dev_err(&pdev->dev, "Failed to request reset GPIO: %d\n", ret);
        return ret;
    }

    ret = devm_gpio_request_one(&pdev->dev, gpio_data->power_gpio, 
                                GPIOF_OUT_INIT_HIGH, "power-gpio");
    if (ret) {
        dev_err(&pdev->dev, "Failed to request power GPIO: %d\n", ret);
        return ret;
    }

    if (gpio_is_valid(gpio_data->irq_gpio)) {
        ret = devm_gpio_request_one(&pdev->dev, gpio_data->irq_gpio, 
                                    GPIOF_IN, "irq-gpio");
        if (ret) {
            dev_err(&pdev->dev, "Failed to request IRQ GPIO: %d\n", ret);
            return ret;
        }
    }

    // 保存设备数据
    platform_set_drvdata(pdev, gpio_data);

    // 执行设备复位序列
    gpio_set_value(gpio_data->reset_gpio, 
                   gpio_data->reset_flags & OF_GPIO_ACTIVE_LOW ? 1 : 0);
    msleep(10);
    gpio_set_value(gpio_data->reset_gpio, 
                   gpio_data->reset_flags & OF_GPIO_ACTIVE_LOW ? 0 : 1);
    msleep(100);

    dev_info(&pdev->dev, "GPIO driver probed successfully\n");
    dev_info(&pdev->dev, "Reset GPIO: %d, Power GPIO: %d, IRQ GPIO: %d\n",
             gpio_data->reset_gpio, gpio_data->power_gpio, gpio_data->irq_gpio);

    return 0;
}

static int my_gpio_remove(struct platform_device *pdev)
{
    dev_info(&pdev->dev, "GPIO driver removed\n");
    return 0;
}

static const struct of_device_id my_gpio_of_match[] = {
    { .compatible = "my-company,gpio-test" },
    { /* sentinel */ }
};
MODULE_DEVICE_TABLE(of, my_gpio_of_match);

static struct platform_driver my_gpio_driver = {
    .probe = my_gpio_probe,
    .remove = my_gpio_remove,
    .driver = {
        .name = "my-gpio-driver",
        .of_match_table = my_gpio_of_match,
    },
};

module_platform_driver(my_gpio_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("GPIO Device Tree Example Driver");
```

## 4 内核配置

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/56d2f070b4beb37bb0266736b1a86715.png)

_上图显示了内核配置菜单中GPIO相关的配置选项_

**关键配置项：**
```makefile
# 基础GPIO支持
CONFIG_GPIOLIB=y

# 设备树GPIO支持
CONFIG_OF_GPIO=y

# GPIO字符设备支持（用户空间访问）
CONFIG_GPIO_CDEV=y

# GPIO调试支持
CONFIG_DEBUG_GPIO=y

# 特定平台的GPIO驱动（根据硬件平台选择）
CONFIG_GPIO_GENERIC=y
```

> [!warning]+ 配置注意事项
> 
> 1. 确保启用 `CONFIG_OF_GPIO` 才能使用OF GPIO函数
> 2. 根据具体硬件平台启用相应的GPIO控制器驱动
> 3. 如需用户空间访问GPIO，需启用 `CONFIG_GPIO_CDEV`

## 5 GPIO标志与极性处理

### 5.1 GPIO标志类型

```c
enum of_gpio_flags {
    OF_GPIO_ACTIVE_LOW = 0x1,
    OF_GPIO_SINGLE_ENDED = 0x2,
    OF_GPIO_OPEN_DRAIN = 0x4,
    OF_GPIO_TRANSITORY = 0x8,
    OF_GPIO_PULL_UP = 0x10,
    OF_GPIO_PULL_DOWN = 0x20,
};
```

### 5.2 正确处理GPIO极性

```c
// 设置GPIO值时考虑极性
static void set_gpio_value_with_polarity(int gpio, enum of_gpio_flags flags, int value)
{
    if (flags & OF_GPIO_ACTIVE_LOW)
        gpio_set_value(gpio, !value);  // 反转逻辑
    else
        gpio_set_value(gpio, value);   // 正常逻辑
}

// 读取GPIO值时考虑极性
static int get_gpio_value_with_polarity(int gpio, enum of_gpio_flags flags)
{
    int value = gpio_get_value(gpio);
    
    if (flags & OF_GPIO_ACTIVE_LOW)
        return !value;  // 反转逻辑
    else
        return value;   // 正常逻辑
}
```

## 6 高级应用与最佳实践

### 6.1 批量GPIO处理

```c
// 批量获取GPIO的示例
static int get_all_gpios(struct device_node *np, struct my_gpio_data *data)
{
    const char *gpio_names[] = {"reset-gpios", "power-gpios", "enable-gpios"};
    int *gpio_nums[] = {&data->reset_gpio, &data->power_gpio, &data->enable_gpio};
    enum of_gpio_flags *gpio_flags[] = {&data->reset_flags, &data->power_flags, &data->enable_flags};
    int i, ret;

    for (i = 0; i < ARRAY_SIZE(gpio_names); i++) {
        *gpio_nums[i] = of_get_named_gpio_flags(np, gpio_names[i], 0, gpio_flags[i]);
        
        if (!gpio_is_valid(*gpio_nums[i])) {
            dev_err(&pdev->dev, "Failed to get %s\n", gpio_names[i]);
            return -EINVAL;
        }

        ret = devm_gpio_request_one(&pdev->dev, *gpio_nums[i], 
                                    GPIOF_OUT_INIT_LOW, gpio_names[i]);
        if (ret) {
            dev_err(&pdev->dev, "Failed to request %s: %d\n", gpio_names[i], ret);
            return ret;
        }
    }

    return 0;
}
```

### 6.2 错误处理与资源管理

> [!tip]+ 最佳实践 
> __1. 使用devm__ 函数族_*
> 
> - `devm_gpio_request_one()` - 自动资源管理
> - `devm_gpiod_get()` - 推荐的新式GPIO描述符API
> 
> **2. 错误处理**
> 
> - 使用 `gpio_is_valid()` 检查GPIO号有效性
> - 处理 `of_get_named_gpio_flags()` 的返回值
> - 提供详细的错误信息
> 
> **3. 极性处理**
> 
> - 正确处理 `GPIO_ACTIVE_LOW` 标志
> - 使用封装函数统一处理极性逻辑
> 
> **4. 性能优化**
> 
> - 缓存GPIO状态，避免频繁读取
> - 批量操作多个GPIO时使用数组方式

### 6.3 调试技巧

```bash
# 查看系统中的GPIO状态
cat /sys/kernel/debug/gpio

# 查看GPIO控制器信息
ls /sys/class/gpio/

# 通过sysfs手动控制GPIO（测试用）
echo 100 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio100/direction
echo 1 > /sys/class/gpio/gpio100/value
```

## 7 总结

**OF GPIO函数提供了强大的设备树GPIO操作能力：**

> [!success]+ 核心要点 
> **1. 函数使用**
> 
> - `of_get_named_gpio_flags()` 是核心函数
> - 正确处理返回值和错误码
> - 注意GPIO极性和标志的处理
> 
> **2. 设备树配置**
> 
> - 正确配置GPIO控制器属性
> - 使用标准的GPIO属性命名规范
> - 设置合适的GPIO标志
> 
> **3. 驱动开发**
> 
> - 使用devm_*函数进行资源管理
> - 实现完善的错误处理机制
> - 考虑GPIO的极性和电气特性
> 
> **4. 内核配置**
> 
> - 确保启用必要的CONFIG选项
> - 根据硬件平台选择合适的GPIO驱动

通过本文的学习，您应该能够熟练地在设备树中配置GPIO资源，并在驱动程序中正确地获取和操作这些GPIO。这些技能是嵌入式Linux驱动开发的重要基础。