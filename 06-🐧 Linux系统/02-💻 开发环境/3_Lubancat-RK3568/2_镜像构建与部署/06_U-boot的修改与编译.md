---
文章标题: "[[06_U-boot的修改与编译]]"
文章作者: Dakkk
文章概要: |
  介绍基于RK处理器的鲁班猫板卡U-boot修改与编译方法，包括源码获取、编译流程、配置文件修改和设备树文件处理等实践操作。
tags:
  - U-boot
  - Rockchip
  - 嵌入式系统
  - 引导程序
  - 设备树
  - 交叉编译
  - RK3568
  - 板级支持包
相关文章:
  - "[[08_Linux内核的编译]]"
  - "[[../../../02-💻 开发环境/2_环境、启动和编译烧录相关(迅为)/3_编译与烧写（保留，直接看Lubancat板卡即可）]]"
  - "[[../../../07-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/1_驱动章节实验环境搭建]]"
  - "[[10_设备树的简介]]"
  - "[[11_设备树的编译]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/06_U-boot的修改与编译.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-04 19:11:18
修改时间: 2025-05-28 00:19:51
---

借助LubanCat-SDK，我们可以很方便的进行U-boot的编译，但是有时候我们还要根据板卡的实际情况来修改U-boot，实现一些定制功能。 为此，我们必须对U-boot进行一个深入的了解。

下面的内容，我们将会了解LubanCat-SDK是如何实现对U-boot的一键构建，构建参数又与哪些文件有关

不同LubanCat板卡使用相同版本的U-Boot源码进行构建，仅少量配置不同， 下述文档以使用RK3568处理器的鲁班猫板卡为例进行说明，使用其他型号处理器的鲁班猫板卡内容基本相同， 仅在必要处指出不同点。
## 1 获取U-boot

由于主线版U-boot对大部分比较新的处理器支持十分有限，所以我们选择使用RK版本的U-Boot：next-dev， 而RK版本的U-Boot又分为两部分，一个是U-boot本身的代码仓库，也就是u-boot目录下的内容， 还有一部分是工具包仓库，存放RK不开源的二进制文件、脚本、打包工具，用于打包生成loader、trust、uboot固件，在rkbin目录下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/72f0d9dc776e7584e81756bee1d9fce4.png)

**❗️注意：** rkbin和U-Boot工程必须保持同级目录关系。

在LubanCat-SDK中已经有了这两部分内容，我们可以直接使用，也可用从官方获取。
### 1.1 下载源代码

官方GitHub：

