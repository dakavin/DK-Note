I2C模块中，有两个设备类型：i2c adapter、i2c client，

- 其中i2c client可以理解为设备驱动模型中的设备，与注册到总线上的i2c驱动绑定
    
- 而i2c adapter也属于注册到i2c总线上的设备，但其不需要与i2c驱动绑定。其主要为i2c驱动提供访问i2c设备的方法，同时一个i2c设备需要依附于i2c adapter（在设备模型中，i2c adapter是依附于其上的i2c client的父device）。
    

## 1 I2c adapter的方法

```C++


/*
 * The following structs are for those who like to implement new bus drivers:
 * i2c_algorithm is the interface to a class of hardware solutions which can
 * be addressed using the same bus algorithms - i.e. bit-banging or the PCF8584
 * to name two of the most common.
 */
struct i2c_algorithm {
        /* If an adapter algorithm can't do I2C-level access, set master_xfer
           to NULL. If an adapter algorithm can do SMBus access, set
           smbus_xfer. If set to NULL, the SMBus protocol is simulated
           using common I2C messages */
        /* master_xfer should return the number of messages successfully
           processed, or a negative value on error */
        int (*master_xfer)(struct i2c_adapter *adap, struct i2c_msg *msgs,
                           int num);
        int (*smbus_xfer) (struct i2c_adapter *adap, u16 addr,
                           unsigned short flags, char read_write,
                           u8 command, int size, union i2c_smbus_data *data);
 
        /* To determine what the adapter supports */
        u32 (*functionality) (struct i2c_adapter *);
};
```

rk3x_i2c_algorithm 结构体定义如下：
```c
// RK3X I2C算法结构体定义
static const struct i2c_algorithm rk3x_i2c_algorithm = {
    .master_xfer     = rk3x_i2c_xfer,
    .functionality   = rk3x_i2c_func,
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dc48fb67fe74dff9844ddc6af5ec7212.png)


我们先来看一下. functionality， functionality用于返回此I2C适配器支持什么样的通信协议，在这里 functionality 就是 rk3x_i2c_func 函数， rk3x_i2c_func 函数内容如下：

```C++
static u32 rk3x_i2c_func(struct i2c_adapter *adap)
{
    return I2C_FUNC_I2C | I2C_FUNC_SMBUS_EMUL | I2C_FUNC_PROTOCOL_MANGLING;
}  
```

## 2 控制器代码

rk3568.dtsi
```c
// 设备树I2C配置示例
i2c2: i2c@fe5b0000 {
    compatible = "rockchip,rk3399-i2c";
    reg = <0x0 0xfe5b0000 0x0 0x1000>;
    clocks = <&cru CLK_I2C2>, <&cru PCLK_I2C2>;
    clock-names = "i2c", "pclk";
    interrupts = <GIC_SPI 40 IRQ_TYPE_LEVEL_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&i2c2m0_xfer>;
    #address-cells = <1>;
    #size-cells = <0>;
    status = "disabled";
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a0f0db4a42f0dc5a634f7454a124fb70.png)


重点关注 i2c1 节点的 compatible 属性值，因为通过 compatible 属性值可以在 Linux 源码里面找到对应的驱动文件。这里 i2c1 节点的 compatible 属性值“rockchip,rk3399-i2c”，在 Linux 源码中搜索这个字符串即可找到对应的驱动文件。 RK3568 的 I2C 适配器驱动驱动文件为drivers/i2c/busses/i2c-rk3x.c，在此文件中有如下内容
```c
// RK3X I2C平台驱动结构体
static struct platform_driver rk3x_i2c_driver = {
    .probe   = rk3x_i2c_probe,
    .remove  = rk3x_i2c_remove,
    .driver  = {
        .name = "rk3x-i2c",
        .of_match_table = rk3x_i2c_match,
        .pm = &rk3x_i2c_pm_ops,
    },
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a73bee020fab4fae39f4944fe25b2c5c.png)

## 3 控制器代码

### 3.1 struct rk3x_i2c

芯片厂商对i2c的抽象：
```c
// RK3X I2C结构体定义
struct rk3x_i2c {
    struct i2c_adapter adap;                     // I2C适配器
    struct device *dev;
    const struct rk3x_i2c_soc_data *soc_data;

