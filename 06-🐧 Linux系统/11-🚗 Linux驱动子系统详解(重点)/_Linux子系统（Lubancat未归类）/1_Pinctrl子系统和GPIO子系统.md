在前面章节，我们有过使用寄存器去编写字符设备的经历了。这种直接在驱动代码中， 通过寄存器映射来对外设进行使用的编程方式，从驱动开发者的角度可以说是灾难。因为每当芯片的寄存器发生了改动，那么底层的驱动几乎得重写。

那么在这个问题上，我们更进了一步，学会了使用设备树来描述外设的各种信息(比如寄存器地址)，而不是将寄存器的这些内容放在驱动代码里。 这样即使设备信息修改了，我们还是可以通过设备树的接口函数，去灵活的获取设备的信息。

那么，在驱动中有没有更通用的方法，可以不涉及到具体的寄存器操作的内容呢？可以使用驱动框架，内核针对每个种类的驱动设计一套成熟的、标准的、典型的驱动实现， 并把不同厂家的同类硬件驱动中相同的部分抽出来自己实现好，再把不同部分留出接口给具体的驱动开发工程师来实现。标准化的驱动实现，统一管理系统资源，维护系统稳定。

本章将为大家简单介绍下内核pinctrl子系统和GPIO子系统的基本概念、主要的数据结构、以及rockchip的pinctrl控制器等。

## 1 Pinctrl子系统

在前面章节，我们知道在许多soc内部包含有多个pin控制器，通过pin控制器的寄存器，我们可以配置一个或者一组引脚的功能和特性。Linux内核为了统一各soc厂商的pin脚管理 提供了pinctrl子系统。该系统的作用：

(1)枚举所有可以控制的pin，在系统初始化的时候，枚举所有可以控制的pin，并标识这些pin；

(2)设定引脚的功能复用，比如复用为GPIO还是SPI等

(3)引脚的配置，比如上下拉，驱动强度，去抖等

pinctrl子系统结构描述：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dfa7f65f1d56c58110e38bd3ba3c8197.png)

如上图所示，pinctrl核心层是内核抽象出来，向下为个SoC pin controler drvier提供底层通信接口的能力， 向上为其他驱动提供了控制pin的能力，比如pin复用、配置引脚的电气特性，同时也为GPIO子系统提供pin操作。而pin控制器驱动层，主要提供了操作pin的方法。

pinctrl子系统的源文件在内核源码/drivers/pinctrl目录下,主要包括核心文件，和其他内核驱动的接口头文件，底层的pin控制器驱动接口，还有其他厂商Pinctrl驱动文件。

### 1.1 重要的概念

系统中到底有多少个引脚，如何描述这些引脚？pinctrl核心层提供struct pinctrl_desc，具体soc厂商根据这个抽象出一个pin controller实例， 把系统中所有的pin描述出来，并建立索引，实际是struct pinctrl_desc结构中pins和npins来完成。 相应的就有pin controller driver。

Pinctrl驱动对引脚的管理，通过抽象出pin groups和function实现，具体以rk3568-lubancat2.dts设备树举例：
```d
&pinctrl {
   pmic {
      pmic_int: pmic_int {
            rockchip,pins =
               <0 RK_PA3 RK_FUNC_GPIO &pcfg_pull_up>;
      };

      soc_slppin_gpio: soc_slppin_gpio {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_GPIO &pcfg_output_low_pull_down>;
      };

      soc_slppin_slp: soc_slppin_slp {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_1 &pcfg_pull_up>;
      };

      soc_slppin_rst: soc_slppin_rst {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_2 &pcfg_pull_none>;
      };

      spk_ctl_gpio: spk_ctl_gpio {
            rockchip,pins = <3 RK_PC5 RK_FUNC_GPIO &pcfg_pull_up>;
      };
   };

   /*........*/

   spi {  //function
      spi3_cs0: spi3-cs0 {  //groups
            rockchip,pins = <4 RK_PC6 RK_FUNC_GPIO &pcfg_pull_up_drv_level_1>;
      };

      spi3_cs1: spi3-cs1 {
            rockchip,pins = <4 RK_PC4 RK_FUNC_GPIO &pcfg_pull_up_drv_level_1>;
      };
   };

   /*........*/
}
```

**Pin groups**

group，就是如上dts中的spi3_cs0: spi3-cs0，表示一组pins，这组pins统一表示了一种功能，比如，spi需要两个cs，而pmic需要用五个引脚。 在定义pins的同时，还会提供对于每个pin的电气特性的配置，如，上下拉电阻、驱动能力等。在内核中用结构体struct group_desc描述。

**function**

