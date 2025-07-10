---
文章标题: "[[5_字符设备驱动—点亮LED灯实验]]"
文章作者: Dakkk
文章概要: |
  本文介绍Linux字符设备驱动开发实战，通过点亮LED灯实验讲解了MMU、地址转换、寄存器操作等核心概念，并提供了完整的驱动程序代码实现。
tags:
  - Linux驱动
  - 字符设备
  - LED控制
  - MMU
  - ioremap
  - GPIO
  - 寄存器操作
  - 设备树
相关文章:
  - "[[7_平台设备驱动]]"
  - "[[5_点亮LED]]"
  - "[[6_STM32CubeMX实现HAL点灯]]"
  - "[[7_GPIO输入按键]]"
  - "[[../../08-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/参考内容/设备树的引进和体验]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/5_字符设备驱动—点亮LED灯实验.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-11 13:53:14
---

经过上两个章节，我们已经了解字符设备驱动程序的基本框架，主要是掌握了如下内容：
- 如何申请和释放设备号（alloc_chrdev_region、unregister_chrdev_region）
- 关联/初始化设备（cdev_init）
- 添加以及注销设备（cdev_add、cdev_del）
**cdev和file_operations结构体非常重要！！！**

本小节，将做一个小实验-点亮led，前面已经通过操作寄存器的方式点亮了LED[9.2 控制心跳灯](../../../../02-💻%20开发环境/3_Lubancat-RK3568/1_快速使用手册/1_快速开始.md#9.2%20控制心跳灯)，本节将学习一下如何在Linux环境下驱动LED灯

首先，需要明白直接操作寄存器和通过驱动程序点亮LED的区别

## 1 设备驱动的作用与本质

直接操作寄存器点亮LED 和 通过驱动程序点亮LED **最本质的区别就是有无使用操作系统**

操作系统的存在大大降低了应用软件与硬件平台的耦合度，充当了硬件和应用软件的纽带。

应用软件只需要调用驱动程序接口API就可以让硬件去完成要求的开发，不需要关系硬件是如何工作的，这将大大提高我们应用程序的可移植性和开发效率

### 1.1 驱动的作用

设备驱动就像硬件设备和操作系统之间的“翻译官”，主要作用是让软件能够控制和使用各种硬件设备。

简单说，驱动程序就是让操作系统"认识"并"使用"硬件设备的桥梁，没有驱动程序，再好的硬件也无法工作。

**驱动的核心工作**
- 直接操作硬件寄存器，按照设备的工作方式进行读写
- 处理设备的各种通信方式：轮询检查、中断响应、DMA数据传输
- 进行内存映射，将物理地址转换为虚拟地址
- 最终让设备正常工作，如：网卡收发数据、显示器显示画面、硬盘存储文件

**两种开发模式**
- **裸机开发（无OS）**
	- 工程师可以自由定义接口
	- 比如控制LED灯：直接写LightOn()、LightOff()函数
- **系统开发（有OS）**
	- 必须遵循操作系统的架构规范
	- 比如Linux系统要求实现file_operations接口
	- 这样驱动才能正确集成到内核中

### 1.2 有无操作系统的区别

**无操作系统（裸机）时的设备驱动**：直接和硬件“对话”，比较简单直接：
- 直接操作寄存器控制硬件
- 每个设备驱动就是一个独立的软件模块
- 包含h文件（定义数据结构和函数声明）和c文件（具体实现）
- 其他模块要用设备时，直接包含头文件调用函数即可
- STM32开发就是典型例子，相对简单

**有操作系统时的设备驱动**：复杂一些，驱动成了“三明治”结构：
- `底层`：仍然需要直接操作硬件的部分（这部分必不可少）
- `上层`：必须提供操作系统规定的标准接口
- 所有同类设备的驱动接口都是统一的，不管具体是什么设备

**操作系统带来的好处：**
- `多任务并发`：可以同时运行多个程序
- `内存管理`：每个进程都有独立的4GB内存空间（32位Linux）
- `统一接口`：应用程序用同样的方式访问所有设备
	- 不管是键盘、硬盘还是网卡，都用write()、read()等函数
	- 程序员不需要关心设备的具体工作方式

## 2 内存管理单元MMU

在Linux环境直接访问物理内存是很危险的，如果用户不小心修改了内存中的数据，很有可能造成错误甚至系统崩溃。**为了解决这些问题内核引入了MMU**

### 2.1 MMU的功能

**MMU是什么？** 
- MMU（内存管理单元）是一个实际的硬件（不是软件），就像CPU和内存之间的“地址翻译官”
- 将虚拟地址翻译成真实的物理地址，同时管理和保护内存

**两种内存访问方式对比**
- `无MMU时`
  ![20250531_151048_325_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4bdb62bc38b2919c83e581db1ecc1e83.png)
	- CPU直接发出物理地址到内存条
	- 简单直接，但功能有限
- `有MMU时`
  ![20250531_151055_285_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0b7de774d01dc0a7c4b57d66fb15f4bd.png)
	- CPU发出虚拟地址→MMU查询页表→翻译成物理地址→访问内存
	- 复杂但功能强大

**什么是虚拟地址和物理地址？**
- `物理地址`：内存条上真实位置
	- 比如8G内存条，第一个单元是0x0000，第六个是0x0005
	- 这时硬件的绝对地址
- `虚拟地址`：程序中使用的地址
	- 32位系统有4G虚拟地址空间（2^32）
	- 程序看到的都是虚拟地址，由MMU翻译成物理地址

**MMU的主要功能**
1. `内存保护`（防止内存被恶意修改）
	- 给内存块设置读、写、执行权限
	- 检查CPU是特权模式还是用户模式
2. `地址转换`
	- 提供统一的内存空间抽象
	- 让程序运行在比物理内存更大的虚拟空间中
	- 进程之间相互隔离，互不干扰

**实际应用**
- 用ioremap映射设备地址，让程序通过虚拟地址直接操作硬件寄存器
- 很多实时系统（uCOS、FreeRTOS）可以无MMU运行，但功能有限

### 2.2 TLB的作用

**问题的由来**：MMU进行地址转换时效率太低
- 一级页表：每次读写数据需要访问2次内存（先查页表，再访问数据）
- 二级页表：每次读写数据需要访问3次内存
- 这样频繁访问内存严重影响CPU性能

**TLB是什么？**
- TLB（Translation Lookaside Buffer，地址转换缓冲器）就是MMU的“小抄本”，专门缓存最近使用过的地址转换结果

**TLB的工作流程**
1. CPU发出虚拟地址
2. MMU首先查看TLB
	- `命中`：直接使用缓存的地址描述符进行权限检查和地址转换
	- `未命中`：去内存中查页表，找到后进行转换，把结果存入TLB

**TLB满了怎么办？**
- TLB容量不大，满了就用`round—robin算法`（轮询算法）
- 找一个旧条目覆盖掉，为新的地址转换让位

**实际效果**：TLB大大提高了地址转换效率，避免了每次都要访问内存查页表的开销

**对开发者的意义**：在Linux环境中，由于开启了MMU，开发者要访问硬件寄存器（物理地址）时，必须使用物理地址到虚拟地址的转换函数，而TLB的存在让这种转换更加高效

**简单理解：** TLB就像是地址转换的“缓存”，把常用的转换结果记住，下次直接使用，避免重复的繁琐查表过程

## 3 地址转换函数

上面提到了物理地址到虚拟地址的转换函数。包括ioremap()地址映射和取消地址映射iounmap()函数

### 3.1 ioremap函数

**ioremap函数的作用：** 将物理地址映射成虚拟地址，让程序能够通过虚拟地址操作硬件寄存器。

**函数原型：**
```c
// 地址映射函数（内核源码/arch/arc/mm/ioremap.c）
void __iomem *ioremap(phys_addr_t paddr,unsigned long size)
#define ioremap ioremap
```
- `paddr`：要映射的物理地址（寄存器地址）
- `size`：需要映射的字节数
- **返回值**：映射后的虚拟地址指针（用这个来操作实际的物理地址）

**为什么需要ioremap？** 在开启MMU的Linux系统中：
- 程序不能直接访问物理地址
- 必须先把物理地址映射为虚拟地址
- 就像51/STM32直接操作寄存器，但多了一个“地址翻译”的步骤

---

理论上，通过ioremap获取到虚拟地址之后，我们便可以直接读写I/O内存，但是为了代码的跨平台性，应该使用linux专门指定的I/O函数

**标准的I/O读写函数** ：
```c
unsigned int ioread8(void __iomem *addr)
unsigned int ioread16(void __iomem *addr)
unsigned int ioread32(void __iomem *addr)

void iowrite8(u8 b, void __iomem *addr)
void iowrite16(u16 b, void __iomem *addr)
void iowrite32(u32 b, void __iomem *addr)
```
- 类似的还有writeb、writew、writel、readb、readw、readl等

> [!tip]+ ARM架构的内存I/O不同点
> - writex(readx)函数与iowritex(ioreadx)有一些区别
> - writex(readx)不进行端序的检查，而iowritex(ioreadx)会进行端序的检查

**读函数：**
- `ioread8()` - 读1字节
- `ioread16()` - 读2字节
- `ioread32()` - 读4字节

**写函数：**
- `iowrite8(data, addr)` - 写1字节
- `iowrite16(data, addr)` - 写2字节
- `iowrite32(data, addr)` - 写4字节

**实际例子（控制LED）：**
```c
#define GPIO0_BASE (0xFDD60000)
#define GPIO0_DR (GPIO0_BASE+0x0000)

va_dr = ioremap(GPIO0_DR,4);  //映射物理地址到虚拟地址：GPIO0的数据寄存器
val = ioread32(va_dr);        //读取寄存器值
val |= (0x00400000);          //设置GPIO0_A6引脚低电平
iowrite32(val,va_dr);         //写回寄存器
```

**简单理解**：ioremap就像给硬件寄存器申请了一个“虚拟门牌号”，程序通过这个虚拟门牌号就能找到并操作真实的硬件寄存器

### 3.2 iounmap函数

**iounmap函数的作用：** 取消之前用ioremap创建的地址映射，释放虚拟地址空间。

**函数原型：**
```c
void iounmap(void *addr)
```
- `addr`：需要取消映射的虚拟地址（ioremap的返回值）
- **返回值**：无

**使用场景：** 当不再需要访问某个硬件寄存器时，应该释放对应的虚拟地址映射。

**代码示例：**
```c
// 之前的映射
va_dr = ioremap(GPIO0_DR, 4);

// 使用虚拟地址操作寄存器
// ... 各种读写操作 ...

// 不再需要时，释放映射
iounmap(va_dr);
```

**为什么要用iounmap？**
- 释放系统资源，避免内存泄漏
- 类似于malloc和free的配对使用
- ioremap和iounmap应该成对出现

**简单理解：** 如果说ioremap是"申请虚拟门牌号"，那么iounmap就是"注销虚拟门牌号"，确保系统资源得到正确释放。

**注意事项：**
- 传入的地址`必须是`ioremap返回的地址
- 释放后不能再使用该虚拟地址
- 驱动卸载时一定要调用iounmap

## 4 点亮LED灯实验

从前面几章内核模块再到字符设备驱动，从理论到实践，总算是一切准备就绪，让我们开始着手手写LED的驱动代码吧。
1. 首先，需要一个LED字符设备结构体，包含要操作的寄存器地址
2. 其次，模块的加载卸载函数
	- 加载函数需要注册设备
	- 卸载函数则需要释放申请的资源
3. 然后是file_operations结构体以及open，write，read相关接口的实现

### 4.1 实验说明

#### 4.1.1 硬件介绍

本节实验使用到lubancat_RK系列板上的系统LED灯（或者自行在40pin上外接led）

#### 4.1.2 硬件原理图分析

LubanCat系列板卡，引出的led引脚可能不同，打开相应的板卡原理图来查看硬件链接，根据不同的引脚查询相应的寄存器地址。（也可以使用gpio引脚连接一个LED灯进行测试）

这里使用到的板卡为Lubancat2为例，查看的原理图文件：[LubanCat2_EBF410044V2_SCH_20230707](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/LubanCat2_EBF410044V2_SCH_20230707.pdf)，板子上只有一个用户灯（也就是系统的心跳灯，具体使用时需要关闭下设备树的leds节点），具体见下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c719367c3fc86975d272d90610592fb6.png)

