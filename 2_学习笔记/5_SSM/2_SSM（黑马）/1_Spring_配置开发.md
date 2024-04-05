
## 1 课程介绍

对于一门新技术，我们需要从`为什么要学`、`学什么`以及`怎么学`这三个方向入手来学习。那对于Spring来说:

### 1.1 为什么要学?

- `从使用和占有率看`
    
    - Spring在*市场的占有率与使用率高*
    - Spring在*企业的技术选型命中率高*
    - 所以说,Spring技术是*JavaEE开发必备技能*，企业开发`技术选型命中率 > 90%  `
    - **说明**:对于未使用Spring的项目一般都是些比较老的项目，大多都处于维护阶段。


- `从专业角度看`
    
    - 随着时代发展，软件规模与功能都呈几何式增长，开发难度也在不断递增，该如何解决?
        - Spring可以`简化开发`，降低企业级开发的复杂性，使开发变得更简单快捷
    - 随着项目规模与功能的增长,遇到的问题就会增多，为了解决问题会引入更多的框架，这些框架如何协调工作?
        - Spring可以`框架整合`，高效整合其他技术，提高企业级应用开发与运行效率

- `综上所述`，Spring是一款非常优秀而且功能强大的框架，不仅要学，而且还要学好。

### 1.2 学什么?

从上面的介绍中，我们可以看到Spring框架主要的优势是在`简化开发`和`框架整合`上，至于如何实现就是我们要学习Spring框架的主要内容:

- `简化开发`: Spring框架中提供了`两个大的核心技术`，分别是:
    - `IOC`
    - `AOP`
        - `事务处理`
- Spring的简化操作都是基于这两块内容,所以这也是Spring学习中最为重要的两个知识点。
- 事务处理属于Spring中AOP的具体应用，可以简化项目中的事务管理，也是Spring技术中的一大亮点。


- `框架整合`: Spring在框架整合这块已经做到了极致，它可以整合市面上几乎所有主流框架，比如:
    
    - MyBatis
    - MyBatis-plus
    - Struts
    - Struts2
    - Hibernate
    - ……
- 这些框架中，我们目前只学习了MyBatis，所以在Spring框架的学习中，主要是学习如何整合MyBatis。
    
- 综上所述，对于Spring的学习，主要学习四块内容:
    1. IOC
    2. 整合Mybatis(IOC的具体应用)
    3. AOP
    4. 声明式事务(AOP的具体应用)

### 1.3 怎么学?

- `学习Spring框架设计思想`
    - 对于Spring来说，它能迅速占领全球市场，不只是说它的某个功能比较强大，更重要是在它的`思想`上。
- `学习基础操作，思考操作与思想间的联系`
    - 掌握了Spring的设计思想，然后就需要通过一些基础操作来思考操作与思想之间的关联关系
- `学习案例，熟练应用操作的同时，体会思想`
    - 会了基础操作后，就需要通过大量案例来熟练掌握框架的具体应用，加深对设计思想的理解。
- 介绍完`为什么要学`、`学什么`和`怎么学`Spring框架后，我们需要重点掌握的是:
    
    - Spring很优秀，需要认真重点的学习
    - Spring的学习主线是IOC、AOP、声明式事务和整合MyBais
- 接下来，咱们就开始进入Spring框架的学习。
## 2 Spring相关概念

### 2.1 初识Spring

#### 2.1.1 Spring家族

- 官网：[https://spring.io](https://spring.io/) 从官网我们可以大概了解到：
    
    - Spring能做什么：用以开发web、微服务以及分布式系统等,光这三块就已经占了JavaEE开发的九成多。
    - Spring并不是单一的一个技术，而是一个大家族，可以从官网的`Projects`中查看其包含的所有技术。
- Spring发展到今天已经形成了一种开发的生态圈,Spring提供了若干个项目,每个项目用于完成特定的功能。
    
    - `Spring已形成了完整的生态圈，也就是说我们可以完全使用Spring技术完成整个项目的构建、设计与开发`。

    - `Spring有若干个项目，可以根据需要自行选择，把这些个项目组合起来，起了一个名称叫Spring全家桶`，如下图所示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f5f2dad0859043d2e357729a625f98a0.png)
- **说明:**

图中的图标都代表什么含义，可以进入 [https://spring.io/projects](https://spring.io/projects) 网站进行对比查看。  
这些技术并不是所有的都需要学习，额外需要重点关注`Spring Framework`、`SpringBoot`和`SpringCloud`:

- `Spring Framework`：Spring框架，是Spring中最早最核心的技术，也是*所有其他技术的基础*。
- `SpringBoot`：Spring是来简化开发，而SpringBoot是来*帮助Spring在简化的基础上能更快速进行开发*
- `SpringCloud`：这个是用来做*分布式之微服务架构的相关开发*。

除了上面的这三个技术外，还有很多其他的技术，也比较流行，如SpringData，SpringSecurity等，这些都可以被应用在我们的项目中。我们这里所*学习的Spring其实指的是Spring Framework*。

#### 2.1.2 Spring发展史

接下来我们介绍下Spring Framework这个技术是如何来的呢?

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8d69cfdb46689e5a25af64a956d0de9a.png)



- IBM(IT公司-国际商业机器公司)在1997年提出了EJB思想,早期的JAVAEE开发大都基于该思想。
- Rod Johnson(Java和J2EE开发领域的专家)在2002年出版的`Expert One-on-One J2EE Design and Development`,书中有阐述在开发中使用EJB该如何做。
- Rod Johnson在2004年出版的`Expert One-on-One J2EE Development without EJB`,书中提出了比EJB思想更高效的实现方案，并且在同年将方案进行了具体的落地实现，这个实现就是Spring1.0。

- 随着时间推移，版本不断更新维护，目前最新的是Spring5
    - Spring1.0是`纯配置文件开发`
    - Spring2.0为了简化开发`引入了注解开发`，此时是配置文件加注解的开发方式
    - Spring3.0已经可以进行`纯注解开发`，使开发效率大幅提升，我们的课程会以注解开发为主
    - Spring4.0根据JDK的版本升级`对个别API进行了调整`
    - Spring5.0已经`全面支持JDK8`，所以学习期间最好用JDK8版本

- 本节介绍了Spring家族与Spring的发展史，需要我们重点掌握的是:
    - Spring其实是Spring家族中的Spring Framework
    - Spring Framework是Spring家族中其他框架的底层基础，学好Spring可以为其他Spring框架的学习打好基础

### 2.2 Spring系统架构

#### 2.2.1 系统架构图

- Spring Framework是Spring生态圈中`最基础的项目`，是`其他项目的根基`。

- Spring Framework的发展也经历了很多版本的变更，每个版本都有相应的调整

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5dc77ae52169d9e5b3d0c708158627b3.png)



- Spring Framework的5版本目前没有最新的架构图，而最新的是4版本，所以接下来主要研究的是4的架构图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/956bfedecd3d2fdc46bc42867bb86a9e.png)


1. `核心层`
    - Core Container：核心容器，这个模块是Spring最核心的模块，*其他的都需要依赖该模块*
2. `AOP层`
    - AOP(Aspect Oriented Programming)：面向切面编程，它依赖核心层容器，目的是在不改变原有代码的前提下对其进行功能增强
    - Aspects：`AOP是思想，Aspects是对AOP思想的具体实现`
3. `数据层`
    - Data Access：数据访问，Spring全家桶中有对数据访问的具体实现技术
    - Data Integration：数据集成，Spring支持整合其他的数据层解决方案，比如Mybatis
    - Transactions：事务，Spring中事务管理是Spring AOP的一个具体实现，也是后期学习的重点内容
4. `Web层`
    - 这一层的内容将在SpringMVC框架具体学习
5. `Test层`
    - Spring主要整合了Junit来完成单元测试和集成测试

#### 2.2.2 课程学习路线

- 介绍完Spring的体系结构后，从中我们可以得出对于Spring的学习主要包含四部分内容，分别是:
    1. Spring的IOC/DI
    2. Spring的AOP
    3. AOP的具体应用,事务管理
    4. IOC/DI的具体应用,整合Mybatis

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d02c2e17858e23ccfe17967576e7472.png)