function，就是一组引脚功能，如上dts中的spi，表示一当前这个pin所代表的的功能。只有一个group所引用的pin的function有效，否则会引起pin的function冲突 比如，一个pin既可以作为普通的gpio，也可以作为SPI的cs引脚，那么，这个pin只能代表一个function，即，要么作为普通的gpio，作为SPI的cs引脚。

**pin state**

一个设备工作在某个状态，所使用的pin和function是唯一确认的，一个固定的组合确认一个固定的状态，这固定的状态在pinctrl子系统中称为“pin state”， 枚举了当下该设备的可能。在其他设备驱动使用时，pinctrl子系统只需设置相应pin state即可。 在设备树中描述就是使用pinctrl-names指定状态名字，pinctrl-x指定引脚状态。

### 1.2 主要的数据结构和接口

从面向对象的思想来看，内核将pinctrl驱动抽象为pinctrl_desc对象，具体到soc厂商的pinctrl驱动便是该对象一个实例， 在驱动所有的pin信息以及对于pin的控制接口实例化成pinctrl_desc，并将pinctrl_desc注册到内核中，如下

```c
struct pinctrl_desc {
   const char *name;
   const struct pinctrl_pin_desc *pins;   //描述一个pin控制器的引脚,
   unsigned int npins;                    //描述该控制器有多少个引脚
   const struct pinctrl_ops *pctlops;     //引脚操作函数，有描述引脚，获取引脚等，全局控制函数
   const struct pinmux_ops *pmxops;       //引脚复用相关的操作函数
   const struct pinconf_ops *confops;     //引脚配置相关
   struct module *owner;
#ifdef CONFIG_GENERIC_PINCONF
   unsigned int num_custom_params;
   const struct pinconf_generic_params *custom_params;
   const struct pin_config_item *custom_conf_items;
#endif
};
```

一般控制器驱动匹配设备，调用probe，最后会调用pinctrl_register函数，向内核注册pinctrl，产生pinctrl_dev，该函数如下：
```c
struct pinctrl_dev *pinctrl_register(struct pinctrl_desc *pctldesc,
                                 struct device *dev, void *driver_data);
```

描述一个引脚的结构体 struct pinctrl_pin_desc：
```c
struct pinctrl_pin_desc {
   unsigned number;
   const char *name;
   void *drv_data;
};
```

很多pin组合在一起，实现特定功能，使用struct group_desc：
```c
struct group_desc {
   const char *name;
   int *pins;
   int num_pins;
   void *data;
};
```

## 2 GPIO子系统

在pinctrl子系统中把pin脚初始化成了普通GPIO后，就可以使用GPIO子系统的接口去操作IO口的电平、中断等。驱动开发者在设备树中添加gpio相关信息， 然后就可以在驱动程序中使用gpio子系统提供的API函数来操作GPIO，极大的方便了驱动开发者使用GPIO。gpio子系统的代码在内核源码/drivers/gpio/目录下。

GPIO子系统结构简单描述如下图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/df5e510beeb85c2c6a9c6a4bf8f7a2f9.png)

GPIO的核心是gpiolib框架，向上提供一些gpio接口给其他驱动调用，向下提供用于gpio资源注册函数。 上层的其他驱动，比如LED的驱动，可以通过函数向gpiolib申请gpio，然后设置和使用gpio。下层的控制器驱动(一般是SOC厂商编写)，启动时会注册gpio资源到gpiolib，比如引脚数量，操作函数等等。

### 2.1 重要结构体

GPIO子系统，主要的数据结构有gpio_device、gpio_chip、gpio_desc等，主要是为了描述gpio控制器，有引脚信息，中断信息，以及相关操作函数等。

```c
// 内核源码/drivers/gpio/gpiolib.h
struct gpio_device {
   int                       id;           //gpio控制器的id，也就是第几个
   struct device             dev;
   struct cdev               chrdev;
   struct device             *mockdev;
   struct module             *owner;
   struct gpio_chip  *chip;
   struct gpio_desc  *descs;
   int                       base;         ////gpio在内核中的编号，申请gpio口时就是根据这个编号来查找
   u16                       ngpio;        //该gpio控制器有多少个引脚
   const char                *label;    //标签
   void                      *data;
   struct list_head        list;

#ifdef CONFIG_PINCTRL
   /*
   * If CONFIG_PINCTRL is enabled, then gpio controllers can optionally
   * describe the actual pin range which they serve in an SoC. This
   * information would be used by pinctrl subsystem to configure
   * corresponding pins for gpio usage.
   */
   struct list_head pin_ranges;
#endif
};
```

