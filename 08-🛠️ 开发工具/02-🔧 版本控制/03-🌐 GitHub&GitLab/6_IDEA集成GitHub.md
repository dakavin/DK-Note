---
文章标题: "[[6_IDEA集成GitHub]]"
文章作者: Dakkk
文章概要: |
  文章详细指导用户如何在IDEA中集成GitHub，实现代码版本控制。内容涵盖配置全局忽略文件、Git与GitHub账户设置，以及在IDEA中进行项目创建、提交、推送和分支操作，帮助初学者快速掌握协同开发。
tags:
  - IDEA
  - GitHub
  - Git
  - 版本控制
  - 集成开发环境
  - .gitignore
  - 分支管理
  - 代码管理
相关文章:
  - "[[../04-💡 实用技巧/0_Git问题解决]]"
  - "[[../04-💡 实用技巧/5_Commit 提交规范]]"
  - "[[1_本章代码及文章获取]]"
  - "[[../04-💡 实用技巧/6_git 同时推送到gitee 和 github]]"
  - "[[7_Gitee和GitLab集成]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/02-🔧 版本控制/03-🌐 GitHub&GitLab/6_IDEA集成GitHub.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

实际的开发中，代码都是采用IDE进行开发，所以我们这里介绍一下IDEA软件是如何集成GitHub远程仓库进行代码版本控制的。这里采用的IDEA版本为2022.2.1,其他版本的IDEA软件会略有差别

## 1 配置全局忽略文件

问题 1:为什么要忽略他们？

答：与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE 工具之 间的差异。 

问题 2：怎么忽略？ 

1）创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 git.ignore） 这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，建议也放在用 家目录下 户

.gitignore 文件模版内容如下：
```shell
# Compiled class file 
*.class

# Log file 
*.log 

# BlueJ files 
*.ctxt 

# Mobile Tools for Java (J2ME) 
.mtj.tmp/ 

# Package Files # 
*.jar 
*.war 
*.nar 
*.ear 
*.zip 
*.tar.gz 
*.rar 

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml hs_err_pid* 
.classpath 
.project 
.settings 
target 
.idea 
*.iml
```

- 正常的javaweb项目
```shell
##ignore this file##
/target/
/.idea/
/.settings/
/.vscode/
/bin/

.classpath
.project
.settings
.idea
 ##filter databfile、sln file##
*.mdb
*.ldb
*.sln
##class file##
*.com
*.class
*.dll
*.exe
*.o
*.so
# compression file
*.7z
*.dmg
*.gz
*.iso
*.jar
*.rar
*.tar
*.zip
*.via
*.tmp
*.err
*.log
*.iml
# OS generated files #
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Icon?
ehthumbs.db
Thumbs.db
.factorypath
/.mvn/
/mvnw.cmd
/mvnw
```

springboot项目
```bash
HELP.md
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**
!**/src/test/**

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache
.log

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr
.mvn
mvnw*

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/

### VS Code ###
.vscode/

### generated files ###
bin/
gen/

### MAC ###
.DS_Store

### Other ###
logs/
log
temp/
```



2）在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）
```shell
[user] 
	name = Layne email = Layne@atguigu.com 
[core] 
	excludesfile = C:/Users/asus/git.ignore 
```

`注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”`

## 2 配置Git软件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7d47dc18eb80388467ed2499f8db7ed.png)
## 3 配置GitHub账号

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/410d5827c324d9ec0442030ca3074d4e.png)

继续点击授权按钮
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c51cad0b927142b94af924509331070c.png)

继续点击授权按钮
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ff809bc6b4b57823981fad997fe6566.png)

输入GitHub账号密码
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0e6dcdf06ae0ea9a5641c50b56a5177d.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1166f68bf58480dc6241df6ec46aa21f.png)

## 4 创建项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e309be7f843ae6e6e44a632445f0f3d.png)

## 5 添加项目代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e978d4b3bcf792999a083e74d2773a6c.png)

## 6 创建本地版本库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a34d953428307960a2b891020aaba93a.png)

## 7 提交本地版本库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c4f5fe3072638b9d3330dbcc28218d3.png)

## 8 创建新的远程版本库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/83e8ff0d9abd6307cd5c972cdec10023.png)

## 9 推送到远程版本库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce72be2f6c5eeef3657601de8de4fae5.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/360497fc521f55051ea6976693581dd9.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/388eb51465595cb04683e15ba81e6041.png)

## 10 查看历史版本

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/07b2d0adaabaa669ace510a832a7be93.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e3076e6b0f53b8835238945bd834218.png)

## 11 分支操作

### 11.1 创建分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c892ec2f6b04961d741b665fea09b15.png)

### 11.2 将分支推送到远程仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a66e5b3202c1402ffff381341ccc1e6.png)

### 11.3 查看远程仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/151e7355d0f01ebbfd60b2ad48cefe04.png)