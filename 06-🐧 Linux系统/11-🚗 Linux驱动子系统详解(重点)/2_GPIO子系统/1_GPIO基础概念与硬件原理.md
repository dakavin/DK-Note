
## 1 引言

在Linux嵌入式开发的世界里，GPIO（通用输入输出）控制是我们接触硬件的第一步。还记得第一次在单片机上点亮LED的兴奋吗？那时候，我们只需要简单地操作寄存器就能控制引脚。但当我们进入Linux的世界，会发现同样的操作变得复杂了许多。

这种复杂性其实是有道理的。想象一下，如果把单片机比作你自己的小屋，那么Linux系统就像一座现代化大厦。在小屋里，你可以随意开关任何电器；但在大厦里，所有的电气设备都由物业统一管理，你需要通过规范的申请流程才能使用。这种管理方式虽然增加了使用的复杂度，但带来了更好的安全性、标准化和可维护性。

本文将帮助你理解Linux GPIO子系统的基本概念，从硬件原理到软件抽象，让你能够在Linux系统中自如地控制GPIO。我们会像剥洋葱一样，一层一层地揭开GPIO子系统的神秘面纱。

## 2 GPIO的本质理解

### 2.1 什么是GPIO

GPIO的全称是General-Purpose Input/Output，翻译过来就是"通用输入输出"。这个名字已经很好地概括了它的特点：**通用**意味着灵活多变，**输入输出**意味着双向通信。

从硬件的角度来说，GPIO就是芯片上的一个引脚，但它不是普通的引脚。普通引脚往往有固定的功能，比如电源引脚只能供电，地引脚只能接地。而GPIO引脚就像变形金刚，可以通过软件配置成不同的功能：
- **输出模式**：GPIO可以输出高电平或低电平，控制外部设备
- **输入模式**：GPIO可以读取外部信号的电平状态
- **中断模式**：当外部信号发生变化时，GPIO可以触发中断通知CPU

让我们通过一个生活化的例子来理解。GPIO就像你家的智能插座，平时它可以控制电器的开关（输出功能），也可以检测是否有电器插入（输入功能），还能在检测到异常时发出警报（中断功能）。

### 2.2 GPIO的实际应用

为了更好地理解GPIO的工作原理，让我们看看两个最常见的应用场景：

**按键输入电路**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/46abdbf6f9314f7a5dea2d817f00ceee.png)

这是一个典型的按键检测电路。电路的工作原理很简单：
- 按键未按下时，GPIO引脚通过上拉电阻连接到VCC，读取到高电平
- 按键按下时，GPIO引脚直接连接到地，读取到低电平

通过检测GPIO引脚的电平变化，我们就能知道按键是否被按下。在代码中，这个检测过程就像不断询问："现在是高电平还是低电平？"

**LED输出控制电路**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c719367c3fc86975d272d90610592fb6.png)

LED控制电路展示了GPIO的输出功能。这里需要注意的是三极管的作用——它就像一个电子开关，**GPIO控制三极管的导通与截止，进而控制LED的亮灭**。使用三极管的好处是可以用GPIO的小电流控制LED的大电流，保护GPIO引脚不被烧坏。

> [!note]+ 背景知识 
> 在实际电路设计中，我们经常会遇到"拉高点亮"和"拉低点亮"两种设计。这取决于LED的连接方式：如果LED的负极接地，正极通过电阻接GPIO，那么GPIO输出高电平时LED点亮；反之，如果LED的正极接电源，负极通过电阻接GPIO，那么GPIO输出低电平时LED点亮。

## 3 RK3568的GPIO硬件架构

### 3.1 GPIO的组织结构

在深入学习GPIO编程之前，我们需要了解RK3568芯片是如何组织管理GPIO的。RK3568采用了分组管理的方式，这就像一个大公司的部门划分：

```txt
RK3568 GPIO组织结构：
- 5个GPIO控制器：GPIO0 到 GPIO4（相当于5个部门）
- 每个控制器管理4个端口：A、B、C、D（相当于每个部门4个小组）
- 每个端口包含8个引脚：0~7（相当于每个小组8个成员）
```

理论上，5×4×8=160个GPIO，但**实际上RK3568只有152个可用的GPIO**。为什么会少了8个呢？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c2317cd3817501608ff31cd23ed1c372.png)

通过查看数据手册，我们发现以下引脚是不存在的：
- GPIO0_D2
- GPIO0_D7
- GPIO2_C7
- GPIO4_D3 ~ GPIO4_D7

