
在掌握了kobject的使用方法后，现在让我们学习kset的实际应用。kset作为kobject的集合管理器，在设备模型中扮演着重要的组织角色。

简单来说，**kset**就像是一个"文件夹"，可以将相关的kobject组织在一起，提供统一的管理和访问方式。

> [!note]+ 学习目标 
> 通过本章学习，你将掌握kset的创建方法、管理机制，以及如何将多个kobject组织到一个kset中进行统一管理。

## 1 kset的核心概念回顾

### 1.1 kset的作用

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e3543fc6e87d43f67d55189d66658bc3.png)

**kset提供的主要功能**：
- **集合管理**：将相关的kobject组织在一起
- **层次结构**：为kobject提供父级容器
- **事件处理**：统一处理集合内对象的uevent事件
- **访问接口**：提供集合级别的属性和操作

> [!tip]+ 理解kset 
> 可以把kset想象成一个"部门"，而kobject就是部门里的"员工"。部门负责管理员工，提供统一的规章制度和工作环境。

### 1.2 kset与kobject的关系

**关系特点**：
- 一个kset可以包含多个kobject
- 一个kobject只能属于一个kset
- kset本身也是一个kobject（包含嵌入的kobject）
- 通过kset可以对其包含的所有kobject进行批量操作

## 2 kset编程接口

### 2.1 核心API函数

**创建和注册**：
```c
// 创建、初始化并注册kset
struct kset *kset_create_and_add(const char *name, 
                                const struct kset_uevent_ops *uevent_ops,
                                struct kobject *parent_kobj);

// 手动创建和注册（分步操作）
void kset_init(struct kset *kset);
int kset_register(struct kset *kset);

// 注销kset
void kset_unregister(struct kset *kset);
```
- **name**：kset的名称，将作为sysfs中的目录名
- **uevent_ops**：可选的uevent操作函数指针
- **parent_kobj**：父kobject，决定kset在sysfs中的位置

### 2.2 kset的创建流程

当我们调用`kset_create_and_add()`时，内核执行以下操作：
1. **分配内存**：为kset结构体分配内存空间
2. **初始化kset**：设置基本属性和链表
3. **初始化嵌入的kobject**：kset自身的kobject表示
4. **注册到sysfs**：在sysfs中创建对应的目录
5. **设置父子关系**：建立与父kobject的关联

> [!info]+ 设计巧思 
> kset通过嵌入kobject的方式，使得kset本身也具备了kobject的所有特性，这体现了Linux内核统一的对象模型设计。

## 3 kset实验和内核案例

### 3.1 自定义kset

通过编写一个内核模块，我们将：
1. 创建一个kset
2. 创建两个kobject并添加到kset中
3. 观察在sysfs中的目录结构
4. 验证kset的集合管理功能

**完整实验代码**：
```c fold
#include <linux/init.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

//定义kobj指针变量
struct kobject *dk_kobj1;
struct kobject *dk_kobj2;

//定义kset指针变量
struct kset *dk_kset;

//定义ktype指针变量，用于kobj
struct kobj_type *dk_ktype;

//模块初始化函数
static int __init dk_kset_init(void){ 
    int ret;

    printk(KERN_INFO "开始创建kset和kobj\n");

    //1.创建并添加kset，名称为dk_kset,没有特殊的uevent操作，父对象为NULL
    dk_kset = kset_create_and_add("dk_kset", NULL, NULL);
    if(!dk_kset){
        printk(KERN_ERR "创建kset失败\n");
        return -ENOMEM;
    }

    printk(KERN_INFO "创建kset成功：%s\n",dk_kset->kobj.name);

    //2.创建第一个kobj添加到kset中
    dk_kobj1 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    if(!dk_kobj1){
        printk(KERN_ERR "创建dk_kobj1失败\n");
        ret = -ENOMEM;
        goto cleanup_kset;
    }
    //设置kobject属于kset
    dk_kobj1->kset = dk_kset;
    //初始化并添加到系统中
    ret = kobject_init_and_add(dk_kobj1, dk_ktype, NULL, "dk_kobj1");
    if(ret){
        printk(KERN_ERR "初始化dk_kobj1失败\n");
        kfree(dk_kobj1);
        goto cleanup_kset;
    }
    printk(KERN_INFO "创建dk_kobj1成功\n");

    //3.创建第二个kobj添加到kset中
    dk_kobj2 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    if(!dk_kobj2){
        printk(KERN_ERR "创建dk_kobj2失败\n");
        ret = -ENOMEM;
        goto cleanup_obj1;
    }
    //设置kobject属于kset
    dk_kobj2->kset = dk_kset;
    //初始化并添加到系统中
    ret = kobject_init_and_add(dk_kobj2, dk_ktype, NULL, "dk_kobj2");
    if(ret){
        printk(KERN_ERR "初始化dk_kobj2失败\n");
        kfree(dk_kobj2);
        goto cleanup_obj1;
    }
    printk(KERN_INFO "创建dk_kobj2成功\n");

    printk(KERN_INFO "创建kset和kobj成功,请查看/sys/kernel/dk_kset目录下的dk_kobj1和dk_kobj2");
    return 0;

cleanup_obj1:
    kobject_put(dk_kobj1);
cleanup_kset:
    kset_unregister(dk_kset);
    return ret;
}

static void __exit dk_kset_eixt(void){
    printk(KERN_INFO "开始删除kset和kobj\n");
    //释放kobj（会自动从kset中移除）
    kobject_put(dk_kobj1);
    kobject_put(dk_kobj2);
    //注销kset
    kset_unregister(dk_kset);
    printk(KERN_INFO "删除kset和kobj成功\n");
}

module_init(dk_kset_init);
module_exit(dk_kset_eixt);

MODULE_LICENSE("GPL");
```

