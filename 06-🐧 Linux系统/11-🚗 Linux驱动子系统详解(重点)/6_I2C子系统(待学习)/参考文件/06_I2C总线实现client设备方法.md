
📢 `Linux` 中的 `I2C` 也是按照平台总线模型设计的，既然也是按照平台总线模型设计的，是不是也分为一个`device` 和一个 `driver` 呢？但是 `I2C` 这里的 `device` 不叫 `device`,而是叫 `client`。在讲 `platform` 的时候就说过，`platform` 是虚拟出来的一条总线，目的是为了实现总线、设备、驱动框架。对于 `I2C` 而言，不需要虚拟出一条总线，直接使用 `I2C` 总线即可。同样，我们也是先从非设备树开始，先来看一下，在没有设备树之前我们是怎么实现的 `I2C` 的 `device` 部分，也就是 `client` 部分。然后再学习有了设备树之后，我们的 `client` 是怎么编写的。

## 非设备树实现 i2c client

下图是i2c_client构建过程数据结构关系图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/66d27d3e7ad09577b2d2e5e2cf4ff4db.png)


在没有使用设备树之前，我们使用的是 i2c_board_info 这个结构体来描述一个 I2C 设备的，i2c_board_info 这个结构体如下：

```C++
struct i2c_board_info
{
        char type[I2C_NAME_SIZE]; /* I2C 设备名字 */
        unsigned short flags; /* 标志 */
        unsigned short addr; /* I2C 器件地址 */
        void *platform_data;
        struct dev_archdata *archdata;
        struct device_node *of_node;
        struct fwnode_handle *fwnode;
        int irq;
```

在这个结构体里面，`type` 和 `addr` 这两个成员变量是必须要设置的，一个是 `I2C` 设备的名字，这个名字就是用来进行匹配用的，一个是 `I2C` 设备的器件地址，也可以使用宏：

```C++
#define I2C_BOARD_INFO(dev_type, dev_addr) \
.type = dev_type, .addr = (dev_addr)
```

可以看出， `I2C_BOARD_INFO` 宏其实就是设置 i2c_board_info 的 type 和 addr 这两个成员变量。 `I2C` 设备和驱动的匹配过程是由 `I2C` 核心来完成的，在 `Linux` 源码的 `drivers/i2c/i2c-core.c` 就是 `I2C` 的核心部分， `I2C` 核心提供了一些与具体硬件无关的 `API` 函数，如下：
**i2c_get_adapter**：获取指定编号的I2C适配器，用于在系统中查找并引用特定的I2C总线控制器

```c
// #include <linux/i2c.h>
struct i2c_adapter *i2c_get_adapter(int                     nr);           // I2C适配器编号
```

- `nr`：要获取的I2C适配器的编号，通常从0开始递增
- **返回值**：
    - 成功：返回指向i2c_adapter结构体的指针
    - 失败：返回NULL

**i2c_put_adapter**：释放I2C适配器的引用，与i2c_get_adapter配对使用来管理适配器的生命周期

```c
// #include <linux/i2c.h>
void i2c_put_adapter(struct i2c_adapter        *adap);          // 要释放的I2C适配器
```

- `adap`：要释放引用的I2C适配器指针，必须是通过i2c_get_adapter获取的有效指针
- **返回值**：
    - 成功：无返回值（void函数）
    - 失败：无返回值（void函数）

**i2c_new_device**：在指定的I2C适配器上创建新的I2C设备，将适配器与具体的I2C器件关联起来

```c
// #include <linux/i2c.h>
struct i2c_client *i2c_new_device(struct i2c_adapter         *adap,         // I2C适配器
                                  struct i2c_board_info const *info);        // 设备板级信息
```

- `adap`：目标I2C适配器指针，设备将在此适配器上创建
- `info`：指向i2c_board_info结构体的常量指针，包含设备的配置信息（如地址、名称等）
- **返回值**：
    - 成功：返回指向新创建的i2c_client结构体的指针
    - 失败：返回NULL

**i2c_unregister_device**：注销并删除一个I2C客户端设备，清理设备资源并从系统中移除

```c
// #include <linux/i2c.h>
void i2c_unregister_device(struct i2c_client   *client);        // 要注销的I2C客户端
```

- `client`：要注销的i2c_client结构体指针，必须是有效的已注册设备
- **返回值**：
    - 成功：无返回值（void函数）
    - 失败：无返回值（void函数）

i2c/i2c-core-base.c

### i2c_new_device

使用接收到的适配器和`i2c`板信息作为参数来创建`i2c`设备并进行注册。如果有匹配的驱动程序，它将调用该驱动程序的探针挂钩。

如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e82b19f2bbdc9140978faf3644cde9b0.png)


