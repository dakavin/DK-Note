
在前两篇文章中，我们了解了中断的基本概念和ARM异常处理机制。现在让我们从更高的视角来理解整个**中断子系统框架**的组织结构。

简单来说，Linux中断子系统就像一座"四层大厦"，每一层都有明确的职责分工。从最底层的硬件信号，到最顶层的驱动程序，这个框架确保了中断处理的高效性和可移植性。

> [!note]+ 重要笔记
> 
> 理解中断子系统的分层架构，有助于我们在驱动开发中准确定位问题，并编写出高质量的中断处理代码。

## 1 中断子系统四层架构

### 1.1 整体架构概览

一个完整的中断子系统框架可以分为四个层次，由上到下分别为**用户层、通用层、硬件相关层和硬件层**。这种分层设计的核心思想是实现硬件抽象和代码复用。

![中断子系统四层架构|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f283f44413998866941163494dab0058.png)

换句话说，就像盖房子一样，每一层都有自己的功能，上层不需要关心下层的具体实现细节，只需要通过标准接口进行交互。这种设计让Linux能够运行在各种不同的硬件平台上，而驱动程序代码却可以保持高度的一致性。

### 1.2 各层详细功能

让我们从上到下详细了解每一层的具体功能和作用：

**用户层（驱动程序层）**：
- 中断的使用者，主要包括各类设备驱动
- 这些驱动程序通过中断相关的接口进行中断的申请和注册
- 当外设触发中断时，用户层驱动程序会进行相应的回调处理，执行特定的操作

就像住在房子里的居民，他们使用房子提供的各种设施（水电、暖气等），但不需要知道这些设施的内部工作原理。

> [!example]+ 示例
> 
> 网卡驱动程序就是典型的用户层，它注册网卡中断处理函数，当网卡接收到数据包时，通过中断通知驱动程序及时处理数据。

**通用层（框架层）**：
- 也可称为框架层，它是硬件无关的层次
- 通用层的代码在所有硬件平台上都是通用的，不依赖于具体的硬件架构或中断控制器
- 通用层提供了统一的接口和功能，用于管理和处理中断，使得驱动程序能够在不同的硬件平台上复用

换句话说，通用层就像房子的标准设施接口（标准的电源插座、水龙头接口），无论房子建在哪里，居民都能以同样的方式使用这些设施。

**硬件相关层**：包含两部分代码
- 一部分是与`特定处理器架构`相关的代码，比如ARM64处理器的中断处理相关代码。这些代码负责处理特定架构的中断机制，包括中断向量表、中断处理程序等。
- 另一部分是`中断控制器`的驱动代码，用于与中断控制器进行通信和配置。这些代码与具体的中断控制器硬件相关

也就是说，这一层就像房子的"适配器层"，负责将标准接口适配到具体的硬件设施上。

**硬件层**：
- 硬件层位于最底层，与具体的硬件连接相关。
- 它包括外设与SoC（系统片上芯片）的物理连接部分。
- 中断信号从外设传递到中断控制器，由中断控制器统一管理和路由到处理器。

简单来说，这就是房子的基础设施层，包括实际的水管、电线、暖气管道等物理设施。

> [!tip]+ 重要提示
> 
> 这种分层设计的最大优势是**代码复用性**。比如，同一个网卡驱动程序可以在不同的ARM平台上运行，而不需要修改上层代码。

## 2 硬件与软件的接口

### 2.1 中断硬件组成

中断硬件主要有三种器件参与，各个外设、中断控制器和CPU。它们之间的关系可以用下面的图来理解：

![中断硬件组成关系](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/17b9a2e0b0a52a49b229e2b144385171.png)

> [!tip]+ 提示 
> 在ARMv8体系结构中，将处理器处理事务的抽象过程定义为PE(Processing Element)，可以将PE简单理解为处理器核心

让我们用一个邮政系统的类比来理解这三个组件的关系：
- **外设**：在发生中断事件的时候，通过外设上的电气信号向中断控制器请求处理。就像各个部门有事情需要汇报时，会向邮政总局发送邮件。

- **中断控制器**：由于中断数量越来越多，需要一个专门设备来管理中断，中断控制器就是起这个作用，它连接外设中断系统和CPU系统的桥梁，接收外部中断，处理后向处理器上报中断信号。以ARM为例，GIC中断控制器，外部中断信号到达后，经过处理标记为FIQ、IRQ等，然后传输给ARM内核。

