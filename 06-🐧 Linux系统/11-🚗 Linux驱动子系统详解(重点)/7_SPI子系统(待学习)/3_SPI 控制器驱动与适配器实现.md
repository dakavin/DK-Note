
在前两篇文章中，我们掌握了SPI硬件协议的基础理论和Linux SPI子系统的架构设计。现在，我们需要深入理解SPI子系统的"心脏"——**SPI控制器驱动**。

想象一下，如果SPI子系统是一个现代化的工厂，那么控制器驱动就是工厂的"生产车间主管"，它负责管理具体的硬件设备，协调生产流程，确保产品（数据）能够高质量、高效率地生产出来。

换句话说，控制器驱动是连接SPI硬件和软件的关键桥梁，它将抽象的SPI操作转换为具体的硬件控制指令。简单来说，无论上层软件发出什么样的SPI操作请求，都需要通过控制器驱动来实际操控硬件完成数据传输。

## 1 控制器驱动概念

### 1.1 控制器作用机制

#### 1.1.1 控制器在子系统中的地位

SPI控制器驱动在整个SPI子系统中扮演着核心角色，就像是交响乐团中的指挥家，它需要协调各个部分的工作，确保整体的和谐运行。

**控制器驱动的核心职责**：

- **硬件抽象**：将不同厂商的SPI硬件抽象为统一的接口
- **资源管理**：管理时钟、中断、DMA、GPIO等硬件资源
- **数据传输**：实现实际的SPI数据传输操作
- **状态控制**：管理SPI设备的工作状态和电源状态
- **错误处理**：处理传输过程中的各种异常情况

#### 1.1.2 控制器驱动的分层结构

**三层处理机制**：

1. **上层接口层**：提供标准的`spi_controller`接口，与SPI核心层交互
2. **中间适配层**：处理消息队列、工作线程、同步机制等通用逻辑
3. **底层硬件层**：直接操作SPI控制器寄存器，控制硬件行为

这种分层设计使得控制器驱动既能适配不同的硬件，又能为上层提供统一的接口。

#### 1.1.3 控制器与核心层的交互

控制器驱动通过一系列**回调函数**与SPI核心层交互：

- **setup**：初始化SPI设备的工作参数
- **transfer_one**：执行单次SPI传输操作
- **set_cs**：控制片选信号的有效性
- **prepare_transfer_hardware**：准备硬件资源
- **cleanup**：清理和释放资源

> [!note]+ 背景知识 SPI控制器驱动通常基于平台驱动模型实现，这样可以很好地利用设备树进行硬件描述和资源管理

### 1.2 平台驱动模型

#### 1.2.1 平台驱动的优势

SPI控制器驱动采用平台驱动模型有诸多优势，就像是为特定工厂设计的专用管理系统：

**设备树集成**：

- 硬件配置参数可以通过设备树灵活配置
- 支持不同板卡的差异化配置
- 便于硬件资源的统一管理

**资源自动管理**：

- 自动处理内存映射、时钟、中断等资源
- 支持电源管理和热插拔
- 提供设备生命周期管理

#### 1.2.2 Rockchip SPI驱动注册

**Rockchip SPI设备树匹配表**：定义Rockchip系列芯片SPI控制器的设备树兼容性字符串匹配表

```c
// drivers/spi/spi-rockchip.c
static const struct of_device_id rockchip_spi_dt_match[] = {
    { .compatible = "rockchip,px30-spi",   },        // PX30芯片SPI控制器
    { .compatible = "rockchip,rv1108-spi", },        // RV1108芯片SPI控制器
    { .compatible = "rockchip,rv1126-spi", },        // RV1126芯片SPI控制器
    { .compatible = "rockchip,rk3036-spi", },        // RK3036芯片SPI控制器
    { .compatible = "rockchip,rk3066-spi", },        // RK3066芯片SPI控制器
    { .compatible = "rockchip,rk3188-spi", },        // RK3188芯片SPI控制器
    { .compatible = "rockchip,rk3228-spi", },        // RK3228芯片SPI控制器
    { .compatible = "rockchip,rk3288-spi", },        // RK3288芯片SPI控制器
    { .compatible = "rockchip,rk3368-spi", },        // RK3368芯片SPI控制器
    { .compatible = "rockchip,rk3399-spi", },        // RK3399芯片SPI控制器
    { },                                             // 数组结束标志
};
MODULE_DEVICE_TABLE(of, rockchip_spi_dt_match);      // 导出设备树匹配表
```

**匹配表详细说明**：

- `px30/rv1108/rv1126`：Rockchip低功耗系列芯片的SPI控制器兼容性标识，主要用于IoT和嵌入式应用
- `rk3036/rk3066/rk3188`：Rockchip早期ARM Cortex-A系列芯片的SPI控制器，支持基础的SPI功能
- `rk3228/rk3288`：Rockchip中端应用处理器系列的SPI控制器，性能更强，支持更高的传输速率
- `rk3368/rk3399`：Rockchip高端64位应用处理器的SPI控制器，支持多种高级特性
- `MODULE_DEVICE_TABLE(of, ...)`：向内核注册设备树匹配表，支持设备树自动匹配和模块加载

#### 1.2.3 平台驱动结构体

**Rockchip SPI平台驱动结构体**：定义Rockchip SPI控制器的平台驱动程序结构

```c
// drivers/spi/spi-rockchip.c
static struct platform_driver rockchip_spi_driver = {
    .driver = {
        .name            = DRIVER_NAME,                          // 驱动程序名称
        .pm              = &rockchip_spi_pm,                     // 电源管理操作
        .of_match_table  = of_match_ptr(rockchip_spi_dt_match),  // 设备树匹配表
    },
    .probe               = rockchip_spi_probe,                   // 设备探测函数
    .remove              = rockchip_spi_remove,                  // 设备移除函数
};
module_platform_driver(rockchip_spi_driver);                    // 注册平台驱动
```

**驱动结构说明**：

- `DRIVER_NAME`：驱动程序的标识名称，通常定义为"rockchip-spi"
- `rockchip_spi_pm`：电源管理操作集，包括suspend/resume等回调函数，支持系统休眠唤醒
- `rockchip_spi_probe`：当匹配的设备被发现时调用的初始化函数，负责硬件初始化和资源分配
- `rockchip_spi_remove`：设备移除或驱动卸载时的清理函数，负责资源释放和硬件停止
- `module_platform_driver`：简化的平台驱动注册宏，自动处理模块加载和卸载

