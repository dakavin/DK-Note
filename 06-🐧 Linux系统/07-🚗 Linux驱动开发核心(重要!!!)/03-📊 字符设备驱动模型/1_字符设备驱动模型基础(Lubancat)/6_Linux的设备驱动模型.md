---
文章标题: "[[6_Linux的设备驱动模型]]"
文章作者: Dakkk
文章概要: |
  文章深入解析Linux设备模型，阐述设备、驱动、总线、类四大核心概念及注册、匹配、绑定、移除工作流程。详细讲解`/sys`文件系统下的设备信息与属性文件交互，并提供自定义总线、设备、驱动及属性代码示例，帮助理解内核驱动分层管理。
tags:
  - Linux内核
  - 设备驱动
  - 设备模型
  - 总线驱动
  - sysfs
  - 驱动开发
  - 内核模块
相关文章:
  - "[[2_📕Linux内核模块]]"
  - "[[1_驱动关键技术]]"
  - "[[../../../../00-🎯 学习路线/1_驱动开发（有线&无线）]]"
  - "[[../../../../02-💻 开发环境/1_训练营笔记/4_Ubuntu搭建Linux开发环境]]"
  - "[[../../../05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/5_中断系统中的设备树]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/6_Linux的设备模型.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-05-06 16:55:39
修改时间: 2025-06-02 11:15:11
---

## 1 前言

### 1.1 总结前面

在前面写的驱动中，可以发现编写驱动有个固定的模式，大致流程可以总结如下
- 实现 加载函数`xxx_init()` 和 卸载函数`xxx_exit()`
- 申请设备号`register_chrdev_region()`
- 初始化字符设备，`cdev_init()`、`cdev_add()`
- 硬件初始化，如时钟寄存器配置使能，GPIO设置为输入输出模式等
- 构建`file_operations`结构体内容，实现硬件的各个相关操作
- 创建设备节点
	- 在终端使用mknod根据设备号来进行创建设备节点
	- 自动创建：使用`class_create`创建设备类，类的下面`device_create`创建设备节点

思维导图如下：
![char_device_driver_flowchart.svg|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7dbab5b0cc630cbf2e489dca430fc46f.svg)
因此，在Linux开发驱动，只要掌握这些 **套路**，开发驱动不是难事。在内核源码的`drivers`中存放了大量的设备驱动代码，可以查看这里的内容，说不定可以找到需要的代码：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b278c344c6939cefdfab04e544fe436c.png)

### 1.2 问题引入

**问题引入**：我们将硬件信息都写在驱动代码里了，根据某个硬件编写的驱动只要修改了一下引脚接口，这个驱动代码就得重新修改才能使用，显然是不合理的，那有没有合适的解决方案呢？

**答案是肯定的！！！😜**

**解决方案**：
- Linux引入了 **设备驱动模型分层** 的概念，将编写的驱动代码分成了两部分
	- `设备`：提供硬件资源
	- `驱动`：使用设备提供的硬件资源
- 由 `总线` 将它们联系起来

![device_driver_model.svg|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/707661b5d5903c27ee91443e503a92c5.svg)

### 1.3 设备驱动模型基础概念

#### 1.3.1 核心概念-四个重要角色

**设备（Device）🔌**
- 挂载在某个总线上的物理设备
- 类似于电脑上的硬件（鼠标、键盘等）
- eg. USB鼠标就是一个挂载在USB总线上的设备

**驱动（Driver）💿**
- 与特定设备相关的软件，负责初始化设备并提供操作方式
- 类似于设备的“说明书”，告诉系统怎么使用这个设备
- eg. 鼠标驱动告诉系统如何读取鼠标的移动和点击

**总线（Bus）🚌**
- 负责管理挂载在对应总线上的设备和驱动
- 类似于公交车站，管理着所有的设备和驱动，帮它们找到彼此
- 作用：
	- 当新设备插入时，总线帮忙找到合适的驱动
	- 当新驱动安装时，总线帮忙找到对应的设备

**类（Class）📁**
- 对于具有相同功能的设备，归结到一种类别进行分类管理
- 类似于，按功能分类的文件夹，比如把所有输入设备放在一起
- eg. 不管是USB鼠标还是蓝牙鼠标，都是input类

#### 1.3.2 /sys目录结构-Linux设备的户口本

**重要概念**：Linux中一切皆文件，设备信息都记录在`/sys`目录下

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1e2f506241b70e8f9fc1cf09da767f24.png)

**/sys/bus - 按总线分类（注册好的总线类型）**
```txt
/sys/bus/
├── usb/           # USB总线
│   ├── devices/   # 这个总线上的所有设备（都是符号链接）
│   └── drivers/   # 这个总线上的所有驱动
├── pci/           # PCI总线  
│   ├── devices/   
│   └── drivers/
└── ...
```
- 每个子目录（总线类型）下都包含两个子目录---devices和drivers文件夹
- 对于devices来说
	- 符号链接指向真正的设备（`/sys/devices/下`）
	- eg. 上图，bus/usb/devices/某个设备 ---> devices/pci()/dev 0:10/usb2
