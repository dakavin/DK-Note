  
📢设备树中的节点被转换为`platform_device`后，设备树中的`reg`属性、`interrupts`属性也会被转换为“`resource`”。这时，你可以使用这个函数取出这些资源。

## Platform device 结构体

platform_device 结构体是用于描述平台设备的数据结构。 它包含了平台设备的各种属性和信息， 用于在内核中表示和管理平台设备。 该结构体定义在内核的“/include/linux/platform_device.h” 文件中， 具体内容如下所示：

```C++
struct platform_device {
        const char *name; // 设备的名称， 用于唯一标识设备
        int id; // 设备的 ID， 可以用于区分同一种设备的不同实例
        bool id_auto; // 表示设备的 ID 是否自动生成
        struct device dev; // 表示平台设备对应的 struct device 结构体， 用于设备的基本管理和操作
        u32 num_resources; // 设备资源的数量
        struct resource *resource; // 指向设备资源的指针
        const struct platform_device_id *id_entry; // 指向设备的 ID 表项的指针， 用于匹配设备和驱动
        char *driver_override; // 强制设备与指定驱动匹配的驱动名称
        /* MFD cell pointer */
        struct mfd_cell *mfd_cell; // 指向多功能设备（ MFD） 单元的指针， 用于多功能设备的描述
        /* arch specific additions */
        struct pdev_archdata archdata; // 用于存储特定于架构的设备数据
};
```

下面对于几个重要的参数和结构体进行讲解

1. const char *name： 设备的名称， 用于唯一标识设备。 必须提供一个唯一的名称， 以便内核能够正确识别和管理该设备。
    
2. int id： 设备的 ID， 可以用于区分同一种设备的不同实例。 这个参数是可选的， 如果不需要使用 ID 进行区分， 可以将其设置为-1，
    
3. struct device dev： 表示平台设备对应的 struct device 结构体， 用于设备的基本管理和操作。必须为该参数提供一个有效的 struct device 对象， 该结构体的 release 方法必须要实现， 否则在编译的时候会报错。
    
4. u32 num_resources： 设备资源的数量。 如果设备具有资源（ 如内存区域、 中断等） ， 则需要提供资源的数量。
    
5. **struct resource *resource： 指向设备资源的指针。 如果设备具有资源， 需要提供一个指向资源数组的指针**， 会在下个小节对该结构体进行详细的讲解。
    

## resource 结构体

struct resource 结构体用于描述系统中的**设备资源（硬件资源）**， 包括内存区域、 I/O 端口、 中断等，该结构体定义在内核的“ /include/linux/ioport.h” 文件中， 具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9ee2e55ba102fd3825b9b96f8c8ad78a.png)


```C++
struct resource {
        resource_size_t start; /* 资源的起始地址 */
        resource_size_t end; /* 资源的结束地址 */
        const char *name; /* 资源的名称 */
        unsigned long flags; /* 资源的标志位 */
        unsigned long desc; /* 资源的描述信息 */
        struct resource *parent; /* 指向父资源的指针 */
        struct resource *sibling; /* 指向同级兄弟资源的指针 */
        struct resource *child; /* 指向子资源的指针 */
        /* 以下宏定义用于保留未使用的字段 */
        ANDROID_KABI_RESERVE(1);
        ANDROID_KABI_RESERVE(2);
        ANDROID_KABI_RESERVE(3);
        ANDROID_KABI_RESERVE(4);
};
```

其中最重要的是前四个参数， 每个参数的具体介绍如下所示：

1. resource_size_t start： 资源的起始地址。 它表示资源的起始位置或者起始寄存器的地址。
    
2. resource_size_t end： 资源的结束地址。 它表示资源的结束位置或者结束寄存器的地址。
    
3. const char *name： 资源的名称。 它是一个字符串， 用于标识和描述资源。
    
4. unsigned long flags： 资源的标志位。 它包含了一些特定的标志， 用于表示资源的属性或者特征。 例如， 可以用标志位来指示资源的可用性、 共享性、 缓存属性等。 flags 参数的具体取值和含义可以根据系统和驱动的需求进行定义和解释， 但通常情况下， 它用于表示资源的属性、 特征或配置选项。 下面是一些常见的标志位及其可能的含义
    

中断控制器：

MEM (芯片寄存器)

IRQ（中断资源）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/74d30a4281f29713b36c646b46feadc3.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c3f2787d1ce94c2cf2343a93213a7f13.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/72363cd9c3f7dbab736ca67b09a01974.png)


## platform_get_resource() 获取

platform_get_resource()函数用于获取设备的资源信息。 它的声明位于<linux/platform_device.h>头文件中， 与平台设备（platform_device）相关。

```C++
struct resource *platform_get_resource(struct platform_device *pdev,unsigned int type, unsigned int num);
```

  
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2a02b3eb36a53d29f02ff07b00ba557a.png)


platform_get_resource()函数用于从平台设备的资源数组中获取指定类型和索引的资源。 在平台设备的资源数组中， 每个元素都是一个 struct resource 结构体， 描述了一个资源的信息，如起始地址、 结束地址、 中断号等。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/45b5ba4184fc89c3416d143fb7011fbd.png)


在上述示例中， 首先通过 platform_get_resource()函数获取平台设备的第一个内存资源（索引为 0） 。 如果获取资源失败（返回 NULL） ， 则可以根据实际情况进行错误处理。 如果获取资源成功， 则可以使用返回的资源指针来访问资源的信息， 如起始地址和结束地址。

通过 platform_get_resource()函数， 可以方便地在驱动程序中获取平台设备的资源信息，并根据这些信息进行后续的操作和配置。

  

## 案例

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0b8767e1642f66137573f6e7d8692960.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0395b57c3ac9b9a5cf95e89412a5c2b4.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0324f291f1918b05377a98365cc4c500.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4a57f09d70f173bfb37e1819882d984e.png)


## platform_get_resource实现

  

`platform_get_resource` 函数源码如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b7fe9cce1a48214f97d3e835b3c264bc.png)


函数分析：

```C++
struct resource *r = &dev->resource[i];
```

这行代码使得不管你是想获取哪一份资源都从第一份资源开始搜索。

```C++
if (type == resource_type(r) && num-- == 0)
```

这行代码首先通过 `type == resource_type(r)` 判断当前这份资源的类型是否匹配，如果匹配则再通过 `num-- == 0` 判断是否是你要的，如果不匹配重新提取下一份资源而不会执行 `num-- == 0` 这一句代码。

通过以上两步就能定位到你要找的资源了，接着把资源返回即可。如果都不匹配就返回`NULL`。

## 直接访问 platform_device 结构体的资源数组

```c
struct resource *res_mem = &pdev->resource[0];
```

在 这种 方法 中， 直接 访 问 platform_device 结 构体 的资 源数 组来 获 取硬 件资 源。pdev->resource 是一个资源数组， 其中存储了设备的硬件资源信息。 通过访问数组的不同索引，可以获取到特定的资源。

  

## 中断资源

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b779e00f4a5541b5da1037eecaa827ac.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f1454ffd88f3a7246494223dc96cc57d.png)
