
借助i2c_transfer、i2c_smbus_xfer、一个实例化的i2c_adapter类型的变量，我们可以与该实例化的i2c_adapter类型变量下所有i2c client的通信。若基于这种方式与i2c client进行通信，那就可以不对具体的i2c client开发具体的驱动程序，即可与该i2c client进行通信了。

基于这种思想，i2c模块为每一个注册到i2c 总线的adapter，均创建了对应的字符设备文件，并为该设备文件提供了读写方法（供系统调用open、read、write、close、ioctl调用）。这样应用程序通过对具体的字符设备文件进行读写操作，即可完成与i2c client的通信（而无需为该i2c client编写内核驱动程序，仅需在应用层为该i2c client开发操作程序即可）。

  

通用 I2C 驱动文件为 **drivers/i2c/i2c-dev.c**，它为 I2C 外设提供了统一的驱动框架,分为 ：

- I2C 客户端 (I2C client)
    
- I2C 驱动 (I2C driver)
    

它为上层应用程序提供了通用的设备节点 **/dev/i2c-X(X 代表 I2C 总线号)**。应用程序可以直接通过打开这个设备节点 **/dev/i2c-X**，并使 用标准的 I/O 操作如 open()、ioctl()、read()、write() 等来与 I2C 从设备进行通信。

该驱动程序一般情况下都是默认使能的，具体路径如下所示，如果进入系统之后发现在/dev 目录下没有 i2c-X 相应的节点，就需要修改内核配置文件了。

  

## i2c字符设备的创建

针对你i2c 字符设备的创建，主要包括两个部分：

  

通过i2c_dev_init接口，注册一个主设备号为I2C_MAJOR，次设备号为0的字符设备，但此时尚未为该字符设备创建设备文件。

在i2c adapter的实例化对象注册到i2c总线时，借助i2c总线的BUS_NOTIFY_ADD_DEVICE通知事件，通过i2cdev_attach_adapter接口，完成具体i2c adapter对应设备文件节点的创建（通过借助kobject ADD的uevent，以及应用层的udevd/mdev，实现设备节点的创建）。

### i2c_dev_init

  

`drivers/i2c/i2c-dev.c`

```C
static int __init i2c_dev_init(void)
{
        int res;

        printk(KERN_INFO "i2c /dev entries driver\n");

        //注册字符设备驱动,主设备号为 I2C_MAJOR,次设备号范围为 0 到 I2C_MINORS-1,设备名为"i2c"
        res = register_chrdev_region(MKDEV(I2C_MAJOR, 0), I2C_MINORS, "i2c");
        if (res)
                goto out;
        // 创建一个 class 对象,名称为 "i2c-dev",用于在用
        i2c_dev_class = class_create(THIS_MODULE, "i2c-dev");
        if (IS_ERR(i2c_dev_class)) {
                res = PTR_ERR(i2c_dev_class);
                goto out_unreg_chrdev;
        }
        // 将 i2c_groups 数组设置为该 class 的 dev_grou
        i2c_dev_class->dev_groups = i2c_groups;

        // 注册一个总线通知函数 i2cdev_notifier,用于追踪 i2c 总线上新添加或删除的适配器
        res = bus_register_notifier(&i2c_bus_type, &i2cdev_notifier);
        if (res)
                goto out_unreg_class;

        // 立即绑定已经存在的 i2c 适配器到 i2c设备
        i2c_for_each_dev(NULL, i2cdev_attach_adapter);

        return 0;

out_unreg_class:
        class_destroy(i2c_dev_class);
out_unreg_chrdev:
        unregister_chrdev_region(MKDEV(I2C_MAJOR, 0), I2C_MINORS);
out:
        printk(KERN_ERR "%s: Driver Initialisation failed\n", __FILE__);
        return res;
}
```

i2cdev_attach_adapter 函数的具体内容：

  

## i2c-dev字符设备文件操作接口

i2c_dev 结构体中的 cdev 字段指定的文件操作集结构体为 i2cdev_fops，具体内容如下所示：

```C
static const struct file_operations i2cdev_fops = {
        .owner          = THIS_MODULE,
        .llseek         = no_llseek,
        .read           = i2cdev_read,
        .write          = i2cdev_write,
        .unlocked_ioctl = i2cdev_ioctl,
        .compat_ioctl   = compat_i2cdev_ioctl,
        .open           = i2cdev_open,
        .release        = i2cdev_release,
};
```

分别实现了常用的 open、read、write 和 ioctl，接下来对上述函数的实现进行讲解，首先 来看 i2cdev_open 函数，函数的具体内容如下所示：

### i2cdev_read

i2cdev 的read接口的内核实现间接调用了 i2c_master_recv() 接口， 在打开 i2cdev 后， 使用ioctl设定要访问的i2c设备的地址， 然后调用read()即可完成读操作

