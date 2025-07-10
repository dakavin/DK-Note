---
文章标题: "[[8_Linux设备树]]"
文章作者: Dakkk
文章概要: |
  文章深入讲解Linux设备树的起源、核心概念与DTS语法结构，并详细阐述节点和属性。它重点介绍了如何使用`of_`函数在内核中获取设备信息，并通过修改设备树、编译、部署及编写平台驱动的实践，完整演示了设备树在嵌入式开发中的应用流程。
tags:
  - Linux
  - 设备树
  - DTS
  - 嵌入式系统
  - 内核驱动
  - 硬件描述
  - OF函数
  - 平台驱动
相关文章:
  - "[[../../08-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/02_设备树（device Tree）的由来]]"
  - "[[../../08-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/03_图解Kernel Device Tree(设备树)的使用]]"
  - "[[10_设备树的简介]]"
  - "[[01_Linux系统构成简单介绍]]"
  - "[[../../08-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/04_设备树中CPU描述 (不需要改)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-05-06 16:55:39
修改时间: 2025-06-10 15:22:41
---

## 1 前言

**问题的由来**：早期Linux系统中，硬件信息是直接写死在内核代码里的。每种不同的硬件平台（比如不同的开发板、不同的处理器），都需要在内核源码中创建专门的文件来描述它们的硬件细节，这些硬件描述文件主要存放在：
- `/arch/arm/plat-xxx/`：平台相关代码
- `/arch/arm/mach-xxx`：机器相关代码

**遇到的问题**：随着ARM处理器种类越来越多，各种开发板层出不穷，内核中描述硬件的文件也越来越多。这就导致了一个问题！！！**Linux内核变得非常臃肿！！！**

**解决方案：设备树**
- 什么是设备树？用一种标准化的格式来描述硬件信息，而不是把这些信息硬编码在内核里
- 设备树的优点
	- 简单：语法清晰，容易理解
	- 易用：修改硬件配置不需要重新编译内核
	- 可重用性强：同一个内核可以通过不同的设备树文件支持不同的硬件平台

**版本变化**：
- Linux 3.x之前：硬件信息写在内核代码里
- Linux 3.x之后：使用设备树描述硬件信息

详细的设备树规范和使用方法，请参考官方文档：[https://www.devicetree.org/](https://www.devicetree.org/)

## 2 设备树简介

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/25f4e4b6dfb7ee05583472d8d60434c5.png)

**设备树的定义**
- 用来告诉操作系统“这个硬件平台上有什么设备”的一个`描述文件`
- 专门描述那些系统无法自动发现的硬件设备，bootloader(uboot)会把设备树传递给内核，内核从中获取硬件信息

> [!tip]+ 提示
> 可以动态获取的硬件信息就不需要设备树了

**设备树描述硬件资源的两个特点**
**1、树状结构描述硬件**
- 本地总线 = 树的“主干” = 根节点
- I2C总线、SPI总线、UART总线 = 树的“枝干”=根节点的子节点
- I2C设备、SPI设备 = 更小的“枝干” = 子节点的子节点
- 除了根节点，每个节点都只有一个父节点

**2、文件可以相互引用**
- 像C语言的头文件，一个设备树文件可以引用另外一个设备树文件
- 多个硬件平台用同一个芯片时，可以把芯片的通用部分写成`.dtsi`文件
- 其他板子直接用`#include xxx.dtsi`引用，实现代码复用

**三个重要概念**：
- **DTS**：.dts格式的源文件，用ASCII文本写的设备树源码，一个.dts文件对应一个硬件平台
    - ARM64架构的源文件在：`/arch/arm64/boot/dts`目录下
- **DTC**：编译设备树的工具，需要手动安装
- **DTB**：编译后生成的二进制文件，就像C语言的.c文件编译生成.bin文件一样

## 3 设备树框架

设备树（Device Tree）就像硬件的“说明书”，它告诉Linux内核系统上有哪些硬件设备，这些设备在哪里，怎么配置。简单来说，设备树是由许多节点（node）和属性（property）组成的树状结构。

以Lubancat2开发板为例，看看设备树的样子，打开者内核源码中的 `/arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dtsi` 文件来了解