- LED的阴极链接到rk3568芯片上GPIO0_C7

> [!warning]+ 注意
> 原理图要看板子后面的编号，不要选错了，我的选择是下面日期的文件
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/30351db9722f51cee01052977ca5fa70.png)


> [!tip]+ 提示
> - RK3568中引脚用GPIO编号，复用型引脚分为5组（0~4），每组里面都有32个复用型引脚，而且又分为4个小组（A、B、C、D），每个小组8个引脚（0~7）
> - 例如：GPIO1_A4是GPIO0大组，第1个小组，第5个引脚

```txt
RK3568 GPIO命名规则：GPIOx_yz
├── x: 组号 (0~4，共5组)
├── y: 小组号 (A/B/C/D，每组4个小组)  
└── z: 引脚号 (0~7，每小组8个引脚)
```

对于LED灯进行控制，也就是对上述GPIO的寄存器进行读写操作，可大致分为一下几个步骤：
1. 使能GPIO时钟（默认开启，不用设置）
2. 设置引脚复用为GPIO（复位默认为GPIO，不同配置）
3. 设置引脚属性（上下拉、速率、驱动能力，默认）
4. 控制GPIO引脚为输出，并输出高低电平

因为GPIO的时钟默认开启，引脚默认复用为GPIO，我们**只需要配置GPIO的引脚输入输出模式及电平即可**