这就像某些部门的某些岗位是空缺的，虽然组织架构上有这个位置，但实际上没有人（引脚）。

### 3.2 GPIO编号计算

在Linux系统中，每个GPIO都有一个唯一的编号。这个编号是如何计算的呢？RK3568使用了一个简单而巧妙的公式：
```c
GPIO pin脚计算公式：pin = bank * 32 + number
GPIO小组编号计算公式：number = group * 8 + X

所以 pin = bank * 32 + group * 8 + X
```

其中：
- `bank`：控制器编号（0~4）
- `group`：端口编号（A=0, B=1, C=2, D=3）
- `X`：端口内的引脚序号（0~7）

让我们通过一个具体的例子来理解这个计算过程。假设我们要计算GPIO0_C7的编号：

```c
bank = 0      // GPIO0控制器
group = 2     // C端口对应数值2
X = 7         // 第7号引脚
number = 2*8 + 7 = 23
pin = 0*32 + 23 = 23
```

所以GPIO0_C7在Linux系统中的编号是23。这个编号在后续的编程中会经常用到，比如申请GPIO资源、设置GPIO方向等。

> [!tip]+ 实用技巧 
> 记住这个计算公式非常重要！在实际开发中，你经常需要在原理图上的引脚标号（如GPIO1_A3）和Linux系统中的GPIO编号之间进行转换。建议制作一个GPIO编号对照表，方便查阅。

## 4 GPIO的硬件特性

rk3568系列对应的手册：
- 数据手册：[Rockchip RK3568 Datasheet V1.2-20210601](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip%20RK3568%20Datasheet%20V1.2-20210601.pdf)
- 参考手册：[Rockchip_RK3568_TRM_Part1_V1.3-20220930P](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip_RK3568_TRM_Part1_V1.3-20220930P.pdf)

### 4.1 引脚复用（IOMUX）

现代的SoC芯片集成度越来越高，但**芯片的引脚数量是有限的**。为了在有限的引脚上实现更多的功能，芯片设计者想出了一个聪明的办法——引脚复用（IOMUX）。

什么是引脚复用呢？简单来说，就是一个物理引脚可以配置成不同的功能。这就像一个多功能会议室，可以根据需要布置成教室、会议室或者活动室。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6316d594901316431aed7d49629f7c07.png)

对于rockchip系类芯片，需要通过**参考手册**以及**数据手册**来确定引脚的复用功能，引脚服用相关信息可以通过数据手册查询

以`GPIO0_C7`为例，查询[Rockchip RK3568 Datasheet V1.2-20210601](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip%20RK3568%20Datasheet%20V1.2-20210601.pdf)手册可知，该引脚可以复用gpio功能，I2S、UART等功能，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cb620d7fdf99d92f0486f414b2ecd015.png)

**定位到GPIO0_C7所对应的控制寄存器**
- 查看参考手册第16.5章的Interface Description可知，有两种（PMUGRF和GRF）复用相关的寄存器
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ac7b1c0e47eac24943e48df53dd90fe7.png)
- 而`GPIO0组复用功能在PMU_GRF寄存器`，所以找到如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/10f7d3f91fd294c0c6008a8d224ed425.png)

- 然后查看上图红框，可以知道控制寄存器名称为 `PMU_GRF_GPIO0C_IOMUX_H`，然后搜索该寄存器的配置，如下图所示
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/30fb21e0e697dca80848bf2eaac5d386.png)

- `PMU_GRF_GPIO0C_IOMUX_H`寄存器，共32位，地址偏移0x0014，是GPIO0_C组的IOMUX控制高位寄存器
	- 高16位都是使能拉，控制低16位的写使能
	- 低16位对应4个引脚，每个引脚占用4bits（实际是3位）
	- 不同的值引脚服用为不同功能
	- 如GPIO0_C7引脚，控制该引脚复位功能的[14:12]位，都为0时，复用为GPIO功能，而系统复位默认为0，本实验可以不进行配置
	- 如果要复用为UART4_RXM0功能，就需要将[14:12]位设置为010，也就是将第13位设置为1，因为高16位控制对应低16位写使能，所以需要将第13+16bit位，也就是低17bit位设置为1，从而才能对第1bit位进行写操作

在Linux系统中，当我们要使用某个引脚作为GPIO时，首先需要将它的**复用功能配置为GPIO模式**。这个配置过程通常由Pinctrl子系统来完成，我们会在后续章节详细介绍。

