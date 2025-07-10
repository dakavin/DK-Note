
## 1 引言：从理论到实践的桥梁

学习了Pinctrl子系统的基本概念、平台实现和数据结构后，我们现在需要学会使用实际的"工具"——Pinctrl的API接口，来完成具体的编程任务。

虽然大部分情况下Pinctrl的配置是自动进行的，但在某些特殊场景下，**驱动开发者需要手动调用API来控制引脚状态**。这些API就像是与引脚管理系统的"对话接口"，通过它们我们可以请求引脚资源、切换引脚状态、查询引脚配置等。

**换句话说**，前面的学习让我们理解了Pinctrl子系统"是什么"和"为什么"，现在我们要解决"怎么用"的问题。掌握这些API的使用方法，就掌握了在运行时动态管理引脚的能力。

## 2 Pinctrl API函数详解

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/026fd9dc7345ad72122c79cfa87b49f0.png)

这个图展示了Pinctrl API的主要函数流程：获取→查找→指定的基本使用模式。

### 2.1 pinctrl_get - 获取设备的pinctrl句柄

**pinctrl_get**：用于获取与给定设备相关联的引脚控制器实例。该函数从设备树中解析pinctrl相关配置，为设备分配对应的引脚控制资源。
```c
// include/linux/pinctrl/consumer.h
struct pinctrl *pinctrl_get(struct device *dev);
```
- `dev`：指向设备结构体的指针，表示需要获取引脚控制器的设备对象
- **返回值**：
	- 成功：返回指向`struct pinctrl`的指针，表示获取到的pinctrl实例
	- 失败：返回ERR_PTR类型的错误指针，可通过`IS_ERR()`和`PTR_ERR()`获取错误信息

该函数的功能是根据给定的设备对象`dev`获取与其相关联的`pinctrl`实例。`pinctrl`是Linux内核中用于管理和控制引脚的框架。通过调用该函数，可以获得设备对象所使用的`pinctrl`实例，以便进行引脚配置和控制操作。

### 2.2 pinctrl_put - 释放pinctrl指针

**pinctrl_put**：用于释放由`pinctrl_get()`函数获得的引脚控制器实例，释放相关的引脚控制资源，确保系统资源的正确回收。
```c
// include/linux/pinctrl/consumer.h
void pinctrl_put(struct pinctrl *p);
```
- `p`：指向需要释放的pinctrl实例的指针
- **返回值**：无返回值

该函数的功能是释放由`pinctrl_get()`函数获得的pinctrl实例，以释放相关资源。在使用完pinctrl实例后，调用该函数可以确保正确释放相关资源，避免内存泄漏。

### 2.3 devm_pinctrl_get - 获取设备的pinctrl句柄（设备管理版本）

**devm_pinctrl_get**：用于获取与给定设备相关联的引脚控制器实例，采用设备管理机制。该函数从设备树中解析pinctrl相关配置，为设备分配对应的引脚控制资源，并在设备移除时自动释放。
```c
// include/linux/pinctrl/consumer.h
struct pinctrl *devm_pinctrl_get(struct device *dev);
```
- `dev`：和pinctrl_get一样
- **返回值**：
    - 成功：返回指向`struct pinctrl`的指针，表示获取到的pinctrl实例
    - 失败：返回ERR_PTR类型的错误指针，可通过`IS_ERR()`和`PTR_ERR()`获取错误信息

该函数的功能与`pinctrl_get`相同，都是根据给定的设备对象`dev`获取与其相关联的`pinctrl`实例。不同之处在于`devm_pinctrl_get`采用了设备管理机制(devres)，当设备被移除或驱动卸载时，系统会自动调用`pinctrl_put`释放相关资源，无需手动释放，这样可以有效避免资源泄漏问题。

### 2.4 pinctrl_lookup_state - 查找引脚状态

**pinctrl_lookup_state**：用于在给定的引脚控制器实例中查找指定名称的引脚控制状态。该函数的主要作用是根据状态名称获取对应的pinctrl状态对象。
```c
// include/linux/pinctrl/consumer.h
struct pinctrl_state *pinctrl_lookup_state(struct pinctrl *p, const char *name);
```
- `p`：指向pinctrl实例的指针，表示要进行状态查找的pinctrl
- `name`：指向状态名称的字符串指针，表示要查找的状态名称
- **返回值**：
	- 成功：返回指向`struct pinctrl_state`的指针，表示找到的pinctrl状态
	- 失败：返回ERR_PTR类型的错误指针，通常是`-ENODEV`（状态不存在）

