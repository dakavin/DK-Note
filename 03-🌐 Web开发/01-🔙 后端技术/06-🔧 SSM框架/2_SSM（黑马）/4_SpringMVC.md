---
文章标题: "[[4_SpringMVC]]" 
文章作者: Dakkk
文章概要: |
  介绍SpringMVC框架基础知识，包括MVC架构、入门案例、请求响应处理、RESTful风格开发和实际案例应用。
tags:
- "SpringMVC"
- "Web开发"
- "MVC架构"
- "RESTful"
- "JSON数据"
- "请求映射"
- "参数传递"
- "响应处理"
相关文章:
- "[[5_SSM整合]]"
- "[[1_Java值传递详解]]"
- "[[2_登录页加点盐：通过 Animate.css 添加入场动画]]"
- "[[2_函数]]"
- "[[2_Servlet]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/2_SSM（黑马）/4_SpringMVC.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:12:02
---

## 1 AOP简介

前面我们在介绍Spring的时候说过，Spring有两个核心的概念，一个是`IOC/DI`，一个是`AOP`

前面已经对`IOC/DI`进行了系统的学习，接下来要学习它的另一个核心内容，就是`AOP`。

`AOP是在不改原有代码的前提下对其进行增强`。

那现在我们围绕这句话，主要学习两方面内容`AOP核心概念`,`AOP作用`
### 1.1 什么是AOP？

- AOP(Aspect Oriented Programming)面向切面编程，是一种编程范式，指导开发者如何组织程序结构
- OOP(Object Oriented Programming)面向对象编程

我们都知道OOP是一种编程思想，那么AOP也是一种编程思想，编程思想主要的内容就是指导程序员该如何编写程序，所以它们两个是不同的`编程范式`。
### 1.2 AOP的作用

- 作用:在不惊动原始设计的基础上为其进行`功能增强`
### 1.3 AOP核心概念

为了能更好的理解AOP的相关概念，我们准备了一个环境，整个环境的内容我们暂时可以不用关注，最主要的类为:`BookDaoImpl`
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        //记录程序当前执行执行（开始时间）  
        Long startTime = System.currentTimeMillis();  
        //业务执行万次  
        for (int i = 0;i<10000;i++) {  
            System.out.println("book dao save ...");  
        }  
        //记录程序当前执行时间（结束时间）  
        Long endTime = System.currentTimeMillis();  
        //计算时间差  
        Long totalTime = endTime-startTime;  
        //输出信息  
        System.out.println("执行万次消耗时间：" + totalTime + "ms");  
    }  
    public void update(){  
        System.out.println("book dao update ...");  
    }  
    public void delete(){  
        System.out.println("book dao delete ...");  
    }  
    public void select(){  
        System.out.println("book dao select ...");  
    }  
}
```

- 代码的内容很简单，就是测试一下万次执行的耗时  
当在App类中从容器中获取bookDao对象后，分别执行其`save`,`delete`,`update`和`select`方法后会有如下的打印结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ba2defabc13303c7bc90d01d958946b.png)


- `疑问`
	- 对于计算万次执行消耗的时间只有save方法有，为什么delete和update方法也会有呢?
	- delete和update方法有，那什么select方法为什么又没有呢?

- 这个案例中其实就使用了Spring的AOP，在不惊动(改动)原有设计(代码)的前提下，想给谁添加额外功能就给谁添加。这个也就是Spring的理念：
	- `无入侵式！无侵入式!`

说了这么多，Spring到底是如何实现的呢?![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/635327f4734b3a66b75ec86cbef83b4c.png)


1. 前面一直在强调，Spring的AOP是对一个类的方法在不进行任何修改的前提下实现增强。对于上面的案例中BookServiceImpl中有`save`,`update`,`delete`和`select`方法,这些方法我们给起了一个名字叫`连接点`
2. 在BookServiceImpl的四个方法中，`update`和`delete`只有打印没有计算万次执行消耗时间，但是在运行的时候已经有该功能，那也就是说`update`和`delete`方法都已经被增强，所以对于需要增强的方法我们给起了一个名字叫`切入点`
3. 执行BookServiceImpl的update和delete方法的时候都被添加了一个计算万次执行消耗时间的功能，将这个功能抽取到一个方法中，换句话说就是存放共性功能的方法，我们给起了个名字叫`通知`
4. 通知是要增强的内容，会有多个，切入点是需要被增强的方法，也会有多个，那哪个切入点需要添加哪个通知，就需要提前将它们之间的关系描述清楚，那么对于通知和切入点之间的关系描述，我们给起了个名字叫`切面`
5. 通知是一个方法，方法不能独立存在需要被写在一个类中，这个类我们也给起了个名字叫`通知类`

至此AOP中的核心概念就已经介绍完了，总结下:

- `连接点(JoinPoint)`：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等
    - `在SpringAOP中，理解为方法的执行`
- `切入点(Pointcut)`:匹配连接点的式子
    - 在SpringAOP中，一个切入点可以描述一个具体方法，也可也匹配多个方法
        - **一个具体的方法**:如com.blog.dao包下的BookDao接口中的无形参无返回值的save方法
        - **匹配多个方法**:所有的save方法/所有的get开头的方法/所有以Dao结尾的接口中的任意方法/所有带有一个参数的方法
    - **连接点范围要比切入点范围大**，是切入点的方法也一定是连接点，但是是连接点的方法就不一定要被增强，所以可能不是切入点。
- `通知(Advice)`:在切入点处执行的操作，也就是共性功能
    - 在SpringAOP中，功能最终以方法的形式呈现
- `通知类：定义通知的类`
- `切面(Aspect):描述通知与切入点的对应关系`
### 1.4 小结

这部分需要掌握的内容是

- 什么是AOP?
- AOP的作用是什么?
- AOP中核心概念分别指的是什么?
    - 连接点
    - 切入点
    - 通知
    - 通知类
    - 切面
## 2 AOP入门案例

### 2.1 需求分析

案例设定：测算接口执行效率，但是这个案例稍微复杂了点，我们对其进行简化。  
简化设定：在方法执行前输出当前系统时间。  
那现在我们使用SpringAOP的注解方式完成在方法执行的前打印出当前系统时间。
### 2.2 思路分析

1. 导入坐标
2. 制作连接点（原始操作，Dao接口及实现类）
3. 制作共性功能（通知类和通知）
4. 定义切入点
5. 绑定切入点和通知的关系（切面）
### 2.3 环境准备

- 创建一个Maven项目
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    public void save();  
    public void update();  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println(System.currentTimeMillis());  
        System.out.println("book dao save ...");  
    }  
  
    public void update() {  
        System.out.println("book dao update ...");  
    }  
}
```

- 创建Spring的配置类
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```

**说明:**

- 目前打印save方法的时候，因为方法中有打印系统时间，所以运行的时候是可以看到系统时间
- 对于update方法来说，就没有该功能
- 我们要使用SpringAOP的方式在不改变update方法的前提下让其具有打印系统时间的功能。
### 2.4 AOP实现步骤

- `步骤一：`添加依赖
```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```
- 因为`spring-context`中已经导入了`spring-aop`,所以不需要再单独导入`spring-aop`  
    导入AspectJ的jar包,AspectJ是AOP思想的一个具体实现，Spring有自己的AOP实现，但是相比于AspectJ来说比较麻烦，所以我们`直接采用Spring整合ApsectJ的方式进行AOP开发`。
    
- `步骤二：`定义接口和实现类  
    准备环境的时候我们已经完成了
    
- `步骤三：`定义通知类和通知  
    通知就是将共性功能抽取出来后形成的方法，共性功能指的就是当前系统时间的打印。  
    类名和方法名没有要求，可以任意。
```java
public class MyAdvice {  
    public void method(){  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤四：`定义切入点  
	BookDaoImpl中有两个方法，分别是update()和save()，我们要增强的是update方法，那么该如何定义呢？
```java
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

**说明:**

- 切入点定义依托一个不具有实际意义的方法进行，即无参数、无返回值、方法体无实际逻辑。
- execution及后面编写的内容，之后我们会专门去学习。

- `步骤五：`制作切面  
	切面是用来描述通知和切入点之间的关系，如何进行关系的绑定?
```java
public class MyAdvice {  
  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt(){}  
      
    @Before("pt()")  
    public void method(){  
        System.out.println(System.currentTimeMillis());  
    }  
}
```
绑定切入点与通知关系，并指定通知添加到原始连接点的具体执行`位置`

- **说明:**`@Before`翻译过来是之前，也就是说通知会在切入点方法执行之前执行，除此之前还有其他四种类型，后面会讲。  

- 那这里就会在执行update()之前，来执行我们的method()，输出当前毫秒值

- `步骤六：`将通知类配给容器并标识其为切面类
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤七：`开启注解格式AOP功能  
	使用`@EnableAspectJAutoProxy`注解
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- `步骤八：`运行程序  
	这次我们再来调用update()
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```

- 控制台成功输出了当前毫秒值![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24b3bd768460ca3c4ce623644a595058.png)


知识点1：`@EnableAspectJAutoProxy`

| 名称  | @EnableAspectJAutoProxy |
| --- | ----------------------- |
| 类型  | **配置类注解**               |
| 位置  | 配置类定义上方                 |
| 作用  | 开启注解格式AOP功能             |

知识点2：`@Aspect`

| 名称  | @Aspect      |
| --- | ------------ |
| 类型  | **类注解**      |
| 位置  | 切面类定义上方      |
| 作用  | 设置当前类为AOP切面类 |

知识点3：`@Pointcut`

|名称|@Pointcut|
|---|---|
|类型|方法注解|
|位置|切入点方法定义上方|
|作用|设置切入点方法|
|属性|value（默认）：切入点表达式|

知识点4：`@Before`

|名称|@Before|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行|
## 3 AOP工作流程

AOP的入门案例已经完成，对于刚才案例的执行过程，我们就得来分析分析，这一节我们主要讲解两个知识点:`AOP工作流程`和`AOP核心概念`。其中核心概念是对前面核心概念的补充。
### 3.1 AOP工作流程

由于AOP是基于Spring容器管理的bean做的增强，所以整个工作过程需要从Spring加载bean说起

- `流程一`：Spring容器启动
    - 容器启动就需要去加载bean,哪些类需要被加载呢?
    - `需要被增强的类，如:BookServiceImpl`
    - `通知类，如:MyAdvice`
    - **注意此时bean对象还没有创建成功**
- `流程二`：读取所有切面配置中的切入点
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void ptx() {  
    }  
  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```
上面这个例子中有两个切入点的配置，但是第一个`ptx()`并没有被使用，所以不会被读取。

- `流程三`：初始化bean，判定bean对应的类中的方法（连接点）是否匹配到任意切入点
    - 注意第一步在容器启动的时候，bean对象还没有被创建成功。
    - 要被实例化bean对象的类中的连接点（方法）和切入点进行匹配![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42cba4fb3bc728f2ff93e910f9bc112b.png)
    - 匹配失败，创建原始对象，如`UserDao`
    - 匹配失败说明不需要增强，直接调用原始对象的方法即可。

- 匹配成功，创建原始对象（`目标对象`）的`代理`对象，如:`BookDao`
    - 匹配成功说明需要对其进行增强
    - 对哪个类做增强，这个类对应的对象就叫做目标对象
    - 因为要对目标对象进行功能增强，而采用的技术是动态代理，所以会为其创建一个代理对象（**此时的bean为它的代理对象**）
    - 最终运行的是代理对象的方法，在该方法中会对原始方法进行功能增强

- `流程四`：获取bean执行方法
    - 获取的bean是原始对象时，调用方法并执行，完成操作
    - 获取的bean是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作


- 下面我们来验证一下容器中是否为代理对象
    - 如果目标对象中的方法`会被增强`，那么容器中将存入的是目标对象的`代理对象`
    - 如果目标对象中的方法`不被增强`，那么容器中将存入的是目标`对象本身`

- `步骤一：`修改App运行类，获取类的类型并输出
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        System.out.println(bookDao);  
        System.out.println(bookDao.getClass());  
    }  
}
```

- `步骤二：`修改MyAdvice类，改为不增强  
	将定义的切入点改为`updatexxx`，而BookDaoImpl类中不存在该方法，所以BookDao中的update方法在执行的时候，就不会被增强  
	所以此时容器中的对象应该是目标对象本身。
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.updatexxx())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤三：`运行程序  
	输出结果如下，确实是目标对象本身，符合我们的预期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/905e5aa1ac40c44bdc3b6631402855b2.png)


- `步骤四：`修改MyAdvice类，改为增强  
	将定义的切入点改为`update`，那么BookDao中的update方法在执行的时候，就会被增强  
	所以容器中的对象应该是目标对象的代理对象
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤五：`运行程序![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/377d2ba8775b27e94af252bb2da04dda.png)



至此对于刚才的结论，我们就得到了验证，这块我们需要注意的是:

📝📝：`不能直接打印对象`，从上面两次结果中可以看出，直接打印对象走的是对象的toString方法，不管是不是代理对象，打印的结果都是一样的，`原因是内部对toString方法进行了重写`。
### 3.2 AOP核心概念

在上面介绍AOP的工作流程中，我们提到了两个核心概念，分别是:

- `目标对象(Target)`：原始功能去掉共性功能对应的类产生的对象，这种对象是无法直接完成最终工作的
- `代理(Proxy)`：目标对象无法直接完成工作，需要对其进行功能回填，通过原始对象的代理对象实现

上面这两个概念比较抽象，简单来说

目标对象就是要增强的类`如:BookServiceImpl类`对应的对象，也叫原始对象，不能说它不能运行，只能说它在运行的过程中对于要增强的内容是缺失的

SpringAOP是在不改变原有设计(代码)的前提下对其进行增强的，它的底层采用的是代理模式实现的，所以要对原始对象进行增强，就需要对原始对象创建代理对象，在代理对象中的方法把通知`如:MyAdvice中的method方法`内容加进去，就实现了增强，这就是我们所说的代理(Proxy)
### 3.3 小结

这部分我们需要掌握的内容有：

- `能说出AOP的工作流程`
- AOP的核心概念
    - 目标对象、连接点、切入点
    - 通知类、通知
    - 切面
    - 代理
- SpringAOP的本质或者可以说底层实现是通过`代理模式`。

## 4 AOP配置管理

### 4.1 AOP切入点表达式

前面我们已经接触过了切入点表达式，下面我们来具体学习一下
```java
@Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")
```

对于AOP中切入点表达式，我们总共会学习三个内容，分别是`语法格式`、`通配符`和`书写技巧`。
#### 4.1.1 语法格式

首先我们先要明确两个概念:

- `切入点`:要进行增强的方法
- `切入点表达式`:要进行增强的方法的描述方式

对于切入点的描述，我们其实是有两种方式的，先来看下前面的例子  
由于BookDaoImpl类实现了BookDao接口，那么有如下两种方式来描述

- `描述方式一`：执行com.blog.dao包下的BookDao接口中的无参数update方法
```java
execution(void com.blog.dao.BookDao.update())
```

- 描述方式二：执行com.blog.dao.`impl包下的BookDaoImpl类中的无参数update方法`
```java
execution(void com.blog.dao.impl.BookDaoImpl.update())
```

📝📝📝：因为调用接口方法的时候最终运行的还是其实现类的方法，所以上面两种描述方式都是可以的。

对于切入点表达式的语法为:

- 切入点表达式标准格式：动作关键字(访问修饰符 返回值 包名.类/接口名.方法名(参数) 异常名）  
    对于这个格式，我们不需要硬记，通过一个例子，理解它:
```java
execution(public User com.blog.service.UserService.findById(int))
```

- `execution`：`动作关键字`，描述切入点的行为动作，例如execution表示执行到指定切入点
- `public`:`访问修饰符`,还可以是public，private等，`可以省略`
- `User`：`返回值`，写返回值类型
- `com.blog.service`：`包名`，多级包使用点连接
- `UserService`:`类/接口名称`
- `findById`：`方法名`
- `int`:`参数`，直接写参数的类型，多个类型用逗号隔开
- `异常名`：方法定义中`抛出指定异常`，`可以省略`

切入点表达式就是要找到需要增强的方法，所以它就是对一个具体方法的描述，但是方法的定义会有很多，所以如果每一个方法对应一个切入点表达式，想想这块就会觉得将来编写起来会比较麻烦，有没有更简单的方式呢?

- 使用通配符
#### 4.1.2 通配符

我们使用通配符描述切入点，`主要的目的就是简化之前的配置`，具体都有哪些通配符可以使用?

- `*`:`单个`独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现  
    匹配com.blog包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法
```java
execution（public * com.blog.*.UserService.find*(*))
```

- `..`：`多个`连续的任意符号，可以独立出现，常用于`简化包名与参数的书写` 
	匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法
```java
execution（public User com..UserService.findById(..))
```

- `+`：专用于匹配子类类型  
	这个使用率较低，描述子类的，`*Service+`，表示所有以Service结尾的接口的子类
```java
execution(* *..*Service+.*(..))
```

- 下面我们来具体分析一下各种用法

- 匹配接口，能匹配到
```java
execution(void com.blog.dao.BookDao.update())
```

- 匹配实现类，能匹配到
```java
execution(void com.blog.dao.impl.BookDaoImpl.update())
```

- **返回值任意**，能匹配到
```java
execution(* com.blog.dao.impl.BookDaoImpl.update())
```

- 返回值任意，但是update方法必须要有一个参数，无法匹配，要想匹配需要在update接口和实现类添加一个参数
```java
execution(* com.blog.dao.impl.BookDaoImpl.update(*))
```

- 返回值为void，com包下的任意包三层包下的任意类的update方法，匹配到的是实现类，能匹配
```java
execution(void com.*.*.*.*.update())
```

- 返回值为void,com包下的任意两层包下的任意类的update方法，匹配到的是接口，能匹配
```java
execution(void com.*.*.*.update())
```

- 返回值为void，方法名是update的任意包下的任意类，能匹配
```java
execution(void *..update())
```

- 匹配项目中任意类的任意方法，能匹配，但是不建议使用这种方式，影响范围广
```java
execution(* *..*(..))
```

- 匹配项目中任意包任意类下只要以u开头的方法，update方法能满足，能匹配
```java
execution(* *..u*(..))
```

- 匹配项目中任意包任意类下只要以e结尾的方法，update和save方法能满足，能匹配
```java
execution(* *..*e(..))
```

- 返回值为void，com包下的任意包任意类任意方法，能匹配，`*`代表的是方法
```java
execution(void com..*())
```

- `将项目中所有业务层方法的以find开头的方法匹配(常用)`
```java
execution(* com.blog.*.*Service.find*(..))
```

- `将项目中所有业务层方法的以save开头的方法匹配(常用)`
```java
execution(* com.blog.*.*Service.save*(..))
```

#### 4.1.3 书写技巧

对于切入点表达式的编写其实是很灵活的，那么在编写的时候，有没有什么好的技巧让我们用用:

- 所有代码按照标准规范开发，否则以下技巧全部失效
- 描述切入点通常`描述接口`，而不描述实现类,如果描述到实现类，就出现紧耦合了
- `访问控制修饰符`针对接口开发均采用public描述（`可省略访问控制修饰符描述`）
- `返回值类型`对于增删改类使用精准类型加速匹配，对于查询类使用`*`通配快速描述
- `包名`书写尽量不使用`..`匹配，效率过低，常用`*`做单个包描述匹配，或精准匹配
- `接口名/类名`书写名称与模块相关的采用`*`匹配，例如UserService书写成`*Service`，绑定业务层接口名
- 方法名书写以`动词`进行`精准匹配`，名词采用`*`匹配，例如`getById`书写成`getBy*`，`selectAll`书写成`select*`
- 参数规则较为复杂，根据业务方法灵活调整
- 通常`不使用异常`作为`匹配`规则

### 4.2 AOP通知类型

前面的案例中，有涉及到如下内容
```java
@Before("pt()")
```

它所代表的含义是将`通知`添加到`切入点`方法执行的`前面`。  

除了这个注解外，还有没有其他的注解，换个问题就是除了可以在前面加，能不能在其他的地方加?
#### 4.2.1 类型介绍

我们先来回顾下AOP通知:

- AOP通知描述了`抽取的共性功能`，根据共性功能抽取的位置不同，最终运行代码时要将其加入到合理的位置

那么具体可以将通知添加到哪里呢？一共提供了5种通知类型

- 前置通知
- 后置通知
- `环绕通知(重点)`
- 返回后通知(了解)
- 抛出异常后通知(了解)

为了更好的理解这几种通知类型，我们来看一张图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e407c4050918f42f3433a13b379458c9.png)


1. **前置通知**：追加功能到方法执行前,类似于在代码1或者代码2添加内容
2. **后置通知**：追加功能到方法执行后,**不管方法执行的过程中有没有抛出异常都会执行**，类似于在代码5添加内容
3. **返回后通知**：追加功能到方法执行后，只有方法正常执行结束后才进行,类似于在代码3添加内容，如果方法执行抛出异常，返回后通知的内容将不会被添加
4. **抛出异常后通知**：追加功能到方法抛出异常后，只有方法执行出异常才进行,类似于在代码4添加内容，只有方法抛出异常后才会被添加
5. **环绕通知**：环绕通知功能比较强大，它可以追加功能到方法执行的前后，这也是比较常用的方式，它可以实现其他四种通知类型的功能，具体是如何实现的，需要我们往下学习。
#### 4.2.2 环境准备

- 创建一个Maven项目
- pom.xml添加Spring依赖
- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    public void update();  
  
