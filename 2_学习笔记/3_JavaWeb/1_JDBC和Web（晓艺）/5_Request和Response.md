
## 1 请求信息封装对象Request

### 1.1 获取请求消息数据处理

1). 获取请求方式 ：GET

 * `String getMethod() `

2). 获取get方式请求参数：name=zhangsan

 * `String getQueryString()`

3）获取请求参数通用方式：不论get还是post请求方式都可以使用

`String  getParameter(String name)`
- 根据参数名称获取参数值  

`Map<String,String[]>  getParameterMap()  `
- 获取所有参数的map集合

4).获取请求URL：

`StringBuffer getRequestURL() `

5). 获取协议及版本：HTTP/1.1

`String getProtocol()`

6). 获取客户机的IP地址：

`String getRemoteAddr()  `               

### 1.2 获取请求头数据

`Enumeration<String> getHeaderNames()`:获取所有的请求头名称

`String getHeader(String name)`:通过请求头的名称获取请求头的值                 

补充知识点：Postman添加请求头

### 1.3 请求转发：

- 一种在服务器内部的资源跳转方式

步骤：

   1).通过request对象获取请求转发器对象：

request.getRequestDispatcher(String path)

   2). 使用RequestDispatcher对象来进行转发：

requestDispatcher.forward(ServletRequest request, ServletResponse response)

特点：

1). 浏览器地址栏路径不发生变化

2). 只能转发到当前服务器内部资源中。

## 2 响应信息封装对象Response

由tomcat服务器创建，servlet中处理，最终返回给客户端（或浏览器），到达客户的形式就是响应报文。

### 2.1 设置响应行

响应行格式：HTTP/1.1 200 ok

设置状态码：setStatus(int sc)

### 2.2 设置响应头：

setHeader(String name, String value)

 例： response.setHeader("Content-type","application/json");

### 2.3 设置响应体：

1）. 获取输出流

   PrintWriter getWriter()

2）. 使用输出流，将数据输出到客户端浏览器

response.setContentType("text/html;charset=utf-8");

response.getWriter().write("你好" );

### 2.4 重定向：

response.sendRedirect("/day15/responseDemo2");

重定向的特点:redirect

1. 地址栏发生变化

2. 重定向可以访问其他站点(服务器)的资源，也可以访问自己的资源

3. 重定向是两次请求。不能使用request对象来共享数据