- 对于drivers来说
	- 都是注册在总线上的驱动
	- 每个driver子目录是一些可以观察和修改的driver参数

**/sys/devices - 按物理结构分类（全局设备结构体系）**
```txt
/sys/devices/
└── pci0000:00/
    └── 0000:00:1d.0/
        └── usb2/
            └── 2-1/   # 具体的USB设备
```
- 按设备的`在总线上的拓扑结构`来显示，就像家谱一样

**/sys/class - 按功能分类（注册在kernel内所有的设备类型）**
```txt
/sys/class/
├── input/         # 输入设备类
│   ├── mouse0     # 鼠标
│   └── keyboard0  # 键盘
├── net/           # 网络设备类
└── ...
```

#### 1.3.3 工作流程

![设备驱动总线模型工作流程图.svg|800](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/53e8fc5fdbf1b5062b1eb2bef7ba8f04.svg)


**第一步：注册阶段**
- 设备注册：新设备插入时，在总线的`设备链表`中登记
- 驱动注册：新驱动安装时，在总线的`驱动链表`中登记

**第二步：匹配阶段（Match）**
- 过程：插入的同时，bus执行一个bus_type结构体中的match方法，帮设备和驱动配对
- 方式：最简单的就是比较名字，名字相同就配对成功
- 举例：USB鼠标设备和USB鼠标驱动名字一样，就配对成功

**第三步：绑定阶段（Probe）**
- 过程：配对成功，调用驱动device_driver结构体中的probe函数
	- probe中获取设备资源，具体的功能由驱动人员自定义
- 作用：驱动开始“上岗”，初始化设备，准备提供服务
- 举例：鼠标驱动开始工作，设备鼠标参数，准备接收鼠标数据

**第四步：移出阶段（Remove）**
- 过程：设备拔除或驱动卸载时，调用device_driver结构体中的remove函数
- 作用：清理工作，释放资源
- 举例：U盘拔除时，驱动清理缓存，释放内存

**关键理解点**
1. **这时一套“机制”**
	- 系统提供了框架（总线、匹配、绑定等流程）
	- `具体的match、probe、remove函数需要我们自己实现`
	- 就像系统提供了“相亲平台”，但具体怎么相处要自己定义
2. **统一管理的好处**
	- 设备热插拔：U盘插入自动识别，拔除自动清理
	- 驱动复用：一个驱动可以支持多个同类设备
	- 系统稳定：统一的管理避免了混乱
3. **用户空间的窗口**
	- 通过 /sys 目录，用户可以查看设备状态
	- 甚至可以通过写文件来控制某些设备参数
	- 这就是sysfs文件系统的作用

接下来对总线、驱动、设备进行进一步的了解
- 如何使用代码来实现自己的总线
- 在自己的总线上创建设备和驱动
- 将驱动的某个控制变，导出到用户空间

## 2 总线

总线就像是计算机内部的“高速公路”，连接着CPU和各种设备。不同类型的设备需要遵守相同的通信规则，这就是总线的作用。

**基本概念**：日常解除的设备都是通过总线来通信的
- 触摸屏通过I2C总线连接
- 鼠标、键盘等HID设备，通过USB总线连接
- 这些设备的`共同作用是将向计算机输入信息（文字、字符、控制命令或采集的数据）`
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ad09aac18bbbdd312890445f6051750d.png)

**总线驱动的工作原理：** 管理着两个重要的链表
- **设备链表**：记录连接到总线的所有设备
- **驱动链表**：记录注册到总线的所有驱动程序
- 当添加新设备或驱动时，总线会自动进行匹配，找到合适的驱动来控制设备
**匹配过程**：负责给设备找到合适的驱动
- 新设备接入时，检查是否有合适的驱动
- 新驱动注册时，检查是否有需要它的设备
- 已经配对成功的设备会被忽略
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0bf4ef658746eb1712807809aead4ce0.png)
**bus_type结构体**：在Linux内核中，用`bus_type`结构体来表示总线
```c
struct bus_type{
	//总线名称
	const char *name;
	const struct attribute_group **bus_groups;
	const struct attribute_group **dev_groups;
	const struct attribute_group **drv_groups;

	// 匹配函数
	int (*match)(struct device *dev, struct device_driver *drv);
	int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
	// 探测函数
	int (*probe)(struct device *dev)
	// 移出函数
	int (*remove)(struct device *dev)
	
	// 睡眠
	int (*suspend)(struct device *dev, pm_message_t state)
	// 唤醒
	int (*resume)(struct device *dev)

	const struct dev_pm_ops *pm;
	struct subsys_private *p;
}
```
- name：总线名称，也就是在 `/sys/bus/` 目录下新的目录名称
- `**_group`：驱动、设备以及总线的属性（内部变量、字符串等）
	- 驱动：`/sys/bus/<bus-name>/driver/<driver-name>/`存放了驱动的默认属性
	- 设备：`/sys/bus/<bus-name>/devices/<driver-name>`中
	- 上述属性文件是可读写的，便于用户来获取和设置这些属性
