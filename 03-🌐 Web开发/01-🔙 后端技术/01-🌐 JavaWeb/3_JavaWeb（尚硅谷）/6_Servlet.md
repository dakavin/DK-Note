---
文章标题: "[[6_Servlet]]" 
文章作者: Dakkk
文章概要: |
  详细介绍Servlet技术，包括静态动态资源概念、Servlet开发流程、生命周期、继承结构、请求响应对象及MVC架构模式。
tags:
- "Servlet"
- "JavaEE"
- "HttpServletRequest"
- "HttpServletResponse"
- "MVC架构"
- "生命周期"
- "Tomcat"
相关文章:
- "[[4_JavaEE的13项规范]]"
- "[[5_JaveWeb软件环境部署及相关设置]]"
- "[[1_Tomcat]]"
- "[[2_Servlet]]"
- "[[7_Linux系统软件安装-1]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/3_JavaWeb（尚硅谷）/6_Servlet.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-03-28 10:36:00
修改时间: 2025-05-27 23:34:19
---

## 1 Servlet简介

### 1.1 静态资源和动态资源

> 静态资源

+ 无需在程序运行时通过代码运行生成的资源,在程序运行之前就写好的资源. 例如:html css js img ,音频文件和视频文件

> 动态资源 

+ 需要在程序运行时通过代码运行生成的资源,在程序运行之前无法确定的数据,运行时动态生成,例如Servlet,Thymeleaf ... ...
+ 动态资源指的不是视图上的动画效果或者是简单的人机交互效果

> 生活举例

+ 去蛋糕店买蛋糕
  + 直接买柜台上已经做好的  : 静态资源
  + 和柜员说要求后现场制作  : 动态资源
### 1.2 Servlet简介

> Servlet (server applet) 是运行在服务端(tomcat)的Java小程序，是sun公司提供一套定义动态资源规范; 从代码层面上来讲Servlet就是一个接口

- 用来接收、处理客户端请求、响应给浏览器的动态资源。在整个Web应用中，Servlet主要负责接收处理请求、协同调度功能以及响应数据。我们可以把Servlet称为Web应用中的**控制器**![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b35034651b1c647025d1547841614008.png)
- 不是所有的JAVA类都能用于处理客户端请求,`能处理客户端请求并做出响应的一套技术标准就是Servlet`
+ `Servlet是运行在服务端的`,所以 Servlet必须在WEB项目中开发且在Tomcat这样的服务容器中运行

> 请求响应与HttpServletRequest和HttpServletResponse之间的对应关系

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c6a6248415e89313b308373aaa07cc7.png)
## 2 Servlet开发流程

### 2.1 目标

> 校验注册时,用户名是否被占用. 通过客户端向一个Servlet发送请求,携带username,如果用户名是'atguigu',则向客户端响应 NO,如果是其他,响应YES
### 2.2 开发过程

> 步骤1 开发一个web类型的module 

+ 过程参照之前

> 步骤2 开发一个UserServlet

``` java
public class UserServlet  extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取请求中的参数
        String username = req.getParameter("username");
        if("atguigu".equals(username)){
            //通过响应对象响应信息
            resp.getWriter().write("NO");
        }else{
            resp.getWriter().write("YES");
        }

    }
}
```

+ 自定义一个类,`要继承HttpServlet类`
+ `重写service方法`,该方法主要就是用于处理用户请求的服务方法
+ HttpServletRequest 代表请求对象,是有请求报文经过tomcat转换而来的,通过该对象可以获取请求中的信息
+ HttpServletResponse 代表响应对象,该对象会被tomcat转换为响应的报文,通过该对象可以设置响应中的信息
+ `Servlet对象的生命周期(创建,初始化,处理服务,销毁)是由tomcat管理的,无需我们自己new`
+ HttpServletRequest HttpServletResponse 两个对象也是有tomcat负责转换,在调用service方法时传入给我们用的