### 2.3 Spring核心概念

在Spring核心概念这部分内容中主要包含`IOC/DI`、`IOC容器`和`Bean`,那么问题就来了，这些都是什么呢?

#### 2.3.1 目前项目中的问题

要想解答这个问题，就需要先分析下目前咱们代码在编写过程中遇到的问题

1. 业务层需要调用数据层的方法，就需要在业务层new数据层的对象
2. 如果数据层的实现类发生变化，那么业务层的代码也需要跟着改变，发生变更后，都需要进行编译打包和重部署
3. 所以，现在代码在编写的过程中存在的问题是：`耦合度偏高`

针对这个问题，该如何解决呢?

- 业务层实现
```java
public class BookServlet extends BookServlet {  
    //这个地方是写死了的  
    private BookDao bookDao = new BookDaoImpl();  
  
    public void save() {  
        bookDao.save();  
    }  
}
```

- 数据层实现
```java
public class BookDaoImpl1 implements BookDao {  
    public void save() {  
        System.out.println("book dao save ... 1");  
    }  
}

public class BookDaoImpl2 implements BookDao {  
    public void save() {  
        System.out.println("book dao save ... 2");  
    }  
}
```

我们就想，如果不new对象，只声明一下，不就可以降低依赖了吗，但是又会引入新的问题，去掉以后程序能运行吗?

- 答案显然是不行的，因为bookDao没有赋值为Null，强行运行就会出空指针异常。  
    所以现在的问题就是，业务层不想new对象，运行的时候又需要这个对象，该咋办呢?
- 针对这个问题，Spring就提出了一个解决方案:
    - `使用对象时，在程序中不要主动使用new产生对象，转换为由外部提供对象`
    - `这种实现思就是Spring的一个核心概念`
    - `IoC(Inversion of Control)控制反转`

#### 2.3.2 IOC、IOC容器、Bean、DI

1. `IOC(Inversion of Control)控制反转`

    - 那什么是控制反转呢？
        - 使用对象时，由主动new产生对象转换为由`外部`提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转。
        - 业务层要用数据层的类对象，以前是自己`new`的
        - 现在自己不new了，交给`别人[外部]`来创建对象
        - `别人[外部]`就反转控制了数据层对象的创建权
        - 这种思想就是控制反转
        - `别人[外部]`指的是什么呢?继续往下看

    - Spring和IOC之间的关系是什么呢?
        - Spring技术对IOC思想进行了实现
        - Spring提供了一个容器，称为`IOC容器`，用来充当IOC思想中的”`外部`”
        - IOC思想中的`别人[外部]`指的就是Spring的IOC容器

    - IOC容器的作用以及内部存放的是什么?
        - *IOC容器负责对象的创建、初始化等一系列工作*，其中包含了数据层和业务层的类对象
        - *被创建或被管理的对象在IOC容器中统称为Bean*
        - IOC容器中放的就是一个个的Bean对象

    - 当IOC容器中创建好service和dao对象后，程序能正确执行么?
        - 不行，因为service运行需要依赖dao对象
        - IOC容器中虽然有service和dao对象
        - 但是service对象和dao对象没有任何关系
        - `需要把dao对象交给service,也就是说要绑定service和dao对象之间的关系`
        - 像这种在容器中建立对象与对象之间的绑定关系就要用到`DI(Dependency Injection)依赖注入`

2. `DI(Dependency Injection)依赖注入`

    - 什么是依赖注入呢?
        - 在容器中建立bean与bean之间的依赖关系的整个过程，称为依赖注入
            - 业务层要用数据层的类对象，以前是自己`new`的
            - 现在自己不new了，靠`别人[外部其实指的就是IOC容器]`来给注入进来
            - 这种思想就是依赖注入

    - IOC容器中哪些bean之间要建立依赖关系呢?
        - 这个需要程序员根据业务需求提前建立好关系，如业务层需要依赖数据层，service就要和dao建立依赖关系

    - 介绍完Spring的IOC和DI的概念后，我们会发现这两个概念的最终目标就是:`充分解耦`，
    - 具体实现靠:
        - 使用IOC容器管理bean（IOC)
        - 在IOC容器内将有依赖关系的bean进行关系绑定（DI）
        - 最终结果为:***使用对象时不仅可以直接从IOC容器中获取，并且获取到的bean已经绑定了所有的依赖关系***.

#### 2.3.3 形象理解

- 从前有个人叫小明，小明有三大爱好，逛知乎，打游戏，抢红包
```java
class Ming extends Person  
{  
    private String name;  
    private int age;  
  
    void read(){  
        //逛知乎  
    }  
  
    void play(){  
        //打游戏  
    }  
  
    void grab(){  
        //抢红包  
    }  
  
}
```

- 但是小明作为一个人类，无法仅靠自己就完成上述功能，他必须`依赖`一部手机，所以他买了一台iPhone6
```java
class iPhone6 extends Iphone  
{  
    void read(String name) {  
        System.out.println(name + "打开了知乎然后编了一个故事");  
    }  
  
    void play(String name) {  
        System.out.println(name + "打开了Apex并开始白给");  
    }  
  
    void grab(String name) {  
        System.out.println(name + "开始抢红包却只抢不发");  
    }  
}
```

- 小明很珍惜自己买的新手机，每天把它牢牢控制在手心,于是小明变成了这样
```java
class Ming extends Person {  
    private String name;  
    private int age;  
  
    public Ming(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    void read() {  
        //逛知乎  
        new iPhone6().read(name);  
    }  
  
    void play() {  
        //打游戏  
        new iPhone6().play(name);  
    }  
  
    void grab() {  
        //抢红包  
        new iPhone6().grab(name);  
    }  
}
```

- 今天是周六，小明不用上班，于是他起床，并依次逛起了知乎，打起了游戏，并抢了个红包。
```java
Ming ming = new Ming("小明", 18);  //小明起床  
ming.read();  
ming.play();  
ming.grab();
```

- 这个时候，我们可以在命令行里看到输出如下
    
    > 小明打开了知乎然后编了一个故事  
    > 小明打开了Apex并开始白给  
    > 小明开始抢红包却只抢不发
    
- 这一天，小明过得很充实，他觉得自己是世界上最幸福的人。

- 但随着时间的推移，手机越来越卡顿，电池寿命也越来越短，到了冬天还会冻关机了，小明很难过，他意识到他需要换一部手机了
- 为了获得更好的使用体验，小明一咬牙一跺脚，买了一台iPhone14 Pro Max，但他现在遇到了一个问题，他之前太过依赖那台iPhone6了，他们已经深深的耦合在一起了，如果要换手机，他必须要拿螺丝刀改造自己，将自己体内所有方法中的iPhone6换成iPhone14
```java
class Ming extends Person {  
    private String name;  
    private int age;  
  
    public Ming(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    void read() {  
        //逛知乎  
        new iPhone14().read(name);  
    }  
  
    void play() {  
        //打游戏  
        new iPhone14().play(name);  
    }  
  
    void grab() {  
        //抢红包  
        new iPhone14().grab(name);  
    }  
}
```

- 虽然过程很辛苦，但小明觉得自己是值得的。随后在晚高峰挤地铁的时候，小明的手机被偷了。为了应急，小明只好重新使用那部刚刚被遗弃的iphone6，但是一想到那漫长的改造过程，小明的心里就说不出的委屈
- 他觉得自己过于依赖手机了，为什么每次手机出什么问题他都要去改造他自己，这不仅仅是过度耦合，简直是本末倒置，他向天空大喊，我不要再控制我的手机了。
- 天空中的造物主，也就是作为程序员的我，听到了他的呐喊，我告诉他，你不用再控制你的手机了，交给我来管理，把控制权交给我。这就叫做控制反转。
- 小明听到了我的话，他既高兴，又有一点害怕，他跪下来磕了几个头，虔诚地说到：“原来您就是传说中的造物主。我听到您刚刚说了 `控制反转` 四个字，就是把手机的控制权从我的手里交给你，但这只是您的想法，是一种思想罢了，要用什么办法才能实现控制反转，又可以让我继续使用手机呢？”
- “呵“，身为造物主的我在表现完不屑以后，扔下了四个大字，“依赖注入！”
- 接下来，伟大的我开始对小明进行惨无人道的改造
```java
class Ming extends Person {  
    private String name;  
    private int age;  
    private Phone phone;  
  
    public Ming(String name, int age, Phone phone) {  
        this.name = name;  
        this.age = age;  
        this.phone = phone;  
    }  
  
    void read() {  
        //逛知乎  
        this.phone.read(name);  
    }  
  
    void play() {  
        //打游戏  
        this.phone.play(name);  
    }  
  
    void grab() {  
        //抢红包  
        this.phone.grab(name);  
    }  
}
```

