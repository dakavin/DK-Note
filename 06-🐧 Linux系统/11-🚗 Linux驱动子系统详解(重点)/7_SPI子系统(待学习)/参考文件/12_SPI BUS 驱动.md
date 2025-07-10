
## 1 数据结构关系图
如下图所示
![spi关系结构图.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ee61426884e9c8915d5bfcd31acaaeb3.png)

## 2 spi init
```c
// 图片1: SPI子系统初始化函数
static int __init spi_init(void)
{
    int status; // 状态返回值

    // 分配SPI缓冲区内存
    buf = kmalloc(SPI_BUFSIZ, GFP_KERNEL);
    if (!buf) {
        status = -ENOMEM; // 内存分配失败
        goto err0;
    }

    // 注册SPI总线类型
    status = bus_register(&spi_bus_type);
    if (status < 0)
        goto err1;

    // 注册SPI主控制器设备类
    status = class_register(&spi_master_class);
    if (status < 0)
        goto err2;

    // 如果启用了SPI从设备支持，注册从设备类
    if (IS_ENABLED(CONFIG_SPI_SLAVE)) {
        status = class_register(&spi_slave_class);
        if (status < 0)
            goto err3;
    }

    // 如果启用了设备树动态配置支持
    if (IS_ENABLED(CONFIG_OF_DYNAMIC))
        WARN_ON(of_reconfig_notifier_register(&spi_of_notifier));
    // 如果启用了ACPI支持
    if (IS_ENABLED(CONFIG_ACPI))
        WARN_ON(acpi_reconfig_notifier_register(&spi_acpi_notifier));

    return 0; // 初始化成功

err3:
    class_unregister(&spi_master_class); // 注销主设备类
err2:
    bus_unregister(&spi_bus_type);       // 注销总线类型
err1:
    kfree(buf);                          // 释放缓冲区
    buf = NULL;
err0:
    return status; // 返回错误状态
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/133caefb4a489a598dcc3e9cb82a2666.png)


### 2.1 总线的注册

```c
// 图片2: SPI总线类型结构体定义 struct bus_type spi_bus_type = { .name = "spi", // 总线名称 .dev_groups = spi_dev_groups, // 设备属性组 .match = spi_match_device, // 设备匹配函数 .uevent = spi_uevent, // 设备事件处理函数 }; EXPORT_SYMBOL_GPL(spi_bus_type); // 导出符号供GPL模块使用
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e5f738b861896cea5c09fc1524e9f010.png)

bus_register 详见[6_SysFS设备驱动管理](../../../08-🏗️%20Linux设备模型与设备树(重点)/1_Linux设备模型基础/6_SysFS设备驱动管理.md)

### 2.2 match

```c
// 图片3: SPI设备和驱动匹配函数
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
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/64c3b24075a06d806b7ad3c3d60cad28.png)


## 3 bus_type

详见：[7_总线详解与实践](../../../08-🏗️%20Linux设备模型与设备树(重点)/1_Linux设备模型基础/7_总线详解与实践.md)

## 4 spi_device

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

- `dev`：继承的标准设备结构体，用于设备模型管理
- `controller/master`：指向管理此设备的SPI控制器，master为兼容性字段
- `max_speed_hz`：设备支持的最大SPI时钟频率，单位为Hz
- `chip_select`：设备的片选信号编号，用于选择具体的SPI设备
- `bits_per_word`：每次传输的字长，通常为8、16或32位
- `mode`：SPI工作模式，包含时钟极性(CPOL)、时钟相位(CPHA)等配置
- `modalias`：设备的模块别名，用于驱动匹配和自动加载
- `driver_override`：强制指定使用的驱动名称，优先级最高


## 5 spi_driver

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

- `id_table`：支持的设备ID列表，用于设备与驱动的匹配
- `probe`：设备探测回调函数，当匹配的设备被发现时调用
- `remove`：设备移除回调函数，设备被移除时调用进行清理
- `shutdown`：系统关闭时的处理函数，可选实现
- `driver`：继承的标准设备驱动结构体，包含驱动基本信息