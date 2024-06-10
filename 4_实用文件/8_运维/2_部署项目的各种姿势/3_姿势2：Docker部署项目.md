## 1 前置操作

**安装Docker，请参考《9_补充：安装Docker（本地和Linux）》**

因为使用Docker的nginx镜像，所以我们需要关闭云服务器的nginx服务

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ba1dd41c576fe1bbd9d6245e3dd1bc78.png)

```shell
# 关闭nginx服务
sudo systemctl stop nginx
# 检查nginx服务状态
sudo systemctl status nginx
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/b50e85d9ce44d14c382bd9e3977209d8.png)


同时需要关闭之前在宝塔界面部署的几个项目

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/adf0dbea288b09c0b15afd14ca67bfd3.png)

## 2 单个项目部署

### 2.1 前端（带域名和SSL证书）

#### 2.1.1 创建相关文件

在home项目目录创建docker目录，目录下的内容如下：
```
--home 
----nginx
------confid.d 
--------home-cert 
----------dakkk.top.key 
----------dakkk.top_bundle.pem 
--------home.conf 
----Dockerfile 
----dist
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/10d4c1f1e6d5eef5eaa737cfe2d0d1d5.png)

其中key和pem文件，直接复制即可

**home.conf配置文件如下**
```conf
server {  
    # 监听 80 端口，支持 IPv4 和 IPv6    listen 80;  
    listen  [::]:80;  
    # 设置服务的域名  
    server_name dakkk.top www.dakkk.top;  
  
    # 对所有 HTTP 请求进行 301 永久重定向到 HTTPS    return 301 https://www.dakkk.top$request_uri;  
}  
  
server {  
    # 监听 443 端口，启用 SSL/TLS 和 HTTP/2    listen 443 ssl http2;  
    server_name  dakkk.top www.dakkk.top;  
  
    # 客户端请求体的最大尺寸  
    client_max_body_size 100M;  
  
    # SSL 证书和私钥的路径  
    ssl_certificate /etc/nginx/cert/home-cert/dakkk.top_bundle.pem;  
    ssl_certificate_key /etc/nginx/cert/home-cert/dakkk.top.key;  
  
    # SSL 会话超时设置  
    ssl_session_timeout 5m;  
  
    # 设置安全的 SSL 加密套件  
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
    # 启用的 TLS 版本  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
    # 优先使用服务器端的加密套件  
    ssl_prefer_server_ciphers on;  
  
    # 配置Home项目的根目录和默认文档  
    root /usr/share/nginx/html/home;  
    index  index.html index.htm;  
  
    # 处理前端路由，尝试直接访问文件，如果找不到则重写到 index.html    location / {  
        try_files $uri $uri/ @router;  
    }  
  
    # 路由回退处理  
    location @router {  
        rewrite ^.*$ /index.html last;  
    }  
  
    # 定义错误页面  
    error_page  500 502 503 504  /50x.html;  
    location = /50x.html {  
        root   /usr/share/nginx/html/home;  
    }  
  
    # 启用 Gzip 压缩，优化文件传输  
    gzip on;  
    gzip_min_length 1k;          # 最小压缩文件大小  
    gzip_comp_level 9;          # 压缩级别  
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml; # 压缩类型  
    gzip_vary on;               # 根据客户端的请求头决定是否启用压缩  
    gzip_disable "MSIE [1-6]\.";  # 禁用旧版 IE 浏览器的 Gzip 压缩  
  
    # 包含 MIME 类型定义  
    include /etc/nginx/mime.types;  
}
```

**Dockerfile文件如下：**

```dockerfile
# 使用 Nginx 的 latest 版本作为基础镜像  
FROM nginx:latest 
  
# 维护者信息（可以填写自己的emial地址）  
LABEL maintainer="xxxx@xxx.xxx"  
  
# 创建存放 SSL 证书的目录结构  
RUN mkdir -p /etc/nginx/cert/home-cert  
  
# 将 SSL 证书复制到容器中  
COPY nginx/config.d/home-cert/dakkk.top.key /etc/nginx/cert/home-cert/dakkk.top.key  
COPY nginx/config.d/home-cert/dakkk.top_bundle.pem /etc/nginx/cert/home-cert/dakkk.top_bundle.pem  
  
# 将 Nginx 配置文件复制到容器中  
COPY nginx/config.d/home.conf /etc/nginx/conf.d/home.conf  
  
# 将 dist 目录复制到容器的网页根目录下  
COPY dist /usr/share/nginx/html/home  
  
# 对外暴露 80 和 443 端口  
EXPOSE 80 443  
  
# 使用 exec 格式启动 Nginx 以优化接收 UNIX 信号  
CMD ["nginx", "-g", "daemon off;"]
```