- match：负责判断设备和驱动是否匹配的回调函数
- uevent：总线上设备发生添加、移出或其他操作时会被调用，用于通知驱动做出相应的对策
- probe：匹配成功后调用的回调函数（驱动提供的）
- remove：设备移出时调用的函数
- suspend/resume：电源管理相关函数，suspend->总线睡眠，resume->总线唤醒状态下执行
- pm：电源管理的结构体，存放了一系列跟总线电源管理有关的函数，与device_driver结构体中的pm_ops有关
- p：存放特定的私有数据，其成员klist_devices和klist_drivers记录了挂载在该总线的设备和驱动

**总线注册API**：Linux内核已经写好了大部分总线驱动，正常情况下不会去注册新总线
**1、注册总线**
```c
int bus_register(struct bus_type *bus);
```
- 参数bus：bus_type结构体指针
- 返回值：成功返回0，失败返回负数

**2、注销总线**
```c
void bus_unregister(struct bus_type *bus);
```
- 参数bus：bus_type结构体指针

**系统中的总线**：成功注册总线后，会在`/sys/bus/`目录下看到：
- 各种总线目录（如i2c、spi、platform等）
- 每个总线目录包含两个子目录
	- `devices`：该总线上的所有设备
	- `drivers`：该总线上的所有驱动
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/85582d26dea5e2c1c1a0bb6abbb05398.png)
## 3 设备

在Linux系统中， 一些都是文件，设备也不例外。设备就是我们要控制的硬件，比如LED、传感器、显示屏等。编写驱动程序的最终目的就是让这些设备能够正常工作。

**设备在系统中的存在形式**：Linux系统用文件的方式来管理所有设备
- `/sys/devices`：记录系统中所有设备的根目录
- `/sys/dev`：记录所有设备节点，实际上都是链接文件，指向devices目录下的文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/90f6d470902af1180fac769c8d7da8a6.png)
- 从图中可以看到，系统中的设备都以文件形式存在，通过符号链接的方式相互关联

**device结构体**：在Linux内核中，用`device`结构体来描述物理设备：
```c
// 内核源码/include/linux/device.h
struct device{
	const char *init_name;               //设备名称
	struct device *parent:               //父设备
	struct bus_type *bus;                //所属总线
	struct device_driver *driver;        //对应驱动
	void *platform_data;                 //平台相关数据
	void *driver_data;                   //驱动私有数据
	struct device_node *of_node;         //设备树节点
	dev_t devt;                          //设备号
	struct class *class;                 //设备类别
	void (*release)(struct device *dev); //注销回调函数
	const struct attribute_group **group;//设备属性
	struct device_private *p;            //私有数据
	.......
}
```
- `init_name`：设备名称，总线匹配时会用这个名字进行匹配
- `parent`：父设备指针，设备之间形成树状结构便于管理
- `bus`：设备所依赖的总线，注册时会挂载到对应总线上
- `of_node`：设备树中匹配的节点信息
	- 若内核使能设备树，总线负责将驱动of_match_table和设备树的compatible属性进行比较之后，将匹配的节点保存到该变量
- `platform_data`：平台相关的私有数据，由具体驱动使用
- `driver_data`：驱动层私有数据，可通过`dev_set/get_drvdata`函数访问
- `class`：设备所属类别，比如输入设备、LED设备等，可以在`/sys/class`目录下找到
- `devt`：设备号，用于在`/sys`目录中导出设备
- `release`：设备注销时的回调函数，**必须定义否则会报错**
- `groups`：设备的属性组
- `p`：私有数据，保存子设备链表等内部管理信息，用于添加到`bus/driver/prent`等设备中的链表头等等。

**设备注册API**
**1、注册设备**
```c
int device_register(struct device *dev);
```
- 参数dev：device结构体指针
- 返回值：成功-0，失败-负数

**2、注销设备**
```c
void device_unregister(struct device *dev);
```
- 参数dev：同上

**设备与总线的关系**：当我们使用`device_register`注册设备时：
1. 设备会被挂载到指定的总线上
2. 在 `/sys/bus/总线名/devices/` 目录下会出现该设备文件
3. 总线会尝试为该设备匹配合适的驱动程序，类似于给每个设备都安排了“户口”，知道它属于哪个总线，有什么功能。

> [!warning]+ 注意
> - 设备注销时一定要定义`release`函数，否则系统会提示错误信息！！！
> - 没有定义，移出设备时，会提示“Device ‘xxxx’ does not have a release() function, it is broken and must be fixed”的错误

## 4 驱动

前面介绍了总线和设备，但设备能否正常工作，最终还是取决于驱动。驱动就像是设备的“操作手册”，告诉了内核：我能控制哪些设备，以及如何让这些设备工作起来。

**驱动的作用**：向内核说明两件事情
- 驱动什么设备：通过设备匹配表来说明
- 怎么控制设备：通过初始化和控制函数来实现