**设备树 (内核源码/arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dtsi)**
- 按道理应该是`arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dts`这个文件的，不过这个文件依赖于rk3568-lubancat-2.dtsi，所以就说dtsi吧
```dts fold file:rk3568-lubancat-2.dtsi
/dts-v1/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pwm/pwm.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/input/rk-input.h>
#include <dt-bindings/display/drm_mipi_dsi.h>
#include <dt-bindings/sensor-dev.h>
#include "rk3568.dtsi"

/ {
    model = "EmbedFire LubanCat2 HDMI";
    compatible = "embedfire,lubancat2", "rockchip,rk3568";

    chosen: chosen {
        bootargs = "earlycon=uart8250,mmio32,0xfe660000 console=ttyFIQ0 root=PARTUUID=614e0000-0000 rw rootwait";
    };

    fiq-debugger {
        compatible = "rockchip,fiq-debugger";
        rockchip,serial-id = <2>;
        rockchip,wake-irq = <0>;
        /* If enable uart uses irq instead of fiq */
        rockchip,irq-mode-enable = <1>;
        rockchip,baudrate = <1500000>;  /* Only 115200 and 1500000 */
        interrupts = <GIC_SPI 252 IRQ_TYPE_LEVEL_LOW>;
        pinctrl-names = "default";
        pinctrl-0 = <&uart2m0_xfer>;
        status = "okay";
    };
/*-------------内容省略--------------*/
    &saradc {
        vref-supply = <&vcca_1v8>;
        status = "okay";
    };

    &tsadc {
        status = "okay";
    };
/*-------------以下内容省略--------------*/
}
```

```dts fold file:rk3568.dtsi
#include <dt-bindings/clock/rk3568-cru.h>
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/interrupt-controller/irq.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/soc/rockchip,boot-mode.h>
#include <dt-bindings/phy/phy.h>
#include <dt-bindings/power/rk3568-power.h>
#include <dt-bindings/soc/rockchip-system-status.h>
#include <dt-bindings/suspend/rockchip-rk3568.h>
#include <dt-bindings/thermal/thermal.h>
#include "rk3568-dram-default-timing.dtsi"

/ {
    compatible = "rockchip,rk3568";

    interrupt-parent = <&gic>;
    #address-cells = <2>;
    #size-cells = <2>;

    aliases {
        csi2dphy0 = &csi2_dphy0;
        csi2dphy1 = &csi2_dphy1;
        csi2dphy2 = &csi2_dphy2;
        dsi0 = &dsi0;
        dsi1 = &dsi1;
        ethernet0 = &gmac0;

/*-------------以下内容省略--------------*/

}
```

从上面的代码可以看出，设备树文件主要分为三个部分：
**1、头文件部分（3-9行）**
- 和C语言一样，设备树也可以使用`#include`来引用头文件，这些头文件有两种：
	- `.h`：定义各种常数和宏
	- `.dtsi`：包含引用的设备树定义
- 比如 `rk3568.dtsi` 就是瑞芯微官方提供的，包含了RK3568芯片平台通用的设备定义

**2、设备树节点（11-30行）**
- 设备树最明显的特征就是由很多大括号`{}`嵌套组成，每个大括号就是一个“节点”
- `/{ ... }`表示根节点，每个设备树只有一个根节点
- 在根节点里面的 `chosen { ... }`、`fiq-debugger { ... }`等都是子节点
- 虽然引用了`rk3568.dtsi`，但最终会合并成一个完整的设备树

**3、节点追加部分（32-39行）**
- 带有 `&` 符号的节点表示对已存在的节点的补充或修改，比如
	- `&saradc { ... }`：表示对已定义的saradc节点添加新的属性
	- `&tsadc { ... }`：表示对已定义的tsadb节点进行配置
	- 这些被追加的节点通常定义在`rk3568.dtsi`等包含文件

简单来说，设备树就是一个树状结构，有一个根节点，下面挂着很多子节点，子节点还可以再包含子节点。每个节点都描述了一个硬件设备或者系统配置信息。

### 3.1 节点基本格式

节点的结构参考：
```dts flod file:节点结构参考
//必要的DTS 文件版本说明
/dts-v1/; 
//包含头文件，可以是dtsi、dts、h等文件
#include "example.dtsi"    
//根节点
/ {  
	// 节点1，名称是node1-name，单元地址和reg属性的第一个地址一致
	node1-name@unit-address{    
		compatible = "xxx,xxxx";
		// 节点属性和属性值，是字符串
		a-string-property = "A string";
		a-string-list-property = "first string", "second string";
		a-byte-data-preperty = [0x00 0x13 0x24 0x36];

		//节点1的子节点1 “label”是标签
		label: child-node1 {
			first-child-property;
			second-child-property = <1>;
			a-string-property = "Hello,world";
		};
		// 子节点2
		child-node2 {
		
		};
	};
	// 节点2
	node2-name {
		an-empty-property;
		// each number cdell is a uint32
		a-cell-property = <1 2 3 4>;
		// 节点2的子节点1
		child-node1 {
			my-cousin = <&cousin>;
		};
	};
	......
};
```

设备树的节点就像是一个个的“盒子”，每个盒子都有自己的名字和内容。节点的基本格式如下：
```dts file:节点的基本格式
node-name@unit-address{
	//节点属性	
};
```

