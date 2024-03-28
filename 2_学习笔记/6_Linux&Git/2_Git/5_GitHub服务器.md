
公司中，我们可以搭建中央服务器让项目组开发人员共享代码，但是如果我们的开发人员都是通过互联网进行协作，而不是在同一个地方，那么开发时，程序文件代码的版本管理就显得更加重要，这就需要搭建一个互联网的版本库，让不同地点的人都可以进行访问。这里我们不用自己搭建。因为GitHub网站已经帮助我们提供了共享版本库功能。所以我们接下来就讲解一下，如何使用GitHub网站所提供的功能使用Git。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210207.png)
## 1、注册网站会员

GitHub官网地址：[https://github.com/](https://github.com/)

填写你的邮箱地址和密码，姓名

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210218.png)

一顿操作，注册完毕后，进入你的主页
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210237.png)

## 2、创建新的仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210242.png)

输入仓库的相关信息
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210251.png)

点击创建按钮，创建新的仓库。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210302.png)

## 3、本地仓库的基本操作指令

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
## 4、SSH免密登录

github支持两种同步方式“https”和“ssh”。如果使用https很简单基本不需要配置就可以使用，但是每次提交代码和下载代码时都需要输入用户名和密码。ssh模式比https模式的一个重要好处就是，每次push、pull、fetch等操作时，不用重复填写遍用户名密码。前提是你必须是这个项目的拥有者或者合作者，且配好了ssh key。
### 4.1 本地生成SSH秘钥

```shell
# ssh-keygen -t rsa -C GitHub账号

ssh-keygen -t rsa -C mikeklay@126.com
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210408.png)

### 4.2 集成用户公钥

执行命令完成后`,在window本地用户.ssh目录C:\Users\用户名\.ssh`下面生成如下名称的公钥和私钥:
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210502.png)

按照操作步骤，将id_rsa.pub文件内容复制到GitHub仓库中
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210517.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210526.png)

点击Add按钮，增加SSH公钥信息
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210536.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923210840.png)
## 5、设定全局用户

```shell
git config --global user.name dakkk

# 这里的邮箱地址需要为GitHub网站的注册账号

git config --global user.email mikeylay@126.com
```

## 6、创建本地库的远程地址

```shell
# 初始化本地仓库

git init

# 设置远程仓库

git remote add origin git@github.com:dakkk-mike/git-test.git
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211032.png)
## 7、新增/提交本地仓库文件

```shell
# 新增文件

git add test.txt

# 提交文件

git commit test.txt
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211109.png)

## 8、推送到GitHub远程仓库

```shell
# 推送文件

git push origin master
```

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211155.png)

## 9、查看GitHub远程仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211328.png)

## 10、增加合作伙伴

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211334.png)

## 11、合作伙伴确认

合作伙伴收到确认后，点击Join按钮继续
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211343.png)

点击Accept Invitation按钮，进行确认
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211351.png)

此时已经可以合作开发了。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211359.png)

## 12、远程仓库fork操作

如果项目存在大量合作伙伴，对于版本库的管理明显是一个特别大的风险，所以如果不想要选择大量的合作伙伴，但依然有人想要对项目代码进行维护，更新和扩展的话，此时，我们就可以使用fork功能。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211411.png)

选择一个你需要fork的开源项目，点击Create fork按钮即可
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211421.png)

这样就等同于创建了一个自己的远程仓库。但是这个远程仓库等同于是一个分支远程仓库，你可以随便操作，并不会影响源仓库，但是如果你的修改，更新想要融合到源仓库中，就需要提交申请了。

- 我们这里首先将文件改一下。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211442.png)

- 发送提交申请![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211459.png)
- 合并修改请求![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211512.png)
- 修改请求确认![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923211520.png)
