---
文章标题: "[[06_设备树中 GPIO 相关属性 (定义)]]"
文章作者: Dakkk
文章概要: |
  讲解Linux设备树中GPIO相关属性的定义和使用，包括gpio-controller标识、#gpio-cells编码、gpio-ranges映射和引脚描述属性的配置方法
tags:
  - 设备树
  - GPIO
  - Linux内核
  - RK3568
  - pinctrl
  - 引脚配置
  - 嵌入式开发
  - BSP
相关文章:
  - "[[07_设备树中 Pinctrl的描述]]"
  - "[[08_dtb 文件格式讲解]]"
  - "[[../../../../01-❇️ 参考资料/0_个人实用技巧/0_linux常用文件夹]]"
  - "[[../../../../02-💻 开发环境/3_Lubancat-RK3568/2_镜像构建与部署/10_设备树的简介]]"
  - "[[../../../../../05-🔧 51&32单片机开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/01-📝 训练营笔记/_Linux之设备树详解/1_笔记/06_设备树中 GPIO 相关属性 (定义).md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-04-29 22:37:30
修改时间: 2025-05-28 00:19:51
---

## 1 设备树节点

我们以rk3568.dtsi设备树中的gpio使用为例子：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/abd3143ca00b797754d37db07022231f.png)

## 2 gpio-controller（身份标识，BSP不用改）

gpio-controller 属性用于标识一个设备节点作为 GPIO 控制器
- GPIO 控制器负责管理和控制 GPIO 引脚的硬件模块或驱动程序
- gpio-controller 属性通常作为设备节点的一个属性出现，位于设备节点的属性列表中

当一个设备节点被标识为 GPIO 控制器时，它通常会定义一组 GPIO 引脚，并提供相关的GPIO 控制和配置功能。其他设备节点可以使用该 GPIO 控制器来控制和管理其 GPIO 引脚。

通过使用 gpio-controller 属性，设备树可以明确标识出 GPIO 控制器设备节点，使系统可以正确识别和管理 GPIO 引脚的配置和控制。
## 3 gpio-cells（不改）

`#gpio-cells` 属性用于指定 GPIO 引脚描述符的编码方式。GPIO 引脚描述符是用于标识和配置 GPIO 引脚的一组值，例如引脚编号、引脚属性等。

`#gpio-cells` 属性的属性值是一个整数，表示用于编码 GPIO 引脚描述符的单元数。通常，这个值为 2

在第一小节的示例中有 1 个 gpio 引脚描述属性,由于#gpio-cells 属性被设置为了 2，所以每个引脚描述属性中会有两个整数，具体内容如下所示：
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eda1dfbadbb106cfe08052160f540440.png)

RK_PC0、GPIO_ACTIVE_LOW 都属于恒定义，会在下面的小节进行讲解。

通过使用#gpio-cells 属性，设备树可以指定 GPIO 引脚描述符的编码方式，使系统能够正确识别和解析 GPIO 引脚的配置和控制
## 4 gpio-ranges（不改）

`gpio-ranges` 属性的格式一般是：
```dts
gpio-ranges = <phandle gpio-controller-base-pin gpio-base gpio-count>;
```
- **phandle**：指向另一个设备树节点的引用（这里是 `&pinctrl`，代表 pinctrl 节点）。
- **gpio-controller-base-pin**：`gpio-controller` 管理的 GPIO 号，在这个范围内的偏移值`对应 pinctrl 节点定义的引脚号偏移`。
- **gpio-base**：`对应的物理引脚起始编号`。
- **gpio-count**：该范围内引脚的`数量`。

gpio-ranges 属性是设备树中一个用于描述 GPIO 范围映射的属性。它通常用于**描述具有大量 GPIO 引脚的 GPIO 控制器，以简化 GPIO 引脚的编码和访问**。

在设备树中，GPIO 控制器的每个引脚都有一个本地编号，用于在控制器内部进行引脚寻址。然而，这些本地编号并不一定与外部引脚的物理编号或其他系统中使用的编号一致。为了解决这个问题，可以**使用 gpio-ranges 属性将本地编号映射到实际的引脚编号**。

gpio-ranges属性是一个包含一系列整数值的列表，每个整数值对应于设备树中的一个GPIO控制器。列表中的每个整数值按照特定的顺序提供以下信息：
1. 外部引脚编号的起始值。
2. GPIO 控制器内部本地编号的起始值。
3. 引脚范围的大小（引脚数量）


在第一小节的示例中 gpio-ranges 属性的值为<&pinctrl 0 0 32>，其中<&pinctrl>表示引用了名为 pinctrl 的引脚控制器节点，0 0 32 表示外部引脚从 0 开始，控制器本地编号从 0 开始，共映射了 32 个引脚
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eae02ac01c91cd46dda6c83a9d134da4.png)

这样，gpio-ranges 属性将 GPIO 控制器的本地编号直接映射到外部引脚编号，使得 GPIO 引脚的编码和访问更加简洁和直观。
## 5 gpio 引脚描述属性

第一小节的设备树中关于 gpio 引脚描述属性相关内容如下所示：

gpio 引脚描述属性个数由#gpio-cells 所决定，因为 gpio0 节点中的#gpio-cells 属性设置为了2，所以上面设备树 gpio 引脚描述属性个数也为 2

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eda1dfbadbb106cfe08052160f540440.png)

其中 RK_PC0 定义在内核源码目录下的“include/dt-bindings/pinctrl/rockchip.h”头文件中，定义了 RK 引脚名和 gpio 编号的宏定义，如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/24bcb2db11e27f67760bdc45d4e1666d.png)

GPIO_ACTIVE_LOW 定义在源码目录下的“include/dt-bindings/gpio/gpio.h”中，表示设置为低电平，同理 GPIO_ACTIVE_HIGH 就表示将这个 GPIO 设置为高电平，但这里只是对设备的描述，具体的设置还是要跟驱动相匹配

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/91f28130d4c11309347bbef1f87ab539.png)
