---
文章标题: "[[2_scp命令，拷贝文件]]"
文章作者: Dakkk
文章概要: |
  介绍Linux系统中scp命令的基本用法，通过SSH协议在不同主机间复制文件，以传输内核模块文件到开发板为例演示具体操作步骤。
tags:
  - scp命令
  - Linux
  - SSH
  - 文件传输
  - 远程复制
  - 开发板
  - 内核模块
  - 网络传输
相关文章:
  - "[[0_课程介绍]]"
  - "[[../../../08-🛠️ 开发工具/02-🔧 版本控制/04-💡 实用技巧/0_Git问题解决]]"
  - "[[1_Linux下的MySQL]]"
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/02-🚀 Docker进阶/1_SpringBoot项目部署（Docker）]]"
  - "[[../../../08-🛠️ 开发工具/04-☁️ 运维部署/02-🤖 自动化部署/10_补充：JetBrains全家桶集成服务器上的Docker服务]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/00-❇️实用技巧/2_scp命令，拷贝文件.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-05-05 21:57:57
修改时间: 2025-05-28 00:19:51
---

scp 命令用于 Linux 之间复制文件和目录，该命令基于ssh，需要搭建好ssh环境，scp命令格式如下：
```shell
scp local_file remote_username@remote_ip:remote_folder
```

**转移文件如下**
```shell
scp hellomodule.ko cat@192.168.31.69:/home/cat/
```

将hellomodule.ko发送到192.168.31.69的/home/cat/目录下，192.168.31.69是开发板ip，需根据实际而定，开发板用户名为cat， 输入yes，然后输入密码进行验证，等待传输完成，这个时候我们开发板就有了hellomodule.ko 这个文件。如果是在开发板进行本地编译则不需要再进行传输

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d057f8c9aa9fd6148a847b7335de0450.png)
