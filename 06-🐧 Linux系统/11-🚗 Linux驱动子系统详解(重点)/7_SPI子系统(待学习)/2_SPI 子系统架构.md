
在前一篇文章中，我们了解了SPI协议的基础理论和硬件原理。现在让我们从"硬件接口"的世界进入"软件抽象"的世界。想象一下，如果说SPI硬件协议是两个人之间的对话规则，那么Linux SPI子系统就像是一个完整的通信管理系统，它不仅管理这些对话规则，还要协调多个参与者、处理消息队列、维护通信状态等等。

换句话说，Linux内核需要将复杂的SPI硬件操作抽象成统一的软件接口，让驱动开发者能够方便地使用SPI功能，而不需要关心底层硬件的具体实现细节。这就像是将复杂的机械操作包装成简单的按钮，用户只需要按一下按钮，就能完成复杂的操作。

## 1 Linux SPI抽象模型

### 1.1 三层架构设计

就像现代企业的管理结构一样，Linux SPI子系统采用了分层管理的思想。想象一个大型公司：最上层是用户界面（给客户使用），中间层是业务管理（处理具体业务逻辑），最底层是技术实现（具体的技术工作）。

![Linux系统中SPI子系统的分层架构图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/43d1b535ddd710bbce73ec00c2087f5a.png)

上图展示了从用户空间应用程序，经过spidev驱动、SPI核心层、SPI控制器驱动，最终到达SPI硬件控制器和外设的完整软件栈结构。

这个三层架构包括：

**用户空间层**：

- **应用程序**：用户编写的业务逻辑程序，就像是公司的客户
- **spidev用户态接口**：提供标准的文件操作接口（open、read、write等），就像是客户服务窗口

**内核空间层**：

- **SPI设备驱动**：针对特定SPI设备的驱动程序，就像是专门的业务专员
- **SPI核心层**：提供统一的SPI操作接口和管理功能，就像是公司的管理层
- **SPI控制器驱动**：管理具体的SPI硬件控制器，就像是技术部门

**硬件层**：

- **SPI控制器**：主机端的SPI硬件控制器，就像是公司的生产设备
- **SPI设备**：连接到SPI总线上的外设，就像是生产线上的各种工具

> [!tip]+ 实用技巧 这种分层设计的好处是各层职责明确，上层不需要关心下层的具体实现，下层为上层提供统一的接口，大大简化了开发难度

### 1.2 核心组件关系

在SPI子系统中，有四个核心组件，它们就像是一个乐团中的不同角色：

![Linux内核SPI子系统的完整架构图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/08a649ebbcf7fc89794542a925ada9b5.png)

上图展示了从SPI控制器注册、设备管理、消息队列处理到底层硬件传输的整个数据流和控制流程，包括工作队列机制和hook函数的调用关系。

**四个核心组件的关系**：

- **`spi_master`**（`spi_controller`）：对SoC的SPI控制器的抽象，就像乐团指挥
- **`spi_bus_type`**：spi的bus_type，代表了硬件上的SPI Bus，就像音乐厅的舞台
- **`spi_device`**：spi从设备，就像乐团中的乐器
- **`spi_driver`**：spi具体设备的驱动，就像乐器的演奏者

换句话说，这四个组件共同构成了SPI子系统的核心架构：控制器管理硬件资源，总线提供匹配机制，设备描述硬件属性，驱动实现具体功能。

### 1.3 与平台模型关系

让我们将SPI子系统与Linux设备模型进行对比，就像是比较两种不同的交通管理系统：

![SPI驱动框架图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ad2d660024588d2080023ebd91448f34.png)

如框架图所示，spi可分为spi总线驱动和spi设备驱动。spi总线驱动已经由芯片厂商提供，我们适当了解其实现机制。而spi设备驱动由我们自己编写，则需要明白其中的原理。

**SPI框架的层次关系**：

- **SPI核心层**：提供SPI控制器驱动和设备驱动的注册方法、注销方法，SPI Core提供操作接口函数，允许一个spi master，spi driver 和spi device初始化时在SPI Core中进行注册，以及推出时进行注销，也提供上层API接口
    
- **SPI主机驱动**：也就是spi控制器驱动(SPI Master Driver)，主要包含SPI硬件体系结构中适配器(spi控制器)的控制，实现spi总线的硬件访问操作
    
- **SPI设备驱动**：对应于spi设备端的驱动程序，通过SPI主机驱动与CPU交换数据
    

简单来说，这就像是一个三层管理结构：核心层负责统一管理，主机驱动负责硬件操作，设备驱动负责具体功能实现。

### 1.4 总线架构完整图

为了更好地理解整个SPI子系统的架构，让我们看看完整的数据结构关系图：

![SPI总线架构完整图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ee61426884e9c8915d5bfcd31acaaeb3.png)

这个完整的关系图展示了：

- SPI控制器如何管理多个设备
- 设备和驱动如何通过总线进行匹配
- 数据传输如何通过消息队列进行管理
- 各个组件之间的调用关系