> 步骤3 在web.xml为UseServlet配置请求的映射路径

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
         version="5.0">

    <servlet>
        <!--给UserServlet起一个别名-->
        <servlet-name>userServlet</servlet-name>
        <servlet-class>com.atguigu.servlet.UserServlet</servlet-class>
    </servlet>


    <servlet-mapping>
        <!--关联别名和映射路径-->
        <servlet-name>userServlet</servlet-name>
        <!--可以为一个Servlet匹配多个不同的映射路径,但是不同的Servlet不能使用相同的url-pattern-->
        <url-pattern>/userServlet</url-pattern>
       <!-- <url-pattern>/userServlet2</url-pattern>-->
        <!--
            /        表示通配所有资源,不包括jsp文件
            /*       表示通配所有资源,包括jsp文件
            /a/*     匹配所有以a前缀的映射路径
            *.action 匹配所有以action为后缀的映射路径
        -->
       <!-- <url-pattern>/*</url-pattern>-->
    </servlet-mapping>

</web-app>
```

+ `Servlet并不是文件系统中实际存在的文件或者目录,所以为了能够请求到该资源,我们需要为其配置映射路径`
+ servlet的请求映射路径配置在web.xml中
+ servlet-name作为servlet的别名,可以自己随意定义,见名知意就好
+ url-pattern标签用于定义Servlet的请求映射路径
+ 一个servlet可以对应多个不同的url-pattern
+ 多个servlet不能使用相同的url-pattern
+ url-pattern中可以使用一些通配写法
  + /        表示通配所有资源,不包括jsp文件
  + /*      表示通配所有资源,包括jsp文件
  + /a/*     匹配所有以a前缀的映射路径
  + `*.action` 匹配所有以action为后缀的映射路径

> 步骤4 开发一个form表单,向servlet发送一个get请求并携带username参数

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="userServlet">
        请输入用户名:<input type="text" name="username" /> <br>
        <input type="submit" value="校验">
    </form>
</body>
</html>
```

> 启动项目,访问index.html ,提交表单测试

+ 使用debug模式运行测试![|500|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c66d72672e3a67de6a984e39266b58b3.png)
> 映射关系图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/31419a1e99089f2e74d7ed0325536341.png)
## 3 Servlet注解方式配置

### 3.1 @WebServlet注解源码

> 官方JAVAEEAPI文档下载地址

- [Java EE - Technologies (oracle.com)](https://www.oracle.com/java/technologies/javaee/javaeetechnologies.html#javaee8)
    
- @WebServlet注解的源码阅读
​
```java
package jakarta.servlet.annotation;  
​  
import java.lang.annotation.Documented;  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
​  
/**  
 * @since Servlet 3.0  
 */  
@Target({ ElementType.TYPE })  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface WebServlet {  
​  
    /**  
     * The name of the servlet  
     * 相当于 servlet-name  
     * @return the name of the servlet  
     */  
    String name() default "";  
​  
    /**  
     * The URL patterns of the servlet  
     * 如果只配置一个url-pattern ,则通过该属性即可,和urlPatterns属性互斥  
     * @return the URL patterns of the servlet  
     */  
    String[] value() default {};  
​  
    /**  
     * The URL patterns of the servlet  
     * 如果要配置多个url-pattern ,需要通过该属性,和value属性互斥  
     * @return the URL patterns of the servlet  
     */  
    String[] urlPatterns() default {};  
​  
    /**  
     * The load-on-startup order of the servlet  
     * 配置Servlet是否在项目加载时实例化  
     * @return the load-on-startup order of the servlet  
     */  
    int loadOnStartup() default -1;  
​  
    /**  
     * The init parameters of the servlet  
     * 配置初始化参数  
     * @return the init parameters of the servlet  
     */  
    WebInitParam[] initParams() default {};  
​  
    /**  
     * Declares whether the servlet supports asynchronous operation mode.  
     *  
     * @return {@code true} if the servlet supports asynchronous operation mode  
     * @see jakarta.servlet.ServletRequest#startAsync  
     * @see jakarta.servlet.ServletRequest#startAsync( jakarta.servlet.ServletRequest,jakarta.servlet.ServletResponse)  
     */  
    boolean asyncSupported() default false;  
​  
    /**  
     * The small-icon of the servlet  
     *  
     * @return the small-icon of the servlet  
     */  
    String smallIcon() default "";  
​  
    /**  
     * The large-icon of the servlet  
     *  
     * @return the large-icon of the servlet  
     */  
    String largeIcon() default "";  
​  
    /**  
     * The description of the servlet  
     *  
     * @return the description of the servlet  
     */  
    String description() default "";  
​  
    /**  
     * The display name of the servlet  
     *  
     * @return the display name of the servlet  
     */  
    String displayName() default "";  
​  
}  
```
### 3.2 @WebServlet注解使用

> 使用@WebServlet注解`替换`Servlet配置（即在web.xml文件中的配置）

``` java
@WebServlet(
        name = "userServlet",
        //value = "/user",
        urlPatterns = {"/userServlet1","/userServlet2","/userServlet"},
        initParams = {@WebInitParam(name = "encoding",value = "UTF-8")},
        loadOnStartup = 6
)
public class UserServlet  extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String encoding = getServletConfig().getInitParameter("encoding");
        System.out.println(encoding);
        // 获取请求中的参数
        String username = req.getParameter("username");
        if("atguigu".equals(username)){
            //通过响应对象响应信息
            resp.getWriter().write("NO");
        }else{
            resp.getWriter().write("YES");
        }
    }
}
```
## 4 Servlet生命周期

### 4.1 简介

> 什么是Servlet的生命周期

-   应用程序中的对象不仅在空间上有层次结构的关系，在时间上也会因为处于程序运行过程中的不同阶段而表现出不同状态和不同行为——这就是对象的生命周期。
-   简单的叙述生命周期，就是对象在容器中从开始创建到销毁的过程。

> Servlet容器

+ Servlet对象是Servlet容器创建的，生命周期方法都是由容器(目前我们使用的是Tomcat)调用的。这一点和我们之前所编写的代码有很大不同。在今后的学习中我们会看到，越来越多的对象交给容器或框架来创建，越来越多的方法由容器或框架来调用，开发人员要尽可能多的将精力放在业务逻辑的实现上。

> Servlet主要的生命周期执行特点

| 生命周期 | 对应方法                                                 | 执行时机               | 执行次数 |
| -------- | -------------------------------------------------------- | ---------------------- | -------- |
| 构造对象 | 构造器                                                   | 第一次请求或者容器启动 | 1        |
| 初始化   | init()                                                   | 构造完毕后             | 1        |
| 处理服务 | service(HttpServletRequest req,HttpServletResponse resp) | 每次请求               | 多次     |
| 销毁     | destory()                                                | 容器关闭               | 1        |

### 4.2 测试

> 开发servlet代码

``` java
package com.atguigu.servlet;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/ServletLifeCycle")
public class ServletLifeCycle  extends HttpServlet {
    public ServletLifeCycle(){
        System.out.println("构造器");
    }