#### 2.1.2 使用docker命令部署

创建app目录
```shell
mkdir -p /app/home
```

将nginx、dist、Dockefile上传到该目录下（使用宝塔/xftp均可）
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/cf6a0c6594d554ab62857f7b13498d83.png)

进入对应的目录，使用docker命令开始部署
```shell
cd /app/home
```

构建Docker镜像文件
```shell
# 构建Docker镜像文件
docker build -t home:v0.0.1 .
# 查看Docker镜像文件
docker images
```

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/417dd8315390f6634b1806d24b79d7ae.png)

运行Docker镜像文件
```shell
docker run -d -p 80:80 -p 443:443 --name home -e TZ=Asia/Shanghai home:v0.0.1

# 检查docker容器
docker ps
```

补充：TZ=Asia/Shanghai，可以将容器内的时区设置为中国时区


![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/dc4f3252c34a0790dcc1054ed1a7b83b.png)

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/99ed230e7ff4c2cb0050a9ac2144e291.png)

#### 2.1.3 集成在IDEA中部署

**注意：不建议使用这种方式，虽然部署比较简单，但是会产生一堆无用镜像，且无法删除**

先参考文章 [9_补充：JetBrains全家桶集成服务器上的Docker服务](9_补充：JetBrains全家桶集成服务器上的Docker服务.md) 完成IDE可以连接远程服务器的docker服务

连接后点击，下面的小齿轮按钮，可以发现Docker容器的情况

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ba8d4bf0153b02a73b303f5146cd7209.png)

运行Dockfile，直接打开Dockerfile，可以看到run和edit的指令

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/1141e8fba93062e52e803ee25708e4ad.png)

edit的相关措施
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ef459213641f71eae57317c12380b7b8.png)

### 2.2 后端

首先，我们要知道对于需要运行后端逻辑和处理动态数据的应用程序来说，通常不会使用nginx的作为后端服务器代理的

这里我们直接使用jdk来完成后端项目的部署，包含maven的构建过程我们在后续CI/CD中讲解
#### 2.2.1 创建相关文件

在usercenter-backend项目目录创建docker目录，目录下的内容如下：
```
--usercenter-backend
----nginx
------confid.d 
--------usercenter-backend-cert
----------usercenter-backend.dakkk.top.key 
----------usercenter-backend.dakkk.top_bundle.pem 
--------usercenter-backend.conf 
----Dockerfile 
----target
------usercenter-backend-0.0.1-SNAPSHOT.jar
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/4a7068dd003b969d4cc9a8cfb8b14c72.png)

**Dockerfile文件如下：**

```dockerfile
# 使用基于 Alpine 的 Java 8 JDK 镜像  
FROM openjdk:8-jdk-alpine  
  
# 设置容器内的时区为上海  
ENV TZ=Asia/Shanghai  
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone  
  
# 暴露 8080 端口，这是应用预设的运行端口  
EXPOSE 8080  
  
# 设置工作目录为 /app，之后的指令都将在这个目录下执行  
WORKDIR /app  
  
# 从构建上下文的 target 目录中复制 jar 文件到容器的工作目录  
COPY target/usercenter-backend-0.0.1-SNAPSHOT.jar /app/App.jar  
  
# 容器启动时运行 Java 应用  
CMD ["java", "-jar", "/app/App.jar", "--spring.profiles.active=prod"]
```

**nginx文件夹只有在给后端设置域名和SSL证书的时候才会使用**
#### 2.2.2 使用docker命令部署

创建app目录
```shell
mkdir -p /app/usercenter-backend
```

将target、Dockerfile上传到该目录下（使用宝塔/xftp均可）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/953e1cca5220f3b7018fbfa53007cd07.png)

进入对应的目录，使用docker命令开始部署
```shell
cd /app/home
```

构建Docker镜像文件
```shell
# 构建Docker镜像文件
docker build -t usercenter-backend:v0.0.1 .
# 查看Docker镜像文件
docker images
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/912547443e4783e5ee2992dffa5c63c8.png)

