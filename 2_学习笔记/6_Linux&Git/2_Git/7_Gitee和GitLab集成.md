# 一、Gitee集成

## 1、注册网站会员

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214307.png)
## 2、用户中心

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214312.png)
## 3、创建远程仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214320.png)

## 4、远程仓库简易操作指令

```shell
# Git 全局设置，修改成自己的信息

git config --global user.name "Aitiger"

git config --global user.email ["12252591+aitiger@user.noreply.gitee.com"](mailto:%2212252591+aitiger@user.noreply.gitee.com%22)

# 创建 git 仓库，基本操作指令和其他远程仓库一致。

mkdir git-study

cd git-study

git init

touch README.md

git add README.md

git commit -m "first commit"

git remote add origin git@gitee.com:aitiger/git-study.git

git push -u origin "master"

# 已有仓库

cd existing_git_repo

git remote add origin git@gitee.com:aitiger/git-study.git

git push -u origin "master"
```
## 5、配置SSH免密登录

### 5.1 本地生成SSH秘钥

```shell
# ssh-keygen -t rsa -C Gitee账号

ssh-keygen -t rsa -C Gitee账号
```
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214521.png)
### 5.2 集成用户公钥

执行命令完成后,在`window本地用户.ssh目录C:\Users\用户名\.ssh`下面生成如下名称的公钥和私钥:

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214538.png)

按照操作步骤，将id_rsa.pub文件内容复制到Gitee仓库中![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214552.png)

## 6、集成IDEA

### 6.1 创建新项目

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215016.png)

### 6.2 安装Gitee插件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215020.png)

### 6.3 配置Gitee账户授权

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215025.png)
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215030.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215038.png)

### 6.4 增加远程地址

```shell
# 增加远程地址
git remote add gitee-study 
git @gitee.com:aitiger/git-study.git
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215052.png)

### 6.5 提交本地代码

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215100.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215107.png)

## 7、多用户协作

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215113.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215121.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215127.png)

# 二、GitLab集成

前面给大家讲解的都是如何使用第三方代码托管平台来管理咱们的代码库。那么我们自己搭建一个这样的平台行不行呢？其实咱们之前已经用Git软件搭建了一个远程版本库，但是功能相对来讲，比较单一，而且操作起来也不像GitHub, Gitee平台那样更加人性化，所以我们这里介绍一个GitLab软件，用于搭建自己的代码托管平台。

## 1、GitLab介绍

GitLab是由GitLabInc开发，使用MIT许可证的基于网络的Git仓库管理工具，且具有wiki和issue跟踪功能。使用Git作为代码管理工具，并在此基础上搭建起来的Web服务。

GitLib由乌克兰程序员DmitriyZaporozhets和ValerySizov开发，它使用Ruby语言写成。后来，一些部分用Go语言重写。GitLab被IBM，Sony，JulichResearchCenter，NASA，Alibab，Invincea，O'ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX等组织使用。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215155.png)
## 2、GitLab软件下载

官网地址：[https://about.gitlab.com/](https://about.gitlab.com/)

这里我们可以根据个人情况，选择下载不同版本的软件：

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215205.png)

我们这里主要是教学，所以下载使用社区版(CE)即可
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215215.png)

这里我们选择下载适用CentOS 7系统的版本
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215224.png)

下载地址：https://packages.gitlab.com/gitlab/gitlab-ce
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215234.png)

如果下载不了，或下载比较慢，可以根据提示在在linux系统中直接采用wget指令下载
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215244.png)

## 3、GitLab安装

### 3.1 安装Linux系统

Linux系统的安装不在本课程学习范畴中，请同学们自行安装CentOS 7即可。
### 3.2 安装GitLab

直接采用下载的RPM软件包安装即可
```shell
sudo rpm -ivh /opt/module/software/gitlab-ce-15.7.0-ce.0.el7.x86_64.rpm
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215314.png)

### 3.3 安装配置依赖项

在CentOS 7上，下面的命令也会在系统防火墙中打开HTTP、HTTPS和SSH访问。这是一个可选步骤，如果您打算仅从本地网络访问极狐GitLab，则可以跳过它

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl

sudo systemctl enable sshd

sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo systemctl reload firewalld

# 为了演示方便，我们也可以直接关闭防火墙

sudo systemctl stop firewalld
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215331.png)

### 3.4 初始化GitLab

```shell
# 配置软件镜像

curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash

# 安装

sudo EXTERNAL_URL="https://linux1" yum install -y gitlab-ce

# 初始化

sudo gitlab-ctl reconfigure
```
### 3.5 启动GitLab

```shell
# 启动

gitlab-ctl start

# 停止

gitlab-ctl stop
```

### 3.6 访问GitLab

使用浏览器访问GitLab，输入网址：[http://linux1/users/sign_in](http://linux1/users/sign_in)
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215354.png)

初始化时，软件会提供默认管理员账户：root,但是密码是随机生成的。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215405.png)

根据提示，在/etc/gitlab/initial_root_password文件中查找密码
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215414.png)

输入账号，密码，进入系统
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215423.png)

### 3.7 修改密码

默认的密码是随机的，且不容易记忆，还会在系统初始化后24小时被删除，所以需要先修改一下密码
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215436.png)

### 3.8 创建项目

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215444.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215451.png)

## 4、集成IDEA

### 4.1 安装GitLab插件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215502.png)

### 4.2 配置GitLab

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215509.png)

### 4.3 创建新项目

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215522.png)
### 4.4 创建本地仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215527.png)
### 4.5 创建新代码

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215532.png)
### 4.6 提交文件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215539.png)
### 4.7 推送远程库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215546.png)

### 4.8 配置远程库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215553.png)

### 4.9 推送文件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215601.png)

### 4.10 合并提交请求

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215608.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215614.png)

合并
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215622.png)

确认文件提交
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923215630.png)