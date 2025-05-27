---
文章标题: "[[1_Tomcat]]" 
文章作者: Dakkk
文章概要: |
  文章涵盖Tomcat基础配置与部署，包括修改默认端口、详解BIO/NIO/APR连接器运行模式及优化参数配置，以及三种Web应用部署方法：直接放webapps、server.xml配置Context，或独立context XML文件部署。
文章标签:
- "Tomcat"
- "配置"
- "部署"
- "连接器"
- "端口"
- "性能优化"
- "server.xml"
- "I/O模式"
相关文章:
- "[[4_Maven启动Tomcat报错]]"
- "[[5_Tomcat10]]"
- "[[1_📕文章管理模块功能分析、表设计]]"
- "[[1_索引常见面试题]]"
- "[[1_Part1]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/1_JavaGuide/1_Java面试题/7_JavaWeb/1_Tomcat.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 Tomcat的缺省端口是多少，怎么修改

> Tomcat的缺省端口是多少，怎么修改

1. 找到Tomcat目录下的conf文件夹
2. 进入conf文件夹里面找到server.xml文件
3. 打开server.xml文件
4. 在server.xml文件里面找到下列信息

```xml
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443" />
```

1. 把port=”8080″改成port=”8888″，并且保存
2. 启动Tomcat，并且在IE浏览器里面的地址栏输入[http://127.0.0.1](https://link.segmentfault.com/?enc=wgmjE27aVIzkrpADbReP4w%3D%3D.0FG5iaU9nG%2BtyxDyOa16WZyP35VYQHr%2BUf%2BnLkj5pdc%3D):8888/

到tomcat主目录下的conf/server.xml文件中修改**,把8080端口改成是8088或者是其他的

## 2 Tomcat 有哪几种Connector 运行模式(优化)？

> tomcat 有哪几种Connector 运行模式(优化)？

1. bio(blocking I/O)
2. nio(non-blocking I/O)
3. apr(Apache Portable Runtime/Apache可移植运行库)

相关解释:

- bio: **传统的Java I/O操作，同步且阻塞IO。**
- nio: **JDK1.4开始支持，同步阻塞或同步非阻塞IO**
- aio(nio.2): **JDK7开始支持，异步非阻塞IO**
- apr: Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地 **提高Tomcat对静态文件的处理性能**

下面是**配置Tomcat运行模式改成是NIO模式，并配置连接池相关参数来进行优化**:

```xml
<!--
<Connector port="8080" protocol="HTTP/1.1"
		   connectionTimeout="20000"
		   redirectPort="8443" />
-->
<!-- protocol 启用 nio模式，(tomcat8默认使用的是nio)(apr模式利用系统级异步io) -->
<!-- minProcessors最小空闲连接线程数-->
<!-- maxProcessors最大连接线程数-->
<!-- acceptCount允许的最大连接数，应大于等于maxProcessors-->
<!-- enableLookups 如果为true,requst.getRemoteHost会执行DNS查找，反向解析ip对应域名或主机名-->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" 
	connectionTimeout="20000"
	redirectPort="8443

	maxThreads=“500” 
	minSpareThreads=“100” 
	maxSpareThreads=“200”
	acceptCount="200"
	enableLookups="false"       
/>
```

apr模式启动起来是比较复杂的，详情可参考:[http://blog.csdn.net/wanglei_storage/article/details/50225779](https://link.segmentfault.com/?enc=QTmaJ1n4H92zToEISrsbZg%3D%3D.iSblNzSMi3R4aelDrqstoFZ8HW6zwHy%2BElhPFSq0S%2B8cuyN3g9H8xSII9vtHjsZmk2sCQhuroROz3faxNu2ayA%3D%3D)

对于bio,nio,nio.2的理解可参考:[http://blog.csdn.net/itismelzp/article/details/50886009](https://link.segmentfault.com/?enc=LT9EEwtkKRH8l2TTk61%2FvA%3D%3D.VhCic3LBXVu05VzcoKJan7Fz7lGeQ21dXuMVyVzlS86MrkpWhxPWsasqLDpH4SM%2BW4QHiaXyx9bSTK2E90AQuw%3D%3D)

## 3 Tomcat有几种部署方式

### 3.1 部署方式1：

1. 直接把Web项目放在webapps下，Tomcat会自动将其部署
2. 在server.xml文件上配置`<Context>`节点，设置相关的属性即可
3. 通过Catalina来进行配置:进入到confCatalinalocalhost文件下，创建一个xml文件，该文件的名字就是站点的名字。编写XML的方式来进行设置。

### 3.2 部署方式2：

- 在其他盘符下创建一个web站点目录，并创建WEB-INF目录和一个html文件。

- 找到Tomcat目录下/conf/server.xml文件

- 在server.xml中的`<Host>`节点下添加如下代码。**path表示的是访问时输入的web项目名，docBase表示的是站点目录的绝对路径**
```xml
<Context path="/web1" docBase="D:\web1"/>
```

- 访问配置好的web站点

### 3.3 部署方式3：

- 进入到confCatalinalocalhost文件下，创建一个xml文件，**该文件的名字就是站点的名字。**
- xml文件的代码如下，**docBase是你web站点的绝对路径**

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<Context 
    docBase="D:\web1" 
    reloadable="true"> 
</Context> 
```

- 访问web站点下的html资源