**节点名称规则**
- 长度：1~31个字符
- 可用字符：数字(0-9)、字母(a-z, A-Z)、逗号(,)、句号(.)、下划线(`_`)、加号(+)、减号(-)
- 必须以大写/小写字母开头，要能描述设备类别

**单元地址(@unit-address)**
- `@`符号是分隔符
- `unit-address`是单元地址，要和节点的`reg`属性第一个地址一致，如果没有`reg`属性，可以省略
- 同级节点的完整名称（node-name@unit-address）必须唯一

> [!warning]+ 注意
> 根节点比较特殊，它没有名称，直接用`/`表示

### 3.2 节点标签

在设备树中，我们经常看到这样的写法
```dts file:节点标签
cpu0: cpu@0{
	//节点内容
};
```

这里的`cpu0`就是节点标签，它的作用是：
- 给节点起个简短的“**外号**”
- 方便其他地方**引用**这个节点
- 可以用来**追加**内容到这个节点

### 3.3 节点路径

就像电脑上的文件路径一样，设备树中每个节点都有唯一的路径
```shell
/node1-name/child-node1
```
- 从根节点`/`开始
- 用`/`分隔不同层级
- 同一层级的节点名必须唯一
- 不同层级的节点名可以相同

### 3.4 节点属性

节点的大括号`{}`里面就是属性，这些属性告诉内核关于硬件的重要信息（板级硬件描述信息），驱动会通过一些API函数获取这些信息。

节点属性分为**标准属性**和**自定义属性**，也就是说我们在设备树中可以根据自己的实际需要定义、添加设备属性。 标准属性的属性名是固定的，自定义属性名可按照要求自行定义。

**compatibled属性**：
```c
compatible = "dakkk,lubancat2","rockchip,rk3568";
```
- 由一个或多个字符串组成，多个字符串时用逗号分隔
- 每一个代表了设备的节点都要有compatible属性
- 系统决定绑定设备到设备驱动的关键
- 也可以用来查找节点
- 另外可以通过节点名或节点路径查找指定的节点
- 举例：
	- 系统初始化时会初始化platform总线上的设备，根据设备节点的“compatible”属性和驱动中的of_match_table对应的值，匹配了就加载对应的驱动


**model属性**
```c
model = "dakkk lubancat2 HDMI";
```
- 字符串格式
- 指定设备的制造商和型号
- 一般格式：`制造商，型号`

**status属性**
```c
status = "okay";  //启用设备
status = "disabled" //禁用设备

/* External sound card */
sound: sound {
    status = "disabled";
};
```
- 字符串格式
- 控制设备是否启用
- 常用值：
	- `okay`(启用)
	- `disabled`(禁用)
	- `fail`(设备不可运行，驱动不支持，待修复)
	- `fail-sss`(设备不可运行，驱动不支持，待修复。sss的值与具体的设备相关)
- 不写默认是启用的

**`#address-cells` 和 `#size-cells` 属性**
```c fold
soc {
    #address-cells = <1>;
    #size-cells = <1>;
    compatible = "simple-bus";
    interrupt-parent = <&gpc>;
    ranges;
    ocrams: sram@900000 {
            compatible = "fsl,lpm-sram";
            reg = <0x900000 0x4000>;
    };
};
```
- u32格式
- 同时存在，用在有子节点的节点中
- “ac”指定了`子节点地址`字段占几个单元格
- “sc”指定了`子节点大小`字段占几个单元格
- 决定了子节点 `reg` 属性格式（即哪些数据是地址，哪些数据是大小）
- 举例：
	- `#address-cells=2，#size-cells=1` -> `reg = <address address size address address size>`
	- 因为每个cells是一个32位宽的数字，例如需要表示一个64位宽的地址时，就要使用两个address单元来表示

**reg属性**
```c
reg = <0x900000 0x4000>;
```
- 属性值类型：地址、大小数据对
- 描述设备的地址和大小（资源设备在其父总线定义的地址空间内的地址）
- 通常表示一块寄存器的起始地址（偏移地址）和长度
- 格式由父节点的 `#address-cells` 和 `#size-cells` 决定
- 上面的例子表示：起始地址 0x900000，地址空间 0x4000

**ranges属性**
```c
ranges; //空的ranges，表示地址直接映射
```
- 属性值类型：任意数量的 <子地址、父地址、地址长度>编码
- 定义子地址空间到父地址空间的映射
- 如果地址空间相同，可以写空的 `ranges;`
- 举例：
	- 对于#address-cells和#size-cells都为1的话，以`ranges=<0x0 0x10 0x20>`为例
	- 将子地址的从`0x0~(0x0 + 0x20)`的地址空间映射到父地址的`0x10~(0x10 + 0x20)`

