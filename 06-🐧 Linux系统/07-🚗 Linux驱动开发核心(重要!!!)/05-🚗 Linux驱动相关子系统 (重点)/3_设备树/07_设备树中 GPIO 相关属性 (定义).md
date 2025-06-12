---
文章标题: "[[07_设备树中 GPIO 相关属性 (定义)]]"
文章作者: Dakkk
文章概要: |
  详细介绍设备树GPIO相关属性配置，包括gpio-controller身份标识、gpio-cells参数定义、gpio-ranges映射关系和GPIO引脚描述属性的使用方法和原理。
tags:
  - 设备树
  - GPIO
  - 嵌入式Linux
  - BSP开发
  - 引脚配置
  - pinctrl
  - rockchip
  - 硬件抽象
相关文章:
  - "[[06_U-boot的修改与编译]]"
  - "[[08_Linux内核的编译]]"
  - "[[1_快速开始]]"
  - "[[1_参考内容/设备树的引进和体验]]"
  - "[[13_Debian根文件系统构建]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/1_Linux之设备树详解（训练营）/06_设备树中 GPIO 相关属性 (定义).md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-11 14:31:59
---

## 1 设备树节点概览

在学习GPIO属性之前，我们先来看一个实际的例子。下面展示的是rk3568.dtsi设备树中的GPIO使用案例，这个例子包含了GPIO控制器的所有核心属性。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/abd3143ca00b797754d37db07022231f.png)

> [!info]+ GPIO控制器的四大核心属性
> 
> - **gpio-controller**：身份标识，表明这是一个GPIO控制器
> - `#gpio-cells`：定义GPIO描述符的参数个数
> - **gpio-ranges**：建立GPIO编号与物理引脚的映射关系
> - **GPIO引脚描述属性**：具体的GPIO配置信息

简单来说，GPIO（General Purpose Input/Output）就是通用输入输出引脚，是微控制器与外部世界交互的"桥梁"。设备树中的GPIO属性就是告诉系统如何正确使用这些引脚。

## 2 gpio-controller属性（身份标识-不改）

**gpio-controller属性**：用于标识一个设备节点作为GPIO控制器的身份标识符。

### 2.1 基本概念

换句话说，gpio-controller就像一个"身份证"，告诉系统"我是一个GPIO控制器，可以管理GPIO引脚"。

- **GPIO控制器**：负责管理和控制GPIO引脚的硬件模块或驱动程序
- **属性位置**：通常作为设备节点的一个属性出现，位于设备节点的属性列表中

### 2.2 工作原理

当一个设备节点被标识为GPIO控制器时，它会发生以下几个关键动作：

1. **定义GPIO引脚组**：控制器会定义一组GPIO引脚
2. **提供控制功能**：提供相关的GPIO控制和配置功能
3. **供其他设备使用**：其他设备节点可以使用该GPIO控制器来控制和管理其GPIO引脚

> [!note]+ BSP开发说明 
> 在BSP（Board Support Package）开发中，gpio-controller属性通常不需要修改，因为它是由芯片厂商在芯片级设备树中预先定义的。

### 2.3 系统识别机制

通过使用gpio-controller属性，设备树可以明确标识出GPIO控制器设备节点，使系统能够：
- 正确识别GPIO控制器
- 管理GPIO引脚的配置和控制
- 建立GPIO引脚与设备的关联关系

## 3 gpio-cells（参数个数定义-不改）

**#gpio-cells属性**：用于指定GPIO引脚描述符的编码方式，定义了每个GPIO引脚需要多少个参数来完整描述。

### 3.1 基本作用

- **参数个数**：属性值是一个整数，表示用于编码GPIO引脚描述符的单元数
- **标准值**：通常这个值为2，表示每个GPIO引脚用2个参数描述

### 3.2 实际应用示例

在第一小节的示例中，由于#gpio-cells属性被设置为2，所以每个引脚描述属性中会有两个整数参数：

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eda1dfbadbb106cfe08052160f540440.png)

> [!example]+ 参数含义解释 图中的`RK_PC0`和`GPIO_ACTIVE_LOW`都属于宏定义：
> 
> - **RK_PC0**：表示具体的GPIO引脚编号
> - **GPIO_ACTIVE_LOW**：表示GPIO的电平状态

### 3.3 系统解析机制

通过使用`#gpio-cells`属性，设备树可以：
- 指定GPIO引脚描述符的编码方式
- 使系统能够正确识别和解析GPIO引脚的配置
- 确保GPIO控制的准确性

> [!warning]+ 开发注意事项 
> `#gpio-cells`属性的值必须与实际GPIO引脚描述符的参数个数保持一致，否则系统无法正确解析GPIO配置。

## 4 gpio-ranges（引脚映射关系-不改）

**gpio-ranges属性**：用于描述GPIO范围映射的属性，主要用于描述具有大量GPIO引脚的GPIO控制器，以简化GPIO引脚的编码和访问。

### 4.1 格式规范和解释