**实验结果验证**：编译并加载模块后，在开发板上验证结果
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/56b86a6567b05a4af9fb13db384f61f5.png)

**在/sys目录下的结构**：
```bash
/sys/
└── dk_kset/              # 我们创建的kset目录
    ├── dk_kobj1          # 第一个kobject
    └── dk_kobj2          # 第二个kobject
```

**验证命令**：
```bash
# 查看创建的目录结构
ls -la /sys/dk_kset/

# 查看kset的属性
ls -la /sys/dk_kset/

# 检查kobject的详细信息
find /sys -name "dk_kset" -type d
tree /sys/dk_kset/
```

### 3.2 内核案例1：/sys/bus目录

让我们看看Linux内核是如何使用kset创建/sys/bus目录的：
```c
// 在drivers/base/bus.c中
static struct kset *bus_kset;

// 初始化总线子系统
int __init buses_init(void)
{
    // 创建bus kset，位于/sys目录下
    bus_kset = kset_create_and_add("bus", &bus_uevent_ops, NULL);
    if (!bus_kset)
        return -ENOMEM;
    
    return 0;
}
```

这段代码创建了`/sys/bus`目录，所有的总线类型都会作为kobject添加到这个kset中
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5698deb844fc5805038cc0bffa5281c6.png)

**结果展示**：
```bash
/sys/bus/
├── platform/     # platform总线
├── i2c/          # I2C总线
├── spi/          # SPI总线
├── usb/          # USB总线
└── pci/          # PCI总线
```

> [!example]+ 实际应用 
> 每个总线类型注册时，都会创建一个kobject并添加到bus_kset中，这样所有总线都统一管理在/sys/bus目录下

### 3.3 内核案例2：kset的uevent机制

kset还可以提供统一的uevent处理机制：
```c
// 定义kset的uevent操作
static const struct kset_uevent_ops my_uevent_ops = {
    .filter = my_uevent_filter,    // 过滤函数
    .name   = my_uevent_name,      // 名称函数
    .uevent = my_uevent,           // 事件处理函数
};

// uevent过滤函数
static int my_uevent_filter(struct kset *kset, struct kobject *kobj)
{
    // 返回1表示允许发送uevent，0表示阻止
    return 1;
}

// uevent名称函数
static const char *my_uevent_name(struct kset *kset, struct kobject *kobj)
{
    return "MY_SUBSYSTEM";
}

// uevent事件函数
static int my_uevent(struct kset *kset, struct kobject *kobj,
                     struct kobj_uevent_env *env)
{
    // 添加环境变量
    add_uevent_var(env, "CUSTOM_VAR=%s", "custom_value");
    return 0;
}

// 创建带uevent功能的kset
struct kset *my_kset = kset_create_and_add("my_subsystem", 
                                           &my_uevent_ops, NULL);
```

> [!tip]+ uevent用途 
> uevent机制允许内核向用户空间发送设备变化通知，这是热插拔和设备管理的基础。

## 4 kset的高级特性