    public int select();  
}

@Repository  
public class BookDaoImpl implements BookDao {  
      
    public void update() {  
        System.out.println("book dao update ...");  
    }  
  
    public int select() {  
        System.out.println("book dao select ...");  
        return 100;  
    }  
}
```

- 创建Spring的配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- 创建通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    public void before() {  
        System.out.println("before advice ...");  
    }  
  
    public void after(){  
        System.out.println("after advice ...");  
    }  
  
    public void around(){  
        System.out.println("around before advice ...");  
        System.out.println("around after advice ...");  
    }  
  
    public void afterReturning(){  
        System.out.println("afterReturning advice ...");  
    }  
  
    public void afterThrowing(){  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```
#### 4.2.3 通知类型的使用

- `前置通知`
	修改MyAdvice，在before方法上添加`@Before`注解
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void before() {  
        System.out.println("before advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30e03ef2bead767d700f0ef7cabee6c6.png)


- `后置通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void before() {  
        System.out.println("before advice ...");  
    }  
  
    @After("pt()")  
    public void after(){  
        System.out.println("after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5a1bdab97b8be16fbd7ff338d80222a.png)


- `环绕通知`：基本使用
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(){  
        System.out.println("around before advice ...");  
        System.out.println("around after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fce365cec03a337b051fedf5dc31b4e2.png)


运行结果中，通知的内容打印出来，但是原始方法的内容却没有被执行。

因为环绕通知需要在原始方法的前后进行增强，所以环绕通知就必须要能对原始操作进行调用，具体如何实现?

- 在方法参数中添加`ProceedingJoinPoint`，同时在需要的位置使用`proceed()`调用原始操作
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        //表示对原始操作的调用  
        pjp.proceed();  
        System.out.println("around after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fe6622bc62f92faa57a048a46459a18e.png)


- 注意事项

- `当原始方法中有返回值时`
- 修改MyAdvice,对BookDao中的select方法添加环绕通知
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        pjp.proceed();  
        System.out.println("around after advice ...");  
    }  
}
```

- 修改App类，调用select方法
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        int select = bookDao.select();  
        System.out.println(select);  
    }  
}
```

- 运行程序，报错![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9378dd38244760bedb101a2315e3f177.png)
	- void就是返回Null
	- 原始方法的返回值是BookDao下的select方法

- 所以如果我们使用环绕通知的话，要根据原始方法的返回值来设置环绕通知的返回值，具体解决方案为:
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        Object res = pjp.proceed();  
        System.out.println("around after advice ...");  
        return res;  
    }  
}
```

- 运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/573fb03039dbeb3795a53628d6cb68a4.png)


- `说明`:
	- 为什么返回的是Object而不是int的主要原因是Object类型更通用。
	- 在环绕通知中是可以对原始方法返回值就行修改的。例如上面的例子，可以改为`return res+666;`，最终的输出结果也会变为766
	- 注意类型转换！

- `返回后通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @AfterReturning("pt()")  
    public void afterReturning() {  
        System.out.println("afterReturning advice ...");  
    }  
}
```

- 运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5a4841c5f6446005f6660d8acc56fb35.png)


- `注意`：  
	- 返回后通知是需要在原始方法`select`正常执行后才会被执行，如果`select()`方法执行的过程中出现了异常，那么返回后通知是不会被执行。后置通知是不管原始方法有没有抛出异常都会被执行。  

- 现在我们在select()方法中加一个异常
```java
public int select() {  
    System.out.println("book dao select ...");  
    int a = 1 / 0;  
    return 100;  
}
```

- 运行程序，输出如下，没有输出`afterReturning advice ...`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e37b41d4461e040caa4da5e3587d12a.png)


- 我们再换成后置输出，运行程序，结果如下，输出了`after advice ...`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/acb6a0660010ffe8cc71bbf81cfdb5ce.png)


- `异常后通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @AfterThrowing("pt()")  
    public void afterThrowing() {  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e91ef3d72d530d3f19dba986d159536a.png)


学习完这5种通知类型，我们来思考下`环绕通知是如何实现其他通知类型的功能的?`

因为环绕通知是可以控制原始方法执行的，所以我们把增强的代码写在调用原始方法的不同位置就可以实现不同的通知类型的功能，如![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0bb6e7e9543c9b7c6694b2b8f4d3560.png)
#### 4.2.4 通知类型总结

知识点1：`@After`

|名称|@After|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法后运行|

知识点2：`@AfterReturning`

|名称|@AfterReturning|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法正常执行完毕后执行|

知识点3：`@AfterThrowing`

|名称|@AfterThrowing|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法运行抛出异常后执行|

知识点4：`@Around`

|名称|@Around|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前后运行|

知识点5：`@Before`

|名称|@Before|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行|

环绕通知注意事项

1. **环绕通知必须依赖形参ProceedingJoinPoint才能实现对原始方法的调用**，进而实现原始方法调用前后同时添加通知
2. 通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行
3. 对原始方法的调用可以不接收返回值，通知方法设置成void即可，**如果接收返回值，最好设定为Object类型**
4. 原始方法的返回值如果是void类型，通知方法的返回值类型可以设置成void,也可以设置成Object
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须要处理Throwable异常

介绍完这么多种通知类型，具体该选哪一种呢?

我们可以通过一些案例加深下对通知类型的学习。

### 4.3 案例1：业务层接口执行效率

#### 4.3.1 需求分析

这个需求也比较简单，前面我们在介绍AOP的时候已经演示过:

- 需求:任意业务层接口执行均可显示其执行效率（执行时长）  
    这个案例的目的是查看每个业务层执行的时间，这样就可以监控出哪个业务比较耗时，将其查找出来方便优化。

具体实现的思路:

1. 开始执行方法之前记录一个时间
2. 执行方法
3. 执行完方法之后记录一个时间
4. 用后一个时间减去前一个时间的差值，就是我们需要的结果。

所以要在方法执行的前后添加业务，经过分析我们将采用`环绕通知`。  
**说明:** 原始方法如果只执行一次，时间太快，两个时间差可能为0，所以我们要执行万次来计算时间差。
#### 4.3.2 环境准备

- 创建一个Maven项目
    
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>  
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.46</version>  
</dependency>  
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
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>  
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.12</version>  
    <scope>test</scope>  
</dependency>
```

- 创建数据库与表
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




- 添加AccountService、AccountServiceImpl、AccountDao与Account类
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

public interface AccountDao {  
    @Insert("insert into tbl_account(`name`,money) values(#{name},#{money}) ")  
    void save(Account account);  
  
    @Update("update tbl_account set `name`=#{name},money=#{money}")  
    void update(Account account);  
  
    @Select("select * from tbl_account")  
    List<Account> findAll();  
  
    @Select("select * from tbl_account where id=#{id}")  
    Account findById(Integer id);  
}

public interface AccountService {  
    void save(Account account);  
    void update(Account account);  
    void delete(Integer id);  
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
  
    public void update(Account account) {  
        accountDao.update(account);  
    }  
  
    public void delete(Integer id) {  
        accountDao.delete(id);  
    }  
  
    public List<Account> findAll() {  
        return accountDao.findAll();  
    }  
  
    public Account findById(Integer id) {  
        return accountDao.findById(id);  
    }  
}
```

- resources下提供一个jdbc.properties
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db?useSSL=false  
jdbc.username=root  
jdbc.password=poassword.
```

- 创建相关配置类
```java
//SpringConfig
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}

//JDBCConfig
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

//MyBatiesConfig
public class MyBatisConfig {  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {  
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();  
        sqlSessionFactoryBean.setTypeAliasesPackage("com.blog.domain");  
        sqlSessionFactoryBean.setDataSource(dataSource);  
        return sqlSessionFactoryBean;  
    }  
  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer() {  
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();  
        mapperScannerConfigurer.setBasePackage("com.blog.dao");  
        return mapperScannerConfigurer;  
    }  
}

//AccountServiceImpl
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    private AccountDao accountDao;  
    public void save(Account account) {  
        accountDao.save(account);  
    }  
  
    public void update(Account account) {  
        accountDao.update(account);  
    }  
  
    public void delete(Integer id) {  
        accountDao.delete(id);  
    }  
  
    public List<Account> findAll() {  
        return accountDao.findAll();  
    }  
  
    public Account findById(Integer id) {  
        return accountDao.findById(id);  
    }  
}
```

- 编写Spring整合Junit的测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class AccountServiceTestCase {  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void testFindById(){  
        Account account = accountService.findById(2);  
    }  
  
    @Test  
    public void testFindAll(){  
        List<Account> accountList = accountService.findAll();  
    }  
}
```
#### 4.3.3 功能开发

- `步骤一：`开启SpringAOP的注解功能  
	在Spring的主配置文件SpringConfig类中添加注解
```java
@EnableAspectJAutoProxy
```

- `步骤二：`创建AOP的通知类
	- 该类要被Spring管理，需要添加`@Component`
	- 要标识该类是一个AOP的切面类，需要添加`@Aspect`
	- 配置切入点表达式，需要添加一个方法，并添加`@Pointcut`
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    public void runSpeed() {  
  
    }  
}
```

- `步骤三：`添加环绕通知  
	在runSpeed()方法上添加@Around
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    @Around("servicePt()")  
    public Object runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
  
        Object res = proceedingJoinPoint.proceed();  
        return res;  
    }  
}
```

- `步骤四：`完成核心业务，记录万次执行的时间
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    @Around("servicePt()")  
    public void runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
        long start = System.currentTimeMillis();  
        for (int i = 0; i < 10000; i++) {  
            proceedingJoinPoint.proceed();  
        }  
        long end = System.currentTimeMillis();  
        System.out.println("业务层接口万次执行时间: " + (end - start) + "ms");  
    }  
}
```

- `步骤五：`运行单元测试类  
	运行结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4d04e14a0a4837df58a7033fc13f2e1b.png)


- `步骤六：` 程序优化  
	目前还存在一个问题，当我们一次执行多个方法时，控制台输出的都是`业务层接口万次执行时间: XXXms`  
	
	我们无法得知具体哪个方法的耗时，那么该如何优化呢？
	  
	ProceedingJoinPoint中有一个getSignature()方法来获取签名，然后调用`getDeclaringTypeName`可以获取类名，`getName()`可以获取方法名
```java
@Around("servicePt()")  
public void runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
    Signature signature = proceedingJoinPoint.getSignature();  
    String typeName = signature.getDeclaringTypeName();  
    String methodName = signature.getName();  
    long start = System.currentTimeMillis();  
    for (int i = 0; i < 10000; i++) {  
        proceedingJoinPoint.proceed();  
    }  
    long end = System.currentTimeMillis();  
    System.out.println("万次执行 " + typeName + "." + methodName + " 耗时" + (end - start) + "ms");  
}
```

再次运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d49f3c36555745d494f0f9681308f920.png)

