
**注意1：** LubanCat_Chip_SDK将于2024年9月10日起逐步停止支持，rk356x板卡和rk3588板卡后续开发支持已经转移到LubanCat_Gen_SDK

**注意2：** LubanCat专用镜像已于2023年4月11日停止支持，部分与专用镜像相关的内容并未删除，这些内容仅供参考，同时本文档在相关位置也会添加 **已停止支持** 的标记

**注意3：** LubanCat Buildroot相关内容已于2023年8月3日停止支持，部分与Buildroot构建相关的内容并未删除，这些内容仅供参考，同时本文档在相关位置也会添加 **已停止支持** 的标记
## 1 简介

LubanCat_Chip_SDK是基于瑞芯微通用Linux SDK工程的深度定制版本，适用于以Rockchip处理器为主芯片的LubanCat-RK系列板卡。

整个SDK工程，目录包含有 buildroot、debian、app、kernel、u-boot、device、external 等目录。 每个目录或其子目录会对应一个 git 工程，提交需要在各自的目录下进行。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d38e5ba31431c5167e77d42c9df2e615.png)

- `app`：(**已停止支持**) 存放上层应用 app，主要是 qcamera/qfm/qplayer/settings 等一些应用程序，在buildroot构建时会使用到这些程序。
- `buildroot`： (**已停止支持**) 基于 buildroot (2018.02-rc3) 开发的根文件系统。
- `debian`：Debian根文件系统构建脚本。
- `device/rockchip`： 存放各芯片板级配置和Parameter文件，以及一些构建与打包固件的脚本和预备文件。
- `external`：(**已停止支持**) 存放第三方相关仓库,包括音频、视频、网络、recovery 等，在buildroot构建时将会用到。
- `kernel`： 存放 kernel 源码。
- `prebuilts`： 存放**交叉编译工具链**。
- `rkbin`： 存放 Rockchip 相关的 Binary 和工具。
- `rockdev`： 存放编译输出固件。
- `tools`： 存放 Linux 和 Windows 操作系统环境下常用工具。
- `u-boot`：存放基于v2017.09版本进行开发的uboot代码。
- `ubuntu`: Ubuntu根文件系统构建脚本
## 2 extboot与rkboot分区对比

**注意1：** 由于基于rkboot分区的LubanCat专用镜像已停止支持，相关SDK配置文件已移动至 device/rockchip/rk356x/.others 目录下，以下相关内容仅做参考。
### 2.1 extboot分区系统镜像特点

extboot分区系统是野火基于瑞芯微Linux_SDK框架搭建的一种LubanCat-RK系列板卡通用镜像实现方式。实现一个镜像烧录到Lubancat使用相同处理器的所有板卡，解决了默认rkboot分区固定设备树导致一个镜像只能失配一款板卡的问题，大大降低了型号众多导致的后期维护复杂性。

extboot分区使用ext4文件系统格式，在编译过程中将所有LubanCat-RK系列板卡设备树都编译并打包到分区内， 并借助SDRADC读取板卡硬件ID，来实现设备树自动切换。同时支持设备树插件，自动更新内核deb包，在线更新内核和驱动模块等功能。

还对系统镜像的分区做了调整，删减一些冗余功能，仅包含uboot、boot、rootfs分区，实现了对系统存储的高效利用。
### 2.2 rkboot分区系统镜像特点（停止支持）

rkboot分区是使用瑞芯微Linux_SDK自带的boot分区打包脚本所构建的分区，仅修改了kernel配置文件和适配板卡设备树，系统更加原汁原味。 使用此类型boot分区的优点是启动快，冗余功能强，进一步配置可以实现OTA，多系统，镜像校验等功能(需二次开发自行适配)，但是像设备树插件这些功能是缺失的。

rkboot分区的系统镜像，一个型号的板卡要对应一个型号的系统镜像，如果硬件发生改变要修改设备树，就要重新构建镜像。
### 2.3 对比

