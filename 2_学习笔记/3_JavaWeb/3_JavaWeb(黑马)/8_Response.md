## 目录

- [1 HTTP协议：响应消息](#1%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%93%8D%E5%BA%94%E6%B6%88%E6%81%AF)
	- [1.1 响应行](#1.1%20%E5%93%8D%E5%BA%94%E8%A1%8C)
	- [1.2 响应头](#1.2%20%E5%93%8D%E5%BA%94%E5%A4%B4)
	- [1.3 响应空行](#1.3%20%E5%93%8D%E5%BA%94%E7%A9%BA%E8%A1%8C)
	- [1.4 响应体](#1.4%20%E5%93%8D%E5%BA%94%E4%BD%93)
- [2 Response对象](#2%20Response%E5%AF%B9%E8%B1%A1)
	- [2.1 设置响应行](#2.1%20%E8%AE%BE%E7%BD%AE%E5%93%8D%E5%BA%94%E8%A1%8C)
	- [2.2 设置响应头](#2.2%20%E8%AE%BE%E7%BD%AE%E5%93%8D%E5%BA%94%E5%A4%B4)
	- [2.3 设置响应体](#2.3%20%E8%AE%BE%E7%BD%AE%E5%93%8D%E5%BA%94%E4%BD%93)
	- [2.4 案例1：重定向](#2.4%20%E6%A1%88%E4%BE%8B1%EF%BC%9A%E9%87%8D%E5%AE%9A%E5%90%91)
	- [2.5 案例2：服务器输出字符数据](#2.5%20%E6%A1%88%E4%BE%8B2%EF%BC%9A%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BE%93%E5%87%BA%E5%AD%97%E7%AC%A6%E6%95%B0%E6%8D%AE)
	- [2.6 案例3：服务器输出字节数据](#2.6%20%E6%A1%88%E4%BE%8B3%EF%BC%9A%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BE%93%E5%87%BA%E5%AD%97%E8%8A%82%E6%95%B0%E6%8D%AE)
	- [2.7 案例4：验证码](#2.7%20%E6%A1%88%E4%BE%8B4%EF%BC%9A%E9%AA%8C%E8%AF%81%E7%A0%81)
- [3 ServletContext对象：](#3%20ServletContext%E5%AF%B9%E8%B1%A1%EF%BC%9A)
	- [3.1 概念](#3.1%20%E6%A6%82%E5%BF%B5)
	- [3.2 获取](#3.2%20%E8%8E%B7%E5%8F%96)
	- [3.3 功能](#3.3%20%E5%8A%9F%E8%83%BD)
- [4 案例：下载](#4%20%E6%A1%88%E4%BE%8B%EF%BC%9A%E4%B8%8B%E8%BD%BD)
	- [4.1 需求](#4.1%20%E9%9C%80%E6%B1%82)
	- [4.2 分析](#4.2%20%E5%88%86%E6%9E%90)
	- [4.3 步骤](#4.3%20%E6%AD%A5%E9%AA%A4)
	- [4.4 中文文件问题](#4.4%20%E4%B8%AD%E6%96%87%E6%96%87%E4%BB%B6%E9%97%AE%E9%A2%98)

## 1 HTTP协议：响应消息

- 服务器端发送给客户端的数据
### 1.1 响应行

1. 组成：协议/版本 响应状态码 状态码描述
2. `响应状态码`：服务器告诉客户端浏览器本次请求和响应的一个状态。
	- 状态码都是3位数字 
		- 1xx：服务器就收客户端消息，但没有接受完成，等待一段时间后，发送1xx多状态码
		- 2xx：<font color="#00b050">成功。代表：200</font>
		- 3xx：重定向。代表：<font color="#00b050">302(重定向</font>，即访问的A资源回应客户端，让其找C资源)，<font color="#00b050">304(访问缓存</font>，即客户端之前访问过的资源，存在了客户端本地，服务器通过304回应，让客户端访问本地缓存)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f74dc1ad50fde25da1086ebd3c3a7f3a.png)
		- 4xx：客户端错误。<font color="#00b050">404（请求路径没有对应的资源）</font> <font color="#00b050">405（请求方式没有对应的doXxx方法）</font>
		- 5xx：服务器端错误。代表：<font color="#00b050">500(服务器内部出现异常)</font>

### 1.2 响应头

1. 格式：头名称： 值
2. 常见的响应头：
	- Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式
	- Content-disposition：服务器告诉客户端以什么格式打开响应体数据
		- 取值：in-line（默认值,在当前页面内打开），attachment;filename=xxx（以附件形式打开响应体。文件下载）


### 1.3 响应空行

### 1.4 响应体

- 传输的数据

- `响应的字符串格式`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b5e8285557f1412f8207b492a458b17b.png)
## 2 Response对象

### 2.1 设置响应行

1. 格式：HTTP/1.1 200 ok
2. 设置状态码：setStatus(int sc) 

### 2.2 设置响应头

- setHeader(String name, String value) 

### 2.3 设置响应体

1. 获取输出流
	* 字符输出流：PrintWriter getWriter()
	* 字节输出流：ServletOutputStream getOutputStream()

2. 使用输出流，将数据输出到客户端浏览器

### 2.4 案例1：重定向

- 资源跳转的方式
	- 告诉浏览器，需要重定向：状态码302
	- 告诉浏览器，跳转资源的路径：响应头location：跳转资源的路径
	- 使用sendRedirect(String s) 方法即可

- `重定向的特点（与转发的区别）`
- forword 和 redirect 的区别 #面试题 

|转发|重定向|
|---|---|
|地址栏路径不会变化|地址栏路径发生变化|
|只能访问当前服务器资源|可以访问其他站点的资源|
|是一次请求，可以使用req对象来共享数据|`是两次请求`，不可以使用req对象来共享数据|
- 路径的写法
	1. `相对路径`：通过相对路径，不可以确定唯一资源
		- 如：./index.html
		- `即不以 / 开头 ，以 . 开头的路径`
		- 规则：找到`当前资源和目标资源之间的相对位置关系`
	1. `绝对路径`：通过绝对路径可以确定唯一资源
		- 如：http://localhost:8080/C15/ResponseDemo1  ---> C15/ResponseDemo1
		- `即以 / 开头的路径，我们可以称为绝对路径`
		- 规则：`判断定义的路径是给谁用的？`即请求从哪里发出
			- 给客户端/浏览器使用，需要加虚拟目录，使用req.getContextPath()获取虚拟目录
			- 给服务器使用，`不需要加虚拟目录，先记住转发，即可`
- 案例如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8fc0373dc20b5e0c510e0af7f27d3298.png)


- `建议，以后都使用绝对路径即可`

```java
@WebServlet("/ResponseDemo1")  
public class ResponseDemo1 extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doPost(request,response);  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        System.out.println("demo1...");  
        //访问responseDemo1，会自动跳转到responseDemo2  
        //1.设置状态码302  
//        response.setStatus(302);  
        //2.设置响应头location  
//        response.setHeader("location","/C15/ResponseDemo2");  
  
        //简单的重定向方法  
//        response.sendRedirect("/C15/ResponseDemo2");  
        response.sendRedirect("http://www.baidu.com"); 

        //优化：动态获取虚拟目录  
        String contextPath = request.getContextPath();  
        response.sendRedirect(contextPath+"/ResponseDemo2");  
//        response.sendRedirect("http://www.baidu.com");
    }  
}

@WebServlet("/ResponseDemo2")  
public class ResponseDemo2 extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doPost(request,response);  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        System.out.println("demo2...");  
    }  
}
```


### 2.5 案例2：服务器输出字符数据

1. 获取字符输出流
	- PrintWriter pw = response.getWriter();  
2. 输出数据
	- pw.write("你好，response");  
3. 乱码问题
	- 浏览器的编码方式，默认与系统的编码方式一直，所以是GBK的  
	- 而流的默认编码：ISO-8859-1
	- 针对与不同的操作系统，其默认编码不同
	- 我们可以告诉浏览器，服务器发送的消息体数据的编码，建议浏览器使用该编码解码
	- `一定要在获取流之前设置`
	- response.setHeader("content-type","text/html;charset=utf-8");  
	- response.setContentType("text/html;charset=utf-8");

```java
@Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        //1.获取字符输出流  
//        PrintWriter pw = response.getWriter();  
        //2.输出数据  
        //注意的是，此流的response获取的，在response完成响应后，会自动销毁  
        //所以此流，不需要flush  
//        pw.write("<h1>hello response</h1>");  
  
        //浏览器的编码方式，默认与系统的编码方式一直，所以是GBK的  
        //而在IDEA中，我们设置了的编码是UTF-8，所以输出中文的时候会发生乱码  
        //所以，获取流之前，将流的默认编码：ISO-8859-1 改为 GBK / utf-8        
        //但是，针对与不同的操作系统，其默认编码不同，增加修改编码的工作  
        //我们可以告诉浏览器，服务器发生的消息体数据的编码，建议浏览器使用该编码解码  
//        response.setCharacterEncoding("utf-8");  
//        response.setHeader("content-type","text/html;charset=utf-8");  
        //简单的形式  
        response.setContentType("text/html;charset=utf-8");  
        PrintWriter pw = response.getWriter();  
        pw.write("你好，response");  
    }
```


### 2.6 案例3：服务器输出字节数据

1. 获取字节输出流
2. 输出数据

```java
@Override  
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
    response.setContentType("text/html;charset=utf-8");  
    //1. 获取字节输出流  
    ServletOutputStream sos = response.getOutputStream();  
    //2. 输出数据  
    sos.write("你好".getBytes());//按什么编码集解码  
    sos.write("你是谁呀".getBytes());  
}
```

### 2.7 案例4：验证码

- 本质：图片
- 目的：防止恶意表单注册

```java
@Override  
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
    int width=100;  
    int height=50;  
    //1. 创建一个对象，代表内存中的图片（验证码的图片对象）  
    BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);  
    //2. 美化图片  
    //2.1 填充背景颜色  
    Graphics g = image.getGraphics();  
    g.setColor(Color.pink);  
    g.fillRect(0,0,width,height);  
    //2.2画边框  
    g.setColor(Color.BLUE);  
    g.drawRect(0,0,width-1,height-1);  
    //2.3 写验证码  
    String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvmxyz0123456789";  
    for (int i = 1; i <=4 ; i++) {  
        char c = str.charAt(new Random().nextInt(str.length()));  
        g.drawString(c+"",width/5*i,height/2);  
    }  
    //2.4 写干扰线  
    g.setColor(Color.green);  
    for (int i = 0; i < 10; i++) {  
        int x1 = new Random().nextInt(100);  
        int y1 = new Random().nextInt(50);  
        int x2 = new Random().nextInt(100);  
        int y2 = new Random().nextInt(50);  
        g.drawLine(x1,y1,x2,y2);  
    }  
    //3. 将图片输出到页面展示  
    ImageIO.write(image,"jpg",response.getOutputStream());  
}
```

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
    <script>        window.onload = function () {  
            //给图片绑定，单击事件  
            var img = document.getElementById("checkCode");  
            img.onclick=function () {  
                //加时间戳，防止本地缓存URL相同的资源  
                var time = new Date().getTime();  
                img.src="/C15/CheckCodeServlet?"+time;  
            }  
            // 给超链接绑定单击事件  
            var a = document.getElementById("change");  
            a.onclick=function (){  
                //加时间戳  
                var time = new Date().getTime();  
                img.src="/C15/CheckCodeServlet?"+time;  
            }  
  
        };  
        // function change1() {  
        //     var img = document.getElementById("checkCode");        
        //     var time = new Date().getTime();        
        //     img.src="/C15/CheckCodeServlet?"+time;        
        // };  
  
    </script>  
</head>  
<body>  
    <img src="/C15/CheckCodeServlet" id="checkCode">  
    <a href="javascript:void(0);" id="change" >看不清，点击换一张</a>  
</body>  
</html>
```

## 3 ServletContext对象：

### 3.1 概念

- `代表整个web应用，可以和程序的容器(服务器)来通信`
- 在我们学习的案例里面，是和Tomcat继续通信的

### 3.2 获取

1. 通过request对象获取
	request.getServletContext();
2. 通过HttpServlet获取
	this.getServletContext();

### 3.3 功能

1. `获取MIME类型`：
	* MIME类型:在互联网<font color="#00b050">通信过程中定义的一种文件数据类型</font>
		* 格式： 大类型/小类型   text/html		image/jpeg

	* 获取：String getMimeType(String file)  
2. `域对象：共享数据`
	1. setAttribute(String name,Object value)
	2. getAttribute(String name)
	3. removeAttribute(String name)

	* <font color="#00b050">ServletContext对象范围：所有用户所有请求的数据</font>
	* <font color="#00b050">谨慎使用</font>
3. `获取文件的真实(服务器)路径`
	1. 方法：String getRealPath(String path)  <font color="#00b050">很重要</font>
```java
@Override  
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
    //获取MIME类型  
    //1.先获取ServletContext对象  
    ServletContext context = this.getServletContext();  
    //2. 获取文件的服务器路径  
    String b = context.getRealPath("/b.txt"); //web目录下资源访问  
    String c = context.getRealPath("/WEB-INF/c.txt"); //web目录下的WEB-INF目录下的资源访问  
    String a = context.getRealPath("WEB-INF/classes/a.txt");  //web目录下的WEB-INF目录下的classes目录下资源访问，即工作空间中src下的资源访问
    System.out.println(b);  
    System.out.println(a);  
}
```

## 4 案例：下载

### 4.1 需求

* 文件下载需求：
	1. 页面显示超链接
	2. 点击超链接后弹出下载提示框
	3. 完成图片文件下载

### 4.2 分析

1. 超链接指向的资源如果能够被浏览器解析，则在浏览器中展示，如果不能解析，则弹出下载提示框。不满足需求
2. 任何资源都必须弹出下载提示框
3. 使用响应头设置资源的打开方式：
	* content-disposition:attachment;filename=xxx

### 4.3 步骤

1. 定义页面，编辑超链接href属性，指向Servlet，传递资源名称filename
2. 定义Servlet
	1. 获取文件名称
	2. 使用字节输入流加载文件进内存
	3. 指定response的响应头： content-disposition:attachment;filename=xxx
	4. 将数据写出到response输出流

### 4.4 中文文件问题

* 解决思路：
	1. 获取客户端使用的浏览器版本信息
	2. 根据不同的版本信息，设置filename的编码方式不同


- `代码`
```java
@WebServlet("/downloadServlet")  
public class downloadServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        //1.获取文件名称  
        String filename = request.getParameter("filename");  
        //2.使用字节输入流加载文件进内存  
        ServletContext context = this.getServletContext();  
        String realPath = context.getRealPath("/img/" + filename);  
        FileInputStream fis = new FileInputStream(realPath);  
        //3.指定response的响应头  
            //设置响应头类型：content-type  
        String mimeType = context.getMimeType(filename);  
        response.setHeader("content-type",mimeType);  
            //设置响应头打开方式：content-disposition  
            //先解决中文文件名问题  
                //获取浏览器请求头  
        String header = request.getHeader("user-agent");  
		        //使用工具类  
        String fileName = DownLoadUtils.getFileName(header, filename);  
        response.setHeader("content-disposition","attachment;filename="+fileName);  
        //4.将数据写出来response输出流  
        ServletOutputStream sos = response.getOutputStream();  
        byte[] buffer = new byte[1024];  
        int len = 0;  
        while ((len=fis.read(buffer))!=-1){  
            sos.write(buffer,0,len);  
        }  
        fis.close();  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        this.doGet(request,response);  
    }  
}
```

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
    <h3>请点击下载</h3>  
    <hr>    <a href="/C15/downloadServlet?filename=九尾.jpg">图片1</a>  
    <a href="/C15/downloadServlet?filename=2.jpg">图片2</a>  
</body>  
</html>
```