运行Docker镜像文件
```shell
docker run -d -p 8080:8080  --name usercenter-backend  usercenter-backend:v0.0.1

# 检查docker容器
docker ps
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/f958677bf378cffba05524be8584072a.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/bfbe1b67d546e744e26a4dd9c823613e.png)

尝试ip访问，可以看到访问成功
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c858e33a133ab492a7dab2c0f78379b5.png)

#### 2.2.3 集成在IDEA中部署

**注意：不建议使用这种方式，虽然部署比较简单，但是会产生一堆无用镜像，且无法删除**

先参考文章 [9_补充：JetBrains全家桶集成服务器上的Docker服务](9_补充：JetBrains全家桶集成服务器上的Docker服务.md) 完成IDE可以连接远程服务器的docker服务

连接后点击，下面的小齿轮按钮，可以发现Docker容器的情况

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/ba8d4bf0153b02a73b303f5146cd7209.png)

运行Dockfile，直接打开Dockerfile，可以看到run和edit的指令

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/97fedafc35ab9009848eb5fadf6075aa.png)


edit的相关措施
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/84a654be4aec80ea11783396bbd2383c.png)

#### 2.2.4 设置域名和SSL证书

在部署多个前端项目的时候讲解，请往后看，我们先把相关文件准备好！

**其中key和pem文件，直接复制即可**

**home.conf配置文件如下**
```conf
server {  
    # 监听 443 https 端口 , 启用 HTTP/2 协议。HTTP/2 是 HTTP 协议的下一版本，它引入了一些性能优化  
    # 例如多路复用（Multiplexing）和头部压缩，以提高页面加载速度。  
    listen 443 ssl http2;  
    # 域名，修改成你自己的  
    server_name usercenter-backend.dakkk.top;  
    # 设置客户端最大可上传的体积  
    client_max_body_size 100M;  
  
    # Nginx 容器中的 SSL 证书和密钥路径（后续会挂载到宿主机的 /docker/nginx/cert/ 目录下）  
    ssl_certificate /etc/nginx/cert/usercenter-backend-cert/usercenter-backend.dakkk.top_bundle.pem;  
    ssl_certificate_key  /etc/nginx/cert/usercenter-backend-cert/usercenter-backend.dakkk.top.key;  
    # SSL 会话设置  
    ssl_session_timeout 5m;  
    # 设置安全的 SSL 加密套件  
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
    # 启用的 TLS 版本  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
    # 优先使用服务器的密码套件  
    ssl_prefer_server_ciphers on;  
    # 启用 HSTS（HTTP Strict Transport Security）  
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;  
  
    location / {  
       # 将所有流量代理到内网 IP 的 8080 端口  
       proxy_pass http://172.17.0.1:8080;  
       # 设置请求头部，将客户端的主机信息传递给后端服务  
       proxy_set_header Host $host;  
       # 设置请求头部，传递客户端的真实 IP 地址给后端服务  
       proxy_set_header X-Real-IP $remote_addr;  
       # 设置请求头部，添加客户端的 IP 地址到 X-Forwarded-For，用于追踪原始请求者  
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
       # 设置请求头部，传递原始请求的 host 和端口，有助于后端服务识别请求来源  
       proxy_set_header X-Host $host:$server_port;  
       # 设置请求头部，传递客户端请求使用的协议 (如 http 或 https)       proxy_set_header X-Scheme $scheme;  
       # 设置请求头部，用于 WebSocket 支持，允许客户端与服务器进行 WebSocket 连接  
       proxy_set_header Upgrade $http_upgrade;  
       # 设置请求头部，用于 WebSocket 支持，保持连接状态为 "upgrade"       proxy_set_header Connection "upgrade";  
       # 设置代理连接的超时时间为 30 秒，如果在此时间内无法连接到后端，连接将被关闭  
       proxy_connect_timeout 30s;  
       # 设置代理读取的超时时间为 86400 秒（一天），如果在此时间内后端没有响应，请求将被关闭  
       proxy_read_timeout 86400s;  
       # 设置代理发送的超时时间为 30 秒，如果在此时间内无法向后端发送数据，连接将被关闭  
       proxy_send_timeout 30s;  
       # 设置代理使用的 HTTP 版本，这里指定使用 HTTP/1.1       proxy_http_version 1.1;  
    }  
  
    # 错误页面处理  
    error_page   500 502 503 504  /50x.html;  
    location = /50x.html {  
        root   /usr/share/nginx/html;  
    }  
  
    # 后端项目没必要开启gzip  
}
```

## 3 多个项目部署

因为个人对nginx理解有限，这里直接给出方案

使用这个方案之前，需要停止之前的home项目，因为这个home项目挂载了nginx镜像，端口为80，占用了我们后续运行的nginx容器
```shell
# 暂停home容器（二选一）
docker stop home
# 删除home容器（二选一，建议直接删除）
docker rm -f home
```

后端项目的容器不需要停止/删除，放着就好了
### 3.1 思路

第一步：我们可以类似于宝塔部署多个项目那样，基于一个nginx服务，然后每个项目都有其独立的xxx.conf配置文件
- 原因解释：在nginx的配置文件 `/etc/nginx/nginx.conf` 中有以下字段，可以读取 `/etc/nginx/conf.d` 目录下任何以 `.conf` 为后缀的配置文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/bfa15e479afc8e943a417a402741e65f.png)
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
### 3.2 Docker安装Nginx容器

**下载 Nginx 镜像**

```shell
docker pull nginx:latest
```

> PS：这里下载的是最新 版本 Nginx 镜像，你可以在 [Docker Hub](https://hub.docker.com/ "Docker Hub") 上查找你想要的可用版本镜像。

命令执行成功后，通过 `docker images` 命令检查一下镜像是否下载成功：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/3c0ba14efd643dd071c081a9abf19d9f.png)

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

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/6b5a62276e3f2e8fed43356e6071e207.png)

Nginx 容器运行成功后，访问 `http://服务器ip地址`，即可看到 Nginx 欢迎页面：

