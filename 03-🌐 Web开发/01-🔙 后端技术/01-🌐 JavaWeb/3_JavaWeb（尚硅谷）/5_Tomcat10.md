---
文章标题: "[[5_Tomcat10]]" 
文章作者: Dakkk
文章概要: |
  详细介绍了Tomcat10服务器的安装配置、目录结构、Web项目标准结构及在IDEA中开发部署Web项目的完整流程。
tags:
- "Tomcat"
- "Web服务器"
- "JavaWeb"
- "IDEA配置"
- "项目部署"
- "Servlet"
- "服务器环境搭建"
- "开发环境"
相关文章:
- "[[6_Tomcat]]"
- "[[5_JaveWeb软件环境部署及相关设置]]"
- "[[1_Web乱码和路径问题总结]]"
- "[[18_日程管理开发]]"
- "[[6_Servlet]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/3_JavaWeb（尚硅谷）/5_Tomcat10.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-03-28 10:36:00
修改时间: 2025-05-28 00:06:25
---

## 1 Web服务器

> Web服务器通常由硬件和软件共同构成。

- 硬件：电脑，提供服务供其它客户电脑访问
    
- 软件：电脑上安装的服务器软件，安装后能提供服务给网络中的其他计算机，将本地文件映射成一个虚拟的url地址供网络中的其他人访问。
![|500|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3025acede124394ce544af4c0a97a548.png)

> 常见的JavaWeb服务器：

-   **Tomcat（Apache）**：当前应用最广的JavaWeb服务器
-   Jetty:更轻量级、更灵活的servlet容器
-   JBoss（Redhat红帽）：支持JavaEE，应用比较广EJB容器 –> SSH轻量级的框架代替
-   GlassFish（Orcale）：Oracle开发JavaWeb服务器，应用不是很广
-   Resin（Caucho）：支持JavaEE，应用越来越广
-   Weblogic（Orcale）：要钱的！支持JavaEE，适合大型项目
-   Websphere（IBM）：要钱的！支持JavaEE，适合大型项目
## 2  Tomcat服务器

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。
### 2.1 安装

> 版本

