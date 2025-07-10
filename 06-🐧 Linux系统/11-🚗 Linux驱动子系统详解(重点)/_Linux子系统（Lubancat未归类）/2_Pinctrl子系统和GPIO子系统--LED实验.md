本章中结合前面一章的讲解，将会编写具体代码使用GPIO子系统实现LED驱动，GPIO子系统要用到pinctrl子系统。

## 1 pinctrl子系统

pinctrl子系统主要用于管理芯片的引脚，比如引脚的复用，引脚上下拉，驱动能力等。rockchip芯片拥有众多的片上外设， 大多数外设需要通过芯片的引脚与外部设备(器件)相连实现相对应的控制，例如我们熟悉的I2C、SPI、LCD、USDHC等等。 而我们知道芯片的可用引脚(除去电源引脚和特定功能引脚)数量是有限的，芯片的设计厂商为了提高硬件设计的灵活性， 一个芯片引脚往往可以做为多个片上外设的功能引脚，以rk3568举例，查阅<<Rockchip_RK3568_Datasheet_xxx>>数据手册，如下图所示。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2f8ad37f7214c859708331994b7e3a56.png)

GPIO1_A6的功能引脚不单单只可以使用在GPIO上，也可以作为多个外设的功能引脚，如I2S引脚，串口的接收发送引脚等， 在设计硬件时我们可以根据需要灵活的选择其中的一个。设计完硬件后每个引脚的功能就确定下来了，假设我们将上面的两个引脚连接 到其他用串口控制的外部设备上，那么这两个引脚功能就做为了UART的接收、发送引脚。在编程过程中，无论是裸机还是驱动， 一般首先要设置引脚的复用功能并且设置引脚的PAD属性(驱动能力、上下拉等等)。

在驱动程序中我们需要手动设置每个引脚的复用功能，不仅增加了工作量，编写的驱动程序不方便移植， 可重用性差等。更糟糕的是缺乏对引脚的统一管理，容易出现引脚的重复定义。 假设我们在I2C的驱动中将UART_RX_DATA引脚和UART_TX_DATA引脚复用为SCL和SDA， 恰好在编写UART驱动驱动时没有注意到UART_RX_DATA引脚和UART_TX_DATA引脚已经被使用， 在驱动中又将其初始化为UART_RX和UART_TX，这样IIC驱动将不能正常工作，并且这种错误很难被发现。

pinctrl子系统是由芯片厂商来实现的，简单来说用于帮助我们管理芯片引脚并自动完成引脚的初始化， 而我们要做的只是在设备树中按照规定的格式写出想要的配置参数即可

### 1.1 pinctrl子系统编写格式以及引脚属性详解

#### 1.1.1 pinctrl设备树节点介绍

首先我们在内核源码/arch/arm64/boot/dts/rockchip/rk3568.dtsi文件中，可以看到如下定义
```dts
pinctrl: pinctrl {
	 compatible = "rockchip,rk3568-pinctrl";
	 rockchip,grf = <&grf>;
	 rockchip,pmu = <&pmugrf>;
	 #address-cells = <2>;
	 #size-cells = <2>;
	 ranges;

	 gpio0: gpio@fdd60000 {
		 compatible = "rockchip,gpio-bank";
		 reg = <0x0 0xfdd60000 0x0 0x100>;
		 interrupts = <GIC_SPI 33 IRQ_TYPE_LEVEL_HIGH>;
		 clocks = <&pmucru PCLK_GPIO0>, <&pmucru DBCLK_GPIO0>;

		 gpio-controller;
		 #gpio-cells = <2>;
		 gpio-ranges = <&pinctrl 0 0 32>;
		 interrupt-controller;
		 #interrupt-cells = <2>;
	 };
/* 剩下内容省略 */
};
```
- **compatible：** 修饰的是与平台驱动做匹配的名字，这里则是与pinctrl子系统的平台驱动做匹配。
    
- **reg：** 表示配置寄存器的基地址。
    

