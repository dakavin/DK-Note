---
文章标题: "[[3_Linux内核模块实验]]"
文章作者: Dakkk
文章概要: |
  文章详细介绍了Linux内核模块的开发与管理。通过“Hello World”模块示例，讲解了模块框架、编译流程和`insmod/rmmod`等基本命令。进一步深入探讨了内核模块传参（`module_param`）和符号共享（`EXPORT_SYMBOL`）机制，并演示了多模块间的依赖加载。
tags:
  - Linux内核模块
  - 内核开发
  - 驱动开发
  - Kbuild
  - module_param
  - EXPORT_SYMBOL
  - insmod
  - modprobe
相关文章:
  - "[[1_驱动关键技术]]"
  - "[[../../../../00-🎯 学习路线/2_嵌入式Linux_Android学习路线(韦)]]"
  - "[[../../../../00-🎯 学习路线/1_驱动开发（有线&无线）]]"
  - "[[../../../05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/1_设备树的引进和体验]]"
  - "[[../../../../05-💻 Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/1_Linux系统/2_如何学习Linux开发]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识/3_Linux内核模块实验.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-05-06 16:55:39
修改时间: 2025-05-30 08:27:46
---

本节实验使用到Lubancat_RK系列板卡（Lubancat2-RK3568）
## 1 hellomodule实验

### 1.1 实验代码讲解

从前面我们已经知道了内核模块的工作原理，这一小节就开始写代码了，下面就展示一个最简单hellomodule框架。
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kernel.h>

static int __init hello_init(void){
    printk(KERN_EMERG "[ KERN_EMEGR] Hello MOdule Init\n");
    printk("[ default ] Hello Module Init\n");
    return 0;
}

