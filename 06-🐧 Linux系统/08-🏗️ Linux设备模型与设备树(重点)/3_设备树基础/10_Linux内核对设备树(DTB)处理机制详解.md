
## 1 目标与处理流程

### 1.1 设备树处理的三大目标

Linux内核处理设备树主要为了实现三个目标：
1. **平台识别（Platform Identification）**：识别当前运行的是什么硬件平台
2. **运行时配置（Runtime Configuration）**：获取内存大小、启动参数等系统配置
3. **设备填充（Device Population）**：创建设备对象供驱动程序使用

### 1.2 核心处理流程

整个设备树处理可以概括为一条转换链：
```txt
DTB → device_node → platform_device → 驱动匹配
```

> [!abstract]+ 处理流程总结
> 
> 1. **DTB传递**：bootloader将DTB地址传给内核
> 2. **DTB解析**：内核将二进制DTB转换为device_node树形结构
> 3. **设备创建**：将device_node转换为platform_device
> 4. **驱动匹配**：platform_device与platform_driver进行匹配

## 2 DTB的传递与获取

> [!warning]+ 注意
> 除了在汇编阶段传递DTB时，存在ARm和ARM64的区别
> 后续C相关函数的处理是一样的

### 2.1 Bootloader传参机制

bootloader给内核传递参数有两种方法：
- **ATAGS方式**（传统方法）
- **DTB方式**（设备树方法）

当bootloader（如uboot）启动Linux内核时，需要告诉内核一些关键信息。这个过程通过设置CPU寄存器来完成

> [!note]+ ARM架构传参约定 
> ARM架构下，bootloader通过以下寄存器传递参数：
> 
> - **r0**：设置为0
> - **r1**：machine ID（使用设备树时未使用）
> - **r2**：**DTB的物理地址**（重点关注）

对于ARM64架构，约定略有不同：
```c
x0 = physical address of device tree blob (dtb) in system RAM
x1 = 0 (reserved for future use)
x2 = 0 (reserved for future use)
x3 = 0 (reserved for future use)
```

简单来说，**x0寄存器保存着DTB的物理地址**，这是bootloader告诉内核"设备树文件在内存的什么位置"。

### 2.2 内核获取DTB地址

内核启动的第一步就是保存DTB地址，这个过程发生在汇编代码中。

**ARM架构的处理**（head.S文件）：
```assembly
// 主要处理步骤
a. __lookup_processor_type : 读取CPU ID，找到对应的处理器信息
b. __vet_atags            : 判断是否存在可用的ATAGS或DTB
c. __create_page_tables   : 创建页表（虚拟地址映射）
d. __enable_mmu           : 使能MMU
e. __mmap_switched        : 切换到虚拟地址
f. 保存参数               : 把r2的值保存到__atags_pointer变量
g. start_kernel           : 跳转到C语言环境
```

**关键的参数传递**：使用的是汇编指令
```c
// head.S/head-common.S中的处理：
__machine_arch_type = r1;  // 把bootloader传来的r1值赋给变量machine_arch_type
__atags_pointer = r2;      // 把bootloader传来的r2值赋给变量atags_pointer(即dtb首地址)
```

> [!tip]+ 关键点理解 
> 虽然涉及汇编，但核心就是：内核把bootloader传来的DTB地址保存起来，供后续C代码使用。

**ARM64架构的处理**：内容有点多，初学不建议看，大概过一眼
![image|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/be68578ac6089b0f494579ddf8945035.png)

ARM64更加简洁，直接将x0寄存器的值（DTB地址）保存到**fdt_pointer**变量中

## 3 平台识别 - 选择正确的machine_desc

### 3.1 为什么需要平台识别

Linux内核需要支持多种硬件平台，每种平台都有自己的初始化代码。内核通过**machine_desc**结构体来描述不同的硬件平台。

> [!question]+ 什么是machine_desc？ 
> machine_desc是描述特定硬件平台的结构体，包含了该平台的初始化函数、中断处理等信息。内核需要找到与当前硬件匹配的machine_desc

### 3.2 匹配机制

匹配过程就像"相亲配对"：