rk3568.dtsi这个文件是芯片厂商官方将芯片的通用的部分单独提出来的一些设备树配置。 在soc节点中汇总了所需引脚的配置信息，pinctrl子系统存储使用着的节点信息。

我们的设备树主要的配置文件在内核源码/arch/arm64/boot/dts/rockchip/rk3568-lubancat2.dts中， 打开rk3568-lubancat2.dts，在文件中搜索“&pinctrl”找到设备树中引用“pinctrl”节点的位置如下所示。

```d
// rk3568-lubancat2.dts中&pinctrl部分内容
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

    headphone {
        hp_det: hp-det {
            rockchip,pins = <0 RK_PA5 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };

    usb {
        vcc5v0_usb20_host_en: vcc5v0-usb20-host-en {
            rockchip,pins = <0 RK_PD5 RK_FUNC_GPIO &pcfg_pull_none>;
        };

        vcc5v0_usb30_host_en: vcc5v0-usb30-host-en {
            rockchip,pins = <0 RK_PD6 RK_FUNC_GPIO &pcfg_pull_none>;
        };

        vcc5v0_otg_vbus_en: vcc5v0-otg-vbus-en {
            rockchip,pins = <0 RK_PD3 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
    /* 剩下内容省略 */
};
```
在这里通过“&pinctrl”在“pinctrl”节点下追加内容。结合设备树源码介绍如下：

- 第2行：如第一个“pmic”节点中，从名字来看我们可以知道该节点大概是描述“pmic”电源管理的引脚功能， 在它的第一个子节点“pmic_int”中，使用了“rockchip,pins”指定的一引脚的复用功能。
    
- 第29-30行，指定了“headphone”子节点使用的引脚的电气特性。
    
- 其余源码都是pinctrl下的子节点了，各自描述了一些外设的使用到的引脚及与之对应的复用功能，它们都是按照一定的格式规范来编写。
    

那么我们会在什么情况下使用到pinctrl呢？我们以&sdmmc1这个外设的节点来看。
```d
&sdmmc1 {
    pinctrl-names = "default", "opendrain", "sleep";
    pinctrl-0 = <&sdmmc1_b4_pins_a>;
    pinctrl-1 = <&sdmmc1_b4_od_pins_a>;
    pinctrl-2 = <&sdmmc1_b4_sleep_pins_a>;
    broken-cd;
    st,neg-edge;
    bus-width = <4>;
    vmmc-supply = <&v3v3>;
    status = "okay";
};
```
- **pinctrl-names：** 描述了sdmmc1外设会使用到的三种引脚状态，分别是default、opendrain、sleep。
    
- **pinctrl-0：** 当外设处于default状态下，则使用pinctrl-0中引用的引脚配置&sdmmc1_b4_pins_a。
    
- **pinctrl-1：** 当外设处于opendrain状态下，则使用pinctrl-1中引用的引脚配置&sdmmc1_b4_od_pins_a。
    
- **pinctrl-2：** 当外设处于sleep状态下，则使用pinctrl-2中引用的引脚配置&sdmmc1_b4_sleep_pins_a。
    

这样以来，我们就指定了这个外设使用到的引脚及其状态。

#### 1.1.2 pinctrl子节点编写格式

那么按照“&pinctrl”下节点的描述形式，我们也可以自己描述一下某个外设的pinctrl。

```dts
//举例说明
 &pinctrl {
     xxx: xxx {
         pins {
                rockchip,pins = <0 RK_PA6 RK_FUNC_GPIO &pcfg_pull_none>;
         };
     };
 };
```

如上述的一个外设xxx，由其使用的引脚为GPIO0_A6，”RK_FUNC_GPIO”设置复用功能为GPIO，每个引脚可以复用的功能具体参考下手册，”&pcfg_pull_none”指定上下拉，这里的没有设置不用上下拉，该引用节点， 可以参考下内核源码arch/arm64/boot/dts/rockchip/rockchip-pinconf.dtsi文件。

