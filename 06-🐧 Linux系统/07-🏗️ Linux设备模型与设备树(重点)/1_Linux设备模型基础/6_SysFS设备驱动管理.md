# SysFS设备驱动管理

在学习了kobject、kset和属性文件后，现在让我们从更高的视角来理解Linux设备管理的全貌：**SysFS设备驱动管理系统**。

简单来说，**SysFS**就是Linux内核向用户空间展示设备信息的"窗口"，它将复杂的内核设备模型以简单直观的文件系统形式呈现出来

> [!note]+ 核心理念 
> Linux系统中一切皆文件。SysFS虚拟文件系统将设备、驱动、总线等内核对象以文件和目录的形式呈现，让用户能够通过文件操作来管理和配置设备。

## 1 SysFS文件系统概述

### 1.1 什么是SysFS

**SysFS文件系统**是Linux内核提供的一种虚拟文件系统，用于向用户空间提供内核中设备、驱动程序和其他内核对象的信息。它以层次结构的方式组织数据，将这些数据表示为文件和目录。

**主要特点**：
- **统一接口**：为浏览和管理内核对象提供统一的方式
- **层次组织**：以目录树的形式反映对象间的关系
- **实时信息**：动态反映内核状态的变化
- **双向交互**：既可以读取信息，也可以配置参数

> [!tip]+ 访问位置 
> SysFS挂载在`/sys`目录下，用户可以通过查看和修改`/sys`目录下的文件来获取和配置内核对象的信息。

### 1.2 SysFS与/dev的区别

理解SysFS，需要先明确它与传统设备文件的区别：

**`/dev`目录**：
- 包含真实的设备文件
- 通常由`udev`在运行时创建
- 用于应用程序的实际设备操作
- 使用`open`、`write`、`ioctl`等函数访问

**`/sys`目录**：
- 由内核在运行时动态导出
- 展示设备、驱动和总线的层次关系
- 主要用于设备信息查询和配置
- 通过文件读写进行交互

> [!example]+ 实际对比 
> `/dev/sda1`是真实的磁盘设备文件，用于数据读写；
> 而`/sys/block/sda/queue/scheduler`是SysFS中的配置文件，用于查看和设置磁盘调度器。

## 2 SysFS目录层次结构

### 2.1 核心目录组织

进入Linux系统的`/sys`目录，可以看到几个重要的子目录：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7f549a5a56f78f58cd34db25989e4199.png)

**设备模型相关的三个重要目录**：
- `/sys/bus` - 总线管理
- `/sys/class` - 设备分类管理
- `/sys/devices` - 设备物理拓扑

### 2.2 /sys/devices：设备物理拓扑

**`/sys/devices`目录**包含了系统中所有设备的物理拓扑结构，反映了设备的真实硬件连接关系
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2c1df2137e9064960577386845908287.png)

**目录特点**：
- 每个设备都有一个对应的子目录
- 目录层次反映设备的物理连接关系
- 包含设备的属性、状态和相关信息
- 通过符号链接与其他目录关联

**典型路径示例**：
```txt
/sys/devices/platform/soc/1c14000.usb/usb1/
/sys/devices/platform/soc/1c68000.ethernet/
/sys/devices/virtual/net/lo/
```

### 2.3 /sys/bus：总线管理

**`/sys/bus`目录**包含了各种总线类型的管理信息，每个总线都有自己的子目录
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5d19fd4668ed472e46b8a6180db5913b.png)


**总线目录结构**：
```txt
/sys/bus/
├── platform/        # Platform总线
├── i2c/             # I2C总线
├── spi/             # SPI总线
├── usb/             # USB总线
└── pci/             # PCI总线
```

**每个总线目录包含**：
- `devices/` - 该总线上的所有设备
- `drivers/` - 该总线的所有驱动
- `uevent` - 总线事件文件
- `drivers_probe` - 驱动探测控制
- `drivers_autoprobe` - 自动探测开关

**I2C总线示例**：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5640871db7157effc71ed2d5cbdbb99a.png)
在I2C总线下可以看到连接的各种I2C设备。

### 2.4 /sys/class：设备功能分类

**`/sys/class`目录**将设备按照功能进行分类，提供了一种逻辑上的设备组织方式
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/281ae7fbfbfad5052db17c6431e7294b.png)


**常见设备类别**：
```txt
/sys/class/
├── gpio/            # GPIO控制
├── net/             # 网络接口
├── block/           # 块设备
├── input/           # 输入设备
├── hwmon/           # 硬件监控
├── thermal/         # 温度管理
└── power_supply/    # 电源管理
```

**设备分类的优势**：使用class进行设备归类带来了显著的好处

**1. 逻辑组织**
- 将同类型设备集中管理
- 建立清晰的设备分类体系
- 便于用户查找和使用

**2. 统一接口**
- 同类设备使用相同的接口和属性
- 简化应用程序的开发
- 提供一致的用户体验

**3. 简化设备发现**
- 应用程序可以在类别目录中查找特定类型的设备
- 无需遍历整个设备树
- 提高设备访问效率

**4. 扩展性和可移植性**
- 新设备可以轻松加入现有类别
- 支持跨平台的设备管理
- 降低系统移植的复杂度