    // 硬件资源
    void __iomem *regs;
    struct clk *clk;
    struct clk *pclk;
    struct notifier_block clk_rate_nb;

    // 设置
    struct i2c_timings t;

    // 同步和通知
    spinlock_t lock;
    wait_queue_head_t wait;
    bool busy;

    // 当前消息
    struct i2c_msg *msg;
    u8 addr;
    unsigned int mode;
    bool is_last_msg;

    // I2C状态机
    enum rk3x_i2c_state state;
    unsigned int processed;
    int error;
    unsigned int suspended:1;

    struct notifier_block i2c_restart_nb;
    bool system_restarting;
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/61eb29b648652947a0a5b21c1cf91ab1.png)

### 3.2 rk3x_i2c_probe

当设备和驱动匹配成功以后 rk3x_i2c_probe 函数就会执行， rk3x_i2c_probe 函数就会完成I2C 适配器初始化工作。

rk3x_i2c_probe 函数主要的工作就是两点：

1. 化 i2c_adapter，设置 i2c_algorithm 为 rk3x_i2c_algorithm，最后向 Linux 内核
    

注册 i2c_adapter。

2. 初始化 I2C1 控制器的相关寄存器。 rk3x_i2c_algorithm 包含 I2C1 适配器与 I2C 设
    

备的通信函数 master_xfer。

```C++
static int rk3x_i2c_probe(struct platform_device *pdev)
{
        struct device_node *np = pdev->dev.of_node;
        const struct of_device_id *match;
        struct rk3x_i2c *i2c;
        struct resource *mem;
        int ret = 0;
        u32 value;
        int irq;
        unsigned long clk_rate;

        i2c = devm_kzalloc(&pdev->dev, sizeof(struct rk3x_i2c), GFP_KERNEL);
        if (!i2c)
                return -ENOMEM;

        match = of_match_node(rk3x_i2c_match, np);
        i2c->soc_data = match->data;

        /* use common interface to get I2C timing properties */
        i2c_parse_fw_timings(&pdev->dev, &i2c->t, true);

        strlcpy(i2c->adap.name, "rk3x-i2c", sizeof(i2c->adap.name));
        i2c->adap.owner = THIS_MODULE;
        i2c->adap.algo = &rk3x_i2c_algorithm;
        i2c->adap.retries = 3;
        i2c->adap.dev.of_node = np;
        i2c->adap.algo_data = i2c;
        i2c->adap.dev.parent = &pdev->dev;

        i2c->dev = &pdev->dev;

        spin_lock_init(&i2c->lock);
        init_waitqueue_head(&i2c->wait);

        i2c->i2c_restart_nb.notifier_call = rk3x_i2c_restart_notify;
        i2c->i2c_restart_nb.priority = 128;
        ret = register_pre_restart_handler(&i2c->i2c_restart_nb);
        if (ret) {
                dev_err(&pdev->dev, "failed to setup i2c restart handler.\n");
                return ret;
        }

        mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
        i2c->regs = devm_ioremap_resource(&pdev->dev, mem);
        if (IS_ERR(i2c->regs))
                return PTR_ERR(i2c->regs);

        /*
         * Switch to new interface if the SoC also offers the old one.
         * The control bit is located in the GRF register space.
         */
        if (i2c->soc_data->grf_offset >= 0) {
                struct regmap *grf;

                grf = syscon_regmap_lookup_by_phandle(np, "rockchip,grf");
                if (!IS_ERR(grf)) {
                        int bus_nr;

                        /* Try to set the I2C adapter number from dt */
                        bus_nr = of_alias_get_id(np, "i2c");
                        if (bus_nr < 0) {
                                dev_err(&pdev->dev, "rk3x-i2c needs i2cX alias");
                                return -EINVAL;
                        }

                        if (i2c->soc_data == &rv1108_soc_data && bus_nr == 2)
                                /* rv1108 i2c2 set grf offset-0x408, bit-10 */
                                value = BIT(26) | BIT(10);
                        else if (i2c->soc_data == &rv1126_soc_data &&
                                 bus_nr == 2)
                                /* rv1126 i2c2 set pmugrf offset-0x118, bit-4 */
                                value = BIT(20) | BIT(4);
                        else
                                /* rk3xxx 27+i: write mask, 11+i: value */
                                value = BIT(27 + bus_nr) | BIT(11 + bus_nr);

                        ret = regmap_write(grf, i2c->soc_data->grf_offset,
                                           value);
                        if (ret != 0) {
                                dev_err(i2c->dev, "Could not write to GRF: %d\n",
                                        ret);
                                return ret;
                        }
                }
        }

        /* IRQ setup */
        irq = platform_get_irq(pdev, 0);
        if (irq < 0) {
                dev_err(&pdev->dev, "cannot find rk3x IRQ\n");
                return irq;
        }

        ret = ccc(&pdev->dev, irq, rk3x_i2c_irq,
                               0, dev_name(&pdev->dev), i2c);
        if (ret < 0) {
                dev_err(&pdev->dev, "cannot request IRQ\n");
                return ret;
        }

        platform_set_drvdata(pdev, i2c);

        if (i2c->soc_data->calc_timings == rk3x_i2c_v0_calc_timings) {
                /* Only one clock to use for bus clock and peripheral clock */
                i2c->clk = devm_clk_get(&pdev->dev, NULL);
                i2c->pclk = i2c->clk;
        } else {
                i2c->clk = devm_clk_get(&pdev->dev, "i2c");
                i2c->pclk = devm_clk_get(&pdev->dev, "pclk");
        }

        if (IS_ERR(i2c->clk)) {
                ret = PTR_ERR(i2c->clk);
                if (ret != -EPROBE_DEFER)
                        dev_err(&pdev->dev, "Can't get bus clk: %d\n", ret);
                return ret;
        }
        if (IS_ERR(i2c->pclk)) {
                ret = PTR_ERR(i2c->pclk);
                if (ret != -EPROBE_DEFER)
                        dev_err(&pdev->dev, "Can't get periph clk: %d\n", ret);
                return ret;
        }

        ret = clk_prepare(i2c->clk);
        if (ret < 0) {
                dev_err(&pdev->dev, "Can't prepare bus clk: %d\n", ret);
                return ret;
        }
        ret = clk_prepare(i2c->pclk);
        if (ret < 0) {
                dev_err(&pdev->dev, "Can't prepare periph clock: %d\n", ret);
                goto err_clk;
        }

        i2c->clk_rate_nb.notifier_call = rk3x_i2c_clk_notifier_cb;
        ret = clk_notifier_register(i2c->clk, &i2c->clk_rate_nb);
        if (ret != 0) {
                dev_err(&pdev->dev, "Unable to register clock notifier\n");
                goto err_pclk;
        }

        clk_rate = clk_get_rate(i2c->clk);
        rk3x_i2c_adapt_div(i2c, clk_rate);

        ret = i2c_add_adapter(&i2c->adap);
        if (ret < 0)
                goto err_clk_notifier;

        return 0;

err_clk_notifier:
        clk_notifier_unregister(i2c->clk, &i2c->clk_rate_nb);
err_pclk:
        clk_unprepare(i2c->pclk);
err_clk:
        clk_unprepare(i2c->clk);
        return ret;
}
```

  

