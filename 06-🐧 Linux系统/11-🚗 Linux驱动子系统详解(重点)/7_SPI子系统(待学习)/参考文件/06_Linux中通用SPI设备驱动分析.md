
参考资料：

- 内核驱动：`drivers\spi\spidev.c`
    
- 内核提供的测试程序：`tools\spi\spidev_fdx.c`
    
- 内核文档：`Documentation\spi\spidev`
    

  

而跟I2C 设备类似， 在Linux内核中也有着通用 SPI 设备驱动， 在本章节将会讲解通用 SPI 设备驱动的使用， 并讲解如何在应用程序中通过 ioctl 对 SPI 进行配置和使用。

宏定义：

arch/arm64/configs/lubancat2_defconfig

CONFIG_SPI_SPIDEV

## 1 内核和设备树配置

通用 SPI 设备驱动在迅为提供的 Linux 内核中默认已经勾选了， 具体路径如下所示：

```C
> Device Drivers
    > SPI support  
```

除了内核支持之外， 还需要修改设备树， 由于之前已经使能了 SPI0， 所以这直接修改之前编写的 mcp2515 设备树节点， 具体设备树为 ：
```dts
&spi3 {
    status = "okay";  // 启用SPI3控制器
    pinctrl-names = "default", "high_speed";  // 定义两种引脚状态模式
    pinctrl-0 = <&spi3m1_cs0 &spi3m1_pins>;  // 默认模式的引脚配置
    pinctrl-1 = <&spi3m1_cs0 &spi3m1_pins_hs>;  // 高速模式的引脚配置
    
    spidev@0 {  // 通用SPI设备，使用片选CS0
        compatible = "rockchip,spidev";  // Rockchip平台的SPI设备驱动
        reg = <0>;  // SPI片选信号CS0
        spi-max-frequency = <24000000>;  // 最大SPI通信频率24MHz
        status = "okay";  // 启用此设备
    };
};
```

这个设备树配置定义了：
- SPI3控制器的启用和多种引脚状态配置
- 一个通用的SPI设备（spidev），支持用户空间直接访问
- 支持最高24MHz的高速SPI通信

## 2 设备节点

开发板启动之后， 如果存在/dev/spidev0.0 设备节点， 证明设备树及内核配置正确， 如下图所示：

```C
/dev/spidev3.0
```

- /dev/spidev3.0 表示一个 SPI 总线上的具体设备。 3.0 是一个标识符， 用于区分系统中的不同 SPI 控制器和设备。 这个标识符由两部分组成：
    
- 第一个数字 3： 表示 SPI 总线的编号。 一个系统中可能有多个 SPI 控制器， 每个控制器对应一个总线编号， 从 0 开始。
    
- 第二个数字 0： 表示连接在该 SPI 总线上的具体设备编号。 一个 SPI 总线上可以连接多个设备， 每个设备通过片选信号（Chip Select, CS） 进行区分， 设备编号从 0 开始。
    

## 3 spidev驱动分析

```c
// SPI设备的主设备号定义
#define SPIDEV_MAJOR        153        /* assigned */

// SPI次设备号的最大数量定义
#define N_SPI_MINORS        32         /* ... up to 256 */
```

这段代码定义了：

- SPI用户空间设备驱动的基本说明和使用注意事项
- SPI设备的主设备号（153）
- 支持的最大次设备数量（32个）
- 说明了SPI全双工传输的特性和设备节点的动态管理


使能`CONFIG_SPI_SPIDEV`，这样`spidev`驱动将会被编译，我们今天来看一下：

驱动流程图:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a502986a36359cd351fa4134360c4182.png)


  

### 3.1 init