> [!example]+ 实际应用对比 
> **使用class的便捷方式**：
> 
> ```bash
> echo 1 > /sys/class/gpio/gpio157/value
> ```
> 
> **不使用class的复杂路径**：
> 
> ```bash
> echo 1 > /sys/devices/platform/fe770000.gpio/gpiochip4/gpio/gpio157/value
> ```

## 3 SysFS构建过程分析

### 3.1 设备模型与SysFS的映射关系

**SysFS的目录结构直接映射了内核的设备模型结构**。让我们通过下图来理解这种映射关系：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/023c769ce5b34783d30830dc591f806a.png)

**核心映射关系**：
- 每个**kobject**对应SysFS中的一个目录
- 每个**attribute**对应目录中的一个文件
- **父子关系**体现为目录的层次结构
- **kset**提供对象的集合管理

### 3.2 设备模型的基础结构

在实际的设备驱动开发中，我们通常不会直接使用kobject，而是使用更高级的结构：

**字符设备结构（设备模型1.0）**：
```c
struct cdev {
    struct kobject kobj;        // 内嵌的kobject
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
};
```

**platform设备结构（设备模型2.0）**：
```c
struct platform_device {
    const char *name;
    int id;
    bool id_auto;
    struct device dev;          // 内嵌device结构
    u32 num_resources;
    struct resource *resource;
    // ...
};
```

**device结构的核心字段**：
```c
struct device {
    struct device *parent;      // 父设备
    struct kobject kobj;        // 对应的kobject
    const char *init_name;      // 设备初始化名称
    const struct device_type *type;  // 设备类型
    struct bus_type *bus;       // 所属总线
    struct device_driver *driver;    // 对应驱动
    struct class *class;        // 所属类
    const struct attribute_group **groups; // 属性组
    // ...
};
```

> [!tip]+ 继承关系 
> 可以把总线、设备、驱动看作是kobject的"派生类"，它们通过继承或扩展kobject来实现与设备模型的集成。

## 4 SysFS注册接口分析

### 4.1 SysFS构建流程

让我们通过下图理解SysFS文件系统的构建过程：

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/91bbf761e4943cc930f4fc4597944a76.png)

**初始化流程**：
1. **内核启动**时初始化SysFS文件系统
2. **创建根目录**：bus、class、devices等
3. **注册各子系统**：总线、设备、驱动依次注册
4. **建立关联关系**：通过符号链接建立对象间的关系

### 4.2 设备注册过程：device_register

让我们详细分析设备注册过程：

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1494199a9cd370a0670e9dd90162b982.png)

**device_register的主要工作**：
1. **在/sys/devices下创建设备目录**
    - 目录名通常是设备名称
    - 在目录中创建设备属性文件
2. **创建subsystem链接**
    - 指向所属的bus或class
    - 表明设备的归属关系
3. **在bus/class中创建反向链接**
    - 从bus或class指向设备
    - 便于从不同角度查看设备

**注册后的目录和链接关系**：

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a2279ef5433da93cd48c18cb905d8437.png)

这种设计体现了数据结构的管理思想：
- **目录** = 结构体
- **文件** = 结构体成员
- **链接** = 指针关系

### 4.3 驱动注册过程：driver_register

驱动的注册相对简单：

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7100cc36fdf4460a169eaead88b16409.png)

**driver_register的主要工作**：
1. **在/sys/bus/xxx-bus/drivers/下创建驱动目录**
2. **初始化驱动属性文件**
3. **创建到module的链接**（如果是模块驱动）
4. **module反向链接到驱动**

**注册后的目录结构**：

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dd593f00e68c7b180d424f512478eae2.png)

### 4.4 总线注册过程：bus_register

总线注册是最复杂的过程：
![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca854e3d8a9396e06635491c9aab8eea.png)

**bus_register的主要工作**：
1. **在/sys/bus/下创建总线目录**
2. **创建devices和drivers子目录**
3. **创建探测相关的属性文件**
    - `drivers_probe` - 手动触发探测
    - `drivers_autoprobe` - 自动探测开关
4. **创建uevent事件文件**
5. **添加总线特有的属性**

**注册完成后的目录结构**：
![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc8cfdf9da8718ac39cfd8103a12b82d.png)

### 4.5 类注册过程：class_register

类的注册最为简单：
![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0202a61a144efac51d2a3018ca2f7d92.png)

**class_register的主要工作**：
- 在`/sys/class/`下创建类目录
- 为设备注册时创建链接做准备
- 设备的实际链接工作在`device_register`中完成

## 5 实际应用案例分析

### 5.1 GPIO子系统在SysFS中的体现

以GPIO为例，看看一个完整的设备子系统在SysFS中是如何组织的：
```txt
/sys/class/gpio/
├── export              # 导出GPIO引脚
├── unexport            # 注销GPIO引脚
├── gpiochip0/          # GPIO控制器0
│   ├── base            # 起始引脚号
│   ├── label           # 控制器标签
│   └── ngpio           # 引脚数量
└── gpio18/             # 已导出的GPIO18
    ├── direction       # 方向控制(in/out)
    ├── value           # 值控制(0/1)
    ├── active_low      # 极性控制
    └── edge            # 中断边沿控制
```

