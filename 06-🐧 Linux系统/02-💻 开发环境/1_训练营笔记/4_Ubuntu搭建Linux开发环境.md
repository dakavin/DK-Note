---
文章标题: "[[4_Ubuntu搭建Linux开发环境]]"
文章作者: Dakkk
文章概要: |
  详细介绍在Ubuntu系统上搭建嵌入式Linux内核开发环境，包括开发工具安装、VSCode配置、clangd设置、内核编译和源码阅读的完整流程。
tags:
  - Ubuntu
  - 嵌入式Linux
  - 内核开发
  - VSCode
  - clangd
  - 交叉编译
  - ARM64
  - 开发环境搭建
相关文章:
  - "[[3_win使用vscode搭建嵌入式Linux开发环境]]"
  - "[[1_开发环境搭建]]"
  - "[[../../08-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/1_驱动章节实验环境搭建]]"
  - "[[14_Ubuntu根文件系统构建（Skip）]]"
  - "[[6_Makefile文件示例]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/02-💻 开发环境/1_训练营笔记/4_Ubuntu搭建Linux开发环境.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-12 19:46:19
---

## 1 安装基本开发工具

```shell
sudo apt update
sudo apt install gcc make  git  bc libssl-dev liblz4-tool device-tree-compiler bison flex u-boot-tools gcc-aarch64-linux-gnu
```

**安装bear**
```shell
sudo apt install bear
```

## 2 安装vscode和插件

安装vscode不做介绍

