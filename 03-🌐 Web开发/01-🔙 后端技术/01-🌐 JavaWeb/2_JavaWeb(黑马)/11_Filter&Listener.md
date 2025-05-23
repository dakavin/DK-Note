
## 1 Filter：过滤器

### 1.1 概念

* 生活中的过滤器：净水器,空气净化器，土匪、
* web中的过滤器：当访问服务器的资源时，过滤器可以将请求拦截下来，完成一些特殊的功能。
* 过滤器的作用：
	* 一般用于完成通用的操作。如：登录验证、统一编码处理、敏感字符过滤...

### 1.2 快速入门

1. 步骤：
	- 定义一个类，实现接口Filter
	- 复写方法
	- 配置拦截路径
		1. web.xml
		2. 注解

2. 代码：
```java
@WebFilter("/*")//访问所有资源之前，都会执行该过滤器  
public class FilterDemo1 implements Filter {  
    @Override  
    public void init(FilterConfig filterConfig) throws ServletException {  
  
    }  
    @Override  
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {  
        System.out.println("filterDemo1被执行了");  
        //放行  
        filterChain.doFilter(servletRequest,servletResponse);  
    }  
  
    @Override  
    public void destroy() {  
  
    }}
```

### 1.3 过滤器细节

1. web.xml配置
```xml
<filter>  
    <filter-name>demo1</filter-name>  
    <filter-class>com.dakkk.filter.FilterDemo1</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>demo1</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

- `对比Servlet的xml配置方式` 
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

2. 过滤器执行流程
	- 执行过滤器
	- 执行放行后的资源
	- 回来执行过滤器放行代码下边的代码![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/674246fe34d063ac567adf9554f78e84.png)



3. 过滤器生命周期方法
	- init:在服务器启动后，会创建Filter对象，然后调用init方法。只执行一次。用于加载资源
	- doFilter:每一次请求被拦截资源时，会执行。执行多次
	- destroy:在服务器关闭后，Filter对象被销毁。如果服务器是正常关闭，则会执行destroy方法。只执行一次。用于释放资源![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eef566967fd324b6f609aceda01cf967.png)



4. 过滤器配置详解
	* `拦截路径配置`：
		1. `具体资源路径： /index.jsp`   只有访问index.jsp资源时，过滤器才会被执行
		2. `拦截目录： /user/*`	访问/user下的所有资源时，过滤器都会被执行
		3. `后缀名拦截： *.jsp`		访问所有后缀名为jsp资源时，过滤器都会被执行
		4. `拦截所有资源：/*`		访问 所有资源时，过滤器都会被执行
	* `拦截方式配置`：资源被访问的方式![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/40c473cfcd4766bca9759719d967c090.png)


		* 注解配置：
			* 设置dispatcherTypes属性
				1. REQUEST：默认值。浏览器直接请求资源
				2. FORWARD：转发访问资源
				3. INCLUDE：包含访问资源
				4. ERROR：错误跳转资源
				5. ASYNC：异步访问资源
		* web.xml配置
			* 设置<dispatcher></dispatcher>标签即可

5. 过滤器链(配置多个过滤器)
	* 执行顺序：如果有两个过滤器：过滤器1和过滤器2
		1. 过滤器1
		2. 过滤器2
		3. 资源执行
		4. 过滤器2
		5. 过滤器1 

	* 过滤器先后顺序问题：
		1. 注解配置：按照`类名的字符串比较规则比较，值小的先执行`
			* 如： AFilter 和 BFilter，AFilter就先执行了。
		2. web.xml配置： \<filter-mapping>谁定义在上边，谁先执行

### 1.4 案例:登录验证

- 逻辑分析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3adf11a690e664a580f93bdb8e20b638.png)


* 需求：
	1. 访问day17_case案例的资源。验证其是否登录
	2. 如果登录了，则直接放行。
	3. 如果没有登录，则跳转到登录页面，提示"您尚未登录，请先登录"。

- 代码
```java
@WebFilter("/*")  
public class LoginFilter implements Filter {  
    @Override  
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {  
        //0. 强转为HttpServlet才能获取路径  
        HttpServletRequest request = (HttpServletRequest) req;  
        //1.获取资源请求的路径  
        String requestURI = request.getRequestURI();  
        //2. 判断是否为登陆的资源路径,需要注意排除css/js/图片/验证码登资源  
        if (requestURI.equals("/login.jsp")||requestURI.equals("/loginServlet")  
                ||requestURI.equals("/checkCodeServlet")||requestURI.equals("/js/")  
                ||requestURI.equals("/css/")||requestURI.equals("/font/")){  
            //放行  
            chain.doFilter(req, resp);  
        }else{  
            //验证用户是否登陆了  
            Object user = request.getSession().getAttribute("user");  
            if (user!=null){  
                //放行  
                chain.doFilter(req, resp);  
            }else{  
                //跳转登陆页面  
                request.setAttribute("login_msg","您尚未登陆，请登陆");  
                request.getRequestDispatcher("/login.jsp").forward(request,resp);  
            }  
        }  
  
    }  
  
    public void init(FilterConfig config) throws ServletException {  
    }  
    public void destroy() {  
    }}
```


### 1.5 案例：敏感词汇过滤

* 需求：
	1. 对day17_case案例录入的数据进行敏感词汇过滤
	2. 敏感词汇参考《敏感词汇.txt》
	3. 如果是敏感词汇，替换为 *** 


* 分析：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4867c131689be5617de80f353a036983.png)


	1. 对`request对象进行增强(AOP代理模式)`。增强获取参数相关方法
	2. 放行。传递代理对象

- `代码`
```java
@WebFilter("/*")  
public class SensitiveWordsFilter implements Filter {  
    @Override  
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {  
        //4. 使用代理模式，增强req的getParameter方法  
        ServletRequest request = (ServletRequest) Proxy.newProxyInstance(req.getClass().getClassLoader(), req.getClass().getInterfaces(), new InvocationHandler() {  
            @Override  
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
                if (method.getName().equals("getParameter")) {  
                    //原来方法可以得到的值  
                    String value = (String) method.invoke(req, args);  
                    //遍历敏感词汇  
                    if (value != null) {  
                        for (String s : list) {  
                            if (value.contains(s)) {  
                                //如果value中存在敏感词汇，则进行更换  
                                value = value.replaceAll(s, "***");  
                                System.out.println(value);  
                            }  
                        }  
                    }  
                    return value;  
                }  
  
                //不是该方法的，直接返回方法原来的返回值  
                return method.invoke(req, args);  
            }  
        });  
  
        //注意一定要放行  
        chain.doFilter(request, resp);  
  
    }  
  
    //3. 将敏感词汇使用list集合存储  
    List<String> list = new ArrayList<>();  
  
    public void init(FilterConfig config) throws ServletException {  
        InputStreamReader isr = null;  
        BufferedReader br = null;  
        try {  
            //0. 注意：先在src文件夹下存放配置文件  
            //1. 使用ServletContext获取敏感词汇文件的真实路径  
            ServletContext servletContext = config.getServletContext();  
            String realPath = servletContext.getRealPath("/WEB-INF/classes/敏感词汇.txt");  
            //2. 注意该文件是GBK格式的，需要使用InputStreamReader  
            isr = new InputStreamReader(new FileInputStream(realPath), "GBK");  
            br = new BufferedReader(isr);  
            String line = null;  
            while ((line = br.readLine()) != null) {  
                list.add(line);  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        } finally {  
            try {  
                if (br != null)  
                    br.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
            try {  
                if (isr != null)  
                    isr.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
  
    public void destroy() {  
    }}
```

### 1.6 增强对象的功能：
* 设计模式：一些通用的解决固定问题的方式
1. `装饰模式`
2. `代理模式`
	* 概念：
		1. <font color="#00b050">真实对象</font>：被代理的对象
		2. <font color="#00b050">代理对象</font>：
		3. <font color="#00b050">代理模式</font>：代理对象代理真实对象，达到增强真实对象功能的目的
	* 实现方式：
		1. 静态代理：有一个类文件描述代理模式 
		2. 动态代理：在内存中形成代理类
			* 实现步骤：
				1. 代理对象和真实对象实现相同的接口
				2. 代理对象 = Proxy.newProxyInstance();
				3. 使用代理对象调用方法。
				4. 增强方法

			* 增强方式：
				1. `增强参数列表`
				2. `增强返回值类型`
				3. `增强方法体执行逻辑`	



## 2 Listener：监听器

### 2.1 概念

* 概念：web的三大组件之一。
	* 事件监听机制
		* 事件	：一件事情
		* 事件源 ：事件发生的地方
		* 监听器 ：一个对象
		* 注册监听：`将事件、事件源、监听器绑定在一起。 当事件源上发生某个事件后，执行监听器代码`

### 2.2 ServletContextListener

- 监听ServletContext对象的创建和销毁


* 方法：
	* `void contextDestroyed(ServletContextEvent sce)` ：<font color="#00b050">ServletContext对象被销毁之前会调用该方法</font>
	* `void contextInitialized(ServletContextEvent sce)` ：<font color="#00b050">ServletContext对象创建后会调用该方法</font>
* 步骤：
	1. 定义一个类，实现ServletContextListener接口
	2. 复写方法
```java
@WebListener  
public class ContextLoaderListener implements ServletContextListener {  
    /**  
     * 监听ServletContext对象创建的。ServletContext对象服务器启动后自动创建  
     * @param servletContextEvent  
     */  
    @Override  
    public void contextInitialized(ServletContextEvent servletContextEvent) {  
        //一般用于加载资源文件  
        //1. 获取servletContext对象  
        ServletContext servletContext = servletContextEvent.getServletContext();  
        //2. 加载资源文件  
        String contextConfigLocation = servletContext.getInitParameter("contextConfigLocation");  
        //3. 获取真实路径  
        String realPath = servletContext.getRealPath(contextConfigLocation);  
        //4. 加载进内存  
        try {  
            FileInputStream fis = new FileInputStream(realPath);  
            System.out.println(fis);  
        } catch (FileNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
  
        System.out.println("ServletContext对象被创建了。。。");  
    }  
  
    /**  
     * 监听ServletContext对象销毁的。ServletContext对象服务器正常关闭后方法被调用  
     * @param servletContextEvent  
     */  
    @Override  
    public void contextDestroyed(ServletContextEvent servletContextEvent) {  
        System.out.println("ServletContext对象被销毁了。。。");  
    }  
}
```

- 配置
- `web.xml`
```xml
<listener>
	<listener-class>
		cn.itcast.web.listener.ContextLoaderListener
	</listener-class>
</listener>
```

- `指定初始化参数<context-param>`
```xml
<context-param>  
    <param-name>contextConfigLocation</param-name>  
    <param-value>/WEB-INF/classes/applicationContext.xml</param-value>  
</context-param>
```

1. 注解：
	* @WebListener