结构体的填充：
```c
// I2C适配器初始化代码片段
static int rk3x_i2c_probe(struct platform_device *pdev)
{
    // ... 其他初始化代码 ...
    
    strlcpy(i2c->adap.name, "rk3x-i2c", sizeof(i2c->adap.name));
    i2c->adap.owner = THIS_MODULE;
    i2c->adap.algo = &rk3x_i2c_algorithm;           // 设置I2C算法
    i2c->adap.retries = 3;
    i2c->adap.dev.of_node = np;
    i2c->algo_data = i2c;
    i2c->adap.dev.parent = &pdev->dev;

    ret = i2c_add_adapter(&i2c->adap);              // 添加I2C适配器
    if (ret < 0)
        goto err_clk_notifier;

    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d803a5a7967163613327ed5d8918f991.png)
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/007b2a3e4aa248084826252a3d3a67dd.png)


中断的解析和注册：
```c
// IRQ设置
static int rk3x_i2c_probe(struct platform_device *pdev)
{
    // ... 其他代码 ...
    
    // IRQ设置
    irq = platform_get_irq(pdev, 0);
    if (irq < 0) {
        dev_err(&pdev->dev, "cannot find rk3x IRQ\n");
        return irq;
    }

    ret = devm_request_irq(&pdev->dev, irq, rk3x_i2c_irq,
                          0, dev_name(&pdev->dev), i2c);
    if (ret < 0) {
        dev_err(&pdev->dev, "cannot request IRQ\n");
        return ret;
    }

    // ... 其他代码 ...
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2e790c241cd226c858e574e0d98b91c7.png)


  

