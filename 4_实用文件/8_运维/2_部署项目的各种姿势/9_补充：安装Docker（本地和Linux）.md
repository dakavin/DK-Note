
参考文章：https://www.quanxiaoha.com/docker/docker-tutorial.html
## 1 本地/Windows安装Docker

### 1.1 先安装 WSL 2

#### 1.1.1 什么是 WSL 2 ？

WSL 是 "Windows Subsystem for Linux" 的缩写，顾名思义，WSL 就是 Windows 系统的 Linux 子系统，其作为Windows 组件搭载在 Windows 10 周年更新（1607）后的 Windows 系统中。

WSL 2 是 WSL 1 的升级版本，是适用于 Linux 的 Windows 子系统体系结构的一个新版本，它支持适用于 Linux 的 Windows 子系统在 Windows 上运行 ELF64 Linux 二进制文件。 它的主要目标是**提高文件系统性能**，以及添加**完全的系统调用兼容性**。

#### 1.1.2 系统要求

想要安装 WSL 2 ，系统最低要求 Windows 10 系统

- 对于 x64 系统：版本 1903 或更高版本，内部版本为 18362 或更高版本。
- 对于 ARM64 系统：版本 2004 或更高版本，内部版本为 19041 或更高版本。

或 Windows 11。

> 低于 18362 的版本不支持 WSL 2，可使用 [Windows Update 助手](https://www.microsoft.com/software-download/windows10) 更新 Windows 版本。

想要知道当前 Windows 系统版本号，可按住 `win` + `R` 快捷键，然后输入 `winver` ，点击【确定】按钮：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/af87311a396ba1d1e9c40f2accdf0cb4.png)

### 1.2 启用虚拟机功能

安装 WSL 2 之前，必须启用“虚拟化”可选功能，**以管理员身份打开 PowerShell 并运行**：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c832ee482cf61310b76028e04a600a60.png)

**重新启动**系统，以完成 WSL 安装并更新到 WSL 2。

安装成功后，打开任务管理器即可看到虚拟化已启用：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/7e0098940f29fa425eb4f7c93acd952f.png)

### 1.3 安装 Docker Desktop

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

#### 1.3.1 安装过程中你可能遇到的问题

在 Docker Desktop 启动过程中，报错如下，导致启动失败：

```
System.InvalidOperationException:
Failed to set version to docker-desktop: exit code: -1　
```

