
因为个人对nginx理解有限，这里直接给出方案

使用这个方案之前，需要停止之前的home项目，因为这个home项目挂载了nginx镜像，端口为80，占用了我们后续运行的nginx容器
```shell
# 暂停home容器（二选一）
docker stop home
# 删除home容器（二选一，建议直接删除）
docker rm -f home
```

后端项目的容器不需要停止/删除，放着就好了
## 1 思路

第一步：我们可以类似于宝塔部署多个项目那样，基于一个nginx服务，然后每个项目都有其独立的xxx.conf配置文件
- 原因解释：在nginx的配置文件 `/etc/nginx/nginx.conf` 中有以下字段，可以读取 `/etc/nginx/conf.d` 目录下任何以 `.conf` 为后缀的配置文件
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/bfa15e479afc8e943a417a402741e65f.png)
第二步：分析nginx需要挂载的数据卷


| 数据卷在服务器的位置                    | 映射nginx容器的位置          |
| ----------------------------- | --------------------- |
| /docker-data/nginx/cert       | /etc/nginx/cert       |
| /docker-data/nginx/conf.d     | /etc/nginx/conf.d     |
| /docker-data/nginx/nginx.conf | /etc/nginx/nginx.conf |
| /docker-data/nginx/html       | /usr/share/nginx/html |
| /docker-data/nginx/logs       | /var/log/nginx        |

- cert：存放各个项目的SSL证书
- conf.d：存放各个项目的conf配置文件
- nginx.conf：默认的配置文件
- html：存放前端项目打包好的内容（即dist目录下的内容）
- logs：nginx容器运行的日志

第三步：配置我们的nginx容器，见下面讲解
## 2 Docker安装Nginx容器

**下载 Nginx 镜像**

```shell
docker pull nginx:latest
```

> PS：这里下载的是最新 版本 Nginx 镜像，你可以在 [Docker Hub](https://hub.docker.com/ "Docker Hub") 上查找你想要的可用版本镜像。

命令执行成功后，通过 `docker images` 命令检查一下镜像是否下载成功：

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/3c0ba14efd643dd071c081a9abf19d9f.png)

---

**运行 Nginx 容器**

```bash
docker run -d -p 80:80 --name nginx nginx:latest
```

参数说明：
- `-p 80:80`: 将容器的 80 端口映射到宿主机的 80 端口上；
- `-d`: 以后台方式运行镜像；
- `--name`: 指定容器的名称为 nginx;

命令执行完成后，通过 `docker ps`命令确认一下容器是否启动成功

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/6b5a62276e3f2e8fed43356e6071e207.png)

Nginx 容器运行成功后，访问 `http://服务器ip地址`，即可看到 Nginx 欢迎页面：

> 提示：访问 `http://服务器ip地址` 在不指定端口的情况下，默认端口号为 80

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a65a5bf1270f85b5c53224059eced27e.png)

---

**复制 Nginx 配置文件至宿主机**

因为容器重启会丢失内部数据，因此要将需要持久化的文件挂载到宿主机中，以防数据丢失。

挂载之前，复制容器中需要持久化的相关配置文件到宿主机的指定路径下：

```bash
# 复制名称为 nginx 容器中 /etc/nginx/nginx.conf 文件夹到宿主机的 /docker-data/nginx 路径下，宿主机的持久化目录根据你的需要自定义路径
docker cp nginx:/etc/nginx/nginx.conf /docker-data/nginx
# 复制名称为 nginx 容器中 /etc/nginx/conf.d 文件到宿主机的 /docker-data/nginx 路径下
docker cp nginx:/etc/nginx/conf.d /docker-data/nginx
```

复制完成后，查看指定路径的配置文件，如下：

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/199edd1c91d1977df9d458fba13a5881.png)

其他文件夹和文件的配置，往下看！

---

**配置 `/docker-data/nginx` 目录，创建其他文件和文件夹**

```shell
# 创建cert文件夹
mkdir -p /docker-data/nginx/cert

# 创建html文件夹
mkdir -p /docker-data/nginx/html

# 创建logs文件夹
mkdir -p /docker-data/nginx/logs
```

---

**删除刚刚启动的 Nginx 容器，新启动一个**

先删除之前启动的 nginx 容器：

```bash
docker rm -f nginx
```

完成后，执行启动 Nginx 容器命令：

```bash
docker run -d \
  --name nginx \
  -p 80:80 \
  -p 443:443 \
  -e TZ=Asia/Shanghai \
  -v /docker-data/nginx/html:/usr/share/nginx/html \
  -v /docker-data/nginx/logs:/var/log/nginx \
  -v /docker-data/nginx/cert:/etc/nginx/cert \
  -v /docker-data/nginx/conf.d:/etc/nginx/conf.d \
  -v /docker-data/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v /etc/localtime:/etc/localtime:ro \
  --restart no \
  nginx:latest nginx -g "daemon off;"


```

参数说明：
- `-d: `容器将在后台运行。
- `--name nginx: `容器将被命名为 `nginx`。
- `-p 80:80 和 -p 443:443: `映射端口 80 和 443，分别用于 HTTP 和 HTTPS 通信。
- `-v: `将主机上的目录挂载到容器内的指定路径。这包括证书、配置文件、日志文件和 HTML 文件。
- `--restart no: `容器退出时不会自动重启。
- `nginx:latest: `使用最新的官方 Nginx 镜像。
- `nginx -g "daemon off;": `容器启动后执行的命令，`nginx -g "daemon off;"` 是让 Nginx 前台运行，防止容器启动后立即退出。
- `-e TZ=Asia/Shanghai`：设置环境变量 `TZ`，指定时区为 `Asia/Shanghai`。
- `-v /etc/localtime:/etc/localtime:ro`：将宿主机的 `/etc/localtime` 文件挂载到容器中，设置为只读模式 (`ro`)，以确保容器使用正确的本地时区。