### 3.3 &rk3x_i2c_algorithm

前面已经描述过了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dc48fb67fe74dff9844ddc6af5ec7212.png)


busses/i2c-rk3x.c

- 关中断:spin_lock_irqsave
    
- 时钟使能
    
- rk3x_i2c_setup
    
- rk3x_i2c_start
    
- spin_unlock_irqrestore
    
- spin_lock_irqsave
    
- i2c_readl
    
- i2c_writel
    

  

重点来看一下 rk3x_i2c_xfer 函数，因为最终就是通过此函数来完成与 I2C 设备通信的， 此函数内容如下：

```C++
static int rk3x_i2c_xfer(struct i2c_adapter *adap,
                         struct i2c_msg *msgs, int num)
{
        struct rk3x_i2c *i2c = (struct rk3x_i2c *)adap->algo_data;
        unsigned long timeout, flags;
        u32 val;
        int ret = 0;
        int i;

        if (i2c->suspended)
                return -EACCES;

        spin_lock_irqsave(&i2c->lock, flags);

        clk_enable(i2c->clk);
        clk_enable(i2c->pclk);

        i2c->is_last_msg = false;

        /*
         * Process msgs. We can handle more than one message at once (see
         * rk3x_i2c_setup()).
         */
        for (i = 0; i < num; i += ret) {
                ret = rk3x_i2c_setup(i2c, msgs + i, num - i);

                if (ret < 0) {
                        dev_err(i2c->dev, "rk3x_i2c_setup() failed\n");
                        break;
                }

                if (i + ret >= num)
                        i2c->is_last_msg = true;

                rk3x_i2c_start(i2c);

                spin_unlock_irqrestore(&i2c->lock, flags);

                timeout = wait_event_timeout(i2c->wait, !i2c->busy,
                                             msecs_to_jiffies(WAIT_TIMEOUT));

                spin_lock_irqsave(&i2c->lock, flags);

                if (timeout == 0) {
                        dev_err(i2c->dev, "timeout, ipd: 0x%02x, state: %d\n",
                                i2c_readl(i2c, REG_IPD), i2c->state);

                        /* Force a STOP condition without interrupt */
                        rk3x_i2c_disable_irq(i2c);
                        val = i2c_readl(i2c, REG_CON) & REG_CON_TUNING_MASK;
                        val |= REG_CON_EN | REG_CON_STOP;
                        i2c_writel(i2c, val, REG_CON);

                        i2c->state = STATE_IDLE;

                        ret = -ETIMEDOUT;
                        break;
                }

                if (i2c->error) {
                   ret = i2c->error;
                        break;
                }
        }

        rk3x_i2c_disable_irq(i2c);
        rk3x_i2c_disable(i2c);

        clk_disable(i2c->pclk);
        clk_disable(i2c->clk);

        spin_unlock_irqrestore(&i2c->lock, flags);

        return ret < 0 ? ret : num;
}
```

- i2c_writel
    
- i2c_readl
    

### 3.4 rk3x_i2c_restart_notify

  

### 3.5 register_pre_restart_handler

  

  

## 4 i2c_add_adapter

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/346e1eab67bc3d8e55f516145c903bf4.png)


向 linux 系统注册一个 i2c 适配器
```c
//linux 系统自动设置 i2c 适配器编号 (adapter->nr)
int i2c_add_adapter(struct i2c_adapter *adapter)
//手动设置 i2c 适配器编号 (adapter->nr)
int i2c_add_numbered_adapter(struct i2c_adapter *adapter)
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/70a0d9d2cd710a25256e16c1ed516455.png)


文件：`drivers/i2c/i2c-core-base.c`
```c
/**
 * i2c_add_adapter - 当其总线编号不重要或者当其总线编号由设备树别名指定时，
 *                  用于声明I2C适配器。
 * 例如动态添加的USB链路或PCI插件卡的I2C适配器基础。
 * 
 * 当此函数返回零时，新的总线编号被分配并存储在adap->nr中，
 * 指定的适配器变为可用于客户端。否则，返回负的errno值。
 */
