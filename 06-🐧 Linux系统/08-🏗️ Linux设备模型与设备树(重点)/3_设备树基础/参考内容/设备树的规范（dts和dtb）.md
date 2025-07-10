---
文章标题: "[[设备树的规范（dts和dtb）]]"
文章作者: Dakkk
文章概要: |
  详细讲解了设备树源文件DTS的语法规则、节点属性格式、引用方法，以及二进制DTB文件的分段布局结构，涵盖从源码到编译后格式的完整技术规范。
tags:
  - 设备树
  - DTS
  - DTB
  - 硬件描述
  - 嵌入式系统
  - Linux内核
  - 节点引用
  - 二进制格式
相关文章:
  - "[[../02_设备树（device Tree）的由来]]"
  - "[[../03_图解Kernel Device Tree(设备树)的使用]]"
  - "[[10_设备树的简介]]"
  - "[[../../../07-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树]]"
  - "[[../04_设备树中CPU描述 (不需要改)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/2_Linux设备树详解（韦东山）/2_设备树的规范（dts和dtb）.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-11 08:46:30
---

## 1 DTS格式

### 1.1 基本语法规则

DTS（Device Tree Source）是一种描述硬件信息的文本格式

**设备树节点格式**
```dts
[label:] node-name[@unit-address] {
    [properties definitions]
    [child nodes]
};
```
- 节点后的@用于区分同一级别下同名的节点，也就是说同一级别下，`name@address`只能有一个
- 每个大括号后面都必须有`;`

**属性（Property）格式**
```dts
// 有值的属性
[label:] property-name = value;

// 无值的属性（相当于一个标志）
[label:] property-name;
```

**属性值的3中类型**：属性的取值只能是以下3种
- arrays of cells：1个或多个32位数据（64位数据用2个32位表示）
	- 如：`<1 0x3 0x123>`
- string：字符串，双引号括起来的
- bytestring：1个或多个字节，必须用两位16进制数来表示，空格可以省略
	- 如：`[00 11 22]`

**实际例子**
```dts
// 1. 32位数组：cell就是一个32位的数据
interrupts = <17 0xc>;

// 2. 64位数据用2个cell表示
clock-frequency = <0x00000001 0x00000000>;

// 3. 以null结尾的字符串
compatible = "simple-bus";

// 4. 字节序列（每个byte用2个16进制数表示）
local-mac-address = [00 00 12 34 56 78];
local-mac-address = [000012345678];       // 紧凑写法

// 5. 多种值的组合，用逗号分隔
compatible = "ns16550","ns8250";
example = <0xf00f0000 19>,"a strange property format";
```

### 1.2 DTS文件整体结构体

```dts
/dts-v1/;                    // 版本声明
[memory reservations]        // 内存保留区域: /memreserve/ <address> <length>;
/ {                         // 根节点开始
    [property definitions]   // 根节点的属性
    [child nodes]           // 子节点
};                          // 根节点结束
```
- 内存保留区域：声明板子上有些内存想自己使用，不想给内核使用
- 像一个树状结构，根节点是整棵树的起点，下面挂着各种子节点

### 1.3 重要的默认属性

#### 1.3.1 根节点的关键（默认）属性

```dts
/ {
	// 子节点的reg属性中，用几个u32来描述地址
	#address-cells = <2>;
	// 子节点的reg属性中，用几个u32来描述大小
	#size-cells = <1>;
	// 定义了一些列的字符串，用来指定内核中那个machine_desc可以支持本设备
	compatible = "manufacturer,board-name";
	// 这个板子的具体型号
	model = "My Developmemt Board";
};
```
- `compatible`告诉内核："我这个板子可以用哪些驱动"
	- 内核可以支持多个单板，每个单板都有对应的machine_desc结构体，该结构体有不同的初始化函数
- `model`用来区分配置相似但型号不同的板子
	- 也就是说，compatible是一样的，这个时候就需要使用model来区分了

#### 1.3.2 内存节点/memory

```dts
memory {
	//必须有的
	device_type = "memory";
	// 内存起始地址和大小
	reg = <0x80000000 0x20000000>;
};
```

#### 1.3.3 启动参数节点/chosen

```dts
chosen {
	// 内核启动参数
	bootargs = "console=ttySAC0,115200 roor=/dev/mtdblock3";
};
```

#### 1.3.4 CPU节点 / cpus

```dts fold
cpus {
	#address-cells = <1>;
	#size-cells = <0>;

	cpu@0 {
		//必须有的
		device_type = "cpu";
		// 这是第0号CPU
		reg = <0>;
	};

	cpu@1 {
        device_type = "cpu";
        // 这是第1号CPU
        reg = <1>;           
    };
};
```

### 1.4 节点引用方法

#### 1.4.1 phandle引用

