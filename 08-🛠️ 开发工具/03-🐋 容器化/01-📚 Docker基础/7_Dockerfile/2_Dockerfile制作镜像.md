---
文章标题: "[[2_Dockerfile制作镜像]]"
文章作者: Dakkk
文章概要: |
  本文详细讲解如何使用 Dockerfile 定制 Docker 镜像，通过 Nginx 示例介绍了 `FROM` 和 `RUN` 指令。阐述了镜像分层、构建上下文概念，并提供了优化镜像大小的建议。适合初学者快速上手 Docker 镜像构建。
tags:
  - Docker
  - Dockerfile
  - 镜像构建
  - FROM指令
  - RUN指令
  - Docker上下文
  - 镜像优化
  - Nginx定制
相关文章:
  - "[[3_Dockerfile常用指令汇总]]"
  - "[[15_使用Docker构建根文件系统（Skip）]]"
  - "[[../../02-🚀 Docker进阶/1_SpringBoot项目部署（Docker）]]"
  - "[[0_导论]]"
  - "[[02_不同LubanCat_SDK对比]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/7_Dockerfile/2_Dockerfile制作镜像.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

本小节中，我们将上手通过 Dockerfile 制作第一个镜像，此镜像也非常简单，即定制一个 Nginx 镜像，唯一不同的是，我们需要将 Nginx 默认的首页欢迎语更改为 `Hello, Nginx by Docker!`。
## 1 开始制作镜像

新建一个空白目录，创建一个名为 `Dockerfile` 的文本文件：

```shell
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

编辑 `Dockerfile`，添加如下指令：

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Nginx by Docker!</h1>' > /usr/share/nginx/html/index.html
```

这个 `Dockerfile` 非常简单，总共也就运用了两条指令：`FROM` 和 `RUN` 。

## 2 FROM 指定基础镜像

制作镜像必须要先声明一个基础镜像，基于基础镜像，才能在上层做定制化操作。

通过 **`FROM`指令可以指定基础镜像**，在 Dockerfile 中，`FROM` 是必备指令，且必须是第一条指令。比如，上面编写的 Dockerfile 文件第一行就是 `FROM nginx`, 表示后续操作都是基于 Ngnix 镜像之上。

### 2.1 特殊的镜像：scratch

通常情况下，基础镜像在 DockerHub 都能找到，如：

- **中间件相关**：`nginx`、`kafka`、`mongodb`、`redis`、`tomcat` 等；
- **开发语言环境** ：`openjdk`、`python`、`golang` 等；
- **操作系统**：`centos` 、`alpine` 、`ubuntu` 等；

除了这些常用的基础镜像外，还有个比较特殊的镜像 : `scratch` 。它表示一个空白的镜像：

```dockerfile
FROM scratch
...
```

以 `scratch` 为基础镜像，表示你不以任何镜像为基础。

## 3 RUN 执行命令

`RUN` 指令用于执行终端操作的 shell 命令，另外，`RUN` 指令也是编写 Dockerfile 最常用的指令之一。它支持的格式有如下两种：

- **1、`shell` 格式**: `RUN <命令>`，这种格式好比在命令行中输入的命令一样。举个栗子，上面编写的 Dockerfile 中的 `RUN` 指令就是使用的这种格式：

```dockerfile
RUN echo '<h1>Hello, Nginx by Docker!</h1>' > /usr/share/nginx/html/index.html
```

- **2、`exec` 格式**: `RUN ["可执行文件", "参数1", "参数2"]`, 这种格式好比编程中调用函数一样，指定函数名，以及传入的参数。

```dockerfile
RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

### 3.1 新手构建镜像的体积太大？太臃肿？

初学 Docker 的小伙伴往往构建出来的镜像体积非常臃肿，这是什么原因导致的？

我们知道，Dockerfile 中每一个指令都会新建一层，过多无意义的层导致很多运行时不需要的东西，都被打包进了镜像内，比如编译环境、更新的软件包等，这就导致了构建出来的镜像体积非常大。

举个例子：

```dockerfile
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```

执行以上 Dockerfile 会创建 3 层，另外，下载的 `redis.tar.gz`也没有删除掉，可优化成下面这样：

```dockerfile
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz \
    && rm redis.tar.gz
