---
文章标题: "[[9_Cookie&Session]]" 
文章作者: Dakkk
文章概要: |
  深入讲解Java Web会话技术Cookie和Session的概念、原理和使用方法，包含验证码登录案例的完整实现
tags:
- "Cookie"
- "Session"
- "会话技术"
- "JSP"
- "HttpSession"
- "验证码"
- "Servlet"
- "Java Web"
相关文章:
- "[[8_会话_过滤器_监听器]]"
- "[[10_※HTTP状态]]"
- "[[4_JavaEE的13项规范]]"
- "[[1_※认识HTTP]]"
- "[[1_解决国内网页无法加载reCaptcha的方法]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/9_Cookie&Session.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 会话技术

1. 会话：`一次会话中包含多次请求和响应`。
	* 一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止
2. 功能：在一次会话的范围内的多次请求间，`共享数据`
	- eg：在商城网站，每次请求和响应都添加一个商品到购物车，最后查看购物车的时候，所有添加的商品都在购物车内
1. 方式：
	1. `客户端会话技术：Cookie`
	2. `服务器端会话技术：Session`

## 2 Cookie

### 2.1 概念

- 客户端会话技术，将数据保存到客户端

### 2.2 快速入门

1. 创建Cookie对象，绑定数据
	* new Cookie(String name, String value) 
2. 发送Cookie对象
	* response.addCookie(Cookie cookie) 
3. 获取Cookie，拿到数据
	* Cookie[]  request.getCookies()  

### 2.3 实现原理

* 基于响应头set-cookie和请求头cookie实现![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6cfbb28006295e593ee81fb1763d8e16.png)


* 案例Demo1中设置Cookie（响应头中发送给浏览器），Demo2中获取Cookie（在请求头中发送Cookie）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d9290a05ac2f0b92582d6bf2a00a25d7.png)



### 2.4 Cookie的细节

1. `一次可不可以发送多个cookie?`
	* `可以`
	* 可以创建多个Cookie对象，使用response<font color="#00b050">调用多次addCookie方法发送cookie即可</font>。
2. cookie在浏览器中保存多长时间？
	1. <font color="#00b050">默认情况下，当浏览器关闭后，Cookie数据被销毁</font>
	2. 持久化存储：
		* `setMaxAge(int seconds)`
			1. 正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效
			2. 负数：默认值
			3. <font color="#00b050">零：删除cookie信息</font>
3. cookie能不能存中文？
	* 在tomcat 8 之前 cookie中不能直接存储中文数据。
		* 需要将中文数据转码---一般采用URL编码(%E3)
	* <font color="#00b050">在tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析</font>
4. `cookie共享问题？`
	1. 假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享？
		* `默认情况下cookie不能共享`

		* <font color="#00b050">setPath(String path):设置cookie的获取范围。默认情况下，设置当前的虚拟目录</font>
			* 如果要共享，则可以将path设置为"/"

	2. 不同的tomcat服务器间cookie共享问题？
		* `setDomain(String path)`:如果设置一级域名相同，那么多个服务器之间cookie可以共享
		* `setDomain(".baidu.com"),那么tieba.baidu.com和news.baidu.com中cookie可以共享`

### 2.5 Cookie的特点和作用

1. `cookie存储数据在客户端浏览器`
2. 浏览器对于单个cookie 的大小有限制(4kb) 以及 对同一个域名下的总cookie数量也有限制(20个)

* 作用：
	1. `cookie一般用于存出少量的不太敏感的数据`
	2. `在不登录的情况下，完成服务器对客户端的身份识别`
		- 即不需要登录任何一个网站，但是在本地浏览器对该网站进行了相关设置，此时服务器会在本地留一个cookie，用于保存此设置

### 2.6 案例：记住上一次访问时间

- 需求：
	1. 访问一个Servlet，如果是第一次访问，则提示：您好，欢迎您首次访问。
	2. 如果不是第一次访问，则提示：欢迎回来，您上次访问时间为:显示时间字符串

- 分析：
	1. 可以采用Cookie来完成
	2. 在服务器中的Servlet判断是否有一个名为lastTime的cookie
		1. 有：不是第一次访问
			1. 响应数据：欢迎回来，您上次访问时间为:2018年6月10日11:50:20
			2. 写回Cookie：lastTime=2018年6月10日11:50:01
		2. 没有：是第一次访问
			1. 响应数据：您好，欢迎您首次访问
			2. 写回Cookie：lastTime=2018年6月10日11:50:01

