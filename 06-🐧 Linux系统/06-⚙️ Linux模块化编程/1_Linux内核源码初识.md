  

我们来认识 Linux 内核源码， 开始真正进入到驱动的世界里面， 不知道各位小伙伴们有没有做好准备呢？

## 初识内核源码

Linux 内核源码的官方网站为 https://www.kernel.org/， 可以在该网站下载最新的 Linux 内核源码。 进入该网站之后如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/02db3764f32ab4c1751171e4acae83c0.png)


从上图可以看到多个版本的内核分支， 分别为主线版本（mainline） 、 稳定版本（stable） 和长期支持版本（longterm） 。 以上各个支线和主线是由 linus torvalds（Linux 之父）所领导。 半导体厂商和一些内核爱好者会在官网下载相应版本的内核源码， 对该源码进行打补丁等操作。 以此让官网的内核源码可以在半导体厂家设计的主控（CPU） 上跑起来， 所以在开发和学习的过程中， 我们并不会直接去 Linux 内核官网下去下载源码， 而且是使用半导体厂家提供的源码包。

但是不论是 Linux 官网的内核源码还是半导体厂家提供的内核源码不影响我们来看它的庐山真面目！

## 内核源码结构

录的内容如下表所示：

|   |   |
|---|---|
|arch|存放不同平台体系相关代码|
|block|存放块设备相关代码|
|crypto|存放加密、 压缩、 CRC 校验等算法相关代码|
|Documentation|存放相关说明文档， 很多实用文档， 包括驱动编写等|
|drivers|存放 Linux 内核设备驱动程序源码。 该目录包含众多驱动， 目录按照设备类别进行分类， 如 char、 block 、 input、 i2c、 spi、 pci、 usb 等。|
|firmware|存放处理器相关的一些特殊固件|
|fs|存放虚拟文件系统代码|
|include|存放内核所需、 与平台无关的头文件|
|init|Linux 系统启动初始化相关的代码|
|ipc|存放进程间通信代码|
|kernel|Linux 内核的核心代码， 包含了进程调度子系统， 以及和进程调度相关的模块。|
|lib|库文件代码， 实现需要在内核中使用的库函数， 例如 CRC、 FIFO、 list、 MD5等。|
|mm|实现存放内存管理代码|
|net|存放网络相关代码|
|samples|存放提供的一些内核编程范例|
|scripts|存放一些脚本文件|
|security|存放系统安全性相关代码|
|sound|存放声音、 声卡相关驱动|
|tools|一些常用工具， 如性能剖析、 自测试等|
|usr|用于生成 initramfs 的代码。|
|virt|提供虚拟机技术（KVM 等） 的支持|

  

## 内核源码结构

1. 使用以下命令对内核源码的进行解压， 解压完成：
    

```C
tar -vxf linux_sdk.tar.gz  
```

2. 使用“cd linux_sdk” 命令进入内核源码目录
    
3. 使用命令“./build.sh kernel” 进行内核源码的编译