**device_driver结构体**：描述驱动程序
```c
// 内核源码/include/linux/device.h
struct device_driver{
	const char *name;                               //驱动名称
	struct bus_type *bus;                           //所属总线
	struct module *owner;                           //驱动拥有者
	const char *mod_name;                           //模块名称
	bool suppress_bind_attrs;                       //是否禁用bind/unbind
	const struct of_device_id *of_match_table;      //设备匹配表
	const struct acpi_device_id *acpi_match_table;  //ACPI匹配表
	int (*probe)(struct device *dev);               //设备初始化函数
	int (*remove)(struct device *dev);              //设备移出函数
	const struct attribute_group **groups;          //驱动属性
	struct driver_private *p;                       //私有属性
}
```
- name：驱动名称，总线匹配时会用这个名字与设备名比较
- bus：驱动所依赖的总线，内核会确保总线先于驱动工作
- owner：驱动拥有者，一般设置为`THIS_MODULE`
- suppress_bind_attrs：bool值，是否通过sysfs导出bind/unbind文件（驱动用于绑定/解绑关联的设备）
- of_match_table：设备匹配表，使用设备树时，与设备树的compatible属性进行比较
- probe：最重要的函数！！！驱动与设备匹配成功后调用，用于初始化设备
- remove：设备移出或系统重启时调用的清理函数
- groups：驱动的属性组

> [!important]+ 🔑 关键理解：probe函数
> - **普通函数从main函数开始执行，而内核驱动程序从probe函数开始执行！**
> - probe函数就是驱动程序的“入口点”

**驱动注册API**
**1、注册驱动**
```c
// 内核源码/include/linux/device.h
int driver_register(struct device_driver *drv);
```
- **参数**：device_driver结构体指针
- **返回值**：成功返回0，失败返回负数

**2、注销驱动**
```c
// 内核源码/include/linux/device.h
void driver_unregister(struct device_driver *drv);
```
- **参数**：同上

**驱动在系统中的位置**
- `/sys/bus/<总线名>/drivers` 目录下
- 总线会自动进行设备与驱动的匹配工作

**驱动工作流程**
1. 注册驱动 → 告诉系统“我来了”
2. 总线匹配 → 系统帮你找到合适的设备
3. 调用probe → 开始初始化设备
4. 正常工作 → 设备可以使用了
5. 调用remove → 设备移出时的清理工作

**记住**：驱动就是设备的大脑，没有合适的驱动，再好的硬件也无法工作

## 5 三者关系（总线、设备和驱动）

到此为止，简单地介绍了总线、设备、驱动的数据结构以及注册/注销接口函数。

下图是总线关联上设备与驱动之后的数据机构关系图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c491c21f38a1ec4030c3269a4d889dcd.png)
- bus_type结构体的成员变量p（struct subsys_private类型）中保存着设备链表klist_devices（struct klist类型）和驱动链表klist_drivers(struct klist类型)
- 设备链表：连接device结构体的成员变量p（struct device_private类型）中的knode_bus(struct klist_node类型)
- 驱动链表：连接device_driver结构体的成员变量p（struct driver_private类型）中的knode_bus（struct klist_node类型）

**注册流程图**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2da2ca3e0adf4bb35b291b04daf5f6f7.png)
- 系统启动后，调用buses_init函数创建 /sys/bus 文件目录（系统开机时已经准备好了）
- 通过总线注册函数bus_register进行总线注册，注册完之后再总线的目录下生成devices和drivers文件夹
- 通过device_register和driver_register函数注册相应的设备和驱动

## 6 attribute属性文件

### 6.1 基本概念

**什么是attribute属性文件**
- 在Linux内核中，当我们注册总线、设备或驱动时，内核会在`/sys`目录下创建相应的子目录
- 这些子目录中的文件就是attribute属性文件
- 它们是内核与用户空间交互的桥梁，让用户可以通过读写这些文件来控制和查看设备信息

**attribute结构体**：内核使用其来描述`/sys`目录下的文件
```c
struct attribute{
	const char *name;
	umode_t mode;
}
```
- name：文件名，决定了在 /sys 目录下显示的文件名称
- mode：文件权限

**attribute_group结构体**：用于管理多个attribute文件
```c
struct attribute_group{
	const char *name;
	umode_t (*is_visible)(struct kobject *,struct attribute *,int);
	struct attribute **attrs;
	struct bin_attribute **bin_attrs;
}
```
- name：属性组名称
- is_visible：函数指针，用于动态控制属性文件的可见性
- attrs：指向attribute数组的指针，包含多个普通属性文件
- bin_attrs：指向二进制attribute数组的指针，用于二进制数据传输

**使用优势**：
- 通过attribute_group可以将多个相关的attribute文件打包在一起，避免一个个单独注册，提高了代码的组织性和效率
- but_type、device、device_driver等重要结构都包含了attribute_group成员，方便统一管理设备的属性文件

### 6.2 设备属性文件

**为什么需要设备属性文件**
- 单片机开发中，想读取某个寄存器的值，需要修改代码并重新编译，而在Linux内核开发中，频繁编译内核既耗时又费力
- 为了解决上面的问题，*Linux提供了设备属性文件接口，可以直接在用户层查询和修改设备信息，避免重新编译内核的麻烦*