static void __eixt hello_exit(void){
    printk("[ default ] Hello Module Exit\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL2");
MODULE_AUTHOR("dakkk");
MODULE_DESCRIPTION("hello module");
MODULE_ALIAS("test module");
```

![hellomodule.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/df51255788d9dd743efaeb7e5f1ecd97.png)

接来下理解每一行代码的含义，并最终在Linux上运行这个模组，验证我们前面的理论，也为下一章驱动打下基础
#### 1.1.1 代码框架分析

Linux内核模块的代码框架通常由下面几个部分组成：
- **模块加载函数(必须)：** 当通过insmod或modprobe命令加载内核模块时，模块的加载函数就会自动被内核执行，完成本模块相关的初始化工作
- **模块卸载函数(必须)：** 当执行rmmod命令卸载模块时，模块卸载函数就会自动被内核自动执行，完成相关清理工作
- **模块许可证声明(必须)：** 许可证声明描述内核模块的许可权限，如果模块不声明，模块被加载时，将会有内核被污染的警告
- **模块参数：** 模块参数是模块被加载时，可以传值给模块中的参数
- **模块导出符号：** 模块可以导出准备好的变量或函数作为符号，以便其他内核模块调用。
- **模块的其他相关信息：** 可以声明模块作者等信息

上面示例的hello module程序只包含上面三个必要部分以及模块的其他信息声明
- 模块参数和导出符号将在下一节实验出现

- 头文件<linux/init.h>和<linux/module.h>，写内核模块必须要包含的
- 在内核模块运行的过程中，它不能依赖于C库函数， 因此用不了printf函数，需要使用单独的打印输出函数printk
- 调用宏module_init来告诉内核，使用hello_init函数来进行初始化
- 调用宏module_exir在内核注册该模块的卸载函数
- 必须声明该模块使用遵循的许可证，这里我们设置为GPL2协议
#### 1.1.2 头文件

前面我们已经接触过了Linux的应用编程，了解到Linux的头文件都存放在/usr/include中。**编写内核模块所需要的头文件，并不在上述说到的目录，而是在Linux内核源码中的include文件夹。**

- **# include <linux/module.h>：** 包含内核模块信息声明的相关函数
- **# include <linux/init.h>：** 包含了 module_init()和 module_exit()函数的声明
- **# include <linux/kernel.h>：** 包含内核提供的各种函数，如printk

`init.h头文件(位于内核源码 /include/linux/init.h)`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/593f62c69e7ff9cbe934db9b98adbfe3.png)
- Init.h头文件主要包含了一些宏定义，还有内核的initcall机制

`module.h头文件(位于内核源码/include/linux/module.h)`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/37db383b3b8bbb091a476d84b1785035.png)
#### 1.1.3 模块加载/卸载函数

**1. 模块加载函数**

 func_init函数在内核模块中也是做与初始化相关的工作，内核模块加载函数，位于`内核源码/linux/init.h`
 ```c
static int __init func_init(void){
	
}
module_init(func_init);
```
- 返回值：
	- 0表示模块初始化成功，并会在 `/sys/module` 下新建一个以模块名为名的目录
	- 非0表示模块初始化失败
- static回顾
	- static修饰的静态局部变量直到程序运行结束以后才释放
	- static的修饰全局变量只能在本文件中访问
	- static修饰的函数只能在本文件中调用
	- 使用原因：因为内核模块代码也是内核代码的一部分，为了避免函数重复，所以加上static

`_init、__initdata`宏定义，位于`内核源码/linux/init.h`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/50c02ed89db16f1912ff17fb2a5686e7.png)
- `___init` 用于修饰函数
	- 表示该函数放到可执行文件的`__init`节区中，该**节区的内容只能用于模块的初始化阶段**
	- 初始化阶段执行完毕之后，这部分的内容就会被释放掉
- `_initdata` 用于修饰变量

`module_init`宏定义
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/482d7805ef39f8ae068065d1fdb10dcd.png)
- 用于通知内核初始化模块的时候， 要使用哪个函数进行初始化
- 会将函数地址加入到相应的节区section中， 这样的话，开机的时候就可以自动加载模块了


**3. printk函数**
- printf：glibc实现的打印函数，工作于用户空间
- printk：内核模块无法使用glibc库函数，内核自身实现的一个类printf函数，但是需要指定打印等级
- `#define KERN_EMERG` “<0>” 通常是系统崩溃前的信息
- `#define KERN_ALERT` “<1>” 需要立即处理的消息
- `#define KERN_CRIT` “<2>” 严重情况
- `#define KERN_ERR` “<3>” 错误情况
- `#define KERN_WARNING` “<4>” 有问题的情况
- `#define KERN_NOTICE` “<5>” 注意信息
- `#define KERN_INFO` “<6>” 普通消息
- `#define KERN_DEBUG` “<7>” 调试信息

查看当前系统printk打印等级：`cat /proc/sys/kernel/printk`， 从左到右依次对应
- 当前控制台日志级别
- 默认消息日志级别
- 最小的控制台级别
- 默认控制台日志级别![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d2374550dce851b8a88691b8ca1ea2b5.png)
**打印内核所有打印信息**：`dmesg`，注意内核log缓冲区大小有限制，缓冲区数据可能被覆盖掉。

#### 1.1.4 许可证

Linux是一款免费的操作系统，采用了GPL协议，允许用户可以任意修改其源代码。

GPL协议的主要内容是**软件产品中即使使用了某个GPL协议产品提供的库**， 衍生出一个新产品，该软件产品都**必须采用GPL协议，即必须是开源和免费使用的**， 可见GPL协议具有传染性。

因此，我们可以在Linux使用各种各样的免费软件。

许可证的定义，位置：`内核源码/include/linux/module.h`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c86f6a71c5464d2602818c25753f79d0.png)
#### 1.1.5 相关信息声明

是内核模块程序结构的最后一部分内容，为了给使用该模块的读者一本“说明书”，属于可有可无的部分， 有则锦上添花，没有也无伤大雅。

内核模块信息声明函数，位置：`内核源码/include/linux/module.h`，自行搜索即可

|函数|作用|
|---|---|
|MODULE_LICENSE()|表示模块代码接受的软件许可协议，Linux内核遵循GPL V2开源协议，内核模块与linux内核保持一致即可。|
|MODULE_AUTHOR()|描述模块的作者信息|
|MODULE_DESCRIPTION()|对模块的简单介绍|
|MODULE_ALIAS()|给模块设置一个别名|
### 1.2 实验准备

