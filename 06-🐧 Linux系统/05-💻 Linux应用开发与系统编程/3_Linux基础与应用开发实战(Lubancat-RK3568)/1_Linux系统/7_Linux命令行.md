---
文章标题: "[[7_Linux命令行]]"
文章作者: Dakkk
文章概要: |
  介绍Linux命令行基础知识，包括Shell概念、终端打开方式、命令提示符组成、基本命令体验和命令格式帮助系统等内容。
tags:
  - Linux
  - 命令行
  - Shell
  - 终端
  - bash
  - 系统基础
  - 用户权限
  - 文件操作
相关文章:
  - "[[../../../01-❇️ 参考资料/0_个人实用技巧/5_Linux相关命令]]"
  - "[[../../../01-❇️ 参考资料/0_个人实用技巧/9_编译追加日志]]"
  - "[[../../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/3_CentOS 安装 Docker]]"
  - "[[../../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/4_Ubuntu 安装 Docker]]"
  - "[[../../../01-❇️ 参考资料/0_个人实用技巧/4_apt命令及问题]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/1_Linux系统/7_Linux命令行.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-05-06 17:04:28
修改时间: 2025-05-28 00:19:51
---


**相关操作可以查看下面的文档链接**
- [1_初识Linux](../../../../03-🐧%20Linux基础/1_初识Linux.md)
- [2_Linux基础命令](../../../../03-🐧%20Linux基础/2_Linux基础命令.md)
- [3_用户和权限](../../../../03-🐧%20Linux基础/3_用户和权限.md)
- [4_Linux实用操作](../../../../03-🐧%20Linux基础/4_Linux实用操作.md)
- [5_Linux常见指令](../../../../03-🐧%20Linux基础/5_Linux常见指令.md)
- [6_Linux常用操作](../../../../03-🐧%20Linux基础/6_Linux常用操作.md)

## 1 什么是命令行（shell）

如下图所示，Shell对外接受用户输入的命令， 对内通过“系统调用”传递给内核，内核执行操作后把输出通过Shell呈现给用户，也就是说， Shell就是一个中间人。而Shell的英文原意是“壳”的意思，也是为了把它与内核区分开来。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a614dea92fd8174d59d3205d6ae398b4.png)

在平时的交流中，有时我们会说打开Shell、终端（Terminal）或控制台（Console）， 严格来说它们实际上不是同一种的东西，但只要明白，当我们说打开Shell、终端或控制台的时候， 通常就是为了使用命令行控制系统。它们的严格区分如下，了解下即可：
- `Shell`：指命令行解释器，常见的解释器有bash，sh，在Ubuntu系统默认用的是bash解释器， 所以有时说bash也是指命令行。
- `终端（Terminal）`：通常指用来运行Shell的程序，视场景的不同有不一样的名称， 如Ubuntu系统自带的叫本地终端，嵌入式开发板常常提供串口进行输入输出的串口终端， 通过网络访问的ssh终端。
- `控制台（Console）`：特指某些终端，通常是指它的物理形态，如带键盘和显示器的物理设备。
## 2 打开终端的方式

板卡终端的打开方式有三种，大家可以选择自己最为方便的方式登录终端
1. 串口连接（适合小白以及板卡在身边的） [3.4 串口终端登录](../../../../02-💻%20开发环境/3_Lubancat-RK3568/1_快速使用手册/1_快速开始.md#3.4%20串口终端登录)
2. ssh连接（适合远程用户）[3.5 SSH登板卡](../../../../02-💻%20开发环境/3_Lubancat-RK3568/1_快速使用手册/1_快速开始.md#3.5%20SSH登板卡)
3. 桌面打开终端（适合桌面端用户）右键即可
## 3 命令行提示符📕

当我们打开终端后，我们可以看到终端本身显示了一行字符，而且按回车后会重复出现：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a4691a00666ca1dd0a95b5ec051bff5c.png)

```shell
cat@lubancat:~#
```

- `cat` ：“@”符号的左侧，它表示的是当前登录用户，上图使用的是cat用户登录。
- `@` ：分隔符号，可理解为at，表示root用户at主机lubancat上。
- `lubancat` ：当前**系统的主机名**。
- `: `：分隔符号。
- `~` ：冒号后表示用户当前所在的目录，此处的波浪线表示当前用户的家目录， root用户的家目录为 `/root` ,普通用户的目录在 `/home/（用户名）`
- `#` ：命令提示符，Linux 用这个符号标识登录的用户权限等级
	- 如果是超级用户， 提示符就是“#”
	- 如果是普通用户，提示符就是“$”。
## 4 命令行体验

不做体验，下面给出指令即可
```shell
cd /home    # 切换到/home目录
pwd         # 显示当前目录
cd ~        # 切换至家目录，~代表家目录，不同用户家目录不一样
pwd         # 显示当前目录
touch hello # 创建叫hello的文件
touch abc.c # 创建叫abc.c的文件
ls          # l是字母L的小写，不要把l输入成数字1或字母i的大写
ls -l       # 同上，两个l都是字母L的小写
```
## 5 命令的格式与帮助

**格式：**
```shell
command [-options] [argument]
```

**帮助：**
```shell
ls --help
ls -h
```

**自动补全：** 使用 `Tab` 键即可

**退出和取消命令**： `ctrl + c`

**命令的本质：** 
- 命令究竟是什么，当我们敲下命令“ls”后，Shell究竟做了什么事情
- [2.1 bin目录](../../../../02-💻%20开发环境/3_Lubancat-RK3568/2_镜像构建与部署/12_根文件系统的介绍.md#2.1%20bin目录) 下包含了很多命令，如下图所示，可以看到ls、lsblk、lsmod等命令程序
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/112e53a9e66041d97d0d4ee379f6e9ab.png)
**which命令：** 查看指定命令的路径