
`.gitignore`是一份用于忽略不必提交的文件的列表，项目中可以根据实际需求统一`.gitignore`文件，减少不必要的文件提交和冲突，净化代码库环境。

## 重写忽略文件后，让其生效

```shell
git rm -r --cached .                       # 清除缓存 
git add .                                  # 追踪文件
git commit -m "更新.gitignore"             # 注释提交
git push origin master                     # 推送远程
```
## 问题 1:为什么要忽略他们？

答：与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE 工具之 间的差异。 

## 问题 2：怎么忽略？ 

1）创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 git.ignore） 这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，建议也放在用 家目录下 户

**全局通用的忽略文件模版**
```shell
HELP.md  
target/  
!.mvn/wrapper/maven-wrapper.jar  
!**/src/main/**/target/  
!**/src/test/**/target/  
  
### STS ###  
.apt_generated  
.classpath  
.factorypath  
.project  
.settings  
.springBeans  
.sts4-cache  
  
### IntelliJ IDEA ###  
.idea  
*.iws  
*.iml  
*.ipr  
  
### NetBeans ###  
/nbproject/private/  
/nbbuild/  
/dist/  
/nbdist/  
/.nb-gradle/  
build/  
!**/src/main/**/build/  
!**/src/test/**/build/  
  
### VS Code ###  
.vscode/  
  
# Log file  
*.log  
/logs*  
  
# BlueJ files  
*.ctxt  
  
# Mobile Tools for Java (J2ME)  
.mtj.tmp/  
  
# Package Files #  
*.jar  
*.war  
*.ear  
*.zip  
*.tar.gz  
*.rar  
*.cmd
```

**.gitignore 文件模版内容如下：**
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

**正常的javaweb项目**
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

**springboot项目**
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

**web项目的忽略文件/个人博客备份**
```
.Ds_store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.vscode/
.idea/
/.idea/
.deploy_git*/
.idea
themes/butterfLy/.git
```


**在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）**
```shell
[user] 
	name = Layne email = Layne@atguigu.com 
[core] 
	excludesfile = C:/Users/asus/git.ignore 
```

`注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”`