**设备属性文件接口**
```c
struct device_attribute{
	struct attribute attr;
	ssize_t (*show)(struct device *dev, struct device_attribute *attr, char *buf);
	ssize_t (*store)(struct device *dev, struct device_attribute *attr, const char *buf, size_t count);
};

#define DEVICE_ATTR(_name, _mode, _show, _store) \
        struct device_attribute dev_attr_##_name = __ATTR(_name, _mode, _show, _store)
extern int device_create_file(struct device *device,
                const struct device_attribute *entry);
extern void device_remove_file(struct device *dev,
                const struct device_attribute *attr);
```

**主要组件说明**：
- DEVICE_ATTR宏：
	- 用于定义device_attribute类型的变量，`##`符号表示拼接，所以变量名会带有`dev_attr`前缀
	- 需要传入四个参数：文件名，文件权限，show回调函数，store回调函数
		- show回调函数对应用户层的`cat`命令（读取文件）
		- store回调函数对应用户层的`echo`命令（写入文件）
	- mode的值，可以参考[6.1 open 函数](../../../../05-💻%20Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/2_使用板卡开发C程序/7_文件操作与系统调用📕.md#6.1%20open%20函数)
- device_create_file函数：创建设备属性文件，在指定设备目录下生成文件
	- device：可以理解在哪个bus的设备目录下创建设备文件
	- device_attribute：用户自定义
- device_remove_file函数：删除设备属性文件，驱动注销时需要调用

### 6.3 驱动属性文件

和设备属性文件相同，主要区别在于`函数参数不同`

**驱动属性文件接口**
```c
struct driver_attribute {
    struct attribute attr;
    ssize_t (*show)(struct device_driver *driver, char *buf);
    ssize_t (*store)(struct device_driver *driver, const char *buf,
            size_t count);
};

#define DRIVER_ATTR_RW(_name) \
    struct driver_attribute driver_attr_##_name = __ATTR_RW(_name)
#define DRIVER_ATTR_RO(_name) \
    struct driver_attribute driver_attr_##_name = __ATTR_RO(_name)
#define DRIVER_ATTR_WO(_name) \
    struct driver_attribute driver_attr_##_name = __ATTR_WO(_name)

extern int __must_check driver_create_file(struct device_driver *driver,
                                    const struct driver_attribute *attr);
extern void driver_remove_file(struct device_driver *driver,
                const struct driver_attribute *attr);
```

**主要组件说明**：
- DRIVER_ATTR_RW/RO/WO宏：定义driver_attribute类型变量，带有driver_attr_前缀
	- RW：可读写
	- RO：只读
	- WO：只写
	- *没有参数来设置show和store回调函数*
- 回调函数命名规则：写驱动代码的时候，需要提供`xxx_store`和`xxx_show`函数，其中`xxx`要与宏定义中的名字（name）一致
- driver_create_file/driver_remove_file：在`/sys/bus/<bus-name>/drivers/<driver-name>/`目录下创建或删除文件

### 6.4 总线属性文件

Linux同样为总线提供了相应的属性文件接口

**总线属性文件接口**
```c
struct bus_attribute {
    struct attribute        attr;
    ssize_t (*show)(struct bus_type *bus, char *buf);
    ssize_t (*store)(struct bus_type *bus, const char *buf, size_t count);
};

#define BUS_ATTR(_name, _mode, _show, _store)       \
        struct bus_attribute bus_attr_##_name = __ATTR(_name, _mode, _show, _store)
extern int __must_check bus_create_file(struct bus_type *,
                    struct bus_attribute *);
extern void bus_remove_file(struct bus_type *, struct bus_attribute *);
```

**主要组件说明**：
- BUS_ATTR宏：定义bus_attribute变量，带有`bus_attr_`前缀
- bus_create_file函数：在`/sys/bus/<bus-name>/` 目录下创建文件
- bus_remove_file函数：移出总线属性文件

## 7 驱动设备模型代码编写和讲解

### 7.1 前言

在设备模型框架下，设备驱动的开发是一件很简单的事情：
1. 分配一个struct device类型的变量，填写必要的信息（成员）后，注册到对应的总线上
2. 创建一个struct device_driver类型的变量，填写必要的信息
3. 在合适的时机（驱动和设备匹配时），调用驱动的probe、release等回调函数

> [!tip]+ 提示
> - 在实际编程中，很少直接使用device和device_driver结构体
> - 一般都是加一层封装，如下一章的platform device等

下面我们来创建一个虚拟的总线xbus，分别挂载了驱动xdrv和设备xdev

### 7.2 编程思路

1. 编写Makefile文件
2. 声明一个总线结构体并创建一个总线xbus，实现match方法，对设备和驱动进行匹配
3. 声明一个设备结构体，挂载到xbus总线上，实现show
4. 声明一个驱动结构体，挂载到xbus总线上，实现probe、remove方法
5. 将总线、设备、驱动导出属性文件到用户空间

### 7.3 Makefile

```makefile
KERNEL_DIR=../../kernel/

ARCH=arm64
CROSS_COMPILE=aarch64-linux-gnu-
export ARCH CROSS_COMPILE

obj-m := xbus.o

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules

.PHONY: clean
clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
```
### 7.4 总线

#### 7.4.1 定义新的总线

```c
// 定义match函数
int xbus_match(struct device *dev, struct device_driver *drv){
    printk("%s-%s\n",__FILE__,__func__);
    // strncmp比较两个字符串，完全相等返回0
    if(!strncmp(dev_name(dev),drv->name,strlen(drv->name))){
        printk("dev & drv match\n");
        return 1;
    }
    return 0;
}

// 定义新的总线
static struct bus_type xbus = {
    .name = "xbus",
    .match = xbus_match,
};
```

#### 7.4.2 导出总线属性文件

```c
// 定义bus_name变量，存放该总线的名称
static char *bus_name = "xbus";

// 提供show回调函数，便于用户通过cat命令，来查询总线名称
ssize_t xbus_test_show(struct bus_type *bus,char *buf){
    // 将格式化数据输出到buf中，并返回buf
    return sprintf(buf,"%s\n",bus_name);
}

// 设置该文件的权限为文件拥有者可读，组内其他成员不可操作
BUS_ATTR(xbus_test,S_IRUSR,xbus_test_show,NULL);
```

#### 7.4.3 注册总线

```c
// 注册总线
static __init int xbus_init(void){
    int ret;
    printk("xbus init\n");

    retr = bus_register(&xbus);
    if(ret)
        printk("xbus register failed\n");
    
    ret = bus_create_file(&xbus,&bus_attr_xbus_test);
    return 0;
}

module_init(xbus_init);

static __exit void xbus_exit(void){
    printk("xbus exit\n");
    bus_remove_file(&xbus, &bus_attr_xbus_test);
    bus_unregister(&xbus);
}

module_exit(xbus_exit);

MODULE_AUTHOR("Dakkk");
MODULE_LICENSE("GPL");
```

#### 7.4.4 完整代码

**完整代码如下**
```c
#include <linux/init.h>
#include <linux/module.h>

#include <linux/device.h>

// 定义match函数
int xbus_match(struct device *dev, struct device_driver *drv){
    printk("%s-%s\n",__FILE__,__func__);
    // strncmp比较两个字符串，完全相等返回0
    if(!strncmp(dev_name(dev),drv->name,strlen(drv->name))){
        printk("dev & drv match\n");
        return 1;
    }
    return 0;
}

// 定义新的总线
static struct bus_type xbus = {
    .name = "xbus",
    .match = xbus_match,
};

EXPORT_SYMBOL(xbus);

// 定义bus_name变量，存放该总线的名称
static char *bus_name = "xbus";

// 提供show回调函数，便于用户通过cat命令，来查询总线名称
ssize_t xbus_test_show(struct bus_type *bus,char *buf){
    // 将格式化数据输出到buf中，并返回buf
    return sprintf(buf,"%s\n",bus_name);
}

// 设置该文件的权限为文件拥有者可读，组内其他成员不可操作
BUS_ATTR(xbus_test,S_IRUSR,xbus_test_show,NULL);

// 注册总线
static __init int xbus_init(void){
    int ret;
    printk("xbus init\n");

    ret = bus_register(&xbus);
    if(ret)
        printk("xbus register failed\n");
    
    ret = bus_create_file(&xbus,&bus_attr_xbus_test);
    return 0;
}

module_init(xbus_init);

static __exit void xbus_exit(void){
    printk("xbus exit\n");
    bus_remove_file(&xbus, &bus_attr_xbus_test);
    bus_unregister(&xbus);
}

module_exit(xbus_exit);

MODULE_AUTHOR("Dakkk");
MODULE_LICENSE("GPL");
```


#### 7.4.5 编译测试

编译后，scp拷贝到板卡上，然后使用insmod命令加载xbus内核模块：
```shell
sudo insmode xbus.ko
```

当我们成功加载编译出来的内核模块时，内核便会出现一种新的总线xbus,如图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cb5a727d4528d0d9774258b2fdff6f17.png)
- devices和drivers目录，目前是空的，并没有什么设备和驱动挂载在总线上
- 红框处便是自定义的总线属性文件，执行命令`sudo cat xbus_test`时，可以看到终端的打印