换句话说，中断控制器就像邮政系统的分拣中心，负责接收各个部门的邮件，然后根据重要程度和目标进行分类，最后送达给相应的接收人。

- **CPU**：CPU的主要功能是运算，CPU并不处理中断优先级，只是接收中断控制器的中断信息，运行中断处理。例如：对于ARM，CPU会接收到IRQ和FIQ信号，分别让ARM进入IRQ mode和FIQ mode。

也就是说，CPU就像公司的领导，专注于核心决策工作，当有重要事务需要处理时，由秘书（中断控制器）进行初步分类和安排。

### 2.2 软件框架

Linux内核软件部分，中断子系统分成4个部分：

**硬件无关的代码**：我们称之为`内核通用中断处理模块`。无论是哪种CPU，哪种控制器，其中断处理的过程都有一些相同的内容，这些相同的内容被抽象出来，和硬件无关。此外，**各个外设的驱动代码中，用一个统一的接口实现irq相关的管理，这些代码组成了内核中断子系统的核心部分**。

**CPU架构相关的中断处理**：和系统使用的具体的CPU架构相关。不同的CPU架构（比如ARM64、x86）有不同的中断处理机制和指令集。

**中断控制器驱动代码**：和系统使用的中断控制器相关。不同的芯片厂商可能使用不同类型的中断控制器。

**普通其他驱动**：这些驱动将使用内核通用中断处理模块的API来实现自己的驱动逻辑。

换句话说，这种软件分层设计确保了代码的模块化和可移植性，就像标准化的工业生产流程一样，每个环节都有明确的职责和接口。

## 3 重要概念说明

在深入理解中断子系统架构之前，我们需要掌握几个重要概念。这些概念就像理解一门外语需要先掌握关键词汇一样重要。

### 3.1 硬件中断ID与虚拟中断号

**HW interrupt ID**：硬件中断ID，就是`GIC硬件上的irq ID`，是GIC标准定义的，这是硬件层面的真实编号。

**IRQ number**：IRQ number是软件方面定义，和硬件无关，CPU需要为每一个外设中断编号，我们称之IRQ Number。这个IRQ number是一个`虚拟的interrupt ID`，仅仅是被CPU用来标识一个外设中断。

简单来说，硬件中断ID就像设备的"出厂编号"，而虚拟中断号就像系统内部的"管理编号"。系统需要建立这两者之间的映射关系，这样硬件发出的中断信号就能正确地传递给对应的软件处理程序。

换句话说，就像每个员工既有身份证号（硬件ID）又有工号（虚拟ID）一样，身份证号是国家统一分配的，而工号是公司内部管理使用的。

### 3.2 IRQ domain

irq domain 翻译过来就是"irq 域"的意思，在内核中是一个比较常见的概念，也就是将某一类资源划分成不同的区域，相同的域下共享一些共同的属性。domain实际上也是对模块化的一种体现，`irq domain的产生是因为GIC对于级联的支持`。实际上，虽然内核对每个GIC都设立了不同的域，但是它只做了一件事：**负责GIC中hwirq和逻辑irq的映射**。

换句话说，irq domain就像一个"翻译官"，负责将硬件中断号翻译成Linux系统能理解的虚拟中断号。也就是说，当系统中有多个中断控制器时，每个控制器都有自己的"翻译官"，确保不会出现"沟通混乱"的情况。

### 3.3 中断向量表（irq_descs数组）

中断向量表就是中断向量的列表，中断向量表在内存中存放，表中存放着中断源所对应的中断处理程序的入口地址。就像一个电话簿，当有中断来电时，系统根据电话号码（中断号）查找对应的接听人（处理程序）。

简单来说，中断向量表建立了"中断类型"到"处理程序地址"的映射关系，让系统能够快速找到合适的处理程序。

### 3.4 中断处理架构

中断会打断内核进程的正常调度和运行，系统对更高吞吐率的追求势必要求中断服务程序尽量短小精悍。实际中，中断需要处理的程序往往不是短小的，对于一些繁重的中断服务程序，内核中断处理程序将分解为两部分：中断上半部和中断下半部。中断上半部分处理简单的紧急的功能（清除中断标志位等），大部分任务都放到下半部分处理。

也就是说，这种架构设计就像医院的急诊分诊制度：先进行快速初步检查（上半部），确定病情类型和紧急程度，然后安排到相应科室进行详细治疗（下半部）。

## 4 上下半部设计思想

### 4.1 设计思想

中断子系统中有一个重要的设计机制，那就是 **Top-half(上半部)** 和 **Bottom-half(下半部)**。