目录：spi/spidev.c
```c
static int __init spidev_init(void)
{
    int status; // 状态返回值

    /* Claim our 256 reserved device numbers. Then register a class
     * that will key udev/mdev to add/remove /dev nodes. Last, register
     * the driver which manages those device numbers.
     */
    
    // 编译时检查：确保N_SPI_MINORS不超过256
    BUILD_BUG_ON(N_SPI_MINORS > 256);
    
    // 注册字符设备驱动，申请主设备号
    status = register_chrdev(SPIDEV_MAJOR, "spi", &spidev_fops);
    if (status < 0)
        return status; // 注册失败直接返回错误码

    // 创建设备类，用于udev自动创建设备节点
    spidev_class = class_create(THIS_MODULE, "spidev");
    if (IS_ERR(spidev_class)) {
        // 类创建失败，注销字符设备驱动
        unregister_chrdev(SPIDEV_MAJOR, spidev_spi_driver.driver.name);
        return PTR_ERR(spidev_class);
    }

    // 注册SPI驱动
    status = spi_register_driver(&spidev_spi_driver);
    if (status < 0) {
        // SPI驱动注册失败，清理已创建的资源
        class_destroy(spidev_class);
        unregister_chrdev(SPIDEV_MAJOR, spidev_spi_driver.driver.name);
    }
    
    return status; // 返回最终状态
}

// 定义模块初始化函数
module_init(spidev_init);
```
这个函数的主要功能是：
- 初始化SPI用户空间设备驱动模块
- 注册字符设备驱动和设备类
- 注册SPI驱动程序
- 包含完整的错误处理和资源清理机制

第一步操作集注册：
```c
static const struct file_operations spidev_fops = {
    .owner =        THIS_MODULE,
    /* REVISIT switch to aio primitives, so that userspace
     * gets more complete API coverage. It'll simplify things
     * too, except for the locking.
     */
    .write =        spidev_write,         // 写操作函数
    .read =         spidev_read,          // 读操作函数
    .unlocked_ioctl = spidev_ioctl,       // 无锁ioctl控制函数
    .compat_ioctl = spidev_compat_ioctl,  // 32位兼容ioctl函数
    .open =         spidev_open,          // 设备打开函数
    .release =      spidev_release,       // 设备释放函数
    .llseek =       no_llseek,            // 不支持文件定位
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e55b0f4c4e1a427db23c6656ed01108c.png)


然后会注册：
```c
static struct spi_driver spidev_spi_driver = {
    .driver = {
        .name =         "spidev",                           // 驱动名称
        .of_match_table = of_match_ptr(spidev_dt_ids),      // 设备树匹配表
        .acpi_match_table = ACPI_PTR(spidev_acpi_ids),      // ACPI匹配表
    },
    .probe =        spidev_probe,        // 设备探测函数
    .remove =       spidev_remove,       // 设备移除函数
    /* NOTE: suspend/resume methods are not necessary here.
     * We don't do anything except pass the requests to/from
     * the underlying controller. The refrigerator handles
     * most issues; the controller driver handles the rest.
     */
};
```

```c
#ifdef CONFIG_OF
static const struct of_device_id spidev_dt_ids[] = {
    { .compatible = "rohm,dh2228fv" },        // ROHM DH2228FV器件
    { .compatible = "lineartechnology,ltc2488" }, // Linear LTC2488 ADC
    { .compatible = "ge,achc" },              // GE ACHC器件
    { .compatible = "semtech,sx1301" },       // Semtech SX1301器件
    { .compatible = "rockchip,spidev" },      // Rockchip通用SPI设备
    {},
};
MODULE_DEVICE_TABLE(of, spidev_dt_ids);      // 导出设备树匹配表
#endif
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc773ed8ebb4663bc52b7fd8808fe363.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/daba78000e3553e2a64e6766e4b85921.png)