这里我们需要知道每个芯片厂商的pinctrl子节点的编写格式并不相同，这不属于设备树的规范，是芯片厂商自定义的。 如果我们想添加自己的pinctrl节点，只要依葫芦画瓢按照上面的格式编写即可。

关于pinctrl节点如何去描述，我们可以在内核文档目录中查找芯片产商给出的文档。 如rockchip官方的pinctrl文档目录如下：

Documentation/devicetree/bindings/pinctrl/rockchip,pinctrl.txt文档

### 1.2 将RGB灯引脚添加到pinctrl子系统
本小节，我们从看原理图开始，一步步将LED灯用到的引脚添加到pinctrl子系统中，具体板卡可能引脚不同，请参考实际板卡的原理图。

#### 1.2.1 查找LED灯使用的引脚

- 以lubuncat2为例，系统LED灯对应的原理图如下所示。
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ba82fa410d8945e9d6598996bc185d81.png)
根据网络名在核心板上找到对应的引脚是: GPIO0_C7。

#### 1.2.2 在pinctrl节点中添加pinctrl子节点

添加子节点很简单，我们只需要将引脚信息以一定格式， 写入到对应的设备树文件中的pinctrl子节点即可。

以lubuncat2为例，在rk3568-lubancat2.dts添加以下内容。
```c
//新增pinctrl子节点
&pinctrl {
    /*----------新添加的内容--------------*/
    led_test {
        led_test_pin: led_test_pin {
            rockchip,pins = <0 RK_PC7 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
};
```

新增的节点名为“led_test”，名字任意选取(在同一节点下不要同名)，长度不要超过32个字符，最好能表达出节点的信息。 “led_test_pin”节点标签，“rockchip,pins”是固定的格式，后面的内容自定义的，我们将通过这个标签引用这个节点。

pins的内容中，我们将LED使用到的GPIO引脚功能配置好了，因为pinctrl各家芯片厂商各异，这里我们就不展开， 具体大家可以参考官方的

Documentation/devicetree/bindings/pinctrl/rockchip,pinctrl.txt

文档，在添加完pinctrl子节点后，系统会根据我们添加的配置信息将引脚初始化为GPIO功能。

到这里关于pinctrl子系统的使用就已经讲解完毕了，接下来介绍GPIO子系统相关的内容。

## 2 GPIO子系统

在没有使用GPIO子系统之前，如果我们想点亮一个LED，首先要得到led相关的配置寄存器，再手动地读、改、写这些配置寄存器实现 控制LED的目的。有了GPIO子系统之后这部分工作由GPIO子系统帮我们完成，我们只需要调用GPIO子系统提供的API函数即可完成GPIO的 控制动作。

在rk3568.dtsi文件中的pinctrl子节点记录着GPIO控制器的寄存器地址，下面我们以GPIO0为例介绍GPIO0子节点相关内容
```dts
// rk3568.dtsi中GPIOA节点内容
/ {
    pinctrl: pinctrl {
    compatible = "rockchip,rk3568-pinctrl";
    rockchip,grf = <&grf>;
    rockchip,pmu = <&pmugrf>;
    #address-cells = <2>;
    #size-cells = <2>;
    ranges;

    gpio0: gpio@fdd60000 {
        compatible = "rockchip,gpio-bank";
        reg = <0x0 0xfdd60000 0x0 0x100>;
        interrupts = <GIC_SPI 33 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&pmucru PCLK_GPIO0>, <&pmucru DBCLK_GPIO0>;

        gpio-controller;
        #gpio-cells = <2>;
        gpio-ranges = <&pinctrl 0 0 32>;
        interrupt-controller;
        #interrupt-cells = <2>;
    };
        /* 剩余内容省略 */
    };
};
```

- **compatible** ：与GPIO子系统的平台驱动做匹配。
    
- **reg** ：GPIO0外设寄存器的基地址，在gpioa的reg属性中GPIOA的寄存器组的映射地址为fdd60000，范围为0x100。
    
