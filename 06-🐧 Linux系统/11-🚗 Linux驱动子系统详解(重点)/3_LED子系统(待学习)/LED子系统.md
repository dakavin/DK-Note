
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cc261fc16db2789ed8497928ae1d4ce8.png)

## 1 引言：什么是LED子系统

在嵌入式Linux系统中，控制一个简单的LED灯看似容易，但如果每个驱动都自己实现LED控制逻辑，会造成代码重复和接口不统一。Linux内核的**LED子系统**就是为了解决这个问题而设计的。

简单来说，LED子系统就像一个"LED管理中心"，它：

- 为所有LED设备提供统一的用户接口
- 支持多种LED触发模式（闪烁、心跳、CPU活动指示等）
- 简化LED驱动的开发工作

> [!tip]+ 理解要点 把LED子系统想象成一个"智能开关面板"：你不需要知道每个LED的具体接线，只需要通过统一的接口就能控制它们的亮灭、闪烁模式等。

## 2 用户态接口：快速上手LED控制

### 2.1 sysfs接口概览

Linux LED子系统通过`sysfs`文件系统向用户空间暴露接口。每个注册到系统的LED都会在`/sys/class/leds/`目录下创建一个子目录。

重启加载设备树后，驱动已经编译进内核，会在`/sys/class/leds`目录下多个`led_test`目录。进入`led_test`目录：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/625a0ff3b529e370df8bca33efa20d08.png)

```C
brightness      max_brightness  subsystem       uevent
device          power           trigger
```

### 2.2 核心文件详解

在用户态，可以直接访问节点`/sys/class/leds/xxx/`下的文件操作`LED`。在该路径下会有`brightness`、`max_brightness`、`trigger`，根据`trigger`的不同，还可能有`delay_off`、`delay_on`、`invert`等。

- `brightness`用于设置`LED`亮度，范围为`0(LED_OFF)~255(LED_FULL)`;
- `max_brightness`用于显示`LED`的最大亮度访问，一般值为`255`;
- `trigger`用于设置`LED`的触发模式，通常可选的有:`none`、`nand-disk`、`mmc0`、`cpu0`、`heartbeat`、`timer`、`default-on`、`oneshot`、`backlight`、`gpio`；
- `delay_off`、`delay_on`用于`trigger`为`timer`等模式时，`LED`亮灭的时间，单位为毫秒；

### 2.3 基本操作示例

```bash
# 1. 点亮LED（设置为最亮）
echo 255 > /sys/class/leds/led_test/brightness

# 2. 熄灭LED
echo 0 > /sys/class/leds/led_test/brightness

# 3. 设置中等亮度
echo 128 > /sys/class/leds/led_test/brightness

# 4. 查看当前支持的触发模式
cat /sys/class/leds/led_test/trigger
# 输出示例：[none] nand-disk mmc0 cpu0 heartbeat timer default-on oneshot backlight gpio
# 方括号[]表示当前选中的模式
```

可以通过`echo 0 > brightness`来关闭`LED`灯，通过`echo timer > trigger`改变触发模式。同时，`cat trigger`可以显示支持的触发模式，以及当前的触发模式：

```C
none nand-disk mmc0 cpu0 [heartbeat] timer default-on oneshot backlight gpio
```

```bash
# 5. 设置为心跳模式（像心跳一样闪烁）
echo heartbeat > /sys/class/leds/led_test/trigger

# 6. 设置为定时器模式
echo timer > /sys/class/leds/led_test/trigger
# 此时会出现两个新文件：delay_on 和 delay_off

# 7. 设置闪烁时间（单位：毫秒）
echo 500 > /sys/class/leds/led_test/delay_on   # 亮500ms
echo 500 > /sys/class/leds/led_test/delay_off  # 灭500ms
```

> [!example]+ 动手实践 试试创建一个脚本，让LED实现"SOS"求救信号的闪烁模式（三短三长三短）。提示：使用brightness文件配合sleep命令。

### 2.4 触发模式详解

LED子系统支持多种触发模式，每种模式都有其特定用途：

|触发模式|说明|应用场景|
|---|---|---|
|`none`|手动控制模式|通过brightness文件直接控制|
|`heartbeat`|心跳模式|系统运行状态指示|
|`timer`|定时器模式|自定义闪烁频率|
|`cpu0`|CPU活动指示|显示CPU负载情况|
|`mmc0`|存储设备活动|SD卡读写指示|
|`default-on`|默认常亮|电源指示灯|
|`oneshot`|单次闪烁|事件通知|

