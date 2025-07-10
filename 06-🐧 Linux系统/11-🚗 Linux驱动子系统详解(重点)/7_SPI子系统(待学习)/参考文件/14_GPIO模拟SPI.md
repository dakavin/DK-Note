
📢SPI 通信协议分为硬件 SPI 和软件 SPI。芯片手册中主要描述硬件 SPI，但当硬件 SPI 不足以满足需求时，可以利用 GPIO 来模拟 SPI。以下是对软件 SPI 的简要说明。

与 I2C 协议相比，SPI 协议相对简单，不需要起始信号、应答信号和终止信号。因此，我们无需从零开始编写模拟 SPI 的驱动代码，可以直接使用 Linux 源码中已有的驱动程序。

  

## 1 内核和设备树配置

首先将模拟 SPI 驱动编译进内核，在 make menuconfig 图形化配置界面中选中如下选项

Device Drivers --->
`  [*]SPI support` -->
    `<*> GPIO-based bitbanging SPI Master //选中`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/359abdc1590bde7803592a1011750281.png)


设备树修改步骤如下所示：

首先对 rk3568-evb1-ddr4-v10.dtsi 设备树进行修改，在根节点添加 SPI5 节点，具体内容如下所示：
```c
// 图片1: SPI5 GPIO模拟SPI设备树配置
spi5: spi@gpio1 {
    compatible = "spi-gpio";                                    // 使用GPIO模拟SPI控制器
    #address-cells = <1>;                                       // 地址单元大小
    gpio-sck = <&gpio1 RK_PB0 GPIO_ACTIVE_LOW>;                // SPI时钟引脚配置
    gpio-miso = <&gpio1 RK_PB0 GPIO_ACTIVE_LOW>;               // 主设备输入从设备输出引脚
    gpio-mosi = <&gpio1 RK_PB1 GPIO_ACTIVE_LOW>;               // 主设备输出从设备输入引脚
    cs-gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_LOW>;                // 片选信号引脚
    num-chipselects = <1>;                                      // 片选信号数量
    pinctrl-names = "default";                                  // 引脚控制状态名称
    pinctrl-0 = <&spi5_gpios>;                                 // 默认引脚控制配置
    status = "disabled";                                        // 设备状态为禁用
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/25e4544379631e9e6e85945e71336f9c.png)

然后对 pinctrl 节点进行追加，追加内容如下所示：
```c
// 图片2: SPI5 GPIO引脚配置组
spi5_gpios: gpios {
    rockchip,pins = <0 RK_PB0 0 &pcfg_pull_none>,    // GPIO0_PB0引脚配置，无上拉下拉
                    <1 RK_PB0 0 &pcfg_pull_none>,    // GPIO1_PB0引脚配置，无上拉下拉
                    <1 RK_PB1 0 &pcfg_pull_none>,    // GPIO1_PB1引脚配置，无上拉下拉
                    <1 RK_PB2 0 &pcfg_pull_none>;    // GPIO1_PB2引脚配置，无上拉下拉
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/17457100e22f3114dbea4c206ceefd63.png)

最后添加SPI按照正常使用方法添加设备即可

## 2 运行测试


首先要确保内核打开了spidev，烧写完成开发板启动之后，使用“ls

/dev/spidev5.0”查看是否存在 spidev5.0 节点，如下图所示

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f7ab67b1c9aaaf1c79a5376f6d4da39a.png)


然后在开发板上短接配置未miso和mosi的引脚，做回环测试，如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/18a3a3947ddcd966ae5de071fa6b95f3.png)


可以看到TX和RX收发的数据是一样，证明SPI回环成功，至此模拟SPI测试就完成了