int i2c_add_adapter(struct i2c_adapter *adapter)
{
    struct device *dev = &adapter->dev;
    int id;

    if (dev->of_node) {
        id = of_alias_get_id(dev->of_node, "i2c");
        if (id >= 0) {
            adapter->nr = id;
            return __i2c_add_numbered_adapter(adapter);
        }
    }

    mutex_lock(&core_lock);
    id = idr_alloc(&i2c_adapter_idr, adapter,
                   __i2c_first_dynamic_bus_num, 0, GFP_KERNEL);
    mutex_unlock(&core_lock);
    if (WARN(id < 0, "couldn't get idr"))
        return id;

    adapter->nr = id;

    return i2c_register_adapter(adapter);           // 注册适配器
}
EXPORT_SYMBOL(i2c_add_adapter);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/68eb31445a17501b303e2ad59dff5351.png)


  

通过接收带有指定的`i2c`适配器`ID`号的`ID`来注册`i2c`适配器设备。然后，注册在相应的`i2c`总线中检测到的`i2c`设备。

  

idr_alloc

### 4.1 i2c_register_adapter

i2c-core-base.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2408c49f4758f40d3c300fff18b1480f.png)


```C++
static int i2c_register_adapter(struct i2c_adapter *adap)
{
        int res = -EINVAL;

        /* Can't register until after driver model init */
        if (WARN_ON(!is_registered)) {
                res = -EAGAIN;
                goto out_list;
        }

        /* Sanity checks */
        if (WARN(!adap->name[0], "i2c adapter has no name"))
                goto out_list;

        if (!adap->algo) {
                pr_err("adapter '%s': no algo supplied!\n", adap->name);
                goto out_list;
        }

        if (!adap->lock_ops)
                adap->lock_ops = &i2c_adapter_lock_ops;

        rt_mutex_init(&adap->bus_lock);
        rt_mutex_init(&adap->mux_lock);
        mutex_init(&adap->userspace_clients_lock);
        INIT_LIST_HEAD(&adap->userspace_clients);

        /* Set default timeout to 1 second if not already set */
        if (adap->timeout == 0)
                adap->timeout = HZ;

        /* register soft irqs for Host Notify */
        res = i2c_setup_host_notify_irq_domain(adap);
        if (res) {
                pr_err("adapter '%s': can't create Host Notify IRQs (%d)\n",
                       adap->name, res);
                goto out_list;
        }

        dev_set_name(&adap->dev, "i2c-%d", adap->nr);
        adap->dev.bus = &i2c_bus_type;
        adap->dev.type = &i2c_adapter_type;
        res = device_register(&adap->dev);
        if (res) {
                pr_err("adapter '%s': can't register device (%d)\n", adap->name, res);
                goto out_list;
        }

        res = of_i2c_setup_smbus_alert(adap);
        if (res)
                goto out_reg;

        dev_dbg(&adap->dev, "adapter [%s] registered\n", adap->name);

        pm_runtime_no_callbacks(&adap->dev);
        pm_suspend_ignore_children(&adap->dev, true);
        pm_runtime_enable(&adap->dev);

#ifdef CONFIG_I2C_COMPAT
        res = class_compat_create_link(i2c_adapter_compat_class, &adap->dev,
                                       adap->dev.parent);
        if (res)
                dev_warn(&adap->dev,
                         "Failed to create compatibility class link\n");
#endif

        i2c_init_recovery(adap);

        /* create pre-declared device nodes */
        of_i2c_register_devices(adap);   //重要
        i2c_acpi_install_space_handler(adap);
        i2c_acpi_register_devices(adap);

        if (adap->nr < __i2c_first_dynamic_bus_num)
                i2c_scan_static_board_info(adap);

        /* Notify drivers */
        mutex_lock(&core_lock);
        bus_for_each_drv(&i2c_bus_type, NULL, adap, __process_new_adapter);
        mutex_unlock(&core_lock);

        return 0;

out_reg:
        init_completion(&adap->dev_released);
        device_unregister(&adap->dev);
        wait_for_completion(&adap->dev_released);
out_list:
        mutex_lock(&core_lock);
        idr_remove(&i2c_adapter_idr, adap->nr);
        mutex_unlock(&core_lock);
        return res;
}
```

### 4.2 `__i2c_add_numbered_adapter`

