  

## 设备驱动的重要知识点

- 结构体：device_driver
    
- 结构体：driver_attribute
    
- 驱动的注册即可driver_register
    
- 驱动属性文件driver_create_file
    

## 驱动注册过程分析driver_register

`drivers/base/driver.c`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7cd0b91336ef1bdd0a82553987c6228c.png)

```C++
int driver_register(struct device_driver *drv)
{
        int ret;
        struct device_driver *other;

        // 检查总线是否已初始化
        if (!drv->bus->p) {
                pr_err("Driver '%s' was unable to register with bus_type '%s' because the bus was not initialized.\n",
                           drv->name, drv->bus->name);
                return -EINVAL;
        }

        // 检查驱动程序的方法是否需要更新
        if ((drv->bus->probe && drv->probe) ||
            (drv->bus->remove && drv->remove) ||
            (drv->bus->shutdown && drv->shutdown))
                printk(KERN_WARNING "Driver '%s' needs updating - please use "
                        "bus_type methods\n", drv->name);

        // 检查驱动程序是否已被注册
        other = driver_find(drv->name, drv->bus);
        if (other) {
                printk(KERN_ERR "Error: Driver '%s' is already registered, "
                        "aborting...\n", drv->name);
                return -EBUSY;
        }

        ret = bus_add_driver(drv); // 将驱动程序添加到总线
        if (ret)
                return ret;
        ret = driver_add_groups(drv, drv->groups);// 添加驱动程序的组属性  
        if (ret) {
                bus_remove_driver(drv);// 移除已添加的驱动程序  
                return ret;
        }
        kobject_uevent(&drv->p->kobj, KOBJ_ADD);// 发送内核对象事件， 通知驱动程序添加成功  

        return ret;// 返回注册结果  
}
EXPORT_SYMBOL_GPL(driver_register);
```

driver_register 函数用于注册设备驱动程序并将其添加到总线中。 以下是该函数的功能解释：

- 第 5~11 行代码 检查总线是否已初始化：
    

首先， 通过 drv->bus 访问设备驱动程序结构体中的总线信息。如果总线的 p 成员为 NULL，表示总线未初始化。 如果总线未初始化， 则打印错误消息， 并返回 -EINVAL 错误码表示无效的参数。

- 第 13 行~19 行代码 检查驱动程序的方法是否需要更新：
    

通过检查驱动程序结构体中的 bus->probe 和 drv->probe、 bus->remove 和 drv->remove、bus->shutdown 和 drv->shutdown 成员是否同时存在来判断。 如果存在需要更新的方法组合，说明驱动程序需要更新。 在这种情况下， 打印警告消息， 建议使用 bus_type 方法进行更新。

- 第 21 行~26 行 检查驱动程序是否已被注册：
    

调用 driver_find 函数来查找是否已经注册了同名的驱动程序。 如果找到同名驱动程序，表示驱动程序已经注册过。 在这种情况下， 打印错误消息， 并返回 -EBUSY 错误码表示设备忙。

- 第 28 行代码 添加驱动程序到总线：
    

调用 bus_add_driver 函数将驱动程序添加到总线。 如果添加失败， 则返回相应的错误码。

- 第 31 行代码 添加驱动程序的组属性：
    

调用 driver_add_groups 函数将驱动程序的组属性添加到驱动程序中。 如果添加失败， 则调用 bus_remove_driver 函数移除已添加的驱动程序， 并返回相应的错误码。

- 第 36 行代码 发送内核对象事件：
    

调用 kobject_uevent 函数向驱动程序的内核对象发送事件， 通知驱动程序已成功添加到系统中。

  

综上所述， driver_register 函数的功能是注册设备驱动程序并将其添加到总线中， 同时进行各种检查和错误处理操作。

## bus_add_driver

在上面代码中， 调用 bus_add_driver 函数将驱动程序添加到总线。 我们来详细分析下bus_add_driver 函数。

`drivers/base/bus.c`

