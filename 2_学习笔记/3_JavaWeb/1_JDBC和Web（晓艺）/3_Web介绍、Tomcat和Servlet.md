
## 一、 Web介绍

web项目通俗的说就是在服务器上跑的，可以进行网络访问的项目

### 1.1 架构

1. C/S：客户端/服务器端

2. B/S：浏览器/服务器端

### 1.2  资源分类

1）. 静态资源：所有用户访问后，得到的结果都是一样的，称为静态资源

如：html,css,JavaScript页面

2）. 动态资源:每个用户访问相同资源后，得到的结果可能不一样。

如：servlet               

### 1.3 网络通信三要素

1. IP：电子设备(计算机)在网络中的唯一标识。

2. 端口：应用程序在计算机中的唯一标识。 0~65536

3. 传输协议：规定了数据传输的规则

如：

jdbc:mysql://localhost:3306/lesson11?serverTimezone=UTC

https://www.baidu.com/

### 1.4 web服务器软件

web服务器软件：接收用户的请求，处理请求，做出响应的一种服务器软件

web服务器：可以安装了web服务器软件的服务器

开发人员可以在web服务器软件中部署web项目，让用户通过网络访问这些项目

- 常见的java相关的web服务器软件：
	- Tomcat：Apache基金组织，中小型的JavaEE服务器，开源的，免费的。
	- webLogic：oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
	- webSphere：IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
	- JBOSS：JBOSS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。

5. Tomcat的安装和使用

1. 下载：http://tomcat.apache.org/
2. 安装：解压压缩包即可。
   - 注意：安装目录建议不要有中文和空格
3.将Tomcat集成到IDEA中，运行项目。

  

## 二、      Servlet

### 2.1 概念

- 运行在服务器软件（如Tomcat）中的的小程序

- Servlet就是一个接口，定义了Java类被浏览器访问到(tomcat识别)的规则。

- 将来我们自定义一个类，实现Servlet接口，复写方法。

### 2.2 快速入门

1）. 创建JavaEE项目

2）. 定义一个类，实现Servlet接口
   public class ServletDemo1 implements Servlet

3）. 实现接口中的抽象方法

4）. 配置Servlet

补充知识点：
- xml是一种可以自定义标签的存储数据的文件，常被用来做配置文件
- 在web.xml中配置：
```xml
   <!--配置Servlet -->
<servlet>
	<servlet-name>demo1</servlet-name>
	<servlet-class>cn.yiliedu.servlet.ServletDemo1</servlet-class>
</servlet>

<!--路径映射规则-->
<servlet-mapping>
	<servlet-name>demo1</servlet-name>
	<url-pattern>/demo1</url-pattern>
</servlet-mapping>
```

5）. 部署Tomcat并访问验证

### 2.3 Servlet执行原理

1）. 当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径

2）. 查找web.xml文件，是否有对应的\<url-pattern>标签体内容。

3）. 如果有，则在找到对应的\<servlet-class>全类名

4）. Tomcat会将Servlet字节码文件加载进内存，并且创建其对象

5）. 调用其方法（init和service方法）

### 2.4 Servlet中的生命周期方法

1）. `被创建：执行init方法，只执行一次`

  Servlet什么时候被创建？

  默认情况下，第一次被访问时，Servlet被创建

单例和多线程安全问题

  Servlet在内存中只存在一个对象，Servlet是单例的

  多个用户同时访问时，可能存在操作同一个成员变量，出现线程安全问题。

  解决：尽量不要在Servlet中定义成员变量。如果定义了成员变量，也不要对修改值。

2）. `提供服务：执行service方法，执行多次`

   每次访问Servlet时，Service方法都会被调用一次。

3）. `被销毁：执行destroy方法，只执行一次`

   Servlet被销毁时执行。服务器关闭时，Servlet被销毁

   只有服务器正常关闭时，才会执行destroy方法。

   destroy方法在Servlet被销毁之前执行，一般用于释放资源


从Servlet3.0版本后支持注解配置，可以省去web.xml
直接在类上使用@WebServlet注解，进行配置
注解是一种jdk或引入的依赖jar所支持的一种特殊标记，支持就是说有相关的处理操作。
注解可以实现某种特定的功能。