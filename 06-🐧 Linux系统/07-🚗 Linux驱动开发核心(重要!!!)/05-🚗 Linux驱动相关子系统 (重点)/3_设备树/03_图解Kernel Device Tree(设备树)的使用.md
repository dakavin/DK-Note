---
文章标题: "[[03_图解Kernel Device Tree(设备树)的使用]]"
文章作者: Dakkk
文章概要: |
  文章详细图解Linux Kernel设备树的概念、结构和使用方法，包括DTS节点、属性、硬件描述等核心内容，为嵌入式开发提供完整的设备树编程指南。
tags:
  - 设备树
  - Linux内核
  - DTS
  - 嵌入式系统
  - ARM
  - 硬件描述
  - bootloader
  - 驱动开发
相关文章:
  - "[[02_设备树（device Tree）的由来]]"
  - "[[04_设备树中CPU描述 (不需要改)]]"
  - "[[8_Linux设备树]]"
  - "[[1_驱动关键技术]]"
  - "[[05_设备树中时钟描述 (被使用)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/1_Linux之设备树详解（训练营）/02_图解Kernel Device Tree(设备树)的使用.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-10 23:12:25
---

## 1 图解DTS

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3181332722c782211658037da1bdde1f.png)
- Node：节点
- Property：属性

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7a20013c5c588814dbda9c25a360920c.png)
- 节点---函数名
- 属性---语句

> [!abstract]+ 设备树核心概念 
> **设备树(Device Tree)是一种用于描述硬件平台的数据结构**，它使用类似于XML的树状结构来描述硬件设备的信息。
> 
> 简单来说，设备树就像是一份"硬件说明书"，告诉系统有哪些硬件设备以及如何使用它们。

换句话说，设备树改变了传统的硬件描述方式。以前我们需要在内核代码中硬编码(hardcode)硬件配置信息，现在通过设备树，**bootloader可以像传递数据库一样将硬件信息传递给内核**。

对于基于ARM CPU的嵌入式系统，传统做法是为每个硬件平台(platform)单独编译内核。但随着ARM在消费电子、桌面系统、服务器系统中的广泛应用，我们希望能像X86那样**用一个通用的内核镜像支持多个不同的硬件平台**。

> [!info]+ 设备树的设计目标 如果我们把内核看作一个黑盒子，那么它需要以下输入参数：
> 
> 1. **识别platform的信息** - 知道运行在什么硬件平台上
> 2. **runtime的配置参数** - 运行时的各种配置
> 3. **设备的拓扑结构以及特性** - 硬件设备的连接关系和属性
> 
>  补充：内核源码中，有文档对上面三项内容的描述，`Documentation/devicetree/booting-without-of.txt` 
> 

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/35c2304a1383cd1c30a99c903d4c86e6.png)

在嵌入式系统启动过程中，bootloader不仅要加载内核并转交控制权，还需要把上述三个重要参数传递给内核。**设备树就是为了实现这个目标而设计的**。

参考文章：[内核对设备树的处理](1_参考内容/内核对设备树的处理.md)

## 2 设备树包含的硬件信息

### 2.1 哪些硬件需要在设备树中描述？

> [!question]+ 常见问题：
> 设备树是否可以描述所有的硬件信息？ 
> 
> **答案：不可以**。能够动态探测到的设备不需要在设备树中描述。

**不需要描述的设备：**
- **USB设备** - 可以通过USB协议自动识别
- **PCI设备** - 可以通过PCI总线自动探测

**需要描述的设备：**
- **USB主控制器** - 虽然USB设备可以自动识别，但SOC上的USB主控制器无法动态识别
- **PCI桥接器** - 如果PCI桥接器不能被自动探测，就需要在设备树中描述

### 2.2 设备树通常描述的硬件信息

> [!note]+ 设备树描述的主要硬件类型  **需要描述的内容一般包括：**

**1. CPUs**
- 处理器的型号、类型
- 寄存器地址范围
- 中断控制器(GIC等)信息

**2. Memory**
- 系统物理内存的起始地址和大小

**3. Buses总线**
- PCI、I2C、SPI、AMBA、PCIe等总线的节点和连接关系

**4. Peripheral connections外设连接**
- 定时器、看门狗、串口(UART)
- 网络设备(以太网控制器)
- 存储设备控制器(SD卡、eMMC、NAND控制器)
- 显示控制器(LCD、GPU等)
- 音频设备
- USB控制器

**5. Interrupt Controllers中断控制器**
- 中断号、中断触发类型、中断控制器配置

**6. GPIO Controllers GPIO控制器**
- GPIO的编号、复用功能、初始状态

