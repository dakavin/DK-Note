## 1 Ubuntu上操作

### 1.1 安装工具

**安装基本开发工具**
```shell
sudo apt update
sudo apt install gcc make  git  bc libssl-dev liblz4-tool device-tree-compiler bison flex u-boot-tools gcc-aarch64-linux-gnu
```

**安装bear**
```shell
sudo apt install bear
```

### 1.2 Imx6ull板卡

下载内核
```shell
$ git clone https://e.coding.net/codebug8/repo.git

$ mkdir -p 100ask_imx6ull-sdk && cd 100ask_imx6ull-sdk

$ ../repo/repo init -u https://gitee.com/weidongshan/manifests.git -b linux-sdk -m imx6ull/100ask_imx6ull_linux4.9.88_release.xml --no-repo-verify

$ ../repo/repo sync -j4
```

配置工具链
```shell
gedit ~/.bashrc
# 或者
vi ~/.bashrc

#在最后加入如下内容：
export ARCH=arm
export CROSS_COMPILE=arm-buildroot-linux-gnueabihf
export PATH=$PATH:/home/lubancat/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux
gnueabihf_sdk-buildroot/bin
```
- 重新关闭和打开终端

**编译内核**
- 编译内核的目的是生成compile_commands.json，执行如下命令：
```shell
$ cd /home/book/100ask_imx6ull-sdk/Linux-4.9.88
$ make 100ask_imx6ull_defconfig
$ bear make zImage -j4
```
- 如果你之前曾经编译过内核但是没有在前面使用bear命令，那么需要重新编译：
```shell
$ make  clean
$ bear make zImage -j4
```

### 1.3 RK3568板卡

**下载内核代码：** 不做介绍，查看下面即可
- [2 获取内核源码](../../02-💾%20Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识/1_驱动章节实验环境搭建.md#2%20获取内核源码)

**配置工具链**
```shell
gedit ~/.bashrc

# 设置内核编译环境变量
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
export PATH=$PATH:/home/dakkk/Lubancat_SDK/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin

source ~/.bashrc
```

**编译内核**
vscode的clangd插件使用compile_commands.json文件来生成索引文件，这样当我们点击某个函数时 可以飞快跳转到它定义的地方

compile_commands.json文件中记录的是每个文件的编译选项，样式如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6aec3196504aa57c988ab58e519d7afd.png)

进入内核源码根目录，根据具体的板卡设置配置文件，RK356x系列板卡用户执行以下命令编译内核源码：

```shell
#清除之前生成的所有文件和配置
make mrproper

# 加载lubancat2_defconfig配置文件，rk356x系列均是该配置文件
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig

# 编译内核，指定平台，指定交叉编译工具，使用8线程进行编译，线程可根据电脑性能自行确定
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8
```

在前面加上bear命令，没有的话apt安装即可，这样就会在当前目录下生成complie_commands.json文件
```shell
sudo make mrproper
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig
sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8
```

也可以在SDK目录下，修改build.sh文件，修改如下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f4631724f099febe0aca63aa4b5352c1.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/502a5b1c4df443cf4450af7005335cfe.png)

**编译成功后就会在当前目录下得到文件compile_commands.json，需要如下修改**
- 在内核目录下创建 `.clangd` 文件，填入如下内容
```shell
touch .clangd
vi .clangd

#内容如下
CompileFlags:
  Add:
    - "--sysroot=/usr/aarch64-linux-gnu"  # 核心！指定交叉编译器的根目录
    - "-I/usr/lib/gcc-cross/aarch64-linux-gnu/9/include"  # 显式添加交叉编译器头文件路径
    - "-Wno-unknown-warning-option"       # 忽略未知参数警告
  Remove:
    - "-nostdinc"  # 确保移除所有残留的 -nostdinc
    - "-mabi=lp64"
```
- `compile_commands.json` 文件，看看如下参数是否是交叉编译器的，是就不需要修改
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0e39bbd10f4179190553d45773b3e7a5.png)
## 2 Win上操作

