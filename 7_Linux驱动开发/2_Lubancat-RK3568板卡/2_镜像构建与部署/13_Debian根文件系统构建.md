
借助LubanCat-SDK我们可以方便的一键构建Debian镜像，但是**Debian根文件系统的构建过程是相对独立的，不依赖SDK的其他部分**。

Debian根文件系统借助live build来进行构建，live build是一组用于构建实时系统映像的脚本。 live build是⼀个⼯具套件，它使⽤⼀个配置目录来完全⾃动化和定制构建live镜像的所有⽅⾯。

除了live build外，我们的Debian构建仓库还有很多构建脚本，可以最大化的减少人工操作，使构建的根文件系统具有一致性。
## 1 Debian系统支持情况

**我们只关注LubanCat_Linux_rk356x_SDK** 包，其仅支持Debian 10 “Buster”。
- lite无桌面版本
- xfce4桌面版本
- xfce4-full桌面版本
## 2 什么是Debian

![image.png|100](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f2112a5ced6cfc7528f111e8a2ebacfc.png)

Debian GNU/Linux(简称Debian),是目前世界最大的非商业性Linux发行版之一, 是⼀种**完全⾃由开放并⼴泛⽤于各种设备的 Linux 操作系统**。

Debian的特点：
- 面向用户：
	- Debian 是自由软件，每個人都能自由使用、修改，以及分发。
	- 稳定且安全。
	- 广泛的硬件支持。
	- 提供平滑的更新。
	- 是许多其他发行版的基础。
	- Debian项目是一個社区。
- 面向开发者
	- 支持多种硬件架构，包括 AMD64、i386，ARM和MIPS等
	- 支持物联网和嵌入式设备
	- 拥有大量的套件，使用deb格式
	- 不同的发布版本
	- 公开的错误跟踪系统

Debian官网：[https://www.debian.org/](https://www.debian.org/)
## 3 Debian根文件系统构建仓库

LubanCat-SDK中就已经包含了完整的Debian根文件系统构建项目，保存在SDK的debian目录下。

进入debian目录下，有以下文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9ccdeb4a1dbe6cec78e16d968e6a1651.png)

- `mk-base-debian.sh`:清理构建目录并调用live build开始构建。
- `mk-buster-rootfs.sh`：添加Rockchip overlay层。
- `mk-image.sh`：将根文件系统打包成img镜像文件
- `overlay`：Rockchip overlay层，主要是**rootfs中的配置文件**
- `overlay-debug`：Rockchip overlay层，主要是**debug脚本和工具**
- `overlay-firmware`：Rockchip overlay层，主要是**wifi/bt/npu的固件**
- `packages`：硬件加速包
- `ubuntu-build-service`：用于搭建构建环境的依赖文件和live build配置文件

目前构建脚本支持三种版本的镜像构建
- lite：无桌面,终端版
- xfce：使用xfce套件的桌面版
- xfce-full：使用xfce套件+更多推荐软件包的桌面版
## 4 Debian根文件系统构建流程

**Debian根文件系统的构建主要分为三个步骤**

`第一步`：使用live build工具构建lite-debian或xfce-debian**根文件系统**。使用mk-base-debian.sh脚本实现。

`第二步`：**添加基于RK处理器进行功能增强的软件包**如GPU驱动和硬件firmware等。使用mk-buster-rootfs.sh脚本实现。

`第三步`：将构建完成的**根文件系统打包成img格式**，方便烧录和下一步处理。通过mk-image.sh脚本实现，在第二步完成后自动调用。
## 5 搭建构建环境

**💡提示：** 在debian目录下执行以下命令
```shell
sudo apt-get install binfmt-support qemu-user-static
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f
```

上面的命令执行过程中可能有警告或报错，这是正常现象，我们直接忽略报错即可。
## 6 构建Debian根文件系统镜像

在ubuntu-build-service目录下，根据lite或xfce版本，armhf或arm64架构的不同， 已经保存了live build的一些预设文件，如软件包列表、用户名、密码、用户组、时区等配置。

我们使用mk-base-debian.sh脚本来调用live build构建相应的Debian根文件系统。

理论上生成的根文件系统已经能在我们的板卡上运行了，不过还没有添加针对板卡的配置， 如网络，显示等，只能运行核心的服务。

下面我们来看一下具体的构建过程：
### 6.1 构建debian-base基础根文件系统

我们在debian目录下运行下面的命令
```shell
./mk-base-debian.sh
```