i2c/i2c-core-base.c
```c
/**
 * i2c_add_numbered_adapter - i2c_add_numbered_adapter，其中nr永远不为-1
 * @adap: 要注册的适配器（其中adap->nr已初始化）
 * 上下文：可以休眠
 * 
 * 有关详细信息，请参见i2c_add_numbered_adapter()。
 */
static int __i2c_add_numbered_adapter(struct i2c_adapter *adap)
{
    int id;

    mutex_lock(&core_lock);
    id = idr_alloc(&i2c_adapter_idr, adap, adap->nr, adap->nr + 1, GFP_KERNEL);
    mutex_unlock(&core_lock);
    if (WARN(id < 0, "couldn't get idr"))
        return id == -ENOSPC ? -EBUSY : id;

    return i2c_register_adapter(adap);
}

```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/794cba55c12d676fbb85fe6005da1244.png)


### 4.3 i2c_setup_host_notify_irq_domain
```c
// I2C设置主机通知IRQ域
static int i2c_setup_host_notify_irq_domain(struct i2c_adapter *adap)
{
    struct irq_domain *domain;

    if (!i2c_check_functionality(adap, I2C_FUNC_SMBUS_HOST_NOTIFY))
        return 0;

    domain = irq_domain_create_linear(adap->dev.fwnode,
                                     I2C_ADDR_7BITS_COUNT,
                                     &i2c_host_notify_irq_ops, adap);
    if (!domain)
        return -ENOMEM;

    adap->host_notify_domain = domain;

    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dd476db4176636f358c60b2744d79830.png)


  

### 4.4 of_i2c_setup_smbus_alert

i2c-core-smbus.c
```c
// I2C设置SMBus警报
#if IS_ENABLED(CONFIG_I2C_SMBUS) && IS_ENABLED(CONFIG_OF)
int of_i2c_setup_smbus_alert(struct i2c_adapter *adapter)
{
    struct i2c_client *client;
    int irq;

    irq = of_property_match_string(adapter->dev.of_node, "interrupt-names",
                                   "smbus_alert");
    if (irq == -EINVAL || irq == -ENODATA)
        return 0;
    else if (irq < 0)
        return irq;

    client = i2c_setup_smbus_alert(adapter, NULL);
    if (!client)
        return -ENODEV;

    return 0;
}
EXPORT_SYMBOL_GPL(of_i2c_setup_smbus_alert);
#endif
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/75328232701a06bf7be2b7a82fcc1c71.png)


### 4.5 pm_suspend_ignore_children
```c
// PM挂起忽略子设备函数
static inline void pm_suspend_ignore_children(struct device *dev, bool enable)
{
    dev->power.ignore_children = enable;
}
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/275f47ea2aa4d1f136da247c303c1dbb.png)


  

### 4.6 of_i2c_register_devices 重要

i2c/i2c-core-of.c

```c
// 注册设备树中的I2C设备
void of_i2c_register_devices(struct i2c_adapter *adap)
{
    struct device_node *bus, *node;
    struct i2c_client *client;

    // 只有在适配器有节点指针设置时才注册子设备
    if (!adap->dev.of_node)
        return;

    dev_dbg(&adap->dev, "of_i2c: walking child nodes\n");

    bus = of_get_child_by_name(adap->dev.of_node, "i2c-bus");
    if (!bus)
        bus = of_node_get(adap->dev.of_node);

    for_each_available_child_of_node(bus, node) {
        if (of_node_test_and_set_flag(node, OF_POPULATED))
            continue;

        client = of_i2c_register_device(adap, node);
        if (IS_ERR(client)) {
            dev_err(&adap->dev,
                    "Failed to create I2C device for %pOF\n",
                    node);
            of_node_clear_flag(node, OF_POPULATED);
        }
    }

    of_node_put(bus);
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ed9894d2b7cc7a5e3d20966e4faff1c1.png)

```c
// 注册单个I2C设备
static struct i2c_client *of_i2c_register_device(struct i2c_adapter *adap,
                                                  struct device_node *node)
{
    struct i2c_client *client;
    struct i2c_board_info info;
    int ret;

    dev_dbg(&adap->dev, "of_i2c: register %pOF\n", node);

    ret = of_i2c_get_board_info(&adap->dev, node, &info);
    if (ret)
        return ERR_PTR(ret);

    client = i2c_new_device(adap, &info);
    if (!client) {
        dev_err(&adap->dev, "of_i2c: Failure registering %pOF\n", node);
        return ERR_PTR(-EINVAL);
    }

    return client;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b62412067317f800178d3eaefda5e066.png)

```c
// 从设备树节点获取I2C板级信息
int of_i2c_get_board_info(struct device *dev, struct device_node *node,
                          struct i2c_board_info *info)
{
    u32 addr;
    int ret;