这个设计的核心理念是：将紧急的工作放置在Top-half中来处理，而将耗时的工作放在Bottom-half中来处理，确保Top-half能尽快处理完成。

简单来说，就像医院的急诊科：

- **上半部**：快速诊断，确定病情，给出紧急处理
- **下半部**：详细治疗，康复护理等耗时工作

或者说，当键盘按下按键时会触发中断：

- **上半部**：处理中断，消除中断标志
- **下半部**：获取按键值，进行后续处理

换句话说，这种设计让系统能够快速响应紧急事件，同时不影响对其他事件的及时处理。

### 4.2 对比串行处理和并行处理

先看看没有下半部机制时会发生什么，**无下半部中断（串行处理）**：

![串行处理问题](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3b6cf4d94188880663bad511e485b03c.png)

> [!warning]+ 警告或注意 
> 上图显示了串行处理的问题：每个中断必须完全处理完毕后才能处理下一个中断，可能导致中断丢失。

**有下半部中断（并行处理）**：

![并行处理优势](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e905c8ac28f851a12384442f0d71338d.png)

> [!success]+ 成功或完成 
> 上图展示了并行处理的优势：上半部快速处理后立即开放中断，下半部可以与其他中断并行执行。

### 4.3 为什么需要上下半部机制

**ARM处理器的中断特性**：
- ARM处理器在进行中断处理时，处理器进入异常模式切换，此时会将`中断进行关闭`
- `处理完成后再将中断打开`

**不分上下半部的问题**：
- 如果中断不分上下半部，那么意味着只有等上一个中断完成处理后才会打开中断
- 下一个中断才能得到处理
- 当某个中断处理时间较长时，很有可能就导致其他中断丢失而无法响应

> [!danger]+ 危险或错误 
> 这个问题显然是难以接受的，比如典型的时钟中断，作为系统的脉搏，它的响应就需要得到保障。

也就是说，没有上下半部机制的系统就像一个只能同时处理一件事的服务窗口，如果前面的客户办理业务很慢，后面的客户就只能无限等待。

### 4.4 工作原理

**优势说明**：
- 中断分成上下半部处理可以提高中断的响应能力
- 在上半部处理完成后便将中断打开（通常上半部处理越快越好）
- 这样就可以响应其他中断了，等到中断退出的时候再进行下半部的处理

**实现机制**： 中断的**Bottom-half机制**，包括了：
- **softirq**：软中断机制
- **tasklet**：基于softirq实现的任务机制
- **workqueue**：工作队列机制
- **中断线程化处理**：将中断处理程序运行在内核线程中

> [!note]+ 重要笔记 
> 其中tasklet又是基于softirq来实现的。这些机制我们在后续章节中会详细学习。

**上半部特点**： 中断上半部是中断服务程序的第一部分，它主要处理一些紧急且需要快速响应的任务。中断上半部的特点是执行时间较短，旨在尽快完成对中断的处理。这些任务可能包括保存寄存器状态、更新计数器等，以便在中断处理完成后能够正确地返回到中断前的执行位置。

换句话说，上半部就像急救医生的初步处理：止血、固定伤骨、稳定生命体征，然后立即将病人转送到相应科室进行详细治疗。

## 5 中断处理基本原则

### 5.1 设计原则

在Linux中断子系统的设计中，遵循了几个重要的基本原则，这些原则确保了系统的稳定性和性能：

**及时性原则**：
- 中断必须能够及时响应，不能让硬件等待太久
- 上半部处理必须尽可能快速，通常在微秒级别完成
- 就像火警警报响起时，必须立即响应，不能等到方便的时候再处理

**最小化原则**：
- 在中断上下文中只做最必要的工作
- 复杂的处理逻辑推迟到下半部或进程上下文中进行
- 就像急救车上只做维持生命的基本处理，复杂手术要到医院进行

**原子性原则**：
- 中断处理过程中不能被调度出去
- 不能在中断处理程序中使用可能导致睡眠的函数
- 就像消防队救火时不能中途去做别的事情

**可重入性原则**：
- 中断处理程序必须设计成可重入的
- 同一个中断处理程序可能在不同CPU上同时执行
- 需要使用适当的同步机制保护共享数据

### 5.2 性能优化原则

**缓存友好**：
- 中断处理程序应该尽量减少内存访问
- 频繁访问的数据结构应该设计为缓存友好的
- 就像经常使用的工具应该放在触手可及的地方

