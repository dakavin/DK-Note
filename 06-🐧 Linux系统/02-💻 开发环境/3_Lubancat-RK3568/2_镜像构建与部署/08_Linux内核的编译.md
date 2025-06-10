---
文章标题: "[[08_Linux内核的编译]]"
文章作者: Dakkk
文章概要: |
  本文详细介绍了Linux内核编译过程，包括获取内核源码、工程结构分析、配置选项设置、编译方法及deb包构建与安装，重点讲解RockChip芯片的内核定制化。
tags:
  - Linux内核编译
  - RockChip
  - 内核配置
  - deb包构建
  - 设备树
  - 交叉编译
  - 嵌入式开发
  - 内核模块
相关文章:
  - "[[../../../02-💻 开发环境/2_环境、启动和编译烧录相关(迅为)/3_编译与烧写（保留，直接看Lubancat板卡即可）]]"
  - "[[../../../07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/1_驱动章节实验环境搭建]]"
  - "[[10_设备树的简介]]"
  - "[[../../../01-❇️ 参考资料/4_常见问题解决/学习问题记录]]"
  - "[[../../../../05-🔧 51&32单片机开发/01-🎯 51单片机/01-📚 基础入门/_常用函数]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/08_Linux内核的编译.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-04 19:11:18
修改时间: 2025-05-28 00:19:51
---


**这里的Linux内核指的是Kernel，或者称内核也是指的Kernel**
## 1 为什么要自己编译kernel

虽然我们提供的内核已经支持绝大多数功能了，但是对于一些需要定制功能的客户来说不一定有他们需要的功能配置，所以本章将讲解如何配置与编译内核
## 2 获取kernel

有三个方法获取Kernel源码
- 一个是Kernel官方内核源码
- 一个是RockChip官方的kernel源码，
- 一个是经过我们修改适配我们板卡的kernel源码，我们这里使用我们提供的源码为例， 对官方源码感兴趣的小伙伴也可以下载来学习配置。

我们的kernel是根据Rockchip官方提供的kernel定制的，Rockchip的kernel是根据kernel官方LTS(长期支持)版本进行芯片适配的。 我们作为嵌入式开发者，一般只需要使用芯片厂商适配好的kernel进行开发即可， 对于从kernel官方下载新版本来适配这是芯片厂商做的事情。

目前LubanCat_Linux_rk356x_SDK使用kernel-4.19版本。LubanCat_Linux_rk3588_SDK使用kernel-5.10版本， LubanCat_Linux_Generic_SDK根据不同板卡使用kernel-5.10或kernel-6.1

由于内核主线对特定芯片的开发时慢于芯片厂商的，并且芯片厂商很少提交到内核主线， 就导致芯片的很多功能并没有被实现，所以目前我们只能使用Rockchip较旧的kernel版本。