> [!tip]+ 实用技巧 使用module_platform_driver宏可以大大简化驱动的注册代码，它会自动生成模块的init和exit函数

## 2 RK3568控制器分析

### 2.1 设备树配置

#### 2.1.1 设备树节点作用

设备树就像是硬件的"身份证"和"使用说明书"，它详细描述了SPI控制器的硬件配置信息。对于RK3568芯片来说，SPI控制器的设备树配置包含了所有必要的硬件参数。

![RK3568控制器框图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8c50fd6fbcf0641b5f58bab59af56894.png)

#### 2.1.2 SPI3控制器设备树配置

**SPI3控制器设备树节点**：定义SPI3控制器的硬件配置参数，包括寄存器地址、中断、时钟和引脚配置

```c
// arch/arm64/boot/dts/rockchip/rk3066.dtsi
spi3: spi@fe640000 {
    compatible      = "rockchip,rk3066-spi";      // 兼容性字符串
    reg             = <0x0 0xfe640000 0x0 0x1000>; // 寄存器地址和大小
    interrupts      = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>; // 中断配置
    #address-cells  = <1>;                        // 地址单元格数量
    #size-cells     = <0>;                        // 大小单元格数量
    clocks          = <&cru CLK_SPI3>, <&cru PCLK_SPI3>; // 时钟源配置
    clock-names     = "spiclk", "apb_pclk";       // 时钟名称
    dmas            = <&dmac0 26>, <&dmac0 27>;    // DMA通道配置
    dma-names       = "tx", "rx";                 // DMA通道名称
    pinctrl-names   = "default", "high_speed";    // 引脚控制配置名称
    pinctrl-0       = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins>; // 默认引脚配置
    pinctrl-1       = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins_hs>; // 高速引脚配置
    status          = "disabled";                 // 设备状态
};
```

#### 2.1.3 设备树属性详解

**核心属性说明**：

- `compatible`：设备兼容性字符串，内核通过此字符串匹配相应的SPI控制器驱动程序
- `reg`：SPI控制器的物理寄存器基地址(0xfe640000)和地址空间大小(0x1000，即4KB)
- `interrupts`：中断配置，使用GIC SPI中断103号，高电平触发方式
- `#address-cells/#size-cells`：定义子设备地址和大小的描述格式，用于SPI从设备节点的地址解析

**时钟和电源属性**：

- `clocks`：引用的时钟源，包括SPI功能时钟(CLK_SPI3)和APB总线时钟(PCLK_SPI3)
- `clock-names`：时钟的逻辑名称，驱动程序通过这些名称获取对应时钟资源

**DMA和传输属性**：

- `dmas/dma-names`：DMA传输配置，分别指定发送和接收使用的DMA通道，提高大数据传输效率
- `pinctrl-x`：引脚复用配置，包括片选信号、数据信号和时钟信号的GPIO配置

**状态控制**：

- `status`：设备初始状态，"disabled"表示默认禁用，需要在板级DTS中启用

> [!warning]+ 重要提醒 设备树配置中的pinctrl信息必须与实际硬件连接一致，否则SPI通信将无法正常工作

### 2.2 设备匹配表

#### 2.2.1 匹配表的重要性

设备匹配表就像是"设备身份验证系统"，它确保内核能够为特定的硬件选择正确的驱动程序。Rockchip提供了一个兼容多种芯片的匹配表。

#### 2.2.2 芯片系列分析

**低功耗系列**：

- `px30-spi`：主要用于智能显示、IoT网关等低功耗应用
- `rv1108-spi`：专注于AI视觉处理的低功耗芯片
- `rv1126-spi`：新一代AIoT应用处理器

**中端应用系列**：

- `rk3036-spi`：入门级多媒体应用处理器
- `rk3228-spi`：支持4K视频的中端处理器
- `rk3288-spi`：高性能多媒体应用处理器

**高端系列**：

- `rk3368-spi`：64位高性能应用处理器
- `rk3399-spi`：旗舰级双核A72+四核A53处理器

#### 2.2.3 匹配机制工作流程

当内核启动时，设备树匹配的工作流程如下：

1. **解析设备树**：内核解析设备树中的SPI控制器节点
2. **提取compatible**：获取节点的compatible属性值
3. **遍历匹配表**：在rockchip_spi_dt_match中查找匹配项
4. **调用probe**：匹配成功后调用rockchip_spi_probe函数
5. **初始化控制器**：完成SPI控制器的硬件初始化

> [!example]+ 动手实践 可以通过查看/sys/bus/platform/drivers/rockchip-spi/目录来确认驱动是否正确匹配和加载

### 2.3 控制器特性

#### 2.3.1 RK3568 SPI控制器优势

RK3568的SPI控制器具有出色的性能和丰富的功能特性，就像是一台高性能的"数据传输设备"：

**基础特性**：

- **协议支持**：默认采用摩托罗拉SPI协议，兼容性好，应用广泛
- **数据位宽**：支持8位和16位数据传输，适应不同设备需求
- **传输速度**：软件可编程时钟频率，传输速率高达50MHz
- **模式支持**：支持SPI 4种传输模式配置(Mode 0-3)

**高级特性**：

- **多片选支持**：每个SPI控制器支持一个到两个片选，可连接多个设备
- **双模式操作**：框架支持slave和master两种模式，应用灵活
- **DMA支持**：支持DMA传输，减轻CPU负担，提高传输效率
- **中断机制**：完善的中断处理机制，支持异步传输

#### 2.3.2 性能指标分析

**传输性能**：

- **最高频率**：50MHz，满足大多数高速SPI设备需求
- **数据吞吐量**：在50MHz频率下，理论传输速度可达6.25MB/s
- **延迟控制**：支持rx-sample-delay配置，优化高频传输的信号质量

**资源利用**：

- **CPU占用**：支持DMA传输，大幅降低CPU占用率
- **内存效率**：支持scatter-gather DMA，优化内存使用
- **功耗控制**：支持动态时钟管理和电源管理