**CPU亲和性**：
- 中断处理应该考虑CPU亲和性
- 尽量在同一个CPU上处理相关的中断
- 减少跨CPU的数据传输开销

**批处理优化**：
- 对于高频中断，考虑使用批处理技术
- 一次性处理多个中断事件，提高效率
- 就像批量处理邮件比逐一处理更高效

## 6 各层接口定义

### 6.1 用户层接口

用户层（驱动程序）主要通过以下接口与中断子系统交互：

**中断申请和释放**：
```c
// 申请中断
int request_irq(unsigned int irq, irq_handler_t handler, 
                unsigned long flags, const char *name, void *dev);

// 释放中断
void free_irq(unsigned int irq, void *dev);
```

**中断控制**：
```c
// 禁用中断
void disable_irq(unsigned int irq);

// 启用中断
void enable_irq(unsigned int irq);

// 禁用本地CPU中断
unsigned long local_irq_save(flags);

// 恢复本地CPU中断
void local_irq_restore(unsigned long flags);
```

### 6.2 通用层接口

通用层提供了硬件无关的中断管理接口：

**IRQ域管理**：
```c
// 创建IRQ域
struct irq_domain *irq_domain_add_linear(struct device_node *of_node,
                                        unsigned int size,
                                        const struct irq_domain_ops *ops,
                                        void *host_data);

// 创建中断映射
unsigned int irq_create_mapping(struct irq_domain *domain, 
                                irq_hw_number_t hwirq);
```

**中断描述符管理**：
```c
// 设置中断处理函数
void irq_set_handler(unsigned int irq, irq_flow_handler_t handle);

// 设置中断芯片
void irq_set_chip(unsigned int irq, struct irq_chip *chip);
```

### 6.3 硬件相关层接口

硬件相关层需要实现特定的接口来适配不同的中断控制器：

**中断控制器接口**：
```c
struct irq_chip {
    const char *name;
    void (*irq_mask)(struct irq_data *data);
    void (*irq_unmask)(struct irq_data *data);
    void (*irq_ack)(struct irq_data *data);
    void (*irq_eoi)(struct irq_data *data);
    // ... 其他方法
};
```

**IRQ域操作接口**：
```c
struct irq_domain_ops {
    int (*match)(struct irq_domain *d, struct device_node *node);
    int (*map)(struct irq_domain *d, unsigned int virq, irq_hw_number_t hw);
    void (*unmap)(struct irq_domain *d, unsigned int virq);
    // ... 其他操作
};
```

### 6.4 接口设计的优势

这种分层接口设计带来了显著的优势：
- **代码复用**：上层代码可以在不同硬件平台间复用 
- **维护简化**：每层只需要关注自己的职责 
- **扩展性好**：新增硬件支持只需要实现底层接口 
- **调试友好**：问题可以精确定位到具体的层次

换句话说，这种接口设计就像标准化的工业接口，确保了不同厂商的组件能够良好协作。

## 7 总结

通过这个章节的学习，我们全面了解了Linux中断子系统的架构设计：

**核心知识掌握**：
- 理解了中断子系统四层架构的设计思想和各层职责
- 掌握了硬件与软件接口的组织方式
- 学习了重要概念：硬件中断ID、虚拟中断号、IRQ domain等
- 了解了上下半部机制的设计原理和工作方式
- 认识了中断处理的基本原则和各层接口定义

**关键理解要点**：
- 分层架构实现了硬件抽象和代码复用
- 上下半部机制平衡了响应性和处理能力
- 统一的接口设计确保了系统的可维护性
- 基本原则指导了高质量中断处理程序的编写

Linux中断子系统的架构设计体现了优秀软件系统的特点：层次清晰、职责分明、接口统一、扩展性强。这种设计为Linux在各种硬件平台上的成功运行提供了坚实的基础。

在掌握了中断子系统的架构原理后，我们就可以进一步学习具体的中断控制器（如GIC）的实现细节，以及如何在实际的驱动程序中正确使用这些接口。

> [!question]+ 常见问题  
> **Q**: 为什么要设计这么多层次，直接让驱动程序操作硬件不是更简单吗？  
> **A**: 分层设计的**核心价值在于抽象和复用**。如果驱动程序直接操作硬件，那么同一个驱动就需要为每种硬件平台编写不同的版本，维护成本极高。而通过分层设计，一个网卡驱动可以在ARM、x86、MIPS等各种平台上运行，大大降低了开发和维护成本。这就像标准化的螺丝接口，不需要为每个品牌的螺丝刀设计专门的螺丝。