该函数的功能是在给定的pinctrl实例p中查找指定名称的pinctrl状态。pinctrl状态是与引脚相关的配置和控制状态，例如引脚模式、电气属性等。

### 2.5 pinctrl_select_state - 激活指定状态

**pinctrl_select_state**：用于将指定的引脚控制状态设置到硬件上。该函数的主要作用是激活特定的pinctrl状态配置，使硬件引脚切换到指定的功能模式。
```c
// include/linux/pinctrl/consumer.h
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
```
- `p`：指向pinctrl实例的指针，表示要进行状态设置的pinctrl
- `s`：指向pinctrl_state实例的指针，表示要设置的目标状态
- **返回值**：
	- 成功：返回0
	- 失败：返回负值错误码，如`-EINVAL`（无效参数）、`-EBUSY`（引脚被占用）

该函数的功能是将指定的pinctrl状态s设置到硬件上。pinctrl状态是与引脚相关的配置和控制状态，例如引脚模式、电气属性等。

## 3 实际应用案例分析

### 3.1 案例1：OV5670摄像头传感器的引脚状态管理

让我们分析一个真实的摄像头传感器驱动中的pinctrl使用：

**1. 先查找状态**：使用`pinctrl_lookup_state`函数查找可用的引脚状态：
```c
// drivers/media/i2c/ov5670.c 1889-1902
	ov5670->pinctrl = devm_pinctrl_get(dev);  // 获取设备的引脚控制句柄
	if (!IS_ERR(ov5670->pinctrl)) {  // 如果成功获取pinctrl
		ov5670->pins_default =
			pinctrl_lookup_state(ov5670->pinctrl,
					     OF_CAMERA_PINCTRL_STATE_DEFAULT);  // 查找默认引脚状态
		if (IS_ERR(ov5670->pins_default))
			dev_err(dev, "could not get default pinstate\n");  // 获取默认状态失败时报错
		ov5670->pins_sleep =
			pinctrl_lookup_state(ov5670->pinctrl,
					     OF_CAMERA_PINCTRL_STATE_SLEEP);  // 查找睡眠引脚状态
		if (IS_ERR(ov5670->pins_sleep))
			dev_err(dev, "could not get sleep pinstate\n");  // 获取睡眠状态失败时报错
	}
```

**2. 再指定状态**：在电源管理函数中使用`pinctrl_select_state`切换引脚状态：
```c
// drivers/media/i2c/ov5670.c 1344-1364
// 关闭OV5670摄像头电源的内部函数
static void __ov5670_power_off(struct ov5670 *ov5670)
{
	int ret;
	// 获取设备指针
	struct device *dev = &ov5670->client->dev;

	// 如果电源关闭的GPIO有效，拉低GPIO进入关闭状态
	if (!IS_ERR(ov5670->pwdn_gpio))
		gpiod_set_value_cansleep(ov5670->pwdn_gpio, 0);

	// 禁用并取消准备外部时钟
	clk_disable_unprepare(ov5670->xvclk);

	// 如果复位GPIO有效，拉低复位GPIO
	if (!IS_ERR(ov5670->reset_gpio))
		gpiod_set_value_cansleep(ov5670->reset_gpio, 0);

	// 如果睡眠引脚状态配置有效
	if (!IS_ERR_OR_NULL(ov5670->pins_sleep)) {
		// 切换引脚到睡眠状态
		ret = pinctrl_select_state(ov5670->pinctrl,
					   ov5670->pins_sleep);
		if (ret < 0)
			dev_dbg(dev, "could not set pins\n");
	}
	
	// 如果电源控制GPIO有效，拉低GPIO关闭电源
	if (!IS_ERR(ov5670->power_gpio))
		gpiod_set_value_cansleep(ov5670->power_gpio, 0);

	// 批量禁用所有电源调节器
	regulator_bulk_disable(OV5670_NUM_SUPPLIES, ov5670->supplies);
}
```