也就是说，整个SPI子系统形成了一个完整的生态系统，每个组件都有明确的职责和接口。

## 2 SPI子系统初始化

### 2.1 spi_init函数

想象一下开办一所学校的过程：首先要建设基础设施，然后注册学校，再设立各个院系，最后配备管理人员。Linux SPI子系统的初始化过程也是这样一步步进行的。

**spi_init函数**：SPI子系统初始化函数，负责建立整个SPI框架的基础设施

```c
// drivers/spi/spi.c
static int __init spi_init(void)
{
    int status;                                          // 状态返回值

    // 分配SPI缓冲区内存
    buf = kmalloc(SPI_BUFSIZ, GFP_KERNEL);             // 分配全局缓冲区
    if (!buf) {
        status = -ENOMEM;                                // 内存分配失败
        goto err0;
    }

    // 注册SPI总线类型
    status = bus_register(&spi_bus_type);               // 注册SPI总线
    if (status < 0)
        goto err1;

    // 注册SPI主控制器设备类
    status = class_register(&spi_master_class);         // 注册主设备类
    if (status < 0)
        goto err2;

    // 如果启用了SPI从设备支持，注册从设备类
    if (IS_ENABLED(CONFIG_SPI_SLAVE)) {
        status = class_register(&spi_slave_class);      // 注册从设备类
        if (status < 0)
            goto err3;
    }

    // 如果启用了设备树动态配置支持
    if (IS_ENABLED(CONFIG_OF_DYNAMIC))
        WARN_ON(of_reconfig_notifier_register(&spi_of_notifier));
    // 如果启用了ACPI支持
    if (IS_ENABLED(CONFIG_ACPI))
        WARN_ON(acpi_reconfig_notifier_register(&spi_acpi_notifier));

    return 0;                                            // 初始化成功

err3:
    class_unregister(&spi_master_class);                // 注销主设备类
err2:
    bus_unregister(&spi_bus_type);                      // 注销总线类型
err1:
    kfree(buf);                                         // 释放缓冲区
    buf = NULL;
err0:
    return status;                                       // 返回错误状态
}
```

![SPI子系统初始化流程图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/133caefb4a489a598dcc3e9cb82a2666.png)

这个初始化过程就像是搭建一座大楼的地基：

1. 首先分配基础资源（内存缓冲区）
2. 然后注册总线类型（建立通信框架）
3. 接着注册设备类（建立管理体系）
4. 最后配置可选功能（设备树和ACPI支持）

### 2.2 总线注册过程

总线注册就像是在城市里修建一条新的公交线路，需要向交通管理部门登记，设立站点，制定运行规则。

**spi_bus_type结构体**：SPI总线类型结构体定义，描述SPI总线的基本属性和操作方法

```c
// drivers/spi/spi.c
struct bus_type spi_bus_type = {
    .name        = "spi",                               // 总线名称为"spi"
    .dev_groups  = spi_dev_groups,                      // 设备属性组
    .match       = spi_match_device,                    // 设备匹配函数
    .uevent      = spi_uevent,                          // 设备事件处理函数
};
EXPORT_SYMBOL_GPL(spi_bus_type);                       // 导出符号供GPL模块使用
```

![总线注册过程图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e5f738b861896cea5c09fc1524e9f010.png)

- `name`：总线名称，在/sys/bus/目录下可以看到
- `dev_groups`：设备属性组，定义了在sysfs中导出的设备属性
- `match`：设备和驱动的匹配函数，决定哪个驱动可以处理哪个设备
- `uevent`：设备事件处理函数，处理设备的热插拔事件

### 2.3 设备类创建

设备类的创建就像是建立学校的院系结构，每个院系管理相应的专业和学生。

在`spi_init`函数中，会创建以下设备类：

**主控制器类注册**：

```c
status = class_register(&spi_master_class);
```

**从设备类注册**（如果启用）：

```c
if (IS_ENABLED(CONFIG_SPI_SLAVE)) {
    status = class_register(&spi_slave_class);
}
```

这些设备类创建后，会在/sys/class/目录下生成对应的目录：

- `/sys/class/spi_master/` - SPI主控制器类
- `/sys/class/spi_slave/` - SPI从设备类（如果启用）

### 2.4 核心层注册

Linux系统在开机的时候就会执行spi_init函数，自动进行spi总线注册：

```c
// drivers/spi/spi.c
static int __init spi_init(void)
{
    int status;
    // ...
    status = bus_register(&spi_bus_type);              // 注册SPI总线
    // ...
    status = class_register(&spi_master_class);        // 注册主设备类
    // ...
}
```

当总线注册成功之后，会在`/sys/bus`下面生成一个spi总线，然后在系统中新增一个设备类，`/sys/class/`目录下会可以找到spi_master类。

简单来说，这个过程就是告诉Linux内核："我们有一种叫做SPI的总线类型，请为它建立管理框架"。

