SpringMVC是隶属于Spring框架的一部分，主要是用来进行Web开发，是对Servlet进行了封装。

SpringMVC是处于Web层的框架，所以其主要作用就是用来接收前段发过来的请求和数据，然后经过处理之后将处理结果响应给前端，所以`如何处理请求和响应是SpringMVC中非常重要的一块内容`。

REST是一种软件架构风格，可以降低开发的复杂性，提高系统的可伸缩性，后期的应用也是非常广泛。

对于SpringMVC的学习，`最终要达成的目标：`

1. 掌握基于SpringMVC获取请求参数和响应JSON数据操作
2. 熟练应用基于RESTFul风格的请求路径设置与参数传递
3. 能根据实际业务建立前后端开发通信协议，并进行实现
4. 基于SSM整合技术开发任意业务模块功能

## 1 SpringMVC概述

学习SpringMVC我们先来回顾下现在Web程序是如何做的，我们现在的Web程序大都基于MVC三层架构来实现的。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29f89be0685c0de66ca2624a2a875465.png)
- 如果所有的处理都交给`Servlet`来处理的话，所有的东西都耦合在一起，对后期的维护和扩展极其不利
    - 所以将后端服务器`Servlet`拆分成三层，分别是`web`、`service`和`dao`
        - `web`层主要由`servlet`来处理，负责页面请求和数据的收集以及响应结果给前端
        - `service`层主要负责业务逻辑的处理
        - `dao`层主要负责数据的增删改查操作
- 但`servlet`处理请求和数据时，存在一个问题：一个`servlet`只能处理一个请求
- 针对`web`层进行优化，采用<font color="#00b050">MVC设计模式</font>，将其设计为`Controller`、`View`和`Model`
    - `controller`负责请求和数据的接收，接收后将其转发给`service`进行业务处理
    - `service`根据需要会调用`dao`对数据进行增删改查
    - `dao`把数据处理完后，将结果交给`service`，`service`再交给`controller`
    - `controller`根据需求组装成`Model`和`View`，`Model`和`View`组合起来生成页面，转发给前端浏览器
    - 这样做的好处就是`controller`可以处理多个请求，并对请求进行分发，执行不同的业务操作

随着互联网的发展，上面的模式因为是同步调用，性能慢，跟不上需求，所以异步调用慢慢的走到了前台，是现在比较流行的一种处理方式。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5b95e425f38f1052ad531f5a7af530d.png)


- 因为是`异步调用，所以后端不需要返回View视图，将其去除`
- 前端如果通过异步调用的方式进行交互，后端就需要将返回的数据转换成JSON格式进行返回
- SpringMVC主要负责的就是
    - controller如何接收请求和数据
    - 如何将请求和数据转发给业务层
    - 如何将响应数据转换成JSON发挥到前端
- SpringMVC是一种基于Java实现MVC模型的轻量级Web框架
    - 优点
        - 使用简单、开发快捷（相比较于Servlet）
        - 灵活性强

这里说的优点，我们通过下面的讲解与联系慢慢体会
## 2 SpringMVC入门案例

因为SpringMVC是一个Web框架，将来是要替换Servlet,所以先来回顾下以前Servlet是如何进行开发的?

1. 创建web工程(Maven结构)
2. 设置tomcat服务器，加载web工程(tomcat插件)
3. 导入坐标(Servlet)
4. 定义处理请求的功能类(UserServlet)
5. 设置请求映射(配置映射关系)

SpringMVC的制作过程和上述流程几乎是一致的，具体的实现流程是什么?

1. 创建web工程(Maven结构)
2. 设置tomcat服务器，加载web工程(tomcat插件)
3. 导入坐标 **(SpringMVC+Servlet)**
4. 定义处理请求的功能类(UserController)
5. 设置请求映射(配置映射关系)
6. 将SpringMVC设定加载到Tomcat容器中
### 2.1 案例制作

- `步骤一：`创建Maven项目
- `步骤二：`导入所需坐标(SpringMVC+Servlet)  
    在`pom.xml`中导入下面两个坐标
    ```xml
<!--servlet-->  
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<!--springmvc-->  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
<!--注意加上Tomcat的插件-->
<plugin>  
  <groupId>org.apache.tomcat.maven</groupId>  
  <artifactId>tomcat7-maven-plugin</artifactId>  
  <version>2.2</version>  
  <configuration>    
	<port>8080</port>  
    <path>/</path>  
  </configuration>  
</plugin>
	```

- `步骤三：`创建SpringMVC控制器类（等同于我们前面做的Servlet）
```java
//定义Controller，使用@Controller定义Bean  
@Controller  
public class UserController {  
    //设置当前访问路径，使用@RequestMapping  
    @RequestMapping("/save")  
    //设置当前对象的返回值类型  
    @ResponseBody  
    public String save(){  
        System.out.println("user save ...");  
        return "{'module':'SpringMVC'}";  
    }  
}
```

- `步骤四：`初始化SpringMVC环境（同Spring环境），设定SpringMVC加载对应的Bean
```java
//创建SpringMVC的配置文件，加载controller对应的bean  
@Configuration  
//  
@ComponentScan("com.blog.controller")  
public class SpringMvcConfig {  
  
}
```

- `步骤五：`初始化Servlet容器，加载SpringMVC环境，并设置SpringMVC技术处理的请求
```java
//定义一个servlet容器的配置类，在里面加载Spring的配置，继承AbstractDispatcherServletInitializer并重写其方法  
public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {  
    //加载SpringMvc容器配置  
    protected WebApplicationContext createServletApplicationContext() {  
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
        context.register(SpringMvcConfig.class);  
        return context;  
    }  
    //设置哪些请求归SpringMvc处理  
    protected String[] getServletMappings() {  
        //所有请求都交由SpringMVC处理  
        return new String[]{"/"};  
    }  
  
    //加载Spring容器配置  
    protected WebApplicationContext createRootApplicationContext() {  
        return null;  
    }  
}
```

- `步骤六：`访问`http://localhost:8080/save`  
    页面上成功出现`{'info':'springmvc'}`，至此我们的SpringMVC入门案例就完成了

- 注意事项:
	- SpringMVC是基于Spring的，在pom.xml只导入了`spring-webmvc`jar包的原因是它会自动依赖spring相关坐标
	- `AbstractDispatcherServletInitializer`类是SpringMVC提供的快速初始化Web3.0容器的抽象类
	- `AbstractDispatcherServletInitializer`提供了三个接口方法供用户实现
	    - `createServletApplicationContext`方法，创建Servlet容器时，加载SpringMVC对应的bean并放入`WebApplicationContext`对象范围中，而`WebApplicationContext`的作用范围为`ServletContext`范围，即整个web容器范围
	    - `getServletMappings`方法，设定SpringMVC对应的请求映射路径，即SpringMVC拦截哪些请求
	    - `createRootApplicationContext`方法，如果创建Servlet容器时需要加载非SpringMVC对应的bean，使用当前方法进行，使用方式和`createServletApplicationContext`相同。
	- `createServletApplicationContext`用来加载SpringMVC环境
	- `createRootApplicationContext`用来加载Spring环境

知识点1：`@Controller`

|名称|@Controller|
|---|---|
|类型|类注解|
|位置|SpringMVC控制器类定义上方|
|作用|设定SpringMVC的核心控制器bean|

知识点2：`@RequestMapping`

|名称|@RequestMapping|
|---|---|
|类型|类注解或方法注解|
|位置|SpringMVC控制器类或方法定义上方|
|作用|设置当前控制器方法请求访问路径|
|相关属性|value(默认)，请求访问路径|

知识点3：`@ResponseBody`

|名称|@ResponseBody|
|---|---|
|类型|类注解或方法注解|
|位置|SpringMVC控制器类或方法定义上方|
|作用|设置当前控制器方法响应内容为当前返回值，无需解析|

### 2.2 入门案例小结

- 一次性工作
    - 创建工程，设置服务器，加载工程
    - 导入坐标
    - 创建web容器启动类，加载SpringMVC配置，并设置SpringMVC请求拦截路径
    - SpringMVC核心配置类（设置配置类，扫描controller包，加载Controller控制器bean）