[https://github.com/Caesar-github/u-boot](https://github.com/Caesar-github/u-boot) next-dev分支

[https://github.com/Caesar-github/rkbin](https://github.com/Caesar-github/rkbin) master分支

野火源码仓库下载链接：

[https://github.com/LubanCat](https://github.com/LubanCat)
### 1.2 下载指定分支

通常一个U-boot仓库往往维护着不同分支的U-boot，进入仓库目录下可通过命令查看及切换U-boot分支
```shell
git clone --branch=main https://github.com/LubanCat/u-boot.git
git clone --branch=master https://github.com/LubanCat/rkbin.git
```
## 2 U-boot编译（Chip）

在LubanCat-SDK中，自动编译脚本基本上都存放在build.sh中，这是SDK的主要功能入口

**❗️注意：** 以下对build.sh脚本中内容的描述仅适用于LubanCat_Chip_SDK。LubanCat_Gen_SDK编译流程相同，但具体内容有所变化，具体可查看device/rockchip/common/scripts/mk-loader.sh

在build.sh中，U-boot的构建函数是function build_uboot() 具体内容如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/fd67ce8e594cf2761c10cb2e5c06fa53.png)

- check_config检查是否有 RK_UBOOT_DEFCONFIG 这个配置变量，定义了U-boot编译用到的配置文件， 这个变量定义在`device/rockchip/rk356x/BoardConfig-*.mk`中

- build_check_cross_compilep判断是否存在arm-none-linux-gnueabihf或aarch64-none-linux-gnu交叉编译工具链， 我们没有使用，这一部分可以忽略。

- prebuild_uboot根据`BoardConfig-*.mk`中的定义来设定编译参数UBOOT_COMPILE_COMMANDS

- cd进入u-boot目录，先清除以前生成的miniloader文件，再判断uboot前一阶段是否为SPL， 如果是的话还要清除spl文件

- 判断RK_RAMDISK_SECURITY_BOOTUP值是否为true，是否使用secure boot来对镜像进行签名认证， 我们没有定义，直接跳过

- `RK_UBOOT_DEFCONFIG_FRAGMENT`：由于RK版本U-boot支持配置文件的overlay功能， 这里判断是否开启了该功能，如果开启的话先将配置文件合并，我们没有使用到

- 判断有没有arm-none-linux-gnueabihf或aarch64-none-linux-gnu来进行编译,没有使用，可以忽略。

- 没有找到上面两个编译器，向`./make.sh`传递构建参数进行构建U-boot操作

- RK_IDBLOCK_UPDATE_SPL和RK_RAMDISK_SECURITY_BOOTUP都没有定义，直接跳过

- finish_build编译完成的一个提示函数。
## 3 U-boot修改

一般来说，要适配一块新的板卡，在U-boot中主要设计三部分的修改和添加，分别是
- board目录下涉及`板卡初始化`的部分
- configs目录下对应`板卡的配置文件`
- arch目录下与`板级外设相关`的文件

得益于Rockchip U-boot对rk系列处理器的完善支持，在U-boot阶段，我们直接根据使用的处理器型号选择Rockchip官方参考板的配置即可。 例如rk356x处理器使用evb_rk3568，rk3588则使用evb_rk3588。

野火仅在U-boot中添加了设备树插件功能的实现。并使用boot_scripts方式进行启动引导，用于实现extboot分区的启动。

boot_scripts启动方式的启动命令脚本在SDK目录下的kernel/arch/arm64/boot/dts/rockchip/uEnv/boot.cmd， 在构建boot分区时进行编译，并打包到extboot分区内，命名为boot.scr，在系统启动前被U-boot读取并加载。 具体流程请查看上一章节 [5 U-boot启动内核过程](05_U-boot的介绍.md#5%20U-boot启动内核过程)
### 3.1 U-boot配置文件

我们编译使用的配置文件由`BoardConfig-*.mk`的RK_UBOOT_DEFCONFIG变量定义， 在LubanCat-RK系列板卡上使用的具体文件是u-boot/configs/rk3568_defconfig和rk3566.config， 这里以LubanCat-2为例，如果我们要修改的话，可以借助menuconfig工具。

首先我们要来到U-boot根目录下，然后执行以下操作:
```shell
# 应用配置文件
make rk3568_defconfig

# 使用menuconfig来管理修改配置文件
make menuconfig
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4a2eeb20d261edb193fc17541ae06fcd.png)

修改完成之后按ESC按键退出，并保存
```shell
# 保存defconfig文件
make savedefconfig

# 覆盖原来的配置文件
cp defconfig configs/rk3568_defconfig
```
### 3.2 设备树文件

设备树文件是板级设备的描述文件，系统通过设备树文件得知板卡上有哪些外设，从而加载相应的驱动使外设正常工作。

原生的U-Boot只支持使用U-Boot自己的DTB，RK平台增加了kernel DTB机制的支持， 即使用kernel DTB去初始化外设。主要目的是为了兼容外设板级差异，如：power、clock、display 等。

二者的作用：
- U-Boot DTB：负责初始化`存储、打印串口`等设备；
- Kernel DTB：负责初始化存储、打印串口以外的设备；

U-Boot初始化时先用U-Boot DTB完成存储、打印串口初始化， 然后从存储上加载Kernel DTB并转而使用这份DTB继续初始化其余外设。

开发者一般不需要修改 U-Boot DTB（除非更换打印串口），RK系列处理器使用的defconfig都已启用kernel DTB机制。 所以通常对于外设的DTS修改，用户应该修改kernel DTB。

关于U-Boot DTB的DTS目录：
- u-boot/arch/arm/dts/rk3568-evb.dts
- u-boot/arch/arm/dts/rk3588-evb.dts

启用kernel DTB机制后：编译阶段会把U-Boot DTS里带 u-boot,dm-pre-reloc 和 u-boot,dm-spl 属性的节点过滤出来， 在此基础上再剔除defconfig中 CONFIG_OF_SPL_REMOVE_PROPS 指定的property， 最终生成u-boot.dtb文件并且追加在u-boot.bin的末尾
## 4 参考资料

RK U-boot添加了很多自定义的功能，但其主体还是基于官方U-boot的，所以关于U-boot的操作都是基本相同的，我们可以参考以下wiki。

官方uboot下载链接: [http://www.denx.de/wiki/uboot/WebHome](http://www.denx.de/wiki/uboot/WebHome)

官方uboot GitHub： [https://github.com/uboot/uboot](https://github.com/uboot/uboot)

uboot wiki: [http://www.denx.de/wiki/uboot/WebHome](http://www.denx.de/wiki/uboot/WebHome)

RK U-boot的自定义功能则可以详细查看Rockchip文档 [《U-Boot v2017(next-dev) 开发指南》](https://github.com/Caesar-github/docs/blob/master/Common/UBOOT/Rockchip_Developer_Guide_UBoot_Nextdev_CN.pdf)