## 1 操作系统概述

### 1.1 硬件和软件

我们所熟知的计算机是由：硬件和软件所组成。

硬件：计算机系统中由电子，机械和光电元件等组成的各种物理装置的总称。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/47b92d3f2cce691e820e11b01d5743cb.png)

软件：是用户和计算机硬件之间的接口和桥梁，用户通过软件与计算机进行交流。

而操作系统，就是软件的一类。

一个完整的计算机：操作系统 + 硬件
### 1.2 操作系统

操作系统是计算机软件的一种，它主要负责：

作为用户和计算机硬件之间的桥梁，调度和管理计算机硬件进行工作。

而计算机，如果没有操作系统，就是一堆无法使用的塑料而已。

- 当计算机拥有了操作系统，就相当于拥有了灵魂，操作系统可以：
	- 调度CPU进行工作
	- 调度内存进行工作
	- 调度硬盘进行数据存储
	- 调度网卡进行网络通讯
	- 调度音响发出声音
	- 调度打印机打印内容
	- ......

- `用户使用操作系统，操作系统安排硬件干活`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9789519d54dbfc7183c64f13a960549.png)
### 1.3 常见操作系统

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/904d6fd36aea6e68b112442d616c8aca.png)



- PC端：Windows、Linux、MacOS
- 移动端：Android、IOS、鸿蒙系统

- 不管是PC操作系统，还是移动操作系统

- 其功能都是：调度硬件进行工作，充当用户和硬件之间的桥梁
## 2 初识Linux

### 2.1 Linux的诞生

Linux创始人: 林纳斯 托瓦兹![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bc54b2de886d22f3f823d9b908e91a66.png)

Linux 诞生于1991年，作者上大学期间

因为创始人在上大学期间经常需要浏览新闻和处理邮件，发现现有的操作系统不好用, 于是他决心自己写一个保护模式下的操作系统，这就是Linux的原型， 当时他21岁，后来经过全世界网友的支持, 现在能够兼容多种硬件，成为最为流行的服务器操作系统之一。

### 2.2 Linux内核

- Linux系统的组成如下：
	- Linux系统内核
	- 系统级应用程序
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f70ee3580059e1aa753a7dd0842c8288.png) 

- 内核提供系统最核心的功能，如：调度CPU、调度内存、调度文件系统、调度网络通讯、调度IO等
- 系统级应用程序，可以理解为出厂自带程序，可供用户快速上手操作系统，如：文件管理器、任务管理器、图片查看、音乐播放等

- 比如，播放音乐，无论用户使用自带音乐播放器或是自行安装的第三方播放器，均是由播放器程序，调用内核提供的相关功能，由内核调度CPU解码、音响发声等

- 可以看出，内核是Linux操作系统最核心的所在，系统级应用程序只是锦上添花。

- Linux内核是免费开源的，任何人都可以下载内核源码并查看且修改

- 可以通过：https://www.kernel.org   去下载Linux内核![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e72d3d6385c404de73506ba4c13234b.png)
### 2.3 Linux发行版

- 内核是免费、开源的，这也就代表了：
	- 任何人都可以获得并修改内核，并且自行集成系统级程序
	- 提供了内核+系统级程序的完整封装，称之为Linux发行版
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1eb0fd9d5fc7fb7573cf0604c68b0d8f.png)


- 任何人都可以封装Linux，目前市面上由非常多的Linux发行版，常用的、知名的如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/900763b310bc977358f2edd7a68aba4d.png)

- 本次课程，我们将基于：
	- CentOS操作系统进行讲解
	- 辅助讲解Ubuntu系统的相关知识

- 注意：不轮是任何的发行版，基础命令都是相同的，只有部分操作不同（如软件安装）
## 3 虚拟机介绍

### 3.1 虚拟机

学习Linux系统，就需要有一个可用的Linux系统。
如何获得？将自己的电脑重装系统为Linux？
NoNo。这不现实，因为Linux系统并不适合日常办公使用。
我们需要借助虚拟机来获得可用的Linux系统环境进行学习。

- 那么，什么是虚拟机呢？![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/026b91e71f07880f7ae9c909d7704159.png)

- 借助虚拟化技术，我们可以在系统中，通过软件：模拟计算机硬件，并给虚拟硬件安装真实的操作系统。

- 这样，就可以在电脑中，虚拟出一个完整的电脑，以供我们学习Linux系统。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4e852f00f067f37cd01d5da776bc1148.png)
## 4 VMware WorkStation安装

### 4.1 虚拟化软件

