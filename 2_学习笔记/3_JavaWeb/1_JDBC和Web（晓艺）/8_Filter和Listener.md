
## 一、 Filter：过滤器

1. 概念：当访问服务器的资源时，过滤器可以将请求拦截下来，进行一些特定的处理，然后可以选择继续传递请求或者不放行。

2. 功能：
	- 对用户请求进行身份认证
	- 敏感字符过滤
	- 修改请求内容或自定义响应内容

## 二、 Listener：监听器

 1. 概念：通过listener 可以监听web 服务器中某一个执行动作，再执行预先定义的动作。

2. 功能：监听servletContext（servlet上下文，对应整个web应用），session，request等对象创建消亡或者修改。

## 三、 Filter、Listener和Servlet的关系

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

总结：

1. 请求到达Servlet会被Filter拦截，如果Filter放行，Servlet处理后回再次经过滤器返回浏览器。

2. 监听器则是监听web应用及请求与响应过程中的一些动作变化。

## 四、 过滤器实战     

1.   定义一个类，实现接口Filter

```java
public class FilterDemo1 implements Filter {

}
```

2.   复写方法

1）. init:在服务器启动后，会创建Filter对象，然后调用init方法。只执行一次

2）. doFilter:每一次请求被拦截资源时，会执行。执行多次

3）. destroy:在服务器关闭后，Filter对象被销毁，只执行一次

3.   配置拦截路径

1）. web.xml

```xml
<filter>  
	<filter-name>demo1</filter-name>  
	<filter-class>com.yiliedu.filter.FilterDemo1</filter-class>  
</filter>  
<filter-mapping>  
	<filter-name>demo1</filter-name>  
	<!-- 拦截路径 -->  
	<url-pattern>/*</url-pattern>  
</filter-mapping>
```

2）注解

` @WebFilter("/*")`

- `拦截匹配规则`：
	1. 拦截所有资源：`/*    `    访问所有资源时，过滤器都会被执行
	2. 拦截目录： `/user/*`    访问/user下的所有资源时，过滤器都会被执行
	3. 后缀名拦截：` *.jsp `    访问所有后缀名为jsp资源时，过滤器都会被执行
	4. 具体资源路径： `/index.jsp`   只有访问index.jsp资源时，过滤器才会被执行

4.   过滤器执行流程
	- 执行过滤器
	- 执行放行后的资源
	- 回来执行过滤器放行代码下边的代码

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

5.   过滤器先后顺序问题：
- web.xml配置： \<filter-mapping>谁定义在上边，谁先执行注解
- 配置：按照类名的字符串比较规则比较，值小的先执行

6.   自定义类实现HttpServletRequestWrapper，在过滤器中将该对象向后传递，增强原request对象的功能

## 五、 常用Listener

1. ServletContextListener:监听ServletContext对象的创建和销毁

2. ServletContext对象：对应一个web应用信息, 里面封装的都是web应用信息

* void contextDestroyed(ServletContextEvent sce) ：ServletContext对象被销毁之前会调用该方法

* void contextInitialized(ServletContextEvent sce) ：ServletContext对象创建后会调用该方法

  * 步骤：

1. 定义一个类，实现ServletContextListener接口

2. 复写方法

3. 配置
	1. web.xml
	2. 注解：
	 * @WebListener

2、HttpSessionListener:监听HttpSession对象的创建和销毁

3、ServletRequestListener :监听ServletRequest对象的创建和销毁