### 4.2 上下拉配置（PULL）

GPIO的上下拉配置是一个容易被忽视但非常重要的特性。为什么需要上下拉呢？

想象一下，当GPIO配置为输入模式，但外部没有连接任何电路时，引脚的电平是不确定的，可能随机变化。这种不确定的状态在数字电路中是不允许的，因为它可能导致误判。上下拉电阻的作用就是给引脚一个确定的默认电平。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/753ad54b947cb52f6d3df82403b70f4a.png)

Rockchip的GPIO支持三种上下拉模式：
1. **`bias-disable`（悬空）**：不启用上下拉，引脚悬空。适用于外部电路已经有上下拉电阻的情况。
2. **`bias-pull-up`（上拉）**：内部连接一个电阻到电源，默认为高电平。常用于按键输入，按键按下时将引脚拉低。
3. **`bias-pull-down`（下拉）**：内部连接一个电阻到地，默认为低电平。适用于需要默认低电平的场合。

---

**定位到GPIO0_C7所对应的上下拉寄存器**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a05eb15e6a232286946d3f2ae5e66ae4.png)

GRF_GPIOA_P是相应GPIO上拉或下拉的控制寄存器，以GPIO0_C7进行说明

GRF_GPIOA_P寄存器具体如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4a86874ced63c494c2acc3966c43a9d4.png)


- 高16bit控制低16bit的写使能
- 其中[15:14]bit控制gpio1a4的上下拉配置
- 如果要将GPIO0_C7设置为上拉模式
	- [15:14]bit位设置为1
	- 14+16bit位设置为1

> [!warning]+ 重要提醒 
> 上下拉配置对所有引脚功能都生效，不仅仅是GPIO功能。比如配置为I2C功能时，通常需要上拉电阻。选择合适的上下拉模式对电路的稳定工作至关重要。

### 4.3 驱动强度（DRIVE-STRENGTH）

驱动强度决定了GPIO输出时能提供的电流大小。这就像水龙头的开度——开得越大，水流越急，但也更容易溅水

以RK3588的GPIO0C7为例，它支持多种驱动强度等级：

```txt
3'b000: 100ohm (Level0) - 最弱驱动
3'b001: 33ohm  (Level1)
3'b010: 50ohm  (Level2)
3'b100: 66ohm  (Level4)
3'b101: 25ohm  (Level5)
3'b110: 40ohm  (Level6) - 最强驱动
```

驱动强度的选择需要考虑以下因素：
- **负载能力**：驱动LED需要较强的驱动能力，而连接到其他芯片的信号线可能需要较弱的驱动
- **信号完整性**：过强的驱动可能导致信号过冲和振铃，影响信号质量
- **功耗**：驱动强度越大，功耗越高

在设备树中，我们可以这样配置驱动强度：
```dts
/* 设置驱动强度为Level5 */
drive-strength = <5>;
```

### 4.4 引脚电平

通过设置GPIO寄存器来设置输入输出、高低电平、中断、抖动等一些引脚的驱动能力，电气属性等，主要通过设置General Register Files (GRF)
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/da3e084032b79edbe0ee209cf8cea59b.png)
- `GPIO_SWPORT_DR_L`：低位引脚数据寄存器，设置高低电平
- `GPIO_SWPORT_DR_H`：高位引脚数据寄存器，设置高低电平

GPIO_SWPORT_DR_L和GPIO_SWPORT_DR_H寄存器具体如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/524007c4458245cf6f647672a1872e14.png)

**以GPIO_SWPORT_DR_H寄存器说明**
- 高16bit控制低16bit的写使能
- 低16位控制GPIO的高低电平，1=高电平，0=低电平
- 这个寄存器实际控制的GPIO数量是16个，也GPIO1中的A和B组
	- GPIO0_C0--->第0位
	- .......
	- GPIO0_D7--->第15位
- 所以如果要控制GPIO0_C7为高电平，就要写GPIO_SWPORT_DR_H寄存器
	- 写使能拉，第（16+7）bit设置为1
	- 高电平，第7bit设置为1

## 5 GPIO子系统在Linux中的地位

### 5.1 为什么需要GPIO子系统

