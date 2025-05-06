
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3181332722c782211658037da1bdde1f.png)
- Node：节点
- Property：属性

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7a20013c5c588814dbda9c25a360920c.png)
- 节点---函数名
- 属性---语句

本质上，`Device Tree`改变了原来用`hardcode`方式将`HW` 配置信息嵌入到内核代码的方法，改用`bootloader`传递一个`DB`的形式

对于基于`ARM CPU`的嵌入式系统，我们习惯于针对每一个`platform`进行内核的编译。但是随着`ARM`在消费类电子上的广泛应用（甚至桌面系统、服务器系统），我们期望`ARM`能够象`X86`那样用一个`kernel image`来支持多个`platform`。在这种情况下，如果我们认为`kernel`是一个`black box`，那么其输入参数应该包括：
- 识别`platform`的信息
- `runtime`的配置参数
- 设备的拓扑结构以及特性

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/35c2304a1383cd1c30a99c903d4c86e6.png)

对于嵌入式系统，在系统启动阶段，`bootloader`会加载内核并将控制权转交给内核，此外，还需要把上述的三个参数信息传递给`kernel`如上图，以便`kernel`可以有较大的灵活性。在`linux kernel`中，`Device Tree`的设计目标就是如此。

## 1 设备树包含的硬件信息

**设备树是否可以描述所有的硬件信息？**
- 不行，因为可以动态探测到的设备是不需要被描述的，例如 USB device，不过对于 SOC 上的 usb host controller，它是无法动态识别的，需要在设备树中描述
- 同样的道理，在computer system中，PCI device 可以被探测到，不需要在设备树中描述，但是 PCI brideg如果不能被探测，就需要描述

**需要描述的内容一般包括：**
- `CPUs`
	- 包括处理器的型号、类型、寄存器地址范围、中断控制器（GIC等）信息等
- `Memory`
	- 描述系统的物理内存起始地址和大小
- `Buses`
	- 如PCI、I2C、SPI、AMBA、PCie等总线的节点和连接关系
- `Peripheral connections`
	- 定时器、看门狗、串口（UART）
	- 网络设备（以太网控制器）
	- 存储设备控制器（SD卡、eMMC、NAND控制器）
	- 显示控制器（LCD、GPU等）
	- 音频设备
	- USB控制器
- `Interrupt Controllers`
	- 中断号、中断触发类型、中断控制器配置
- `GPIO Controllers`
	- GPIO的编号、复用功能、初始状态
- `Clock Controllers`
	- 各种外设时钟频率、时钟父子关系和时钟使能状态
## 2 设备树示例（线头）

为了了解`Device Tree`的结构，我们首先给出一个`Device Tree`的示例： 最重要的属性 `compatible`, `reg`, `clocks`, `interrupts`, and `status`

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3181332722c782211658037da1bdde1f.png)

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7a20013c5c588814dbda9c25a360920c.png)
## 3 节点的介绍

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2958db4d11d1c404b0d1184e8e79efbd.png)
### 3.1 根节点

设备树使用一种层次结构，其中的根节点（Root Node）是整个设备树的起点（根节点）。
根节点由一个以 `/` 开头的标识符来表示，然后使用 `{}` 来包含根节点所包括的内容，一个最简单的根节点示例如下：
```dts
/dts-v1/; // 设备树版本信息

/ {

}
```

其中第一行的设备树中的版本信息行 dts-v1 是可选的， 可以根据需要选择是否保留。 这行注释通常用于指定设备树的语法版本。 如果您不需要在设备树中指定版本信息， 可以将其删除。
### 3.2 子节点

设备树中的子节点是根节点的直接子项， 用于描述具体的硬件设备或设备集合。 子节点采用以下特定的格式来表示:
![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3b881953de78903b958e5f28926e61e6.png)

以下是对这些部分的详细介绍：
**（1） 节点标签（Label） （可选）**
- 节点标签是一个`可选的`标识符， 用于在设备树中引用该节点。 标签允许其他节点直接引用此节点， 以便在设备树中建立引用关系

**（2） 节点名称（Node Name）**
- 节点名称是一个字符串， 用于唯一标识该节点在设备树中的位置。 节点名称通常是硬件设备的名称， 但必须在设备树中是`唯一的`。