```C++
int bus_add_driver(struct device_driver *drv)
{
        struct bus_type *bus;
        struct driver_private *priv;
        int error = 0;
        
        // 获取总线对象
        bus = bus_get(drv->bus);
        if (!bus)
                return -EINVAL;// 返回无效参数错误码

        pr_debug("bus: '%s': add driver %s\n", bus->name, drv->name);
        // 分配并初始化驱动程序私有数据
        priv = kzalloc(sizeof(*priv), GFP_KERNEL);
        if (!priv) {
                error = -ENOMEM;
                goto out_put_bus;
        }
        klist_init(&priv->klist_devices, NULL, NULL);
        priv->driver = drv;
        drv->p = priv;
        priv->kobj.kset = bus->p->drivers_kset;
        
        // 初始化并添加驱动程序的内核对象
        error = kobject_init_and_add(&priv->kobj, &driver_ktype, NULL,
                                     "%s", drv->name);
        if (error)
                goto out_unregister;
        
        // 将驱动程序添加到总线的驱动程序列表
        klist_add_tail(&priv->knode_bus, &bus->p->klist_drivers);
        if (drv->bus->p->drivers_autoprobe) {//默认就是1
                error = driver_attach(drv);
                if (error)
                        goto out_unregister;
        }
        // 将驱动程序添加到模块
        module_add_driver(drv->owner, drv);

        // 创建驱动程序的 uevent 属性文件
        error = driver_create_file(drv, &driver_attr_uevent);
        if (error) {
                printk(KERN_ERR "%s: uevent attr (%s) failed\n",
                        __func__, drv->name);
        }
        // 添加驱动程序的组属性
        error = driver_add_groups(drv, bus->drv_groups);
        if (error) {
                /* How the hell do we get out of this pickle? Give up */
                printk(KERN_ERR "%s: driver_create_groups(%s) failed\n",
                        __func__, drv->name);
        }
        
        // 如果驱动程序不禁止绑定属性文件， 则添加绑定属性文件
        if (!drv->suppress_bind_attrs) {
                error = add_bind_files(drv);
                if (error) {
                        /* Ditto */
                        printk(KERN_ERR "%s: add_bind_files(%s) failed\n",
                                __func__, drv->name);
                }
        }

        return 0;

out_unregister:
        kobject_put(&priv->kobj);
        /* drv->p is freed in driver_release()  */
        drv->p = NULL;
out_put_bus:
        bus_put(bus);
        return error;
}
```

### driver_attach

在上述函数中， 使用 driver_attach 函数来探测设备， 我们进一步分析下 driver_attach 函数。

目录：base/dd.c

```C
/**
 * driver_attach - try to bind driver to devices.
 * @drv: driver.
 *
 * Walk the list of devices that the bus has on it and try to
 * match the driver with each one.  If driver_probe_device()
 * returns 0 and the @dev->driver is set, we've found a
 * compatible pair.
 */
int driver_attach(struct device_driver *drv)
{
        return bus_for_each_dev(drv->bus, NULL, drv, __driver_attach);
}
EXPORT_SYMBOL_GPL(driver_attach);
```

bus_for_each_dev 函数实现如下所示：

目录：base/bus.c

```C
int bus_for_each_dev(struct bus_type *bus, struct device *start,
                     void *data, int (*fn)(struct device *, void *))
{
        struct klist_iter i;
        struct device *dev;
        int error = 0;

         // 检查总线对象是否存在
        if (!bus || !bus->p)
                return -EINVAL; // 返回无效参数错误码

        // 初始化设备列表迭代器
        klist_iter_init_node(&bus->p->klist_devices, &i,
                             (start ? &start->p->knode_bus : NULL));
        
        // 遍历设备列表并执行指定的函数
        while (!error && (dev = next_device(&i)))
                error = fn(dev, data);
                
        // 退出设备列表迭代器
        klist_iter_exit(&i);
        return error;// 返回执行过程中的错误码（如果有）
}
EXPORT_SYMBOL_GPL(bus_for_each_dev);
```

这个函数的作用是遍历指定总线上的所有设备， 并对每个设备执行指定的函数 fn。 以下是函数的参数和功能解释：

- bus： 指定要遍历的总线对象。
    
- start： 指定开始遍历的设备对象。 如果为 NULL， 则从总线的第一个设备开始遍历。data： 传递给函数 fn 的额外数据。
    
- fn： 指定要执行的函数， 该函数接受一个设备对象和额外数据作为参数， 并返回一个整数错误码。
    

  

- 第 9 行代码中， 首先， 检查传入的总线对象是否存在以及与该总线相关的私有数据是否存
    

在。 如果总线对象或其私有数据不存在， 返回 -EINVAL 表示无效的参数错误码。