#### 2.3.3 应用场景

**高速数据传输**：

- SPI Flash存储器读写
- 高分辨率显示屏驱动
- 高速ADC/DAC数据采集

**实时控制应用**：

- 电机控制器通信
- 传感器数据读取
- 实时通信协议

> [!tip]+ 实用技巧 对于高频SPI应用，建议启用DMA传输并适当配置rx-sample-delay参数以获得最佳性能

### 2.4 代码路径说明

#### 2.4.1 驱动代码结构

SPI驱动代码按照功能和层次进行组织，就像是一个结构清晰的"软件工厂"：

**核心驱动文件**：

```
drivers/spi/spi.c                    // SPI驱动框架核心实现
drivers/spi/spi-rockchip.c          // Rockchip SPI控制器具体实现
drivers/spi/spidev.c                // SPI设备节点创建，用户态接口
```

**测试和工具**：

```
drivers/spi/spi-rockchip-test.c     // SPI测试驱动，需手动添加编译
Documentation/spi/spidev_test.c     // 用户态SPI测试工具
```

#### 2.4.2 代码功能分析

**spi.c - 框架核心**：

- 实现SPI子系统的核心功能
- 提供设备和驱动的注册管理
- 实现消息队列和传输调度

**spi-rockchip.c - 硬件适配**：

- 实现Rockchip芯片的SPI控制器驱动
- 处理硬件相关的初始化和传输操作
- 提供平台相关的电源管理和错误处理

**spidev.c - 用户接口**：

- 创建/dev/spidevX.Y设备节点
- 提供用户空间的SPI访问接口
- 实现字符设备的文件操作

#### 2.4.3 开发和调试建议

**代码修改路径**：

- **添加新芯片支持**：修改spi-rockchip.c中的匹配表
- **性能优化**：在transfer_one函数中优化传输逻辑
- **功能扩展**：在probe函数中添加新的硬件特性支持

**调试工具使用**：

- **内核调试**：使用spi-rockchip-test.c进行内核级测试
- **用户态测试**：使用spidev_test.c进行应用层测试
- **性能分析**：通过ftrace跟踪SPI传输性能

> [!note]+ 背景知识 Rockchip的SPI驱动支持热插拔和动态配置，这使得它在嵌入式应用中具有很好的灵活性

## 3 控制器初始化详解

### 3.1 probe函数完整分析

#### 3.1.1 probe函数的核心作用

probe函数就像是一个"工厂车间的开工仪式"，它负责完成SPI控制器从硬件发现到可用状态的全部初始化工作。这个过程包括资源分配、硬件配置、功能设置等多个关键步骤。

#### 3.1.2 probe函数主要流程

probe函数的执行遵循以下三个核心步骤：

1. **获取设备树资源**：解析设备树节点，获取硬件配置参数
2. **向上层提供操作接口**：实现transfer_one等关键回调函数
3. **向内核注册设备**：将控制器注册到SPI子系统

#### 3.1.3 Rockchip私有结构体

在详细分析probe函数之前，我们需要了解Rockchip定义的私有结构体，它就像是控制器的"个人档案"：

```c
// drivers/spi/spi-rockchip.c
struct rockchip_spi {
    struct device               *dev;                    // 设备指针
    struct clk                  *spiclk;                 // SPI功能时钟
    struct clk                  *apb_pclk;               // APB总线时钟
    void __iomem                *regs;                   // 寄存器映射地址
    dma_addr_t                  dma_addr_rx;             // 接收DMA地址
    dma_addr_t                  dma_addr_tx;             // 发送DMA地址
    const void                  *tx;                     // 发送数据指针
    void                        *rx;                     // 接收数据指针
    unsigned int                tx_left;                 // 剩余发送字节数
    unsigned int                rx_left;                 // 剩余接收字节数
    atomic_t                    state;                   // 控制器状态
    u32                         fifo_len;                // FIFO缓冲区深度
    u32                         freq;                    // SPI时钟频率
    u8                          n_bytes;                 // 每次传输字节数
    u8                          rsd;                     // 读采样延迟
    bool                        cs_asserted[ROCKCHIP_SPI_MAX_CS_NUM]; // 片选状态数组
    struct pinctrl_state        *high_speed_state;       // 高速引脚状态
    bool                        slave_abort;             // 从模式中止标志
    bool                        gpio_requested;          // GPIO请求标志
    bool                        cs_inactive;             // 片选非活动状态
    struct spi_transfer         *xfer;                   // 当前传输结构
    spinlock_t                  lock;                    // 防止I/O并发访问
};
```

这个结构体包含了控制器运行所需的所有信息，从时钟配置到DMA地址，从状态管理到并发控制。

### 3.2 资源获取映射

#### 3.2.1 控制器内存分配

probe函数首先需要分配和初始化SPI控制器结构体：

```c
// probe函数关键代码段 - 控制器分配
static int rockchip_spi_probe(struct platform_device *pdev)
{
    int ret;
    struct rockchip_spi *rs;
    struct spi_controller *ctlr;
    struct resource *mem;
    struct device_node *np = pdev->dev.of_node;
    bool slave_mode;
    
    // 判断当前控制器模式（主/从）
    slave_mode = of_property_read_bool(np, "spi-slave");

    // 根据模式分配不同类型的控制器
    if (slave_mode)
        ctlr = spi_alloc_slave(&pdev->dev, sizeof(struct rockchip_spi));
    else
        ctlr = spi_alloc_master(&pdev->dev, sizeof(struct rockchip_spi));

    if (!ctlr)
        return -ENOMEM;

    // 在platform_device中记录spi_controller
    platform_set_drvdata(pdev, ctlr);

    // 获取私有数据指针
    rs = spi_controller_get_devdata(ctlr);
    ctlr->slave = slave_mode;
```

#### 3.2.2 寄存器地址映射

接下来进行寄存器地址的映射，这就像是为控制器配置"办公地址"：

```c
    // 获取基本IO资源并进行映射
    mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    rs->regs = devm_ioremap_resource(&pdev->dev, mem);
    if (IS_ERR(rs->regs)) {
        ret = PTR_ERR(rs->regs);
        goto err_put_ctlr;
    }

    // 初始化自旋锁
    spin_lock_init(&rs->lock);
```

