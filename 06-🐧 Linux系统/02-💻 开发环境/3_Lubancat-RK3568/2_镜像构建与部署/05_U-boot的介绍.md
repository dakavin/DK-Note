---
文章标题: "[[05_U-boot的介绍]]"
文章作者: Dakkk
文章概要: |
  介绍U-boot引导程序的功能特性、启动流程、常用命令操作和环境参数配置，以RK3568处理器的LubanCat板卡为例详细说明U-boot的使用方法
tags:
  - U-boot
  - 引导程序
  - 嵌入式Linux
  - RK3568
  - 内核启动
  - mmc命令
  - 环境变量
  - 文件系统
相关文章:
  - "[[06_U-boot的修改与编译]]"
  - "[[../../../00-🎯 学习路线/2_嵌入式Linux_Android学习路线(韦)]]"
  - "[[1_Docker 镜像]]"
  - "[[1_MinGW编译器的安装和配置]]"
  - "[[2_MySQL的数据目录]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/05_U-boot的介绍.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-04 19:11:18
修改时间: 2025-05-28 00:19:51
---

## 1 U-boot简介

U-boot是从FADSROM、8xxROM、PPCBOOT逐步发展演化而来的。U-boot发展至今，已经可以实现非常多的功能
- 在操作系统方面，它不仅`支持嵌入式Linux系统的引导`，还支持NetBSD,VxWorks, QNX, RTEMS, ARTOS, LynxOS, Android等嵌入式操作系统的引导
- 在CPU架构方面， U-boot支持PowerPC、MIPS、x86、ARM、NIOS、XScale等诸多常用系列的处理器。

一般来说BootLoader必须**提供系统上电时的初始化代码**，在系统上电时**初始化相关环境**后， BootLoader需要**引导完整的操作系统**，然后**将控制器交给操作系统**。

简单来说`BootLoader是一段小程序`，它在系统上电时执行，通过这段小程序可以`将硬件设备进行初始化`，如CPU、SDRAM、Flash、串口、网络等，初始化完毕后调用操作系统内核。
## 2 启动U-boot

不同LubanCat板卡使用相同版本的U-Boot源码进行构建，仅少量配置不同， 下述文档以使用RK3568处理器的鲁班猫板卡为例进行说明，使用其他型号处理器的鲁班猫板卡内容基本相同， 仅在必要处指出不同点。

接下来我们以野火LubanCat板卡为例，使用RockChip提供的基于U-boot v2017深度定制的版本（称为next-dev），介绍U-boot的使用，

将板卡Debug串口与电脑连接，在上电后，U-boot启动kernel之前按下键盘的`ctrl+c键：进入U-Boot命令行模式`。如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d910d8840e60bd6db7a60020815de2af.png)

可以看出U-boot打印出了板子的一些基本信息，包括CPU、内存等信息。 还包括了在板卡初始化过程中的一些提示信息，如屏幕显示和时钟频率等。
## 3 U-boot快捷键

我们除了使用快捷按键进入U-Boot命令行模式，next-dev还提供了其他的串口组合快捷键，常用的如下
- ctrl+c：进入 U-Boot 命令行模式；
- ctrl+d：进入 loader 烧写模式；
- ctrl+b：进入 maskrom 烧写模式；
## 4 U-boot命令

当不清楚U-boot支持什么命令时， 可输入 **help** 或 **?** 可查看U-boot支持的命令列表，如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/56f944676aaef60fbec297a4da78ce9a.png)

可看到U-boot支持很多的命令，功能十分强大，与linux类似，在执行某条U-boot命令时， 可使用 _tab_ 自动补全命令，在没有命令名冲突的情况下可以使用命令的前几个字母作为命令的输入， 例如想要执行 **reset** 命令，输入 _res_ 或 _re_ 即可。

**❗️错误：** 默认不支持使用saveenv命令保存环境变量，因为可能会覆盖原有的默认env配置导致系统无法启动，如果要修改环境变量，请修改uboot源码。

当需要具体使用哪个命令时，可使用 **“help 命令”** 或 **“? 命令”** 的方式查看具体命令的使用说明。以 **“help printenv”** 为例，
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0d2d6b874bc60bde89d1a1f6119e4424.png)