- `说明`
	- 当前测试的接口执行效率仅仅是一个理论值，并不是一次完整的执行过程。  
	- 这块只是通过该案例把AOP的使用进行了学习，具体的实际值是有很多因素共同决定的。

### 4.4 案例2：AOP通知获取数据

目前我们写AOP仅仅是在原始方法前后追加一些操作，接下来我们要说说`AOP中数据相关的内容`，我们将从`获取参数`、`获取返回值`和`获取异常`三个方面来研究切入点的相关信息。  
前面我们介绍通知类型的时候总共讲了五种，那么对于这五种类型都会有参数，返回值和异常吗?  
我们先来逐一分析下:

- 获取切入点方法的参数，`所有的通知类型都可以获取参数`
    - `JoinPoint`：适用于前置、后置、返回后、抛出异常后通知
    - `ProceedingJoinPoint`：适用于环绕通知
- 获取切入点方法返回值，`前置和抛出异常后通知是没有返回值，后置通知可有可无`，所以不做研究
    - 返回后通知
    - 环绕通知
- 获取切入点方法运行异常信息，`异常信息前置和返回后通知是不会有，后置通知可有可无`，所以不做研究
    - 抛出异常后通知
    - 环绕通知
#### 4.4.1 环境准备

- 创建一个maven项目
- 添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```

- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    String findName(int id);  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id) {  
        System.out.println("id:" + id);  
        return "TestName";  
    }  
}
```

- 创建Spring配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- 编写通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Before("pt()")  
    public void before(){  
        System.out.println("before advice ...");  
    }  
  
    @After("pt()")  
    public void after(){  
        System.out.println("after advice ...");  
    }  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object res = pjp.proceed();  
        return res;  
    }  
  
    @AfterReturning("pt()")  
    public void afterReturning(){  
        System.out.println("afterReturning advice ...");  
    }  
  
    @AfterThrowing("pt()")  
    public void afterThrowing(){  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        String name = bookDao.findName(9527);  
        System.out.println(name);  
    }  
}
```
#### 4.4.2 获取参数

- `非环绕通知获取方式`  
	在方法上添加JoinPoint，通过JoinPoint来获取方法参数
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Before("pt()")  
    public void before(JoinPoint joinPoint){  
        Object[] args = joinPoint.getArgs();  
        System.out.println(Arrays.toString(args));  
        System.out.println("before advice ...");  
    }  
}
```

运行App类，可以获取如下内容，说明参数9527已经被获取![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fafab1c8af0267a622effd0039c66655.png)


- **思考:方法的参数只有一个，为什么获取的是一个数组?**
    - 因为参数的个数是不固定的，所以使用数组更通配些。
    - 如果将参数改成两个会是什么效果呢?

- 修改BookDao和BookDaoImpl类
```java
public interface BookDao {  
    String findName(int id, String name) {  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id, String name) {  
        System.out.println("id:" + id);  
        return "TestName";  
    }  
}
```

- 修改App类，调用方法传入多个参数
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        String name = bookDao.findName(9527,"Tony");  
        System.out.println(name);  
    }  
}
```

- 输出结果如下，两个参数都已经被获取到![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7c9a0b66aa0b92b770e07c929169e6a.png)



- `环绕通知获取方式`  
	环绕通知使用的是ProceedingJoinPoint，因为ProceedingJoinPoint是JoinPoint类的子类，所以对于ProceedingJoinPoint类中应该也会有对应的`getArgs()`方法，我们去验证下
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        Object res = pjp.proceed();  
        return res;  
    }  
}
```

- 运行App后查看运行结果，说明ProceedingJoinPoint也是可以通过getArgs()获取参数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61a4949eff3704c1f924f3f5d85508de.png)
`注意:`

- `pjp.proceed()方法是有两个构造方法，分别是`:
    - `proceed()`
    - `proceed(Object[] object)`
- 调用无参数的proceed，当原始方法有参数，会在调用的过程中自动传入参数
- 所以调用这两个方法的任意一个都可以完成功能
- 但是当需要修改原始方法的参数时，就只能采用带有参数的方法,如下
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = pjp.proceed(args);  
        return res;  
    }  
}
```

- 运行程序，输出结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/948d72ec30bd414cef9da577d5dde65c.png)


- 有了这个特性后，我们就`可以在环绕通知中对原始方法的参数进行拦截过滤`，避免由于参数的问题导致程序无法正确运行，还可以根据参数来给予不同的权限，提高代码的健壮性。
#### 4.4.3 获取返回值

对于返回值，只有返回后`AfterReturing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

- 环绕通知获取返回值
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = pjp.proceed(args); 
        return res;  
    }  
}
```
上述代码中，`res`就是方法的返回值，我们是可以直接获取，不但可以获取，如果需要还可以进行修改。

- 返回后通知获取返回值
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @AfterReturning(value = "pt()", returning = "res")  
    public void afterReturning(Object res){  
        System.out.println("afterReturning advice ..." + res);  
    }  
}
```

运行程序，输出如下，成功获取了返回值![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/769bdd520eb3fefe700ce38286f3fe9d.png)



- `几点注意`:

1. 参数名的问题  
    赋给returning的值，必须与Object类型参数名一致，上面的代码中均为`res`
2. afterReturning方法参数类型的问题  
    参数类型可以写成String，但是为了能匹配更多的参数类型，建议写成Object类型
3. afterReturning方法参数的顺序问题  
    `如果存在JoinPoint参数，则必须将其放在第一位，否则运行将报错`
```java
public void afterReturning(JoinPoint jp,Object res)
```
#### 4.4.4 获取异常(了解)

对于获取抛出的异常，只有抛出异常后`AfterThrowing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

- `环绕通知获取异常`  
    这块比较简单，以前我们是抛出异常，现在只需要将异常捕获，就可以获取到原始方法的异常信息了
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = null;  
        try {  
            res = pjp.proceed(args);  
        } catch (Throwable e) {  
            throw new RuntimeException(e);  
        }  
        return res;  
    }  
}
```
在catch方法中就可以获取到异常，至于获取到异常以后该如何处理，这个就和你的业务需求有关了。
    
- `抛出异常后通知获取异常`
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt() {  
    }  
  
    @AfterThrowing(value = "pt()", throwing = "throwable")  
    public void afterThrowing(Throwable throwable) {  
        System.out.println("afterThrowing advice ..." + throwable);  
    }  
}
```

- 那现在我们只需要让原始方法抛一个异常来看看效果
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id, String name) {  
        System.out.println("id:" + id);  
        int a = 1 / 0;  
        return "TestName";  
    }  
}
```

- 运行程序，输出如下，成功输出了异常![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ef8c843001ac39fbb3fa0b26ffeea4b.png)


- 至此，AOP通知如何获取数据就已经讲解完了，数据中包含`参数`、`返回值`、`异常(了解)`。

### 4.5 案例3：百度网盘密码数据兼容处理

#### 4.5.1 需求分析

需求：对百度网盘分享链接输入密码时尾部多输入的空格做兼容处理。  
问题描述：
- 当我们从别人发给我们的内容中复制提取码的时候，有时候会多复制到一些空格，直接粘贴到百度的提取码输入框
- 但是百度那边记录的提取码是没有空格的
- 这个时候如果不做处理，直接对比的话，就会引发提取码不一致，导致无法访问百度盘上的内容
- 所以多输入一个空格可能会导致项目的功能无法正常使用。
- 此时我们就想能不能将输入的参数先帮用户去掉空格再操作呢?
    - 答案是可以的，我们只需要在业务方法执行之前对所有的输入参数进行格式处理——trim()
- 那要对所有的参数都需要去除空格么?
    - 也没有必要，一般只需要针对字符串处理即可。

- 以后涉及到需要去除前后空格的业务可能会有很多，这个去空格的代码是每个业务都写么?
    - 可以考虑使用AOP来统一处理。
- AOP有五种通知类型，该使用哪种呢?
    - 我们的需求是将原始方法的参数处理后在参与原始方法的调用，能做这件事的就只有环绕通知。
#### 4.5.2 环境准备

- 创建一个Maven项目
    
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```

- 添加ResourcesDao和ResourcesDaoImpl，ResourcesService，ResourcesServiceImpl类
```java
public interface ResourceDao {  
    boolean readResource(String url,String password);  
}

public class ResourceDaoImpl implements ResourceDao {  
    public boolean readResource(String url, String password) {  
        //模拟校验  
        return password.equals("root");  
    }  
}

public interface ResourceService {  
    public boolean openURL(String url,String password);  
}

@Service  
public class ResourceServiceImpl implements ResourceService {  
    @Autowired  
    private ResourceDao resourceDao;  
    public boolean openURL(String url, String password) {  
        return resourceDao.readResource(url,password);  
    }  
}
```

- 创建Spring配置类
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        ResourceService service = context.getBean(ResourceService.class);  
        boolean flag = service.openURL("https://pan.baidu.com/xx", "root");  
        System.out.println(flag);  
    }  
}
```

现在项目的效果是，当输入密码为”root”控制台打印为true,如果密码改为”root “控制台打印的是false  

需求是使用AOP将参数进行统一处理，不管输入的密码`root`前后包含多少个空格，最终控制台打印的都是true

#### 4.5.3 具体实现

- `步骤一：`开启SpringAOP的注解功能
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- `步骤二：`编写通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        for (int i = 0; i < args.length; i++) {  
            if ((String.class).equals(args[i].getClass())){  
                args[i] = args[i].toString().trim();  
            }  
        }  
        Object res = pjp.proceed(args);  
        return res;  
    }  
}
```

- `步骤三：`运行程序  
    不管密码`root`前后是否加空格，最终控制台打印的都是true

## 5 AOP总结

### 5.1 AOP核心概念

- 概念：AOP(Aspect Oriented Programming)面向切面编程，一种编程范式
- 作用：在不惊动原始设计的基础上为方法进行功能`增强`
- 核心概念
    - 代理（Proxy）：SpringAOP的核心本质是采用`代理模式`实现的
    - 连接点（JoinPoint）：在SpringAOP中，理解为任意方法的执行
    - 切入点（Pointcut）：匹配连接点的式子，也是具有共性功能的方法描述
    - 通知（Advice）：若干个方法的共性功能，在切入点处执行，最终体现为一个方法
    - 切面（Aspect）：描述通知与切入点的对应关系
    - 目标对象（Target）：被代理的原始对象成为目标对象

### 5.2 切入点表达式

- 切入点表达式标准格式：动作关键字(访问修饰符 返回值 包名.类/接口名.方法名（参数）异常名)
```java
execution(* com.itheima.service.*Service.*(..))
```

- 切入点表达式描述通配符：
    - 作用：用于快速描述，范围描述
    - `*`：匹配任意符号（常用）
    - `..` ：匹配多个连续的任意符号（常用）
    - `+`：匹配子类类型

- 切入点表达式书写技巧
    1. 按`标准规范`开发
    2. 查询操作的返回值建议使用`*`匹配
    3. 减少使用`..`的形式描述包，效率低
    4. `对接口进行描述`，使用`*`表示模块名，例如UserService的匹配描述为`*Service`
    5. 方法名书写保留动词，例如get，使用`*`表示名词，例如getById匹配描述为getBy`*`
    6. 参数根据实际情况灵活调整
### 5.3 五种通知类型

- 前置通知
- 后置通知
- 环绕通知（重点）
    - 环绕通知依赖形参ProceedingJoinPoint才能实现对原始方法的调用
    - 环绕通知`可以隔离原始方法的调用执行`
    - 环绕通知返回值设置为Object类型
    - 环绕通知中可以对原始方法调用过程中出现的异常进行处理
- 返回后通知
- 抛出异常后通知
### 5.4 通知中获取参数

- 获取切入点方法的参数，所有的通知类型都可以获取参数
    - JoinPoint：适用于前置、后置、返回后、抛出异常后通知
    - ProceedingJoinPoint：适用于环绕通知
- 获取切入点方法返回值，前置和抛出异常后通知是没有返回值，后置通知可有可无，所以不做研究
    - 返回后通知
    - 环绕通知
- 获取切入点方法运行异常信息，前置和返回后通知是不会有，后置通知可有可无，所以不做研究
    - 抛出异常后通知
    - 环绕通知

## 6 AOP事务管理

### 6.1 Spring事务简介

#### 6.1.1 相关概念

相关概念

- 事务作用：在数据层`保障一系列的数据库操作同成功同失败`
- Spring事务作用：`在数据层或业务层`保障一系列的数据库操作同成功同失败

数据层有事务我们可以理解，为什么业务层也需要处理事务呢？举个简单的例子

- 转账业务会有两次数据层的调用，一次是加钱一次是减钱
- 把事务放在数据层，加钱和减钱就有两个事务
- 没办法保证加钱和减钱同时成功或者同时失败
- 这个时候就需要将事务放在业务层进行处理。

Spring为了管理事务，提供了一个`平台事务管理器` `PlatformTransactionManager`
```java
public interface PlatformTransactionManager extends TransactionManager {  
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;  
  
    void commit(TransactionStatus var1) throws TransactionException;  
  
    void rollback(TransactionStatus var1) throws TransactionException;  
}
```

commit是用来提交事务，rollback是用来回滚事务。

PlatformTransactionManager只是一个接口，Spring还为其提供了一个具体的实现:
```java
public class DataSourceTransactionManager extends AbstractPlatformTransactionManager implements ResourceTransactionManager, InitializingBean {  
    @Nullable  
    private DataSource dataSource;  
    private boolean enforceReadOnly;  
···  
···  
  
}
```

从名称上可以看出，我们`只需要给它一个DataSource对象`，它就可以帮你去在业务层管理事务。其`内部采用的是JDBC的事务`。所以说如果你持久层采用的是JDBC相关的技术，就可以采用这个事务管理器来管理你的事务。而Mybatis内部采用的就是JDBC的事务，所以后期我们Spring整合Mybatis就采用的这个`DataSourceTransactionManager`事务管理器。
#### 6.1.2 转账案例---需求分析

接下来通过一个案例来学习下Spring是如何来管理事务的。

先来分析下需求:

- 需求: 实现任意两个账户间转账操作
- 需求微缩: A账户减钱，B账户加钱

为了实现上述的业务需求，我们可以按照下面步骤来实现下:

1. 数据层提供基础操作，指定账户减钱（outMoney），指定账户加钱（inMoney）
2. 业务层提供转账操作（transfer），调用减钱与加钱的操作
3. 提供2个账号和操作金额执行转账操作
4. 基于Spring整合MyBatis环境搭建上述操作
#### 6.1.3 转账案例---环境搭建

- `步骤一：`准备数据表  
	Tom和Jerry初始金额都是1000
```sql
CREATE DATABASE spring_db CHARACTER SET utf8;  
USE spring_db;  
CREATE TABLE tbl_account(  
    id INT PRIMARY KEY AUTO_INCREMENT,  
    NAME VARCHAR(35),  
    money DOUBLE  
);  
INSERT INTO tbl_account(`name`,money) VALUES('Tom',1000),('Jerry',1000);
```

- `步骤二：`创建项目导入jar包
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
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

- `步骤三：`根据表创建模型类
```java
public class Account {  
    private Integer id;  
    private String name;  
    private Double money;  
  
    public Account() {  
    }  
  
