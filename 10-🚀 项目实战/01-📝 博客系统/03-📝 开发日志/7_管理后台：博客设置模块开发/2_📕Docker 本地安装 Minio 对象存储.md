---
文章标题: "[[2_📕Docker 本地安装 Minio 对象存储]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了如何使用Docker在本地搭建Minio对象存储服务作为图床。它阐述了Minio的优势，并提供了从Docker镜像下载、数据卷挂载到容器运行、Minio控制台配置桶及公共访问的完整操作步骤，旨在帮助开发者快速搭建本地图片存储解决方案。
文章标签:
- "Minio"
- "Docker"
- "对象存储"
- "S3兼容"
- "图床"
- "本地部署"
- "数据持久化"
相关文章:
- "[[1_数据卷]]"
- "[[2_安装MySQL]]"
- "[[2_数据卷容器]]"
- "[[3_安装Nginx]]"
- "[[8_补充：本地使用Docker安装Jenkins]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/7_管理后台：博客设置模块开发/2_📕Docker 本地安装 Minio 对象存储.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-05-04 21:42:14
修改时间: 2025-05-27 00:57:28
---

因为博客设置模块涉及到图片上传，如上传博客 LOGO 图片、作者头像，以及后续发布文章也==需要支持图片上传==。所以，我们首先需要==搭建一个图床服务==，这里小哈选型的是 Minio 对象存储，它不光可以存储图片，还能存储文件、视频等，非常强大。
## 1. 什么是 MinIO？

MinIO 是一个开源的对象存储服务器。这意味着它允许你在互联网上存储大量数据，比如文件、图片、视频等，而不需要依赖传统的文件系统。MinIO 的特点在于它非常灵活、易于使用，同时也非常强大，可以在你的应用程序中方便地集成。

## 2. 为什么使用 MinIO？

- **可伸缩性和性能：** MinIO 允许你在需要时轻松地扩展存储容量，无需中断服务。它具有出色的性能，可以处理大量的并发读取和写入请求。
    
- **开源和自由：** MinIO 是开源软件，遵循 Apache License 2.0 许可证，这意味着你可以自由地使用、修改和分发它。
    
- **容器化部署：** MinIO 提供了容器化部署的支持，可以在各种平台上快速部署和运行，包括本地开发机、云服务器和容器编排环境（如 Docker）。
    
- **兼容性：** MinIO 提供了 S3 兼容的 API，这意味着它可以与任何兼容 Amazon S3 的应用程序无缝集成，为你的应用程序提供强大的对象存储能力。
    
- **易用性：** MinIO 的配置和管理非常简单，它提供了直观的Web控制台和命令行工具，帮助你方便地管理存储桶和对象。

总的来说，MinIO 是一个灵活、高性能、易用且开源的对象存储解决方案，适用于各种规模的应用程序，特别是那些需要大规模数据存储和访问的项目。

## 3. Docker 搭建 Minio 服务

了解了 `Minio` 是什么后，接下来我们开始安装它。这里小哈使用的 Docker 来安装，更加简单一些。

首先，你需要确保你的机器已经安装成功了 Docker，不清楚如何安装 Docker 的童鞋，可以翻阅前面的[《后端环境安装》](https://www.quanxiaoha.com/column/10003.html) 小节。

### 3.1 选择一个 Minio 镜像

然后，我们在浏览器中访问地址：[https://hub.docker.com/](https://hub.docker.com/) ， 输入关键词 _minio/minio_, 找到 `Minio` 镜像：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e7eaaf3dd11fe19c47eb13aad44f918c.png)


点击进去，点击 _Tags_ 标签选项，小哈这里选择的是最新的一个发行版本：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/fcc78c91510e0adfeb179f9c6c63389c.png)


### 3.2 下载 Minio 镜像

_点击右侧复制命令_，打开命令行，执行该命令拉取镜像：

```docker
docker pull minio/minio:RELEASE.2024-05-01T01-11-10Z
```

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ce8c6e469dda4f3c4be58f5a42c8f462.png)


镜像下载成功后，执行 `docker images` , 如果列表中有 `minio/minio` 镜像，则表示镜像下载成功了：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c076d653ec1de2b77aa1639deaafefb5.png)

### 3.3 新建数据挂载目录

下载镜像成功后，我们在某个盘下，这里选择的是 `E:` 盘，新建一个 `/docker` 文件夹，然后在该文件夹中再新建一个 `/minio` 文件夹，如下图所示：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/aafe7220619ea3df7b8d2aaf1c8061d6.png)

新建该文件夹的目的是，后面通过镜像运行 `Minio` 容器时，可以将容器内的数据目录，挂载到宿主机的 `E:\docker-data\minio` 目录下，防止容器重启后，会导致数据丢失的问题。

### 3.4 运行 Docker Minio 容器

然后，通过该镜像运行 `Minio` 容器，命令如下：

```
docker run -d \
   -p 9000:9000 \
   -p 9090:9090 \
   --name minio \
   -v E:\docker-data\minio\data:/data \
   -e "MINIO_ROOT_USER=dakkk" \
   -e "MINIO_ROOT_PASSWORD=123456" \
   minio/minio:RELEASE.2024-05-01T01-11-10Z server /data --console-address ":9090"
```