#### 4.1.3 📕对LED进行寄存器配置

关于rk3568芯片寄存器的内容，我们这里快速梳理一遍会使用到的寄存器，不会深究个寄存器如何去配置

rk3568系列对应的手册：
- 数据手册：[Rockchip RK3568 Datasheet V1.2-20210601](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip%20RK3568%20Datasheet%20V1.2-20210601.pdf)
- 参考手册：[Rockchip_RK3568_TRM_Part1_V1.3-20220930P](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip_RK3568_TRM_Part1_V1.3-20220930P.pdf)

##### 4.1.3.1 引脚复用

对于rockchip系类芯片，需要通过**参考手册**以及**数据手册**来确定引脚的复用功能，引脚服用相关信息可以通过数据手册查询

以`GPIO0_C7`为例，查询[Rockchip RK3568 Datasheet V1.2-20210601](../../../../01-❇️%20参考资料/3_板卡参考手册/1_Lubancat-RK3568/Rockchip%20RK3568%20Datasheet%20V1.2-20210601.pdf)手册可知，该引脚可以复用gpio功能，I2S、UART等功能，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cb620d7fdf99d92f0486f414b2ecd015.png)


**定位到GPIO0_C7所对应的控制寄存器**
- 查看参考手册第16.5章的Interface Description可知，有两种（PMUGRF和GRF）复用相关的寄存器，而`GPIO0组复用功能在PMU_GRF寄存器`，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/10f7d3f91fd294c0c6008a8d224ed425.png)

