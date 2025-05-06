## 1 框架学习法

嵌入式Linux学习是交叉学科，知识体系庞杂，主要有以下内容
- C语言
- 数字电路基础 (单片机)
- ARM体系结构
- 硬件设计（ARM接口技术)
- Linux系统与管理
- Linux系统开发
- Linux驱动开发
- BootLoader(UBOOT)
- QT和C++
- Android系统和JAVA

举例：系统引导程序Uboot
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8fa937df49c81124e87648e6e095e295.png)
**入门级：**
1. 首先执行的片外程序
2. 引导内核
3. 生命周期、调用关系
4. 编译、烧写及命令

**工程师级：**
5. 启动过程简单分析
6. 修改读秒时间
7. 启动模式配置
8. CMDLINE
9. 启动时的点灯方法

**专家级：**
10. 源码分析
11. 系统移植
12. 增加功能及创新开发

**框架学习贯穿嵌入式学习的始终**
- 初学者应了解的框架
- Uboot框架
- 文件系统框架
- Linux基础框架
- Linux应用框架
- Linux驱动框架
- Android学习框架
- QT学习框架
- 硬件框架
## 2 Linux系统框架（裸机到OS）

### 2.1 裸机和Linux对比

- 裸机程序简单易懂
- 为什么要用Linux？
	- 可以解决更复杂的问题：网络、图形、多任务
	- 让产品开发变得更容易

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/acc1896f2a41cbb985159065d1ead359.png)

**Linux可以看成一个大软件/大程序**
- Linux大部分用C语言编写，少量汇编
- Linux是个大程序，函数库
- 我们是站在巨人的肩膀上做事
- Linux提供了大量的资源
	- 网络协议/多任务
	- 内存管理/设备管理
- Linux让产品开发更简单
- 学习有难度，如学骑自行车
- Linux架构越来越复杂，但使用越来越方便
- 使用的方便性是以复杂的架构为代价

**应用和驱动**
- 有了Linux，使得软件开发人员分化成两类：`应用与驱动`
- 应用开发人员可以不懂底层驱动，应用开发只关注业务逻辑，而驱动开发关注硬件特性;
- 应用程序通过"系统调用'来使用内核资源
- 驱动是Linux内核的一部分，驱动的架构越来越复杂，目的是为了我们做的事情越来越少

### 2.2 Linux内核态和用户态

- 用户态的程序不能直接访问硬件资源
- 内核态和用户态不仅是软件上的抽象，ARM处理器本身在硬件上就支持这两种状态
- ARM处理器的工作模式：
	- 用户模式
	- 系统模式
	- 中断模式
- 应用直接访问硬件会触发异常中断
- 内核态和用户态的划分使得系统更加安全，内核级有更高的特权。
- 进一步理解'`系统调用`'，它是用户态调用内核态函数的方法，一般通过`软中断`的方式
- 软中断是软件指令触发，ARM有对应指令，不同于按键等外部中断

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8785b85a99b4d1c1b7121049bf54476e.png)

### 2.3 Linux的文件系统

- 文件系统可直观理解为`Windows`上的文件资源管理器，应用程序放在文件系统当中
- Linux启动后一定要挂接一个文件系统，但`VxWorks，ucos`等并不需要挂接
- 文件系统可大可小，通过构造文件系统可衍生`QT，Ubuntu，Android`等系统
- `Linux的重要思想：一切皆文件`
- 硬件的操作（串口/led/按键）都可归结为：`read，write，open，close`
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/79f699c0e6571fc3f8d91809704b67ca.png)
**初学者需要搞清楚的三个文件**
- 引导程序（bootloader）：uboot.bin / uboot.imx
- Linux内核镜像：zImage
- 文件系统镜像：system.img / rootfs.tar.ba2
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e89d4cb866f837bdde754087de3f3c10.png)
初级很多工作都是围绕这三个知识点展开的，如开发环境搭建、编译系统、烧写系统。
不同的系统文件名会有差异，设备树文件（可以看成Linux内核的一部分）
## 3 Linux应用程序编程框架

### 3.1 简介

Linux应用编程即Linux系统编程，是基于Linux内核之上，基于系统调用或者库函数的编程。

Linux内核中设置了一组用于实现各种系统功能的子程序，称为系统调用。系统调用由操作系统核心提供，运行于核心态；而普通函数调用由函数库或用户自己提供，运行于用户态。

系统调用在ARM系统一般用“软中断”的方式来实现，ARM处理器的工作模式：
- 用户模式
- 系统模式
- 中断模式

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e4136fc813603ab3a1cc407c4f9357a4.png)