-   版本：企业用的比较广泛的是8.0和9.0,目前比较新正式发布版本是Tomcat10.0, Tomcat11仍然处于测试阶段。
-   JAVAEE 版本和Servlet版本号对应关系 [Jakarta EE Releases](https://jakarta.ee/release/) 

| **Servlet** Version | EE Version       |
| :------------------ | ---------------- |
| 6.1                 | Jakarta EE ?     |
| 6.0                 | Jakarta EE 10    |
| 5.0                 | Jakarta EE 9/9.1 |
| 4.0                 | JAVA EE 8        |
| 3.1                 | JAVA EE 7        |
| 3.1                 | JAVA EE 7        |
| 3.0                 | JAVAEE 6         |

+ Tomcat 版本和Servlet版本之间的对应关系

| **Servlet** Version | **Tomcat ** Version | **JDK** Version                         |
| :------------------ | :------------------ | :-------------------------------------- |
| 6.1                 | 11.0.x              | 17 and later                            |
| 6.0                 | 10.1.x              | 11 and later                            |
| 5.0                 | 10.0.x (superseded) | 8 and later                             |
| 4.0                 | 9.0.x               | 8 and later                             |
| 3.1                 | 8.5.x               | 7 and later                             |
| 3.1                 | 8.0.x (superseded)  | 7 and later                             |
| 3.0                 | 7.0.x (archived)    | 6 and later (7 and later for WebSocket) |


> 下载

-   Tomcat官方网站：[http://tomcat.apache.org/](http://tomcat.apache.org/ "http://tomcat.apache.org/")
-   安装版：需要安装，一般不考虑使用。
-   解压版: 直接解压缩使用，我们使用的版本。![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24c00799ef3e9a595bf521135e368552.png)
> `安装`

1. 正确安装JDK并配置JAVA_HOME和CATALINA_HOMR(以JDK17为例 [https://injdk.cn](https://injdk.cn/)中可以下载各种版本的JDK)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3df2001b85825a53908a4967dc84db09.png)
2. 解压tomcat到非中文无空格目录
3. 点击bin/startup.bat启动
4. 打开浏览器输入 [http://localhost:8080](http://localhost:8080/)访问测试
5. 直接关闭窗口或者运行 bin/shutdown.bat关闭tomcat
6. 处理dos窗口日志中文乱码问题: 修改`conf/logging.properties`,将所有的UTF-8修改为GBK
## 3 Tomcat目录及测试

- `bin`：该目录下存放的是二进制可执行文件，如果是安装版，那么这个目录下会有两个exe文件：tomcat10.exe、tomcat10w\.exe，前者是在控制台下启动Tomcat，后者是弹出GUI窗口启动Tomcat；如果是解压版，那么会有startup.bat和shutdown.bat文件，startup.bat用来启动Tomcat，但需要先配置JAVA\_HOME环境变量才能启动，shutdawn.bat用来停止Tomcat；

- `conf`：这是一个非常非常重要的目录，这个目录下有四个最为重要的文件：

  - **server.xml：配置整个服务器信息。例如修改端口号。默认HTTP请求的端口号是：8080**

  - tomcat-users.xml：存储tomcat用户的文件，这里保存的是tomcat的用户名及密码，以及用户的角色信息。可以按着该文件中的注释信息添加tomcat用户，然后就可以在Tomcat主页中进入Tomcat Manager页面了；

    ``` html
    <tomcat-users xmlns="http://tomcat.apache.org/xml"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
                  version="1.0">	
    	<role rolename="admin-gui"/>
    	<role rolename="admin-script"/>
    	<role rolename="manager-gui"/>
    	<role rolename="manager-script"/>
    	<role rolename="manager-jmx"/>
    	<role rolename="manager-status"/>
    	<user 	username="admin" 
    			password="admin" 
    			roles="admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status"
    	/>
    </tomcat-users>
    ```

- web.xml：部署描述符文件，这个文件中注册了很多MIME类型，即文档类型。这些MIME类型是客户端与服务器之间说明文档类型的，如用户请求一个html网页，那么服务器还会告诉客户端浏览器响应的文档是text/html类型的，这就是一个MIME类型。客户端浏览器通过这个MIME类型就知道如何处理它了。当然是在浏览器中显示这个html文件了。但如果服务器响应的是一个exe文件，那么浏览器就不可能显示它，而是应该弹出下载窗口才对。MIME就是用来说明文档的内容是什么类型的！

  - context.xml：对所有应用的统一配置，通常我们不会去配置它。

- `lib`：Tomcat的类库，里面是一大堆jar文件。如果需要添加Tomcat依赖的jar文件，可以把它放到这个目录中，当然也可以把应用依赖的jar文件放到这个目录中，这个目录中的jar所有项目都可以共享之，但这样你的应用放到其他Tomcat下时就不能再共享这个目录下的jar包了，所以建议只把Tomcat需要的jar包放到这个目录下；

- `logs`：这个目录中都是日志文件，记录了Tomcat启动和关闭的信息，如果启动Tomcat时有错误，那么异常也会记录在日志文件中。

- `temp`：存放Tomcat的临时文件，这个目录下的东西可以在停止Tomcat后删除！

- **`webapps`：存放web项目的目录，其中每个文件夹都是一个项目**；如果这个目录下已经存在了目录，那么都是tomcat自带的项目。其中ROOT是一个特殊的项目，在地址栏中访问：http://127.0.0.1:8080，没有给出项目目录时，对应的就是ROOT项目.http://localhost:8080/examples，进入示例项目。其中examples"就是项目名，即文件夹的名字。

- `work`：运行时生成的文件，最终运行的文件都在这里。通过webapps中的项目生成的！可以把这个目录下的内容删除，再次运行时会生再次生成work目录。当客户端用户访问一个JSP文件时，Tomcat会通过JSP生成Java文件，然后再编译Java文件生成class文件，生成的java和class文件都会存放到这个目录下。

- LICENSE：许可证。

- NOTICE：说明文件。

## 4 Web项目的标准结构

一个标准的可以用于发布的WEB项目标准结构如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/50e2e6506f63ee8e21e2b0fe8575247b.png)
- `app 本应用根目录`
    - `static 非必要目录`,约定俗成的名字,一般在此处放静态资源 ( css js img)
        
    - `WEB-INF 必要目录`,必须叫WEB-INF,受保护的资源目录,浏览器通过url不可以直接访问的目录
        
        - `classes 必要目录`,src下源代码,配置文件,编译后会在该目录下,web项目中如果没有源码,则该目录不会出现
            
        - `lib 必要目录`,项目依赖的jar编译后会出现在该目录下,web项目要是没有依赖任何jar,则该目录不会出现
            
        - `web.xml 必要文件`,web项目的基本配置文件. 较新的版本中可以没有该文件,但是学习过程中还是需要该文件
            
    - `index.html 非必要文件`,index.html/index.htm/index.jsp为默认的欢迎页

> url的组成部分和项目中资源的对应关系![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9d746b7144924759abc7342afa99bf4.png)
## 5 Web项目部署的方式

> 方式1 直接将编译好的项目放在webapps目录下 (已经演示)

> 方式2 将编译好的项目打成war包放在webapps目录下,tomcat启动后会自动解压war包(其实和第一种一样)

> 方式3 可以将项目放在非webapps的其他目录下,在tomcat中通过配置文件指向app的实际磁盘路径

- 在磁盘的自定义目录上准备一个app![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79bd7277a48ea6a9063d87a4e54577d4.png)
- 在tomcat的conf下创建Catalina/localhost目录,并在该目录下准备一个app.xml文件
```xml
<!-- 
	path: 项目的访问路径,也是项目的上下文路径,就是在浏览器中,输入的项目名称
    docBase: 项目在磁盘中的实际路径
 -->
<Context path="/app" docBase="D:\mywebapps\app" />
```
- 启动tomcat访问测试即可
## 6 IEDA中开发并部署web项目

### 6.1 IDEA关联本地Tomcat

> 可以在创建项目前设置本地tomcat,也可以在打开某个项目的状态下找到settings

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7533e18fa3bed09cecc2a0cb021a50b5.png)

> 找到 Build,Execution,Eeployment下的Application Servers ,找到+号

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/621a732249cca60636aca61a0e10a261.png)

> 选择Tomcat Server

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6cecf693e925d13ef49a040cc8209b49.png)

> 选择tomcat的安装目录

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e1a645526687faf85931d346a0b68b0.png)