## 3 SPI总线模型

### 3.1 总线定义分析

想象一下火车站的运营系统：有固定的铁轨（总线），有不同的列车（设备），有列车调度员（驱动），还有一套完整的调度规则。SPI总线模型就是这样一套完整的管理系统。

### 3.2 总线定义详解

**spi_bus_type总线定义**：定义SPI总线的核心属性和行为

```c
// drivers/spi/spi.c
struct bus_type spi_bus_type = {
    .name           = "spi",                            // 总线名称
    .dev_groups     = spi_dev_groups,                   // 设备组属性
    .match          = spi_match_device,                 // 设备匹配函数
    .uevent         = spi_uevent,                       // uevent事件处理函数
};
```

- `name`：总线的名称，这是总线在系统中的唯一标识
- `dev_groups`：设备属性组，定义了设备在sysfs中的属性文件
- `match`：匹配函数指针，设定了spi设备和spi驱动的匹配规则
- `uevent`：事件处理函数，处理设备的插入和移除事件

换句话说，这个结构体就是SPI总线的"身份证"，定义了它的基本信息和能力。

### 3.3 匹配机制详解

设备和驱动的匹配就像是相亲过程，需要通过多个条件来判断是否合适。SPI子系统提供了多种匹配方式，就像是多个相亲标准。

![设备驱动匹配流程图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/64c3b24075a06d806b7ad3c3d60cad28.png)

**spi_match_device函数**：SPI设备和驱动匹配函数，实现多种匹配策略

```c
// drivers/spi/spi.c
static int spi_match_device(struct device *dev, struct device_driver *drv)
{
    const struct spi_device *spi = to_spi_device(dev);     // 获取SPI设备
    const struct spi_driver *sdrv = to_spi_driver(drv);    // 获取SPI驱动

    /* Check override first, and if set, only use the named driver */
    // 检查是否有驱动覆盖设置，如果有则只使用指定的驱动
    if (spi->driver_override)
        return strcmp(spi->driver_override, drv->name) == 0;

    /* Attempt an OF style match */
    // 尝试设备树风格的匹配
    if (of_driver_match_device(dev, drv))
        return 1;

    /* Then try ACPI */
    // 然后尝试ACPI匹配
    if (acpi_driver_match_device(dev, drv))
        return 1;

    // 如果驱动有ID表，使用ID表进行匹配
    if (sdrv->id_table)
        return !!spi_match_id(sdrv->id_table, spi);

    // 最后比较modalias字符串
    return strcmp(spi->modalias, drv->name) == 0;
}
```

### 3.4 匹配函数分析

这个匹配函数就像是一个多层筛选系统，按照优先级依次尝试不同的匹配方式：

1. **driver_override匹配**：最高优先级，如果设备指定了特定驱动，就只用这个驱动
2. **设备树匹配**：通过设备树节点的compatible属性进行匹配
3. **ACPI匹配**：通过ACPI表格进行匹配
4. **ID表匹配**：通过驱动的ID表进行匹配
5. **名称匹配**：最后通过设备名称进行匹配

### 3.5 匹配优先级

函数提供了四种匹配方式，设备树匹配方式和acpi匹配方式以及id_table匹配方式，如果前面三种都没有匹配成功，则通过设备名进行配对。

**匹配优先级顺序**：

1. **驱动覆盖**：强制指定驱动（优先级最高）
2. **设备树匹配**：OF风格匹配（推荐方式）
3. **ACPI匹配**：ACPI表格匹配（x86平台常用）
4. **ID表匹配**：传统ID表匹配
5. **名称匹配**：通过modalias匹配（最后选择）

当设备树匹配或acpi匹配成功时，优先使用这些方式，因为它们能提供更丰富的设备信息。

> [!tip]+ 实用技巧 在实际开发中，推荐使用设备树匹配方式，因为它能提供设备的详细配置信息，包括引脚配置、时钟频率等参数

## 4 核心数据结构

### 4.1 spi_controller详解

想象一下学校的教务处，它负责管理所有的教学资源、协调各个班级的课程安排、处理学生的各种申请。`spi_controller`就是SPI世界中的"教务处"，负责管理整个SPI总线的运行。

实际上，`spi_master`就是`spi_controller`结构体的别名：

```c
// include/linux/spi/spi.h
#define spi_master  spi_controller
```

这个设计是为了保持向后兼容性，就像是公司改名后，旧的招牌还能继续使用一样。

### 4.2 spi_master兼容层

在内核版本4.19中，为了保持兼容性，`spi_master`被定义为`spi_controller`的别名：

```c
// include/linux/spi/spi.h 内核版本4.19
#define spi_master  spi_controller
```

这种设计确保了老代码在新内核中仍然可以正常工作，就像是给老式插头配了一个转换器。

### 4.3 spi_controller完整分析

**struct spi_controller**：SPI控制器结构体，是整个SPI子系统的核心管理者

