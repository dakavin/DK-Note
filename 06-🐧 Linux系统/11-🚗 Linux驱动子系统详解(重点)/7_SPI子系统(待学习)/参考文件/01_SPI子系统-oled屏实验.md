
串行外设接口(Serial Peripheral interface)简称SPI，是一种高速的，全双工，同步的通信总线，并且在芯片的管脚上只占用四根线，节约了芯片的管脚。 本章我们介绍下SPI相关的基础知识、内核SPI框架和以spi接口的oled显示屏为例讲解spi驱动程序的编写。

本章主要分为五部分内容。

- 第一部分，spi驱动基本知识，简单讲解SPI物理总线、时序和模式。
    
- 第二部分，分析spi驱动框架和后续使用到的核心数据结构。
    
- 第三部分，分析spi总线驱动和spi核心层以及spi控制器。
    
- 第四部分，编写驱动时会使用到的函数，如同步、异步等。
    
- 第五部分，实验，spi驱动oled液晶屏。

## 1 spi基本知识

### 1.1 spi物理总线

spi总线都可以挂载多个设备，spi支持标准的一主多从，全双工半双工通信等。其中四根控制线包括：
- SCK：时钟线，数据收发同步
- MOSI：数据线，主设备数据发送、从设备数据接收
- MISO：数据线，从设备数据发送，主设备数据接收
- NSS：片选信号线

下图是一个SPI总线通信系统，一个主机MCU通过共享的SCK、MOSI、MISO总线连接三个从机，并通过不同的片选信号SS1、SS2、SS3来选择与哪个从机进行通信
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8e1b6751d6629d97b1e155cd01ce9cff.png)

i2c通过i2c设备地址选择通信设备，而spi通过片选引脚选中要通信的设备。

spi接口支持有多个片选引脚，连接多个SPI从设备，当然也可以使用外部GPIO扩展SPI设备的数量， 这样一个spi接口可连接的设备数由片选引脚树决定。

- 如果使用spi接口提供的片选引脚，spi总线驱动会处理好什么时候选spi设备。
    
- 如果使用外部GPIO作为片选引脚需要我们在spi设备驱动中设置什么时候选中spi。(或者在配置SPI时指定使用的片选引脚)。
    

通常情况下无特殊要求我们使用spi接口提供的片选引脚。

### 1.2 spi时序

下图是一个SPI通信时序图，显示NSS片选信号拉低后，主机通过SCK时钟在MOSI线上发送数据，同时在MISO线上接收从机数据，数据按MSB到LSB的顺序在时钟边沿进行采样和输出
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/556b797dd07c1b681e4c26e892693773.png)

- 起始信号：NSS 信号线由高变低
    
- 停止信号：NSS 信号由低变高
    
- 数据传输：在 SCK的每个时钟周期 MOSI和 MISO同时传输一位数据，高/低位传输没有硬性规定
    
    - 传输单位：8位或16位
        
    - 单位数量：允许无限长的数据传输

### 1.3 spi通信模式

根据总线空闲时 SCK 的时钟状态以及数据采样时刻，SPI的工作模式分为四种：

| SPI模式 | CPOL | CPHA | 空闲时SCK时钟 | 采样时刻 |
| ----- | ---- | ---- | -------- | ---- |
| 0     | 0    | 0    | 低电平      | 奇数边沿 |
| 1     | 0    | 1    | 低电平      | 偶数边沿 |
| 2     | 1    | 0    | 高电平      | 奇数边沿 |
| 3     | 1    | 1    | 高电平      | 偶数边沿 |

- 时钟极性 CPOL：指 SPI 通讯设备处于空闲状态时，SCK信号线的电平信号：
    
    - CPOL=0时，SCK在空闲状态时为低电平
        
    - CPOL=1时，SCK在空闲状态时为高电平
        
- 时钟相位 CPHA：数据的采样的时刻：
    
    - CPHA=0时，数据在SCK时钟线的“奇数边沿”被采样
        
    - CPHA=1时，数据在SCK时钟线的“偶数边沿”被采样

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b8529a13e53ffd1b4d502a712c14afe.png)

如上图所示：

SCK信号线在空闲状态为低电平时，CPOL=0；空闲状态为高电平时，CPOL=1。

CPHA=0，数据在 SCK 时钟线的“奇数边沿”被采样，当 CPOL=0 的时候，时钟的奇数边沿是上升沿，当 CPOL=1 的时候，时钟的奇数边沿是下降沿。

在linux内核中，有定义这几种通讯模式，一般使用较多的是模式0和模式3

```c
// 内核源码/include/linux/spi/spi.h
#define     SPI_CPHA        0x01                    /* clock phase */
#define     SPI_CPOL        0x02                    /* clock polarity */
#define     SPI_MODE_0      (0|0)                   /* (original MicroWire) */
#define     SPI_MODE_1      (0|SPI_CPHA)
#define     SPI_MODE_2      (SPI_CPOL|0)
#define     SPI_MODE_3      (SPI_CPOL|SPI_CPHA)
```

更多有关SPI通信协议的内容可以参考 **【野火®】零死角玩转STM32** 中spi章节。

## 2 spi驱动框架

spi设备驱动和i2c设备驱动非常相似，可对比学习。这一小节主要介绍spi驱动框架以及主要的结构体。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ad2d660024588d2080023ebd91448f34.png)

如框架图所示，spi可分为spi总线驱动和spi设备驱动。spi总线驱动已经由芯片厂商提供，我们适当了解其实现机制。 而spi设备驱动由我们自己编写，则需要明白其中的原理。spi设备驱动涉及到字符设备驱动、SPI核心层、SPI主机驱动，具体功能如下。

