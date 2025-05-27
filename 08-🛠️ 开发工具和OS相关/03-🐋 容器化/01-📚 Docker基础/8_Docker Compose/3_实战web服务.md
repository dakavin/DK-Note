---
文章标题: "[[3_实战web服务]]" 
文章作者: Dakkk
文章概要: |
  文章通过Docker Compose搭建了一个简单的Web服务，使用Spring Boot和Redis实现访问计数功能。详细讲解了Spring Boot应用、Dockerfile和docker-compose.yml的配置与构建，展示了如何将多服务应用容器化并部署运行。
tags:
- "Docker Compose"
- "Spring Boot"
- "Redis"
- "Dockerfile"
- "Web服务"
- "容器化"
- "多服务部署"
相关文章:
- "[[5_技术二面（华为Od）]]"
- "[[秒杀+分布式锁项目]]"
- "[[1_SpringBoot项目部署（Docker）]]"
- "[[0_导读]]"
- "[[0_课程介绍]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/8_Docker Compose/3_实战web服务.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

本小节中，我们将通过 Docker Compose 搭建一个简单的网站。

## 1 需求

这个简单的网站需实现功能：**每当用户访问时，后台进行计数，并在页面中显示总访问次数**。效果如下：

![web 服务被访问次数+1](https://img.quanxiaoha.com/quanxiaoha/166467848358134 "web 服务被访问次数+1")

## 2 技术选型

- Web 应用：Spring Boot
- Redis 缓存（用于计数用户访问次数）

## 3 开始

### 3.1 web 应用

新建一个 Spring Boot 项目， 主代码如下：

```java
package com.example.dockerdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@SpringBootApplication
@Controller
public class DockerDemoApplication {

    private final RedisTemplate redisTemplate;

    public static void main(String[] args) {
        SpringApplication.run(DockerDemoApplication.class, args);
    }

    public DockerDemoApplication(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    @GetMapping("/")
    @ResponseBody
    public String index() {
    	// 对 redis 里面的被访问数执行 +1 操作
        Long viewNum = redisTemplate.opsForValue().increment("view_num");
        return String.format("<h1>页面总访问次数: %s</h1>", viewNum);
    }

}

```

Spring Boot 的 `application.yml` 配置文件如下：

```java
spring:
  redis:
    host: redis
    port: 6379
```

### 3.2 Dockerfile

为 Spring Boot 编写 Dockerfile :

```Dockerfile
# FROM 指定使用哪个镜像作为基准
FROM openjdk:8-jdk-alpine
# 数据卷
VOLUME /tmp
# 复制文件到镜像中
COPY docker-demo-0.0.1-SNAPSHOT.jar app.jar
# 指定容器内的时区
RUN echo "Asia/Shanghai" > /etc/timezone;
# ENV为设置环境变量
ENV JAVA_OPTS=""
# ENTRYPOINT 为启动时运行的命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar"]
```

编译打包 Spring Boot, 然后将 Dockerfile 文件放置在 `jar` 包的同级目录下，执行如下命令开始构建 `web` 镜像：

```
docker build -t web:test .
```

执行成功后，通过 `docker images` 可以看到构建成功的本地镜像:

![构建 Spring Boot Docker 镜像](https://img.quanxiaoha.com/quanxiaoha/166467690411164 "构建 Spring Boot Docker 镜像")

### 3.3 docker-compose.yml

编写 `docker-compose.yml` 文件：

```
version: '3'
services:
  web:
    image: "web:test"
    ports:
      - "8080:8080"

  redis:
    image: "redis:latest"
```

### 3.4 运行 compose 项目

在 `docker-compose.yml` 文件的同级目录下，执行运行命令：

```
docker-compose up
```

![运行 compose 项目](https://img.quanxiaoha.com/quanxiaoha/166467800825395 "运行 compose 项目")

`compose` 项目运行成功后，访问 web 服务，可以看到访问次数 `+1` 功能成功实现：

![web 服务被访问次数+1](https://img.quanxiaoha.com/quanxiaoha/166467848358134 "web 服务被访问次数+1")