**（3） 单元地址（Unit Address） （ 可选）**
- 单元地址用于标识设备的实例。 它可以是一个整数、 一个十六进制值或一个字符串， 具体取决于设备的要求。
- `单元地址的目的是区分相同类型的设备的不同实例`， 例如在下图中名为 cpu 的节点通过它们的单元地址值 0 和 1 来区分， 名称为 ethernet 的节点通过其单元地址值 fe002000 和 fe003000 来区分。
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1b5b5374f131e818b93d47e675c744e1.png)
**（4） 属性定义（ Properties Definitions）**
- 属性定义是`一组键值对， 用于描述设备的配置和特性`。 属性可以根据设备的需求进行定义， 例如寄存器地址、 中断号、 时钟频率等， 关于这些属性会在后面的小节中进行讲解

**（5） 子节点（ Child Nodes）**
- 子节点是`当前节点的子项`， 用于`进一步描述硬件设备的子组件或配置`。 子节点可以包含自己的属性定义和更深层次的子节点， 形成设备树的层次结构。

### 3.3 别名节点

`aliases` 节点定义了一些别名。为何要定义这个`node`呢？
- 因为`Device tree`是树状结构，当要引用一个`node`的时候要指明相对于`root node`的`full path`，例如`/node1/child-node1`
- 如果多次引用，每次都要写这么复杂的字符串多少是有些麻烦，因此可以在`aliases`节点定义一些设备节点`full path`的缩写
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4d3cea0f8be124edbc6e7b38b3b03b39.png)
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e6849de9fe3657707cae46d03e2485a4.png)
- 也就是使用serial0来引用uart0这个节点
### 3.4 CPU节点（必须有）

对于根节点，必须有一个`cpus`的`child node`来描述系统中的`CPU`信息
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
### 3.5 Memory节点（必备）

`memory device node`是所有设备树文件的必备节点，它定义了系统物理内存的`layout`

`device_type`属性定义了该`node`的设备类型，例如`cpu`、`serial`等。对于`memory node`，其`device_type`必须等于`memory`

`reg`属性定义了访问该`device node`的地址信息，该属性的值被解析成任意长度的（`address`，`size`）数组，具体用多长的数据来表示`address`和`size`是在其`parent node`中定义（`#address-cells`和`#size-cells`）

对于`device node`，`reg`描述了`memory-mapped IO register`的`offset`和`length`。对于`memory node`，定义了该`memory`的起始地址和长度
### 3.6 可选节点

`chosen node`主要用来描述由系统`firmware`指定的`runtime parameter`

如果存在`chosen`这个`node`，其`parent node`必须是名字是`“/”`的根节点

原来通过`tag list`传递的一些`linux kernel`的运行时参数可以通过`Device Tree`传递，例如：
- `command line`可以通过`bootargs`这个`property`这个属性传递
- `initrd`的开始地址也可以通过`linux,initrd-start`这个`property`这个属性传递
- 在实际中，建议增加一个`bootargs`的属性，例如：
```Bash
chosen {        bootargs = "console=ttymxc0,115200";    }; 
```

我们知道，`device tree`用于`HW platform`识别，`runtime parameter`传递以及硬件设备描述。`chosen`节点并没有描述任何硬件设备节点的信息，它只是传递了`runtime parameter`。
## 4 属性

![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d9caf3e6405741b69ef0f7a70cdcc088.png)

### 4.1 Compatible属性

在设备树中， **compatible** 属性用于描述设备的兼容性信息。 它是设备树中重要的属性之一，用于识别设备节点与驱动程序之间的匹配关系。
- `用于匹配Platform device和platform driver的`

**compatible** 属性的值是一个字符串或字符串列表， 用于指定设备节点与相应的驱动程序或设备描述符兼容的规则。 通常， compatible 属性的值由设备的厂商定义， 并且在设备树中使用。

以下是一些常见的 **compatible** 属性值的示例：
- 单个字符串值： 例如 `"vendor,device"`， 用于指定设备节点与特定厂商的特定设备兼容
- 字符串列表： 例如 `["vendor,device1", "vendor,device2"]`， 用于指定设备节点与多个设备兼容， 通常用于设备节点具有多种变体或配置
- 通配符匹配： 例如 `"vendor,*"`， 用于指定设备节点与特定厂商的所有设备兼容， 不考虑具体的设备标识