- SPI核心层：提供SPI控制器驱动和设备驱动的注册方法、注销方法，SPI Core提供操作接口函数，允许一个spi master，spi driver 和spi device初始化时在SPI Core中进行注册，以及推出时进行注销，也提供上层API接口。
    
- SPI主机驱动：也就是spi控制器驱动(SPI Master Driver)，主要包含SPI硬件体系结构中适配器(spi控制器)的控制，实现spi总线的硬件访问操作。
    
- SPI设备驱动：对应于spi设备端的驱动程序，通过SPI主机驱动与CPU交换数据。

### 2.1 关键数据结构体

这小节将列出整个spi驱动框架所涉及的关键数据结构，可以先浏览下，后续代码中遇到这些数据结构时再回来看详细定义。

#### 2.1.1 spi_master

spi_master是SPI控制器接口，实际也就是spi_controller结构体。
```c
// 内核源码/include/linux/spi/spi.h 内核版本4.19
    #define spi_master  spi_controller
```

#### 2.1.2 spi_controller

spi_controller结构体部分成员变量已经被省略，下面列出的是spi_controller结构体关键成员变量：
```c
// 内核源码/include/linux/spi/spi.h
struct spi_controller {
    struct device   dev;
    ...
    struct list_head list;
    s16                     bus_num;
    u16                     num_chipselect;
    ...
    struct spi_message              *cur_msg;
    ...
    int                     (*setup)(struct spi_device *spi);
    int                     (*transfer)(struct spi_device *spi,
                        struct spi_message *mesg);
    void            (*cleanup)(struct spi_device *spi);
    struct kthread_worker           kworker;
    struct task_struct              *kworker_task;
    struct kthread_work             pump_messages;
    struct list_head                queue;
    struct spi_message              *cur_msg;

    ...
    int (*transfer_one)(struct spi_controller *ctlr, struct spi_device *spi,struct spi_transfer *transfer);
    int (*prepare_transfer_hardware)(struct spi_controller *ctlr);
    int (*transfer_one_message)(struct spi_controller *ctlr,struct spi_message *mesg);
    void (*set_cs)(struct spi_device *spi, bool enable);
    ...
    int                     *cs_gpios;
}
```

spi_controller中包含了各种函数指针，这些函数指针会在SPI核心层中被使用。

- **list：** 链表节点，芯片可能有多个spi控制器
    
- **bus_num：** spi控制器编号
    
- **num_chipselect：** spi片选信号的个数，对不同的从设备进行区分
    
- **cur_msg：** spi_message结构体类型，我们发送的信息都会被封装在这个结构体中。cur_msg，当前正带处理的消息队列
    
- **transfer：** 用于把数据加入控制器的消息队列中
    
- **cleanup：** 当spi_master被释放的时候，完成清理工作
    
- **kworker：** 内核线程工人，spi可以使用异步传输方式发送数据
    
- **pump_messages：** 具体传输工作
    
- **queue：** 所有等待传输的消息队列挂在该链表下
    
- **transfer_one_message：** 发送一个spi消息，类似IIC适配器里的algo->master_xfer，产生spi通信时序
    
- **cs_gpios：** 记录spi上具体的片选信号。

#### 2.1.3 spi_driver

```c
// 内核源码/include/linux/spi/spi.h
struct spi_driver {
    const struct spi_device_id *id_table;
    int                     (*probe)(struct spi_device *spi);
    int                     (*remove)(struct spi_device *spi);
    void                    (*shutdown)(struct spi_device *spi);
    struct device_driver    driver;
};
```

- **id_table：** 用来和spi进行配对。
    
- **.probe：** spi设备和spi驱动匹配成功后，回调该函数指针
    

可以看到spi设备驱动结构体和我们之前讲过的i2c设备驱动结构体 _i2c_driver_ 、平台设备驱动结构体 _platform_driver_ 拥有相同的结构，用法也相同。

#### 2.1.4 spi_device

在spi驱动中一个spi设备结构体代表了一个具体的spi设备，它保存着这个spi设备的详细信息，也可以说是配置信息。 当驱动和设备匹配成功后(例如设备树节点)我们可以从.prob函数的参数中得到spi_device结构体。

```c
// 内核源码/include/linux/spi/spi.h
struct spi_device {
    struct device           dev;
    struct spi_controller   *controller;
    struct spi_controller   *master;        /* compatibility layer */
    u32                     max_speed_hz;
    u8                      chip_select;
    u8                      bits_per_word;
    u16                     mode;
   #define  SPI_CPHA        0x01                    /* clock phase */
   #define  SPI_CPOL        0x02                    /* clock polarity */
   #define  SPI_MODE_0      (0|0)                   /* (original MicroWire) */
   #define  SPI_MODE_1      (0|SPI_CPHA)
   #define  SPI_MODE_2      (SPI_CPOL|0)
   #define  SPI_MODE_3      (SPI_CPOL|SPI_CPHA)
   #define  SPI_CS_HIGH     0x04                    /* chipselect active high? */
   #define  SPI_LSB_FIRST   0x08                    /* per-word bits-on-wire */
   #define  SPI_3WIRE       0x10                    /* SI/SO signals shared */
   #define  SPI_LOOP        0x20                    /* loopback mode */
   #define  SPI_NO_CS       0x40                    /* 1 dev/bus, no chipselect */
   #define  SPI_READY       0x80                    /* slave pulls low to pause */
   #define  SPI_TX_DUAL     0x100                   /* transmit with 2 wires */
   #define  SPI_TX_QUAD     0x200                   /* transmit with 4 wires */
   #define  SPI_RX_DUAL     0x400                   /* receive with 2 wires */
   #define  SPI_RX_QUAD     0x800                   /* receive with 4 wires */
    int                     irq;
    void                    *controller_state;
    void                    *controller_data;
    char                    modalias[SPI_NAME_SIZE];
    int                     cs_gpio;        /* chip select gpio */

    /* the statistics */
    struct spi_statistics   statistics;
};
```