```C++
struct i2c_client *
i2c_new_device(struct i2c_adapter *adap, struct i2c_board_info const *info)
{
        struct i2c_client       *client;
        int                     status;

        client = kzalloc(sizeof *client, GFP_KERNEL);
        if (!client)
                return NULL;

        client->adapter = adap;

        client->dev.platform_data = info->platform_data;
        client->flags = info->flags;
        client->addr = info->addr;

        client->init_irq = info->irq;
        if (!client->init_irq)
                client->init_irq = i2c_dev_irq_from_resources(info->resources,
                                                         info->num_resources);
        client->irq = client->init_irq;

        strlcpy(client->name, info->type, sizeof(client->name));

        status = i2c_check_addr_validity(client->addr, client->flags);
        if (status) {
                dev_err(&adap->dev, "Invalid %d-bit I2C address 0x%02hx\n",
                        client->flags & I2C_CLIENT_TEN ? 10 : 7, client->addr);
                goto out_err_silent;
        }

        /* Check for address business */
        status = i2c_check_addr_ex(adap, i2c_encode_flags_to_addr(client));
        if (status)
                dev_err(&adap->dev,
                        "%d i2c clients have been registered at 0x%02x",
                        status, client->addr);

        client->dev.parent = &client->adapter->dev;
        client->dev.bus = &i2c_bus_type;
        client->dev.type = &i2c_client_type;
        client->dev.of_node = of_node_get(info->of_node);
        client->dev.fwnode = info->fwnode;

        i2c_dev_set_name(adap, client, info, status);

        if (info->properties) {
                status = device_add_properties(&client->dev, info->properties);
                if (status) {
                        dev_err(&adap->dev,
                                "Failed to add properties to client %s: %d\n",
                                client->name, status);
                        goto out_err_put_of_node;
                }
        }

        status = device_register(&client->dev);
        if (status)
                goto out_free_props;

        dev_dbg(&adap->dev, "client [%s] registered with bus id %s\n",
                client->name, dev_name(&client->dev));

        return client;

out_free_props:
        if (info->properties)
                device_remove_properties(&client->dev);
out_err_put_of_node:
        of_node_put(info->of_node);
out_err_silent:
        kfree(client);
        return NULL;
}
EXPORT_SYMBOL_GPL(i2c_new_device);
```

- 从代码行7分配i2c_client结构后，使用适配器指针和接收到的i2c板信息作为参数进行设置。
    
- 如果在代码行18〜19中没有irq信息，则从资源中检索它。
    
- 在代码行23中，将板信息中的名称设置为i2c_client的名称。
    
- 在代码行25〜33中检查设备地址是否有效。
    

使用10位地址时，该地址不得超过0x3ff。

使用7位地址时，该地址不得超过0x01〜0x7f的范围。

  

### i2c client
```c
// I2C客户端结构体定义
struct i2c_client {
    unsigned short flags;                    // 设备标志，详见下文
    unsigned short addr;                     // 芯片地址 - 注意：7位地址存储在低7位中
    char name[I2C_NAME_SIZE];               // 设备名称
    struct i2c_adapter *adapter;            // 我们所在的适配器
    struct device dev;                      // 设备结构体
    int init_irq;                          // 初始化时设置的中断号
    int irq;                               // 设备发出的中断号
    struct list_head detected;             // 检测到的设备链表
#if IS_ENABLED(CONFIG_I2C_SLAVE)
    i2c_slave_cb_t slave_cb;               // 从模式的回调函数
#endif
};
```
- addr，flags：分别表示从设备的地址和访问操作标志，最初源头为由开发人员构造和初始化的i2c_board_info结构体
- adapter：在从设备与适配器匹配后，在后端的i2c_new_device中被初始化

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f770c5d3cf6ac68a4974d62edc58b09c.png)


## 设备树实现 i2c

在使用了设备树以后，就不用这么复杂了，使用设备树的时候只要在对应的 `I2C` 节点下创建相应设备的节点即可，比如我想添加一个触摸芯片 `FT5X06` 的设备，我就可以在对应的 `I2C` 的节点下这样写，如下所示：

```C++
&i2c1 {
        status = "okay"
        ft5x06:ft5x06@38 {
                status = "disabled";
                 compatible = "edt,edt-ft5306";
                 reg = <0x38>;
                touch-gpio = <&gpio0 RK_PB5 IRQ_TYPE_EDGE_RISING>;
                interrupt-parent = <&gpio0>;
                interrupts = <RK_PB5 IRQ_TYPE_LEVEL_LOW>;
                reset-gpios = <&gpio0 RK_PB6 GPIO_ACTIVE_LOW>;
                touchscreen-size-x = <800>;
                touchscreen-size-y = <1280>;
                touch_type = <1>;
        };

        gt9xx:gt9xx_ts@14 {
                compatible = "goodix,gt9xx";
                reg = <0x14>;
                interrupt-parent = <&gpio0>;
                interrupts = <RK_PB5 IRQ_TYPE_LEVEL_LOW>;
                reset-gpios = <&gpio0 RK_PB6 GPIO_ACTIVE_LOW>;
                touch-gpio = <&gpio0 RK_PB5 IRQ_TYPE_EDGE_RISING>;
                status = "disabled";
                tp-size = <911>;
                max-x = <1024>;
                max-y = <600>;
        
        };
};
```