    memset(info, 0, sizeof(*info));

    if (of_modalias_node(node, info->type, sizeof(info->type)) < 0) {
        dev_err(dev, "of_i2c: modalias failure on %pOF\n", node);
        return -EINVAL;
    }

    ret = of_property_read_u32(node, "reg", &addr);
    if (ret) {
        dev_err(dev, "of_i2c: invalid reg on %pOF\n", node);
        return ret;
    }

    if (addr & I2C_TEN_BIT_ADDRESS) {
        addr &= ~I2C_TEN_BIT_ADDRESS;
        info->flags |= I2C_CLIENT_TEN;
    }

    if (addr & I2C_OWN_SLAVE_ADDRESS) {
        addr &= ~I2C_OWN_SLAVE_ADDRESS;
        info->flags |= I2C_CLIENT_SLAVE;
    }

    info->addr = addr;
    info->of_node = node;
    info->fwnode = of_fwnode_handle(node);

    if (of_property_read_bool(node, "host-notify"))
        info->flags |= I2C_CLIENT_HOST_NOTIFY;

    if (of_get_property(node, "wakeup-source", NULL))
        info->flags |= I2C_CLIENT_WAKE;

    return 0;
}
EXPORT_SYMBOL_GPL(of_i2c_get_board_info);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3a56ca6a2cea4794866657ee2a8fd7be.png)


### 4.7 i2c_acpi_install_space_handler

i2c-core-acpi.c

  

### 4.8 i2c_acpi_register_devices

i2c-core-acpi.c

  

### 4.9 bus_for_each_drv

base/bus.c
```c
// 遍历总线上的每个驱动程序并调用@fn函数
// 我们遍历属于@bus的每个驱动程序，并为每个驱动程序调用@fn。
// 如果@fn返回除0以外的任何值，我们就会中断并返回它。
// 如果@start不为NULL，我们将其用作列表的头部。
// 
// 注意：我们不返回返回非零值的驱动程序，也不会为该驱动程序保持引用计数的增量。
// 如果调用者需要知道该信息，它必须设置它在回调中。如果它还必须确保不增加引用计数，
// 这样它在返回给调用者之前不会消失。
int bus_for_each_drv(struct bus_type *bus, struct device_driver *start,
                     void *data, int (*fn)(struct device_driver *, void *))
{
    struct klist_iter i;
    struct device_driver *drv;
    int error = 0;

    if (!bus)
        return -EINVAL;

    klist_iter_init_node(&bus->p->klist_drivers, &i,
                         start ? &start->p->knode_bus : NULL);
    while ((drv = next_driver(&i)) && !error)
        error = fn(drv, data);
    klist_iter_exit(&i);
    return error;
}
EXPORT_SYMBOL_GPL(bus_for_each_drv);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4cb7499839806689597a17cf122601ab.png)


### 4.10 `__process_new_adapter`

```c
// I2C适配器注册完成后的初始化恢复
static int i2c_register_adapter(struct i2c_adapter *adap)
{
    // ... 其他初始化代码 ...

    i2c_init_recovery(adap);

    // 创建预声明的设备节点
    of_i2c_register_devices(adap);
    i2c_acpi_install_space_handler(adap);
    i2c_acpi_register_devices(adap);

    if (adap->nr < __i2c_first_dynamic_bus_num)
        i2c_scan_static_board_info(adap);

    // 通知驱动程序
    mutex_lock(&core_lock);
    bus_for_each_drv(&i2c_bus_type, NULL, adap, __process_new_adapter);
    mutex_unlock(&core_lock);

    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fb29b42964c940b3c9b5a83ff4989da4.png)


i2c/i2c-core-base.c