- **dev：** device类型结构体。这是一个设备结构体，我们把它称为spi设备结构体、i2c设备结构体、平台设备结构体都是“继承”自设备结构体。它们根据各自的特点添加自己的成员，spi设备添加的成员就是后面要介绍的成员
    
- **controller：** 当前spi设备挂载在那个spi控制器
    
- **master：** spi_master类型的结构体。在总线驱动中，一个spi_master代表了一个spi总线，这个参数就是用于指定spi设备挂载到那个spi总线上
    
- **max_speed_hz：** 指定SPI通信的最大频率
    
- **chip_select：** spi总选用于区分不同SPI设备的一个标号，不要误以为他是SPI设备的片选引脚。指定片选引脚的成员在下面
    
- **bits_per_word：** 指定SPI通信时一个字节多少位，也就是传输单位
    
- **mode：** SPI工作模式，工作模式如以上代码中的宏定义。包括时钟极性、位宽等等，这些宏定义可以使用或运算“|”进行组合，这些宏定义在SPI协议中有详细介绍，这里不再赘述
    
- **irq：** 如果使用了中断，它用于指定中断号
    
- **cs_gpio：** 片选引脚。在设备树中设置了片选引脚，驱动和设别树节点匹配成功后自动获取片选引脚，我们也可以在驱动总通过设置该参数自定义片选引脚
    
- **statistics：** 记录spi名字，用来和spi_driver进行配对。

#### 2.1.5 spi_transfer

在spi设备驱动程序中，spi_transfer结构体用于指定要发送的数据，后面称为 _传输结构体_ ：
```c

struct spi_transfer {
    /* it's ok if tx_buf == rx_buf (right?)
     * for MicroWire, one buffer must be null
     * buffers must work with dma_*map_single() calls, unless
     *   spi_message.is_dma_mapped reports a pre-existing mapping
     */
    const void      *tx_buf;
    void            *rx_buf;
    unsigned        len;

    dma_addr_t      tx_dma;
    dma_addr_t      rx_dma;
    struct sg_table tx_sg;
    struct sg_table rx_sg;

    unsigned        cs_change:1;
    unsigned        tx_nbits:3;
    unsigned        rx_nbits:3;
#define     SPI_NBITS_SINGLE        0x01 /* 1bit transfer */
#define     SPI_NBITS_DUAL          0x02 /* 2bits transfer */
#define     SPI_NBITS_QUAD          0x04 /* 4bits transfer */
    u8              bits_per_word;
    u16             delay_usecs;
    u32             speed_hz;

    struct list_head transfer_list;
};
```

传输结构体的成员较多，需要我们自己设置的很少，这里只介绍我们常用的配置项。

- **tx_buf：** 发送缓冲区，用于指定要发送的数据地址。
    
- **rx_buf：** 接收缓冲区，用于保存接收得到的数据，如果不接收不用设置或设置为NULL.
    
- **len：** 要发送和接收的长度，根据SPI特性发送、接收长度相等。
    
- **tx_dma、rx_dma：** 如果使用了DAM,用于指定tx或rx DMA地址。
    
- **bits_per_word：** speed_hz，分别用于设置每个字节多少位、发送频率。如果我们不设置这些参数那么会使用默认的配置，也就是我初始化spi是设置的参数。

#### 2.1.6 spi_message

总的来说spi_transfer结构体保存了要发送(或接收)的数据，而在SPI设备驱动中数据是以“消息”的形式发送。 spi_message是消息结构体，我们把它称为消息结构体，发送一个消息分四步， 依次为定义消息结构体、初始化消息结构体、“绑定”要发送的数据(也就是初始化好的spi_transfer结构)、执行发送。

spi_message结构体定义如下所示：
```c
struct spi_message {
    struct list_head        transfers;

    struct spi_device       *spi;

    unsigned                is_dma_mapped:1;

    /* REVISIT:  we might want a flag affecting the behavior of the
     * last transfer ... allowing things like "read 16 bit length L"
     * immediately followed by "read L bytes".  Basically imposing
     * a specific message scheduling algorithm.
     *
     * Some controller drivers (message-at-a-time queue processing)
     * could provide that as their default scheduling algorithm.  But
     * others (with multi-message pipelines) could need a flag to
     * tell them about such special cases.
     */

    /* completion is reported through a callback */
    void                    (*complete)(void *context);
    void                    *context;
    unsigned                frame_length;
    unsigned                actual_length;
    int                     status;

    /* for optional use by whatever driver currently owns the
     * spi_message ...  between calls to spi_async and then later
     * complete(), that's the spi_master controller driver.
     */
    struct list_head        queue;
    void                    *state;
};
```

spi_message结构体成员我们比较陌生，如果我们不考虑具体的发送细节我们可以不用了解这些成员的含义，因为spi_message的初始化以及“绑定”spi_transfer传输结构体都是由内核函数实现。 唯一要说明的是第二个成员“spi”，它是一个spi_device类型的指针，我们讲解spi_device结构体时说过，一个spi设备对应一个spi_device结构体，这个成员就是用于指定消息来自哪个设备。

### 2.2 spi核心层

#### 2.2.1 spi总线注册