```C
static ssize_t i2cdev_read(struct file *file, char __user *buf, size_t count,
                loff_t *offset)
{
        char *tmp;
        int ret;

        struct i2c_client *client = file->private_data;

        if (count > 8192)
                count = 8192;

        tmp = kzalloc(count, GFP_KERNEL);
        if (tmp == NULL)
                return -ENOMEM;

        pr_debug("i2c-dev: i2c-%d reading %zu bytes.\n",
                iminor(file_inode(file)), count);

        ret = i2c_master_recv(client, tmp, count);
        if (ret >= 0)
                if (copy_to_user(buf, tmp, ret))
                        ret = -EFAULT;
        kfree(tmp);
        return ret;
}
```

  

### i2cdev_write

  

i2cdev 的write接口的内核实现间接调用了 i2c_master_send() 接口， 在打开i2cdev后， 使用ioctl设定要访问的i2c设备的地址， 然后调用write()即可完成读操作

```C
static ssize_t i2cdev_write(struct file *file, const char __user *buf,
                size_t count, loff_t *offset)
{
        int ret;
        char *tmp;
        struct i2c_client *client = file->private_data;

        if (count > 8192)
                count = 8192;

        tmp = memdup_user(buf, count);
        if (IS_ERR(tmp))
                return PTR_ERR(tmp);

        pr_debug("i2c-dev: i2c-%d writing %zu bytes.\n",
                iminor(file_inode(file)), count);

        ret = i2c_master_send(client, tmp, count);
        kfree(tmp);
        return ret;
}
```

  

### i2cdev_ioctl

```C
static long i2cdev_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
        struct i2c_client *client = file->private_data;
        unsigned long funcs;

        dev_dbg(&client->adapter->dev, "ioctl, cmd=0x%02x, arg=0x%02lx\n",
                cmd, arg);

        switch (cmd) {
        case I2C_SLAVE:
        case I2C_SLAVE_FORCE:
                if ((arg > 0x3ff) ||
                    (((client->flags & I2C_M_TEN) == 0) && arg > 0x7f))
                        return -EINVAL;
                if (cmd == I2C_SLAVE && i2cdev_check_addr(client->adapter, arg))
                        return -EBUSY;
                /* REVISIT: address could become busy later */
                client->addr = arg;
                return 0;
        case I2C_TENBIT:
                if (arg)
                        client->flags |= I2C_M_TEN;
                else
                        client->flags &= ~I2C_M_TEN;
                return 0;
        case I2C_PEC:
                /*
                 * Setting the PEC flag here won't affect kernel drivers,
                 * which will be using the i2c_client node registered with
                 * the driver model core.  Likewise, when that client has
                 * the PEC flag already set, the i2c-dev driver won't see
                 * (or use) this setting.
                 */
                if (arg)
                        client->flags |= I2C_CLIENT_PEC;
                else
                        client->flags &= ~I2C_CLIENT_PEC;
                return 0;
        case I2C_FUNCS:
                funcs = i2c_get_functionality(client->adapter);
                return put_user(funcs, (unsigned long __user *)arg);

        case I2C_RDWR: {
                struct i2c_rdwr_ioctl_data rdwr_arg;
                struct i2c_msg *rdwr_pa;

                if (copy_from_user(&rdwr_arg,
                                   (struct i2c_rdwr_ioctl_data __user *)arg,
                                   sizeof(rdwr_arg)))
                        return -EFAULT;

                if (!rdwr_arg.msgs || rdwr_arg.nmsgs == 0)
                        return -EINVAL;

                /*
                 * Put an arbitrary limit on the number of messages that can
                 * be sent at once
                 */
                if (rdwr_arg.nmsgs > I2C_RDWR_IOCTL_MAX_MSGS)
                        return -EINVAL;

                rdwr_pa = memdup_user(rdwr_arg.msgs,
                                      rdwr_arg.nmsgs * sizeof(struct i2c_msg));
                if (IS_ERR(rdwr_pa))
                        return PTR_ERR(rdwr_pa);

                return i2cdev_ioctl_rdwr(client, rdwr_arg.nmsgs, rdwr_pa);
        }

        case I2C_SMBUS: {
                struct i2c_smbus_ioctl_data data_arg;
                if (copy_from_user(&data_arg,
                                   (struct i2c_smbus_ioctl_data __user *) arg,
                                   sizeof(struct i2c_smbus_ioctl_data)))
                        return -EFAULT;
                return i2cdev_ioctl_smbus(client, data_arg.read_write,
                                          data_arg.command,
                                          data_arg.size,
                                          data_arg.data);
        }
        case I2C_RETRIES:
                if (arg > INT_MAX)
                        return -EINVAL;

                client->adapter->retries = arg;
                break;
        case I2C_TIMEOUT:
                if (arg > INT_MAX)
                        return -EINVAL;

                /* For historical reasons, user-space sets the timeout
                 * value in units of 10 ms.
                 */
                client->adapter->timeout = msecs_to_jiffies(arg * 10);
                break;
        default:
                /* NOTE:  returning a fault code here could cause trouble
                 * in buggy userspace code.  Some old kernel bugs returned
                 * zero in this case, and userspace code might accidentally
                 * have depended on that bug.
                 */
                return -ENOTTY;
        }
        return 0;
}
```