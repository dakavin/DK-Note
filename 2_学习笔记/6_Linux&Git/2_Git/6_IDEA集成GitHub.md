
实际的开发中，代码都是采用IDE进行开发，所以我们这里介绍一下IDEA软件是如何集成GitHub远程仓库进行代码版本控制的。这里采用的IDEA版本为2022.2.1,其他版本的IDEA软件会略有差别

## 0、配置全局忽略文件

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

## 1、配置Git软件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212432.png)
## 2、配置GitHub账号

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212439.png)

继续点击授权按钮
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212448.png)

继续点击授权按钮
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212459.png)

输入GitHub账号密码
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212511.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212517.png)

## 3、创建项目

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212733.png)

## 4、添加项目代码

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212737.png)

## 5、创建本地版本库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212742.png)

## 6、提交本地版本库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212812.png)

## 7、创建新的远程版本库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212833.png)

## 8、推送到远程版本库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212848.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212856.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212904.png)

## 9、查看历史版本

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212910.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212918.png)

## 10、分支操作

### 10.1 创建分支

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212926.png)

### 10.2 将分支推送到远程仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923212935.png)

### 10.3 查看远程仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923214251.png)