**关键操作说明**：

- `platform_get_resource`：从平台设备中获取内存资源信息
- `devm_ioremap_resource`：将物理地址映射到虚拟地址空间
- `spin_lock_init`：初始化自旋锁，用于保护并发访问

> [!tip]+ 实用技巧 使用devm_系列函数可以实现资源的自动管理，当设备被移除时会自动释放资源

### 3.3 时钟配置

#### 3.3.1 时钟资源获取

SPI控制器需要两个时钟：功能时钟和总线时钟，就像是工厂需要"生产用电"和"照明用电"：

```c
    // APB总线时钟获取和使能
    rs->apb_pclk = devm_clk_get(&pdev->dev, "apb_pclk");
    if (IS_ERR(rs->apb_pclk)) {
        dev_err(&pdev->dev, "Failed to get apb_pclk\n");
        ret = PTR_ERR(rs->apb_pclk);
        goto err_put_ctlr;
    }

    // SPI功能时钟获取
    rs->spiclk = devm_clk_get(&pdev->dev, "spiclk");
    if (IS_ERR(rs->spiclk)) {
        dev_err(&pdev->dev, "Failed to get spi_pclk\n");
        ret = PTR_ERR(rs->spiclk);
        goto err_put_ctlr;
    }

    // 启用APB时钟
    ret = clk_prepare_enable(rs->apb_pclk);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to enable apb_pclk\n");
        goto err_put_ctlr;
    }

    // 启用SPI时钟
    ret = clk_prepare_enable(rs->spiclk);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to enable spi_clk\n");
        goto err_disable_apbclk;
    }
```

#### 3.3.2 时钟配置参数

**时钟配置要点**：

- **apb_pclk**：APB总线时钟，用于寄存器访问，通常固定频率
- **spiclk**：SPI功能时钟，用于数据传输，可变频率
- **时钟关系**：SPI输出时钟由spiclk分频产生，最小2分频

**频率配置建议**：

- spiclk频率 ≥ 2 × spi-max-frequency
- spiclk频率不要低于24MHz
- 对于高频应用，建议使用100MHz以上的spiclk

#### 3.3.3 时钟管理机制

```c
    // 初始化时禁用SPI芯片
    spi_enable_chip(rs, false);
    
    // 记录设备和频率信息
    rs->dev = &pdev->dev;
    rs->freq = clk_get_rate(rs->spiclk);        // 获取实际时钟频率
```

> [!warning]+ 重要提醒 时钟的使能顺序很重要：先使能APB时钟，再使能SPI功能时钟，这样可以确保寄存器访问的正确性

### 3.4 中断和DMA配置

#### 3.4.1 中断资源配置

中断机制就像是工厂的"紧急通知系统"，确保重要事件能够及时得到处理：

```c
    // 中断注册
    ret = platform_get_irq(pdev, 0);               // 获取中断号
    if (ret < 0)
        goto err_disable_spiclk;

    // 注册线程化中断处理函数
    ret = devm_request_threaded_irq(&pdev->dev, ret, rockchip_spi_isr, NULL,
                    IRQF_ONESHOT, dev_name(&pdev->dev), ctlr);
    if (ret)
        goto err_disable_spiclk;
```

**中断配置特点**：

- 使用线程化中断（`threaded_irq`）提高系统响应性
- `IRQF_ONESHOT`标志确保中断处理的原子性
- 中断处理函数为`rockchip_spi_isr`

#### 3.4.2 DMA通道配置

DMA配置就像是为工厂配置"自动化输送带"，减轻CPU的工作负担：

```c
    // 请求发送DMA通道
    ctlr->dma_tx = dma_request_chan(rs->dev, "tx");
    if (IS_ERR(ctlr->dma_tx)) {
        if (PTR_ERR(ctlr->dma_tx) == -EPROBE_DEFER) {
            ret = -EPROBE_DEFER;
            goto err_disable_pm_runtime;
        }
        dev_warn(rs->dev, "Failed to request TX DMA channel\n");
        ctlr->dma_tx = NULL;
    }

    // 请求接收DMA通道
    ctlr->dma_rx = dma_request_chan(rs->dev, "rx");
    if (IS_ERR(ctlr->dma_rx)) {
        if (PTR_ERR(ctlr->dma_rx) == -EPROBE_DEFER) {
            ret = -EPROBE_DEFER;
            goto err_free_dma_tx;
        }
        dev_warn(rs->dev, "Failed to request RX DMA channel\n");
        ctlr->dma_rx = NULL;
    }

    // 配置DMA地址
    if (ctlr->dma_tx && ctlr->dma_rx) {
        rs->dma_addr_tx = mem->start + ROCKCHIP_SPI_TXDR;
        rs->dma_addr_rx = mem->start + ROCKCHIP_SPI_RXDR;
        ctlr->can_dma = rockchip_spi_can_dma;
    }
```

**DMA配置说明**：

- 分别请求发送和接收DMA通道
- 配置DMA的源地址和目标地址
- 设置`can_dma`回调函数判断是否使用DMA传输

#### 3.4.3 电源管理配置

```c
    // 运行时电源管理设置
    pm_runtime_set_active(&pdev->dev);
    pm_runtime_enable(&pdev->dev);
```

这些配置确保了SPI控制器具备完整的硬件功能支持。

![probe函数执行流程图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0bb106fb4677289a43ed6847d7e674a5.png)

如上图所示，展示了Rockchip SPI控制器的初始化配置过程，包括设置电源管理、SPI模式位、传输速度限制以及各种回调函数的赋值。

> [!example]+ 动手实践 可以通过检查/proc/interrupts文件来确认SPI中断是否正确注册，通过/sys/kernel/debug/dma_pools查看DMA通道分配情况

## 4 控制器注册流程

### 4.1 spi_controller分配

#### 4.1.1 控制器内存分配机制

SPI控制器的分配就像是为新员工准备办公室和工作用品，需要为控制器分配内存空间和设置基本属性。

**spi_alloc_master**：分配SPI主控制器结构体，为控制器分配内存并进行基础初始化