在裸机开发中，我们可以直接操作GPIO的寄存器来控制引脚。但在Linux这样的多任务操作系统中，直接操作寄存器会带来很多问题：
1. **资源冲突**：多个驱动可能同时操作同一个GPIO，导致冲突
2. **安全隐患**：用户程序可以随意访问硬件，破坏系统稳定性
3. **可移植性差**：不同芯片的GPIO寄存器定义不同，代码无法复用

为了解决这些问题，Linux内核提供了GPIO子系统。它就像一个中间层，向上为驱动程序提供统一的API接口，向下屏蔽了不同硬件的差异。

### 5.2 GPIO子系统的基本架构

GPIO子系统的架构可以用下面这张图来表示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/df5e510beeb85c2c6a9c6a4bf8f7a2f9.png)

整个GPIO子系统分为三层：

**应用层**：包括`用户空间的应用程序`和`内核中的设备驱动`（如LED驱动、按键驱动等）。它们通过GPIO子系统提供的API来操作GPIO。

**GPIO核心层（gpiolib）**：这是GPIO子系统的核心，提供了两套API接口给应用层使用
- 基于`描述符的新接口（gpiod_xxx）`：推荐使用，功能更完善
- 基于`整数的传统接口（gpio_xxx）`：为了兼容老代码而保留

**硬件层**：`GPIO控制器驱动`，由芯片厂商提供，负责具体的硬件操作。

这种分层设计的好处是显而易见的：
- 驱动开发者不需要关心底层硬件细节
- 更换硬件平台时，上层代码基本不需要修改
- 内核可以统一管理GPIO资源，避免冲突

## 6 GPIO与Pinctrl子系统的关系

在学习GPIO子系统时，一个容易让人困惑的点是GPIO子系统和Pinctrl子系统的职责划分。让我们明确它们各自的职责：

**Pinctrl子系统 - "硬件配置管家"**
- **功能复用配置**：决定引脚用作什么功能（GPIO、UART、SPI、I2C等）
- **电气特性设置**：配置引脚的物理特性
    - 上下拉电阻（pull-up/pull-down）
    - 驱动强度（drive strength）
    - 电压域设置
    - 开漏/推挽模式等

**GPIO子系统 - "逻辑操作执行者"**
- **方向控制**：设置GPIO为输入或输出模式
- **电平操作**：读取输入电平、设置输出电平（高/低）
- **中断处理**：GPIO中断的注册和处理
- **GPIO生命周期管理**：申请、释放GPIO资源

> [!example]+ 形象比喻
> 就像装修和使用房间：
> - **Pinctrl**像装修工程师：负责房间的基础设施（水电线路、插座位置、电压规格）
> - **GPIO**像房间使用者：负责日常操作（开关灯、插拔电器）

**工作流程**
```txt
1. Pinctrl配置阶段：
   - 将引脚复用为GPIO功能
   - 设置上下拉电阻、驱动强度等电气特性

2. GPIO操作阶段：
   - 设置GPIO方向（输入/输出）
   - 进行电平读写操作
```

**关键理解**
- **电气特性（上下拉、驱动强度）属于Pinctrl子系统**，而**逻辑电平操作（高低电平）属于GPIO子系统**。
- 当我们在GPIO子系统中申请一个GPIO时，GPIO子系统会自动调用Pinctrl子系统完成引脚的复用配置和电气特性设置，这个过程对驱动开发者是透明的。
- 这样的设计分工明确：Pinctrl负责"硬件怎么连"，GPIO负责"软件怎么用"

## 7 本章小结

通过本章的学习，我们建立了对GPIO的基础认知：

1. **GPIO是什么**：通用输入输出接口，可以灵活配置功能的引脚
2. **GPIO的应用**：输入检测（如按键）、输出控制（如LED）、中断触发
3. **硬件特性**：引脚复用、上下拉配置、驱动强度设置等
4. **系统架构**：Linux通过GPIO子系统统一管理GPIO资源

这些基础知识就像建筑的地基，虽然看不见，但支撑着整个GPIO编程体系。在下一章中，我们将深入学习GPIO子系统的核心数据结构和编程接口，开始真正的GPIO编程之旅。

> [!example]+ 动手实践 
> 在你的开发板上执行 `ls /sys/class/gpio/` 命令，观察系统中的GPIO控制器。试着找到你的开发板原理图，对照本章内容计算几个GPIO的编号，加深理解。

记住，学习Linux驱动开发就像学习一门新语言，需要大量的练习和实践。不要担心一开始理解不深，随着实践的深入，这些概念会越来越清晰。让我们继续前进！