对应的device信息在设备树里面：
```c
&spi3 {
    status = "okay";          // 启用SPI3控制器
    
    spidev@0 {               // 片选CS0上的SPI设备
        compatible = "spidev";           // 通用SPI设备
        reg = <0>;                       // SPI片选信号CS0
        spi-max-frequency = <10000000>;  // 最大SPI频率10MHz
    };
};
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8b6066d6dcc420ae33791b0e7038d39e.png)

### 3.2 probe

匹配之后，spidev.c的`spidev_probe`会被调用，它会：

- 分配一个spidev_data结构体，用来记录对于的spi_device
    
- spidev_data会被记录在一个链表里
    
- 分配一个次设备号，以后可以根据这个次设备号在链表里找到spidev_data
    
- device_create：这会生产一个设备节点`/dev/spidevB.D`，B表示总线号，D表示它是这个SPI Master下第几个设备
    

```C
static int spidev_probe(struct spi_device *spi)
{
        struct spidev_data      *spidev;
        int                     status;
        unsigned long           minor;

        /*
         * spidev should never be referenced in DT without a specific
         * compatible string, it is a Linux implementation thing
         * rather than a description of the hardware.
         */
        WARN(spi->dev.of_node &&
             of_device_is_compatible(spi->dev.of_node, "spidev"),
             "%pOF: buggy DT: spidev listed directly in DT\n", spi->dev.of_node);

        spidev_probe_acpi(spi);

        /* Allocate driver data */
        spidev = kzalloc(sizeof(*spidev), GFP_KERNEL);
        if (!spidev)
                return -ENOMEM;

        /* Initialize the driver data */
        spidev->spi = spi;
        spin_lock_init(&spidev->spi_lock);
        mutex_init(&spidev->buf_lock);

        INIT_LIST_HEAD(&spidev->device_entry);

        /* If we can allocate a minor number, hook up this device.
         * Reusing minors is fine so long as udev or mdev is working.
         */
        mutex_lock(&device_list_lock);
        minor = find_first_zero_bit(minors, N_SPI_MINORS);
        if (minor < N_SPI_MINORS) {
                struct device *dev;
                //创建设备号
                spidev->devt = MKDEV(SPIDEV_MAJOR, minor);
                //创建设备
                dev = device_create(spidev_class, &spi->dev, spidev->devt,
                                    spidev, "spidev%d.%d",
                                    spi->master->bus_num, spi->chip_select);
                status = PTR_ERR_OR_ZERO(dev);
        } else {
                dev_dbg(&spi->dev, "no minor number available!\n");
                status = -ENODEV;
        }
        if (status == 0) {
                //设置位
                set_bit(minor, minors);
                //把spidev添加到device_list
                list_add(&spidev->device_entry, &device_list);
        }
        mutex_unlock(&device_list_lock);

        spidev->speed_hz = spi->max_speed_hz;

        if (status == 0)
                spi_set_drvdata(spi, spidev);
        else
                kfree(spidev);

        return status;
}
```

### 3.3 spidev_fops

```c
static const struct file_operations spidev_fops = {
    .owner =        THIS_MODULE,
    /* REVISIT switch to aio primitives, so that userspace
     * gets more complete API coverage. It'll simplify things
     * too, except for the locking.
     */
    .write =        spidev_write,         // 写操作函数
    .read =         spidev_read,          // 读操作函数
    .unlocked_ioctl = spidev_ioctl,       // 无锁ioctl控制函数
    .compat_ioctl = spidev_compat_ioctl,  // 32位兼容ioctl函数
    .open =         spidev_open,          // 设备打开函数
    .release =      spidev_release,       // 设备释放函数
    .llseek =       no_llseek,            // 不支持文件定位
};
```

#### 3.3.1 spidev_write
```c
// 图片1: SPI设备写操作函数
/* Write-only message with current device setup */
static ssize_t
spidev_write(struct file *filp, const char __user *buf,
            size_t count, loff_t *f_pos)
{
    struct spidev_data    *spidev;        // SPI设备数据指针
    ssize_t               status = 0;     // 操作状态
    unsigned long         missing;        // 拷贝失败的字节数

    /* chipselect only toggles at start or end of operation */
    // 检查数据长度是否超过缓冲区大小
    if (count > bufsiz)
        return -EMSGSIZE;

    spidev = filp->private_data;          // 获取设备私有数据

    mutex_lock(&spidev->buf_lock);        // 加锁保护缓冲区
    // 从用户空间拷贝数据到内核缓冲区
    missing = copy_from_user(spidev->tx_buffer, buf, count);
    if (missing == 0)
        status = spidev_sync_write(spidev, count);  // 执行同步写操作
    else
        status = -EFAULT;                 // 拷贝失败返回错误
    mutex_unlock(&spidev->buf_lock);      // 解锁

    return status;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e4172aab9182c28a0a9af82033b332c0.png)

