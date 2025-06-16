---
文章标题: "[[3_Dockerfile常用指令汇总]]"
文章作者: Dakkk
文章概要: |
  本文汇总了 Dockerfile 常用指令，涵盖文件复制、容器启动、环境变量、数据卷、端口暴露、健康检查等。详细阐述各指令语法、功能、区别与最佳实践，旨在帮助读者高效构建和管理 Docker 镜像。
tags:
  - Docker
  - Dockerfile
  - 镜像构建
  - 容器化
  - CMD
  - ENTRYPOINT
  - COPY
相关文章:
  - "[[../../02-🚀 Docker进阶/1_SpringBoot项目部署（Docker）]]"
  - "[[15_使用Docker构建根文件系统（Skip）]]"
  - "[[../8_Docker Compose/3_实战web服务]]"
  - "[[../../../04-☁️ 运维部署/02-🤖 自动化部署/3_姿势2：Docker部署项目]]"
  - "[[../../../04-☁️ 运维部署/02-🤖 自动化部署/8_补充：本地使用Docker安装Jenkins]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/7_Dockerfile/3_Dockerfile常用指令汇总.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 1 COPY 复制文件

**复制指令，支持从上下文目录中复制文件或者文件夹到容器里的指定路径。**

`COPY` 指令同 `RUN` 指令一样，也支持两种命令格式：

```Dockerfile
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
或者
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

参数说明：
- `[--chown=<user>:<group>]` : 可选参数，可以改变被复制文件或文件夹在容器中的所属用户和所属组；
- `源路径` : 源文件或者源文件夹，支持通配符表达式，规则需要满足 Go 的 [filepath.Match](https://golang.org/pkg/path/filepath/#Match) 规则，例如：

```Dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

> 注意： 如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

- `目标路径` : 复制到容器内的指定路径，无需提前创建好，路径不存在的话，会自动创建。

> 注意：目标路径可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。

## 2 ADD 更高级的复制文件

`ADD` 指令与 `COPY` 指令功能类似，都可以复制文件或文件夹（同样的需求下，官方推荐使用 `COPY` 指令）。格式同样支持两种：

```Dockerfile
ADD [--chown=<user>:<group>] <源路径1>...  <目标路径>
或者
ADD [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

不同的是 ， `ADD` 指令在 `COPY` 的基础之上加了一些功能：

- 1、比如 `<源路径>` 可以是一个 URL, 这种情况下，Docker 引擎会去下载 URL 对应的文件并放到 `<目标路径>` 下。
- 2、如果 `<源路径>` 是一个压缩文件，如 `tar` 、`gzip` 、 `bzip2` 、`xz` 等， `ADD` 指令将自动解压此文件到 `<目标路径>` 下。

```Dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```

### 2.1 最佳实践

Docker 官方文档中更推荐使用 `COPY` , 因为 `COPY` 语义更加明确，乍一看就知道是复制文件，而 `ADD` 则包含了很多复杂的功能，行为不够清晰。另外，`ADD` 指令会让构建缓存失效，从而会让镜像构建变得缓慢。

那么，什么时候推荐使用 `ADD` 呢？
- 当你有自动解压缩的需求时，适合使用 `ADD` 指令。
## 3 CDM 容器启动命令

**`CMD` 指令用于启动容器时，指定需要运行的程序以及参数**。使用格式与 `RUN` 指令类似：

```Dockerfile
CMD <shell 命令> 
CMD ["<可执行文件>", "<参数1>", "<参数2>", ...] # 官方推荐格式
CMD ["<参数1>", "<参数2>", ...]  # 此种写法在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数。
```

官方更推荐使用第二种 `exec` 格式，此种格式在解析时会被解析成 JSON 数组，因此一定要使用双引号 `"` , 而不是单引号 `'` 。

如果你使用第一种 `shell` 格式，最终还是会转成第二种格式，实际命令会被包装成 `sh -c` 的参数进行执行，比如：

```Dockerfile
CMD echo $HOME
```

会被转成：

```Dockerfile
CMD [ "sh", "-c", "echo $HOME" ]
```

### 3.1 CMD 与 RUN 的不同点

`CMD` 与 `RUN` 指令不同点在于二者的运行时间点不同：

- `CMD` 指令在 `docker run` 时运行; （即给存在的容器进行运行的）
- `RUN` 是在 `docker build` 时运行；（即通过Dockerfile构建镜像时运行的）

### 3.2 注意

如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。
## 4 ENTRYPOINT 入口点

