---
文章标题: "[[07_Linux的介绍]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux操作系统的起源、发展历程、内核架构特点，详细阐述Linux内核五大子系统组成及其功能，说明Linux采用单内核但融合微内核优势的设计理念。
tags:
- "Linux"
- "Unix"
- "操作系统"
- "内核架构"
- "进程管理"
- "内存管理"
- "文件系统"
- "设备管理"
- "网络子系统"
相关文章:
- "[[0_Linux系统编程]]"
- "[[1_开发环境搭建]]"
- "[[4_进程基础]]"
- "[[0_笔记（重点备忘）]]"
- "[[0_课程介绍]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/07_Linux的介绍.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-04 19:11:18
修改时间: 2025-05-28 00:19:51
---

## 1 Unix简介

Unix系统源自贝尔实验室，提供源码。在Unix中几乎所有东西都被当成文件对待。
## 2 Linux简介

芬兰人Linus最早开发，是类Unix系统，但不是Unix，实现了Unix的API（具体实现和Unix可能不同）

而Linux使用GPLv2开源协议
## 3 Linux的发展史

1991

8月25号 : 21岁的芬兰学生Linus Benedict Torvalds 在comp.os.minix 新闻组上宣布了它正在编写一个免费的操作系统。

9月1号 : Linux 0.01在网上发布。

2007

6月6号 : 华硕在2007的台北电脑展上展出了两款“易PC”（Eee PC）：701和1001。第1批易PC预装的是Xandros Linux， 这是一个基于Debian，轻量级的为适应小屏幕进行过优化的Linux发行版。

8月8号 : 2007年Linux基金会由开源发展实验室(OSDL)和自由标准组织(FSG)联合成立。这个基金会目的是赞助Linux创始人Linus的工作。 基金会得到了主要的Linux和开源公司，包括富士通，HP，IBM，Intel，NEC，Oracle，Qualcomm，三星和来自世界各地的开发者的支持。

11月5号 : 与之前大家推测的发布Gphone不同，Google宣布组建开放手机联盟(Open Handset Alliance)和发布Android， 它被称为“第一个真正开放的综合移动设备平台”。

2011

5月11号 : 2011年Google I/O大会发布了Chrombook。 这是一款运行着所谓云操作系统Chrome OS的笔记本。Chome OS是基于Linux内核的。

6月21号 : Linus Torvalds 发布了Linux3.0版本。

对linux发展史感兴趣的也可以到网上自行搜索linux发展史
## 4 单内核与微内核区别

单内核：所有内核从整体上作为一个单独大过程实现，运行于内核地址空间，内核通信简单。 内核通常以单个静态二进制文件存放于磁盘上。简单性能高，大多数Unix都是单内核。

微内核：微内核功能划分为多个过程，各个过程运行在单独地址空间，需要通过进程间通信IPC处理微内核通信， 只有请求强烈的过程才运行在内核态，其他过程在用户态。IPC开销大，且涉及用户态和内核态上下文切换。 所以多数的微内核实现（Windows NT、OS X）将所有微内核过程都运行于内核态
## 5 Linux内核

`Linux是单内核，但吸取微内核精华`：模块化涉及，抢占式内核，支持内核线程，可动态装载内核模块

Kernel即是Linux内核，Linux内核采用宏内核架构，即Linux大部分功能都会在内核中实现， 如进程管理、内存管理、设备管理、文件管理以及网络管理等功能，Linux在发展的过程中， 引入了`内核模块（Loadable Kernel Module，LKM）机制`，内核模块全称为`动态可加载内核模块`， 就是在内核运行时可以动态加载一组目标代码来实现某些特定的功能，在这过程中不需要重新编译内核就可以实现动态扩展。
## 6 Linux内核组成📕

Linux内核主要由5部分组成，分别为：**进程管理子系统**，**内存管理子系统**，**文件子系统**，**网络子系统**，**设备子系统**，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/57933f2a84bb964befc6ed225c0bb119.png)

- `进程管理`：负责进程的创建和销毁，进程的调度
- `内存管理`：负责内存的分配和回收，记录哪些内存被哪些进程使用，管理虚拟内存，将内存的物理地址和逻辑地址做一个映射，主要由MMU进行转换，页表的方式
- `文件系统`：
	- 这里的文件系统不仅仅是硬盘的抽象管理，它也可以是某些IO口的抽象；
	- 文件系统屏蔽了底层的细节，为上层提供统一的接口；
	- **Linux中一切皆文件**
- `设备管理`：
	- **设备管理功能主要由驱动 程序提供**，主要任务是控制设备完成输入或输出操作；
	- linux把设备看作是特殊的文件，系统通过处理文件的接口（虚拟文件系统VFS）来管理和控制各种设备
- `网络功能`：网络功能指的是**除了驱动程序提供的基本硬件操作外**，还有系统提供的机制和功能，比如TCP协议，地址解析等
## 7 Linux官方站点

官方地址为： [https://www.kernel.org/](https://www.kernel.org/)

在这可以下载最新版本的内核文件，但是大部分是没适配相应芯片的，适配芯片是芯片原厂干的事，我们一般不去为一款芯片做适配， 我们也没这个能力去适配，我们作为嵌入式开发人员主要职责是进行内核的配置与使用， 不要把大量的精力用于去适配一款芯片，这对初学者非常劝退，所以本书主要讲解**内核的配置和使用**。

目前LubanCat-2和LubanCat-1板卡已加入Linux Kernel，自 mainline 6.3-rc1 获得支持，相关内容可访问 [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/arm64/boot/dts/rockchip](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/arm64/boot/dts/rockchip) 查看 或访问 [https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/rockchip](https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/rockchip) 查看

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c501397713f90b677881f4de15c27c9b.png)
