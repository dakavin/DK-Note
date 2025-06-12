
目录：linux/of_gpio.h

宏：CONFIG_OF_GPIO

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a23fbdf216f202095f91f5094f7db47a.png)

## API介绍

先通过of_get_named_gpio_flags获取设备树中的配置信息指定某一个gpio_pin

目录：drivers/gpio/gpiolib-of.c

  

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca171133dc7d7b22c852a7a034c3f78c.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/830ddb08d48e12f06dbbed41873d3a4c.png)


## 案例

设备树：

条件- 状态status 等于 okay

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ef62d76276c1fc761732d034ec64c481.png)


驱动解析

input/touchscreen/gslx680_pad.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d7f88315796efd6693d146005f3c850e.png)


**放到deconfig**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/56d2f070b4beb37bb0266736b1a86715.png)
