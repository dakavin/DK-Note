
现在我们需要从更高的视角来理解整个**中断子系统框架**的组织结构。在掌握了基础概念后，让我们深入了解Linux是如何设计这个复杂而精妙的中断处理架构的。

简单来说，Linux中断子系统就像一座"四层大厦"，每一层都有明确的职责分工。从最底层的硬件信号，到最顶层的驱动程序，这个框架确保了中断处理的高效性和可移植性。

> [!note]+ 重要笔记
> 
> 理解中断子系统的分层架构，有助于我们在驱动开发中准确定位问题，并编写出高质量的中断处理代码。

## 1 中断子系统的四层架构

### 1.1 整体架构概览

一个完整的中断子系统框架可以分为四个层次，由上到下分别为**用户层、通用层、硬件相关层和硬件层**。这种分层设计的核心思想是实现硬件抽象和代码复用。

![中断子系统四层架构|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f283f44413998866941163494dab0058.png)

换句话说，就像盖房子一样，每一层都有自己的功能，上层不需要关心下层的具体实现细节，只需要通过标准接口进行交互。这种设计让Linux能够运行在各种不同的硬件平台上，而驱动程序代码却可以保持高度的一致性。

### 1.2 各层详细功能

**用户层**：
- 中断的使用者，主要包括各类设备驱动
- 这些驱动程序通过中断相关的接口进行中断的申请和注册
- 当外设触发中断时，用户层驱动程序会进行相应的回调处理，执行特定的操作

> [!example]+ 示例
> 
> 网卡驱动程序就是典型的用户层，它注册网卡中断处理函数，当网卡接收到数据包时，通过中断通知驱动程序及时处理数据。

**通用层**：
- 也可称为框架层，它是硬件无关的层次
- 通用层的代码在所有硬件平台上都是通用的，不依赖于具体的硬件架构或中断控制器
- 通用层提供了统一的接口和功能，用于管理和处理中断，使得驱动程序能够在不同的硬件平台上复用

**硬件相关层**：包含两部分代码
- 一部分是与`特定处理器架构`相关的代码，比如ARM64处理器的中断处理相关代码。这些代码负责处理特定架构的中断机制，包括中断向量表、中断处理程序等。
- 另一部分是`中断控制器`的驱动代码，用于与中断控制器进行通信和配置。这些代码与具体的中断控制器硬件相关

**硬件层**：
- 硬件层位于最底层，与具体的硬件连接相关。
- 它包括外设与SoC（系统片上芯片）的物理连接部分。
- 中断信号从外设传递到中断控制器，由中断控制器统一管理和路由到处理器。

> [!tip]+ 重要提示
> 
> 这种分层设计的最大优势是**代码复用性**。比如，同一个网卡驱动程序可以在不同的ARM平台上运行，而不需要修改上层代码。

## 2 中断系统硬件简介

### 2.1 中断硬件组成

中断硬件主要有三种器件参与，各个外设、中断控制器和CPU。它们之间的关系可以简单看下下面图片：

![中断硬件组成关系|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/17b9a2e0b0a52a49b229e2b144385171.png)

> [!tip]+ 提示 
> 在ARMv8体系结构中，将处理器处理事务的抽象过程定义为PE(Processing Element)，可以将PE简单理解为处理器核心

**外设**：在发生中断事件的时候，通过外设上的电气信号向中断控制器请求处理。就像各个部门有事情需要汇报时，会向总管理处发送请求。

**中断控制器**：由于中断数量越来越多，需要一个专门设备来管理中断，中断控制器就是起这个作用，它连接外设中断系统和CPU系统的桥梁，接收外部中断，处理后向处理器上报中断信号。以arm为例，GIC中断控制器，外部中断信号以后，经过处理标记为FIQ、IRQ等，然后传输给arm内核。

**CPU**：CPU的主要功能是运算，CPU并不处理中断优先级，只是接收中断控制器的中断信息，运行中断处理。例如：对于ARM，cpu会接收到是IRQ和FIQ信号，分别让ARM进入IRQ mode和FIQ mode。

### 2.2 软件框架

linux内核软件部分，中断子系统分成4个部分：

**硬件无关的代码**：我们称之内核通用中断处理模块。无论是哪种CPU，那种控制器，其中断处理的过程都有一些相同的内容，这些相同的内容被抽象出来，和硬件无关。此外，各个外设的驱动代码中，用一个统一的接口实现irq相关的管理，这些代码组成了内核中断子系统的核心部分。