**设备树侧**：
- 根节点的**compatible属性**列出一系列字符串
- 表示兼容的单板名称，从"最匹配"到"次匹配"
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a972db3b41a2dc5987c9742797f4b520.png)

**内核侧**：
- 每个machine_desc都有**dt_compat成员**
- 指向支持的单板名称数组
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/807c5f2c8af3bd3d4e0e1890631f5708.png)
### 3.3 匹配算法

> [!example]+ 匹配过程示例 
> 假设设备树compatible = "smdk2440", "smdk2410", "smdk24xx"
> 
> 1. 先用"smdk2440"与所有machine_desc比较，匹配成功得分为1
> 2. 如果没找到，用"smdk2410"比较，匹配成功得分为2
> 3. 继续用"smdk24xx"比较，匹配成功得分为3
> 4. **得分越低越匹配**（1分最好）

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a3c0e1cb6a6dc4799fe4cd8fe3b3905f.png)

### 3.4 代码调用流程

```c
start_kernel()                                        // 内核入口
 └── setup_arch()                                    // 架构相关初始化
     └── setup_machine_fdt(__atags_pointer)          // 通过DTB选择machine
         ├── early_init_dt_verify()                  // 验证DTB有效性
         │   └── initial_boot_params = params        // 设置全局DTB指针
         └── of_flat_dt_match_machine()              // 匹配最佳machine_desc
             └── 遍历所有machine_desc，找最佳匹配
```

