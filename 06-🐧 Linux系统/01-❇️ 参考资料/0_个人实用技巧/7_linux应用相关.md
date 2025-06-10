---
文章标题: "[[7_linux应用相关]]"
文章作者: Dakkk
文章概要: |
  介绍Linux系统中查看文件系统使用情况和定位头文件的基本命令，包括df和locate工具的使用方法。
tags:
  - Linux
  - 系统管理
  - 文件系统
  - 头文件
  - df命令
  - locate命令
  - 系统监控
  - 开发工具
相关文章:
  - "[[../../03-🐧 Linux基础/6_Linux常用操作]]"
  - "[[../../05-💻 Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/1_Linux系统/9_Linux文件目录]]"
  - "[[3_CentOS 安装 Docker]]"
  - "[[4_Ubuntu 安装 Docker]]"
  - "[[../../02-💻 开发环境/3_Lubancat-RK3568/2_镜像构建与部署/07_Linux的介绍]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/00-❇️实用技巧/7_linux应用相关.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-05-13 20:42:10
修改时间: 2025-05-28 00:36:50
---
## 1 查看系统当前使用的文件使用

```shell
#在主机或板卡上执行如下命令
df -T
```

## 2 定位头文件

```shell
#我们需要安装该软件
sudo apt install locate
#然后需要更新一下数据
sudo updatedb
#然后就可以使用了

locate sys/stat.h
```
## 1 查看系统当前使用的文件使用

```shell
#在主机或板卡上执行如下命令
df -T
```

## 2 定位头文件

```shell
#我们需要安装该软件
sudo apt install locate
#然后需要更新一下数据
sudo updatedb
#然后就可以使用了

locate sys/stat.h
```