linux系统在开机的时候就会执行，自动进行spi总线注册。
```c
//内核源码/drivers/spi/spi.c
static int __init spi_init(void)
{
    int     status;
    ...
    status = bus_register(&spi_bus_type);
    ...
    status = class_register(&spi_master_class);
    ...
}
```
当总线注册成功之后，会在sys/bus下面生成一个spi总线，然后在系统中新增一个设备类，sys/class/目录下会可以找到spi_master类。

#### 2.2.2 spi总线定义

spi_bus_type 总线定义，会在spi总线注册时使用。
```c
struct bus_type spi_bus_type = {
    .name           = "spi",
    .dev_groups     = spi_dev_groups,
    .match          = spi_match_device,
    .uevent         = spi_uevent,
};
```
.match函数指针，设定了spi设备和spi驱动的匹配规则，具体如下spi_match_device。

#### 2.2.3 spi_match_device函数

```c
static int spi_match_device(struct device *dev, struct device_driver *drv)
{
    const struct spi_device *spi = to_spi_device(dev);
    const struct spi_driver *sdrv = to_spi_driver(drv);

    /* Attempt an OF style match */
    if (of_driver_match_device(dev, drv))
        return 1;

    /* Then try ACPI */
    if (acpi_driver_match_device(dev, drv))
        return 1;

    if (sdrv->id_table)
        return !!spi_match_id(sdrv->id_table, spi);

    return strcmp(spi->modalias, drv->name) == 0;
}
```

函数提供了四种匹配方式，设备树匹配方式和acpi匹配方式以及id_table匹配方式，如果前面三种都没有匹配成功，则通过设备名进行配对。

### 2.3 spi控制器驱动

这小节简单介绍下rk3568的控制器，使用的rk3568芯片有4个spi控制器，对应的设备树存在4个节点
```c
// rk3568.dtsi
spi3: spi@fe640000 {

    compatible = "rockchip,rk3066-spi";
    reg = <0x0 0xfe640000 0x0 0x1000>;
    interrupts = <GIC_SPI 106 IRQ_TYPE_LEVEL_HIGH>;
    #address-cells = <1>;
    #size-cells = <0>;
    clocks = <&cru CLK_SPI3>, <&cru PCLK_SPI3>;
    clock-names = "spiclk", "apb_pclk";
    dmas = <&dmac0 26>, <&dmac0 27>;
    dma-names = "tx", "rx";
    pinctrl-names = "default", "high_speed";
    pinctrl-0 = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins>;
    pinctrl-1 = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins_hs>;
    status = "disabled";
    };
```
- **reg** ：为spi3寄存器组相关的起始地址为0xfe640000，寄存器长度为0x1000。
    
- **compatible** 属性值与主机驱动匹配；在rk3568的SPI控制器驱动文件为内核源码/drivers/spi/spi-rockchip.c中可以找到：

```c
// 内核源码/drivers/spi/spi-rockchip.c
static const struct of_device_id rockchip_spi_dt_match[] = {
    { .compatible = "rockchip,px30-spi",   },
    { .compatible = "rockchip,rv1108-spi", },
    { .compatible = "rockchip,rv1126-spi", },
    { .compatible = "rockchip,rk3036-spi", },
    { .compatible = "rockchip,rk3066-spi", },    { .compatible = "rockchip,rk3188-spi", },
    { .compatible = "rockchip,rk3228-spi", },
    { .compatible = "rockchip,rk3288-spi", },
    { .compatible = "rockchip,rk3368-spi", },
    { .compatible = "rockchip,rk3399-spi", },
    { },
};
```

驱动控制器通过下面module_platform_driver()注册：
```c
// 内核源码/drivers/spi/spi-rockchip.c
static struct platform_driver rockchip_spi_driver = {
    .driver = {
        .name       = DRIVER_NAME,
        .pm = &rockchip_spi_pm,
        .of_match_table = of_match_ptr(rockchip_spi_dt_match),
    },
    .probe = rockchip_spi_probe,
    .remove = rockchip_spi_remove,
};

module_platform_driver(rockchip_spi_driver);
```

控制器驱动源码中, module_platform_driver()宏：
```c
// 内核源码/include/linux/platform_device.h
#define module_platform_driver(__platform_driver) \
    module_driver(__platform_driver, platform_driver_register, \
            platform_driver_unregister)
```

module_driver()展开如下:
```c
// 内核源码/include/linux/device.h
#define module_driver(__driver, __register, __unregister, ...) \
static int __init __driver##_init(void) \
{ \
    return __register(&(__driver) , ##__VA_ARGS__); \
} \
module_init(__driver##_init); \
```

- __driver：即为module_platform_driver()宏中的__platform_driver，也就是rockchip_spi_driver。
    
- __register：platform_driver_register
    
- __unregister：platform_driver_unregister
    
- ##__VA_ARGS__：可变参数
    

向函数传入platform_driver结构体类型的spi_imx_driver结构体变量，module_platform_driver(rockchip_spi_driver)， 间接调用platform_driver_register 和 platform_driver_unregister，实现平台驱动函数的注册和注销。

当匹配到“”rockchip,rk3066-spi””时，调用rockchip_spi_probe函数，进行初始化，获取设备树节点信息，初始化spi时钟、dma、中断等， 最后控制器的注册(详细阅读内核源码/drivers/spi/spi-rockchip.c)。

