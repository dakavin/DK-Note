
## 1 层次：
下图是Linux系统中SPI子系统的分层架构图，展示了从用户空间应用程序，经过spidev驱动、SPI核心层、SPI控制器驱动，最终到达SPI硬件控制器和外设的完整软件栈结构
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/43d1b535ddd710bbce73ec00c2087f5a.png)


## 2 结构体关系图：

- `spi_master`(`spi_controller`)：对 `SoC`的 `SPI` 控制器的抽象
    
- `spi_bus_type`：`spi` 的 `bus_type`，代表了硬件上的 `SPI Bus`
    
- `spi_device`：`spi` 从设备
    
- `spi_driver`：`spi` 具体设备的驱动

下图是Linux内核SPI子系统的完整架构图，展示了从SPI控制器注册、设备管理、消息队列处理到底层硬件传输的整个数据流和控制流程，包括工作队列机制和hook函数的调用关系
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/08a649ebbcf7fc89794542a925ada9b5.png)


  

### 2.1 spi_controller 控制器

[03_SPI控制器驱动分析](03_SPI控制器驱动分析.md)

### 2.2 spi_bus_type 设备

需要关注match匹配规则：
```c
struct bus_type spi_bus_type = {                    // SPI总线类型结构体定义
    .name       = "spi",                            // 总线名称为"spi"
    .dev_groups = spi_dev_groups,                   // 设备组属性
    .match      = spi_match_device,                 // 设备匹配函数
    .uevent     = spi_uevent,                       // uevent事件处理函数
};
EXPORT_SYMBOL_GPL(spi_bus_type);                    // 导出符号供其他模块使用
```

```c
static int spi_match_device(struct device *dev, struct device_driver *drv)
{
    const struct spi_device *spi = to_spi_device(dev);      // 获取SPI设备结构体
    const struct spi_driver *sdrv = to_spi_driver(drv);     // 获取SPI驱动结构体

    /* check override first, and if set, only use the named driver */
    if (spi->driver_override)                                // 检查是否有驱动覆盖
        return strcmp(spi->driver_override, drv->name) == 0; // 比较覆盖驱动名称

    /* Attempt an OF style match */
    if (of_driver_match_device(dev, drv))                    // 尝试设备树匹配
        return 1;                                            // 匹配成功返回1

    /* Then try ACPI */
    if (acpi_driver_match_device(dev, drv))                  // 尝试ACPI匹配
        return 1;                                            // 匹配成功返回1

    if (sdrv->id_table)                                      // 检查驱动ID表
        return !!spi_match_id(sdrv->id_table, spi);         // 通过ID表匹配

    return strcmp(spi->modalias, drv->name) == 0;           // 最后通过modalias匹配
}
```

### 2.3 spi_device 设备

struct spi_device结构表示SPI设备，定义在include/linux/spi/spi.h中：

**struct spi_device**：SPI设备结构体，代表SPI总线上的一个从设备，包含设备属性、通信参数和硬件配置信息

```c
// include/linux/spi/spi.h
struct spi_device {
    struct device                          dev;                  // 设备模型基础结构
    struct spi_master                      *master;              // 关联的SPI主控制器
    u32                                    max_speed_hz;         // 最大通信速度
    u8                                     chip_select;          // 片选信号编号
    u8                                     bits_per_word;        // 每个字的位数
    u16                                    mode;                 // SPI通信模式
    int                                    irq;                  // 中断号
    int                                    cs_gpio;              // 芯片选择GPIO
};
```

- `dev`：内嵌的设备结构体，用于集成到Linux设备模型中，支持sysfs导出和设备管理
- `master`：指向控制此设备的SPI主控制器，建立设备与控制器的关联关系
- `max_speed_hz`：设备支持的最大时钟频率，单位为Hz，用于限制通信速度
- `chip_select`：片选信号的编号，用于在多设备总线上选择特定设备进行通信
- `bits_per_word`：每次传输的数据位宽，通常为8位、16位或32位
- `mode`：SPI通信模式，包括时钟极性(CPOL)和时钟相位(CPHA)设置
- `irq`：设备的中断请求号，用于异步事件通知和中断驱动的数据传输
- `cs_gpio`：片选信号对应的GPIO引脚号，用于软件控制片选信号

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2cfccfa7742e037cca109917a55e83e7.png)


一些没有意义的字段已被删除。结构中元素的含义如下。

- **master**：表示设备所连接的SPI控制器（总线）。
    
- **max_speed_hz**：该芯片（在当前开发板上）使用的最大时钟频率；该参数可以在驱动程序内更改。每次传输时可以使用spi_transfer.speed_hz改写该参数。稍后将讨论SPI传输。
    
- **chip_select**：用以启用需要与之通信的芯片，从而区分主控处理的芯片。chip_select默认为低电平有效。通过添加SPI_CS_HIGH标志，可以在模式中更改此行为。
    
- **irq**：这代表中断号（在开发板初始化文件中或通过DT注册为设备资源），应该将它传递给request_irq()来接收此设备的中断。
    