- 然后查看上图红框，可以知道控制寄存器名称为 `PMU_GRF_GPIO0C_IOMUX_H`，然后搜索该寄存器的配置，如下图所示
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/30fb21e0e697dca80848bf2eaac5d386.png)

- `PMU_GRF_GPIO0C_IOMUX_H`寄存器，共32位，地址偏移0x0014，是GPIO0A组的IOMUX控制高位寄存器
	- 高16位都是使能拉，控制低16位的写使能
	- 低16位对应4个引脚，每个引脚占用4bits（实际是3位）
	- 不同的值引脚服用为不同功能
	- 如GPIO0_C7引脚，控制该引脚复位功能的[14:12]位，都为0时，复用为GPIO功能，而系统复位默认为0，本实验可以不进行配置
	- 如果要复用为UART4_RXM0功能，就需要将[14:12]位设置为010，也就是将第13位设置为1，因为高16位控制对应低16位写使能，所以需要将第13+16bit位，也就是低17bit位设置为1，从而才能对第1bit位进行写操作

##### 4.1.3.2 引脚电平（DR）

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

##### 4.1.3.3 引脚输入输出模式（DDR）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/dfe5a5dac4004d282606ca949c002e4c.png)
- GPIO_SWPORT_DDR_L：低位引脚数据方向寄存器，控制输入或者输出
- GPIO_SWPORT_DDR_H：高位引脚数据方向寄存器，控制输入或者输出

GPIO_SWPORT_DDR_L和GPIO_SWPORT_DDR_H寄存器具体如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/021cd625d948e878e9b0e390fa686fc6.png)

**以GPIO_SWPORT_DDR_L寄存器说明**
- 高16bit控制低16bit的写使能
- 低16位控制GPIO的输出方向，1=输出，0=输入
- 这个寄存器实际控制的GPIO数量是16个，也GPIO0中的C和D组
	- GPIO0_C0--->第0位
	- .......
	- GPIO0_D7--->第15位
- 所以如果要控制GPIO0_C7的输出，就要写GPIO_SWPORT_DDR_L寄存器
	- 写使能拉，第（16+7）bit设置为1
	- 高电平，第7bit设置为1