    public Account(Integer id, String name, Double money) {  
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
  
    public Double getMoney() {  
        return money;  
    }  
  
    public void setMoney(Double money) {  
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

- `步骤四：`创建Dao接口
```java
public interface AccountDao {  
  
    @Update("update tbl_account set money = money + #{money} where name = #{name}")  
    void inMoney(@Param("name") String name,@Param("money") Double money);  
  
    @Update("update tbl_account set money = money - #{money} where name = #{name}")  
    void outMoney(@Param("name") String name, @Param("money") Double money);  
}
```

- `步骤五：`创建Service接口和实现类
```java
public interface AccountService {  
    /**  
     * 转账操作  
     * @param out 转出方  
     * @param in 转入方  
     * @param money 金额  
     */  
    public void transfer(String out,String in,Double money);  
}

@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        accountDao.inMoney(in, money);  
    }  
}
```

- `步骤六：`添加jdbc.properties文件
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db?useSSL=false  
jdbc.username=root  
jdbc.password=poassword.
```

- `步骤七：`创建JdbcConfig配置类
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

- `步骤八：`创建MybatisConfig配置类
```java
public class MyBatisConfig {  
  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){  
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();  
        factory.setTypeAliasesPackage("com.blog.domain");  
        factory.setDataSource(dataSource);  
        return factory;  
    }  
  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer(){  
        MapperScannerConfigurer msc = new MapperScannerConfigurer();  
        msc.setBasePackage("com.blog.dao");  
        return msc;  
    }  
}
```

- `步骤九:`创建SpringConfig配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤十：`编写测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class AccountServiceTest {  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void testTransfer() {  
        accountService.transfer("Tom", "Jerry", 100D);  
    }  
}
```
#### 6.1.4 事务管理

上述环境，运行单元测试类，会执行转账操作，`Tom`的账户会减少100，`Jerry`的账户会加100。

这是正常情况下的运行结果，但是如果在转账的过程中出现了异常，如
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        int a = 1 / 0;  
        accountDao.inMoney(in, money);  
    }  
}
```

这个时候就模拟了转账过程中出现异常的情况，此时进行转账，`Tom`的账户会减少100，而`Jerry`的账户却不会增加100  
那我们来分析一下刚才的结果

- 程序正常执行时，账户金额A减B加，没有问题
- 程序出现异常后，转账失败，但是异常之前操作成功，异常之后操作失败，整体业务失败

当程序出问题后，我们需要让事务进行回滚，而且这个事务应该是加在业务层上，而Spring的事务管理就是用来解决这类问题的。

Spring事务管理具体的实现步骤如下

- `步骤一：`在需要被事务管理的方法上添加`@Transactional`注解
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        int a = 1 / 0;  
        accountDao.inMoney(in, money);  
    }  
}
```

注意:`@Transactional`可以写在接口类上、接口方法上、实现类上和实现类方法上
- 写在接口类上，该接口的所有实现类的所有方法都会有事务
- 写在接口方法上，该接口的所有实现类的该方法都会有事务
- 写在实现类上，该类中的所有方法都会有事务
- 写在实现类方法上，该方法上有事务
- `建议写在接口的方法上，降低耦合`

- `步骤二：`在JdbcConfig类中配置事务管理器
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
  
    //配置事务管理器，mybatis使用的是jdbc事务  
    @Bean  
    public PlatformTransactionManager platformTransactionManager(DataSource dataSource){  
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();  
        transactionManager.setDataSource(dataSource);  
        return transactionManager;  
    }  
}
```

注意：事务管理器要根据使用技术进行选择，Mybatis框架使用的是JDBC事务，可以直接使用`DataSourceTransactionManager`

- `步骤三：`开启事务注解`@EnableTransactionManagement`
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@EnableTransactionManagement  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤四：`运行测试类  
    运行程序之后，我们去数据库查看Tom和Jerry的金额，发现没有变化  
    那么说明在转换的业务出现错误后，事务就可以控制回滚，保证数据的正确性。

知识点1：`@EnableTransactionManagement`

|名称|@EnableTransactionManagement|
|---|---|
|类型|配置类注解|
|位置|配置类定义上方|
|作用|设置当前Spring环境中开启注解式事务支持|

知识点2：`@Transactional`

|名称|@Transactional|
|---|---|
|类型|接口注解 类注解 方法注解|
|位置|业务层接口上方 业务层实现类上方 业务方法上方|
|作用|为当前业务层方法添加事务（如果设置在类或接口上方则类或接口中所有方法均添加事务）|
### 6.2 Spring事务角色

这部分我们重点要理解两个概念，分别是`事务管理员`和`事务协调员`。

当未开启Spring事务之前![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b07b03c1ed52b8e8fc1a37d938cbd45b.png)
- AccountDao的outMoney因为是修改操作，会开启一个事务T1
- AccountDao的inMoney因为是修改操作，会开启一个事务T2
- AccountService的transfer没有事务，
    - 运行过程中如果没有抛出异常，则T1和T2都正常提交，数据正确
    - 如果在两个方法中间抛出异常，T1因为执行成功提交事务，T2因为抛异常不会被执行
    - 就会导致数据出现错误

当开启Spring的事务管理后![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3645aa16de6686079f37219350d4f8c9.png)


- transfer上添加了@Transactional注解，在该方法上就会有一个事务T
- AccountDao的outMoney方法的事务T1加入到transfer的事务T中
- AccountDao的inMoney方法的事务T2加入到transfer的事务T中
- 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性。

通过上面例子的分析，我们就可以得到如下概念:

- 事务管理员：发起事务方，在Spring中`通常指代业务层开启事务的方法`
- 事务协调员：加入事务方，在Spring中`通常指代数据层方法`，也可以是业务层方法

注意：目前的事务管理是基于`DataSourceTransactionManager`和`SqlSessionFactoryBean`使用的是同一个数据源。
### 6.3 Spring事务属性

这部分我们主要学习三部分内容`事务配置`、`转账业务追加日志`、`事务传播行为`。
#### 6.3.1 事务配置

|属性|作用|示例|
|---|---|---|
|readOnly|设置是否为只读事务|readOnly = true 只读事务|
|timeout|设置事务超时时间|timeout = -1(永不超时)|
|rollbackFor|设置事务回滚异常(class)|rollbackFor{NullPointException.class}|
|rollbackForClassName|设置事务回滚异常（String)|同上格式为字符串|
|noRollbackFor|设置事务不回滚异常(class)|noRollbackFor{NullPointExceptior.class}|
|noRollbackForClassName|设置事务不回滚异常(String)|同上格式为字符串|
|isolation|设置事务隔离级别|isolation = Isolation. DEFAULT|
|propagation|设置事务传播行为|…|

上面这些属性都可以在`@Transactional`注解的参数上进行设置。

- readOnly：true只读事务，false读写事务，增删改要设为false,查询设为true。
    
- timeout:设置超时时间单位秒，在多长时间之内事务没有提交成功就自动回滚，-1表示不设置超时时间。
    
- rollbackFor:当出现指定异常进行事务回滚
    
- noRollbackFor:当出现指定异常不进行事务回滚
    
    - 思考:出现异常事务会自动回滚，这个是我们之前就已经知道的
        
    - noRollbackFor是设定对于指定的异常不回滚，这个好理解
        
    - rollbackFor是指定回滚异常，对于异常事务不应该都回滚么，为什么还要指定?
        
- 事实上Spring的事务只会对`Error异常`和`RuntimeException异常`及其子类进行事务回滚，其他的异常类型是不会回滚的，如下面的代码就不会回滚
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional  
    public void transfer(String out, String in, Double money) throws IOException {  
        accountDao.outMoney(out, money);  
        if (true) throw new IOException();  
        accountDao.inMoney(in, money);  
    }  
}
```
所以当我们运行程序之后，Tom会少100块钱，而Jerry不会多100块钱，这100块钱就凭空消失了

- 此时就可以使用rollbackFor属性来设置出现IOException异常回滚
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional(rollbackFor = {IOException.class})  
    public void transfer(String out, String in, Double money) throws IOException {  
        accountDao.outMoney(out, money);  
        if (true) throw new IOException();  
        accountDao.inMoney(in, money);  
    }  
}
```

- rollbackForClassName等同于rollbackFor,只不过属性为异常的类全名字符串
- noRollbackForClassName等同于noRollbackFor，只不过属性为异常的类全名字符串
- isolation设置事务的隔离级别
	- DEFAULT :默认隔离级别, 会采用数据库的隔离级别
	- READ_UNCOMMITTED : 读未提交
	- READ_COMMITTED : 读已提交
	- REPEATABLE_READ : 重复读取
	- SERIALIZABLE: 串行化

介绍完上述属性后，还有最后一个事务的传播行为，为了讲解该属性的设置，我们需要完成下面的案例。
#### 6.3.2 转账业务追加日志案例

##### 6.3.2.1 需求分析

- 在前面的转账案例的基础上添加新的需求，完成转账后记录日志。
    - 需求：实现任意两个账户间转账操作，并对每次转账操作在数据库进行留痕
    - 需求微缩：A账户减钱，B账户加钱，数据库记录日志

- 基于上述的业务需求，我们来分析下该如何实现：
    1. 基于转账操作案例添加日志模块，实现数据库中记录日志
    2. 业务层转账操作（transfer），调用减钱、加钱与记录日志功能

- 需要注意一点就是，我们这个案例的预期效果为:
    - `无论转账操作是否成功，均进行转账操作的日志留痕`
##### 6.3.2.2 环境准备

- `步骤一：`创建日志表
```SQL
create table tbl_log(  
   id int primary key auto_increment,  
   info varchar(255),  
   createDate datetime  
)
```

- `步骤二：`添加LogDao接口
```java
public interface LogDao {  
  
    @Insert("insert into tbl_log(info, createDate) VALUES (#{info},now())")  
    void log(String info);  
}
```

- `步骤三：`添加LogService接口和实现类
```java
public interface LogService {  
    void log(String out, String in, Double money);  
}

@Service  
public class LogServiceImpl implements LogService {  
    @Autowired  
    private LogDao logDao;  
  
    @Transactional  
    public void log(String out, String in, Double money) {  
        logDao.log(out + "向" + in + "转账" + money + "元");  
    }  
}
```

- `步骤四：`在转账的业务中添加记录日志
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
    @Autowired  
    protected LogService logService;  
  
    @Transactional(rollbackFor = {IOException.class})  
    public void transfer(String out, String in, Double money) throws IOException {  
        try {  
            accountDao.outMoney(out, money);  
            accountDao.inMoney(in, money);  
        } finally {  
            logService.log(out, in, money);  
        }  
    }  
}
```

- `步骤五：`运行程序
    
    - 当程序正常运行，tbl_account表中转账成功，tbl_log表中日志记录成功
    - 当转账业务之间出现异常(int i =1 / 0),转账失败，tbl_account成功回滚，但是tbl_log表未添加数据，说明也回滚了
    - 这个结果和我们想要的不一样，什么原因?该如何解决?
        - 失败原因：日志的记录与转账操作隶属同一个事务，同成功同失败（同回滚）
        - 解决方案：继续往下看
        - 预期效果：无论转账操作是否成功，日志必须保留

#### 6.3.3 事务传播行为

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/794ef04d222965baaba26cb088745478.png)



对于上述案例的分析:

- log方法、inMoney方法和outMoney方法都属于增删改，分别有事务T1,T2,T3
- transfer因为加了`@Transactional`注解，也开启了事务T
- 前面我们讲过Spring事务会把T1,T2,T3都加入到事务T中
- 所以当转账失败后，`所有的事务都回滚`，导致日志没有记录下来
- 这和我们的需求不符，这个时候我们就想能不能让log方法单独是一个事务呢?

要想解决这个问题，就需要用到事务传播行为，所谓的事务传播行为指的是:

- 事务传播行为：事务协调员对事务管理员所携带事务的处理态度。
    - 具体如何解决，就需要用到之前我们没有说的`propagation属性`。

- 修改logService改变事务的传播行为
```java
@Service  
public class LogServiceImpl implements LogService {  
    @Autowired  
    private LogDao logDao;  
  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    public void log(String out, String in, Double money) {  
        logDao.log(out + "向" + in + "转账" + money + "元");  
    }  
}
```

- 运行后，就能实现我们想要的结果，不管转账是否成功，都会记录日志。

事务传播行为的可选值

|传播属性|事务管理员|事务协调员|
|---|---|---|
|REQUIRED(默认)|开启T|加入T|
|---|无|新建T2|
|REQUIRES_NEW|开启T|新建T2|
|---|无|新建T2|
|SUPPORTS|开启T|加入T|
|---|无|无|
|NOT_SUPPORTED|开启T|无|
|---|无|无|
|MANDTORY|开启T|加入T|
|---|无|ERROR|
|NEVER|开启T|ERROR|
|---|无|无|
|NESTED|设置savePoint,一旦事务回滚，事务将回滚到savePoint处，交由客户响应提交/回滚|   |
---
文章标题: "[[4_SpringMVC]]" 
文章作者: Dakkk
文章概要: |
  介绍SpringMVC框架基础知识，包括MVC架构、入门案例、请求响应处理、RESTful风格开发和实际案例应用。
tags:
- "SpringMVC"
- "Web开发"
- "MVC架构"
- "RESTful"
- "JSON数据"
- "请求映射"
- "参数传递"
- "响应处理"
相关文章:
- "[[5_SSM整合]]"
- "[[1_Java值传递详解]]"
- "[[2_登录页加点盐：通过 Animate.css 添加入场动画]]"
- "[[2_函数]]"
- "[[2_Servlet]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/2_SSM（黑马）/4_SpringMVC.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:12:14
---

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
<!--而且SpringMVC的版本不能超过5,否则会报错-->
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

1. **服务器启动，执行ServletContainerInitConfig类，初始化web容器**
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

2. **执行createServletApplicationContext方法，创建了WebApplicationContext对象**
	- 该方法加载SpringMVC的配置类SpringMvcConfig来初始化SpringMVC的容
	```java
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    context.register(SpringMvcConfig.class);  
    return context;  
}
	```

3. **加载SpringMvcConfig配置类**
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

- !!!运行程序之前，我们还需要把`SpringMvcConfig`配置类上的`@ComponentScan`注解注释掉，否则不会报错，将正常输出 , 出现问题的原因是
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

关于请求参数的传递与接收是**和请求方式有关系的**，目前比较常见的两种请求方式为：
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
- http://localhost/user/commonParam?name=Jerry
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

- `步骤四：`查看控制台，输出如下
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e02a09e240c6cbf16a38e5953e2d2b3.png)

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

因为异步调用是目前常用的主流方式，所以我们需要**更关注的就是如何返回JSON数据**，对于其他只需要认识了解即可。
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

- 访问`http://localhost:8080/toJsonPojo`，页面上成功出现JSON类型数据
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1a869f2276ad503510194ca5ed037b2.png)


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

- 访问`http://localhost:8080/toJsonList`，页面上成功出现JSON集合类型数据![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d0c702d682e4f7c9452b67479530c92.png)

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
@RequestMapping(value = "/users",method = RequestMethod.DELETE)  
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
<!--而且SpringMVC的版本不能超过5,否则会报错-->
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

1. **服务器启动，执行ServletContainerInitConfig类，初始化web容器**
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

2. **执行createServletApplicationContext方法，创建了WebApplicationContext对象**
	- 该方法加载SpringMVC的配置类SpringMvcConfig来初始化SpringMVC的容
	```java
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    context.register(SpringMvcConfig.class);  
    return context;  
}
	```

3. **加载SpringMvcConfig配置类**
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

- !!!运行程序之前，我们还需要把`SpringMvcConfig`配置类上的`@ComponentScan`注解注释掉，否则不会报错，将正常输出 , 出现问题的原因是
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

关于请求参数的传递与接收是**和请求方式有关系的**，目前比较常见的两种请求方式为：
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
- http://localhost/user/commonParam?name=Jerry
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

- `步骤四：`查看控制台，输出如下
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e02a09e240c6cbf16a38e5953e2d2b3.png)

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

因为异步调用是目前常用的主流方式，所以我们需要**更关注的就是如何返回JSON数据**，对于其他只需要认识了解即可。
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

- 访问`http://localhost:8080/toJsonPojo`，页面上成功出现JSON类型数据
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1a869f2276ad503510194ca5ed037b2.png)


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

- 访问`http://localhost:8080/toJsonList`，页面上成功出现JSON集合类型数据![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d0c702d682e4f7c9452b67479530c92.png)

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
@RequestMapping(value = "/users",method = RequestMethod.DELETE)  
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
## 1 AOP简介

前面我们在介绍Spring的时候说过，Spring有两个核心的概念，一个是`IOC/DI`，一个是`AOP`

前面已经对`IOC/DI`进行了系统的学习，接下来要学习它的另一个核心内容，就是`AOP`。

`AOP是在不改原有代码的前提下对其进行增强`。

那现在我们围绕这句话，主要学习两方面内容`AOP核心概念`,`AOP作用`
### 1.1 什么是AOP？