### 4.1 层次化管理

kset可以建立多层次的管理结构
```c
// 创建父级kset
struct kset *parent_kset = kset_create_and_add("parent", NULL, NULL);

// 创建子级kset，父对象是parent_kset的kobject
struct kset *child_kset = kset_create_and_add("child", NULL, &parent_kset->kobj);

// 结果：/sys/parent/child/
```

### 4.2 引用计数管理

kset会正确管理其包含对象的引用计数：
```c
// 当kobject加入kset时
kobject->kset = my_kset;
kset_get(my_kset);  // kset引用计数增加

// 当kobject从kset移除时
kset_put(my_kset);  // kset引用计数减少
```

### 4.3 遍历kset中的对象

```c
// 遍历kset中的所有kobject
struct kobject *kobj;
struct list_head *entry;

spin_lock(&my_kset->list_lock);
list_for_each(entry, &my_kset->list) {
    kobj = container_of(entry, struct kobject, entry);
    // 处理每个kobject
    printk("找到kobject: %s\n", kobject_name(kobj));
}
spin_unlock(&my_kset->list_lock);
```

> [!warning]+ 并发安全 
> 遍历kset时一定要使用适当的锁保护，避免并发访问导致的问题。

## 5 常见使用模式

### 5.1 模式1：设备分类管理

```c
// 创建设备类型kset
struct kset *sensor_kset = kset_create_and_add("sensors", NULL, NULL);

// 添加不同类型的传感器设备
struct kobject *temp_sensor = kzalloc(sizeof(struct kobject), GFP_KERNEL);
temp_sensor->kset = sensor_kset;
kobject_init_and_add(temp_sensor, &sensor_type, NULL, "temperature");

struct kobject *humid_sensor = kzalloc(sizeof(struct kobject), GFP_KERNEL);
humid_sensor->kset = sensor_kset;
kobject_init_and_add(humid_sensor, &sensor_type, NULL, "humidity");

// 结果：/sys/sensors/temperature 和 /sys/sensors/humidity
```

### 5.2 模式2：功能模块管理

```c
// 为每个功能模块创建kset
struct kset *network_kset = kset_create_and_add("network", NULL, NULL);
struct kset *storage_kset = kset_create_and_add("storage", NULL, NULL);

// 在各自的kset下管理相关设备
// 网络设备添加到network_kset
// 存储设备添加到storage_kset
```

## 6 调试和故障排除

### 6.1 常见问题及解决方案

**问题1：kset创建失败**
```c
struct kset *my_kset = kset_create_and_add("test", NULL, NULL);
if (!my_kset) {
    printk(KERN_ERR "kset创建失败，可能原因：\n");
    printk(KERN_ERR "1. 内存不足\n");
    printk(KERN_ERR "2. 名称冲突\n");
    printk(KERN_ERR "3. 父对象无效\n");
    return -ENOMEM;
}
```

**问题2：kobject添加到kset失败**
```c
// 确保在kobject_init_and_add之前设置kset
my_kobject->kset = my_kset;  // 必须在init_and_add之前设置
ret = kobject_init_and_add(my_kobject, &my_type, NULL, "name");
```

**问题3：内存泄漏**
```c
// 正确的清理顺序
static void cleanup_resources(void)
{
    // 1. 先释放所有kobject
    if (mykobject01) kobject_put(mykobject01);
    if (mykobject02) kobject_put(mykobject02);
    
    // 2. 再注销kset
    if (mykset) kset_unregister(mykset);
}
```

### 6.2 调试技巧

```bash
# 查看kset信息
cat /sys/kernel/debug/kobject_uevent_seqnum
echo 1 > /sys/kernel/debug/kobject/debug

# 监控对象创建和删除
dmesg | grep -i kobject

# 检查sysfs结构
find /sys -name "your_kset_name" -type d
tree /sys/your_kset_name/
```

> [!abstract]+ 实践总结 
> 通过本章的学习，我们掌握了：
> 
> - **kset的基本概念**和在设备模型中的作用
> - **kset的创建方法**和API使用
> - **kobject与kset的关联**机制
> - **内核实际应用**案例分析
> - **高级特性**如uevent处理和层次化管理
> - **常见问题**的调试和解决方法

kset作为kobject的集合管理器，为设备模型提供了强大的组织和管理能力。下一章我们将学习如何为这些对象添加属性文件，实现用户空间与内核的交互！