以下是一个示例， 展示了如何在设备树中使用 compatible 属性：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a3177201003770edcd63da3135febdfd.png)
- vendor：厂商标识
- device：设备的具体型号或版本表示
- 在这个示例中， my_device 节点具有 compatible 属性， 其值为 "vendor,device"。 这个值用于标识设备节点与特定厂商的特定设备兼容

compatible 属性也可以具有多个匹配值， 用于指定设备节点与多个设备或驱动程序的兼容性规则。 这种情况下， compatible 属性的值是一个字符串列表， 每个字符串表示一个匹配值。

以下是一个示例， 展示了具有多个匹配值的 compatible 属性的用法：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/bb78ed6895abecd717d24f468903d081.png)
- 在 这 个 示 例 中 ， my_device 节 点 具 有 compatible 属 性 ， 其 值 为` ["vendor,device1", "vendor,device2"]`。 这表示设备节点与厂商vendor的 device1 和 device2 兼容。

通过使用 compatible 属性， 设备树可以提供设备和驱动程序之间的匹配信息

**当设备树被操作系统或设备管理软件解析时， 会根据设备节点的 compatible 属性值来选择适合的驱动程序进行设备的初始化和配置**

**案例：**
- 对于dts文件如下，
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e70ddb095353df127171cd459d48a1f1.png)
- 我们可以在kernel/drivers中找到对应的解析字段
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e28a1c843157521ca4b38b7de5480ef9.png)
- 可以看到上述c文件有引用
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/714de90ff84695ccbe533978e77d56af.png)
- 这个of_device_id 会给到device driver去使用

**所以，我们可以通过compatible这个字段来找到其对应的驱动代码**
### 4.2 model属性

在设备树中， model 属性用于描述设备的型号或者名称。 它通常作为设备节点的一个属性，用来提供关于设备的标识信息。 model 属性是可选的， 但在实际应用中经常被使用。

model 属性的值是一个字符串， 可以是设备的型号、 名称、 或者其他标识符， 用来识别设备。 该值通常由设备的厂商定义， 并且在设备树中使用。

以下是一个示例， 展示了如何在设备树中使用 model 属性：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/bf279b58e229d1df27fccf41c899203a.png)

在这个示例中， my_device 节点具有 model 属性， 其值为 "My Device XYZ"。 这个值描述了设备的型号或名称为 "My Device XYZ"。

model 属性通常用于标识和区分不同的设备， **特别是当设备节点的 compatible 属性相同或相似时。 通过使用不同的 model 属性值， 可以更加准确地确定所使用的设备类型**。
### 4.3 reg属性

`reg 属性用于在设备树中指定设备的寄存器地址和大小`， 提供了与设备树中的物理设备之间的寄存器映射关系

reg 属性可以在设备节点中有单个值格式和列表值格式这两种常见格式， 接下来将对这两种格式进行介绍：
#### 4.3.1 单个值格式

```Bash
reg = <address size>;
```

这种格式适用于描述单个寄存器的情况。 其中， address 是设备的起始寄存器地址， 可以是一个整数或十六进制值。 size 表示寄存器的大小， 即占用的字节数。

例如， 假设有一个设备节点 my_device， 使用单个值格式的 reg 属性来描述一个 4 字节寄存器的地址和大小， 可以这样定义：
```shell
my_device{
	compatible = "vendor,device";
	reg = <0x0001 0x4>;
	// 其他属性和子节点的定义
}
```
这个示例中， `my_device` 设备节点的 `reg` 属性值为 `<0x1000 0x4>`， 表示从地址`0x1000` 开始的 4 字节寄存器区域。
#### 4.3.2 列表值格式

```Bash
reg = <address size address1 size1 address2 size2 ...>;
```

当设备具有多个寄存器区域时， 可以使用列表值格式的 reg 属性来描述每个寄存器区域的地址和大小。 通过这种方式， 可以指定多个寄存器的位置和大小， 以描述设备的完整寄存器映射。

例如， 考虑一个设备节点 my_device， 它具有两个寄存器区域， 分别是 8 字节和 4 字节大小的寄存器。 可以使用列表值格式的 reg 属性来描述这种情况：
```shell
my_device{
	compatible = "vendor,device";
	reg = <0x0001 0x8 0x2000 0x4>;
	// 其他属性和子节点的定义
}
```

