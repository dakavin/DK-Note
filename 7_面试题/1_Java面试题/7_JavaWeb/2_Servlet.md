
## 1 Servlet生命周期

> Servlet生命周期?

- **第一次访问**Servlet，我们发现**init()和service()都被调用了**

- **第二次访问**Servlet，**service()被调用了**

- 第三次访问Servlet，**还是service()被调用了**

- 当我们**关闭Tomcat服务器**的时候，**destroy()被调用了！**

Servlet生命周期可分为5个步骤

1. **加载Servlet**。当Tomcat第一次访问Servlet的时候，**Tomcat会负责创建Servlet的实例**
2. **初始化**。当Servlet被实例化后，Tomcat会**调用init()方法初始化这个对象**
3. **处理服务**。当浏览器**访问Servlet**的时候，Servlet **会调用service()方法处理请求**
4. **销毁**。当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，**让该实例释放掉所占的资源**。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁
5. **卸载**。当Servlet调用完destroy()方法后，等待垃圾回收。如果**有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作**。

- 简单总结：**只要访问Servlet，service()就会被调用。init()只有第一次访问Servlet的时候才会被调用。destroy()只有在Tomcat关闭的时候才会被调用。**

## 2 get方式和post方式有何区别

数据携带上:

- GET方式：在URL地址后附带的参数是有限制的，其数据容量通常不能超过1K。
- POST方式：可以在请求的实体内容中向服务器发送数据，传送的数据量无限制。

请求参数的位置上:

- GET方式：请求参数放在URL地址后面，以?的方式来进行拼接
- POST方式:请求参数放在HTTP请求包中

用途上:

- GET方式一般用来获取数据
- POST方式一般用来提交数据
    
    - 原因:
        
        - 首先是因为GET方式携带的数据量比较小，无法带过去很大的数量
        - POST方式提交的参数后台更加容易解析(使用POST方式提交的中文数据，后台也更加容易解决)
        - GET方式比POST方式要快