##### 4.1.3.4 引脚上下拉

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a05eb15e6a232286946d3f2ae5e66ae4.png)

GRF_GPIOA_P是相应GPIO上拉或下拉的控制寄存器，以GPIO0_C7进行说明

GRF_GPIOA_P寄存器具体如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4a86874ced63c494c2acc3966c43a9d4.png)


- 高16bit控制低16bit的写使能
- 其中[15:14]bit控制gpio1a4的上下拉配置
- 如果要将GPIO0_C7设置为上拉模式
	- [15:14]bit位设置为1
	- 14+16bit位设置为1

### 4.2 代码讲解

#### 4.2.1 定义GPIO寄存器物理地址

以GPIO0_C7为例，需要先确定GPIO1的基地址，其余寄存器均在基地址上进行偏移，查询[Rockchip_RK3568_TRM_Part1_V1.3-20220930P](../../5_硬件资源/Rockchip_RK3568_TRM_Part1_V1.3-20220930P.pdf) 手册中的`1.1 Address Mapping`可知，GPIO1的基地址为`0xFDD60000`，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/974d008efca1476910956636d4566411.png)


> [!tip]+ 提示
> - 复用已经是GPIO了，不用设置
> - 上下拉这里用不到，不用设置
> - 只需要设置电平和方向即可

确定基地址后，需要确定电平和方向寄存器对于基地址的偏移量，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9ae6c2c367a083b18b7aeba93a8ce356.png)


从上图可知需要设置的寄存器地址为base+offset，编写代码如下：
```c
#define GPIO0_BASE (0xFDD60000)

//一个寄存器32位，高16位都是使能，低16位对应16个引脚，控制电平
//GPIO1_A0 ... GPIO1_A7 ... GPIO1_B7
#define GPIO0_DR_L (GPIO0_BASE + 0x0000) //低16位引脚(A~B)
#define GPIO0_DR_H (GPIO0_BASE + 0x0004) //高16位引脚(C~D)

//控制方向，同上
#define GPIO0_DDR_L (GPIO0_BASE + 0x0008)
#define GPIO0_DDR_H (GPIO0_BASE + 0x000C)
```

代码使用宏定义，定义LED灯使用到的GPIO资源物理地址，后面需要将这些寄存器地址映射到虚拟地址上

#### 4.2.2 编写LED字符设备结构体且初始化

```c
// led灯的结构体
struct led_chrdev{
	struct cdev dev;
	unsigned int __iomem *va_dr;  // 数据寄存器，设置输出的电压
	unsigned int __iomem *va_ddr; // 数据方向寄存器，输入或者输出

	unsigned int led_pin; //引脚
}

// RGB灯的结构体数组，针对多个led的情况
static struct led_chrdev led_cdev[DEV_CNT] = {
    // 偏移，GPIO0_C7偏移7位
    {.led_pin = 7},
};
```

#### 4.2.3 内核RGB模块的加载和卸载函数

**第一部分：RGB模块的加载函数**
- 映射LED结构体中的虚拟地址，将虚拟地址和GPIO的物理寄存器一一对应
- 调用alloc_chrdev_region()函数申请一个未被占用的设备号
- 调用`class_create()`函数创建一个RGB灯的设备类
- 分别给三个LED建立其对应的字符设备结构体cdev和led_chrdev_fops关联
- 初始化字符设备结构体，最后注册并创建设备

**第二部分：RGB模块的卸载函数**
- 释放LED结构体内映射的虚拟地址
- 调用device_destory()函数，移出一个设备，并且删除`/sys/devices/virtual`目录下对应的设备目录，及`/dev`目录下对应的设备文件
- 调用cdev_del()函数来释放散列表中的对象以及cdev结构本身
- 释放被占用的设备号，及删除设备类