> 提示：访问 `http://服务器ip地址` 在不指定端口的情况下，默认端口号为 80

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a65a5bf1270f85b5c53224059eced27e.png)

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

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/199edd1c91d1977df9d458fba13a5881.png)

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

### 3.3 多项目部署

使用宝塔或xftp打开 `/docker-data/nginx` 目录

**先配置cert目录**

将之前的各个项目的cert文件夹直接拷贝到服务器的cert目录下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/4816ce79d282f7fb0f5ada2acc98e58f.png)

---

**配置conf.d目录**

先创建`default.conf`，具体内容如下
```conf
server {
    listen       80;
    listen  [::]:80;

    # 启用访问日志
    access_log  /var/log/nginx/access.log  main;

    # 设置字符集
    charset utf-8;

    # 根目录和默认文件设置
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ =404;
    }

    # 统一管理错误页面
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
        internal;
    }

    # Gzip 压缩配置
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 2;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # 禁止直接访问 .htaccess 文件
    location ~ /\.ht {
        deny all;
    }
}
```

导入我们之前编写的各个项目的nginx.conf配置文件

最后情况如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/12fcbb8464b9b47d3488a244c71ce8af.png)

---

**配置html目录**

主要是将前端项目的dist文件，放到对应的目录下

在`/docker-data/nginx/html`目录下，分别创建home目录和usercenter-frontend目录

然后将dist目录下的内容分别拷贝到对应的项目目录中

如图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/634904d40d54bd9de36bf25bf3a76d11.png)

---

**测试**

尝试访问`home`项目，可以发现安全的访问到了

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/885b6ccf52bbe385be419f987fd164e3.png)


尝试访问`usercenter-frontend`项目，可以发现安全的访问到了

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/1ce40bcba037d50d77d337f194a9af90.png)

尝试访问`usercenter-backend`项目，可以发现安全的访问到了

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/3e0d591aed22f87fe17858d9bee8b945.png)