```c
// 内核源码/include/linux/gpio/driver.h
struct gpio_chip {
   const char                *label; //GPIO端口的名字，标签
   struct device             *dev;
   struct module             *owner;

   /*操作gpio口的方法*/
   int                       (*request)(struct gpio_chip *chip,
                  unsigned offset);
   void                      (*free)(struct gpio_chip *chip,
                  unsigned offset);
   int                       (*direction_input)(struct gpio_chip *chip,
                  unsigned offset);
   int                       (*get)(struct gpio_chip *chip,
                  unsigned offset);
   int                       (*direction_output)(struct gpio_chip *chip,
                  unsigned offset, int value);
   int                       (*set_debounce)(struct gpio_chip *chip,
                  unsigned offset, unsigned debounce);
   void                      (*set)(struct gpio_chip *chip,
                  unsigned offset, int value);
   int                       (*to_irq)(struct gpio_chip *chip,
                  unsigned offset);
   /*.....*/
   int                       base;                     //gpio在内核中的编号，申请gpio口时就是根据这个编号来查找
   u16                       ngpio;                 //该控制器的GPIO数目
   const char                *const *names;
   unsigned          can_sleep;
   /*......*/
};
```

```c
// 内核源码/include/linux/gpio/driver.h
struct gpio_desc {
   struct gpio_device        *gdev;      //gpio_device里面描述了gpio信息，详细看下内核源码/drivers/gpio/gpiolib.h
   unsigned long             flags;
   /* flag symbols are bit numbers */
   #define FLAG_REQUESTED    0
   #define FLAG_IS_OUT       1
   #define FLAG_EXPORT       2       /* protected by sysfs_lock */
   #define FLAG_SYSFS        3       /* exported via /sys/class/gpio/control */
   #define FLAG_ACTIVE_LOW   6       /* value has active low */
   #define FLAG_OPEN_DRAIN   7       /* Gpio is open drain type */
   #define FLAG_OPEN_SOURCE 8        /* Gpio is open source type */
   #define FLAG_USED_AS_IRQ 9        /* GPIO is connected to an IRQ */
   #define FLAG_IS_HOGGED    11      /* GPIO is hogged */
   #define FLAG_TRANSITORY 12        /* GPIO may lose value in sleep or reset */

   /* Connection label */
   const char                *label;
   /* Name of the GPIO */
   const char                *name;
};
```

一个gpio_device用来表示一个GPIO控制器，GPIO Controller中每一个引脚用gpio_desc表示，引脚的相关操作函数和中断相关在gpio_chip中。

### 2.2 常用API函数讲解

GPIO子系统有两套接口，一套是描述符(descriptor-based)的，相关api函数都是以 `gpiod_` 为前缀，可能前面还有 `devm_`,表示设备资源管理，一种自动释放资源的机制。 另外一套是旧的以 `gpio_` 为前缀。

**1. 获取GPIO编号函数of_get_named_gpio**

GPIO子系统大多数API函数会用到GPIO编号。GPIO编号可以通过of_get_named_gpio函数从设备树中获取。
```c
// 内核源码include/linux/of_gpio.h
 static inline int of_get_named_gpio(struct device_node *np, const char *propname, int index)
```
- **np：** 指定设备节点。
- **propname：** GPIO属性名，与设备树中定义的属性名对应。
- **index：** 引脚索引值，在设备树中一条引脚属性可以包含多个引脚，该参数用于指定获取那个引脚。
**返回值：**
- **成功：** 获取的GPIO编号(这里的GPIO编号是根据引脚属性生成的一个非负整数)，
- **失败:** 返回负数。

**2. GPIO申请函数gpio_request**

