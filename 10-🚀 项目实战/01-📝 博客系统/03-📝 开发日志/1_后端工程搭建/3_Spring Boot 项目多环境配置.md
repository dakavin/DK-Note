---
文章标题: "[[3_Spring Boot 项目多环境配置]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了Spring Boot多环境配置，利用Spring Profile实现不同环境的配置隔离。通过`application-{profile}.yml`文件管理差异化配置，讲解了`spring.profiles.active`属性、命令行等多种激活方式。提供了实战示例与验证，帮助开发者高效管理应用部署。
文章标签:
- "Spring Boot"
- "Spring Profile"
- "多环境配置"
- "配置文件"
- "环境管理"
- "YAML"
- "部署"
相关文章:
- "[[2_Maven和Springboot下多环境的区别]]"
- "[[1_properties 和 yml]]"
- "[[1_SpringBoot项目部署（Docker）]]"
- "[[7_Springboot]]"
- "[[0_参考文章索引]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/3_Spring Boot 项目多环境配置.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:34
修改时间: 2024-04-17 08:59:18
---

## 1 什么是多环境

在日常的开发工作中，我们的**应用程序往往需要在不同的环境中运行**，例如：开发环境、测试环境和生产环境。**每个环境中的配置参数可能都会有所不同**，例如数据库连接信息、文件服务器的 IP 地址等。Spring Boot 提供了非常方便的方式来管理这些不同环境的配置。

## 2 Spring Profile介绍

Spring Profile 是 Spring 框架用于处理不同环境配置的解决方案。Profile 可以帮助我们在不改变应用代码的情况下，根据当前环境动态地激活或者切换不同的配置。

Spring Boot 为每个 Profile 提供了一个独立的 `application.properties`（或 `application.yml`）配置文件。默认情况下，Spring Boot 使用的是 `application.properties` 文件。当你激活一个特定的 Profile 时，Spring Boot 会查找名为 `application-{profile}.properties` 的文件，并把其中的属性加载到 Spring Environment 中。
## 3 如何创建和使用Profile

假设我们有三个环境：开发（`dev`）、测试（`test`）、生产（`prod`）。那么我们可以创建以下几个配置文件：

- `application.properties`: 存放所有环境通用的配置。
- `application-dev.properties`: 存放开发环境的特殊配置。
- `application-test.properties`: 存放测试环境的特殊配置。
- `application-prod.properties`: 存放生产环境的特殊配置。
## 4 配置文件内容示例

在 `application.properties` 中，我们可能会**配置一些公共的属性**：
```properties
# 应用名称
app.name=blog
```

在 `application-dev.properties` 中，我们可以**配置一些一些开发环境特定的配置**，如本地调试要使用的数据库连接等：
```properties
# 数据库连接信息
spring.datasource.url=jdbc:mysql://localhost:3306/dev_db
```

在 `application-test.properties` 和 `application-prod.properties` 中也可以类似地配置测试和生产环境的特定属性。
## 5 如何激活Profile

在 Spring Boot 中，有多种方式可以激活 Profile：

1、 **在 Spring Boot 的 `application.properties` 或 `application.yml` 中设置 `spring.profiles.active` 属性**。例如，你可以添加以下配置来激活 "dev" 环境：
```properties
# 企业级项目开发中，一般项目默认会激活 dev 环境
spring.profiles.active=dev
```

2、 **在命令行中设置 `spring.profiles.active` 系统属性**。例如，你可以使用以下命令来启动你的应用，并激活 "prod" 环境：
```shell
# 企业级项目开发中，针对生产环境，一般通过启动命令再指定激活 prod 环境
java -jar $APP_NAME --spring.profiles.active=prod
```

 3、 **在操作系统的环境变量中设置 `SPRING_PROFILES_ACTIVE`**（这种方式比较少用到）。
## 6 实战

### 6.1 新建多环境配置文件

针对 `weblog` 项目来说，由于它的定位是个人博客，不涉及团队开发，我们只需要配置两个环境就 OK 了：

- `dev` : 本地开发环境，兼顾测试，测试完成后直接上线；
- `prod` : 生产环境；

打开 `weblog-web` 入口模块，将 `/resources` 目录下的 `/static` 和 `/templates` 都删除掉，它们用于服务端渲染的项目，因为我们是前后端分离，所以不需要：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/311652e68a0c70714bd94a9594808b5b.png)

然后，将 `application.properties` 的后缀改为 `.yml` 格式，另外，再新建 `application-dev.yml` 开发环境与 `application-prod.yml` 生产环境:
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a3b4aea8bfb760fff99e11061fea1167.png)
### 6.2 激活默认环境

编辑 `applicaiton.yml`, 添加通用配置：
```yml
spring:  
    profiles:  
        # 默认激活 dev 环境  
        active: dev  
server:  
    # 应用服务 WEB 访问端口  
    port: 8080
```

由于目前还是项目搭建阶段，暂时就配置以上内容，后面需要啥，如数据库连接信息、Elasticsearch 连接信息等，用到时再添加。

## 7 验证是否生效

配置完成后，再次启动项目，观察控制台日志，若看到 `The following profiles are active: dev` , 则表示激活 `dev` 环境成功啦
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1214acd9b5d3cb856980384fe2dc820a.png)
## 8 小结

通过使用 Spring Profile，我们可以轻松地管理在不同环境中运行的应用程序的配置，从而使得应用程序的部署和运维更加方便。最后，我们给 `weblog` 项目配置了 `dev` 开发环境、`prod` 生产环境，并将默认环境激活为 `dev`， 验证了环境配置已经生效。