- 多次工作
    - 定义处理请求的控制器类
    - 定义处理请求的控制器方法，并配置映射路径（@RequestMapping）与返回json数据（@ResponseBody）
### 2.3 工作流程解析

这里将SpringMVC分为两个阶段来分析，分别是`启动服务器初始化过程`和`单次请求过程`
#### 2.3.1 启动服务器初始化过程

1. 服务器启动，执行ServletContainerInitConfig类，初始化web容器
	- 功能类似于web.xml
	```java
public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {  
  
    protected WebApplicationContext createServletApplicationContext() {  
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
        context.register(SpringMvcConfig.class);  
        return context;  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    protected WebApplicationContext createRootApplicationContext() {  
        return null;  
    }  
}
	```

2. 执行createServletApplicationContext方法，创建了WebApplicationContext对象
	- 该方法加载SpringMVC的配置类SpringMvcConfig来`初始化SpringMVC的容器`
	```java
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    context.register(SpringMvcConfig.class);  
    return context;  
}
	```

3. 加载SpringMvcConfig配置类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
public class SpringMvcConfig {  
  
}
```

4. 执行`@ComponentScan`加载对应的bean
    - 扫描指定包及其子包下所有类上的注解，如Controller类上的`@Controller`注解

5. 加载`UserController`，每个`@RequestMapping`的名称对应一个具体的方法
	- 此时就建立了 `/save` 和 `save()`方法的对应关系
	```java
@Controller  
public class UserController {  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("user save ...");  
        return "{'module':'SpringMVC'}";  
    }  
}
	```

6. 执行`getServletMappings`S方法，设定SpringMVC拦截请求的路径规则
	- `/`代表所拦截请求的路径规则，只有被拦截后才能交给SpringMVC来处理请求
	```java
protected String[] getServletMappings() {  
    return new String[]{"/"};  
}
	```
#### 2.3.2 单次请求过程

1. 发送请求`http://localhost:8080/save`
2. web容器发现该请求满足SpringMVC拦截规则，将请求交给SpringMVC处理
3. 解析请求路径/save
4. 由`/save`匹配执行对应的方法`save()`
    - 上面的第5步已经将请求路径和方法建立了对应关系，通过`/save`就能找到对应的`save()`方法
5. 执行`save()`
6. 检测到有`@ResponseBody`直接将`save()`方法的返回值作为响应体返回给请求方

### 2.4 Bean加载控制

#### 2.4.1 问题分析

入门案例的内容已经做完了，在入门案例中我们创建过一个`SpringMvcConfig`的配置类，在之前学习Spring的时候也创建过一个配置类`SpringConfig`。这两个配置类都需要加载资源，那么它们分别都需要加载哪些内容?

我们先来回顾一下项目结构  
`com.blog`下有`config`、`controller`、`service`、`dao`这四个包

- `config`目录存入的是配置类，写过的配置类有:
    - ServletContainersInitConfig
    - SpringConfig
    - SpringMvcConfig
    - JdbcConfig
    - MybatisConfig
- `controller`目录存放的是`SpringMVC`的`controller`类
- `service`目录存放的是`service`接口和实现类
- `dao`目录存放的是`dao/Mapper`接口

controller、service和dao这些类都需要被容器管理成bean对象，那么到底是该让`SpringMVC`加载还是让`Spring`加载呢?

- `SpringMVC`控制的bean
    - 表现层bean,也就是`controller`包下的类
- `Spring`控制的bean
    - 业务bean(`Service`)
    - 功能bean(`DataSource`,`SqlSessionFactoryBean`,`MapperScannerConfigurer`等)

分析清楚谁该管哪些bean以后，接下来要解决的问题是如何让`Spring`和`SpringMVC`分开加载各自的内容。
#### 2.4.2 思路分析

对于上面的问题，解决方案也比较简单

- 加载Spring控制的bean的时候，`排除掉`SpringMVC控制的bean

那么具体该如何实现呢？

- 方式一：Spring加载的bean设定扫描范围`com.blog`，排除掉`controller`包内的bean
- 方式二：Spring加载的bean设定扫描范围为精确扫描，具体到`service`包，`dao`包等
- 方式三：不区分Spring与SpringMVC的环境，加载到同一个环境中(`了解即可`)

	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14d2d0a88ca3f6fdcb2e0370d7950ba6.png)


#### 2.4.3 环境准备

在入门案例的基础上追加一些类来完成环境准备

- 导入坐标
```xml
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.16</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.5.6</version>  
</dependency>  
  
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.46</version>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>
```

- `com.blog.config`下新建`SpringConfig`类
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 创建一张数据表
```sql
create table tb_user(  
    id int primary key auto_increment,  
    name varchar(25),  
    age int  
)
```

- 新建`com.blog.service`，`com.blog.dao`，`com.blog.domain`包，并编写如下几个类
```java
@Data
public class User {  
    private Integer id;  
    private String name;  
    private Integer age;  
}

public interface UserDao {  
    @Insert("insert into tb_user(`name`,age) values (#{name},#{age})") 
    public void save(User user);  
}

public interface UserService {  
    void save(User user);  
}

public class UserServiceImpl implements UserService {  
    public void save(User user) {  
        System.out.println("user service ...");  
    }  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        System.out.println(context.getBean(UserController.class));  
    }  
}
```

#### 2.4.4 设置bean加载控制

- 运行App运行类，如果Spring配置类扫描到了UserController类，则会正常输出，否则将报错  
	当前配置环境下，将正常输出
	`com.blog.controller.UserController@8e0379d`

- 解决方案一：修改Spring配置类，设定扫描范围为精准范围
```java
@Configuration  
@ComponentScan({"com.blog.dao","com.blog.service"})  
public class SpringConfig {  
}
```
再次运行App运行类，报错`NoSuchBeanDefinitionException`，说明Spring配置类没有扫描到UserController，目的达成

- 解决方案二：修改Spring配置类，设定扫描范围为com.blog，排除掉controller包中的bean
```java
@Configuration  
@ComponentScan(value = "com.blog",  
    excludeFilters = @ComponentScan.Filter(  
            type = FilterType.ANNOTATION,  
            classes = Controller.class  
    ))  
public class SpringConfig {  
}
```

- excludeFilters属性：设置扫描加载bean时，排除的过滤规则
- type属性：设置排除规则，当前使用按照bean定义时的注解类型进行排除
    - ANNOTATION：按照注解排除
    - ASSIGNABLE_TYPE:按照指定的类型过滤
    - ASPECTJ:按照Aspectj表达式排除，基本上不会用
    - REGEX:按照正则表达式排除
    - CUSTOM:按照自定义规则排除
- classes属性：设置排除的具体注解类，当前设置排除`@Controller`定义的bean

- 运行程序之前，我们还需要把`SpringMvcConfig`配置类上的`@ComponentScan`注解注释掉，否则不会报错，将正常输出

- 出现问题的原因是
    - Spring配置类扫描的包是`com.blog`
    - SpringMVC的配置类，`SpringMvcConfig`上有一个`@Configuration`注解，也会被Spring扫描到
    - SpringMvcConfig上又有一个`@ComponentScan`，把controller类又给扫描进来了
    - 所以如果不把`@ComponentScan`注释掉，Spring配置类将Controller排除，但是因为扫描到SpringMVC的配置类，又将其加载回来，演示的效果就出不来
    - 解决方案，也简单，把SpringMVC的配置类移出Spring配置类的扫描范围即可。

运行程序，同样报错`NoSuchBeanDefinitionException`，目的达成

最后一个问题，有了Spring的配置类，要想在tomcat服务器启动将其加载，我们需要修改ServletContainersInitConfig
```java
public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {  
    //加载SpringMvc配置  
    protected WebApplicationContext createServletApplicationContext() {  
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
        context.register(SpringMvcConfig.class);  
        return context;  
    }  
    //设置哪些请求归SpringMvc处理  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    //加载Spring容器配置  
    protected WebApplicationContext createRootApplicationContext() {  
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
        context.register(SpringConfig.class);  
        return context;  
    }  
}
```