    @Override
    public void init() throws ServletException {
        System.out.println("初始化方法");
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("service方法");
    }

    @Override
    public void destroy() {
        System.out.println("销毁方法");
    }
}

```

> 配置servlet

```xml
<servlet>
	<servlet-name>servletLifeCycle</servlet-name>
	<servlet-class>com.atguigu.servlet.ServletLifeCycle</servlet-class>
	<!--load-on-startup
		如果配置的是正整数则表示容器在启动时就要实例化Servlet,
		数字表示的是实例化的顺序
	-->
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>servletLifeCycle</servlet-name>
	<url-pattern>/servletLiftCycle</url-pattern>
</servlet-mapping>
```

> 测试：

- 启动tomcat，第一次访问/ServletLifeCycle
	1. 构造器
	2. 初始化方法
	3. service方法
- 后续刷新页面
	- 重复service方法
- 关闭tomcat
	- 执行销毁方法

### 4.3 总结

1. 通过生命周期测试我们发现Servlet对象在容器中是单例的
    
2. 容器是可以处理并发的用户请求的,每个请求在容器中都会开启一个线程
    
3. 多个线程可能会使用相同的Servlet对象,所以在Servlet中,`我们不要轻易定义一些容易经常发生修改的成员变量,因为在并发请求下，会引发线程安全问题`
    
4. load-on-startup中定义的正整数表示实例化顺序,如果数字重复了,容器会自行解决实例化顺序问题,但是应该避免重复
    
5. Tomcat容器中,已经定义了一些随系统启动实例化的servlet,我们自定义的servlet的load-on-startup尽量不要占用数字1-5
## 5 Servlet继承结构

- HttpServlet抽象类继承于GenericServlet抽象类

- GenericServlet抽象类实现了Servlet接口

- GenericServlet抽象类：侧重于除了service方法，其他方法的处理

- HttpServlet抽象类：侧重于service方法的处理

### 5.1 Servlet接口

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2743ab244130f6c9eef4fe301d312ed6.png)

> 接口及方法说明

- Servlet 规范接口,所有的Servlet必须实现
    
    - public void init(ServletConfig config) throws ServletException;
        - 初始化方法,容器在构造servlet对象后,自动调用的方法,容器负责实例化一个ServletConfig对象,并在调用该方法时传入
            
        - ServletConfig对象可以为Servlet 提供初始化参数
            
    - public ServletConfig getServletConfig();
        - 获取ServletConfig对象的方法,后续可以通过该对象获取Servlet初始化参数
            
    - public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
        - 处理请求并做出响应的服务方法,每次请求产生时由容器调用
            
        - 容器创建一个ServletRequest对象和ServletResponse对象,容器在调用service方法时,传入这两个对象
            
    - public String getServletInfo();
        - 获取ServletInfo信息的方法
            
    - public void destroy();
        - Servlet实例在销毁之前调用的方法
### 5.2 GenericServlet 抽象类

> 源码解释

+ GenericServlet 抽象类是对Servlet接口一些固定功能的粗糙实现,以及对service方法的再次抽象声明,并定义了一些其他相关功能方法
  + private transient ServletConfig config; 
    + 初始化配置对象作为属性
  + public GenericServlet() { } 
    + 构造器,为了满足继承而准备
  + public void destroy() { } 
    + 销毁方法的平庸实现
  + public String getInitParameter(String name) 
    + 获取初始参数的快捷方法
  + public `Enumeration<String>` getInitParameterNames() 
    + 返回所有初始化参数名的方法
  + public ServletConfig getServletConfig()
    +  获取初始Servlet初始配置对象ServletConfig的方法
  + public ServletContext getServletContext()
    +  获取上下文对象ServletContext的方法
  + public String getServletInfo() 
    + 获取Servlet信息的平庸实现
  + public void init(ServletConfig config) throws ServletException() 
    + 初始化方法的实现,并在此调用了init的重载方法
  + public void init() throws ServletException 
    + 重载init方法,为了让我们自己定义初始化功能的方法
  + public void log(String msg) 
  + public void log(String message, Throwable t)
    +  打印日志的方法及重载
  + public abstract void service(ServletRequest req, ServletResponse res) throws ServletException, IOException; 
    + 服务方法再次声明
  + public String getServletName() 
    + 获取ServletName的方法

### 5.3 HttpServlet 抽象类

> 解释

- abstract class HttpServlet extends GenericServlet HttpServlet抽象类,除了基本的实现以外,增加了更多的基础功能
    
    - private static final String METHOD_DELETE = "DELETE";
    - private static final String METHOD_HEAD = "HEAD";
    - private static final String METHOD_GET = "GET";
    - private static final String METHOD_OPTIONS = "OPTIONS";
    - private static final String METHOD_POST = "POST";
    - private static final String METHOD_PUT = "PUT";
    - private static final String METHOD_TRACE = "TRACE";
        
        - 上述属性用于定义常见请求方式名常量值
            
    - public HttpServlet() {}
        
        - 构造器,用于处理继承
            
    - public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException
        
        - 对服务方法的实现
            
        - 在该方法中,`将请求和响应对象转换成对应HTTP协议的HttpServletRequest HttpServletResponse对象`
            
        - 调用重载的service方法
            
    - public void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException
        
        - 重载的service方法,被重写的service方法所调用
            
        - 在该方法中,通过请求方式判断,调用具体的do***方法完成请求的处理
            
    - protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doHead(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doPut(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doDelete(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doOptions(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
    - protected void doTrace(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
        
        - 对应不同请求方式的处理方法
            
        - 除了doOptions和doTrace方法,其他的do*** 方法都在故意响应错误信息，`即我们自己写的基础类，必须重写service方法或doxxx方法`

### 5.4 自定义 Servlet

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d28bb0175dbb8d60a0841aa539937f8e.png)

自定义Servlet中,必须要对处理请求的方法进行重写

+ `要么重写service方法`
+ `要么重写doGet/doPost方法`
## 6 ServletConfig 和 ServletContext

### 6.1 ServletConfig的使用（开发用的少）

> ServletConfig是什么

- 为Servlet提供初始配置参数的一种对象,`每个Servlet都有自己独立唯一的ServletConfig对象`
    
- 容器会为每个Servlet实例化一个ServletConfig对象,并通过Servlet生命周期的init方法传入给Servlet作为属性![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c9f679b70e5e1917a7a6fbd82e2dcc73.png)
> ServletConfig是一个接口,定义了如下API

```java 
public interface ServletConfig {  
    String getServletName();  
    ServletContext getServletContext();  
    String getInitParameter(String var1);  
    Enumeration<String> getInitParameterNames();  
}
```

| 方法名                     | 作用                                                      |
| ----------------------- | ------------------------------------------------------- |
| getServletName()        | 获取<servlet-name>HelloServlet</servlet-name>定义的Servlet名称 |
| getServletContext()     | 获取ServletContext对象                                      |
| getInitParameter()      | 获取配置Servlet时设置的『初始化参数』，根据名字获取值                          |
| getInitParameterNames() | 获取所有初始化参数名组成的Enumeration对象                              |

> ServletConfig怎么用,测试代码如下

- 定义Servlet
```java