Kernel官方内核源码：[https://www.kernel.org/](https://www.kernel.org/)

Rockchip内核源码: [https://github.com/rockchip-linux/kernel/](https://github.com/rockchip-linux/kernel/)

LubanCat内核源码：[https://github.com/LubanCat/kernel](https://github.com/LubanCat/kernel)

LubanCat内核源码分支说明：
- lbc-develop-5.10: 适用于LubanCat_Linux_Generic_SDK的5.10内核，版本较新
- lbc-develop-6.1: 适用于LubanCat_Linux_Generic_SDK的6.1内核，版本最新，板卡逐步支持中
- `stable-4.19-rk356x`：适用于LubanCat_Linux_rk356x_SDK的稳定版内核
- stable-4.19-rk356x-rt104：适用于LubanCat_Linux_rk356x_SDK的实时内核
- stable-5.10-rk3588：适用于LubanCat_Linux_rk3588_SDK的稳定版内核
- stable-5.10-rk3588-rt89：适用于LubanCat_Linux_rk3588_SDK的实时内核

使用我们提供的内核源码
```shell
# LubanCat_Gen_SDK
git clone -b lbc-develop-5.10 https://github.com/LubanCat/kernel
git clone -b lbc-develop-6.1 https://github.com/LubanCat/kernel

# LubanCat_Chip_SDK
git clone -b stable-4.19-rk356x https://github.com/LubanCat/kernel
git clone -b stable-5.10-rk3588 https://github.com/LubanCat/kernel
```
## 3 kernel工程结构分析

学习一个软件，尤其是开源软件，首先应该从分析软件的工程结构开始。 一个好的软件有良好的工程结构，对于读者学习和理解软件的架构以及工作流程都有很好的帮助。

内核源码目录如下：
```shell
arch            COPYING        drivers   init          kernel       
make_deb.sh     README         sound     block         CREDITS        
firmware        ipc            lib       Makefile      samples   
tools           build_image    crypto    fs            Kbuild   
LICENSES        mm             scripts   usr           certs        
Documentation   include        Kconfig   MAINTAINERS   net          
security        virt
```

我们可以看到Linux内核源码目录下是有非常多的目录，且目录下也有非常多的文件， 下面我们简单分析一下这些目录的主要作用，仅列出一些常见的目录。

- `arch`：
	- 主要包含和硬件结构相关的代码，如arm、x86、MIPS、PPC，每种CPU平台占一个目录
	- arch中的目录下存放的是各个平台以及其平台芯片对Linux内核进程调度、内存管理、中断等支持，以及每个具体的Soc和电路板的板机支持代码
- `block`:
	- 在Linux中block表示块设备（以块(多个字节组成的整体，类似于扇区) 为单位来整体访问），譬如说SD卡、Nand、硬盘等都是块设备
	- block目录下放的是一些Linux存储体系中关于块设备管理的代码

- `crypto`:这个目录存放的是常用加密和散列算法（如md5、AES、SHA等），还有一些压缩和CRC校验算法
- `Documentation`：内核各部分的文档描述。
- `drivers`：**设备驱动程序** ，里面列出了Linux内核支持的所有硬件设备的驱动源代码，每个不同的驱动占用一个子目录，如char、block、net、mtd、i2c等
- `fs`：就是`file-system`，里面包含了Linux所支持的各种文件系统，如EXT、FAT、NTFS、JFFS2等
- `init`：内核初始化代码，就是linux内核启动时初始化内核的代码
- `ipc`：`inter process commuication` ，进程间通信，就是linux进程间通信的代码
- `kernel` ：kernel就是Linux内核，是Linux中最核心的部分，包括进程调度、定时器等，而和平台相关的一部分代码放在`arch/*/kernel`目录下
- `lib` ：库的意思，lib目录下存放的都是一些公用的有用的库函数，注意这里的库函数和C语言的库函数不一样的，因为在内核编程中是不能用C语言标准函数库的，所以需要使用lib中的库函数，除此之外与处理器结构相关的库函数代码被放在 `arch/*/lib` 目录下
- `mm` ：包含了所有**独立于cpu体系结构的内存管理代码**，如页式存储管理内存的分配和释放等，而与具体硬件体系结构相关的内存代码管理位于 `arch/*/mm` 目录下
- `net` ：网络协议栈相关代码，net目录下实现各种常见的网络协议
- `scripts` ：脚本文件，不是linux内核工作时使用的，而是用于配置编译内核的
- `security` ：内核安全模型相关的代码，例如最有名的SELINUX
- `sound` ：ALSA、OSS**音频设备的驱动核心代码和常用设备驱动**
- `usr` ：实现用于打包和压缩的cpio等
## 4 内核配置选项（Chip_SDK）

在不同版本的内核和不同系列的鲁班猫板卡使用的配置文件都保存在内核目录的arch/arm64/configs中， 具体使用的配置文件如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/aab0a80a23eb68c150849bac8ce18dd9.png)

- Kernel 4.19.232：LubanCat-RK356x系列板卡都使用lubancat2_defconfig

**💡提示：** 本节需要输入命令的操作，需在kernel目录下进行

以LubanCat-2板卡为例进行说明，**Linux内核的配置系统**由三个部分组成，分别是：
- `Makefile`：分布在 Linux内核源代码根目录及各层目录中，定义 Linux 内核的编译规则；
- `配置文件`：给用户提供配置选择的功能，如Kconfig文件定义了配置项， 在编译时，使用 `arch/arm64/configs/lubancat2_defconfig` 文件对配置项进行赋值；
- `配置工具`：包括配置命令解释器（对配置脚本中使用的配置命令进行解释） 和配置用户界面（linux提供基于字符界面、基于Ncurses 图形界面以及 基于 Xwindows 图形界面的用户配置界面，各自对应于make config、make menuconfig 和 make xconfig）

**❗️注意：** 如果自定义配置文件， 编译时要在板卡对应的device/rockchip/rk356x/BoardConfig.mk文件中修改RK_KERNEL_DEFCONFIG的定义
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/eb5f4c20540356cb52bc19822d80df29.png)

我们可以通过 一下命令来查看我们的配置 
```shell
#使用图形界面配置需要额外安装 libncurses-dev
sudo apt install libncurses-dev

#执行命令
make menuconfig KCONFIG_CONFIG=arch/arm64/configs/lubancat2_defconfig ARCH=arm64
```

make menuconfig是一个基于文本选择的配置界面， 推荐在字符终端下使用， 而这个配置文件为 `lubancat2_defconfig` 此时就可以看到在lubancat2_defconfig的配置选择， 可以通过键盘的”上”、”下”、”左”、”右”、”回车”、”空格”、”?”、”ESC”等按键进行选择配置，具体见：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7acd32c911b1cdfdf2411ba77c7567d0.png)