对于上面的`ServletContainerInitConfig`配置类，Spring还提供了一种更简单的配置方式，可以不用再去创建`AnnotationConfigWebApplicationContext`对象，不用手动`register`对应的配置类  
我们改用继承它的子类`AbstractAnnotationConfigDispatcherServletInitializer`，然后重写三个方法即可
```java
public class ServletContainerInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[]{SpringConfig.class};  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
}
```

知识点：`@ComponentScan`

|名称|@ComponentScan|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|设置spring配置类扫描路径，用于加载使用注解格式定义的bean|
|相关属性|excludeFilters:排除扫描路径中加载的bean,需要指定类别(type)和具体项(classes)  <br>includeFilters:加载指定的bean，需要指定类别(type)和具体项(classes)|
## 3 PostMan工具的使用

### 3.1 创建WorkSpace工作空间

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/518d7df77f3fae76983a234d905ac8ab.png)


### 3.2 发送请求

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c71e0e5ae219a7f028104c50610e6f8.png)


### 3.3 保存当前请求

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9c3478fff038c8c21e46b4dfbb98309.png)



## 4 请求与响应

前面我们已经完成了入门案例相关的知识学习，接来了我们就需要针对SpringMVC相关的知识点进行系统的学习。  
SpringMVC是web层的框架，主要的作用是接收请求、接收数据、响应结果。  
所以这部分是学习SpringMVC的重点内容，这里主要会讲解四部分内容:

- 请求映射路径
- 请求参数
- 日期类型参数传递
- 响应JSON数据
### 4.1 设置请求映射路径

#### 4.1.1 环境准备

- 创建一个Maven项目
- 导入坐标  
    这里暂时只导`servlet`和`springmvc`的就行
```xml
<!--servlet-->  
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<!--springmvc-->  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

- 编写UserController和BookController
```java
@Controller  
public class UserController {  
  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("user save ..");  
        return "{'module':'user save'}";  
    }  
  
    @RequestMapping("/delete")  
    @ResponseBody  
    public String delete(){  
        System.out.println("user delete ..");  
        return "{'module':'user delete'}";  
    }  
}

@Controller  
public class BookController {  
  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("book save ..");  
        return "{'module':'book module'}";  
    }  
}
```

- 创建`SpringMvcConfig`配置类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
public class SpringMvcConfig {  
}
```

- 创建`ServletContainersInitConfig`类，初始化web容器
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[0];  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
}
```

- 直接启动Tomcat服务器，会报错![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f827acf2e00669687f0aaa5121ff518b.png)

从错误信息可以看出:

- `UserController`有一个save方法，访问路径为`http://localhost/save`
- `BookController`也有一个save方法，访问路径为`http://localhost/save`
- 当访问`http://localhost/save`的时候，到底是访问`UserController`还是`BookController`?
#### 4.1.2 问题分析

团队多人开发，每人设置不同的请求路径，冲突问题该如何解决?

- 解决思路:为不同模块设置模块名作为请求路径前置
    - 对于Book模块的save,将其访问路径设置`http://localhost/book/save`
    - 对于User模块的save,将其访问路径设置`http://localhost/user/save`

这样在同一个模块中出现命名冲突的情况就比较少了。
#### 4.1.3 设置映射路径

- 修改Controller
```java
@Controller  
public class UserController {  
  
    @RequestMapping("/user/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("user save ..");  
        return "{'module':'user save'}";  
    }  
  
    @RequestMapping("/user/delete")  
    @ResponseBody  
    public String delete(){  
        System.out.println("user delete ..");  
        return "{'module':'user delete'}";  
    }  
}

@Controller  
public class BookController {  
  
    @RequestMapping("/book/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("book save ..");  
        return "{'module':'book module'}";  
    }  
}
```
问题是解决了，但是每个方法前面都需要进行修改，写起来比较麻烦而且还有很多重复代码，如果/user后期发生变化，所有的方法都需要改，耦合度太高。

- 优化路径配置
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("user save ..");  
        return "{'module':'user save'}";  
    }  
  
    @RequestMapping("/delete")  
    @ResponseBody  
    public String delete(){  
        System.out.println("user delete ..");  
        return "{'module':'user delete'}";  
    }  
}

@Controller  
@RequestMapping("/book")  
public class BookController {  
  
    @RequestMapping("/save")s  
    @ResponseBody  
    public String save(){  
        System.out.println("book save ..");  
        return "{'module':'book module'}";  
    }  
}
```

注意:

- 当类上和方法上都添加了`@RequestMapping`注解，前端发送请求的时候，要和两个注解的value值相加匹配才能访问到。
- `@RequestMapping`注解value属性前面加不加`/`都可以
### 4.2 请求参数

请求路径设置好后，只要确保页面发送请求地址和后台Controller类中配置的路径一致，就可以接收到前端的请求，接收到请求后，如何接收页面传递的参数?

关于请求参数的传递与接收是和请求方式有关系的，目前比较常见的两种请求方式为：

- `GET`
- `POST`

针对于不同的请求前端如何发送，后端如何接收?
#### 4.2.1 环境准备

- 继续使用上面的环境即可，编写两个模型类`User`类和`Address`类
```java
@Data
public class User {  
    private String name;  
    private int age;   
}

@Data
public class Address {  
    private String province;  
    private String city;
}
```

- 同时修改一下`UserController`类
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(String name){  
        System.out.println("普通参数传递name --> " + name);  
        return "{'module':'commonParam'}";  
    }  
}
```
#### 4.2.2 参数传递

- `GET发送单个参数`
- 启动Tomcat服务器，发送请求与参数：
- http://localhost/commonParam?name=Jerry
- 接收参数
```java
@Controller  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(String name){  
        System.out.println("普通参数传递name --> " + name);  
        return "{'module':'commonParam'}";  
    }  
}
```
注意get请求的key需与commonParam中的形参名一致  
控制台输出`普通参数传递name --> Jerry`

- `GET发送多个参数`
- 发送请求与参数：
- localhost:8080/user/commonParam?name=Jerry&age=18
- 接收参数
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(String name,int age){  
        System.out.println("普通参数传递name --> " + name);  
        System.out.println("普通参数传递age --> " + age);  
        return "{'module':'commonParam'}";  
    }  
}
```

- 控制台输出![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2076e0a5d77e1f2e696850c3e5f96a08.png)



- `POST发送参数`![[![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/70666cc327f88685331b1341d63cfe24.png)


- 接收参数，和GET一致，不用做任何修改
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(String name,int age){  
        System.out.println("普通参数传递name --> " + name);  
        System.out.println("普通参数传递age --> " + age);  
        return "{'module':'commonParam'}";  
    }  
}
```

- 控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84b649334d840895a7d656ef0ea15545.png)


- `POST请求中文乱码 ` 
    如果我们在发送post请求的时候，使用了中文，则会出现乱码
    
- 解决方案：配置过滤器
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[0];  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    //处理乱码问题  
    @Override  
    protected Filter[] getServletFilters() {  
        CharacterEncodingFilter filter = new CharacterEncodingFilter();  
        filter.setEncoding("utf-8");  
        return new Filter[]{filter};  
    }  
}
```

- 重启Tomcat服务器，并发送post请求，使用中文，控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/05974521cf3bd0358499b463e6681d29.png)


### 4.3 五种类型参数传递

前面我们已经能够使用GET或POST来发送请求和数据，所携带的数据都是比较简单的数据，接下来在这个基础上，我们来研究一些比较复杂的参数传递，常见的参数种类有

- 普通类型
- POJO类型参数
- 嵌套POJO类型参数
- 数组类型参数
- 集合类型参数

下面我们就来挨个学习这五种类型参数如何发送，后台如何接收
#### 4.3.1 普通类型

普通参数：url地址传参，地址参数名与形参变量名相同，定义形参即可接收参数。

- 发送请求与参数：`localhost:8080/user/commonParam?name=Helsing&age=1024`
- 后台接收参数
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(String name,int age){  
        System.out.println("普通参数传递name --> " + name);  
        System.out.println("普通参数传递age --> " + age);  
        return "{'module':'commonParam'}";  
    }  
}
```

