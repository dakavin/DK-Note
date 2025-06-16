

📢本篇将介绍通过设备树 `device_node` 转换成 `platform_device`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/df6f1ace3ad8f391a8af8908f9273834.png)


## DTB转换规则

---

`device` 部分是用 `platform_device` 结构体来描述硬件资源的， 所以内核最终会将内核认识的 `device_node` 树转换 `platform_ device`， 但是并不是所有的 `device_node` 都会被转换成 `platform_ device`， 只有满足要求的才会转换成 `platform_ device`，转换成 `platform_device` 的节点可以在`/sys/bus/platform/devices` 下查看， 那 `device_node` 节点要满足什么要求才会被转换成 `platform_device` 呢?

- 规则 1： 首先遍历根节点下包含 `compatible` 属性的子节点， 对于每个子节点， 创建一个对应的 `platform_device`。
    
- 规则 2： 遍历包含 `compatible` 属性为 “`simple-bus`”、 “`simple-mfd`” 或 “`isa`” 的节点以及它们的子节点。 如果子节点包含 `compatible` 属性值则会创建一个对应的 `platform_device`。
    
- 规则 3：检查节点的 `compatible` 属性是否包含 “`arm`” 或 “`primecell`”。 如果是， 则不将该节点转换为 `platform_device`， 而是将其识别为 `AMBA` 设备。
    

## 转换源码分析

---

`device_node` 转换成 `platform_device` 的函数调用流程图如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0788d6e18eb7fe93a4cc39910c7b3010.png)


首先进入到内核源码目录下的“`drivers/of/platform.c`” 文件中， 找到第 `555` 行， 具体内容如下所示：

```C
arch_initcall_sync(of_platform_default_populate_init);
```

`arch_initcall_sync` 是 `Linux` 内核中的一个函数， 用于在内核初始化过程中执行架构相关的初始化函数。 它属于内核的初始化调用机制， 用于确保在系统启动过程中适时地调用特定架构的初始化函数。

在 `Linux` 内核的初始化过程中， 各个子系统和架构会注册自己的初始化函数。 这些初始化函数负责完成特定子系统或架构相关的初始化工作， 例如初始化硬件设备、 注册中断处理程序、设置内存映射等。 而 `arch_initcall_sync` 函数则用于调用与当前架构相关的初始化函数。

当内核启动时 ， 调用 `rest_init()` 函数来启动初始化过程 。 在初始化过程中 ，`arch_initcall_sync` 函数会被调用， 以确保所有与当前架构相关的初始化函数按照正确的顺序执行。 这样可以保证在启动过程中， 特定架构相关的初始化工作得到正确地完成。

而 `of_platform_default_populate_init` 函数的作用是在内核初始化过程中自动解析设备树，并根据设备树中的设备节点创建对应的 `platform_device` 结构。 它会遍历设备树中的设备节点，并为每个设备节点创建一个对应的 `platform_device` 结构， 然后将其注册到内核中， 使得设备驱动程序能够识别和操作这些设备。 该函数的具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/27a713492c28843166e6a0c76d6cf382.png)
