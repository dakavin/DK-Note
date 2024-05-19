
本小节中，我们将学习如何在 Windows 系统中安装 Docker 环境。

## 1 先安装 WSL 2

### 1.1 什么是 WSL 2 ？

WSL 是 "Windows Subsystem for Linux" 的缩写，顾名思义，WSL 就是 Windows 系统的 Linux 子系统，其作为Windows 组件搭载在 Windows 10 周年更新（1607）后的 Windows 系统中。

WSL 2 是 WSL 1 的升级版本，是适用于 Linux 的 Windows 子系统体系结构的一个新版本，它支持适用于 Linux 的 Windows 子系统在 Windows 上运行 ELF64 Linux 二进制文件。 它的主要目标是**提高文件系统性能**，以及添加**完全的系统调用兼容性**。

#### 1.1.1 系统要求

想要安装 WSL 2 ，系统最低要求 Windows 10 系统

- 对于 x64 系统：版本 1903 或更高版本，内部版本为 18362 或更高版本。
- 对于 ARM64 系统：版本 2004 或更高版本，内部版本为 19041 或更高版本。

或 Windows 11。

> 低于 18362 的版本不支持 WSL 2，可使用 [Windows Update 助手](https://www.microsoft.com/software-download/windows10) 更新 Windows 版本。

想要知道当前 Windows 系统版本号，可按住 `win` + `R` 快捷键，然后输入 `winver` ，点击【确定】按钮：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/af87311a396ba1d1e9c40f2accdf0cb4.png)

## 2 启用虚拟机功能

安装 WSL 2 之前，必须启用“虚拟化”可选功能，**以管理员身份打开 PowerShell 并运行**：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c832ee482cf61310b76028e04a600a60.png)

**重新启动**系统，以完成 WSL 安装并更新到 WSL 2。

安装成功后，打开任务管理器即可看到虚拟化已启用：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/7e0098940f29fa425eb4f7c93acd952f.png)

## 3 安装 Docker Desktop

1、访问 Docker Desktop 官方下载地址：[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/) ， 选择对应平台的 Docker Desktop 安装包点击下载：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c91b49ea60c49ad95e7668cce1a8f874.png)

2、下载成功后，双击开始安装

3、安装之前的相关配置：
- Use WSL 2 instead of Hyper-V (recommended) : 启用虚拟化，以 WSL 2 替代 Hyper-V;
- Add shortcut to desktop : 安装成功后添加桌面快捷启动图标；

**将两个选项都勾选上**，然后点击【ok】,开始安装：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/93066d4a3813b7a33b0e5b238ac43712.png)


4、 安装成功后，点击【Close and restart】按钮重启系统：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a29b8069e8ea531039ac43aa3e0f0527.png)

5、重启系统成功后，会自动显示如下弹框，点击【Accept】按钮接受协议：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/f79122db7e76dc3b632edb921dff0be3.png)

> 💡 注意：你可能会弹出如下图所示的警告，告诉你 _WSL kernel version too low_ :
> 
> ![](https://img.quanxiaoha.com/quanxiaoha/169511570105136)
> 
> _如何解决呢？步骤如下：_
> 
> 1、打开 `cmd` 命令行，执行命令 `wsl --update` 更新 wsl：
> ![](https://img.quanxiaoha.com/quanxiaoha/169511591912319)
> 
> 2、如果启动 Docker 还是连接错误，在命令行中，执行命令 `netsh winsock reset` 进行重启：
> 

6、Docker 启动成功后，跳过引导介绍，看到下面界面表示 Docker 运行成功了：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/98c1acf4c487c7627d7a981a04748b8e.png)

### 3.1 安装过程中你可能遇到的问题

小哈在 Docker Desktop 启动过程中，报错如下，导致启动失败：

```
System.InvalidOperationException:
Failed to set version to docker-desktop: exit code: -1　
```

![启动 Docker Desktop 失败](https://img.quanxiaoha.com/quanxiaoha/166254581034382 "启动 Docker Desktop 失败")

#### 3.1.1 解决方案

**通过管理员权限运行 PowerShell**, 执行如下命令：

```shell
netsh winsock reset
```

重启计算机，即可正常启动 Docker Desktop 。

## 4 查看当前 Docker 版本

在 PowerShell 中执行如下命令，可打印 Docker 版本号：

```
docker -v
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c3bce5dc256dde4293ae75a32399805d.png)

## 5 验证 Docker Desktop 桌面版是否能够正常使用

在 PowerShell 中执行如下命令：

```
docker run hello-world
```

若输出如下，则表示 Docker 安装成功，且能够正常工作：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/42c0db8b6fb625a8f406bf7271ca6a08.png)

打开 Docker Desktop 可查看到刚刚的 `hello-world` 镜像：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e0c506ec17dd91a1b801d077ac58f8c6.png)