插件参考：[2.1 安装vscode](3_win使用vscode搭建嵌入式Linux开发环境.md#2.1%20安装vscode)

常用快捷键参考：[2.6 常用快捷键](3_win使用vscode搭建嵌入式Linux开发环境.md#2.6%20常用快捷键)
## 3 配置clangd

参考：[2.5 配置clangd](3_win使用vscode搭建嵌入式Linux开发环境.md#2.5%20配置clangd)
- 参考是配置vscode本身的settings.json文件的

下面是用于代码所在文件夹的`.clangd`配置的

**这个是使用bear编译后可以这么写**
```yml
CompileFlags:
  Add:
    # 核心交叉编译配置
    - "--target=aarch64-linux-gnu"        # 强制指定ARM64目标架构
    - "--sysroot=/usr/aarch64-linux-gnu"  # 系统根目录（需验证路径有效性）
    - "--gcc-toolchain=/usr/aarch64-linux-gnu # 显式指定工具链位置
    - "-I/usr/lib/gcc-cross/aarch64-linux-gnu/9/include"  # 交叉编译器头文件
    
    # 抑制警告
    - "-Wno-unknown-attributes"          # 忽略GCC特有属性（如section/aligned）
    - "-Wno-gnu-statement-expression"    # 忽略GNU语句表达式
    - "-Wno-unused-command-line-argument" # 忽略残留参数警告

  Remove:
    # 移除与clang冲突的GCC参数（持续补充）
    - "-nostdinc"                      # 防止屏蔽系统头文件
    - "-mabi=lp64"                     # clang默认已处理ABI

```

**编译前，无法索引到kernel对应的文件，可以添加如下内容**
```yml
CompileFlags:
  Add:
    # 核心交叉编译配置
    - "--target=aarch64-linux-gnu"
    - "--sysroot=/usr/aarch64-linux-gnu"
    - "--gcc-toolchain=/usr/aarch64-linux-gnu"

    # 内核头文件路径（根据实际路径调整）
    - "-I/home/dakkk/Lubancat_SDK/kernel/include"
    - "-I/home/dakkk/Lubancat_SDK/kernel/arch/arm64/include"
    - "-I/home/dakkk/Lubancat_SDK/kernel/include/uapi"
    - "-I/home/dakkk/Lubancat_SDK/kernel/arch/arm64/include/uapi"
    - "-I/home/dakkk/Lubancat_SDK/kernel/arch/arm64/include/generated"
    - "-I/home/dakkk/Lubancat_SDK/kernel/include/generated"

    # 交叉编译器头文件路径（关键）
    - "-I/usr/aarch64-linux-gnu/include"
    - "-I/usr/lib/gcc/aarch64-linux-gnu/9/include"  # 根据实际GCC版本调整

    # 必要宏定义
    - "-D__KERNEL__"
    - "-D__linux__"
    - "-D__aarch64__"  # 明确指定架构

    # 兼容性选项
    - "-Wno-unknown-attributes"
    - "-Wno-gnu-variable-sized-type-not-at-end"
    - "-Wno-address-of-packed-member"

  Remove:
    - "-nostdinc"
    - "-mabi=lp64"
```

> [!bug]+ 无法找到types.h中的__u64类型
> - 创建必要的符号链接
> 	- cd /home/dakkk/Lubancat_SDK/kernel/include
> 	- ln -sf ../arch/arm64/include/asm asm

## 4 使用WindTerm（暂时不用）

WindTerm是Linux环境下好用的终端软件，GUI界面、支持ssh、串口等协议，可以记录历史命令， 我们使用它来**打开串口操作开发板**。

在Ubuntu中使用浏览器打开 https://github.com/kingToolbox/WindTerm/releases/tag/2.5.0，下载 Linux版本的软件包：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e711fcadcd17b202e89f7aae294dc5a6.png)

把下载到的WindTerm_2.5.0_Linux_Portable_x86_64.tar.gz放到/home/book目录下，执行解压命令：
```shell
cd /home/book
tar xzf WindTerm_2.5.0_Linux_Portable_x86_64.tar.gz
cd WindTerm_2.5.0/
chmod +x WindTerm
```

以后就可以在桌面系统打开终端后执行以下命令打开WindTerm：
```shell
cd /home/book/WindTerm_2.5.0/
sudo ./WindTerm
```

**配置WindTerm**
- WindTerm的使用方法可以参考： https://zhuanlan.zhihu.com/p/468501270
- 配置目的：
	- 解决问题：我们使用WindTerm打开/dev/ttyUSB0或/dev/ttyACM0等串口时，不加sudo命令就会 碰到权限问题
	- 方便使用：我们想在Ubuntu左侧启动栏点击鼠标就启动WindTerm
- 解决权限问题：执行如下命令把book用户放入组(dialout、tty)即可：
```shell
sudo newgrp dialout tty
sudo usermod -a -G dialout book
sudo usermod -a -G tty book
```
- 放入启动栏：创建一个文件 `/usr/share/applications/windterm.desktop `，内容如下：
```shell
[Desktop Entry]
Name=WindTerm
Comment=A professional cross-platform SSH/Sftp/Shell/Telnet/Serial terminal
GenericName=Connect Client
Exec=/home/book/WindTerm_2.5.0/WindTerm
Type=Application
Icon=/home/book/WindTerm_2.5.0/windterm.png
StartupNotify=false
StartupWMClass=Code
Categories=Application;Development
Actions=new-empty-window
Keywords=windterm
[Desktop Action new-empty-window]
Name=New Empty Window
Icon=/home/book/WindTerm_2.5.0/windterm.png
Exec=/home/book/WindTerm_2.5.0/WindTerm
```
- 然后增加可执行权限、并启动程序：
```shell
sudo  chmod +x /usr/share/applications/windterm.desktop /home/book/WindTerm_2.5.0/WindTerm
```

最后如下图操作，以后就可以在Ubuntu左侧的启动栏点击鼠标启动WindTerm：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5546a49b1247090d0bf9757c214647cc.png)

## 5 安装计算器

```shell
sudo snap install uno-calculator
```

然后启动它、添加进Ubuntu左侧的启动栏，入下图操作
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c37c2e398c79b881b0b58c9f57379e86.png)

## 6 下载和编译内核

可以查看
- [1.3 RK3568板卡](3_win使用vscode搭建嵌入式Linux开发环境.md#1.3%20RK3568板卡)

下面只补充ubuntu系统编译内核特殊的部分

```shell
# 安装交叉编译工具链
sudo apt install gcc-aarch64-linux-gnu
sudo apt install binutils-aarch64-linux-gnu

# 验证
which aarch64-linux-gnu-gcc
aarch64-linux-gnu-gcc --version
which aarch64-linux-gnu-ld

# 添加旧版本工具链仓库（如果尚未添加）
sudo apt-add-repository ppa:ubuntu-toolchain-r/test
sudo apt update

# 安装 GCC 9 的交叉编译器
sudo apt install gcc-9-aarch64-linux-gnu g++-9-aarch64-linux-gnu

# 验证
aarch64-linux-gnu-gcc-9 --version
```

如果要回复GCC13默认版本
```shell
sudo update-alternatives --config aarch64-linux-gnu-gcc
# 选择 GCC 13 对应的编号
```


在前面加上bear命令，没有的话apt安装即可，这样就会在当前目录下生成complie_commands.json文件
```shell
# 清除内核编译产物
sudo make mrproper

# 编译内核时指定 GCC 9
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8
```

## 7 使用vscode阅读内核源码

### 7.1 使用compile_commands.json

使用bear编译内核后，会在内核所在目录生成文件compile_commands.json，不过要检查一下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6a6ddb986abbfffd658025d707f9ab49.png)

### 7.2 vscode打开内核目录

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/490c4795042582cef1f2b3d495a1560b.png)

