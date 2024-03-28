# 一、操作系统概述

## 1、硬件和软件

我们所熟知的计算机是由：硬件和软件所组成。

硬件：计算机系统中由电子，机械和光电元件等组成的各种物理装置的总称。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923001846281.png)

软件：是用户和计算机硬件之间的接口和桥梁，用户通过软件与计算机进行交流。

而操作系统，就是软件的一类。

一个完整的计算机：操作系统 + 硬件
## 2、操作系统

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

- `用户使用操作系统，操作系统安排硬件干活`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923001904503.png)
## 3、常见操作系统

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923001922986.png)



- PC端：Windows、Linux、MacOS
- 移动端：Android、IOS、鸿蒙系统

- 不管是PC操作系统，还是移动操作系统

- 其功能都是：调度硬件进行工作，充当用户和硬件之间的桥梁
# 二、初识Linux

## 1、Linux的诞生

Linux创始人: 林纳斯 托瓦兹![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923001935981.png)

Linux 诞生于1991年，作者上大学期间

因为创始人在上大学期间经常需要浏览新闻和处理邮件，发现现有的操作系统不好用, 于是他决心自己写一个保护模式下的操作系统，这就是Linux的原型， 当时他21岁，后来经过全世界网友的支持, 现在能够兼容多种硬件，成为最为流行的服务器操作系统之一。

## 2、Linux内核

- Linux系统的组成如下：
	- Linux系统内核
	- 系统级应用程序
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923001951850.png) 

- 内核提供系统最核心的功能，如：调度CPU、调度内存、调度文件系统、调度网络通讯、调度IO等
- 系统级应用程序，可以理解为出厂自带程序，可供用户快速上手操作系统，如：文件管理器、任务管理器、图片查看、音乐播放等

- 比如，播放音乐，无论用户使用自带音乐播放器或是自行安装的第三方播放器，均是由播放器程序，调用内核提供的相关功能，由内核调度CPU解码、音响发声等

- 可以看出，内核是Linux操作系统最核心的所在，系统级应用程序只是锦上添花。

- Linux内核是免费开源的，任何人都可以下载内核源码并查看且修改

- 可以通过：https://www.kernel.org   去下载Linux内核![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002014073.png)
## 3、Linux发行版

- 内核是免费、开源的，这也就代表了：
	- 任何人都可以获得并修改内核，并且自行集成系统级程序
	- 提供了内核+系统级程序的完整封装，称之为Linux发行版
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002025254.png)


- 任何人都可以封装Linux，目前市面上由非常多的Linux发行版，常用的、知名的如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002035490.png)

- 本次课程，我们将基于：
	- CentOS操作系统进行讲解
	- 辅助讲解Ubuntu系统的相关知识

- 注意：不轮是任何的发行版，基础命令都是相同的，只有部分操作不同（如软件安装）
# 三、虚拟机介绍

## 1、虚拟机

学习Linux系统，就需要有一个可用的Linux系统。
如何获得？将自己的电脑重装系统为Linux？
NoNo。这不现实，因为Linux系统并不适合日常办公使用。
我们需要借助虚拟机来获得可用的Linux系统环境进行学习。

- 那么，什么是虚拟机呢？![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002043053.png)

- 借助虚拟化技术，我们可以在系统中，通过软件：模拟计算机硬件，并给虚拟硬件安装真实的操作系统。

- 这样，就可以在电脑中，虚拟出一个完整的电脑，以供我们学习Linux系统。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002049683.png)
# 四、VMware WorkStation安装

## 1、虚拟化软件

- 通过虚拟化技术，可以虚拟出计算机的硬件，那么如何虚拟呢？
- 我们可以通过提供虚拟化的软件来获得虚拟机。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002055814.png)

## 2、VMware WorkStation

课程选用VMware WorkStation软件来提供虚拟机。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002105970.png)

下载地址： https://www.vmware.com/cn/products/workstation-pro.html

`默认安装即可`

- 软件安装完成后，验证一下网络适配器是否正常配置![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002117401.png)
- 或者通过快捷键：win + r , 输入ncpa.cpl回车即可打开
# 五、在VMware上安装Linux

## 1、下载CentOS操作系统

- 首先，我们需要下载操作系统的安装文件，本次使用CentOS7.6版本进行学习：https://vault.centos.org/7.6.1810/isos/x86_64/   (最后的/不要漏掉）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002125427.png)

- 或者直接使用如下链接下载：https://vault.centos.org/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso

## 2、在VMware中安装CentOS操作系统

- 打开VMware软件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002135518.png)


- 按照步骤创建虚拟机：
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002212157.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002217523.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002222933.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002226572.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002230856.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002235310.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002238542.png)

- 点击完成后，即开启了CentOS系统的安装，耐心等待安装完成即可，后续都是自动化的。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002253188.png)

# 六、远程连接Linux

## 1、图形化、命令行