```c
// include/linux/spi/spi.h
static inline struct spi_controller *spi_alloc_master(struct device *host,
                                                      unsigned int size)
{
    return __spi_alloc_controller(host, size, false);  // false表示主模式
}
```

#### 4.1.2 控制器分配详细实现

**__spi_alloc_controller**：SPI控制器分配的核心实现函数，处理内存分配和基础设置

```c
// drivers/spi/spi.c
struct spi_controller *__spi_alloc_controller(struct device *dev,
                                              unsigned int size, bool slave)
{
    struct spi_controller   *ctlr;

    if (!dev)
        return NULL;

    // 分配控制器结构体和私有数据的内存
    ctlr = kzalloc(size + sizeof(*ctlr), GFP_KERNEL);  
    if (!ctlr)
        return NULL;

    // 初始化内嵌的device结构体
    device_initialize(&ctlr->dev);    
    ctlr->bus_num = -1;                                 // 总线编号初始化为-1（自动分配）
    ctlr->num_chipselect = 1;                           // 默认片选数量为1
    ctlr->slave = slave;                                // 设置主/从模式标志
    
    // 设置设备类
    if (IS_ENABLED(CONFIG_SPI_SLAVE) && slave)
        ctlr->dev.class = &spi_slave_class;             // 从设备类
    else
        ctlr->dev.class = &spi_master_class;            // 主设备类
        
    ctlr->dev.parent = dev;                             // 设置父设备
    pm_suspend_ignore_children(&ctlr->dev, true);       // 电源管理设置
    
    // 设置私有数据指针，指向分配内存中控制器结构体之后的部分
    spi_controller_set_devdata(ctlr, &ctlr[1]);

    return ctlr;
}
```

**关键设计要点**：

- `size + sizeof(*ctlr)`：一次性分配控制器结构体和私有数据的内存空间
- `&ctlr[1]`：私有数据指针指向控制器结构体之后的内存区域
- `device_initialize`：初始化内嵌的device结构，融入Linux设备模型

#### 4.1.3 内存布局图解

![控制器内存布局](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f47624286ed2bdc02b77baa780d68868.png)

如上图所示，展示了SPI控制器内存分配的布局结构，控制器结构体和私有数据在同一块连续内存中。

> [!note]+ 背景知识 这种内存分配方式的优势是一次分配、统一管理，避免了内存碎片化，同时简化了释放流程

### 4.2 控制器参数设置

#### 4.2.1 基本参数配置

在probe函数中，需要为控制器设置各种工作参数，就像是为新设备配置"工作规范"：

```c
    // SPI控制器基本参数设置
    ctlr->auto_runtime_pm = true;                       // 启用自动运行时电源管理
    ctlr->bus_num = pdev->id;                           // 设置总线编号
    
    // 支持的模式位设置
    ctlr->mode_bits = SPI_CPOL | SPI_CPHA | SPI_LOOP | SPI_LSB_FIRST | SPI_CS_HIGH;
    
    // 主从模式特殊配置
    if (slave_mode) {
        ctlr->mode_bits |= SPI_NO_CS;                   // 从模式不需要片选控制
        ctlr->slave_abort = rockchip_spi_slave_abort;   // 从模式中止函数
    } else {
        ctlr->flags = SPI_MASTER_GPIO_SS;               // 主模式使用GPIO片选
    }
    
    // 设备配置参数
    ctlr->num_chipselect = ROCKCHIP_SPI_MAX_CS_NUM;     // 支持的片选数量
    ctlr->dev.of_node = pdev->dev.of_node;              // 关联设备树节点
    
    // 支持的数据位宽
    ctlr->bits_per_word_mask = SPI_BPW_MASK(16) | SPI_BPW_MASK(8) | SPI_BPW_MASK(4);
    
    // 速度限制设置
    ctlr->min_speed_hz = rs->freq / BAUDR_SCKDV_MAX;    // 最小速度
    ctlr->max_speed_hz = min(rs->freq / BAUDR_SCKDV_MIN, MAX_SCLK_OUT); // 最大速度
```

#### 4.2.2 模式位详解

**SPI模式位配置**：

- `SPI_CPOL/SPI_CPHA`：支持四种标准SPI模式
- `SPI_LOOP`：环回模式，用于测试
- `SPI_LSB_FIRST`：支持低位优先传输
- `SPI_CS_HIGH`：支持高电平有效的片选信号

#### 4.2.3 速度计算机制

**速度计算公式**：

- 最小速度 = spiclk频率 / 最大分频比
- 最大速度 = min(spiclk频率 / 最小分频比, 硬件最大限制)

### 4.3 回调函数配置

#### 4.3.1 核心回调函数设置

回调函数就像是控制器的"技能包"，定义了控制器能够执行的各种操作：

```c
    // 核心回调函数配置
    ctlr->set_cs = rockchip_spi_set_cs;                 // 片选控制函数
    ctlr->setup = rockchip_spi_setup;                   // 设备初始化函数
    ctlr->cleanup = rockchip_spi_cleanup;               // 清理函数
    ctlr->transfer_one = rockchip_spi_transfer_one;     // 单次传输函数
    ctlr->max_transfer_size = rockchip_spi_max_transfer_size; // 最大传输大小函数
    ctlr->handle_err = rockchip_spi_handle_err;         // 错误处理函数
```

#### 4.3.2 回调函数功能说明

**set_cs函数**：

- 功能：控制片选信号的有效性
- 调用时机：每次传输开始和结束时
- 重要性：确保设备选择的正确性

**transfer_one函数**：

- 功能：执行单次SPI传输操作
- 调用时机：核心层需要传输数据时
- 重要性：这是数据传输的核心实现

**setup函数**：

- 功能：初始化SPI设备的工作参数
- 调用时机：设备首次使用或参数变更时
- 重要性：确保设备工作在正确的模式下

#### 4.3.3 错误处理机制

```c
    ctlr->handle_err = rockchip_spi_handle_err;
```

错误处理函数负责处理传输过程中的各种异常情况，包括超时、CRC错误、DMA错误等。

### 4.4 控制器注册

#### 4.4.1 注册函数调用

完成所有配置后，需要将控制器注册到SPI子系统：

```c
    // 控制器注册
    ret = devm_spi_register_controller(&pdev->dev, ctlr);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to register controller\n");
        goto err_free_dma_rx;
    }

    return 0;   // 注册成功
```