**name和device_type属性（很少使用，废弃）**
```dts fold
example{
    name = "name"
}
	//根节点下
	cpus {
		#address-cells = <2>;
		#size-cells = <0>;
	
		cpu0: cpu@0 {
				device_type = "cpu";
				compatible = "arm,cortex-a55";
				reg = <0x0 0x0>;
				enable-method = "psci";
				clocks = <&scmi_clk 0>;
				operating-points-v2 = <&cpu0_opp_table>;
				cpu-idle-states = <&CPU_SLEEP>;
				#cooling-cells = <2>;
				dynamic-power-coefficient = <187>;
	};
```

### 3.5 追加/修改节点内容

```c
&cpu0{
	cpu-supply = <&vdd-cpu>;
};
```
- 并不包含在根节点“/{…}”内
- 用`&`符号引用已存在的节点标签
- 可以向原有节点添加新的属性
- 这个节点可能定义在当前文件或包含的其他文件中

### 3.6 特殊节点

**aliases子节点**
```c fold
    aliases {
            csi2dphy0 = &csi2_dphy0;
            csi2dphy1 = &csi2_dphy1;
            csi2dphy2 = &csi2_dphy2;
    /*----------- 省略------------*/
            mmc0 = &sdhci;
            mmc1 = &sdmmc0;
            mmc2 = &sdmmc1;
            mmc3 = &sdmmc2;
            serial0 = &uart0;
            serial1 = &uart1;
            serial2 = &uart2;
    /*----------- 以下省略------------*/
}
```
- 为其他节点起别名，如使用serial0来指代uart0节点
- 方便驱动程序快速找到节点（一般是使用节点路径一步步找到节点，也可以使用别名一步到位）
- 类似于给节点起个简短的“绰号”

**chosen子节点**
```c
chosen {
	bootargs = "earlycon=uart8250,mmio32,0xfe660000 console=ttyFIQ0 root=PARTUUID=614e0000-0000 rw rootwait";
}
```
- 不代表实际硬件，用于给内核传递启动参数
- uboot会通过这个节点（通道）向内核传递配置信息，这部分内容是uboot和内核自动完成的，作为初学者我们不必深究

在中断、时钟部分也有自己的节点标准属性，随着深入的学习我们会详细介绍这些节点标准属性。

## 4 如何获取设备树节点信息

在设备树中“节点”对应实际硬件中的设备，我们在设备树中添加一个“led”节点，那么就可以从这个节点获取编写led驱动需要用到的信息，如控制寄存器地址，led时钟控制寄存器地址等

下面开始学习 **如何从设备树的设备节点获取数据**。内核提供了一组函数用于从设备节点获取资源（设备节点中定义的属性），这些函数以`of_`开头，称为OF操作函数

> [!tip]+ 提示
> 下面的代码，都可以在`内核源码/include/linux/of.h`中找到

### 4.1 查找节点函数

#### 4.1.1 device_node结构体详解

找到节点后，都会返回一个`device_node`结构体，这是设备树操作的核心数据结构
```c fold file:device_node结构体
struct device_node{
	// 节点名称
	const char *name;
	// 节点类型（device_type属性值）
	const char *type
	// 节点句柄
	phandle phandle;
	// 节点完整名称
	const char *full_name;
	// 固件节点句柄
	struct fwnode_handle fwnode;

	//属性链表
	struct property *properties;
	//已删除的属性
	struct property *deadprops;
	//父节点指针
	struct device_node *parent;
	//子节点指针
	struct device_node *child;
	//兄弟节点指针
	struct device_node *sibling;

#if defined(CONFIG_OF_KOBI)
	struct kobject kobj;
#endif
	unsigned long _flags;
	void *data
#if defined(CONFIG_SPARC)
	const char *path_component_name;
	unsigned int unique_id;
	struct of_irq_controller *irq_trans;
#endif
};
```
- name：节点的name属性值
- type：节点的device_type属性值
- full_name：节点的完整名称，在device_node结构体后面放一个字符串，full_name指向它
- properties：链表，连接该节点的所有属性
- parent/child/sibiling：用于构建设备树层次结构

#### 4.1.2 by 节点路径

```c
struct device_node *of_find_node_by_path(const char *path)
```
- **path**：设备树中节点的完整路径（类型文件系统路径）
- **返回值**：
	- 成功->`device_node`结构体指针
	- 失败->返回`NULL`

**使用场景**：知道设备节点的完整路径时使用，比如`"/soc/uart@12345"`

#### 4.1.3 by 节点名字

```c
struct device_node *of_find_node_by_name(struct device_node *from， const char *name);
```
- **from**：从哪个节点开始查找（不包括节点本身），NULL表示从根节点开始
- **name**：要查找的节点名称
- **返回值**：
	- 成功->`device_node`结构体指针
	- 失败->返回`NULL`

#### 4.1.4 by 节点类型