- 随后我们来模拟小明的一天
```java
Phont phone = new Iphone14();   //创建一个iphone14的实例  
if(phone.isBroken() == true){   //如果iphone14不可用，则使用旧版手机  
    phone = new Iphone6();  
}  
Ming ming = new Ming("小明",18,phone);    //小明不用关心是什么手机，他只要玩就行了。  
ming.read();  
ming.play();  
ming.grab();
```

- 我们先看一下iphone14 是否可以使用，如果不可以使用，则直接换成iphone6,然后唤醒小明，并把手机塞到他的手里，换句话说，把他所依赖的手机直接注入到他的身上，他不需要关心自己拿的是什么手机，他只要直接使用就可以了。
- 这就是`依赖注入`。
- 随后小明的生活开始变得简单了起来，而他把省出来的时间都用来写笔记了，他在笔记本上这样写到
>我曾经有很强的控制欲，过度依赖于我的手机，导致我和手机之间耦合程度太高，只要手机出现一点点问题，我都要改造我自己，这实在是既浪费时间又容易出问题。自从我把控制权交给了造物主，他每天在唤醒我以前，就已经替我选好了手机，我只要按照平时一样玩手机就可以了，根本不用关心是什么手机。即便手机出了问题，也可以由造物主直接搞定，不需要再改造我自己了，我现在买了七部手机，都交给了造物主，每天换一部，美滋滋！  
>我也从其中获得了这样的感悟： 如果一个类A 的功能实现需要借助于类B，那么就称类B是类A的依赖，如果在类A的内部去实例化类B，那么两者之间会出现较高的耦合，一旦类B出现了问题，类A也需要进行改造，如果这样的情况较多，每个类之间都有很多依赖，那么就会出现牵一发而动全身的情况，程序会极难维护，并且很容易出现问题。要解决这个问题，就要把A类对B类的控制权抽离出来，交给一个第三方去做，把控制权反转给第三方，就称作控制反转（IOC Inversion Of Control）。控制反转是一种思想，是能够解决问题的一种可能的结果，而依赖注入（Dependency Injection）就是其最典型的实现方法。由第三方（我们称作IOC容器）来控制依赖，把他通过构造函数、属性或者工厂模式等方法，注入到类A内，这样就极大程度的对类A和类B进行了解耦。

#### 2.3.4 核心概念小结

重点要理解`什么是IOC/DI思想`、`什么是IOC容器`和`什么是Bean`：

1. 什么IOC/DI思想?
    - IOC:控制反转，控制反转的是对象的创建权
    - DI:依赖注入，绑定对象与对象之间的依赖关系
2. 什么是IOC容器?
    - Spring创建了一个容器用来存放所创建的对象，这个容器就叫IOC容器
3. 什么是Bean?
    - 容器中所存放的一个个对象就叫Bean或Bean对象

## 3 入门案例

- 介绍完Spring的核心概念后，接下来我们得思考一个问题就是，Spring到底是如何来实现IOC和DI的，那接下来就通过一些简单的入门案例，来演示下具体实现过程:

### 3.1 IOC入门案例

对于入门案例，我们得先`分析思路`然后再`代码实现`

#### 3.1.1 思路分析

1. Spring是使用容器来管理bean对象的，那么`管什么?`
    - 主要`管理项目中所使用到的类对象`，比如(Service和Dao)
2. 如何`将被管理的对象告知IOC容器?`
    - 使`用配置文件`
3. 被管理的对象交给IOC容器，要想从容器中获取对象，就先得思考`如何获取到IOC容器?`
    - `Spring框架提供相应的接口`
4. IOC容器得到后，如何从容器中`获取bean?`
    - 调用Spring框架提供对应`接口中的方法`
5. 使用Spring`导入哪些坐标?`
    - 用别人的东西，就需要`在pom.xml添加对应的依赖`


- 总结：管Service和Dao、xml配置到IoC、使用接口获取IoC、使用接口中的方法获取IoC中的Bean、提前需要导pom文件中对应的依赖

#### 3.1.2 代码实现

- 需求分析:将BookServiceImpl和BookDaoImpl交给Spring管理，并从容器中获取对应的bean对象进行方法调用。

1. 创建Maven的java项目  
    这个没啥好说的
2. pom.xml添加Spring的依赖jar包
```xml
<dependency>  
  <groupId>org.springframework</groupId>  
  <artifactId>spring-context</artifactId>  
  <version>5.2.10.RELEASE</version>  
</dependency>
```
3. 创建BookDao，BookDaoImpl，BookService和BookServiceImpl四个类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService {  
    private BookDao bookDao = new BookDaoImpl();  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

4. resources下添加spring配置文件

>右键 -> 新建 -> XML配置文件 -> Spring配置

5. 在配置文件中完成bean的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
      <!--  
      bean标签标示配置bean  
      id属性标示给bean起名字  
      class属性表示给bean定义类型，  
      得是具体的实现类而不是接口，要靠这个造对象的  
      -->  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl"/>  
</beans>
```

>`注意事项：bean定义时id属性在同一个上下文中(配置文件)不能重复`

6. 获取IOC容器  
	使用Spring提供的接口完成IOC容器的创建，创建App类，编写main方法
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
  
    }  
}
```

7. 从容器中获取对象进行方法调用  
	使用getBean(String name)方法，其name参数就是我们在bean配置的id，通过这个id来造对象
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookService bookService = (BookService) context.getBean("bookService");  
        bookService.save();  
    }  
}
```

8. 运行程序  
	测试结果如下
	>book service save …  
	>book dao save …
	
- 至此，Spring的IOC入门案例已经完成，但是在`BookServiceImpl`的类中依然存在`BookDaoImpl`对象的new操作，它们之间的耦合度还是比较高，这块该如何解决，就需要用到下面的`DI(依赖注入)`。

### 3.2 DI入门案例

- 对于DI的入门案例，我们依然先`分析思路`然后再`代码实现`

#### 3.2.1 思路分析

1. 要想实现依赖注入，必须要基于IOC管理Bean
    - DI的入门案例要依赖于前面的IOC入门案例
2. Service中使用new形式创建的Dao对象是否保留？
    - `不保留，这样才能解耦合，最终要使用IOC容器中的bean对象`
3. Service中需要的Dao对象如何进入到Service中？
    - `在Service中提供一个方法（例如提供一个set方法），让Spring的IOC容器可以通过该方法传入bean对象`，也就达到了不是自己new，而是外部提供
4. `Service与Dao之间的关系如何描述？`
    - `使用配置文件`

#### 3.2.2 代码实现

需求：基于IOC入门案例，在BookServiceImpl类中删除new对象的方式，使用Spring的DI完成Dao层的注入

1. 删除业务层中使用new的方式创建的dao对象
```java
public class BookServiceImpl implements BookService {  
    //private BookDao bookDao = new BookDaoImpl();  
    private BookDao bookDao;  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

2. 在业务层提供BookDao的setter方法  
	我们在set方法中加一条输出语句，看看是否被调用了
```java
public class BookServiceImpl implements BookService {  
    private BookDao bookDao;  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
        System.out.println("set方法被调用啦");  
    }  
}
```

3. 在配置文件中添加依赖注入的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
  