- AOP(Aspect Oriented Programming)面向切面编程，是一种编程范式，指导开发者如何组织程序结构
- OOP(Object Oriented Programming)面向对象编程

我们都知道OOP是一种编程思想，那么AOP也是一种编程思想，编程思想主要的内容就是指导程序员该如何编写程序，所以它们两个是不同的`编程范式`。
### 1.2 AOP的作用

- 作用:在不惊动原始设计的基础上为其进行`功能增强`
### 1.3 AOP核心概念

为了能更好的理解AOP的相关概念，我们准备了一个环境，整个环境的内容我们暂时可以不用关注，最主要的类为:`BookDaoImpl`
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        //记录程序当前执行执行（开始时间）  
        Long startTime = System.currentTimeMillis();  
        //业务执行万次  
        for (int i = 0;i<10000;i++) {  
            System.out.println("book dao save ...");  
        }  
        //记录程序当前执行时间（结束时间）  
        Long endTime = System.currentTimeMillis();  
        //计算时间差  
        Long totalTime = endTime-startTime;  
        //输出信息  
        System.out.println("执行万次消耗时间：" + totalTime + "ms");  
    }  
    public void update(){  
        System.out.println("book dao update ...");  
    }  
    public void delete(){  
        System.out.println("book dao delete ...");  
    }  
    public void select(){  
        System.out.println("book dao select ...");  
    }  
}
```

- 代码的内容很简单，就是测试一下万次执行的耗时  
当在App类中从容器中获取bookDao对象后，分别执行其`save`,`delete`,`update`和`select`方法后会有如下的打印结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ba2defabc13303c7bc90d01d958946b.png)


- `疑问`
	- 对于计算万次执行消耗的时间只有save方法有，为什么delete和update方法也会有呢?
	- delete和update方法有，那什么select方法为什么又没有呢?

- 这个案例中其实就使用了Spring的AOP，在不惊动(改动)原有设计(代码)的前提下，想给谁添加额外功能就给谁添加。这个也就是Spring的理念：
	- `无入侵式！无侵入式!`

说了这么多，Spring到底是如何实现的呢?![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/635327f4734b3a66b75ec86cbef83b4c.png)


1. 前面一直在强调，Spring的AOP是对一个类的方法在不进行任何修改的前提下实现增强。对于上面的案例中BookServiceImpl中有`save`,`update`,`delete`和`select`方法,这些方法我们给起了一个名字叫`连接点`
2. 在BookServiceImpl的四个方法中，`update`和`delete`只有打印没有计算万次执行消耗时间，但是在运行的时候已经有该功能，那也就是说`update`和`delete`方法都已经被增强，所以对于需要增强的方法我们给起了一个名字叫`切入点`
3. 执行BookServiceImpl的update和delete方法的时候都被添加了一个计算万次执行消耗时间的功能，将这个功能抽取到一个方法中，换句话说就是存放共性功能的方法，我们给起了个名字叫`通知`
4. 通知是要增强的内容，会有多个，切入点是需要被增强的方法，也会有多个，那哪个切入点需要添加哪个通知，就需要提前将它们之间的关系描述清楚，那么对于通知和切入点之间的关系描述，我们给起了个名字叫`切面`
5. 通知是一个方法，方法不能独立存在需要被写在一个类中，这个类我们也给起了个名字叫`通知类`

至此AOP中的核心概念就已经介绍完了，总结下:

- `连接点(JoinPoint)`：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等
    - `在SpringAOP中，理解为方法的执行`
- `切入点(Pointcut)`:匹配连接点的式子
    - 在SpringAOP中，一个切入点可以描述一个具体方法，也可也匹配多个方法
        - **一个具体的方法**:如com.blog.dao包下的BookDao接口中的无形参无返回值的save方法
        - **匹配多个方法**:所有的save方法/所有的get开头的方法/所有以Dao结尾的接口中的任意方法/所有带有一个参数的方法
    - **连接点范围要比切入点范围大**，是切入点的方法也一定是连接点，但是是连接点的方法就不一定要被增强，所以可能不是切入点。
- `通知(Advice)`:在切入点处执行的操作，也就是共性功能
    - 在SpringAOP中，功能最终以方法的形式呈现
- `通知类：定义通知的类`
- `切面(Aspect):描述通知与切入点的对应关系`
### 1.4 小结

这部分需要掌握的内容是

- 什么是AOP?
- AOP的作用是什么?
- AOP中核心概念分别指的是什么?
    - 连接点
    - 切入点
    - 通知
    - 通知类
    - 切面
## 2 AOP入门案例

### 2.1 需求分析

案例设定：测算接口执行效率，但是这个案例稍微复杂了点，我们对其进行简化。  
简化设定：在方法执行前输出当前系统时间。  
那现在我们使用SpringAOP的注解方式完成在方法执行的前打印出当前系统时间。
### 2.2 思路分析

1. 导入坐标
2. 制作连接点（原始操作，Dao接口及实现类）
3. 制作共性功能（通知类和通知）
4. 定义切入点
5. 绑定切入点和通知的关系（切面）
### 2.3 环境准备

- 创建一个Maven项目
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    public void save();  
    public void update();  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println(System.currentTimeMillis());  
        System.out.println("book dao save ...");  
    }  
  
    public void update() {  
        System.out.println("book dao update ...");  
    }  
}
```

- 创建Spring的配置类
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```

**说明:**

- 目前打印save方法的时候，因为方法中有打印系统时间，所以运行的时候是可以看到系统时间
- 对于update方法来说，就没有该功能
- 我们要使用SpringAOP的方式在不改变update方法的前提下让其具有打印系统时间的功能。
### 2.4 AOP实现步骤

- `步骤一：`添加依赖
```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```
- 因为`spring-context`中已经导入了`spring-aop`,所以不需要再单独导入`spring-aop`  
    导入AspectJ的jar包,AspectJ是AOP思想的一个具体实现，Spring有自己的AOP实现，但是相比于AspectJ来说比较麻烦，所以我们`直接采用Spring整合ApsectJ的方式进行AOP开发`。
    
- `步骤二：`定义接口和实现类  
    准备环境的时候我们已经完成了
    
- `步骤三：`定义通知类和通知  
    通知就是将共性功能抽取出来后形成的方法，共性功能指的就是当前系统时间的打印。  
    类名和方法名没有要求，可以任意。
```java
public class MyAdvice {  
    public void method(){  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤四：`定义切入点  
	BookDaoImpl中有两个方法，分别是update()和save()，我们要增强的是update方法，那么该如何定义呢？
```java
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

**说明:**

- 切入点定义依托一个不具有实际意义的方法进行，即无参数、无返回值、方法体无实际逻辑。
- execution及后面编写的内容，之后我们会专门去学习。

- `步骤五：`制作切面  
	切面是用来描述通知和切入点之间的关系，如何进行关系的绑定?
```java
public class MyAdvice {  
  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt(){}  
      
    @Before("pt()")  
    public void method(){  
        System.out.println(System.currentTimeMillis());  
    }  
}
```
绑定切入点与通知关系，并指定通知添加到原始连接点的具体执行`位置`

- **说明:**`@Before`翻译过来是之前，也就是说通知会在切入点方法执行之前执行，除此之前还有其他四种类型，后面会讲。  

- 那这里就会在执行update()之前，来执行我们的method()，输出当前毫秒值

- `步骤六：`将通知类配给容器并标识其为切面类
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤七：`开启注解格式AOP功能  
	使用`@EnableAspectJAutoProxy`注解
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- `步骤八：`运行程序  
	这次我们再来调用update()
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```

- 控制台成功输出了当前毫秒值![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24b3bd768460ca3c4ce623644a595058.png)


知识点1：`@EnableAspectJAutoProxy`

| 名称  | @EnableAspectJAutoProxy |
| --- | ----------------------- |
| 类型  | **配置类注解**               |
| 位置  | 配置类定义上方                 |
| 作用  | 开启注解格式AOP功能             |

知识点2：`@Aspect`

| 名称  | @Aspect      |
| --- | ------------ |
| 类型  | **类注解**      |
| 位置  | 切面类定义上方      |
| 作用  | 设置当前类为AOP切面类 |

知识点3：`@Pointcut`

|名称|@Pointcut|
|---|---|
|类型|方法注解|
|位置|切入点方法定义上方|
|作用|设置切入点方法|
|属性|value（默认）：切入点表达式|

知识点4：`@Before`

|名称|@Before|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行|
## 3 AOP工作流程

AOP的入门案例已经完成，对于刚才案例的执行过程，我们就得来分析分析，这一节我们主要讲解两个知识点:`AOP工作流程`和`AOP核心概念`。其中核心概念是对前面核心概念的补充。
### 3.1 AOP工作流程

由于AOP是基于Spring容器管理的bean做的增强，所以整个工作过程需要从Spring加载bean说起

- `流程一`：Spring容器启动
    - 容器启动就需要去加载bean,哪些类需要被加载呢?
    - `需要被增强的类，如:BookServiceImpl`
    - `通知类，如:MyAdvice`
    - **注意此时bean对象还没有创建成功**
- `流程二`：读取所有切面配置中的切入点
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void ptx() {  
    }  
  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```
上面这个例子中有两个切入点的配置，但是第一个`ptx()`并没有被使用，所以不会被读取。

- `流程三`：初始化bean，判定bean对应的类中的方法（连接点）是否匹配到任意切入点
    - 注意第一步在容器启动的时候，bean对象还没有被创建成功。
    - 要被实例化bean对象的类中的连接点（方法）和切入点进行匹配![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42cba4fb3bc728f2ff93e910f9bc112b.png)
    - 匹配失败，创建原始对象，如`UserDao`
    - 匹配失败说明不需要增强，直接调用原始对象的方法即可。

- 匹配成功，创建原始对象（`目标对象`）的`代理`对象，如:`BookDao`
    - 匹配成功说明需要对其进行增强
    - 对哪个类做增强，这个类对应的对象就叫做目标对象
    - 因为要对目标对象进行功能增强，而采用的技术是动态代理，所以会为其创建一个代理对象（**此时的bean为它的代理对象**）
    - 最终运行的是代理对象的方法，在该方法中会对原始方法进行功能增强

- `流程四`：获取bean执行方法
    - 获取的bean是原始对象时，调用方法并执行，完成操作
    - 获取的bean是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作


- 下面我们来验证一下容器中是否为代理对象
    - 如果目标对象中的方法`会被增强`，那么容器中将存入的是目标对象的`代理对象`
    - 如果目标对象中的方法`不被增强`，那么容器中将存入的是目标`对象本身`

- `步骤一：`修改App运行类，获取类的类型并输出
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        System.out.println(bookDao);  
        System.out.println(bookDao.getClass());  
    }  
}
```

- `步骤二：`修改MyAdvice类，改为不增强  
	将定义的切入点改为`updatexxx`，而BookDaoImpl类中不存在该方法，所以BookDao中的update方法在执行的时候，就不会被增强  
	所以此时容器中的对象应该是目标对象本身。
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.updatexxx())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤三：`运行程序  
	输出结果如下，确实是目标对象本身，符合我们的预期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/905e5aa1ac40c44bdc3b6631402855b2.png)


- `步骤四：`修改MyAdvice类，改为增强  
	将定义的切入点改为`update`，那么BookDao中的update方法在执行的时候，就会被增强  
	所以容器中的对象应该是目标对象的代理对象
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```

- `步骤五：`运行程序![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/377d2ba8775b27e94af252bb2da04dda.png)



至此对于刚才的结论，我们就得到了验证，这块我们需要注意的是:

📝📝：`不能直接打印对象`，从上面两次结果中可以看出，直接打印对象走的是对象的toString方法，不管是不是代理对象，打印的结果都是一样的，`原因是内部对toString方法进行了重写`。
### 3.2 AOP核心概念

在上面介绍AOP的工作流程中，我们提到了两个核心概念，分别是:

- `目标对象(Target)`：原始功能去掉共性功能对应的类产生的对象，这种对象是无法直接完成最终工作的
- `代理(Proxy)`：目标对象无法直接完成工作，需要对其进行功能回填，通过原始对象的代理对象实现

上面这两个概念比较抽象，简单来说

目标对象就是要增强的类`如:BookServiceImpl类`对应的对象，也叫原始对象，不能说它不能运行，只能说它在运行的过程中对于要增强的内容是缺失的

SpringAOP是在不改变原有设计(代码)的前提下对其进行增强的，它的底层采用的是代理模式实现的，所以要对原始对象进行增强，就需要对原始对象创建代理对象，在代理对象中的方法把通知`如:MyAdvice中的method方法`内容加进去，就实现了增强，这就是我们所说的代理(Proxy)
### 3.3 小结

这部分我们需要掌握的内容有：

- `能说出AOP的工作流程`
- AOP的核心概念
    - 目标对象、连接点、切入点
    - 通知类、通知
    - 切面
    - 代理
- SpringAOP的本质或者可以说底层实现是通过`代理模式`。

## 4 AOP配置管理

### 4.1 AOP切入点表达式

前面我们已经接触过了切入点表达式，下面我们来具体学习一下
```java
@Pointcut("execution(void com.blog.dao.impl.BookDaoImpl.update())")
```

对于AOP中切入点表达式，我们总共会学习三个内容，分别是`语法格式`、`通配符`和`书写技巧`。
#### 4.1.1 语法格式

首先我们先要明确两个概念:

- `切入点`:要进行增强的方法
- `切入点表达式`:要进行增强的方法的描述方式

对于切入点的描述，我们其实是有两种方式的，先来看下前面的例子  
由于BookDaoImpl类实现了BookDao接口，那么有如下两种方式来描述

- `描述方式一`：执行com.blog.dao包下的BookDao接口中的无参数update方法
```java
execution(void com.blog.dao.BookDao.update())
```

- 描述方式二：执行com.blog.dao.`impl包下的BookDaoImpl类中的无参数update方法`
```java
execution(void com.blog.dao.impl.BookDaoImpl.update())
```

📝📝📝：因为调用接口方法的时候最终运行的还是其实现类的方法，所以上面两种描述方式都是可以的。

对于切入点表达式的语法为:

- 切入点表达式标准格式：动作关键字(访问修饰符 返回值 包名.类/接口名.方法名(参数) 异常名）  
    对于这个格式，我们不需要硬记，通过一个例子，理解它:
```java
execution(public User com.blog.service.UserService.findById(int))
```

- `execution`：`动作关键字`，描述切入点的行为动作，例如execution表示执行到指定切入点
- `public`:`访问修饰符`,还可以是public，private等，`可以省略`
- `User`：`返回值`，写返回值类型
- `com.blog.service`：`包名`，多级包使用点连接
- `UserService`:`类/接口名称`
- `findById`：`方法名`
- `int`:`参数`，直接写参数的类型，多个类型用逗号隔开
- `异常名`：方法定义中`抛出指定异常`，`可以省略`

切入点表达式就是要找到需要增强的方法，所以它就是对一个具体方法的描述，但是方法的定义会有很多，所以如果每一个方法对应一个切入点表达式，想想这块就会觉得将来编写起来会比较麻烦，有没有更简单的方式呢?

- 使用通配符
#### 4.1.2 通配符

我们使用通配符描述切入点，`主要的目的就是简化之前的配置`，具体都有哪些通配符可以使用?

- `*`:`单个`独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现  
    匹配com.blog包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法
```java
execution（public * com.blog.*.UserService.find*(*))
```

- `..`：`多个`连续的任意符号，可以独立出现，常用于`简化包名与参数的书写` 
	匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法
```java
execution（public User com..UserService.findById(..))
```

- `+`：专用于匹配子类类型  
	这个使用率较低，描述子类的，`*Service+`，表示所有以Service结尾的接口的子类
```java
execution(* *..*Service+.*(..))
```

- 下面我们来具体分析一下各种用法

- 匹配接口，能匹配到
```java
execution(void com.blog.dao.BookDao.update())
```

- 匹配实现类，能匹配到
```java
execution(void com.blog.dao.impl.BookDaoImpl.update())
```

- **返回值任意**，能匹配到
```java
execution(* com.blog.dao.impl.BookDaoImpl.update())
```

- 返回值任意，但是update方法必须要有一个参数，无法匹配，要想匹配需要在update接口和实现类添加一个参数
```java
execution(* com.blog.dao.impl.BookDaoImpl.update(*))
```

- 返回值为void，com包下的任意包三层包下的任意类的update方法，匹配到的是实现类，能匹配
```java
execution(void com.*.*.*.*.update())
```

- 返回值为void,com包下的任意两层包下的任意类的update方法，匹配到的是接口，能匹配
```java
execution(void com.*.*.*.update())
```

- 返回值为void，方法名是update的任意包下的任意类，能匹配
```java
execution(void *..update())
```

- 匹配项目中任意类的任意方法，能匹配，但是不建议使用这种方式，影响范围广
```java
execution(* *..*(..))
```

- 匹配项目中任意包任意类下只要以u开头的方法，update方法能满足，能匹配
```java
execution(* *..u*(..))
```

- 匹配项目中任意包任意类下只要以e结尾的方法，update和save方法能满足，能匹配
```java
execution(* *..*e(..))
```

- 返回值为void，com包下的任意包任意类任意方法，能匹配，`*`代表的是方法
```java
execution(void com..*())
```

- `将项目中所有业务层方法的以find开头的方法匹配(常用)`
```java
execution(* com.blog.*.*Service.find*(..))
```

- `将项目中所有业务层方法的以save开头的方法匹配(常用)`
```java
execution(* com.blog.*.*Service.save*(..))
```

#### 4.1.3 书写技巧

对于切入点表达式的编写其实是很灵活的，那么在编写的时候，有没有什么好的技巧让我们用用:

- 所有代码按照标准规范开发，否则以下技巧全部失效
- 描述切入点通常`描述接口`，而不描述实现类,如果描述到实现类，就出现紧耦合了
- `访问控制修饰符`针对接口开发均采用public描述（`可省略访问控制修饰符描述`）
- `返回值类型`对于增删改类使用精准类型加速匹配，对于查询类使用`*`通配快速描述
- `包名`书写尽量不使用`..`匹配，效率过低，常用`*`做单个包描述匹配，或精准匹配
- `接口名/类名`书写名称与模块相关的采用`*`匹配，例如UserService书写成`*Service`，绑定业务层接口名
- 方法名书写以`动词`进行`精准匹配`，名词采用`*`匹配，例如`getById`书写成`getBy*`，`selectAll`书写成`select*`
- 参数规则较为复杂，根据业务方法灵活调整
- 通常`不使用异常`作为`匹配`规则

### 4.2 AOP通知类型

前面的案例中，有涉及到如下内容
```java
@Before("pt()")
```

它所代表的含义是将`通知`添加到`切入点`方法执行的`前面`。  

除了这个注解外，还有没有其他的注解，换个问题就是除了可以在前面加，能不能在其他的地方加?
#### 4.2.1 类型介绍

我们先来回顾下AOP通知:

- AOP通知描述了`抽取的共性功能`，根据共性功能抽取的位置不同，最终运行代码时要将其加入到合理的位置

那么具体可以将通知添加到哪里呢？一共提供了5种通知类型

- 前置通知
- 后置通知
- `环绕通知(重点)`
- 返回后通知(了解)
- 抛出异常后通知(了解)

为了更好的理解这几种通知类型，我们来看一张图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e407c4050918f42f3433a13b379458c9.png)


1. **前置通知**：追加功能到方法执行前,类似于在代码1或者代码2添加内容
2. **后置通知**：追加功能到方法执行后,**不管方法执行的过程中有没有抛出异常都会执行**，类似于在代码5添加内容
3. **返回后通知**：追加功能到方法执行后，只有方法正常执行结束后才进行,类似于在代码3添加内容，如果方法执行抛出异常，返回后通知的内容将不会被添加
4. **抛出异常后通知**：追加功能到方法抛出异常后，只有方法执行出异常才进行,类似于在代码4添加内容，只有方法抛出异常后才会被添加
5. **环绕通知**：环绕通知功能比较强大，它可以追加功能到方法执行的前后，这也是比较常用的方式，它可以实现其他四种通知类型的功能，具体是如何实现的，需要我们往下学习。
#### 4.2.2 环境准备

- 创建一个Maven项目
- pom.xml添加Spring依赖
- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    public void update();  
  