### 7.5 设备

现在我们来注册一个新的设备，主要是完成两个工作：
- 一个是名字，这时总线相匹配的一句
- 另一个就是总线，该设备挂载在哪个总线上

这里，我们注册一个设备xdev，定义一个变量id，将该变量导出到用户空间，使得用户可以通过sysfs文件系统来修改该变量的值

#### 7.5.1 定义新的设备

```c
// 引用xbus.ko模块导出的xbus
extern struct bus_type xbus;

void xdev_release(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
}

// 定义了一个名为xdev的设备，将其挂载在xbus上
static struct device xdev = {
    .init_name = "xdev",
    .bus = &xbus,
    .release = xdev_release,
};
```

#### 7.5.2 导出设备属性文件

```c
// 导出到用户空间的变量
unsigned long id = 0;

// show回调函数中，直接将id的值通过sprintf函数拷贝到buf中
ssize_t xdev_id_show(struct device *dev,struct device_attribute *attr,char *buf){
    return sprintf(buf,"%ld\n",id);
}

// store回调函数，利用kstrtoul函数
// 该函数有三个参数，其中第二个参数采用二进制，这里传入的是10,即buf中的内容将转换为10进制的数传给id
// 实现了通过sysfs修改驱动的目的
ssize_t xdev_id_store(struct device *dev, struct device_attribute *attr,const char *buf,size_t count){
    int val = 0;
    // 将用户空间传入的字符串buf，转换为数值，然后给id用
    val = kstrtoul(buf,10,&id);
    return count;
}

// 定义xdev_id，设置文件权限仅为文件拥有者可读可写
DEVICE_ATTR(xdev_id,S_IRUSR|S_IWUSR,xdev_id_show,xdev_id_store);
```

