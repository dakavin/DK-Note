---
文章标题: "[[4_Git远程服务器]]" 
文章作者: Dakkk
文章概要: |
  文章详细指导如何在CentOS上从源码搭建Git远程服务器。内容涵盖Git安装、环境配置、创建Git用户、SSH免密登录以及初始化裸仓库。最后，演示了客户端如何克隆、提交并推送代码，实现团队共享版本库。
tags:
- "Git"
- "Git服务器"
- "CentOS"
- "Linux"
- "版本控制"
- "SSH"
- "源码编译"
- "远程仓库"
相关文章:
- "[[5_GitHub服务器]]"
- "[[7_Gitee和GitLab集成]]"
- "[[1_本章代码及文章获取]]"
- "[[1_初识Linux]]"
- "[[1_Linux下的MySQL]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/02-🔧 版本控制/02-🚀 Git进阶/4_Git远程服务器.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

在之前的操作中，所有的操作都是基于本地机器完成的。如果在公司中，一个项目是共用一个版本库的。那么所有的开发人员都应该对同一个版本库进行操作。因为Git软件本身就是用于Linux系统开发所设计的版本管理软件，所以项目中搭建的共享版本库也应该以linux系统为主。那么接下来，咱们就演示一下在CentsOS服务器中搭建Git服务器。
## 1 下载Git软件（Linux版本）

官网下载地址：[https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.1.tar.gz](https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.1.tar.gz)

将下载后的压缩文件上传到Linux系统中
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5630cd12da75973ae12405fe024838a5.png)
## 2 安装Git软件

### 2.1 解压Git

```shell
# 将压缩文件解压到自定义位置

tar -zxvf git-2.38.1.tar.gz -C **/opt/module/**

# 可以更改名字，变得简短一些，好操作

cd /opt/module

mv git-2.38.1/ git
```
### 2.2 安装依赖

解压后，我们需要编译源码，不过在此之前需要安装编译所需要的依赖，耐心等待安装完成，中途出现提示的时候输入y并按回车。

```shell
#

yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fd69cb328e5e21fffb223e901579b446.png)

### 2.3 删除旧版Git

安装编译源码所需依赖的时候，yum操作回自动安装旧版本的Git。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/605398aaabe1f9e44a9c108a13cd969f.png)

```shell
# 删除旧版本的Git

yum -y remove git
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dd6f2461a394e45c57c02d0feeb7057f.png)

### 2.4 编译、安装Git

```shell
# 进入到Git软件的解压目录

cd /opt/module/git

# 编译时，prefix设定为Git软件安装目录

make prefix=/usr/local/git all

# 安装Git

make prefix=/usr/local/git install
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/033d06e44f1443d4d8d0f55f02eb735a.png)

### 2.5 配置环境变量

修改linux系统中/etc/profile文件，配置环境变量
```shell
# 配置环境变量

export PATH=$PATH:/usr/local/git/bin

# 刷新环境，让环境变量立即生效

source /etc/profile
```
### 2.6 建立链接文件

```shell
# git安装路径是/usr/local/git，不是默认路径

ln -s /usr/local/git/bin/git-upload-pack /usr/bin/git-upload-pack

ln -s /usr/local/git/bin/git-receive-pack /usr/bin/git-receive-pack
```
### 2.7 测试安装

```shell
# 获取git软件版本

git --version
```
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa3eb05578b666f5d7914857bd2bf13d.png)
## 3 创建Git用户

因为Git服务器需要安装在linux系统上，当使用远程客户端操作时，就需要提供相应的Git账号进行提交的，如果你的仓库文件的用户不是git的话，是root用户或者别的用户，那么你git push ,它是不允许的，因为你的git用户没有权限。你可以给这个文件创立git用户，或者修改文件夹的权限让所有用户都可以更改

```shell
# 增加用户

adduser git

# 设定密码

passwd git

# 或者
chmod -R 777 /usr/local/git
```
## 4 SSH免密登录

`建议在windows上进行操作练习`

### 4.1 Linux服务端操作

```shell
# 进入用户目录

cd /home/git

# 在git用户根目录创建.ssh目录

sudo mkdir .ssh

sudo touch .ssh/authorized_keys

# 设定.ssh目录，authorized_keys的权限

sudo chmod -R 700 /home/git/.ssh

sudo chmod 600 /home/git/.ssh/authorized_keys
```

### 4.2 windows客户端操作

```shell
# 在客户端生成SSH密钥

# 默认生成的密钥用户就是当前用户，需要和之前的全局配置保持一致

user.name=18801@LAPTOP-J9IRK5BM

user.email=18801@LAPTOP-J9IRK5BM

# 按照提示三次回车即可

ssh-keygen -t rsa
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/47ca24e54178c7ebf6117aa5413dbab3.png)

在用户根目录的.ssh文件夹内，id_rsa.pub就是我们要的公钥
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4745e55b4bd682b2a1512997905b0a73.png)

将文件中的内容复制到服务器端的.ssh/authorized_keys文件中
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5ba0e3d7306641e2eb3099f9eb854855.png)
## 5 创建Git版本库

### 5.1 创建文件目录

```shell
# 进入用户目录

cd /home/git

# 创建版本库目录

mkdir git-rep

# 设定文件所属用户

sudo chown git:git git-rep
```
### 5.2 初始化版本库

```shell
# 进入仓库目录

cd /home/git/git-rep

# 初始化仓库，和前面的git init略有不同

git init -bare test.git

# 设定文件所属用户

sudo chown -R git:git test.git
```
## 6 远程访问Git版本库

### 6.1 将远程仓库克隆到本地

```shell
# 将远程仓库克隆到本地，形成本地仓库

# 克隆远程仓库 => 用户@主机名:仓库地址

git clone git@linux1:/home/git/git-rep/test.git
```

### 6.2 提交文件到本地仓库

```shell
# 增加文件

git add client.txt

# 提交文件

git commit -m 'client'
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5838b0231339e7003e4dc2e85c849fcf.png)

### 6.3 将本地仓库同步到远程仓库

```shell
# 同步远程仓库

# 远程仓库默认有个别名叫origin，将本地仓库的文件推送（push）到远程仓库

**# git push** **远程仓库别名** **分支名称**

git push origin master
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/666be6e77e53459ec24df424c4547c27.png)

### 6.4 查看远程仓库

```shell
# 服务器端切换用户

su git

# 进入仓库

cd /home/git/git-rep/test.git

# 切换到主干分支

git checkout master

# 查看git日志

git log
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/feb585ecbc6a2d51b3a38f5f295049da.png)