> [!note]+ 背景知识 Linux LED子系统在2.6.21版本引入，统一了不同平台的GPIO操作接口。在此之前，每个平台都有自己的LED控制方式，造成了代码的碎片化。

## 3 设备树配置：让内核识别你的LED

### 3.1 什么是设备树

设备树（Device Tree）就像是硬件的"说明书"，告诉Linux内核：

- 系统中有哪些硬件设备
- 这些设备连接在哪里
- 设备的属性是什么

### 3.2 LED设备树配置示例

节点属性添加参考内核源码`Documentation/devicetree/bindings/leds/leds-gpio.txt`文件。设备树插件如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/aa4e495f8b8e654c3aa2b1534b555659.png)

- 第1-2行：`LED`灯设备节点使用到的头文件
- 第7行：可选的属性，label一般表示led的名字
- 第8行：可选的属性，表示默认设置`led`打开"`on`"，还可以设置为"`off`"或"`keep`"
- 第9行：实际的引脚，可以根据具体板卡的原理图，然后替换

### 3.3 实际应用：RK3568平台示例

rockchip/rk3568-lubancat-2.dtsi：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dba57d1e94f0e7ef975cbda9eeb67d60.png)

### 3.4 属性解析过程

属性的解析：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fdf389cf4cb12e48b31c5139b39ea2d3.png)

内核在解析LED设备树节点时，会按以下步骤处理各个属性：

1. **compatible属性匹配**：驱动通过`compatible = "gpio-leds"`进行匹配
2. **子节点遍历**：解析每个LED配置
3. **属性读取优先级**：
    - `label` → 如果存在，用作LED名称
    - `linux,default-trigger` → 设置默认触发器
    - `default-state` → 设置初始状态
    - `gpios` → GPIO引脚配置（必需）

> [!warning]+ 重要提醒 GPIO编号在不同平台可能不同，需要查看具体的硬件手册。比如RK3568平台的GPIO3_C5对应的设备树表示为`<&gpio3 RK_PC5>`。

## 4 内核驱动架构：LED子系统的内部机制

### 4.1 整体架构概览

`LED`子系统的驱动都在`drivers/leds/`下面。核心的文件是`led-class.c`、`led-core.c`、`led-triggers.c`；可选触发器方式有：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/48c57e38b591aae4e8674826971adfbb.png)

```C
obj-$(CONFIG_LEDS_TRIGGER_TIMER)        += ledtrig-timer.o      //定时触发
obj-$(CONFIG_LEDS_TRIGGER_ONESHOT)      += ledtrig-oneshot.o    //单次触发
obj-$(CONFIG_LEDS_TRIGGER_IDE_DISK)     += ledtrig-ide-disk.o   //硬盘触发
obj-$(CONFIG_LEDS_TRIGGER_HEARTBEAT)    += ledtrig-heartbeat.o  //心跳触发
obj-$(CONFIG_LEDS_TRIGGER_BACKLIGHT)    += ledtrig-backlight.o  //背光设置
obj-$(CONFIG_LEDS_TRIGGER_GPIO)         += ledtrig-gpio.o       //GPIO触发
obj-$(CONFIG_LEDS_TRIGGER_CPU)          += ledtrig-cpu.o        //CPU触发     
obj-$(CONFIG_LEDS_TRIGGER_DEFAULT_ON)   += ledtrig-default-on.o //默认开
obj-$(CONFIG_LEDS_TRIGGER_TRANSIENT)    += ledtrig-transient.o  
obj-$(CONFIG_LEDS_TRIGGER_CAMERA)       += ledtrig-camera.o
```

`leds-gpio.c`和`leds-xxx.c`对应具体的设备，比如现在某个`LED`接在了`GPIO`上，理论上我们就要编写`leds-gpio.c`这个驱动文件，但`LED`子系统已经帮我们做好了，我们只需要在设备树文件添加相应的设备信息即可。

### 4.2 核心文件职责

下面对主要文件的内容进行分析：

- **`led-core.c`**：抽象出`LED`操作逻辑，封装成函数导出，供其它文件使用：
    
    - `led_init_core()`：核心初始化；
    - `led_blink_set()`：设置led闪烁时间；
    - `led_blink_set_oneshot()`：闪烁一次；
    - `led_stop_software_blink()`：led停止闪烁；
    - `led_set_brightness()`：设置led的亮度；
    - `led_set_brightness_nopm`：如果可以休眠，设置led的亮度保证休眠可用；
    - `led_set_brightness_nosleep`：如果没有休眠，设置led的亮度；
    - `led_set_brightness_sync`：设置led亮度同步；
    - `led_update_brightness`：更新亮度；
    - `led_sysfs_disable`：用户态关闭；
    - `led_sysfs_enable`：用户态打开；
    - `leds_list`：leds链表；
    - `leds_list_lock`：leds链表锁；