```

如上，仅仅使用了一个 `RUN` 指令，并使用 `&&` 将各个命令串联起来。之前的 3 层被简化为了 1 层，同时删除了无用的压缩包。

> Dockerfile 支持 shell 格式命令末尾添加 `\` 换行，以及行首通过 `#` 进行注释。保持良好的编写习惯，如换行、注释、缩进等，可以让 Dockerfile 更易于维护。

## 4 构建镜像

Dockerfile 文件编写好了以后，就可以通过它构建镜像了。接下来，我们来构建前面定制的 nginx 镜像，==首先，进入到该 Dockerfile 所在的目录下，执行如下命令==：

```
docker build -t nginx:test .
```

> 注意：命令的最后有个点 `.` , 很多小伙伴不注意会漏掉，`.`指定**上下文路径**，也表示在当前目录下。

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/3719a04a44e78d894c2a68a5cee03bfb.png)


构建命令执行完成后，执行 `docker images` 命令查看本地镜像是否构建成功：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/67256a620e5ec1fff1d6458c39e6afcc.png)

镜像构建成功后，运行 Nginx 容器：

```
docker run -d -p 80:80 --name nginx nginx:test
```

容器运行成功后，访问 `localhost:80`, 可以看到首页已经被成功修改了：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ff80e8991c9a50d214a9120665d142d8.png)

## 5 镜像构建之上下文路径

前面的构建命令最后有一个 `.`, 它表示上下文路径，它又是个啥？

```
docker build -t nginx:test .
```

理解它之前，我们要知道 Docker 在运行时分为 Docker 引擎和客户端工具，是一种 C/S 架构。看似我们在命令收入了一行 Docker 命令，立即就执行了，背后其实是将命令提交给了客户端，然后客户端通过 API 与 Docker 引擎交互，==真正干活的其实是 Docker 引擎==。

话说回来，在构建镜像时，经常会需要通过 `COPY` 、`ADD` 指令将一些本地文件复制到镜像中。而刚才我们也说到了，执行 `docker build` 命令并非直接在本地构建，而是通过 Docker 引擎来完成的，那么要如何解决 Docker 引擎获取本地文件的问题呢？

于是引入了上下文的概念。构建镜像时，指定上下文路径，客户端会将路径下的所有内容打包，并上传给 Docker 引擎，这样它就可以获取构建镜像所需的一切文件了。

> 注意：上下文路径下不要放置一些无用的文件，否则会导致打包发送的体积过大，速度缓慢而导致构建失败。当然，我们也可以想编写 `.gitignore` 一样的语法写一个 `.dockerignore`, 通过它可以忽略上传一些不必要的文件给 Docker 引擎。

## 6 `docker build` 的其他用法

### 6.1 通过 Git repo 构建镜像

除了通过 Dockerfile 来构建镜像外，还可以直接通过 URL 构建，比如从 Git repo 中构建：

```shell
# $env:DOCKER_BUILDKIT=0
# export DOCKER_BUILDKIT=0

$ docker build -t hello-world https://github.com/docker-library/hello-world.git#master:amd64/hello-world

Step 1/3 : FROM scratch
 --->
Step 2/3 : COPY hello /
 ---> ac779757d46e
Step 3/3 : CMD ["/hello"]
 ---> Running in d2a513a760ed
Removing intermediate container d2a513a760ed
 ---> 038ad4142d2b
Successfully built 038ad4142d2b
```

上面的命令指定了构建所需的 Git repo, 并且声明分支为 `master`, 构建目录为 `amd64/hello-world`。运行命令后，Docker 会自行 `git clone` 这个项目，切换分支，然后进入指定目录开始构建。

### 6.2 通过 tar 压缩包构建镜像

```
$ docker build http://server/context.tar.gz
```

如果给定的 URL 是个 `tar` 压缩包，那么 Docker 会自动下载这个压缩包，并自动解压，以其作为上下文开始构建。

### 6.3 从标准输入中读取 Dockerfile 进行构建

```
docker build - < Dockerfile
```

或

```
cat Dockerfile | docker build -
```

标准输入模式下，如果传入的是文本文件，Docker 会将其视为 `Dockerfile`，并开始构建。需要注意的是，这种模式是没有上下文的，它无法像其他方法那样将本地文件通过 `COPY` 指令打包进镜像。

### 6.4 从标准输入中读取上下文压缩包进行构建

```
$ docker build - < context.tar.gz
```

标准输入模式下，如果传入的是压缩文件，如 `tar.gz` 、`gzip` 、 `bzip2` 等，Docker 会解压该压缩包，并进入到里面，将里面视为上下文，然后开始构建。