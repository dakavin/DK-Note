
Docker Repository 用于镜像的集中存储、分发的地方。有了它，镜像在构建完成后，在其他机器上就可以非常方便的下载使用这个镜像了。

一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，==一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本==。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签，表示最新的一个版本。

以 Ubuntu 镜像为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，`16.04`, `18.04`。我们可以通过 `ubuntu:16.04`，或者 `ubuntu:18.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

仓库名经常以 _两段式路径_ 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

## 1 公有仓库

公有仓库是允许用户免费上传、下载的公开镜像服务。比如官方的 [Docker Hub](https://hub.docker.com/) ，也是默认的 Docker Repository，里面拥有着大量的高质量镜像。但是国内访问它可能比较慢，国内的云服务商提供了针对 Docker Hub 的镜像服务（`Registry Mirror`），这些镜像服务被称为**镜像加速器**。国内常见有阿里云加速器、网易加速器、DaoCloud 加速器等。

## 2 私有仓库

除了公有仓库外，用户还可以在本地搭建私有仓库。Docker 官方提供了 [Docker Registry](https://hub.docker.com/_/registry/) 镜像，可以直接使用做为私有 Registry 服务。

开源的 Docker Registry 镜像只提供了 [Docker Registry API](https://docs.docker.com/registry/spec/api/) 的服务端实现，足以支持 `docker` 命令，不影响使用。但不包含图形界面，以及镜像维护、用户管理、访问控制等高级功能。

除了官方的 Docker Registry 外，还有第三方软件实现了 Docker Registry API，甚至提供了用户界面以及一些高级功能。比如，Harbor 和 Sonatype Nexus。