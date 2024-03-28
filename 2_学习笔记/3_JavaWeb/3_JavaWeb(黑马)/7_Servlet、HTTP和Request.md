## 目录

- [1 Servlet](#1%20Servlet)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 快速入门](#1.2%20%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)
	- [1.3 执行原理](#1.3%20%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86)
	- [1.4 Servlet中的生命周期方法](#1.4%20Servlet%E4%B8%AD%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%96%B9%E6%B3%95)
	- [1.5 Servlet3.0](#1.5%20Servlet3.0)
	- [1.6 Servlet的体系结构](#1.6%20Servlet%E7%9A%84%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84)
	- [1.7 Servlet访问路径](#1.7%20Servlet%E8%AE%BF%E9%97%AE%E8%B7%AF%E5%BE%84)
- [2 HTTP](#2%20HTTP)
	- [2.1 概念](#2.1%20%E6%A6%82%E5%BF%B5)
	- [2.2 请求消息数据格式](#2.2%20%E8%AF%B7%E6%B1%82%E6%B6%88%E6%81%AF%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F)
	- [2.3 响应消息数据格式](#2.3%20%E5%93%8D%E5%BA%94%E6%B6%88%E6%81%AF%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F)
- [3 Request](#3%20Request)
	- [3.1 request对象和response对象的原理](#3.1%20request%E5%AF%B9%E8%B1%A1%E5%92%8Cresponse%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%8E%9F%E7%90%86)
	- [3.2 request对象继承体系结构](#3.2%20request%E5%AF%B9%E8%B1%A1%E7%BB%A7%E6%89%BF%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84)
	- [3.3 request功能](#3.3%20request%E5%8A%9F%E8%83%BD)
		- [3.3.1 获取请求消息数据](#3.3.1%20%E8%8E%B7%E5%8F%96%E8%AF%B7%E6%B1%82%E6%B6%88%E6%81%AF%E6%95%B0%E6%8D%AE)
		- [3.3.2 其他功能(重要！！！)](#3.3.2%20%E5%85%B6%E4%BB%96%E5%8A%9F%E8%83%BD(%E9%87%8D%E8%A6%81%EF%BC%81%EF%BC%81%EF%BC%81))
- [4 案例：用户登录](#4%20%E6%A1%88%E4%BE%8B%EF%BC%9A%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95)
- [5 BeanUtils工具类](#5%20BeanUtils%E5%B7%A5%E5%85%B7%E7%B1%BB)

## 1 Servlet

### 1.1 概念

- server applet
- 运行在服务器端的小程序
	* Servlet就是一个接口，定义了Java类`被浏览器访问到(tomcat识别)的规则`。
	* 将来我们自定义一个类，`实现Servlet接口，复写方法`

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Servlet.bmp)

### 1.2 快速入门

1. 创建JavaEE项目
2. 定义一个类，实现Servlet接口
	* public class ServletDemo1 implements Servlet
3. 实现接口中的抽象方法
4. 配置Servlet
	 在web.xml中配置：
```xml
<!--    配置Servlet-->  
<servlet>  
	<servlet-name>demo1</servlet-name>  
	<servlet-class>com.dakkk.web.servlet.ServletDemo1</servlet-class>  
</servlet>  

<servlet-mapping>        
	<servlet-name>demo1</servlet-name>  
	<url-pattern>/demo1</url-pattern>  
</servlet-mapping>
```


### 1.3 执行原理

1. 当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径
2. 查找web.xml文件，是否有对应的`<url-pattern>`标签体内容。
3. 如果有，则在找到对应的`<servlet-class>`全类名
4. tomcat会将字节码文件加载进内存，并且创建其对象
5. 调用其方法
6. 图解执行原理![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Servlet%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86.bmp)

- `结论`：
	- `我们写的Servlet类的实现类，不需要创建对象，也不需要调用方法`
	- `只需要在Servlet类的抽象方法中，编写我们需要的逻辑代码即可`
	- `对象的创建，方法的调用都是由Web服务器执行的`
	- `所以，Servlet的执行，需要依赖于外部的Web容器`

### 1.4 Servlet中的生命周期方法

1. `被创建`：`执行init方法，只执行一次`
	* Servlet什么时候被创建？
		* 默认情况下，第一次被访问时，Servlet被创建
		* 可以配置执行Servlet的创建时机。
			* 在`<servlet>`标签下配置
				1. 第一次被访问时，创建
					* `<load-on-startup>`的值为负数，默认情况
				2. 在服务器启动时，创建
					* `<load-on-startup>`的值为0或正整数
					* 有些Servlet加载需要依赖于其他Servlet，这个时候，其他的Servlet可以使用这个方法![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074529675.png)



* Servlet的init方法，只执行一次，说明一个Servlet在内存中只存在一个对象，`Servlet是单例的`
	* 多个用户同时访问时，可能存在线程安全问题。
	* 解决：尽量不要在Servlet中定义成员变量。即使定义了成员变量，也不要对修改值

2. `提供服务`：`执行service方法，执行多次`
	* 每次访问Servlet时，Service方法都会被调用一次。
3. `被销毁`：`执行destroy方法，只执行一次`
	* Servlet被销毁时执行。服务器关闭时，Servlet被销毁
	* 只有服务器正常关闭时，才会执行destroy方法。
	* `destroy方法在Servlet被销毁之前执行`，一般用于释放资源


### 1.5 Servlet3.0

* 好处：
	* `支持注解配置`。可以不需要web.xml了。
	* [[0_基础加强#3.4 自定义注解||注解回顾]]

* 步骤：
	1. 创建JavaEE项目，选择Servlet的版本3.0以上，可以不创建web.xml
	2. 定义一个类，实现Servlet接口
	3. 复写方法
	4. 在类上使用@WebServlet注解，进行配置
```java
@WebServlet("Servlet资源路径")


@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebServlet {
	String name() default "";//相当于<Servlet-name>

	String[] value() default {};//代表urlPatterns()属性配置

	String[] urlPatterns() default {};//相当于<url-pattern>

	int loadOnStartup() default -1;//相当于<load-on-startup>

	WebInitParam[] initParams() default {};

	boolean asyncSupported() default false;

	String smallIcon() default "";

	String largeIcon() default "";

	String description() default "";

	String displayName() default "";
}
```

### 1.6 Servlet的体系结构

Servlet -- 接口
	| 实现
GenericServlet -- 抽象类，实现Servlet
	| 继承
HttpServlet  -- 抽象类，继承GenericServlet

* `GenericServlet`：将Servlet接口中其他的方法做了默认空实现，`只将service()方法作为抽象`
	* 将来定义Servlet类时，可以继承GenericServlet，实现service()方法即可

* `HttpServlet`：对http协议的一种封装，简化操作
	1. 定义类继承HttpServlet
	2. 复写doGet/doPost方法

- `HttpServlet的原理`
- `就是把service方法中判断请求方式给封装称为doGet/doPost方法`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074548068.png)
### 1.7 Servlet访问路径

- `urlpartten:Servlet访问路径`
1. 一个Servlet可以定义多个访问路径 ： `@WebServlet({"/d4","/dd4","/ddd4"})`
2. 路径定义规则：
	1. /xxx：路径匹配
	2. /xxx/xxx:多层路径，目录结构
	3. `*.do：扩展名匹配`

## 2 HTTP

### 2.1 概念

- Hyper Text Transfer Protocol 超文本传输协议
	* 传输协议：定义了，客户端和服务器端通信时，`发送数据的格式`
	* 特点：
		1. 基于`TCP/IP的高级协议`
		2. 默认端口号:80
		3. 基于`请求/响应模型`的:`一次请求对应一次响应`
		4. 无状态的：每次请求之间相互独立，不能交互数据

	* 历史版本：
		* 1.0：每一次请求响应都会建立新的连接
		* 1.1：复用连接

- `HTTP中规定了客户端的请求消息格式 和 服务端的响应消息的格式`

### 2.2 请求消息数据格式

1. `请求行`
	请求方式        请求url         请求协议/版本
	GET             /login.html	    HTTP/1.1

	* 请求方式：
		* HTTP协议有7中请求方式，常用的有2种
			* `GET`：
				1. `请求参数在请求行中，在url后`。
				2. 请求的url`长度有限制的`
				3. 不太安全
			* `POST`：
				1. `请求参数在请求体中`
				2. 请求的url`长度没有限制的`
				3. 相对安全
2. `请求头`：<font color="#d83931">客户端浏览器告诉服务器一些信息</font>
	请求头名称: 请求头值
	* 常见的请求头：
		1. User-Agent：浏览器告诉服务器，我访问你使用的浏览器版本信息
			* 可以在服务器端获取该头的信息，`解决浏览器的兼容性问题`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074606519.png)
		2. Referer：http://localhost/login.html
			* `告诉服务器，我(当前请求)从哪里来？`
				* 作用：
					- 防盗链+统计工作![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074615062.png)

1. `请求空行`
	空行，就是用于`分割POST请求的请求头和请求体的`。
4. `请求体`(正文)：
	* 封装<font color="#d83931">POST请求消息的请求参数的</font>

- `字符串格式`
```http
//请求行
POST /login.html	HTTP/1.1

//请求头，注意之间没有分隔的
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://localhost/login.html
Connection: keep-alive
Upgrade-Insecure-Requests: 1
//请求空行，分隔 请求头 和 请求体
//请求体，注意只有post方法才有
username=zhangsan	
```

### 2.3 响应消息数据格式

- `在下一节课中有介绍！`

## 3 Request

### 3.1 request对象和response对象的原理

1. `request和response对象是由服务器创建的`。我们来使用它们
2. request对象是来`获取请求消息`，response对象是来`设置响应消息`
3. 原理![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074623340.png)

### 3.2 request对象继承体系结构

ServletRequest		--	接口
	|	继承
HttpServletRequest	-- 接口
	|	实现
org.apache.catalina.connector.RequestFacade 类(tomcat)

### 3.3 request功能

#### 3.3.1 获取请求消息数据

1. `获取请求行数据`
	* 假如请求行数据如下：
	* `GET /day14/demo1?name=zhangsan HTTP/1.1`
	* 方法：
		1. 获取请求方式 ：GET/POST
			* `String getMethod()  `
		2. <font color="#d83931">获取虚拟目录</font>：/day14
			* `String getContextPath()`
		3. 获取Servlet路径: /demo1
			* `String getServletPath()`
		4. 获取get方式请求参数：name=zhangsan
			* `String getQueryString()`
		5. <font color="#d83931">获取请求URI</font>：/day14/demo1
			* `String getRequestURI()`:		/day14/demo1
			* `StringBuffer getRequestURL()`  :http://localhost/day14/demo1

			* URL:统一资源定位符 ： http://localhost/day14/demo1	中华人民共和国
			* URI：统一资源标识符 : /day14/demo1					共和国
		
		6. 获取协议及版本：HTTP/1.1
			* `String getProtocol()` :继承于ServletRequest的方法

		7. 获取客户机的IP地址：
			* `String getRemoteAddr()` :继承于ServletRequest的方法

2. `获取请求头数据`
	* 方法：
		* `String getHeader(String name)`:<font color="#d83931">通过请求头的名称获取请求头的值</font>
		* `Enumeration<String> getHeaderNames()`:获取所有的请求头名称

3. `获取请求体数据`:
	* 请求体：<font color="#d83931">只有POST请求方式，才有请求体</font>，在请求体中封装了POST请求的请求参数
	* 步骤：
		1. `获取流对象`
			*  `BufferedReader getReader()`：获取字符输入流，只能操作字符数据
			*  `ServletInputStream getInputStream()`：获取字节输入流，可以操作所有类型数据
				* 在文件上传知识点后讲解
		2. `再从流对象中拿数据`

#### 3.3.2 其他功能(重要！！！)

1. `获取请求参数通用方式`：不论get还是post请求方式都可以使用下列方法来获取请求参数
	1. `String getParameter(String name)`:<font color="#d83931">根据参数名称获取参数值</font>    username=zs&password=123
	2. `String[] getParameterValues(String name)`:根据参数名称获取参数值的数组  hobby=xx&hobby=game
	3. `Enumeration<String> getParameterNames()`:获取所有请求的参数名称
	4. `Map<String,String[]> getParameterMap()`:<font color="#d83931">获取所有参数的map集合</font>

	* 中文乱码问题：
		* get方式：tomcat 8 已经将get方式乱码问题解决了
		* post方式：会乱码
			* 解决：在获取参数前，设置request的编码`request.setCharacterEncoding("utf-8");`
			* 因为国内默认浏览器输入的是gbk字符，所以要修改过来

2. `请求转发`：一种在服务器内部的资源跳转方式
	1. 步骤：
		1. 通过request对象获取`请求转发器对象`：`RequestDispatcher getRequestDispatcher(String path)`
		2. 使用RequestDispatcher对象来进行转发：`forward(ServletRequest request, ServletResponse response)` 

	2. `特点`： #面试高频
		1. <font color="#d83931">浏览器地址栏路径不发生变化</font>
		2. <font color="#d83931">只能转发到当前服务器内部资源中</font>
		3. <font color="#d83931">转发是一次请求，即使用的是同一个请求</font>

3. `共享数据`：
	* 域对象：一个有作用范围的对象，可以在范围内共享数据
	* request域：代表一次请求的范围，<font color="#d83931">一般用于请求转发的多个资源中共享数据</font>
	* 方法：
		1. `void setAttribute(String name,Object obj)`:存储数据
		2. `Object getAttritude(String name)`:通过键获取值
		3. `void removeAttribute(String name)`:通过键移除键值对

4. `获取ServletContext`：
	* ServletContext getServletContext()

## 4 案例：用户登录

* 用户登录案例需求：
	1. 编写login.html登录页面
		username & password 两个输入框
	2. 使用Druid数据库连接池技术,操作mysql，test数据库中user表
	3. 使用UserDao和javaBean技术
	4. 登录成功跳转到SuccessServlet展示：登录成功！用户名,欢迎您
	5. 登录失败跳转到FailServlet展示：登录失败，用户名或密码错误


- 表单中填写的`action= 虚拟目录 + Servlet访问路径`

- login.html代码
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
    <form action="/C14test/LoginServlet" method="post">  
        用户名:<input type="text" name="name"> <br>  
        密码:<input type="password" name="password"><br>  
  
        <input type="submit" value="登录">  
  
    </form>  
</body>  
</html>
```

- JavaBean代码，省略

- UserDao代码
```java
public class UserDao {  
  
    private static QueryRunner qr = new QueryRunner();  
  
    public static User getUser(User loginUser){  
        String name = loginUser.getName();  
        String password = loginUser.getPassword();  
        Connection con = null;  
        User user = null;  
        try {  
            con = JDBCUtils.getConnection();  
            con.setAutoCommit(false);  
            String sql = "SELECT name,password FROM user WHERE name=? AND password=?";  
            user = qr.query(con, sql, new BeanHandler<>(User.class), name, password);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        } finally {  
            DbUtils.closeQuietly(con);  
        }  
        return user;  
    }  
}
```

- utils代码
```java
public class JDBCUtils {  
    private static DataSource ds;  
    static{  
        try {  
//            InputStream is = ClassLoader.getSystemResourceAsStream("JDBC.properties");  
            InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("JDBC.properties");  
            Properties ps = new Properties();  
            ps.load(is);  
            ds = DruidDataSourceFactory.createDataSource(ps);  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
  
    public static DataSource getDataSource(){  
        return ds;  
    }  
    public static Connection getConnection() throws SQLException {  
        return ds.getConnection();  
    }  
}
```


- Servlet代码
```java
@WebServlet("/LoginServlet")  
public class LoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doPost(request,response);  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        request.setCharacterEncoding("UTF-8");  
        String name = request.getParameter("name");  
        String password = request.getParameter("password");  
        User loginUser = new User(name,password);  
        User verifyUser = UserDao.getUser(loginUser);  
        if (verifyUser!=null){  
            request.setAttribute("name",name);  
            request.getRequestDispatcher("/SuccessServlet").forward(request,response);  
        }else {  
            request.getRequestDispatcher("FailServlet").forward(request,response);  
        }  
    }  
}

@WebServlet("/SuccessServlet")  
public class SuccessServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        Object name = request.getAttribute("name");  
        response.setContentType("text/html; charset=utf-8");  
        response.getWriter().write("登录成功！"+name+",欢迎您");  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doGet(request,response);  
    }  
}

@WebServlet("/FailServlet")  
public class FailServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        response.setContentType("text/html; charset=utf-8");  
        response.getWriter().write("登录失败，用户名或密码错误");  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doGet(request,response);  
    }  
}
```




- `案例文件结构`
- ![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074640919.png)



## 5 BeanUtils工具类

- 简化数据封装，主要用于封装JavaBean的

-  JavaBean：标准的Java类
	1. 要求：
		1. 类必须被public修饰
		2. 必须提供空参的构造器
		3. 成员变量必须使用private修饰
		4. 提供公共setter和getter方法
	2. 功能：封装数据

-  概念：
	成员变量：
	属性：<font color="#d83931">setter和getter方法截取后的产物</font>
		例如：getUsername() --> Username--> username


- 方法：
	1. setProperty()
	2. getProperty()
	3. `populate(Object obj , Map map):将map集合的键值对信息，封装到对应的JavaBean对象中`