**CPU架构相关的中断处理**：和系统使用的具体的CPU架构相关。不同的CPU架构（比如ARM64、x86）有不同的中断处理机制和指令集。

**中断控制器驱动代码**：和系统使用的中断控制器相关。不同的芯片厂商可能使用不同类型的中断控制器。

**普通其他驱动**：这些驱动将使用内核通用中断处理模块的API来实现自己的驱动逻辑。

## 3 重要概念说明

### 3.1 硬件中断ID与虚拟中断号

**HW interrupt ID**：硬件中断ID，就是`GIC硬件上的irq ID`，是GIC标准定义的，这是硬件层面的真实编号。

**IRQ number**：IRQ number是软件方面定义，和硬件无关，CPU需要为每一个外设中断编号，我们称之IRQ Number。这个IRQ number是一个`虚拟的interrupt ID`，仅仅是被CPU用来标识一个外设中断。

简单来说，硬件中断ID就像设备的"出厂编号"，而虚拟中断号就像系统内部的"管理编号"。系统需要建立这两者之间的映射关系，这样硬件发出的中断信号就能正确地传递给对应的软件处理程序。

### 3.2 IRQ domain

irq domain 翻译过来就是"irq 域"的意思，在内核中是一个比较常见的概念，也就是将某一类资源划分成不同的区域，相同的域下共享一些共同的属性。domain实际上也是对模块化的一种体现，`irq domain的产生是因为GIC对于级联的支持`。实际上，虽然内核对每个GIC都设立了不同的域，但是它只做了一件事：**负责GIC中hwirq和逻辑irq的映射**。

换句话说，irq domain就像一个"翻译官"，负责将硬件中断号翻译成Linux系统能理解的虚拟中断号。

### 3.3 中断向量表

中断向量表就是中断向量的列表，中断向量表在内存中存放，表中存放着中断源所对应的中断处理程序的入口地址。就像一个电话簿，当有中断来电时，系统根据电话号码（中断号）查找对应的接听人（处理程序）。

### 3.4 中断处理架构

中断会打断内核进程的正常调度和运行，系统对更高吞吐率的追求势必要求中断服务程序尽量短小精悍。实际中，中断需要处理程序往往不是短小的，对于一些繁重的中断服务程序，内核中断处理程序将分解为两部分：中断上半部和中断下半部。中断上半部分处理简单的紧急的功能（清除中断标志位等），大部分任务都放到下半部分处理。

## 4 GIC中断控制器详解

### 4.1 GIC版本演进和特性

**中断控制器GIC**（Generic Interrupt Controller）是中断子系统框架硬件层中的一个关键组件，用于管理和控制中断。它接收来自各种中断源的中断请求，并根据预先配置的中断优先级、屏蔽和路由规则，将中断请求分发给适当的处理器核心或中断服务例程。

GIC是由ARM公司提出设计规范，当前有四个版本，GIC V1-v4。设计规范中最常用的，有3个版本V2.0、V3.1、V4.1，GICv3版本设计主要运行在Armv8-A, Armv9-A等架构上。

> [!note]+ 重要笔记
> 
> ARM公司并给出一个实际的控制器设计参考，比如GIC-400(支持GIC v2架构)、GIC-500(支持GIC v3架构)、GIC-600(支持GIC v3和GIC v4架构)。最终芯片厂商可以自己实现GIC或者直接购买ARM提供的设计。

每个GIC版本及相应特性如下表所示：

![GIC版本对比|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3485d6f70fd2f0e6997f91ab02fda5a6.png)

![GIC版本特性|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5e59c0ab41e48b420699f1725ab27ed6.png)

从表格中我们可以看出：
- V1是最老的版本，已经被废弃了
- V2~V4目前正在大量的使用
- **GIC V2**是给ARMv7-A架构使用的，比如Cortex-A7、Cortex-A9、Cortex-A15等
- **V3和V4**是给ARMv8-A/R架构使用的，也就是64位芯片使用的