- 代码实现：
```java
@Override  
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
    //设置响应编码  
    resp.setContentType("text/html;charset=utf-8");  
    //1.存在Cookie  
    Cookie[] cookies = req.getCookies();  
    if (cookies.length>1){  
        for (Cookie c : cookies) {  
            String name = c.getName();  
            System.out.println(name);  
            if ("lastTime".equals(name)){  
                //获取时间  
                LocalDateTime date = LocalDateTime.now();  
                DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");  
                String strTime = date.format(dtf);  
                //因为有空格，所以进行url编码  
                strTime= URLEncoder.encode(strTime,"utf-8");  
                //发送消息  
                resp.getWriter().write("<h1>欢迎回来，你上次访问时间为："+ URLDecoder.decode(c.getValue(),"utf-8") +"</h1>");  
                //重新修改cookie的值  
                c.setValue(strTime);  
                //设置cookie响应回去！！！  
                resp.addCookie(c);  
                break;            }  
        }  
    }else if (cookies.length==1){ //2.不存在Cookie  
        //获取时间  
        LocalDateTime date = LocalDateTime.now();  
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");  
        String strTime = date.format(dtf);  
        //因为有空格，所以进行url编码  
        strTime= URLEncoder.encode(strTime,"utf-8");  
        //发送消息  
        resp.getWriter().write("<h1>您好，欢迎您首次访问</h1>");  
        Cookie c = new Cookie("lastTime",strTime);  
        resp.addCookie(c);  
    }  
}
```

## 3 JSP：入门

### 3.1 概念

* Java Server Pages： java服务器端页面
	* 可以理解为：`一个特殊的页面，其中既可以指定定义html标签，又可以定义java代码`
	* `用于简化书写！！`

### 3.2 原理

* JSP本质上就是一个Servlet![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d138e806b447c5735be3ec66c5455948.png)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7a62d947056d81d0dd8b2490ada93f23.png)



### 3.3 JSP的脚本

- JSP定义Java代码的方式

1. `<%  代码 %>`：<font color="#00b050">定义的java代码，在service方法中</font>。service方法中可以定义什么，该脚本中就可以定义什么。
2. `<%! 代码 %>`：定义的java代码，在jsp转换后的java类的成员位置，即<font color="#00b050">在index_jsp.java类中的成员属性</font>；
3. `<%= 代码 %>`：`定义的java代码，会输出到页面上`。输出语句中可以定义什么，该脚本中就可以定义什么。`注意`要输出变量的值，此时变量指的是service中的局部变量

### 3.4 JSp的内置对象

* 在jsp页面中不需要获取和创建，可以直接使用的对象
* `jsp一共有9个内置对象`
* 今天学习3个：
	* `request`
	* `response`
	* `out`：<font color="#00b050">字符输出流对象</font>。可以将数据输出到页面上。<font color="#00b050">和response.getWriter()类似</font>
		* response.getWriter()和out.write()的区别：
			* 在tomcat服务器真正给客户端做出响应之前，<font color="#00b050">会先找response缓冲区数据，再找out缓冲区数据</font>。
			* `response.getWriter()数据输出永远在out.write()之前`

### 3.5 案例：改造2.6案例

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<html>  
<head>  
    <title>访问时间页面</title>  
</head>  
<body>  
    <%  
        Cookie[] cookies = request.getCookies();  
        //1.Cookie存在  
            if (cookies.length>1&&cookies!=null){  
                for (Cookie cookie : cookies) {  
                    String name = cookie.getName();  
                    if ("lastTime".equals(name)){  
                        //获取当前时间  
                        LocalDateTime date = LocalDateTime.now();  
                        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");  
                        String format = date.format(dtf);  
                        //获取cookie记录的时间  
                        String value = cookie.getValue();  
                        out.print("<h1>欢迎再次访问，上次访问时间为："+ URLDecoder.decode(value,"utf-8")+"</h1>");  
                        //更新cookie的时间  
                        cookie.setValue(URLEncoder.encode(format,"utf-8"));  
                        response.addCookie(cookie);  
                    }  
                }  
            } else if (cookies.length<=1&&cookies!=null) {//2.Cookie不存在  
                //获取当前时间  
                LocalDateTime date = LocalDateTime.now();  
                DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");  
                String format = date.format(dtf);  
                //添加cookie并更新cookie的时间  
                Cookie c = new Cookie("lastTime",URLEncoder.encode(format,"utf-8"));  
                response.addCookie(c);  
                out.print("<h1>您好，欢迎首次访问！！</h1>");  
            }  
    %>  