- 第 13 行~14 行代码中， 接下来， 初始化设备列表迭代器， 以便遍历总线上的设备。 使用
    

klist_iter_init_node 函数初始化一个设备列表迭代器。 传递总线对象的设备列表klist_devices、 迭代器对象 i， 以及可选的起始设备的节点指针。 然后， 在一个循环中遍历设备列表并执行指定的函数。

- 第 17 行代码中， 使用 next_device 函数从迭代器中获取下一个设备。 如果存在下一个设备， 则调用传入的函数指针 fn， 并将当前设备和额外的数据参数传递给它。 如果执行函数时出现错误， 将错误码赋值给 error。
    
- 第 21 行代码中， 最后， 退出设备列表迭代器， 释放相关资源。 使用 klist_iter_exit 函数退出设备列表的迭代器。
    

总的来说， bus_for_each_dev() 函数主要是提供了一个遍历指定总线上的设备对象列表，并对每个设备对象进行特定操作的快捷方式， 可以用于驱动程序中需要管理和操作大量设备实例的场景。

fn 要执行的函数为__driver_attach 函数， 函数实现如下所示：

```C
static int __driver_attach(struct device *dev, void *data)
{
        struct device_driver *drv = data;// 传入的数据参数作为设备驱动对象
        int ret;

        /*
         * Lock device and try to bind to it. We drop the error
         * here and always return 0, because we need to keep trying
         * to bind to devices and some drivers will return an error
         * simply if it didn't support the device.
         *
         * driver_probe_device() will spit a warning if there
         * is an error.
         */

        ret = driver_match_device(drv, dev);// 尝试将驱动程序绑定到设备上
        if (ret == 0) {
                /* no match */
                return 0;// 如果没有匹配， 则返回 0
        } else if (ret == -EPROBE_DEFER) {
                dev_dbg(dev, "Device match requests probe deferral\n");
                driver_deferred_probe_add(dev);// 请求推迟探测设备
        } else if (ret < 0) {
                dev_dbg(dev, "Bus failed to match device: %d", ret);
                return ret;// 总线无法匹配设备， 返回错误码
        } /* ret > 0 means positive match */

        if (driver_allows_async_probing(drv)) {
                /*
                 * Instead of probing the device synchronously we will
                 * probe it asynchronously to allow for more parallelism.
                 *
                 * We only take the device lock here in order to guarantee
                 * that the dev->driver and async_driver fields are protected
                 */
                dev_dbg(dev, "probing driver %s asynchronously\n", drv->name);
                device_lock(dev);// 锁定设备以保护 dev->driver 和 async_driver 字段
                if (!dev->driver) {
                        get_device(dev);
                        dev->p->async_driver = drv;// 设置设备的异步驱动程序
                        async_schedule(__driver_attach_async_helper, dev);// 异步调度驱动程序的附加处理函数
                }
                device_unlock(dev);// 解锁设备
                return 0;
        }

        device_driver_attach(drv, dev);// 同步探测设备并绑定驱动程序

        return 0;// 返回 0 表示成功执行驱动程序附加操作
}
```

上述函数中使用 driver_match_device 函数尝试将驱动程序绑定到设备上， 我们再来看看driver_match_device 函数。

```C
static inline int driver_match_device(struct device_driver *drv,
                                      struct device *dev)
{
        return drv->bus->match ? drv->bus->match(dev, drv) : 1;
}
```

这是一个内联函数 driver_match_device 的代码片段。 该函数用于检查设备是否与驱动程序匹配。 以下是对代码的解释：

- drv： 指向设备驱动程序对象的指针。
    
- dev： 指向设备对象的指针。
    

函数的执行过程如下：

- drv->bus->match(dev, drv)表示调用总线对象的 match 函数， 并将设备对象和驱动程序对象作为参数传递给该函数。 dev 是用于匹配的设备对象。 drv 是用于匹配的驱动程序对象。
    
- 如果总线对象的 match 函数返回 0， 则表示设备与驱动程序不匹配， 函数将返回 0。 返回值为 0 表示不匹配
    
- 如果总线对象的 match 函数返回非零值（大于 0） ， 则表示设备与驱动程序匹配， 函数将返回 1。 返回值为 1 表示匹配。
    
- 如果总线对象的 match 函数不存在（为 NULL） ， 则默认认为设备与驱动程序匹配， 函数将返回 1。
    

