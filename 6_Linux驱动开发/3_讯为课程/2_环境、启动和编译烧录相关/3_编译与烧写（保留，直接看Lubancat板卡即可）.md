## 1 Android编译和烧写（后续学习）

## 2 Linux编译和烧写

### 2.1 SDK包介绍

开发Linux少不了和uboot、kernel，rootfs打交道。一些低端处理器这些源码一般都是单独提供的。

但是随着处理器性能的提高，软件的依赖也越来越复杂，所以往往这些源码都是以SDk包的形式提供，SDk包里面有`uboot，kerne，rootfs以及库文件和其他厂家自已的文件`

我们用Lubancat-RK3568板卡提供的SDK包，可以看到鲁班猫提供的SDK源码文件中有uboot和kernel
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0e52dbab391f19f5eccaae7da9a7d122.png)

- build.sh：编译整个SDK中的源码
- device：设备相关配置文件，设备树，设备驱动源码或设备抽象层代码
- lubancat-bin：预编译好的二进制文件或工具
- mkfirmware.sh：用于制作固件镜像
- rkbin：Rockchip官方提供的各类固件镜像及烧录工具
- tools：各种开发工具、辅助脚本等
- ubuntu：基于Ubuntu的root文件系统（rootfs）
- debian：基于debian系统的rootfs文件
- kernel：Linux内核相关源码、配置文件、补丁、设备树源码等
- u-boot：uboot源码
- 等等

SDK包的出现，使得开发者更加容易进行开发，不过底层架构是变复杂了
### 2.2 编译Linux源码说明

瑞芯微官方的Linux源码里面只有buildroot、debian和yocto这三种文件系统，并没有其他的系
统，如Ubuntu，openwrt等等。所以Lubancat单独给RK3568适配了这些系统，统称为其他Linux系统固件，其他Linux系统固件和buildroot、debian和yocto系统使用的是相同的uboot源码和内核源码。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e9d86d8003035fc231058db08205a4cb.png)

**相同的uboot + 相同的kernel + 不同的文件系统 = xxx系统**

例如：我们常说的QT系统，本质就是把 `QT库` 加入到文件系统中，然后和uboot、kernel组合，称为QT系统

**所以，后面对于uboot和kernel，我们只需要学习一遍即可**

### 2.3 获取RK提供的Linux源码

直接在网盘下载对应的压缩包
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f2347b6fb4dc047857014198ffc89fac.png)

然后上传到Ubuntu系统的home目录下，并解压
- 注意：如果要编译yocto系统，需要将源码包拷贝到 home目录下的Linux目录下，没有此目录就创建一个。因为，编译yocto系统需要科学上网

解压完成后的目录如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9a64ede9d1ed05e6454046df11b3ef0c.png)

**文件解释如下：**
- `app`：存放**上层应用程序**的目录，主要是gcamera/gfm/qplayer/settings等一些应用程序。
- `build.sh`：编译用的脚本，使用方法后面会教。
- `device/rockchip`：存放每个平台的一些编译和打包固件的脚步和预备文件。
- `docs`：存放RK开发指导文件、平台支持列表、工具使用文档、Linux开发指南等。
- `envsetup.sh`：要修改文件系统时候要设置的环境脚本。
- `extermal`：存放相关的库，包括音频，视频等。
- `Makefile`：整个SDK包编译的Makefile
- `mkfirmware.sh`：固件打包使用的脚本，默认在当前路径下的rockdev目录。
- `prebuilts`：存放**交叉编译工具链**。
- `rkbin`：存放固件和工具。
- `rkflash.sh`：linux下的系统烧录脚本。
- `rockdev`：存放编译输出固件的目录（整个SDK包编译完成后就会创建）。
- `tools`：存放固件和工具的目录。

- `u-boot`：U-boot源码目录。
- `kernel`：kernel 源码。
- `yocto`：基于yoctogatesgarth3.2开发的根文件系统。
- `debian`：基于debian10开发的根文件系统。
- `buildroot`：基于buildroot（2018.02-rc3）开发的根文件系统
### 2.4 整编

**如果外接屏幕，需要修改相关设备树**
- 参考连接：[11. 屏幕与触摸 — 快速使用手册—基于LubanCat-RK356x系列板卡 文档](https://doc.embedfire.com/linux/rk356x/quick_start/zh/latest/quick_start/screen/screen.html#)

快速使用：
```shell
# 查看板卡链接的配置文件
ls -l /boot/uEnv/uEnv.txt
# Lubancat2板卡链接的是uEnvLubanCat2-V2.txt，直接cat
```

可以看到hdmi是关闭的状态
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1f0fc0ebe1f6f2edc57e47d0fc75ff7b.png)