</body>  
</html>
```


## 4 Session：主菜

### 4.1 概念

- 服务器端会话技术，在`一次会话`的多次请求间共享数据，`将数据保存在服务器端的对象中`。`HttpSession`

### 4.2 快速入门

1. 获取HttpSession对象：
	HttpSession session = request.getSession();
2. 使用HttpSession对象：
	Object getAttribute(String name)  
	void setAttribute(String name, Object value)
	void removeAttribute(String name)  

### 4.3 原理

- 服务器如何确保一次会话范围内，多次获取的Session对象是同一个？？？![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bba0af38cdea99abb9db1c5ac90c5e17.png)


* `Session的实现是依赖于Cookie的`

### 4.4 细节

1. 当客户端关闭后，服务器不关闭，两次获取session是否为同一个？
	* `默认情况下。不是。`
	* 如果需要相同，则可以创建Cookie,键为JSESSIONID，设置最大存活时间，<font color="#00b050">让cookie持久化保存</font>。
```java
Cookie c = new Cookie("JSESSIONID",session.getId());
c.setMaxAge(60*60); //单位为s
response.addCookie(c);
```

2. 客户端不关闭，服务器关闭后，两次获取的session是同一个吗？
	* `不是同一个`，<font color="#00b050">但是要确保数据不丢失。tomcat自动完成以下工作</font>
		* session的钝化：
			* `在服务器正常关闭之前，将session对象系列化到硬盘上`
		* session的活化：
			* `在服务器启动后，将session文件转化为内存中的session对象即可`
		
3. session什么时候被销毁？
	1. `服务器关闭`
	2. `session对象调用invalidate() `
	3. `session默认失效时间 30分钟`，
	4. 在tomcat是默认配置的，路径为tomcat所在文件夹，然后conf文件夹，找到web.xml文件修改即可![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/64aafdf6d60cb43cce7cccab0ed8796a.png)



### 4.5 特点

 1. session用于存储一次会话的多次请求的数据，存在服务器端
 2. session可以存储任意类型，任意大小的数据

* `session与Cookie的区别`：
	1. session存储数据在服务器端，Cookie在客户端
	2. session没有数据大小限制，Cookie有
	3. session数据安全，Cookie相对于不安全

## 5 案例：验证码

- 案例需求：
	1. 访问带有验证码的登录页面login.jsp
	2. 用户输入用户名，密码以及验证码。
		* 如果用户名和密码输入有误，跳转登录页面，提示:用户名或密码错误
		* 如果验证码输入有误，跳转登录页面，提示：验证码错误
		* 如果全部输入正确，则跳转到主页success.jsp，显示：用户名,欢迎您

- 分析：
	1. 编写一个带有验证码的`登录页面login.jsp`
		- 用户名 input标签
		- 密码 input标签
		- input标签
		- 验证码图片 ---> src= `验证码servlet`
		- 提交验证 input标签
	2. 连接到一个`判断Servlet`
		- 通过request获取各个标签中的数据
		- 先判断验证码，因为判断用户名和密码需要访问数据库，这样的话，耗时；
			- 判断验证码的情况，需要`验证码servlet`共享数据，服务器端共享数据建议使用session进行共享；
			- 从session中获取验证码数据
				- 验证码有误，跳转到登录页面，并提示：验证码错误
					- 我们可以使用request转发，进行跳转
					- 并且设置提示信息
				- 验证码正确，
					- 存储用户名和密码为一个UserDao，进行判断
					- 如果有误
						- 继续使用request转发，进行跳转
						- 并且设置提示信息
					- 如果正确
						- 跳转到主页success.jsp
						- 显示：用户名，欢迎你！

- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b55042006e834614dfdc5c5a6e023cc8.png)



- `细节`:
	1. 验证码错误或 账户名和密码错误，会一致在页面展示null，可以使用三元运算符
	2. 登录成功后，后退页面，验证码仍存在且不变，后退后可重复登录
		- 获取验证码后删除，在第一个判断中添加验证码是否为空的判断即可

- 代码

- 登录页面和成功页面
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<html>  
<head>  
    <title>登录页面</title>  
    <meta charset="utf-8">  
    <meta http-equiv="X-UA-Compatible" content="IE=edge">  
    <meta name="viewport" content="width=device-width, initial-scale=1">  
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->  
    <title>Bootstrap HelloWorld</title>  
  
    <!-- Bootstrap -->  
    <link href="css/bootstrap.min.css" rel="stylesheet">  
    <!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->  
    <script src="js/jquery-3.2.1.min.js"></script>  
    <!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->  
    <script src="js/bootstrap.min.js"></script>  
    <style>        #username, #password, #checkCode {  
            width: 200px;  
        }  
  
        #adiv {  
            margin-top: 10px;  
            margin-bottom: 10px;  
        }  
        .error{  
            color: red;  
        }  
    </style>  
    <script>        window.onload = function () {  
            var img = document.getElementById("imgChange");  
            var a = document.getElementById("aChange");  
  
            function change() {  
                var time = new Date().getTime();  
                img.src = "/C16test/ImageServlet?"+time;  
            }  
            img.onclick = change;  
            a.onclick = change;  
        }  
    </script>  
</head>  
<body>  
<form action="/C16test/CheckServlet" method="post">  
    <div class="form-group">  
        <label for="username">用户名</label>  
        <input type="text" class="form-control" id="username" name="name" placeholder="请输入用户名">  
    </div>  
    <div class="form-group">  
        <label for="password">密码</label>  
        <input type="password" class="form-control" id="password" name="password" placeholder="请输入密码">  
    </div>  
    <div class="form-group">  
        <label for="checkCode">二维码</label>  
        <input type="text" class="form-control" id="checkCode" name="checkCode" placeholder="请输入验证码">  
    </div>  
    <div>        <img src="/C16test/ImageServlet" id="imgChange">  
    </div>  
    <div id="adiv">  
        <a href="javascript:void(0);" id="aChange">看不清楚，点击换一张</a>  
    </div>  
    <div class="error">  
<%--        运用三元表达式，消除null的出现--%>  
        <%=request.getAttribute("c1")==null?"":request.getAttribute("c1")%>  
    </div>  
    <div class="error">  
        <%=request.getAttribute("c2")==null?"":request.getAttribute("c2")%>  
    </div>  
    <button type="submit" class="btn btn-default">Submit</button>  
</form>  
</body>  
</html>


<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<html>  
<head>  
    <title>Title</title>  
</head>  
<body>  
  
    <h1><%=request.getSession().getAttribute("success")%>,欢迎您</h1>  
  
</body>  
</html>
```