**关于kstrtoul()函数的解析**
- 将一个字符串转换为一个无符号长整型的数据，内核中常用的字符串转数字函数
```c
// 内核源码/include/linux/kernel.h
static inline int __must_check kstrtoul(const char *s, unsigned int base, unsigned long *res)
{
    /*
     * We want to shortcut function call, but
     * __builtin_types_compatible_p(unsigned long, unsigned long long) = 0.
     */
    if (sizeof(unsigned long) == sizeof(unsigned long long) &&
        __alignof__(unsigned long) == __alignof__(unsigned long long))
            return kstrtoull(s, base, (unsigned long long *)res);
    else
            return _kstrtoul(s, base, res);
}
```
- s：字符串的起始地址，该字符串必须以空字符(`\0`)结尾
- base：转换基数，指定数字的进制
	- `base = 0`：自动判断字符串类型并按相应进制解析
	    - "0x"或"0X"开头：按十六进制解析（如"0xa"解析为10）
	    - "0"开头：按八进制解析
	    - 其他：按十进制解析
	- `base = 10`：强制按十进制解析
	- `base = 16`：强制按十六进制解析
	- 其他值：按指定进制解析
- res：一个指向被转换成功后的结果的地址
- 返回值：转换成功-0，溢出-ERANGE，解析出错-EINVAL

#### 7.5.3 注册设备

```c
//设备结构体以及属性文件结构体注册
static __init int xdev_init(void){
    int ret;
    printk("xdev init\n");

    ret = device_register(&xdev);
    if(ret)
        printk("Unable to register xdev\n");
    
    device_create_file(&xdev,&dev_attr_xdev_id);
    return 0;
}

module_init(xdev_init);

//设备结构体以及属性文件结构体注销
static __exit void xdev_exit(void){
    printk("xdev exit\n");
    device_remove_file(&xdev, &dev_attr_xdev_id);
    device_unregister(&xdev);
}

module_exit(xdev_exit);

MODULE_AUTHOR("dakkk");
MODULE_LICENSE("GPL");
```

#### 7.5.4 完整代码

**完整的代码如下：**
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/device.h>

// 引用xbus.ko模块导出的xbus
extern struct bus_type xbus;

void xdev_release(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
}

// 定义了一个名为xdev的设备，将其挂载在xbus上
static struct device xdev = {
    .init_name = "xdev",
    .bus = &xbus,
    .release = xdev_release,
};

// 导出到用户空间的变量
unsigned long id = 0;

// show回调函数中，直接将id的值通过sprintf函数拷贝到buf中
ssize_t xdev_id_show(struct device *dev,struct device_attribute *attr,char *buf){
    return sprintf(buf,"%ld\n",id);
}

// store回调函数，利用kstrtoul函数
// 该函数有三个参数，其中第二个参数采用二进制，这里传入的是10,即buf中的内容将转换为10进制的数传给id
// 实现了通过sysfs修改驱动的目的
ssize_t xdev_id_store(struct device *dev, struct device_attribute *attr,const char *buf,size_t count){
    int val = 0;
    // 将用户空间传入的字符串buf，转换为数值，然后给id用
    val = kstrtoul(buf,10,&id);
    return count;
}

// 定义xdev_id，设置文件权限仅为文件拥有者可读可写
DEVICE_ATTR(xdev_id,S_IRUSR|S_IWUSR,xdev_id_show,xdev_id_store);

//设备结构体以及属性文件结构体注册
static __init int xdev_init(void){
    int ret;
    printk("xdev init\n");

    ret = device_register(&xdev);
    if(ret)
        printk("Unable to register xdev\n");
    
    device_create_file(&xdev,&dev_attr_xdev_id);
    return 0;
}

module_init(xdev_init);

//设备结构体以及属性文件结构体注销
static __exit void xdev_exit(void){
    printk("xdev exit\n");
    device_remove_file(&xdev, &dev_attr_xdev_id);
    device_unregister(&xdev);
}

