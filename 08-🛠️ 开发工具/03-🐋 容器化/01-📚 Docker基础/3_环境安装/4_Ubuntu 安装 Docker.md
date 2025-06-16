---
文章标题: "[[4_Ubuntu 安装 Docker]]"
文章作者: Dakkk
文章概要: |
  本文详细介绍了在Ubuntu上安装Docker的多种方法，包括APT（推荐）、手动deb包和脚本安装，并提供了系统前置条件、用户组配置及安装验证步骤，旨在帮助用户快速部署Docker环境。
tags:
  - Docker
  - Ubuntu
  - 安装
  - APT
  - Linux
  - 容器
  - 系统配置
  - Docker Compose
相关文章:
  - "[[1_开发环境搭建]]"
  - "[[4_apt命令及问题]]"
  - "[[../8_Docker Compose/2_安装与卸载]]"
  - "[[../5_容器/3_进入容器]]"
  - "[[2_Linux入门基础]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/3_环境安装/4_Ubuntu 安装 Docker.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

想在 Ubuntu 系统上安装 Docker，需要满足一些前置条件。

## 1 前置条件

### 1.1 操作系统要求

Docker 支持以下版本的 Ubuntu `64` 位操作系统：

- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Impish 21.10
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

CPU 架构要求：`x86_64` (或 `amd64`), `armhf`, `arm64`, 和 `s390x` 均支持 Docker 安装。

### 1.2 卸载旧版本

如果你之前安装过老版本的 Docker , 请先执行如下面命令卸载：

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 2 开始安装 Docker

Ubuntu 支持如下几种方法安装 Docker:

1. 通过 `apt` 安装，后续升级更方便（推荐方法）；
2. 手动下载 `.deb` 包安装，完全手动管理升级；
3. 使用脚本自动安装 Docker, 适用于测试、开发环境中；

### 2.1 使用 APT 安装（推荐方式）

1. 更新 `apt` 包索引：

```
sudo apt-get update
```

2. 由于 `apt` 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们需要添加使用 HTTPS 传输的软件包以及 CA 证书。

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3. 为了确认所下载软件包的合法性，需要添加 Docker 的 GPG 密钥，为了确保下载速度，走的国内源，注释的为国外源：

```
sudo mkdir -p /etc/apt/keyrings
# 国内源
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg


# 官方源
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

4. 向 `sources.list` 中添加 Docker 软件源:

```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# 官方源
#  echo \
#   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
#   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 2.1.1 安装 Docker

更新 apt 软件包索引，安装最新版本的 Docker Engine、containerd 和 Docker Compose：

```
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

> 注意： 运行 `apt-get update` 报 GPG 错误？
> 
> ​ 你默认的 umask 可能设置不正确，导致无法检测到 repo 的公钥文件。运行以下命令，然后再次尝试更新 apt 索引：`sudo chmod a+r /etc/apt/keyrings/docker.gpg`.

##### 2.1.1.1 如何安装指定版本的 Docker ?

1. 要想安装特定版本的 Docker, 需要先获取 repo 中可用的版本号，然后再安装：

```
apt-cache madison docker-ce

docker-ce | 5:20.10.16~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.15~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.14~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.13~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
```

2. 第二列中显示的即为版本号，如`5:20.10.16~3-0~ubuntu-jammy`.

```
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
```

### 2.2 通过 `.deb` 包手动安装

如果你无法通过 `apt` 安装 Docker, 还可以手动下载指定版本的 `.deb` 包来安装。注意，每次升级时，同样需要下载一个新的 `.deb` 包。

1. 访问 [https://download.docker.com/linux/ubuntu/dists/](https://download.docker.com/linux/ubuntu/dists/) ， 选择你的 Ubuntu 版本，然后进入 `pool/stable/`, 选择对应的 CPU 架构： `amd64`、 `armhf`、`arm64`或`s390x`，下载`.deb` 安装包。
    
2. 安装 Docker, 将下面的路径更改为你下载的 `.deb` 包路径:
    
    ```
    sudo dpkg -i /path/to/package.deb
    ```
    

### 2.3 使用脚本自动安装 Docker

在测试或开发环境中， Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套脚本安装，另外可以通过 `--mirror` 选项使用国内源进行安装：

```
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

> 若你想安装测试版的 Docker, 可以从 test.docker.com 获取脚本后再执行它：
> 
> ```
> $ curl -fsSL test.docker.com -o get-docker.sh
> ```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 的稳定(stable)版本安装在 Ubuntu 系统中。

## 3 启动 Docker

```
# 设置 Docker 服务开机自动启动
sudo systemctl enable docker
# 启动 Docker 服务
sudo systemctl start docker
```

## 4 建立 Docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。

出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

1. 建立 `docker` 组：

```
sudo groupadd docker
```

2. 将当前用户加入 `docker` 组：

```
sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

## 5 验证 Docker 是否安装成功

运行 Docker 并打印 `hello-world`:

```
sudo docker run hello-world
```

此命令会下载测试镜像，并基于此镜像运行容器，然后在打印 `hello-world` 后退出容器。若成功打印信息，表示 Docker 安装成功。