```c
// 图片2: 同步写操作内联函数
static inline ssize_t
spidev_sync_write(struct spidev_data *spidev, size_t len)
{
    struct spi_transfer  t = {            // SPI传输结构体
        .tx_buf        = spidev->tx_buffer,   // 发送缓冲区
        .len           = len,                 // 传输长度
        .speed_hz      = spidev->speed_hz,    // 传输速度
    };
    struct spi_message   m;               // SPI消息结构体

    spi_message_init(&m);                 // 初始化SPI消息
    spi_message_add_tail(&t, &m);         // 添加传输到消息队列
    return spidev_sync(spidev, &m);       // 执行同步传输
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7dd9f132b14cb7f07a3ba0dcb418bbdd.png)

```c
// 图片3&4: 同步传输执行函数
static ssize_t
spidev_sync(struct spidev_data *spidev, struct spi_message *message)
{
    int status;                           // 操作状态
    struct spi_device *spi;               // SPI设备指针

    spin_lock_irq(&spidev->spi_lock);     // 加自旋锁并禁用中断
    spi = spidev->spi;                    // 获取SPI设备
    spin_unlock_irq(&spidev->spi_lock);   // 解锁并恢复中断

    if (spi == NULL)
        status = -ESHUTDOWN;              // 设备已关闭
    else
        status = spi_sync(spi, message);   // 执行SPI同步传输

    if (status == 0)
        status = message->actual_length;   // 成功时返回实际传输长度

    return status;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b2e83a59766cb1af8f7284833bb56299.png)

spi_sync 详见：[10_SPI数据的传输](10_SPI数据的传输.md)

#### 3.3.2 spidev_read

```c
// 图片5: SPI设备读操作函数
/* Read-only message with current device setup */
static ssize_t
spidev_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
{
    struct spidev_data    *spidev;        // SPI设备数据指针
    ssize_t               status = 0;     // 操作状态

    /* chipselect only toggles at start or end of operation */
    // 检查数据长度是否超过缓冲区大小
    if (count > bufsiz)
        return -EMSGSIZE;

    spidev = filp->private_data;          // 获取设备私有数据

    mutex_lock(&spidev->buf_lock);        // 加锁保护缓冲区
    status = spidev_sync_read(spidev, count);  // 执行同步读操作
    if (status > 0) {
        unsigned long    missing;         // 拷贝失败的字节数

        // 将接收缓冲区数据拷贝到用户空间
        missing = copy_to_user(buf, spidev->rx_buffer, status);
        if (missing == status)
            status = -EFAULT;             // 拷贝完全失败
        else
            status = status - missing;     // 返回成功拷贝的字节数
    }
    mutex_unlock(&spidev->buf_lock);      // 解锁

    return status;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c4ca1cb4a4af1cf6c3ebfe2f59b6fa67.png)


#### 3.3.3 spidev_open

```c
// 图片6: SPI设备打开函数
static int spidev_open(struct inode *inode, struct file *filp)
{
    struct spidev_data    *spidev;        // SPI设备数据指针
    int                   status = -ENXIO; // 初始状态为设备不存在

    mutex_lock(&device_list_lock);        // 加锁保护设备列表
    
    // 遍历设备列表查找匹配的设备
    list_for_each_entry(spidev, &device_list, device_entry) {
        if (spidev->devt == inode->i_rdev) {
            status = 0;                   // 找到设备
            break;
        }
    }
    
    if (status) {
        // 未找到设备，输出调试信息
        pr_debug("spidev: nothing for minor %d\n", iminor(inode));
        goto err_find_dev;
    }

    // 分配发送缓冲区
    if (!spidev->tx_buffer) {
        spidev->tx_buffer = kmalloc(bufsiz, GFP_KERNEL);
        if (!spidev->tx_buffer) {
            dev_dbg(&spidev->spi->dev, "open/ENOMEM\n");
            status = -ENOMEM;
            goto err_find_dev;
        }
    }

    // 分配接收缓冲区
    if (!spidev->rx_buffer) {
        spidev->rx_buffer = kmalloc(bufsiz, GFP_KERNEL);
        if (!spidev->rx_buffer) {
            dev_dbg(&spidev->spi->dev, "open/ENOMEM\n");
            status = -ENOMEM;
            goto err_alloc_rx_buf;
        }
    }

    spidev->users++;                      // 增加用户计数
    filp->private_data = spidev;          // 设置文件私有数据
    nonseekable_open(inode, filp);        // 标记文件不可定位

    mutex_unlock(&device_list_lock);      // 解锁设备列表
    return 0;

err_alloc_rx_buf:
    kfree(spidev->tx_buffer);             // 释放发送缓冲区
    spidev->tx_buffer = NULL;
err_find_dev:
    mutex_unlock(&device_list_lock);      // 解锁设备列表
    return status;
}
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cfa73e14009a5efd7757e22afa6877e6.png)


#### 3.3.4 spidev_ioctl

可以分为两类：

1. spi通信参数获取
    
2. spi通信参数设置
    
3. 数据的传输
    

  

