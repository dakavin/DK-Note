
设备树（Device Tree） 是一种硬件描述机制， 用于在嵌入式系统和操作系统中描述硬件设备的特性、 连接关系和配置信息。 它提供了一种与平台无关的方式来描述硬件， 使得内核与硬件之间的耦合度降低， 提高了系统的可移植性和可维护性。

**基础/背景知识**
- 我们使用 `platform_device 结构体`来对硬件设备进行描述， 这是一种传统的平台总线设备描述方式
- 每个 `platform_device 结构` 表示一个特定的硬件设备， 并通过注册到平台总线上来使得内核能够与该设备进行通信和交互
- 该结构包含设备的名称、 资源（如内存地址、 中断号等） 、 设备驱动程序等信息



## 1 设备树的由来

随着时间的推移， Linux 内核中的 ARM 部分存在着大量的平台相关配置代码， 这些代码通常是杂乱而重复的， 导致了维护的困难和工作量的增加。

在 2011 年 3 月 17 日， Linux 的创始人 Linus Torvalds 在 ARM Linux 邮件列表中发表了一封帖子， 他表达了对 ARM 架构配置方式的不满， 并宣称"Gaah. Guys, this whole ARM thing is a f* c king pain in the ass"。这引起了广泛的讨论和反思。 ARM 社区中的开发者们开始认识到， 传统的平台相关配置方式已经变得不可持续， 需要一种更加先进和可扩展的方法来解决这个问题。

下图是`Linus`认为的垃圾代码（不可复用）目录：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/a0f9dbcdc1b6a153d30dde315a407980.png)
为了应对这一挑战， ARM 社区开始探索新的硬件描述机制， 并逐渐形成了设备树的概念

- 设备树提供了一种更加灵活和可移植的描述硬件的机制， 将设备的描述信息转移到设备树中
- **设备树使用一种结构化的数据格式**， 通过描述**设备节点、 属性和连接关系等信息**， 使得硬件的描述与具体的平台无关， 同时允许多个平台共享相同的设备树描述
- 设备树的引入为 ARM 架构上的 Linux 内核带来了革命性的变化。它提供了一种统一的硬件描述方式， 使得不同芯片和板级的支持更加简单和灵活
- 此外， 设备树还提供了硬件配置的可视化和可读性， 方便开发者理解和调试硬件

随着时间的推移， 设备树逐渐成为了嵌入式系统和 Linux 内核中描述硬件的标准方式。 它不仅在 ARM 架构上得到了广泛应用， 也被扩展到其他架构和平台上。

## 2 设备树的组成

### 2.1 组成

当描述设备树（Device Tree）时， 通常会涉及到以下几个关键术语： **DTS**、 **DTSI**、 **DTB** 和 **DTC**。

**DTS**
- DTS是设备树的源文件，采用一种类似于文本的语法来描述硬件设备的结构、属性和连接关系。
- DTS文件以 `.dts` 为拓展名，通常由开发人员编写，可读的，主要用于描述设备树的层次结构和属性信息

**DTSI**
- DTSI文件是设备树源文件的包含文件。它 **拓展** 了DTS文件的功能，用于定义可复用的设备树片段
- DTSI文件以 `.dtsi` 为拓展名，可以在多个DTS文件中包含和共享。通过使用DTSI，可以提高设备树的可复用性和可维护性
- **作用类似于C语言中的头文件**

**DTC（类似GCC）**
- DTC 是设备树的编译器。它是一个命令行工具，用于将DTS和DTSI文件编译成DTB文件
- DTC将文本格式的设备树源代码转换为二进制的设备树表现形式，便于系统能够加载和解析。DTC是设备树开发中一个重要的工具

**DTB**
- DTB 是设备树的二进制表现形式。DTB文件是通过将DTS或DTSI文件编译而成的二进制文件
- 以 `.dtb` 为扩展名，DTB文件包含了设备树的结构、属性和连接信息，被操作系统加载和解析
- 在运行时，操作系统使用DTB文件来动态识别和管理硬件设备

### 2.2 关系