- 通过虚拟化技术，可以虚拟出计算机的硬件，那么如何虚拟呢？
- 我们可以通过提供虚拟化的软件来获得虚拟机。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb8990b5e2204d016bd3d7b3996e4073.png)

### 4.2 VMware WorkStation

课程选用VMware WorkStation软件来提供虚拟机。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/18dc7bd997e4ad34c3c68f52c0bc63e9.png)

下载地址： https://www.vmware.com/cn/products/workstation-pro.html

`默认安装即可`

- 软件安装完成后，验证一下网络适配器是否正常配置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f5ec46c3e720712b346e8288ebff732b.png)
- 或者通过快捷键：win + r , 输入ncpa.cpl回车即可打开
## 5 在VMware上安装Linux

### 5.1 下载CentOS操作系统

- 首先，我们需要下载操作系统的安装文件，本次使用CentOS7.6版本进行学习：https://vault.centos.org/7.6.1810/isos/x86_64/   (最后的/不要漏掉）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa5a3071435af2f389feb664ef365e41.png)

- 或者直接使用如下链接下载：https://vault.centos.org/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso

### 5.2 在VMware中安装CentOS操作系统

- 打开VMware软件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cc8e4776b55e566c7be036dcf123b7f9.png)


- 按照步骤创建虚拟机：
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/688ba047a23949b77a38f42eba881992.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84d56a69262d6ed260c48372653f2422.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b44c2ea593d8d9a211803bceef452db3.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b6d9ab753eca2238403431731b5f2b24.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4072f0adb638a3cae8b5884cca7c03e.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/226bea7f9da094118b6b6e9ca72b0f9a.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/226bea7f9da094118b6b6e9ca72b0f9a.png)

- 点击完成后，即开启了CentOS系统的安装，耐心等待安装完成即可，后续都是自动化的。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7d03da48c9d37a525803d922ce90750f.png)

## 6 远程连接Linux

### 6.1 图形化、命令行

- 对于操作系统的使用，有2种使用形式：
	- 图形化页面使用操作系统
	- 以命令的形式使用操作系统

- 不论是Windows还是Linux亦或是MacOS系统，都是支持这两种使用形式。

	- 图形化：使用操作系统提供的图形化页面，以获得`图形化反馈`的形式去使用操作系统。
	- 命令行：使用操作系统提供的各类命令，以获得`字符反馈`的形式去使用操作系统。

### 6.2 Windows系统的图形化和命令行

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8f22d86401b33a7b92766a594896c626.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9f2b96cf1edd405fc5eca53b7bb16c48.png)

### 6.3 Linux系统的图形化和命令行

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8afd65eb6df51ab56685fe69b96ab78b.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/95cde38acf2d4a29600a63331cc3e4af.png)

### 6.4 使用命令行学习Linux系统

- 尽管图形化是大多数人使用计算机的第一选择，但是在Linux操作系统上，这个选择被反转了。

- 无论是企业开发亦或是个人开发，使用Linux操作系统，多数都是使用的：`命令行`。

- 这是因为：
	- Linux从诞生至今，在图形化页面的优化上，并未重点发力。所以Linux操作系统的图形化页面：不好用、不稳定。
	
	- 在开发中，使用命令行形式，效率更高，更加直观，并且资源占用低，程序运行更稳定。

- 所以，后续的课程学习中，我们：
	- 除了在少数需要做对照讲解的情况下会使用图形化页面
	
	- 其余`都会以命令行`的形式去讲解Linux操作系统的使用

### 6.5 FinalShell

- 既然决定使用命令行去学习Linux操作系统，那么就必须丰富一下工具的使用。

- 我们使用VMware可以得到Linux虚拟机，但是在VMware中操作Linux的命令行页面不太方便，主要是：

	- 内容的复制、粘贴跨越VMware不方便

	- 文件的上传、下载跨越VMware不方便

	- 也就是和Linux系统的各类交互，跨越VMware不方便

- 我们可以通过第三方软件，FinalShell，远程连接到Linux操作系统之上。并通过FinalShell去操作Linux系统。这样各类操作都会十分的方便。

- `FinalShell的下载地址`为：
	- Windows:http://www.hostbuf.com/downloads/finalshell_install.exe
	- Mac:http://www.hostbuf.com/downloads/finalshell_install.pkg

下载完成后双击打开安装。

### 6.6 Windows系统安装FinalShell

按照提示一直下一步即可安装完成。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d40d19911f4663839b3819b52adec24d.png)
### 6.7 连接到Linux系统