```c
// include/linux/spi/spi.h
struct spi_controller {
    struct device                          dev;                   // 设备结构体基础
    struct list_head                       list;                  // 控制器链表节点
    s16                                    bus_num;               // SPI控制器编号
    u16                                    num_chipselect;        // 片选信号个数
    struct spi_message                     *cur_msg;              // 当前处理的消息
    int                                    (*setup)(struct spi_device *spi);           // 设备设置函数
    int                                    (*transfer)(struct spi_device *spi,         // 传输函数
                                                      struct spi_message *mesg);
    void                                   (*cleanup)(struct spi_device *spi);        // 清理函数
    struct kthread_worker                  kworker;               // 内核线程工人
    struct task_struct                     *kworker_task;         // 工作线程任务
    struct kthread_work                    pump_messages;         // 消息处理工作
    struct list_head                       queue;                 // 消息队列
    int                                    (*transfer_one)(struct spi_controller *ctlr,    // 单次传输函数
                                                           struct spi_device *spi,
                                                           struct spi_transfer *transfer);
    int                                    (*prepare_transfer_hardware)(struct spi_controller *ctlr);  // 硬件准备函数
    int                                    (*transfer_one_message)(struct spi_controller *ctlr,        // 消息传输函数
                                                                   struct spi_message *mesg);
    void                                   (*set_cs)(struct spi_device *spi, bool enable);             // 片选控制函数
    int                                    *cs_gpios;             // 片选GPIO数组
};
```

**核心成员详解**：

- `dev`：内嵌的设备结构体，用于集成到Linux设备模型中
- `list`：链表节点，芯片可能有多个spi控制器，通过链表管理
- `bus_num`：spi控制器编号，用于区分不同的SPI控制器
- `num_chipselect`：spi片选信号的个数，对不同的从设备进行区分
- `cur_msg`：当前正在处理的消息队列，表示控制器当前的工作状态
- `setup`：设备设置函数，用于配置SPI设备的工作参数
- `transfer`：用于把数据加入控制器的消息队列中
- `cleanup`：当spi_master被释放的时候，完成清理工作
- `kworker`：内核线程工人，spi可以使用异步传输方式发送数据
- `pump_messages`：具体传输工作的处理函数
- `queue`：所有等待传输的消息队列挂在该链表下
- `transfer_one_message`：发送一个spi消息，类似I2C适配器里的algo->master_xfer，产生spi通信时序
- `cs_gpios`：记录spi上具体的片选信号GPIO数组

### 4.4 spi_device详解

就像每个学生都有一张学生证记录个人信息一样，每个SPI设备都有一个`spi_device`结构体记录设备的详细信息。

**struct spi_device**：代表SPI总线上的一个设备，包含设备的配置参数、状态信息和统计数据

```c
// include/linux/spi/spi.h
struct spi_device {
    struct device                          dev;                   // 标准设备结构体
    struct spi_controller                  *controller;           // 所属的SPI控制器
    struct spi_controller                  *master;               // 兼容性层，指向控制器
    u32                                    max_speed_hz;          // 最大通信频率(Hz)
    u8                                     chip_select;           // 片选信号编号
    u8                                     bits_per_word;         // 每个字的位数
    u16                                    mode;                  // SPI通信模式配置
    int                                    irq;                   // 中断号
    void                                   *controller_state;     // 控制器私有状态数据
    void                                   *controller_data;      // 控制器私有数据
    char                                   modalias[SPI_NAME_SIZE]; // 设备模块别名
    const char                             *driver_override;      // 驱动覆盖名称
    int                                    cs_gpio;               // 片选GPIO引脚号
    struct spi_statistics                  statistics;            // SPI传输统计信息
};
```

### 4.5 spi_device完整分析

**关键成员详解**：

- `dev`：device类型结构体。这是一个设备结构体，我们把它称为spi设备结构体、i2c设备结构体、平台设备结构体都是"继承"自设备结构体
- `controller`：当前spi设备挂载在那个spi控制器
- `master`：spi_master类型的结构体。在总线驱动中，一个spi_master代表了一个spi总线，这个参数就是用于指定spi设备挂载到那个spi总线上
- `max_speed_hz`：指定SPI通信的最大频率
- `chip_select`：spi总选用于区分不同SPI设备的一个标号，不要误以为他是SPI设备的片选引脚
- `bits_per_word`：指定SPI通信时一个字节多少位，也就是传输单位
- `mode`：SPI工作模式，工作模式如代码中的宏定义。包括时钟极性、位宽等等
- `irq`：如果使用了中断，它用于指定中断号
- `cs_gpio`：片选引脚。在设备树中设置了片选引脚，驱动和设别树节点匹配成功后自动获取片选引脚
- `statistics`：记录spi名字，用来和spi_driver进行配对

![SPI设备结构示意图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2cfccfa7742e037cca109917a55e83e7.png)

### 4.6 spi_device详细说明

关于SPI模式的详细配置，系统提供了标准的宏定义：