可以根据屏幕进行选配，行首不带 `#` 即可完成屏幕开启操作，没有相关应用的话，直接填写即可
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e83059dcdea7cb42be8893c222548af4.png)

**查看一下编译脚本**：进入SDK主目录下，输入如下指令
```shell
./build -h
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/25f5d14ae6aac30a6008c9ebfa61dfe0.png)

- 可以看到上面的选项
	- all：全部编译
	- cleanall：编译清除
	- buildroot：编译默认的文件系统
	- 等等

所以全编的指令（全编和打包镜像）
```shell
./build
# 或者
./build all
```

当然也有其他方式具体如下
```shell
# 第一步：设置要编译的文件系统(选一个即可)
export RK_ROOTFS_SYSTEM=buildroot
export RK_ROOTFS_SYSTEM=yocto
export RK_ROOTFS_SYSTEM=debian
export RK_ROOTFS_SYSTEM=ubuntu
# 如果编译yocto，SDK源码一定要在home目录下的Linux子目录下

# 第二步：编译和打包镜像
./build.sh all
./build.sh firmware
./build.sh updateimg
```

编译成功后会在rockdev目录下生成update.img镜像
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/13310d07336e1bc54591a95d731a8d1e.png)

### 2.5 编译uboot

```shell
./build.sh uboot
```

产物在SDK下的u-boot子目录中，名为uboot.img
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/458f516ac3994a193e9f6693998e4101.png)

可以看到编译的信息
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/50ab6af6469033e4879aa0987885f89a.png)

**修改uboot后，如何编译uboot呢？**
```shell
# 进入uboot目录
cd u-boot/
# 设置平台为arm64
export ARCH=arm64
# 打开配置菜单
make menuconfig
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a7f37ea40918e2ba4c9d314581afff0a.png)

修改好的配置，会保存在 `.config` 这个文件中
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b95f9877b25ea433c0c0e635563ad113.png)

在编译uboot的时候，会使用默认的配置文件进行编译，默认的配置文件位于 `/configs/rk3568_defconfig`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1b30dba8d819ee56d618503face2d3b6.png)

所以在修改完uboot的配置文件需要使用命令 `cp .config configs/rk3568_defconfig` 保存修改配置到默认的配置文件，然后在进行uboot的编译

注意：最好备份一下，原来的默认uboot配置文件

### 2.6 单独编译内核/设备树（讯为）

#### 2.6.1 编译内核

在SDK目录下，输入 `./build.sh kernel` 命令对kernel镜像进行编译，编译完成后镜像文件 `kernel.img` 和 `resource.img` 会打包成 `boot.img` 放到kernel目录下，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/09660e83c21cd241be1908ddce1c2bba.png)

注意：在编译内核的时候，会将设备树一起编译

**可以看一下编译内核的打印信息**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1ae892c2eb4fb58c14fc6d9c624c96e9.png)


#### 2.6.2 修改内核如何编译

在编译内核的时候会使用默认的配置文件进行编译，默认配置文件在内核目录下，路径为：`/arch/arm64/configs/lubancat2_defconfig`

### 2.7 编译内核（lubancat）

**编译内核源码**
```shell
#清除之前生成的所有文件和配置
make mrproper

# 加载lubancat2_defconfig配置文件，rk356x系列均是该配置文件
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig

# 编译内核，指定平台，指定交叉编译工具，使用8线程进行编译，线程可根据电脑性能自行确定
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8
```

如果没有交叉编译工具，或者编译工具版本不匹配也可以使用Lubancat-SDK源码的gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu版本的编译工具链。

执行以下命令获取并配置编译工具的环境变量：

RK356x系列板卡用户执行以下命令:
```shell
#获取编译工具链
git clone https://github.com/LubanCat/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.git

#导出环境变量，需要根据实际指定编译工具链的绝对路径
export PATH=/root/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin:$PATH

#查看编译工具链，如果COLLECT_LTO_WRAPPER变量为指定的路径，即配置成功
aarch64-linux-gnu-gcc -v
```

以上配置为临时导出环境变量，打开其他终端或者重启都需要重新导出环境变量，如需永久保存需要将导出环境变量的命令写入~/.bashrc文件末尾，并执行source ~/.bashrc 重新加载配置。

SDK中也包含了交叉编译工具链，位置在SDK源码/prebuilts/gcc/linux-x86/aarch64/目录下，也可以使用以上方法导出环境变量
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6b17a78f3e2fcabdd53c186a9dad9561.png)

### 2.8 编译和加载内核驱动模块（lubancat）

#### 2.8.1 编译