下面代码的3个LED同主设备号，次设备号不一样
```c
static __init int led_chrdev_init(void){
    int i = 0;
    dev_t cur_dev;
    unsigned int val = 0;

    printk("led_chrdev init(lubancat2 GPIO0_C7)\n");

    led_cdev[0].va_dr = ioremap(GPIO0_DR_H,4);
    led_cdev[0].va_ddr = ioremap(GPIO0_DDR_H,4);
    // 注册主设备号
    alloc_chrdev_region(&devno,0,DEV_CNT,DEV_NAME);
    // 创建设备类
    led_chrdev_class = class_create(THIS_MODULE,"led_chrdev");
    // 循环添加同一驱动程序的设备
    for(;i<DEV_CNT;i++){
        //初始化cdev
        cdev_init(&led_cdev[i].dev,&led_chrdev_fops);
        led_cdev[i].dev.owner = THIS_MODULE;
        //获取cdev的完整设备号
        cur_dev = MKDEV(MAJOR(devno),MINOR(devno)+i);
        //添加该设备到散列表
        cdev_add(&led_chrdev[i].dev,cur_dev,1);
        //设备节点创建工作 -> /dev/DEV_NAME%d 
        device_create(led_chrdev_class,NULL,cur_dev,NULL,DEV_NAME"%d",i);
    }
    return 0;
}
module_init(led_chrdev_init);

static __exit void led_chrdev_exit(void){ 
    int i;
    dev_t cur_dev;
    printk("led chrdev exit(lubancat2 GPIO0_C7)\n");

    // 释放虚拟地址
    for(i=0;i<DEV_CNT;i++){
        iounmap(led_cdev[i].va_dr);
        iounmap(led_cdev[i].va_ddr);
    }

    //删除设备节点和cdev
    for(i=0;i<DEV_CNT;i++){
        cur_dev = MKDEV(MAJOR(devno),MINOR(devno)+i);
        device_destory(led_chrdev_class,cur_dev);
        cdev_del(&led_cdev[i].cdev);
    }

    //释放设备号
    unregister_chrdev_region(devno, DEV_CNT);
    //删除设备类
    class_destroy(led_chrdev_class);
}
module_exit(led_chardev_exit);
```

#### 4.2.4 file_operations结构体成员函数的实现

**open函数需要完成的工作**

**第一步：通过container_of函数获取led_chrdev结构体的首地址**

**第二步：文件私有数据**
- 将file->private_data 指向设备结构体，用于保存用户自定义设备结构体的地址
- 后续，可以通过该私有数据区访问设备结构体的成员

**第三步：通过ioremap()函数实现地址的映射**
- 便于通过操作程序中的虚拟地址来间接控制物理寄存器
- 后面会使用设备树（设备树插件）的方式区描述寄存器及其相关属性，这里先埋下伏笔，循序渐进，顺藤摸瓜，便于真正理解并掌握linux驱动精髓

**第四步：通过ioread32()和iowrite32()等函数操作寄存器**
- 这里再强调一遍，即使理论上可以直接操作这段虚拟地址了但是Linux并不建议这么做

```c
static int led_chrdev_open(struct inode *inode,struct file *filp){
    unsigned int val = 0;
    // 通过结构体中的dev成员，找到这个结构体变量的首地址
    struct led_chrdev *led_cdev = (struct led_chrdev *)container_of(inode->i_cdev,struct led_chrdev, dev);
    // 把文件的私有地址指向设备结构体ledcdev
    filp->private_data = container_of(inode->i_cdev,struct led_chrdev,dev);

    printk("open\n");

    //设置输出模式
    val = ioread32(led_cdev->va_ddr);
    // 高位使能
    val |= ((unsigned int)0x1 << (led_cdev->led_pin+16));
    // 低位设置方向为输出
    val |= ((unsigned int)0x1 << (led_cdev->led_pin));
    iowrite32(val,led_cdev->va_ddr);

    //设置高电平
    val = ioread32(led_cdev->va_dr);
    // 高位使能
    val |= ((unsigned int)0x1 << (led_cdev->led_pin+16));
    // 低位设置方向为输出
    val |= ((unsigned int)0x1 << (led_cdev->led_pin));
    iowrite32(val,led_cdev->va_dr);

    return 0;
}
```

**下面接着分析write函数的实现**
```c
static ssize_t led_chrdev_write(struct filp *filp,const char __user *bf,size_t count,loff_t *ppos){
    unsigned long val = 0;
    char ret = 0;
    struct led_chrdev *led_cdev = (struct led_chrdev *)filp->private_data;

    get_user(ret,buf);
    val = ioread32(led_cdev->va_dr);
    if(ret == '0'){
        val |= ((unsigned int)0x01 << (led_cdev->led_pin+16));
        // 设置GPIO引脚输出低电平
        val &= ~((unsigned int)0x01 << (led_cdev->led_pin));
    }else{
        val |= ((unsigned int)0x01 << (led_cdev->led_pin+16));
        // 设置GPIO引脚输出高电平
        val |= ((unsigned int)0x01 << (led_cdev->led_pin));    
    }
    iowrite32(val,led_cdev->va_dr);

    return count;
}
```

