
## 1、Git 概述

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fcd92b8ebf317686a8fbb3ec5531ad8.png)

Git是一个免费的，开源的`分布式版本控制系统`，可以快速高效地处理从小型或大型的各种项目。Git易于学习，占用空间小，性能快得惊人。
## 2、SCM概述

`SCM（Software Configuration Management`，软件配置管理）是一种标识、组织和控制修改的技术。它应用于整个软件生存周期。

作为评价一个大中型软件开发过程是否正确，合理，有效的重要手段，CMM(Capability Maturity Model )能力成熟度模型提供了不同等级的标准流程，对软件开发过程（流程）进行了约束和建议, 而作为CMM 2级的一个关键域（Key Practice Area，KPA），SCM软件在整个软件的开发活动中占有很重要的位置。

Git软件比Subversion、CVS、Perforce和ClearCase等SCM（Software Configuration Management软件配置管理）工具具有性价比更高的本地分支、方便的暂存区域和多个工作流等功能。
## 3、Git安装

#### 3.1 软件下载

软件官网地址为：[https://git-scm.com/](https://git-scm.com)

软件下载地址为： https://github.com/git-for-windows/git/releases/download/v2.40.0.windows.1/Git-2.40.0-64-bit.exe

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e5c70a48736bdee48492cd06ba6725d8.png)

最早Git是在Linux上开发的，很长一段时间内，Git也只能在Linux和Unix系统上跑。不过，慢慢地有人把它移植到了Windows上。现在，Git可以在Linux、Unix、Mac和Windows这几大平台上正常运行了。由于开发机大多数情况都是windows，所以本教程选择相对简单的Windows系统软件版本进行下载，此处我们下载Windows系统的2.40.0版本软件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/201d46193e4df6a3284d337e9f199f99.png)
#### 3.2 软件安装

查看 GNU 协议，可以直接点击下一步。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fb2c193a417e6a6af01ed287e8e403d.png)

选择 Git 安装位置，要求是非中文并且没有空格的目录，然后下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d16fcab1f1490ca66ac427a90a263d05.png)
Git 选项配置，推荐默认设置，然后下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/39b22106a03c53937f298859003079d7.png)
Git 安装目录名，不用修改，直接点击下一步![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c327a653dfe84fa5a2e91b53eed55db9.png)

Git 的默认编辑器，建议使用默认的 Vim 编辑器，然后点击下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cadc03a7b6e9d6816134a4e9b9852e29.png)

默认分支名设置，选择让 Git 决定，分支名默认为 master，下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f15006e7ac561afd05010f9486eb4ac7.png)
修改 Git 的环境变量，选第一个，不修改环境变量，只在 Git Bash 里使用 Git。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9dbe5026d7b8b96299599a9790961de8.png)
选择后台客户端连接协议，选默认值 OpenSSL，然后下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/834e9b9505c21307cb611afd7e301395.png)
配置 Git 文件的行末换行符，Windows 使用 CRLF，Linux 使用 LF，选择第一个自动 转换，然后继续下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d05e35d9c8c13ab6328aefb186b411dc.png)
选择 Git 终端类型，选择默认的 Git Bash 终端，然后继续下一步![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d1fd60dadabce8a9d836c568bb48768a.png)
选择 Git pull 合并的模式，选择默认，然后下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/91080d1c6c01e5ec4b58c117c3311b18.png)
选择 Git 的凭据管理器，选择默认的跨平台的凭据管理器，然后下一步。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cfc876b8286669333a1bb6aa99c37788.png)
其他配置，选择默认设置，然后下一步
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e697a4bcfddfed425a9eafc74f65906.png)

实验室功能，技术还不成熟，有已知的 bug，不要勾选，然后点击右下角的 Install 按钮，开始安装 Git。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b34d8693fb21f09a48f20fab62445de0.png)
点击 Finsh 按钮，Git 安装成功![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c98a59f8c4adb377c15847e85533e3d3.png)
#### 3.3 软件测试

在Windows桌面空白处，点击鼠标右键，弹出右键菜单![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3266f3ac9f0cd50839fc9f2d72828294.png)
Git软件安装后，会在右键菜单中增加两个菜单

- Git GUI Here

- Git Bash Here

此处仅仅是为了验证Git软件安装的效果，所以选择Git Bash Here菜单, 选择后，Windows系统弹出Git软件的命令行黑窗口，

窗口弹出后，可以输入Git软件的操作指令。此时我们使用键盘输入操作指令：git -v或 git --version，查看当前Git软件的安装版本。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a528c7b2906413d4a35f78c5c4465ed8.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a528c7b2906413d4a35f78c5c4465ed8.png)

输入指令回车后，如果黑窗口中打印出咱们安装的软件版本2.40.0，Git软件安装成功了。