![pinctrl配置和控制器注册](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/deca4190518e781d7547a72d77d29d40.png)

如上图所示，展示了SPI驱动中获取pinctrl配置、查找高速引脚状态，并向内核注册SPI控制器的过程，包含了相应的错误处理逻辑。

#### 4.4.2 注册流程详解

**spi_register_controller**：SPI控制器注册的核心函数，完成控制器的系统集成

```c
// drivers/spi/spi.c
int spi_register_controller(struct spi_controller *ctlr)
{
    struct device           *dev = ctlr->dev.parent;   // 获取父设备
    struct boardinfo        *bi;
    int                     status;
    int                     id, first_dynamic;

    if (!dev)
        return -ENODEV;

    // 检查操作函数是否完整
    status = spi_controller_check_ops(ctlr);
    if (status)
        return status;

    // 总线编号分配逻辑
    if (ctlr->bus_num >= 0) {     
        // 如果指定了控制器编号，使用指定的编号
        mutex_lock(&board_lock);
        id = idr_alloc(&spi_master_idr, ctlr, ctlr->bus_num,
                ctlr->bus_num + 1, GFP_KERNEL);
        mutex_unlock(&board_lock);
        if (WARN(id < 0, "couldn't get idr"))
            return id == -ENOSPC ? -EBUSY : id;
        ctlr->bus_num = id;
    } else if (ctlr->dev.of_node) { 
        // 设备树模式：从设备树获取总线编号
        id = of_alias_get_id(ctlr->dev.of_node, "spi");
        if (id >= 0) {
            ctlr->bus_num = id;
            mutex_lock(&board_lock);
            id = idr_alloc(&spi_master_idr, ctlr, ctlr->bus_num,
                           ctlr->bus_num + 1, GFP_KERNEL);
            mutex_unlock(&board_lock);
            if (WARN(id < 0, "couldn't get idr"))
                return id == -ENOSPC ? -EBUSY : id;
        }
    }
    
    if (ctlr->bus_num < 0) { 
        // 动态分配总线编号
        first_dynamic = of_alias_get_highest_id("spi");
        if (first_dynamic < 0)
            first_dynamic = 0;
        else
            first_dynamic++;

        mutex_lock(&board_lock);
        id = idr_alloc(&spi_master_idr, ctlr, first_dynamic,
                       0, GFP_KERNEL);
        mutex_unlock(&board_lock);
        if (WARN(id < 0, "couldn't get idr"))
            return id;
        ctlr->bus_num = id;
    }
```

### 4.5 队列初始化

#### 4.5.1 队列初始化入口

**spi_controller_initialize_queue**：初始化SPI控制器的消息队列处理机制

```c
// drivers/spi/spi.c
static int spi_controller_initialize_queue(struct spi_controller *ctlr)
{
    int ret;

    // 设置传输函数为队列传输模式
    ctlr->transfer = spi_queued_transfer;
    
    // 如果没有设置单消息传输函数，使用默认的
    if (!ctlr->transfer_one_message)
        ctlr->transfer_one_message = spi_transfer_one_message;

    // 初始化队列
    ret = spi_init_queue(ctlr);
    if (ret) {
        dev_err(&ctlr->dev, "problem initializing queue\n");
        goto err_init_queue;
    }
    
    // 标记队列已初始化
    ctlr->queued = true;
    
    // 启动队列
    ret = spi_start_queue(ctlr);
    if (ret) {
        dev_err(&ctlr->dev, "problem starting queue\n");
        goto err_start_queue;
    }

    return 0;

err_start_queue:
    spi_destroy_queue(ctlr);
err_init_queue:
    return ret;
}
```

#### 4.5.2 队列初始化作用

**队列机制的优势**：

- **异步处理**：支持异步的SPI传输操作
- **优先级管理**：可以实现传输请求的优先级调度
- **资源优化**：避免CPU在传输等待中的浪费
- **并发支持**：支持多个SPI设备的并发操作

### 4.6 工作线程创建

#### 4.6.1 工作线程初始化

**spi_init_queue**：初始化消息队列和工作线程，建立异步处理机制

```c
// drivers/spi/spi.c
static int spi_init_queue(struct spi_controller *ctlr)
{
    struct sched_param param = { .sched_priority = MAX_RT_PRIO - 1 };

    ctlr->running = false;                              // 初始状态为未运行
    ctlr->busy = false;                                 // 初始状态为不忙

    // 初始化内核工作线程
    kthread_init_worker(&ctlr->kworker);      
    
    // 创建并启动内核线程处理工作
    ctlr->kworker_task = kthread_run(kthread_worker_fn, &ctlr->kworker,   
                                     "%s", dev_name(&ctlr->dev));
    if (IS_ERR(ctlr->kworker_task)) {
        dev_err(&ctlr->dev, "failed to create message pump task\n");
        return PTR_ERR(ctlr->kworker_task);
    }
    
    // 初始化消息处理工作，设置工作函数为spi_pump_messages
    kthread_init_work(&ctlr->pump_messages, spi_pump_messages);   

    // 如果控制器要求实时优先级，设置调度策略
    if (ctlr->rt) {
        dev_info(&ctlr->dev,
                "will run message pump with realtime priority\n");
        sched_setscheduler(ctlr->kworker_task, SCHED_FIFO, &param);
    }

    return 0;
}
```

#### 4.6.2 工作线程机制

**工作线程的特点**：

- **专用线程**：每个SPI控制器有独立的工作线程
- **异步执行**：工作线程独立于调用者执行
- **实时支持**：可以配置为实时优先级，降低传输延迟
- **消息驱动**：通过工作队列机制处理传输请求

### 4.7 队列启动

#### 4.7.1 队列启动实现

**spi_start_queue**：启动消息队列处理机制，开始接受传输请求

