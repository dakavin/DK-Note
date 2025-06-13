
学习任何编程语言时，我们都会从经典的"HelloWorld"程序开始。驱动程序开发也不例外！今天我们要编写第一个Linux驱动程序，用最简单的方式打开内核编程的大门。

简单来说，**驱动程序**就是操作系统与硬件设备之间的"翻译官"，它让系统能够正确地控制和使用各种硬件设备。

## 1 第一个驱动程序代码

让我们先来看看HelloWorld驱动的完整代码：

```c
#include <linux/module.h>
#include <linux/kernel.h>

static int __init helloworld_init(void) //驱动入口函数
{
    printk(KERN_EMERG "helloworld_init\r\n");//注意： 内核打印用 printk 而不是 printf
    return 0;
} 

static void __exit helloworld_exit(void) //驱动出口函数
{
    printk(KERN_EMERG "helloworld_exit\r\n");
} 

module_init(helloworld_init); //告诉linux模块入口函数，加载模块代码到操作系统
module_exit(helloworld_exit); //卸载
MODULE_LICENSE("GPL v2"); //同意 GPL 开源协议
MODULE_VERSION("1.0"); //驱动的版本
MODULE_AUTHOR("Cavium Inc");
MODULE_DESCRIPTION("helloworld Driver");  //lsmod
```

> [!note]+ 重要笔记 
> 虽然这段代码看起来很简单，但它包含了一个完整驱动程序的基本框架，麻雀虽小，五脏俱全！

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d973c5d6a9e58dc7a51e81669e01f310.png)

> [!info]+ 重要信息 
> 上图展示了驱动程序在Linux系统中的工作流程，从用户空间到内核空间的交互过程。

## 2 驱动程序基本框架详解

Linux驱动程序就像一座建筑，需要有固定的结构才能正常工作。让我们来了解这个"建筑"的各个组成部分。

Linux驱动的基本框架主要由以下几个部分组成：

**必需部分（缺一不可）：**
1. **模块加载函数**：就是驱动程序的"开机启动"代码。当你使用命令加载驱动模块时，内核会自动执行这个函数，完成驱动的初始化工作
2. **模块卸载函数**：这是驱动程序的"关机清理"代码。当卸载驱动模块时，内核会执行这个函数，做好善后工作
3. **模块许可证声明**：这就像给驱动程序办理"身份证"。如果不声明许可证，模块加载时会收到"内核被污染（kernel tainted）"的警告。可接受的声明包括"GPL"和"GPL v2"。

> [!warning]+ 警告或注意 
> 许可证声明虽然简单，但绝对不能省略，否则会影响驱动的正常加载。

**可选部分（锦上添花）：** 
4. **模块参数**：就像给程序传参数一样，模块参数是在加载驱动时可以传递给它的值，让驱动更加灵活。
5. **模块导出符号**：如果你的驱动想要"分享"一些函数或变量给其他模块使用，就需要导出这些符号。
6. **模块作者信息**：包括版本号、作者、描述等信息，方便管理和维护。

## 3 驱动编译实验

了解了代码结构，接下来我们来实际动手编译这个驱动。Linux内核驱动有多种编译方式，每种方式适用于不同的开发场景。

简单来说，我们可以把驱动编译成独立的模块文件，也可以直接编译到内核镜像中。

### 3.1 在内核源码外部编译成内核模块

这是最常见的驱动开发方式，特别适合第三方驱动开发。这种方式会把驱动编译成一个独立的.ko文件，可以动态加载和卸载，就像给电脑安装和卸载软件一样灵活。

**第一步：创建驱动代码文件**
- 创建和kernel目录统计的目录my_driver：`my_driver/1_LinuxModule/1_helloworld`

**第二步：编写外部编译Makefile**
```makefile
KERNEL_DIR= ../../../kernel/
ARCH = arm64
CROSS_COMPILE = aarch64-linux-gnu-
export ARCH CROSS_COMPILE

# 以模块的方式编译，注意文件名要和c文件一致
obj-m := hello.o 

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules -j8
.PHONY: clean
clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
	rm -f $(out)
```
- **内核源码目录(KERNEL_DIR)**：内核模块属于内核的一段代码，只是位置不在内核源码内，所以编译时需要指定内核源码目录
- **架构设置(ARCH)**：64位ARM架构，适用于现代ARM处理器
- **交叉编译链(CROSS_COMPILE)**：aarch64-linux-gnu-
	- 告诉编译器要为ARM64架构的目标设备编译代码，而不是当前的x86电脑。
	- 可以使用指令`ls -la /usr/bin/aarch64-linux-gnu-*`查看系统中的交叉编译器：
	  ![image|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/651753fda8998af7729711430a1f63fe.png)
- **编译目标设置**
	- **obj-m**：表示编译成.ko模块文件，可以动态加载
	- **obj-y**：表示编译到内核中，系统启动时自动加载
- **编译规则说明**
	- **-C $(KERNEL_DIR)**：切换到内核源码目录执行make
	- **M=$(CURDIR)**：指定模块源码在当前目录
	- **modules**：编译模块目标
	- **-j8**：使用8个线程并行编译，提高编译速度

**第三步：编译驱动模块**
1. **清理之前的编译文件**
```bash
make clean
```
2. **开始编译**
```bash
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
```

> [!tip]+ 重要提示 
> bear命令用于生成clangd索引需要的compile_commands.json文件，如果不需要代码补全功能，可以直接使用make命令

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f8ca7f0cb9b872603d7b8f6bdf9b2186.png)

> [!success]+ 成功或完成 编译成功后，会生成hello.ko文件，这就是我们的驱动模块！

**第四步：测试驱动模块**