- 第 `3` 行触摸屏所使用的 `FT5x06` 芯片节点，挂载 `I2C-2` 节点下；“`@`”后面的“`38`”就是 `ft5x06` 的 `I2C` 器件地址
    
- 第 `5` 行 `compatible` 用于和驱动程序的 `compatible` 匹配；
    
- 第 `6` 行 `reg` 属性描述 `ft5x` 的器件地址为 `0x38`；
    
- 第 `8` 行 `interrupt-parent` 属性描述中断 `IO` 对应的 `GPIO` 组为 `GPIO0`；
    
- 第 `9` 行 `interrupts` 属性描述中断 `IO` 对应的是 `GPIO0` 组别的 `B` 组的 `5` 号引脚；
    
- 第 `10` 行 `reset-gpios` 属性描述复位 `IO` 对应的 `GPIO0` 组别的 `B` 组的 `6` 号引脚；
    

因为我们的开发板默认是设备树的镜像， 我们进入到开发板的`/sys/bus/i2c/devices/`目录下，因为通过查找原理图发现我们屏幕使用的是`i2c2`，但这里我们要进入到 `0-0038`，查看 `name` 为 `ft5x0x_ts`

  

### `i2c_client` 结构体的生成

```c
//设备树节点示例 - vdd_cpu电压调节器配置
&i2c0 {
    status = "okay";
    
    vdd_cpu: tcs4525@1c {
        compatible = "tcs,tcs452x";
        reg = <0x1c>;
        vin-supply = <&vcc5v0_sys>;
        regulator-compatible = "fan53555-reg";
        regulator-name = "vdd_cpu";
        regulator-min-microvolt = <712500>;
        regulator-max-microvolt = <1390000>;
        regulator-ramp-delay = <2300>;
        fcs,suspend-voltage-selector = <1>;
        regulator-boot-on;
        regulator-always-on;
        regulator-state-mem {
            regulator-off-in-suspend;
        };
    };
};
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1e3a26aa19b3574af5c8a8fb55a9413c.png)


### `i2c_driver` 驱动

regulator/fan53555.c

```c
// FAN53555电压调节器驱动定义
static struct i2c_driver fan53555_regulator_driver = {
    .driver = {
        .name = "fan53555-regulator",
        .of_match_table = of_match_ptr(fan53555_dt_ids),
    },
    .probe = fan53555_regulator_probe,
    .shutdown = fan53555_regulator_shutdown,
    .id_table = fan53555_id,
};

// 使用module_i2c_driver宏注册驱动
module_i2c_driver(fan53555_regulator_driver);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d95a5f763fd6e0f3e103d919c37fc0a8.png)


#### module_i2c_driver

linux/i2c.h
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
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f6972f7378999b061461d40c6345754c.png)


#### fan53555_regulator_probe

```c
// FAN53555电压调节器探测函数
static int fan53555_regulator_probe(struct i2c_client *client,
                                    const struct i2c_device_id *id)
{
    struct device_node *np = client->dev.of_node;
    struct fan53555_device_info *di;
    struct fan53555_platform_data *pdata;
    struct regulator_config config = { };
    unsigned int val;
    int ret;

    // 分配设备信息结构体内存
    di = devm_kzalloc(&client->dev, sizeof(struct fan53555_device_info),
                      GFP_KERNEL);
    if (!di)
        return -ENOMEM;

    di->desc.of_map_mode = fan53555_map_mode;

    // 获取平台数据
    pdata = dev_get_platdata(&client->dev);
    if (!pdata)
        pdata = fan53555_parse_dt(&client->dev, np, &di->desc);
    
    // 更多初始化代码...
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cd4b1a3aa0e91bcfd94a96fd5a6a1966.png)


## 案例

### 非设备树方法

net/ethernet/intel/igb/igb_hwmon.c
```c
// I350传感器板级信息定义
static struct i2c_board_info i350_sensor_info = {
    I2C_BOARD_INFO("i350bb", (0xf8 >> 1)),
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8ed6a76d2091d7db8850d7e4cb6e3c70.png)

```c
// 初始化I2C客户端
client = i2c_new_device(&adapter->i2c_adap, &i350_sensor_info);
if (client == NULL) {
    dev_info(&adapter->pdev->dev,
             "Failed to create new i2c device.\n");
    rc = -ENODEV;
    goto exit;
}
adapter->i2c_client = client;
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/64a423bc268de538613b99a1f96def19.png)


### 设备树方法

在i2c5节点追加ap3216c子节点
i2c_client:(device)
```c
// 设备树节点示例 - I2C5总线上的AP3216C光传感器
&i2c5 {
    ap3216c@1e {
        compatible = "alientek,ap3216c";
        reg = <0x1e>;
    };
};
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1eba1092a84e872439d30226217f5f20.png)