- **interrupts** ：表示中断控制信息，用了一个中断号，都是SPI中断源
    
- **clocks** ：初始化GPIO外设时钟信息
    
- **gpio-controller** ：表示gpioa是一个GPIO控制器
    
- **#gpio-cells** ：表示有多少个cells来描述GPIO引脚，这个里使用2，表示一个cell描述那个引脚，一个ceel描述有效电平。
    
- **gpio-ranges** : gpio引脚映射范围
    
- **#interrupt-controller** : 是中断控制器
    
- **#interrupt-cells** :表示用多少个cells来描述一个中断
    

大家大致有个了解就可以了，一般芯片产商会将这部分信息完善好。

gpio0这个节点对整个gpio0进行了描述。使用GPIO子系统时需要往设备树中添加设备节点，在驱动程序中使用GPIO子系统提供的API 实现控制GPIO的效果。

### 2.1 在设备树中添加RGB灯的设备树节点

相比之前设备树led灯设备节点(没有使用GPIO子系统)，以下只需要增加GPIO属性定义，基于GPIO子系统的led_test设备树节点。

以lubuncat2为例，添加到rk3568-lubancat2.dts设备树的 `根节点内` ，添加完成后的设备树如下所示。
```d
/*添加led_test节点*/
led_test: led_test {
    status = "okay";
    compatible = "fire,led_test";
    default-state = "on";
    gpios = <&gpio0 RK_PC7 GPIO_ACTIVE_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&led_test_pin>;
};
```
- 第4行，设置“compatible”属性值，与led的平台驱动做匹配。
    
- 第8行，指定RGB灯的引脚pinctrl信息，上一小节我们定义了pinctrl节点，并且标签设置为“led_test_pin”， 在这里我们引用了这个pinctrl信息。
    
- 第6行，引用某个引脚，一般使用[name]-gpios格式，指定引脚使用的哪个GPIO,名称可以自定义，节点值编写格式如下所示:
    
- “&gpio0”，设置是哪个gpio控制器或者哪个pin组。
    
- “RK_PC7”，指定在该bank下的引脚索引。
    
- “GPIO_ACTIVE_HIGH”，这是一个宏定义，指默认是高电平，低电平有效选择“GPIO_ACTIVE_LOW”高电平有效选择“GPIO_ACTIVE_HIGH”。

### 2.2 在设备树中注释sys灯的设备树节点

系统led灯使用了内核自带的led驱动，由于会与本实验冲突，我们需要将其屏蔽掉。

找到leds的节点信息，将status设置为disabled，从而关闭leds节点。

```dts
// 屏蔽leds冲突
    leds: leds {
            /*status = "okay";*/            
            status = "disabled";            
            compatible = "gpio-leds";

            sys_status_led: sys-status-led {
                    label = "sys_status_led";
                    linux,default-trigger = "heartbeat";
                    default-state = "on";
                    gpios = <&gpio0 RK_PC7 GPIO_ACTIVE_LOW>;
                    pinctrl-names = "default";
                    pinctrl-0 = <&sys_status_led_pin>;
            };
```

或者使用追加节点信息的方式将leds节点的状态设置为disabled，在对应板卡的dts中合适的位置添加：

```dts
&leds {
    status = "disabled";
    };
```

### 2.3 编译、下载设备树验证修改结果

本章前两小节我们分别在设备树中将led灯使用的引脚添加到pinctrl子系统，然后又在设备树中添加了led_test设备树节点。 这一小节将会编译、下载修改后的设备树，用新的设备树启动系统，然后检查是否有led_test设备树节点产生。

编译内核时会自动编译设备树，这样做的缺点是编译时间会很长。 在内核目录下，执行如下命令，只编译设备树：