- 登录检查的servlet，判断servlet
```java
@WebServlet("/CheckServlet")  
public class CheckServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        //告诉浏览器以utf-8的方式解码  
        resp.setContentType("test/html;charset=utf-8");  
        //告诉服务器将浏览器的所有请求按utf-8方式转码  
        req.setCharacterEncoding("utf-8");  
        //获取表单的验证码  
        String formCheckCode = req.getParameter("checkCode");  
        //获取网页生成的验证码  
        HttpSession session = req.getSession();  
        String checkCode = (String) session.getAttribute("checkCode");  
        //!!删除session中存储的验证码，防止后退时，验证码仍然没变  
        session.removeAttribute("checkCode");  
        //判断验证码是否一致，忽略大小写  
        if (checkCode!=null&&checkCode.equalsIgnoreCase(formCheckCode)){//如果验证码正确  
            //获取数据  
            User loginUser = new User();  
            Map<String, String[]> map = req.getParameterMap();  
            try {  
                BeanUtils.populate(loginUser,map);  
            } catch (IllegalAccessException e) {  
                e.printStackTrace();  
            } catch (InvocationTargetException e) {  
                e.printStackTrace();  
            }  
            User user = UserDao.getLoginUser(loginUser);  
            if (user==null){  
                //提示用户名或密码错误  
                req.setAttribute("c2","用户名或密码错误");  
                //转发到登录页面  
                req.getRequestDispatcher("/login.jsp").forward(req,resp);  
            }else{  
                //跳转到主页  
                session.setAttribute("success",user.getName());  
                resp.sendRedirect(req.getContextPath()+"/success.jsp");  
            }  
        }else{//如果验证码错误  
            req.setAttribute("c1","验证码错误");  
            //转发到登录页面  
            req.getRequestDispatcher("/login.jsp").forward(req,resp);  
        }  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        doGet(req, resp);  
    }  
}
```



- 验证码案例，参考[[8_Response#2.7 案例4：验证码||验证码案例]]
	- 需要注意的是，要写一个StringBuilder获取二维码字符


- 验证用户名和密码，参考[[7_Servlet、HTTP和Request#4、案例：用户登录|用户登录案例]]