- **`led-class.c`**：维护`LED`子系统的所有`LED`设备，为`LED`设备提供
    
    - 注册操作函数：`led_classdev_register()`/`of_led_classdev_register()`/`devm_of_led_classdev_register()`/`devm_led_classdev_register()`；
    - 注销操作函数：`led_classdev_unregister()`/`devm_led_classdev_unregister()`；
    - 电源管理的休眠和恢复操作函数：`led_classdev_suspend()`、`led_classdev_resume()`；
    - 同时提供基本的用户态操作接口，`brightness`、`max_brightness`等；
- **`led-triggers.c`**：维护`LED`子系统的所有触发器，为触发器提供
    
    - 注册操作函数：`led_trigger_register()`、`devm_led_trigger_register()`，`led_trigger_register_simple()`；
    - 注销操作函数：`led_trigger_unregister()`、`led_trigger_unregister_simple()`；
    - 以及其它触发器相关的操作函数；
- **`ledtrig-timer.c` `ledtrig-xxxx.c`**：以`ledtrig-timer.c`为例的触发器，入口函数调用`led_trigger_register()`注册触发器，注册时候传入`led_trigger`结构体，里面有`activate`和`deactivate`成员函数指针，这里的作用是生成`delay_on`、`delay_off`文件；同时还提供`delay_on`和`delay_off`的用户态操作接口；卸载时，使用`led_trigger_unregister()`注销触发器。
    
- **`leds-gpio.c` `leds-xxxx.c`**：以`leds-gpio.c`为例的`LED`设备，在通过设备树或者其它途径匹配到设备信息后，将调用`probe()`函数，然后再根据设备信息设置`led_classdev`，最后调用`devm_of_led_classdev_register()`注册LED设备。
    

### 4.3 关键数据结构

数据结构：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3d450dd5c59b3fb69fa930736cd6e351.png)

**struct led_classdev** - LED设备的核心结构体，就像是每个LED的"身份证"，包含了LED的所有信息和操作方法：

```c
struct led_classdev {
    const char *name;                      /* LED设备名称 */
    enum led_brightness brightness;        /* 当前亮度 */
    enum led_brightness max_brightness;    /* 最大亮度 */
    int flags;                            /* 设备标志 */
    
    /* 设置LED亮度的回调函数 */
    void (*brightness_set)(struct led_classdev *led_cdev,
                          enum led_brightness brightness);
    
    /* 获取LED亮度的回调函数 */
    enum led_brightness (*brightness_get)(struct led_classdev *led_cdev);
    
    /* 闪烁设置回调 */
    int (*blink_set)(struct led_classdev *led_cdev,
                     unsigned long *delay_on,
                     unsigned long *delay_off);
    
    struct device *dev;                   /* 设备指针 */
    struct list_head node;                /* 用于链表管理 */
    
    const char *default_trigger;          /* 默认触发器 */
    /* ... 更多成员 ... */
};
```

## 5 驱动代码分析：从注册到工作

### 5.1 平台设备驱动的注册

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6465ac388215fa4d23bd6a685c55a70c.png)

- 第1-4行：`of_match_table`表用于驱动和设备匹配，这里将匹配主设备树`leds`节点。
- 第5行：宏`MODULE_DEVICE_TABLE(gtype,name)`，在这里该宏会会生成一个符号表，把该设备加入到设备队列中，该方式支持使用平台设备匹配驱动。
- 第7-14行：定义平台驱动结构体，填充`of_match_table`段，设备树的方式匹配驱动。
- 第16行：使用`module_platform_driver`宏注册平台驱动，该led驱动是一个`platform_driver`的对象。

### 5.2 module_platform_driver宏展开

关于`module_platform_driver()`宏，在内核源码`/include/linux/platform_device.h`中有定义：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4886a23647466704f03abd42ef96964f.png)

继续展开`module_driver()`宏：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9dc3b1bc3260495963c52d86e2fd0602.png)

可以看到，最终该宏是执行`module_init()`和`module_exit()`进行驱动模块的注册和注销，其中"##"表示前后两部分粘和起来。

换句话说，`module_platform_driver(gpio_led_driver)`最终会生成：

