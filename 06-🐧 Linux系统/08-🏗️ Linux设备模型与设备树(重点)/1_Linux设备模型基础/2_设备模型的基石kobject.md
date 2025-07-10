
在上一章我们了解了设备模型的发展历程和核心价值，现在让我们深入到设备模型的"心脏"部分：**kobject**和**kset**

简单来说，**kobject**就像是设备模型中的"基石"，而**kset**则是管理这些基石的"容器"。它们是Linux内核中所有设备对象的基础。

> [!note]+ 核心概念 
> kobject和kset是Linux内核设备模型的基础框架，理解它们是掌握设备模型的关键。每个kobject都对应sysfs中的一个目录，这是用户空间与内核交互的重要方式。

## 1 kobject：设备模型的基石

### 1.1 kobject的基本概念

**kobject**（内核对象）是内核中抽象出来的通用对象模型，用于表示内核中的各种实体。换句话说，kobject提供了一种统一的接口和机制，用于管理和操作内核对象

**kobject和核心作用**：
- 为内核对象提供统一的管理结构
- 建立对象之间的层次关系
- 实现引用计数管理
- 在sysfs中创建对应的目录结构

### 1.2 kobject结构体详解

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7f4618f5ab1561605fc45959d9371030.png)

kobject结构体定义在`include/linux/kobject.h`文件中：
```c
struct kobject {
    const char              *name;        // 对象名称
    struct list_head        entry;        // 链表节点
    struct kobject          *parent;      // 父对象指针
    struct kset             *kset;        // 所属的kset
    struct kobj_type        *ktype;       // 对象类型
    struct kernfs_node      *sd;          // sysfs目录项
    struct kref             kref;         // 引用计数
    unsigned int state_initialized:1;     // 初始化状态
    unsigned int state_in_sysfs:1;        // 是否在sysfs中
    unsigned int state_add_uevent_sent:1; // 是否发送了add事件
    unsigned int state_remove_uevent_sent:1; // 是否发送了remove事件
    unsigned int uevent_suppress:1;       // 是否抑制uevent
};
```
- `name`：kobject的名称，通用用于在`/sys`目录下创建对应的目录
- `entry`：用于将kobject链接到父kobject的子对象列表中，以建立层次关系，即**同级之间的kobj链表**
- `parent`：指向父kobject，**建立对象之间的层次关系**
- `kset`：指向包含该kobject的kset，用于**组织和管理kobject**
- `ktype`：定义kobject类型的结构体，**描述对象的属性和操作**
- `kref`：引用计数，确保对象在使用时不被意外释放
- `unsigned int *`：表示一些状态标志和配置选项

> [!tip]+ 层次关系 
> 通过parent指针，kobject可以建立类似目录树的层次结构，这正对应了sysfs文件系统的目录组织方式

### 1.3 kobject和sysfs的对应关系

每个kobject都会在sysfs中创建一个对应的目录：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cea607d86f7be01e225bd591621407b6.png)
- bus：总线相关
- class：设备类相关
- devices：设备相关
- kernel：内核相关
- 以此类推

> [!example]+ 实际应用 
> 当我们查看平台总线时，进入/sys/bus目录，可以看到platform、i2c、spi等各种总线目录，每个目录都对应一个kobject
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9623bb55b4dfdd006c19603ce2d5bd07.png)

### 1.4 kobject的树状结构

kobject通过parent指针形成树状关系，如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/16372965823beda680fa02f15d6ea519.png)
- 一个kobject可以有一个父kobject和多个子kobject
- 类似于文件系统的目录结构，一个目录可以有一个父目录和多个子目录， 通过目录的路径可以表示目录之间的层次关系
- 便于遍历、查找和管理
- 体现了设备模型的层次化组织

## 2 kobj_type：定义对象行为

### 2.1 kobj_type的作用

![whiteboard_exported_image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a1a7bfa3e511e8599c65e6d49bdfd382.png)