- 首先，先查询到Linux系统的IP地址![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1aec29da4b548521925199e4af89ed88.png)
- 打开Finshell软件，配置到Linux系统的连接![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7d1d0dc96a6fe535d3419b90e0adaa27.png)
- 按图示配置连接，并点击确定![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/734d1ccdf2278e3d11114815257e7a2e.png)
- 打开连接管理器![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab169636c3cec57a0e155df8cb24134e.png)
- 双击刚刚配置好的连接![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f104236c749d2f87a51dca623a94989a.png)
- 点击接受并保存![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fafca9d22c56eaf27a937132c43bbad7.png)
- 如图连接成功![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e3ad4bf50eb2daeae2baca0e45cfb908.png)
## 7 拓展：WSL

- `WSL（Windows Subsystem for LInux）`
- 基于WSL我们可以得到Ubuntu发行版环境，可以拓展除CentOS发行版之外的额外体验和知识

### 7.1 为什么要用WSL

- WSL作为Windows10系统带来的全新特性，正在逐步颠覆开发人员既有的选择
	- 传统方式获取Linux操作系统环境，是安装完整的虚拟机，如VMware
	- 使用WSL，可以以非常轻量化的方式，得到Linux系统环境

- 目前，开发者正在逐步抛弃以虚拟机的形式获取Linux系统环境，而在逐步拥抱WSL环境。

- 所以，为什么要用WSL，其实很简单：
	- 开发人员都在用，大家都用的，我们也要学习
	- 实在是太方便了，简单、好用、轻量化、省内存

### 7.2 什么是WSL

- WSL：Windows Subsystem for Linux，是用于Windows系统之上的Linux子系统。

- 作用很简单，可以在Windows系统中获得Linux系统环境，并完全直连计算机硬件，无需通过虚拟机虚拟硬件。

- 简而言之：Windows10的WSL功能，可以无需单独虚拟一套硬件设备，就可以直接使用主机的物理硬件，构建Linux操作系统，并不会影响Windows系统本身的运行![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3346b402c6979fa5dd0163a7a297e673.png)
### 7.3 WSL部署

- WSL是Windows10自带功能，需要开启，无需下载![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/40033a0e8f092fc85a6ebcc18ed6e125.png)
- 点击确定后会进行部署，最后重启即可。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f3c148f94e030847fd13c3aff5d65ce.png)
- 打开Windows应用商店，搜索Ubuntu，点击获取并安装![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/abc8b3c16d1c856ef7692cbd4b500ed5.png)
- 点击启动![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a7a0bfb6ee285ce13feedcfd2bdf5f4.png)
- 输入用户名用以创建一个用户：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce3e7531cfc9a2e168dedd75177149b9.png)
- 输入两次密码确认（注意，输入密码没有反馈，不用理会，正常输入即可）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/68719b038fbf1ca751d56958c99b753b.png)
- 至此，得到了一个可用的Ubuntu操作系统环境![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8d2d278a780bd05add84568b3330de5b.png)
### 7.4 安装Windows Terminal软件

- Ubuntu自带的终端窗口软件不太好用，我们可以使用微软推出的：Windows Terminal软件

- 在应用商店中搜索terminal关键字，找到Windows Terminal软件下载并安装![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1bbd12bc5be4f215ce6c469d4cbc1528.png)
- 再次打开Windows Terminal软件，即默认使用Ubuntu系统了（WSL）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fb05890d3abe2e6fe864032763f4a79.png)

## 8 拓展：虚拟机快照

### 8.1 虚拟机快照

- 在学习阶段我们无法避免的可能损坏Linux操作系统。
- 如果损坏的话，重新安装一个Linux操作系统就会十分麻烦。

- VMware虚拟机（Workstation和Funsion）支持为虚拟机制作快照。
- 通过快照将当前虚拟机的状态保存下来，在以后可以通过快照恢复虚拟机到保存的状态。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cd8d18224462cecc892d5df1490639f.png)
### 8.2 在VMware中制作并还原快照

- 建议虚拟机关机后再进行制作快照

- 鼠标右键虚拟机，点击快照，进入快照管理器![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ffd898a847517469eda77b209b65e96b.png)
- 点击拍摄快照![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c6fa020eafeb6ab0ae4425e9f28c14a.png)
- 快照创建完成![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/25bed9f7640b0791d9259e75923c9da7.png)
- 恢复快照，点击快照后，点击转到![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/160e8978ada48ca83432db2846782ce9.png)