关于U-boot命令的使用可参考U-boot官方链接： [http://www.denx.de/wiki/DULG/Manual](http://www.denx.de/wiki/DULG/Manual) **5.9. uboot Command Line Interface** 部分。
### 4.1 U-boot常见命令

U-boot命令众多，下面介绍常用的U-boot命令,详细的U-boot命令使用方式请使用 **help [命令]** 查看

|命令|说明|举例|
|---|---|---|
|help|列出当前U-boot所有支持的命令||
|help [命令]|查看指定命令的帮助|help printenv|
|reset|重启U-boot||
|printenv|打印所有环境参数的值||
|printenv [参数名]|查看指定的环境参数值|printenv bootdelay|
|setenv|设置/修改/删除环境参数的值|setenv bootdelay 3|
|saveenv|保存环境参数||
|ping|检测网络是否连通|ping 192.168.0.1|
|md|查看内存地址上的值|md.b 0x80000000 10<br><br>以字节查看0x80000000后0x10个数据|
|mw|用于修改内存地址上的值|mw.b 0x80000000 ff 10<br><br>以字节修改0x80000000后0x10个数据为ff|
|echo|打印信息，与linux下的echo类似||
|run|在执行某条环境参数命令|run bootcmd，执行bootcmd|
|bootz|在内存中引导内核启动||
|ls|查看文件系统中目录下的文件||
|load|从文件系统中加载二进制文件到内存||

以上为用户较为常用使用的部分命令，具体的使用方式可使用 **help [命令]** 查看。
### 4.2 mmc命令

mmc命令能够对如sd卡以及emmc类的存储介质进行操作，以下进行简单说明， 对于mmc命令不熟悉可使用 **help mmc** 查看相关命令的帮助，常用功能如下所示

| 命令        | 说明             |
| --------- | -------------- |
| mmc list  | 查看板子上mmc设备     |
| mmc dev   | 查看/切换当前默认mmc设备 |
| mmc info  | 查看当前mmc设备信息    |
| mmc part  | 查看当前mmc设备分区    |
| mmc read  | 读取当前mmc设备数据    |
| mmc write | 写入当前mmc设备数据    |
| mmc erase | 擦除当前mmc设备数据    |
#### 4.2.1 查看mmc设备

使用 **mmc list** 查看板卡相关设备，示例板卡板载emmc，可看到打印信息如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3cb1f8f61601017fd1cbc2d72ff251a5.png)

使用 **mmc dev index** 切换当前mmc设备，index为设备号，根据**mmc list**的结果，可知eMMC的设备号为0
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9375374df3fc738b519239c79a031fe8.png)

使用 **mmc info** 查看当前mmc设备的信息。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f71d884857def9cf796a1ea615d33ffc.png)

#### 4.2.2 查看分区信息

使用 **mmc part** 列出当前mmc设备分区
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4957226f3f1b662aff545f884d4e40d1.png)

### 4.3 文件系统操作命令

U-boot能够对ext2/3/4以及fat文件系统设备进行访问， 可使用`fstype命令`判断存储介质分区使用的是什么类型的文件系统。 以mmc介质为例，判断后两个分区的文件系统类型
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5d7bdf8ae30646af2ddc3c344c3c5f3e.png)

文件系统为ext4的第三个分区对应rootfs根文件系统。

知道了文件系统的类型即可使用相对应的命令对分区内容进行操作了。

ext4文件系统的命令使用方式和FAT使用方式相似，仅命令名不同， U-boot提供的ext文件系统命令如下

| 命令        | 说明                     |
| --------- | ---------------------- |
| ext4ls    | 查看存储设备的ext4分区里的内容      |
| ext4load  | 从ext4分区里读出文件到指定的内存地址   |
| ext4write | 把内存上的数据存储到ext4分区的一个文件里 |

下面以将/etc/apt/sources.list的内容读取到内存实例，简单说明U-boot对ext4文件系统操作。

1. 查看/etc/apt目录中的文件内容
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/73fe554f255e44d4401668b401c927dd.png)
2. 将/etc/apt/sources.list 文件读取到内存地址0x8000 0000处
```shell
=> ext4load mmc 0:3  0x80000000  /etc/apt/sources.list
464 bytes read in 701 ms (0 Bytes/s)
```

3. 查看内存0x8000 0000的部分数据内存
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c57258146b509bca24c01f1ebea0a877.png)
## 5 U-boot启动内核过程

`bootcmd与bootargs可以说是U-boot最重要的两个环境参数`， U-boot执行完毕之后，如果没有按下回车，则会自动执行bootcmd命环境参数里的内容， 而bootargs则是传递给内核的启动参数。

使用 **printenv bootcmd** 可查看bootcmd的内容
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/983455679397d2b89b6fe4edbf21e69c.png)

如果烧录了rkboot分区的镜像，会执行bootrkp，通过rk定义的启动流程来启动内核。 具体内容可参考RK官方文档 [《U-Boot v2017(next-dev) 开发指南》](https://github.com/Caesar-github/docs/blob/master/Common/UBOOT/Rockchip_Developer_Guide_UBoot_Nextdev_CN.pdf) 2.4启动流程

而如果烧录了extboot分区镜像， bootcmd最终会执行distro_bootcmd，同样可以使用 printenv distro_bootcmd 查看distro_bootcmd的内容如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6cc453c179722673e230c02065ba1dc9.png)

也就是说distro_bootcmd会执行 **bootcmd_mmc1、bootcmd_mmc0、bootcmd_mtd1、bootcmd_mtd2、bootcmd_mtd0、bootcmd_usb0、bootcmd_pxe、bootcmd_dhcp** 这9个环境参数

在前面我们知道，mmc1表示的sd卡的存储设备，mmc0表示的emmc设备，也就是说当sd卡插在板子时，若sd卡装有系统则会优先从sd卡内启动
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/72e97391032f787e5814a798dd861786.png)