**SPI工作模式宏定义**：定义SPI通信的时钟极性、时钟相位以及四种标准工作模式组合

```c
// include/linux/spi/spi.h
#define SPI_CPHA                          0x01                 // 时钟相位控制位
#define SPI_CPOL                          0x02                 // 时钟极性控制位
#define SPI_MODE_0                        (0|0)                // 模式0：CPOL=0,CPHA=0
#define SPI_MODE_1                        (0|SPI_CPHA)         // 模式1：CPOL=0,CPHA=1
#define SPI_MODE_2                        (SPI_CPOL|0)         // 模式2：CPOL=1,CPHA=0
#define SPI_MODE_3                        (SPI_CPOL|SPI_CPHA) // 模式3：CPOL=1,CPHA=1
```

- `SPI_CPHA`：时钟相位标志位，控制数据在时钟的上升沿还是下降沿采样
- `SPI_CPOL`：时钟极性标志位，控制空闲时时钟信号的电平状态
- `SPI_MODE_0`：标准SPI模式0，空闲时钟为低电平，第一个时钟边沿采样数据
- `SPI_MODE_1`：SPI模式1，空闲时钟为低电平，第二个时钟边沿采样数据
- `SPI_MODE_2`：SPI模式2，空闲时钟为高电平，第一个时钟边沿采样数据
- `SPI_MODE_3`：SPI模式3，空闲时钟为高电平，第二个时钟边沿采样数据

关于SPI模式，它们的构建有两个特征：

**CPOL（时钟极性）**：

- 0：初始时钟状态为低电平，第一个边沿是上升沿
- 1：初始时钟状态为高电平，第一个状态为下降沿

**CPHA（时钟相位）**：

- 0：数据在下降沿（从高到低转换）锁存，而输出在上升沿改变
- 1：数据在上升沿（从低到高转换）锁存，并在下降沿输出

### 4.7 spi_driver详解

如果说`spi_device`是学生证，那么`spi_driver`就是老师的教师资格证，它描述了这个驱动程序能够处理哪些设备，以及如何处理它们。

**struct spi_driver**：SPI设备驱动程序结构体，定义了驱动的回调函数和设备匹配表

```c
// include/linux/spi/spi.h
struct spi_driver {
    const struct spi_device_id             *id_table;             // 设备ID匹配表
    int                                    (*probe)(struct spi_device *spi);     // 设备探测函数
    int                                    (*remove)(struct spi_device *spi);    // 设备移除函数
    void                                   (*shutdown)(struct spi_device *spi);  // 系统关闭处理函数
    struct device_driver                   driver;                // 标准设备驱动结构体
};
```

### 4.8 spi_driver完整分析

**关键成员详解**：

- `id_table`：用来和spi进行配对，指向spi_device_id数组的指针
- `probe`：spi设备和spi驱动匹配成功后，回调该函数指针
- `remove`：设备移除回调函数，设备被移除时调用进行清理
- `shutdown`：系统关闭时的处理函数，可选实现
- `driver`：继承的标准设备驱动结构体，包含驱动基本信息

可以看到spi设备驱动结构体和我们之前讲过的i2c设备驱动结构体`i2c_driver`、平台设备驱动结构体`platform_driver`拥有相同的结构，用法也相同。

### 4.9 spi_driver详细说明

就像需要i2c_device_id处理I2C设备一样，对于SPI设备，必须使用spi_device_id，以便提供device_id数组来匹配设备：

**struct spi_device_id**：SPI设备标识结构体，用于驱动程序识别和匹配特定的SPI设备

```c
// include/linux/mod_devicetable.h
struct spi_device_id {
    char                                   name[SPI_NAME_SIZE];  // 设备名称字符串
    kernel_ulong_t                         driver_data;          // 驱动私有数据
};
```

- `name`：设备名称字符串，长度限制为SPI_NAME_SIZE，用于与spi_device的modalias字段进行匹配
- `driver_data`：驱动程序的私有数据字段，通常存储设备特定的配置信息

**示例：MCP251x CAN控制器设备ID表**

```c
// drivers/net/can/spi/mcp251x.c
static const struct spi_device_id mcp251x_id_table[] = {
    {
        .name        = "mcp2510",                                    // MCP2510设备名称
        .driver_data = (kernel_ulong_t)CAN_MCP251X_MCP2510,         // MCP2510设备类型标识
    },
    {
        .name        = "mcp2515",                                    // MCP2515设备名称  
        .driver_data = (kernel_ulong_t)CAN_MCP251X_MCP2515,         // MCP2515设备类型标识
    },
    {
        .name        = "mcp25625",                                   // MCP25625设备名称
        .driver_data = (kernel_ulong_t)CAN_MCP251X_MCP25625,        // MCP25625设备类型标识
    },
    { }                                                              // 数组结束标志
};
MODULE_DEVICE_TABLE(spi, mcp251x_id_table);                         // 导出设备表供模块加载器使用
```

