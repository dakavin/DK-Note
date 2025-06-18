

📢`Linux`系统中一切皆文件。

## 什么是sysfs设备驱动管理

sysfs 文件系统是 Linux 内核提供的一种虚拟文件系统， 用于向用户空间提供内核中设备，驱动程序和其他内核对象的信息。 它以一种层次结构的方式组织数据， 并将这些数据表示为文件和目录， 使得用户空间可以通过文件系统接口访问和操作内核对象的属性。

sysfs 提供了一种统一的接口， 用于浏览和管理内核中的设备、 总线、 驱动程序和其他内核对象。 它在 /sys 目录下挂载， 用户可以通过查看和修改 /sys 目录下的文件和目录来获取和配置内核对象的信息。

## 虚拟文件系统 sysfs 目录层次

我们进入到 Linux 系统的/sys 目录下， 可以看到如下文件夹

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/98a2a57f3bb92ab9723c3ff0174d9edd.png)


和设备模型有关的文件夹为 bus， class， devices。 完整路径为如下所示：

- /sys/bus
    
- /sys/class
    
- /sys/devices
    

**/sys/devices**： 该目录包含了系统中所有设备的子目录。 每个设备子目录代表一个具体的设备， 通过其路径层次结构和符号链接反映设备的关系和拓扑结构。 每个设备子目录中包含了设备的属性、 状态和其他相关信息。 如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/91555b7f1a82cb55958bc7c80bc877e2.png)


**/sys/bus：** 该目录包含了总线类型的子目录。 每个子目录代表一个特定类型的总线， 例如PCI、 USB 等。 每个总线子目录中包含与该总线相关的设备和驱动程序的信息。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bafb78f4f287b28cfe59142d6eee6701.png)


比如 I2C 总线下连接的设备， 如下所示 ：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f2d75992f441722cba236e1bac3b9409.png)


**/sys/class**： 该目录包含了设备类别的子目录。 每个子目录代表一个设备类别， 例如磁盘、网络接口等。 每个设备类别子目录中包含了属于该类别的设备的信息。 如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2b1ab71316159ed2f29e80866eb39c07.png)


使用 class 进行归类的好处有以下几点：

1. 逻辑上的组织： 通过将设备按照类别进行归类， 可以在设备模型中建立逻辑上的组织结构。 这样， 相关类型的设备可以被放置在同一个类别目录下， 使得设备的组织结构更加清晰和可管理。
    
2. 统一的接口和属性： 每个设备类别目录下可以定义一组统一的接口和属性， 用于描述和配置该类别下所有设备的共同特征和行为。 这样， 对于同一类别的设备， 可以使用相同的方法和属性来操作和配置， 简化了设备驱动程序的编写和维护。
    
3. 简化设备发现和管理： 通过将设备进行分类， 可以提供一种简化的设备发现和管理机制。 用户和应用程序可以在类别目录中查找和识别特定类型的设备， 而无需遍历整个设备模型。这样， 设备的发现和访问变得更加高效和方便。
    
4. 扩展性和可移植性： 使用`class`进行归类可以提供一种扩展性和可移植性的机制。 当引入新的设备类型时， 可以将其归类到现有的类别中， 而无需修改现有的设备管理和驱动程序。这种扩展性和可移植性使得系统更加灵活， 并且对于开发人员和设备供应商来说， 更容易集成新设备。
    

  

比如应用现在要设置 gpio

如果使用类可以直接使用以下命令：

```C
echo 1 > /sys/class/gpio/gpio157/value  
```

如果不使用类， 使用以下命令：

```C
echo 1 > /sys/devices/platform/fe770000.gpio/gpiochip4/gpio/gpio157/value  
```

  

## 虚拟文件系统 sysfs 目录层次

为什么说 kobject 和 kset 是设备模型的基本框架呢？ 本小节来进行详细阐述。

当使用 kobject 时， 通常不会单独使用它， 而是将其嵌入到一个数据结构中。 这样做的目的是将高级对象接入到设备模型中。 比如 cdev 结构体和 platform_device 结构体， 如下所示：

cdev 结构体如下所示， 其成员有 kobject ：

设备模型1.0

```C
struct cdev {
    struct kobject kobj;//内嵌到 cdev 中的 kobject
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
}
```

设备模型2.0

platform_device结构体如下所示， 其成员有device结构体， 在device结构体中包含了kobject结构体。

```C
struct platform_device {
    const char*name;
    int id;
    bool id_auto;
    struct device dev;
    u32 num_resources;
    struct resource *resource;
    const struct platform_device_id *id_entry;
    char *driver_override; /* Driver name to force a match */
    /* MFD cell pointer */
    struct mfd_cell *mfd_cell;
    /* arch specific additions */
    struct pdev_archdata archdata;
};
```

device 结构体如下所示：

```C
struct device {
//设备的父设备
struct device *parent;
//私有指针
struct device_private *p;
//对应的 kobj
struct kobject kobj;
//设备初始化的名字
const char *init_name;
//设备类型
const struct device_type *type;
//设备所属的总线
struct bus_type *bus;
struct device_driver *driver;
...........
//设备所属的类
struct class *class;
//设备的属性组
const struct attribute_group **groups; /* optional groups */
...........
};
```

所以我们也可以把总线， 设备， 驱动看作是 kobject 的派生类。 因为他们都是设备模型中的实体， 通过继承或扩展 kobject 来实现与设备模型的集成。