### 2.1 安装vscode

官网安装即可，不做介绍，主要写一些用得到的插件

依次输入下列插件名字，安装：
- C/C++
- C/C++ Extension Pack
- C/C++ Snippets
- Clangd
- Remote SSH
- Code Runner
- Code Spell Checker
- vscode-icons
- compareit
- DeviceTree
- 通义灵码
- Bracket Pair Colorization Toggler
- Rainbow Highlighter
	- 高亮文字：shift + alt + z
	- 取消高亮：shift + alt + a
- Arm Assembly
- Chinese
- Hex Editor
- One Dark Pro
- Markdown All in One
- Markdown Preview Enhanced

### 2.2 设置SSH

vscode自带的ssh程序有Bug，我们需要替换ssh，可以使用GIT工具自带的ssh，所以先安装Git： 下载： https://gitforwindows.org/ 
- 安装：双击即可

然后替换ssh，确保GIT工具的路径下有ssh.exe后，如下替换：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f48dc9a7e632955e1d3c65eb26583dca.png)

### 2.3 远程登录服务器

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/255e99caad1a088e1aa0e30c6dfd6c1e.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/21f78c420723fa7f6cf2b68808fd4364.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9383a71b1989bdb55b72800dede7d724.png)

**设置免密登录**
- 这不是必须的，后续使用vscode访问远程服务器时，你可以一直使用密码登录。
- 如果想免密登录的话，需要生成ssh秘钥，先在windows的命令行执行：
```shell
ssh-keygen
```
- 然后再修改vscode配置：
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f3a97012b214ca77bc49d043c887afa6.png)
最后把前面生成的id_rsa.pub复制到Ubuntu目录/home/lubancat：
```shell
mkdir /home/book/.ssh
cat /home/book/id_rsa.pub >> /home/book/.ssh/authorized_keys
chmod 700 /home/lubancat/.ssh
chmod 600 /home/lubancat/.ssh/authorized_keys
sudo /usr/sbin/sshd restart
```
### 2.4 在服务器上安装插件

vscode连接上服务器后，查看本地插件，发现有如下字样的插件就点击"Install in SSH"：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/de07c4fe6bfc06d83e78d13b7a0798f6.png)

### 2.5 配置clangd

前面只是安装clangd插件，它的使用还需要一个运行在Linux服务器上的clangd程序

我们以后使用vscode打开C文件时，会提示你安装clangd程序，它会安装最新版本(版本15)，但是这个 版本有一些Bug，所以我们手工安装版本13
![](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0157efe7b054e1a0c6330c6574c7ba82.png)

使用浏览器打开 https://github.com/clangd/clangd/releases?page=3
- 下载Linux安装包：
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e12a57cea5fe4d01cddc0d88d38d1ec6.png)
- 把下载到的clangd-linux-19.1.2.zip放到/home/lubancat目录下，执行解压命令：
```shell
cd ~
unzip clangd-linux-19.1.2.zip
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/dc6f46dff1b1c3e33577e5dd259e05da.png)

**开始配置**
- 在Windows的vscode界面按下图步骤打开setting.json文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/10e11a696ef1982d449f9c2fadcac21d.png)
- 在setting.json中写入如下内容(我们第1次打开源码目录后，这个文件可能被自动修改，你需要再次修改它)：
```json
{
    "C_Cpp.default.intelliSenseMode": "linux-gcc-arm",
    "C_Cpp.intelliSenseEngine": "Disabled",
    "clangd.path": "/home/lubancat/clangd_19.1.2/bin/clangd",
    "clangd.arguments": [
    "--log=verbose",
    "--query-driver=/usr/bin/aarch64-linux-gnu-gcc",  // 指向交叉编译器
    "--background-index",
    "--header-insertion=never"
    ],
}
```

C/C++插件里的intellisense和clangd是冲突的，如果我们没有手工设置setting.json，当使用vscode打 开C文件时也会提示禁止intellisense，点击鼠标即可禁止。它的本质也是修改setting.json。
### 2.6 常用快捷键

打开C文件后，在文件里点击右键就可以看到大部分快捷键
```txt
输入文件名打开文件: Ctrl + P
跳到某行: Ctrl + G + 行号
打开文件并跳到某行: Ctrl + p 文件名:行号
列出文件里的函数 : Ctrl + Shift + O，可以输入函数名跳转
函数/变量跳转: 按住Ctrl同时使用鼠标左键点击、F12
前进: Ctrl + Shift + 
后退: Ctrl + Alt + 
列出引用 : Shift + F12
查找所有引用 : Alt + Shift + F12
切换侧边栏展示/隐藏: Ctrl + B
打开命令菜单: Ctrl + Shift + P
手动触发建议: Ctrl + Space
手动触发参数提示: Ctrl + Shift + Space
打开/隐藏终端: Ctrl + `(Tab上方的那个键)
重命名符号: F2
当前配置调试: F5
上/下滚编辑器: Ctrl + ↑/↓
搜索/替换 : Ctrl + F/H
高亮文字：shift + alt + z
取消高亮：shift + alt + a
```
## 3 vscode阅读内核源码