**最后分析一下release函数的实现**
- 当最后一个打开设备的用户进程执行close()系统调用时，内核将调用驱动程序release()函数
- release()函数的主要任务
	- 清理未结束的输入输出从挨揍
	- 释放资源
	- 用户自定义排他标志的复位等
- 虚拟地址也需要使用iounmap()函数释放，不过在驱动模块退出的时候才释放，这里不用操作
```c
static int led_chrdev_release(struct inode *inode,struct file *filp){
    return 0;
}
```

#### 4.2.5 LED驱动完整代码

到这里我们的代码已经分析完成了，下面是本驱动的完整代码
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/cdev.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/io.h>

#define DEV_NAME "dk_led_chrdev"
#define DEV_CNT (1)

// GPIO1的基地址
#define GPIO0_BASE (0xFDD60000)

//一个寄存器32位，高16位都是使能，低16位对应16个引脚，控制电平
//GPIO1_A0 ... GPIO1_A7 ... GPIO1_B7
#define GPIO0_DR_L (GPIO0_BASE + 0x0000) //低16位引脚(A~B)
#define GPIO0_DR_H (GPIO0_BASE + 0x0004) //高16位引脚(C~D)
//控制方向，同上
#define GPIO0_DDR_L (GPIO0_BASE + 0x0008)
#define GPIO0_DDR_H (GPIO0_BASE + 0x000C)

static dev_t devno;
struct class *led_chrdev_class;

struct led_chrdev{
    struct cdev dev;
    unsigned int __iomem *va_dr;    //设置输出的电压
    unsigned int __iomem *va_ddr;   //设置输出的方向
    unsigned int led_pin; //偏移
};

static int led_chrdev_open(struct inode *inode,struct file *filp){
    unsigned int val = 0;
    // 通过结构体中的dev成员，找到这个结构体变量的首地址
    struct led_chrdev *led_cdev = (struct led_chrdev *)container_of(inode->i_cdev,struct led_chrdev, dev);
    // 把文件的私有地址指向设备结构体led_chrdev
    filp->private_data = container_of(inode->i_cdev,struct led_chrdev,dev);

    printk("open\n");

    //设置输出模式
    val = ioread32(led_cdev->va_ddr);
    // 高位使能
    val |= ((unsigned int)0x1 << (led_cdev->led_pin+16));
    // 低位设置方向为输出
    val |= ((unsigned int)0x1 << (led_cdev->led_pin));
    iowrite32(val,led_cdev->va_ddr);

    //设置高电平
    val = ioread32(led_cdev->va_dr);
    // 高位使能
    val |= ((unsigned int)0x1 << (led_cdev->led_pin+16));
    // 低位设置方向为输出
    val |= ((unsigned int)0x1 << (led_cdev->led_pin));
    iowrite32(val,led_cdev->va_dr);

    return 0;
}

static int led_chrdev_release(struct inode *inode,struct file *filp){
    return 0;
}

static ssize_t led_chrdev_write(struct file *filp,const char __user *buf,size_t count,loff_t *ppos){
    unsigned long val = 0;
    char ret = 0;
    struct led_chrdev *led_cdev = (struct led_chrdev *)(filp->private_data);

    get_user(ret,buf);
    val = ioread32(led_cdev->va_dr);
    if(ret == '0'){
        val |= ((unsigned int)0x01 << (led_cdev->led_pin+16));
        // 设置GPIO引脚输出低电平
        val &= ~((unsigned int)0x01 << (led_cdev->led_pin));
    }else{
        val |= ((unsigned int)0x01 << (led_cdev->led_pin+16));
        // 设置GPIO引脚输出高电平
        val |= ((unsigned int)0x01 << (led_cdev->led_pin));    
    }
    iowrite32(val,led_cdev->va_dr);

    return count;
}

static struct file_operations led_chrdev_fops = {
    .owner = THIS_MODULE,
    .open = led_chrdev_open,
    .release = led_chrdev_release,
    .write = led_chrdev_write,
};

static struct led_chrdev led_cdev[DEV_CNT] = {
    // 偏移，GPIO0_C7偏移4位
    {.led_pin = 7},
};