这个例子展示了摄像头传感器如何通过pinctrl API在关机时将引脚切换到睡眠状态，实现：GPIO复用功能改变，达到省电的目的。

### 3.2 案例2：电源管理芯片的多状态控制

对于需要更复杂状态管理的设备，如**电源管理芯片**，可以定义多种状态：
```dts
// 设备树中的多状态定义
&rk817 {
    pinctrl-names = "default", "pmic-sleep", "pmic-power-off", "pmic-reset";
    pinctrl-0 = <&pmic_int>;                           // 默认状态
    pinctrl-1 = <&soc_slppin_slp>, <&rk817_slppin_slp>;   // 睡眠状态
    pinctrl-2 = <&soc_slppin_gpio>, <&rk817_slppin_pwrdn>; // 关机状态
    pinctrl-3 = <&soc_slppin_gpio>, <&rk817_slppin_rst>;   // 复位状态
};
```

对应的驱动代码可能是这样的：
```c
static int pmic_set_power_state(struct device *dev, const char *state_name)
{
    struct pinctrl *pinctrl;
    struct pinctrl_state *state;
    int ret;

    // 获取pinctrl实例
    pinctrl = dev_get_drvdata(dev);
    if (!pinctrl) {
        dev_err(dev, "No pinctrl data\n");
        return -ENODEV;
    }

    // 查找指定状态
    state = pinctrl_lookup_state(pinctrl, state_name);
    if (IS_ERR(state)) {
        dev_err(dev, "Cannot find %s state\n", state_name);
        return PTR_ERR(state);
    }

    // 应用状态
    ret = pinctrl_select_state(pinctrl, state);
    if (ret) {
        dev_err(dev, "Failed to select %s state: %d\n", state_name, ret);
        return ret;
    }

    dev_info(dev, "Switched to %s state\n", state_name);
    return 0;
}

// 系统睡眠时调用
static int pmic_suspend(struct device *dev)
{
    return pmic_set_power_state(dev, "pmic-sleep");
}

// 系统唤醒时调用
static int pmic_resume(struct device *dev)
{
    return pmic_set_power_state(dev, "default");
}

// 系统关机时调用
static void pmic_shutdown(struct platform_device *pdev)
{
    pmic_set_power_state(&pdev->dev, "pmic-power-off");
}
```

## 4 Pinctrl API内部实现分析

让我们深入了解`pinctrl_select_state`函数的内部实现：
```c
/**
 * pinctrl_select_state() - select/activate/program a pinctrl state to HW
 * @p: the pinctrl handle for the device that requests configuration
 * @state: the state handle to select/activate/program
 */
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *state)
{
		// 如果当前状态与目标状态相同，直接返回，无需切换
        if (p->state == state)
                return 0;

		// 提交新的引脚状态，进行切换
        return pinctrl_commit_state(p, state);
}
EXPORT_SYMBOL_GPL(pinctrl_select_state);
```
- 这个函数首先检查当前状态是否已经是目标状态，如果是则直接返回。否则调用`pinctrl_commit_state`来实际执行状态切换

```c
// drivers/pinctrl/core.c
static int pinctrl_commit_state(struct pinctrl *p, struct pinctrl_state *state)
{
	// 引脚设置结构体指针
	struct pinctrl_setting *setting, *setting2;
	// 保存旧的引脚状态
	struct pinctrl_state *old_state = p->state;
	int ret;

	// 如果存在需要切换状态
	if (p->state) {
		/*
		 * For each pinmux setting in the old state, forget SW's record
		 * of mux owner for that pingroup. Any pingroups which are
		 * still owned by the new state will be re-acquired by the call
		 * to pinmux_enable_setting() in the loop below.
		 */
		 // 遍历旧状态的所有设置
		list_for_each_entry(setting, &p->state->settings, node) {
			// 如果不是引脚复用组类型，则跳过当前循环
			if (setting->type != PIN_MAP_TYPE_MUX_GROUP)
				continue;
			// 禁用旧的引脚复用设置
			pinmux_disable_setting(setting);
		}
	}

	// 清空当前状态指针
	p->state = NULL;
	
	// 应用新状态的所有设置...
	// 这里会调用具体平台的set_mux等函数
}
```

这个函数展示了状态切换的核心逻辑：先禁用旧状态的设置，然后应用新状态的设置。