```c
struct device_node *of_find_node_by_type(struct device_node *from,const char *type)；
```
- **from**：从哪个节点开始查找（不包括节点本身），NULL表示从根节点开始
- **type**：要查找的节点类型（对应device_node->type）
- **返回值**：
	- 成功->`device_node`结构体指针
	- 失败->返回`NULL`

#### 4.1.5 by 节点类型&compatible属性

**特点**：比前面的函数多了一个compatible属性作为筛选条件，查找更精确

```c
struct device_node *of_find_compatible_node(struct device_node *from, const char *type, const char *compatible);
```
- **from**：从哪个节点开始查找（不包括节点本身），NULL表示从根节点开始
- **type**：要查找的节点类型（对应device_node->type）
- **compatible**：节点的compatible属性值
- **返回值**：
	- 成功->`device_node`结构体指针
	- 失败->返回`NULL`

#### 4.1.6 by 匹配表

**最强大的查找函数，可以同时匹配多个条件，筛选最精确**

```c
static inline struct device_node *of_find_matching_node_and_match(struct device_node *from, const struct of_device_id *matches, const struct of_device_id **match)
```
- **from**：从哪个节点开始查找（不包括节点本身），NULL表示从根节点开始
- **matches**：源匹配表，包含多种匹配条件
- **match**：查找到的匹配结果

**of_device_id结构体**
```c file:of_device_id结构体
struct of_device_id {
	//节点name属性值
	char name[32];
	//节点device_type属性值
	char type[32];
	//节点compatible属性值
	char compatible[128];
	//链表，连接该节点的所有属性
	const void *data;
};
```

#### 4.1.7 find 父节点

```c
struct device_node *of_get_parent(const struct device_node *node)
```
- node：要查找其父节点的节点
- 返回值
	- 成功：返回父节点的`device_node`指针
	- 失败：返回`NULL`

#### 4.1.8 find 子节点

```c
struct device_node *of_get_next_child(const struct device_node *node, struct device_node *prev)
```
- node：要查找其子节点的父节点
- prev：前一个子节点，NULL表示查找第一个子节点
- 返回值
	- 成功：返回子节点的`device_node`指针
	- 失败：返回`NULL`

**使用方法**：迭代查找过程
- 查找第一个子节点：prev = NULL
- 查找第二个子节点：prev = 第一个子节点
- 依次类推...

#### 4.1.9 函数方式总结

**按查找方式分类**
- **路径查找**：通过完整路径直接定位 - `of_find_node_by_path` 
	- 路径是从设备树源文件(.dts)中的到的
- **属性查找**：根据节点属性在指定节点后查找
	- `of_find_node_by_name`
	- `of_find_node_by_type`
	- `of_find_compatible_node`
	- `of_find_matching_node_and_match` 
- **关系查找**：根据父子关系查找
	- `of_get_parent`
	- `of_get_next_child`

**共同特点**：
- 所有函数返回值类型都是`struct device_node *`
- 查找失败都返回`NULL`
- 成功后可以通过返回的`device_node`结构体获取节点的详细信息

### 4.2 提取属性值的of函数

上一小节讲解了7个查找节点的函数，它们都是返回device_node这个结构体指针

找到设备节点后，我们通常需要读取节点中的属性信息，Linux内核提供了一系列函数来操作设备树节点的属性。

> [!tip]+ 提示
> 下面的代码，都可以在`内核源码/include/linux/of.h`中找到

#### 4.2.1 property结构体详解

```c fold file:property结构体
struct property{
	//属性名
	char *name;
	//属性长度
	int length;
	//属性值
	void *value;
	//下一个属性(链表结构)
	struct property *next;
	
#if defined(CONFIG_OF_DYNAMIC) || defined(CONFIG_SPARC)
	unsigned long _flags;
#endif
#if defined(CONFIG_OF_PROMTREE)
	unsigned int unique_id;
#endif
#if defined(CONFIG_OF_KOBJ)
	struct bin_attribute attr;
#endif
}
```
- name：属性的名称
- length：属性值的字节长度
- value：属性值的数据指针
- next：指向下一个属性（所有属性组成链表）

#### 4.2.2 查找节点属性

```c
struct property *of_find_property(const srtuct device_node *np,const char name,int *lenp)
```
- **np**：要获取属性的设备节点指针
- **name**：属性名称
- **lenp**：输出参数，返回属性值的大小
- **返回值**：
	- 成功：返回`property`结构体指针
	- 失败：返回`NULL`

#### 4.2.3 读取整型属性

内核提供了读取不同位整型数据的函数组，支持8、16、32、64位整数