```c
// drivers/spi/spi.c
static int spi_start_queue(struct spi_controller *ctlr)
{
    unsigned long flags;

    // 获取自旋锁并关中断（防止竞争条件）
    spin_lock_irqsave(&ctlr->queue_lock, flags);  

    // 检查队列状态
    if (ctlr->running || ctlr->busy) {
        spin_unlock_irqrestore(&ctlr->queue_lock, flags);
        return -EBUSY;                              // 队列已经在运行
    }

    // 设置队列为运行状态
    ctlr->running = true;
    ctlr->cur_msg = NULL;                           // 清空当前消息
    
    spin_unlock_irqrestore(&ctlr->queue_lock, flags);

    // 将消息处理工作加入工作队列
    kthread_queue_work(&ctlr->kworker, &ctlr->pump_messages);  

    return 0;
}
```

#### 4.7.2 队列启动后的工作流程

**消息处理流程**：

1. **工作入队**：`kthread_queue_work`将pump_messages工作加入队列
2. **工作线程执行**：工作线程执行`spi_pump_messages`函数
3. **消息处理**：从消息队列中取出并处理SPI传输请求
4. **循环处理**：处理完成后继续等待新的工作

> [!example]+ 动手实践 可以通过查看/proc/[pid]/comm来确认SPI工作线程的创建情况，线程名通常为"spi[bus_num]"

![控制器注册流程图](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7f35c3ec9ee80839991f8a4a4794a618.png)

如上图所示，展示了SPI控制器注册的完整流程，包括参数检查、总线编号分配、设备注册和队列初始化等步骤。

## 5 数据传输实现

### 5.1 transfer_one实现

#### 5.1.1 transfer_one函数的核心作用

`transfer_one`函数是SPI控制器驱动的"心脏"，它负责执行实际的硬件数据传输操作。就像是工厂的"生产线操作员"，接收上级指令后直接操控机器完成生产任务。

#### 5.1.2 函数原型和返回值

**rockchip_spi_transfer_one**：执行单次SPI传输操作的核心函数

```c
// drivers/spi/spi-rockchip.c
static int rockchip_spi_transfer_one(struct spi_controller *ctlr,
                                    struct spi_device *spi,
                                    struct spi_transfer *xfer)
{
    struct rockchip_spi *rs = spi_controller_get_devdata(ctlr);
    int ret = 0;
    
    // 保存当前传输结构体
    rs->xfer = xfer;
    
    // 根据传输长度选择传输方式
    if (xfer->len > rs->fifo_len) {
        // 大数据量使用DMA传输
        ret = rockchip_spi_transfer_dma(rs, xfer);
    } else {
        // 小数据量使用CPU传输
        ret = rockchip_spi_transfer_cpu(rs, xfer);
    }
    
    return ret;
}
```

**返回值含义**：

- **0**：传输完成，可以继续处理下一个传输
- **1**：传输正在进行中，需要等待完成通知
- **负值**：传输发生错误，错误码表示具体错误类型

#### 5.1.3 传输方式选择策略

**传输方式判断逻辑**：

- 传输长度 > FIFO长度：使用DMA传输，减轻CPU负担
- 传输长度 ≤ FIFO长度：使用CPU传输，避免DMA开销

这种设计就像是选择不同的运输方式：小件货物用快递，大件货物用卡车。

### 5.2 消息处理机制

#### 5.2.1 消息处理入口

**spi_pump_messages**：内核工作线程的消息处理函数，负责从队列中取出并处理SPI消息

```c
// drivers/spi/spi.c
static void spi_pump_messages(struct kthread_work *work)
{
    // 通过container_of宏从work结构体获取包含它的spi_controller结构体指针
    struct spi_controller *ctlr =
        container_of(work, struct spi_controller, pump_messages);

    // 调用实际的消息处理函数，参数true表示在内核线程上下文中
    __spi_pump_messages(ctlr, true);
}
```

#### 5.2.2 消息处理核心逻辑

**__spi_pump_messages**：消息队列处理的核心实现，管理整个传输调度过程

```c
// drivers/spi/spi.c (关键部分)
static void __spi_pump_messages(struct spi_controller *ctlr, bool in_kthread)
{
    unsigned long flags;
    bool was_busy = false;
    int ret;

    // 加锁保护队列访问
    spin_lock_irqsave(&ctlr->queue_lock, flags);

    // 检查是否有消息正在处理
    if (ctlr->cur_msg) {                 
        spin_unlock_irqrestore(&ctlr->queue_lock, flags);
        return;                         // 有消息在处理，直接返回
    }

    // 检查队列是否为空
    if (list_empty(&ctlr->queue) || !ctlr->running) {  
        if (!ctlr->busy) {                         
            spin_unlock_irqrestore(&ctlr->queue_lock, flags);
            return;                     // 队列为空且控制器不忙，直接返回
        }
        // 执行清理工作...
    }

    // 从消息队列获取第一个spi_message
    ctlr->cur_msg = list_first_entry(&ctlr->queue, struct spi_message, queue);

    // 将消息从队列中删除
    list_del_init(&ctlr->cur_msg->queue);  
    
    if (ctlr->busy)           
        was_busy = true;
    else
        ctlr->busy = true;              // 设置控制器为忙碌状态
        
    spin_unlock_irqrestore(&ctlr->queue_lock, flags);

    // 准备传输硬件
    if (!was_busy && ctlr->prepare_transfer_hardware) {
        ret = ctlr->prepare_transfer_hardware(ctlr);
        if (ret) {
            ctlr->cur_msg->status = ret;
            spi_finalize_current_message(ctlr);
            return;
        }
    }

    // 执行消息传输
    ret = ctlr->transfer_one_message(ctlr, ctlr->cur_msg);   
    if (ret) {
        dev_err(&ctlr->dev,
                "failed to transfer one message from queue\n");
    }
}
```

#### 5.2.3 消息处理状态机

**消息处理的状态转换**：

1. **空闲状态**：等待新消息到达
2. **准备状态**：准备硬件资源
3. **传输状态**：执行实际的数据传输
4. **完成状态**：清理资源，通知完成
5. **回到空闲**：继续处理下一个消息

### 5.3 片选控制

#### 5.3.1 片选控制的重要性

片选控制就像是"设备选择器"，确保SPI总线上只有一个设备被激活进行通信。这对于多设备SPI总线来说至关重要。

#### 5.3.2 片选控制实现

**spi_transfer_one_message**：单个消息传输的完整实现，包含片选控制逻辑