注意，执行的时候需要将 `\` 替换成空格，放到一行中来执行，最终命令如下：

```
docker run -d -p 9000:9000 -p 9090:9090 --name minio -v E:\docker-data\minio\data:/data -e "MINIO_ROOT_USER=dakkk" -e "MINIO_ROOT_PASSWORD=123456" minio/minio:RELEASE.2024-05-01T01-11-10Z server /data --console-address ":9090"
```

解释一下上述命令各选项的含义：

- `docker run`: 运行 Docker 容器的命令。
- `-d` : 表示后台运行该容器；
- `-p 9000:9000`: 将宿主机的 9000 端口映射到容器的 9000 端口。MinIO 默认的 HTTP API 端口是 9000。
- `-p 9090:9090`: 将宿主机的 9090 端口映射到容器的 9090 端口。这是 MinIO 的 Web 控制台的端口。
- `--name minio`: 给容器取了一个名字，这里是 "minio"。
- `-v E:\docker\minio\data:/data`: 将宿主机上的 `E:\docker\minio\data` 目录映射到容器内的 `/data`目录。这是 MinIO 存储数据的地方。如果你希望数据在容器删除后仍然保存，可以将数据目录映射到宿主机。
- `-e "MINIO_ROOT_USER=quanxiaoha"`: 设置 MinIO 的管理员用户名为 "quanxiaoha"。这是用于 MinIO Web 控制台和 API 的初始管理员用户名。
- `-e "MINIO_ROOT_PASSWORD=quanxiaoha"`: 设置 MinIO 的管理员密码为 "quanxiaoha"。这是用于 MinIO Web 控制台和 API 的初始管理员密码。
- `minio/minio:RELEASE.2023-09-30T07-02-29Z`: 这是 MinIO 的 Docker 镜像版本。
- `server /data --console-address ":9090"`: 启动 MinIO 服务器，并将数据存储在容器内的`/data`目录。`--console-address ":9090"`表示 MinIO 的Web 控制台将在容器的 9090 端口上运行。

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d0fadd039e8bde16aecaab4d78c8a747.png)

执行该命令后，再执行 `docker ps` 命令，可查看正在运行的容器，若如下图所示，容器列表中出现了 `minio` ，则表示 `Minio` 后台运行成功了：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/494e04a5c992de8a795b0fc2b9ea339a.png)


> TIP : 如果 `Minio` 容器运行未成功，就需要通过日志来定位问题了，可以重新执行 `docker run` 命令，并将 `-d` 参数去掉，以前台的方式运行容器，即可看到启动日志了。
> 
> 账户和密码也是有要求的
> ![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/824a93d87bdca22ff528bd254b9ba4ae.png)


## 4. 访问 Minio 控制台

浏览器访问地址 `http://localhost:9090` ，可访问 MinIO 的 Web 控制台：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d420ff23313242ae5cb16de41a9030cb.png)

输入运行容器时，指定的用户名/密码：`mikeylay/abc123456` , 进入到 `Minio` 的管理后台

## 5. 新建一个 Bucket 桶

进入后台后，点击 _Create a Bucket_ 创建一个 `Bucket` 桶，用于存储图片：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ff292506e953034dd812cace713d7f3d.png)


输入 Bucket Name, 我们将其命名为 _weblog_， 然后点击 _Create Bucket_ 按钮：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c3c21e2557226db6a20840a7e2ac35e2.png)


创建成功后，在 Buckets 列表中就可以看到刚刚新建的桶了：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/1b2aaabdd542c3011d5cde7b24124f6b.png)

## 6. 设置 Bucket 为公共读

因为我们上传的图片需要被公网访问到，所以，还需要设置 `Bucket` 为公共读，默认为 `Private` 私有。点击想要设置的桶，然后编辑 `Access Policy`:

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/f7f5d0cb76cf70bebd0ed6b9963642fa.png)


将 Access Policy 选项选择为 _Public_ 公共读，点击 _Set_ 设置按钮

设置成功后，就可以看到 Access Policy 一栏变更为 _Public_ 了：

## 7. 上传一张图片

相关设置完成后，我们直接在后台上传一张本地的图片，测试看看能够正常上传成功。点击 _Object Browser -> Upload_ 上传图片：

![](https://img.quanxiaoha.com/quanxiaoha/169660482349794)


可以看到上传成功了：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/641ddbb6960475cc5b29705b2d5b5036.png)


接下来，我们直接在浏览器中，来访问该图片的直链：`http://127.0.0.1:9000/weblog/111.jpg` , 看看能否被正常访问：

> 💡 TIP: 本地访问路径格式为 _请求地址:端口号 + 桶名称 + 图片的名称_，上线后会申请域名，格式为 _域名 + 桶名称 + 图片的名称_，例如 https://img.quanxiaoha.com/weblog/111.jpg。

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/2ba6100976ff6898c07217e51a31df0e.png)


OK , 可以看到该图片能够被正常访问。到这里，本地的 `Minio` 对象存储服务就搭建好了，后续博客中的相关图片，都会上传到 `Minio` 中，然后，数据库中会直接存储图片的直链地址。

## 8. 结语

本小节中，小哈带着大家通过 Docker 容器，在本地环境中，将 Minio 对象存储服务搭建起来了，以作图床使用。 这样，也可以方便的进行本地图片上传，当然，如果小伙伴们有服务器，也可以安装 Linux 服务器中来安装使用。在项目最终上线时，小哈会再次演示如何在 Linux 服务器中来安装它。