具体的调用过程，可以参考：[3.2 详细的匹配过程](参考内容/内核对设备树的处理(重要).md#3.2%20详细的匹配过程)

## 4 运行时配置信息的提取

找到正确的硬件平台后，内核需要从设备树中提取系统配置信息。

### 4.1 需要提取的关键信息

内核主要关注三个特殊节点：
1. **/chosen节点**：包含启动参数
2. **根节点**：包含地址/大小的表示方式
3. **/memory节点**：包含内存信息

### 4.2 具体处理过程

```c
start_kernel                                    // init/main.c
 └── setup_arch(&command_line);                // arch/arm/kernel/setup.c
     └── mdesc = setup_machine_fdt(__atags_pointer);  // arch/arm/kernel/devtree.c
         └── early_init_dt_scan_nodes();       // drivers/of/ftd.c
             ├── of_scan_flat_dt(early_init_dt_scan_chosen, boot_command_line); // 扫描/chosen节点
             ├── of_scan_flat_dt(early_init_dt_scan_root, NULL); // 扫描根节点
             └── of_scan_flat_dt(early_init_dt_scan_memory, NULL); // 扫描/memory节点
                 
```

具体的调用过程，可以参考：[4.1 函数调用过程](参考内容/内核对设备树的处理(重要).md#4.1%20函数调用过程)

**各节点的处理结果**：

> [!info]+ 提取的信息
> 
> - **/chosen节点**：提取bootargs属性 → 存入**boot_command_line**全局变量
> - **根节点**：提取#address-cells和#size-cells → 存入**dt_root_addr_cells**和**dt_root_size_cells**
> - **/memory节点**：提取reg属性（基地址+大小） → 调用**memblock_add()**注册内存

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/66646b2b8ba9ec02d6b155066504c485.png)

## 5 DTB转换为device_node

### 5.1 为什么要转换

> [!note]+ 转换的必要性
> 
> - **转换前**：DTB是扁平的二进制数据，不便于操作
> - **转换后**：device_node是树形结构，便于遍历和查找

这个过程称为"unflatten"（展开），就像把压缩包解压成文件夹结构。

### 5.2 转换过程

转换分为两遍扫描：
```c
start_kernel                               // init/main.c
 └── setup_arch(&command_line);           // arch/arm/kernel/setup.c
     ├── arm_memblock_init(mdesc);         // arch/arm/kernel/setup.c
     │   ├── early_init_fdt_reserve_self();
     │   │   └── /* Reserve the dtb region */
     │   │       // 把DTB所占区域保留下来, 即调用: memblock_reserve
     │   │       early_init_dt_reserve_memory_arch(__pa(initial_boot_params),
     │   │                     fdt_totalsize(initial_boot_params), 0);
     │   └── early_init_fdt_scan_reserved_mem();  // 根据dtb中的memreserve信息, 调用memblock_reserve, 告诉内核这块内存不要动！
     │   ----------------------下面是分析的重点--------------------------------
     └── unflatten_device_tree();          // arch/arm/kernel/setup.c
			 └── __unflatten_device_tree()
			     ├── // 第一遍：计算需要的内存大小
			     │   size = unflatten_dt_nodes(blob, NULL, dad, NULL)
			     ├── // 分配内存
			     │   mem = dt_alloc(size + 4, __alignof__(struct device_node))
			     └── // 第二遍：填充数据结构
			         unflatten_dt_nodes(blob, mem, dad, mynodes)
```

具体的调用过程，可以参考：[5.2 函数调用的过程](参考内容/内核对设备树的处理(重要).md#5.2%20函数调用的过程)

> [!tip]+ 两遍扫描的智慧 
> 第一遍只计算大小不分配内存，第二遍才真正创建数据结构。这样可以一次性分配所需内存，避免内存碎片。

### 5.3 核心数据结构

**device_node结构体**（表示一个设备节点）：
```c
struct device_node {
    const char *name;              // 节点名称
    const char *type;              // 设备类型
    phandle phandle;               // 节点句柄
    const char *full_name;         // 完整路径名
    struct property *properties;   // 属性链表
    struct device_node *parent;    // 父节点
    struct device_node *child;     // 子节点
    struct device_node *sibling;   // 兄弟节点
    // ... 其他成员
};
```

**property结构体**（表示一个属性）：
```c
struct property {
    char *name;                    // 属性名
    int length;                    // 属性值长度
    void *value;                   // 属性值
    struct property *next;         // 下一个属性
};
```

### 5.4 转换示例

以下面的设备树为例：

```dts
/ {
    leds {
        act {
            gpios = <&gpio 42 GPIO_ACTIVE_HIGH>;
        };
        pwr {
            label = "PWR";
            gpios = <&expgpio 2 GPIO_ACTIVE_LOW>;
        };
    };
    wifi_pwrseq: wifi-pwrseq {
        compatible = "mmc-pwrseq-simple";
        reset-gpios = <&expgpio 1 GPIO_ACTIVE_LOW>;
    };
};
```

转换后的结构关系：

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c08daa0da18efea0b02ca9f03409d034.png)

> [!success]+ 转换结果 
> 所有device_node通过parent、child、sibling指针连接成一棵树，根节点保存在全局变量**of_root**中。

## 6 device_node转换为platform_device

### 6.1 转换的意义

> [!question]+ 为什么还要转换？ 
> device_node只是设备的静态描述，而platform_device是Linux设备模型中的"活"设备，可以与驱动程序绑定。

### 6.2 转换规则

**并非所有device_node都会转换为platform_device**，必须满足以下条件：

> [!warning]+ 转换条件
> 
> 1. **基本条件**：节点必须包含**compatible属性**
> 2. **位置条件**（满足其一）：
>     - 是根节点的直接子节点
>     - 父节点的compatible包含特殊值：
>         - "simple-bus"
>         - "simple-mfd"
>         - "isa"
>         - "arm,amba-bus"

![image.png|500|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8d994ae7d56958c1d6b12627b6811d32.png)

### 6.3 特殊总线的处理

**I2C/SPI总线的特殊规则**：

- **控制器节点**：转换为platform_device
- **设备节点**：不转换，由总线驱动处理

> [!example]+ 转换示例
> 
> ```d
> / {
>     mytest {
>         compatible = "mytest","simple-bus";  // 转换为platform_device
>         mytest@0 {
>             compatible = "mytest-0";         // 也转换（父节点是simple-bus）
>         };
>     };
>     
>     i2c {
>         compatible = "samsung,i2c";          // 转换（I2C控制器）
>         at24c02 {
>             compatible = "at24c02";          // 不转换（由I2C驱动处理）
>         };
>     };
> };
> ```

### 6.4 转换源码流程分析

转换过程通过递归遍历实现：
```c
of_platform_default_populate_init()                   // 初始化函数
 └── of_platform_populate()                          // 遍历设备树
     └── for_each_child_of_node(root, child) {      // 遍历根节点的子节点,root = of_root
            of_platform_bus_create()                 // 创建platform_device
            └── of_platform_device_create_pdata()    // 实际创建设备
                └── of_device_alloc()                // 分配并初始化
         }
```

具体的调用过程，可以参考：[6.5 函数调用过程](参考内容/内核对设备树的处理(重要).md#6.5%20函数调用过程)


### 6.5 资源的转换

device_node中的属性会转换为platform_device的资源：
- **reg属性** → **IORESOURCE_MEM**（内存资源）
- **interrupts属性** → **IORESOURCE_IRQ**（中断资源）

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/534185ef81ca5dfa62a0f763698190a2.png)

> [!info]+ 资源转换的意义 
> 驱动程序可以通过标准的platform_get_resource()函数获取这些资源，而不需要自己解析设备树。

## 7 总线设备的特殊处理

具体的调用过程，可以参考：[6.6 总线设备的特殊处理](参考内容/内核对设备树的处理(重要).md#6.6%20总线设备的特殊处理)

### 7.1 I2C总线处理流程

I2C控制器的platform_driver在probe函数中会创建I2C设备：

```c
i2c_add_numbered_adapter()                           // 注册I2C适配器
 └── i2c_register_adapter()
     └── of_i2c_register_devices()                   // 扫描设备树中的I2C设备
         └── for_each_available_child_of_node() {    // 遍历子节点
                of_i2c_register_device()             // 创建i2c_client
                └── i2c_new_device()                 // 注册到I2C总线
             }
```

> [!note]+ I2C设备创建过程
> 
> 1. I2C控制器节点 → platform_device
> 2. platform_driver的probe函数执行
> 3. 扫描I2C节点的子节点
> 4. 每个子节点 → i2c_client

### 7.2 SPI总线处理流程

SPI的处理流程类似：

```c
spi_register_controller()                            // 注册SPI控制器
 └── of_register_spi_devices()                      // 扫描设备树中的SPI设备
     └── for_each_available_child_of_node() {       // 遍历子节点
            of_register_spi_device()                 // 创建spi_device
            ├── spi_alloc_device()                   // 分配设备
            ├── of_spi_parse_dt()                    // 解析属性
            └── spi_add_device()                     // 注册到SPI总线
         }
```

## 8 platform_device与platform_driver的匹配

### 8.1 匹配时机

匹配发生在两个时机：
1. **注册platform_driver时**：遍历已有的platform_device
2. **注册platform_device时**：遍历已有的platform_driver

### 8.2 匹配规则

> [!important]+ 匹配优先级（从高到低）
> 
> 1. **driver_override匹配**：设备指定了特定驱动
> 2. **设备树匹配**：compatible属性匹配
> 3. **ID表匹配**：设备名与驱动ID表匹配
> 4. **名称匹配**：设备名与驱动名相同

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e76b99c87d492e0d9d4bc154049c9d59.png)
1. **driver_override匹配**：`platform_dev.driver_override` vs `platform_driver.drv->name`
2. **设备树匹配**：`platform_dev.dev.of_node.compatible` vs `platform_driver.drv->of_match_table`的某一项
3. **ID表匹配**：`platform_dev.name` vs `platform_driver.id_table`的某一项
4. **名称匹配**：`platform_dev.name` vs `platform_driver.drv->name`

**任何一个匹配成功，即认为匹配成功！**

### 8.3 匹配过程详解

**注册platform_driver时的匹配**：
```c
platform_driver_register()
 └── driver_register()
     └── bus_add_driver()
         └── driver_attach()
             └── bus_for_each_dev()          // 遍历所有设备
                 └── __driver_attach()
                     ├── driver_match_device() // 调用platform_match
                     └── driver_probe_device() // 匹配成功则probe
```

> [!tip]+ 理解probe函数 
> probe函数是驱动的"初始化函数"，只有设备和驱动匹配成功后才会调用。在probe中完成硬件初始化、资源申请等工作。

### 8.4 设备树匹配的实现

设备树匹配是最常用的方式：

```c
// 驱动中定义匹配表
static const struct of_device_id my_of_match[] = {
    { .compatible = "vendor,my-device", },
    { .compatible = "vendor,my-device-v2", },
    { }
};

static struct platform_driver my_driver = {
    .driver = {
        .name = "my-driver",
        .of_match_table = my_of_match,  // 设置匹配表
    },
    .probe = my_probe,
    .remove = my_remove,
};
```

## 9 实践示例：编写设备树匹配的驱动

### 9.1 设备树节点

```dts
myled {
    compatible = "mycompany,myled";
    reg = <0x40000000 0x1000>;
    interrupts = <0 29 4>;
    clock-frequency = <100000000>;
    status = "okay";
};
```

### 9.2 驱动代码框架

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>

static int myled_probe(struct platform_device *pdev)
{
    struct resource *res;
    int irq;
    u32 freq;
    
    // 获取内存资源
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res) {
        dev_err(&pdev->dev, "Failed to get memory resource\n");
        return -ENODEV;
    }
    
    // 获取中断资源
    irq = platform_get_irq(pdev, 0);
    if (irq < 0) {
        dev_err(&pdev->dev, "Failed to get IRQ\n");
        return irq;
    }
    
    // 读取自定义属性
    if (of_property_read_u32(pdev->dev.of_node, 
                             "clock-frequency", &freq)) {
        dev_warn(&pdev->dev, "clock-frequency not specified\n");
        freq = 100000000; // 默认值
    }
    
    dev_info(&pdev->dev, "Probed: mem=%pR, irq=%d, freq=%u\n",
             res, irq, freq);
    
    // TODO: 初始化硬件
    
    return 0;
}

