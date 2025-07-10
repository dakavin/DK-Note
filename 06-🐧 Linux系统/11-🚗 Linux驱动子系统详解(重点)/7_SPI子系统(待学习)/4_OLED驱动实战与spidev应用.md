
## 1 引言

理论知识学习后，最重要的是动手实践。本章将通过一个完整的SPI OLED显示屏驱动实例，展示如何编写SPI设备驱动。同时介绍Linux内核提供的通用SPI设备驱动spidev，它允许用户空间程序直接访问SPI设备。

通过本章学习，你将掌握SPI设备驱动的编写方法，以及如何在应用层使用SPI设备。

## 2 OLED屏幕驱动实验

本章配套源码和设备树插件位于`linux_driver/SPI_OLED`。

### 2.1 硬件介绍

本实验以lubancat2板卡为例，其他Lubancat_RK系列板卡同样可以，需要自行查找选定spi接口。oled模块使用的是[《野火【OLED屏_SPI_0.96寸】模块资料》](https://doc.embedfire.com/products/link/zh/latest/module/screen/oled_spi_0.96.html)。

#### 2.1.1 硬件连接

在oled驱动中我们使用lubuncat2的spi3，可以通过rk3568数据手册查到，spi使用到的引脚如下：

![20250626_200854_991_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/69b43868c828923200ad57c261636322.png)

oled屏和板卡引脚对应连接如下表：

|OLED显示屏引脚|对应板卡GPIO|说明|板卡引脚(排针)|
|---|---|---|---|
|MOSI|GPIO4_C3|MOSI引脚|MOSI|
|未使用||MISO引脚||
|CLK|GPIO4_C2|SPI时钟引脚|MCLK|
|D/C|GPIO3_A7|数据、命令控制引脚|GPIO3_A7|
|CS|GPIO4_C6|片选引脚|CS0|
|GND|||GND|
|VCC|||3.3V|

> [!tip]+ 提示 lubancat2板卡引出引脚，请看"LubanCat-RK系列板卡快速使用手册"的40pin引脚对照图。

#### 2.1.2 设备树插件

设备树插件书写格式不变，我们重点讲解spi_oled设备节点：

```c
// linux_driver/SPI_OLED/lubancat-spi3-m1-oled-overlay.dts
/*
* Copyright (C) 2022 - All Rights Reserved by
* EmbedFire LubanCat
*/
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/clock/rk3568-cru.h>
#include <dt-bindings/interrupt-controller/irq.h>

&spi3{
    status = "okay";
    pinctrl-names = "default", "high_speed";
    pinctrl-0 = <&spi3m1_cs0 &spi3m1_pins>;
    pinctrl-1 = <&spi3m1_cs0 &spi3m1_pins_hs>;
    cs-gpios = <&gpio4 RK_PC6 GPIO_ACTIVE_LOW>;

    spi_oled@0 {
        status = "okay";
        compatible = "fire,spi_oled";
        reg = <0>;                        //chip select 0:cs0  1:cs1
        spi-max-frequency = <24000000>;   //spi output clock
        dc_control_pin = <&gpio3 RK_PA7 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&spi_oled_pin>;
    };
};

&pinctrl {
    spi_oled {
        spi_oled_pin: spi_oled_pin {
            rockchip,pins = <3 RK_PA7 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
};
```

设备树节点说明：

- 第14-18行：开启spi3，指定使用的pinctrl节点，也就是说指定spi3要使用的引脚
- 第18行：指定使用的片选引脚，我们这里使用的是GPIO4_C6
- 第21行：向spi3节点追加spi_oled设备节点
- 第23行：设置reg属性为0,表示spi_oled连接到spi3的通道0
- 第24行：设置SPI传输的最大频率，根据实际oled设备
- 第25行：指定spi_oled使用的D/C控制引脚，在驱动程序中会控制该引脚设置发送的是命令还是数据

### 2.2 实验代码讲解

#### 2.2.1 编程思路

spi_oled驱动使用设备树插件方式开发,主要工作包三部分内容：

1. 编写spi_oled的设备树插件，开启spi3，在spi3设备节点下添加spi_oled节点
2. 编写spi_oled驱动程序，包含驱动的入口、出口函数实现，.probe函数实现，file_operations函数集实现
3. 编写简单测试应用程序

#### 2.2.2 驱动的入口和出口函数

驱动入口和出口函数与I2C_mpu6050驱动相似，只是把i2c替换为spi，源码如下所示：

```c
// linux_driver/SPI_OLED/spi_oled.c
/*指定 ID 匹配表*/
static const struct spi_device_id oled_device_id[] = {
    {"fire,spi_oled", 0},
    {}};

/*指定设备树匹配表*/
static const struct of_device_id oled_of_match_table[] = {
    {.compatible = "fire,spi_oled"},
    {}};

/*spi总线设备结构体*/
struct spi_driver oled_driver = {
    .probe = oled_probe,
    .remove = oled_remove,
    .id_table = oled_device_id,
    .driver = {
            .name = "spi_oled",
            .owner = THIS_MODULE,
            .of_match_table = of_match_ptr(oled_of_match_table),
    },
};

/*
*驱动初始化函数
*/
static int __init oled_driver_init(void)
{
    int error;
    int ret = -1; //保存错误状态码
    pr_info("oled_driver_init\n");

    /*---------------------注册 字符设备部分-----------------*/

    //采用动态分配的方式，获取设备编号，次设备号为0，
    //设备名称spi_oled，可通过命令cat  /proc/devices查看
    //DEV_CNT为1，当前只申请一个设备编号
    ret = alloc_chrdev_region(&oled_devno, 0, DEV_CNT, DEV_NAME);
    if (ret < 0)
    {
        printk("fail to alloc oled_devno\n");
        goto alloc_err;
    }

    //关联字符设备结构体cdev与文件操作结构体file_operations
    oled_chr_dev.owner = THIS_MODULE;
    cdev_init(&oled_chr_dev, &oled_chr_dev_fops);

    // 添加设备至cdev_map散列表中
    ret = cdev_add(&oled_chr_dev, oled_devno, DEV_CNT);
    if (ret < 0)
    {
        printk("fail to add cdev\n");
        goto add_err;
    }

    /*创建类 */
    class_oled = class_create(THIS_MODULE, DEV_NAME);

    /*创建设备 DEV_NAME 指定设备名，*/
    device_oled = device_create(class_oled, NULL, oled_devno, NULL, DEV_NAME);

    error = spi_register_driver(&oled_driver);
    if (error < 0) {
        device_destroy(class_oled, oled_devno);                //清除设备
        class_destroy(class_oled);                                     //清除类
        cdev_del(&oled_chr_dev);                                       //清除设备号
        unregister_chrdev_region(oled_devno, DEV_CNT); //取消注册字符设备
    }
    return error;

add_err:
    // 添加设备失败时，需要注销设备号
    unregister_chrdev_region(oled_devno, DEV_CNT);
    printk(" error! \n");
alloc_err:

    return -1;
}

/*
*驱动注销函数
*/
static void __exit oled_driver_exit(void)
{
    pr_info("oled_driver_exit\n");

    spi_unregister_driver(&oled_driver);
    gpio_free(oled_control_pin_number);

    /*删除设备*/
    device_destroy(class_oled, oled_devno);            //清除设备
    class_destroy(class_oled);                                         //清除类
    cdev_del(&oled_chr_dev);                                           //清除设备号
    unregister_chrdev_region(oled_devno, DEV_CNT); //取消注册字符设备
}
```

代码说明：

- 第2-9行：这里定义了两个匹配表，第一个是传统的匹配表(可省略)。第二个是和设备树节点匹配的匹配表，保证与设备树节点.compatible属性设定值相同即可
- 第12-21行：定义spi_driver类型结构体。该结构体可类比i2c_driver和platform_driver
- 第26-95行：驱动的入口和出口函数，在入口函数只需要注册一个spi驱动和注册字符设备，在出口函数中注销它们

#### 2.2.3 probe函数实现

在.probe函数中完成两个主要工作是，申请gpio控制D/C引脚和初始化spi：

```c
static int oled_probe(struct spi_device *spi)
{
    struct device_node *node=spi->dev.of_node;

    printk(KERN_EMERG " match successed \n");

    /*获取 oled 的 D/C 控制引脚并设置为输出，默认高电平*/
    oled_control_pin_number = of_get_named_gpio(node, "dc_control_pin", 0);

    printk("oled_control_pin_number = %d,\n ", oled_control_pin_number);

    gpio_request(oled_control_pin_number, "dc_control_pin");
    gpio_direction_output(oled_control_pin_number, 1);

    /*初始化spi*/
    oled_spi_device = spi;
    oled_spi_device->mode = SPI_MODE_0;
    oled_spi_device->max_speed_hz = 2000000;
    spi_setup(oled_spi_device);

    /*打印oled_spi_device 部分内容*/
    printk("max_speed_hz = %d\n", oled_spi_device->max_speed_hz);
    printk("chip_select = %d\n", (int)oled_spi_device->chip_select);
    printk("bits_per_word = %d\n", (int)oled_spi_device->bits_per_word);
    printk("mode = %02X\n", oled_spi_device->mode);
    printk("cs_gpio = %02X\n", oled_spi_device->cs_gpio);

        return 0;

}
```

.probe函数介绍如下：

- 第8-13行：获取spi_oled的D/C控制引脚。并设置为高电平
- 第16行：.probe函数传回的spi_device结构体，根据之前讲解，该结构体代表了一个spi设备，我们通过它配置SPI
- 第17行：设置SPI模式为SPI_MODE_0
- 第18行：设置最高频率为2000000，设备树中也设置了该属性，则这里设置的频率为最终值

#### 2.2.4 字符设备操作函数实现

字符设备操作函数集是驱动对外的接口，我们要在这些函数中实现对spi_oled的初始化、写入、关闭等等工作。这里共实现三个函数，.open函数用于实现spi_oled的初始化，.write函数用于向spi_oled写入显示数据，.release函数用于关闭spi_oled。

**.open函数实现**

在open函数中完成spi_oled的初始化，代码如下：

```c
/*字符设备操作函数集，open函数实现*/
static int oled_open(struct inode *inode, struct file *filp)
{
    spi_oled_init(); //初始化显示屏
    return 0;
}

/*oled 初始化函数*/
void spi_oled_init(void)
{
    /*初始化oled*/
    oled_send_commands(oled_spi_device, oled_init_data, sizeof(oled_init_data));

    /*清屏*/
    oled_fill(0x00);
}

static int oled_send_command(struct spi_device *spi_device, u8 *commands, u16 lenght)
{
    int error = 0;
    struct spi_message *message;   //定义发送的消息
    struct spi_transfer *transfer; //定义传输结构体

    /*申请空间*/
    message = kzalloc(sizeof(struct spi_message), GFP_KERNEL);
    transfer = kzalloc(sizeof(struct spi_transfer), GFP_KERNEL);
    /*设置 D/C引脚为低电平*/
    gpio_direction_output(oled_control_pin_number, 0);
    /*填充message和transfer结构体*/
    transfer->tx_buf = commands;
    transfer->len = lenght;
    spi_message_init(message);
    spi_message_add_tail(transfer, message);
    error = spi_sync(spi_device, message);

    kfree(message);
    kfree(transfer);
    if (error != 0)
    {
            printk("spi_sync error! \n");
            return -1;
    }
    return error;
}
```

如上代码所示，open函数只调用了自定义spi_oled_init函数，在spi_oled_init函数函数最终会调用oled_send_command函数初始化spi_oled，然后调用清屏函数。这里主要讲解oled_send_command函数：

- 第21、22行：定义spi_message结构体和spi_transfer结构体
- 第25、26行：为节省内核栈空间这里使用kzalloc为它们分配空间
- 第28行：设置 D/C引脚为低电平，前面说过，spi_oled的D/C引脚用于控制发送的命令或数据，低电平时表示发送的是命令
- 第30-34行：这里就是我们之前讲解的发送流程依次为初始化spi_transfer结构体指定要发送的数据、初始化消息结构体、将消息结构体添加到队尾部、调用spi_sync函数执行同步发送
- 第36-43行：释放空间

**.write函数实现**

.write函数用于接收来自应用程序的数据，并显示这些数据。函数实现如下所示：

```c
/*字符设备操作函数集，.write函数实现*/
static int oled_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *off)
{
    int copy_number=0;
    /*申请内存*/
    oled_display_struct *write_data;
    write_data = (oled_display_struct*)kzalloc(cnt, GFP_KERNEL);
    copy_number = copy_from_user(write_data, buf,cnt);
    oled_display_buffer(write_data->display_buffer, write_data->x, write_data->y, write_data->length);
    /*释放内存*/
    kfree(write_data);
    return 0;
}

static int oled_display_buffer(u8 *display_buffer, u8 x, u8 y, u16 length)

/*数据发送结构体*/
typedef struct oled_display_struct
{
    u8 x;
    u8 y;
    u32 length;
    u8 display_buffer[];
}oled_display_struct;
```

代码介绍如下：

- 第2行：.write函数，我们重点关注两个参数buf保存来自应用程序的数据地址，参数cnt指定数据长度
- 第6-8行：使用kzalloc分配空间并拷贝用户数据
- 第9行：调用自定义函数oled_display_buffer显示数据
- 第25行：oled_display_struct结构体是自定义的一个结构体。它是一个可变长度结构体，参数x、y用于指定数据显示位置，参数length指定数据长度，柔性数组display_buffer[]用于保存来自用户空间的显示数据

**.release函数实现**

.release函数功能仅仅是向spi_oled显示屏发送关闭显示命令：

```c
/*字符设备操作函数集，.release函数实现*/
static int oled_release(struct inode *inode, struct file *filp)
{
    oled_send_command(oled_spi_device, 0xae);//关闭显示
    return 0;
}
```

#### 2.2.5 编写测试应用程序

测试应用程序主要用来测试驱动，实现oled显示屏实现刷屏、显示文字、显示图片。测试程序需要用到字符以及图片的点阵数据保存在oled_code_table.c文件，为方便管理我们编写了一个简单makefile文件方便我们编译程序。

其makefile文件如下所示：

```makefile
out_file_name = "test_app"

all: test_app.c oled_code_table.c
    aarch64-linux-gnu-gcc $^ -o $(out_file_name)

.PHONY: clean
clean:
    rm $(out_file_name)
```

下面是我们的测试程序源码：

```c
/*点阵数据*/
extern unsigned char F16x16[];
extern unsigned char F6x8[][6];
extern unsigned char F8x16[][16];
extern unsigned char BMP1[];

int main(int argc, char *argv[])
{
    int error = -1;
    /*打开文件*/
    int fd = open("/dev/spi_oled", O_RDWR);
    if (fd < 0)
    {
        printf("open file : %s failed !\n", argv[0]);
        return -1;
    }

    while(1)
    {
        /*显示图片*/
        show_bmp(fd, 0, 0, BMP1, X_WIDTH*Y_WIDTH/8);

        sleep(2);
        oled_fill(fd, 0, 0, 127, 7, 0x00);  //清屏

        oled_show_F16X16_letter(fd,0, 0, F16x16, 4);  //显示汉字
        oled_show_F8X16_string(fd,0,2,"F8X16:THIS IS SPI TEST APP");
        oled_show_F6X8_string(fd, 0, 6,"F6X8:THIS IS SPI TEST APP");

        sleep(2);
        oled_fill(fd, 0, 0, 127, 7, 0x00);  //清屏

        oled_show_F8X16_string(fd,0,0,"Testing is completed");

        sleep(2);
        oled_fill(fd, 0, 0, 127, 7, 0x00);  //清屏
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

测试程序很简单，完整代码请参考配套例程，结合代码简单介绍如下：

- 第2-5行：测试程序要用到的点阵数据，我们显示图片、汉字之前都要把它们转化为点阵数据
- 第11行：打开spi_oled的设备节点
- 第21行：显示图片测试
- 第27-29行：测试显示汉字和不同规格的字符

### 2.3 实验准备

#### 2.3.1 编译设备树插件

编写好设备树插件后，进行编译。rk356x系列在内核源码根目录下执行以下命令：

```shell
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig

make ARCH=arm64 -j4 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

在"内核源码的/arch/arm64/boot/dts/rockchip/overlays"目录下，会生成lubancat-spi3-m1-oled-overlay.dtbo。

#### 2.3.2 编译驱动程序

将 **linux_driver/SPI_OLED/** 放到内核源码同级目录，执行里面的MakeFile，生成spi_oled.ko。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4b99133403179d65f25336315b96cbf9.png)

#### 2.3.3 编译应用程序

将 **linux_driver/SPI_OLED/test_app** 目录中执行里面的MakeFile，生成test_app。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/595a998ad8742f981ee65c90a4556bf7.png)

### 2.4 程序运行结果

将前面生成的设备树插件、驱动程序、应用程序通过scp等方式拷贝到开发板。

**加载设备树和驱动文件**

在lubancat2板卡上，将设备树插件拷贝到板卡/boot/dtb/overlay/目录下，并打开/boot/uEnv目录下的uEnvLubanCat2.txt文件添加 _dtoverlay=/dtb/overlay/lubancat-spi3-m1-oled-overlay.dtbo_ ，保存重启。

![20250626_201429_716_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8c4e141e97bd04443589e7a522fc63da.png)

加载驱动程序使用命令 `sudo insmod spi_oled.ko`，将会打印match successed和spi相关信息：

![20250626_201432_050_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1b300514c0c3f5a6ca38503e6ccd60d7.png)

**测试效果**

驱动加载成功后直接运行测试应用程序 `sudo ./test_app`，正常情况下显示屏会显示并自动切换设定的内容，如下所示：

![20250626_201435_998_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/eaa9b07226aaeb28f3be6e3909f2b78a.png)

## 3 Linux通用SPI设备驱动spidev

与I2C设备类似，在Linux内核中也有着通用SPI设备驱动。在本节将会讲解通用SPI设备驱动的使用，并讲解如何在应用程序中通过ioctl对SPI进行配置和使用。

### 3.1 内核和设备树配置

通用SPI设备驱动在迅为提供的Linux内核中默认已经勾选了，具体路径如下所示：

```txt
> Device Drivers
    > SPI support  
        <*> User mode SPI device driver support
```

宏定义：`CONFIG_SPI_SPIDEV`

除了内核支持之外，还需要修改设备树：

```dts
&spi3 {
    status = "okay";  // 启用SPI3控制器
    pinctrl-names = "default", "high_speed";  // 定义两种引脚状态模式
    pinctrl-0 = <&spi3m1_cs0 &spi3m1_pins>;  // 默认模式的引脚配置
    pinctrl-1 = <&spi3m1_cs0 &spi3m1_pins_hs>;  // 高速模式的引脚配置
    
    spidev@0 {  // 通用SPI设备，使用片选CS0
        compatible = "rockchip,spidev";  // Rockchip平台的SPI设备驱动
        reg = <0>;  // SPI片选信号CS0
        spi-max-frequency = <24000000>;  // 最大SPI通信频率24MHz
        status = "okay";  // 启用此设备
    };
};
```

### 3.2 设备节点

开发板启动之后，如果存在/dev/spidev3.0设备节点，证明设备树及内核配置正确：

```
/dev/spidev3.0
```

- /dev/spidev3.0 表示一个SPI总线上的具体设备。3.0是一个标识符，用于区分系统中的不同SPI控制器和设备
- 第一个数字3：表示SPI总线的编号
- 第二个数字0：表示连接在该SPI总线上的具体设备编号

### 3.3 spidev驱动分析

```c
// SPI设备的主设备号定义
#define SPIDEV_MAJOR        153        /* assigned */

// SPI次设备号的最大数量定义
#define N_SPI_MINORS        32         /* ... up to 256 */
```

驱动流程图：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a502986a36359cd351fa4134360c4182.png)

#### 3.3.1 初始化

目录：spi/spidev.c

```c
static int __init spidev_init(void)
{
    int status; // 状态返回值

    /* Claim our 256 reserved device numbers. Then register a class
     * that will key udev/mdev to add/remove /dev nodes. Last, register
     * the driver which manages those device numbers.
     */
    
    // 编译时检查：确保N_SPI_MINORS不超过256
    BUILD_BUG_ON(N_SPI_MINORS > 256);
    
    // 注册字符设备驱动，申请主设备号
    status = register_chrdev(SPIDEV_MAJOR, "spi", &spidev_fops);
    if (status < 0)
        return status; // 注册失败直接返回错误码

    // 创建设备类，用于udev自动创建设备节点
    spidev_class = class_create(THIS_MODULE, "spidev");
    if (IS_ERR(spidev_class)) {
        // 类创建失败，注销字符设备驱动
        unregister_chrdev(SPIDEV_MAJOR, spidev_spi_driver.driver.name);
        return PTR_ERR(spidev_class);
    }

    // 注册SPI驱动
    status = spi_register_driver(&spidev_spi_driver);
    if (status < 0) {
        // SPI驱动注册失败，清理已创建的资源
        class_destroy(spidev_class);
        unregister_chrdev(SPIDEV_MAJOR, spidev_spi_driver.driver.name);
    }
    
    return status; // 返回最终状态
}

// 定义模块初始化函数
module_init(spidev_init);
```

第一步操作集注册：

```c
static const struct file_operations spidev_fops = {
    .owner =        THIS_MODULE,
    /* REVISIT switch to aio primitives, so that userspace
     * gets more complete API coverage. It'll simplify things
     * too, except for the locking.
     */
    .write =        spidev_write,         // 写操作函数
    .read =         spidev_read,          // 读操作函数
    .unlocked_ioctl = spidev_ioctl,       // 无锁ioctl控制函数
    .compat_ioctl = spidev_compat_ioctl,  // 32位兼容ioctl函数
    .open =         spidev_open,          // 设备打开函数
    .release =      spidev_release,       // 设备释放函数
    .llseek =       no_llseek,            // 不支持文件定位
};
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e55b0f4c4e1a427db23c6656ed01108c.png)