**7. Clock Controllers时钟控制器**
- 各种外设时钟频率、时钟父子关系和时钟使能状态

## 3 设备树图示

>[!example]+ 设备树的重要属性
>最重要的属性包括：`compatible`, `reg`, `clocks`, `interrupts`, `status`

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3181332722c782211658037da1bdde1f.png)

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7a20013c5c588814dbda9c25a360920c.png)

## 4 节点的介绍

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2958db4d11d1c404b0d1184e8e79efbd.png)
### 4.1 根节点

**设备树使用层次结构，根节点(Root Node)是整个设备树的起点**。根节点由斜杠`/`标识，使用大括号`{}`包含所有内容

**一个最简单的根节点示例如下**：
```dts
/dts-v1/; // 设备树版本信息

/ {

}
```
- 第一行的`dts-v1`是版本信息，是可选的。这行用于指定设备树的语法版本，如果不需要可以删除

### 4.2 子节点

子节点是根节点的直接子项，用于描述具体的硬件设备或设备集合。子节点采用特定格式：
![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3b881953de78903b958e5f28926e61e6.png)

> [!info]+ 子节点的组成部分详解

**1. 节点标签(Label) - 可选**
- 节点标签是可选的标识符，用于在设备树中引用该节点
- **作用：** 允许其他节点直接引用此节点，建立引用关系

**2. 节点名称(Node Name) - 必需**
- 用于唯一标识该节点在设备树中的位置
- 通常是硬件设备的名称，**必须在设备树中唯一**

**3. 单元地址(Unit Address) - 可选**
- 用于标识设备的实例，可以是整数、十六进制值或字符串
- **目的：** 区分相同类型设备的不同实例
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1b5b5374f131e818b93d47e675c744e1.png)
	- 名为cpu的节点通过单元地址0和1来区分
	- 名为ethernet的节点通过单元地址fe002000和fe003000来区分

**4. 属性定义(Properties Definitions)**
- **一组键值对，用于描述设备的配置和特性**
- 可以定义寄存器地址、中断号、时钟频率等

**5. 子节点(Child Nodes)**
- 当前节点的子项，用于进一步描述硬件设备的子组件或配置
- 可以包含自己的属性定义和更深层次的子节点，形成设备树的层次结构

### 4.3 别名节点

`aliases`节点定义了一些别名，解决了设备树引用复杂的问题

> [!question]+ 为什么需要别名节点？ 
> 设备树是树状结构，引用一个节点时需要指明相对于根节点的完整路径，例如`/node1/child-node1`。如果多次引用同一个节点，每次都写这么复杂的字符串会很麻烦。
  
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4d3cea0f8be124edbc6e7b38b3b03b39.png)
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e6849de9fe3657707cae46d03e2485a4.png)
> [!tip]+ 别名使用示例如上图 
> 在aliases节点中定义了设备节点完整路径的缩写，这样就可以使用`serial0`来引用`uart0`这个节点，大大简化了引用过程。

### 4.4 CPU节点（必须有）

> [!warning]+ 重要提醒
> 对于根节点，**必须有一个`cpus`的子节点**来描述系统中的CPU信息

```C
cpus {address-cells = <2;size-cells = <0;
                cpu_l0: cpu@0 {
                        device_type = "cpu";
                        compatible = "arm,cortex-a53", "arm,armv8";
                        reg = <0x0 0x0;
                        enable-method = "psci";cooling-cells = <2; /* min followed by max */
                        clocks = <&cru ARMCLKL;
                        dynamic-power-coefficient = <100;};
```
### 4.5 Memory节点（必备）

> [!warning]+ 必备节点 
> **memory设备节点是所有设备树文件的必备节点**，它定义了系统物理内存的布局(layout)。

**关键属性解释：**
- **device_type属性**
    - 定义节点的设备类型，如cpu、serial等
    - 对于memory节点，device_type**必须等于"memory"**