|   |   |   |
|---|---|---|
|功能|extboot|rkboot|
|启动速度|稍慢|快|
|使用相同处理器板卡镜像通用|支持|不支持|
|系统内切换设备树|支持|不支持|
|设备树插件|支持|不支持|
|存储空间利用率|较高|较低|
|在线升级内核|支持|不支持|

对于两种类型分区的系统镜像，其uboot做了通用处理，rootfs也做了通用处理，所以**uboot.img和rootfs.img是可以通用的**
## 3 SDK 开发环境搭建

LubanCat_Chip_SDK是基于Ubuntu LTS 系统开发测试的，在开发过程中，主要是用Ubuntu 20.04版本， 为了不必要的麻烦，推荐用户使用Ubuntu20.04及以上版本

**注意：** LubanCat-RK3588系列SDK不支持Ubuntu20.04以下版本环境进行镜像构建

安装SDK依赖的软件包
```shell
# 安装SDK构建所需要的软件包
# 整体复制下面内容到终端中安装
sudo apt install git ssh make gcc libssl-dev liblz4-tool u-boot-tools curl \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python3-pip libncurses5-dev python3-pyelftools dpkg-dev
```
## 4 安装repo

```shell
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
# 如果上面的地址无法访问，可以用下面的：
# curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
echo PATH=~/bin:$PATH >> ~/.bashrc
source ~/.bashrc
```

执行完上面的命令后来验证repo是否安装成功能正常运行
```shell
repo --version

#返回以下信息
#返回的信息根据Ubuntu版本的不同略有差异
<repo not installed>
repo launcher version 2.32
    (from /home/he/bin/repo)
git 2.25.1
Python 3.8.10 (default, Nov 14 2022, 12:59:47)
[GCC 9.4.0]
OS Linux 5.15.0-60-generic (#66~20.04.1-Ubuntu SMP Wed Jan 25 09:41:30 UTC 2023)
CPU x86_64 (x86_64)
Bug reports: https://bugs.chromium.org/p/gerrit/issues/entry?template=Repo+tool+issue
```
## 5 SDK源码获取（在线获取）

LubanCat_Chip_SDK的代码被划分为了若干git仓库分别进行版本管理， 可以使用repo工具对这些git仓库进行统一的下载，提交，切换分支等操作。

运行以下命令，将在当前用户的家目录下创建一个名为LubanCat_SDK的目录，用来放入SDK源码
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2afead854fef37648f20ee435ed9beaa.png)
### 5.1 切换PY3版本

```shell
#查看当前Python版本
python -V
```

若返回的版本号为Python3版本，则无需再切换Python版本。若为Python2版本或未发现python，则可以用以下方式切换：
```shell
#查看当前系统安装的Python版本有哪些
ls /usr/bin/python*

#将python链接到python3
sudo ln -sf /usr/bin/python3 /usr/bin/python

#重新查看默认Python版本
python -V
```

此时系统默认Python版本切换为python3
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cc61ea4588a648337324da283286ebb4.png)
### 5.2 Git配置

设置自己的git信息，以确保后续拉取代码时正常进行，如果不需要提交代码的话可以随意设置用户名和邮箱地址
```shell
git config --global user.name "your name"
git config --global user.email "your mail"
```
### 5.3 SDK在线下载并同步

LubanCat_Chip_SDK可以适配不同的板卡。由于使用这种方式要从Github拉取大量的仓库， 体积很大，对于不能快速访问Github的用户不建议使用这种方式

```shell
cd ~/LubanCat_SDK

# 拉取LubanCat-RK356x系列Linux_SDK
repo init -u https://github.com/LubanCat/manifests.git -b linux -m rk356x_linux_release.xml

# 拉取LubanCat-RK3588系列Linux_SDK
repo init -u https://github.com/LubanCat/manifests.git -b linux -m rk3588_linux_release.xml

#如果运行以上命令失败，提示：fatal: Cannot get https://gerrit.googlesource.com/git-repo/clone.bundle
#则可以在以上命令中添加选项 --repo-url https://mirrors.tuna.tsinghua.edu.cn/git/git-repo

.repo/repo/repo sync -c -j4
```