```c
// drivers/spi/spi.c (关键片选控制部分)
static int spi_transfer_one_message(struct spi_controller *ctlr,
                                    struct spi_message *msg)
{
    struct spi_transfer *xfer;
    bool keep_cs = false;
    int ret = 0;

    // 设置片选信号为有效
    spi_set_cs(msg->spi, true);

    // 逐个处理消息中的传输
    list_for_each_entry(xfer, &msg->transfers, transfer_list) {
        
        if (xfer->tx_buf || xfer->rx_buf) {
            reinit_completion(&ctlr->xfer_completion);

            // 调用实际的传输函数
            ret = ctlr->transfer_one(ctlr, msg->spi, xfer);
            if (ret < 0) {
                dev_err(&msg->spi->dev,
                        "SPI transfer failed: %d\n", ret);
                goto out;
            }

            // 如果返回值大于0，表示传输正在进行，需要等待完成
            if (ret > 0) {
                ret = spi_transfer_wait(ctlr, msg, xfer);
                if (ret < 0)
                    msg->status = ret;
            }
        }

        // 处理传输延迟
        if (xfer->delay_usecs) {
            u16 us = xfer->delay_usecs;
            if (us <= 10)
                udelay(us);             // 短延迟使用忙等待
            else
                usleep_range(us, us + DIV_ROUND_UP(us, 10)); // 长延迟使用睡眠
        }

        // 处理片选信号变化
        if (xfer->cs_change) {
            if (list_is_last(&xfer->transfer_list, &msg->transfers)) {
                keep_cs = true;         // 最后一个传输，保持片选有效
            } else {
                spi_set_cs(msg->spi, false);    // 释放片选
                udelay(10);                     // 短暂延迟
                spi_set_cs(msg->spi, true);     // 重新激活片选
            }
        }

        msg->actual_length += xfer->len;        // 累计实际传输长度
    }

out:
    // 传输完成，释放片选信号（除非设置了keep_cs）
    if (ret != 0 || !keep_cs)
        spi_set_cs(msg->spi, false);

    // 设置消息状态
    if (msg->status == -EINPROGRESS)
        msg->status = ret;

    // 错误处理
    if (msg->status && ctlr->handle_err)
        ctlr->handle_err(ctlr, msg);

    // 释放资源
    spi_res_release(ctlr, msg);

    // 完成当前消息处理
    spi_finalize_current_message(ctlr);

    return ret;
}
```

#### 5.3.3 片选控制要点

**片选控制的关键时机**：

1. **消息开始**：激活目标设备的片选信号
2. **传输间隙**：根据cs_change标志决定是否变更片选
3. **消息结束**：释放片选信号，让总线回到空闲状态

**片选控制的设计考虑**：

- **原子性**：整个消息的传输过程中保持片选有效
- **灵活性**：支持传输间的片选变化需求
- **兼容性**：支持不同极性的片选信号

#### 5.3.4 传输完成处理

**spi_finalize_current_message**：传输完成后的清理和通知处理

```c
// drivers/spi/spi.c
void spi_finalize_current_message(struct spi_controller *ctlr)
{
    struct spi_message *mesg;
    unsigned long flags;
    
    spin_lock_irqsave(&ctlr->queue_lock, flags);
    mesg = ctlr->cur_msg;
    ctlr->cur_msg = NULL;                       // 清空当前消息
    ctlr->cur_msg_prepared = false;
    
    // 将消息处理工作重新加入队列，继续处理下一个消息
    kthread_queue_work(&ctlr->kworker, &ctlr->pump_messages);
    spin_unlock_irqrestore(&ctlr->queue_lock, flags);

    mesg->state = NULL;
    
    // 调用完成回调函数通知上层
    if (mesg->complete)
        mesg->complete(mesg->context);
}
```

这个函数完成以下重要工作：

- 清理当前消息状态
- 重新启动消息处理循环
- 调用用户注册的完成回调函数

> [!tip]+ 实用技巧 在设计SPI传输时，合理设置cs_change和delay_usecs参数可以优化传输效率和兼容性

---

## 本章总结

通过本章的深入学习，我们全面掌握了SPI控制器驱动的开发方法和实现原理。

### 核心知识点回顾

**控制器驱动概念理解**：

- 控制器驱动是SPI子系统的核心组件，负责硬件抽象和资源管理
- 基于平台驱动模型实现，充分利用设备树和Linux设备模型的优势
- 通过回调函数机制与SPI核心层进行交互

**RK3568控制器分析**：

- 设备树配置包含了控制器的完整硬件描述信息
- 支持多种Rockchip芯片的统一驱动架构
- 具备高性能、多功能的SPI传输特性

**控制器初始化掌握**：

- probe函数完成从硬件发现到可用状态的全部初始化
- 资源获取包括内存映射、时钟配置、中断和DMA设置
- 完善的错误处理机制确保初始化的可靠性

**注册流程精通**：

- spi_controller分配采用一体化内存管理策略
- 参数设置涵盖模式位、速度限制、回调函数等多个方面
- 队列机制和工作线程实现了高效的异步传输处理

**数据传输实现**：

- transfer_one函数是硬件操作的核心实现
- 消息处理机制通过状态机实现复杂的传输调度
- 片选控制确保多设备环境下的通信可靠性

### 设计思想总结

**分层设计**：控制器驱动在硬件和软件之间建立了清晰的抽象层

**资源管理**：采用设备模型和自动资源管理，简化了驱动开发

**异步处理**：工作线程和消息队列机制提供了高效的异步处理能力

**错误处理**：完善的错误处理机制确保了系统的稳定性和可靠性

### 开发实践要点

- 正确理解和使用平台驱动模型是控制器驱动开发的基础
- 设备树配置的准确性直接影响驱动的工作效果
- 回调函数的实现质量决定了控制器的性能和稳定性
- 消息处理机制的优化对系统整体性能具有重要影响

> [!note]+ 后续学习指引 在掌握了控制器驱动的实现原理后，下一步可以学习SPI设备驱动的开发，了解如何基于控制器驱动开发具体的SPI设备应用

这些深入的控制器驱动知识为后续的SPI系统开发和优化提供了坚实的技术基础。理解了控制器驱动的工作原理，就能更好地开发高性能、高可靠性的SPI应用系统。