比如我们选择配置板卡的ov5648摄像头驱动： `ov5648` ， 如果找不到这个配置选项在哪里，可以使用 `make menuconfig` 中的搜索功能， 在英文输入法状态下按下”/”则可以进行搜索，输入”ov5648”找到改配置选项的位置， 当输入错误时，可使用 Ctrl+退格键 删除输入。 具体见：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f907e1f03a5a62c275ea862029bb68fe.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ed1dd53a8b17346659b5b3f2f09990fc.png)

从图中可以很明显看出 `ov5648` 配置选项位于 `-> Device Drivers` 选项下的 `-> Multimedia support (MEDIA_SUPPORT [=y])` 下的 `-> I2C Encoders, decoders, sensors and other helper chips` 的选项中， 其实也可以按下 `"1"` 直接可以定位到对应的选项， 然后选中以下内容即可，具体见图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2cfa76789007adbccae4f8a29ab28758.png)

可使用 `y、n、m` 键更改ov5648驱动的配置时， 其中y表示编译进内核中，m表示编译成模块，n表示不编译。 也可使用空格选择ov5648驱动的配置选项。

想要配置其他的驱动也是如此。

修改完成后，选择右下角Save进行保存， **注意不要保存到原路径，而是保存到.config** ，然后使用以下命令来保存defconfig文件并覆盖原来的配置文件。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f165382dbebdfe9c0fdf50abfdaa2f8b.png)

```shell
# 保存defconfig文件
make savedefconfig ARCH=arm64

# 覆盖原来的配置文件
cp defconfig arch/arm64/configs/lubancat2_defconfig
```

这样保存的原因是配置文件默认是精简版本的，编译使用时会和默认的配置文件进行比较从而得到完整的配置，如果直接保存则是完整版本的，会比精简版多几千行配置，不利于观察、修改。
## 5 kernel编译

**💡提示：** 本节需要输入命令的操作，需在SDK根目录下进行

在LubanCat-SDK中，自动编译脚本基本上都存放在build.sh中，这是SDK的主要功能入口

对于kernel的编译，有两种方式。一种是rk标准的boot分区，称为rkboot分区(已弃用)，另一种是野火修改的extboot分区。

rkboot分区是以二进制形式打包的，各个部分存放的地址经过了严格定义，读取文件时通过二进制文件头去判断文件的种类，目前LubanCat板卡已停止支持。

`重点：` 我们修改的extboot分区，是以ext4格式存储文件的，在系统内是可读可修改的，大大增加了便利性。 我们以此为基础，为extboot分区系统增加了**在线更新内核版本**，**修改设备树插件**，**切换主设备树的功能**， 最终实现了一个镜像通用所有使用同一型号处理器的LubanCat板卡的功能

### 5.1 extboot分区编译（Chip）