- **reg属性**
    - 定义访问该设备节点的地址信息
    - 值被解析成任意长度的(address, size)数组
    - 具体的address和size长度在父节点中定义(#address-cells和#size-cells)
    - 对于memory节点，定义了该内存的起始地址和长度

### 4.6 可选节点

**chosen节点**主要用来描述由系统firmware指定的运行时参数(runtime parameter)

> [!info]+ chosen节点的要求 
> 如果存在chosen节点，其父节点必须是名字为"/"的根节点。

**chosen节点的作用：**
- 传递原来通过tag list传递的linux kernel运行时参数
- **command line**可以通过`bootargs`属性传递
- **initrd**的开始地址可以通过`linux,initrd-start`属性传递

- 在实际中，建议增加一个`bootargs`的属性，例如：
```Bash
chosen {
	bootargs = "console=ttymxc0,115200";
}; 
```

> [!note]+ chosen节点的特点 
> chosen节点并没有描述任何硬件设备信息，它只是传递了运行时参数。这体现了设备树的三大功能：硬件平台识别、运行时参数传递、硬件设备描述。

## 5 属性

![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d9caf3e6405741b69ef0f7a70cdcc088.png)

### 5.1 Compatible属性

> [!abstract]+ Compatible属性的核心作用 
> **compatible属性是设备树中最重要的属性之一**，用于描述设备的兼容性信息，实现设备节点与驱动程序之间的匹配关系。

简单来说，**compatible属性用于匹配Platform device和platform driver**。

**Compatible属性值的格式：**

1. **单个字符串值**
    - 格式：`"vendor,device"`
    - 用于指定设备节点与特定厂商的特定设备兼容
2. **字符串列表**
    - 格式：`["vendor,device1", "vendor,device2"]`
    - 用于指定设备节点与多个设备兼容
3. **通配符匹配**
    - 格式：`"vendor,*"`
    - 用于指定设备节点与特定厂商的所有设备兼容

> [!example]+ 单个字符串值示例 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a3177201003770edcd63da3135febdfd.png)
> 
> - **vendor：**厂商标识
>     
> - **device：**设备的具体型号或版本
>     
> 
> 在这个示例中，my_device节点的compatible属性值为"vendor,device"，标识设备节点与特定厂商的特定设备兼容。

> [!example]+ 多个匹配值示例 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bb78ed6895abecd717d24f468903d081.png)
> 
> 在这个示例中，my_device节点的compatible属性值为`["vendor,device1", "vendor,device2"]`，表示设备节点与厂商vendor的device1和device2兼容。

> [!tip]+ Compatible属性的工作原理 
> 当设备树被操作系统或设备管理软件解析时，会根据设备节点的compatible属性值来选择适合的驱动程序进行设备的初始化和配置。

> [!example]+ 实际代码匹配过程 
> **1. 设备树文件中的定义：** 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e70ddb095353df127171cd459d48a1f1.png)
> 
> **2. 在kernel/drivers中找到对应的解析：** 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e28a1c843157521ca4b38b7de5480ef9.png)
> 
> **3. 驱动程序中的引用：** 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/714de90ff84695ccbe533978e77d56af.png)
> 
> 这个of_device_id会给到platform_driver去使用，然后在platform_bus的match方法进行比较

> [!success]+ 重要技巧 
> **通过compatible字段，我们可以快速找到对应的驱动代码**，这对于理解和调试硬件驱动非常有用。

### 5.2 model属性

**model属性用于描述设备的型号或者名称**，通常作为设备节点的一个属性，提供关于设备的标识信息

> [!info]+ model属性的特点
> 
> - model属性是**可选的**，但在实际应用中经常被使用
>     
> - 属性值是一个字符串，可以是设备的型号、名称或其他标识符
>     
> - 该值通常由设备的厂商定义
>     

> [!example]+ model属性使用示例 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bf279b58e229d1df27fccf41c899203a.png)
> 
> 在这个示例中，my_device节点的model属性值为"My Device XYZ"，描述了设备的型号或名称。

> [!tip]+ model属性的重要作用 
> model属性通常用于标识和区分不同的设备，**特别是当设备节点的compatible属性相同或相似时**。通过使用不同的model属性值，可以更加准确地确定所使用的设备类型。

### 5.3 reg属性

> [!abstract]+ reg属性的核心功能 
> **reg属性用于在设备树中指定设备的寄存器地址和大小**，提供了与设备树中的物理设备之间的寄存器映射关系。

reg属性有两种常见格式：

#### 5.3.1 单个值格式

```Bash
reg = <address size>;
```
这种格式适用于描述单个寄存器的情况：
- **address：** 设备的起始寄存器地址，可以是整数或十六进制值
- **size：** 寄存器的大小，即占用的字节数

**示例：**
```shell
my_device{
	compatible = "vendor,device";
	reg = <0x0001 0x4>;
	// 其他属性和子节点的定义
}
```
- 这个示例中， `my_device` 设备节点的 `reg` 属性值为 `<0x1000 0x4>`， 表示从地址`0x1000` 开始的 4 字节寄存器区域

#### 5.3.2 列表值格式

```Bash
reg = <address size address1 size1 address2 size2 ...>;
```
- 当设备具有多个寄存器区域时，可以使用列表值格式来描述每个寄存器区域的地址和大小