如果同步失败可以重新运行sync命令来同步
### 5.4 SDK离线安装下载

由于Github服务器在国外，拉取这么多的仓库需要很多时间，还可能因为网络不畅通而导致下载失败。 为此，将需要使用的仓库整体打包，使用网盘下载的方式，以减少连接Github导致的问题。

**下载地址**
- 访问百度网盘资源介绍页面获取SDK源码压缩包: [8-SDK源码压缩包](https://doc.embedfire.com/linux/rk356x/quick_start/zh/latest/quick_start/baidu_cloud/baidu_cloud.html#id1)
- 根据鲁班猫板卡的型号下载对应版本的最近日期的压缩包即可
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5948558adaafafa5e0f5851e73acd52b.png)
**注意：** 在开发过程中，当SDK源码进入Release版本或有重大Bug时，将会更新源码压缩包。源码压缩包解压到本地后，当有Release版本更新时，可以借助Github更新到最新版本。

**解压源码：**
- 以下过程以LubanCat_Linux_rk356x_SDK进行演示，实际文件名称以自己下载的SDK为准
```shell
# 安装tar压缩工具,一般来说系统默认安装了
sudo apt install tar

# 在用户家目录创建LubanCat_SDK目录
mkdir ~/LubanCat_SDK

# 将下载的SDK源码移动到LubanCat_SDK目录下，xxx为日期
mv LubanCat_Linux_rk356x_SDK_xxx.tgz ~/LubanCat_SDK

# 进入LubanCat_SDK目录
cd ~/LubanCat_SDK

# 解压SDK压缩包
tar -xzvf LubanCat_Linux_rk356x_SDK_xxx.tgz

# 检出.repo目录下的git仓库
.repo/repo/repo sync -l

# 将所有的源码仓库同步到最新版本
.repo/repo/repo sync -c
```

- 如果repo sync -c执行时提示网络连接超时，请检查并能否通畅访问github。 确认可以正常访问github的话，可以重复多次执行repo sync -c命令来进行同步。 若无法访问github，可以忽略同步源码仓库到最新版本这一步骤。
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/776e98a2841a2550b7f7d04f8a53ef98.png)
- 解压完成后checkout到指定的提交。
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7e27699e2c242f2cb7240173173427c0.png)
- 一般情况下网盘保存的离线源码包已经是最新版本，如果距离离线源码包下载时间不久，可以忽略从Github在线更新这一步。
### 5.5 SDK更新

我们会对LubanCat_Chip_SDK不断更新，并将修改的内容实时同步到Github， 如果需要在本地LubanCat_Chip_SDK同步更新内容，则可以借助repo或git来实现。

**方式一：使用repo更新整个SDK**
- 首先要更新.repo/manifests，里面保存了repo的配置文件，记录了仓库的版本信息。
```shell
# 进入.repo/manifests目录
cd .repo/manifests

# 切换分支到Linux
git checkout linux

# 拉取最新的manifests
git pull

#返回SDK根目录
cd ~/LubanCat_SDK

# 同步远端仓库
.repo/repo/repo sync -c
```