```dts
pic@10000000 {
    phandle = <1>;           // 给这个节点分配一个唯一ID
    interrupt-controller;
};

another-device-node {
    interrupt-parent = <1>;   // 通过phandle值1来引用上面的节点
};
```
- pic代表某个中断控制器
- ano代表某个产生中断的设备，要将中断传给pic，就需要指定其中断父节点（**引用**）

#### 1.4.2 label引用（推荐）

```dts
PIC: pic@10000000 {         // 给节点起个标签名
    interrupt-controller;
};

another-device-node {
    interrupt-parent = <&PIC>;  // 用&符号+标签名来引用
};
```
- **说明**：使用label更直观，编译器会自动转换为phandle方式

### 1.5 举例说明

对于引用了dtsi的dts文件来说，如果dtsi有一个led节点，在dts如何修改呢？
- 可以**对应层级下**直接写一个led节点，然后修改属性的值
- 也可以引用dtsi的led节点（确定有label），引用的格式为`&label {}`，然后修改属性的值
	- 注意：引用**必须放在根节点外面**

## 2 DTB格式

DTB是DTS的二进制版本，采用分段式布局，既保持了结构的清晰性，又优化了存储效率和加载速度。

### 2.1 概述

DTB（Device Tree Blob）是DTS文件编译后的二进制格式；内核启动时直接读取DTB文件来获取硬件信息。

### 2.2 DTB文件的整体布局

DTB文件就像一个精心组织的“文件柜”，按照固定的顺序存放不同类型的信息：

```txt
			 ------------------------------
     base -> |  struct boot_param_header  |  ← 文件头（说明书的目录）
             ------------------------------
             |      (alignment gap) (*)   |  ← 对齐填充
             ------------------------------
             |      memory reserve map    |  ← 内存保留映射表
             ------------------------------
             |      (alignment gap)       |  ← 对齐填充
             ------------------------------
             |                            |
             |    device-tree structure   |  ← 设备树结构数据（主要内容）
             |                            |
             ------------------------------
             |      (alignment gap)       |  ← 对齐填充
             ------------------------------
             |                            |
             |     device-tree strings    |  ← 字符串存储区
             |                            |
      -----> ------------------------------
      |
      |
      --- (base + totalsize)                ← 文件总大小
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6067f6cd4732cdddb960e8b329e0ffe4.png)


### 2.3 各部分详细说明

**DTB在存放格式是大端序（Big endian）**
- 大端序：0x12345678，高位在低地址
- 小端序：0x12345678，高位在高地址
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d3f7fb1d21385a72fabd1730d80a669b.png)


> [!warning]+ 注意
> - 大端序和小端序只对数字的存储有区别，对于字符串是没有区别的
> - 对于字符串“abc”，a永远放在低地址

**1、struct boot_param_header（文件头）**：DTB文件的“目录”，告诉系统：
- 这个文件有多大
- 各个部分在文件中的位置
- 文件的版本信息

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0e23b82b37005acbe591e4ce5c63eb29.png)


**2、alignment gap（对齐填充）**
- 为了让数据按照特定字节边界对齐（一般是4或8字节），在各个部分之后可能会插入一些填充字节
- 就像书本印刷时的空白页，确保内容排版整齐

**3、memory reserve map（内存保留映射表）**：记录哪些内存区域是被保留的，不能被内核随意使用。比如：
- bootloader占用的内存
- 其他重要数据占用的内存

**4、device-tree structure（设备树结构数据）**：DTB文件的`核心部分`，包含了所有的设备节点和属性信息：
- 节点的层次结构
- 每个节点的属性和值
- 节点之间的引用关系
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/23d03d538d12106b51323f559d2de7bf.png)
- 在dtb的二进制中
	- 0x00000001表示节点的开始，后面接上节点的名字
	- 0x00000003表述节点的属性，后面接上struct结构体（len-属性，即val的长度，nameoff-在字符串存储区的偏移），再加上实际的val值
	- 0x00000002表示节点的结束
	- 0x00000009表示整个设备树结构数据的结束


**5、device-tree strings（字符串存储区）**：
- 为了节省空间，DTB把所有的字符串（如属性名、节点名）集中存放在这里，其他地方只需要引用这些字符串的位置即可

### 2.4 理解要点

**空间优化**：DTB采用紧凑的二进制格式，相比文本格式的DTS文件，占用空间小，加载速度快

**结构化存储**：虽然是二进制格式，但DTB保持了清晰的结构分层，便于内核解析

**对齐要求**：各部分之间的对齐填充确保了数据访问的效率，符合处理器的内存访问要求

## 3 参考文档

- **官方文档**：[https://www.devicetree.org/specifications/](https://www.devicetree.org/specifications/)
- **内核文档**：Documentation/devicetree/booting-without-of.txt，主要内容如下
	- platform identification
	- runtime  configuration
	- device population