更详细的介绍可以参考 [《Arm® Generic Interrupt Controller Architecture Specification》 ](https://developer.arm.com/documentation/ihi0069/e/?lang=en)

### 4.2 GICv3基本架构

在RK3568上使用的GIC版本为GICv3，相应的中断控制器模型如下所示：

![GIC V3.0逻辑组成|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/19d8f51bac7d4f26da3cac40581fd3aa.png)

![GICv3控制器内部模块|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9860f1de52aec63dbe933ea5dde720ff.png)

GIC v3主要这几部分组成：Distributor、CPU interface、Redistributor、ITS。

GIC v3中，将cpu interface从GIC中抽离，放入到了cpu中，cpu interface通过AXI Stream，与gic进行通信。

当GIC要发送中断，GIC通过AXI stream接口，给cpu interface发送中断命令，cpu interface收到中断命令后，根据中断线映射配置，决定是通过IRQ还是FIQ管脚，向cpu发送中断。

#### 4.2.1 Distributor（中断仲裁器）

**Distributor**包含影响所有处理器核心中断的全局设置，它就像一个"中央调度员"，负责管理所有共享外设中断（`SPI`）。

Distributor提供了一些编程接口或者说是寄存器，我们可以通过Distributor的编程接口实现如下操作：

- **全局的开启或关闭CPU的中断**：这是最基本的总开关功能
- **控制任意一个中断请求的开启和关闭**：精细化控制每个中断源
- **中断优先级控制**：决定哪些中断更重要，优先处理
- **指定中断发生时将中断请求发送到那些CPU**：在多核系统中分配中断处理负载
- **interrupt属性设定**：设置每个"外部中断"的触发方式（边缘触发或者电平触发）

#### 4.2.2 Redistributor（重新分配器）

对于每个连接的处理器核心（PE），都有一个重新分配器（Redistributor）。换句话说，每个CPU核心都有自己的"专属管家"。

**Redistributor**提供以下编程接口：

- **启用和禁用SGI和PPI**：管理软件中断和私有外设中断
- **设置SGI和PPI的优先级级别**：配置私有中断的优先级
- **将每个PPI设置为电平触发或边沿触发**：控制私有中断的触发方式
- **将每个SGI和PPI分配给一个中断组**：实现中断分组管理
- **控制SGI和PPI的状态**：管理私有中断的生命周期
- **内存中数据结构的基址控制**：支持LPI的相关中断属性和挂起状态
- **电源管理支持**：配合CPU的电源状态

#### 4.2.3 CPU接口

每个重新分配器都连接到一个CPU接口。CPU接口就像处理器的"中断接收窗口"。

**CPU接口**提供以下编程接口：

- **打开或关闭CPU interface向连接的CPU assert中断事件**：控制中断处理的总开关
- **中断确认(acknowledging an interrupt)**：通知系统已经开始处理中断
- **中断处理完毕的通知**：完成中断处理流程
- **为处理器设置中断优先级掩码**：过滤低优先级中断
- **设置处理器的抢占策略**：控制中断嵌套行为
- **确定挂起的中断请求中优先级最高的中断请求**：选择最重要的待处理中断

简单来说，CPU接口可以开启或关闭发往CPU的中断请求，CPU中断开启后只有优先级高于"中断优先级掩码"的中断请求才能被发送到CPU。在任何时候CPU都可以从（CPU接口寄存器）读取当前活动的最高优先级。

> [!tip]+ 重要提示
> 
> GICv3相比GICv2的一个重要改进是引入了Redistributor，这使得每个CPU核心都有独立的私有中断管理能力，提高了多核系统的扩展性。

#### 4.2.4 ITS(Interrupt translation service)

ITS 是GIC v3架构中的一种可选硬件机制，ITS提供了一种将基于消息的中断转换为LPI的软件机制，它是 在支持LPI的配置中可选地支持

ITS接收LPI中断，进行解析，然后发送到对应的redistributor，再由redistributor将中断信息，发送给cpu interface

## 5 中断分类、ID和状态管理

### 5.1 四种中断类型

![中断类型总览|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/249733194c641d3597f9d44df3a1acef.png)

图中左侧部分就是中断源，中间部分就是GIC控制器，最右侧就是中断控制器向处理器内核发送中断信息。我们重点要看的肯定是中间的GIC部分。

GIC-V3支持四种类型的中断，分别是**SGI、PPI、SPI和LPI**，每个中断类型都有其特定的用途和特点：

**SGI（Software Generated Interrupt，软件生成中断）**：
- **用途**：处理器之间的通信，由软件触发引起的中断
- **触发方式**：通过向GIC中的寄存器GICD_SGIR写入数据来触发
- **中断号范围**：ID0 - ID15
- **典型应用**：多核系统中，一个核心通知其他核心执行某些任务
    - 内核中的IPI：inter-processor interrupts 就是基于SGI

**PPI（Private Peripheral Interrupt，私有外设中断）**：
- **用途**：针对特定处理器核心的外设中断，每个核肯定有自己独有的中断
- **特点**：不与其他处理器核心共享
- **中断号范围**：ID16 - ID31
- **典型应用**：每个CPU核心的专用定时器中断

**SPI（Shared Peripheral Interrupt，共享外设中断）**：
- **用途**：全局外设中断，可以路由到指定的处理器核心，所有核心共享的中断
- **特点**：允许多个处理器核心接收同一个中断，那些外部中断都属于SPI中断（注意！不是SPI总线那个中断）
- **中断号范围**：ID32 - ID1019
- **典型应用**：网卡、硬盘等公共外设的中断

**LPI（Locality-specific Peripheral Interrupt，特定局部外设中断）**：
- **特点**：GICv3中引入的新型中断类型
- **配置方式**：配置存储在内存表中，而不是寄存器中
- **传递方式**：总是基于消息的中断

![中断类型详细分类|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6aef36412388e3510b6d9f03209a8594.png)

> [!example]+ 示例
> 
> 在四核处理器系统中，当网卡接收到数据包时，会产生一个SPI中断。GIC可以将这个中断路由到任意一个空闲的处理器核心，实现负载均衡。

> [!info]+ 示例
> 
> 就像公司的消息通知系统一样：SPI中断是"全员通知"（任何人都可以处理），PPI中断是"个人专属消息"（只能特定人处理），SGI中断是"内部协调消息"（员工之间互相通信）。

### 5.2 中断ID分配规则

中断源有很多，为了区分这些不同的中断源肯定要给他们分配一个**唯一ID**，这些ID就是**中断ID，称为INTID**。每一个CPU最多支持1020个中断ID，中断ID号为ID0~ID1019。

这1020个ID包含了PPI、SPI和SGI，那么这三类中断是如何分配这1020个中断ID的呢？这1020个ID分配如下表：

|INTID|中断类型|说明|
|---|---|---|
|0-15|SGI|每个CPU核都有自己的16个|
|16-31|PPI|每个CPU核都有自己的16个|
|32-1019|SPI|988个ID分配给SPI，例如GPIO中<br>断、串口中断等这些外部中断，<br>具体由SOC厂商定义|
|1020-1023|---|特殊中断号|
|1024 - 8191|---|保留|
|8192-MAX|LPI||

> [!tip]+ 重要提示
> 
> 比如RK3568的外设中断ID对应的中断源可以在《Rockchip_RK3568_TRM_Part1_V1.3-20220930P》中找到详细的解释。

在RK3568参考手册的"1.3 System Interrupt Connection"小节中，中断ID如图所示（由于表太大，这里只是截取其中一部分）：

![RK3568中断ID分配|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f15b5c117db16b4c5a8263318abb2e52.png)

换句话说，每个芯片厂商都会提供这样一份"中断号码本"，告诉我们每个外设对应的具体中断ID。

> [!tip]+ 提示
> 
> - 架构理论规定的最大值：SPI是32~1019，而PPI是16~31
> - 实际RK3568对应的ID：SPI是256个，PPI是3个
> - 256个SPI已经足够覆盖GPIO、UART、SPI、I2C等所有外设中断需求

rk3568的中断控制器是GIC-600，我们简单介绍该中断控制器，GIC-600是支持GIC v3版本，是arm一个实际的控制器设计参考。rk3568的GIC流程框图如下：

![RK3568 GIC流程框图|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3c8637f0588b327b14d9fbbb305975c8.png)

![RK3568中断号分配详表|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3b4a16f1e8dce346e4796a92ca3df8cc.png)

结合rk3568的设备树，可以看下GIC-600寄存器地址：

|寄存器|地址范围|
|---|---|
|Distributor|0xfd400000~0xfd410000|
|ITS|0xfd440000~0xfd460000|
|Redistributor|0xfd460000~0xfd520000|

### 5.3 中断状态机

每个中断都维护一个状态机，支持Inactive、Pending、Active、Active and pending。中断处理的状态机如下图：

![中断状态机|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d630f80139e969c34254cdec49a50898.png)

**四种中断状态**：

- **Inactive（非活动状态）**：中断源当前未被触发，处于待命状态
- **Pending（等待状态）**：中断源已被触发，但尚未被处理器核心确认
- **Active（活动状态）**：中断源已被触发，并且已被处理器核心确认，正在处理中
- **Active and Pending（活动且等待状态）**：已确认一个中断实例，同时另一个中断实例正在等待处理

> [!warning]+ 警告或注意
> 
> Active and Pending状态表明同一个中断源在短时间内连续触发了多次，这种情况在高频中断中比较常见，需要在中断处理程序中妥善处理。

中断状态的转换反映了完整的处理流程：

1. **Inactive → Pending**：外围设备发出中断请求
2. **Pending → Active**：处理器确认并开始处理中断
3. **Active → Inactive**：处理器完成中断处理

**---------------让我们跟踪一个完整的中断处理过程---------------**
**第一步：中断发送**
- 外围设备将中断发送给分发器
- 如果中断当前状态是inactive，则切换得到pending
- 如果已经是active状态，则切换到active and pending
**第二步：中断转发**
- 分发器从所有pending状态的中断中选出优先级最高的
- 转发到目标处理器的处理器接口
- 通过接口将中断信号发送给对应的处理器
**第三步：中断确认**
- 处理器开始执行对应的中断处理程序
- 程序会先读取处理器接口的 **中断确认寄存器**，获得具体的**中断号**
- 该操作会自动将分发器中的中断状态从pending切换到active
**第四步：设备处理**
- 中断处理程序根据中断号识别是哪个设备发出的中断
- 然后调用对应设备的专用处理程序
**第五步：结束处理**
- 处理完成后，程序将中断号写入处理器接口的 **中断结束寄存器**，通知系统中断处理完毕
- 分发器收到通知后，将中断状态从active切换回inactive，或从active and pending切换到pending

![中断处理完整流程|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/501e9f30a44a28d8c6bad0b7ae1ce3ef.png)

### 5.4 中断触发方式

每个外设中断可以是以下两种类型之一：

**边沿触发（Edge-triggered）**：这是一种在检测到中断信号上升沿时触发的中断。触发后保持触发状态，直到满足清除条件。适用于瞬时事件，如按键按下。

**电平触发（Level-sensitive）**：这是一种在中断信号电平处于活动状态时触发的中断。在电平不处于活动状态时取消触发。适用于持续状态，如外设数据准备就绪。

> [!tip]+ 重要提示
> 
> 选择合适的触发方式对中断处理的可靠性至关重要。边沿触发响应快但可能丢失信号，电平触发更可靠但需要及时清除中断源。

## 6 中断号管理机制

### 6.1 虚拟中断号vs硬件中断号

在Linux内核中，我们使用**IRQ number**和**HW interrupt ID**两个ID来标识一个来自外设的中断。理解这两个概念的区别对于掌握中断系统至关重要。

**IRQ number（虚拟中断号）**：CPU需要为每一个外设中断编号，我们称之IRQ Number。这个IRQ number是一个虚拟的interrupt ID，和硬件无关。仅仅是被CPU用来标识一个外设中断。类似于"身份证号码"，全系统唯一。

**HW interrupt ID（硬件中断号）**：对于GIC中断控制器而言，它收集了多个外设的interrupt request line并`向上传递`。GIC中断控制器需要对外设中断进行编码。GIC中断控制器用HW interrupt ID来标识外设的中断。类似于"门牌号"，在`同一个控制器内唯一`。

> [!note]+ 重要笔记
> 
> 如果只有一个GIC中断控制器，那IRQ number和HW interrupt ID是可以一一对应的。但在级联情况下，就需要irq domain机制来管理映射关系。

### 6.2 中断域映射机制（irq_domain）

在GIC中断控制器级联的情况下，仅仅用HW interrupt ID就不能唯一标识一个外设中断，还需要知道该HW interrupt ID所属的GIC中断控制器（HW interrupt ID在不同的Interrupt controller上是会重复编码的）。

这样，CPU和中断控制器在标识中断上就有了一些不同的概念。对于驱动工程师而言，我们和CPU视角是一样的，我们只希望得到一个IRQ number，而不关心具体是哪个GIC中断控制器上的哪个HW interrupt ID。

为了管理这种复杂的层次结构，Linux内核引入了**中断域（irq_domain）** 概念。

简单来说，中断域就像一个"翻译器"，负责将每个中断控制器的**HW interrupt ID映射成全局唯一的IRQ number**

> [!tip]+ 重要提示
> 
> 这样一个好处是在中断相关的硬件发生变化的时候，驱动软件不需要修改。因此，Linux kernel中的中断子系统需要提供一个将HW interrupt ID映射到IRQ number上来的机制，也就是irq domain。

### 6.3 映射方法的选择

中断域支持三种不同的映射策略，每种都有其适用场景：

**线性映射**：使用固定大小的数组，`硬件中断号直接作为数组索引`。这种方式查找速度最快，适合硬件中断号数量固定且较小（<256）的场景。

```c
static inline struct irq_domain *irq_domain_add_linear(struct device_node *of_node,
                           unsigned int size,
                           const struct irq_domain_ops *ops,
                           void *host_data)
{
      return __irq_domain_add(of_node_to_fwnode(of_node), size, size, 0, ops, host_data);
}
```

- of_node：指向设备树节点的指针，用于关联中断域与具体硬件
- size：中断域支持的最大中断数量
- ops：实现hwirq到virq映射逻辑的回调函数
- host_data：指向控制器私有数据的指针

**树映射**：使用基数树数据结构保存映射关系。这种方式内存效率高，适合硬件中断号可能很大但实际使用的中断数量相对较少的场景。

```c
static inline struct irq_domain *irq_domain_add_tree(struct device_node *of_node,
                            const struct irq_domain_ops *ops,
                            void *host_data)
{
      return __irq_domain_add(of_node_to_fwnode(of_node), 0,～0, 0, ops, host_data);
}
```

**不映射**：适用于功能强大的中断控制器，这些控制器的硬件中断号可以直接配置为Linux中断号，无需额外映射。例如PowerPC架构的MPIC控制器。

```c
static inline struct irq_domain *irq_domain_add_nomap(struct device_node *of_node,
                            unsigned int max_irq,
                            const struct irq_domain_ops *ops,
                            void *host_data)
{
      return __irq_domain_add(of_node_to_fwnode(of_node), 0, max_irq, max_irq, ops, host_data);
}
```

> [!tip]+ 实现细节 
> 所有分配函数最终都会调用`__irq_domain_add()`来完成实际工作：分配irq_domain结构体、初始化成员变量，并将新的中断域添加到全局链表irq_domain_list中。

### 6.4 映射操作

创建中断域后，需要建立具体的映射关系：

**创建映射**：使用`irq_create_mapping()`函数像中断域添加新的映射关系
```c
unsigned int irq_create_mapping(struct irq_domain  *domain, irq_hw_number_t hwirq);
```
- 该函数接收中断域和硬件中断号作为参数，返回分配的Linux中断号
- 内部流程是**先分配一个全局唯一的Linux中断号**，然后在指定的中断域中建立硬件中断号到Linux中断号的映射关系

**查找映射**：中断处理过程中需要根据hwirq快速找到virq，使用`irq_find_mapping()`函数
```c
unsigned int irq_find_mapping(struct irq_domain *domain, irq_hw_number_t hwirq);
```
- 这个函数是中断处理的关键环节，它接收中断域和硬件中断号，返回对应的Linux中断号，让系统能够正确调用相应的中断处理程序

## 7 总结

通过这个章节的学习，我们全面了解了Linux中断子系统的框架结构：

> [!abstract]+ 摘要或总结
> 
> **核心知识掌握**：
> 
> - 理解了中断子系统四层架构的设计思想和各层职责
> - 掌握了GICv3中断控制器的组成和工作原理
> - 学习了四种中断类型的特点和应用场景
> - 了解了中断状态机的工作流程和状态转换
> - 掌握了中断号管理和irq domain映射机制
> - 认识了硬件抽象在系统设计中的重要作用

中断子系统框架为Linux提供了统一、高效的中断处理机制。理解这个框架有助于我们在后续的驱动开发中正确使用中断功能，编写出高质量的设备驱动程序。这种分层设计不仅保证了系统的稳定性和可移植性，也为开发者提供了清晰的开发接口和调试思路。

> [!question]+ 常见问题
> 
> **Q**: 为什么需要虚拟中断号和硬件中断号两套编号系统？ 
> **A**: 虚拟中断号提供了硬件抽象，使驱动程序可以在不同硬件平台间移植。当硬件发生变化时，只需要修改底层映射关系，而不需要修改驱动代码。这种设计让同一个驱动程序可以在不同的ARM芯片上运行，大大提高了代码的复用性和系统的可维护性。