当使用ioctl函数进行SPI通信时，常用的request参数主要有以下几种：

- `SPI_IOC_RD_MODE`：用于读取当前SPI通信的模式设置。这个请求参数将模式信息读取到一个整数变量中，以便检查当前SPI通信的极性和相位设置。
    
- `SPI_IOC_WR_MODE`：用于设置SPI通信的模式。您需要提供一个整数值，该值通常由两位二进制数字组成，表示SPI通信的极性和相位。
    
- `SPI_IOC_RD_BITS_PER_WORD`：用于读取每个数据字的位数。这个请求参数将位数信息读取到一个整数变量中。
    
- `SPI_IOC_WR_BITS_PER_WORD`：用于设置每个数据字的位数。您需要提供一个整数值，以指定要发送和接收的每个数据字的位数。
    
- `SPI_IOC_RD_MAX_SPEED_HZ`：用于读取SPI总线的最大速度。这个请求参数将速度信息读取到一个整数变量中，以便检查当前SPI总线的最大传输速度。
    
- `SPI_IOC_WR_MAX_SPEED_HZ`：用于设置SPI总线的最大速度。您需要提供一个整数值，以指定要使用的最大传输速度。
    
- `SPI_IOC_MESSAGE(N)`：用于执行SPI传输的读写操作。这个请求参数需要一个指向 `struct spi_ioc_transfer` 数组的指针，每个元素描述了一个SPI传输操作，可以执行多个操作。
    
- `SPI_IOC_RD_LSB_FIRST`：用于读取LSB（Least Significant Bit）优先设置。这个请求参数将LSB优先信息读取到一个整数变量中。
    
- `SPI_IOC_WR_LSB_FIRST`：用于设置LSB优先设置。您需要提供一个整数值，以指定是否要使用LSB优先模式。
    

```C
static long
spidev_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
        int                     retval = 0;
        struct spidev_data      *spidev;
        struct spi_device       *spi;
        u32                     tmp;
        unsigned                n_ioc;
        struct spi_ioc_transfer *ioc;

        /* Check type and command number */
        if (_IOC_TYPE(cmd) != SPI_IOC_MAGIC)
                return -ENOTTY;

        /* guard against device removal before, or while,
         * we issue this ioctl.
         */
        spidev = filp->private_data;
        spin_lock_irq(&spidev->spi_lock);
        spi = spi_dev_get(spidev->spi);
        spin_unlock_irq(&spidev->spi_lock);

        if (spi == NULL)
                return -ESHUTDOWN;

        /* use the buffer lock here for triple duty:
         *  - prevent I/O (from us) so calling spi_setup() is safe;
         *  - prevent concurrent SPI_IOC_WR_* from morphing
         *    data fields while SPI_IOC_RD_* reads them;
         *  - SPI_IOC_MESSAGE needs the buffer locked "normally".
         */
        mutex_lock(&spidev->buf_lock);

        switch (cmd) {
        /* read requests */
        case SPI_IOC_RD_MODE:
                retval = put_user(spi->mode & SPI_MODE_MASK,
                                        (__u8 __user *)arg);
                break;
        case SPI_IOC_RD_MODE32:
                retval = put_user(spi->mode & SPI_MODE_MASK,
                                        (__u32 __user *)arg);
                break;
        case SPI_IOC_RD_LSB_FIRST:
                retval = put_user((spi->mode & SPI_LSB_FIRST) ?  1 : 0,
                                        (__u8 __user *)arg);
                break;
        case SPI_IOC_RD_BITS_PER_WORD:
                retval = put_user(spi->bits_per_word, (__u8 __user *)arg);
                break;
        case SPI_IOC_RD_MAX_SPEED_HZ:
                retval = put_user(spidev->speed_hz, (__u32 __user *)arg);
                break;

        /* write requests */
        case SPI_IOC_WR_MODE:
        case SPI_IOC_WR_MODE32:
                if (cmd == SPI_IOC_WR_MODE)
                        retval = get_user(tmp, (u8 __user *)arg);
                else
                        retval = get_user(tmp, (u32 __user *)arg);
                if (retval == 0) {
                        u32     save = spi->mode;

                        if (tmp & ~SPI_MODE_MASK) {
                                retval = -EINVAL;
                                break;
                        }

                        tmp |= spi->mode & ~SPI_MODE_MASK;
                        spi->mode = (u16)tmp;
                        retval = spi_setup(spi);
                        if (retval < 0)
                                spi->mode = save;
                        else
                                dev_dbg(&spi->dev, "spi mode %x\n", tmp);
                }
                break;
        case SPI_IOC_WR_LSB_FIRST:
                retval = get_user(tmp, (__u8 __user *)arg);
                if (retval == 0) {
                        u32     save = spi->mode;

                        if (tmp)
                                spi->mode |= SPI_LSB_FIRST;
                        else
                                spi->mode &= ~SPI_LSB_FIRST;
                        retval = spi_setup(spi);
                        if (retval < 0)
                                spi->mode = save;
                        else
                                dev_dbg(&spi->dev, "%csb first\n",
                                                tmp ? 'l' : 'm');
                }
                break;
        case SPI_IOC_WR_BITS_PER_WORD:
                retval = get_user(tmp, (__u8 __user *)arg);
                if (retval == 0) {
                        u8      save = spi->bits_per_word;

                        spi->bits_per_word = tmp;
                        retval = spi_setup(spi);
                        if (retval < 0)
                                spi->bits_per_word = save;
                        else
                                dev_dbg(&spi->dev, "%d bits per word\n", tmp);
                }
                break;
        case SPI_IOC_WR_MAX_SPEED_HZ:
                retval = get_user(tmp, (__u32 __user *)arg);
                if (retval == 0) {
                        u32     save = spi->max_speed_hz;

                        spi->max_speed_hz = tmp;
                        retval = spi_setup(spi);
                        if (retval >= 0)
                                spidev->speed_hz = tmp;
                        else
                                dev_dbg(&spi->dev, "%d Hz (max)\n", tmp);
                        spi->max_speed_hz = save;
                }
                break;

        default:
                /* segmented and/or full-duplex I/O request */
                /* Check message and copy into scratch area */
                ioc = spidev_get_ioc_message(cmd,
                                (struct spi_ioc_transfer __user *)arg, &n_ioc);
                if (IS_ERR(ioc)) {
                        retval = PTR_ERR(ioc);
                        break;
                }
                if (!ioc)
                        break;  /* n_ioc is also 0 */

                /* translate to spi_message, execute */
                retval = spidev_message(spidev, ioc, n_ioc);
                kfree(ioc);
                break;
        }

        mutex_unlock(&spidev->buf_lock);
        spi_dev_put(spi);
        return retval;
}
```