module_exit(xdev_exit);

MODULE_AUTHOR("dakkk");
MODULE_LICENSE("GPL");
```

#### 7.5.5 编译测试

编译后，scp拷贝到板卡上，然后使用insmod命令加载xbus内核模块：
```shell
sudo insmode xdev.ko
```

当我们成功加载编译出来的内核模块时，可以看到在`/sys/bus/xbus/devices/`中多了个设备xdev，它是个链接文件，最终指向了`/sys/devices`中的设备：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/42afc53251adf08977178d13c5c482c1.png)

直接切换到xdev目录下，可以看到，自定义的属性文件xdev_id。通过echo以及cat命令，可以进行修改和查询，如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b6b765a9d65faea10e7ee5e65e2adc0f.png)

### 7.6 驱动

关于驱动部分，由于本章实验没有具体的物理设备，因此，没有涉及到设备初始化、设备的函数接口等内容

#### 7.6.1 定义新的驱动

```c
extern struct bus_type xbus;

int xdrv_probe(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
    return 0;
}

int xdrv_remove(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
    return 0;
}

//定义一个驱动结构体drv，名字需要和设备的名字相同
static struct device_driver xdrv = {
    .name = "xdev",
    //该驱动挂载在已经注册好的总线xbus下
    .bus = &xbus,
    // 驱动和设备匹配完成之后，便会执行驱动的probe函数
    .probe = xdrv_probe,
    // 当注销驱动时，需要关闭物理设备的某些功能等
    .remove = xdrv_remove,
};
```

#### 7.6.2 导出驱动属性文件

```c
char *name = "xdrv";

//保证store和show函数的前缀与驱动属性文件一致
ssize_t drvname_show(struct device_driver *drv,char *buf){
    return sprintf(buf,"%s\n",name);
}

//定义一个drvname属性文件
DRIVER_ATTR_RO(drvname);
```

#### 7.6.3 注册驱动

```c
/注册驱动以及驱动属性文件
static __init int xdrv_init(void){
    int ret;
    printk("xdrv init\n");
    ret = driver_register(&xdrv);
    ret = driver_create_file(&xdrv,&driver_attr_drvname);
    return 0;
}

module_init(xdrv_init);

//注销驱动以及驱动属性文件
static __exit void xdrv_exit(void){
    printk("xdrv exit\n");
    driver_remove_file(&xdrv, &driver_attr_drvname);
    driver_unregister(&xdrv);
}

module_exit(xdrv_exit);

MODULE_AUTHOR("dakkk");
MODULE_LICENSE("GPL");
```

#### 7.6.4 完整代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/device.h>

extern struct bus_type xbus;
int xdrv_probe(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
    return 0;
}

int xdrv_remove(struct device *dev){
    printk("%s-%s\n",__FILE__,__func__);
    return 0;
}

//定义一个驱动结构体drv，名字需要和设备的名字相同
static struct device_driver xdrv = {
    .name = "xdev",
    //该驱动挂载在已经注册好的总线xbus下
    .bus = &xbus,
    // 驱动和设备匹配完成之后，便会执行驱动的probe函数
    .probe = xdrv_probe,
    // 当注销驱动时，需要关闭物理设备的某些功能等
    .remove = xdrv_remove,
};

char *name = "xdrv";

//保证store和show函数的前缀与驱动属性文件一致
ssize_t drvname_show(struct device_driver *drv,char *buf){
    return sprintf(buf,"%s\n",name);
}

//定义一个drvname属性文件
DRIVER_ATTR_RO(drvname);


//注册驱动以及驱动属性文件
static __init int xdrv_init(void){
    int ret;
    printk("xdrv init\n");
    ret = driver_register(&xdrv);
    ret = driver_create_file(&xdrv,&driver_attr_drvname);
    return 0;
}

module_init(xdrv_init);

//注销驱动以及驱动属性文件
static __exit void xdrv_exit(void){
    printk("xdrv exit\n");
    driver_remove_file(&xdrv, &driver_attr_drvname);
    driver_unregister(&xdrv);
}

module_exit(xdrv_exit);

MODULE_AUTHOR("dakkk");
MODULE_LICENSE("GPL");
```

#### 7.6.5 编译测试

编译后，scp拷贝到板卡上，然后使用insmod命令加载xbus内核模块：
```shell
sudo insmode xdrv.ko
```

成功加载驱动后，可以看到/sys/bus/xbus/driver多了个驱动xdev目录，如图所示：在该目录下存在一个我们自定义的属性文件， 使用cat命令读该文件的内容，终端会打印字符串“xdrv”
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8cece827880a3aa64af9d406d8888ec4.png)

使用命令 `dmesg | tail` 来查看模块加载过程的打印信息，当我们加载完设备和驱动之后，总线开始进行匹配，执行match函数， 发现这两个设备的名字是一致的，就将设备和驱动关联到一起，最后会执行驱动的probe函数。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/849788041d2c7dba809aa86da457c6f3.png)
