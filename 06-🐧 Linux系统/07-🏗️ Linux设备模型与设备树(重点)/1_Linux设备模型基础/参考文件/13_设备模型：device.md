## device 重点知识点

- 结构体：device
    
- 结构体：device_attribute
    
- 注册设备接口：device_register
    
- 创建设备属性文件：device_create_file
    

## device_register 函数分析

device_register 函数原型定义在 drivers/base/core.c 文件中， 如下所示：

```C++
int device_register(struct device *dev)
{
        device_initialize(dev);
        return device_add(dev);
}
EXPORT_SYMBOL_GPL(device_register);
```

上述代码展示了一个名为 **device_register** 的函数实现， 用于注册设备到内核中。 函数接受一个指向 struct device 类型的设备对象指针作为参数。 首先， 代码调用 **device_initialize** 函数对设备对象进行初始化。 接下来， 代码调用 device_add 函数将设备添加到内核中。 **device_add**函数会将设备添加到设备总线的设备列表中， 并执行与设备添加相关的操作， 例如分配设备号、创建设备节点等。 最后， 函数返回设备添加的结果， 通常是一个整数值表示成功或失败的状态码。

## device_add

目录： drivers/base/core.c

在 drivers/base/core.c 文件中的 device_ add 函数中调用了 bus_ probe_ device 函数。

```C
int device_add(struct device *dev)
{
        struct device *parent;
        struct kobject *kobj;
        struct class_interface *class_intf;
        int error = -EINVAL;
        struct kobject *glue_dir = NULL;

        dev = get_device(dev);
        if (!dev)
                goto done;

        if (!dev->p) {
                error = device_private_init(dev);
                if (error)
                        goto done;
        }
        /*
         * for statically allocated devices, which should all be converted
         * some day, we need to initialize the name. We prevent reading back
         * the name, and force the use of dev_name()
         */
        if (dev->init_name) {
                dev_set_name(dev, "%s", dev->init_name);
                dev->init_name = NULL;
        }

        /* subsystems can specify simple device enumeration */
        if (!dev_name(dev) && dev->bus && dev->bus->dev_name)
                dev_set_name(dev, "%s%u", dev->bus->dev_name, dev->id);

        if (!dev_name(dev)) {
                error = -EINVAL;
                goto name_error;
        }

        pr_debug("device: '%s': %s\n", dev_name(dev), __func__);

        parent = get_device(dev->parent);
        kobj = get_device_parent(dev, parent);
        if (IS_ERR(kobj)) {
                error = PTR_ERR(kobj);
                goto parent_error;
        }
        if (kobj)
                dev->kobj.parent = kobj;

        /* use parent numa_node */
        if (parent && (dev_to_node(dev) == NUMA_NO_NODE))
                set_dev_node(dev, dev_to_node(parent));

        /* first, register with generic layer. */
        /* we require the name to be set before, and pass NULL */
        error = kobject_add(&dev->kobj, dev->kobj.parent, NULL);
        if (error) {
                glue_dir = get_glue_dir(dev);
                goto Error;
        }

        /* notify platform of device entry */
        if (platform_notify)
                platform_notify(dev);

        error = device_create_file(dev, &dev_attr_uevent);
        if (error)
                goto attrError;

        error = device_add_class_symlinks(dev);
        if (error)
                goto SymlinkError;
        error = device_add_attrs(dev);
        if (error)
                goto AttrsError;
        error = bus_add_device(dev);
        if (error)
                goto BusError;
        error = dpm_sysfs_add(dev);
        if (error)
                goto DPMError;
        device_pm_add(dev);
        if (MAJOR(dev->devt)) {
                error = device_create_file(dev, &dev_attr_dev);
                if (error)
                        goto DevAttrError;

                error = device_create_sys_dev_entry(dev);
                if (error)
                        goto SysEntryError;

                devtmpfs_create_node(dev);
        }

        /* Notify clients of device addition.  This call must come
         * after dpm_sysfs_add() and before kobject_uevent().
         */
        if (dev->bus)
                blocking_notifier_call_chain(&dev->bus->p->bus_notifier,
                                             BUS_NOTIFY_ADD_DEVICE, dev);

        kobject_uevent(&dev->kobj, KOBJ_ADD);

        /*
         * Check if any of the other devices (consumers) have been waiting for
         * this device (supplier) to be added so that they can create a device
         * link to it.
         *
         * This needs to happen after device_pm_add() because device_link_add()
         * requires the supplier be registered before it's called.
         *
         * But this also needs to happen before bus_probe_device() to make sure
         * waiting consumers can link to it before the driver is bound to the
         * device and the driver sync_state callback is called for this device.
         */
        if (dev->fwnode && !dev->fwnode->dev) {
                dev->fwnode->dev = dev;
                fw_devlink_link_device(dev);
        }

        bus_probe_device(dev);
        if (parent)
                klist_add_tail(&dev->p->knode_parent,
                               &parent->p->klist_children);
        if (dev->class) {
                mutex_lock(&dev->class->p->mutex);
                /* tie the class to the device */
                klist_add_tail(&dev->knode_class,
                               &dev->class->p->klist_devices);

                /* notify any interfaces that the device is here */
                list_for_each_entry(class_intf,
                                    &dev->class->p->interfaces, node)
                        if (class_intf->add_dev)
                                class_intf->add_dev(dev, class_intf);
                mutex_unlock(&dev->class->p->mutex);
        }
done:
        put_device(dev);
        return error;
 SysEntryError:
        if (MAJOR(dev->devt))
                device_remove_file(dev, &dev_attr_dev);
 DevAttrError:
        device_pm_remove(dev);
        dpm_sysfs_remove(dev);
 DPMError:
        bus_remove_device(dev);
 BusError:
        device_remove_attrs(dev);
 AttrsError:
        device_remove_class_symlinks(dev);
 SymlinkError:
        device_remove_file(dev, &dev_attr_uevent);
 attrError:
        kobject_uevent(&dev->kobj, KOBJ_REMOVE);
        glue_dir = get_glue_dir(dev);
        kobject_del(&dev->kobj);
 Error:
        cleanup_glue_dir(dev, glue_dir);
parent_error:
        put_device(parent);
name_error:
        kfree(dev->p);
        dev->p = NULL;
        goto done;
}
EXPORT_SYMBOL_GPL(device_add);
```