这段代码使用了条件运算符 `? :` 来判断总线对象的 match 函数是否存在， 以便选择执行

相应的逻辑。 如果总线对象没有提供 match 函数， 那么默认认为设备与驱动程序匹配， 返回值为1

如果设备和驱动匹配上， 会继续执行 device_driver_attach 函数， 如下所示：

```C
/**
 * device_driver_attach - attach a specific driver to a specific device
 * @drv: Driver to attach
 * @dev: Device to attach it to
 *
 * Manually attach driver to a device. Will acquire both @dev lock and
 * @dev->parent lock if needed.
 */
int device_driver_attach(struct device_driver *drv, struct device *dev)
{
        int ret = 0;

        __device_driver_lock(dev, dev->parent);

        /*
         * If device has been removed or someone has already successfully
         * bound a driver before us just skip the driver probe call.
         */
        if (!dev->p->dead && !dev->driver)
                ret = driver_probe_device(drv, dev);

        __device_driver_unlock(dev, dev->parent);

        return ret;
}
```

driver_probe_device 函数， 如下所示：

```C
/**
 * driver_probe_device - attempt to bind device & driver together
 * @drv: driver to bind a device to
 * @dev: device to try to bind to the driver
 *
 * This function returns -ENODEV if the device is not registered,
 * 1 if the device is bound successfully and 0 otherwise.
 *
 * This function must be called with @dev lock held.  When called for a
 * USB interface, @dev->parent lock must be held as well.
 *
 * If the device has a parent, runtime-resume the parent before driver probing.
 */
int driver_probe_device(struct device_driver *drv, struct device *dev)
{
        int ret = 0;
        
        // 检查设备是否已注册， 如果未注册则返回错误码 -ENODEV
        if (!device_is_registered(dev))
                return -ENODEV;

        // 打印调试信息， 表示设备与驱动程序匹配
        pr_debug("bus: '%s': %s: matched device %s with driver %s\n",
                 drv->bus->name, __func__, dev_name(dev), drv->name);

        // 获取设备供应商的运行时引用计数
        pm_runtime_get_suppliers(dev);
        
        // 如果设备有父设备， 获取父设备的同步运行时引用计数
        if (dev->parent)
                pm_runtime_get_sync(dev->parent);

        // 等待设备的运行时状态达到稳定
        pm_runtime_barrier(dev);
        // 根据初始化调试标志选择调用真实的探测函数
        if (initcall_debug)
                ret = really_probe_debug(dev, drv);
        else
                ret = really_probe(dev, drv);
        // 请求设备进入空闲状态（省电模式）
        pm_request_idle(dev);

        // 如果设备有父设备， 释放父设备的运行时引用计数
        if (dev->parent)
                pm_runtime_put(dev->parent);

        // 释放设备供应商的运行时引用计数
        pm_runtime_put_suppliers(dev);
        // 返回探测函数的执行结果
        return ret;
}
```

经过前面代码的分析， 总结设备和驱动匹配流程函数， 如下图所示

![whiteboard_exported_image (1).png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc91daabb376c402856058ba82fbd42b.png)


probe 函数的执行， 我们来分析 really_probe 函数。