## 5 常用编程模式

### 5.1 模式1：基本的引脚状态管理

这是最常见的使用模式，适用于大多数设备：
```c
struct my_device {
    struct device *dev;
    struct pinctrl *pinctrl;
    struct pinctrl_state *default_state;
    struct pinctrl_state *sleep_state;
    // 其他设备特定数据...
};

static int my_device_probe(struct platform_device *pdev)
{
    struct my_device *mydev;
    int ret;

    mydev = devm_kzalloc(&pdev->dev, sizeof(*mydev), GFP_KERNEL);
    if (!mydev)
        return -ENOMEM;

    mydev->dev = &pdev->dev;

    // 获取pinctrl句柄
    mydev->pinctrl = devm_pinctrl_get(&pdev->dev);
    if (IS_ERR(mydev->pinctrl)) {
        dev_err(&pdev->dev, "Failed to get pinctrl\n");
        return PTR_ERR(mydev->pinctrl);
    }

    // 查找状态
    mydev->default_state = pinctrl_lookup_state(mydev->pinctrl, "default");
    if (IS_ERR(mydev->default_state)) {
        dev_err(&pdev->dev, "Cannot find default state\n");
        return PTR_ERR(mydev->default_state);
    }

    mydev->sleep_state = pinctrl_lookup_state(mydev->pinctrl, "sleep");
    if (IS_ERR(mydev->sleep_state)) {
        dev_info(&pdev->dev, "No sleep state available\n");
        mydev->sleep_state = NULL;
    }

    // 应用默认状态（通常系统已经自动应用了）
    ret = pinctrl_select_state(mydev->pinctrl, mydev->default_state);
    if (ret) {
        dev_err(&pdev->dev, "Failed to select default state\n");
        return ret;
    }

    platform_set_drvdata(pdev, mydev);
    return 0;
}
```

### 5.2 模式2：运行时动态状态切换

适用于需要根据工作状态动态调整引脚配置的设备：
```c
static int my_device_set_mode(struct my_device *mydev, int mode)
{
    struct pinctrl_state *target_state;
    const char *state_name;
    int ret;

    switch (mode) {
    case DEVICE_MODE_ACTIVE:
        target_state = mydev->default_state;
        state_name = "default";
        break;
    case DEVICE_MODE_SLEEP:
        target_state = mydev->sleep_state;
        state_name = "sleep";
        break;
    case DEVICE_MODE_LOW_POWER:
        target_state = mydev->low_power_state;
        state_name = "low-power";
        break;
    default:
        dev_err(mydev->dev, "Invalid mode: %d\n", mode);
        return -EINVAL;
    }

    if (!target_state) {
        dev_err(mydev->dev, "State %s not available\n", state_name);
        return -ENODEV;
    }

    ret = pinctrl_select_state(mydev->pinctrl, target_state);
    if (ret) {
        dev_err(mydev->dev, "Failed to switch to %s state: %d\n", 
                state_name, ret);
        return ret;
    }

    dev_info(mydev->dev, "Switched to %s mode\n", state_name);
    return 0;
}
```

### 5.3 模式3：错误处理和回滚

完善的错误处理是健壮驱动的重要特征：
```c
static int my_device_configure_pins(struct my_device *mydev)
{
    struct pinctrl_state *old_state;
    int ret;

    // 保存当前状态，以便出错时回滚
    old_state = mydev->current_state;

    // 尝试切换到新状态
    ret = pinctrl_select_state(mydev->pinctrl, mydev->target_state);
    if (ret) {
        dev_err(mydev->dev, "Failed to select target state: %d\n", ret);
        
        // 尝试回滚到原来的状态
        if (old_state) {
            int rollback_ret = pinctrl_select_state(mydev->pinctrl, old_state);
            if (rollback_ret) {
                dev_err(mydev->dev, "Failed to rollback state: %d\n", 
                        rollback_ret);
            }
        }
        return ret;
    }

    // 更新当前状态记录
    mydev->current_state = mydev->target_state;
    return 0;
}
```

## 6 调试和问题排查

### 6.1 常见错误及解决方法

