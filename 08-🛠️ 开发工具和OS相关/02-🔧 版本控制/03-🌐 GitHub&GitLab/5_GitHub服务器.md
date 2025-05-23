
公司中，我们可以搭建中央服务器让项目组开发人员共享代码，但是如果我们的开发人员都是通过互联网进行协作，而不是在同一个地方，那么开发时，程序文件代码的版本管理就显得更加重要，这就需要搭建一个互联网的版本库，让不同地点的人都可以进行访问。这里我们不用自己搭建。因为GitHub网站已经帮助我们提供了共享版本库功能。所以我们接下来就讲解一下，如何使用GitHub网站所提供的功能使用Git。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/290a2f5e5a2539f989a55c6dd59d58c2.png)
## 1 注册网站会员

GitHub官网地址：[https://github.com/](https://github.com/)

填写你的邮箱地址和密码，姓名

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a5fd076c40f293a3e0415fd643114fe.png)

一顿操作，注册完毕后，进入你的主页
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d1287f24931d8822cfa1e36ee81d5e4.png)

## 2 创建新的仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a90de3051c03bcdabb79a74dc2eeacba.png)

输入仓库的相关信息
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f1ff2c0daa2d6b4415c3f5cd16923d42.png)

点击创建按钮，创建新的仓库。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4433bbad82a8d65b7114f2c7030da4df.png)

## 3 本地仓库的基本操作指令

```shell
# create a new repository on the command line

echo "# git-study" >> README.md

git init

git add README.md

git commit -m "first commit"

git branch -M main

git remote add origin git@github.com:Aitiger-coffee/git-study.git

git push -u origin main

# push an existing repository from the command line

git remote add origin git@github.com:Aitiger-coffee/git-study.git

git branch -M main

git push -u origin main
```
## 4 SSH免密登录

github支持两种同步方式“https”和“ssh”。如果使用https很简单基本不需要配置就可以使用，但是每次提交代码和下载代码时都需要输入用户名和密码。ssh模式比https模式的一个重要好处就是，每次push、pull、fetch等操作时，不用重复填写遍用户名密码。前提是你必须是这个项目的拥有者或者合作者，且配好了ssh key。
### 4.1 本地生成SSH秘钥

```shell
# ssh-keygen -t rsa -C GitHub账号

ssh-keygen -t rsa -C mikeklay@126.com
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea225923539b548bf96704b43cb27a21.png)

### 4.2 集成用户公钥

执行命令完成后`,在window本地用户.ssh目录C:\Users\用户名\.ssh`下面生成如下名称的公钥和私钥:
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e253640b51fe8ffb24ba6acf61504fb7.png)

按照操作步骤，将id_rsa.pub文件内容复制到GitHub仓库中
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/adfc5bfb848c0a152d5c01e490f9eaaa.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/539312c8e69660928036b86e21b9adc4.png)

点击Add按钮，增加SSH公钥信息
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/db625bd26f02d9de56644cf7aa3dd10c.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9d34992b14fcdf279ff276a19c57147.png)
## 5 设定全局用户

```shell
git config --global user.name dakkk

# 这里的邮箱地址需要为GitHub网站的注册账号

git config --global user.email mikeylay@126.com
```

## 6 创建本地库的远程地址

```shell
# 初始化本地仓库

git init

# 设置远程仓库

git remote add origin git@github.com:dakkk-mike/git-test.git
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53f1352df6cb1a983a89005d22f7e46e.png)
## 7 新增/提交本地仓库文件

```shell
# 新增文件

git add test.txt

# 提交文件

git commit test.txt
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8936f9b93af8fdc7c75386c840237c9.png)

## 8 推送到GitHub远程仓库

```shell
# 推送文件

git push origin master
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e80bf33ed3461eaa6029e57ee592a47c.png)

## 9 查看GitHub远程仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1c2e56ffeb5edc26927ffd66d9ed498f.png)

## 10 增加合作伙伴

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9454ba95640c9876820763191c0773da.png)

## 11 合作伙伴确认

合作伙伴收到确认后，点击Join按钮继续
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/896eb7e149c1e3a0255f5fb7997a3535.png)

点击Accept Invitation按钮，进行确认
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4c4696fabb3f18372b96976fc696814.png)

此时已经可以合作开发了。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b491f006fada93a3159d6ba03340e797.png)

## 12 远程仓库fork操作

如果项目存在大量合作伙伴，对于版本库的管理明显是一个特别大的风险，所以如果不想要选择大量的合作伙伴，但依然有人想要对项目代码进行维护，更新和扩展的话，此时，我们就可以使用fork功能。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23d03a02bed810baf7eba388094f780e.png)

选择一个你需要fork的开源项目，点击Create fork按钮即可
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8865849d8ad2458daf31ab90364c0adc.png)

这样就等同于创建了一个自己的远程仓库。但是这个远程仓库等同于是一个分支远程仓库，你可以随便操作，并不会影响源仓库，但是如果你的修改，更新想要融合到源仓库中，就需要提交申请了。

- 我们这里首先将文件改一下。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c6750f1c5dbf45ecafcf9c0f08b5012.png)

- 发送提交申请![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d8e980781dba1846ae73a9602f2aa3f9.png)
- 合并修改请求![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a8506c13db3bc076867075c2b3b788d.png)
- 修改请求确认![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/537f89d2d2f7ba7b0b12863842db2451.png)