选择要构建的Debian版本,这里我们选择xfce版本，输入2并按下Enter按键，根据提示输入用户密码
```shell
---------------------------------------------------------
please enter TARGET version number:
请输入要构建的根文件系统版本:
[0] Exit Menu
[1] lite
[2] xfce
[3] xfce-full
---------------------------------------------------------
2
```

整个构建时间较长，等待命令结束以后我们看一下debian目录下的文件变化：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0c493c7430c753dd6c19e86371373b7b.png)

新增的文件 `linaro-buster-xfce-alip-20250417.tar.gz` 就是刚刚通过live build构建的基础debian根文件系统的压缩包
 
### 6.2 构建完整的debian根文件系统

通过上一步骤构建的根文件系统已经可以在板卡上运行了，为了进一步优化在LubanCat上运行的效果， 我们还要**添加Rockchip overlay层**，里面主要是一些配置文件和固件，用于添加或覆盖根文件系统中原有的配置文件。
```shell
./mk-buster-rootfs.sh
```

选择要构建的Debian版本,这里我们选择xfce版本，输入2并按下Enter按键，根据提示输入用户密码
```shell
---------------------------------------------------------
please enter TARGET version number:
请输入要构建的根文件系统版本:
[0] Exit Menu
[1] lite
[2] xfce
[3] xfce-full
---------------------------------------------------------
2
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/67229d48cfd8797b33c6a9ab23539797.png)

构建完成后，在debian目录下增加了`binary目录`。里面存放的是解压后的根文件系统， 我们将要覆盖或添加的文件复制进去，然后通过chroot命令进行修改。
### 6.3 打包debian-lite根文件系统镜像

在脚本./mk-buster-rootfs.sh的最后，自动调用了 IMAGE_VERSION=$TARGET ./mk-image.sh 脚本来打包镜像, TARGET就是我们在运行脚本时选择的Debian版本。

脚本运行完成后，我们就得到了名为linaro-xfce-rootfs.img的根文件系统镜像文件。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3c3a56f52f70de353a66776025ff907a.png)

## 7 定制Debian根文件系统

由于镜像体积的限制，我们提供的定制Debian镜像预装了一部分常用软件， 但是在用户开发时可能还需要预装更多的软件，以及对根文件系统做进一步的定制。 以下部分将对根文件系统的修改做具体说明。
### 7.1 添加预装软件包

对于预装软件的添加，我们**建议放在mk-buster-rootfs.sh脚本中**， 这样在我们做修改以后，只要重复添加Rockchip overlay层和打包img镜像的过程即可， 可以节约大量的开发时间

比如我们想要预装git和vim到根文件系统中， 则可以在mk-buster-rootfs.sh添加以下内容
```shell
export APT_INSTALL="apt-get install -fy --allow-downgrades"
#添加的位置在 export APT_INSTALL 下一行

#添加的内容是
echo -e "\033[47;36m ---------- LubanCat -------- \033[0m"
\${APT_INSTALL} git vim
```

下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1e014b7635e494abd3e0b6d183aff018.png)

### 7.2 添加外设firmware

如果我们使用无线网卡这样的外设，就需要向根文件系统中添加网卡的firmware， 这时直接将对应的firmware存放在overlay-firmware/目录下，按根文件系统中的路径保存
### 7.3 添加服务项及配置文件

我们希望对有些服务项的配置进行自定义，就可以在overlay/目录下添加对应的配置文件。 制作根文件系统的过程中，在添加Rockchip overlay层的时候，就会添加或替换根文件系统中原有的配置文件， 以实现对配置文件自定义的效果。

这里我们以Debian控制台登录时的banner配置为例，他的配置文件在根文件系统的/etc/update-motd.d目录下， 这里对应的是overlay/etc/update-motd.d/目录
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c67d42dbe6d53605bbda45ea4b81dbaa.png)

我们在overlay/etc/update-motd.d/中新建一个名为00-header的文件，在文件中添加以下内容：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e53dc51238cd5dbf8558c30094f034ac.png)

这个脚本的作用是生成动态的banner。

在添加配置文件以后，我们重新构建镜像，再烧录到板卡启动，就可以达到我们想要的banner效果。

**❗️注意：** 如果添加的是shell脚本，需要在创建文件后修改文件权限为775，否则在根文件系统中可能无法执行。
### 7.4 重新打包根文件系统镜像

在对根文件系统的构建脚本做出修改之后，我们要重新打包debian-xfce/lite-rootfs.img镜像
- 如果我们没有修改live build的配置文件，则不需要重复构建基础镜像部分
- 如果修改了配置文件，则需重新构建，要手动删除基础镜像压缩包，然后使用mk-base-debian.sh脚本构建
```shell
# 删除基础镜像压缩包
rm linaro-*-alip-*.tar.gz