将所写的代码的主目录，放在和内核代码同级目录下，然后进入所写代码的路径
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/44a2903aea8ee0b5b8c83c421bd17454.png)

**Makefile编写**
- 因为内核模块属于内核的一段代码，只是位置不在内核源码，所以编译的时候需要用到内核源码目录
- 得益于Linux内核使用的Kbuild系统，所以编译内核模块时，需要指定环境变量ARCH和CROSS_COMPILE的值
- 编写的内容如下
```makefile
# 指定内核目录，相对路径
KERNEL_DIR=../../../kernel/
# arm64体系结构
ARCH=arm64
# 指定交叉编译工具链
CROSS_COMPILE=aarch64-linux-gnu-
# 设置临时环境变量（编译参数）
export  ARCH  CROSS_COMPILE

# 以模块的方式编译
obj-m := hellomodule.o

all:
# make持续的实际调用
# -C:跳转到内核源码目录下去执行那里的Makefile
#  M:返回当前目录返回到当前目录来执行当前的Makefile
# modules编译模块
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules

.PHONE:clean

clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean	
```

**编译命令**
```shell
#使用了低版本的交叉编译器
#bear用于生成clangd索引时需要的compile_command.json文件
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
#如果本身就是低版本的交叉编译器
bear --make
```

编译成功后，实验目录下会生成名为“hellomodule.ko”的驱动模块文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d24420e1a73de0c9e01a2b9ec1cd2949.png)
### 1.2 程序运行结果

#### 1.2.1 内核模块相关命令（加载）

**lsmod命令**
- **功能：** 列出当前内核中加载的模块，格式化显示在终端
- **原理：** 将/proc/modules中的信息调整一下格式输出
-  lsmod输出列表有一列 Used by， 它表明此模块正在被其他模块使用，显示了模块之间的依赖关系

**insmod命令**
- 通过该命令可以加载模块到内核中（sudo权限，且不依赖其他模块）
- 不确定是否使用到其他模块的符号，可以尝试modprobe，后面会有它的详细用法
- 通过insmod命令加载hellomodule.ko内核模块到内存时
	- 会自动执行module_init()函数，进行初始化操作，该函数打印了`hello module init`
	- 再次查看已载入系统的内核模块，我们就会在列表中发现hellomodule.ko的身影 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/42836641636efa54cd1e033b10422459.png)
- 有些内核模块有依赖关系，不能直接用insmod加载，需要前加载前置模块，下一节实验会讲到

**modprobe命令**
- 功能同insmod，可加载内核模块
- **核心特性**：自动处理模块依赖关系并按需加载
- **使用前提**：
  1. 需先执行`depmod -a`生成依赖配置
  2. 模块必须存放在`/lib/modules/<内核版本>`目录

**depmod命令**
- 用于创建模块关系依赖的文件，给modprobe命令使用
- 在Linux系统中，`/lib/modules`目录包含了与内核版本号相关文件夹，用来存放的模块和配置信息
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5d32ef3e2352ccd0550d2217376ac083.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e9f234291911a110555feee3abab5689.png)
- 主要关注配置文件 `modules.dep`，其列出了模块之间的依赖关系，执行`depmod -a`时，会把依赖关系写入到该配置文件中

**rmmod命令**
- 将内核中运行的模块删除，只需要传给它路径就能实现
- 原理：卸载某个内存模块时，内存模块会自动执行`*_exit()`函数，进行清理操作
- 在我们的hellomodule中，*exit函数在控制台没有显示，可以用dmesg查看* ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/883289a7d036fc45b269dc2f10d038d9.png)
	- 原因：与printk的打印等级有关，前面有关于printk函数有详细讲解
- 不会卸载一个模块所依赖的模块，需要依次卸载，当然是用modprobe -r 可以一键卸载

**modinfo命令**
- 显示我们在内核模块中定义的几个宏 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/911afcf31c4628dd059ef82949ccc3da.png)
- 这些信息在模块代码中由相关内核模块信息声明函数声明
#### 1.2.2 系统自动加载模块