public class ServletA extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        ServletConfig servletConfig = this.getServletConfig();  
        // 根据参数名获取单个参数  
        String value = servletConfig.getInitParameter("param1");  
        System.out.println("param1:"+value);  
        // 获取所有参数名  
        Enumeration<String> parameterNames = servletConfig.getInitParameterNames();  
        // 迭代并获取参数名  
        while (parameterNames.hasMoreElements()) {  
            String paramaterName = parameterNames.nextElement();  
            System.out.println(paramaterName+":"+servletConfig.getInitParameter(paramaterName));  
        }  
    }  
}  
```

+ 配置Servlet

``` xml
  <servlet>
       <servlet-name>ServletA</servlet-name>
       <servlet-class>com.atguigu.servlet.ServletA</servlet-class>
       <!--配置ServletA的初始参数-->
       <init-param>
           <param-name>param1</param-name>
           <param-value>value1</param-value>
       </init-param>
       <init-param>
           <param-name>param2</param-name>
           <param-value>value2</param-value>
       </init-param>
   </servlet>

    <servlet-mapping>
        <servlet-name>ServletA</servlet-name>
        <url-pattern>/servletA</url-pattern>
    </servlet-mapping>
```

- 也可以通过@WebServlet注解进行配置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c36c07ab6c0a1b213a673043047fce8.png)

### 6.2 ServletContext的使用（重要）

> ServletContext是什么

+ ServletContext对象有称呼为上下文对象,或者叫应用域对象(后面统一讲解域对象)
+ 容器会为`每个app创建一个独立的唯一的ServletContext对象`
+ ServletContext对象`为所有的Servlet所共享`
+ ServletContext可以为所有的Servlet提供初始配置参数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/655fbb54ae905b459e6b088be04fc4ab.png)

> ServletContext怎么用

+ 配置ServletContext参数

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
         version="5.0">

    <context-param>
        <param-name>paramA</param-name>
        <param-value>valueA</param-value>
    </context-param>
    <context-param>
        <param-name>paramB</param-name>
        <param-value>valueB</param-value>
    </context-param>
</web-app>
```