static int myled_remove(struct platform_device *pdev)
{
    // TODO: 清理资源
    return 0;
}

static const struct of_device_id myled_of_match[] = {
    { .compatible = "mycompany,myled", },
    { }
};

// ？？
MODULE_DEVICE_TABLE(of, myled_of_match);

static struct platform_driver myled_driver = {
    .driver = {
        .name = "myled",
        .of_match_table = myled_of_match,
    },
    .probe = myled_probe,
    .remove = myled_remove,
};

module_platform_driver(myled_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("My LED Driver");
```

> [!note]+ 驱动要点
> 
> 1. 定义of_device_id匹配表
> 2. 在probe函数中获取资源
> 3. 使用设备树API读取自定义属性
> 4. 正确处理错误情况

## 10 常见问题与注意事项

### 10.1 设备没有被创建为platform_device

> [!warning]+ 检查要点
> 
> 1. 节点是否有compatible属性？
> 2. 节点位置是否正确？（根节点子节点或simple-bus下）
> 3. 父节点是否是I2C/SPI控制器？（这种情况不会创建platform_device）

### 10.2 驱动和设备不匹配

> [!bug]+ 排查步骤
> 
> 1. 检查compatible字符串是否完全一致（包括大小写）
> 2. 确认of_device_id数组以空项结尾
> 3. 检查MODULE_DEVICE_TABLE是否正确声明

### 10.3 无法获取资源

> [!failure]+ 可能原因
> 
> 1. 设备树中reg或interrupts属性格式错误
> 2. 父节点的#address-cells/#size-cells设置不正确
> 3. 资源索引超出范围

## 11 总结

> [!abstract]+ 设备树处理全流程总结
> 
> 1. **启动阶段**：bootloader将DTB地址通过寄存器传给内核
> 2. **平台识别**：通过compatible属性匹配正确的machine_desc
> 3. **配置提取**：从特殊节点提取内存、启动参数等信息
> 4. **结构转换**：将扁平的DTB展开为device_node树形结构
> 5. **设备创建**：符合条件的device_node转换为platform_device
> 6. **驱动匹配**：通过compatible等方式匹配驱动和设备
> 7. **设备初始化**：匹配成功后调用probe函数完成初始化

理解这个流程对于Linux驱动开发至关重要。设备树机制让Linux内核能够支持多种硬件平台，而不需要修改内核代码。通过掌握设备树的处理机制，你就能编写出正确的设备树文件和对应的驱动程序。