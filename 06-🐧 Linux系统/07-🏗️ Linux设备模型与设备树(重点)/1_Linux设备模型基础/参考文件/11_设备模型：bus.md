## BUS总线重点知识点

**结构体：**

- struct bus_type
    
- struct bus_attribte
    

**如何注册一个总线**

- 注册接口： bus_register
    
- 实现总线的match方法
    
- 在bus下创建属性文件
    

  

## bus_type结构体


总线是处理器与一个或者多个设备连接的通道。将系统中的设备按照总线来组织非常直观，有助于设备的管理。总线在内核中通过 bus_type 结构体来表示，其结构如下：

```C
struct bus_type { 
        const char *name;                        //总线类型名称
        struct bus_attribute *bus_attrs;         //总线属性
        struct device_attribute *dev_attrs;      //设备属性
        struct driver_attribute *drv_attrs;      //驱动属性
        int (*match)(struct device *dev, struct device_driver *drv);    //匹配检查函数
        int (*uevent)(struct device *dev, struct kobj_uevent_env *env); //uevent 控制函数
        ...
        struct subsys_private *p;
        struct lock_class_key lock_key
}
```

其中 subsys_private 是一个很重要的结构体，其定义为：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/94b9a758747f4517a63a88347c1706e9.png)


```C
struct subsys_private {
        struct kset subsys;                 //该 kobject 是总线在内核中的表示
        struct kset *drivers_kset;          //属于该总线的 driver 的 kset 
        struct kset *devices_kset;          //属于该总线的 device 的 kset 
        struct klist klist_devices;         //该总线上 device，klist 形式组织，用于遍历 devices_kset 
        struct klist klist_drivers;         //该总线上 driver，klist 形式组织，用于遍历 drivers_kset 
        struct blocking_notifier_head bus_notifier; // bus notifier 链表
        unsigned int drivers_autoprobe:1;    //驱动自动加载标志位
        struct bus_type *bus;                //该结构体的回指指针
}
```

struct subsys_private 是一个结构体， 用于保存驱动核心子系统（bus） 的私有信息。 每个子系统都可以有私有数据， 这些私有数据存储在 struct subsys_private 结构体中。

## bus_register的注册流程分析

接下来我们追踪 bus_register 函数， 函数原型如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca854e3d8a9396e06635491c9aab8eea.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bb1c07b259034df4466d96edfc8e8309.png)


```C
/*
bus_register - 注册一个驱动核心子系统
@bus: 要注册的总线
注册总线时， 首先将总线与 kobject 基础设施关联起来， 然后注册属于该总线的子系统：
归属于该子系统的设备和驱动程序。
*/
int bus_register(struct bus_type *bus)
{ 
    int retval;
    struct subsys_private *priv;
    struct lock_class_key *key = &bus->lock_key;
    
    // 分配并初始化一个 subsys_private 结构体， 用于保存子系统的相关信息
    priv = kzalloc(sizeof(struct subsys_private), GFP_KERNEL);
    if (!priv)
        return -ENOMEM;
    priv->bus = bus;
    bus->p = priv;
    
    // 初始化一个阻塞通知链表 bus_notifier
    BLOCKING_INIT_NOTIFIER_HEAD(&priv->bus_notifier);
    
    // 设置子系统的名称
    retval = kobject_set_name(&priv->subsys.kobj, "%s", bus->name);
    if (retval)
        goto out;
        
    // 设置子系统的 kset 和 ktype
    priv->subsys.kobj.kset = bus_kset;
    priv->subsys.kobj.ktype = &bus_ktype;
    priv->drivers_autoprobe = 1;
    
    // 注册子系统的 kset  ///sys/bus/xxx-name 
    retval = kset_register(&priv->subsys);
    if (retval)
        goto out;
       
    // 在总线上创建一个属性文件
    retval = bus_create_file(bus, &bus_attr_uevent);
    if (retval)
        goto bus_uevent_fail;
    
    // 创建并添加 "devices" 子目录的 kset
    priv->devices_kset = kset_create_and_add("devices", NULL, &priv->subsys.kobj);
    if (!priv->devices_kset) {
        retval = -ENOMEM;
        goto bus_devices_fail;
    } 
    
    // 创建并添加 "drivers" 子目录的 kset
    priv->drivers_kset = kset_create_and_add("drivers", NULL, &priv->subsys.kobj);
    if (!priv->drivers_kset) {
        retval = -ENOMEM;
        goto bus_drivers_fail;
    } 
    
    // 初始化接口链表、 互斥锁和设备/驱动的 klist
    INIT_LIST_HEAD(&priv->interfaces);
    __mutex_init(&priv->mutex, "subsys mutex", key);
    klist_init(&priv->klist_devices, klist_devices_get, klist_devices_put);
    klist_init(&priv->klist_drivers, NULL, NULL);
    
    // 添加驱动探测文件
    retval = add_probe_files(bus);
    if (retval)
        goto bus_probe_files_fail;
    
    // 添加总线的属性组
    retval = bus_add_groups(bus, bus->bus_groups);
    if (retval)
    goto bus_groups_fail;
    
    // 打印调试信息， 表示总线注册成功
    pr_debug("bus: '%s': registered\n", bus->name);
    return 0;
    
    bus_groups_fail:
    remove_probe_files(bus);
    bus_probe_files_fail:
    kset_unregister(bus->p->drivers_kset);
    bus_drivers_fail:
    kset_unregister(bus->p->devices_kset);
    bus_devices_fail:
    bus_remove_file(bus, &bus_attr_uevent);
    bus_uevent_fail:
    kset_unregister(&bus->p->subsys);
    
    out:
    kfree(bus->p);
    bus->p = NULL;
    return retval;
} 
EXPORT_SYMBOL_GPL(bus_register);
```