如果形参与地址参数名不一致该如何解决?例如地址参数名为`username`，而形参变量名为`name`，因为前端给的是`username`，后台接收使用的是`name`,两个名称对不上，会导致接收数据失败

- 解决方案：使用`@RequestParam`注解
    - 发送请求与参数：`localhost:8080/user/commonParam?username=Helsing&age=1024`
    - 后台接收参数
```java
@Controller  
@RequestMapping("/user")  
public class UserController {  
  
    @RequestMapping("/commonParam")  
    @ResponseBody  
    public String commonParam(@RequestParam("username") String name, int age){  
        System.out.println("普通参数传递name --> " + name);  
        System.out.println("普通参数传递age --> " + age);  
        return "{'module':'commonParam'}";  
    }  
}
```
#### 4.3.2 POJO数据类型

简单数据类型一般处理的是参数个数比较少的请求，如果参数比较多，那么后台接收参数的时候就比较复杂，这个时候我们可以考虑使用POJO数据类型。

- POJO参数：请求参数名与形参对象属性名相同，定义POJO类型形参即可接收参数

此时需要使用前面准备好的两个POJO类
```java
@Data
public class User {  
    private String name;  
    private int age;   
}

@Data
public class Address {  
    private String province;  
    private String city;
}
```

- 发送请求和参数：`localhost:8080/user/pojoParam?name=Helsing&age=1024`
- 后台接收数据
```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递  
@RequestMapping("/pojoParam")  
@ResponseBody  
public String pojoParam(User user){  
    System.out.println("POJO参数传递user --> " + user);  
    return "{'module':'pojo param'}";  
}
```

- 注意:
    - POJO参数接收，前端GET和POST发送请求数据的方式不变。
    - 请求参数key的名称要和POJO中属性的名称一致，否则无法封装。
#### 4.3.3 嵌套POJO类型

- 环境准备  
	我们先将之前写的Address类，嵌套在User类中
```java
@Data
public class User {  
    private String name;  
    private int age;  

    private Address address;  
}
```

- 嵌套POJO参数：请求参数名与形参对象属性名相同，按照对象层次结构关系即可接收嵌套POJO属性参数
    
- 发送请求和参数：`localhost:8080/user/pojoParam?name=Helsing&age=1024&address.province=Beijing&address.city=Beijing`
```java
@RequestMapping("/pojoParam")  
@ResponseBody  
public String pojoParam(User user){  
    System.out.println("POJO参数传递user --> " + user);  
    return "{'module':'pojo param'}";  
}
```

- `注意：请求参数key的名称要和POJO中属性的名称一致，否则无法封装`
#### 4.3.4 数组类型

举个简单的例子，如果前端需要获取用户的爱好，爱好绝大多数情况下都是多选，如何发送请求数据和接收数据呢?

- 数组参数：请求参数名与形参对象属性名相同且请求参数为多个，定义数组类型即可接收参数

- 发送请求和参数：`localhost:8080/user/arrayParam?hobbies=sing&hobbies=jump&hobbies=rap&hobbies=basketball`
- 后台接收参数
```java
@RequestMapping("/arrayParam")  
@ResponseBody  
public String arrayParam(String[] hobbies){  
    System.out.println("数组参数传递user --> " + Arrays.toString(hobbies));  
    return "{'module':'array param'}";  
}
```
#### 4.3.5 集合类型

数组能接收多个值，那么集合是否也可以实现这个功能呢?

- 发送请求和参数：`localhost:8080/user/listParam?hobbies=sing&hobbies=jump&hobbies=rap&hobbies=basketball`
- 后台接收参数
```java
@RequestMapping("/listParam")  
@ResponseBody  
public String listParam(List hobbies) {  
    System.out.println("集合参数传递user --> " + hobbies);  
    return "{'module':'list param'}";  
}
```

- 运行程序，报错`java.lang.IllegalArgumentException: Cannot generate variable name for non-typed Collection parameter type`
    - 错误原因：SpringMVC将List看做是一个POJO对象来处理，将其创建一个对象并准备把前端的数据封装到对象中，但是List是一个接口无法创建对象，所以报错。
- 解决方案是：使用`@RequestParam`注解
```java
@RequestMapping("/listParam")  
@ResponseBody  
public String listParam(@RequestParam List hobbies) {  
    System.out.println("集合参数传递user --> " + hobbies);  
    return "{'module':'list param'}";  
}
```

知识点：`@RequestParam`

|名称|@RequestParam|
|---|---|
|类型|形参注解|
|位置|SpringMVC控制器方法形参定义前面|
|作用|绑定请求参数与处理器方法形参间的关系|
|相关参数|required：是否为必传参数  <br>defaultValue：参数默认值|
### 4.4 JSON数据参数参数

现在比较流行的开发方式为异步调用。前后台以异步方式进行交换，传输的数据使用的是JSON，所以前端如果发送的是JSON数据，后端该如何接收?

对于JSON数据类型，我们常见的有三种:

- json普通数组（`[“value1”,”value2”,”value3”,…]`）
- json对象（`{key1:value1,key2:value2,…}`）
- json对象数组（`[{key1:value1,…},{key2:value2,…}]`）

下面我们就来学习以上三种数据类型，前端如何发送，后端如何接收
#### 4.4.1 JSON普通数组

- `步骤一：`导入坐标
```xml
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>2.9.0</version>  
</dependency>
```

- `步骤二：`开启SpringMVC注解支持  
	使用`@EnableWebMvc`，在SpringMVC的配置类中开启SpringMVC的注解支持，这里面就包含了将JSON转换成对象的功能。