- 对于操作系统的使用，有2种使用形式：
	- 图形化页面使用操作系统
	- 以命令的形式使用操作系统

- 不论是Windows还是Linux亦或是MacOS系统，都是支持这两种使用形式。

	- 图形化：使用操作系统提供的图形化页面，以获得`图形化反馈`的形式去使用操作系统。
	- 命令行：使用操作系统提供的各类命令，以获得`字符反馈`的形式去使用操作系统。

## 2、Windows系统的图形化和命令行

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002310295.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002316710.png)

## 3、Linux系统的图形化和命令行

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002322525.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002326342.png)

## 4、使用命令行学习Linux系统

- 尽管图形化是大多数人使用计算机的第一选择，但是在Linux操作系统上，这个选择被反转了。

- 无论是企业开发亦或是个人开发，使用Linux操作系统，多数都是使用的：`命令行`。

- 这是因为：
	- Linux从诞生至今，在图形化页面的优化上，并未重点发力。所以Linux操作系统的图形化页面：不好用、不稳定。
	
	- 在开发中，使用命令行形式，效率更高，更加直观，并且资源占用低，程序运行更稳定。

- 所以，后续的课程学习中，我们：
	- 除了在少数需要做对照讲解的情况下会使用图形化页面
	
	- 其余`都会以命令行`的形式去讲解Linux操作系统的使用

## 5、FinalShell

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

## 6、Windows系统安装FinalShell

按照提示一直下一步即可安装完成。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002340690.png)
## 7、连接到Linux系统

- 首先，先查询到Linux系统的IP地址![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002347034.png)
- 打开Finshell软件，配置到Linux系统的连接![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002350373.png)
- 按图示配置连接，并点击确定![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002354310.png)
- 打开连接管理器![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002413135.png)
- 双击刚刚配置好的连接![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002416506.png)
- 点击接受并保存![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002421651.png)
- 如图连接成功![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002428544.png)
# 七、拓展：WSL

- `WSL（Windows Subsystem for LInux）`
- 基于WSL我们可以得到Ubuntu发行版环境，可以拓展除CentOS发行版之外的额外体验和知识

## 1、为什么要用WSL

- WSL作为Windows10系统带来的全新特性，正在逐步颠覆开发人员既有的选择
	- 传统方式获取Linux操作系统环境，是安装完整的虚拟机，如VMware
	- 使用WSL，可以以非常轻量化的方式，得到Linux系统环境

- 目前，开发者正在逐步抛弃以虚拟机的形式获取Linux系统环境，而在逐步拥抱WSL环境。

- 所以，为什么要用WSL，其实很简单：
	- 开发人员都在用，大家都用的，我们也要学习
	- 实在是太方便了，简单、好用、轻量化、省内存

## 2、什么是WSL

- WSL：Windows Subsystem for Linux，是用于Windows系统之上的Linux子系统。

- 作用很简单，可以在Windows系统中获得Linux系统环境，并完全直连计算机硬件，无需通过虚拟机虚拟硬件。

- 简而言之：Windows10的WSL功能，可以无需单独虚拟一套硬件设备，就可以直接使用主机的物理硬件，构建Linux操作系统，并不会影响Windows系统本身的运行![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002505549.png)
## 3、WSL部署

- WSL是Windows10自带功能，需要开启，无需下载![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002509997.png)
- 点击确定后会进行部署，最后重启即可。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002512930.png)
- 打开Windows应用商店，搜索Ubuntu，点击获取并安装![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002515770.png)
- 点击启动![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002519084.png)
- 输入用户名用以创建一个用户：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002522209.png)
- 输入两次密码确认（注意，输入密码没有反馈，不用理会，正常输入即可）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002525512.png)
- 至此，得到了一个可用的Ubuntu操作系统环境![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002531169.png)
## 4、安装Windows Terminal软件

- Ubuntu自带的终端窗口软件不太好用，我们可以使用微软推出的：Windows Terminal软件

- 在应用商店中搜索terminal关键字，找到Windows Terminal软件下载并安装![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002534254.png)
- 再次打开Windows Terminal软件，即默认使用Ubuntu系统了（WSL）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002537435.png)

# 八、拓展：虚拟机快照

## 1、虚拟机快照

- 在学习阶段我们无法避免的可能损坏Linux操作系统。
- 如果损坏的话，重新安装一个Linux操作系统就会十分麻烦。

- VMware虚拟机（Workstation和Funsion）支持为虚拟机制作快照。
- 通过快照将当前虚拟机的状态保存下来，在以后可以通过快照恢复虚拟机到保存的状态。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002541196.png)
## 2、在VMware中制作并还原快照

- 建议虚拟机关机后再进行制作快照

- 鼠标右键虚拟机，点击快照，进入快照管理器![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002544149.png)
- 点击拍摄快照![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002547330.png)
- 快照创建完成![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002550956.png)
- 恢复快照，点击快照后，点击转到![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923002553676.png)