**kobj_type**描述kobject的行为方式，它定义了对象的操作方法：
```c
struct kobj_type {
    void (*release)(struct kobject *);      // 释放函数
    const struct sysfs_ops *sysfs_ops;      // sysfs操作
    struct attribute **default_attrs;       // 默认属性
};
```
- `release`：kobject_put函数进行对象释放时，调用的回调函数，必须实现
- `sysfs_ops`：定义sysfs属性的读写操作，允许内核对象共享这个公共的操作
- `default_attrs`：指向struct attribute元素列表的指针，它们将用作此类型每个对象的默认属性

> [!warning]+ 重要提醒 
> release函数必须实现，否则在对象释放时可能导致内存泄漏或系统不稳定。

### 2.2 sysfs_ops操作接口

```c
struct sysfs_ops {
    ssize_t (*show)(struct kobject *kobj, struct attribute *attr, char *buf);
    ssize_t (*store)(struct kobject *kobj, struct attribute *attr, 
                     const char *buf, size_t size);
};
```
- `show`：读取属性值时调用
- `store`：写入属性值时调用

## 3 kset：kobject的集合管理器

**kset（内核对象集合）** 是用于组织和管理一组相关kobject的容器。它提供了层次化的组织结构，可以将一组相关的kobject组织在一起

### 3.1 kset结构体详解

kset 在内核里面用 struct kset 结构体来表示， 定义在 `include/linux/kobject.h` 头文件中， 如下所示：
```c
struct kset {
    struct list_head list;                    // 链表头
    spinlock_t list_lock;                     // 链表保护锁
    struct kobject kobj;                      // 嵌入的kobject
    const struct kset_uevent_ops *uevent_ops; // uevent操作
};
```
- **list**：用于将kset链接到全局kset链表，以便对kset进行遍历和管理
- **list_lock**：保护对kset链表的并发访问，确保线程安全
- **kobj**：kset本身也是一个kobject，可以在/sys目录下创建对应的目录，即可以使用这个kobj来管理和操作kset
- **uevent_ops**：处理kset相关的uevent事件，例如在添加或删除 kobject 时发送相应的 uevent 通知

> [!info]+ 设计巧思 
> kset通过嵌入一个kobject，使得kset本身也成为设备模型的一部分，这体现了Linux内核一致性的设计理念。

### 3.2 kset与kobject的关系

在 Linux 内核中， kset 和 kobject 是相关联的两个概念， 它们之间存在一种层次化的关系，如下图所示：
![whiteboard_exported_image (1).png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ded18a779aa1450fff95ae6fba459de6.png)
1. kset 是 kobject 的一种扩展： kset 可以被看作是 kobject 的一种特殊形式， 它扩展了 kobject并提供了一些额外的功能。 kset 可以包含多个 kobject， 形成一个层次化的组织结构。
2. kobject 属于一个 kset： 每个 kobject 都属于一个 kset。kobject 结构体中的 `struct kset *kset`字段指向所属的 kset。 这个关联关系表示了 kobject 所在的集合或组织。

**关系特点**：
- 一个kset可以包含多个kobject
- 一个kobject只能属于一个kset
- kset为kobject提供集合管理和操作接口，用于组织和管理具有相似特性或关系的kobject


> [!tip]+ 管理优势 
> 通过kset，可以方便地对一组相关的kobject进行批量操作和管理，提高了系统的组织性和效率。

**设备模型示例：**
![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4a169544a5900fd8c86a806ee5ddc19b.png)
**上层部分**
- bus_kset：最顶层的管理，像一个“总线管理中心”，负责管理系统中所有的总线类型
- xxx_bus_kset：具体的总线实例（如USB总线、PCI总线、虚拟总线等）
- xxx_bus_type：总线类型的核心，包含
	- device_kset：管理该总线上的所有设备
	- driver_kset：管理该总线上的所有驱动程序
	- klist_devices：设备列表
	- klist_driver：驱动列表
