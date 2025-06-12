
📢本篇章将介绍**DTB**（Device Tree Blob）数据解析的完整过程，换句话说，就是内核如何读懂设备树文件并创建对应的设备。

> [!info]+ 什么是DTB？ 
> DTB是设备树的二进制格式，类似于一个"硬件说明书"，告诉内核系统中有哪些硬件设备以及它们的配置信息。

## 1 内核启动阶段获得DTB位置指针

以**ARM64**架构为例，内核启动时需要先找到DTB文件的位置。这个过程就像是内核启动时第一件事就是找到"硬件说明书"放在哪里。

- 文件位置：`内核源码/arch/arm64/kernel/head.S +78`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bccccfa2d85f3e195c20c82ba1d17dac.png)

### 1.1 启动方式说明

在开启**UEFI**支持时，`add x13, x18, #0x16`这个代码实际上是为了满足**EFI**格式的"**MZ**"头。简单来说：
1. **UEFI启动**：如果使用UEFI来启动kernel，会识别出MZ头并走UEFI启动的流程
2. **传统启动**：如果是普通的启动过程（如使用uboot的booti进行引导），那么第一条指令就是一条dummy指令，第二条就跳转到stext运行

> [!tip]+ 关键寄存器 
> **x0寄存器**保存的是DTB里blob块的物理地址，这个地址由uboot设置，相当于uboot告诉内核"设备树文件在这个地址"

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9a31a62841d06d5db5e96cbb9b2d42db.png)

### 1.2 ARM64寄存器约定

ARM64 Linux对启动时寄存器有严格的规定：
```C
x0 = physical address of device tree blob (dtb) in system RAM.
x1 = 0 (reserved for future use)
x2 = 0 (reserved for future use)
x3 = 0 (reserved for future use)
```
- 换句话说，**只有x0寄存器有用，它保存着DTB的物理地址，其他寄存器都预留给未来使用**

### 1.3 地址保存过程

内核会将**DTB的物理地址**保存在**x21寄存器**中，然后调用`__inval_cache_range`函数来清理指定区域的缓存。

> [!note]+ 缓存清理机制 
> 如果指定内存区域跨越了cacheline，那么对两边跨越的地址使用clean + invalidate，对中间区域可以直接invalidate，这样可以加快清理速度。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4cac2515fdcfa469a625b04f0337b25d.png)

通过`str_l x21, fdt_pointer, x5`这句关键代码，将**DTB的地址**保存在**fdt_pointer指针**里，以便后续内核可以找到DTB并解析设备树文件

> [!abstract]+ 整体启动流程 
> 整个Linux启动过程可以理解为：找到设备树 → 解析设备树 → 创建设备 → 加载驱动


## 2 DTB解析过程 - 从二进制到设备节点

**start_kernel**阶段是DTB解析的核心阶段，这里会处理DTB文件生成**device_node结构体**以及它们之间的组织关系。简单来说，就是把二进制的设备树文件转换成内核能理解的数据结构。

### 2.1 解析调用流程

![image|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0a5f7bcb03e1f125899ecc0b7ad4827a.png)

整体的调用逻辑就像一个流水线，每个函数负责解析设备树的不同部分。

### 2.2 设备树解析示例

下面以一个实际的设备树为例，看看解析前后的变化：
```C
/ {
    leds {
        act {
            gpios = <&gpio 42 GPIO_ACTIVE_HIGH>;
        };
        pwr {
            label = "PWR";
            gpios = <&expgpio 2 GPIO_ACTIVE_LOW>;
            default-state = "keep";
            linux,default-trigger = "default-on";
        };
    };
    wifi_pwrseq: wifi-pwrseq {
        compatible = "mmc-pwrseq-simple";
        reset-gpios = <&expgpio 1 GPIO_ACTIVE_LOW>;
    };
};
```

> [!example]+ 设备树节点说明
> 
> - **根节点(/)**：整个设备树的起点
> - **leds节点**：描述LED灯的配置
> - **wifi_pwrseq节点**：描述WiFi电源序列的配置
> - **compatible属性**：告诉内核这个设备需要什么驱动

### 2.3 解析后的数据结构关系

该设备树经过**populate_node()** 解析处理后，各节点的关系如下：

```C
- /节点（根节点）：
  - parent：NULL（没有父节点）
  - sibling：NULL（没有兄弟节点）
  - child：wifi_pwrseq（子节点是wifi_pwrseq）

- leds节点：
  - parent：/（父节点是根节点）
  - sibling：NULL（没有兄弟节点）
  - child：pwr（子节点是pwr）

- act节点：
  - parent：leds（父节点是leds）
  - sibling：NULL（没有兄弟节点）
  - child：NULL（没有子节点）

- pwr节点：
  - parent：leds（父节点是leds）
  - sibling：act（兄弟节点是act）
  - child：NULL（没有子节点）

- wifi_pwrseq节点：
  - parent：/（父节点是根节点）
  - sibling：leds（兄弟节点是leds）
  - child：NULL（没有子节点）
```

> [!tip]+ 节点关系理解 
> 可以把这个结构想象成一个家族树：根节点是祖先，其他节点通过parent（父）、sibling（兄弟）、child（子）的关系连接起来。

## 3 device_node与device绑定 - 从设备节点到平台设备

解析完设备树节点后，内核需要为这些节点创建实际的设备对象，这个过程就是**device_node**到**platform_device**的转换。

### 3.1 设备创建规则

> [!warning]+ 重要规则 
> 内核会为设备树root节点下所有带**'compatible'属性**的节点都分配并注册一个**platform_device**。

换句话说：
1. **必须有compatible属性**：这个属性告诉内核该设备需要什么驱动
2. **符合特定条件的节点**：如果某节点的compatible符合特定matches条件，则会为该节点下所有带compatible属性的子节点也创建platform_device

### 3.2 绑定流程说明

整体调用流程按照以下步骤进行：
1. **扫描设备树节点**：遍历所有解析好的device_node
2. **检查compatible属性**：判断节点是否需要创建设备
3. **创建platform_device**：为符合条件的节点创建设备对象
4. **注册到设备模型**：将设备加入到统一设备模型中

> [!success]+ 最终结果 
> 至此，为所有设备树中符合条件的node都创建了platform_device结构体，node下描述的硬件资源也解析到了platform_device中，并通过dev成员将该node描述的设备加入了平台设备模型。

## 4 总结

整个DTB解析过程可以概括为三个主要阶段：
1. **定位阶段**：内核启动时通过寄存器获得DTB的物理地址
2. **解析阶段**：将二进制DTB文件解析成device_node数据结构
3. **绑定阶段**：为device_node创建对应的platform_device，完成设备注册

> [!abstract]+ 核心理念 
> 设备树就像是硬件的"身份证"，记录了系统中所有硬件的信息。内核通过解析这个"身份证"，知道了系统中有哪些硬件，然后为每个硬件创建对应的软件对象，最后加载相应的驱动程序来控制这些硬件。

这个过程确保了内核能够正确识别和管理系统中的所有硬件设备，为后续的驱动加载和设备操作奠定了基础。
