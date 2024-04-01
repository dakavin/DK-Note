
## 1、 乱码问题

> 码问题产生的根本原因是什么

1. `数据的编码和解码使用的不是同一个字符集`
2. `使用了不支持某个语言文字的字符集`
    

> 各个字符集的兼容性

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2905e0762e77a8ffee17fd227315ec0e.png)

- 由上图得知,上述字符集都兼容了ASCII
    
- ASCII中有什么? 英文字母和一些通常使用的符号,所以这些东西无论使用什么字符集都不会乱码
### 1. 1 HTML乱码

> 设置项目文件的字符集要使用一个支持中文的字符集

+ 查看当前文件的字符集![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/391c5161fbc55afa6fe0108127cc92db.png)
+ 查看项目字符集 配置,将Global Encoding 全局字符集,Project Encoding 项目字符集, Properties Files 属性配置文件字符集设置为UTF-8![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a95023f7ec6612310a564bcb27d5d546.png)
> 当前视图文件的字符集通过<meta charset="UTF-8"> 来告知浏览器通过什么字符集来解析当前文件

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    中文
</body>
</html>
```

### 1.2 Tomcat控制台乱码

> 在tomcat10.1.7这个版本中,修改 tomcat/conf/logging.properties中,所有的UTF-8为GBK即可

+ 修改前![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bab5d5f0a2bdac6915ef3de52371c8ef.png)
+ 修改后![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/992abb26749bfc3fe7972ad6fc1e9b92.png)
> sout乱码问题,设置JVM加载.class文件时使用UTF-8字符集

- 设置虚拟机加载.class文件的字符集和编译时使用的字符集一致![||380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/18b826c1d59ae2331b98c16bb977628b.png)
### 1.3 请求乱码

#### 1. GET请求方式乱码分析

+ GET方式提交参数的方式是将参数放到URL后面,如果使用的不是UTF-8,那么会对参数进行URL编码处理
+ HTML中的 `<meta charset='字符集'/>` 影响了GET方式提交参数的URL编码
+ tomcat10.1.7的URI编码默认为 UTF-8
+ 当GET方式提交的参数URL编码和tomcat10.1.7默认的URI编码不一致时,就会出现乱码

> GET请求方式乱码演示

+ 浏览器解析的文档的`<meta charset="GBK" /> `![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3a75786d284abb032f68e6d614abe5f9.png)
+ GET方式提交时,会对数据进行URL编码处理 ,是`将GBK 转码为 "百分号码"`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36e59cd8cb284ffa9133b575686dd651.png)
+ tomcat10.1.7 默认使用UTF-8对URI进行解析,造成前后端使用的字符集不一致,出现乱码![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4f7a7ddd46f9196f7ce928d1f534fa5f.png)
> GET请求方式乱码解决

- 方式1 :设置GET方式提交的编码和Tomcat10.1.7的URI默认解析编码一致即可 (推荐)![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04473298ad8bea31a3fb55a7af4b02b9.png)
- 方式2 : 设置Tomcat10.1.7的URI解析字符集和GET请求发送时所使用URL转码时的字符集一致即可,修改conf/server.xml中 Connecter 添加 URIEncoding="GBK" (不推荐)![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/81bb22be6505bce323acb2a8840e7518.png)

#### 2. POST请求方式乱码分析

+ POST请求将参数放在请求体中进行发送
+ 请求体使用的字符集受到了`<meta charset="字符集"/>` 的影响
+ Tomcat10.1.7 默认使用UTF-8字符集对请求体进行解析
+ 如果请求体的URL转码和Tomcat的请求体解析编码不一致,就容易出现乱码

> POST方式乱码演示

+ POST请求请求体受到了`<meta charset="字符集"/>` 的影响

> POST方式乱码演示

+ POST请求请求体受到了<meta charset="字符集"/> 的影响![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f5c5cfd34d6dd716bb6a04e8fe0e88e1.png)

+ 请求体中,将GBK数据进行 URL编码![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3245d4d68b47911e6d6971165508e350.png)

+ 后端默认使用UTF-8解析请求体,出现字符集不一致,导致乱码![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9f58e0fd9373492ad55cad1f7fd0749d.png)
> POST请求方式乱码解决

+ 方式1 : 请求时,使用UTF-8字符集提交请求体 (推荐)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/db7af758f582c4a99aaa74ab04e230d4.png)

+ 方式2 : 后端在获取参数前,设置解析请求体使用的字符集和请求发送时使用的字符集一致 (不推荐)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8a24f66eb47c43558b3507f4edd7bad7.png)

### 1.4 响应乱码

> 响应乱码分析

+ 在Tomcat10.1.7中,向响应体中放入的数据默认使用了工程编码 UTF-8
+ 浏览器在接收响应信息时,使用了不同的字符集或者是不支持中文的字符集就会出现乱码

> 响应乱码演示

+ 服务端通过response对象向响应体添加数据![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/af734d435069adbe8f35e4fcf3b2bc9a.png)
+ 浏览器接收数据解析乱码![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fd408bd785d53d46641c80fb50869687.png)
> 响应乱码解决

+ 方式1 : 手动设定浏览器对本次响应体解析时使用的字符集(不推荐)
	+ edge和 chrome浏览器没有提供直接的比较方便的入口,不方便

+ 方式2: 后端通过设置响应体的字符集和浏览器解析响应体的默认字符集一致(不推荐)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e00550829caa17d24190d5ececf426c.png)

方式3: 通过设置content-type响应头,告诉浏览器以指定的字符集解析响应体(推荐)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30e0217800467cb83d3328e70176916c.png)
## 2、 路径问题

> 相对路径和绝对路径

+ 相对路径
	+ 相对路径的规则是: 以当前资源所在的路径为出发点去寻找目标资源
	+ 相对路径不以 / 开头
	+ 在file协议下,使用的是磁盘路径
	+ 在http协议下,使用的是url路径
	+ 相对路径中可以使用 ./表示当前资源所在路径,可以省略不写
	+ 相对路径中可以使用../表示当前资源所在路径的上一层路径,需要时要手动添加

+ 绝对路径
	+ 绝对路径的规则是: 使用以一个固定的路径做出出发点去寻找目标资源,和当前资源所在的路径没有关系
	+ 绝对路径要以/ 开头
	+ 绝对路径的写法中,不以当前资源的所在路径为出发点,所以不会出现  ./ 和../
	+ 不同的项目和不同的协议下,绝对路径的基础位置可能不同,要通过测试确定
	+ 绝对路径的好处就是:无论当前资源位置在哪,寻找目标资源路径的写法都一致

+ 应用场景
	1. 前端代码中,href src action 等属性
		- 根据项目根目录，确定绝对路径
		- 根据当前资源所在目录，确定相对路径
	2. 请求转发和重定向中的路径
		- 转发
			- 相对路径：写法和前端一样
			- `绝对路径：forward方法自带项目根路径`
		- 重定向：
			- 相对路径和绝对路径的写法和前端一样

### 2.1 前端路径问题

前端项目结构![|200|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d2948b1ae1d4638075bd937938e3ac4.png)
#### 1 相对路径

> 相对路径情况1:`web/index.html中引入web/static/img/logo.png`

+ 访问index.html的url为   :  http://localhost:8080/web03_war_exploded/index.html
+ 当前资源为                      :  index.html
+ 当前资源的所在路径为  : http://localhost:8080/web03_war_exploded/
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ index.html中定义的了    : `<img src="static/img/logo.png"/>`
	+ 寻找方式就是在当前资源所在路径(http://localhost:8080/web03_war_exploded/)后拼接src属性值(static/img/logo.png),正好是目标资源正常获取的url(http://localhost:8080/web03_war_exploded/static/img/logo.png)

> 相对路径情况2:`web/a/b/c/test.html中引入web/static/img/logo.png`

+ 访问test.html的url为      :  http://localhost:8080/web03_war_exploded/a/b/c/test.html
+ 当前资源为                      :  test.html
+ 当前资源的所在路径为  : http://localhost:8080/web03_war_exploded/a/b/c/
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ test.html中定义的了       : `<img src="../../../static/img/logo.png"/>`
+ 寻找方式就是在当前资源所在路径(http://localhost:8080/web03_war_exploded/a/b/c/)后拼接src属性值(../../../static/img/logo.png),其中 ../可以抵消一层路径,正好是目标资源正常获取的url(http://localhost:8080/web03_war_exploded/static/img/logo.png)

> 相对路径情况3:web/WEB-INF/views/view1.html中引入web/static/img/logo.png

+ view1.html在WEB-INF下,需要通过Servlet请求转发获得

``` java
@WebServlet("/view1Servlet")
public class View1Servlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        RequestDispatcher requestDispatcher = req.getRequestDispatcher("WEB-INF/views/view1.html");
        requestDispatcher.forward(req,resp);
    }
}
```

+ 访问view1.html的url为   :  http://localhost:8080/web03_war_exploded/view1Servlet
+ 当前资源为                      :  view1Servlet
+ 当前资源的所在路径为  : http://localhost:8080/web03_war_exploded/
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ view1.html中定义的了    : `<img src="static/img/logo.png"/>`
+ 寻找方式就是在当前资源所在路径(http://localhost:8080/web03_war_exploded/)后拼接src属性值(static/img/logo.png),正好是目标资源正常获取的url(http://localhost:8080/web03_war_exploded/static/img/logo.png)

#### 2. 绝对路径

> 绝对路径情况1:`web/index.html中引入web/static/img/logo.png`

+ 访问index.html的url为   :  http://localhost:8080/web03_war_exploded/index.html
+ 绝对路径的基准路径为  :  http://localhost:8080
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ index.html中定义的了    : `<img src="/web03_war_exploded/static/img/logo.png"/>`
+ 寻找方式就是在基准路径(http://localhost:8080)后面拼接src属性值(/web03_war_exploded/static/img/logo.png),得到的正是目标资源访问的正确路径

> 绝对路径情况2:`web/a/b/c/test.html中引入web/static/img/logo.png`

+ 访问test.html的url为   :  http://localhost:8080/web03_war_exploded/a/b/c/test.html
+ 绝对路径的基准路径为  :  http://localhost:8080
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ test.html中定义的了    : `<img src="/web03_war_exploded/static/img/logo.png"/>`
+ 寻找方式就是在基准路径(http://localhost:8080)后面拼接src属性值(/web03_war_exploded/static/img/logo.png),得到的正是目标资源访问的正确路径

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <!-- 绝对路径写法 -->
    <img src="/web03_war_exploded/static/img/logo.png">
</body>
</html>
```