**下层部分**
- device：代表具体的硬件设备
- xxx_device：特定类型的设备封装
- driver_private：驱动私有数据
- device_driver：具体的驱动程序

## 4 常用API

### 4.1 kobject操作相关

**创建和初始化**
```c
// 创建
struct kobject *kobject_create(void);
// 初始化
void kobject_init(struct kobject *kobi, struct kobj_type *ktype);
```

> [!tip]+ 提示
> 在创建kobject的时候，如果后续不做初始化操作的话，会赋值一个默认的ktype：`dynamic_kobj_ktype`
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4e94c5bc3e7078ff09122f9abce1b199.png)


**添加到系统**
```c
// 添加到系统
int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...);
// 初始化并添加
int kobject_init_and_add(struct kobject *kobj, struct kobj_type *ktype, struct kobject *parent, const char *fmt, ...); 
```

**便捷函数**
```c
// 创建并添加
struct kobject *kobject_create_and_add(const char *name, struct kobject *parent);
```

**使用示例**：
```c
// 方法一：使用便捷函数
struct kobject *mykobj;
mykobj = kobject_create_and_add("hello", NULL);

// 方法二：分步操作
struct kobject *mykobj;
mykobj = kobject_create();
if (mykobj) {
    kobject_init(mykobj, &mytype);
    if (kobject_add(mykobj, NULL, "%s", "hello")) {
        kobject_put(mykobj);
        mykobj = NULL;
    }
}
```

### 4.2 kset操作相关

**创建和管理**：
```c
struct kset *kset_create_and_add(const char *name,
                                const struct kset_uevent_ops *uevent_ops,
                                struct kobject *parent_kobj); // 创建并添加kset
void kset_unregister(struct kset *kset); // 注销kset
```

**将kobject添加到kset**：
```c
// 设置kobject的kset字段
mykobj->kset = my_kset;
// 然后初始化并添加kobject
kobject_init_and_add(mykobj, &mytype, NULL, "name");
```

> [!tip]+ 使用建议 
> 在实际开发中，推荐使用便捷函数如`kobject_create_and_add`和`kset_create_and_add`，它们内部会正确处理初始化和错误情况。

## 5 内核中的实际应用

### 5.1 系统目录创建示例

内核中创建/sys/class和/sys/devices目录的代码：

```c
static struct kobject *class_kobj   = NULL;
static struct kobject *devices_kobj = NULL;

/* 创建/sys/class */
class_kobj = kobject_create_and_add("class", NULL);
if (!class_kobj) {
    return -ENOMEM;
}

/* 创建/sys/devices */
devices_kobj = kobject_create_and_add("devices", NULL);
if (!devices_kobj) {
    return -ENOMEM;
}
```

### 5.2 kset使用示例

```c
static struct kobject foo_kobj, bar_kobj;

// 创建kset
example_kset = kset_create_and_add("kset_example", NULL, kernel_kobj);

// 将kobject加入kset
foo_kobj.kset = example_kset;
bar_kobj.kset = example_kset;

// 初始化并添加kobject
retval = kobject_init_and_add(&foo_kobj, &foo_ktype, NULL, "foo_name");
retval = kobject_init_and_add(&bar_kobj, &bar_ktype, NULL, "bar_name");
```

> [!abstract]+ 章节总结 kobject和kset构成了Linux设备模型的基础框架：
> 
> - **kobject**是所有内核对象的基类，提供统一的对象模型
> - **kset**是kobject的容器，提供集合管理功能
> - 每个kobject在sysfs中对应一个目录
> - kobj_type定义了对象的行为和属性
> - 通过层次化的组织，构建了完整的设备树

这套框架为上层的总线、设备、驱动提供了统一的基础，是理解整个设备模型的关键。下一章我们将通过实际编程来掌握kobject的使用方法！