```c
//8位整数读取函数
int of_property_read_u8_array(const struct device_node *np, const char *propname, u8 *out_values, size_t sz)

//16位整数读取函数
int of_property_read_u16_array(const struct device_node *np, const char *propname, u16 *out_values, size_t sz)

//32位整数读取函数
int of_property_read_u32_array(const struct device_node *np, const char *propname, u32 *out_values, size_t sz)

//64位整数读取函数
int of_property_read_u64_array(const struct device_node *np, const char *propname, u64 *out_values, size_t sz)
```
- **np**：要获取属性的设备节点指针
- **propname**：要获取的属性名称
- **out_values**：`输出`参数，保存读取到的数据
- **sz**：`输入`参数，指定读取的数据个数(长度)
- **返回值**：
	- 成功：返回0
	- 失败：返回错误码
		- `-EINVAL`：属性不存在
		- `-ENODATA`：没有要读取的数据
		- `-EOVERFLOW`：属性值列表太小

#### 4.2.4 读取整型属性（简化版）

这组函数是对上面数组读取函数的简化封装，专门用于`读取单个整数值`，即将读取的长度固定为1

```c
int of_property_read_u8(const struct device_node *np, const char *propname, u8 *out_values);

int of_property_read_u16(const struct device_node *np, const char *propname, u16 *out_values);

int of_property_read_u32(const struct device_node *np, const char *propname, u32 *out_values);

int of_property_read_u64(const struct device_node *np, const char *propname, u64 *out_values);
```

**使用场景**：当属性值只包含一个整数时，使用这组函数更简洁方便

#### 4.2.5 读取字符串属性函数

设备节点中有很多字符串属性（如compatible、status、type等），内核提供了专门的字符串读取函数

**1、基础字符串读取函数- of_property_read_string**
```c
int of_property_read_string(const struct device_node *np,const char *propname, const char **out_string)
```
- **np**：要获取属性的设备节点指针
- **propname**：要获取的属性名称
- **out_string**：输出参数，返回字符串指针（指向属性值的首地址）
- **返回值**
	- 成功：返回0
	- 失败：返回错误状态码

> [!warning]+ 注意事项
> 这个函数只能获取属性值的首地址（第一个字符串），如果属性包含多个字符串（在内存中连续存储，使用`0`分隔），需要手动处理地址偏移，比较麻烦

**2、推荐的字符串读取函数- of_property_read_string_index**
```c
int of_property_read_string_index(struct device_node *np, const char *propname, int index, const char **out_string)
```
- **index**：指定读取第几个字符串（从0开始计数）

> [!abstract]+ **优势**
> 对比第一个函数，增加了index参数，可以直接指定读取属性中第几个字符串，比较方便

#### 4.2.6 读取布尔型属性函数

```c
static inline bool of_property_read_bool(const struct device_node *np, const char *propname);
```
- **np**：要获取属性的设备节点指针
- **propname**：要获取的属性名称
- **返回值**
	- true：属性存在
	- false：属性不存在

> [!tip]+ **特殊说明**
> 这个函数比较特殊，不是读取布尔属性的具体值，而是检查这个属性是否存在
> - 如果属性存在，返回`true`
> - 如果属性不存在，返回`false`
>如果需要获取布尔属性的具体值，可以使用前面介绍的"万能"函数`of_find_property`

#### 4.2.7 属性操作函数总结

**1、通用属性查找**：`of_find_property` - 可以获取任何类型的属性
**2、整型属性读取**：`of_property_read_uX_array`和`of_property_read_uX` - 专门读取整数
**3、字符串属性读取**：`of_property_read_string`和`of_property_read_string_index` - 专门读取字符串
**4、布尔属性检查**：`of_property_read_bool` - 检查属性是否存在

> [!tip]+ **使用建议**
> - 读取单个整数：使用`of_property_read_uX`系列
> - 读取整数数组：使用`of_property_read_uX_array`系列
> - 读取字符串：推荐使用`of_property_read_string_index`
> - 检查属性存在性：使用`of_property_read_bool`
> - 获取复杂属性：使用`of_find_property`后自行解析

### 4.3 内存映射相关of函数

在设备树的设备节点中大多会包含一些内存相关的属性，如常用的reg属性。通常情况下，得到的寄存器地址之后还需要通过ioremao函数转换为虚拟地址。

内核提供了of函数，可以提取这些属性，并自动完成物理地址到虚拟地址的转换

**of_iomap函数**
```c
void __iomem *of_iomap(struct device_node *np,int index)
```
- **np**：要获取属性的设备节点指针
- **index**：指定映射属性的那一段（从0开始），如reg的某一段
- **返回值**：
	- 成功：转换得到的地址
	- 失败：NULL

**of_address_to_resource函数**：获取设备树设置的物理地址
```c
int of_address_to_resource(struct device_node *np,int index,struct resource *r)
```
- **np**：要获取属性的设备节点指针
- **index**：指定映射属性的那一段（从0开始），如reg的某一段
- **r**：`输出`参数，接收得到的物理地址信息
- **返回值**：
	- 成功：转换得到的地址
	- 失败：NULL