在 Linux 内核中， kobject 是一个通用的基础结构， 用于构建设备模型。 每个 kobject 实例对应于 sys 目录下的一个目录， 这个目录包含了该 kobject 相关的属性， 操作和状态信息。 如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9875686a5b8dab1c7a52179c8920e3e6.png)


因此， 可以说 kobject 是设备模型的基石， 通过创建对应的目录结构和属性文件， 它提

供了一个统一的接口和框架， 用于管理和操作设备模型中的各个实体。

## Sysfs设备驱动管理相关注册接口

---

`Linux`系统中一切皆文件。

设备文件在哪里呢？它在`/dev`目录下，也在`/sys`目录下。它们直接有什么区别呢？

- `/dev`目录：该目录下面的文件是真实的设备文件，是应用层通过`mknod`创建的文件，通常系统中是由`udev`在运行时创建的。我们通常使用`open`、`write`、`ioctl`等函数操作设备，通常就是操作`/dev`目录下面的文件，它会间接调用到底层的驱动函数。
    
- `/sys`目录：这是由内核在运行时导出的，目的就是通过文件系统展示出设备、驱动和总线等层次关系。这也是这章节的重点。 那么先通过下图看一下`sysfs`文件系统是如何搭建起来的，图中左边是初始化流程，右边是对应sysfs文件系统的目录结构。
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/91bbf761e4943cc930f4fc4597944a76.png)


可以看到`sysfs`并不单纯是设备驱动相关的，还包括了内核模块、内核参数以及电源等诸多管理，当然这些在某个角度上也可以视为设备或者驱动模块。不过它们不是我们分析的重点，我们着重需要分析的主要是设备驱动相关的，这图只是给了我们心中一个谱，知道它都有些什么，在哪里初始化的。

这里不准备直接通过`sysfs`文件目录层次结构进行分析`sysfs`对设备驱动的管理。我们从设备、驱动和总线等的注册添加流程进行分析`sysfs`文件目录层次构造的。它们主要的注册函数分别为`device_register`、`driver_register`、`bus_register`，还有免不了的`class_register`。

### device_register都做了些什么？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1494199a9cd370a0670e9dd90162b982.png)


通过上图的`device_register`调用栈，可以看到设备主要的初始化都是在`/sys/devices/***/`下面创建一个自己名字的目录（例如`xxx-device`），然后在里面创建自己的属性文件接口。同时它会创建一个`subsystem`的链接，指向`bus`或者`class`，表示它归属的类型，挂在`bus`下面意味着它是由某个`bus`管控着，如果挂在`class`下面，这只是一个视角问题，其实质也是表示它具备着某些共同属性，乃至管理操作属性上的一致。然后在`/sys/bus`或者`/sys/class`下面就没有必要创建重复的东西了，直接创建一个链接指向`device`，意味着从它们的目录去看，可以看到`bus`或者`class`都管着哪些设备。经过`device_register`后，创建的目录和链接关系如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a2279ef5433da93cd48c18cb905d8437.png)


这里的文件及目录的管理方式起本质是数据结构的管理思想，目录可以视为结构体，文件是数据结构成员，而链接文件是数据指针。上图实际上就是这几个数据结构除了自身的初始化外，然后就是建立结构之间的关联关系。

### driver_register都做了些什么？

那么通过接着看一下`driver_register`都做了些什么？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7100cc36fdf4460a169eaead88b16409.png)


`driver_register`如上图调用关系，可以看到它比`device`的注册做的事情少多了。它主要是在`/sys/bus/xxx-bus/drivers`目录下创建自己名字的目录，然后在里面初始化驱动属性文件。同时创建链接指向`module`，表示该驱动是由哪个内核模块提供功能，同样`module`也反向指向驱动，表示它提供的是什么样的驱动能力，当然只有驱动模块才会有指向驱动的链接。经过`driver_register`后，其构建的目录和链接关系如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dd593f00e68c7b180d424f512478eae2.png)


接着看一下`bus_register`都做了什么？

### bus_register都做了些什么？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca854e3d8a9396e06635491c9aab8eea.png)


通过上图可以看到，它主要是在`/sys/bus`目录下面创建以自己名字命名的`bus`目录（这里名字举例为`xxx-bus`），然后会创建好`devices`和`drivers`给与它管理的设备和驱动进行注册，同时创建驱动的探测`drivers_probe`和`drivers_autoprobe`属性文件，以提供给用户触发去探测。当然还少不了`uevent`事件文件的创建，最后还有一些bus自己特有的属性。注册完`bus`后，将会生成目录结构如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc8cfdf9da8718ac39cfd8103a12b82d.png)


最后看一下`class_register`都做了些什么？

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0202a61a144efac51d2a3018ca2f7d92.png)


`class_register`的行为比较简单，仅仅是在`/sys/class`下面创建一个自己名字命名的目录，主要是提供给设备注册挂入链接。挂链接的动作在`device_register`里面完成。

至此，我们就大概对`sysfs`文件系统上的设备驱动注册和大概的关系逻辑了。如果通过`tree`命令去查看`/sys`肯定会发现`/sys/devices`目录下面的`PCI`、`USB`等设备的文件夹层层叠叠，这就涉及到了它们具体内部的一个管理关系了，这里不进行展开叙述。