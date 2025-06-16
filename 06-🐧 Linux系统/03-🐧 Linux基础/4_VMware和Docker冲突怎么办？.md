---
文章标题: "[[4_VMware和Docker冲突怎么办？]]"
文章作者: Dakkk
文章概要: |
  介绍了Windows 10系统中VMware与Docker因Hyper-V虚拟化内核冲突的原因，提供了通过bcdedit命令切换hypervisorlaunchtype参数来解决冲突的简单方法。
tags:
  - VMware
  - Docker
  - Hyper-V
  - Windows 10
  - 虚拟化
  - bcdedit
  - 系统冲突
  - 容器技术
相关文章:
  - "[[../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/1_Window 安装 Docker]]"
  - "[[../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/1_介绍/2_ Docker VS 虚拟机]]"
  - "[[../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/4_镜像/3_查看镜像信息]]"
  - "[[../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/4_镜像/5_删除镜像]]"
  - "[[0_导论]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/02-⚙️ 系统基础/01-💻 命令行操作/4_VMware和Docker冲突怎么办？.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:19:51
---

参考文章：[VMware和Docker冲突怎么办？超级简单_docker vmware冲突-CSDN博客](https://blog.csdn.net/weixin_43038752/article/details/112062334?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-112062334-blog-116693203.235%5Ev43%5Epc_blog_bottom_relevance_base2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-112062334-blog-116693203.235%5Ev43%5Epc_blog_bottom_relevance_base2&utm_relevant_index=2)

## 为何冲突?

Vmware自带虚拟化内核，但是在win10中Docker的工作需要依赖Hyper-V，本质上是Hyper-v和Vmware内核之间的冲突**，毕竟二者提供了相同的功能。

PS: 安装Hyper-V服务其实也不是一个简单的事情，尤其是当你的系统是win10家庭版的时候，需要多走点流程，但是也很简单：
```cmd
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```

命名为XXX.cmd即可。点击运行，剩下的就是机器要求重启，按它的需求来就行了，在windows功能里看到下面这个图就行：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d7065b9d6df14b83503be9c9742cb667.png)


## 如何解决

直接贴解决方案：powershell里（管理员权限win+x）

1、当你想用VMware
```shell
bcdedit /set hypervisorlaunchtype off
```

2、当你想用Docker

```shell
bcdedit /set hypervisorlaunchtype auto
```

每次切换之后必须进行reboot（重启电脑）。
这个方法下系统的功能和服务都不要管，也不要尝试手动去切换

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/992f1f9c1931b6f1cbecd4c9f9ae494b.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/01cb1959534dee046c527b6109b26a7f.png)

启动docker时服务里其实有以上服务就行了。