**方式二：使用git更新单独的源码仓库**
- 有时只想更新某个仓库，而不是去更新整个SDK。 或者已经对SDK的某些仓库做出了修改，使用repo同步的话就会失败。 此时就需要对单个仓库进行更新了。
- 这里以Kernel仓库为例
```shell
# 进入kernel目录下
cd kernel

# 检出到当前提交所在的分支
# 可以查看.repo/manifests/rk356x_linux_release.xml
# name="kernel"条目中dest-branch就是要切换的分支
git checkout stable-4.19-rk356x

# 拉取git仓库
git pull
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/32feb3db0a698ca4a869fd72edbf94be.png)

**注意：** 不同SDK或板卡使用的分支不同，详细分支支持情况请查看 **Linux内核的编译** 章节
## 6 自动构建

LubanCat板卡对应的配置文件列表:

|   |   |   |   |
|---|---|---|---|
|镜像说明|boot分区类似|文件系统|对应BoardConfig配置文件|
|LubanCat-RK3566系列通用镜像|extboot分区|debian|BoardConfig-LubanCat-RK3566-debian-(版本).mk|
|LubanCat-RK3566系列通用镜像|extboot分区|ubuntu|BoardConfig-LubanCat-RK3566-ubuntu-(版本).mk|
|LubanCat-RK3568系列通用镜像|extboot分区|debian|BoardConfig-LubanCat-RK3568-debian-(版本).mk|
|LubanCat-RK3568系列通用镜像|extboot分区|ubuntu|BoardConfig-LubanCat-RK3568-ubuntu-(版本).mk|

- LubanCat-RK3566通用镜像适用于LubanCat-1系列和LubanCat-0系列全部板卡
- LubanCat-RK3568通用镜像适用于LubanCat-2系列全部板卡

LubanCat_Chip_SDK的构建脚本可以实现自动构建，具体操作方式如下：
- 在SDK根目录下，执行以下命令，以`选择SDK配置文件，来选择要构建的板卡和文件系统类型`
```shell
./build.sh lunch
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/51c9c21c5dae572df6a6c9e3359f1ccb.png)

- 也可以直接设置SDK配置文件，方法如下
```shell
# 选择构建LubanCat-RK3566系列板卡通用debian镜像
./build.sh BoardConfig-LubanCat-RK3568-debian-xfce.mk
```

选择SDK配置文件以后，还要安装debian根文件系统构建依赖的软件包，操作如下。
```shell
#安装本地软件包
sudo dpkg -i debian/ubuntu-build-service/packages/*
sudo apt-get install -f
```

**注意：** 由于debian和ubuntu根文件系统不同版本构建所需的依赖包版本不同，当切换了根文件系统版本的选择以后，就需要根据自己所选的根文件系统版本安装不同的依赖包。

如果选择了ubuntu根文件系统的SDK配置文件，就需要安装ubuntu根文件系统构建的依赖软件包，操作如下。
```shell
#安装本地软件包
sudo dpkg -i ubuntu/ubuntu-build-service/packages/*
sudo apt-get install -f
```

安装过程中可能会报错，这是正常现象，忽略即可。待软件包安装完成后，就可以进行一键构建了。
```shell
# 一键编译u-Boot，kernel，Rootfs并打包为update.img镜像
./build.sh
```

`警告：` 如果在编译过程中出现如下图所示的提示，请根据对应设备树文件中的&pmu_io_domains节点中的电压值选择， 并注意顺序从pmuio2-supply开始。其中vccio_acodec为3v3、vccio_sd为3v3
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0ccfc5e304a4ab34fc3928a85e2e4cf7.png)

构建好的镜像保存在rockdev/目录下，同时在也可以使用 **./build.sh save** 命令在IMAGE/目录下进行备份。
## 7 分步构建

在进行固件开发时，一键构建就显得过于耗时了，每一个改动都要构建整个镜像并重新打包， 这无疑是巨大的时间浪费。此时可以使用SDK单个模块构建的功能。
### 7.1 选择SDK配置文件

首先，还是要选择SDK的配置文件，这里以LubanCat2系列板卡的通用debian镜像为例
```shell
# 选择SDK配置文件
./build.sh BoardConfig-LubanCat-RK3568-debian-xfce.mk
```
### 7.2 U-Boot构建

```shell
./build.sh uboot
```

构建生成的U-boot镜像为`u-boot/uboot.img`
### 7.3 kernel构建

**extboot分区**

extboot分区内核镜像，使用的配置文件为BoardConfig-LubanCat-RK356x-xxx.mk

要先生成内核deb包，然后再编译内核并将生成的deb包打包进extboot分区。

执行以下命令自动完成 kernel 的构建及打包。
```shell
./build.sh kerneldeb

./build.sh extboot
```

