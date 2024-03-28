# 目录

- [八、IOC/DI注解开发](#八、IOC/DI注解开发)
	- [1、环境准备](#1%E3%80%81%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
	- [2、注解开发定义bean](#2%E3%80%81%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91%E5%AE%9A%E4%B9%89bean)
	- [3、纯注解开发模式](#3%E3%80%81%E7%BA%AF%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F)
		- [思路分析](#%E6%80%9D%E8%B7%AF%E5%88%86%E6%9E%90)
		- [实现步骤](#%E5%AE%9E%E7%8E%B0%E6%AD%A5%E9%AA%A4)
		- [小结](#%E5%B0%8F%E7%BB%93)
	- [4、注解开发bean的作用范围和生命周期](#4%E3%80%81%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91bean%E7%9A%84%E4%BD%9C%E7%94%A8%E8%8C%83%E5%9B%B4%E5%92%8C%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
		- [bean的作用范围](#bean%E7%9A%84%E4%BD%9C%E7%94%A8%E8%8C%83%E5%9B%B4)
		- [bean的生命周期](#bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
		- [小结](#%E5%B0%8F%E7%BB%93)
	- [5、注解开发依赖注入](#5%E3%80%81%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)
		- [环境准备](#%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
		- [注解实现按照类型注入](#%E6%B3%A8%E8%A7%A3%E5%AE%9E%E7%8E%B0%E6%8C%89%E7%85%A7%E7%B1%BB%E5%9E%8B%E6%B3%A8%E5%85%A5)
		- [注解实现只能找名称注入](#%E6%B3%A8%E8%A7%A3%E5%AE%9E%E7%8E%B0%E5%8F%AA%E8%83%BD%E6%89%BE%E5%90%8D%E7%A7%B0%E6%B3%A8%E5%85%A5)
		- [简单数据类型注入](#%E7%AE%80%E5%8D%95%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E6%B3%A8%E5%85%A5)
		- [注解读取properties配置文件](#%E6%B3%A8%E8%A7%A3%E8%AF%BB%E5%8F%96properties%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

- [九、IOC/DI注解开发管理第三方bean](#九、IOC/DI注解开发管理第三方bean)
	- [1、环境准备](#1%E3%80%81%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
	- [2、注解开发管理第三方bean](#2%E3%80%81%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91%E7%AE%A1%E7%90%86%E7%AC%AC%E4%B8%89%E6%96%B9bean)
	- [3、引入外部配置类](#3%E3%80%81%E5%BC%95%E5%85%A5%E5%A4%96%E9%83%A8%E9%85%8D%E7%BD%AE%E7%B1%BB)
		- [使用包扫描引入(不推荐)](#%E4%BD%BF%E7%94%A8%E5%8C%85%E6%89%AB%E6%8F%8F%E5%BC%95%E5%85%A5(%E4%B8%8D%E6%8E%A8%E8%8D%90))
		- [使用@Import引入](#%E4%BD%BF%E7%94%A8@Import%E5%BC%95%E5%85%A5)
	- [4、注解开发实现为第三方bean注入资源](#4%E3%80%81%E6%B3%A8%E8%A7%A3%E5%BC%80%E5%8F%91%E5%AE%9E%E7%8E%B0%E4%B8%BA%E7%AC%AC%E4%B8%89%E6%96%B9bean%E6%B3%A8%E5%85%A5%E8%B5%84%E6%BA%90)
		- [简单数据类型](#%E7%AE%80%E5%8D%95%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
		- [引用数据类型](#%E5%BC%95%E7%94%A8%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)


- [十、注解开发总结](#十、注解开发总结)

- [十一、Spring整合](#十一、Spring整合)
	- [1、整合MyBatis](#1%E3%80%81%E6%95%B4%E5%90%88MyBatis)
		- [环境准备](#%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
		- [思路分析](#%E6%80%9D%E8%B7%AF%E5%88%86%E6%9E%90)
		- [整合步骤](#%E6%95%B4%E5%90%88%E6%AD%A5%E9%AA%A4)
	- [2、整合Junit](#2%E3%80%81%E6%95%B4%E5%90%88Junit)
		- [依赖是junit的整合方式](#%E4%BE%9D%E8%B5%96%E6%98%AFjunit%E7%9A%84%E6%95%B4%E5%90%88%E6%96%B9%E5%BC%8F)
		- [依赖是junit-jupiter-api的整合方式](#%E4%BE%9D%E8%B5%96%E6%98%AFjunit-jupiter-api%E7%9A%84%E6%95%B4%E5%90%88%E6%96%B9%E5%BC%8F)

# 八、IOC/DI注解开发

Spring的IOC/DI对应的`配置开发`已经讲解完成，但是使用起来相对来说还是比较复杂的，复杂的地方在`配置文件`。  
Spring到底是如何简化代码开发的呢?  
要想真正简化开发，就需要用到Spring的`注解开发`，Spring对注解支持的版本历程:

- 2.0版开始支持注解
- 2.5版注解功能趋于完善
- 3.0版支持纯注解开发

关于注解开发，这里会讲解两块内容`注解开发定义bean`和`纯注解开发`。  
注解开发定义bean用的是2.5版提供的注解，纯注解开发用的是3.0版提供的注解。

## 1、环境准备

- 创建一个Maven项目
- pom.xml添加Spring的依赖
- resources下添加applicationContext.xml
- 添加BookDao、BookDaoImpl、BookService、BookServiceImpl类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ..." );  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService {  
    public void save() {  
        System.out.println("book service save ...");  
    }  
}
```

- 创建运行类App
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");  
        bookDao.save();  
    }  
}
```
## 2、注解开发定义bean

- `步骤一：`删除原有的XML配置  
	将配置文件中的bean标签删除掉
```xml
<bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>
```

- `步骤二：`在Dao上添加注解  
	在BookDaoImpl类上添加`@Component`注解
```java
@Component("bookDao")  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}
```
`注意：@Component注解不可以添加在接口上，因为接口是无法创建对象的。`

- `步骤三：`配置Spring的注解包扫描  
	为了让Spring框架能够扫描到写在类上的注解，需要在配置文件上进行包扫描
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xsi:schemaLocation="  
            http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans.xsd  
            http://www.springframework.org/schema/context  
            http://www.springframework.org/schema/context/spring-context.xsd  
        ">  
    <context:component-scan base-package="com.blog"/>  
</beans>
```

- 说明：component-scan
    
    - component:组件,Spring将管理的bean视作自己的一个组件
    - scan:扫描  
        base-package指定Spring框架扫描的包路径，它会扫描指定包及其子包中的所有类上的注解。
    - 包路径越多`如:com.blog.dao.impl`，扫描的范围越小速度越快
    - 包路径越少`如:com.blog`,扫描的范围越大速度越慢
    - 一般扫描到项目的组织名称即Maven的groupId下`如:com.blog`即可。

- `步骤四：`运行程序
- `步骤五：`Service上添加注解  
	在BookServiceImpl类上也添加`@Component`交给Spring框架管理
```java
@Component  
public class BookServiceImpl implements BookService {  
    public void save() {  
        System.out.println("book service save ...");  
    }  
}
```

- `步骤六：`运行程序  
	在App类中，从IOC容器中获取BookServiceImpl对应的bean对象
```java
public class App {  
    public static void main(String[] args) {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        //按照名称获取bean  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        //按照类型获取bean  
        BookService bookService = context.getBean(BookService.class);  
        bookDao.save();  
        bookService.save();  
    }  
}
```

结果如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211552981.png)


- **说明:**
	- BookServiceImpl类没有起名称，所以在App中是按照类型来获取bean对象
	- `@Component`注解如果不起名称，会有一个默认值就是`当前类名首字母小写`，所以也可以按照名称获取，如
```java
BookService bookService = (BookService) context.getBean("bookServiceImpl");
```

对于@Component注解，还衍生出了其他三个注解`@Controller`、`@Service`、`@Repository`  

通过查看源码会发现：这三个注解和@Component注解的作用是一样的，为什么要衍生出这三个呢?  

这是方便我们后期在编写类的时候能很好的区分出这个类是属于`表现层`、`业务层`还是`数据层`的类。
## 3、纯注解开发模式

上面已经可以使用注解来配置bean,但是依然有用到配置文件，在配置文件中对包进行了扫描，Spring在3.0版已经支持纯注解开发，使用Java类替代配置文件，开启了Spring快速开发赛道，那么具体如何实现?
### 思路分析

实现思路为：
- 将配置文件applicationContext.xml删掉，用类来替换
### 实现步骤

- `步骤一：`创建配置类  
	创建一个配置类SpringConfig
```java
public class SpringConfig {  
}
```

- `步骤二：`标识该类为配置类  
	在配置类上面加一个`@Configuration`注解，将其标识为一个配置类，用于替换掉`applicationContext.XML`
```java
@Configuration  
public class SpringConfig {  
}
```

- `步骤三：`用注解替换包扫描配置  
	在配置类上添加包扫描注解`@ComponentScan`替换`<context:component-scan base-package=""/>`
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- `步骤四：`创建运行类并执行  
	创建一个新的运行类`AppForAnnotation`
```java
public class AppForAnnotation {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
        BookService bookService = context.getBean(BookService.class);  
        bookService.save();  
    }  
}
```

 运行AppForAnnotation,可以看到两个对象依然被获取成功![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211559622.png)


至此，纯注解开发的方式就已经完成了，主要内容包括：

- Java类替换Spring核心配置文件
    - `@Configuration`注解用于设定当前类为配置类
    - `@ComponentScan`注解用于设定扫描路径，此注解只能添加一次，多个数据请用数组格式
```java
@ComponentScan({com.blog.service","com.blog.dao"})
```

- 读取Spring核心配置文件初始化容器对象切换为读取Java配置类初始化容器对象
```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
```

- 知识点：`@Configuration`

|名称|@Configuration|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|设置该类为spring配置类|
|属性|value（默认）：定义bean的id|

- 知识点：`@ComponentScan`

|名称|@ComponentScan|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|设置spring配置类扫描路径，用于加载使用注解格式定义的bean|
|属性|value（默认）：扫描路径，此路径可以逐层向下扫描|
### 小结

这部分要重点掌握的是使用注解完成Spring的bean管理，需要掌握的内容为:

- 记住`@Component`、`@Controller`、`@Service`、`@Repository`这四个注解
- applicationContext.xml中`<context:component-san/>`的作用是指定扫描包路径，注解为`@ComponentScan`
- `@Configuration`标识该类为配置类，使用类替换`applicationContext.xml`文件
- `ClassPathXmlApplicationContext`是加载XML配置文件
- `AnnotationConfigApplicationContext`是加载配置类
## 4、注解开发bean的作用范围和生命周期

使用注解已经完成了bean的管理，接下来按照前面所学习的内容，将通过配置实现的内容都换成对应的注解实现，包含两部分内容:`bean作用范围(scope)`和`bean生命周期(init和destroy)`。
### bean的作用范围

- 修改`AppForAnnotation`类，并运行查看结果
```java
public class AppForAnnotation {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao1 = (BookDao) context.getBean("bookDao");  
        BookDao bookDao2 = (BookDao) context.getBean("bookDao");  
        System.out.println(bookDao1);  
        System.out.println(bookDao2);  
    }  
}
```

- 结果如下，说明默认情况下bean是单例![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211605987.png)


- 要想将BookDaoImpl变成非单例，只需要在其类上添加`@scope`注解
```java
@Component("bookDao")  
@Scope("prototype")  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}
```

- 再次运行程序，查看结果，输出的两个地址值不一致，说明已经成功变为非单例的了![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211608927.png)


- 知识点：`@scope`

|名称|@Scope|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|设置该类创建对象的作用范围，可用于设置创建出的bean是否为单例对象|
|属性|value（默认）：定义bean作用范围，`默认值singleton（单例）`，`可选值prototype（非单例）`|
### bean的生命周期

- 在BookDaoImpl中添加两个方法，`init`和`destroy`，方法名可以任意，再添加一个构造方法
```java
@Component("bookDao")  
public class BookDaoImpl implements BookDao {  
      
    public BookDaoImpl() {  
        System.out.println("construct ... ");  
    }  
  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
  
    public void init() {  
        System.out.println("init ... ");  
    }  
  
    public void destroy() {  
        System.out.println("destroy ... ");  
    }  
}
```

- 如何对方法进行标识，哪个是初始化方法，哪个是销毁方法?  
	只需要在对应的方法上添加`@PostConstruct`和`@PreDestroy`注解即可。
```java
@Component("bookDao")  
public class BookDaoImpl implements BookDao {  
  
    public BookDaoImpl() {  
        System.out.println("construct ... ");  
    }  
  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
  
    @PostConstruct  // 在构造方法之后执行，替换 init-method  
    public void init() {  
        System.out.println("init ... ");  
    }  
  
    @PreDestroy // 在销毁方法之前执行,替换 destroy-method  
    public void destroy() {  
        System.out.println("destroy ... ");  
    }  
}
```

- 要想看到两个方法执行，需要注意的是`destroy`只有在容器关闭的时候，才会执行，所以需要修改App的类
```java
public class AppForAnnotation {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        System.out.println(bookDao);  
        context.registerShutdownHook();//关闭容器  
    }  
}
```

- 运行AppForAnnotation类查看打印结果，证明init和destroy方法都被执行了，且init确实是在构造方法后执行的。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211613579.png)


![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211617175.png)


知识点`@PostConstruct`

|名称|@PostConstruct|
|---|---|
|类型|方法注解|
|位置|方法上|
|作用|设置该方法为初始化方法|
|属性|无|

知识点`@PreDestroy`

|名称|@PreDestroy|
|---|---|
|类型|方法注解|
|位置|方法上|
|作用|设置该方法为销毁方法|
|属性|无|
### 小结

配置文件中的bean标签中的  

`id`对应`@Component("")`，`@Controller("")`，`@Service("")`，`@Repository("")`  

`scope`对应`@scope()`  

`init-method`对应`@PostConstruct`  

`destroy-method`对应`@PreDestroy`

## 5、注解开发依赖注入

- Spring为了使用注解简化开发，并没有提供`构造函数注入`、`setter注入`对应的注解，只提供了自动装配的注解实现。
### 环境准备

- 创建一个Maven项目
- pom.xml添加Spring的依赖
- 添加一个配置类`SpringConfig`
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 添加BookDao、BookDaoImpl、BookService、BookServiceImpl类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ..." );  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService {  
    public void save() {  
        System.out.println("book service save ...");  
    }  
}
```

- 创建运行类AppForAnnotation
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookService bookService = ctx.getBean(BookService.class);  
        bookService.save();  
    }  
}
```
- 环境准备好后，直接运行App类会有问题，因为还没有提供配置注入BookDao的，所以bookDao对象为Null,调用其save方法就会报`控指针异常`。
### 注解实现按照类型注入

对于这个问题使用注解该如何解决?

- 在BookServiceImpl类的bookDao属性上添加`@Autowired`注解
```java
@Service  
public class BookServiceImpl implements BookService {  
    @Autowired  
    private BookDao bookDao;  
  
//    public void setBookDao(BookDao bookDao) {  
//        this.bookDao = bookDao;  
//    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

**注意:**

- `@Autowired`可以写在属性上，也可也写在setter方法上，最简单的处理方式是`写在属性上并将setter方法删除掉`
- 为什么setter方法可以删除呢?
    - 自动装配基于反射设计创建对象并通过`暴力反射`为私有属性进行设值
    - 普通反射只能获取public修饰的内容
    - 暴力反射除了获取public修饰的内容还可以获取private修改的内容
    - 所以此处无需提供setter方法

- `@Autowired是按照类型注入`，那么对应BookDao接口如果有多个实现类，比如添加BookDaoImpl2
```java
@Repository  
public class BookDaoImpl2 implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...2");  
    }  
}
```

这个时候再次运行App，就会报错`NoUniqueBeanDefinitionException`  
此时，按照类型注入就无法区分到底注入哪个对象，解决方案:`按照名称注入`

- 先给两个Dao类分别起个名称
```java
@Repository("bookDao")  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ..." );  
    }  
}  
@Repository("bookDao2")  
public class BookDaoImpl2 implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...2" );  
    }  
}
```

- 此时就可以注入成功，但是得思考个问题:
    
- @Autowired是按照类型注入的，给BookDao的两个实现起了名称，它还是有两个bean对象，为什么不报错?
- @Autowired默认按照类型自动装配，如果IOC容器中同类的Bean找到多个，就按照变量名和Bean的名称匹配。因为变量名叫`bookDao`而容器中也有一个`booDao`，所以可以成功注入。
- 那下面这种情况可以成功注入吗
```java
@Repository("bookDao1")  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ..." );  
    }  
}  
@Repository("bookDao2")  
public class BookDaoImpl2 implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...2" );  
    }  
}
```

还是不行的，因为按照类型会找到多个bean对象，此时会按照`bookDao`名称去找，因为IOC容器只有名称叫`bookDao1`和`bookDao2`，所以找不到，会报`NoUniqueBeanDefinitionException`
### 注解实现只能找名称注入

当根据类型在容器中找到多个bean,注入参数的属性名又和容器中bean的名称不一致，这个时候该如何解决，就需要使用到`@Qualifier`来`指定注入哪个名称的bean对象`。`@Qualifier`注解后的值就是需要注入的bean的名称。
```java
@Service  
public class BookServiceImpl implements BookService {  
    @Autowired  
    @Qualifier("bookDao1")  
    private BookDao bookDao;  
      
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

`注意:@Qualifier不能独立使用，必须和@Autowired一起使用`
### 简单数据类型注入

- 引用类型看完，简单类型注入就比较容易懂了。简单类型注入的是基本数据类型或者字符串类型，下面在`BookDaoImpl`类中添加一个`name`属性，用其进行简单类型注入
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    private String name;  
    public void save() {  
        System.out.println("book dao save ..." + name);  
    }  
}
```

- 数据类型换了，对应的注解也要跟着换，这次使用`@Value`注解，将值写入注解的参数中就行了
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    @Value("Stephen")  
    private String name;  
    public void save() {  
        System.out.println("book dao save ..." + name);  
    }  
}
```

注意数据格式要匹配，如将”abc”注入给int值，这样程序就会报错。

介绍完后，会有一种感觉就是这个注解好像没什么用，跟直接赋值是一个效果，还没有直接赋值简单，所以这个注解存在的意义是什么?继续往下看
### 注解读取properties配置文件

`@Value`一般会被用在从properties配置文件中读取内容进行使用，具体如何实现?

- `步骤一：`在resource下准备一个properties文件
```properties
name=Stephen
```

- `步骤二：`使用注解加载properties配置文件  
	在配置类上添加`@PropertySource`注解
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
public class SpringConfig {  
}
```

- `步骤三：`使用@Value读取配置文件中的内容
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    @Value("${name}")  
    private String name;  
    public void save() {  
        System.out.println("book dao save ..." + name);  
    }  
}
```

- `步骤四：`运行程序  
	运行App类，查看运行结果，说明配置文件中的内容已经被加载![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211628101.png)


**注意:**

- 如果读取的properties配置文件有多个，可以使用`@PropertySource`的属性来指定多个
```java
@PropertySource({"jdbc.properties","xxx.properties"})
```

- `@PropertySource`注解属性中不支持使用通配符`*`,`运行会报错`
```java
@PropertySource({"*.properties"})
```

- `@PropertySource`注解属性中可以把`classpath:`加上,代表从当前项目的根路径找文件
```java
@PropertySource({"classpath:jdbc.properties"})
```

知识点1：`@Autowired`

|名称|@Autowired|
|---|---|
|类型|属性注解 或 方法注解（了解） 或 方法形参注解（了解）|
|位置|属性定义上方 或 标准set方法上方 或 类set方法上方 或 方法形参前面|
|作用|为引用类型属性设置值|
|属性|required：true/false，定义该属性是否允许为null|

知识点2：`@Qualifier`

|名称|@Qualifier|
|---|---|
|类型|属性注解 或 方法注解（了解）|
|位置|属性定义上方 或 标准set方法上方 或 类set方法上方|
|作用|为引用类型属性指定注入的beanId|
|属性|value（默认）：设置注入的beanId|

知识点3：`@Value`

|名称|@Value|
|---|---|
|类型|属性注解 或 方法注解（了解）|
|位置|属性定义上方 或 标准set方法上方 或 类set方法上方|
|作用|为 基本数据类型 或 字符串类型 属性设置值|
|属性|value（默认）：要注入的属性值|

知识点4：`@PropertySource`

|名称|@PropertySource|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|加载properties文件中的属性值|
|属性|value（默认）：设置加载的properties文件对应的文件名或文件名组成的数组|
# 九、IOC/DI注解开发管理第三方bean

前面定义bean的时候都是在自己开发的类上面写个注解就完成了，但如果是第三方的类，这些类都是在jar包中，我们没有办法在类上面添加注解，这个时候该怎么办?

遇到上述问题，我们就需要有一种更加灵活的方式来定义bean,这种方式不能在原始代码上面书写注解，一样能定义bean,这就用到了一个全新的注解`@Bean`。
## 1、环境准备

- 创建一个Maven项目
- pom.xml添加Spring的依赖
- 添加一个配置类`SpringConfig`
```java
@Configuration  
public class SpringConfig {  
}
```

- 添加BookDao、BookDaoImpl类
```java
public interface BookDao {  
    public void save();  
}  
@Repository  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ..." );  
    }  
}
```

- 创建运行类App
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);  
    }  
}
```

## 2、注解开发管理第三方bean

在上述环境中完成对`Druid`数据源的管理，具体的实现步骤为

- `步骤一：`导入对应的jar包
```xml
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.16</version>  
</dependency>
```

- `步骤二：`在配置类中添加一个方法  
	注意`该方法的返回值就是要创建的Bean对象类型`
```java
@Configuration  
public class SpringConfig {  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:13306/spring_db");  
        dataSource.setUsername("root");  
        dataSource.setPassword("PASSWORD");  
        return dataSource;  
    }  
}
```

- `步骤三：`在方法上添加`@Bean`注解  
	`@Bean`注解的作用是将方法的返回值作为一个Spring管理的bean对象
```java
@Configuration  
public class SpringConfig {  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:13306/spring_db");  
        dataSource.setUsername("root");  
        dataSource.setPassword("PASSWORD");  
        return dataSource;  
    }  
}
```

- `步骤四：`从IOC容器中获取对象并打印
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);  
        DataSource dataSource = ctx.getBean(DataSource.class);  
        System.out.println(dataSource);  
    }  
}
```

输出如下![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211637007.png)


- 至此使用`@Bean`来管理第三方bean的案例就已经完成。
- 如果有多个bean要被Spring管理，直接在配置类中多写几个方法，方法上添加@Bean注解即可。
## 3、引入外部配置类

如果把所有的第三方bean都配置到Spring的配置类`SpringConfig`中，虽然可以，但是不利于代码阅读和分类管理，所有我们就想能不能按照类别将这些bean配置到不同的配置类中?

那么对于数据源的bean，我们可以把它的配置单独放倒一个`JdbcConfig`类中
```java
public class JdbcConfig {  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:13306/spring_db");  
        dataSource.setUsername("root");  
        dataSource.setPassword("PASSWORD");  
        return dataSource;  
    }  
}
```

那现在又有了一个新问题，这个配置类如何能被Spring配置类加载到，并创建DataSource对象在IOC容器中?

针对这个问题，有两个解决方案，接着往下看
### 使用包扫描引入(不推荐)

- `步骤一：`在Spring的配置类上添加包扫描  
	注意要将JdbcConfig类放在包扫描的地址下
```java
@Configuration  
@ComponentScan("com.blog.config")  
public class SpringConfig {  
}
```

- `步骤二：`在JdbcConfig上添加`@Configuration`注解  
	JdbcConfig类要放入到`com.blog.config`包下，需要被Spring的配置类扫描到即可
```java
@Configuration  
public class JdbcConfig {  
    @Bean  
    public DataSource dataSource(){  
        DruidDataSource ds = new DruidDataSource();  
        ds.setDriverClassName("com.mysql.jdbc.Driver");  
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");  
        ds.setUsername("root");  
        ds.setPassword("root");  
        return ds;  
    }  
}
```

- `步骤三：`运行程序  
	仍然可以获取到bean对象并输出到控制台![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211641278.png)


这种方式虽然能够扫描到，但是不能很快的知晓都引入了哪些配置类(因为把包下的所有配置类都扫描了)，所有这种方式不推荐使用。
### 使用@Import引入

方案一实现起来有点小复杂，Spring早就想到了这一点，于是又给我们提供了第二种方案。  

这种方案可以不用加`@Configuration`注解，但是必须在Spring配置类上使用`@Import`注解手动引入需要加载的配置类

- `步骤一：`去除JdbcConfig类上的注解
```java
public class JdbcConfig {  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:13306/spring_db");  
        dataSource.setUsername("root");  
        dataSource.setPassword("PASSWORD");  
        return dataSource;  
    }  
}
```

- `步骤二：`在Spring配置类中引入
```java
@Configuration  
@Import(JdbcConfig.class)  
public class SpringConfig {  
}
```

**注意:**

- 扫描注解可以移除
- @Import参数需要的是一个数组，可以引入多个配置类。
- `@Import注解在配置类中只能写一次`

- `步骤三：`运行程序  
	依然能获取到bean对象并打印控制台![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211645874.png)


知识点1：`@Bean`

|名称|@Bean|
|---|---|
|类型|方法注解|
|位置|方法定义上方|
|作用|设置该方法的返回值作为spring管理的bean|
|属性|value（默认）：定义bean的id|

知识点2：`@Import`

|名称|@Import|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|导入配置类|
|属性|value（默认）：定义导入的配置类类名，  <br>当配置类有多个时使用数组格式一次性导入多个配置类|

## 4、注解开发实现为第三方bean注入资源

在使用@Bean创建bean对象的时候，如果方法在创建的过程中需要其他资源该怎么办?

这些资源会有两大类，分别是`简单数据类型` 和`引用数据类型`。
### 简单数据类型

对于下面代码关于数据库的四要素不应该写死在代码中，应该是从properties配置文件中读取。如何来优化下面的代码?
```java
public class JdbcConfig {  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:13306/spring_db");  
        dataSource.setUsername("root");  
        dataSource.setPassword("PASSWORD");  
        return dataSource;  
    }  
}
```

- `步骤一：`提供对应的四个属性
```java
public class JdbcConfig {  
  
    private String driver;  
    private String url;  
    private String username;  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(username);  
        dataSource.setPassword(password);  
        return dataSource;  
    }  
}
```

- `步骤二：`使用`@Value`注解
```java
public class JdbcConfig {  
    @Value("com.mysql.jdbc.Driver")  
    private String driver;  
    @Value("jdbc:mysql://localhost:13306/spring_db")  
    private String url;  
    @Value("root")  
    private String username;  
    @Value("PASSWORD")  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(username);  
        dataSource.setPassword(password);  
        return dataSource;  
    }  
}
```

- 扩展  
    现在的数据库连接四要素还是写在代码中，需要做的是将这些内容提取到jdbc.properties配置文件，在上面我们已经实现过了，这里再来复习一遍
    
1. resources目录下添加jdbc.properties
2. 配置文件中提供四个键值对分别是数据库的四要素
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db  
jdbc.username=root  
jdbc.password=PASSWORD.
```

3. 使用@PropertySource加载jdbc.properties配置文件
4. 修改@Value注解属性的值，将其修改为`${key}`，key就是键值对中的键的值
```java
@PropertySource("jdbc.properties")  
public class JdbcConfig {  
    @Value("${jdbc.driver}")  
    private String driver;  
    @Value("${jdbc.url}")  
    private String url;  
    @Value("${jdbc.username}")  
    private String username;  
    @Value("${jdbc.password}")  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(username);  
        dataSource.setPassword(password);  
        return dataSource;  
    }  
}
```

输出结果如下![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211652313.png)


### 引用数据类型

假设在构建DataSource对象的时候，需要用到BookDao对象，该如何把BookDao对象注入进方法内让其使用呢?

- `步骤一：`在SpringConfig中扫描BookDao  
    扫描的目的是让Spring能管理到BookDao，也就是要让IOC容器中有一个BookDao对象
```java
@Configuration  
@ComponentScan("com.blog.dao")  
@Import(JdbcConfig.class)  
public class SpringConfig {  
}
```

- `步骤二：`在JdbcConfig类的方法上添加参数  
	引用类型注入只需要为bean定义方法设置形参即可，容器会`根据类型`自动装配对象。
```java
@Bean  
//将IOC中的Bean放在方法形参上即可
public DataSource dataSource(BookDao bookDao) {  
    bookDao.save();  
    DruidDataSource dataSource = new DruidDataSource();  
    dataSource.setDriverClassName(driver);  
    dataSource.setUrl(url);  
    dataSource.setUsername(username);  
    dataSource.setPassword(password);  
    return dataSource;  
}
```

- `步骤三：`运行程序  
	结果如下，说明bookDao已经成功注入![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211656144.png)


# 十、注解开发总结



![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211659159.png)



- 其中，经常用到的注解有
- Conponenet（service）
- ConponentScan
- Atuowired
- Bean
# 十一、Spring整合
## 1、整合MyBatis

### 环境准备

- 在准备环境的同时，我们也来回顾一下MyBatis开发的相关内容

- `步骤一：`准备数据库表  
	MyBatis是用来操作数据库表的，所以我们先来创建库和表
```sql
create database spring_db character set utf8;  
use spring_db;  
create table tbl_account(  
    id int primary key auto_increment,  
    name varchar(35),  
    money double  
);  
  
INSERT INTO tbl_account(`name`,money) VALUES  
('Tom',2800),  
('Jerry',3000),  
('Jhon',3100);
```

- `步骤二：`创建项目导入依赖
```xml
<dependencies>  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>5.1.46</version>  
    </dependency>  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis</artifactId>  
        <version>3.5.6</version>  
    </dependency>  
    <dependency>  
        <groupId>com.alibaba</groupId>  
        <artifactId>druid</artifactId>  
        <version>1.1.16</version>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-context</artifactId>  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
</dependencies>
```

- `步骤三：`根据表创建模型类
```java
public class Account {  
    private Integer id;  
    private String name;  
    private Double money;  
  
    public Account() {  
    }  
  
    public Account(Integer id, String name, double money) {  
        this.id = id;  
        this.name = name;  
        this.money = money;  
    }  
  
    public Integer getId() {  
        return id;  
    }  
  
    public void setId(Integer id) {  
        this.id = id;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public double getMoney() {  
        return money;  
    }  
  
    public void setMoney(double money) {  
        this.money = money;  
    }  
  
    @Override  
    public String toString() {  
        return "Account{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                ", money=" + money +  
                '}';  
    }  
}
```

- `步骤四：`创建Dao接口（在之前是Mapper接口，且要配置一个对应的xml文件，不过这里没涉及到复杂的sql语句，所以没配置xml文件，采用注解开发）
```java
public interface AccountDao {  
  
    @Insert("insert into tbl_account(name, money) VALUES (#{name}, #{money})")  
    void save(Account account);  
  
    @Delete("delete from tbl_account where id = #{id}")  
    void delete(Integer id);  
  
    @Update("update tbl_account set `name` = #{name}, money = #{money}")  
    void update(Account account);  
  
    @Select("select * from tbl_account")  
    List<Account> findAll();  
  
    @Select("select * from tbl_account where id = #{id}")  
    Account findById(Integer id);  
}
```

- `步骤五：`创建Service接口和实现类
```java
public interface AccountService {  
    void save(Account account);  
  
    void delete(Integer id);  
  
    void update(Account account);  
  
    List<Account> findAll();  
  
    Account findById(Integer id);  
}


@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    private AccountDao accountDao;  
  
    public void save(Account account) {  
        accountDao.save(account);  
    }  
  
    public void delete(Integer id) {  
        accountDao.delete(id);  
    }  
  
    public void update(Account account) {  
        accountDao.update(account);  
    }  
  
    public List<Account> findAll() {  
        return accountDao.findAll();  
    }  
  
    public Account findById(Integer id) {  
        return accountDao.findById(id);  
    }  
}
```

- `步骤六：`添加jdbc.properties文件
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db  
jdbc.username=root  
jdbc.password=PASSWORD.
```

- `步骤七：`添加Mybatis核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
    <!--读取外部properties配置文件-->  
    <properties resource="jdbc.properties"></properties>  
    <!--别名扫描的包路径-->  
    <typeAliases>  
        <package name="com.blog.domain"/>  
    </typeAliases>  
    <!--数据源-->  
    <environments default="mysql">  
        <environment id="mysql">  
            <transactionManager type="JDBC"></transactionManager>  
            <dataSource type="POOLED">  
                <property name="driver" value="${jdbc.driver}"></property>  
                <property name="url" value="${jdbc.url}"></property>  
                <property name="username" value="${jdbc.username}"></property>  
                <property name="password" value="${jdbc.password}"></property>  
            </dataSource>  
        </environment>  
    </environments>  
    <!--映射文件扫描包路径-->  
    <mappers>  
        <package name="com.blog.dao"></package>  
    </mappers>  
</configuration>
```

- `步骤八：`编写应用程序
```java
public class App {  
    public static void main(String[] args) throws IOException {  
        // 1. 创建SqlSessionFactoryBuilder对象  
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();  
        // 2. 加载mybatis-config.xml配置文件  
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");  
        // 3. 创建SqlSessionFactory对象  
        SqlSessionFactory factory = sqlSessionFactoryBuilder.build(inputStream);  
        // 4. 获取SqlSession  
        SqlSession sqlSession = factory.openSession();  
        // 5. 获取mapper  
        AccountDao mapper = sqlSession.getMapper(AccountDao.class);  
        //6. 执行方法进行查询  
        Account account = mapper.findById(2);  
        System.out.println(account);  
        //7. 释放资源  
        sqlSession.close();  
    }  
}
```

- `步骤九：`运行程序，结果如下![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211706530.png)


### 思路分析

Mybatis的基础环境我们已经准备好了，接下来就得分析下在上述的内容中，哪些对象可以交给Spring来管理?

- Mybatis程序核心对象分析  
	- 从图中可以获取到，`真正需要交给Spring管理的是SqlSessionFactory`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211709881.png)
	- 整合Mybatis，就是`将Mybatis用到的内容交给Spring管理`，分析下配置文件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211711885.png)


说明:

- 第一部分读取外部properties配置文件，Spring有提供具体的解决方案`@PropertySource`,需要交给Spring
- 第二部分起别名包扫描，为SqlSessionFactory服务的，需要交给Spring
- 第三部分主要用于做连接池，Spring之前我们已经整合了Druid连接池，这块也需要交给Spring
- 前面三部分一起都是为了创建SqlSession对象用的，那么用Spring管理SqlSession对象吗?回忆下SqlSession是由SqlSessionFactory创建出来的，所以只需要将SqlSessionFactory交给Spring管理即可。
- 第四部分是Mapper接口和映射文件[如果使用注解就没有该映射文件]，这个是在获取到SqlSession以后执行具体操作的时候用，所以它和SqlSessionFactory创建的时机都不在同一个时间，可能需要单独管理。
### 整合步骤

前面我们已经分析了Spring与Mybatis的整合，大体需要做两件事，

- 第一件事是：`Spring要管理MyBatis中的SqlSessionFactory`
- 第二件事是：`Spring要管理Mapper接口的扫描`

那我们下面就开始来整合

- `步骤一：`项目中导入整合需要的jar包
```xml
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

- `步骤二：`创建Spring的主配置类
```java
//配置类注解  
@Configuration  
//包扫描，主要扫描的是项目中的AccountServiceImpl类  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- `步骤三：`创建数据源的配置类  
	在配置类中完成数据源的创建
```java
public class JdbcConfig {  
    @Value("${jdbc.driver}")  
    private String driver;  
    @Value("${jdbc.url}")  
    private String url;  
    @Value("${jdbc.username}")  
    private String username;  
    @Value("${jdbc.password}")  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(username);  
        dataSource.setPassword(password);  
        return dataSource;  
    }  
}
```

- `步骤四：`主配置类中读properties并引入数据源配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import(JdbcConfig.class)  
public class SpringConfig {  
}
```

- `步骤五：`创建Mybatis配置类并配置SqlSessionFactory
```java
public class MyBatisConfig {  
  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {  
        //定义bean，SqlSessionFactoryBean，用于产生SqlSessionFactory对象  
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();  
        //设置模型类的别名扫描  
        sqlSessionFactory.setTypeAliasesPackage("com.blog.domain");  
        //设置数据源  
        sqlSessionFactory.setDataSource(dataSource);  
        return sqlSessionFactory;  
    }  
    //定义bean，返回MapperScannerConfigurer对象  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer() {  
        MapperScannerConfigurer msc = new MapperScannerConfigurer();  
        msc.setBasePackage("com.blog.dao");  
        return msc;  
    }  
}
```

说明：

- 使用SqlSessionFactoryBean封装SqlSessionFactory需要的环境信息![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211717497.png)


- SqlSessionFactoryBean是前面我们讲解FactoryBean的一个子类，在该类中将SqlSessionFactory的创建进行了封装，简化对象的创建，我们只需要将其需要的内容设置即可。
- 方法中有一个参数为dataSource,当前Spring容器中已经创建了Druid数据源，类型刚好是DataSource类型，此时在初始化SqlSessionFactoryBean这个对象的时候，发现需要使用DataSource对象，而容器中刚好有这么一个对象，就自动加载了DruidDataSource对象。
    
- `sqlSessionFactory.setTypeAliasesPackage("com.blog.domain");`，替换掉配置文件中的
```xml
<typeAliases>  
    <package name="com.blog.domain"/>  
</typeAliases>
```

- `sqlSessionFactory.setDataSource(dataSource);`，替换掉配置文件中的
```xml
<environments default="mysql">  
    <environment id="mysql">  
        <transactionManager type="JDBC"></transactionManager>  
        <dataSource type="POOLED">  
            <property name="driver" value="${jdbc.driver}"></property>  
            <property name="url" value="${jdbc.url}"></property>  
            <property name="username" value="${jdbc.username}"></property>  
            <property name="password" value="${jdbc.password}"></property>  
        </dataSource>  
    </environment>  
</environments>
```

- 使用MapperScannerConfigurer加载Dao接口，创建代理对象保存到IOC容器中![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211721180.png)



- `步骤六：`主配置类中引入Mybatis配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤七：`编写运行类  
	在运行类中，从IOC容器中获取Service对象，调用方法获取结果
```java
public class App {  
    public static void main(String[] args) throws IOException {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        AccountService accountService = context.getBean(AccountService.class);  
        Account account = accountService.findById(1);  
        System.out.println(account);  
    }  
}
```

- `步骤八：`运行程序![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240327205656.png)

至此，Spring与Mybatis的整合就已经完成了，其中主要用到的两个类分别是:

- `SqlSessionFactoryBean`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211733166.png)


- `MapperScannerConfigurer`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922211740465.png)


## 2、整合Junit

### 依赖是junit的整合方式

- `步骤一：`引入依赖
```xml
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.12</version>  
    <scope>test</scope>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

- `步骤二：`编写测试类
```java
//设置类运行器  
@RunWith(SpringJUnit4ClassRunner.class)  
//设置Spring环境对应的配置类  
@ContextConfiguration(classes = {SpringConfig.class})//加载配置类  
public class AccountServiceTest {  
    //支持自动装配注入bean  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void test(){  
        Account account = accountService.findById(1);  
        System.out.println(account);  
    }  
  
    @Test  
    public void selectAll(){  
        List<Account> accounts = accountService.findAll();  
        System.out.println(accounts);  
    }  
}
```

如果出现如图错误，则是Junit版本太低，最少要按照课程使用4.12版本的junit![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240327210334.png)

**注意:**

- 单元测试，如果测试的是注解配置类，则使用`@ContextConfiguration(classes = 配置类.class)`
- 单元测试，如果测试的是配置文件，则使用`@ContextConfiguration(locations={配置文件名,...})`
- Junit运行后是基于Spring环境运行的，所以Spring提供了一个专用的类运行器，这个务必要设置，这个类运行器就在Spring的测试专用包中提供的，导入的坐标就是这个东西`SpringJUnit4ClassRunner`
- 上面两个配置都是固定格式，当需要测试哪个bean时，使用自动装配加载对应的对象，下面的工作就和以前做Junit单元测试完全一样了

知识点1：`@RunWith`

|名称|@RunWith|
|---|---|
|类型|测试类注解|
|位置|测试类定义上方|
|作用|设置JUnit运行器|
|属性|value（默认）：运行所使用的运行期|

知识点2：`@ContextConfiguration`

|名称|@ContextConfiguration|
|---|---|
|类型|测试类注解|
|位置|测试类定义上方|
|作用|设置JUnit加载的Spring核心配置|
|属性|classes：核心配置类，可以使用数组的格式设定加载多个配置类  <br>locations:配置文件，可以使用数组的格式设定加载多个配置文件名称|
### 依赖是junit-jupiter-api的整合方式

1.  整合测试环境作用

    好处1：不需要自己创建IOC容器对象了

    好处2：任何需要的bean都可以在测试类中直接享受自动装配

2.  导入相关依赖
    ```xml
    <!--junit5测试-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>6.0.6</version>
        <scope>test</scope>
    </dependency>
    ```
3.  整合测试注解使用
    ```java
    //@SpringJUnitConfig(locations = {"classpath:spring-context.xml"})  //指定配置文件xml
    @SpringJUnitConfig(value = {BeanConfig.class})  //指定配置类
    public class Junit5IntegrationTest {
        
        @Autowired
        private User user;
        
        @Test
        public void testJunit5() {
            System.out.println(user);
        }
    }
    ```