代码如下(部分被省略)：
```c
// 内核源码/drivers/spi/spi-rockchip.c
static int rockchip_spi_probe(struct platform_device *pdev)
{
    int ret;
    struct rockchip_spi *rs;
    struct spi_controller *ctlr;
    struct resource *mem;
    struct device_node *np = pdev->dev.of_node;
    u32 rsd_nsecs;
    bool slave_mode;
    struct pinctrl *pinctrl = NULL;

    slave_mode = of_property_read_bool(np, "spi-slave"); //确认spi是做主设备还是从设备

    if (slave_mode)
        ctlr = spi_alloc_slave(&pdev->dev,
                sizeof(struct rockchip_spi));
    else
        ctlr = spi_alloc_master(&pdev->dev,
                sizeof(struct rockchip_spi)); //分配一个SPI master

    if (!ctlr)
        return -ENOMEM;

    platform_set_drvdata(pdev, ctlr); //设置驱动数据，存储分配的内存指针，即分配的ctrl

    rs = spi_controller_get_devdata(ctlr);
    ctlr->slave = slave_mode;           //标记是否是从设备

    /*..............*/

    spi_enable_chip(rs, false);

    ret = platform_get_irq(pdev, 0);
    if (ret < 0)
        goto err_disable_spiclk;

    ret = devm_request_threaded_irq(&pdev->dev, ret, rockchip_spi_isr, NULL,
            IRQF_ONESHOT, dev_name(&pdev->dev), ctlr);
    if (ret)
        goto err_disable_spiclk;

    /*..............*/

    pm_runtime_set_active(&pdev->dev);
    pm_runtime_enable(&pdev->dev);

    ctlr->auto_runtime_pm = true;
    ctlr->bus_num = pdev->id;
    ctlr->mode_bits = SPI_CPOL | SPI_CPHA | SPI_LOOP | SPI_LSB_FIRST | SPI_CS_HIGH;
    if (slave_mode) {
        ctlr->mode_bits |= SPI_NO_CS;
        ctlr->slave_abort = rockchip_spi_slave_abort;
    } else {
        ctlr->flags = SPI_MASTER_GPIO_SS;
    }
    ctlr->num_chipselect = ROCKCHIP_SPI_MAX_CS_NUM;
    ctlr->dev.of_node = pdev->dev.of_node;
    ctlr->bits_per_word_mask = SPI_BPW_MASK(16) | SPI_BPW_MASK(8) | SPI_BPW_MASK(4);
    ctlr->min_speed_hz = rs->freq / BAUDR_SCKDV_MAX;
    ctlr->max_speed_hz = min(rs->freq / BAUDR_SCKDV_MIN, MAX_SCLK_OUT);

    ctlr->set_cs = rockchip_spi_set_cs;
    ctlr->setup = rockchip_spi_setup;
    ctlr->cleanup = rockchip_spi_cleanup;
    ctlr->transfer_one = rockchip_spi_transfer_one;
    ctlr->max_transfer_size = rockchip_spi_max_transfer_size;
    ctlr->handle_err = rockchip_spi_handle_err;

    ctlr->dma_tx = dma_request_chan(rs->dev, "tx");
    if (IS_ERR(ctlr->dma_tx)) {
        /* Check tx to see if we need defer probing driver */
        if (PTR_ERR(ctlr->dma_tx) == -EPROBE_DEFER) {
            ret = -EPROBE_DEFER;
            goto err_disable_pm_runtime;
        }
        dev_warn(rs->dev, "Failed to request TX DMA channel\n");
        ctlr->dma_tx = NULL;
    }

    ctlr->dma_rx = dma_request_chan(rs->dev, "rx");
    if (IS_ERR(ctlr->dma_rx)) {
        if (PTR_ERR(ctlr->dma_rx) == -EPROBE_DEFER) {
            ret = -EPROBE_DEFER;
            goto err_free_dma_tx;
        }
        dev_warn(rs->dev, "Failed to request RX DMA channel\n");
        ctlr->dma_rx = NULL;
    }

    if (ctlr->dma_tx && ctlr->dma_rx) {
        rs->dma_addr_tx = mem->start + ROCKCHIP_SPI_TXDR;
        rs->dma_addr_rx = mem->start + ROCKCHIP_SPI_RXDR;
        ctlr->can_dma = rockchip_spi_can_dma;
    }

    /*.....*/

    pinctrl = devm_pinctrl_get(&pdev->dev); //获取pinctrl
    if (!IS_ERR(pinctrl)) {
        rs->high_speed_state = pinctrl_lookup_state(pinctrl, "high_speed"); //查找这个pin的default状态
        if (IS_ERR_OR_NULL(rs->high_speed_state)) {
            dev_warn(&pdev->dev, "no high_speed pinctrl state\n");
            rs->high_speed_state = NULL;
        }
    }

    ret = devm_spi_register_controller(&pdev->dev, ctlr);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to register controller\n");
        goto err_free_dma_rx;
    }

    return 0;

    /*.........*/
}
```

- 第19行： 申请内存，实例化master；
    
- 第33-40行：获取中断号，设置中断函数；
    
- 第47-94行：初始化控制器，DMA和运行时电源管理等
    
- 第107行：使用devm_spi_register_controller函数向SPI子系统注册SPI控制器。

### 2.4 spi设备驱动

spi总线驱动，由硬件供应商提供，我们只需要了解，学习其原理就行。 下面涉及的函数，我们将会在spi设备驱动中使用。

spi设备的注册和注销函数分别在驱动的入口和出口函数中调用，这与平台设备驱动、i2c设备驱动相同，

spi设备注册和注销函数如下：
```c
int spi_register_driver(struct spi_driver *sdrv)
static inline void spi_unregister_driver(struct spi_driver *sdrv)
```

对比i2c设备的注册和注销函数，不难发现把“spi”换成“i2c”就是i2c设备的注册和注销函数了，并且用法相同。

**参数：**

- **spi** spi_driver类型的结构体(spi设备驱动结构体)，一个spi_driver结构体就代表了一个spi设备驱动
    