    <!--主要变化在这里-->  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <!--配置server与dao的关系-->  
        <!--  
            property标签表示配置当前bean的属性  
            name属性表示配置哪一个具体的属性(这里是配置bookService的bookDao属性)  
            ref属性表示参照哪一个bean(参照当前配置文件中的bookDao)  
        -->  
        <property name="bookDao" ref="bookDao"></property>  
    </bean>  
</beans>
```

>- `注意:配置中的两个bookDao的含义是不一样的`
    - name=”bookDao”中`bookDao`的作用是让Spring的IOC容器在获取到名称后，将首字母大写，前面加set找对应的`setBookDao()`方法进行对象注入
    - ref=”bookDao”中`bookDao`的作用是让Spring能在IOC容器中找到id为`bookDao`的Bean对象给`bookService`进行注入

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8af38d0df81e11cf335ddfd264b9ab4.png)



4. 运行程序调用方法  
	测试结果如下
	>set方法被调用啦  
	>book service save …  
	>book dao save …

## 4 IOC相关内容

通过前面两个案例，我们已经学习了`bean如何定义配置`，`DI如何定义配置`以及`容器对象如何获取`的内容，接下来主要是把这三块内容展开进行详细的讲解，深入的学习下这三部分的内容，首先是bean基础配置。
### 4.1 bean基础配置

对于bean的配置中，主要会讲解`bean基础配置`,`bean的别名配置`,`bean的作用范围配置`(重点),这三部分内容
#### 4.1.1 bean的基础配置---id和class

- 对于bean的基础配置，在前面的案例中已经使用过:
```xml
<bean id="" class=""/>
```

- 其中，bean标签的功能、使用方式以及id和class属性的作用，我们通过一张图来描述下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a1fe7258b8e8c45091d7d4587c8d3a8f.png)

#### 4.1.2 bean的name属性

我们可以在bean标签中配置name属性，来充当别名，下面我们来演示

1. 配置别名  
    打开spring的配置文件`applicationContext.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <!--name:为bean指定别名，别名可以有多个，使用逗号，分号，空格进行分隔-->  
    <bean id="bookService" name="service1 service2 service3" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"></property>  
    </bean>  
</beans>
```

2. 根据名称容器中获取bean对象
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        //此处根据bean标签的id属性和name属性的任意一个值来获取bean对象  
        BookService bookService = (BookService) context.getBean("service2");  
        bookService.save();  
    }  
}
```

3. 运行程序  
	测试结果如下：
	>set方法被调用啦  
	>book service save …  
	>book dao save …

`注意事项`：
- bean依赖注入的ref属性指定bean，必须在容器中存在，而ref的值也可以是name里的别名，不过还是建议用id值来注入
- 如果我们在调用getBean(String name)方法时，传入了一个不存在该名称的bean对象，则会报错`NoSuchBeanDefinitionException`，此时我们要检查一下是哪边写错了（例如bean的id和name都没有service100，而getBean的参数却写了service100）
#### 4.1.3 bean作用范围scope配置

关于bean的作用范围是bean属性配置的一个重点内容。  
bean的scope有两个取值：

- `singleton：单例（默认）`
- `prototype：非单例`
##### 4.1.3.1 验证IOC容器中对象是否为单例

- 验证思路：我们只需要对同一个bean创建两个对象，然后打印二者的地址值，看看是否一致
- 代码实现
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        //我这里使用了别名，其实还是同一个bean  
        BookService bookService2 = (BookService) context.getBean("service2");  
        BookService bookService3 = (BookService) context.getBean("service3");  
        System.out.println(bookService2);  
        System.out.println(bookService3);  
    }  
}
```

输出结果如下，地址值一致，确实是单例的
>com.blog.service.impl.BookServiceImpl@25bbe1b6  
>com.blog.service.impl.BookServiceImpl@25bbe1b6

- 那如果我想创建出来非单例的bean对象，该如何实现呢?
    
    - 配置bean的scope属性为prototype

##### 4.1.3.2 配置bean为非单例

在Spring配置文件中，配置scope属性来实现bean的非单例创建

- 在Spring的配置文件中，修改`<bean>`的scope属性
```xml
<bean id="bookService" name="service1 service2 service3" class="com.blog.service.impl.BookServiceImpl" scope="prototype">  
    <property name="bookDao" ref="bookDao"></property>  
</bean>
```

- 运行程序  
	输出结果如下，地址值不一致
	>com.blog.service.impl.BookServiceImpl@25bbe1b6  
	>com.blog.service.impl.BookServiceImpl@5702b3b1

##### 4.1.3.3 scope使用后续思考

介绍完`scope`属性以后，我们来思考几个问题:

- 为什么bean默认为单例?
    - bean为单例的意思是在Spring的IOC容器中只会有该类的一个对象
    - bean对象只有一个就`避免了对象的频繁创建与销毁，达到了bean对象的复用，性能高`
- bean在容器中是单例的，会不会产生线程安全问题?
    - 如果对象是有状态对象，即该对象有成员变量可以用来存储数据的，
    - 因为所有请求线程共用一个bean对象，所以会存在线程安全问题。
    - 如果对象是无状态对象，即该对象没有成员变量没有进行数据存储的，
    - 因方法中的局部变量在方法调用完成后会被销毁，所以不会存在线程安全问题。
- 哪些bean对象适合交给容器进行管理?
    - `表现层对象（controller）`
    - `业务层对象（service）`
    - `数据层对象（dao）`
    - `工具对象（util）`
- 哪些bean对象不适合交给容器进行管理?
    - 封装实例的域对象（domain，pojo），因为会引发线程安全问题，所以不适合。

### 4.2 bean实例化

- 对象已经能交给Spring的IOC容器来创建了，但是容器是如何来创建对象的呢?
    
    - 就需要研究下`bean的实例化过程`，在这块内容中主要解决两部分内容，分别是
        - bean是如何创建的
        - 实例化bean的三种方式，`构造方法`,`静态工厂`和`实例工厂`
- 在讲解这三种创建方式之前，我们需要先确认一件事:
    
    - bean本质上就是对象，对象在new的时候会使用构造方法完成，那创建bean也是使用构造方法完成的。
        - 基于这个知识点出发，我们来验证spring中bean的三种创建方式，

#### 4.2.1 构造方法实例化

- 在之前的BookDaoImpl类中添加一个无参构造函数，并打印一句话，方便观察结果。
```java
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
  
    public BookDaoImpl() {  
        System.out.println("book dao constructor is running ...");  
    }  
}
```

- 运行程序  
	输出结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae805c243e106482d925ca1eabe6002f.png)


- 接下来我们将构造器私有化继续测试
```java
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
  
    private BookDaoImpl() {  
        System.out.println("book dao constructor is running ...");  
    }  
}
```

- 运行程序，能执行成功，说明内部走的依然是构造函数，能访问到类中的私有构造方法，`显而易见Spring底层用的是反射`![![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4132247c513c7af71a1388f6c2daa538.png)


- 我们在构造函数中添加一个参数试试
```java
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
  
    public BookDaoImpl(int i) {  
        System.out.println("book dao constructor is running ...");  
    }  
}
```

- 运行程序，程序会报错`NoSuchMethodException`，说明`Spring底层使用的是类的无参构造方法。`

- 完事儿之后记得将空参构造器还原回去，把其中的输出语句也删了，方便我们进行下一步的测试

#### 4.2.2 静态工厂实例化（了解）

- 创建一个工厂类BookDaoFactory并提供一个静态方法
```java
//静态工厂创建对象  
public class BookDaoFactory {  
    public static BookDao getBookDaoImpl(){  
        return new BookDaoImpl();  
    }  
}
```

- 修改App运行类，在类中通过工厂获取对象
```java
public class App {  
    public static void main(String[] args) {  
        //通过静态工厂创建对象  
        BookDao bookDao = BookDaoFactory.getBookDaoImpl();  
        bookDao.save();  
    }  
}
```

- 运行后，可以查看到结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0b80c8b113d4409c7f2bd6ee2d0bc42.png)


- 那我们如何将这种方式交给Spring来管理呢？

- 这就要用到Spring中的静态工厂实例化的知识了，具体实现步骤为:

1. 在spring的配置文件`application.properties`修改bookDao的bean
```xml
<bean id="bookDao" class="com.blog.factory.BookDaoFactory" factory-method="getBookDaoImpl"/>
```
class:工厂类的类全名  
factory-mehod:具体工厂类中创建对象的方法名

2. 在App运行类，使用从IOC容器中获取bean的方法进行运行测试
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
    }  
}
```