### 7.3 触发clangd

在vscode里打开任意一个C文件，就会触发clangd建立索引
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a9f33ee89fcfdf56a40bc35fdea3b42e.png)

### 7.4 验证

在创建索引的过程中，可以使用如下命令查看.cache目录，它会不断变大(最终大小在60M左右)：
```shell
du -h .cache/
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2ad26243d8cdce1749d98677d3dc0b82.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4533d564ec4a41282fcc3f126cf6f3d8.png)

## 8 vscode阅读外部源码

比如我们编写了hello驱动程序，它用到内核里的头文件、函数，我们点击hello驱动里的函数时，想打开内核的文件，需要创建一个workspace：
- 里面含有内核目录、hello驱动源码目录
- 内核目录下有compile_commands.json
 - hello驱动源码目录下有compile_commands.json
### 8.1 创建workspace

单独打开kernel目录，然后保存为workspace

![Snipaste_2025-05-19_22-24-04.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b0e2def5f7d262231b0df4446f515bd0.png)

### 8.2 驱动目录加入workspace

假设驱动程序位于这个目录：` ~/Lubancat_SDK/lubancat_rk_code_storage/linux_driver/module/hellomodule`

**编译驱动，会生成compile_commands.json**
```shell
cd ~/Lubancat_SDK/lubancat_rk_code_storage/linux_driver/module/hellomodule
bear -- make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/be6f6527627105af800b774194268465.png)

然后检查compile_commands.json，下面这种情况不需要修改
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d3e36156e30b41e9e12e4c6fd70f7bd5.png)

将文件夹，加入workspace
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9dcab0d64747a8da908b5da08bf8fbc8.png)

### 8.3 验证

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/48e90b1780953f7acc9a329dcb1b1c64.png)

## 9 常见错误

### 9.1 无法跳转

第1步：确认已经关闭intellisense 和 确认有clangd

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3fa3d1fc370342bffe5b25dece6b82c3.png)

第2步：确认源码目录下有compile_commands.json，并且文件里面记录C文件、"gcc"被改成了"/usr/bin/aarch64-linux-gnu-gcc"：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/634f445258eb79a9d07b7e1c95821477.png)

### clangd可以索引，但是C/C++还是检索报错

在当前用户目录下创建如下文件 `.vscode/c_cpp_properties.json`
```shell
{
  "version": 4,
  "configurations": [
    {
      "name": "aarch64-linux-gnu",
      "intelliSenseMode": "linux-gcc-arm",
      "compilerPath": "/usr/bin/aarch64-linux-gnu-gcc",
      "compilerArgs": [
        "--target=aarch64-linux-gnu",
        "--sysroot=/usr/aarch64-linux-gnu"
      ],
      "includePath": [
        "${workspaceFolder}/**",
        "/usr/aarch64-linux-gnu/include",
        "/usr/aarch64-linux-gnu/include/linux",
        "/usr/lib/gcc-cross/aarch64-linux-gnu/9/include",
        "/usr/include"
      ],
      "defines": [],
      "browse": {
        "path": [
          "${workspaceFolder}",
          "/usr/aarch64-linux-gnu/include",
          "/usr/aarch64-linux-gnu/include/linux",
          "/usr/lib/gcc-cross/aarch64-linux-gnu/9/include",
          "/usr/include"
        ],
        "limitSymbolsToIncludedHeaders": true,
        "databaseFilename": "${workspaceFolder}/.vscode/browse.vc.db"
      }
    }
  ]
}
```

在有compile_command.json文件的同级目录下创建 `.clangd`
```shell
CompileFlags:
  Add:
    # 核心交叉编译配置
    - "--target=aarch64-linux-gnu"        # 强制指定ARM64目标架构
    - "--sysroot=/usr/aarch64-linux-gnu"  # 系统根目录（需验证路径有效性）
    - "--gcc-toolchain=/usr/aarch64-linux-gnu"  # 显式指定工具链位置
    - "-I/usr/lib/gcc-cross/aarch64-linux-gnu/9/include"  # 交叉编译器头文件
    
    # 抑制警告
    - "-Wno-unknown-attributes"          # 忽略GCC特有属性（如section/aligned）
    - "-Wno-gnu-statement-expression"    # 忽略GNU语句表达式
    - "-Wno-unused-command-line-argument" # 忽略残留参数警告

  Remove:
    # 移除与clang冲突的GCC参数（持续补充）
    - "-nostdinc"                      # 防止屏蔽系统头文件
    - "-mabi=lp64"                     # clang默认已处理ABI

```

ok解决！！！