**❗️注意：** 以下内容中对function build_extboot()的描述仅适用于LubanCat_Chip_SDK。在LubanCat_Gen_SDK中编译流程基本相同，具体内容可以查看device/rockchip/common/scripts/mk-kernel.sh

由于在打包extboot分区时，会打包内核deb包，所以在构建extboot分区镜像前，我们**先要进行下一步骤，构建内核deb包。**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8f4bb0b102c43487f05c495780b066f1.png)

1. check_config检查配置文件是否存在
2. build_check_cross_compile设置交叉编译参数
3. 应用定义的内核配置文件
4. 编译内核镜像
5. 编译设备树
6. 创建EXTBOOT_DIR临时文件夹，存放用于生成boot分区内的文件
7. 复制内核镜像、设备树文件、设备树插件、uEnv环境变量文件
8. 判断是否添加ramdisk镜像
9. 复制启动logo、内核配置文件、System.map、内核deb包
10. 以ext4格式将EXTBOOT_DIR文件夹中的文件打包到extboot.img

在后续的步骤中，会将extboot.img软链接到rockdev/boot.img
## 6 构建内核deb包

我们在内核运行make bindeb-pkg 会生成至多5个Debian软件包，在LubanCat-SDK中我们可以直接使用以下命令构建。
```shell
./build.sh kerneldeb
```

**❗️注意：** 以下内容中对function build_kerneldeb()的描述仅适用于LubanCat_Chip_SDK。在LubanCat_Gen_SDK中编译流程基本相同，具体内容可以查看device/rockchip/common/scripts/mk-kernel.sh

其构建脚本如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/608b74107156cb7c50b9056197a2f8af.png)

1. check_config检查配置文件是否存在
2. build_check_cross_compile设置交叉编译参数
3. 应用定义的内核配置文件
4. 编译内核deb包

生成的软件包如下：
- linux-image-version：一般包括内核镜像和内核模块，我们还添加了设备树及设备树插件。
- linux-headers-version：包括创建外部模块所需的头文件
- linux-firmware-image-version：包括某些驱动程序所需的固件(这里不生成)
- linux-image-version-dbg：在linux-image-version基础上多了debug符号
- linux-libc-dev：包括GNU glibc之类与用户程序库相关的标头

**💡提示：** version为内核版本，如`4.19.232`

其中我们最主要用到的就是linux-image和linux-headers，而linux-firmware我们则可以通过网络进行安装
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d8879311d542c549fdad6ba43585c197.png)

- 通过安装linux-headers，我们可以`直接在板卡上安装gcc编译器进行本地编译`，具体的使用方法详见应用部署章节
- 通过安装linux-image则可以`更新内核镜像、设备树和设备树插件、内核模块`。

### 6.1 内核deb包的安装

我们将生成的deb包通过USB存储设备或网络复制到运行Ubuntu或Debian镜像的板卡中，使用以下命令安装软件包

**❗️注意：** 注意在更新时不要断电，否则可能会导致系统损坏无法启动。

```shell
# 内核为4.19.232版本
sudo dpkg -i linux-headers-4.19.xxx_4.19.xxx-xxx_arm64.deb
sudo dpkg -i linux-image-4.19.xxx_4.19.xxx-xxx_arm64.deb
```

在安装本地deb包时，要在deb包所在的目录下运行上面命令，xxx为deb包实际对应的数字。

等待deb包安装完成后重启，即可完成内核更新。

除了直接使用deb包进行本地更新以外，还可以使用野火软件源进行在线更新
```shell
sudo apt update

# 更新全部要更新的软件包
sudo apt upgrade

#查看内核版本
uname -a

# 只更新linux-image和linux-headers
# 内核为4.19.232
sudo apt install linux-image-4.19.232
sudo apt install linux-headers-4.19.232

# 内核为5.10.160
sudo apt install linux-image-5.10.160
sudo apt install linux-headers-5.10.160

# 内核为5.10.198及以上版本
sudo apt install linux-image-5.10.xxx-芯片型号
sudo apt install linux-headers-5.10.xxx-芯片型号

# 内核为6.1.xxx版本
sudo apt install linux-image-6.1.xxx-芯片型号
sudo apt install linux-headers-6.1.xxx-芯片型号
```

升级完成后重启板卡，即可完成内核更新。


