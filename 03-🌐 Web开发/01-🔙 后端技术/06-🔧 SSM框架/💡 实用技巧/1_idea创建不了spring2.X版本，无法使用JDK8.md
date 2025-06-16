---
文章标题: "[[1_idea创建不了spring2.X版本，无法使用JDK8]]"
文章作者: Dakkk
文章概要: |
  介绍Spring 2.X版本停止维护导致IDEA无法创建该版本项目的问题，提供三种解决方案：阿里云创建、官网下载、降级Spring 3.X版本。
tags:
  - Spring Boot
  - IDEA
  - JDK8
  - Spring版本
  - 阿里云
  - 项目创建
  - 版本兼容
相关文章:
  - "[[../../../../08-🛠️ 开发工具/03-🐋 容器化/02-🚀 Docker进阶/1_SpringBoot项目部署（Docker）]]"
  - "[[2_搭建多模块工程（通过 Maven Archetype）]]"
  - "[[0_项目介绍]]"
  - "[[../../../../08-🛠️ 开发工具/02-🔧 版本控制/04-💡 实用技巧/0_Git问题解决]]"
  - "[[1_搭建多模块工程（Spring Initializr）]]"
文章分类: 🌐 Web开发
文章路径: 03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/1_idea创建不了spring2.X版本，无法使用JDK8.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 解释原因
spring2.X版本在2023年11月24日停止维护了，因此创建spring项目时不再有2.X版本的选项，只能从3.1.X版本开始选择

而Spring3.X版本不支持JDK8，最低支持JDK17，因此JDK8也无法主动选择了

当然，停止维护只代表我们无法用idea主动创建spring2.X版本的项目了，不代表我们无法使用，该使用依然能使用，丝毫不受影响

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bdd0e8d50d2d2d0384735227695aa873.png)

## 2 解决方案

### 2.1 阿里云创建Spring2.X版本的项目

修改Server URL为https://start.aliyun.com  

- 原来的URL为 start.spring.io

目前阿里云还是支持创建Spring2.X版本的项目的

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8a4cc013ea1c83aba97a8fb2d495a529.png)

然后就可以愉快的创建项目了

需要注意的是，通过阿里云创建的项目，初始结构与通过Spring官方创建的项目有所不同，但完全不影响使用，放心

### 2.2 阿里云官网创建Spring2.X版本的项目

打开阿里云官网 [https://start.aliyun.com](https://start.aliyun.com/ "https://start.aliyun.com")

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/98d72c4fb4ac0ee0cf8c735c0074be18.png)

创建过程很简单，此处不再展示，记得选择依赖，创建完毕后保存本地：

- 先点击获取代码，后点击下载代码包，下载代码包即下载该项目的压缩包
- 会git操作的也可以用git命令下载该项目文件，只是操作不同罢了，结果都是得到一个Spring2.X版本的初始项目文件

后续解压缩后直接用idea打开此项目即可

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5d61eea74e2cbc2bc53ecfdeb49530e5.png)

可以将此压缩包**保存**，每次新建项目时复制出一个新的项目文件，idea直接打开即可，压缩包可以当一个**永久的备份**，毕竟说不定哪天阿里云也创建不了spring2.X版本的项目了呢

### 2.3 下载JDK17，创建Spring3.X版本

#### 2.3.1 修改pom.xml

![||380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e82f4264e24ce1d673acaee503e23737.png)

修改完毕后启动一下项目看能否启动成功，若启动成功说明该修改的都修改好了，若报错，报错内容为JDK17/8不是国内源之类的问题，则继续修改，总共需要修改5个地方

#### 2.3.2 修改IDEA设置

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8212dd953da7c086f1cfd379e708769b.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7ff09bb0a6b4a1261f026b0b67c6d0e.png)

Modules这里也修改成JDK8![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/791018a53cacbefa8da8317d66f514a3.png)
这里也要修改![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/750d790ff5d975f417c93a62716f0b11.png)

设置这里也要进行修改![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23c76bc984e6566c1bb747f003d9e69b.png)