3. 运行后，结果如下，与我们自己直接new对象没太大区别，而且还麻烦了，那这种方式的意义是什么呢？![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/06fa1a7c1d915201922c22da321262d7.png)


4. 解惑  
	在工厂的静态方法中，我们除了new对象还可以做其他的一些业务操作，而之前new对象的方式就无法添加其他的业务内容
```java
public class BookDaoFactory {  
    public static BookDao getBookDaoImpl() {  
        System.out.println("book dao factory setup ...");//模拟必要的业务操作  
        //这里还可以加一大堆业务逻辑  
        return new BookDaoImpl();  
    }  
}
```

- 介绍完静态工厂实例化后，这种方式一般是用来兼容早期的一些老系统，所以`了解为主`。

#### 4.2.3 实例工厂和FactoryBean

##### 4.2.3.1 实例工厂实例化（了解）

接下来继续来研究Spring的第三种bean的创建方式`实例工厂实例化`

- 修改工厂类`BookDaoFactory`的get方法，注意此处和静态工厂的工厂类不一样的地方是方法`不是静态方法`
```java
public class BookDaoFactory {  
    //唯一的区别就是去掉的static  
    public BookDao getBookDaoImpl() {  
        System.out.println("book dao factory setup ...");//模拟必要的业务操作  
        //这里还可以加一大堆业务逻辑  
        return new BookDaoImpl();  
    }  
}
```

- 修改App运行类，在类中通过工厂获取对象，由于不是静态方法了，所以我们需要先创建实例工厂对象，然后再用实例工厂对象调用方法
```java
public class App {  
    public static void main(String[] args) {  
        //创建实例工厂对象  
        BookDaoFactory bookDaoFactory = new BookDaoFactory();  
        //通过实例工厂对象创建对象  
        BookDao bookDao = bookDaoFactory.getBookDaoImpl();  
        bookDao.save();  
    }  
}
```

- 运行结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/959a8d4e6b3d14644b6653dba22c6839.png)


- 那么对于上面这种实例工厂的方式如何交给Spring管理呢?

1. 在spring配置文件中修改bookDao的bean
```xml
<bean id="bookDaoFactory" class="com.blog.factory.BookDaoFactory"/>  
<bean id="bookDao" factory-bean="bookDaoFactory" factory-method="getBookDaoImpl"/>
```

实例化工厂运行的顺序是:

- 创建实例化工厂对象,对应的是第一行配置
- 调用对象中的方法来创建bean，对应的是第二行配置
    - factory-bean:工厂的实例对象
    - factory-method:工厂对象中的具体创建对象的方法名

2. 在App运行类，使用从IOC容器中获取bean的方法进行运行测试
```java
public class App {  
    public static void main(String[] args) {  
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
    }  
}
```

3. 运行后，可以查看到结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/21b5abccbe9c92b021e0c1f3e6e7144c.png)


- 实例工厂实例化的方式就已经介绍完了，配置的过程还是比较复杂，要写两行配置，而且这两行还是高耦合的，所以Spring为了简化这种配置方式就提供了一种叫`FactoryBean`的方式来简化开发。

##### 4.2.3.2 FactoryBean（重点）

- 具体的使用步骤为：

1. 创建一个`BookDaoFactoryBean`类，实现`FactoryBean`接口，重写接口方法
```java
public class BookDaoFactoryBean implements FactoryBean<BookDao> {  
  
    public BookDao getObject() throws Exception {  
        return new BookDaoImpl();  
    }  
  
    public Class<?> getObjectType() {  
        return BookDao.class;  
    }  
}
```

2. 在Spring的配置文件中修改bookDao的bean
```xml
<bean id="bookDao" class="com.blog.factory.BookDaoFactoryBean"></bean>
```

3. App运行类不用做任何修改，直接运行，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e518496c4499a5fb6f60369ed2399af8.png)


- 这种方式在Spring去整合其他框架的时候会被用到，所以这种方式需要我们理解掌握。

- 查看源码会发现，FactoryBean接口其实会有三个方法，分别是
```java
T getObject() throws Exception;  
  
Class<?> getObjectType();  
  
default boolean isSingleton() {  
    return true;  
}
```

- 方法一：getObject()，被重写后，在方法中进行对象的创建并返回
    
- 方法二：getObjectType(),被重写后，主要返回的是被创建类的Class对象
    
- 方法三：没有被重写，因为它已经给了默认值，从方法名中可以看出其作用是设置对象是否为单例，默认true，这里就不加以验证了

#### 4.2.4 bean实例化小结

- bean是如何创建的呢?
    - 通过构造方法
- Spring的IOC实例化对象的三种方式分别是:
    - 构造方法(常用)
    - 静态工厂(了解)
    - 实例工厂(了解)
        - FactoryBean(实用)  
            这些方式中，重点掌握`构造方法`和`FactoryBean`即可。
            >需要注意的一点是，构造方法在类中默认会提供，但是如果重写了构造方法，默认的就会消失，在使用的过程中需要注意，如果需要重写构造方法，最好把默认的构造方法也重写下。

### 4.3 bean的生命周期

关于bean的相关知识还有最后一个是`bean的生命周期`，对于生命周期，我们主要围绕着`bean生命周期控制`来讲解

- 首先理解下什么是生命周期?
    - 从创建到消亡的完整过程,例如人从出生到死亡的整个过程就是一个生命周期。
- bean生命周期是什么?
    - `bean对象从创建到销毁的整体过程`。
- bean生命周期控制是什么?
    - `在bean创建后到销毁前做一些事情`。
- 现在我们面临的问题是如何在bean的创建之后和销毁之前把我们需要添加的内容添加进去。

#### 4.3.1 生命周期设置

具体的控制有两个阶段:

- bean创建之后，想要添加内容，比如用来初始化需要用到资源
- bean销毁之前，想要添加内容，比如用来释放用到的资源

1. 添加初始化和销毁方法  
    针对这两个阶段，我们在BookDaoImpl类中分别添加两个方法，方法名随便取
```java
public class BookDaoImpl implements BookDao {  
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

2. 配置生命周期  
	修改bookDao的配置
```xml
<bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl" init-method="init" destroy-method="destroy"></bean>
```

3. 运行程序
	输出结果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/64f655f1b8fbf447366317898ff54e65.png)


从结果中可以看出，init方法执行了，但是destroy方法却未执行，这是为什么呢?

- Spring的IOC容器是运行在JVM中
- 运行main方法后,JVM启动,Spring加载配置文件生成IOC容器,从容器获取bean对象，然后调方法执行
- `main方法执行完后，JVM退出，这个时候IOC容器中的bean还没有来得及销毁就已经结束了`
- 所以没有调用对应的destroy方法

知道了出现问题的原因，具体该如何解决呢?继续往下看

#### 4.3.2 close关闭容器

- `ApplicationContext中没有close方法，它的子类中有close方法`
- `所以需要将ApplicationContext更换成ClassPathXmlApplicationContext，然后调用close方法就好啦`
```java
public class App {  
    public static void main(String[] args) {  
        //ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
        context.close();  
    }  
}
```

- 运行程序，输出如下，可以看到destory正常输出：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c07a853afa85b204f2e0fd74b9734f46.png)


#### 4.3.3 注册钩子关闭容器

- 在容器未关闭之前，提前设置好回调函数，让JVM在退出之前回调此函数来关闭容器
    
- `调用context的registerShutdownHook()方法`
```java
public class App {  
    public static void main(String[] args) {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
        context.registerShutdownHook();  
    }  
}
```

>注意：registerShutdownHook在ApplicationContext中也没有

- 运行后，查询打印结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0faa303835aeed65d486e3dbe3e9a71.png)

- 那两种方式介绍完后，close和registerShutdownHook选哪个?

	- 相同点:这两种都能用来关闭容器
	- 不同点:close()是在调用的时候关闭，registerShutdownHook()是在JVM退出前调用关闭。
	    - 那么registerShutdownHook()方法可以在任意位置调用，下面的代码中将其放在了第二行，仍能正常输出，但要是将其换成close()方法，则会报错`BeanFactory not initialized or already closed`，这里就是already closed
```java
public class App {  
    public static void main(String[] args) {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        context.registerShutdownHook();  
        BookDao bookDao = (BookDao) context.getBean("bookDao");  
        bookDao.save();  
    }  
}
```

- 开发中到底用哪个呢？
    - 答案是两个都不用
    - 分析上面的实现过程，会发现添加初始化和销毁方法，即需要编码也需要配置，实现起来步骤比较多也比较乱。
    - Spring给我们提供了两个接口来完成生命周期的控制，好处是可以不用再进行配置`init-method`和`destroy-method`


- `接下来在BookServiceImpl完成这两个接口的使用（了解）`
    
    - 修改BookServiceImpl类，添加两个接口`InitializingBean`， `DisposableBean`并实现接口中的两个方法`afterPropertiesSet`和`destroy`

```java
public class BookServiceImpl implements BookService, InitializingBean, DisposableBean {  
    private BookDao bookDao;  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
  