编译完成后，需要将驱动传输到目标设备并进行测试。传输方式有多种：
- 通过adb方式传输
- 通过scp命令传输：[2_scp命令，拷贝文件](../01-❇️%20参考资料/0_个人实用技巧/2_scp命令，拷贝文件.md)
- 通过NFS网络文件系统：[5_挂载NFS网络文件系统](../05-💻%20Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/4_补充部分/5_挂载NFS网络文件系统.md)

```bash
# 传输驱动文件到设备
adb push hello.ko /mnt
# 修改权限
chmod 777 /mnt/hello.ko
# 加载驱动模块
insmod hello.ko
# 查看已加载的模块
lsmod
# 查看内核日志，验证效果
dmesg
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b3637531113c329aceb502522301b71e.png)

> [!example]+ 示例 
> 当你运行dmesg命令时，应该能看到"HelloWorld_init"的打印信息，说明驱动加载成功！


### 3.2 在内核源码内部编译成内核模块(正式发布使用)

这种方式将驱动代码直接放在内核源码树内部，与内核一起编译。适合开发板厂商集成自家的驱动，或者需要随内核一起发布的驱动。

**第一步：在内核驱动目录创建代码**
- 进入内核源码的drivers目录，创建test子目录：
```bash
cd ~/LubanCat_SDK/kernel/drivers
mkdir test
cd test
```
- 将驱动代码放入test目录中，文件名为hello.c（代码内容同上）。

**第二步：修改drivers目录下的Makefile**
```bash
~/LubanCat_SDK/kernel/drivers$ vim Makefile
```
在最后一行添加：
```makefile
# add by dakkk
obj-y += test/
```

> [!info]+ 重要信息 
> obj-y表示需要始终编译到内核中的对象，不是作为可加载模块。换句话说，这样编译的代码会在内核启动时自动执行，无法动态卸载

**第三步：创建test目录的Makefile**
- 在test目录下创建Makefile：
```makefile
obj-m += hello.o
```

> [!note]+ 重要笔记 
> 虽然父目录使用obj-y包含test目录，但test目录内部仍然可以使用obj-m编译成模块

**第四步：编译整个内核**
```bash
cd ~/LubanCat_SDK/kernel
# 清除内核编译产物
sudo make mrproper

# 编译内核时指定 GCC 9
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8

# 或者使用build.sh编译内核的方式
./build.sh kerneldeb

./build.sh extboot
```
编译完成后，hello.ko文件会出现在test目录中

> [!warning]+ 警告或注意 
> 这种方式需要编译整个内核模块系统，时间较长，通常用于正式产品集成

### 3.3 使用Kconfig条件编译

**Kconfig是Linux内核的配置管理系统**，它允许我们选择性地编译代码。简单来说，就像电脑软件的"自定义安装"一样，你可以选择安装哪些组件。

**第一步：修改drivers目录下的Kconfig**
```bash
~/LubanCat_SDK/kernel/drivers$ vim Kconfig
```

在适当位置添加：
```kconfig
source "drivers/test/Kconfig"
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/480cb3fcce356bbee7231576fe7932c4.png)

**第二步：创建test目录的Kconfig文件**
```bash
cd ~/LubanCat_SDK/kernel/drivers/test
touch Kconfig
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/986aed61414a6f388a48167f15e00075.png)

**第三步：编写Kconfig配置文件**
```kconfig
# SPDX-License-Identifier: GPL-2.0
menu "test"
config ROCKCHIP_TEST
        bool "ROCKCHIP_TEST"
        default n
        help
          test module.
endmenu
```

> [!note]+ 重要笔记 
> 这个配置文件定义了一个名为"ROCKCHIP_TEST"的配置项，默认不编译(default n)。bool表示这是一个布尔选项，只能选择编译或不编译

**第四步：修改test目录的Makefile**
```makefile
obj-$(CONFIG_ROCKCHIP_TEST) += hello.o
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7d166127cba79c1d0bfe64f05ccfa24f.png)

换句话说，只有当CONFIG_ROCKCHIP_TEST被启用时，hello.o才会被编译

**第五步：修改defconfig文件**
编辑默认配置文件：
```bash
vim ./kernel/arch/arm64/configs/lubancat2_defconfig
```

添加配置项：
```txt
CONFIG_ROCKCHIP_TEST=y
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/216edfa484562f1069d7f15c14ab4091.png)

> [!tip]+ 重要提示 
> 通过修改defconfig文件，可以让我们的驱动在内核编译时默认被包含进去

**第六步：使用menuconfig配置**
也可以通过图形化界面进行配置：
```bash
make menuconfig
```
在菜单中找到"Device Drivers" -> "test" -> "ROCKCHIP_TEST"，选择启用。

**第七步：编译验证**
```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules -j8
```

> [!abstract]+ 摘要或总结 Kconfig系统的优势在于：
> 
> - 可以选择性编译功能，减少内核体积
> - 便于管理大量驱动选项
> - 支持依赖关系管理
> - 提供图形化配置界面

## 4 总结

通过这个HelloWorld驱动实验，我们学会了：

> [!abstract]+ 摘要或总结
> 
> **关键技能掌握：**
> 
> - 理解了Linux驱动程序的基本框架和各个组成部分
> - 学会了编写最基础的驱动代码
> - 掌握了两种驱动编译方式：独立模块和内核集成
> - 了解了Makefile和Kconfig配置系统的作用
> - 完成了从编写代码到设备测试的完整流程

虽然这只是一个简单的HelloWorld程序，但它包含了驱动开发的核心概念。就像学会了走路，接下来我们就可以学习跑步了！下一步，我们将深入学习更复杂的驱动开发技术

> [!warning]+ 警告或注意 
> 驱动程序运行在内核空间，任何错误都可能导致系统崩溃。在实际开发中，一定要先在虚拟机或测试设备上验证，确保代码正确后再部署到正式环境。