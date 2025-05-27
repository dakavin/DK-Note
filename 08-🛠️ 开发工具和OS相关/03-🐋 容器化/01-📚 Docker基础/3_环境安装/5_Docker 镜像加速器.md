---
文章标题: "[[5_Docker 镜像加速器]]" 
文章作者: Dakkk
文章概要: |
  文章旨在解决国内用户从Docker Hub下载镜像慢的问题。通过配置Docker镜像加速器，以阿里云为例，详细演示了在CentOS、Mac、Windows、Ubuntu等系统上配置加速器的具体步骤，有效提升Docker镜像的拉取效率。
tags:
- "Docker"
- "镜像加速器"
- "阿里云"
- "Docker Hub"
- "配置"
- "Linux"
- "macOS"
- "Windows"
相关文章:
- "[[2_安装与卸载]]"
- "[[9_补充：安装Docker（本地和Linux）]]"
- "[[1_Linux下的MySQL]]"
- "[[1_MinGW编译器的安装和配置]]"
- "[[2_RocketMQ安装]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/3_环境安装/5_Docker 镜像加速器.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

Docker 默认情况下是从官方 Docker Hub 下载镜像，但是由于 Docker Hub 服务器在国外的缘故，国内用户在下载镜像的时候，会非常慢。

想要解决这个问题，我们可以配置国内的镜像加速器，如阿里云加速器、DaoCloud、网易加速器、百度云加速器等等。

接下来，小哈将演示如何配置阿里云镜像加速器。

## 1 登录阿里云，进入控制台

阿里云官网：[https://home.console.aliyun.com/home/dashboard/ProductAndService](https://home.console.aliyun.com/home/dashboard/ProductAndService)

进入到阿里云控制台后，搜索关键字【**镜像服务**】, 点击【**容器镜像服务**】：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a46d6c929ed99f7be75ecbfc181b6b9c.png)

## 2 复制镜像服务器地址

点击左边菜单栏【**镜像加速器**】，复制加速器地址：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/8e0e5dbfa6d9bbdcd5a43f7dbb117c89.png)

## 3 开始配置镜像加速器

**注意，将下面配置中的镜像加速器地址改为你自己的。**
### 3.1 CentOS

#### 3.1.1 安装／升级 Docker 客户端

推荐安装 1.10.0 以上版本的 Docker 客户端，参考文档 [docker-ce](https://yq.aliyun.com/articles/110806)

#### 3.1.2 配置镜像加速器

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

### 3.2 Mac

#### 3.2.1 安装／升级 Docker 客户端

对于 10.10.3 以下的用户 推荐使用 Docker Toolbox

Mac 安装文件：[http://mirrors.aliyun.com/docker-toolbox/mac/docker-toolbox/](http://mirrors.aliyun.com/docker-toolbox/mac/docker-toolbox/)

对于 10.10.3 以上的用户 推荐使用 Docker for Mac

Mac 安装文件：[http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/](http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/)

#### 3.2.2 配置镜像加速器

针对安装了 Docker Toolbox 的用户，您可以参考以下配置步骤：

创建一台安装有 Docker 环境的 Linux 虚拟机，指定机器名称为 default，同时配置 Docker 加速器地址。

```
docker-machine create --engine-registry-mirror=https://xxx.mirror.aliyuncs.com -d virtualbox default
```

查看机器的环境配置，并配置到本地，并通过 Docker 客户端访问 Docker 服务。

```
docker-machine env defaulteval "$(docker-machine env default)"docker info
```

针对安装了 Docker for Mac 的用户，您可以参考以下配置步骤：

在任务栏点击 Docker Desktop 应用图标 -> Perferences，在左侧导航菜单选择 Docker Engine，在右侧输入栏编辑 json 文件。将

https://xxx.mirror.aliyuncs.com 加到 "registry-mirrors" 的数组里，点击 Apply & Restart 按钮，等待 Docker 重启并应用配置的镜像加速器。

![Windows 配置阿里云镜像加速器](https://img.quanxiaoha.com/quanxiaoha/170729531681236 "Windows 配置阿里云镜像加速器")Windows 配置阿里云镜像加速器

![Mac 配置阿里云加速器](https://img.quanxiaoha.com/quanxiaoha/165629707515382 "Mac 配置阿里云加速器")Mac 配置阿里云加速器

### 3.3 Windows

#### 3.3.1 安装／升级 Docker 客户端

对于 Windows 10 以下的用户，推荐使用 Docker Toolbox

Windows 安装文件：[http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)

对于 Windows 10 以上的用户 推荐使用 Docker for Windows

Windows 安装文件：[http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/](http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/)

#### 3.3.2 配置镜像加速器

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


#### 3.3.3 注意

Docker for Windows 和 Docker Toolbox 互不兼容，如果同时安装两者的话，需要使用 hyperv 的参数启动。

```
docker-machine create --engine-registry-mirror=https://xxx.mirror.aliyuncs.com -d hyperv default
```

Docker for Windows 有两种运行模式，一种运行 Windows 相关容器，一种运行传统的 Linux 容器。同一时间只能选择一种模式运行。

### 3.4 Ubuntu

#### 3.4.1 安装／升级 Docker 客户端

推荐安装 1.10.0 以上版本的 Docker 客户端，参考文档 [docker-ce](https://yq.aliyun.com/articles/110806)

#### 3.4.2 配置镜像加速器

针对 Docker 客户端版本大于 1.10.0 的用户

您可以通过修改 daemon 配置文件 /etc/docker/daemon.json 来使用加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{"registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```