> 绝对路径情况3:`web/WEB-INF/views/view1.html中引入web/static/img/logo.png`

+ view1.html在WEB-INF下,需要通过Servlet请求转发获得

``` java
@WebServlet("/view1Servlet")
public class View1Servlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        RequestDispatcher requestDispatcher = req.getRequestDispatcher("WEB-INF/views/view1.html");
        requestDispatcher.forward(req,resp);
    }
}
```

+ 访问view1.html的url为   :  http://localhost:8080/web03_war_exploded/view1Servlet
+ 绝对路径的基准路径为  :  http://localhost:8080
+ 要获取的目标资源url为 :  http://localhost:8080/web03_war_exploded/static/img/logo.png
+ view1.html中定义的了    : `<img src="/web03_war_exploded/static/img/logo.png"/>`
+ 寻找方式就是在基准路径(http://localhost:8080)后面拼接src属性值(/static/img/logo.png),得到的正是目标资源访问的正确路径

#### 3. base标签的使用

> base标签定义页面相对路径公共前缀

+ base 标签定义在head标签中,用于定义相对路径的公共前缀
+ `base 标签定义的公共前缀只在相对路径上有效,绝对路径中无效`
+ 如果相对路径开头有 ./ 或者../修饰,则base标签对该路径同样无效

> index.html 和a/b/c/test.html 以及view1Servlet 中的路径处理

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--定义相对路径的公共前缀,将相对路径转化成了绝对路径-->
    <base href="/web03_war_exploded/">
</head>
<body>
    <img src="static/img/logo.png">
</body>
</html>
```

#### 4. 缺省项目上下文路径

> 项目上下文路径变化问题

+ 通过 base标签虽然解决了相对路径转绝对路径问题,但是base中定义的是项目的上下文路径
+ 项目的上下文路径是可以随意变化的
+ 一旦项目的上下文路径发生变化,所有base标签中的路径都需要改

> 解决方案

+ `将项目的上下文路径进行缺省设置,设置为 /,所有的绝对路径中就不必填写项目的上下文了,直接就是/开头即可`
### 2.2 重定向中的路径问题

> 目标 :由/x/y/z/servletA重定向到a/b/c/test.html

``` java
@WebServlet("/x/y/z/servletA")
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        
    }
}