![启动 Docker Desktop 失败](https://img.quanxiaoha.com/quanxiaoha/166254581034382 "启动 Docker Desktop 失败")

#### 1.3.2 解决方案

**通过管理员权限运行 PowerShell**, 执行如下命令：

```shell
netsh winsock reset
```

重启计算机，即可正常启动 Docker Desktop 。

### 1.4 查看当前 Docker 版本

在 PowerShell 中执行如下命令，可打印 Docker 版本号：

```
docker -v
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c3bce5dc256dde4293ae75a32399805d.png)

### 1.5 验证 Docker Desktop 桌面版是否能够正常使用

在 PowerShell 中执行如下命令：

```
docker run hello-world
```

若输出如下，则表示 Docker 安装成功，且能够正常工作：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/42c0db8b6fb625a8f406bf7271ca6a08.png)

打开 Docker Desktop 可查看到刚刚的 `hello-world` 镜像：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e0c506ec17dd91a1b801d077ac58f8c6.png)

## 2 Linux安装Docker

### 2.1 准备工作

#### 2.1.1 操作系统要求

Docker 支持以下 64 位的 CentOS 版本, 且内核版本不低于 3.10：

- CentOS 7
- CentOS 8
- 更高版本...

#### 2.1.2 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，如之前有安装，需使用以下命令卸载旧版本：

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 2.2 开始安装

#### 2.2.1 使用 yum 安装

执行以下命令安装依赖包：

```
$ sudo yum install -y yum-utils
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

执行下面的命令添加 `yum` 软件源：

```
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo
```

如果需要测试版本的 Docker 请执行以下命令：

```
$ sudo yum-config-manager --enable docker-ce-test
```

#### 2.2.2 安装 Docker

更新 `yum` 软件源缓存，并安装 `docker-ce`。

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### 2.3 CentOS8 额外设置

由于 CentOS8 防火墙使用了 `nftables`，但 Docker 尚未支持 `nftables`， 我们可以使用如下设置使用 `iptables`：

更改 `/etc/firewalld/firewalld.conf`

```
# FirewallBackend=nftables
FirewallBackend=iptables
```

或者执行如下命令：

```
$ firewall-cmd --permanent --zone=trusted --add-interface=docker0

$ firewall-cmd --reload
```

### 2.4 使用脚本安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装，另外可以通过 `--mirror` 选项使用国内源进行安装：

> 若你想安装测试版的 Docker, 请从 test.docker.com 获取脚本

```
# $ curl -fsSL test.docker.com -o get-docker.sh
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
# $ sudo sh get-docker.sh --mirror AzureChinaCloud
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 的稳定(stable)版本安装在系统中。

### 2.5 启动 Docker

```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

### 2.6 建立 docker 用户组

默认情况下，`docker` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```
$ sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

### 2.7 验证 Docker 是否安装成功

```
$ docker run --rm hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

若能正常输出以上信息，则说明安装成功。

### 2.8 你可能会遇到的问题

如果在 CentOS 使用 Docker 看到下面的这些警告信息：

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

请添加内核配置参数以启用这些功能。

```
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后重新加载 `sysctl.conf` 即可

```
$ sudo sysctl -p
```


## 3 修改镜像源

Docker 默认情况下是从官方 Docker Hub 下载镜像，但是由于 Docker Hub 服务器在国外的缘故，国内用户在下载镜像的时候，会非常慢。

想要解决这个问题，我们可以配置国内的镜像加速器，如阿里云加速器、DaoCloud、网易加速器、百度云加速器等等。

接下来，小哈将演示如何配置阿里云镜像加速器。

### 3.1 登录阿里云，进入控制台

阿里云官网：[https://home.console.aliyun.com/home/dashboard/ProductAndService](https://home.console.aliyun.com/home/dashboard/ProductAndService)

进入到阿里云控制台后，搜索关键字【**镜像服务**】, 点击【**容器镜像服务**】：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a46d6c929ed99f7be75ecbfc181b6b9c.png)

### 3.2 复制镜像服务器地址

点击左边菜单栏【**镜像加速器**】，复制加速器地址：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/8e0e5dbfa6d9bbdcd5a43f7dbb117c89.png)

### 3.3 开始配置镜像加速器

**注意，将下面配置中的镜像加速器地址改为你自己的。**
#### 3.3.1 CentOS

##### 3.3.1.1 安装／升级 Docker 客户端

推荐安装 1.10.0 以上版本的 Docker 客户端，参考文档 [docker-ce](https://yq.aliyun.com/articles/110806)

##### 3.3.1.2 配置镜像加速器

针对 Docker 客户端版本大于 1.10.0 的用户

您可以通过修改 daemon 配置文件 `/etc/docker/daemon.json` 来使用加速器

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{"registry-mirrors": ["https://ukggsdf6.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 3.3.2 Windows

##### 3.3.2.1 安装／升级 Docker 客户端

对于 Windows 10 以下的用户，推荐使用 Docker Toolbox

Windows 安装文件：[http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)

对于 Windows 10 以上的用户 推荐使用 Docker for Windows

Windows 安装文件：[http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/](http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/)

##### 3.3.2.2 配置镜像加速器

==针对安装了 Docker Toolbox 的用户，您可以参考以下配置步骤==：

创建一台安装有 Docker 环境的 Linux 虚拟机，指定机器名称为 default，同时配置 Docker 加速器地址。

```
docker-machine create --engine-registry-mirror=https://xxx.mirror.aliyuncs.com -d virtualbox default
```

查看机器的环境配置，并配置到本地，并通过 Docker 客户端访问 Docker 服务。

```
docker-machine env defaulteval "$(docker-machine env default)"docker info
```

==针对安装了 Docker for Windows 的用户，您可以参考以下配置步骤==：

在系统右下角托盘图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Docker Daemon。编辑窗口内的 JSON 串，填写下方加速器地址：

```
{"registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
```

编辑完成后点击 Apply 保存按钮，等待 Docker 重启并应用配置的镜像加速器。

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/208f2850deb9f2c9812b3fef3bf58476.png)


##### 3.3.2.3 注意

Docker for Windows 和 Docker Toolbox 互不兼容，如果同时安装两者的话，需要使用 hyperv 的参数启动。

```
docker-machine create --engine-registry-mirror=https://xxx.mirror.aliyuncs.com -d hyperv default
```

Docker for Windows 有两种运行模式，一种运行 Windows 相关容器，一种运行传统的 Linux 容器。同一时间只能选择一种模式运行。