构建生成的kernel镜像为`kernel/extboot.img`
### 7.4 rootfs构建

LubanCat主要支持Ubuntu、Debian、OpenWrt这几种rootfs， 不同rootfs的构建过程不同，这里分开说明。

**注意：** 由于OpenWrt系统的构建不使用LubanCat_Chip_SDK，所以不在此处说明，具体构建过程查看LubanCatWRT构建说明
#### 7.4.1 Debian

首先要确保SDK的配置文件与要构建的rootfs一致， 如果当前配置文件与要构建的rootfs不一致，需要先切换配置文件。
```shell
# 选择SDK配置文件
./build.sh BoardConfig-LubanCat-RK3568-debian-xfce.mk

# 构建Debian
./build.sh debian
```

构建生成的rootfs镜像为debian/linaro-xfce-rootfs.img，同时被软链接到rockdev/rootfs.ext4

**注意：** 只有不存在debian/linaro-xfce-rootfs.img时，才会重新构建Debian根文件系统。如果要重新构建linaro-xfce-rootfs.img，需要先手动删除。

**提示：** 由于构建不同版本的rootfs很容易遇到环境依赖问题，推荐不需要重新构建Debian根文件系统的用户直接下载提供的定制根文件系统镜像或使用Docker构建，详情请查看Debian根文件系统构建独立章节

由于从头构建根文件系统会从网络获取很多文件，并且耗费很多时间。不需要自行构建根文件系统的用户可以将发布的系统镜像中的rootfs分区提取出来直接使用。

rootfs分区提取需要使用解包工具，使用方法请查看本文档 **完整镜像的解包和打包** 章节。

提取出rootfs.img镜像以后，将rootfs.img镜像移动到SDK目录下的debian目录中，并根据解包的系统镜像的名称，重命名为linaro-(桌面版本)-rootfs.img，如linaro-xfce-rootfs.img。

重命名完成后，重新选择与根文件系统镜像一致的配置文件进行编译
```shell
# 选择SDK配置文件
./build.sh BoardConfig-LubanCat-RK3568-debian-xfce.mk

# 构建Debian
./build.sh debian

# 返回的提示信息：跳过镜像构建，删除rootfs镜像后以便重新构建Debian根文件系统镜像
[    Already Exists IMG,  Skip Make Debian Scripts    ]
[ Delate linaro-xfce-rootfs.img To Rebuild Debian IMG ]
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/62cb2fc0666fd24bafe3174d8b9cc6f3.png)


如果返回的信息与文中一致，则使用解包后的根文件系统镜像成功
#### 7.4.2 Ubuntu

目前提供Ubuntu18.04/20.04/22.04的根文件系统构建脚本，默认使用20.04版本分支，如要构建其他版本，在构建前要先拉取构建源码仓库并切换到对应分支，具体操作请查看Ubuntu根文件系统构建独立章节。

**注解：** 推荐使用Ubuntu20.04版本，这是主要支持版本。

首先要确保SDK的配置文件与要构建的rootfs一致， 如果当前配置文件与要构建的rootfs不一致，需要先切换配置文件。
```shell
# 选择SDK配置文件
./build.sh BoardConfig-LubanCat-RK3568-ubuntu-xfce.mk