**DTS**、 **DTSI**、 **DTB** 和 **DTC** 之间的关系：
1. 开发人员使用文本编辑器编写 DTS 和 DTSI 文件， 描述硬件设备的层次结构、 属性和连接关系。
2. DTSI 文件可以在多个 DTS 文件中包含和共享， 以提高设备树的可重用性和可维护性。
3. 使用 DTC 编译器， 开发人员将 DTS 和 DTSI 文件编译成二进制的 DTB 文件
4. 操作系统在启动过程中加载和解析 DTB 文件， 以识别和管理硬件设备。

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/bc23e66bf50cf6363c16fc905649120d.png)

**板卡所使用的dtb文件**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/f00aaafd3febf2549b3bbcbf47b65b66.png)

SDK中的dtb文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4c30ea2b0a58accc4d85e4b1dbd4c378.png)

SDK中的dts文件
- 路径：`kernel/arch/arm64/boot/dts/rockchip/rk3568-lubancat-2-v2.dts`
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4aa4848f04c213eab71c57160e295328.png)
通过ag来检索，包含dtsi的文件有哪些
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/deee7b591985a2f00b5cbe0d8bdce9ac.png)

## 3 设备树文件

**ARM体系结构：** 
- ARM 体系结构下的设备树源文件通常存放在 `arch/arm/boot/dts/` 目录中
- 该目录是设备树源文件的根目录

**ARM64体系结构：**
- ARM64 体系结构下的设备树源文件通常存放在 `arch/arm64/boot/dts/`目录 及其子目录中
- 该目录也是设备树源文件的根目录， 并包含了针对不同 ARM64 平台和设备的子目录

**子目录结构**
- 在 ARM64 的子目录中， 同样会按照硬件平台、 设备类型或制造商进行组织和分类
- 这些子目录的命名可能与特定芯片厂商（如 Qualcomm、 NVIDIA、 Samsung） 有关，由于我们本手册使用的 soc 是瑞芯微的 rk3568 ， 所 以匹配的设备树目录`arch/arm64/boot/dts/rockchip`
- 每个子目录中可能包含多个设备树文件， 用于描述不同的硬件配置和设备类型

**SDK中包含的硬件配置和设备类型**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/e7c89a5c7243128c18ed2812505e1eb1.png)
## 4 设备树的编译

设备树的编译是将设备树源文件（如上述的.dts 文件）转换为二进制的设备树表示形式（.dtb文件） 的过程。 编译器通常被称为 DTC（Device Tree Compiler）

在 Linux 内核源码中， DTC（ Device Tree Compiler） 的源代码和相关工具通常存放在scripts/dtc/目录中， 如下图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/01886b82d717b015ba09de8d58bcdf84.png)

在编译完源码之后 dtc 设备树编译器会默认生成， 如果没有生成相应的 dtc 可执行文件，可以查看在内核默认配置文件中 CONFIG_DTC 是否使能
### 4.1 编译

在 Linux 环境中， 可以使用以下命令将设备树源文件编译为二进制设备树文件：
```shell
dtc -I dts -O dtb -o output.dtb input.dts
```

其中， `input.dts`是输入的设备树源文件， `output.dtb`是编译后的二进制设备树文件。编译器会验证设备树源文件的语法和语义， 生成与硬件描述相对应的设备树表示形式。
### 4.2 反编译

设备树的反编译是将二进制设备树文件转换回设备树源文件的过程， 以便进行查看、 编辑或修改。 反编译器通常也是 DTC。

在 Linux 环境中， 可以使用以下命令将二进制设备树文件反编译为设备树源文件：
```shell
dtc -I dtb -O dts -o output.dts input.dtb
```
其中， input.dtb 是输入的二进制设备树文件， output.dts 是反编译后的设备树源文件

反编译器会将二进制设备树文件解析并还原为文本形式的设备树源文件， 使其可读性更好

### 4.3 实验

下面来进行一下实际的设备树编译和反编译的演示， 首先创建一个名为 test.dts 的设备树文件， 文件内容如下所示：
```dts
/dts-v1/;
/
{

};
```

这个设备树很简单， 只包含了根节点/， 而根节点中没有任何子节点或属性。 这个示例并没有描述任何具体的硬件设备或连接关系， 它只是一个最基本的设备树框架。