resource结构体，可以参考[2.2 何为设备信息？](7_平台设备驱动.md#2.2%20何为设备信息？)

这里介绍了三类常用的of函数，这些基本满足我们的需求，其他of函数后续如果使用到我们在详细介绍

## 5 向设备树中添加设备节点实验

### 5.1 实验说明

在实际开发中，我们通常不会从零开始编写设备树，而是基于官方提供的设备树进行修改和扩展

**实验环境**
- **开发板**：野火Lubancat_RK系列（以lubancat2为例）
- **系统**：Ubuntu 20.04
- **设备树文件**：rk3568-lubancat-2-v2.dts
- **参考设备树**：
	- rk3568-lubancat-2.dtsi（led节点所在的设备树文件）
	- rk3568.dtsi（官方主干设备树，几千行代码）

**权限提醒**：验中如出现`Permission denied`或类似错误，请注意用户权限问题。大部分硬件外设操作需要root权限，解决方案：
- 在命令前加`sudo`
- 以root用户运行程序

### 5.2 代码讲解

**常见操作场景**
1. 向设备节点中增加新节点
2. 向现有设备节点追加数据
3. 编写设备树插件

**确定板卡使用的dtb文件，从而定位dts文件**
```shell
ls -l /boot
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9ffb63237cce395b28352c7077ca4073.png)

**设备树文件位置**：
- `arch/arm64/boot/dts/rockchip/rk3568-lubancat-2-v2.dts`
- `arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dtsi`
- `device_tree/rk3568.dtsi`

**添加设备节点实例**：在rk3568-lubancat-2.dtsi中添加一个led_test节点
```dts
	/* 添加led_test节点 */
	get_dts_info_test : get_dts_info_test {
		compatible = "get_dts_info_test",
		#address-cells = <1>;
		#size-cells = <1>;

		led@0xfdd60000 {
			compatible = "dk,led_test",
			reg = <0xfdd60000 0x00000100>;
			status = "okay";
		}
	};
```
**父节点get_dts_info_test**
- `compatible = "get_dts_info_test"`：兼容性标识
- `#address-cells = <1>`：子节点地址字段占用1个cell
- `#size-cells = <1>`：子节点大小字段占用1个cell

**子节点led@0xfdd60000**
- `compatible = "fire,led_test"`：设备兼容性字符串
- `reg = <0xfdd60000 0x00000100>`：寄存器地址和大小
	- `0xfdd60000`：GPIO0控制寄存器首地址
	- `0x00000100`：地址长度（$1*16^2 = 256字节$）
- `status = "okay"`：设备状态（启用）

**重要说明**
- 节点名`led@0xfdd60000`中的地址`0xfdd60000`必须与reg属性的第一个地址一致
- 由于父节点设置了`#address-cells = <1>, #size-cells = <1>`，所以reg属性格式为"地址，长度"

**设备树编译**：可以参考[11_设备树的编译](../../../../02-💻%20开发环境/3_Lubancat-RK3568/2_镜像构建与部署/11_设备树的编译.md)
- 方式一：编译内核的时候会把设备树一起编译（比较慢）
```shell
# 编译kerneldeb文件
./build.sh kerneldeb

# 编译extboot分区
./build.sh extboot
```
- 方式二：还是编译内核，不过在内核目录下编译
```shell
# 清除内核编译产物
sudo make mrproper

# 编译内核时指定 GCC 9
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8
```
- 方式三：在内核目录下编译设备树（推荐，比较快）
```shell
# 先指定配置文件
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

# 然后编译设备树
sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8 dtbs
```

> [!tip]+ 提示
> 这里编译指令比较长，是因为使用的是Ubuntu 24.04LTS版本，存在一些环境上的差异


**编译结果**
- 先确定刚开始的dtb创建时间
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f7b2bba6c5eb8c95739c35c8aaa0af79.png)
- 成功编译如下图
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/05ce80c9d4f7d33ed16f63346baf9aec.png)
- 编译后的生成目录和时间如下
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/eef49ef29e80a50e5047a893f07d6558.png)
### 5.3 程序结果

**加载设备树**
1. 通过scp将编译好的设备树文件拷贝到开发板
2. 使用mv命令，替换`/boot/dtb/rk3568-lubancat-2-v2.dtb`
3. 重启开发板（U-Boot会在启动时加载该设备树文件到内存供内核使用）

**验证结果**
**1、查看设备树节点**：对应文件系统中`/proc/device-tree`目录下的文件和文件夹
**2、进入设备目录**：
```shell
cd /proc/device-tree
ls # 可以看到get_dts_info_test目录
```