rk356x系列板卡执行以下命令：
```shell
#加载配置文件
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig
#使用dtbs参数单独编译设备树
make ARCH=arm64 -j4 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

编译成功后会在“内核源码arch/arm64/boot/dts/rockchip/目录下生成对应的dtb文件，将其替换掉板子 /boot/dtb/目录下的dtb文件并重启开发板。

使用新的设备树重新启动之后正常情况下会在开发板的“/proc/device-tree”目录下生成“led_test”设备树节点，如下所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1de8f8b27439ddf68d3cfb40cb9090e4.png)

从上图可以看到“/proc/device-tree”目录下有“led_test”和“leds”节点，并且“led_test”节点的状态为“okay”，“leds”节点的状态为“disabled”。

至此，我们的设备已经添加到了系统中，下面我们可以尝试编写驱动来使用我们的LED设备了。 不过在这之前，一些函数可以看下前面一章GPIO子系统小结

## 3 实验说明与代码讲解

**硬件介绍**

本节实验使用到lubancat2板上的led灯

### 3.1  实验代码讲解

**本章的示例代码目录为：linux_driver/gpio_subsystem_led**

程序包含两个C语言文件，一个是驱动程序，驱动程序在平台总线基础上编写。 另一个是一个简单的测试程序，用于测试驱动是否正常。

#### 3.1.1 驱动程序讲解

驱动程序大致分为三个部分，第一部分，编写平台设备驱动的入口和出口函数。第二部分，编写平台设备的.probe函数, 在probe函数中实现字符设备的注册和LED灯的初始化。第三部分，编写字符设备函数集，实现open和write函数。

**平台驱动入口和出口函数实现**

源码如下：
```c
static const struct of_device_id led_ids[] = {
    { .compatible = "fire,led_test"},
    { /* sentinel */ }
};

/*定义平台设备结构体*/
struct platform_driver led_platform_driver = {
    .probe = led_probe,
    .driver = {
            .name = "leds-platform",
            .owner = THIS_MODULE,
            .of_match_table = led_ids,
    }
};

/*
*驱动初始化函数
*/
static int __init led_platform_driver_init(void)
{
    int DriverState;

    DriverState = platform_driver_register(&led_platform_driver);

    printk(KERN_EMERG "\tDriverState is %d\n",DriverState);
    return 0;
}

/*
*驱动注销函数
*/
static void __exit led_platform_driver_exit(void)
{
    printk(KERN_EMERG "led_test exit!\n");
    /*删除设备*/
    device_destroy(class_led, led_devno);             //清除设备
    class_destroy(class_led);                         //清除类
    cdev_del(&led_chr_dev);                           //清除设备号
    unregister_chrdev_region(led_devno, DEV_CNT);     //取消注册字符设备

    platform_driver_unregister(&led_platform_driver);
}

module_init(led_platform_driver_init);
module_exit(led_platform_driver_exit);