    public int select();  
}

@Repository  
public class BookDaoImpl implements BookDao {  
      
    public void update() {  
        System.out.println("book dao update ...");  
    }  
  
    public int select() {  
        System.out.println("book dao select ...");  
        return 100;  
    }  
}
```

- 创建Spring的配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- 创建通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    public void before() {  
        System.out.println("before advice ...");  
    }  
  
    public void after(){  
        System.out.println("after advice ...");  
    }  
  
    public void around(){  
        System.out.println("around before advice ...");  
        System.out.println("around after advice ...");  
    }  
  
    public void afterReturning(){  
        System.out.println("afterReturning advice ...");  
    }  
  
    public void afterThrowing(){  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        bookDao.update();  
    }  
}
```
#### 4.2.3 通知类型的使用

- `前置通知`
	修改MyAdvice，在before方法上添加`@Before`注解
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void before() {  
        System.out.println("before advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30e03ef2bead767d700f0ef7cabee6c6.png)


- `后置通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Before("pt()")  
    public void before() {  
        System.out.println("before advice ...");  
    }  
  
    @After("pt()")  
    public void after(){  
        System.out.println("after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5a1bdab97b8be16fbd7ff338d80222a.png)


- `环绕通知`：基本使用
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(){  
        System.out.println("around before advice ...");  
        System.out.println("around after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fce365cec03a337b051fedf5dc31b4e2.png)


运行结果中，通知的内容打印出来，但是原始方法的内容却没有被执行。

因为环绕通知需要在原始方法的前后进行增强，所以环绕通知就必须要能对原始操作进行调用，具体如何实现?

- 在方法参数中添加`ProceedingJoinPoint`，同时在需要的位置使用`proceed()`调用原始操作
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void com.blog.dao.BookDao.update())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        //表示对原始操作的调用  
        pjp.proceed();  
        System.out.println("around after advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fe6622bc62f92faa57a048a46459a18e.png)


- 注意事项

- `当原始方法中有返回值时`
- 修改MyAdvice,对BookDao中的select方法添加环绕通知
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public void around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        pjp.proceed();  
        System.out.println("around after advice ...");  
    }  
}
```

- 修改App类，调用select方法
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        int select = bookDao.select();  
        System.out.println(select);  
    }  
}
```

- 运行程序，报错![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9378dd38244760bedb101a2315e3f177.png)
	- void就是返回Null
	- 原始方法的返回值是BookDao下的select方法

- 所以如果我们使用环绕通知的话，要根据原始方法的返回值来设置环绕通知的返回值，具体解决方案为:
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("around before advice ...");  
        Object res = pjp.proceed();  
        System.out.println("around after advice ...");  
        return res;  
    }  
}
```

- 运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/573fb03039dbeb3795a53628d6cb68a4.png)


- `说明`:
	- 为什么返回的是Object而不是int的主要原因是Object类型更通用。
	- 在环绕通知中是可以对原始方法返回值就行修改的。例如上面的例子，可以改为`return res+666;`，最终的输出结果也会变为766
	- 注意类型转换！

- `返回后通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @AfterReturning("pt()")  
    public void afterReturning() {  
        System.out.println("afterReturning advice ...");  
    }  
}
```

- 运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5a4841c5f6446005f6660d8acc56fb35.png)


- `注意`：  
	- 返回后通知是需要在原始方法`select`正常执行后才会被执行，如果`select()`方法执行的过程中出现了异常，那么返回后通知是不会被执行。后置通知是不管原始方法有没有抛出异常都会被执行。  

- 现在我们在select()方法中加一个异常
```java
public int select() {  
    System.out.println("book dao select ...");  
    int a = 1 / 0;  
    return 100;  
}
```

- 运行程序，输出如下，没有输出`afterReturning advice ...`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e37b41d4461e040caa4da5e3587d12a.png)


- 我们再换成后置输出，运行程序，结果如下，输出了`after advice ...`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/acb6a0660010ffe8cc71bbf81cfdb5ce.png)


- `异常后通知`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(int com.blog.dao.BookDao.select())")  
    private void pt() {  
    }  
  
    @AfterThrowing("pt()")  
    public void afterThrowing() {  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 运行程序，输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e91ef3d72d530d3f19dba986d159536a.png)


学习完这5种通知类型，我们来思考下`环绕通知是如何实现其他通知类型的功能的?`

因为环绕通知是可以控制原始方法执行的，所以我们把增强的代码写在调用原始方法的不同位置就可以实现不同的通知类型的功能，如![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0bb6e7e9543c9b7c6694b2b8f4d3560.png)
#### 4.2.4 通知类型总结

知识点1：`@After`

|名称|@After|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法后运行|

知识点2：`@AfterReturning`

|名称|@AfterReturning|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法正常执行完毕后执行|

知识点3：`@AfterThrowing`

|名称|@AfterThrowing|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法运行抛出异常后执行|

知识点4：`@Around`

|名称|@Around|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前后运行|

知识点5：`@Before`

|名称|@Before|
|---|---|
|类型|方法注解|
|位置|通知方法定义上方|
|作用|设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行|

环绕通知注意事项

1. **环绕通知必须依赖形参ProceedingJoinPoint才能实现对原始方法的调用**，进而实现原始方法调用前后同时添加通知
2. 通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行
3. 对原始方法的调用可以不接收返回值，通知方法设置成void即可，**如果接收返回值，最好设定为Object类型**
4. 原始方法的返回值如果是void类型，通知方法的返回值类型可以设置成void,也可以设置成Object
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须要处理Throwable异常

介绍完这么多种通知类型，具体该选哪一种呢?

我们可以通过一些案例加深下对通知类型的学习。

### 4.3 案例1：业务层接口执行效率

#### 4.3.1 需求分析

这个需求也比较简单，前面我们在介绍AOP的时候已经演示过:

- 需求:任意业务层接口执行均可显示其执行效率（执行时长）  
    这个案例的目的是查看每个业务层执行的时间，这样就可以监控出哪个业务比较耗时，将其查找出来方便优化。

具体实现的思路:

1. 开始执行方法之前记录一个时间
2. 执行方法
3. 执行完方法之后记录一个时间
4. 用后一个时间减去前一个时间的差值，就是我们需要的结果。

所以要在方法执行的前后添加业务，经过分析我们将采用`环绕通知`。  
**说明:** 原始方法如果只执行一次，时间太快，两个时间差可能为0，所以我们要执行万次来计算时间差。
#### 4.3.2 环境准备

- 创建一个Maven项目
    
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>  
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.46</version>  
</dependency>  
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
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>  
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.12</version>  
    <scope>test</scope>  
</dependency>
```

- 创建数据库与表
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




- 添加AccountService、AccountServiceImpl、AccountDao与Account类
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

public interface AccountDao {  
    @Insert("insert into tbl_account(`name`,money) values(#{name},#{money}) ")  
    void save(Account account);  
  
    @Update("update tbl_account set `name`=#{name},money=#{money}")  
    void update(Account account);  
  
    @Select("select * from tbl_account")  
    List<Account> findAll();  
  
    @Select("select * from tbl_account where id=#{id}")  
    Account findById(Integer id);  
}

public interface AccountService {  
    void save(Account account);  
    void update(Account account);  
    void delete(Integer id);  
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
  
    public void update(Account account) {  
        accountDao.update(account);  
    }  
  
    public void delete(Integer id) {  
        accountDao.delete(id);  
    }  
  
    public List<Account> findAll() {  
        return accountDao.findAll();  
    }  
  
    public Account findById(Integer id) {  
        return accountDao.findById(id);  
    }  
}
```

- resources下提供一个jdbc.properties
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db?useSSL=false  
jdbc.username=root  
jdbc.password=poassword.
```

- 创建相关配置类
```java
//SpringConfig
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}

//JDBCConfig
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

//MyBatiesConfig
public class MyBatisConfig {  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {  
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();  
        sqlSessionFactoryBean.setTypeAliasesPackage("com.blog.domain");  
        sqlSessionFactoryBean.setDataSource(dataSource);  
        return sqlSessionFactoryBean;  
    }  
  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer() {  
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();  
        mapperScannerConfigurer.setBasePackage("com.blog.dao");  
        return mapperScannerConfigurer;  
    }  
}

//AccountServiceImpl
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    private AccountDao accountDao;  
    public void save(Account account) {  
        accountDao.save(account);  
    }  
  
    public void update(Account account) {  
        accountDao.update(account);  
    }  
  
    public void delete(Integer id) {  
        accountDao.delete(id);  
    }  
  
    public List<Account> findAll() {  
        return accountDao.findAll();  
    }  
  
    public Account findById(Integer id) {  
        return accountDao.findById(id);  
    }  
}
```

- 编写Spring整合Junit的测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class AccountServiceTestCase {  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void testFindById(){  
        Account account = accountService.findById(2);  
    }  
  
    @Test  
    public void testFindAll(){  
        List<Account> accountList = accountService.findAll();  
    }  
}
```
#### 4.3.3 功能开发

- `步骤一：`开启SpringAOP的注解功能  
	在Spring的主配置文件SpringConfig类中添加注解
```java
@EnableAspectJAutoProxy
```

- `步骤二：`创建AOP的通知类
	- 该类要被Spring管理，需要添加`@Component`
	- 要标识该类是一个AOP的切面类，需要添加`@Aspect`
	- 配置切入点表达式，需要添加一个方法，并添加`@Pointcut`
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    public void runSpeed() {  
  
    }  
}
```

- `步骤三：`添加环绕通知  
	在runSpeed()方法上添加@Around
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    @Around("servicePt()")  
    public Object runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
  
        Object res = proceedingJoinPoint.proceed();  
        return res;  
    }  
}
```

- `步骤四：`完成核心业务，记录万次执行的时间
```java
@Component  
@Aspect  
public class ProjectAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void servicePt() {  
    }  
  
    @Around("servicePt()")  
    public void runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
        long start = System.currentTimeMillis();  
        for (int i = 0; i < 10000; i++) {  
            proceedingJoinPoint.proceed();  
        }  
        long end = System.currentTimeMillis();  
        System.out.println("业务层接口万次执行时间: " + (end - start) + "ms");  
    }  
}
```

- `步骤五：`运行单元测试类  
	运行结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4d04e14a0a4837df58a7033fc13f2e1b.png)


- `步骤六：` 程序优化  
	目前还存在一个问题，当我们一次执行多个方法时，控制台输出的都是`业务层接口万次执行时间: XXXms`  
	
	我们无法得知具体哪个方法的耗时，那么该如何优化呢？
	  
	ProceedingJoinPoint中有一个getSignature()方法来获取签名，然后调用`getDeclaringTypeName`可以获取类名，`getName()`可以获取方法名
```java
@Around("servicePt()")  
public void runSpeed(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
    Signature signature = proceedingJoinPoint.getSignature();  
    String typeName = signature.getDeclaringTypeName();  
    String methodName = signature.getName();  
    long start = System.currentTimeMillis();  
    for (int i = 0; i < 10000; i++) {  
        proceedingJoinPoint.proceed();  
    }  
    long end = System.currentTimeMillis();  
    System.out.println("万次执行 " + typeName + "." + methodName + " 耗时" + (end - start) + "ms");  
}
```

再次运行程序，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d49f3c36555745d494f0f9681308f920.png)

- `说明`
	- 当前测试的接口执行效率仅仅是一个理论值，并不是一次完整的执行过程。  
	- 这块只是通过该案例把AOP的使用进行了学习，具体的实际值是有很多因素共同决定的。

### 4.4 案例2：AOP通知获取数据

目前我们写AOP仅仅是在原始方法前后追加一些操作，接下来我们要说说`AOP中数据相关的内容`，我们将从`获取参数`、`获取返回值`和`获取异常`三个方面来研究切入点的相关信息。  
前面我们介绍通知类型的时候总共讲了五种，那么对于这五种类型都会有参数，返回值和异常吗?  
我们先来逐一分析下:

- 获取切入点方法的参数，`所有的通知类型都可以获取参数`
    - `JoinPoint`：适用于前置、后置、返回后、抛出异常后通知
    - `ProceedingJoinPoint`：适用于环绕通知