    public void setBookDao(BookDao bookDao) {  
        System.out.println("set ... ");  
        this.bookDao = bookDao;  
    }  
  
    public void destroy() throws Exception {  
        System.out.println("service destroy ... ");  
    }  
  
    public void afterPropertiesSet() throws Exception {  
        System.out.println("service init ... ");  
    }  
}
```

- BookServiceImpl的bean配置如下
```xml
<bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
    <property name="bookDao" ref="bookDao"></property>  
</bean>
```

- 重新运行App类，输出结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4a5ab3f68bde7ba1c587189c07fdde17.png)



- `小细节`
	- 对于InitializingBean接口中的afterPropertiesSet方法，翻译过来为`属性设置之后`。
	- 对于BookServiceImpl来说，bookDao是它的一个属性
	- setBookDao方法是Spring的IOC容器为其注入属性的方法
	- 思考:afterPropertiesSet和setBookDao谁先执行?
	    - 从方法名分析，猜想应该是setBookDao方法先执行
	    - 验证思路，在setBookDao方法中添加一局输出语句，看看谁先输出

```java
public void setBookDao(BookDao bookDao) {  
    System.out.println("set ... ");  
    this.bookDao = bookDao;  
}
```

- 重新运行，结果如下，`service init`在`set`之后，符合我们的预期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e579b608ec0391b222a905800bd2b48.png)


>可能存在的疑问：明明我们没调用bookService，但为什么上面的输出结果中有service呢？  
>
>解惑：所有IOC容器中的bean的初始化和销毁都会运行，所以service也会运行

#### 4.3.4 bean生命周期小结

1. 关于Spring中对bean生命周期控制提供了两种方式:
    - 在配置文件中的bean标签中添加`init-method`和`destroy-method`属性
    - 类实现`InitializingBean`与`DisposableBean`接口
2. 对于bean的生命周期控制在bean的整个生命周期中所处的位置如下
    - 初始化容器
        - 1.创建对象(内存分配)
        - 2.执行构造方法
        - 3.执行属性注入(set操作)（`set ...`）
        - 4.执行bean初始化方法（`service init ...`）
    - 使用bean
        - 执行业务操作（`book dao save ...`）
    - 关闭/销毁容器
        - 执行bean销毁方法（`service destroy ...`）
3. 关闭容器的两种方式:
    - ConfigurableApplicationContext是ApplicationContext的子类，子类才有下面两种方法
        - close()方法
        - registerShutdownHook()方法


## 5 DI相关内容

上面我们已经完成了bean相关操作的讲解，接下来就进入第二个大的模块`DI依赖注入`。

我们先来思考

- 向一个类中传递数据的方式有几种?
    - `普通方法(set方法)`
    - `构造方法`

- 依赖注入描述了在容器中建立bean与bean之间的依赖关系的过程，如果bean运行需要的是数字或字符串呢?
    - `引用类型`
    - `简单类型(基本数据类型与String)`

- Spring就是基于上面这些知识点，为我们提供了两种注入方式，分别是:
    - `setter注入`
        - 简单类型
        - 引用类型
    - `构造器注入`
        - 简单类型
        - 引用类型

### 5.1 setter注入

- 对于setter方式注入引用类型的方式之前已经学习过，快速回顾下:
- 在bean中定义引用类型属性，并提供可访问的set方法
```java
public class BookServiceImpl implements BookService {  
    private BookDao bookDao;  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
}
```

- 配置中使用property标签ref属性注入引用类型对象
```xml
<bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
    <property name="bookDao" ref="bookDao"></property>  
</bean>
```

- 我们再来回顾一下配置中的两个bookDao的含义
>配置中的两个bookDao的含义是不一样的
>
>	name=”bookDao”中bookDao的作用是让Spring的IOC容器在获取到名称后，将首字母大写，前面加set找对应的setBookDao()方法进行对象注入
>	 
>	ref=”bookDao”中bookDao的作用是让Spring能在IOC容器中找到id为bookDao的Bean对象给bookService进行注入

#### 5.1.1 环境准备

- 先来做一些准备工作，我们在dao包下新建一个UserDao接口
```java
public interface UserDao {  
    public void save();  
}
```

- 然后新建一个UserDaoImpl类实现UserDao接口
```java
public class UserDaoImpl implements UserDao {  
    public void save() {  
        System.out.println("user dao save ...");  
    }  
}
```

- 然后修改我们之前的BookDao等类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService {  
    private BookDao bookDao;  
  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

- 配置文件如下
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"></bean>  
  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"></property>  
    </bean>  
</beans>
```

- 修改App运行类，加载Spring的IOC容器，并从中获取对应的bean对象
```java
public class App {  
    public static void main(String[] args) {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookService bookService = (BookService) context.getBean("bookService");  
        bookService.save();  
    }  
}
```

#### 5.1.2 注入引用数据类型

需求:在bookServiceImpl对象中注入userDao

1. 在BookServiceImpl中声明userDao属性
2. 为userDao属性提供setter方法
3. 在配置文件中使用property标签注入


-  `步骤一：`声明userDao属性并提供setter方法
```java
public class BookServiceImpl implements BookService {  
    private BookDao bookDao;  
    //声明属性  
    private UserDao userDao;  
    //提供setter  
    public void setUserDao(UserDao userDao) {  
        this.userDao = userDao;  
    }  
  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
        userDao.save();  
    }  
}
```

- `步骤二：`在配置文件中注入配置  
	在applicationContext.xml配置文件中使用property标签注入
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"></bean>  
    <bean id="userDao" class="com.blog.dao.impl.UserDaoImpl"></bean>  
  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"></property>  
        <property name="userDao" ref="userDao"></property>  
    </bean>  
</beans>
```

- `步骤三：`运行程序，结果如下，userDao已经成功注入。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f0e59d2cddbc07442a7316a5beff23a2.png)


#### 5.1.3 注入简单数据类型

需求：给BookDaoImpl注入一些简单数据类型的数据  
参考引用数据类型的注入，我们可以推出具体的步骤为:

1. 在BookDaoImpl类中声明对应的简单数据类型的属性
2. 为这些属性提供对应的setter方法
3. 在applicationContext.xml中配置

**思考:**

- 引用类型使用的是`<property name="" ref=""/>`,简单数据类型还是使用ref吗?
- ref是指向Spring的IOC容器中的另一个bean对象的，对于简单数据类型，没有对应的bean对象，该如何配置呢?使用value来配置`<property name="" value=""/>`
    
- `步骤一：`声明属性并提供setter方法  
    这里举例就用`String dataBaseName`和`int connectionCount`这两个属性，同时在save()方法的输出语句中加上这两个属性
```java
public class BookDaoImpl implements BookDao {  
    private String dataBaseName;  
    private int connectionCount;  
  
    public void setDataBaseName(String dataBaseName) {  
        this.dataBaseName = dataBaseName;  
    }  
  
    public void setConnectionCount(int connectionCount) {  
        this.connectionCount = connectionCount;  
    }  
  