我们可以使用depmod和modprode工具，来实现模块hellomodule.ko在板子开机自动加载，方式如下
- 将模块放到 `/lib/modules/内核版本` 目录下
	- 内核版本，可以通过命令`uname -r`查询
- 然后使用depmod建立模块之间的依赖关系（即更新一下modules.dep文件），命令为`sudo depmod -a`，就可以在modules.dep文件中看到模块依赖关系，命令如下
```shell
cat /lib/modules/4.19.232/modules.dep | grep hellomodule
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7b099a2d5f3c3742a1d612e59bde9d6a.png)

- 最后在 `/etc/modules` 或者 `/etc/modules-load.d/<filename>.conf` 中加上hellomodule模块
	- 模块 **不写成 .ko 形式** 代表模块与内核紧耦合，有些是系统必须要跟内核紧耦合的，如mm系统
	- 一般写成 .ko 形式更好，这样出现错误也不会导致内核出现panic错误，如果集成到内核，出错了就会出现panic
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0ecdad67af1a6dbf48db0019adb33bb6.png)
重启开发板，lsmod可以查看到模块开机就被加载到内核里面了
## 2 内核模块传参和符号共享实验

### 2.1 实验代码讲解

#### 2.1.1 内核模块传参代码讲解

有时我们需要根据不同的应用场景给内核传递不同的参数，例如在程序中开启调试模式、设置详细输出模式以及制定与具体模块相关的选项，都可以**通过参数的形式来改变模块的行为**

**Linux内核提供一个宏来实现模块的参数传递**
- 代码路径：`内核源码/include/linux/moduleparam.h`
```shell
#define module_param(name, type, perm)				\
	module_param_named(name, name, type, perm)

#define module_param_array(name, type, nump, perm)		\
	module_param_array_named(name, name, type, nump, perm)
```
- `name`：参数名称（变量名）
- `type`：参数的类型，内核支持--->byte，short，ushort，int，uint，long，ulong，charp，bool，invbool
	- charp：字符指针，即char*
	- bool：布尔类型，0/1，ture=1，false=0
	- invbool：反布尔类型，0/1，ture=0，false=1
	- char：用该类型时，传参只能是byte
- `perm`：该文件的权限（**参数也是文件，具有权限！！！**）

| 权限类别  | 标志位     | 数字表示（八进制）无执行权限！ | 说明     |
| ----- | ------- | --------------- | ------ |
| 当前用户  | S_IRUSR | 0400            | 用户可读   |
|       | S_IWUSR | 0300            | 用户可写   |
| 当前用户组 | S_IRGRP | 0040            | 同组用户可读 |
|       | S_IWGRP | 0030            | 同组用户可写 |
| 其他用户  | S_IROTH | 0004            | 其他用户可读 |
|       | S_IWOTH | 0003            | 其他用户可写 |
 
> [!warning]
> - 文件权限唯独没有关于可执行权限的设置，该文件不允许它具有可执行权限
> - 强行给该参数赋予表示可执行权限的参数值S_IXUGO， 那么最终生成的内核模块在加载时会提示错误，见下图
> ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/fba50a9ca45739261b9e1e0695289600.png)
---

宏的相关定义已经解释完毕了，下面开始编写实验代码
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kernel.h>

// 定义4个常见变量
// 使用module_param宏来声明这4个参数
static int itype = 0;
module_param(itype, int, 0);

static bool btype = 0;
module_param(btype, bool, 0644);

static char ctype = 0;
module_param(ctype, byte, 0);

static char *stype = 0;
module_param(stype, charp, 0644);

static int __init param_init(void){
    printk(KERN_ALERT,"param init!\n");
    printk(KERN_ALERT,"itype = %d\n",itype);
    printk(KERN_ALERT,"btype = %d\n",btype);
    printk(KERN_ALERT,"ctype = %d\n",ctype);
    printk(KERN_ALERT,"stype = %s\n",stype);
    return 0;
}

static void __exit param_exit(void){
    printk(KERN_ALERT,"module exit!\n");
}

MODULE_LICENSE("GPL2");
MODULE_AUTHOR("dakkk");
MODULE_DESCRIPTION("module_param");
MODULE_ALIAS("module_param");
```