- 获取切入点方法返回值，`前置和抛出异常后通知是没有返回值，后置通知可有可无`，所以不做研究
    - 返回后通知
    - 环绕通知
- 获取切入点方法运行异常信息，`异常信息前置和返回后通知是不会有，后置通知可有可无`，所以不做研究
    - 抛出异常后通知
    - 环绕通知
#### 4.4.1 环境准备

- 创建一个maven项目
- 添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```

- 添加BookDao和BookDaoImpl类
```java
public interface BookDao {  
    String findName(int id);  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id) {  
        System.out.println("id:" + id);  
        return "TestName";  
    }  
}
```

- 创建Spring配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- 编写通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Before("pt()")  
    public void before(){  
        System.out.println("before advice ...");  
    }  
  
    @After("pt()")  
    public void after(){  
        System.out.println("after advice ...");  
    }  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object res = pjp.proceed();  
        return res;  
    }  
  
    @AfterReturning("pt()")  
    public void afterReturning(){  
        System.out.println("afterReturning advice ...");  
    }  
  
    @AfterThrowing("pt()")  
    public void afterThrowing(){  
        System.out.println("afterThrowing advice ...");  
    }  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        String name = bookDao.findName(9527);  
        System.out.println(name);  
    }  
}
```
#### 4.4.2 获取参数

- `非环绕通知获取方式`  
	在方法上添加JoinPoint，通过JoinPoint来获取方法参数
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Before("pt()")  
    public void before(JoinPoint joinPoint){  
        Object[] args = joinPoint.getArgs();  
        System.out.println(Arrays.toString(args));  
        System.out.println("before advice ...");  
    }  
}
```

运行App类，可以获取如下内容，说明参数9527已经被获取![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fafab1c8af0267a622effd0039c66655.png)


- **思考:方法的参数只有一个，为什么获取的是一个数组?**
    - 因为参数的个数是不固定的，所以使用数组更通配些。
    - 如果将参数改成两个会是什么效果呢?

- 修改BookDao和BookDaoImpl类
```java
public interface BookDao {  
    String findName(int id, String name) {  
}

@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id, String name) {  
        System.out.println("id:" + id);  
        return "TestName";  
    }  
}
```

- 修改App类，调用方法传入多个参数
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        BookDao bookDao = context.getBean(BookDao.class);  
        String name = bookDao.findName(9527,"Tony");  
        System.out.println(name);  
    }  
}
```

- 输出结果如下，两个参数都已经被获取到![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7c9a0b66aa0b92b770e07c929169e6a.png)



- `环绕通知获取方式`  
	环绕通知使用的是ProceedingJoinPoint，因为ProceedingJoinPoint是JoinPoint类的子类，所以对于ProceedingJoinPoint类中应该也会有对应的`getArgs()`方法，我们去验证下
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        Object res = pjp.proceed();  
        return res;  
    }  
}
```

- 运行App后查看运行结果，说明ProceedingJoinPoint也是可以通过getArgs()获取参数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61a4949eff3704c1f924f3f5d85508de.png)
`注意:`

- `pjp.proceed()方法是有两个构造方法，分别是`:
    - `proceed()`
    - `proceed(Object[] object)`
- 调用无参数的proceed，当原始方法有参数，会在调用的过程中自动传入参数
- 所以调用这两个方法的任意一个都可以完成功能
- 但是当需要修改原始方法的参数时，就只能采用带有参数的方法,如下
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = pjp.proceed(args);  
        return res;  
    }  
}
```

- 运行程序，输出结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/948d72ec30bd414cef9da577d5dde65c.png)


- 有了这个特性后，我们就`可以在环绕通知中对原始方法的参数进行拦截过滤`，避免由于参数的问题导致程序无法正确运行，还可以根据参数来给予不同的权限，提高代码的健壮性。
#### 4.4.3 获取返回值

对于返回值，只有返回后`AfterReturing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

- 环绕通知获取返回值
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = pjp.proceed(args); 
        return res;  
    }  
}
```
上述代码中，`res`就是方法的返回值，我们是可以直接获取，不但可以获取，如果需要还可以进行修改。

- 返回后通知获取返回值
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @AfterReturning(value = "pt()", returning = "res")  
    public void afterReturning(Object res){  
        System.out.println("afterReturning advice ..." + res);  
    }  
}
```

运行程序，输出如下，成功获取了返回值![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/769bdd520eb3fefe700ce38286f3fe9d.png)



- `几点注意`:

1. 参数名的问题  
    赋给returning的值，必须与Object类型参数名一致，上面的代码中均为`res`
2. afterReturning方法参数类型的问题  
    参数类型可以写成String，但是为了能匹配更多的参数类型，建议写成Object类型
3. afterReturning方法参数的顺序问题  
    `如果存在JoinPoint参数，则必须将其放在第一位，否则运行将报错`
```java
public void afterReturning(JoinPoint jp,Object res)
```
#### 4.4.4 获取异常(了解)

对于获取抛出的异常，只有抛出异常后`AfterThrowing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

- `环绕通知获取异常`  
    这块比较简单，以前我们是抛出异常，现在只需要将异常捕获，就可以获取到原始方法的异常信息了
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) {  
        Object[] args = pjp.getArgs();  
        System.out.println(Arrays.toString(args));  
        args[0] = 9421;  
        Object res = null;  
        try {  
            res = pjp.proceed(args);  
        } catch (Throwable e) {  
            throw new RuntimeException(e);  
        }  
        return res;  
    }  
}
```
在catch方法中就可以获取到异常，至于获取到异常以后该如何处理，这个就和你的业务需求有关了。
    
- `抛出异常后通知获取异常`
```java
@Component  
@Aspect  
public class MyAdvice {  
  
    @Pointcut("execution(* com.blog.dao.BookDao.findName(..))")  
    public void pt() {  
    }  
  
    @AfterThrowing(value = "pt()", throwing = "throwable")  
    public void afterThrowing(Throwable throwable) {  
        System.out.println("afterThrowing advice ..." + throwable);  
    }  
}
```

- 那现在我们只需要让原始方法抛一个异常来看看效果
```java
@Repository  
public class BookDaoImpl implements BookDao {  
    public String findName(int id, String name) {  
        System.out.println("id:" + id);  
        int a = 1 / 0;  
        return "TestName";  
    }  
}
```

- 运行程序，输出如下，成功输出了异常![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ef8c843001ac39fbb3fa0b26ffeea4b.png)


- 至此，AOP通知如何获取数据就已经讲解完了，数据中包含`参数`、`返回值`、`异常(了解)`。

### 4.5 案例3：百度网盘密码数据兼容处理

#### 4.5.1 需求分析

需求：对百度网盘分享链接输入密码时尾部多输入的空格做兼容处理。  
问题描述：
- 当我们从别人发给我们的内容中复制提取码的时候，有时候会多复制到一些空格，直接粘贴到百度的提取码输入框
- 但是百度那边记录的提取码是没有空格的
- 这个时候如果不做处理，直接对比的话，就会引发提取码不一致，导致无法访问百度盘上的内容
- 所以多输入一个空格可能会导致项目的功能无法正常使用。
- 此时我们就想能不能将输入的参数先帮用户去掉空格再操作呢?
    - 答案是可以的，我们只需要在业务方法执行之前对所有的输入参数进行格式处理——trim()
- 那要对所有的参数都需要去除空格么?
    - 也没有必要，一般只需要针对字符串处理即可。

- 以后涉及到需要去除前后空格的业务可能会有很多，这个去空格的代码是每个业务都写么?
    - 可以考虑使用AOP来统一处理。
- AOP有五种通知类型，该使用哪种呢?
    - 我们的需求是将原始方法的参数处理后在参与原始方法的调用，能做这件事的就只有环绕通知。
#### 4.5.2 环境准备

- 创建一个Maven项目
    
- pom.xml添加Spring依赖
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```

- 添加ResourcesDao和ResourcesDaoImpl，ResourcesService，ResourcesServiceImpl类
```java
public interface ResourceDao {  
    boolean readResource(String url,String password);  
}

public class ResourceDaoImpl implements ResourceDao {  
    public boolean readResource(String url, String password) {  
        //模拟校验  
        return password.equals("root");  
    }  
}

public interface ResourceService {  
    public boolean openURL(String url,String password);  
}

@Service  
public class ResourceServiceImpl implements ResourceService {  
    @Autowired  
    private ResourceDao resourceDao;  
    public boolean openURL(String url, String password) {  
        return resourceDao.readResource(url,password);  
    }  
}
```

- 创建Spring配置类
```java
@Configuration  
@ComponentScan("com.blog")  
public class SpringConfig {  
}
```

- 编写App运行类
```java
public class App {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        ResourceService service = context.getBean(ResourceService.class);  
        boolean flag = service.openURL("https://pan.baidu.com/xx", "root");  
        System.out.println(flag);  
    }  
}
```

现在项目的效果是，当输入密码为”root”控制台打印为true,如果密码改为”root “控制台打印的是false  

需求是使用AOP将参数进行统一处理，不管输入的密码`root`前后包含多少个空格，最终控制台打印的都是true

#### 4.5.3 具体实现

- `步骤一：`开启SpringAOP的注解功能
```java
@Configuration  
@ComponentScan("com.blog")  
@EnableAspectJAutoProxy  
public class SpringConfig {  
}
```

- `步骤二：`编写通知类
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(* com.blog.service.*Service.*(..))")  
    public void pt(){}  
  
    @Around("pt()")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        Object[] args = pjp.getArgs();  
        for (int i = 0; i < args.length; i++) {  
            if ((String.class).equals(args[i].getClass())){  
                args[i] = args[i].toString().trim();  
            }  
        }  
        Object res = pjp.proceed(args);  
        return res;  
    }  
}
```

- `步骤三：`运行程序  
    不管密码`root`前后是否加空格，最终控制台打印的都是true

## 5 AOP总结

### 5.1 AOP核心概念

- 概念：AOP(Aspect Oriented Programming)面向切面编程，一种编程范式
- 作用：在不惊动原始设计的基础上为方法进行功能`增强`
- 核心概念
    - 代理（Proxy）：SpringAOP的核心本质是采用`代理模式`实现的
    - 连接点（JoinPoint）：在SpringAOP中，理解为任意方法的执行
    - 切入点（Pointcut）：匹配连接点的式子，也是具有共性功能的方法描述
    - 通知（Advice）：若干个方法的共性功能，在切入点处执行，最终体现为一个方法
    - 切面（Aspect）：描述通知与切入点的对应关系
    - 目标对象（Target）：被代理的原始对象成为目标对象

### 5.2 切入点表达式

- 切入点表达式标准格式：动作关键字(访问修饰符 返回值 包名.类/接口名.方法名（参数）异常名)
```java
execution(* com.itheima.service.*Service.*(..))
```

- 切入点表达式描述通配符：
    - 作用：用于快速描述，范围描述
    - `*`：匹配任意符号（常用）
    - `..` ：匹配多个连续的任意符号（常用）
    - `+`：匹配子类类型

- 切入点表达式书写技巧
    1. 按`标准规范`开发
    2. 查询操作的返回值建议使用`*`匹配
    3. 减少使用`..`的形式描述包，效率低
    4. `对接口进行描述`，使用`*`表示模块名，例如UserService的匹配描述为`*Service`
    5. 方法名书写保留动词，例如get，使用`*`表示名词，例如getById匹配描述为getBy`*`
    6. 参数根据实际情况灵活调整
### 5.3 五种通知类型

- 前置通知
- 后置通知
- 环绕通知（重点）
    - 环绕通知依赖形参ProceedingJoinPoint才能实现对原始方法的调用
    - 环绕通知`可以隔离原始方法的调用执行`
    - 环绕通知返回值设置为Object类型
    - 环绕通知中可以对原始方法调用过程中出现的异常进行处理
- 返回后通知
- 抛出异常后通知
### 5.4 通知中获取参数

- 获取切入点方法的参数，所有的通知类型都可以获取参数
    - JoinPoint：适用于前置、后置、返回后、抛出异常后通知
    - ProceedingJoinPoint：适用于环绕通知
- 获取切入点方法返回值，前置和抛出异常后通知是没有返回值，后置通知可有可无，所以不做研究
    - 返回后通知
    - 环绕通知
- 获取切入点方法运行异常信息，前置和返回后通知是不会有，后置通知可有可无，所以不做研究
    - 抛出异常后通知
    - 环绕通知

## 6 AOP事务管理

### 6.1 Spring事务简介

#### 6.1.1 相关概念

相关概念

- 事务作用：在数据层`保障一系列的数据库操作同成功同失败`
- Spring事务作用：`在数据层或业务层`保障一系列的数据库操作同成功同失败

数据层有事务我们可以理解，为什么业务层也需要处理事务呢？举个简单的例子

- 转账业务会有两次数据层的调用，一次是加钱一次是减钱
- 把事务放在数据层，加钱和减钱就有两个事务
- 没办法保证加钱和减钱同时成功或者同时失败
- 这个时候就需要将事务放在业务层进行处理。

Spring为了管理事务，提供了一个`平台事务管理器` `PlatformTransactionManager`
```java
public interface PlatformTransactionManager extends TransactionManager {  
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;  
  
    void commit(TransactionStatus var1) throws TransactionException;  
  
    void rollback(TransactionStatus var1) throws TransactionException;  
}
```

commit是用来提交事务，rollback是用来回滚事务。

PlatformTransactionManager只是一个接口，Spring还为其提供了一个具体的实现:
```java
public class DataSourceTransactionManager extends AbstractPlatformTransactionManager implements ResourceTransactionManager, InitializingBean {  
    @Nullable  
    private DataSource dataSource;  
    private boolean enforceReadOnly;  
···  
···  
  
}
```

从名称上可以看出，我们`只需要给它一个DataSource对象`，它就可以帮你去在业务层管理事务。其`内部采用的是JDBC的事务`。所以说如果你持久层采用的是JDBC相关的技术，就可以采用这个事务管理器来管理你的事务。而Mybatis内部采用的就是JDBC的事务，所以后期我们Spring整合Mybatis就采用的这个`DataSourceTransactionManager`事务管理器。
#### 6.1.2 转账案例---需求分析

接下来通过一个案例来学习下Spring是如何来管理事务的。

先来分析下需求:

- 需求: 实现任意两个账户间转账操作
- 需求微缩: A账户减钱，B账户加钱

为了实现上述的业务需求，我们可以按照下面步骤来实现下:

1. 数据层提供基础操作，指定账户减钱（outMoney），指定账户加钱（inMoney）
2. 业务层提供转账操作（transfer），调用减钱与加钱的操作
3. 提供2个账号和操作金额执行转账操作
4. 基于Spring整合MyBatis环境搭建上述操作
#### 6.1.3 转账案例---环境搭建

- `步骤一：`准备数据表  
	Tom和Jerry初始金额都是1000
```sql
CREATE DATABASE spring_db CHARACTER SET utf8;  
USE spring_db;  
CREATE TABLE tbl_account(  
    id INT PRIMARY KEY AUTO_INCREMENT,  
    NAME VARCHAR(35),  
    money DOUBLE  
);  
INSERT INTO tbl_account(`name`,money) VALUES('Tom',1000),('Jerry',1000);
```

- `步骤二：`创建项目导入jar包
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
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

- `步骤三：`根据表创建模型类
```java
public class Account {  
    private Integer id;  
    private String name;  
    private Double money;  
  
    public Account() {  
    }  
  