需要将数组嵌入struct spi_device_id，以便把驱动程序需要管理的设备ID通知SPI核心，并在该驱动程序结构上调用MODULE_DEVICE_TABLE宏。

### 4.10 spi_transfer详解

想象一下快递包裹的投递单，上面记录着收件人、发件人、包裹内容、投递要求等信息。`spi_transfer`就是SPI通信中的"投递单"，描述了一次具体的数据传输操作。

**struct spi_transfer**：SPI传输结构体，描述单次SPI数据传输的所有参数和要求

```c
// include/linux/spi/spi.h
struct spi_transfer {
    const void                             *tx_buf;               // 发送缓冲区指针
    void                                   *rx_buf;               // 接收缓冲区指针
    unsigned                               len;                   // 传输数据长度
    dma_addr_t                             tx_dma;                // 发送DMA地址
    dma_addr_t                             rx_dma;                // 接收DMA地址
    struct sg_table                        tx_sg;                 // 发送散列表
    struct sg_table                        rx_sg;                 // 接收散列表
    unsigned                               cs_change:1;           // 片选变化标志
    unsigned                               tx_nbits:3;            // 发送位宽
    unsigned                               rx_nbits:3;            // 接收位宽
#define SPI_NBITS_SINGLE                   0x01                   // 1位传输
#define SPI_NBITS_DUAL                     0x02                   // 2位传输
#define SPI_NBITS_QUAD                     0x04                   // 4位传输
    u8                                     bits_per_word;         // 每字位数
    u16                                    delay_usecs;           // 延迟微秒数
    u32                                    speed_hz;              // 传输速度
    struct list_head                       transfer_list;         // 传输链表节点
};
```

**重要成员说明**：

- `tx_buf`：发送缓冲区，用于指定要发送的数据地址
- `rx_buf`：接收缓冲区，用于保存接收得到的数据，如果不接收不用设置或设置为NULL
- `len`：要发送和接收的长度，根据SPI特性发送、接收长度相等
- `tx_dma、rx_dma`：如果使用了DMA，用于指定tx或rx DMA地址
- `bits_per_word、speed_hz`：分别用于设置每个字节多少位、发送频率。如果我们不设置这些参数那么会使用默认的配置
- `cs_change`：决定这次传输完成后chip_select线的状态
- `delay_usecs`：表示这次传输之后的延迟（以微秒为单位）

### 4.11 spi_message详解

如果说`spi_transfer`是单个快递包裹，那么`spi_message`就是一批快递包裹的集合，它们需要一起处理，要么全部成功，要么全部失败。

**struct spi_message**：SPI消息结构体，封装一个或多个SPI传输操作，保证原子性

```c
// include/linux/spi/spi.h
struct spi_message {
    struct list_head                       transfers;             // 传输列表
    struct spi_device                      *spi;                  // 目标SPI设备
    unsigned                               is_dma_mapped:1;       // 是否使用DMA映射
    void                                   (*complete)(void *context);    // 完成回调函数
    void                                   *context;              // 回调函数参数
    unsigned                               frame_length;          // 帧长度
    unsigned                               actual_length;         // 实际传输长度
    int                                    status;                // 传输状态
    struct list_head                       queue;                 // 消息队列节点
    void                                   *state;                // 状态信息
};
```

**关键成员说明**：

- `transfers`：构成消息的传输列表，多个spi_transfer通过这个链表连接
- `spi`：指定消息来自哪个设备，一个spi设备对应一个spi_device结构体
- `is_dma_mapped`：通知控制器是否使用DMA执行传输
- `complete`：这是事务完成时调用的回调，context是传递给回调的参数
- `frame_length`：这将自动设置为消息中的总字节数
- `actual_length`：这是在所有成功段中传输的字节数
- `status`：报告传输状态。成功为0，否则为-errno

简单来说，spi_transfer结构体保存了要发送(或接收)的数据，而在SPI设备驱动中数据是以"消息"的形式发送。

## 5 数据流向分析

### 5.1 传输数据结构关系

想象一下物流系统的运作：单个包裹（transfer）被打包成一个货运批次（message），多个批次排队等待处理（queue），最后由运输车队（controller）依次配送。SPI的数据传输就是这样一个完整的物流系统。

![SPI子系统数据结构关系图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/754c802a99aac6b2d63baf12dbde7528.png)

上图展示了SPI传输的层次化组织架构：

**传输层级结构**：

- **spi_transfer()**：最基本的传输单元，包含tx_buf（发送缓冲区）等字段，通过transfers_list连接到传输链表中，多个transfer可以组合成一个完整的SPI操作
    
- **spi_transfer_list**：连接多个spi_transfer结构的链表，将相关的传输操作组织在一起
    
- **spi_message()**：更高层的消息结构，包含transfers、queue等字段，一个message可以包含多个transfer，代表一次完整的SPI通信事务
    
- **spi master端的链表**：SPI主控制器维护的消息队列，将多个message排队等待处理，实现SPI传输的调度和管理
    