> 点击ok

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c17c67668e50d0ed4e5b66df4955704.png)

> 关联完毕

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/511ada8bb943b83a391e494c40953c10.png)

### 6.2 IDEA创建web工程

> 推荐先创建一个空项目,这样可以在一个空项目下同时存在多个modules,不用后续来回切换之前的项目,当然也可以忽略此步直接创建web项目

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4fec5e3c71b7bbdc3c318f3207314a9c.png)

> 检查项目的SDK,语法版本,以及项目编译后的输出目录

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33ec83eebb7262007dc618644e1d005e.png)

> 先创建一个普通的JAVA项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cb610ba0083b429345e066ac517cb1d.png)

> 检查各项信息是否填写有误

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10cb0c485e86e26c195bdab33d4be723.png)

> 创建完毕后,为项目添加Tomcat依赖

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b97d1109f608b9065fdb76f67c312444.png)

> 选择modules,添加  framework support

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2698454b65676bcdd3d5d8b14e0dc64.png)

> 选择Web Application 注意Version,勾选  Create web.xml
> 补充：Version与Tomcat的版本有关，主要是和Servlet版本相关，具体看[安装](#安装)

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8d8bb1f5ef3b89c73471a57cd68a04c.png)

> 删除index.jsp ,替换为 index.html

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac2987db806065801b1df07ea643e69b.png)