- **mode**：定义数据如何计时，设备驱动程序可以更改它。默认时，对于传输的每个字，数据时钟是最高有效位（MSB）优先。
    

这有4种SPI模式，这些模式在include/linux/spi/spi.h中根据以下宏在内核中定义：

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

- `SPI_CPHA`：时钟相位标志位，控制数据在时钟的上升沿还是下降沿采样，0表示第一个边沿采样，1表示第二个边沿采样
- `SPI_CPOL`：时钟极性标志位，控制空闲时时钟信号的电平状态，0表示空闲时为低电平，1表示空闲时为高电平
- `SPI_MODE_0`：标准SPI模式0，空闲时钟为低电平，第一个时钟边沿采样数据，对应原始MicroWire协议
- `SPI_MODE_1`：SPI模式1，空闲时钟为低电平，第二个时钟边沿采样数据
- `SPI_MODE_2`：SPI模式2，空闲时钟为高电平，第一个时钟边沿采样数据
- `SPI_MODE_3`：SPI模式3，空闲时钟为高电平，第二个时钟边沿采样数据

关于SPI模式，它们的构建有两个特征。

- CPOL：这是最初的时钟极性。 asqW``
    

0：初始时钟状态为低电平，第一个边沿是上升沿。

1：初始时钟状态为高电平，第一个状态为下降。

- CPHA：这是时钟相位，选择在哪个边沿采样数据。
    
      0：数据在下降沿（从高到低转换）锁存，而输出在上升沿改变。
    
      1：数据在上升沿（从低到高转换）锁存，并在下降沿输出。
    

|      |      |      |                                         |
| ---- | ---- | ---- | --------------------------------------- |
| Mode | CPOL | CPHA | kernel macro                            |
| 0    | 0    | 0    | #define SPI_MODE_0 (0\|0)               |
| 1    | 0    | 1    | #define SPI_MODE_1 (0\|SPI_CPHA)        |
| 2    | 1    | 0    | #define SPI_MODE_2 (SPI_CPOL\|0)        |
| 3    | 1    | 1    | #define SPI_MODE_3 (SPI_CPOL\|SPI_CPHA) |

### 2.4 spi_driver 驱动

**struct spi_driver**：SPI驱动程序结构体，定义SPI设备驱动的核心回调函数和设备匹配信息

```c
// include/linux/spi/spi.h
struct spi_driver {
    const struct spi_device_id             *id_table;            // 支持的设备ID表
    int                                    (*probe)(struct spi_device *spi);     // 设备探测函数
    int                                    (*remove)(struct spi_device *spi);    // 设备移除函数
    void                                   (*shutdown)(struct spi_device *spi);  // 设备关闭函数
    struct device_driver                   driver;               // 设备驱动基础结构
};
```

- `id_table`：指向spi_device_id数组的指针，定义此驱动支持的所有设备类型和名称
- `probe`：设备探测函数指针，当匹配的设备被发现时调用，用于初始化设备和分配资源
- `remove`：设备移除函数指针，当设备被移除或驱动卸载时调用，用于清理资源和停止设备
- `shutdown`：系统关闭时的设备处理函数，用于安全地关闭设备以防止数据丢失
- `driver`：内嵌的通用设备驱动结构体，提供与设备模型的集成接口

就像需要i2c_device_id处理I2C设备一样，对于SPI设备，必须使用spi_device_id，以便提供device_id数组来匹配设备。它在include/linux/mod_ devicetable.h中定义：

**struct spi_device_id**：SPI设备标识结构体，用于驱动程序识别和匹配特定的SPI设备

```c
// include/linux/mod_devicetable.h
struct spi_device_id {
    char                                   name[SPI_NAME_SIZE];  // 设备名称字符串
    kernel_ulong_t                         driver_data;          // 驱动私有数据
};
```

- `name`：设备名称字符串，长度限制为SPI_NAME_SIZE，用于与spi_device的modalias字段进行匹配
- `driver_data`：驱动程序的私有数据字段，通常存储设备特定的配置信息、功能标志或指向设备特定数据结构的指针

需要将数组嵌入struct spi_device_id，以便把驱动程序需要管理的设备ID通知SPI核心，并在该驱动程序结构上调用MODULE_DEVICE_TABLE宏。当然，该宏的第一个参数是设备所在总线的名称。在这个例子中，它是SPI：

案例：net/can/spi/mcp251x.c

**MCP251x CAN控制器设备ID表**：定义MCP251x系列CAN控制器的SPI设备标识表，用于驱动匹配和设备识别

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

**数组成员说明**：

- `"mcp2510"`：MCP2510 CAN控制器的设备名称，用于设备树或平台代码中的设备匹配
- `"mcp2515"`：MCP2515 CAN控制器的设备名称，最常用的MCP251x系列芯片型号
- `"mcp25625"`：MCP25625 CAN控制器的设备名称，集成了CAN收发器的增强型芯片
- `CAN_MCP251X_MCP25XX`：对应各芯片型号的枚举值，用于驱动程序内部区分不同芯片的功能特性
- `MODULE_DEVICE_TABLE`：向内核模块加载器注册设备表，支持热插拔和自动模块加载功能