```java
@Configuration  
@ComponentScan("com.blog.controller")  
//开启json数据类型自动转换  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- `步骤三：`PostMan发送JSON数据![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d75362a8c19544eaf25c9eafb7e74e2.png)


- `步骤四：`后台接收参数，参数前添加`@RequestBody`  
	使用`@RequestBody`注解将外部传递的json数组数据映射到形参的集合对象中作为数据
```java
@RequestMapping("/jsonArrayParam")  
@ResponseBody  
public String jsonArrayParam(@RequestBody List<String> hobbies) {  
    System.out.println("JSON数组参数传递hobbies --> " + hobbies);  
    return "{'module':'json array param'}";  
}
```
#### 4.4.2 JSON对象

- 请求和数据的发送:
```json
{  
    "name":"菲茨罗伊",  
    "age":"27",  
    "address":{  
        "city":"萨尔沃",  
        "province":"外域"  
    }  
}
```

- 接收请求和参数
```java
@RequestMapping("/jsonPojoParam")  
@ResponseBody  
public String jsonPojoParam(@RequestBody User user) {  
    System.out.println("JSON对象参数传递user --> " + user);  
    return "{'module':'json pojo param'}";  
}
```
#### 4.4.3 JSON对象数组

- 发送请求和数据
```json
[  
    {  
        "name":"菲茨罗伊",  
        "age":"27",  
        "address":{  
            "city":"萨尔沃",  
            "province":"外域"  
        }  
    },  
    {  
        "name":"地平线",  
        "age":"136",  
        "address":{  
            "city":"奥林匹斯",  
            "province":"外域"  
        }  
    }  
]
```

- 接收请求和参数
```java
@RequestMapping("/jsonPojoListParam")  
@ResponseBody  
public String jsonPojoListParam(@RequestBody List<User> users) {  
    System.out.println("JSON对象数组参数传递user --> " + users);  
    return "{'module':'json pojo list param'}";  
}
```
#### 4.4.4 小结

SpringMVC接收JSON数据的实现步骤为:

1. 导入jackson包
2. 开启SpringMVC注解驱动，在配置类上添加`@EnableWebMvc`注解
3. 使用PostMan发送JSON数据
4. Controller方法的参数前添加`@RequestBody`注解

知识点1：`@EnableWebMvc`

|名称|@EnableWebMvc|
|---|---|
|类型|配置类注解|
|位置|SpringMVC配置类定义上方|
|作用|开启SpringMVC多项辅助功能|

知识点2：`@RequestBody`

|名称|@RequestBody|
|---|---|
|类型|形参注解|
|位置|SpringMVC控制器方法形参定义前面|
|作用|将请求中请求体所包含的数据传递给请求参数，此注解一个处理器方法只能使用一次|

`@RequestBody`与`@RequestParam`区别

- 区别
    - `@RequestParam`用于接收url地址传参，表单传参【application/x-www-form-urlencoded】
    - `@RequestBody`用于接收json数据【application/json】
- 应用
    - 后期开发中，发送json格式数据为主，`@RequestBody`应用较广
    - 如果发送非json格式数据，选用`@RequestParam`接收请求参数
### 4.5 日期类型参数传递

日期类型比较特殊，因为对于日期的格式有N多中输入方式，比如

- 2088-08-18
- 2088/08/18
- 08/18/2088
- …

针对这么多日期格式，SpringMVC该如何接收呢？下面我们来开始学习

- `步骤一：`编写方法接收日期数据
```java
@RequestMapping("/dateParam")  
@ResponseBody  
public String dateParam(Date date) {  
    System.out.println("参数传递date --> " + date);  
    return "{'module':'date param'}";  
}
```

- `步骤二：`启动Tomcat服务器
- `步骤三：`使用PostMan发送请求：`localhost:8080/user/dateParam?date=2077/12/21`

- `步骤四：`查看控制台，输出如下![[Pasted image 20230831031358.png]]
- `步骤五：`更换日期格式  
	为了能更好的看到程序运行的结果，我们在方法中多添加一个日期参数
```java
@RequestMapping("/dateParam")  
@ResponseBody  
public String dateParam(Date date1,Date date2) {  
    System.out.println("参数传递date1 --> " + date1);  
    System.out.println("参数传递date2 --> " + date2);  
    return "{'module':'date param'}";  
}
```

使用PostMan发送请求，携带两个不同的日期格式，`localhost:8080/user/dateParam?date1=2077/12/21&date2=1997-02-13`

发送请求和数据后，页面会报400，`The request sent by the client was syntactically incorrect.`  

错误的原因是将`1997-02-13`转换成日期类型的时候失败了，原因是SpringMVC默认支持的字符串转日期的格式为`yyyy/MM/dd`,而我们现在传递的不符合其默认格式，SpringMVC就无法进行格式转换，所以报错。  
解决方案也比较简单，需要使用`@DateTimeFormat`注解

```java
@RequestMapping("/dateParam")  
@ResponseBody  
public String dateParam(Date date1,@DateTimeFormat(pattern = "yyyy-MM-dd") Date date2) {  
    System.out.println("参数传递date1 --> " + date1);  
    System.out.println("参数传递date2 --> " + date2);  
    return "{'module':'date param'}";  
}
```

- `步骤六：`携带具体时间的日期  
接下来我们再来发送一个携带具体时间的日期，如`localhost:8080/user/dateParam?date1=2077/12/21&date2=1997-02-13&date3=2022/09/09 16:34:07`，那么SpringMVC该怎么处理呢？  
继续修改UserController类，添加第三个参数，同时使用`@DateTimeFormat`来设置日期格式

```java
@RequestMapping("/dateParam")  
@ResponseBody  
public String dateParam(Date date1,  
                        @DateTimeFormat(pattern = "yyyy-MM-dd") Date date2,  
                        @DateTimeFormat(pattern ="yyyy/MM/dd HH:mm:ss") Date date3) {  
    System.out.println("参数传递date1 --> " + date1);  
    System.out.println("参数传递date2 --> " + date2);  
    System.out.println("参数传递date3 --> " + date3);  
    return "{'module':'date param'}";  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fa1f916a439c7e740b057eabb6727941.png)



知识点：`@DateTimeFormat`

|名称|@DateTimeFormat|
|---|---|
|类型|形参注解|
|位置|SpringMVC控制器方法形参前面|
|作用|设定日期时间型数据格式|
|相关属性|pattern：指定日期时间格式字符串|
#### 4.5.1 内部实现原理

我们首先先来思考一个问题:

- 前端传递字符串，后端使用日期Date接收
- 前端传递JSON数据，后端使用对象接收
- 前端传递字符串，后端使用Integer接收
- 后台需要的数据类型有很多种
- 在数据的传递过程中存在很多类型的转换

`问`:谁来做这个类型转换?

- `答`:SpringMVC

`问`:SpringMVC是如何实现类型转换的?

- `答`:SpringMVC中提供了很多类型转换接口和实现类

在框架中，有一些类型转换接口，其中有:

1. `Converter`接口  
	注意：Converter所属的包为`org.springframework.core.convert.converter`
```java
/**  
*    S: the source type  
*    T: the target type  
*/  
@FunctionalInterface  
public interface Converter<S, T> {  
    @Nullable  
    //该方法就是将从页面上接收的数据(S)转换成我们想要的数据类型(T)返回  
    T convert(S source);  
}
```

到了源码页面我们按Ctrl + H可以来看看`Converter`接口的层次结构  
这里给我们提供了很多对应`Converter`接口的实现类，用来实现不同数据类型之间的转换![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/49b88998bebaa69ff521f161b2648995.png)


2.  `HttpMessageConverter`接口  
    该接口是实现对象与JSON之间的转换工作  
    注意：需要在SpringMVC的配置类把`@EnableWebMvc`当做标配配置上去，不要省略
### 4.6 响应

SpringMVC接收到请求和数据后，进行一些了的处理，当然这个处理可以是转发给Service，Service层再调用Dao层完成的，不管怎样，处理完以后，都需要将结果告知给用户。

比如：根据用户ID查询用户信息、查询用户列表、新增用户等。  
对于响应，主要就包含两部分内容：

- 响应页面
- 响应数据
    - 文本数据
    - json数据

因为异步调用是目前常用的主流方式，所以我们需要更关注的就是如何返回JSON数据，对于其他只需要认识了解即可。
#### 4.6.1 环境准备

在之前的环境上加点东西就可以了

- 在webapp下新建`page.jsp`
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<html>  
<head>  
    <title>Title</title>  
</head>  
<body>  
HELLO SPRING MVC!!  
</body>  
</html>
```

- 修改`UserController`
```java
@Controller  
public class UserController {  
  
}
```
#### 4.6.2 响应页面（了解）

- `步骤一：`设置返回页面
```java
@Controller  
public class UserController {  
    @RequestMapping("/toJumpPage")  
    //注意  
    //1.此处不能添加@ResponseBody,如果加了该注入，会直接将page.jsp当字符串返回前端  
    //2.方法需要返回String  
    public String toJumpPage(){  
        System.out.println("跳转页面");  
        return "page.jsp";  
    }  
}
```

- `步骤二：`启动程序测试  
    打开浏览器，访问`http://localhost:8080/toJumpPage`  
    将跳转到`page.jsp`页面，并展示`page.jsp`页面的内容
#### 4.6.3 返回文本数据（了解）

`步骤一：`设置返回文本内容
```java
@RequestMapping("toText")  
//此时就需要添加@ResponseBody，将`response text`当成字符串返回给前端  
//如果不写@ResponseBody，则会将response text当成页面名去寻找，找不到报404  
@ResponseBody  
public String toText(){  
    System.out.println("返回纯文本数据");  
    return "response text";  
}
```

- `步骤二：`启动程序测试  
    浏览器访问`http://localhost:8080/toText`  
    页面上出现`response text`文本数据
#### 4.6.4 响应JSON数据

- 响应POJO对象
```java
@RequestMapping("toJsonPojo")  
@ResponseBody  
public User toJsonPojo(){  
    System.out.println("返回json对象数据");  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(9527);  
    return user;  
}
```
返回值为实体类对象，设置返回值为实体类类型，即可实现返回对应对象的json数据，需要依赖`@ResponseBody`注解和`@EnableWebMvc`注解

