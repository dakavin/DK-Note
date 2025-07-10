
i2c 总线驱动由芯片厂商提供，如果我们使用 ST 官方提供的 Linux 内核， i2c 总线驱动已经保存在内核中，并且默认情况下已经编译进内核。

下面结合源码简单介绍 i2c 总线的运行机制。

- 1、注册 I2C 总线
    
- 2、将 I2C 驱动添加到 I2C 总线的驱动链表中
    
- 3、遍历 I2C 总线上的设备链表，根据 i2c_device_match 函数进行匹配，如果匹配调用i2c_device_probe 函数
    
- 4、 i2c_device_probe 函数会调用 I2C 驱动的 probe 函数

下图是i2c子系统驱动模型
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d81157e75a3263f79689a9d5a41603e0.png)


## 1 i2c 总线定义

```c

// I2C总线类型定义
struct bus_type i2c_bus_type = {
    .name       = "i2c",
    .match      = i2c_device_match,
    .probe      = i2c_device_probe,
    .remove     = i2c_device_remove,
    .shutdown   = i2c_device_shutdown,
};
EXPORT_SYMBOL_GPL(i2c_bus_type);
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7441784e3efc894504b11b60eb45a2a0.png)


i2c 总线维护着两个链表 (I2C 驱动、 I2C 设备)，管理 I2C 设备和 I2C 驱动的匹配和删除等

## 2 i2c 总线注册

linux 启动之后，默认执行 i2c_init。

在i2c_init接口中通过调用bus_register，进行i2c总线的注册。

```c
// I2C子系统初始化函数
static int __init i2c_init(void)
{
    int retval;
    
    ...
    
    retval = bus_register(&i2c_bus_type);
    if (retval)
        return retval;
    
    is_registered = true;
    
    ...
    
    retval = i2c_add_driver(&dummy_driver);
    if (retval)
        // 错误处理代码
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/938c4db76f3d261c6dec3ba19700bd65.png)


- bus_register 注册总线 i2c_bus_type，总线定义如上所示。
    
- i2c_add_driver 注册设备 dummy_driver。
    

## 3 i2c 设备和 i2c 驱动匹配规则

```c
// I2C设备匹配函数
static int i2c_device_match(struct device *dev, struct device_driver *drv)
{
    struct i2c_client   *client = i2c_verify_client(dev);
    struct i2c_driver   *driver;
    
    // 尝试OF风格匹配
    if (i2c_of_match_device(drv->of_match_table, client))
        return 1;
    
    // 然后尝试ACPI风格匹配
    if (acpi_driver_match_device(dev, drv))
        return 1;
    
    driver = to_i2c_driver(drv);
    
    // 最后进行I2C匹配
    if (i2c_match_id(driver->id_table, client))
        return 1;
    
    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/35c99af96849f6e2d4ae498524662f62.png)


- **of_driver_match_device：** 设备树匹配方式，比较 I2C 设备节点的 compatible 属性和of_device_id 中的 compatible 属性
    
- **acpi_driver_match_device：** ACPI 匹配方式
    
- **i2c_match_id：** i2c 总线传统匹配方式，比较 I2C 设备名字和 i2c 驱动的 id_table->name 字段是否相等
    

  

在 i2c 总线驱动代码源文件中，我们只简单介绍重要的几个点，如果感兴趣可自行阅读完整的 i2c驱动源码。通常情况下，看驱动程序首先要找到驱动的入口和出口函数，驱动入口和出口位于驱动的末尾，如下所示 :

驱动入口和出口函数 (内核源码/drivers/i2c/busses/i2c-rk3x.c) :

## 4 i2c_device_probe接口分析

该接口为i2c总线的probe，该接口一般也就是调用driver的probe接口，实现探测操作。

  

该接口的流程图如下，相对来说还是比较简单的，我们主要说些i2c模块相关的实现：

1. 首先根据dev->type是否为i2c_client_type来确定是否需要进行设备与驱动的绑定操作，这主要是因为i2c_adapter、i2c_client均会注册到i2c总线上，此处即为区分i2c_clinet、i2c_adapter。
    
2. 当区分i2c_client后，则直接调用i2c_driver->probe，而不是直接调用device_driver->probe，其实大多数模块device_driver->probe与其xxx_driver->probe是相同的。
    

  

## 5 注册 I2C 设备驱动

### 5.1 i2c_add_driver

I2C 设备的注册使用的函数为 i2c_add_driver， 被定义在内核源码的“include/linux/i2c.h”目录下， 具体内容如下所示：
```c
// I2C驱动添加宏定义
#define i2c_add_driver(driver) \
    i2c_register_driver(THIS_MODULE, driver)
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8fc7eba30254540fde126d23951b336c.png)


