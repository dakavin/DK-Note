## 目录

- [1 网络基础](#1%20%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80)
	- [1.1 URI 和 URL](#1.1%20URI%20%E5%92%8C%20URL)
	- [1.2 访问网络的流程](#1.2%20%E8%AE%BF%E9%97%AE%E7%BD%91%E7%BB%9C%E7%9A%84%E6%B5%81%E7%A8%8B)
	- [1.3 网络分层](#1.3%20%E7%BD%91%E7%BB%9C%E5%88%86%E5%B1%82)
	- [1.4 网络协议](#1.4%20%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE)
	- [1.5 IP协议](#1.5%20IP%E5%8D%8F%E8%AE%AE)
	- [1.6 TCP协议](#1.6%20TCP%E5%8D%8F%E8%AE%AE)
	- [1.7 DNS协议](#1.7%20DNS%E5%8D%8F%E8%AE%AE)
	- [1.8 Http协议](#1.8%20Http%E5%8D%8F%E8%AE%AE)
	- [1.9 各种协议与 HTTP 协议的关系](#1.9%20%E5%90%84%E7%A7%8D%E5%8D%8F%E8%AE%AE%E4%B8%8E%20HTTP%20%E5%8D%8F%E8%AE%AE%E7%9A%84%E5%85%B3%E7%B3%BB)
- [2 HTTP协议详解](#2%20HTTP%E5%8D%8F%E8%AE%AE%E8%AF%A6%E8%A7%A3)
	- [2.1 HTTP是一种不保存状态的协议](#2.1%20HTTP%E6%98%AF%E4%B8%80%E7%A7%8D%E4%B8%8D%E4%BF%9D%E5%AD%98%E7%8A%B6%E6%80%81%E7%9A%84%E5%8D%8F%E8%AE%AE)
	- [2.2 HTTP请求方法类型](#2.2%20HTTP%E8%AF%B7%E6%B1%82%E6%96%B9%E6%B3%95%E7%B1%BB%E5%9E%8B)
	- [2.3 HTTP请求报文](#2.3%20HTTP%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87)
	- [2.4 HTTP响应报文](#2.4%20HTTP%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87)
- [3 Servlet的实现类](#3%20Servlet%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%B1%BB)
	- [3.1 Servlet的体系结构](#3.1%20Servlet%E7%9A%84%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84)
	- [3.2 servlet处理流程：](#3.2%20servlet%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B%EF%BC%9A)

## 1 网络基础

### 1.1 URI 和 URL

- URI（Uniform Resource Identifier，统一资源标识符）

- URN（uniform resource name，统一资源命名），是通过名字来标识资源。

- URL（Uniform Resource Locator，统一资源定位符），是通过地址来标识资源

- URL的格式：
	`protocol://hostname:port/path/?query`

- 协议://主机名或域名:端口/路径/?参数键值对

### 1.2 访问网络的流程

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

### 1.3 网络分层

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

`总结：下层为上层服务，上层最接近用户，下层最接近网线`

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

1）发送端在层与层之间传输数据时，是一个封装的过程

每经过一层时必定会被打上一个该层所属的首部信息。

2）反之，接收端在层与层传输数据时，是一个解封装的过程。

每经过一层时会把对应的首部消去。

### 1.4 网络协议

概念：发送方和接收方约定好的某种规范

作用：发送数据的格式

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

### 1.5 IP协议

IP（Internet Protocol）网际协议位于网络层

`IP 协议的作用是把各种数据包传送给对方`

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

### 1.6 TCP协议

TCP 位于传输层，提供可靠的字节流服务。

TCP 协议为了更容易传送大数据才把数据分割，而且 TCP 协议能够确认数据最终是否送达到对方。

为了准确无误地将数据送达目标处，TCP 协议采用了三次握手

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

`经典面试题：为什么tcp连接时要三次握手？为什么二次握手不行？`

从确认的发送能力和接收能力角度考虑

### 1.7 DNS协议

   位于应用层

功能：域名解析成 IP 地址

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image014.jpg)

### 1.8 Http协议

Hyper Text Transfer Protocol 超文本传输协议

位于应用层

       功能：客户端和服务器端通信时，发送数据和接收数据的格式

### 1.9 各种协议与 HTTP 协议的关系

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image016.jpg)

       经典面试题：浏览器输入网址后的流程是什么？

    从上图总结

  

## 2 HTTP协议详解

### 2.1 HTTP是一种不保存状态的协议

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image018.jpg)

**为什么要这么做？**

这是为了更快的处理大量的请求，确保协议的可伸缩性。而特意把HTTP设计得这么简单的。

那么我要保存状态的话要怎么做？  
HTTP/1.1提出了相应的解决方法，虽然HTTP1.1也是无状态的协议，但是引入了Cookie技术。有了Cookie再使用HTTP通信，就可以管理状态了。后面我们会学习Cookie。

### 2.2 HTTP请求方法类型

请求方法类型：

1.   ` GET`

2.    `POST`

3.    OPTIONS

4.    HEAD

5.    PUT

6.    DELETE

7.    TRACE

8.    CONNECT  
  

`GET方法`。获取资源。用来请求访问一杯URI识别的资源。指定的资源经过服务器解析后返回的响应内容。

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image020.jpg)

`POST方法`。传输内容实体。虽然GET方法也可以用来传输内容实体，但是我们一般都不怎么做。POST的主要目的并不是获取响应的主体内容。

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image022.jpg)

### 2.3 HTTP请求报文

请求报文是由请求方法、请求 URI、协议版本、可选的请求首部字段和内容实体构成的

以post方式为例：

格式：

状态行(请求行)

报文头（请求头、响应首部字段）

空行

正文（请求体）

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image024.jpg)

注意：`get方式的报文没有内容实体`

可以在chrome浏览器中F12查看报文

### 2.4 HTTP响应报文

响应报文基本上由协议版本、状态码(表示请求成功或失败的数字 代码)、用以解释状态码的原因短语、可选的响应首部字段以及实体主体构成。

格式：

状态行(响应行)

报文头（响应头、响应首部字段）

空行

正文（响应主体）  
  

  
![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image026.jpg)

在起始行开头的 HTTP/1.1 表示服务器对应的 HTTP 版本。  
紧挨着的 200 OK 表示请求的处理结果的状态码(status code)和原因短语(reason-phrase)。下一行显示了创建响应的日期时间，是首部字段(header field)内的一个属性。  
接着以一空行分隔，之后的内容称为资源实体的主体(entity body)。  
  

补充：

HTTP协议以明文方式发送内容，不提供任何方式的数据加密

HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密, 数据传输的安全。


## 3 Servlet的实现类

### 3.1 Servlet的体系结构 

Servlet -- 接口
       |
GenericServlet -- 抽象类
       |
HttpServlet  -- 抽象类

  * `GenericServlet`：将Servlet接口中其他的方法做了默认空实现，只将service()方法作为抽象
	- 定义Servlet类时，可以继承GenericServlet，实现service()方法即可

  * `HttpServlet`：对http协议的一种封装，简化操作，它已经实现了service()方法，并在方法根据请求方法类型的不同，分别调用对应do方法（如doGet、doPost）

`我们可以直接自定义类继承HttpServlet，复写doGet/doPost方法`

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response){}

protected void doPost(HttpServletRequest request, HttpServletResponse response){}
```

### 3.2 servlet处理流程：

请求到达自定义servlet，先执行service方法（会去调用父类Httpservlet的service方法），父类HttpServlet的service方法根据请求方法类型的不同，分别调用对应do方法（如doGet、doPost），如果子类重写HttpServlet的doGet和doPost方法，那么实际调用的是子类的doGet和doPost方法。

request和response是tomcat服务器创建，在调用该方法时传入的，封装的信息分别是请求信息和响应信息。