**返回值：**

- **成功：** 0
    
- **失败：** 其他任何值都为错误码

#### 2.4.1 spi_setup函数
函数设置spi设备的片选信号、传输单位、最大传输速率等，函数中调用spi控制器的成员controller->setup()， 也就是master->setup,在前面的函数rockchip_spi_probe()中初始化了“ctlr->setup = rockchip_spi_setup;”。
```c
// 内核源码/drivers/spi/spi.c
int spi_setup(struct spi_device *spi)
```

**参数：**

- **spi** spi_device spi设备结构体
    

**返回值：**

- **成功：** 0
    
- **失败：** 其他任何值都为错误码

#### 2.4.2 spi_message_init函数

初始化spi_message，
```c
// 内核源码/include/linux/spi/spi.h
static inline void spi_message_init(struct spi_message *m)
{
    memset(m, 0, sizeof *m);
    spi_message_init_no_memset(m);
}
```

**参数：**

- **m** spi_message 结构体指针，spi_message结构体定义和介绍可在前面关键数据结构中找到。
    

**返回值：** 无。

#### 2.4.3 spi_message_add_tail函数

```c
// 内核源码/include/linux/spi/spi.h
static inline void spi_message_add_tail(struct spi_transfer *t, struct spi_message *m)
{
    list_add_tail(&t->transfer_list, &m->transfers);
}
```

这个函数很简单就是将将spi_transfer结构体添加到spi_message队列的末尾。

### 2.5 spi同步与互斥

spi_message通过成员变量queue将一系列的spi_message串联起来，第一个spi_message挂在struct list_head queue下面 spi_message还有struct list_head transfers成员变量，transfer也是被串联起来的，如下图所示。
![20250626_200704_969_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/238fa8c32f8bb146ed453f7e1890687e.png)

#### 2.5.1 SPI同步传输数据

阻塞当前线程进行数据传输，spi_sync()内部调用__spi_sync()函数，mutex_lock()和mutex_unlock()为互斥锁的加锁和解锁。
```c
// 内核源码/drivers/spi/spi.c
int spi_sync(struct spi_device *spi, struct spi_message *message)
{
    int ret;

    mutex_lock(&spi->controller->bus_lock_mutex);
    ret = __spi_sync(spi, message);
    mutex_unlock(&spi->controller->bus_lock_mutex);

    return ret;
}
```

`__spi_sync()`函数实现如下：
```c
// 内核源码/drivers/spi/spi.c
static int __spi_sync(struct spi_device *spi, struct spi_message *message)
{
    int status;
    struct spi_controller *ctlr = spi->controller;
    unsigned long flags;

    status = __spi_validate(spi, message);
    if (status != 0)
        return status;

    message->complete = spi_complete;
    message->context = &done;
    message->spi = spi;
    ...
    if (ctlr->transfer == spi_queued_transfer) {
        spin_lock_irqsave(&ctlr->bus_lock_spinlock, flags);

        trace_spi_message_submit(message);

        status = __spi_queued_transfer(spi, message, false);

        spin_unlock_irqrestore(&ctlr->bus_lock_spinlock, flags);
    } else {
        status = spi_async_locked(spi, message);
    }

    if (status == 0) {
        ...
        wait_for_completion(&done);
        status = message->status;
    }
    message->context = NULL;
    return status;
}
```

- 第7-9行： 函数内部首先调用__spi_validate对spi各个通信参数进行校验
    
- 第11-13行： 对message结构体进行初始化，其中第11行，当消息发送完毕后，spi_complete回调函数将被执行。
    
- 第30行： 阻塞当前线程，当message发送完成时结束阻塞。

#### 2.5.2 SPI异步传输数据

```c
// 内核源码/drivers/spi/spi.c
int spi_async(struct spi_device *spi, struct spi_message *message)
{
    ...
    ret = __spi_async(spi, message);
    ...
}
```
在驱动程序中调用async时不会阻塞当前进程，只是把当前message结构体添加到当前spi控制器成员queue的末尾。 然后在内核中新增加一个工作，这个工作的内容就是去处理这个message结构体。

```c
// 内核源码/drivers/spi/spi.c
static int __spi_async(struct spi_device *spi, struct spi_message *message)
{
    struct spi_controller *ctlr = spi->controller;
    ...
    return ctlr->transfer(spi, message);
}
```

## 3 oled屏幕驱动实验

spi_oled驱动和我们上一节介绍的i2c_mpu6050设备驱动非常相似，可对比学习，推荐先学习i2c_mpu6050驱动。

**本章配套源码和设备树插件位于linux_driver/SPI_OLED**。

### 3.1 硬件介绍

