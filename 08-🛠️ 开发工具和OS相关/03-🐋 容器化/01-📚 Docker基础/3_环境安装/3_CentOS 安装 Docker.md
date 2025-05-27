---
文章标题: "[[3_CentOS 安装 Docker]]" 
文章作者: Dakkk
文章概要: |
  本文详细指导如何在CentOS 7/8上安装Docker。内容包括卸载旧版、通过yum或脚本安装Docker CE、配置国内源、处理CentOS 8防火墙问题、启动服务、配置用户组、安装验证及常见问题解决。
tags:
- "CentOS"
- "Docker"
- "安装"
- "yum"
- "Linux"
- "容器"
- "Docker CE"
- "系统配置"
相关文章:
- "[[4_Ubuntu 安装 Docker]]"
- "[[3_进入容器]]"
- "[[9_补充：安装Docker（本地和Linux）]]"
- "[[1_初识Linux]]"
- "[[1_开发环境搭建]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/3_环境安装/3_CentOS 安装 Docker.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 1 准备工作

### 1.1 操作系统要求

Docker 支持以下 64 位的 CentOS 版本, 且内核版本不低于 3.10：

- CentOS 7
- CentOS 8
- 更高版本...

### 1.2 卸载旧版本

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

## 2 开始安装

### 2.1 使用 yum 安装

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

#### 2.1.1 安装 Docker

更新 `yum` 软件源缓存，并安装 `docker-ce`。

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

## 3 CentOS8 额外设置

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

## 4 使用脚本安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装，另外可以通过 `--mirror` 选项使用国内源进行安装：

> 若你想安装测试版的 Docker, 请从 test.docker.com 获取脚本

```
# $ curl -fsSL test.docker.com -o get-docker.sh
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
# $ sudo sh get-docker.sh --mirror AzureChinaCloud
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 的稳定(stable)版本安装在系统中。

## 5 启动 Docker

```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

## 6 建立 docker 用户组

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

## 7 验证 Docker 是否安装成功

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

## 8 你可能会遇到的问题

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