---
文章标题: "[[2_嵌入式Linux_Android学习路线(韦)]]"
文章作者: Dakkk
文章概要: |
  介绍嵌入式Linux和Android学习路线，从程序员三大方向出发，详细规划了从裸机开发到驱动程序的完整技术栈学习路径
tags:
  - 嵌入式Linux
  - Android
  - 驱动开发
  - ARM
  - bootloader
  - u-boot
  - 裸机开发
  - 系统开发
相关文章:
  - "[[1_驱动关键技术]]"
  - "[[4_Linux框架学习法(迅为)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/03-🌐 韦神课程/1_嵌入式学习路线/1_嵌入式Linux_Android学习路线.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-02 22:19:47
修改时间: 2025-05-02 22:48:29
---
## 1 程序员的三大方向

程序员的方向，一般可以分为 3 类：专业领域、业务领域、操作系统领域。你了解它们
后，按兴趣选择吧。对于专业领域，我提供不了建议。业务，也就是应用程序，它跟操作系
统并不是截然分开的：
- 专业领域：学术研究 工程实现
- 业务领域：界面显示 业务逻辑
- 操作系统领域：在嵌入式领域 Linux 一家独大！
		① 为产品规划硬件：
		按需求、性能、成本选择主芯片，搭配周边外设，交由硬件开发人员设计。
		② 给单板制作、安装操作系统、编写驱动
		③ 定制维护、升级等系统方案
		④ 还可能要配置、安装 Android 等 GUI 系统：
		⑤ 为应用开发人员配置开发环境
		⑥ 从系统角度解决疑难问题
		这个领域，通常被称为“底层系统”或是“驱动开发”

**嵌入式 Linux+Android 系统包含哪些内容**
- 简单地说，嵌入式 LINUX 系统里含有 bootloader、内核、驱动程序、根文件系统、应用程序这 5 大块。而应用程序，我们又可以分为：C/C++、Android
- 所以，嵌入式 Linux+Android 系统包含以下 6 部分内容：
	-  bootloader
	- Linux 内核
	- 驱动程序
	- 使用 C/C++编写的应用程序
	- Android 系统本身
	- Android 应用程序

Android 跟 Linux 的联系实在太大了，它的应用是如此广泛，学习了 Linux 之后没有理由停
下来不学习 Android。在大多数智能设备中，运行的是 Linux 操作系统；它上面要么安装有
Android，要么可以跟 Android 手机互联。现在，Linux+Android 已成标配
## 2 怎么学习嵌入式Linux操作系统

本文假设您是零基础，以实用为主，用最快的时间让你入门；后面也会附上想深入学习时可
以参考的资料。在实际工作中，我们从事的是“操作系统”周边的开发，并不会太深入学
习、修改操作系统本身
- 操作系统具有进程管理、存储管理、文件管理和设备管理等功能，这些核心功能非常稳定可靠，基本上不需要我们修改代码。我们只需要针对自己的硬件完善驱动程序
- 学习驱动时必定会涉及其他知识，比如存储管理、进程调度。当你深入理解了驱动程序后，也会加深对操作系统其他部分的理解
- Linux 内核中大部分代码都是设备驱动程序，可以认为 Linux 内核由各类驱动构成但是，要成为该领域的高手，一定要深入理解 Linux 操作系统本身，要去研读它的源代码。在忙完工作，闲暇之余，可以看看这些书：
	- 赵炯的《linux 内核完全注释》，这本比较薄，推荐这本。他后来又出了《Linux 内核完全剖析》，太厚了，搞不好看了后面就忘记前面了
	- 毛德操、胡希明的《LINUX 核心源代码情景分析》，此书分上下册，巨厚无比。当作字典看即可：想深入理解某方面的知识，就去看某章节

基于快速入门，上手工作的目的，您先不用看上面的书，**先按本文学习**

**入门路线图**
- 假设您是零基础，我们规划了如下入门路线图。前面的知识，是后面知识的基础，建议按顺序学习。每一部分，不一定需要学得很深入透彻，下面分章节描述
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/31198fd2c48e9faa212acc0ccb8ed3c8.png)
**怎么学习 ARM+Linux 的裸机开发**
- 学习裸机开发的目的有两个：
	- 掌握裸机程序的结构，为后续的 u-boot 作准备
	- 练习硬件知识，即：怎么看原理图、芯片手册，怎么写代码来操作硬件
	- 
后面的 u-boot 可以认为是裸机程序的集合，我们在裸机开发中逐个掌握各个部件，再集合
起来就可以得到一个 u-boot 了。后续的驱动开发，也涉及硬件操作，你可以在裸机开发中
学习硬件知识

注意：如果你并不关心裸机的程序结构，不关心 bootloader 的实现，这部分是可以先略过
的。在后面的驱动视频中，我们也会重新讲解所涉及的硬件知识

推荐两本书：杜春蕾的《ARM 体系结构与编程》，韦东山的《嵌入式 Linux 应用开发完全手
册》。后者也许是国内第 1 本涉及在 PC Linux 环境下开发的 ARM 裸机程序的书，如果我说
错了，请原谅我书读得少。

**bootloader 的学习**
bootloader 有很多种，vivi、u-boot 等等，最常用的是 u-boot。

u-boot 功能强大、源码比较多，对于编程经验不丰富、阅读代码经验不丰富的人，一开始
可能会觉得难以掌握。但是，u-boot 的主要功能就是：启动内核。它涉及：`读取内核到内存、设置启动参数、启动内核`。按照这个主线，我们尝试自己从零编写一个 bootloader，
这个程序相对简单，可以让我们快速理解 u-boot 主要功能的实现。