换句话说，整个架构体现了"传输->消息->队列"的三级组织结构，支持复杂的SPI通信场景，如多段传输、批量操作等。

### 5.2 队列管理机制

在进行spi数据传输的时候，如果有同一时段有多个spi msg要处理，则会将要处理的msg连成一个链表，等待依次处理，该链表头一般都是包含了spi_master{}的实际控制端。

spi数据传输主要使用了`spi_message`和`spi_transfer`结构，多个`spi_transfer`构成一个`spi_message`。

**队列处理机制**：

1. **消息入队**：当应用程序发起SPI传输时，消息被添加到控制器的队列中
2. **队列调度**：控制器按照先进先出的原则处理队列中的消息
3. **并发控制**：使用互斥锁确保队列操作的原子性
4. **状态管理**：跟踪每个消息的处理状态

### 5.3 控制器队列处理

spi_message通过成员变量queue将一系列的spi_message串联起来，第一个spi_message挂在struct list_head queue下面，spi_message还有struct list_head transfers成员变量，transfer也是被串联起来的。

![SPI队列管理机制图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/238fa8c32f8bb146ed453f7e1890687e.png)

**队列处理流程**：

1. **消息队列**：多个spi_message通过queue字段连接成链表
2. **传输队列**：每个spi_message内部包含多个spi_transfer
3. **工作线程**：控制器使用工作线程异步处理队列中的消息
4. **同步机制**：提供同步和异步两种传输方式

#### 5.3.1 SPI同步传输数据

阻塞当前线程进行数据传输，spi_sync()内部调用__spi_sync()函数，mutex_lock()和mutex_unlock()为互斥锁的加锁和解锁：

**spi_sync函数**：阻塞式SPI数据传输函数，确保数据传输的完整性

```c
// drivers/spi/spi.c
int spi_sync(struct spi_device *spi, struct spi_message *message)
{
    int ret;

    mutex_lock(&spi->controller->bus_lock_mutex);       // 加锁保护总线
    ret = __spi_sync(spi, message);                     // 执行同步传输
    mutex_unlock(&spi->controller->bus_lock_mutex);     // 释放总线锁

    return ret;                                         // 返回传输结果
}
```

#### 5.3.2 SPI异步传输数据

**spi_async函数**：非阻塞式SPI数据传输函数，支持异步操作

```c
// drivers/spi/spi.c
int spi_async(struct spi_device *spi, struct spi_message *message)
{
    // ...
    ret = __spi_async(spi, message);                    // 执行异步传输
    // ...
}
```

在驱动程序中调用async时不会阻塞当前进程，只是把当前message结构体添加到当前spi控制器成员queue的末尾。然后在内核中新增加一个工作，这个工作的内容就是去处理这个message结构体。

**同步与异步传输的区别**：

- **同步传输**：调用函数后会等待传输完成才返回，适合简单的阻塞式操作
- **异步传输**：调用函数后立即返回，传输在后台进行，适合需要并发处理的场景

## 6 设备驱动接口函数

### 6.1 spi设备驱动注册

spi设备的注册和注销函数分别在驱动的入口和出口函数中调用，这与平台设备驱动、i2c设备驱动相同：

**spi_register_driver函数**：注册SPI设备驱动到内核

```c
// drivers/spi/spi.c
int spi_register_driver(struct spi_driver *sdrv);     // 注册SPI驱动
static inline void spi_unregister_driver(struct spi_driver *sdrv); // 注销SPI驱动
```

- `spi`：spi_driver类型的结构体(spi设备驱动结构体)，一个spi_driver结构体就代表了一个spi设备驱动
- **返回值**：成功返回0，失败返回错误码

### 6.2 spi_setup函数

函数设置spi设备的片选信号、传输单位、最大传输速率等，函数中调用spi控制器的成员controller->setup()：

**spi_setup函数**：配置SPI设备的工作参数，如传输模式、频率等

```c
// drivers/spi/spi.c
int spi_setup(struct spi_device *spi);
```

- `spi`：spi_device spi设备结构体
- **返回值**：成功返回0，失败返回错误码

### 6.3 消息初始化函数

**spi_message_init函数**：初始化SPI消息结构体，准备进行数据传输

```c
// include/linux/spi/spi.h
static inline void spi_message_init(struct spi_message *m)
{
    memset(m, 0, sizeof *m);                           // 清零消息结构体
    spi_message_init_no_memset(m);                     // 初始化链表等结构
}
```

- `m`：spi_message结构体指针，spi_message结构体定义和介绍可在前面关键数据结构中找到
- **返回值**：无

**spi_message_add_tail函数**：将传输结构体添加到消息队列的末尾

```c
// include/linux/spi/spi.h
static inline void spi_message_add_tail(struct spi_transfer *t, struct spi_message *m)
{
    list_add_tail(&t->transfer_list, &m->transfers);   // 添加到传输列表末尾
}
```

- `t`：要添加的spi_transfer结构体指针
- `m`：目标spi_message结构体指针

