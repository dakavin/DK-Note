
## 一、 会话保持技术

### 1.1 什么是会话

- 浏览器与服务器交互的过程，浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止（一般都是浏览器关闭）

- 会话的特点：
	- 一次会话包含很多次请求和响应
	- 但是http请求是无状态的：每次的请求都是独立的，它的执行情况和结果与前面的请求和之后的请求是无直接关系的
	- 所以需要一种机制来让服务器记得客户端

### 1.2 会话保持举例：记录登录状态

- 我们在购物的时候不可能每次下单都再去登录一次，那么就需要有相应的会话保持机制

### 1.3  会话保持机制

1. 客户端会话技术：Cookie----一种在请求头中添加的特殊标记

2. 服务器端会话技术：Session、Token

## 二、 Cookie

### 2.1 概念

- 客户端会话技术，将数据保存到客户端

### 2.2 快速入门

- 使用步骤：
  1. 创建Cookie对象，绑定数据
		new Cookie(String name, String value)
  2. 发送Cookie对象
		response.addCookie(Cookie cookie)
  3. 获取Cookie，拿到数据
		Cookie[]  request.getCookies() 

### 2.3 实现原理

 - 基于响应头set-cookie和请求头cookie实现

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

### 2.4 Cookie的细节

1. 一次可以发送多个cookie

2. cookie在浏览器中保存多长时间？

1）. 默认情况下，当浏览器关闭后，Cookie数据被销毁

2). 持久化存储：

setMaxAge(int seconds)

seconds为正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效

seconds为负数：当浏览器关闭后，Cookie数据被销毁

seconds为零：立即删除该Cookie信息

3. cookie可以存中文

在tomcat 8.5 之后，cookie支持中文数据。

### 2.5 Cookie的特点

1. Cookie存储数据在客户端浏览器

2. Cookie一般用于存出少量的不太敏感的数据，不能存密码之内的危险数据

### 2.6 浏览器同源策略

浏览器只会把同源的cookie发给服务器，同源是指协议、域名和端口号相同