`ENTRYPOINT` 的功能和 [CMD](https://www.quanxiaoha.com/docker/dockerfile-cmd.html) 一样，都用于指定容器启动程序以及参数，格式如下：

```Dockerfile
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

> **注意**：如果 Dockerfile 中存在多个 ENTRYPOINT 指令，仅最后一个生效。

### 4.1 ENTRYPOINT 和 CMD 的不同之处

对于 `CMD` 指令， 执行 `docker run` 命令如果有传递参数，这些参数是可以覆盖 Dockerfile 中的`CMD` 指令参数，但是对于 `ENTRYPOINT` 来说，这些参数会被传递给 `ENTRYPOINT`，而不是覆盖。

> 如果想覆盖 Dockerfile 中 `ENTRYPOINT` 参数，需要在执行 `docker run` 命令时使用 `--entrypoint` 选项。

接下来，举例一个场景，让我们更容易理解 `ENTRYPOINT` 的应用场景：

#### 4.1.1 ENTRYPOINT 应用场景

假设我们需要一个打印当前公网 IP 的镜像，那么可以先用 `CMD` 来实现, Dockerfile 文件如下：

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```

然后，执行 `docker build -t myip .` 来构建镜像，如果需要打印当前公网 IP，只需要执行：

```shell
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```

这么看起来好像可以直接把镜像当做命令使用了，不过是命令总有参数，如果我们希望加参数呢？

比如从上面的 `CMD` 中可以看到实质的命令是 `curl`，如果我们希望显示 HTTP 头信息，就需要加上 `-i` 参数。那么我们可以直接加 `-i` 参数给 `docker run myip` 么？

```shell
$ docker run myip -i
```

会报如下错误：

```shell
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```

上面已经说到，`docker run` 传递的参数会替换 `CMD` 的默认值 ，因此这里的 `-i` 替换了原来的 `CMD`，而不是添加在原来的 `curl -s http://myip.ipip.net` 后面， 而 `-i` 根本不是命令，所以自然找不到报错。

那么如果希望支持 `-i` 这参数，我们就必须重新完整的输入这个命令：

```shell
$ docker run myip curl -s http://myip.ipip.net -i
```

这显然不是一个优雅的解决方案，这个时候 `ENTRYPOINT` 就上场了, 因为它可以传递参数，而不是覆盖。现在我们重新用 `ENTRYPOINT` 来实现这个镜像：

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

这次我们再来尝试直接使用 `docker run myip -i`：

```shell
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通

$ docker run myip -i
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Tue, 22 Nov 2016 05:12:40 GMT
Content-Type: text/html; charset=UTF-8
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
X-Cache: MISS from cache-2
X-Cache-Lookup: MISS from cache-2:80
X-Cache: MISS from proxy-2_6
Transfer-Encoding: chunked
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
Connection: keep-alive

当前 IP：61.148.226.66 来自：北京市 联通
```

可以看到，这次成功了。
## 5 ENV 设置环境变量

通过 `ENV` 指令设置环境变量，在后续的指令中，可以直接使用这个环境变量。

使用格式有两种：

```Dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

### 5.1 ENV 使用示例

以下示例中，先通过 `ENV` 指令定义了 `NODE_VERSION` 环境变量，在后续的 `RUN` 指令中使用到了它：

```Dockerfile
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

下列指令均可以支持使用环境变量： `ADD`、`COPY`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`、`RUN` 。
## 6 ARG 构建参数

**`ARG` 指令用于指定构建参数**，与 `ENV` 功能一样，都是设置环境变量。不同点在于作用域不一样, `ARG` 声明的环境变量仅对 Dockerfile 内有效，也就是说仅对 `docker build` 的时候有效，将来容器运行的时候不会存在这些环境变量的。

使用格式如下：
```Dockerfile
ARG <参数名>[=<默认值>]
```

### 6.1 ARG 注意点

#### 6.1.1 脱敏数据安全问题

不要使用 `ARG` 保存密码之类的敏感信息，因为通过 `docker history` 可以看到所有数据。

#### 6.1.2 作用域问题

ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```Dockerfile
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo ${DOCKER_USERNAME}
```

使用上述 Dockerfile 会发现无法输出 `${DOCKER_USERNAME}` 变量的值，要想正常输出，你必须在 `FROM` 之后再次指定 `ARG` ：

```Dockerfile
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

对于多阶段构建，尤其要注意这个问题:

```Dockerfile
# 这个变量在每个 FROM 中都生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 1

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 2
```

对于上述 Dockerfile 两个 `FROM` 指令都可以使用 `${DOCKER_USERNAME}`，对于在各个阶段中使用的变量都必须在每个阶段分别指定：

```Dockerfile
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```
## 7 VOLUME 定义匿名数

`VOLUME` 指令用于定义匿名[1_数据卷](../6_数据管理/1_数据卷.md)。

使用格式如下：

```dockerfile
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

举个例子：

```dockerfile
VOLUMN /data
```

上面的 `/data` 目录在容器运行时会自动挂载为匿名卷，这样可以保证容器存储层不发生写操作，同时，如果用户后面忘记指定挂载，也能保证应用正常运行，从而避免因容器重启导致数据丢失。

当然，运行容器时可以通过 `-v` 覆盖这个挂载匿名数据卷的设置，比如：

```shell
$ docker run -d -v data:/data xxxx
```

通过这行命令，可以将 `data` 数据卷挂载到 `/data` 目录，覆盖了 Dockerfile 中 `VOLUMN /data` 这行挂载匿名数据卷的配置。
## 8 EXPOSE 暴露端口

使用格式如下：

```dockerfile
EXPOSE <端口1> [<端口2>...]
```

`EXPOSE` 指令用于暴露容器运行时提供服务的端口。注意，这仅仅是一个声明，容器实际运行时，并不会开启这个声明的端口。

在 Dockerfile 中写入 `EXPOSE` 指令好处如下：
- 1、帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；
- 2、在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。
### 8.1 EXPOSE 与 docker run -p 有什么区别？

`docker run -p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
## 9 WORKDIR 指定工作目录

**`WORKDIR` 指令用于指定工作目录**，后面各层的当前目录即为 `WORKDIR` 指定的目录，如果该目录不存在，`WORKDIR` 会自动建立目录。

使用格式如下：

```
WORKDIR <工作目录路径>
```

### 9.1 为什么需要 `WORKDIR` ?

初学者在学习 `Dockerfile` 时，会将其当作 Shell 脚本来写，比如：

```
RUN cd /app
RUN echo "hello" > world.txt
```

通过这个 `Dockerfile` 构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者内容不是 `hello` 。为什么会发生这种情况呢？因为在 Shell 中，连续两行命令执行是在同一个进程中；而在 `docker build` 构建镜像过程中，每一个 `RUN` 命令都会新建一层，执行环境是完全不同的。

```
WORKDIR /app
```

定义了 `WORKDIR /app` 后，就可以指定后面每层的工作目录了。
## 10 USER 指定当前用户

`USER` 指令用于指定后续命令的用户身份，注意，用户需事先建立好，否则无法切换。

使用格式如下：
```
USER <用户名>[:<用户组>]
```

示例：
```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```
## 11 HEALTHCHECK 健康检查

**`HEALTHCHECK` 指令用于设置 Docker 要如何判断容器状态是否正常**。Docker 1.12 版本后引入了该指令。

使用格式如下：

```
HEALTHCHECK [选项] CMD <命令> #设置检查容器健康状况的命令
HEALTHCHECK NONE #如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
```

`HEALTHCHECK` 支持下列选项：
- `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
- `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- `--retries=<次数>`：健康检查重试次数，若指定次数依然失败，则将容器状态视为 `unhealthy`，默认 3 次。

当一个镜像指定了 `HEALTHCHECK` 后，初启动容器时，初始状态为 `starting`, 当 `HEALTHCHECK` 指令检查成功后，容器状态会变为 `healthy`，如果重试多次失败，则会变为 `unhealthy`。

> **注意：** 当 Dockerfile 中出现多个 `HEALTHCHECK` 时，只有最后一个生效。

### 使用示例

```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
CMD curl -fs http://localhost/ || exit 1
```

上面的 Dockerfile 中，通过 `HEALTHCHECK` 指令设置了每 5 秒检查一次，超时时间为 3 秒，若 3 秒没有响应则视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。
## 12 ONBUILD 二次构建

`ONBUILD` 是一个延迟执行的特殊指令，它后面允许跟着其他指令，如 `RUN` 、`COPY` 等，这些指令在构建当前镜像时并不会被执行，而是当前镜像构建好了以后，后面再次构建镜像时，以此镜像为基础镜像，二次构建镜像时才会被执行。

使用格式：
```
ONBUILD <其它指令>
```

示例：

```
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```
## 13 LABEL 为镜像添加元数据

`LABEL` 指令用于给镜像添加一些元数据 (metadata)， 数据为键值对的形式，格式如下：

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

比如声明镜像的作者、文档地址等：

```
LABEL org.opencontainers.image.authors="quanxiaoha"

LABEL org.opencontainers.image.documentation="https://www.quanxiaoha.com"
```
## 14 SHELL 指令

`SHELL` 指令可以指定 `RUN` 、`ENTRYPOINT`、`CMD` 指令执行的 shell 命令，Linux 中默认为 `["/bin/sh", "-c"]` 。

使用格式如下：

```
SHELL ["executable", "parameters"]
```

示例：

```
SHELL ["/bin/sh", "-cex"]

# /bin/sh -cex "nginx"
ENTRYPOINT nginx
```

```
SHELL ["/bin/sh", "-cex"]

# /bin/sh -cex "nginx"
CMD nginx
```