#构建基础镜像
mk-base-debian.sh
```
- 如果修改了mk-buster-rootfs.sh、overlay、overlay-debug、overlay-firmware、packages的内容，则需要执行
```shell
#构建完整的根文件系统镜像
./mk-buster-rootfs.sh
```
## 8 使用SDK一键构建

在完成 **搭建构建环境** 以后，我们也可以直接使用一键构建命令， 就可以构建我们提供的定制根文件系统镜像了， 还可以借助SDK的镜像打包功能，将U-boot、内核等部分也一并打包成一个完整的系统镜像
### 8.1 SDK配置文件说明（Chip）

LubanCat板卡的SDK配置文件存放在device/rockchip/rk356x/目录内，以BoardConfig-LubanCat-CPU型号-系统类型-系统版本.mk来命名
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/36ba3dcbc259994442d4b26b11cb08f1.png)

我们来看Debian根文件系统的配置文件，这里以BoardConfig-LubanCat-RK3568-debian-xfce.mk为例， 主要说明与Debian根文件相关的设置。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/de82a7e56d3a0066932bad03157694d5.png)

- RK_SOC:定义SOC类型,在./mk-buster-rootfs.sh中使用
- RK_PKG_NAME：定义镜像发布打包时的名称
- RK_ROOTFS_SYSTEM：定义根文件系统类型
- RK_DEBIAN_VERSION：定义Debian发行版，目前只支持buster
- RK_ROOTFS_TARGET：定义根文件系统版本
- RK_ROOTFS_DEBUG： 是否添加rockchip overlay_debug
### 8.2 build.sh中的自动构建脚本

Debian根文件系统的一键构建功能，主要由build.sh脚本中的以下函数实现

**❗️注意：** 以下内容中对function build_debian()的描述仅适用于LubanCat_Chip_SDK。在LubanCat_Gen_SDK中编译流程基本相同，具体内容可以查看device/rockchip/common/scripts/mk-rootfs.sh

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/991726f4d432faa2b8f891b3e53769f9.png)

其工作流程如下
- 使用echo命令打印相关配置信息。
- 判断linaro-$RK_ROOTFS_TARGET-rootfs.img是否存在( **$RK_ROOTFS_TARGET** 为配置文件中定义的是否为桌面版)，存在则跳过构建过程，不存在则运行构建命令。根文件系统构建时间很长，不希望频繁构建根文件系统或使用已经构建好的根文件系统镜像。
- 判断构建基础镜像是否存在，不存在则构建基础镜像，若存在则跳过基础镜像构建过程。除了首次构建外，我们一般不会去修改基础镜像。
- 以基础镜像为基础，添加rockchip overlay。
- 打包完整镜像。
### 8.3 编译前的准备工作

在编译开始前，首先要确保SDK的配置文件与要编译的rootfs一致， 如果当前配置文件与要编译的rootfs不一致，需要先切换配置文件
```shell
# 选择SDK配置文件 LubanCat_Chip_SDK直接指定
./build.sh BoardConfig-xxx-debian-版本.mk

# 选择SDK配置文件 LubanCat_Gen_SDK直接指定
./build.sh LubanCat_xxx_debian_版本_defconfig

# 选择SDK配置文件-按序号选择
./build.sh lunch
```

在设置正确的配置文件后，我们就可以进行下一步的构建工作了
### 8.4 单独构建rootfs并打包

我们使用以下命令构建Debian根文件系统，
```shell
# 构建Debian
./build.sh debian
```

编译生成的rootfs镜像为linaro-$RK_ROOTFS_TARGET-rootfs.img，同时被软链接到rockdev/rootfs.ext4
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b3ac61b42c386470f5850713648dbcee.png)

构建完成后就可以将独立的分区表、boot.img、uboot.img等分区镜像打包成一个完整的镜像了

在执行以下操作前，请确保已经事先完成了U-Boot编译、Kernel编译、以及刚刚完成的单独构建rootfs的过程。

确保无误后，执行以下命令
```shell
# 固件打包
./mkfirmware.sh

# 生成update.img
./build.sh updateimg
```

打包完成后，生成的镜像为rockdev/update.img，我们可以使用烧录工具将update.img烧录到板卡eMMC中或SD卡中。
### 8.5 一键构建完整镜像文件

在设置正确的配置文件后,执行以下命令就可以一键完成U-Boot、Kernel和rootfs的编译，并生成update.img镜像
```shell
# 一键编译
./build.sh
```

打包完成后，生成的镜像为rockdev/update.img，我们可以使用烧录工具将update.img烧录到板卡eMMC中或SD卡中。