**使用示例**：
```bash
# 导出GPIO18
echo 18 > /sys/class/gpio/export

# 设置为输出模式
echo out > /sys/class/gpio/gpio18/direction

# 设置高电平
echo 1 > /sys/class/gpio/gpio18/value

# 读取当前值
cat /sys/class/gpio/gpio18/value
```

> [!example]+ 实际应用 
> 这种设计让用户可以在不编写驱动程序的情况下，直接通过shell脚本控制GPIO，大大简化了硬件控制的复杂度

### 5.2 网络接口在SysFS中的管理

网络接口的SysFS结构：
```txt
/sys/class/net/
├── eth0/               # 以太网接口
│   ├── address         # MAC地址
│   ├── mtu             # 最大传输单元
│   ├── operstate       # 操作状态
│   ├── speed           # 连接速度
│   ├── statistics/     # 统计信息
│   │   ├── rx_bytes    # 接收字节数
│   │   ├── tx_bytes    # 发送字节数
│   │   ├── rx_packets  # 接收包数
│   │   └── tx_packets  # 发送包数
│   └── carrier         # 载波状态
└── lo/                 # 回环接口
    └── ...
```

**查看网络统计信息**：
```bash
# 查看接收的字节数
cat /sys/class/net/eth0/statistics/rx_bytes

# 查看网卡状态
cat /sys/class/net/eth0/operstate

# 查看MAC地址
cat /sys/class/net/eth0/address
```

### 5.3 块设备在SysFS中的表示

块设备的SysFS结构更加复杂，包含了大量的性能调优参数：
```txt
/sys/block/sda/
├── queue/              # 队列参数
│   ├── scheduler       # I/O调度器
│   ├── read_ahead_kb   # 预读大小
│   ├── max_sectors_kb  # 最大扇区数
│   └── nr_requests     # 请求队列大小
├── sda1/               # 分区1
├── sda2/               # 分区2
├── size                # 设备大小
├── removable           # 是否可移除
└── ro                  # 是否只读
```

**性能调优示例**：
```bash
# 查看当前I/O调度器
cat /sys/block/sda/queue/scheduler

# 修改调度器为deadline
echo deadline > /sys/block/sda/queue/scheduler

# 调整预读大小
echo 256 > /sys/block/sda/queue/read_ahead_kb
```

## 6 SysFS的高级特性

### 6.1 热插拔事件支持

SysFS与uevent机制紧密结合，支持设备的热插拔：
```bash
# 监控设备热插拔事件
udevadm monitor --udev

# 手动触发uevent
echo add > /sys/devices/platform/my_device/uevent
```

### 6.2 设备电源管理

SysFS暴露了丰富的电源管理接口：
```txt
/sys/devices/platform/my_device/power/
├── control             # 电源控制模式
├── runtime_status      # 运行时状态
├── runtime_usage       # 使用计数
└── wakeup              # 唤醒能力
```

### 6.3 设备拓扑关系

通过符号链接，SysFS展示了复杂的设备拓扑关系：
```bash
# 查看设备的驱动
ls -l /sys/devices/platform/my_device/driver

# 查看驱动管理的设备
ls -l /sys/bus/platform/drivers/my_driver/
```

## 7 调试和问题排查

### 7.1 常用调试命令

```bash
# 查看完整的设备树
tree /sys/devices/ | head -50

# 查看特定总线的设备
ls /sys/bus/platform/devices/

# 查看设备的uevent信息
cat /sys/devices/platform/my_device/uevent

# 查找特定设备
find /sys -name "*my_device*" -type d

# 监控文件系统变化
inotifywait -m -r /sys/class/gpio/
```

### 7.2 常见问题及解决

**问题1：设备目录未创建**
- 检查设备注册是否成功
- 确认kobject初始化是否正确
- 查看内核日志中的错误信息

**问题2：属性文件无法访问**
- 检查文件权限设置
- 确认sysfs_ops是否正确实现
- 验证show/store函数的返回值

**问题3：符号链接断开**
- 检查引用计数是否正确管理
- 确认对象销毁顺序是否合理
- 验证父子关系设置是否正确

### 7.3 性能监控

```bash
# 查看SysFS的内存使用
cat /proc/slabinfo | grep -E "(kobject|kset)"

# 监控设备数量变化
watch -n 1 'find /sys/devices -name "uevent" | wc -l'

# 查看总线上的设备数量
ls /sys/bus/*/devices/ | wc -l
```

> [!abstract]+ 章节总结 
> 通过本章学习，我们全面理解了SysFS设备驱动管理系统：
> 
> - **SysFS的整体架构**和在Linux系统中的重要地位
> - **三大核心目录**（devices、bus、class）的组织原理
> - **设备注册流程**及其在SysFS中的体现
> - **实际应用案例**展示了SysFS的强大功能
> - **调试技巧**帮助解决实际开发中的问题

SysFS作为Linux设备模型的"可视化界面"，为我们理解和管理设备提供了强大的工具。下一章我们将深入学习总线的详细实现和实践应用！