    public void save() {  
        System.out.println("book dao save ..." + dataBaseName + "," + connectionCount);  
    }  
}
```

- `步骤二：`在配置文件中进行注入配置  
	在applicationContext.xml配置文件中使用property标签注入
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl">  
        <property name="dataBaseName" value="mysql"></property>  
        <property name="connectionCount" value="100"></property>  
    </bean>  
    <bean id="userDao" class="com.blog.dao.impl.UserDaoImpl"></bean>  
  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"></property>  
        <property name="userDao" ref="userDao"></property>  
    </bean>  
</beans>
```
>value:后面跟的是简单数据类型，对于参数类型，Spring在注入的时候会自动转换，但是不能写一个错误的类型，例如`connectionCount`是`int`类型，你却给他传一个`abc`，这样的话，spring在将`abc`转换成int类型的时候就会报错。

- `步骤三：`运行程序，查看输出，两个简单数据类型也成功注入![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f3ebab79d852245c097d531fb8c0e1d.png)


- 那么对于setter注入方式的基本使用就已经介绍完了
    
    - 对于引用数据类型使用的是`<property name="" ref=""/>`
    - 对于简单数据类型使用的是`<property name="" value=""/>`

### 5.2 构造器注入（了解）

#### 5.2.1 环境准备

- 修改BookDao、BookDaoImpl、UserDao、UserDaoImpl、BookService和BookServiceImpl类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
      
    private String databaseName;  
    private int connectionNum;  
      
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}

public interface UserDao {  
    public void save();  
}