然后会注册：

```c
static struct spi_driver spidev_spi_driver = {
    .driver = {
        .name =         "spidev",                           // 驱动名称
        .of_match_table = of_match_ptr(spidev_dt_ids),      // 设备树匹配表
        .acpi_match_table = ACPI_PTR(spidev_acpi_ids),      // ACPI匹配表
    },
    .probe =        spidev_probe,        // 设备探测函数
    .remove =       spidev_remove,       // 设备移除函数
    /* NOTE: suspend/resume methods are not necessary here.
     * We don't do anything except pass the requests to/from
     * the underlying controller. The refrigerator handles
     * most issues; the controller driver handles the rest.
     */
};
```

```c
#ifdef CONFIG_OF
static const struct of_device_id spidev_dt_ids[] = {
    { .compatible = "rohm,dh2228fv" },        // ROHM DH2228FV器件
    { .compatible = "lineartechnology,ltc2488" }, // Linear LTC2488 ADC
    { .compatible = "ge,achc" },              // GE ACHC器件
    { .compatible = "semtech,sx1301" },       // Semtech SX1301器件
    { .compatible = "rockchip,spidev" },      // Rockchip通用SPI设备
    {},
};
MODULE_DEVICE_TABLE(of, spidev_dt_ids);      // 导出设备树匹配表
#endif
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc773ed8ebb4663bc52b7fd8808fe363.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/daba78000e3553e2a64e6766e4b85921.png)

#### 3.3.2 probe函数

匹配之后，spidev.c的`spidev_probe`会被调用，它会：

- 分配一个spidev_data结构体，用来记录对于的spi_device
- spidev_data会被记录在一个链表里
- 分配一个次设备号，以后可以根据这个次设备号在链表里找到spidev_data
- device_create：这会生产一个设备节点`/dev/spidevB.D`，B表示总线号，D表示它是这个SPI Master下第几个设备

```C
static int spidev_probe(struct spi_device *spi)
{
        struct spidev_data      *spidev;
        int                     status;
        unsigned long           minor;

        /*
         * spidev should never be referenced in DT without a specific
         * compatible string, it is a Linux implementation thing
         * rather than a description of the hardware.
         */
        WARN(spi->dev.of_node &&
             of_device_is_compatible(spi->dev.of_node, "spidev"),
             "%pOF: buggy DT: spidev listed directly in DT\n", spi->dev.of_node);

        spidev_probe_acpi(spi);

        /* Allocate driver data */
        spidev = kzalloc(sizeof(*spidev), GFP_KERNEL);
        if (!spidev)
                return -ENOMEM;

        /* Initialize the driver data */
        spidev->spi = spi;
        spin_lock_init(&spidev->spi_lock);
        mutex_init(&spidev->buf_lock);

        INIT_LIST_HEAD(&spidev->device_entry);

        /* If we can allocate a minor number, hook up this device.
         * Reusing minors is fine so long as udev or mdev is working.
         */
        mutex_lock(&device_list_lock);
        minor = find_first_zero_bit(minors, N_SPI_MINORS);
        if (minor < N_SPI_MINORS) {
                struct device *dev;
                //创建设备号
                spidev->devt = MKDEV(SPIDEV_MAJOR, minor);
                //创建设备
                dev = device_create(spidev_class, &spi->dev, spidev->devt,
                                    spidev, "spidev%d.%d",
                                    spi->master->bus_num, spi->chip_select);
                status = PTR_ERR_OR_ZERO(dev);
        } else {
                dev_dbg(&spi->dev, "no minor number available!\n");
                status = -ENODEV;
        }
        if (status == 0) {
                //设置位
                set_bit(minor, minors);
                //把spidev添加到device_list
                list_add(&spidev->device_entry, &device_list);
        }
        mutex_unlock(&device_list_lock);

        spidev->speed_hz = spi->max_speed_hz;

        if (status == 0)
                spi_set_drvdata(spi, spidev);
        else
                kfree(spidev);

        return status;
}
```

#### 3.3.3 file_operations操作函数

spidev提供了完整的文件操作函数，包括read、write、ioctl等：

```c
static const struct file_operations spidev_fops = {
    .owner =        THIS_MODULE,
    .write =        spidev_write,         // 写操作函数
    .read =         spidev_read,          // 读操作函数
    .unlocked_ioctl = spidev_ioctl,       // 无锁ioctl控制函数
    .compat_ioctl = spidev_compat_ioctl,  // 32位兼容ioctl函数
    .open =         spidev_open,          // 设备打开函数
    .release =      spidev_release,       // 设备释放函数
    .llseek =       no_llseek,            // 不支持文件定位
};
```

**write函数**

```c
/* Write-only message with current device setup */
static ssize_t
spidev_write(struct file *filp, const char __user *buf,
            size_t count, loff_t *f_pos)
{
    struct spidev_data    *spidev;        // SPI设备数据指针
    ssize_t               status = 0;     // 操作状态
    unsigned long         missing;        // 拷贝失败的字节数

    /* chipselect only toggles at start or end of operation */
    // 检查数据长度是否超过缓冲区大小
    if (count > bufsiz)
        return -EMSGSIZE;

    spidev = filp->private_data;          // 获取设备私有数据

    mutex_lock(&spidev->buf_lock);        // 加锁保护缓冲区
    // 从用户空间拷贝数据到内核缓冲区
    missing = copy_from_user(spidev->tx_buffer, buf, count);
    if (missing == 0)
        status = spidev_sync_write(spidev, count);  // 执行同步写操作
    else
        status = -EFAULT;                 // 拷贝失败返回错误
    mutex_unlock(&spidev->buf_lock);      // 解锁

    return status;
}
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e4172aab9182c28a0a9af82033b332c0.png)

