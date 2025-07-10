
## 1 RK3568功能特点

以下是`RK3568` `SPI` 驱动支持的一些特性︰

- 默认采用摩托罗拉 `SPI` 协议
    
- 支持 `8` 位和 `16` 位
    
- 软件可编程时钟频率和传输速率高达 `50MHz`
    
- 支持 `SPI` `4` 种传输模式配置
    
- 每个 `SPI` 控制器支持一个到两个片选
    
- 框架支持 `slave` 和 `master` 两种模式
    

##内核软件

### 1.1 代码路径

```C
drivers/spi/spi.c                                                 spi驱动框架
drivers/spi/spi-rockchip.c rk                         spi各接口实现
drivers/spi/spidev.c                                         创建spi设备节点，用户态使用。
drivers/spi/spi-rockchip-test.c                 spi测试驱动，需要自己手动添加到Makefile编译
Documentation/spi/spidev_test.c                 用户态spi测试工具
```

### 1.2 SPI 设备配置 —— RK 芯片作 Master 端

内核配置

```C
Device Drivers --->[*] SPI support ---><* Rockchip SPI controller driver
```

`DTS` 节点配置

```C
&spi1 { //引用spi 控制器节点
        status = "okay";//assigned-clock-rates = <200000000>; //默认不用配置，SPI 设备工作时钟值，io 时钟由工作时钟分频获取
        dma-names = "tx","rx"; //使能DMA模式，通讯长度少于32字节不建议用，置空赋值去掉使能，如"dma-names;";//rx-sample-delay-ns = <10>; //默认不用配置，读采样延时
        spi_test@10 {
                compatible ="rockchip,spi_test_bus1_cs0"; //与驱动对应的名字
                reg = <0;
                spi-cpha; //设置 CPHA = 1，不配置则为 0
                spi-cpol; //设置 CPOL = 1，不配置则为 0
                spi-lsb-first; //IO 先传输 lsb
                spi-max-frequency = <24000000; //spi clk输出的时钟频率，不超过50M
                status = "okay"; //使能设备节点};};
```

`spiclkassigned-clock-rates` 和 `spi-max-frequency` 的配置说明：

- `spi-max-frequency` 是 `SPI` 的输出时钟，由 `SPI` 工作时钟 `spiclk assigned-clock-rates`内部分频后输出，由于内部至少 `2` 分频，所以关系是 s`piclk assigned-clock-rate`s >= `2*spi-max-frequency`；
    
- 假定需要 `50MHz` 的 `SPI` `IO` 速率，可以考虑配置（记住内部分频为偶数分频）`spi_clk assigned-clock-rates = <100000000>`，`spi-max-frequency =<50000000>`，即工作时钟 `100 MH`z（PLL 分频到一个不大于 `100MHz` 但最接近的值），然后内部二分频最终 `IO` 接近 `50 MHz`；
    
- `spiclk assigned-clock-rates` 不要低于 `24M`，否则可能有问题；
    
- 如果需要配置 `spi-cpha` 的话， 要求 `spiclk assigned-clock-rates <= 6M`, `1M <=spi-max-frequency >=3M`。
    

### 1.3 SPI 设备配置 —— RK 芯片作 Slave 端

`SPI` 做 `slave` 使用的接口和 `master` 模式一样，都是 `spi_read` 和 `spi_write`。 内核配置

```C
Device Drivers --->[*] SPI support --->[*] SPI slave protocol handlers
```

`DTS` 节点配置

```C
&spi1 {
        status = "okay";
        dma-names = "tx","rx";
        spi-slave; //使能 slave 模式
                slave { //按照框架要求，SPI slave子节点的命名需以 "slave" 开始
                compatible ="rockchip,spi_test_bus1_cs0";
                reg = <0;
                id = <0;
        };
 };
```

> 注意： 实际使用场景可以考虑主从之间增加一个 `gpio`，主设备输出来通知从设备 `transfer ready` 来减少`CPU`忙等时间

### 1.4 SPI 设备驱动介绍

设备驱动使用`spi`需要调用`spi_register_driver`注册`spi`设备：

```C++
#include <linux/init.h>
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/spi/spi.h>

static int spi_test_probe(struct spi_device *spi)
{
        int ret;

        if(!spi)
                return -ENOMEM;
        spi->bits_per_word= 8;
        ret= spi_setup(spi);
        if(ret < 0) {
                dev_err(&spi->dev,"ERR: fail to setup spi\n");
                return-1;
        }
        return ret;
}

static int spi_test_remove(struct spi_device *spi)
{
        printk("%s\n",__func__);
        return 0;
}

static const struct of_device_id spi_test_dt_match[]= {
        {.compatible = "rockchip,spi_test_bus1_cs0", },
        {.compatible = "rockchip,spi_test_bus1_cs1", },
        {},
};

MODULE_DEVICE_TABLE(of, spi_test_dt_match);

static struct spi_driver spi_test_driver = {
        .driver = {
                .name = "spi_test",
                .owner = THIS_MODULE,
                .of_match_table = of_match_ptr(spi_test_dt_match),
        },
        .probe = spi_test_probe,
        .remove = spi_test_remove,
};

static int __init spi_test_init(void)
{
        int ret = 0;
        ret = spi_register_driver(&spi_test_driver);
        return ret;
}
module_init(spi_test_init);

static void __exit spi_test_exit(void)
{
        return spi_unregister_driver(&spi_test_driver);
}
module_exit(spi_test_exit);
```

对 `SPI` 读写操作请参考 `include/linux/spi/spi.h`，以下简单列出几个

```C
static inline int spi_write(struct spi_device *spi,const void *buf, size_t len)static inline int spi_read(struct spi_device *spi,void *buf, size_t len)static inline int spi_write_and_read(structspi_device *spi, const void *tx_buf, void *rx_buf,size_t len)
```

## 2 常见问题

### 2.1 SPI 无信号

- 调试前确认驱动有跑起来
    
- 确保 `SPI` 4 个引脚的 `IOMUX` 配置无误
    
- 确认 `TX` 送时，`TX` 引脚有正常的波形，`CLK` 有正常的 `CLOCK` 信号，`CS` 信号有拉低
    
- 如果 `clk` 频率较高，可以考虑提高驱动强度来改善信号
    
- 如何简单判断 `SPI` `DMA` 是否使能，串口打印如无以下关键字则 `DMA` 使能成功：
    

```C
[ 0.457137] Failed to request TX DMA channel
[ 0.457237] Failed to request RX DMA channel
```

### 2.2 延迟采样时钟配置方案

对于 `SPI` `io` 速率较高的情形，正常 `SPI` `mode` 可能依旧无法匹配外接器件输出延时，`RK SPI master read` 可能无法采到有效数据，需要启用 `SPI rsd` 逻辑来延迟采样时钟。 `RK SPI rsd（read sample delay）`控制逻辑有以下特性：

- 可配值为 `0，1，2，3`
    
- 延时单位为 `1 spi_clk cycle`，即控制器工作时钟
    

`rx-sample-delay` 实际延时为 `dts` 设定值最接近的 `rsd` 有效值为准，以 `spi_clk` `200MHz`，周期 `5ns` 为例： `rsd` 实际可配延迟为 `0`，`5ns`，`10ns`，`15ns`，`rx-sample-delay` 设定 `12ns`，接近有效值 `10ns`，所以最终为 `10ns`延时。