### 创建设备属性文件：device_create_file

  

  

### 设备device与总线的关联

drivers/base/bus.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6b4f18e26d55f13c2f69b2aad65ddff5.png)


该代码是一个设备添加到总线的函数 bus_add_device()。 下面是对该函数的分析：

1. bus_get(dev->bus)： 通过设备的 bus 字段获取设备所属的总线类型（ bus_type） 的指针。这个函数会增加总线的引用计数， 确保总线在设备添加过程中不会被释放。
    
2. if (bus)： 检查总线类型指针是否有效。
    
3. pr_debug("bus: '%s': add device %s\n", bus->name, dev_name(dev))： 打印调试信息， 包括总线名称和设备名称， 以便跟踪设备添加的过程。
    
4. device_add_groups(dev, bus->dev_groups)： 将设备添加到总线类型的设备组（ dev_groups）中。 设备组是一组属性文件， 用于在设备的 sysfs 目录中显示和设置设备的属性。
    
5. sysfs_create_link(&bus->p->devices_kset->kobj, &dev->kobj, dev_name(dev))： 在总线类型的设备集（devices_kset） 的内核对象（ kobj） 下创建设备的符号链接。 这个符号链接将设备的sysfs 目录链接到总线类型的设备集目录中。
    
6. sysfs_create_link(&dev->kobj, &dev->bus->p->subsys.kobj, "subsystem")： 在设备的内核对象（kobj） 下创建指向总线类型子系统（ subsystem） 的符号链接。 这个符号链接将设备的 sysfs目录链接到总线类型子系统的目录中。
    
7. klist_add_tail(&dev->p->knode_bus, &bus->p->klist_devices)： 将设备的节点添加到总线类型的设备列表中。 这个步骤用于维护总线类型下的设备列表。
    

### bus_probe_device 匹配

bus_porbe_device

-->device_initial_probe

-->__device_attach

  

重点分析：__device_attach

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/740da1bf93d736d5ddfdbd4353db34fb.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/28799cb0c630e9c510ad53aa01c84c17.png)


Bus 中Match的调用

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/654113cacf60670ffb2f02e4acd008d2.png)


扫描：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/975cf89bec6c5f452d45db20ed8a1a05.png)