MODULE_LICENSE("GPL");
```

- 第2-15行：为代码的第一部分，仅实现.probe函数和.driver，当驱动和设备匹配成功后会执行该函数， 这个函数的函数实现我们在后面介绍。.driver描述这个驱动的属性，包括.name驱动的名字，.owner驱动的所有者, .of_match_table驱动匹配表，用于匹配驱动和设备。驱动设备匹配表定义为“led_test”在这个表里只有一个匹配值 “.compatible = “fire,led_test” ”这个值要与我们在设备树中led_test设备树节点的“compatible”属性相同。
    
- 第17-40行：第二、三部分是平台设备的入口和出口函数，函数实现很简单，在入口函数中注册平台驱动，在出口函数中注销平台驱动。
    

**平台驱动.probe函数实现**

当驱动和设备匹配后首先会probe函数，我们在probe函数中实现RGB的初始化、注册一个字符设备。 后面将会在字符设备操作函数(open、write)中实现对RGB等的控制。函数源码如下所示。
```c
/*----------------平台驱动函数集-----------------*/
static int led_probe(struct platform_device *pdv)
{

    int ret = 0;  //用于保存申请设备号的结果

    printk("match successed\n");

    /*获取RGB的设备树节点*/
    led_device_node = of_find_node_by_path("/led_test");
    if(led_device_node == NULL)
    {
        printk(KERN_EMERG "get led_test failed!  \n");
    }

    led = of_get_named_gpio(led_device_node, "gpios", 0);

    printk("led = %d \n",led);

    /*设置gpio输出高电平*/
    gpio_direction_output(led, 1);

    /*---------------------注册 字符设备部分-----------------*/

    //第一步
    //采用动态分配的方式，获取设备编号，次设备号为0，
    //设备名称为rgb-leds，可通过命令cat  /proc/devices查看
    //DEV_CNT为1，当前只申请一个设备编号
    ret = alloc_chrdev_region(&led_devno, 0, DEV_CNT, DEV_NAME);
    if(ret < 0){
        printk("fail to alloc led_devno\n");
        goto alloc_err;
    }

    //第二步
    //关联字符设备结构体cdev与文件操作结构体file_operations
    led_chr_dev.owner = THIS_MODULE;
    cdev_init(&led_chr_dev, &led_chr_dev_fops);

    //第三步
    //添加设备至cdev_map散列表中
    ret = cdev_add(&led_chr_dev, led_devno, DEV_CNT);
    if(ret < 0)
    {
        printk("fail to add cdev\n");
        goto add_err;
    }

    //第四步
    /*创建类 */
    class_led = class_create(THIS_MODULE, DEV_NAME);

    /*创建设备*/
    device = device_create(class_led, NULL, led_devno, NULL, DEV_NAME);

    return 0;

add_err:
    //添加设备失败时，需要注销设备号
    unregister_chrdev_region(led_devno, DEV_CNT);
    printk("\n error! \n");
alloc_err:
    return -1;

}
```

- 第10-14行：使用of_find_node_by_path函数找到并获取led_test在设备树中的设备节点。 参数“/led_test”是要获取的设备树节点在设备树中的路径，如果要获取的节点嵌套在其他子节点中需要写出节点所在的完整路径。
    
- 第17-22行：使用函数of_get_named_gpio函数获取GPIO号，读取成功则返回读取得到的GPIO号。 “gpios”指定GPIO的名字，这个参数要与led_test设备树节点中GPIO属性名对应， 参数“0”指定引脚索引，我们的设备树中一条属性中只定义了一个引脚，我们只有一个所以设置为0。
    
- 第25-27行，将GPIO设置为输出模式，默认输出电平为高电平。
    
- 第32-65行，字符设备相关内容，这部分内容在字符设备章节已经详细介绍这里不再赘述。
    

**实现字符设备函数**

字符设备函数我们只需要实现open函数和write函数。函数源码如下。
```c
/*------------------第一部分---------------*/
/*字符设备操作函数集*/
static struct file_operations  led_chr_dev_fops =
{
    .owner = THIS_MODULE,
    .open = led_chr_dev_open,
    .write = led_chr_dev_write,
};

/*------------------第二部分---------------*/
/*字符设备操作函数集，open函数*/
static int led_chr_dev_open(struct inode *inode, struct file *filp)
{
    printk("open \n");
    return 0;
}