1. **获取pinctrl失败**
```c
pinctrl = devm_pinctrl_get(dev);
if (IS_ERR(pinctrl)) {
    if (PTR_ERR(pinctrl) == -EPROBE_DEFER) {
        dev_info(dev, "Pinctrl not ready, deferring probe\n");
    } else {
        dev_err(dev, "Failed to get pinctrl: %ld\n", PTR_ERR(pinctrl));
    }
    return PTR_ERR(pinctrl);
}
```

2. **状态查找失败**
```c
state = pinctrl_lookup_state(pinctrl, "custom-state");
if (IS_ERR(state)) {
    if (PTR_ERR(state) == -ENODEV) {
        dev_info(dev, "Custom state not defined in DT\n");
        // 使用默认状态作为备选
        state = mydev->default_state;
    } else {
        dev_err(dev, "Error looking up custom state: %ld\n", PTR_ERR(state));
        return PTR_ERR(state);
    }
}
```

### 6.2 调试方法

可以通过内核的调试接口来检查pinctrl的状态：
```bash
# 查看系统中的pinctrl设备（控制器）
cat /sys/kernel/debug/pinctrl/pinctrl-devices

# 查看设备的pinctrl使用情况
cat /sys/kernel/debug/pinctrl/pinctrl-handles

# 查看具体控制器的引脚状态
cat /sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pins
```

## 7 使用注意事项

### 7.1 资源管理

推荐使用`devm_`版本的函数，让系统自动管理资源：
```c
// 推荐：使用devm版本
struct pinctrl *pinctrl = devm_pinctrl_get(dev);

// 不推荐：手动管理资源
struct pinctrl *pinctrl = pinctrl_get(dev);
// 需要在出错时手动调用 pinctrl_put(pinctrl);
```

### 7.2 状态切换时机

**不要在中断上下文中调用pinctrl API**，因为这些函数可能会睡眠：
```c
// 错误：在中断中直接调用
static irqreturn_t my_interrupt_handler(int irq, void *data)
{
    // pinctrl_select_state(pinctrl, sleep_state); // 这是错误的！
    
    // 正确：使用工作队列
    schedule_work(&my_work);
    return IRQ_HANDLED;
}

static void my_work_handler(struct work_struct *work)
{
    // 在工作队列中可以安全调用
    pinctrl_select_state(pinctrl, sleep_state);
}
```

### 7.3 错误码检查

所有API调用都要检查返回值：
```c
struct pinctrl_state *state;
int ret;

state = pinctrl_lookup_state(pinctrl, "sleep");
if (IS_ERR(state)) {
    dev_warn(dev, "Sleep state not available: %ld\n", PTR_ERR(state));
    return PTR_ERR(state);
}

ret = pinctrl_select_state(pinctrl, state);
if (ret) {
    dev_err(dev, "Failed to select sleep state: %d\n", ret);
    return ret;
}
```

> [!tip]+ 实用技巧 
> 在开发过程中，可以通过查看`/sys/kernel/debug/pinctrl/pinctrl-handles`文件来了解系统中各设备的引脚使用情况，有助于发现引脚冲突问题。

> [!warning]+ 重要提醒 
> 不要在中断上下文中调用pinctrl API，因为这些函数可能会睡眠。如果需要在中断中切换引脚状态，应该使用工作队列或其他延迟执行机制。

> [!example]+ 动手实践 
> 尝试为自己的驱动添加多种引脚状态支持，实现在不同工作模式下的引脚配置切换，并添加完善的错误处理机制。

## 8 总结

Pinctrl API的使用遵循几个重要原则：
1. **资源管理优先**：尽量使用`devm_`版本的函数，让系统自动管理资源
2. **错误处理完善**：所有API调用都要检查返回值，并提供适当的错误处理
3. **状态管理清晰**：明确记录当前状态，避免状态混乱
4. **调试信息丰富**：提供足够的调试信息，便于问题排查

**换句话说**，正确使用Pinctrl API不仅能让我们的驱动更加健壮，还能提高系统的整体稳定性。虽然大多数情况下引脚配置是自动的，但掌握这些API的使用方法，让我们在面对复杂的引脚管理需求时游刃有余。

通过本章的学习，我们掌握了Pinctrl API的完整使用方法，包括基本的API调用、实际应用案例、常用编程模式和调试技巧。这些知识为我们开发高质量的Linux驱动程序提供了坚实的基础。