    public Account(Integer id, String name, Double money) {  
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
  
    public Double getMoney() {  
        return money;  
    }  
  
    public void setMoney(Double money) {  
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

- `步骤四：`创建Dao接口
```java
public interface AccountDao {  
  
    @Update("update tbl_account set money = money + #{money} where name = #{name}")  
    void inMoney(@Param("name") String name,@Param("money") Double money);  
  
    @Update("update tbl_account set money = money - #{money} where name = #{name}")  
    void outMoney(@Param("name") String name, @Param("money") Double money);  
}
```

- `步骤五：`创建Service接口和实现类
```java
public interface AccountService {  
    /**  
     * 转账操作  
     * @param out 转出方  
     * @param in 转入方  
     * @param money 金额  
     */  
    public void transfer(String out,String in,Double money);  
}

@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        accountDao.inMoney(in, money);  
    }  
}
```

- `步骤六：`添加jdbc.properties文件
```properties
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/spring_db?useSSL=false  
jdbc.username=root  
jdbc.password=poassword.
```

- `步骤七：`创建JdbcConfig配置类
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

- `步骤八：`创建MybatisConfig配置类
```java
public class MyBatisConfig {  
  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){  
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();  
        factory.setTypeAliasesPackage("com.blog.domain");  
        factory.setDataSource(dataSource);  
        return factory;  
    }  
  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer(){  
        MapperScannerConfigurer msc = new MapperScannerConfigurer();  
        msc.setBasePackage("com.blog.dao");  
        return msc;  
    }  
}
```

- `步骤九:`创建SpringConfig配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤十：`编写测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class AccountServiceTest {  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void testTransfer() {  
        accountService.transfer("Tom", "Jerry", 100D);  
    }  
}
```
#### 6.1.4 事务管理

上述环境，运行单元测试类，会执行转账操作，`Tom`的账户会减少100，`Jerry`的账户会加100。

这是正常情况下的运行结果，但是如果在转账的过程中出现了异常，如
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        int a = 1 / 0;  
        accountDao.inMoney(in, money);  
    }  
}
```

这个时候就模拟了转账过程中出现异常的情况，此时进行转账，`Tom`的账户会减少100，而`Jerry`的账户却不会增加100  
那我们来分析一下刚才的结果

- 程序正常执行时，账户金额A减B加，没有问题
- 程序出现异常后，转账失败，但是异常之前操作成功，异常之后操作失败，整体业务失败

当程序出问题后，我们需要让事务进行回滚，而且这个事务应该是加在业务层上，而Spring的事务管理就是用来解决这类问题的。

Spring事务管理具体的实现步骤如下

- `步骤一：`在需要被事务管理的方法上添加`@Transactional`注解
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional  
    public void transfer(String out, String in, Double money) {  
        accountDao.outMoney(out, money);  
        int a = 1 / 0;  
        accountDao.inMoney(in, money);  
    }  
}
```

注意:`@Transactional`可以写在接口类上、接口方法上、实现类上和实现类方法上
- 写在接口类上，该接口的所有实现类的所有方法都会有事务
- 写在接口方法上，该接口的所有实现类的该方法都会有事务
- 写在实现类上，该类中的所有方法都会有事务
- 写在实现类方法上，该方法上有事务
- `建议写在接口的方法上，降低耦合`

- `步骤二：`在JdbcConfig类中配置事务管理器
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
  
    //配置事务管理器，mybatis使用的是jdbc事务  
    @Bean  
    public PlatformTransactionManager platformTransactionManager(DataSource dataSource){  
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();  
        transactionManager.setDataSource(dataSource);  
        return transactionManager;  
    }  
}
```

注意：事务管理器要根据使用技术进行选择，Mybatis框架使用的是JDBC事务，可以直接使用`DataSourceTransactionManager`

- `步骤三：`开启事务注解`@EnableTransactionManagement`
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@EnableTransactionManagement  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤四：`运行测试类  
    运行程序之后，我们去数据库查看Tom和Jerry的金额，发现没有变化  
    那么说明在转换的业务出现错误后，事务就可以控制回滚，保证数据的正确性。

知识点1：`@EnableTransactionManagement`

|名称|@EnableTransactionManagement|
|---|---|
|类型|配置类注解|
|位置|配置类定义上方|
|作用|设置当前Spring环境中开启注解式事务支持|

知识点2：`@Transactional`

|名称|@Transactional|
|---|---|
|类型|接口注解 类注解 方法注解|
|位置|业务层接口上方 业务层实现类上方 业务方法上方|
|作用|为当前业务层方法添加事务（如果设置在类或接口上方则类或接口中所有方法均添加事务）|
### 6.2 Spring事务角色

这部分我们重点要理解两个概念，分别是`事务管理员`和`事务协调员`。

当未开启Spring事务之前![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b07b03c1ed52b8e8fc1a37d938cbd45b.png)
- AccountDao的outMoney因为是修改操作，会开启一个事务T1
- AccountDao的inMoney因为是修改操作，会开启一个事务T2
- AccountService的transfer没有事务，
    - 运行过程中如果没有抛出异常，则T1和T2都正常提交，数据正确
    - 如果在两个方法中间抛出异常，T1因为执行成功提交事务，T2因为抛异常不会被执行
    - 就会导致数据出现错误

当开启Spring的事务管理后![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3645aa16de6686079f37219350d4f8c9.png)


- transfer上添加了@Transactional注解，在该方法上就会有一个事务T
- AccountDao的outMoney方法的事务T1加入到transfer的事务T中
- AccountDao的inMoney方法的事务T2加入到transfer的事务T中
- 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性。

通过上面例子的分析，我们就可以得到如下概念:

- 事务管理员：发起事务方，在Spring中`通常指代业务层开启事务的方法`
- 事务协调员：加入事务方，在Spring中`通常指代数据层方法`，也可以是业务层方法

注意：目前的事务管理是基于`DataSourceTransactionManager`和`SqlSessionFactoryBean`使用的是同一个数据源。
### 6.3 Spring事务属性

这部分我们主要学习三部分内容`事务配置`、`转账业务追加日志`、`事务传播行为`。
#### 6.3.1 事务配置

|属性|作用|示例|
|---|---|---|
|readOnly|设置是否为只读事务|readOnly = true 只读事务|
|timeout|设置事务超时时间|timeout = -1(永不超时)|
|rollbackFor|设置事务回滚异常(class)|rollbackFor{NullPointException.class}|
|rollbackForClassName|设置事务回滚异常（String)|同上格式为字符串|
|noRollbackFor|设置事务不回滚异常(class)|noRollbackFor{NullPointExceptior.class}|
|noRollbackForClassName|设置事务不回滚异常(String)|同上格式为字符串|
|isolation|设置事务隔离级别|isolation = Isolation. DEFAULT|
|propagation|设置事务传播行为|…|

上面这些属性都可以在`@Transactional`注解的参数上进行设置。

- readOnly：true只读事务，false读写事务，增删改要设为false,查询设为true。
    
- timeout:设置超时时间单位秒，在多长时间之内事务没有提交成功就自动回滚，-1表示不设置超时时间。
    
- rollbackFor:当出现指定异常进行事务回滚
    
- noRollbackFor:当出现指定异常不进行事务回滚
    
    - 思考:出现异常事务会自动回滚，这个是我们之前就已经知道的
        
    - noRollbackFor是设定对于指定的异常不回滚，这个好理解
        
    - rollbackFor是指定回滚异常，对于异常事务不应该都回滚么，为什么还要指定?
        
- 事实上Spring的事务只会对`Error异常`和`RuntimeException异常`及其子类进行事务回滚，其他的异常类型是不会回滚的，如下面的代码就不会回滚
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional  
    public void transfer(String out, String in, Double money) throws IOException {  
        accountDao.outMoney(out, money);  
        if (true) throw new IOException();  
        accountDao.inMoney(in, money);  
    }  
}
```
所以当我们运行程序之后，Tom会少100块钱，而Jerry不会多100块钱，这100块钱就凭空消失了

- 此时就可以使用rollbackFor属性来设置出现IOException异常回滚
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
  
    @Transactional(rollbackFor = {IOException.class})  
    public void transfer(String out, String in, Double money) throws IOException {  
        accountDao.outMoney(out, money);  
        if (true) throw new IOException();  
        accountDao.inMoney(in, money);  
    }  
}
```

- rollbackForClassName等同于rollbackFor,只不过属性为异常的类全名字符串
- noRollbackForClassName等同于noRollbackFor，只不过属性为异常的类全名字符串
- isolation设置事务的隔离级别
	- DEFAULT :默认隔离级别, 会采用数据库的隔离级别
	- READ_UNCOMMITTED : 读未提交
	- READ_COMMITTED : 读已提交
	- REPEATABLE_READ : 重复读取
	- SERIALIZABLE: 串行化

介绍完上述属性后，还有最后一个事务的传播行为，为了讲解该属性的设置，我们需要完成下面的案例。
#### 6.3.2 转账业务追加日志案例

##### 6.3.2.1 需求分析

- 在前面的转账案例的基础上添加新的需求，完成转账后记录日志。
    - 需求：实现任意两个账户间转账操作，并对每次转账操作在数据库进行留痕
    - 需求微缩：A账户减钱，B账户加钱，数据库记录日志

- 基于上述的业务需求，我们来分析下该如何实现：
    1. 基于转账操作案例添加日志模块，实现数据库中记录日志
    2. 业务层转账操作（transfer），调用减钱、加钱与记录日志功能

- 需要注意一点就是，我们这个案例的预期效果为:
    - `无论转账操作是否成功，均进行转账操作的日志留痕`
##### 6.3.2.2 环境准备

- `步骤一：`创建日志表
```SQL
create table tbl_log(  
   id int primary key auto_increment,  
   info varchar(255),  
   createDate datetime  
)
```

- `步骤二：`添加LogDao接口
```java
public interface LogDao {  
  
    @Insert("insert into tbl_log(info, createDate) VALUES (#{info},now())")  
    void log(String info);  
}
```

- `步骤三：`添加LogService接口和实现类
```java
public interface LogService {  
    void log(String out, String in, Double money);  
}

@Service  
public class LogServiceImpl implements LogService {  
    @Autowired  
    private LogDao logDao;  
  
    @Transactional  
    public void log(String out, String in, Double money) {  
        logDao.log(out + "向" + in + "转账" + money + "元");  
    }  
}
```

- `步骤四：`在转账的业务中添加记录日志
```java
@Service  
public class AccountServiceImpl implements AccountService {  
    @Autowired  
    protected AccountDao accountDao;  
    @Autowired  
    protected LogService logService;  
  
    @Transactional(rollbackFor = {IOException.class})  
    public void transfer(String out, String in, Double money) throws IOException {  
        try {  
            accountDao.outMoney(out, money);  
            accountDao.inMoney(in, money);  
        } finally {  
            logService.log(out, in, money);  
        }  
    }  
}
```

- `步骤五：`运行程序
    
    - 当程序正常运行，tbl_account表中转账成功，tbl_log表中日志记录成功
    - 当转账业务之间出现异常(int i =1 / 0),转账失败，tbl_account成功回滚，但是tbl_log表未添加数据，说明也回滚了
    - 这个结果和我们想要的不一样，什么原因?该如何解决?
        - 失败原因：日志的记录与转账操作隶属同一个事务，同成功同失败（同回滚）
        - 解决方案：继续往下看
        - 预期效果：无论转账操作是否成功，日志必须保留

#### 6.3.3 事务传播行为

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/794ef04d222965baaba26cb088745478.png)



对于上述案例的分析:

- log方法、inMoney方法和outMoney方法都属于增删改，分别有事务T1,T2,T3
- transfer因为加了`@Transactional`注解，也开启了事务T
- 前面我们讲过Spring事务会把T1,T2,T3都加入到事务T中
- 所以当转账失败后，`所有的事务都回滚`，导致日志没有记录下来
- 这和我们的需求不符，这个时候我们就想能不能让log方法单独是一个事务呢?

要想解决这个问题，就需要用到事务传播行为，所谓的事务传播行为指的是:

- 事务传播行为：事务协调员对事务管理员所携带事务的处理态度。
    - 具体如何解决，就需要用到之前我们没有说的`propagation属性`。

- 修改logService改变事务的传播行为
```java
@Service  
public class LogServiceImpl implements LogService {  
    @Autowired  
    private LogDao logDao;  
  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    public void log(String out, String in, Double money) {  
        logDao.log(out + "向" + in + "转账" + money + "元");  
    }  
}
```

- 运行后，就能实现我们想要的结果，不管转账是否成功，都会记录日志。

事务传播行为的可选值

|传播属性|事务管理员|事务协调员|
|---|---|---|
|REQUIRED(默认)|开启T|加入T|
|---|无|新建T2|
|REQUIRES_NEW|开启T|新建T2|
|---|无|新建T2|
|SUPPORTS|开启T|加入T|
|---|无|无|
|NOT_SUPPORTED|开启T|无|
|---|无|无|
|MANDTORY|开启T|加入T|
|---|无|ERROR|
|NEVER|开启T|ERROR|
|---|无|无|
|NESTED|设置savePoint,一旦事务回滚，事务将回滚到savePoint处，交由客户响应提交/回滚|   |
---
文章标题: "[[4_SpringMVC]]" 
文章作者: Dakkk
文章概要: |
  介绍SpringMVC框架基础知识，包括MVC架构、入门案例、请求响应处理、RESTful风格开发和实际案例应用。
tags:
- "SpringMVC"
- "Web开发"
- "MVC架构"
- "RESTful"
- "JSON数据"
- "请求映射"
- "参数传递"
- "响应处理"
相关文章:
- "[[5_SSM整合]]"
- "[[1_Java值传递详解]]"
- "[[2_登录页加点盐：通过 Animate.css 添加入场动画]]"
- "[[2_函数]]"
- "[[2_Servlet]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/2_SSM（黑马）/4_SpringMVC.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:12:14
---

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
<!--而且SpringMVC的版本不能超过5,否则会报错-->
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

1. **服务器启动，执行ServletContainerInitConfig类，初始化web容器**
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

2. **执行createServletApplicationContext方法，创建了WebApplicationContext对象**
	- 该方法加载SpringMVC的配置类SpringMvcConfig来初始化SpringMVC的容
	```java
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    context.register(SpringMvcConfig.class);  
    return context;  
}
	```

3. **加载SpringMvcConfig配置类**
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

- !!!运行程序之前，我们还需要把`SpringMvcConfig`配置类上的`@ComponentScan`注解注释掉，否则不会报错，将正常输出 , 出现问题的原因是
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

关于请求参数的传递与接收是**和请求方式有关系的**，目前比较常见的两种请求方式为：
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
- http://localhost/user/commonParam?name=Jerry
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

- `步骤四：`查看控制台，输出如下
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e02a09e240c6cbf16a38e5953e2d2b3.png)

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

因为异步调用是目前常用的主流方式，所以我们需要**更关注的就是如何返回JSON数据**，对于其他只需要认识了解即可。
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

- 访问`http://localhost:8080/toJsonPojo`，页面上成功出现JSON类型数据
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1a869f2276ad503510194ca5ed037b2.png)


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

- 访问`http://localhost:8080/toJsonList`，页面上成功出现JSON集合类型数据![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d0c702d682e4f7c9452b67479530c92.png)

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
@RequestMapping(value = "/users",method = RequestMethod.DELETE)  
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
<!--而且SpringMVC的版本不能超过5,否则会报错-->
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

1. **服务器启动，执行ServletContainerInitConfig类，初始化web容器**
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

2. **执行createServletApplicationContext方法，创建了WebApplicationContext对象**
	- 该方法加载SpringMVC的配置类SpringMvcConfig来初始化SpringMVC的容
	```java
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    context.register(SpringMvcConfig.class);  
    return context;  
}
	```

3. **加载SpringMvcConfig配置类**
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

- !!!运行程序之前，我们还需要把`SpringMvcConfig`配置类上的`@ComponentScan`注解注释掉，否则不会报错，将正常输出 , 出现问题的原因是
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

关于请求参数的传递与接收是**和请求方式有关系的**，目前比较常见的两种请求方式为：
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
- http://localhost/user/commonParam?name=Jerry
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

- `步骤四：`查看控制台，输出如下
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e02a09e240c6cbf16a38e5953e2d2b3.png)

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

因为异步调用是目前常用的主流方式，所以我们需要**更关注的就是如何返回JSON数据**，对于其他只需要认识了解即可。
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

- 访问`http://localhost:8080/toJsonPojo`，页面上成功出现JSON类型数据
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1a869f2276ad503510194ca5ed037b2.png)


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

- 访问`http://localhost:8080/toJsonList`，页面上成功出现JSON集合类型数据![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d0c702d682e4f7c9452b67479530c92.png)

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
@RequestMapping(value = "/users",method = RequestMethod.DELETE)  
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