```

#### 1. 相对路径写法

``` java
@WebServlet("/x/y/z/servletA")
public class ServletA extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 相对路径重定向到test.html
        resp.sendRedirect("../../../a/b/c/test.html");
    }
}
```

#### 2. 绝对路径写法

  ``` java
  //绝对路径中,要写项目上下文路径
  //resp.sendRedirect("/web03_war_exploded/a/b/c/test.html");
  // 通过ServletContext对象动态获取项目上下文路径
  //resp.sendRedirect(getServletContext().getContextPath()+"/a/b/c/test.html");
  // 缺省项目上下文路径时,直接以/开头即可
  resp.sendRedirect("/a/b/c/test.html");
  ```

`可以通过getContextPath方法获取项目的根路径（上下文路径）`

### 2.3 请求转发中的路径问题

> 目标 :由x/y/servletB请求转发到a/b/c/test.html

``` java
@WebServlet("/x/y/servletB")
public class ServletB extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

    }
}
```

#### 1. 相对路径写法

  ``` java
  @WebServlet("/x/y/servletB")
  public class ServletB extends HttpServlet {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          RequestDispatcher requestDispatcher = req.getRequestDispatcher("../../a/b/c/test.html");
          requestDispatcher.forward(req,resp);
      }
  }
  
  
  ```

#### 2. 绝对路径写法

  ```java
  @WebServlet("/x/y/servletB")
  public class ServletB extends HttpServlet {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          RequestDispatcher requestDispatcher = req.getRequestDispatcher("/a/b/c/test.html");
          requestDispatcher.forward(req,resp);
      }
  }
  ```

#### 3. 目标资源内相对路径处理

+ 此时需要注意,请求转发是服务器行为,浏览器不知道,地址栏不变化,相当于我们访问test.html的路径为http://localhost:8080/web03_war_exploded/x/y/servletB

+ 那么此时 test.html资源的所在路径就是http://localhost:8080/web03_war_exploded/x/y/所以test.html中相对路径要基于该路径编写,如果使用绝对路径则不用考虑

  ``` html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
      <!--
  		当前资源路径是     http://localhost:8080/web03_war_exploded/x/y/servletB
          当前资源所在路径是  http://localhost:8080/web03_war_exploded/x/y/
          目标资源路径=所在资源路径+src属性值 
  		http://localhost:8080/web03_war_exploded/x/y/../../static/img/logo.png
          http://localhost:8080/web03_war_exploded/static/img/logo.png
  		得到目标路径正是目标资源的访问路径	
      -->
  <img src="../../static/img/logo.png">
  </body>
  </html>
  ```
