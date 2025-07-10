📢 本篇将详细介绍`LED`子系统。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cc261fc16db2789ed8497928ae1d4ce8.png)

## 设备树编写

节点属性添加参考内核源码`Documentation/devicetree/bindings/leds/leds-gpio.txt`文件 设备树插件如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/aa4e495f8b8e654c3aa2b1534b555659.png)


- 第1-2行： `LED`灯设备节点使用到的头文件
    
- 第7行： 可选的属性，label一般表示led的名字
    
- 第8行： 可选的属性，表示默认设置`led`打开“`on`”，还可以设置为“`off`”或“`keep`”
    
- 第9行： 实际的引脚，可以根据具体板卡的原理图，然后替换
    

rockchip/rk3568-lubancat-2.dtsi
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dba57d1e94f0e7ef975cbda9eeb67d60.png)

属性的解析：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fdf389cf4cb12e48b31c5139b39ea2d3.png)


  

## LED子系统

### 用户态

在用户态，可以直接访问节点`/sys/class/leds/xxx/`下的文件操作`LED`。 在该路径下会有`brightness`、`max_brightness`、`trigger`，根据`trigger`的不同，还可能有`delay_off`、`delay_on`、`invert`等。

  

- `brightness`用于设置`LED`亮度，范围为`0(LED_OFF)~255(LED_FULL)`;
    
- `max_brightness`用于显示`LED`的最大亮度访问，一般值为`255`;
    
- `trigger`用于设置`LED`的触发模式，通常可选的有:`none`、`nand-disk`、`mmc0`、`cpu0`、`heartbeat`、`timer`、`default-on`、`oneshot`、`backlight`、`gpio`；
    
- `delay_off`、`delay_on`用于`trigger`为`timer`等模式时，`LED`亮灭的时间，单位为毫秒；
    

### 内核驱动

  `LED`子系统的驱动都在`drivers/leds/`下面。 核心的文件是`led-class.c`、`led-core.c`、`led-triggers.c`； 可选触发器方式有：
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

下面对主要文件的内容进行分析：

- `led-core.c`： 抽象出`LED`操作逻辑，封装成函数导出，供其它文件使用： `led_init_core()`：核心初始化； `led_blink_set()`：设置led闪烁时间； `led_blink_set_oneshot()`：闪烁一次； `led_stop_software_blink()`：led停止闪烁； `led_set_brightness()`：设置led的亮度； `led_set_brightness_nopm`：如果可以休眠，设置led的亮度保证休眠可用； `led_set_brightness_nosleep`：如果没有休眠，设置led的亮度； `led_set_brightness_sync`：设置led亮度同步； `led_update_brightness`：更新亮度； `led_sysfs_disable`：用户态关闭； `led_sysfs_enable`：用户态打开； `leds_list`：leds链表； `leds_list_lock`：leds链表锁；
    
- `led-class.c`： 维护`LED`子系统的所有`LED`设备，为`LED`设备提供 注册操作函数:`led_classdev_register()`/`of_led_classdev_register()`/`devm_of_led_classdev_register()`/ `devm_led_classdev_register()`； 注销操作函数:`led_classdev_unregister()`/`devm_led_classdev_unregister()`； 电源管理的休眠和恢复操作函数：`led_classdev_suspend()`、`led_classdev_resume()`； 同时提供基本的用户态操作接口，`brightness`、`max_brightness`等；
    
- `led-triggers.c`： 维护`LED`子系统的所有触发器，为触发器提供 注册操作函数：`led_trigger_register()`、`devm_led_trigger_register()`，`led_trigger_register_simple()`； 注销操作函数：`led_trigger_unregister()`、`led_trigger_unregister_simple()`； 以及其它触发器相关的操作函数；
    
- `ledtrig-timer.c` `ledtrig-xxxx.c`： 以`ledtrig-timer.c`为例的触发器，入口函数调用`led_trigger_register()`注册触发器，注册时候传入`led_trigger`结构体，里面有`activate`和`deactivate`成员函数指针，这里的作用是生成`delay_on`、`delay_off`文件； 同时还提供`delay_on`和`delay_off`的用户态操作接口； 卸载时，使用`led_trigger_unregister()`注销触发器。
    
- `leds-gpio.c` `leds-xxxx.c`： 以`leds-gpio.c`为例的`LED`设备，在通过设备树或者其它途径匹配到设备信息后，将调用`probe()`函数，然后再根据设备信息设置`led_classdev`，最后调用`devm_of_led_classdev_register()`注册LED设备。
    

## 驱动代码

### 平台设备驱动的注册

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6465ac388215fa4d23bd6a685c55a70c.png)

- 第1-4行： `of_match_table`表用于驱动和设备匹配，这里将匹配主设备树`leds`节点。
    
- 第5行： 宏`MODULE_DEVICE_TABLE(gtype,name)`，在这里该宏会会生成一个符号表，把该设备加入到设备队列中，该方式支持使用平台设备匹配驱动。
    
- 第7-14行： 定义平台驱动结构体，填充`of_match_table`段，设备树的方式匹配驱动。
    
- 第16行： 使用`module_platform_driver`宏注册平台驱动，该led驱动是一个`platform_driver`的对象。
    

关于`module_platform_driver()`宏，在内核源码`/include/linux/platform_device.h`中有定义：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4886a23647466704f03abd42ef96964f.png)


继续展开`module_driver()`宏：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9dc3b1bc3260495963c52d86e2fd0602.png)


可以看到，最终该宏是执行`module_init()`和`module_exit()`进行驱动模块的注册和注销，其中””表示前后两部分粘和起来。

### 平台设备驱动的probe

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3220c9f8ced562ff4ad020c181ca5fbe.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3b9fabd25edeb63932a0636d1c94cb7b.png)


create_gpio_led

-- devm_of_led_classdev_register

drivers/leds/led-class.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c9945103aa4817f4cbf5cb66d1b5180e.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ee25c90be6f5ed47b6f7f41c12adabdc.png)


led_update_brightness

led_init_core

led-core.c

## 数据结构

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3d450dd5c59b3fb69fa930736cd6e351.png)


## 使用方法

重启加载设备树后，驱动已经编译进内核，会在`/sys/class/leds`目录下多个`led_test`目录。 进入`led_test`目录

led-class.c
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/625a0ff3b529e370df8bca33efa20d08.png)


```C
brightness      max_brightness  subsystem       uevent
device          power           trigger
```

可以通过`echo 0 > brightness`来关闭`LED`灯，通过`echo timer > trigger`改变触发模式。 同时，`cat trigger`可以显示支持的触发模式，以及当前的触发模式：

```C
none nand-disk mmc0 cpu0 [heartbeat] timer default-on oneshot backlight gpio
```