**read函数**

```c
/* Read-only message with current device setup */
static ssize_t
spidev_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
{
    struct spidev_data    *spidev;        // SPI设备数据指针
    ssize_t               status = 0;     // 操作状态

    /* chipselect only toggles at start or end of operation */
    // 检查数据长度是否超过缓冲区大小
    if (count > bufsiz)
        return -EMSGSIZE;

    spidev = filp->private_data;          // 获取设备私有数据

    mutex_lock(&spidev->buf_lock);        // 加锁保护缓冲区
    status = spidev_sync_read(spidev, count);  // 执行同步读操作
    if (status > 0) {
        unsigned long    missing;         // 拷贝失败的字节数

        // 将接收缓冲区数据拷贝到用户空间
        missing = copy_to_user(buf, spidev->rx_buffer, status);
        if (missing == status)
            status = -EFAULT;             // 拷贝完全失败
        else
            status = status - missing;     // 返回成功拷贝的字节数
    }
    mutex_unlock(&spidev->buf_lock);      // 解锁

    return status;
}
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c4ca1cb4a4af1cf6c3ebfe2f59b6fa67.png)

**ioctl函数**

ioctl函数提供了丰富的控制功能，包括：

1. SPI通信参数获取
2. SPI通信参数设置
3. 数据的传输

常用的request参数：

- `SPI_IOC_RD_MODE`：读取当前SPI通信模式
- `SPI_IOC_WR_MODE`：设置SPI通信模式
- `SPI_IOC_RD_BITS_PER_WORD`：读取每个数据字的位数
- `SPI_IOC_WR_BITS_PER_WORD`：设置每个数据字的位数
- `SPI_IOC_RD_MAX_SPEED_HZ`：读取SPI总线的最大速度
- `SPI_IOC_WR_MAX_SPEED_HZ`：设置SPI总线的最大速度
- `SPI_IOC_MESSAGE(N)`：执行SPI传输的读写操作
- `SPI_IOC_RD_LSB_FIRST`：读取LSB优先设置
- `SPI_IOC_WR_LSB_FIRST`：设置LSB优先模式

### 3.4 spidev缺点

使用read、write函数时，只能读、写，这是半双工方式。使用ioctl可以达到全双工的读写。

但是spidev有2个缺点：

- 不支持中断
- 只支持同步操作，不支持异步操作：就是read/write/ioctl这些函数只能执行完毕才可返回

## 4 总结

本章通过两个实例详细介绍了SPI设备驱动的开发：

1. **OLED驱动实例**：
    
    - 完整展示了SPI设备驱动的编写流程
    - 包含设备树配置、驱动编写、应用测试
    - 重点介绍了SPI数据传输的实现
2. **spidev通用驱动**：
    
    - 介绍了Linux内核提供的通用SPI驱动
    - 分析了spidev的实现原理
    - 说明了如何在用户空间访问SPI设备

> [!tip]+ 实用技巧 对于简单的SPI设备，推荐直接使用spidev驱动，可以大大简化开发工作。只有当需要中断支持或异步操作时，才需要编写专门的设备驱动。

下一章将介绍SPI数据传输机制的内部实现，以及如何进行SPI调试和性能优化。