```c
static int __init gpio_led_driver_init(void)
{
    return platform_driver_register(&gpio_led_driver);
}
module_init(gpio_led_driver_init);

static void __exit gpio_led_driver_exit(void)
{
    platform_driver_unregister(&gpio_led_driver);
}
module_exit(gpio_led_driver_exit);
```

### 5.3 平台设备驱动的probe

当设备树中的LED节点与驱动匹配成功后，`probe`函数会被调用：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3220c9f8ced562ff4ad020c181ca5fbe.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3b9fabd25edeb63932a0636d1c94cb7b.png)

### 5.4 create_gpio_led函数调用链

create_gpio_led → devm_of_led_classdev_register

drivers/leds/led-class.c：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c9945103aa4817f4cbf5cb66d1b5180e.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ee25c90be6f5ed47b6f7f41c12adabdc.png)

当调用`devm_led_classdev_register()`时，会触发以下调用链：

```txt
create_gpio_led()
    └── devm_of_led_classdev_register()
        └── devm_led_classdev_register()
            └── led_classdev_register()
                ├── led_classdev_next_name()  // 生成LED名称
                ├── led_update_brightness()    // 更新初始亮度
                ├── led_init_core()           // 初始化核心功能
                └── device_create()           // 创建设备节点
```

led_update_brightness、led_init_core等函数在led-core.c中实现。

> [!tip]+ API函数说明 **devm_led_classdev_register**：注册LED设备（支持自动资源管理）
> 
> ```c
> // include/linux/leds.h
> int devm_led_classdev_register(struct device *parent, struct led_classdev *led_cdev);
> ```
> 
> - `parent`：父设备指针
> - `led_cdev`：LED类设备结构体
> - **返回值**：
>     - 成功：返回0
>     - 失败：返回负的错误码

## 6 实践指南：调试技巧和常见问题

### 6.1 调试LED驱动的步骤

1. **确认设备树加载**

```bash
# 查看设备树是否正确加载
ls /proc/device-tree/leds/
# 应该能看到你配置的LED节点
```

2. **检查驱动加载**

```bash
# 查看LED驱动是否加载
lsmod | grep led
# 或查看内核日志
dmesg | grep -i led
```

3. **验证GPIO状态**

```bash
# 使用gpioinfo查看GPIO状态
gpioinfo | grep -A 5 -B 5 "led"
```

### 6.2 常见问题及解决方案

> [!warning]+ 问题1：LED设备没有出现在/sys/class/leds/下 **可能原因**：
> 
> - 设备树配置错误
> - GPIO被其他驱动占用
> - 驱动没有正确加载
> 
> **解决方法**：
> 
> 1. 检查dmesg中的错误信息
> 2. 确认GPIO没有被其他设备使用
> 3. 检查设备树编译是否有错误

> [!warning]+ 问题2：LED无法控制 **可能原因**：
> 
> - GPIO极性配置错误（ACTIVE_HIGH/ACTIVE_LOW）
> - 硬件连接问题
> - 权限不足
> 
> **解决方法**：
> 
> 1. 尝试切换GPIO极性配置
> 2. 用万用表测量GPIO引脚电平
> 3. 使用sudo或修改文件权限

### 6.3 性能优化建议

1. **使用硬件PWM**：如果需要精确控制LED亮度，考虑使用PWM而不是GPIO
2. **批量操作**：控制多个LED时，尽量批量操作减少系统调用
3. **选择合适的触发器**：使用内核触发器比用户空间轮询更高效

## 7 总结

Linux LED子系统通过统一的框架简化了LED设备的管理：

1. **用户接口统一**：所有LED都通过`/sys/class/leds/`访问
2. **功能丰富**：支持多种触发模式，满足不同应用需求
3. **易于扩展**：添加新的LED设备只需配置设备树
4. **驱动简单**：大部分工作由LED子系统完成

通过本文的学习，你应该能够：

- ✅ 使用sysfs接口控制LED
- ✅ 编写LED设备树配置
- ✅ 理解LED子系统的架构
- ✅ 调试LED相关问题

> [!tip]+ 进阶学习 掌握LED子系统后，可以尝试：
> 
> 1. 编写自定义的LED触发器
> 2. 实现RGB LED的颜色控制
> 3. 将LED与其他子系统联动（如网络、温度监控等）

记住，LED虽小，但它展示了Linux设备驱动的基本思想：**抽象硬件差异，提供统一接口**。这个思想贯穿整个Linux内核！