GET方式比POST方式要快，详情可看:[https://www.cnblogs.com/strayling/p/3580048.html](https://link.segmentfault.com/?enc=GbEwmt1wbtm47qUiYirJoQ%3D%3D.I48aZv2n34ayOACBTdQUPHPkUd5ZcQ%2BtP93G1bSM54LAu3LO8UbNaAdFJsl3EghF9JU8E7cbvGUcS02tYSHHhg%3D%3D)

## 3 Servlet相关 API

> doGet与doPost方法的两个参数是什么

1. HttpServletRequest：封装了与请求相关的信息
2. HttpServletResponse：封装了与响应相关的信息

> 获取页面的元素的值有几种方式，分别说一下

1. request.getParameter() 返回客户端的请求参数的值
2. request.getParameterNames() 返回所有可用属性名的枚举
3. request.getParameterValues() 返回包含参数的所有值的数组

> request.getAttribute()和request.getParameter()区别

用途上:

- request.getAttribute()， **一般用于获取request域对象的数据**(在跳转之前把数据使用setAttribute来放到request对象上)
- request.getParameter()， **一般用于获取客户端提交的参数**

存储数据上:

- request.getAttribute()可以获取Objcet对象
- request.getParameter()只能获取字符串(这也是为什么它一般用于获取客户端提交的参数)

## 4 forward和redirect的区别

> forward和redirect的区别

- **实际发生位置不同，地址栏不同**
    
- 转发是发生在服务器的
	- **转发是由服务器进行跳转的**，细心的朋友会发现，在转发的时候，**浏览器的地址栏是没有发生变化的**，在我访问Servlet111的时候，即使跳转到了Servlet222的页面，浏览器的地址还是Servlet111的。也就是说**浏览器是不知道该跳转的动作，转发是对浏览器透明的**。通过上面的转发时序图我们也可以发现，**实现转发只是一次的http请求**，**一次转发中request和response对象都是同一个**。这也解释了，为什么可以使用**request作为域对象进行Servlet之间的通讯。**

- 重定向是发生在浏览器的

- **用法不同:**
    
    - 很多人都搞不清楚转发和重定向的时候，**资源地址究竟怎么写**。有的时候要把应用名写上，有的时候不用把应用名写上。很容易把人搞晕。记住一个原则： **给服务器用的直接从资源名开始写，给浏览器用的要把应用名写上**
        
        - request.getRequestDispatcher("/资源名 URI").forward(request,response)
            
            - **转发时"/"代表的是本应用程序的根目录【zhongfucheng】**
        - response.send("/web应用/资源名 URI");
            
            - **重定向时"/"代表的是webapps目录**
- **能够去往的URL的范围不一样:**
    
    - **转发是服务器跳转只能去往当前web应用的资源**
    - **重定向是服务器跳转，可以去往任何的资源**
- **传递数据的类型不同**
    
    - **转发的request对象可以传递各种类型的数据，包括对象**
    - **重定向只能传递字符串**
- **跳转的时间不同**
    
    - **转发时：执行到跳转语句时就会立刻跳转**
    - **重定向：整个页面执行完之后才执行跳转**

那么转发(forward)和重定向(redirect)使用哪一个？

- 根据上面说明了转发和重定向的区别也可以很容易概括出来**。转发是带着转发前的请求的参数的。重定向是新的请求**。

典型的应用场景：

1. 转发: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
2. 重定向: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了

## 5 tomcat容器是如何创建servlet类实例？用到了什么原理？

1. 当容器启动时，会读取在webapps目录下所有的web应用中的web.xml文件，然后对 **xml文件进行解析，并读取servlet注册信息**。然后，将每个应用中注册的servlet类都进行加载，并通过 **反射的方式实例化**。（有时候也是在第一次请求时实例化）
2. 在servlet注册时加上`<load-on-startup>1</load-on-startup>`如果为正数，则在一开始就实例化，如果不写或为负数，则第一次请求实例化。


## 6 什么是cookie？Session和cookie有什么区别？

> 什么是cookie？

Cookie是由W3C组织提出，最早由netscape社区发展的一种机制

- 网页之间的**交互是通过HTTP协议传输数据的，**而Http协议是**无状态的协议**。无状态的协议是什么意思呢？**一旦数据提交完后，浏览器和服务器的连接就会关闭，再次交互的时候需要重新建立新的连接**。
- 服务器无法确认用户的信息，于是乎，W3C就提出了：**给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息**。通行证就是Cookie

> Session和cookie有什么区别？

- **从存储方式上比较**
    
    - Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
    - Session可以存储任何类型的数据，可以把Session看成是一个容器
- **从隐私安全上比较**
    
    - **Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密
    - **Session存储在服务器上，对客户端是透明的**。不存在敏感信息泄露问题。
- **从有效期上比较**
    
    - Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在的
    - **Session的保存在服务器中，设置maxInactiveInterval属性值来确定Session的有效期。并且Session依赖于名为JSESSIONID的Cookie，该Cookie默认的maxAge属性为-1。如果关闭了浏览器，该Session虽然没有从服务器中消亡，但也就失效了。**
- **从对服务器的负担比较**
    
    - Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
    - Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。
- **从浏览器的支持上比较**
    
    - 如果浏览器禁用了Cookie，那么Cookie是无用的了！
    - 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。
- **从跨域名上比较**
    
    - Cookie可以设置domain属性来实现跨域名
    - Session只在当前的域名内有效，不可夸域名

## 7 Servlet安全性问题

由于Servlet是单例的，当多个用户访问Servlet的时候，**服务器会为每个用户创建一个线程**。**当多个用户并发访问Servlet共享资源的时候就会出现线程安全问题**。

原则：

1. 如果一个**变量需要多个用户共享**，则应当在访问该变量的时候，**加同步机制synchronized (对象){}**
2. 如果一个变量**不需要共享**，则**直接在 doGet() 或者 doPost()定义**.这样不会存在线程安全问题