上面4个模块参数，**会以模块参数为名的文件，存在于 `/sys/module/模块名/parameters/` 目录**，其中itype和ctype的权限为0，所以无权限查看该参数
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/54e46261c5e4234053e50f0cf60c078f.png)

#### 2.1.2 符号共享代码讲解

[2_📕Linux内核模块](2_📕Linux内核模块.md)中已经详细分析了关于导出符号的内核源码

**符号就是在内核模块中导出函数和变量，在加载模块时被记录在公共内核符号表中，以供其他模块调用**

有了这个机制，可以允许我们使用分层的实现解决一些复杂的模块设计，即编写一个驱动时，可以把驱动按照功能分成几个内核模块，**借助符号共享去实现模块之间的接口调用，变量共享**。

```c
#define EXPORT_SYMBOL(sym) \\
__EXPORT_SYMBOL(sym, "")
```

EXPORT_SYMBOL宏的作用：**向内核导出符号，便于其他模块使用该符号**
- parametermodule.c
```c
// ....param_exit后面添加
// 导出变量
EXPORT_SYMBOL(itype);

int my_add(int a,int b){
    return a+b;
}

// 导出函数my_add
EXPORT_SYMBOL(my_add);

int my_sub(int a,int b){
    return a-b;
}

// 导出函数my_sub
EXPORT_SYMBOL(my_sub);
// ....后面接上module_init调用
```

- calculation.h
	- 声明额外的变量itype、my_add和my_sub函数
```h
#ifndef __CALCULATION_H__
#define __CALCULATION_H__syn

extern int itype;

int my_add(int a, int b);
int my_sub(int a, int b);

#endif
```

- calculation.c中使用extern关键字声明的参数itype，调用my_add()、my_sub()函数进行计算
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

#include "calculation.h"

static int __init calculation_init(void){
    printk(KERN_ALERT "calculation init\n");
    printk(KERN_ALERT "itype+1 = %d , itype - 1 = %d\n",my_add(itype,1),my_sub(itype,1));
    return 0;
}

static void __exit calculation_exit(void){
    printk(KERN_ALERT "calculation exit\n");
}

module_init(calculation_init);
module_exit(calculation_exit);

MODULE_LICENSE("GPL2");
MODULE_AUTHOR("DAKKK");
MODULE_DESCRIPTION("calculation module");
MODULE_ALIAS("calculation module");
```

### 2.2 实验准备

#### 2.2.1 makefile说明

```Makefile
...省略代码...
obj-m := parametermodule.o calculation.o
...省略代码...
```
#### 2.2.2 编译说明

输入如下命令来编译驱动模块：
```shell
#使用了低版本的交叉编译器
#bear用于生成clangd索引时需要的compile_command.json文件
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
#如果本身就是低版本的交叉编译器
bear --make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/008d1fb2318eeb26268f1d88726035d1.png)

### 2.3 程序运行结果

通过scp将编译好的ko拷贝到开发板中，加载ko并传参
```shell
sudo insmod parametermodule.ko itype=123 btype=1 ctype=200 stype=abc
```

查看内核导出的符号表 `cat /proc/kallsyms | grep my_add`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/fba2635d6b898a14b8f11828b426a79a.png)

将两个内核模板放入板卡的内核模块目录下 `/lib/modules/4.19.232/kernel`
```shell
sudo mv parametermodule.ko calculation.ko /lib/modules/4.19.232/kernel/
```

然后执行，depmod -a之后，再查看modules.dep配置文件，可以发现calculation.ko依赖parametermodule.ko文件，并且记录了模块存放的位置
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b30c566924d102ea0cad0a615197f1ba.png)

执行`modprobe calcution`命令会自动将parametermodule.ko模块加载
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6f4af34d97cd621b36680e4c6afc1fa4.png)