> `处理配置文件`

- 在工程下创建resources目录,专门用于存放配置文件(都放在src下也行,单独存放可以尽量避免文件集中存放造成的混乱)
	- 标记目录为资源目录,不标记的话则该目录不参与编译![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cbb5ea292db0d3b49d702e7f98a991f6.png)
	- 标记完成后,显示效果如下![|200|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/347b878f65d419f61d8481e157293230.png)
> 处理依赖jar包问题

- 在WEB-INF下创建lib目录
    
- 必须在WEB-INF下,且目录名必须叫lib!!!
    
- 复制jar文件进入lib目录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35694958a60fb2414f0200600469d2b2.png)
- 将lib目录添加为当前项目的依赖,后续可以用maven统一解决
- 补充：`lib文件夹没有jar包，是无法出现add as library选项的` ![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b745c56d3299d20f5b26d3502cb60979.png)
- 环境级别推荐选择module 级别,降低对其他项目的影响,name可以空着不写
- 通过Project Structure，查看当前项目有那些环境依赖
- 在此位置,可以通过-号解除依赖![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71f047daad457024c434e64baa1e1ece.png)
### 6.3 IDEA部署运行web项目

> 检查idea是否识别modules为web项目并存在将项目构建成发布结构的配置

+ 就是检查工程目录下,web目录有没有特殊的识别标记![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cbcedcc80c0337a0d3646144bc4fa891.png)
+ 以及artifacts下,有没有对应 war_exploded,如果没有,就点击+号添加![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/957adb125411f2db84217e78b72957aa.png)
> 点击向下箭头,出现 Edit Configurations选项

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/06b5c8834bf82f47d04bb87b27967ccb.png)

> 出现运行配置界面

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8df920740b3ed0be50bbb264604c9ca.png)



> 点击+号,添加本地tomcat服务器

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb281180e0b59b11a5cc7a786e871151.png)

> 因为IDEA 只关联了一个Tomcat,红色部分就只有一个Tomcat可选

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80b17111c3869dd6dac4147c5ca55c4f.png)

> 选择Deployment,通过+添加要部署到Tomcat中的artifact

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cad25cc7c8f755e5981c0ba816de1d47.png)

> applicationContext中是默认的项目上下文路径,也就是url中需要输入的路径,这里可以自己定义,可以和工程名称不一样,也可以不写,但是要保留/,我们这里暂时就用默认的

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53ceeadbaac85b0d92d9e3ce1bf1368e.png)

> 点击apply 应用后,回到Server部分. After Launch是配置启动成功后,是否默认自动打开浏览器并输入URL中的地址,HTTP port是Http连接器目前占用的端口号

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/99e92f9344bb87a5e83e62adc81e2de5.png)

> 点击OK后,启动项目,访问测试

+ 绿色箭头是正常运行模式
+ "小虫子"是debug运行模式![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/658e1b24a23238c33d59f8bf7e6ffbe2.png)

+ 点击后,查看日志状态是否有异常![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cac1069434310f57f4be01d659717db3.png)

+ 浏览器自动打开并自动访问了index.html欢迎页![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/86ec0b76585476fae2a0ea5a28d3f5a6.png)
### 6.4 工程结构与发布的项目结构之间的目录对应关系

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9a9410fcade6938cafa794695ad74e4.png)

`注意：resources文件夹的内容，最后保存在classes文件夹中`

所以调取资源的时候，写相对路径即可！

### 6.5 IDEA部署并运行项目的原理

- idea并没有直接进将编译好的项目放入tomcat的webapps中
    
- idea根据关联的tomcat,创建了一个tomcat副本,将项目部署到了这个副本中
    