1. 分配并初始化一个 subsys_private 结构体， 该结构体用于保存子系统的相关信息。
    
2. 将 bus 结构体与 subsys_private 关联起来。
    
3. 初始化一个阻塞通知链表 bus_notifier。
    
4. 通过 kobject_set_name() 设置子系统的名称。
    
5. 设置子系统的相关属性， 如 kset 和 ktype。
    
6. 注册子系统的 kset。
    
7. 创建并添加 "devices" 和 "drivers" 两个子目录的 kset。
    
8. 初始化子系统的接口链表、 互斥锁和设备/驱动的 klist。
    
9. 添加驱动探测文件。
    
10. 添加总线的属性组。
    
11. 打印调试信息， 表示总线注册成功。
    

  

结果：

两个子目录：devices、drivers

三个属性文件:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a75d49ea50d69312bf76afb86f0b5859.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1dfecd0f25ba9b6538d4a157f7c310bd.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4585ca28208beca4e7e0ca5c26704519.png)

## bus_create_file 过程分析

bus_create_file() 函数用于在总线的 sysfs 目录下创建一个属性文件
```c
int bus_create_file(struct bus_type *bus, struct kobject *kobj, const struct attribute *attr);
```

参数说明：
- bus： 指向总线类型结构体 struct bus_type 的指针， 表示要创建属性文件的总线。
- kobj： 指向内核对象 struct kobject 的指针， 表示要在其下创建属性文件的内核对象。
- attr： 指向属性 struct attribute 的指针， 表示要创建的属性文件的属性。

返回值： 成功时返回 0， 否则返回负数错误代码。

调用 bus_create_file()函数之前， 需要先定义好属性结构体 struct attribute， 并将其相关字段填充好。 通常， 属性结构体会包含以下字段：

```C
struct bus_attribute mybus_attr = {
    .attr = {
        .name = "value",
        .mode = 0664,
    },
    .show = mybus_show,
};
ret = bus_create_file(&mybus, &mydevice.kobj, &mybus_attr.attr);
```

上述示例代码创建了一个名为 "value" 的属性文件， 并指定了访问权限为 0664。 在创建属性文件时， 还可以指定其他属性的回调函数， 如 .show、 .store 等， 以实现对属性值的读取和写入操作。

请注意， 在使用 bus_create_file() 函数之前， 需要确保总线对象和内核对象已正确初始化和注册。

## 详细

- priv->bus = bus;将 priv 结构体中的 bus 成员设置为当前注册的总线。 这样做的目的是将 bus 成员与当前总线建立关联。 通过将 bus 成员设置为当前总线， priv 结构体可以获取并访问与该总线相关的信息和功能。 这种关联可以使 priv 结构体在操作当前总线时更加方便和高效
    
- bus->p = priv;将 priv 结构体指针存储在当前注册的总线结构体的成员 p 中，目的是让当前注册的总线结构体能够快速地找到并访问与之关联的 priv 结构体。 通过将 priv结构体指针存储在总线结构体的成员中， 总线可以轻松地获取与之相关的私有数据结构。 这种关联使得总线能够直接访问和操作与特定总线相关的数据和功能， 而无需通过其他方式来查找或传递指针。
    

  

klist_init 函数用于初始化两个内核链表(klist)， 分别是 priv->klist_devices 和priv->klist_drivers。

  