public class UserDaoImpl implements UserDao {  
    public void save() {  
        System.out.println("user dao save ...");  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService{  
    private BookDao bookDao;  
  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

- 配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"/>  
    </bean>  
</beans>
```

- 运行类
```java
public class App {  
    public static void main( String[] args ) {  
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookService bookService = (BookService) ctx.getBean("bookService");  
        bookService.save();  
    }  
}
```
#### 5.2.2 构造器注入引用数据类型

接下来，在上面这个环境中来完成构造器注入的学习:

需求：将BookServiceImpl类中的bookDao修改成使用构造器的方式注入。

1. 将bookDao的setter方法删除掉
2. 添加带有bookDao参数的构造方法
3. 在applicationContext.xml中配置

- `步骤一：`删除setter方法并提供构造方法  
	在BookServiceImpl类中将bookDao的setter方法删除掉,并添加带有bookDao参数的构造方法
```java
public class BookServiceImpl implements BookService{  
    private BookDao bookDao;  
  
    public BookServiceImpl(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

- `步骤二：`配置文件中进行配置构造方式注入  
	在applicationContext.xml中配置
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <constructor-arg name="bookDao" ref="bookDao"/>  
    </bean>  
</beans>
```
>说明：在标签`<constructor-arg>`中
>	name属性对应的值为构造函数中方法`形参的参数名`，必须要保持一致。
>	ref属性指向的是spring的IOC容器中其他bean对象。

- `步骤三：`运行程序  
	运行App类，查看结果，说明bookDao已经成功注入。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2db0a6b9e7ee146b341add7da4e5bf16.png)


#### 5.2.3 构造器注入多个引用数据类型

需求：在BookServiceImpl使用构造函数注入多个引用数据类型，比如userDao

1. 声明userDao属性
2. 生成一个带有bookDao和userDao参数的构造函数
3. 在applicationContext.xml中配置注入

- `步骤一：`提供多个属性的构造函数  
	在BookServiceImpl声明userDao并提供多个参数的构造函数，save方法中记得调用userDao.save()
```java
public class BookServiceImpl implements BookService {  
    private BookDao bookDao;  
    private UserDao userDao;  
  
    public BookServiceImpl(BookDao bookDao, UserDao userDao) {  
        this.bookDao = bookDao;  
        this.userDao = userDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
        userDao.save();  
    }  
}
```

- `步骤二：`在配置文件中配置多参数注入
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="userDao" class="com.blog.dao.impl.UserDaoImpl"/>  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <constructor-arg name="bookDao" ref="bookDao"/>  
        <constructor-arg name="userDao" ref="userDao"/>  
    </bean>  
</beans>
```

- `步骤三：`运行程序  
	结果中出现了userDao的输出，说明userDao成功注入![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b36b2704b695f8fadd52548005a169c9.png)


#### 5.2.4 构造器注入多个简单数据类型

需求:在BookDaoImpl中，使用构造函数注入databaseName和connectionNum两个参数。  
参考引用数据类型的注入，我们可以推出具体的步骤为:

1. 提供一个包含这两个参数的构造方法
2. 在applicationContext.xml中进行注入配置

- `步骤一：`添加多个简单属性并提供构造方法  
	修改BookDaoImpl类，添加构造方法，同时在save()方法中输出这两个属性
```java
public class BookDaoImpl implements BookDao {  
  
    private String databaseName;  
    private int connectionNum;  
  
    public BookDaoImpl(String databaseName, int connectionNum) {  
        this.databaseName = databaseName;  
        this.connectionNum = connectionNum;  
    }  
  
    public void save() {  
        System.out.println("book dao save ..." + databaseName + "," + connectionNum);  
    }  
}
```

- `步骤二：`配置完成多个属性构造器注入
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="userDao" class="com.blog.dao.impl.UserDaoImpl"/>  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl">  
        <constructor-arg name="databaseName" value="mysql"></constructor-arg>  
        <constructor-arg name="connectionNum" value="100"></constructor-arg>  
    </bean>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <constructor-arg name="bookDao" ref="bookDao"/>  
        <constructor-arg name="userDao" ref="userDao"/>  
    </bean>  
</beans>
```

- `步骤三：`运行程序![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b53d7e588732215bcdb25d03217cbc64.png)



#### 5.2.5 目前存在的问题

- `<constructor-arg>`标签内的name，必须与构造函数中的参数名一致，这两块存在紧耦合。
- 那么我们怎么解决这个问题呢？
- 在解决这个问题之前，需要提前说明的是，这个参数名发生变化的情况并不多，所以上面的还是比较主流的配置方式，下面介绍的，我们以了解为主。

- `方式一`：删除name属性，添加type属性，按照类型注入
    - 这种方式可以解决构造函数形参名发生变化带来的耦合问题
    - 但是如果构造方法参数中有类型相同的参数，这种方式就不太好实现了
```xml
<bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl">  
    <constructor-arg type="java.lang.String" value="mysql"></constructor-arg>  
    <constructor-arg type="int" value="9421"></constructor-arg>  
</bean>
```

- `方式二`：删除type属性，添加index属性，按照索引下标注入，下标从0开始

- 这种方式可以解决参数类型重复问题
- 但是如果构造方法参数顺序发生变化后，这种方式又带来了耦合问题
```xml
<bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl">  
    <constructor-arg index="0" value="mysql"></constructor-arg>  
    <constructor-arg index="1" value="9421"></constructor-arg>  
</bean>
```

介绍完两种参数的注入方式，具体我们该如何选择呢?

1. 强制依赖使用构造器进行，使用setter注入有概率不进行注入导致null对象出现
    - 强制依赖指对象在创建的过程中必须要注入指定的参数
2. 可选依赖使用setter注入进行，灵活性强
    - 可选依赖指对象在创建过程中注入的参数可有可无
3. Spring框架倡导使用构造器，第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
4. 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
5. 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
6. `自己开发的模块推荐使用setter注入`
#### 5.2.6 小结

这部分主要讲解的是Spring的依赖注入的实现方式:

- setter注入
	- 简单数据类型![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/812830b3c6d99605e21958525e7ff02a.png)


	- 引用数据类型![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c47a07540892fdd9071fd6d6c22c8ef.png)


- 构造器注入
	- 简单数据类型![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53f07de1086d54be18e7e5f051078c05.png)


	- 引用数据类型![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ecfdc627b91cd54460eb26f396c246bd.png)


- 依赖注入的方式选择上
    - 建议使用setter注入
    - 第三方技术根据情况选择
### 5.3 自动配置

前面花了大量的时间把Spring的注入去学习了下，总结起来就两个字`麻烦`。

- 问:麻烦在哪?
    - 答:配置文件的编写配置上。
- 问:有更简单方式么?
    - 答:有，自动配置

所以什么是自动配置以及如何实现自动配置，就是接下来要学习的内容

#### 5.3.1 什么是依赖自动装配?

IOC容器根据bean所依赖的资源在容器中`自动查找并注入`到bean中的过程称为自动装配
#### 5.3.2 自动装配方式有哪些？

- 按类型（常用）
- 按名称
- 按构造方法
- 不启用自动装配
#### 5.3.3 环境准备

- 修改BookDao、BookDaoImpl、BookService和BookServiceImpl类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
      
    private String databaseName;  
    private int connectionNum;  
      
    public void save() {  
        System.out.println("book dao save ...");  
    }  
}

public interface BookService {  
    public void save();  
}

public class BookServiceImpl implements BookService{  
    private BookDao bookDao;  
  
    public void setBookDao(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

- 配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl">  
        <property name="bookDao" ref="bookDao"/>  
    </bean>  
</beans>
```

- App运行类
```java
public class App {  
    public static void main( String[] args ) {  
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookService bookService = (BookService) ctx.getBean("bookService");  
        bookService.save();  
    }  
}
```

#### 5.3.4 完成自动装配的配置

自动装配只需要修改applicationContext.xml配置文件即可:

1. 将`<property>`标签删除
2. 在`<bean>`标签中添加autowire属性

- 首先来实现按照类型注入的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
<!--    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>-->  
<!--    既然是按类型注入了，那么id写不写都无所谓了-->  
    <bean class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl" autowire="byType"/>  
</beans>
```

- 运行程序，结果如下，说明已经成功注入了![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/72845eb3dfe31ccac535091da95819fd.png)


>注意事项：
>	需要注入属性的类中对应属性的`setter`方法不能省略
>	
>	被注入的对象必须要被Spring的IOC容器管理
>	
>	按照类型在Spring的IOC容器中如果找到多个对象，会报`NoUniqueBeanDefinitionException`

- 当一个类型在IOC中有多个对象，还想要注入成功，这个时候就需要按照名称注入，配置方式如下
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <!--这里就有两个同一类型的bean，但是id不一样-->  
    <bean id="bookDao1" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookDao2" class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean class="com.blog.dao.impl.BookDaoImpl"/>  
    <bean id="bookService" class="com.blog.service.impl.BookServiceImpl" autowire="byName"/>  
</beans>
```

- 同时修改BookServiceImpl类汇总的`setBookDao`方法，将其重命名为`setBookDao1`
```java
public class BookServiceImpl implements BookService{  
    private BookDao bookDao;  
  
    public void setBookDao1(BookDao bookDao) {  
        this.bookDao = bookDao;  
    }  
  
    public void save() {  
        System.out.println("book service save ...");  
        bookDao.save();  
    }  
}
```

- 运行程序，结果如下，说明已经成功注入了![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4057c6a1dc14d869e7867f28dec9b8f2.png)


- 疑惑：为什么刚刚修改的是setBookDao的方法名，而不是将bookDao属性修改为bookDao1呢？按照名称注入中的名称指的是什么?
- 解惑：
    - 因为bookDao是private修饰的，外部类无法直接访问
    - 所以外部类只能通过属性的set方法进行访问
    - 对外部类来说，setBookDao方法名，去掉set后首字母小写是其属性名
        - 为什么是去掉set首字母小写?
        - 这个规则是set方法生成的`默认规则`，set方法的生成是把属性名首字母大写前面加set形成的方法名
    - 所以按照名称注入，其实是和对应的set方法有关，但是如果按照标准起名称，属性名和set对应的名是一致的

#### 5.3.5 小结

- 如果按照名称去找对应的bean对象，找不到则注入Null
- 当某一个类型在IOC容器中有多个对象，`按照名称注入只找其指定名称对应的bean对象，不会报错`
- 两种方式介绍完后，以后用的更多的是`按照类型`注入。
- 最后对于依赖注入，需要注意一些其他的配置特征:
    1. 自动装配用于引用类型依赖注入，`不能对简单类型进行操作`
    2. 使用按类型装配时（byType）`必须保障容器中相同类型的bean唯一，推荐使用`
    3. 使用按名称装配时（byName）`必须保障容器中具有指定名称的bean`，因变量名与配置耦合，`不推荐使用`
    4. `自动装配优先级低于setter注入与构造器注入`，同时出现时自动装配配置失效

### 5.4 集合注入

前面我们已经能完成引入数据类型和简单数据类型的注入，但是还有一种数据类型`集合`，集合中既可以装简单数据类型也可以装引用数据类型，对于集合，在Spring中该如何注入呢?

先来回顾下，常见的集合类型有哪些?

- 数组
- List
- Set
- Map
- Properties

针对不同的集合类型，该如何实现注入呢?接着往下看
#### 5.4.1 环境准备

- 修改BookDaoImpl类
```java
public interface BookDao {  
    public void save();  
}

public class BookDaoImpl implements BookDao {  
  
    private int[] array;  
  
    private List<String> list;  
  
    private Set<String> set;  
  
    private Map<String,String> map;  
  
    private Properties properties;  
  
    public void save() {  
        System.out.println("book dao save ...");  
  
        System.out.println("遍历数组:" + Arrays.toString(array));  
  
        System.out.println("遍历List" + list);  
  
        System.out.println("遍历Set" + set);  
  
        System.out.println("遍历Map" + map);  
  
        System.out.println("遍历Properties" + properties);  
    }  
  
    public void setArray(int[] array) {  
        this.array = array;  
    }  
  
    public void setList(List<String> list) {  
        this.list = list;  
    }  
  
    public void setSet(Set<String> set) {  
        this.set = set;  
    }  
  
    public void setMap(Map<String, String> map) {  
        this.map = map;  
    }  
  
    public void setProperties(Properties properties) {  
        this.properties = properties;  
    }  
}
```

- 修改配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="bookDao" class="com.blog.dao.impl.BookDaoImpl"/>  
</beans>
```

- 修改App运行时类
```java
public class App {  
    public static void main( String[] args ) {  
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");  
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");  
        bookDao.save();  
    }  
}
```

- 准备工作完毕，接下来，在上面这个环境中来完成`集合注入`的学习，下面所有的配置方式，都是在bookDao的bean标签中使用`<property>`进行注入
#### 5.4.2 注入数组类型

```xml
<property name="array">  
    <array>  
        <value>100</value>  
        <value>200</value>  
        <value>300</value>  
    </array>  
</property>
```
#### 5.4.3 注入List类型

```xml
<property name="list">  
    <list>  
        <value>张三</value>  
        <value>ABC</value>  
        <value>123</value>  
    </list>  
</property>
```
#### 5.4.4 注入Set类型

```xml
<property name="set">  
    <set>  
        <value>100</value>  
        <value>200</value>  
        <value>ABC</value>  
        <value>ABC</value>  
    </set>  
</property>
```
#### 5.4.5 注入Map类型

```xml
<property name="map">  
    <map>  
        <entry key="探路者" value="马文"/>  
        <entry key="次元游记兵" value="恶灵"/>  
        <entry key="易位窃贼" value="罗芭"/>  
    </map>  
</property>
```
#### 5.4.6 注入Properties类型

```xml
<property name="properties">  
    <props>  
        <prop key="暴雷">沃尔特·菲茨罗伊</prop>  
        <prop key="寻血猎犬">布洛特·亨德尔</prop>  
        <prop key="命脉">阿杰·切</prop>  
    </props>  
</property>
```

- 配置完成之后，运行查看结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/515b23a210fb34871ddcc88bd7e36b38.png)


- 说明：

	- property标签表示setter方式注入，构造方式注入constructor-arg标签内部也可以写`<array>`、`<list>`、`<set>`、`<map>`、`<props>`标签

	- List的底层也是通过数组实现的，所以`<list>`和`<array>`标签是可以混用

	- 集合中要添加引用类型，只需要把`<value>`标签改成`<ref>`标签，这种方式用的比较少