可以看到 i2c_add_driver 是一个宏定义， 这个宏定义的作用是为了简化注册 I2C 设备驱动程序的过程， 实际注册 I2C 设备的函数为 i2c_register_driver， 该函数定义在“drivers/i2c/i2c-co re-base.c” 文件中， 具体内容如下所示：

```C++
int i2c_register_driver(struct module *owner, struct i2c_driver *driver)
{
    int res;
    /* 在驱动模型初始化完成之前,无法注册驱动程序 */
    if (WARN_ON(!is_registered))
        return -EAGAIN;
    /* 将驱动程序添加到驱动核心的 i2c 驱动列表中 */
    driver->driver.owner = owner;
    driver->driver.bus = &i2c_bus_type;
    INIT_LIST_HEAD(&driver->clients);
    /* 当注册返回时,驱动核心会为所有匹配但尚未绑定的设备调用 probe() 函数 */
    res = driver_register(&driver->driver);
    if (res)
        return res;
    pr_debug("driver [%s] registered\n", driver->driver.name);
    /* 遍历所有已经存在的适配器 */
    i2c_for_each_dev(driver, __process_new_driver);
    return 0;
}
   
```

该函数的主要作用是将 I2C 设备驱动程序注册到驱动核心中,并初始化相关数据结构,这里传入的数据结构类型为 i2c_driver， 需要我们在驱动程序编写的时候进行填充， 该结构体定义在“include/linux/i2c.h” 头文件中， 具体内容如下所示：

```C++
struct i2c_driver {
    unsigned int class; // 驱动程序所属的设备类型
    int (*probe)(struct i2c_client *, const struct i2c_device_id *); // 探测并绑定设备的回调函数
    int (*remove)(struct i2c_client *); // 从设备上解绑驱动程序的回调函数
    int (*probe_new)(struct i2c_client *); // 新的探测设备并绑定的回调函数
    void (*shutdown)(struct i2c_client *); // 设备关闭时调用的回调函数
    void (*alert)(struct i2c_client *, enum i2c_alert_protocol protocol,
    unsigned int data); // 设备报警时调用的回调函数,格式和含义取决于所使用的协议
    int (*command)(struct i2c_client *client, unsigned int cmd, void *arg); // 用于执行设备特定功能的命令回调函数
    struct device_driver driver; // 设备驱动程序基础结构
    const struct i2c_device_id *id_table; // 与该驱动程序匹配的设备 ID 表
    int (*detect)(struct i2c_client *, struct i2c_board_info *); // 用于自动创建设备的探测回调函数
    const unsigned short *address_list; // 与该驱动程序匹配的设备地址列表
    struct list_head clients; // 与该驱动程序绑定的 I2C 设备列表
    bool disable_i2c_core_irq_mapping; // 禁用 I2C 核心中断映射的标志
};
```

在调用 i2c_add_driver 函数注册 I2C 设备之前， 需要先填充 i2c_driver 结构体， 然后实现的各种回调函数， 跟前面讲解的平台总线内容相同。

### 5.2 module_i2c_driver

```c
/**
 * module_i2c_driver() - 用于注册模块化I2C驱动的辅助宏
 * @__i2c_driver: i2c_driver结构体
 *
 * 用于不在模块初始化/退出时执行任何特殊操作的I2C驱动的辅助宏。
 * 这消除了大量样板代码。每个模块只能使用此宏一次，
 * 调用它会替换module_init()和module_exit()
 */
#define module_i2c_driver(__i2c_driver) \
    module_driver(__i2c_driver, i2c_add_driver, \
                  i2c_del_driver)
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1fb55310391c6420046ce6b20835b06f.png)


相当于这个宏：

i2c_add_driver(__i2c_driver);

i2c_del_driver(__i2c_driver)

## 6 注册 I2C 设备驱动案例

```c
// MAX8649设备ID表
static const struct i2c_device_id max8649_id[] = {
    { "max8649", 0 },
    { }
};
MODULE_DEVICE_TABLE(i2c, max8649_id);

// MAX8649驱动结构体
static struct i2c_driver max8649_driver = {
    .probe      = max8649_regulator_probe,
    .driver     = {
        .name   = "max8649",
    },
    .id_table   = max8649_id,
};

// MAX8649初始化函数
static int __init max8649_init(void)
{
    return i2c_add_driver(&max8649_driver);
}
subsys_initcall(max8649_init);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/87790f6385939c317421be8d2c0149f7.png)