gpio-ranges属性的标准格式如下：
```dts
gpio-ranges = <phandle gpio-controller-base-pin gpio-base gpio-count>;
```
- **phandle**：指向另一个设备树节点的引用（如`&pinctrl`，代表pinctrl节点）
- **gpio-controller-base-pin**：GPIO控制器管理的GPIO号，在这个范围内的偏移值对应pinctrl节点定义的引脚号偏移
- **gpio-base**：对应的物理引脚起始编号
- **gpio-count**：该范围内引脚的数量

### 4.3 映射机制原理

在设备树中，GPIO控制器面临一个重要问题：

> [!question]+ 为什么需要gpio-ranges？ 
> GPIO控制器的每个引脚都有一个本地编号，用于在控制器内部进行引脚寻址。然而，这些本地编号并不一定与外部引脚的物理编号或其他系统中使用的编号一致。

为了解决这个编号不一致的问题，gpio-ranges属性**将本地编号映射到实际的引脚编号**

### 4.4 映射信息组成

gpio-ranges属性是一个包含一系列整数值的列表，每个整数值按照特定的顺序提供以下信息：
1. **外部引脚编号的起始值**
2. **GPIO控制器内部本地编号的起始值**
3. **引脚范围的大小**（引脚数量）

### 4.5 实际案例分析

在第一小节的示例中，gpio-ranges属性的值为`<&pinctrl 0 0 32>`：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eae02ac01c91cd46dda6c83a9d134da4.png)

让我们分解这个配置：
- **&pinctrl**：引用了名为pinctrl的引脚控制器节点
- **第一个0**：外部引脚从0开始
- **第二个0**：控制器本地编号从0开始
- **32**：共映射了32个引脚

> [!success]+ 映射效果 
> 这样，gpio-ranges属性将GPIO控制器的本地编号直接映射到外部引脚编号，使得GPIO引脚的编码和访问更加简洁和直观。

## 5 gpio 引脚描述属性（具体配置信息）

GPIO引脚描述属性是设备树中最直接的GPIO配置信息，它告诉系统具体**要使用哪个GPIO引脚**以及**如何配置它**。

### 5.1 参数个数关系

第一小节的设备树中关于GPIO引脚描述属性的相关内容如下所示：

![image|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eda1dfbadbb106cfe08052160f540440.png)

> [!info]+ 参数个数决定因素 
> GPIO引脚描述属性个数由#gpio-cells所决定
> 
> 因为gpio0节点中的#gpio-cells属性设置为2，所以上面设备树GPIO引脚描述属性个数也为2。

### 5.2 引脚编号宏定义

**RK_PC0**：定义在内核源码目录下的`include/dt-bindings/pinctrl/rockchip.h`头文件中，这个文件定义了`RK引脚名`和`GPIO编号`的宏定义
![image|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/24bcb2db11e27f67760bdc45d4e1666d.png)

> [!tip]+ 开发建议 
> 使用宏定义而不是直接的数字，可以提高代码的可读性和可维护性。当需要修改引脚配置时，只需要更改宏定义即可。

### 5.3 电平状态宏定义

**GPIO_ACTIVE_LOW**：定义在源码目录下的`include/dt-bindings/gpio/gpio.h`中，用于表示GPIO的电平状态
![image|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/91f28130d4c11309347bbef1f87ab539.png)

### 5.4 电平状态说明

- **GPIO_ACTIVE_LOW**：表示设置为低电平有效
- **GPIO_ACTIVE_HIGH**：表示设置为高电平有效

> [!warning]+ 重要提醒 
> 这里的电平设置只是对设备的描述，具体的GPIO电平控制还需要与驱动程序相匹配。设备树中的配置需要与驱动程序的实现保持一致。

### 5.5 实际应用场景

在实际开发中，GPIO引脚描述属性的应用场景包括：
1. **LED控制**：定义LED连接的GPIO引脚和有效电平
2. **按键检测**：配置按键连接的GPIO引脚和触发电平
3. **设备使能**：设置设备使能引脚的GPIO配置
4. **状态指示**：配置状态指示灯的GPIO引脚

> [!abstract]+ 本章总结 
> GPIO引脚描述属性是设备树GPIO配置的核心部分，它通过两个参数（引脚编号和电平状态）完整描述了GPIO的使用方式。理解这些属性的含义和配置方法，是进行嵌入式Linux开发的基础技能。

> [!success]+ 全文总结 
> 通过本文的学习，我们掌握了设备树GPIO系统的四个核心概念：
> 
> 1. **gpio-controller**：身份标识，标明GPIO控制器
> 2. `#gpio-cells`：定义参数个数，通常为2
> 3. **gpio-ranges**：建立映射关系，简化引脚访问
> 4. **GPIO引脚描述属性**：具体配置信息，包含引脚编号和电平状态
> 
> 这四个属性相互配合，构成了完整的GPIO描述体系。在实际开发中，理解这些概念有助于正确配置GPIO功能，避免硬件控制错误。