gpio_request函数()[](https://doc.embedfire.com/linux/rk356x/driver/zh/latest/linux_driver/subsystem_pinctrl_gpio.html#id14 "永久链接至代码")

```c
// 内核源码drivers/gpio/gpiolib-legacy.c
static inline int gpio_request(unsigned gpio, const char *label);
```
**参数：**
- **gpio:** 要申请的GPIO编号，该值是函数of_get_named_gpio的返回值。
- **label:** 引脚名字，相当于为申请得到的引脚取了个别名。

**返回值：**
- **成功:** 返回0，
- **失败:** 返回负数。


**3. GPIO释放函数**
```c
// 内核源码drivers/gpio/gpiolib-legacy.c
static inline void gpio_free(unsigned gpio);
```
gpio_free函数与gpio_request是一对相反的函数，一个申请，一个释放。一个GPIO只能被申请一次， 当不再使用某一个引脚时记得将其释放掉。

**参数：**

- **gpio：** 要释放的GPIO编号。
    

**返回值：** **无**

**4. GPIO输出设置函数gpio_direction_output**

用于将引脚设置为输出模式。
```c
// 内核源码include/asm-generic/gpio.h
static inline int gpio_direction_output(unsigned gpio , int value);
```
**函数参数：**

- **gpio:** 要设置的GPIO的编号。
    
- **value:** 输出值，1，表示高电平。0表示低电平。
    

**返回值：**

- **成功:** 返回0
    
- **失败:** 返回负数。
    

**5. GPIO输入设置函数gpio_direction_input**
用于将引脚设置为输入模式。

```c
// 内核源码include/asm-generic/gpio.h
static inline int gpio_direction_input(unsigned gpio)
```
**函数参数：**

- **gpio:** 要设置的GPIO的编号。
    

**返回值：**

- **成功:** 返回0
    
- **失败:** 返回负数。
    

**6. 获取GPIO引脚值函数gpio_get_value**

用于获取引脚的当前状态。无论引脚被设置为输出或者输入都可以用该函数获取引脚的当前状态。
```c
// 内核源码include/asm-generic/gpio.h
static inline int gpio_get_value(unsigned gpio);
```

**函数参数：**

- **gpio:** 要获取的GPIO的编号。
    

**返回值：**

- **成功:** 获取得到的引脚状态
    
- **失败:** 返回负数
    

**7. 设置GPIO输出值gpio_set_value**

该函数只用于那些设置为输出模式的GPIO.
```c
// 内核源码include/asm-generic/gpio.h
static inline int gpio_direction_output(unsigned gpio, int value);
```

**函数参数**

- **gpio：** 设置的GPIO的编号。
    
- **value：** 设置的输出值，为1输出高电平，为0输出低电平。
    

**返回值：**

- **成功:** 返回0
    
- **失败:** 返回负数
    

一些常用的新接口函数：

|函数|功能|
|---|---|
|gpiod_get|设备树中gpio属性只有一组值时，获取GPIO|
|gpiod_get_index|设备树gpio有多组值时，通过INDEX获取GPIO|
|gpiod_get_direction|获取gpio的状态(输入或输出)|
|gpiod_direction_input|将GPIO配置为输入|
|gpiod_get_value|获取GPIO的逻辑值|
|gpiod_set_value|设置gpio引脚值|
|gpiod_put|释放gpio资源，是在gpiod_get和gpiod_get_index中申请的|

一些详细使用说明，请参考内核源码Documentation/driver-api/gpio/board.rst 、Documentation/driver-api/gpio/consumer.rst等

### 2.3 GPIO子系统和sysfs

sysfs的gpio接口，也可以设置和管理GPIO，到板卡的/sys/class/gpio目录下，可以看到：

```shell
cat@lubancat:/sys/class/gpio$ ls
export  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
cat@lubancat:/sys/class/gpio/gpiochip0$ ls
base  device  label  ngpio  power  subsystem  uevent  /*base代表该组控制器的基引脚编号,ngpio代表该控制器的gpio引脚数量，label是设备树的节点标签*/
```

在该目录下每个gpiochipX代表一个gpio控制器， 对于export和unexport,export允许我们在用户空间往该文件输引脚索引，往内核请求GPIO(如果没有被申请过)然后创建一个节点，unexport与之相反。例如下面：

```shell
cat@lubancat:/sys/class/gpio$ echo 6 | sudo tee  export > /dev/null         /*往export输入6，会在当前目下创建一个gpio6节点*/
cat@lubancat:/sys/class/gpio$ ls
export  gpio6  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
cat@lubancat:/sys/class/gpio$ cd gpio6/ && ls
active_low  device  direction  edge  power  subsystem  uevent  value       /*往direction输入“out”，设置该引脚输出，然后往value输入“1”，该gpio输出高电平*/
cat@lubancat:/sys/class/gpio/gpio6$ echo out | sudo tee direction > /dev/null
cat@lubancat:/sys/class/gpio/gpio6$ cat direction
out

cat@lubancat:/sys/class/gpio$ echo 6 | sudo tee  unexport > /dev/null      /*往unexport输入6，会删除当前目下的gpio6节点*/
cat@lubancat:/sys/class/gpio$ ls
export  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
```


> [!tip]+ 重要
> 上面是以lubuncat2板卡为例，GPIO控制器是有5个，引脚索引pins = 32 * bank + 8 * group + x ；bank是0~4，group:0~3，对应A~D；例如GPIO0_A6的pins =32 * 0+8 * 0+6 = 6

## 3 GPIO子系统和Pinctrl子系统之间的耦合关系

Pinctrl子系统管理所有的引脚，GPIO子系统是这些引脚的用途之一， GPIO子系统会向Pinctrl子系统申请引脚，并设置为GPIO功能。申请GPIO的时候会去检查该gpio所对应的pin是否已经被其他驱动申请作其他功能了,如果已经被申请则申请时会报错。

## 4 参考链接

[https://www.kernel.org/doc/html/v4.13/driver-api/pinctl.html](https://www.kernel.org/doc/html/v4.13/driver-api/pinctl.html)