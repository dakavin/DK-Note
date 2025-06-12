  

当我们遇到非标准属性或无法理解的属性时， 要如何处理呢？

这时候就不得不提到 bindings 文档了

Documentation/devicetree/bindings 目录是 Linux 内核源码中的一个重要目录， 用于存储设备树（Device Tree） 的 bindings 文档。 设备树是一种描述硬件平台和设备配置的数据结构， 它以一种可移植和独立于具体硬件的方式描述了设备的属性、 寄存器配置、 中断信息等。

bindings 目录中的文档提供了有关设备树的各种设备和驱动程序的详细说明和用法示例。这些文档对于开发人员来说非常重要， 因为它们提供了在设备树中描述硬件和配置驱动程序所需的属性和约定。 bindings 目录截图如下:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f33dae84e6d4d48b5a997257b75a6ff9.png)


接下来对 Documentation/devicetree/bindings 目录的一些常见子目录和其内容的概述：

- arm： 包含与 ARM 体系结构相关的设备和驱动程序的 bindings 文档。
    
- clock： 包含与时钟设备和时钟控制器相关的 bindings 文档。
    
- dma： 包含与直接内存访问（DMA） 控制器和设备相关的 bindings 文档。
    
- gpio： 包含与通用输入输出（GPIO） 控制器和设备相关的 bindings 文档。
    
- i2c： 包含与 I2C 总线和设备相关的 bindings 文档。
    
- interrupt-controller： 包含与中断控制器相关的 bindings 文档。
    
- media： 包含与多媒体设备和驱动程序相关的 bindings 文档。
    
- mfd： 包含与多功能设备（ MFD） 子系统和设备相关的 bindings 文档。
    
- networking： 包含与网络设备和驱动程序相关的 bindings 文档。
    
- power： 包含与电源管理子系统和设备相关的 bindings 文档。
    
- spi： 包含与 SPI 总线和设备相关的 bindings 文档。
    
- usb： 包含与 USB 控制器和设备相关的 bindings 文档。
    
- video： 包含与视频设备和驱动程序相关的 bindings 文档。
    

每个子目录中的文档通常以.txt 或.yaml 的扩展名保存， 使用文本或 YAML 格式编写。 这些文档提供了有关设备树中属性的详细说明、 属性的语法、 可选值和用法示例。 它们还描述了设备树的约定和最佳实践， 以帮助开发人员正确地配置和描述硬件设备和驱动程序。

  

通过阅读 Documentation/devicetree/bindings 目录中的文档， 开发人员可以了解各种设备和驱动程序的设备树属性的含义和用法， 以便正确地配置和描述硬件平台和设备。 这有助于实现硬件与软件之间的正确匹配和交互， 使系统能够正确识别和使用硬件设备。

  

  

看i2c-rk3x.txt