内核模块加载到内核，可以将内核模块编译成单独的模块，在内核启动后由用户手动动态加载， 也可以将模块直接编译进内核，在内核启动时就自动加载。

测试一般是单独编译成内核模块，然后手动加载，方便调试，同时也节省时间。

野火提供了驱动教程的源码，可以执行以下命令获取：
```shell
# 从github获取
git clone https://github.com/LubanCat/lubancat_rk_code_storage

# 或者从gitee获取
git clone https://gitee.com/LubanCat/lubancat_rk_code_storage
```

获取到源码后，源码目录下的linux_driver文件夹就是存放驱动教程的例程文件，将其配套驱动程序代码放置到 `内核代码同级目录` ，原因是编译内核模块时， 驱动程序需要依赖编译好的Linux内核，驱动模块中的Makefile中指定了内核的路径，为方便使用例程，请放至同一目录结构下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cadf02af534e5253e477ce08e82f8466.png)

内核驱动模块对象所需的构建步骤和编译很复杂，它利用了linux内核构建系统的强大功能， 目前我们还不需要深入了解这部分知识，利用简单的Make工具就能编译出我们想要的内核驱动模块。 这里以编译hellomodule内核模块为例，使用命令进入hellomodule目录，然后使用make：
```shell
cd linux_driver/module/hellomodule/
make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3162f97d886c6b8ab059805629d4b20a.png)

**在module/hellomodule/下，新增hellomodule.ko文件，这就是编译生成的内核驱动模块。**

我们可以看一下 hellomodule目录下的Makefile文件
```makefile
KERNEL_DIR=../../../kernel/
ARCH=arm64
CROSS_COMPILE=aarch64-linux-gnu-
export  ARCH  CROSS_COMPILE

obj-m := hellomodule.o

all:
   $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules

.PHONE:clean

clean:
   $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
```

- **第1行：** 指定内核目录，根据自己编译内核时指定的输出目录，可以相对路径或者绝对路径，如果编译内核时没有指定特定输出目录，那么就将这个变量指定到内核源码的根目录。
- **第2行：** arm64体系结构.
- **第3行：** 指定交叉编译工具链，可以使用Lubancat-SDK的交叉编译工具gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu。
- **第4行：** 导出环境变量。
- **第6行：** 表示以模块进行编译。
- **第8行：** all只是个标号，可以自行定义，是make的默认执行目标。

- **第9行：** 
	- `$(MAKE)`的MAKE是Makefile中的宏变量，要引用宏变量要使用符号。这里实际上就是指向make程序，所以这里也可以把`$(MAKE)`换成make
	- `make -C`是make命令的一个选项，-C作用是changedirectory， -C dir 就是转到dir目录
	- `M=$(CURDIR)`：返回当前目录，这句话的意思是：当make执行默认的目标all时，
		- `-C(KVDIR)`指明跳转到内核源码目录下去执行那里的Makefile
		- `-C $(KERNEL_DIR)`指明跳转到内核源码目录下去执行那里的Makefile
		- `M=(CURDIR)`表示又返回到当前目录来执行当前的Makefile。

- **第11行：** clean 就是删除后面这些由make生成的文件。

#### 2.8.2 加载

编译好内核驱动模块，可以通过多种方式将hellomodule.ko拷贝到开发板，，我们可以使用 `NFS网络文件系统` 、 `scp命令` 、 `sftp命令` 等。

scp 命令用于 Linux 之间复制文件和目录，该命令基于ssh，需要搭建好ssh环境，scp命令格式如下：
```shell
scp local_file remote_username@remote_ip:remote_folder
```

**转移文件如下**
```shell
scp hellomodule.ko cat@192.168.31.69:/home/cat/
```

将hellomodule.ko发送到192.168.31.69的/home/cat/目录下，192.168.31.69是开发板ip，需根据实际而定，开发板用户名为cat， 输入yes，然后输入密码进行验证，等待传输完成，这个时候我们开发板就有了hellomodule.ko 这个文件。如果是在开发板进行本地编译则不需要再进行传输。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d057f8c9aa9fd6148a847b7335de0450.png)

安装卸载内核驱动模块使用insmod和rmmod命令：
```shell
#进入家目录
cd /home/cat/

# 加载内核模块
sudo insmod hellomodule.ko

#查看当前加载的内核模块
lsmod

# 卸载内核模块
sudo rmmod hellomodule.ko
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a66cc018394cbe43b71dac4aef0fe95d.png)

查看加载的内核模块，可以使用命令lsmod，其他信息也可以到/sys/module目录下查看，例如：加载成功hellomodule.ko模块后， 可以到/sys/module/hellomodule下查看。