### 3.1 打开目录

确保Ubuntu上Linux内核源码目录下已经有了文件compile_commands.json

打开内核中任意一个文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/449534758363601786ca2ad6753d9964.png)

### 3.2 触发索引建立

就会触发clangd建立索引
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6553253c7c7433582659b7d0a4c6e332.png)

**重建索引方式**
- 进入内核目录，删除索引文件
```shell
rm -rf ~/.cache/clangd
```
- 进入vscode，按下 `Ctrl+Shift+P`，输入 `Restart Language Server`
### 3.3 验证
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c9cea507eee15124fa4f65c949f2c2b4.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e0170d0cc8f0bcf1befdc3c43e6316dd.png)

## 4 vscode阅读内核外部源码

比如我们编写了hello驱动程序，它用到内核里的头文件、函数，我们点击hello驱动里的函数时，想打开内核的文件，需要创建一个workspace：
- 里面含有内核目录、hello驱动源码目录
- 内核目录下有compile_commands.json
 - hello驱动源码目录下有compile_commands.json
### 4.1 创建workspace

单独打开kernel目录，然后保存为workspace

![Snipaste_2025-05-19_22-24-04.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b0e2def5f7d262231b0df4446f515bd0.png)

### 4.2 驱动目录加入workspace

在vscode界面创建workspace，保存在内核的上一层目录中
假设驱动程序位于这个目录：` ~/Lubancat_SDK/lubancat_rk_code_storage/linux_driver/module/hellomodule`

**编译驱动，会生成compile_commands.json**
```shell
cd ~/Lubancat_SDK/lubancat_rk_code_storage/linux_driver/module/hellomodule
bear -- make
```

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/be6f6527627105af800b774194268465.png)

然后检查compile_commands.json，下面这种情况不需要修改
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d3e36156e30b41e9e12e4c6fd70f7bd5.png)

将文件夹，加入workspace
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9dcab0d64747a8da908b5da08bf8fbc8.png)

### 4.3 验证

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/48e90b1780953f7acc9a329dcb1b1c64.png)

## 5 常见错误

### 5.1 clangd持续建立index，导致ssh连接崩溃

参考文章：[clangd background indexing crash 后台创建索引，导致clangd进程崩溃_indexing with clangd-CSDN博客](https://blog.csdn.net/qq_39723634/article/details/142186973)

根据clangd官网([Troubleshooting](https://clangd.llvm.org/troubleshooting#crashes-in-background-indexing "Troubleshooting"))描述 加入参数
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a44e318623b466e02bfb26db5b2ffba9.png)

找到如下文件：`~/.config/Code/User/setting.json`，添加参数
```json
--log=verbose
--background-index=0
```