# 构建Ubuntu
./build.sh ubuntu
```

构建生成的rootfs镜像为ubuntu/ubuntu-xfce-rootfs.img，同时被软链接到rockdev/rootfs.ext4

**✏️注意：** 只有不存在ubuntu/ubuntu-xfce-rootfs.img时，才会重新构建Ubuntu根文件系统。如果要重新构建ubuntu-xfce-rootfs.img，需要先手动删除。

**💡提示：** 由于构建不同版本的rootfs很容易遇到环境依赖问题，推荐不需要重新构建Ubuntu根文件系统的用户直接下载提供的定制根文件系统镜像或使用Docker构建，详情请查看Ubuntu根文件系统构建独立章节

由于从头构建根文件系统会从网络获取很多文件，并且耗费很多时间。不需要自行构建根文件系统的用户可以将发布的系统镜像中的rootfs分区提取出来直接使用。

rootfs分区提取需要使用解包工具，使用方法请查看本文档 **完整镜像的解包和打包** 章节。

提取出rootfs.img镜像以后，将rootfs.img镜像移动到SDK目录下的ubuntu目录中，并根据解包的系统镜像的名称，重命名为ubuntu-(桌面版本)-rootfs.img，如ubuntu-xfce-rootfs.img。

重命名完成后，重新选择与根文件系统镜像一致的配置文件进行编译
```shell
# 选择SDK配置文件
./build.sh BoardConfig-LubanCat-RK3568-ubuntu-xfce.mk

# 构建Ubuntu
./build.sh ubuntu

# 返回的提示信息：跳过镜像构建，删除rootfs镜像后以便重新构建Ubuntu根文件系统镜像
[    Already Exists IMG,  Skip Make Ubuntu Scripts    ]
[ Delate ubuntu-xfce-rootfs.img To Rebuild Ubuntu IMG ]
```

如果返回的信息与文中一致，则使用解包后的根文件系统镜像成功
### 7.5 镜像打包

当u-Boot，kernel，Rootfs都构建完成以后，需要再执行./mkfirmware.sh 进行固件打包， 主要是检查分区表文件是否存在，各个分区是否与分区表配置对应，并根据配置文件将所有的文件复制或链接到rockdev/内

为了方便镜像的发布，还可以将各个分立的分区打包成一个文件，打包好的文件就能用于烧录了。

```shell
# 固件打包
./mkfirmware.sh

# 生成update.img
./build.sh updateimg
```

以debian系统为例，rockdev/目录下的文件如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9873e38b057d4250843686c47a5c76a6.png)

## 8 SDK配置文件说明

如果说./build.sh 脚本是肌肉，完成SDK各部分的构建工作，那SDK的配置文件则是大脑，控制着肌肉的运作方式。

虽然看起来SDK的配置文件条目繁多，但在使用中，需要去修改的并不多。

这里以LubanCat-RK系列配置文件为例，其他板卡的配置文件也类似
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a70aaa7ebf4a8a2fced79478839c0755.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3bd8c9f1d9d9c8be355b8865ebd8425d.png)

- RK_ARCH：定义处理器架构，无需修改
- RK_UBOOT_DEFCONFIG：Uboot配置文件
- RK_KERNEL_DEFCONFIG：Kernel配置文件
- RK_KERNEL_DEFCONFIG_FRAGMENT：Kernel配置文件的叠加配置
- RK_KERNEL_DTS：内核设备树文件名
- RK_BOOT_IMG：boot分区镜像名称，无需修改
- RK_KERNEL_IMG：内核镜像所在位置：无需修改
- RK_PARAMETER：分区表名称，无需修改
- RK_CFG_BUILDROOT：Buildroot配置文件，无需修改
- RK_CFG_RECOVERY：revcovery分区构建的配置文件，无需修改
- RK_JOBS：编译线程数，可根据电脑配置酌情修改
- RK_TARGET_PRODUCT：目标芯片，无需修改
- RK_ROOTFS_TYPE：rootfs的格式，无需修改
- RK_ROOTFS_IMG：rootfs镜像的位置，无需修改
- RK_ROOTFS_SYSTEM：rootfs的类型，目前支持ubuntu、debian，无需修改
- RK_DEBIAN_VERSION：debian发行版本，暂不支持修改
- RK_UBUNTU_VERSION：Ubuntu发行版本，暂不支持修改
- RK_ROOTFS_TARGET：rootfs是否支持桌面显示，xfce:桌面版 lite：控制台版 xfce-full:桌面版+推荐软件包
- RK_ROOTFS_DEBUG：rootfs是否添加Debug工具，debug添加、none不添加
- RK_EXTBOOT：是否使用exboot内核分区