```C
static int really_probe(struct device *dev, struct device_driver *drv)
{
        int ret = -EPROBE_DEFER;// 初始化返回值为延迟探测
        // 获取当前延迟探测计数
        int local_trigger_count = atomic_read(&deferred_trigger_count);
        // 判断是否启用了驱动移除测试
        bool test_remove = IS_ENABLED(CONFIG_DEBUG_TEST_DRIVER_REMOVE) &&
                           !drv->suppress_bind_attrs;

        if (defer_all_probes) {
                /*
                 * defer_all_probes 的值只能通过 device_defer_all_probes_enable() 设置，
                 * 而该函数会紧接着调用 wait_for_device_probe()， 以避免任何竞争情况。
                 */
                dev_dbg(dev, "Driver %s force probe deferral\n", drv->name);
                driver_deferred_probe_add(dev);
                return ret;
        }

        ret = device_links_check_suppliers(dev);// 检查设备的供应者链路
        if (ret == -EPROBE_DEFER)
                driver_deferred_probe_add_trigger(dev, local_trigger_count);// 将设备添加到延迟探测触发列表
        if (ret)
                return ret;

        atomic_inc(&probe_count);// 增加探测计数
        pr_debug("bus: '%s': %s: probing driver %s with device %s\n",
                 drv->bus->name, __func__, drv->name, dev_name(dev));
        if (!list_empty(&dev->devres_head)) {
                dev_crit(dev, "Resources present before probing\n");
                ret = -EBUSY;
                goto done;
        }

re_probe:
        dev->driver = drv;

        /* 如果使用了 pinctrl， 绑定引脚 */
        ret = pinctrl_bind_pins(dev);
        if (ret)
                goto pinctrl_bind_failed;

        ret = dma_configure(dev);// 配置 DMA
        if (ret)
                goto probe_failed;

        if (driver_sysfs_add(dev)) {// 添加驱动的 sysfs
                printk(KERN_ERR "%s: driver_sysfs_add(%s) failed\n",
                        __func__, dev_name(dev));
                goto probe_failed;
        }

        if (dev->pm_domain && dev->pm_domain->activate) {
                // 如果设备有电源管理域并且存在激活函数，激活电源管理域
                ret = dev->pm_domain->activate(dev);
                if (ret)
                        goto probe_failed;
        }
        // 如果总线有探测函数， 调用总线的探测函数
        if (dev->bus->probe) {
                ret = dev->bus->probe(dev);
                if (ret)
                        goto probe_failed;
        // 否则调用驱动的探测函数
        } else if (drv->probe) {
                ret = drv->probe(dev);
                if (ret)
                        goto probe_failed;
        }
        // 如果启用了驱动移除测试
        if (test_remove) {
                test_remove = false;
                // 如果总线有移除函数， 调用总线的移除函数
                if (dev->bus->remove)
                        dev->bus->remove(dev);
                // 否则调用驱动的移除函数
                else if (drv->remove)
                        drv->remove(dev);
                // 释放设备的资源
                devres_release_all(dev);
                // 移除驱动的 sysfs
                driver_sysfs_remove(dev);
                dev->driver = NULL;
                dev_set_drvdata(dev, NULL);
                // 如果设备有电源管理域并且存在解除函数， 解除电源管理域
                if (dev->pm_domain && dev->pm_domain->dismiss)
                        dev->pm_domain->dismiss(dev);
                // 重新初始化电源管理运行时
                pm_runtime_reinit(dev);
                // 重新进行探测
                goto re_probe;
        }
        // 完成 pinctrl 的初始化
        pinctrl_init_done(dev);
        // 如果设备有电源管理域并且存在同步函数， 同步电源管理域
        if (dev->pm_domain && dev->pm_domain->sync)
                dev->pm_domain->sync(dev);
        // 驱动绑定成功
        driver_bound(dev);
        ret = 1;
        pr_debug("bus: '%s': %s: bound device %s to driver %s\n",
                 drv->bus->name, __func__, dev_name(dev), drv->name);
        goto done;
probe_failed:
        if (dev->bus)
                blocking_notifier_call_chain(&dev->bus->p->bus_notifier,
                                             BUS_NOTIFY_DRIVER_NOT_BOUND, dev);
pinctrl_bind_failed:
        device_links_no_driver(dev);
        devres_release_all(dev);
        dma_deconfigure(dev);
        driver_sysfs_remove(dev);
        dev->driver = NULL;
        dev_set_drvdata(dev, NULL);
        if (dev->pm_domain && dev->pm_domain->dismiss)
                dev->pm_domain->dismiss(dev);
        pm_runtime_reinit(dev);
        dev_pm_set_driver_flags(dev, 0);

        switch (ret) {
        case -EPROBE_DEFER:
                /* Driver requested deferred probing */
                dev_dbg(dev, "Driver %s requests probe deferral\n", drv->name);
                driver_deferred_probe_add_trigger(dev, local_trigger_count);
                break;
        case -ENODEV:
        case -ENXIO:
                pr_debug("%s: probe of %s rejects match %d\n",
                         drv->name, dev_name(dev), ret);
                break;
        default:
                /* driver matched but the probe failed */
                printk(KERN_WARNING
                       "%s: probe of %s failed with error %d\n",
                       drv->name, dev_name(dev), ret);
        }
        /*
         * Ignore errors returned by ->probe so that the next driver can try
         * its luck.
         */
        ret = 0;
done:
        atomic_dec(&probe_count);
        wake_up_all(&probe_waitqueue);
        return ret;
}
```