这个函数很简单就是将spi_transfer结构体添加到spi_message队列的末尾。

### 6.4 数据传输使用示例

消息提交给总线之前，必须先使用`spi_message_init`初始化，这将结构中的每个元素清零并初始化transfers列表。对于要添加到消息中的每个传输，应该在该传输上调用`spi_message_add_tail`，把传输加入transfers列表队列。

**单个传输SPI消息事务示例**：

```c
char tx_buf[] = { 
    0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 
    0xFF, 0x40, 0x00, 0x00, 0x00, 
    0x00, 0x95, 0xEF, 0xBA, 0xAD, 
    0xF0, 0x0D, 
};  

char rx_buf[10] = {0,}; 
int ret; 
struct spi_message single_msg; 
struct spi_transfer single_xfer; 

single_xfer.tx_buf = tx_buf;                           // 设置发送缓冲区
single_xfer.rx_buf = rx_buf;                           // 设置接收缓冲区
single_xfer.len    = sizeof(tx_buf);                   // 设置传输长度
single_xfer.bits_per_word = 8;                         // 设置传输位宽

spi_message_init(&single_msg);                         // 初始化消息
spi_message_add_tail(&single_xfer, &single_msg);       // 添加传输到消息
ret = spi_sync(spi, &single_msg);                      // 执行同步传输
```

**多传输消息事务示例**：

```c
struct { 
    char buffer[10]; 
    char cmd[2];  
    int foo; 
} data; 

struct data my_data[3]; 
initialize_data(my_data, ARRAY_SIZE(my_data)); 

struct spi_transfer   multi_xfer[3]; 
struct spi_message    multi_msg; 
int ret; 

multi_xfer[0].rx_buf = data[0].buffer;                 // 接收数据
multi_xfer[0].len = 5; 
multi_xfer[0].cs_change = 1;                           // 传输后改变片选

multi_xfer[1].tx_buf = data[1].cmd;                    // 发送命令A
multi_xfer[1].len = 2; 
multi_xfer[1].cs_change = 1; 

multi_xfer[2].rx_buf = data[2].buffer;                 // 接收命令B响应
multi_xfer[2].len = 10; 

spi_message_init(&multi_msg);                          // 初始化消息
spi_message_add_tail(&multi_xfer[0], &multi_msg);      // 添加传输1
spi_message_add_tail(&multi_xfer[1], &multi_msg);      // 添加传输2
spi_message_add_tail(&multi_xfer[2], &multi_msg);      // 添加传输3
ret = spi_sync(spi, &multi_msg);                       // 执行同步传输
```

也就是说，整个SPI子系统通过这种分层队列机制，实现了高效的数据传输管理，既支持简单的单次传输，也支持复杂的批量传输操作。

> [!tip]+ 实用技巧 在实际应用中，建议优先使用同步传输方式，因为它简单可靠。只有在需要高性能或者避免阻塞的场景下，才考虑使用异步传输

---

## 本章总结

通过本章的学习，我们全面了解了Linux SPI子系统的架构设计和核心组件。

### 核心知识点回顾

**抽象模型理解**：

- Linux SPI子系统采用三层架构设计，分离用户空间、内核空间和硬件层
- 四个核心组件（controller、bus_type、device、driver）协同工作
- 与平台设备模型类似的设计思想，便于理解和扩展

**子系统初始化**：

- spi_init函数完成整个子系统的基础设施建设
- 总线注册、设备类创建、核心层注册的完整流程
- 为后续的设备和驱动注册奠定基础

**总线模型精通**：

- spi_bus_type定义了SPI总线的基本属性
- 多层次的设备驱动匹配机制，提供了灵活的匹配策略
- 匹配优先级确保了设备能找到最合适的驱动

**数据结构掌握**：

- spi_controller是整个SPI子系统的核心管理者
- spi_device描述了SPI设备的所有属性和配置
- spi_driver定义了驱动程序的能力和接口
- spi_transfer和spi_message构成了数据传输的基础

**数据流向分析**：

- 理解了从transfer到message再到queue的层次结构
- 掌握了队列管理和并发控制机制
- 了解了同步和异步传输的实现原理

### 与其他子系统的联系

SPI子系统的设计体现了Linux内核的统一设计思想：

- 设备模型的一致性
- 分层架构的清晰性
- 接口设计的标准化

### 实践指导意义

这些架构知识为后续的实际开发提供了重要指导：

- 理解了如何设计设备树节点
- 掌握了驱动程序的基本框架
- 学会了如何进行数据传输操作

> [!note]+ 后续学习指引 掌握了SPI子系统的架构后，下一步可以学习具体的SPI控制器驱动实现，了解如何在实际硬件平台上使用这些抽象接口

这些架构知识为后续的SPI控制器驱动分析和设备驱动开发打下了坚实的理论基础。在接下来的章节中，我们将深入学习SPI控制器的具体实现，以及如何编写实际的SPI设备驱动程序。