本实验以lubancat2板卡为例，其他Lubancat_RK系列板卡同样可以，需要自行查找选定spi接口，oled模块使用的是 [《野火【OLED屏_SPI_0.96寸】模块资料》](https://doc.embedfire.com/products/link/zh/latest/module/screen/oled_spi_0.96.html) 。

#### 3.1.1 硬件连接

在oled驱动中我们使用lubuncat2的spi3，可以通过rk3568数据手册查到，spi使用到的引脚如下。
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

> [!tip]+ 提示
> lubancat2板卡引出引脚，请看“LubanCat-RK系列板卡快速使用手册”的40pin引脚对照图。

#### 3.1.2 设备树插件

设备树插件书写格式不变，我们重点讲解spi_oled设备节点。

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

- 第14-18行: 开启spi3，指定使用的pinctrl节点，也就是说指定spi3要使用的引脚,详细描述如下:

```c
// 内核源码/arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi
spi3 {
    /*...............*/
    spi3m1_pins: spi3m1-pins {
        rockchip,pins =
            /* spi3_clkm1 */
            <4 RK_PC2 2 &pcfg_pull_none>,
            /* spi3_misom1 */
            <4 RK_PC5 2 &pcfg_pull_none>,
            /* spi3_mosim1 */
            <4 RK_PC3 2 &pcfg_pull_none>;
    };

    spi3m1_cs0: spi3m1-cs0 {
        rockchip,pins =
            /* spi3_cs0m1 */
            <4 RK_PC6 2 &pcfg_pull_none>;
    };

    spi3m1_cs1: spi3m1-cs1 {
        rockchip,pins =
            /* spi3_cs1m1 */
            <4 RK_PD1 2 &pcfg_pull_none>;
    };
};

spi3-hs {
    /*...............*/
            spi3m1_pins_hs: spi3m1-pins {
                    rockchip,pins =
                            /* spi3_clkm1 */
                            <4 RK_PC2 2 &pcfg_pull_up_drv_level_1>,
                            /* spi3_misom1 */
                            <4 RK_PC5 2 &pcfg_pull_up_drv_level_1>,
                            /* spi3_mosim1 */
                            <4 RK_PC3 2 &pcfg_pull_up_drv_level_1>;
            };

            spi3m1_cs0_hs: spi3m1-cs0 {
                    rockchip,pins =
                            /* spi3_cs0m1 */
                            <4 RK_PC6 2 &pcfg_pull_up_drv_level_1>;
            };

            spi3m1_cs1_hs: spi3m1-cs1 {
                    rockchip,pins =
                            /* spi3_cs1m1 */
                            <4 RK_PD1 2 &pcfg_pull_up_drv_level_1>;
            };
    };
```

- 第18行: 指定使用的片选引脚，我们这里使用的是GPIO4_C6。
    
- 第21行: 向spi3节点追加spi_oled设备节点
    
- 第23行: 设置reg属性为0,表示spi_oled连接到spi3的通道0
    
- 第24行: 设置SPI传输的最大频率，根据实际oled设备
    
- 第25行: 指定spi_oled使用的D/C控制引脚，在驱动程序中会控制该引脚设置发送的是命令还是数据。
    

向pinctrl子系统添加引脚具体内容可参考 _GPIO子系统章节_ ，设备树的几个引脚与spi_oled显示屏引脚对应关系、引脚的功能、以及在开发板上的位置如前面表格所示。 需要注意的是spi_oled显示屏没有MISO引脚，直接空出即可，spi_oled显示屏需要一个额外的引脚连接D/C, 用于控制spi发送的是数据还是控制命令(高电平是数据，低电平是控制命令)。

### 3.2 实验代码讲解

#### 3.2.1 编程思路

spi_oled驱动使用设备树插件方式开发,主要工作包三部分内容。

- 第一，编写spi_oled的设备树插件，开启spi3，在spi3设备节点下添加spi_oled节点。(硬件部分已介绍)，
    
- 第二，编写spi_oled驱动程序，包含驱动的入口、出口函数实现，.prob函数实现，file_operations函数集实现。
    
- 第三，编写简单测试应用程序。

#### 3.2.2 驱动的入口和出口函数

驱动入口和出口函数与I2C_mpu6050驱动相似，只是把i2c替换为spi，源码如下所示。
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

- 第2-9行： 这里定义了两个匹配表，第一个是传统的匹配表(可省略)。第二个是和设备树节点匹配的匹配表，保证与设备树节点.compatible属性设定值相同即可。
    
- 第12-21行： 定义spi_driver类型结构体。该结构体可类比i2c_driver和platform_driver。
    
- 第26-95行： 驱动的入口和出口函数，在入口函数只需要注册一个spi驱动和注册字符设备，在出口函数中注销它们。

#### 3.2.3 probe函数实现

在.prob函数中完成两个主要工作是，申请gpio控制D/C引脚和初始化spi。

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

.prob函数介绍如下：

- 第8-13行： 获取spi_oled的D/C控制引脚。并设置为高电平。
    
- 第16行： .prob函数传回的spi_device结构体，根据之前讲解，该结构体代表了一个spi设备，我们通过它配置SPI,这里设置的内容将会覆盖设备树节点中设置的内容。
    
- 第17行： 设置SPI模式为SPI_MODE_0。
    
- 第18行： 设置最高频率为2000000，设备树中也设置了该属性，则这里设置的频率为最终值。

#### 3.2.4 字符设备操作函数实现

字符设备操作函数集是驱动对外的接口，我们要在这些函数中实现对spi_oled的初始化、写入、关闭等等工作。 这里共实现三个函数，.open函数用于实现spi_oled的初始化，.write函数用于向spi_oled写入显示数据，.release函数用于关闭spi_oled。

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

如上代码所示，open函数只调用了自定义spi_oled_init函数，在spi_oled_init函数函数最终会调用oled_send_command函数初始化spi_oled，然后调用清屏函数。 这里主要讲解oled_send_command函数：

- 第21、22行： 定义spi_message结构体和spi_transfer结构体。
    
- 第25、26行： 为节省内核栈空间这里使用kzalloc为它们分配空间，这两个结构体大约占用100字节，推荐这样做。
    
- 第28行： 设置 D/C引脚为低电平，前面说过，spi_oled的D/C引脚用于控制发送的命令或数据，低电平时表示发送的是命令。
    
- 第30-34行： 这里就是我们之前讲解的发送流程依次为初始化spi_transfer结构体指定要发送的数据、初始化消息结构体、将消息结构体添加到队尾部、调用spi_sync函数执行同步发送。。
    
- 第36-43行： 释放空间。
    

**.write函数实现**

.write函数用于接收来自应用程序的数据，并显示这些数据。函数实现如下所示：
```c
/*字符设备操作函数集，.write函数实现*/
static int oled_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *off)
{
    int copy_number=0;
    /*申请内存*/
    oled_display_struct *write_data;
    write_data = (oled_display_struct*)kzalloc(cnt, GFP_KERNEL);
    copy_number = copy_from_user(write_data, buf,cnt);
    oled_display_buffer(write_data->display_buffer, write_data->x, write_data->y, write_data->length);
    /*释放内存*/
    kfree(write_data);
    return 0;
}

static int oled_display_buffer(u8 *display_buffer, u8 x, u8 y, u16 length)

/*数据发送结构体*/
typedef struct oled_display_struct
{
    u8 x;
    u8 y;
    u32 length;
    u8 display_buffer[];
}oled_display_struct;
```
代码介绍如下：

- 第2行： .write函数，我们重点关注两个参数buf保存来自应用程序的数据地址，我们需要把这些数据拷贝到内核空间才能使用，参数cnt指定数据长度。
    
- 第6行： 定义oled_display_struct结构体并保存来自用户空间的数据。
    
- 第7-8行： 使用kzalloc为oled_display_struct结构体分配空间，因为在应用程序中使用相同的结构体，所以这里直接根据参数“cnt”分配空间，分配成功后执行“copy_from_user”即可。
    
- 第9行： 调用自定义函数oled_display_buffer显示数据。
    
- 第11行： 释放空间
    
- 第15行： 函数原型如第四部分所示，参数display_buffer指定要显示的点阵数据x、y用于指定显示起始位置，length指定显示长度。具体函数实现也很简单，这里不再赘述。
    
- 第25行： oled_display_struct结构体是自定义的一个结构体。它是一个可变长度结构体，参数 x 、y用于指定数据显示位置，参数length指定数据长度，柔性数组display_buffer[]用于保存来自用户空间的显示数据。
    

**.release函数实现**

.release函数功能仅仅是向spi_oled显示屏发送关闭显示命令，源码如下：
```c
/*字符设备操作函数集，.release函数实现*/
static int oled_release(struct inode *inode, struct file *filp)
{
    oled_send_command(oled_spi_device, 0xae);//关闭显示
    return 0;
}
```

#### 3.2.5 编写测试应用程序

测试应用程序主要用来测试驱动，实现oled显示屏实现刷屏、显示文字、显示图片。 测试程序需要用到字符以及图片的点阵数据保存在oled_code_table.c文件，为方便管理我们编写了一个简单makefile文件方便我们编译程序。

其makefile文件如下所示，也可以直接使用命令编译：
```c
out_file_name = "test_app"

all: test_app.c oled_code_table.c
    aarch64-linux-gnu-gcc $^ -o $(out_file_name)

.PHONY: clean
clean:
    rm $(out_file_name)
```

下面是我们的测试程序源码。如下所示：
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

- 第2-5行： 测试程序要用到的点阵数据，我们显示图片、汉字之前都要把它们转化为点阵数据。野火spi_oled模块配套资料提供有转换工具以及使用说明。
    
- 第11行： 打开spi_oled的设备节点，这个根据自己的驱动而定，我们使用的驱动源码就是这个路径。
    
- 第18行： 显示图片测试，这里需要说明的是由于测试程序不那么完善，图片显示起始位置x坐标应当设置为0，这样在循环显示时才不会乱。显示长度应当为显示屏的像素数除以8，因为每个字节8位，这8位控制8个像素点。
    
- 第22-25行： 测试显示汉字和不同规格的字符。
    
- 第28-33行： 显示测试结束提示语。

### 3.3 实验准备

#### 3.3.1 编译设备树插件

编写好设备树插件后，进行编译。

rk356x系列在内核源码根目录下执行以下命令：
```shell
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig

make ARCH=arm64 -j4 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

在“内核源码的/arch/arm64/boot/dts/rockchip/overlays”目录下，会生成lubancat-spi3-m1-oled-overlay.dtbo， 最后生成的lubancat-spi3-m1-oled-overlay.dtbo文件就是oled屏的设备树插件

#### 3.3.2 编译驱动程序

将 **linux_driver/SPI_OLED/** 放到内核源码同级目录，执行里面的MakeFile，生成spi_oled.ko。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4b99133403179d65f25336315b96cbf9.png)


#### 3.3.3 编译应用程序

将 **linux_driver/SPI_OLED/test_app** 目录中执行里面的MakeFile，生成test_app。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/595a998ad8742f981ee65c90a4556bf7.png)


### 3.4 程序运行结果

将前面生成的设备树插件、驱动程序、应用程序通过scp等方式拷贝到开发板。

**加载设备树和驱动文件**

在lubancat2板卡上，将设备树插件拷贝到板卡/boot/dtb/overlay/目录下, 并打开/boot/uEnv目录下的uEnvLubanCat2.txt文件添加 _dtoverlay=/dtb/overlay/lubancat-spi3-m1-oled-overlay.dtbo_ , 保存重启。
![20250626_201429_716_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8c4e141e97bd04443589e7a522fc63da.png)

加载驱动陈程序使用命令 `sudo insmod spi_oled.ko` ,将会打印match successed和spi相关信息:
![20250626_201432_050_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1b300514c0c3f5a6ca38503e6ccd60d7.png)



**测试效果**

驱动加载成功后直接运行测试应用程序 `sudo ./test_app` ，正常情况下显示屏会显示并自动切换设定的内容，如下所示:
![20250626_201435_998_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/eaa9b07226aaeb28f3be6e3909f2b78a.png)