+ 在Servlet中获取ServletContext并获取参数
```java
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       
        // 获取servletContext
        // 方法1：GenericServlet的getServletContext方法
        // 方法2：使用ServletConfig对象获取
        // 方法3：使用request对象获取
        ServletContext servletContext = this.getServletContext();
        // 获取所有参数名为paramA的value
        String valueA = servletContext.getInitParameter("paramA");
        System.out.println("paramA:"+valueA);
        // 获取所有参数名
        Enumeration<String> initParameterNames = servletContext.getInitParameterNames();
        // 迭代并获取参数名
        while (initParameterNames.hasMoreElements()) {
            String paramaterName = initParameterNames.nextElement();
            System.out.println(paramaterName+":"+servletContext.getInitParameter(paramaterName));
        }
    }
}
```
### 6.3 ServletContext其他重要的API

> `获取资源的真实路径（本地绝对路径）`
> 避免项目部署在其他主机后，需要更换路径的问题
> 主要涉及文件的上下传问题

``` java
String realPath = servletContext.getRealPath("资源在web目录中的路径");
```

+ 例如我们的目标是需要获取项目中某个静态资源的路径，不是工程目录中的路径，而是**部署目录中的路径**；我们如果直接拷贝其在我们电脑中的完整路径的话其实是有问题的，因为如果该项目以后部署到公司服务器上的话，路径肯定是会发生改变的，所以我们需要`使用代码动态获取资源的真实路径.  只要使用了servletContext动态获取资源的真实路径`，**那么无论项目的部署路径发生什么变化，都会动态获取项目运行时候的实际路径**，所以就不会发生由于写死真实路径而导致项目部署位置改变引发的路径错误问题

> 获取项目的上下文路径（即项目的访问路径）
> 主要涉及：后端动态获取项目根路径

![|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/72550dd23ae7442c9e3b6af90b021734.png)
[2.6.5 IDEA部署并运行项目的原理](5_XML和HTTP.md#2.6.5%20IDEA部署并运行项目的原理)

``` java
String contextPath = servletContext.getContextPath();
```

+ 项目的部署名称,也叫项目的上下文路径,在部署进入tomcat时所使用的路径,该路径是可能发生变化的,通过该API动态获取项目真实的上下文路径,可以**帮助我们解决一些后端页面渲染技术或者请求转发和响应重定向中的路径问题**

>  域对象的相关API

+ 域对象: 一些用于存储数据和传递数据的对象,传递数据不同的范围,我们称之为不同的域,不同的域对象代表不同的域,共享数据的范围也不同

+ `ServletContext代表应用`,所以ServletContext域也叫作应用域,是`webapp中最大的域`,可以在本应用内实现数据的共享和传递

+ webapp中的三大域对象,分别是`应用域,会话域,请求域`

+ `后续我们会将三大域对象统一进行讲解和演示`,三大域对象都具有的API如下

| API                                         | 功能解释       |
| ------------------------------------------- | ---------- |
| void setAttribute(String key,Object value); | 向域中存储/修改数据 |
| Object getAttribute(String key);            | 获得域中的数据    |
| void removeAttribute(String key);           | 移除域中的数据    |
|                                             |            |

## 7 HttpServletRequest

### 7.1 简介

> HttpServletRequest是什么

+ HttpServletRequest是一个接口,其父接口是ServletRequest
+ HttpServletRequest是Tomcat将请求报文转换封装而来的对象,在Tomcat调用service方法时传入
+ HttpServletRequest代表客户端发来的请求,所有请求中的信息都可以通过该对象获得
### 7.2 常见API

> HttpServletRequest怎么用

+ 获取`请求行信息`相关(`方式,请求的url,协议及版本`)

| API                           | 功能解释                    |
| ----------------------------- | ----------------------- |
| StringBuffer getRequestURL(); | 获取客户端请求的url（`全路径`）      |
| `String getRequestURI()`;     | 获取客户端请求项目中的具体资源         |
| int getServerPort();          | 获取客户端发送请求时的端口（`代理的端口号`） |
| int getLocalPort();           | 获取本应用在所在容器的端口           |
| int getRemotePort();          | 获取客户端程序的端口              |
| String getScheme();           | 获取请求协议                  |
| String getProtocol();         | 获取请求协议及版本号              |
| `String getMethod()`;         | 获取请求方式                  |
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c6dd9106ebd95694551685e9ac76454a.png)