- 访问`http://localhost:8080/toJsonPojo`，页面上成功出现JSON类型数据![[Pasted image 20230831093635.png]]
- `HttpMessageConverter`接口帮我们实现了对象与JSON之间的转换工作，我们只需要在`SpringMvcConfig`配置类上加上`@EnableWebMvc`注解即可

- 响应POJO集合对象
```java
@RequestMapping("toJsonList")  
@ResponseBody  
public List<User> toJsonList(){  
    List<User> users = new ArrayList<User>();  
  
    User user1 = new User();  
    user1.setName("马文");  
    user1.setAge(27);  
    users.add(user1);  
  
    User user2 = new User();  
    user2.setName("马武");  
    user2.setAge(28);  
    users.add(user2);  
  
    return users;  
}
```

- 访问`http://localhost:8080/toJsonList`，页面上成功出现JSON集合类型数据![[Pasted image 20230831093711.png]]
知识点：`@ResponseBody`

|名称|@ResponseBody|
|---|---|
|类型|方法\类注解|
|位置|SpringMVC控制器方法定义上方和控制类上|
|作用|设置当前控制器返回值作为响应体,  <br>写在类上，该类的所有方法都有该注解功能|
|相关属性|pattern：指定日期时间格式字符串|

说明:

- 该注解可以写在类上或者方法上
- 写在类上就是该类下的所有方法都有`@ReponseBody`功能
- 当方法上有`@ReponseBody`注解后
    - 方法的返回值为字符串，会将其作为文本内容直接响应给前端
    - 方法的返回值为对象，会将对象转换成JSON响应给前端

此处又使用到了类型转换，内部还是通过`HttpMessageConverter`接口完成的，所以`Converter`除了前面所说的功能外，它还可以实现:

- 对象转Json数据(POJO -> json)
- 集合转Json数据(Collection -> json)

## 5 RESTFul风格

### 5.1 REST简介

REST，表现形式状态转换，它是一种软件架构`风格`  
当我们想表示一个网络资源时，可以使用两种方式：

- 传统风格资源描述形式
    - `http://localhost/user/getById?id=1` 查询id为1的用户信息
    - `http://localhost/user/saveUser` 保存用户信息
- REST风格描述形式
    - `http://localhost/user/1`
    - `http://localhost/user`

传统方式一般是一个请求url对应一种操作，这样做不仅麻烦，而且也不安全，通过请求的`URL`地址，就大致能推测出该`URL`实现的是什么操作  
反观REST风格的描述，请求地址变简洁了，而且只看请求`URL`并不很容易能猜出来该`UR`L的具体功能

所以`REST`的优点有：

- 隐藏资源的访问行为，无法通过地址得知该资源是何种操作
- 书写简化

那么问题也随之而来，一个相同的`URL`地址既可以是增加操作，也可以是修改或者查询，那么我们该如何区分该请求到底是什么操作呢？

- 按照REST风格访问资源时，使用`行为动作`区分对资源进行了何种操作
    - `http://localhost/users` 查询全部用户信息 `GET`（查询）
    - `http://localhost/users/1` 查询指定用户信息 `GET`（查询）
    - `http://localhost/users` 添加用户信息 `POST`（新增/保存）
    - `http://localhost/users` 修改用户信息 `PUT`（修改/更新）
    - `http://localhost/users/1` 删除用户信息 `DELETE`（删除）

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78ce771547b2f34345178fb2020a5284.png)



搞清楚了什么是REST分各个后，后面会经常提到一个概念叫`RESTful`，那么什么是`RESTful`呢？

- 根据REST风格对资源进行访问称为`RESTful`

在我们后期的开发过程中，大多数都是遵循`REST`风格来访问我们的后台服务。
### 5.2 RESTful入门案例

#### 5.2.1 环境准备

- 新建一个web的maven项目
- 导入坐标
```xml
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>2.9.0</version>  
</dependency>
```

- 创建对应的配置类
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[0];  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    //乱码处理  
    @Override  
    protected Filter[] getServletFilters() {  
        CharacterEncodingFilter filter = new CharacterEncodingFilter();  
        filter.setEncoding("utf-8");  
        return new Filter[]{filter};  
    }  
}

@Configuration  
@ComponentScan("com.blog.controller")  
//开启JSON数据类型自动转换  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- 编写模型类User
```java
@Data
public class User {  
    private String name;  
    private int age;
}
```

- 编写`UserController`
```java
@Controller  
public class UserController {  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(@RequestBody User user){  
        System.out.println("user save ..." + user);  
        return "{'module':'user save'}";  
    }  
    @RequestMapping("/delete")  
    @ResponseBody  
    public String delete(Integer id){  
        System.out.println("user delete ..." + id);  
        return "{'module':'user delete'}";  
    }  
    @RequestMapping("/update")  
    @ResponseBody  
    public String update(@RequestBody User user){  
        System.out.println("user update ..." + user);  
        return "{'module':'user update'}";  
    }  
    @RequestMapping("/getById")  
    @ResponseBody  
    public String getById(Integer id){  
        System.out.println("user getById ..." + id);  
        return "{'module':'user getById'}";  
    }  
    @RequestMapping("/getAll")  
    @ResponseBody  
    public String getAll(){  
        System.out.println("user getAll ...");  
        return "{'module':'user getAll'}";  
    }  
}
```
#### 5.2.2 思路分析

需求:将之前的增删改查替换成`RESTful`的开发方式。

1. 之前不同的请求有不同的路径,现在要将其修改为统一的请求路径
    - 修改前: 新增：`/save`，修改: `/update`，删除 `/delete`..
    - 修改后: 增删改查：`/users`
2. 根据`GET`查询、`POST`新增、`PUT`修改、`DELETE`删除对方法的请求方式进行限定
3. 发送请求的过程中如何设置请求参数?
#### 5.2.3 修改RESTful风格

- `新增`  
	将请求路径更改为`/users`，并设置当前请求方法为`POST`
```java
@RequestMapping(value = "/users", method = RequestMethod.POST)  
@ResponseBody  
public String save(@RequestBody User user) {  
    System.out.println("user save ..." + user);  
    return "{'module':'user save'}";  
}
```

使用method属性限定该方法的访问方式为`POST`，如果使用`GET`请求将报错  
发送POST请求与参数：
```json
{  
    "name":"菲茨罗伊",  
    "age":"27"  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6435a9592a13aad7d15b62c6a1fbb910.png)


- `删除`  
	将请求路径更改为`/users`，并设置当前请求方法为`DELETE`
```java
@RequestMapping(value = "/delete",method = RequestMethod.DELETE)  
@ResponseBody  
public String delete(Integer id){  
    System.out.println("user delete ..." + id);  
    return "{'module':'user delete'}";  
}
```

- 但是现在的删除方法没有携带所要删除数据的id，所以针对RESTful的开发，如何携带数据参数?
- 修改@RequestMapping的value属性，将其中修改为`/users/{id}`，目的是和路径匹配
- 在方法的形参前添加`@PathVariable`注解
```java
@RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)  
@ResponseBody  
public String delete(@PathVariable Integer id){  
    System.out.println("user delete ..." + id);  
    return "{'module':'user delete'}";  
}
```

发送`DELETE`请求访问`localhost:8080/users/9421`  
控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10a385665e500115b4bca39d6e847bfd.png)


疑问：如果方法形参的名称和路径`{}`中的值不一致，该怎么办?  
例如`"/users/{id}"`和`delete(@PathVariable Integer userId)`

- 解答：如果这两个值不一致，就无法获取参数，此时我们可以在注解后面加上属性，让注解的属性值与`{}`中的值一致即可，具体代码如下
```java
@RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)  
@ResponseBody  
public String delete(@PathVariable("id") Integer userId){  
    System.out.println("user delete ..." + userId);  
    return "{'module':'user delete'}";  
}
```

疑问：如果有多个参数需要传递该如何编写?  
前端发送请求时使用`localhost:8080/users/9421/Tom`，路径中的`9421`和`Tom`就是我们想传递的两个参数

- 解答：我们在路径后面再加一个`/{name}`，同时在方法参数中增加对应属性即可
```java
@RequestMapping(value = "/users/{id}/{name}",method = RequestMethod.DELETE)  
@ResponseBody  
public String delete(@PathVariable("id") Integer userId,@PathVariable String name){  
    System.out.println("user delete ..." + userId + ":" + name);  
    return "{'module':'user delete'}";  
}
```

发送`DELETE`请求访问`localhost:8080/users/9421/Tom`  
控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1c7e0d6054e7fb0ed57df7c5ea01cc0f.png)


- `修改`  
	将请求路径更改为`/users`，并设置当前请求方法为`PUT`
```java
@RequestMapping(value = "/users",method = RequestMethod.PUT)  
@ResponseBody  
public String update(@RequestBody User user){  
    System.out.println("user update ..." + user);  
    return "{'module':'user update'}";  
}
```

发送`PUT`请求`localhost:8080/users`，访问并携带参数
```json
{  
    "name":"菲茨罗伊",  
    "age":"27"  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d58b92645c93f9cf144d36376d3fe718.png)