**Linux应用编程包含丰富的内容：**
- 文件IO
- 多进程和多线程
- 网络Socket编程
- 时间函数、文件系统操作
- 用户管理、内存管理等

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ce04c4c49670b1fc471489024b09a343.png)

### 3.2 Linux文件IO

- 一些皆文件（串口/LED等）
- 文件IO实际就是多文件的读写操作
	- open()
	- read()
	- write()
	- ioctl()
	- close()
	- 文件描述符（句柄）、fd=open（"文件目录"）
- 标准IO（带缓冲的IO），可移植性更好

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/16f345e5687bea3b61e33653951a87b1.png)

**Linux文件IO的5种模型**
- 阻塞
- 非阻塞
- 信号
- 多路复用
- 异步

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2106f8098d7d36541739e1cc55faad0e.png)

### 3.3 Linux多进程

多进程是多任务在Linux上的具体实现，多任务是操作系统的基本功能。

进程的三种状态：就绪、阻塞、运行
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f31f05cdb6652667aa6f31ee786cadfd.png)

通过fork()函数来创建子进程
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/202ca9b556ec38321f92f8a793530e5a.png)

**进程间的通信方式（重要）**
- 管道（匿名管道和命名管道）
- 信号
- 信号量
- 消息队列
- 共享内存
- 套接字

### 3.4 Linux多线程

- 单个进程内可以运行多个线程，线程是操作系统时间片调度的最小单位
- 每个线程可使用进程的全局变量
- 线程生成函数：`pthread_create()`
- 线程同步(共用资源，合理分配资源的方式)
	- 互斥锁
	- 条件变量
	- 读写锁
	- 信号量

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1b7119e4c938ea55bd028e0487c7c8f7.png)

### 3.5 网络通信（socket编程）

网络通信是重点！！！很多操作系统的网络编程方式是一样的，都采用了socket方式
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7c5eb9925e1799c86333bbeaeeedb20a.png)

### 3.6 应用编程框架路线

**入门**
- 文件IO基础
- 多线程多进程基础
- 网络socket编程
- 时间函数

**进阶**
- 5种IO模型
- 进程通信
- 线程同步
- 标准IO/文件共享dup/文件锁fcntl
## 4 Linux驱动架构演进

### 4.1 驱动学习难点

现象：网络资料丰富，涵盖方方面面的知识点，Linux没有秘密，Linux是开源知识，通用知识

难点：
- 知识体系庞大，逻辑关系复杂
- 驱动架构很复杂
- 老架构依然可用，很好的继承和发展

发展：
- 原始架构：设备节点（mknod）（2001-2004）Linux_v2.4
- 平台总线：自动生成节点（2004-2011）Linux_v2.6
- 设备树：（2012-至今）Linux_v3.x

总体来说
- 产品开发越来越简单，开发工作量减少
- 架构越来越复杂

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4ff0b57969e3864df20a01fa1c285e88.png)

### 4.2 Linux驱动的原始架构（Linux V2.4）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f69f4536a8381cf0edfb0d1f573b6ed5.png)

- **设备节点，即设备文件（/dev/xxx），它是上层应用和底层驱动的桥梁**
- Linux：设备即文件--->read、write、ioctl、close、open
- 主设备号、次设备号、mknod()
- **结构体 file_opreations（函数指针）**
- **register_chrdev() ---> 系统注册到内核中，将设备节点和驱动函数关联起来**
- 用户态：read() -> sys_read() -> vfs_read() -> 驱动read()
- 原始架构很重要，里面的知识并没有被淘汰，而是被封装和继承了

`原始架构是应用和驱动分开了`
### 4.3 平台总线架构（platform）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f817f277c4b514c78a67ff1e569943cc.png)

- Linux V2.6版本，封装了原始架构，更加抽象
- 引入了设备驱动模型（`sysfs`），使得热插拔/电源管理得以加强
- 使得做产品更加省事省力，实现了BSP（板级支持包）和驱动的分离

`平台总线架构是在驱动部分，进一步区分了平台设备和平台驱动`

### 4.4 Linux设备树

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3897f04630555c2dd1ce693c7ad01147.png)

- 设备资源独立了出来（arch/arm/mach-xxxx/board-xxxx.c），从c文件发展为dts设备树脚本文件（arch/arm/boot/dts/xxx.dts）
- 换个板子，不需要编译Linux系统，只需要换个设备树文件就好
- `bootloader参与传递设备资源，即启动时把设备树文件传给内核`

`Linux设备树`