在这个示例中， my_device 设备节点的 reg 属性值为 <0x1000 0x8 0x2000 0x4>， 表示设备有两个寄存器区域。 第一个寄存器区域从地址 0x1000 开始， 大小为 8 字节； 第二个寄存器区域从地址 0x2000 开始， 大小为 4 字节。

通过使用 reg 属性， 设备树可以提供有关设备寄存器布局和寄存器访问方式的信息。 这对于操作系统的设备驱动程序很重要， 因为它们需要了解设备的寄存器映射以正确地与设备进行交互和配置。
### 4.4 寻址属性

`#address-cells` 和 `#size-cells` 属性用于指定在上个小节中要设置的设备树中地址单元和大小单元的位数。 它们提供了设备树解析所需的元数据， 以正确解释设备的地址和大小信息。 下面对两个属性分别进行介绍：
#### 4.4.1 address-cells属性

**#address-cells** 属性是一个位于设备树根节点的特殊属性， 它`指定了设备树中地址单元的位数`。 地址单元是设备树中用于表示设备地址的单个单位。 它通常是一个整数， 可以是十进制或十六进制值。

**#address-cells** 属性的值告诉解析设备树的软件在解释设备地址时应该使用多少位来表示一个地址单元。

默认情况下， **#address-cells** 的值为 2， 表示使用两个单元来表示一个设备地址。 这意味着设备的地址将由两个整数（每个整数使用指定位数的位） 组成。

例如， 对于一个使用两个 32 位（4 字节） 整数表示地址的设备， 可以在设备树的根节点中设置 **#address-cells** 属性为 <2>。
#### 4.4.2 size-cells属性

**#size-cells** 属性也是一个位于设备树根节点的特殊属性， 它`指定了设备树中大小单元的位数`。 大小单元是设备树中用于表示设备大小的单个单位。 它通常是一个整数， 可以是十进制或十六进制值。

**#size-cells** 属性的值告诉解析设备树的软件在解释设备大小时应该使用多少位来表示一个大小单元。

默认情况下， **#size-cells** 的值为 1， 表示使用一个单元来表示一个设备的大小。 这意味着设备的大小将由一个整数（使用指定位数的位） 表示。

例如， 对于一个使用一个 32 位（4 字节） 整数表示大小的设备， 可以在设备树的根节点中设置 **#size-cells** 属性为 <1>。

这两个属性的存在是为了使设备树能够灵活地描述各种设备的地址和大小表示方式。 通过在设备树的根节点中设置适当的 **#address-cells** 和 **#size-cells** 值， 设备树解析软件能够正确地解释设备节点中的地址和大小信息。
#### 4.4.3 案例

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b009bd765ab0e3e9b46bf9a3fb7a1274.png)

- 在 `root nnode` 节点中，`#address-cells` 设置为 `2`，`#size-cells` 设置为 `2`
	- 所以，spi0这个子节点的reg，有两个address和两个size
- 在 `spi` 节点中，`#address-cells` 设置为 `1`，`#size-cells` 设置为 `0`
	- 所在，在device0这个子节点的reg，只有一个address

再举例，查看`u-boot/arch/arm/dts/rk3568.dtsi`，可以发现cpu这个子节点如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ca0b669ffd363ae4b0a0a03b30ada631.png)

### 4.5 节点状态

在设备树中， status 属性用于描述设备或节点的状态。 它是设备树中常见的属性之一， 用于表示设备或节点的可用性或操作状态。
- status 属性的值可以是以下几种：
- "okay"： 表示设备或节点正常工作， 可用。
- "disabled"： 表示设备或节点被禁用， 不可用。
- "reserved"： 表示设备或节点已被保留， 暂时不可用。
- "fail"： 表示设备或节点初始化或操作失败， 不可用。

以下是一个示例， 展示了如何在设备树中使用 status 属性：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8ff66fbb738773d30313fdf583d4633f.png)

在这个示例中， my_device 节点具有 status 属性， 其值为 "okay"。 这表示设备处于正常工作状态， 可用。

通过使用 status 属性， 设备树可以动态地控制设备的启用和禁用状态。 这对于在系统启动过程中选择性地启用或禁用设备， 或者在运行时根据特定条件调整设备状态非常有用。
### 4.6 其他属性

其他的自定义属性，是由其对应的驱动代码独立去解析的