### 2.9 编译和加载设备树（lubancat）

#### 2.9.1 编译

Linux 3.x 之后的版本引入了设备树（Device Tree）的概念和机制。设备树是一种用于描述硬件平台的静态数据结构，包含了有关硬件设备、总线、中断控制器等信息的详细描述。 后面我们写的驱动需要依赖设备树，所以在这里先介绍如何编译设备树、加载设备树。

**使用内核工具编译设备树**

在编译 Linux 内核时，会生成名为 dtc（Device Tree Compiler）的工具，该工具用于自动编译设备树源文件（.dts 或 .dtsi 文件）为二进制的设备树文件（.dtb 文件）。我们也可以使用命令 `sudo apt install device-tree-compiler` 下载dtc编译工具。 dtc工具使用示例如下：
```shell
# 编译 dts 为 dtb
内核目录/scripts/dtc/dtc -I dts -O dtb -o xxx.dtb xxx.dts

#也可以下载dtc工具进行编译
sudo apt install device-tree-compiler
dtc -I dts -O dtb -o xxx.dtb xxx.dts
```
- 内核使用dtc工具的命令大致如上所示，实际上设备树中有非常多的依赖关系，所以一般情况下， 设备树不仅仅只是通过一个dtc命令就能将编译出来的。以上为伪代码，仅供参考使用，了解即可。

**使用内核的构建脚本编译设备树**

我们可以尝试着通过内核的构建脚本去编译设备树，我们所要用到的设备树文件都存放在 `内核源码/arch/arm64/boot/dts/rockchip` 里面。

前面提到了`编译内核时会自动去编译设备树`，但是编译内核很耗时，所以我们推荐使用如下命令只编译设备树。

rk356x系列板卡执行以下命令：
```shell
#加载配置文件
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig
#使用dtbs参数单独编译设备树
make ARCH=arm64 -j4 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

前面我们提到了编译内核时会去编译设备树，那么此时在没有修改、添加设备树的情况下使用构建脚本单独编译设备树是不会有任何输出的，我们可以手动修改设备树文件再进行编译测试

例如修改 `内核源码/arch/arm64/boot/dts/rockchip` 下的 `xxx.dts` 文件再进行编译。找到已经编译成dtb文件对应的dts源文件，可简单在dts文件中加个空格，以防错误修改导致编译报错。以下以修改rk3568-lubancat-2.dts为例

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/dfd87c7ad0aac136a2404ec8723f43cb.png)

执行命令后会编译修改过的设备树，编译成功后生成的设备树文件(.dtb)位于源码目录下的 `内核源码/arch/arm64/boot/dts/rockchip` 

#### 2.9.2 加载

加载设备树，将编译好的新设备树文件，替换对应板卡的设备树，替换 `/boot/dtb/` 目录下的设备树文件即可

**确定板卡使用的设备树文件**：查看 `/boot/` 目录下的软连接从而确认当前板卡使用的设备树文件
```shell
ls -l /boot/
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/18c94cb5ee14952f76626dc41c0764df.png)

可以从上图中看到rk-kernel.dtb链接到了dtb/rk3568-lubancat-2-v2.dtb，在系统启动过程中会读取rk-kernel.dtb作为系统设备树，那么笔者使用的鲁班猫2板卡实际读取的设备树为/boot/dtb/rk3568-lubancat-2-v2.dtb 。因此，如果需要修改和替换系统加载的设备树，那么就要修改rk-kernel.dtb软链接的设备树

**替换设备树**：通过SCP或NFS将 `内核源码/arch/arm64/boot/dts/rockchip/` 目录下编译生成的设备树拷贝到开发板上，替换/boot/dtb/目录下的 rk3568-lubancat-2-v2.dtb

在系统上查看设备树加载情况，比如我们设备树根目录下有设备树节点 `leds`，我们可以通过以下的方式加载并查看新的设备树是否生效了。 设备树中的设备树节点在文件系统中有与之对应的文件，位于“/proc/device-tree”目录。查看“/proc/device-tree”目录如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/10971dd03922211bee838abdc412c7b1.png)

接着进入led文件夹，可以发现led节点中定义的属性以及它的子节点，如下所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/58112a003df1792619cba69ecc63bea6.png)
- 在节点属性中有一个name属性，我们查看dts源码并没有发现leds节点中定义了name属性，这个name属性自动生成的，为保存节点名。
- 这里的`属性是一个文件，而子节点是一个文件夹`，我们进入“sys-status-led”文件夹。 里面有compatible、name、status等属性文件。 我们可以使用“cat”命令查看这些属性文件，如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/55610e1932e24fd299a6bc8dd2b6e5eb.png)