/*------------------第三部分---------------*/
/*字符设备操作函数集，write函数*/
static ssize_t led_chr_dev_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
    unsigned char write_data; //用于保存接收到的数据

    int error = copy_from_user(&write_data, buf, cnt);
    if(error < 0) {
            return -1;
    }

    /*设置led的引脚输出电平*/
    if(write_data)
    {
        gpio_direction_output(led, 1);  // 引脚输出高电平，红灯灭
    }
    else
    {
        gpio_direction_output(led, 0);    //引脚输出底电平，红灯亮
    }

    return 0;
}
```

- 代码3-8行:定义字符设备操作函数集，这里主要实现open和write函数即可。
    
- 代码12-16行：实现open函数，在平台驱动的prob函数中已经初始化了GPIO,这里不用做任何操作
    
- 代码20-39行：write函数实现也很简单，首先使用“copy_from_user”函数将来自应用层的数据“拷贝”内核层。根据命令值使用“gpio_direction_output”函数控制LED灯的亮灭。
    

#### 3.1.2 应用程序讲解

应用程序编写比较简单，我们只需要打开设备节点文件，写入命令然后关闭设备节点文件即可。源码如下所示。
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main(int argc, char *argv[])
{
    printf("led test\n");
    /*判断输入的命令是否合法*/
    if(argc != 2)
    {
        printf(" command error ! \n");
        printf(" usage : sudo test_app num [num can be 0 or 1]\n");
        return -1;
    }

    /*打开文件*/
    int fd = open("/dev/led_test", O_RDWR);
    if(fd < 0)
    {
        printf("open file : %s failed !\n", argv[0]);
        return -1;
    }

    unsigned char command = atoi(argv[1]);  //将受到的命令值转化为数字;

    /*写入命令*/
    int error = write(fd,&command,sizeof(command));
    if(error < 0)
    {
        printf("write file error! \n");
        close(fd);
        /*判断是否关闭成功*/
    }

    /*关闭文件*/
    error = close(fd);
    if(error < 0)
    {
        printf("close file error! \n");
    }

    return 0;
}
```

结合代码各部分说明如下：

- 代码10-15行：判断命令是否有效。再运行应用程序时我们要传递一个控制命令，所以参数长度是2。
    
- 代码19-24行：打开设备文件。参数“/dev/led_test”用于指定设备节点文件，设备节点文件名是在驱动程序中设置的， 这里保证与驱动一致即可。
    
- 代码26-43行：由于从main函数中获取的参数是字符串，这里首先要将其转化为数字。最后条用write函数写入命令然后关闭文件即可。

### 3.2 实验准备

在板卡上的部分GPIO可能会被系统占用，运行代码时出现“Device or resource busy”或者运行代码卡死等等现象，需要注释设备树插件或者更改设备树，然后重启系统，释放相应的GPIO引脚。

> [!tip]+ 重要
> 如出现 `Permission denied` 或类似字样，请注意用户权限，大部分操作硬件外设的功能，几乎都需要root用户权限，简单的解决方案是在执行语句前加入sudo或以root用户运行程序。

#### Makefile修改说明

**修改Makefile并编译生成驱动程序**

Makefile程序并没有大的变化，修改后的Makefile如下所示。
```makefile
KERNEL_DIR=../../kernel/

ARCH=arm64
CROSS_COMPILE=aarch64-linux-gnu-
export  ARCH  CROSS_COMPILE

obj-m := led_test.o
out = led_app.o

all:
        $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules
        $(CROSS_COMPILE)gcc -o $(out) led_app.c

.PHONE:clean

clean:
        $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
```

- 代码第2行：变量“KERNEL_DIR”保存的是内核所在路径，这个需要根据自己内核所在位置设定。
    
- 代码第7行：“obj-m := led_test.o”中的“led_test.o”要与驱动源码名对应。Makefile 修改完成后执行如下命令编译驱动。
    

进入对应的驱动目录执行命令：
```shell
make
```

正常情况下会在当前目录生成led_test.ko驱动文件和led_app应用程序。

### 下载验证

前两小节我们已经编译出了.ko驱动和应用程序，将驱动程序和应用程序添加到开发板中(推荐使用之前讲解的scp或者NFS共享文件夹)， 然后执行如下命令加载驱动：

命令：
```shell
insmod ./led_test.ko
```

在驱动程序中，我们在.probe函数中注册字符设备并创建了设备文件，设备和驱动匹配成功后.probe函数已经执行， 所以正常情况下在“/dev/”目录下已经生成了“led_test”设备节点，如下所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3d134d7d446f602452b181ca0be28136.png)

驱动加载成功后直接运行应用程序如下所示。

命令：
```shell
./led_app <命令>
```

执行结果如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c53980cbab6e8b2ef88c790606481a13.png)
命令是一个“unsigned char”型数据，输入0，设置引脚输出低电平，灯亮，输入1，设置引脚高电平，灯灭。