static __init int led_chrdev_init(void){
    int i = 0;
    dev_t cur_dev;

    printk("led_chrdev init(lubancat2 GPIO0_C7)\n");

    led_cdev[0].va_dr = ioremap(GPIO0_DR_H,4);
    led_cdev[0].va_ddr = ioremap(GPIO0_DDR_H,4);
    // 注册主设备号
    alloc_chrdev_region(&devno,0,DEV_CNT,DEV_NAME);
    // 创建设备类，名称为led_chrdev
    // 决定/sys/class/led_chrdev/dk_led_chrdev0  ← 类名是 "led_chrdev"
    led_chrdev_class = class_create(THIS_MODULE,"led_chrdev");
    // 循环添加同一驱动程序的设备
    for(;i<DEV_CNT;i++){
        //初始化cdev
        cdev_init(&led_cdev[i].dev,&led_chrdev_fops);
        led_cdev[i].dev.owner = THIS_MODULE;
        //获取cdev的完整设备号
        cur_dev = MKDEV(MAJOR(devno),MINOR(devno)+i);
        //添加该设备到散列表
        cdev_add(&led_cdev[i].dev,cur_dev,1);
        //设备节点创建工作 -> /dev/DEV_NAME%d 
        device_create(led_chrdev_class,NULL,cur_dev,NULL,DEV_NAME"%d",i);
    }
    return 0;
}
module_init(led_chrdev_init);

static __exit void led_chrdev_exit(void){ 
    int i;
    dev_t cur_dev;
    printk("led chrdev exit(lubancat2 GPIO0_C7)\n");

    // 释放虚拟地址
    for(i=0;i<DEV_CNT;i++){
        iounmap(led_cdev[i].va_dr);
        iounmap(led_cdev[i].va_ddr);
    }

    //删除设备节点和cdev
    for(i=0;i<DEV_CNT;i++){
        cur_dev = MKDEV(MAJOR(devno),MINOR(devno)+i);
        device_destroy(led_chrdev_class,cur_dev);
        cdev_del(&led_cdev[i].dev);
    }

    //释放设备号
    unregister_chrdev_region(devno, DEV_CNT);
    //删除设备类
    class_destroy(led_chrdev_class);
}
module_exit(led_chrdev_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("dakkk");
```

### 4.3 实验准备

如出现 `Permission denied` 或类似字样，请注意用户权限，大部分操作硬件外设的功能，几乎都需要root用户权限，简单的解决方案是在执行语句前加入sudo或以root用户运行程序。

#### 4.3.1 LED驱动Makefile

```makefile
KERNEL_DIR=../../kernel
ARCH=arm64
CROSS_COMPILE=aarch64-linux-gnu
export ARCH CROSS_COMPILE

obj-m := led_cdev.o

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules

.PHONY: clean

clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
```

#### 4.3.2 编译命令说明

```shell
#使用了低版本的交叉编译器
#bear用于生成clangd索引时需要的compile_command.json文件
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
#如果本身就是低版本的交叉编译器
bear --make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/fb1f02ce5cdb101189c16744f41eee09.png)

### 4.4 程序运行结果

在鲁班猫系列板卡，系统设备树中均默认使能了`LED`的设备功能，需要关闭设备树的leds节点，可以修改leds节点的 `status = "okay";` 为 `status = "disabled";`，然后编译设备树镜像替换，也可以在板卡中直接使用一下命令关闭系统leds驱动对LED的控制
```shell
# 将led的亮度调为0，同时led的触发条件自动变为none
sudo sh -c 'echo 0 > /sys/class/leds/sys_led/brightness'
```

通过scp将ko文件拷贝到开发板，然后执行命令加载
```shell
scp led_cdev.ko cat@192.168.31.69:/home/cat/

sudo insmod led_cdev.ko
```

然后可以在/dev目录下找到`dk_led_chrdev0`这个设备，可以通过直接给设备写入1/0来控制LED的亮灭
```shell
# 查看led设备
ls /dev/dk*
#绿灯亮
sudo sh -c 'echo 0 >/dev/dk_led_chrdev0'
#绿灯灭
sudo sh -c 'echo 1 >/dev/dk_led_chrdev0'
```

**如果灯无法操作，可以检查一下gpio**
```shell
# 查看GPIO状态
cat /sys/kernel/debug/gpio
# 或者
cat /proc/interrupts | grep gpio
```