- `根据ID查询`  
	将请求路径更改为`/users/{id}`，并设置当前请求方法为`GET`
```java
@RequestMapping(value = "/users/{id}",method = RequestMethod.GET)  
@ResponseBody  
public String getById(@PathVariable Integer id){  
    System.out.println("user getById ..." + id);  
    return "{'module':'user getById'}";  
}
```

发送`GET`请求访问`localhost:8080/users/2077`  
控制台输出如下![[Pasted image 20230831094334.png]]
- `查询所有`  
	将请求路径更改为`/users`，并设置当前请求方法为`GET`
```java
@RequestMapping(value = "/users",method = RequestMethod.GET)  
@ResponseBody  
public String getAll(){  
    System.out.println("user getAll ...");  
    return "{'module':'user getAll'}";  
}
```

发送`GET`请求访问`localhost:8080/users`  
控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ee93ba157e8c2d915419022651e0a25.png)


- 整体代码
```java
@Controller  
public class UserController {  
    @RequestMapping(value = "/users", method = RequestMethod.POST)  
    @ResponseBody  
    public String save(@RequestBody User user) {  
        System.out.println("user save ..." + user);  
        return "{'module':'user save'}";  
    }  
  
  
    @RequestMapping(value = "/users/{id}/{name}",method = RequestMethod.DELETE)  
    @ResponseBody  
    public String delete(@PathVariable("id") Integer userId,@PathVariable String name){  
        System.out.println("user delete ..." + userId + ":" + name);  
        return "{'module':'user delete'}";  
    }  
  
    @RequestMapping(value = "/users",method = RequestMethod.PUT)  
    @ResponseBody  
    public String update(@RequestBody User user){  
        System.out.println("user update ..." + user);  
        return "{'module':'user update'}";  
    }  
  
    @RequestMapping(value = "/users/{id}",method = RequestMethod.GET)  
    @ResponseBody  
    public String getById(@PathVariable Integer id){  
        System.out.println("user getById ..." + id);  
        return "{'module':'user getById'}";  
    }  
  
    @RequestMapping(value = "/users",method = RequestMethod.GET)  
    @ResponseBody  
    public String getAll(){  
        System.out.println("user getAll ...");  
        return "{'module':'user getAll'}";  
    }  
}
```

从整体代码来看，有些臃肿，好多代码都是重复的，下一小节我们就会来解决这个问题
#### 5.2.4 小结

RESTful入门案例，我们需要记住的内容如下:

1. 设定Http请求动作(动词)
```java
@RequestMapping(value="",method = RequestMethod.POST|GET|PUT|DELETE)
```

2. 设定请求参数(路径变量)
```java
@RequestMapping(value="/users/{id}",method = RequestMethod.DELETE)  
@ReponseBody  
public String delete(@PathVariable Integer id){  
}
```

知识点：`@PathVariable`

|名称|@PathVariable|
|---|---|
|类型|形参注解|
|位置|SpringMVC控制器方法形参定义前面|
|作用|绑定路径参数与处理器方法形参间的关系，要求路径参数名与形参名一一对应|

关于接收参数，我们学过三个注解`@RequestBody`、`@RequestParam`、`@PathVariable`，这三个注解之间的区别和应用分别是什么?

- 区别
    - `@RequestParam`用于接收url地址传参或表单传参
    - `@RequestBody`用于接收JSON数据
    - `@PathVariable`用于接收路径参数，使用{参数名称}描述路径参数
- 应用
    - 后期开发中，发送请求参数超过1个时，以JSON格式为主，`@RequestBody`应用较广
    - 如果发送非JSON格式数据，选用`@RequestParam`接收请求参数
    - 采用`RESTful`进行开发，当参数数量较少时，例如1个，可以采用`@PathVariable`接收请求路径变量，通常用于传递id值
### 5.3 RESTful快速开发

做完了上面的`RESTful`的开发，就感觉好麻烦，主要体现在以下三部分

- 每个方法的`@RequestMapping`注解中都定义了访问路径`/users`，重复性太高。
    - 解决方案：将`@RequestMapping`提到类上面，用来定义所有方法共同的访问路径。
- 每个方法的`@RequestMapping`注解中都要使用method属性定义请求方式，重复性太高。
    - 解决方案：使用`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`代替
- 每个方法响应json都需要加上`@ResponseBody`注解，重复性太高。
    - 解决方案：
        - 将`@ResponseBody`提到类上面，让所有的方法都有`@ResponseBody`的功能
        - 使用`@RestController`注解替换`@Controller`与`@ResponseBody`注解，简化书写

- 修改后的代码
```java
@RestController  
@RequestMapping("/users")  
public class UserController {  
    @PostMapping  
    public String save(@RequestBody User user) {  
        System.out.println("user save ..." + user);  
        return "{'module':'user save'}";  
    }  
  
    @DeleteMapping("/{id}/{name}")  
    public String delete(@PathVariable("id") Integer userId, @PathVariable String name) {  
        System.out.println("user delete ..." + userId + ":" + name);  
        return "{'module':'user delete'}";  
    }  
  
    @PutMapping()  
    public String update(@RequestBody User user) {  
        System.out.println("user update ..." + user);  
        return "{'module':'user update'}";  
    }  
  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id) {  
        System.out.println("user getById ..." + id);  
        return "{'module':'user getById'}";  
    }  
  
    @GetMapping  
    public String getAll() {  
        System.out.println("user getAll ...");  
        return "{'module':'user getAll'}";  
    }  
}
```
### 5.4 RESTful案例

#### 5.4.1 需求分析

- 需求一：图片列表查询，从后台返回数据，将数据展示在页面上![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7769052f8f390d5096f608f1f899f2ca.png)


- 需求二：新增图片，将新增图书的数据传递到后台，并在控制台打印![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33a7a712f805432a49dbbf15dae2976d.png)


- 说明：此次案例的重点是在SpringMVC中如何使用RESTful实现前后台交互，所以本案例并没有和数据库进行交互，所有数据使用`假数据`来完成开发。

- 步骤分析:

	1. 搭建项目导入jar包
	2. 编写Controller类，提供两个方法，一个用来做列表查询，一个用来做新增
	3. 在方法上使用RESTful进行路径设置
	4. 完成请求、参数的接收和结果的响应
	5. 使用PostMan进行测试
	6. 将前端页面拷贝到项目中
	7. 页面发送ajax请求
	8. 完成页面数据的展示
#### 5.4.2 环境准备

- 创建一个web的maven项目
    
- 导入坐标
```xml
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>2.9.0</version>  
</dependency>
<!-- 一键配置Pojo类 -->  
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <version>1.18.28</version>  
</dependency>
```

- 创建对应的配置类
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[0];  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    @Override  
    protected Filter[] getServletFilters() {  
        CharacterEncodingFilter filter = new CharacterEncodingFilter();  
        filter.setEncoding("utf-8");  
        return new Filter[]{filter};  
    }  
}

@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- 编写模型类Book
```java
@Data
public class Book {  
   private Integer id;  
   private String type;  
   private String name;  
   private String description;  
}
```

