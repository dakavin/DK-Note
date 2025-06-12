在学习 C 语言或者其他语言的时候， 我们通常是打印一句“helloworld” 来开启编程世界的大门。 学习驱动程序编程亦可以如此， 使用 helloworld 作为我们的第一个驱动程序。

  

接下来开始编写第一个驱动程序—helloworld。

  

```C++
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

看似非常简单的 helloworld 驱动代码， 却五脏俱全。 一个简单的 helloworld 驱动包含驱动的基本框架。 我们继续往下看。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d973c5d6a9e58dc7a51e81669e01f310.png)


## 1 驱动的基本框架

Linux 驱动的基本框架主要由模块加载函数， 模块卸载函数， 模块许可证声明， 模块参数，模块导出符号， 模块作者信息等几部分组成， 其中模块参数， 模块导出符号， 模块作者信息是可选的部分， 也就是可要可不要。 剩余部分是必须有的。 我们来看一下这几个部分的作用：

- 1 模块加载函数
    

当使用加载驱动模块时， 内核会执行模块加载函数， 完成模块加载函数中的初始化工作。

- 2 模块卸载函数
    

当卸载某模块时， 内核会执行模块卸载函数， 完成模块卸载函数中的退出工作。

- 3 模块许可证声明
    

许可证声明描述了内核模块的许可权限， 如果不声明模块许可， 模块在加载的时候， 会收到“内核被污染（kernel tainted） ” 的警告。 可接受的内核模块声明许可包括“GPL”“GPL v2”。

- 4 模块参数（可选择）
    

模块参数是模块被加载的时候可以传递给它的值。

- 5 模块导出符号（可选择）
    

内核模块可以导出的符号， 如果导出， 其他模块可以使用本模块中的变量或函数。

- 6 模块作者信息等说明（可选择）
    

## 2 实验

Build-in kernel第一步：

~/LubanCat_SDK/kernel/drivers$ vim Makefile

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/41e26f3b23162507ae5f5f2b92ede6ff.png)


第二步：

Touch 创建文件

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1f33897d78a6cd23d26176d249ca4f36.png)


第三步：

hell.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/339f7c668a84a9ec5f45559274916d8b.png)


Makefile

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ca4061c28709a14e5d83068f046ee7c7.png)


## 3 Makefile

- ARCH 架构 ARM、ARM64
    
- 交叉编译链： CROSS_COMPILE=aarch64-linux-gnu-
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f29518b54c125dd6bcadf6c0a4043be4.png)


  

- 编译对象：obj-m += hell.o
    
    - m 代表编译成ko
        
    - y 代表编译成内核中
        
- 内核源码路径：KDIR
    
- 查看当前路径：PWD ?= $(shell pwd)
    
- 编译：all:
    

make -C $(KDIR) M=$(PWD) modules #make操作

- 清除：clean:
    

make -C $(KDIR) M=$(PWD) clean #make clean操作

## 4 操作

- make clean
    
- make
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6f7139577be50eaebb6afceec89cc7a7.png)


编译完成：

adb push hell.ko /mnt

chmod 777 /mnt/hell.ko

insmod hell.ko

lsmod

dmesg ---- 实验效果

## 5 Kconfig



![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d3c5e874610a7e444a67e21027acd8d3.png)


  

步骤：

1. ~/LubanCat_SDK_back/kernel/drivers$ vim Kconfig
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/480cb3fcce356bbee7231576fe7932c4.png)


2. touch Kconfig
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/986aed61414a6f388a48167f15e00075.png)


3. 修改Kconfig
    

```C
# SPDX-License-Identifier: GPL-2.0
menu "test"

config ROCKCHIP_TEST
        bool "ROCKCHIP_TEST"
        default n
        help
          test module.

endmenu 
```

4. 修改Makefile
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7d166127cba79c1d0bfe64f05ccfa24f.png)


  

此时 hell驱动 默认不会编译。

5. 修改deconfig
    

./kernel/arch/arm64/configs/lubancat2_defconfig

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4a3108b8e9eacefec14aba969a6eab10.png)
