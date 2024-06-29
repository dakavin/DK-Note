## 1 Gitee集成

### 1.1 注册网站会员

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59acea7b0f2b951cbf5063c32fa2f6bd.png)
### 1.2 用户中心

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a663ec55f5315efe9aa87b17726a989.png)
### 1.3 创建远程仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fbd66bd7f0ef59333c42d7a1ddada446.png)

### 1.4 远程仓库简易操作指令

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
### 1.5 配置SSH免密登录

#### 1.5.1 本地生成SSH秘钥

```shell
# ssh-keygen -t rsa -C Gitee账号

ssh-keygen -t rsa -C Gitee账号
```
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/666253e8543d813e9b728a58c05c13e9.png)
#### 1.5.2 集成用户公钥

执行命令完成后,在`window本地用户.ssh目录C:\Users\用户名\.ssh`下面生成如下名称的公钥和私钥:

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0138a7917b1d1661aad6066230372129.png)

按照操作步骤，将id_rsa.pub文件内容复制到Gitee仓库中![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d7e864bbfc7ea17c77e364f0e1e7c714.png)

### 1.6 集成IDEA

#### 1.6.1 创建新项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/732de7ecf324332c52c8b652b075f5c3.png)

#### 1.6.2 安装Gitee插件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5dd99d7bd59ee76de9086f1b7cef08c7.png)

#### 1.6.3 配置Gitee账户授权

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4a2b3c608905ffaef3b9664ea98dbf63.png)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56c564a170dab8f11999c2df5328383c.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61aa0a14e659d7729d56a9bacd238577.png)

#### 1.6.4 增加远程地址

```shell
# 增加远程地址
git remote add gitee-study 
git @gitee.com:aitiger/git-study.git
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/957d05627dd37efaed3420dcbae81ab1.png)

#### 1.6.5 提交本地代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f5cb747ea1e5550f89cfa1e5b76f04e3.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/de25d376340b18a58dd2d344f7441dcd.png)

### 1.7 多用户协作

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/356773b1573af03ec7fe4395b533c028.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fa793300b4c2f793604dad524787ac9e.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e62ee5d12013715598229afe0a8c7253.png)

## 2 GitLab集成

前面给大家讲解的都是如何使用第三方代码托管平台来管理咱们的代码库。那么我们自己搭建一个这样的平台行不行呢？其实咱们之前已经用Git软件搭建了一个远程版本库，但是功能相对来讲，比较单一，而且操作起来也不像GitHub, Gitee平台那样更加人性化，所以我们这里介绍一个GitLab软件，用于搭建自己的代码托管平台。

### 2.1 GitLab介绍

GitLab是由GitLabInc开发，使用MIT许可证的基于网络的Git仓库管理工具，且具有wiki和issue跟踪功能。使用Git作为代码管理工具，并在此基础上搭建起来的Web服务。

GitLib由乌克兰程序员DmitriyZaporozhets和ValerySizov开发，它使用Ruby语言写成。后来，一些部分用Go语言重写。GitLab被IBM，Sony，JulichResearchCenter，NASA，Alibab，Invincea，O'ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX等组织使用。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/97dfba2999883e8e78cf78ba575ec08c.png)
### 2.2 GitLab软件下载

官网地址：[https://about.gitlab.com/](https://about.gitlab.com/)

这里我们可以根据个人情况，选择下载不同版本的软件：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2fdf745733526d6a213a0360e90854df.png)

我们这里主要是教学，所以下载使用社区版(CE)即可
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9648502ddd1d1cb0f5b01a259bbd3382.png)

这里我们选择下载适用CentOS 7系统的版本
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5d4df0dcfa5d55752e3bf3cf1f60dc7.png)

下载地址：https://packages.gitlab.com/gitlab/gitlab-ce
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/df56318527d55e53fd9efe1c3029b4f4.png)

如果下载不了，或下载比较慢，可以根据提示在在linux系统中直接采用wget指令下载
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b66ef2bbc5e2e56f1eb4b90b5266691.png)

### 2.3 GitLab安装

#### 2.3.1 安装Linux系统

Linux系统的安装不在本课程学习范畴中，请同学们自行安装CentOS 7即可。
#### 2.3.2 安装GitLab

直接采用下载的RPM软件包安装即可
```shell
sudo rpm -ivh /opt/module/software/gitlab-ce-15.7.0-ce.0.el7.x86_64.rpm
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9bd01ae9a9583ad6b87ff86b36dab190.png)

#### 2.3.3 安装配置依赖项

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

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c6be2c88734d8de19e24e4c4c2b5fac.png)

#### 2.3.4 初始化GitLab

```shell
# 配置软件镜像

curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash

# 安装

sudo EXTERNAL_URL="https://linux1" yum install -y gitlab-ce

# 初始化

sudo gitlab-ctl reconfigure
```
#### 2.3.5 启动GitLab

```shell
# 启动

gitlab-ctl start

# 停止

gitlab-ctl stop
```

#### 2.3.6 访问GitLab

使用浏览器访问GitLab，输入网址：[http://linux1/users/sign_in](http://linux1/users/sign_in)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c873744691f0ec2b48131d8a8f006cba.png)

初始化时，软件会提供默认管理员账户：root,但是密码是随机生成的。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b6fd418e0528a62561921ae90d2a3ed8.png)

根据提示，在/etc/gitlab/initial_root_password文件中查找密码
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6039372b89fe337a3486534bf80231bc.png)

输入账号，密码，进入系统
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7777a2e8a97e6f77d4f3a4906ca3c20e.png)

#### 2.3.7 修改密码

默认的密码是随机的，且不容易记忆，还会在系统初始化后24小时被删除，所以需要先修改一下密码
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/184ca2f6ac456cb269e313d3b0bfccd2.png)

#### 2.3.8 创建项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2822a418d95454324398406018b39f2.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e79d09ecd9c1220c83f2fd6cfec6ad3.png)

### 2.4 集成IDEA

#### 2.4.1 安装GitLab插件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c4206f9a75a69c21c2aaa96a7a186e66.png)

#### 2.4.2 配置GitLab

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9fd3dd9ef8afccb165ad8137ef8a62d3.png)

#### 2.4.3 创建新项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf1c462ef2bf4b8ee0c5646c8b32e628.png)
#### 2.4.4 创建本地仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aeecc2e45495aa8b6e7e0c9bff76c886.png)
#### 2.4.5 创建新代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a5bcf8fc08364e0b080bbc6d6e258330.png)
#### 2.4.6 提交文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3720a6074fec618e6b61d2367098c63e.png)
#### 2.4.7 推送远程库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7650ec2124e760c2033742ad86f7cca2.png)

#### 2.4.8 配置远程库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20e2f6784afb68ee267aa7582c38c166.png)

#### 2.4.9 推送文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e05ff897770086dd9ecddcee9cd5ecbb.png)

#### 2.4.10 合并提交请求

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56fa7612ffe85434f5ede745aa4628cd.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6dc46135af9271ffc77c3f0ad8083fd2.png)

合并
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a345842bd517964853cb6e388cdeef68.png)

确认文件提交
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cd6b52ae625d50318ec7e1ea707c983.png)