```c
// 处理新适配器
static int __process_new_adapter(struct device_driver *d, void *data)
{
    return i2c_do_add_adapter(to_i2c_driver(d), data);
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e6383e3a085e5c332f1e3f94a240de32.png)

```c
// 向适配器添加驱动程序
static int i2c_do_add_adapter(struct i2c_driver *driver,
                              struct i2c_adapter *adap)
{
    // 检测该总线上支持的设备，并实例化它们
    i2c_detect(adap, driver);

    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3bc61667fb51f731198e94645e095652.png)


i2c/i2c-core-base.c

```c
// I2C设备检测函数
static int i2c_detect(struct i2c_adapter *adapter, struct i2c_driver *driver)
{
    const unsigned short *address_list;
    struct i2c_client *temp_client;
    int i, err = 0;
    int adap_id = i2c_adapter_id(adapter);

    address_list = driver->address_list;
    if (!driver->detect || !address_list)
        return 0;

    // 如果适配器丢失了基于类的实例化，则发出警告
    if (adapter->class == I2C_CLASS_DEPRECATED) {
        dev_dbg(&adapter->dev,
                "This adapter dropped support for I2C classes and won't auto-detect %s devices anymore. "
                "If you need it, check 'Documentation/i2c/instantiating-devices' for alternatives.\n",
                driver->driver.name);
        return 0;
    }

    // 如果类不匹配，则在此处停止
    if (!(adapter->class & driver->class))
        return 0;

    // 设置临时客户端以帮助检测回调
    temp_client = kzalloc(sizeof(struct i2c_client), GFP_KERNEL);
    if (!temp_client)
        return -ENOMEM;
    temp_client->adapter = adapter;

    for (i = 0; address_list[i] != I2C_CLIENT_END; i += 1) {
        dev_dbg(&adapter->dev,
                "found normal entry for adapter %d, addr 0x%02x\n",
                adap_id, address_list[i]);
        temp_client->addr = address_list[i];
        err = i2c_detect_address(temp_client, driver);
        if (unlikely(err))
            break;
    }

    kfree(temp_client);
    return err;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ab2a50434b038f2f2fdab4795e20153d.png)

```c
// 检测特定地址的I2C设备
static int i2c_detect_address(struct i2c_client *temp_client,
                              struct i2c_driver *driver)
{
    struct i2c_board_info info;
    struct i2c_adapter *adapter = temp_client->adapter;
    int addr = temp_client->addr;
    int err;

    // 确保地址有效
    err = i2c_check_7bit_addr_validity_strict(addr);
    if (err) {
        dev_warn(&adapter->dev, "Invalid probe address 0x%02x\n",
                 addr);
        return err;
    }

    // 如果已经在使用，跳过，无需编码标志
    if (i2c_check_addr_busy(adapter, addr))
        return 0;

    // 确保在此地址有某些东西
    if (!i2c_default_probe(adapter, addr))
        return 0;

    // 最后调用自定义检测函数
    memset(&info, 0, sizeof(struct i2c_board_info));
    info.addr = addr;
    err = driver->detect(temp_client, &info);
    if (err) {
        // 如果检测失败返回-ENODEV。我们捕获它
        // 这里，就像这不是一个错误一样。
        return err == -ENODEV ? 0 : err;
    }

    // 一致性检查
    if (info.type[0] == '\0') {
        dev_err(&adapter->dev,
                "%s detection function provided no name for 0x%02x\n",
                driver->driver.name, addr);
    } else {
        struct i2c_client *client;

        // 检测成功，实例化设备
        if (adapter->class & I2C_CLASS_DEPRECATED)
            dev_warn(&adapter->dev,
                     "This adapter will soon drop class based instantiation of devices. "
                     "Please make sure client 0x%02x gets instantiated by other means. "
                     "Check 'Documentation/i2c/instantiating-devices' for details.\n",
                     info.type, info.addr);

        dev_dbg(&adapter->dev, "Creating %s at 0x%02x\n",
                info.type, info.addr);
        client = i2c_new_device(adapter, &info);
        if (client)
            list_add_tail(&client->detected, &driver->clients);
        else
            dev_err(&adapter->dev, "Failed creating %s at 0x%02x\n",
                    info.type, info.addr);
    }
    return 0;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/90ce5d1587f72e28d84be2cf8975685d.png)


to_i2c_driver

```c
// I2C驱动转换宏定义 - linux/i2c.h 第307行
#define to_i2c_driver(d) container_of(d, struct i2c_driver, driver)
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/887675d82de3b58b3a9ca33a884ef40b.png)