+ 获得`请求头`信息相关

| API                                   | 功能解释               |
| ------------------------------------- | ---------------------- |
| String getHeader(String headerName);  | 根据头名称获取请求头   |
| Enumeration<String> getHeaderNames(); | 获取所有的请求头名字   |
| String getContentType();              | 获取content-type请求头 |

+ 获得`请求参数`相关

| API                                                     | 功能解释               |
| ------------------------------------------------------- | ------------------ |
| `String getParameter(String parameterName)`;            | 根据请求参数名获取请求单个参数值   |
| `String[] getParameterValues(String parameterName)`;    | 根据请求参数名获取请求多个参数值数组 |
| `Enumeration<String> getParameterNames()`;              | 获取所有请求参数名          |
| `Map<String, String[]> getParameterMap()`;              | 获取所有请求参数的键值对集合     |
| BufferedReader getReader() throws IOException;          | 获取读取请求体的字符输入流      |
| ServletInputStream getInputStream() throws IOException; | 获取读取请求体的字节输入流      |
| int getContentLength();                                 | 获得请求体长度的字节数        |
- `注意：`
	- `在input类型为checkbox（复选框的时候），同个name会有多个value，应该使用第2个方法`
	- 其余未着重的方法，主要用于处理请求体中有JSON字符串、文件的情况

+ 其他API

| API                                            | 功能解释               |
| ---------------------------------------------- | ------------------ |
| `String getServletPath()`;                     | 获取请求的Servlet的映射路径  |
| `ServletContext getServletContext()`;          | 获取ServletContext对象 |
| `Cookie[] getCookies()`;                       | 获取请求中的所有cookie     |
| `HttpSession getSession()`;                    | 获取Session对象        |
| `void setCharacterEncoding(String encoding) `; | 设置请求体字符集           |
- 注意：
	- getContextPath() 和 getServletPath() 方法的区别![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a550f17bab65966f825f8b4a08f402a3.png)
	- 一般我们将ContextPath设置为 `/`
## 8 HttpServletResponse

### 8.1 简介

> HttpServletResponse是什么

+ HttpServletResponse是一个接口,其父接口是ServletResponse
+ HttpServletResponse是Tomcat预先创建的,在Tomcat调用service方法时传入
+ HttpServletResponse代表对客户端的响应,该对象会被转换成响应的报文发送给客户端,通过该对象我们可以设置响应信息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c6a6248415e89313b308373aaa07cc7.png)
### 8.2 常见API

> HttpServletRequest怎么用

+ 设置`响应行`相关

| API                        | 功能解释       |
| -------------------------- | -------------- |
| void setStatus(int  code); | 设置响应状态码 |


+ 设置`响应头`相关

| API                                                    | 功能解释                              |
| ------------------------------------------------------ | --------------------------------- |
| void setHeader(String headerName, String headerValue); | 设置/修改响应头键值对                       |
| `void setContentType(String contentType)`;             | 设置content-type响应头及响应字符集(设置MIME类型) |

+ 设置`响应体`相关

| API                                                       | 功能解释                                |
| --------------------------------------------------------- | ----------------------------------- |
| PrintWriter getWriter() throws IOException;               | 获得向响应体放入信息的字符输出流                    |
| ServletOutputStream getOutputStream() throws IOException; | 获得向响应体放入信息的字节输出流                    |
| `void setContentLength(int length)`;                      | 设置响应体的字节长度,其实就是在设置content-length响应头 |

+ `其他API`