**示例**：
```shell
my_device{
	compatible = "vendor,device";
	reg = <0x0001 0x8 0x2000 0x4>;
	// 其他属性和子节点的定义
}
```
在这个示例中， my_device 设备节点有两个寄存器区域
- 第一个寄存器区域从地址 0x1000 开始， 大小为 8 字节； 
- 第二个寄存器区域从地址 0x2000 开始， 大小为 4 字节。

> [!note]+ reg属性的重要性
> 通过使用reg属性，设备树可以提供有关设备寄存器布局和寄存器访问方式的信息。这对于操作系统的设备驱动程序很重要，因为它们需要了解设备的寄存器映射以正确地与设备进行交互和配置。

### 5.4 寻址属性

**#address-cells和#size-cells属性用于指定设备树中地址单元和大小单元的位数**。它们提供了设备树解析所需的元数据，以正确解释设备的地址和大小信息

#### 5.4.1 `#address-cells`属性

> [!info]+ `#address-cells`属性说明 
> `**#address-cells`属性**是一个位于设备树根节点的特殊属性，它**指定了设备树中地址单元的位数**

**关键信息：**
- 地址单元是设备树中用于表示设备地址的单个单位
- 通常是一个整数，可以是十进制或十六进制值
- 告诉解析软件在解释设备地址时应该使用多少位来表示一个地址单元
- **默认值为2**，表示使用两个单元来表示一个设备地址

#### 5.4.2 `#size-cells`属性

> [!info]+ `#size-cells`属性说明 
> **`#size-cells`属性**也是一个位于设备树根节点的特殊属性，它**指定了设备树中大小单元的位数**。

**关键信息：**
- 大小单元是设备树中用于表示设备大小的单个单位
- 通常是一个整数，可以是十进制或十六进制值
- 告诉解析软件在解释设备大小时应该使用多少位来表示一个大小单元
- **默认值为1**，表示使用一个单元来表示一个设备的大小

#### 5.4.3 案例

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b009bd765ab0e3e9b46bf9a3fb7a1274.png)
**在root node节点中：**
- `#address-cells`设置为`2`
- `#size-cells`设置为`2`
- **结果：** spi0这个子节点的reg有两个address和两个size

**在spi节点中：**
- `#address-cells`设置为`1`
- `#size-cells`设置为`0`
- **结果：**device0这个子节点的reg只有一个address

再举例，查看`u-boot/arch/arm/dts/rk3568.dtsi`，可以发现cpu这个子节点如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ca0b669ffd363ae4b0a0a03b30ada631.png)

> [!tip]+ 寻址属性的意义 
> 这两个属性的存在是为了使设备树能够灵活地描述各种设备的地址和大小表示方式。
> 
> 通过在设备树的根节点中设置适当的#address-cells和#size-cells值，设备树解析软件能够正确地解释设备节点中的地址和大小信息。

### 5.5 节点状态

**status属性用于描述设备或节点的状态**，是设备树中常见的属性之一，用于表示设备或节点的可用性或操作状态。

> [!info]+ status属性的可选值 status属性的值可以是以下几种：
> 
> - **"okay"：**表示设备或节点正常工作，可用
>     
> - **"disabled"：**表示设备或节点被禁用，不可用
>     
> - **"reserved"：**表示设备或节点已被保留，暂时不可用
>     
> - **"fail"：**表示设备或节点初始化或操作失败，不可用
>     

> [!example]+ status属性使用示例 
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8ff66fbb738773d30313fdf583d4633f.png)
> 
> 在这个示例中，my_device节点的status属性值为"okay"，表示设备处于正常工作状态，可用。

> [!tip]+ status属性的重要作用 
> 通过使用status属性，设备树可以动态地控制设备的启用和禁用状态。这对于以下场景非常有用：
> 
> - 在系统启动过程中选择性地启用或禁用设备
>     
> - 在运行时根据特定条件调整设备状态
>     
> - 进行硬件调试和测试
>     

### 5.6 其他属性

> [!note]+ 自定义属性的处理 
> 除了上述标准属性外，设备树还可以包含其他自定义属性。**这些自定义属性是由其对应的驱动代码独立解析的**。

换句话说，每个设备驱动程序可以定义自己特有的属性，并在驱动代码中解析这些属性来获取设备的特定配置信息。这种灵活性使得设备树能够适应各种不同的硬件设备需求。

---

> [!success]+ 总结 
> 设备树是现代嵌入式系统中描述硬件的重要工具。通过理解设备树的结构、节点类型和各种属性，我们可以更好地进行硬件开发和调试工作。
> 
> **掌握设备树的关键在于理解compatible属性的匹配机制、reg属性的地址映射、以及各种寻址属性的配置方法**。