- klist_init(&priv->klist_devices, klist_devices_get, klist_devices_put);这行代码初始化了名为priv->klist_devices 的内核链表。 klist_devices_get 和 klist_devices_put 是两个回调函数， 用于在向链表添加或移除元素时执行相应的操作。 通常， 这些回调函数用于在链表中的每个元素被引用或释放时执行额外的操作。 例如， 当设备被添加到链表时， klist_devices_get函数可能会增加设备的引用计数； 当设备从链表中移除时， klist_devices_put 函数可能会减少设备的引用计数。
    
- klist_init(&priv->klist_drivers, NULL, NULL);这行代码初始化了名为 priv->klist_drivers 的内核链表， 但与第一个初始化不同， 这里没有提供回调函数。 因此， 这个链表在添加或移除元素时不会执行额外的操作。 这种情况下， 链表主要用于存储驱动程序对象， 而不需要附加的处理逻辑。
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4a169544a5900fd8c86a806ee5ddc19b.png)


## 那么什么是子系统呢？

在 Linux 中， 子系统是一种机制， 用于将特定功能的实现抽象为一个独立的实体。 它提供了一种方便的方式， 将相关的代码和数据结构组织在一起， 以实现特定的功能。 子系统可以被视为一个功能模块， 它封装了相关的功能和操作， 使得用户和应用程序可以通过统一的接口与其交互。

  

在 Linux 中， 存在许多常见的子系统， 每个子系统都负责实现特定的功能。 以下是一些常见的子系统示例。

虚拟文件系统（**VFS**） 子系统： VFS 子系统提供了对不同文件系统的统一访问接口， 使得应

  

用程序可以透明地访问各种文件系统（如 ext4、 NTFS、 FAT 等） ， 而无需关心底层文件系

统的具体实现。

- 设备驱动子系统： 设备驱动子系统管理和控制硬件设备的驱动程序。 它提供了与硬件设备交互的接口， 使得应用程序可以通过驱动程序与设备进行通信和控制。
    
- 网络子系统： 网络子系统负责管理和控制网络相关的功能。 它包括网络协议栈、 套接字接口、 网络设备驱动程序等， 用于实现网络通信和网络协议的处理。
    
- 内存管理子系统： 内存管理子系统负责管理系统的物理内存和虚拟内存。 它包括内存分配、页面置换、 内存映射等功能， 用于有效地分配和管理系统的内存资源。
    
- 进程管理子系统： 进程管理子系统负责管理和控制系统中的进程。 它包括进程的创建、 调度、 终止等功能， 以及进程间通信的机制， 如信号、 管道、 共享内存等。
    
- 电源管理子系统： 电源管理子系统负责管理和控制系统的电源管理功能。 它可以用于控制电源的开关、 电源模式的切换、 节能功能的实现等。
    
- 文件系统子系统： 文件系统子系统负责管理和控制文件系统的创建、 格式化、 挂载、 数据存取等操作。 它支持各种文件系统类型， 如 ext4、 FAT、 NTFS 等。
    
- 图形子系统： 图形子系统负责管理和控制图形显示功能， 包括显示驱动程序、 窗口管理、图形渲染等。 它提供了图形界面的支持， 使得用户可以通过图形方式与计算机交互。
    

  

  

## 总结

通过分析 bus_register 函数， 我们对设备模型有了更深层次的感悟， 如下所示：

1. kobject 和 kset 是设备模型的基本框架， 它们可以嵌入到其他结构体中以提供设备模型的功能。 kobject 代表设备模型中的一个对象， 而 kset 则是一组相关的 kobject 的集合。
    
2. 属性文件在设备模型中具有重要作用， 它们用于在内核空间和用户空间之间进行数据交换。 属性文件可以通过 sysfs 虚拟文件系统在用户空间中表示为文件， 用户可以读取或写入这些文件来与设备模型进行交互。 属性文件允许用户访问设备的状态、 配置和控制信息， 从而实现了设备模型的管理和配置。
    
3. sysfs 虚拟文件系统在设备模型中扮演关键角色， 它可以将设备模型的组织层次展现出来。 通过 sysfs， 设备模型中的对象、 属性和关系可以以目录和文件的形式在用户空间中表示。这种组织形式使用户能够以层次结构的方式浏览和管理设备模型， 从而方便地获取设备的信息、 配置和状态。 sysfs 提供了一种统一的接口， 使用户能够通过文件系统操作来与设备模型进行交互， 提供了设备模型的可视化和可操作
    

  

## 案例

i2c:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fb222cffd714f16437ddfd9348b9806b.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8e81f59009db0f4e456848b5c15db692.png)
