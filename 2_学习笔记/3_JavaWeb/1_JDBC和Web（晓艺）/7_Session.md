
## 1 Session

### 1.1 概念：

- 服务器端会话技术，将数据保存在服务器端的对象中,在一次会话的多次请求间共享数据。

### 1.2 常用方法：

1. 获取HttpSession对象：

     HttpSession session = request.getSession();

注意：

 i.  如果请求头中存在名为JSESSIONID的cookie,我们调用request.getSession()，会根据这个cookie的值（也就是sessionID）去服务器中找对应的session，如果找不到，也么重新创建一个新session，并在response中添加一个cookie(“JSESSIONID”, 新session的id)；如果能找到，那么返回对应的session.

ii. 如果请求头中不存在名为JSESSIONID的cookie，request.getSession()会直接创建一个新session，并在response中添加一个cookie(“JSESSIONID”, 新session的id)

2. 使用session对象：

   //获取存在session对象中的属性

     Object getAttribute(String name) 

   //往session中装入属性

     void setAttribute(String name, Object value)

   //删除存在session中的属性

     void removeAttribute(String name)      

### 1.3 原理

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

`Session的实现依赖于Cookie：`

主要是依赖一个名为JSESSIONID的cookie，存放的值为session的id，只有通过这个id才能找到之前在服务器中存储的session。

**4.**    **高频面试题**

1. 一般情况下，当客户端关闭后，服务器不关闭，两次获取session是否为同一个？

     默认情况下。不是。

     如果需要相同，则可以创建Cookie,键为JSESSIONID，设置最大存活时间，让cookie持久化保存。
```java
Cookie c = new Cookie("JSESSIONID",session.getId());

c.setMaxAge(60*60);

response.addCookie(c);
```

2. session什么时候被销毁？

     1. 服务器关闭

     2. session对象调用invalidate() 。

     3. session默认失效时间 30分钟

           可以通过如下方法设置session的存活时间，单位为秒

session.setMaxInactiveInterval(int seconds);

### 1.4 session的特点

1. session用于存储一次会话的多次请求的数据，存在服务器端

2. session可以存储任意类型，任意大小的数据

3. session存储数据在服务器端，Cookie在客户端

4. session相对安全，Cookie相对于不安全