bootcmd_mmc0与bootcmd_mmc1均设置各自 **devnum** 环境参数的值，最后运行 mmc_boot 环境参数，mmc_boot内容如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/222b0f736fa6925630d13a8816961c79.png)

在mmc_boot中设置 **devtype** 环境参数的值，最后运行 scan_dev_for_boot_part 环境参数，其作用是扫描设备中带有启动标志的分区
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8274162a716f927e80b165e91835ee6a.png)

**💡提示：** 若从U-boot中直接使用printenv查看会显得格式很乱，我们可以在U-boot源码中查看
- include/configs/rk3568_common.h
- include/configs/rockchip-common.h
- include/config_distro_bootcmd.h

在源文件config_distro_bootcmd.h中我们可以看到scan_dev_for_boot_part的定义
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/46615c54feb5108b9bf04026851645d7.png)

上述脚本先判断是否存在带 **-bootable** 标志的分区，如果存在就记录分区号， 然后运行 scan_dev_for_boot去扫描boot分区是否存在启动所需的文件。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7c42497e46edca4e1cfbc2d49cfcd99b.png)

scan_dev_for_boot的定义这是依次运行scan_dev_for_scripts、scan_dev_for_extlinux。 如果要改变不同启动方式的顺序，可以修改这一部分

scan_dev_for_extlinux用于扫描boot分区是否存在extlinux.conf配置文件， 存在的话会读取配置文件进行启动。配置文件里定义了内核、设备树等，还可以图形化选取不同版本内核启动，感兴趣可以去了解一下。

scan_dev_for_scripts用于扫描boot分区是否存在boot.scr文件，存在的话就会按照boot.scr脚本中的启动流程进行启动。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eb8173868506b76e818019957595a77a.png)

上述脚本在scan_dev_for_scripts中先去检测我们的boot分区中有没有名称为boot.scr.uimg或boot.scr的启动脚本， 如果检测到的话先输出”Found U-Boot script 脚本路径”这个提示信息，然后运行boot_a_script。如果检测不到的话就跳出。

在我们的鲁班猫板卡boot分区打包时已经编译了boot.scr的源文件并进行打包，所以这里会继续执行boot_a_script

在boot_a_script中读取boot分区中的boot.scr文件并加载到内存中，并使用source运行这个脚本。

boot.scr的源文件储存在SDK目录下的`kernel/arch/arm64/boot/dts/rockchip/uEnv/boot.cmd`，下面是boot.cmd中的内容
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c438b534639e27391cbf2e8cea999e97.png)

1. 打印提示信息，提示运行到了boot.scr脚本
2. 判读boot分区中是否存在uEnv/uEnv.txt这个文件，如果存在的话就继续执行下面的脚本
3. load命令读取uEnv文件到内存，里面保存了自定义的环境变量，如内核，设备树插件等信息
4. env import命令导入读取到的环境变量
5. setenv命令设置bootargs环境变量，以便启动内核时向内核传递参数。具体内容是设置了rootfs所在的分区，还将uEnv.txt文件中的cmdline追加到了bootargs
6. printenv命令打印bootargs环境变量到控制台，便于调试
7. 从boot分区加载initrd镜像到内存
8. 从boot分区加载kernel镜像到内存
9. 从boot分区加载主设备树rk-kernel.dt到内存
10. fdt命令读取加载的主设备树，并清空/chosen节点中的bootargs的内容，否则会覆盖步骤5设置的bootargs
11. dtfile命令读取uEnv文件中定义的设备树插件，并合并到主设备树中
12. 使用booti命令启动内核

经过上面的流程，我们就离开了uboot的工作区域，进入到了内核中。如果以上流程失败，将会继续运行scan_dev_for_extlinux， 解决由于有时设备树插件加载失败等原因导致无法启动的问题。
## 6 U-boot环境参数介绍

U-boot中环境参数为我们提供一种不修改U-boot源码的情况下， 能够修改kernel启动倒计时、ip地址、以及向内核传递不同的参数等。

在板子上使用 **printenv** 可查看板子上所有的环境参数， 使用 **setenv** 添加/修改/删除环境参数，具体说明如下所示
```shell
#设置新的环境参数名为abc，值为100
=> setenv abc 100
=> echo $abc
= 100

#将值修改为200
=> setenv abc 200
=> echo $abc
200

#删除abc环境参数
=> setenv abc
=> echo $abc
```

默认情况下使用setenv命令修改环境参数重启后就会消失， 若想要掉电保存需要执行 **saveenv** 将环境参数保存到存储介质。

**❗️警告：** 不建议在uboot中使用此操作，会覆盖默认的环境变量

U-boot上有一些官方规定的环境变量，这些环境变量在U-boot有着特殊的作用， 可通过以下链接查看： [https://www.denx.de/wiki/view/DULG/UBootEnvVariables](https://www.denx.de/wiki/view/DULG/UBootEnvVariables)