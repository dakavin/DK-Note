
## 1 JDK

- JDK8.0版本及之前的版本
	- 需要部署JAVA_HOME环境变量
	- 其中JAVA_HOME的地址为jdk安装所在的文件夹
	- 然后在path环境变量中 `%JAVA_HONM%/bin`即可

- JDK8.0之后的版本
	- 直接安装即可，会自动生成一个path环境变量

- 之间的关系，如果想要那个版本先响应，在path环境变量中直接置顶即可！！！

## 2 IDEA

### 2.1 普通项目的创建

- 新建项目的时候，`注意项目的存放位置！！！`

- 项目中的out文件夹不显示的时候，在project structure中，找到project中的out属性，创建一个out文件夹即可
- 然后项目中的模块的out，继承项目即可


### 2.2 使用Tomcat部署Web项目

- 新建一个module
- 鼠标右键module --- add Framework Support
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/987573c3cd7e35ae7108ad12cd4637c7.png)

- 选择JavaEE下的Web Application --- next即可，`JavaEE不同，web的版本也不同`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbe87f428777aceccf2e8dfd1387da9a.png)

- 打开上方的run Configuration选项
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/591c1ae9a1424192e443b56ebbeec267.png)

- 添加一个新的tomcat服务器
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2bdf19388f15fab4467fdcbe4e0cebb1.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/47ecdfc6444f7acb2aa5718a495f4178.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6d9b7138e263d6369fc41bd0ccd7f62d.png)

- 解决tomcat控制台乱码问题
- 不修改Tomcat配置文件logging.properties，而是在帮助的编辑自定义VM选项里加上
- `-Dfile.encoding=UTF-8`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/91a7f8d5f74ca4e9ccf8e65aa3cc0bf9.png)


- 关于Tomcat localhost log 和 tomcat catalina log乱码问题
	- 找到tomcat所在文件夹
	- 打开conf文件夹
	- 找到logging.properties文件
- 注意IDEA中控制台，即编辑页面都是UTF-8编码格式的
```Properties
#修改tomcat catalina log
1catalina.org.apache.juli.AsyncFileHandler.encoding = GBK

#修改Tomcat localhost log
2localhost.org.apache.juli.AsyncFileHandler.encoding = GBK
```

- 关于IDEA控制台上Tomcat localhost log不显示错误代码
- 同样在logging.properties文件
![[Pasted image 20230705201359.png]]

```properties
#添加一行代码
#java.util.logging.ConsoleHandler

#最后效果
org.apache.catalina.core.ContainerBase.[Catalina].[localhost].level = INFO

org.apache.catalina.core.ContainerBase.[Catalina].[localhost].handlers = 2localhost.org.apache.juli.AsyncFileHandler,java.util.logging.ConsoleHandler
```

## 3 关于进程占用问题

### 3.1 直接修改Tomcat端口配置

将JMX port的端口号修改为其他端口号，被占用的端口会自行销毁，不用理会

### 3.2 cmd直接杀死进程

```cmd
netstat -aon | findstr 1099

taskkill -f -pid PID 
```


如果报错：netstat不是内部或外部命令，也不是可运行的程序！！！

则进入system32文件夹中打开cmd

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14ad8481ef00cf7f2e21ef53ee213a10.png)


### 3.3 找不到1099端口号

打开控制面板，启动或关闭windows功能

找到Hyper-V,并关闭即可

重启电脑

## 4 解决乱码问题

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ca3a2b7b24feb56674d241ab014b484.png)

## 5 解决无法new Servlet问题

- 打开Project Structure
- 选择Facets
- 选中对应的模块
- 勾上sources root

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bed44e2049e48ceb55e0eb933e7d20f1.png)


- `修改模板`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/77a98e4aea45c482670c859208a3fa33.png)

## 6 解决无法读取jar包问题

- 在WEB-INF文件夹中新建 classes 和 lib文件夹
- 修改`.class文件`输出位置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f73b426dbad95211e7b497bebdcb2d8.png)
- 修改`lib文件`输出位置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/74526520920a64d0fdccd01c6c282d48.png)
- 忘记修改lib文件输出位置，或者直接将jar导入module的情况下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9e294a213f30883a53089a376ce287f.png)


## 7 解决Tomcat8版本，中文Cookie乱码问题

## 8 解决第一次访问服务页面，自动生成一个IDEA的cookie问题

直接清除浏览器cookie缓存即可！！！

## 9 自定义Servlet模板

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78c467dad064b846904fd1119d5a3a37.png)

## 10 关于Filter在IDEA中执行了2次或3次问题处理

### 10.1 **指定浏览器**

1、把默认浏览器替换成指定浏览器![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a459d7b9e4c804b2ecc1847d018fc77b.png)

2、然后指定IDEA中的默认浏览器选项Default Browsers:First listed![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7d265d85efb58379c9aac68f9f4cbaf0.png)

3、运行Tomcat程序，然后会自动弹出Chrome网页，如果弹出的Chrome网页doFilter次数依然不变，那么可能是Chrome没选择正确，我选的是ChromeCoreLauncher.exe，另外还有一个是ChromeCore.exe。弹出网页之后手动输入http://localhost:8080/地址，最后返回Tomcat终端查看程序。
`发现filter执行了2次`

### 10.2 **不指定浏览器**

1、去勾![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f517744751ef29f1bfe3df666d0fabad.png)

2、`此时filter执行1次`

 不指定浏览器是因为服务器在启动访问index.jsp页面时实际上是向浏览器发送了俩次请求，一次是程序本身Tomcat向浏览器发送的请求，一次是浏览器自动发送的/favicon.ico请求。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7a2b02b20f1c94bde4c64a0e35e89859.png)