- idea的tomcat副本在`C:\用户\当前用户\AppData\Local\JetBrains\IntelliJIdea2022.2\tomcat\`中
    
- idea的tomcat副本并不是一个完整的tomcat,副本里只是准备了和当前项目相关的配置文件而已
    
- idea启动tomcat时,是让本地tomcat程序按照tomcat副本里的配置文件运行
    
- idea的tomcat副本部署项目的模式是通过`conf/Catalina/localhost/*.xml`配置文件的形式实现项目部署的
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e909a080453b17b9df91c7d9feac0646.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dc37c85cf65ef98c82c303d4bef7747e.png)
## 1 Web服务器

> Web服务器通常由硬件和软件共同构成。

- 硬件：电脑，提供服务供其它客户电脑访问
    
- 软件：电脑上安装的服务器软件，安装后能提供服务给网络中的其他计算机，将本地文件映射成一个虚拟的url地址供网络中的其他人访问。
![|500|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3025acede124394ce544af4c0a97a548.png)

> 常见的JavaWeb服务器：

-   **Tomcat（Apache）**：当前应用最广的JavaWeb服务器
-   Jetty:更轻量级、更灵活的servlet容器
-   JBoss（Redhat红帽）：支持JavaEE，应用比较广EJB容器 –> SSH轻量级的框架代替
-   GlassFish（Orcale）：Oracle开发JavaWeb服务器，应用不是很广
-   Resin（Caucho）：支持JavaEE，应用越来越广
-   Weblogic（Orcale）：要钱的！支持JavaEE，适合大型项目
-   Websphere（IBM）：要钱的！支持JavaEE，适合大型项目
## 2  Tomcat服务器

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。
### 2.1 安装

> 版本

-   版本：企业用的比较广泛的是8.0和9.0,目前比较新正式发布版本是Tomcat10.0, Tomcat11仍然处于测试阶段。
-   JAVAEE 版本和Servlet版本号对应关系 [Jakarta EE Releases](https://jakarta.ee/release/) 

| **Servlet** Version | EE Version       |
| :------------------ | ---------------- |
| 6.1                 | Jakarta EE ?     |
| 6.0                 | Jakarta EE 10    |
| 5.0                 | Jakarta EE 9/9.1 |
| 4.0                 | JAVA EE 8        |
| 3.1                 | JAVA EE 7        |
| 3.1                 | JAVA EE 7        |
| 3.0                 | JAVAEE 6         |

+ Tomcat 版本和Servlet版本之间的对应关系

| **Servlet** Version | **Tomcat ** Version | **JDK** Version                         |
| :------------------ | :------------------ | :-------------------------------------- |
| 6.1                 | 11.0.x              | 17 and later                            |
| 6.0                 | 10.1.x              | 11 and later                            |
| 5.0                 | 10.0.x (superseded) | 8 and later                             |
| 4.0                 | 9.0.x               | 8 and later                             |
| 3.1                 | 8.5.x               | 7 and later                             |
| 3.1                 | 8.0.x (superseded)  | 7 and later                             |
| 3.0                 | 7.0.x (archived)    | 6 and later (7 and later for WebSocket) |


> 下载

-   Tomcat官方网站：[http://tomcat.apache.org/](http://tomcat.apache.org/ "http://tomcat.apache.org/")
-   安装版：需要安装，一般不考虑使用。
-   解压版: 直接解压缩使用，我们使用的版本。![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24c00799ef3e9a595bf521135e368552.png)
> `安装`

1. 正确安装JDK并配置JAVA_HOME和CATALINA_HOMR(以JDK17为例 [https://injdk.cn](https://injdk.cn/)中可以下载各种版本的JDK)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3df2001b85825a53908a4967dc84db09.png)
2. 解压tomcat到非中文无空格目录
3. 点击bin/startup.bat启动
4. 打开浏览器输入 [http://localhost:8080](http://localhost:8080/)访问测试
5. 直接关闭窗口或者运行 bin/shutdown.bat关闭tomcat
6. 处理dos窗口日志中文乱码问题: 修改`conf/logging.properties`,将所有的UTF-8修改为GBK
## 3 Tomcat目录及测试

- `bin`：该目录下存放的是二进制可执行文件，如果是安装版，那么这个目录下会有两个exe文件：tomcat10.exe、tomcat10w\.exe，前者是在控制台下启动Tomcat，后者是弹出GUI窗口启动Tomcat；如果是解压版，那么会有startup.bat和shutdown.bat文件，startup.bat用来启动Tomcat，但需要先配置JAVA\_HOME环境变量才能启动，shutdawn.bat用来停止Tomcat；

- `conf`：这是一个非常非常重要的目录，这个目录下有四个最为重要的文件：

  - **server.xml：配置整个服务器信息。例如修改端口号。默认HTTP请求的端口号是：8080**

  - tomcat-users.xml：存储tomcat用户的文件，这里保存的是tomcat的用户名及密码，以及用户的角色信息。可以按着该文件中的注释信息添加tomcat用户，然后就可以在Tomcat主页中进入Tomcat Manager页面了；

    ``` html
    <tomcat-users xmlns="http://tomcat.apache.org/xml"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
                  version="1.0">	
    	<role rolename="admin-gui"/>
    	<role rolename="admin-script"/>
    	<role rolename="manager-gui"/>
    	<role rolename="manager-script"/>
    	<role rolename="manager-jmx"/>
    	<role rolename="manager-status"/>
    	<user 	username="admin" 
    			password="admin" 
    			roles="admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status"
    	/>
    </tomcat-users>
    ```

- web.xml：部署描述符文件，这个文件中注册了很多MIME类型，即文档类型。这些MIME类型是客户端与服务器之间说明文档类型的，如用户请求一个html网页，那么服务器还会告诉客户端浏览器响应的文档是text/html类型的，这就是一个MIME类型。客户端浏览器通过这个MIME类型就知道如何处理它了。当然是在浏览器中显示这个html文件了。但如果服务器响应的是一个exe文件，那么浏览器就不可能显示它，而是应该弹出下载窗口才对。MIME就是用来说明文档的内容是什么类型的！

  - context.xml：对所有应用的统一配置，通常我们不会去配置它。

- `lib`：Tomcat的类库，里面是一大堆jar文件。如果需要添加Tomcat依赖的jar文件，可以把它放到这个目录中，当然也可以把应用依赖的jar文件放到这个目录中，这个目录中的jar所有项目都可以共享之，但这样你的应用放到其他Tomcat下时就不能再共享这个目录下的jar包了，所以建议只把Tomcat需要的jar包放到这个目录下；

- `logs`：这个目录中都是日志文件，记录了Tomcat启动和关闭的信息，如果启动Tomcat时有错误，那么异常也会记录在日志文件中。

- `temp`：存放Tomcat的临时文件，这个目录下的东西可以在停止Tomcat后删除！

- **`webapps`：存放web项目的目录，其中每个文件夹都是一个项目**；如果这个目录下已经存在了目录，那么都是tomcat自带的项目。其中ROOT是一个特殊的项目，在地址栏中访问：http://127.0.0.1:8080，没有给出项目目录时，对应的就是ROOT项目.http://localhost:8080/examples，进入示例项目。其中examples"就是项目名，即文件夹的名字。

- `work`：运行时生成的文件，最终运行的文件都在这里。通过webapps中的项目生成的！可以把这个目录下的内容删除，再次运行时会生再次生成work目录。当客户端用户访问一个JSP文件时，Tomcat会通过JSP生成Java文件，然后再编译Java文件生成class文件，生成的java和class文件都会存放到这个目录下。

- LICENSE：许可证。

- NOTICE：说明文件。

## 4 Web项目的标准结构

一个标准的可以用于发布的WEB项目标准结构如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/50e2e6506f63ee8e21e2b0fe8575247b.png)
- `app 本应用根目录`
    - `static 非必要目录`,约定俗成的名字,一般在此处放静态资源 ( css js img)
        
    - `WEB-INF 必要目录`,必须叫WEB-INF,受保护的资源目录,浏览器通过url不可以直接访问的目录
        
        - `classes 必要目录`,src下源代码,配置文件,编译后会在该目录下,web项目中如果没有源码,则该目录不会出现
            
        - `lib 必要目录`,项目依赖的jar编译后会出现在该目录下,web项目要是没有依赖任何jar,则该目录不会出现
            
        - `web.xml 必要文件`,web项目的基本配置文件. 较新的版本中可以没有该文件,但是学习过程中还是需要该文件
            
    - `index.html 非必要文件`,index.html/index.htm/index.jsp为默认的欢迎页

> url的组成部分和项目中资源的对应关系![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9d746b7144924759abc7342afa99bf4.png)
## 5 Web项目部署的方式

> 方式1 直接将编译好的项目放在webapps目录下 (已经演示)

> 方式2 将编译好的项目打成war包放在webapps目录下,tomcat启动后会自动解压war包(其实和第一种一样)

> 方式3 可以将项目放在非webapps的其他目录下,在tomcat中通过配置文件指向app的实际磁盘路径

- 在磁盘的自定义目录上准备一个app![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79bd7277a48ea6a9063d87a4e54577d4.png)
- 在tomcat的conf下创建Catalina/localhost目录,并在该目录下准备一个app.xml文件
```xml
<!-- 
	path: 项目的访问路径,也是项目的上下文路径,就是在浏览器中,输入的项目名称
    docBase: 项目在磁盘中的实际路径
 -->
<Context path="/app" docBase="D:\mywebapps\app" />
```
- 启动tomcat访问测试即可
## 6 IEDA中开发并部署web项目

### 6.1 IDEA关联本地Tomcat

> 可以在创建项目前设置本地tomcat,也可以在打开某个项目的状态下找到settings

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7533e18fa3bed09cecc2a0cb021a50b5.png)

> 找到 Build,Execution,Eeployment下的Application Servers ,找到+号

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/621a732249cca60636aca61a0e10a261.png)

> 选择Tomcat Server

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6cecf693e925d13ef49a040cc8209b49.png)

> 选择tomcat的安装目录

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e1a645526687faf85931d346a0b68b0.png)

> 点击ok

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c17c67668e50d0ed4e5b66df4955704.png)

> 关联完毕

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/511ada8bb943b83a391e494c40953c10.png)

### 6.2 IDEA创建web工程

> 推荐先创建一个空项目,这样可以在一个空项目下同时存在多个modules,不用后续来回切换之前的项目,当然也可以忽略此步直接创建web项目

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4fec5e3c71b7bbdc3c318f3207314a9c.png)

> 检查项目的SDK,语法版本,以及项目编译后的输出目录

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33ec83eebb7262007dc618644e1d005e.png)

> 先创建一个普通的JAVA项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cb610ba0083b429345e066ac517cb1d.png)

> 检查各项信息是否填写有误

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10cb0c485e86e26c195bdab33d4be723.png)

> 创建完毕后,为项目添加Tomcat依赖

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b97d1109f608b9065fdb76f67c312444.png)

> 选择modules,添加  framework support

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2698454b65676bcdd3d5d8b14e0dc64.png)

> 选择Web Application 注意Version,勾选  Create web.xml
> 补充：Version与Tomcat的版本有关，主要是和Servlet版本相关，具体看[安装](#安装)

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8d8bb1f5ef3b89c73471a57cd68a04c.png)

> 删除index.jsp ,替换为 index.html

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac2987db806065801b1df07ea643e69b.png)

> `处理配置文件`

- 在工程下创建resources目录,专门用于存放配置文件(都放在src下也行,单独存放可以尽量避免文件集中存放造成的混乱)
	- 标记目录为资源目录,不标记的话则该目录不参与编译![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cbb5ea292db0d3b49d702e7f98a991f6.png)
	- 标记完成后,显示效果如下![|200|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/347b878f65d419f61d8481e157293230.png)
> 处理依赖jar包问题

- 在WEB-INF下创建lib目录
    
- 必须在WEB-INF下,且目录名必须叫lib!!!
    
- 复制jar文件进入lib目录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35694958a60fb2414f0200600469d2b2.png)
- 将lib目录添加为当前项目的依赖,后续可以用maven统一解决
- 补充：`lib文件夹没有jar包，是无法出现add as library选项的` ![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b745c56d3299d20f5b26d3502cb60979.png)
- 环境级别推荐选择module 级别,降低对其他项目的影响,name可以空着不写
- 通过Project Structure，查看当前项目有那些环境依赖
- 在此位置,可以通过-号解除依赖![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71f047daad457024c434e64baa1e1ece.png)
### 6.3 IDEA部署运行web项目

> 检查idea是否识别modules为web项目并存在将项目构建成发布结构的配置

+ 就是检查工程目录下,web目录有没有特殊的识别标记![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cbcedcc80c0337a0d3646144bc4fa891.png)
+ 以及artifacts下,有没有对应 war_exploded,如果没有,就点击+号添加![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/957adb125411f2db84217e78b72957aa.png)
> 点击向下箭头,出现 Edit Configurations选项

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/06b5c8834bf82f47d04bb87b27967ccb.png)

> 出现运行配置界面

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8df920740b3ed0be50bbb264604c9ca.png)



> 点击+号,添加本地tomcat服务器

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb281180e0b59b11a5cc7a786e871151.png)

> 因为IDEA 只关联了一个Tomcat,红色部分就只有一个Tomcat可选

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80b17111c3869dd6dac4147c5ca55c4f.png)

> 选择Deployment,通过+添加要部署到Tomcat中的artifact

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cad25cc7c8f755e5981c0ba816de1d47.png)

> applicationContext中是默认的项目上下文路径,也就是url中需要输入的路径,这里可以自己定义,可以和工程名称不一样,也可以不写,但是要保留/,我们这里暂时就用默认的

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53ceeadbaac85b0d92d9e3ce1bf1368e.png)

> 点击apply 应用后,回到Server部分. After Launch是配置启动成功后,是否默认自动打开浏览器并输入URL中的地址,HTTP port是Http连接器目前占用的端口号

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/99e92f9344bb87a5e83e62adc81e2de5.png)

> 点击OK后,启动项目,访问测试

+ 绿色箭头是正常运行模式
+ "小虫子"是debug运行模式![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/658e1b24a23238c33d59f8bf7e6ffbe2.png)

+ 点击后,查看日志状态是否有异常![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cac1069434310f57f4be01d659717db3.png)

+ 浏览器自动打开并自动访问了index.html欢迎页![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/86ec0b76585476fae2a0ea5a28d3f5a6.png)
### 6.4 工程结构与发布的项目结构之间的目录对应关系

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9a9410fcade6938cafa794695ad74e4.png)

`注意：resources文件夹的内容，最后保存在classes文件夹中`

所以调取资源的时候，写相对路径即可！

### 6.5 IDEA部署并运行项目的原理

- idea并没有直接进将编译好的项目放入tomcat的webapps中
    
- idea根据关联的tomcat,创建了一个tomcat副本,将项目部署到了这个副本中
    
- idea的tomcat副本在`C:\用户\当前用户\AppData\Local\JetBrains\IntelliJIdea2022.2\tomcat\`中
    
- idea的tomcat副本并不是一个完整的tomcat,副本里只是准备了和当前项目相关的配置文件而已
    
- idea启动tomcat时,是让本地tomcat程序按照tomcat副本里的配置文件运行
    
- idea的tomcat副本部署项目的模式是通过`conf/Catalina/localhost/*.xml`配置文件的形式实现项目部署的
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e909a080453b17b9df91c7d9feac0646.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dc37c85cf65ef98c82c303d4bef7747e.png)