| API                                                          | 功能解释                       |
| ------------------------------------------------------------ | -------------------------- |
| void sendError(int code, String message) throws IOException; | 向客户端响应错误信息的方法,需要指定响应码和响应信息 |
| `void addCookie(Cookie cookie)`;                             | 向响应体中增加cookie              |
| `void setCharacterEncoding(String encoding)`;                | 设置响应体字符集                   |

> `MIME类型`

^cdb658

+ MIME类型,可以理解为文档类型,用户表示传递的数据是属于什么类型的文档
+ 浏览器可以根据MIME类型决定该用什么样的方式解析接收到的响应体数据
+ 可以这样理解: 前后端交互数据时,告诉对方发给对方的是 html/css/js/图片/声音/视频/... ...
+ tomcat/conf/web.xml中配置了常见文件的拓展名和MIMIE类型的对应关系
+ 常见的MIME类型举例如下

| 文件拓展名                  | MIME类型               |
| --------------------------- | ---------------------- |
| .html                       | text/html              |
| .css                        | text/css               |
| .js                         | application/javascript |
| .png /.jpeg/.jpg/... ...    | image/jpeg             |
| .mp3/.mpe/.mpeg/ ... ...    | audio/mpeg             |
| .mp4                        | video/mp4              |
| .m1v/.m1v/.m2v/.mpe/... ... | video/mpeg             |
## 9 请求转发和响应重定向

### 9.1 概述

> 什么是请求转发和响应重定向

+ 请求转发和响应重定向是web应用中，间接访问项目资源的两种手段,也是Servlet控制页面跳转的两种手段

+ `请求转发通过HttpServletRequest实现`
+ `响应重定向通过HttpServletResponse实现`

+ 请求转发生活举例: 张三找李四借钱,李四没有,李四找王五,让王五借给张三
+ 响应重定向生活举例:张三找李四借钱,李四没有,李四让张三去找王五,张三自己再去找王五借钱

### 9.2 请求转发

> 请求转发运行逻辑图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e17c6b992edd5f0e5541992ee932287.png)

> 请求转发特点(背诵)

+ 请求转发`通过HttpServletRequest对象的getRequestDispatcher(资源)方法来获取请求转发器`，然后通过该请求转发器的`forward方法实现转发的`
+ 请求转发是`服务器内部的行为,对客户端是屏蔽`的
+ `客户端只发送了一次请求,客户端地址栏不变`
+ `服务端只产生了一对请求和响应对象`,这一对请求和响应对象会继续传递给下一个资源
+ 因为全程只有一个HttpServletRequset对象,所以请求参数可以传递,请求域中的数据也可以传递
+ 请求转发可以转发给其他Servlet动态资源,也可以转发给一些静态资源以实现页面跳转，不能转发到本项目以外的外部资源
+ 请求转发可以转发给WEB-INF下受保护的资源

> 请求转发测试代码

- ServletA

```java
@WebServlet("/servletA")  
public class ServletA extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        //  获取请求转发器  
        //  转发给servlet  ok  
        RequestDispatcher  requestDispatcher = req.getRequestDispatcher("servletB");  
        //  转发给一个视图资源 ok  
        //RequestDispatcher requestDispatcher = req.getRequestDispatcher("welcome.html");  
        //  转发给WEB-INF下的资源  ok  
        //RequestDispatcher requestDispatcher = req.getRequestDispatcher("WEB-INF/views/view1.html");  
        //  转发给外部资源   no  
        //RequestDispatcher requestDispatcher = req.getRequestDispatcher("http://www.atguigu.com");  
        //  获取请求参数  
        String username = req.getParameter("username");  
        System.out.println(username);  
        //  向请求域中添加数据  
        req.setAttribute("reqKey","requestMessage");  
        //  做出转发动作  
        requestDispatcher.forward(req,resp);  
    }  
}
```

- ServletB

```java
@WebServlet("/servletB")  
public class ServletB extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        // 获取请求参数  
        String username = req.getParameter("username");  
        System.out.println(username);  
        // 获取请求域中的数据  
        String reqMessage = (String)req.getAttribute("reqKey");  
        System.out.println(reqMessage);  
        // 做出响应  
        resp.getWriter().write("servletB response");          
    }  
}
```

- 打开浏览器，测试
### 9.3 响应重定向

> 响应重定向运行逻辑图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/963fb691b98620652db8b0421f219526.png)

> 响应重定向特点(背诵)

