
## 1、Git 概述

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923174232.png)

Git是一个免费的，开源的`分布式版本控制系统`，可以快速高效地处理从小型或大型的各种项目。Git易于学习，占用空间小，性能快得惊人。
## 2、SCM概述

`SCM（Software Configuration Management`，软件配置管理）是一种标识、组织和控制修改的技术。它应用于整个软件生存周期。

作为评价一个大中型软件开发过程是否正确，合理，有效的重要手段，CMM(Capability Maturity Model )能力成熟度模型提供了不同等级的标准流程，对软件开发过程（流程）进行了约束和建议, 而作为CMM 2级的一个关键域（Key Practice Area，KPA），SCM软件在整个软件的开发活动中占有很重要的位置。

Git软件比Subversion、CVS、Perforce和ClearCase等SCM（Software Configuration Management软件配置管理）工具具有性价比更高的本地分支、方便的暂存区域和多个工作流等功能。
## 3、Git安装

#### 3.1 软件下载

软件官网地址为：[https://git-scm.com/](https://git-scm.com)

软件下载地址为： https://github.com/git-for-windows/git/releases/download/v2.40.0.windows.1/Git-2.40.0-64-bit.exe

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923174326.png)

最早Git是在Linux上开发的，很长一段时间内，Git也只能在Linux和Unix系统上跑。不过，慢慢地有人把它移植到了Windows上。现在，Git可以在Linux、Unix、Mac和Windows这几大平台上正常运行了。由于开发机大多数情况都是windows，所以本教程选择相对简单的Windows系统软件版本进行下载，此处我们下载Windows系统的2.40.0版本软件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923174347.png)
#### 3.2 软件安装

查看 GNU 协议，可以直接点击下一步。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160341.png)

选择 Git 安装位置，要求是非中文并且没有空格的目录，然后下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160404.png)
Git 选项配置，推荐默认设置，然后下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160448.png)
Git 安装目录名，不用修改，直接点击下一步![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160519.png)

Git 的默认编辑器，建议使用默认的 Vim 编辑器，然后点击下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160611.png)

默认分支名设置，选择让 Git 决定，分支名默认为 master，下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160745.png)
修改 Git 的环境变量，选第一个，不修改环境变量，只在 Git Bash 里使用 Git。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921160843.png)
选择后台客户端连接协议，选默认值 OpenSSL，然后下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161157.png)
配置 Git 文件的行末换行符，Windows 使用 CRLF，Linux 使用 LF，选择第一个自动 转换，然后继续下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161445.png)
选择 Git 终端类型，选择默认的 Git Bash 终端，然后继续下一步![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161521.png)
选择 Git pull 合并的模式，选择默认，然后下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161614.png)
选择 Git 的凭据管理器，选择默认的跨平台的凭据管理器，然后下一步。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161655.png)
其他配置，选择默认设置，然后下一步
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161733.png)

实验室功能，技术还不成熟，有已知的 bug，不要勾选，然后点击右下角的 Install 按钮，开始安装 Git。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161841.png)
点击 Finsh 按钮，Git 安装成功![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161932.png)
#### 3.3 软件测试

在Windows桌面空白处，点击鼠标右键，弹出右键菜单![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921161955.png)
Git软件安装后，会在右键菜单中增加两个菜单

- Git GUI Here

- Git Bash Here

此处仅仅是为了验证Git软件安装的效果，所以选择Git Bash Here菜单, 选择后，Windows系统弹出Git软件的命令行黑窗口，

窗口弹出后，可以输入Git软件的操作指令。此时我们使用键盘输入操作指令：git -v或 git --version，查看当前Git软件的安装版本。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921162051%201.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230921162051.png)

输入指令回车后，如果黑窗口中打印出咱们安装的软件版本2.40.0，Git软件安装成功了。