**3、查看节点内容**
```shell
cd get_dts_info_test
ls  # 显示节点属性和子节点
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b55a06394f47839c17996af1b559044c.png)

> [!tip]+ **节点结构体说明**
> - 属性：显示为文件
> - 子节点：显示为文件夹
> - name属性：自动生成，保存节点名称

**4、查看属性内容**
```shell
cat compatible  # 显示：dk,led_test
cat reg         # 显示寄存器信息（二进制格式）
cat status      # 显示：okay
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/091c3193f432c595197b6ddef4a2c17c.png)

### 5.4 实验总结

- 在设备树中添加了名为"get_dts_info_test"的父节点
- 在父节点下添加了"led@0xfdd60000"子节点
- 定义了必要的属性（compatible、reg、status等）
- 编译并加载了修改后的设备树
- 在文件系统中验证了节点的存在和属性内容

这个过程展示了设备树从编写、编译到加载验证的完整流程，为后续的驱动开发打下基础。

## 6 在驱动中获取节点属性实验

讲解如何使用of函数，从我们添加的节点中获取设备节点属性，驱动是基于平台驱动

### 6.1 代码讲解

完整代码如下
```c fold
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>

#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/module.h>
#include <linux/of.h>
#include <linux/platform_device.h>

//设备树节点
struct device_node  *led_test_device_node;
//节点
struct device_node  *led_device_node;
//定义属性结构体指针
struct property     *led_property;
int size = 0;
//保存读取到的属性值
unsigned int out_values[18];

static const struct of_device_id of_gpio_leds_match[] = {
    {.compatible = "get_dts_info_test"},
    {},
};

static int get_dts_info_probe(struct platform_device *pdev){
    int error_status = -1;
    pr_info("%s\n",__func__);

    //获取DTS属性信息
    led_test_device_node = of_find_node_by_path("/get_dts_info_test");
    if(led_test_device_node == NULL){
        printk(KERN_ALERT "get led_device_node failed!\n");
        return error_status;
    }

    //根据设备节点结构体输出节点的基本信息
    //输出节点名
    printk(KERN_ALERT "name: %s",led_test_device_node->name);
    //输出子结点的名字
    printk(KERN_ALERT "child name: %s",led_test_device_node->child->name);

    //获取子结点
    led_device_node = of_get_next_child(led_test_device_node, NULL);
    if(led_device_node == NULL){
        printk(KERN_ALERT "\n get led_device_node failed!\n");
        return error_status;
    }
    //输出子节点名
    printk(KERN_ALERT "name:%s\n",led_device_node->name);
    //输出子结点的父结点名
    printk(KERN_ALERT "parenet name:%s\n",led_device_node->parent->name);

    //获取property属性
    led_property = of_find_property(led_device_node, "compatible", &size);
    if(led_property == NULL){
        printk(KERN_ALERT "\n get led_property failed!\n");
        return error_status;
    }
    //实际读取到的长度
    printk(KERN_ALERT "size = %d\n",size);
    //输出属性名
    printk(KERN_ALERT "name: %s\n",led_property->name);
    //输出属性长度
    printk(KERN_ALERT "length: %d\n",led_property->length);
    //属性值
    printk(KERN_ALERT "value: %s\n",(char *)led_property->value);

    //获取reg地址属性
    error_status = of_property_read_u32_array(led_device_node, "reg", out_values, 2);
    if(error_status != 0){
        printk(KERN_ALERT "get out_values failed!\n");
        return -1;
    }
    printk(KERN_ALERT "0x%08X ",out_values[0]);
    printk(KERN_ALERT "0x%08X ",out_values[1]);

    return 0;
}

static int get_dts_info_remove(struct platform_device *pdev){
    pr_info("%s\n",__func__);
    return 0;
}

static struct platform_driver get_dts_info_driver = {
    .probe = get_dts_info_probe,
    .remove = get_dts_info_remove,
    .driver = {
        .name = "get_dts_info",
        .of_match_table = of_match_ptr(of_gpio_leds_match),
    },
};

module_platform_driver(get_dts_info_driver);

MODULE_DESCRIPTION("平台驱动：一个简单获取设备树属性的实验");
MODULE_LICENSE("GPL");
```


编译驱动模块
```shell
#使用了低版本的交叉编译器
#bear用于生成clangd索引时需要的compile_command.json文件
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
#如果本身就是低版本的交叉编译器
bear --make
```

该文件夹会产生get_dts_info.ko驱动模块

### 6.2 程序结果

译成功后将驱动.ko拷贝到开发板，使用insmod安装驱动模块然后可以看到:
```shell
sudo insmod get_dts_info.ko

# 查看内容
dmesg | tail
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6a86334e29a50e008bdcbf2e19f87104.png)