- 编写BookController
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
}
```
#### 5.4.3 后台接口开发

- 编写Controller类，并使用RESTful配置
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
  
    @PostMapping  
    public String save(@RequestBody Book book){  
        System.out.println("book save --> " + book);  
        return "{'module':'book save success'}";  
    }  
  
    @GetMapping  
    public List<Book> getAll(){  
        System.out.println("book getAll running ..");  
        ArrayList<Book> bookList = new ArrayList<Book>();  
  
        Book book1 = new Book();  
        book1.setType("计算机");  
        book1.setName("SpringMVC入门教程");  
        book1.setDescription("小试牛刀");  
        bookList.add(book1);  
  
        Book book2 = new Book();  
        book2.setType("计算机");  
        book2.setName("SpringMVC实战教程");  
        book2.setDescription("一代宗师");  
        bookList.add(book2);  
  
        Book book3 = new Book();  
        book3.setType("计算机丛书");  
        book3.setName("SpringMVC实战教程进阶");  
        book3.setDescription("一代宗师呕心创作");  
        bookList.add(book3);  
          
        return bookList;  
    }  
}
```

- 使用PostMan进行测试

- 测试新增
```json
{  
    "type":"计算机",  
    "name":"SpringMVC入门教程",  
    "description":"小试牛刀"  
}
```

访问`localhost:8080/books`，发送POST请求与数据，控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/017c08e29af9821f43b8b45fcd7fe837.png)


- 测试查询  
	访问`localhost:8080/books`，发送GET请求，查询到如下内容
```json
[  
    {  
        "id": null,  
        "type": "计算机",  
        "name": "SpringMVC入门教程",  
        "description": "小试牛刀"  
    },  
    {  
        "id": null,  
        "type": "计算机",  
        "name": "SpringMVC实战教程",  
        "description": "一代宗师"  
    },  
    {  
        "id": null,  
        "type": "计算机丛书",  
        "name": "SpringMVC实战教程进阶",  
        "description": "一代宗师呕心创作"  
    }  
]
```

- 控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f75bdb528dc947ad3af75bbc29105e70.png)


#### 5.4.4 页面访问处理

- `步骤一：`导入提供好的静态页面
- `步骤二：`访问pages目录下的books.html
    
    - 打开浏览器访问`http://localhost:8080/pages/book.html`，报404，为什么呢？
    - SpringMvcConfig拦截了所有资源路径
```java
protected String[] getServletMappings() {  
    return new String[]{"/"};  
}
```

- 解决方案：SpringMVC需要将静态资源进行放行
```java
@Configuration  
public class SpringMvcSupport extends WebMvcConfigurationSupport {  
    //设置静态资源访问过滤，当前类需要设置为配置类，并被扫描加载  
    @Override  
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {  
        //当访问/pages/xxx时候，从/pages目录下查找内容  
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");  
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");  
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");  
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");  
    }  
}
```

- 该配置类是在config目录下，SpringMVC扫描的是controller包，所以该配置类还未生效，要想生效需要将SpringMvcConfig配置类进行修改
```java
@Configuration  
//将我们刚刚写的配置类也扫描进去  
@ComponentScan({"com.blog.controller","com.blog.config"})  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- 步骤三：修改books.html页面
```html
<!DOCTYPE html>  
  
<html>  
<head>  
    <!-- 页面meta -->  
    <meta charset="utf-8">  
    <title>SpringMVC案例</title>  
    <!-- 引入样式 -->  
    <link rel="stylesheet" href="../plugins/elementui/index.css">  
    <link rel="stylesheet" href="../plugins/font-awesome/css/font-awesome.min.css">  
    <link rel="stylesheet" href="../css/style.css">  
</head>  
  
<body class="hold-transition">  
  
<div id="app">  
  
    <div class="content-header">  
        <h1>图书管理</h1>  
    </div>  
  
    <div class="app-container">  
        <div class="box">  
            <div class="filter-container">  
                <el-input placeholder="图书名称" style="width: 200px;" class="filter-item"></el-input>  
                <el-button class="dalfBut">查询</el-button>  
                <el-button type="primary" class="butT" @click="openSave()">新建</el-button>  
            </div>  
  
            <el-table size="small" current-row-key="id" :data="dataList" stripe highlight-current-row>  
                <el-table-column type="index" align="center" label="序号"></el-table-column>  
                <el-table-column prop="type" label="图书类别" align="center"></el-table-column>  
                <el-table-column prop="name" label="图书名称" align="center"></el-table-column>  
                <el-table-column prop="description" label="描述" align="center"></el-table-column>  
                <el-table-column label="操作" align="center">  
                    <template slot-scope="scope">  
                        <el-button type="primary" size="mini">编辑</el-button>  
                        <el-button size="mini" type="danger">删除</el-button>  
                    </template>  
                </el-table-column>  
            </el-table>  
  
            <div class="pagination-container">  
                <el-pagination  
                        class="pagiantion"  
                        @current-change="handleCurrentChange"  
                        :current-page="pagination.currentPage"  
                        :page-size="pagination.pageSize"  
                        layout="total, prev, pager, next, jumper"  
                        :total="pagination.total">  
                </el-pagination>  
            </div>  
  
            <!-- 新增标签弹层 -->  
            <div class="add-form">  
                <el-dialog title="新增图书" :visible.sync="dialogFormVisible">  
                    <el-form ref="dataAddForm" :model="formData" :rules="rules" label-position="right"  
                             label-width="100px">  
                        <el-row>  
                            <el-col :span="12">  
                                <el-form-item label="图书类别" prop="type">  
                                    <el-input v-model="formData.type"/>  
                                </el-form-item>  
                            </el-col>  
                            <el-col :span="12">  
                                <el-form-item label="图书名称" prop="name">  
                                    <el-input v-model="formData.name"/>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                        <el-row>  
                            <el-col :span="24">  
                                <el-form-item label="描述">  
                                    <el-input v-model="formData.description" type="textarea"></el-input>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                    </el-form>  
                    <div slot="footer" class="dialog-footer">  
                        <el-button @click="dialogFormVisible = false">取消</el-button>  
                        <el-button type="primary" @click="saveBook()">确定</el-button>  
                    </div>  
                </el-dialog>  
            </div>  
  
        </div>  
    </div>  
</div>  
</body>  
  
<!-- 引入组件库 -->  
<script src="../js/vue.js"></script>  
<script src="../plugins/elementui/index.js"></script>  
<script type="text/javascript" src="../js/jquery.min.js"></script>  
<script src="../js/axios-0.18.0.js"></script>  
  
<script>  
    var vue = new Vue({  
  
        el: '#app',  
  
        data: {  
            dataList: [],//当前页要展示的分页列表数据  
            formData: {},//表单数据  
            dialogFormVisible: false,//增加表单是否可见  
            dialogFormVisible4Edit: false,//编辑表单是否可见  
            pagination: {},//分页模型数据，暂时弃用  
        },  
  
        //钩子函数，VUE对象初始化完成后自动执行  
        created() {  
            this.getAll();  
        },  
  
        methods: {  
            // 重置表单  
            resetForm() {  
                //清空输入框  
                this.formData = {};  
            },  
  
            // 弹出添加窗口  
            openSave() {  
                this.dialogFormVisible = true;  
                this.resetForm();  
            },  
  
            //添加  
            saveBook() {  
                axios.post("/books", this.formData).then((res) => {  
  
                });  
            },  
  
            //主页列表查询  
            getAll() {  
                axios.get("/books").then((res) => {  
                    this.dataList = res.data;  
                });  
            },  
  
        }  
    })  
</script>  
</html>
```

- 由于按钮的事件都绑定好了，所以我们重点只关注`saveBook()`和`getAll()`这两个函数就行

- 当调用`getAll()`函数时，我们需要将响应的JSON数据赋给dataList即可，而响应JSON数据我们在上一小节已经完成了
- 当调用`saveBook()`函数时，发送POST请求，并将formData的数据响应给后台，我们这里的操作只是在控制台输出了一下