1. ##### spi_ioc_transfer
    

2. spidev_get_ioc_message
    
3. spidev_message
    

  

## 4 spi_register_driver分析

```txt
// 图片1: SPI驱动注册相关宏定义和函数声明 (linux/spi/spi.h)
// 第273行: spi_register_driver宏定义
#define spi_register_driver(driver) \
    __spi_register_driver(THIS_MODULE, driver)
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6d369f1f3bf0720e253c2ab9e3ede69b.png)


spi/spi.c

```c
// 图片2: SPI驱动注册函数实现
/**
 * __spi_register_driver - register a SPI driver
 * @owner: owner module of the driver to register
 * @sdrv: the driver to register
 * Context: can sleep
 *
 * Return: zero on success, else a negative error code.
 */
int __spi_register_driver(struct module *owner, struct spi_driver *sdrv)
{
    sdrv->driver.owner = owner;               // 设置驱动的所有者模块
    sdrv->driver.bus = &spi_bus_type;         // 设置总线类型为SPI总线
    sdrv->driver.probe = spi_drv_probe;       // 设置设备探测函数
    sdrv->driver.remove = spi_drv_remove;     // 设置设备移除函数
    if (sdrv->shutdown)                       // 如果定义了shutdown函数
        sdrv->driver.shutdown = spi_drv_shutdown;  // 设置系统关闭函数
    return driver_register(&sdrv->driver);    // 注册到设备驱动框架
}

// 导出符号供GPL许可的模块使用
EXPORT_SYMBOL_GPL(__spi_register_driver);
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/15a50d68d67578ce60dc55e7e479bec4.png)


## 5 spi_dev缺点

使用read、write函数时，只能读、写，这是半双工方式。

使用ioctl可以达到全双工的读写。

但是spidev有2个缺点：

- 不支持中断
    
- 只支持同步操作，不支持异步操作：就是read/write/ioctl这些函数只能执行完毕才可返回