+ 响应重定向`通过HttpServletResponse对象的sendRedirect方法实现`
+ 响应重定向是`服务端通过302响应码和路径`,告诉客户端自己去找其他资源,是在服务端提示下的,客户端的行为
+ `客户端至少发送了两次请求,客户端地址栏是要变化的`
+ 服务端产生了多对请求和响应对象,且请求和响应对象不会传递给下一个资源
+ 因为全程产生了多个HttpServletRequset对象,所以请求参数不可以传递,请求域中的数据也不可以传递
+ 重定向可以是其他Servlet动态资源,也可以是一些静态资源以实现页面跳转
+ 重定向不可以到给WEB-INF下受保护的资源
+ 重定向可以到本项目以外的外部资源

> 响应重定向测试代码

+ ServletA

``` java
@WebServlet("/servletA")
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //  获取请求参数
        String username = req.getParameter("username");
        System.out.println(username);
        //  向请求域中添加数据
        req.setAttribute("reqKey","requestMessage");
        //  响应重定向
        // 重定向到servlet动态资源 OK
        resp.sendRedirect("servletB");
        // 重定向到视图静态资源 OK
        //resp.sendRedirect("welcome.html");
        // 重定向到WEB-INF下的资源 NO
        //resp.sendRedirect("WEB-INF/views/view1");
        // 重定向到外部资源
        //resp.sendRedirect("http://www.atguigu.com");
    }
}
```

+ ServletB

``` java
@WebServlet("/servletB")
public class ServletB extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 获取请求参数
        String username = req.getParameter("username");
        System.out.println(username);
        // 获取请求域中的数据
        String reqMessage = (String)req.getAttribute("reqKey");
        System.out.println(reqMessage);
        // 做出响应
        resp.getWriter().write("servletB response");

    }
}
```

+ 打开浏览器,输入以下url测试


## 10 MVC架构模式

>  MVC（Model View Controller）是软件工程中的一种**软件架构模式**，它把软件系统分为   **模型**、**视图**和 **控制器** 三个基本部分。用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。

- 高内聚低耦合：修改一部分代码，对其他模块的影响较低
- 开闭原则：对拓展开发，对修改关闭

+ **M**：Model 模型层,具体功能如下
  1. 存放和数据库对象的实体类以及一些用于存储非数据库表完整相关的VO对象
  2. 存放一些对数据进行逻辑运算操作的的一些业务处理代码

+ **V**：View 视图层,具体功能如下
  1. 存放一些视图文件相关的代码 html css js等
  2. 在前后端分离的项目中,后端已经没有视图文件,该层次已经衍化成独立的前端项目

+ **C**：Controller 控制层,具体功能如下
  1. 接收客户端请求,获得请求数据
   2. 将准备好的数据响应给客户端

> MVC模式下,项目中的常见包

+ M:
  1. `实体类包(pojo /entity /bean)` 专门存放和数据库对应的实体类和一些VO对象
  2. `数据库访问包(dao/mapper)`  专门存放对数据库不同表格CURD方法封装的一些类
  3. `服务包(service)`                       专门存放对数据进行业务逻辑运算的一些类

+ C:
  1. `控制层包(controller)`

+ V:
  1. web目录下的视图资源 html css js img 等
  2. 前端工程化后,在后端项目中已经不存在了

`非前后端分离的MVC`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/46989d809dbe38c77769ee2ab42324be.png)
`前后端分离的MVC`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58a1254c2afd94b8e43fc6511ac6053f.png)
## 11 补充

### 11.1 servlet-api导入问题

- HttpServlet类是属于servlet-api.jar这个jar包的
- 而这个jar包，本身就在Tomcat中的bin文件夹中
- 所以使用的时候，在项目导入Tomcat即可，且是provided的（表示编码的时候需要，运行的时候不需要--Tomcat本身提供了）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/afdac2af65f84523acbeda962c9ef0d8.png)
- 如果我们，在WEB-INF文件夹下创建了lib目录，并导入了这个jar包，在运行文件的时候，就会把这个jar包导出，增加了项目的体积![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75806cb211842df234a05da69f263595.png)

### 11.2 Content-Type响应头的问题

[MIME类型](#^cdb658)
- Content-Type本质是响应头之一
- 返回给客户端一个MIME类型，告诉客户端应该按MIME类型进行处理响应数据
- 在tomcat文件夹中，找到conf文件夹中的web.xml文件，即可找到MIME-TYPE与字符串对应的类型
- MIME类型（处理静态资源）
	- text/html：客户端按照html格式进行解析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/348855b228fb794012871f19aba3360e.png)
	- image/png：客户端按照png格式进行解析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e13a6f4eefdf08d4732366b806bf4a16.png)
- 像servlet这些，就不是静态资源，如果在代码中没有设置Content-Type，客户端不知道当作将响应当作什么来处理
	- 设置Content-Type，让客户端当成html来解析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0f45b13370a458d2ce5b4d606c319cb6.png)