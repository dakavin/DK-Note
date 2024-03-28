# 本文摘要：

- 理解并实现分模块开发
- 能够使用聚合工程快速构建项目
- 能够使用继承简化项目配置
- 能够根据需求配置生成、开发、测试环境，并在各个环境间切换运行
- 了解Maven的私服
# 一、分模块开发

## 1、分模块开发设计

1. 按照功能拆分
    - 我们之前的项目都是在一个模块中，比如上篇文章的SSM整合，虽然这样做，功能也都实现了，但是也存在一些问题，我们拿银行的项目举例，来聊聊这件事
        - 网络没有那么发达的时候，我们需要到银行柜台或者取款机进行业务操作
        - 随着互联网的发展，我们有了电脑之后，就可以在网页上登录银行网站使用U盾进行业务操作
        - 再来就是随着智能手机的普及，我们只需要用手机登录App就可以进行业务操作
    - 上面三个场景出现的时间轴是不同的，如果非要把三个场景的模块代码放到同一个项目，那么当其中某一个模块代码出现问题，就会导致整个项目都无法正常启动，从而导致银行的多个业务都无法正常办理，所以我们会`按照功能`将项目进行拆分

2. 按照模块拆分
    - 比如电商项目中，有订单和商品两个模块，订单中需要包含商品的详细信息，所以需要商品的模型类，商品模块也会用到商品的模型类，这个时候如果两个模块中都写模型类，就会出现重复代码，后期维护的成本就比较高。我们就想能不能把它们公共的部分抽取成一个独立的模块，其他模块想要使用就可以像`导入第三方依赖的坐标`一样来使用我们自己抽取的模块，这样就解决了代码重复的问题，这种拆分方式就是我们所说的`按照模块`拆分

- 之前的项目包结构如下
    - `com.blog.config`目录存放的是相关的配置类
    - `com.blog.controller`编写的是Controller类
    - `com.blog.dao`存放的是Dao接口，因为使用的是Mapper接口代理方式，所以没有实现类包
    - `com.blog.service`存的是Service接口，`com.blog.service.impl`存放的是Service实现类
- 那我们现在就可以把这些包按照功能拆分成若干个子模块，方便模块之间的相互调用，接口供销
- 这样的话，项目中的每一层都可以单独维护，也可以很方便的被别人使用。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212551101.png)



那么关于分模块开发的意义，我们就讲完了，说了这么多好处，那么该如何实现呢?
## 2、分模块开发实现

前面我们已经完成了SSM整合，接下来，我们就基于SSM整合的项目来实现对项目的拆分。
### 2.1 环境准备

复制一份之前的ssm项目，重命名为`maven_01_ssm`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212558137.png)


### 2.2 抽取domain层

- `步骤一：`创建新模块
    - 创建一个名为`maven_02_pojo`的maven项目
- `步骤二：`项目中创建domain包
    - 在`maven_02_pojo`中创建`com.blog.domain`包，并将`maven_01_ssm`的Book类拷贝到该包中
- `步骤三：`删除原项目中的`domain`包
    - 删除后，`maven_01_ssm`项目中用到`Book`的类中都会爆红
    - 要想解决上述问题，我们需要在`maven_01_ssm`中添加`maven_02_pojo`的依赖。
- `步骤四：`
    - 在`maven_01_ssm`项目的pom.xml添加`maven_02_pojo`的依赖
```xml
<dependency>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_02_pojo</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```
- 因为添加了依赖，所以在`maven_01_ssm`中就已经能找到Book类，所以刚才的爆红提示就会消失。

- `步骤五：`编译`maven_01_ssm`项目(控制台会报错)![[Pasted image 20230831214409.png]]
- 意思就是找不到`maven_02_pojo`这个jar包
	- 为啥找不到呢？
		- 原因是Maven会从本地仓库找对应的jar包，但是本地仓库又不存在该jar包所以会报错。
		- 在IDEA中是有`maven_02_pojo`这个项目，所以我们只需要将`maven_02_pojo`项目安装到本地仓库即可。

- `步骤六：`将项目安装本地仓库
    - 将需要被依赖的项目`maven_02_pojo`，使用maven的`install`命令，把其安装到Maven的本地仓库中
    - 之后再次执行`maven_01_ssm`的`compile`的命令后，就已经能够成功编译。
### 2.3 抽取dao层

- `步骤一：`创建新模块
    - 创建一个名为`maven_03_dao`的maven项目
- `步骤二：`项目中创建`dao`包
    - 在`maven_03_dao`项目中创建`com.blog.dao`包，并将`maven_01_ssm`中BookDao类拷贝到该包中
    - 在`maven_03_dao`中会有如下几个问题需要解决下
        - 项目`maven_03_dao`的BookDao接口中Book类找不到报错
            - 解决方案在`maven_03_dao`项目的pom.xml中添加`maven_02_pojo`项目
```xml
<dependency>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_02_pojo</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

- 项目`maven_03_dao`的BookDao接口中，Mybatis的增删改查注解报错

- 解决方案在`maven_03_dao`项目的pom.xml中添加`mybatis`的相关依赖
```xml
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
```

- 最后记得使用maven的`install`命令，把其安装到Maven的本地仓库中

- `步骤三：`删除原项目中的`dao`包
	- 删除Dao包以后，因为`maven_01_ssm`中的BookServiceImpl类中有使用到Dao的内容，所以需要在`maven_01_ssm`的pom.xml添加`maven_03_dao`的依赖
```xml
<dependency>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_03_dao</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

- `步骤四：`运行测试
    - 启动Tomcat服务器，访问`http://localhost:8080/pages/books.html`
    - 将抽取后的项目进行运行，测试之前的增删改查功能依然能够使用。
### 2.4 小结

对于项目的拆分，大致会有如下几个步骤

1. 创建Maven模块
2. 书写模块代码
    - 分模块开发需要先针对模块功能进行设计，再进行编码。不会先将工程开发完毕，然后进行拆分。拆分方式可以按照功能拆也可以按照模块拆。
3. 通过maven指令安装模块到本地仓库(install 指令)
    - 由于maven指令只能安装到自己电脑的仓库里，那么团队内部开发需要发布模块功能，需要到团队内部可共享的仓库中(私服)，私服我们后面会讲解。
# 二、依赖管理

我们现在已经能把项目拆分成一个个独立的模块，当在其他项目中想要使用独立出来的这些模块，只需要在其pom.xml使用`<dependency>`标签来进行jar包的引入即可。  
`<dependency>`其实就是依赖，关于依赖管理里面都涉及哪些内容，我们就一个个来学习下:

- 依赖传递
- 可选依赖
- 排除依赖

我们先来说说什么是依赖:

- 依赖指当前项目运行所需的jar一个项目可以设置多个依赖。
- 格式为:
```xml
<!--设置当前项目所依赖的所有jar-->  
<dependencies>  
    <!--设置具体的依赖-->  
    <dependency>  
        <!--依赖所属群组id-->  
        <groupId>org.springframework</groupId>  
        <!--依赖所属项目id-->  
        <artifactId>spring-webmvc</artifactId>  
        <!--依赖版本号-->  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
</dependencies>
```
## 1、依赖传递与冲突问题

首先我们要明白一点：依赖是具有传递性的![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212611002.png)


说明：A代表自己的项目；B，C，D，E，F，G代表的是项目所依赖的jar包；D1和D2、E1和E2代表是相同jar包的不同版本

1. A依赖了B和C，B和C有分别依赖了其他jar包，所以在A项目中就可以使用上面所有jar包，这就是所说的依赖传递
2. 依赖传递有直接依赖和间接依赖
    - 相对于A来说，A直接依赖B和C，间接依赖了D1，E1，G，F，D2和E2
    - 相对于B来说，B直接依赖了D1和E1，间接依赖了G
    - 直接依赖和间接依赖是一个相对的概念
3. 因为有依赖传递的存在，就会导致jar包在依赖的过程中出现冲突问题，具体什么是冲突？Maven是如何解决冲突的？

这里所说的`依赖冲突`是指项目依赖的某一个jar包，有多个不同的版本，因而造成类包版本冲突。

- `情况一:` 在maven_01_ssm的pom.xml中添加两个不同版本的Junit依赖：
```xml
<dependencies>  
    <dependency>  
      <groupId>junit</groupId>  
      <artifactId>junit</artifactId>  
      <version>4.12</version>  
      <scope>test</scope>  
    </dependency>  
  
    <dependency>  
      <groupId>junit</groupId>  
      <artifactId>junit</artifactId>  
      <version>4.11</version>  
      <scope>test</scope>  
    </dependency>  
</dependencies>
```

- 调换位置，刷新maven面板，我们会发现，maven的dependencies面板上总是显示使用的是后加载的jar包  
    于是我们得出一个结论：
- `特殊优先：当同级配置了相同资源的不同版本，后配置的覆盖先配置的`。
    
- `情况二:`路径优先：当依赖中出现相同的资源时，层级越深，优先级越低，层级越浅，优先级越高
    
    - A通过B间接依赖到E1
    - A通过C间接依赖到E2
    - A就会间接依赖到E1和E2，Maven会按照层级来选择，E1是2度，E2是3度，所以最终会选择E1
- `情况三:`声明优先：当资源在相同层级被依赖时，配置顺序靠前的覆盖配置顺序靠后的
    
    - A通过B间接依赖到D1
    - A通过C间接依赖到D2
    - D1和D2都是两度，这个时候就不能按照层级来选择，需要按照声明来，谁先声明用谁，也就是说B在C之前声明，这个时候使用的是D1，反之则为D2

但是对于上面的结果，我们也不用刻意去记，一切以maven的dependencies面板上显示的为准
## 2、可选依赖和排除依赖

依赖传递介绍完以后，我们来思考一个问题，假如

- `maven_01_ssm` 依赖了 `maven_03_dao`
- `maven_03_dao` 依赖了 `maven_02_pojo`
- 因为现在有依赖传递，所以`maven_01_ssm`能够使用到`maven_02_pojo`的内容
- 如果说现在不想让`maven_01_ssm`依赖到`maven_02_pojo`，有哪些解决方案?
    
    说明：在真实使用的过程中，`maven_01_ssm`中是需要用到`maven_02_pojo`的，我们这里只是用这个例子描述我们的需求。因为有时候，`maven_03_dao`出于某些因素的考虑，就是不想让别人使用自己所依赖的`maven_02_pojo`。
    

- `方案一：`可选依赖
    
    - 可选依赖指对外隐藏当前所依赖的资源—-`不透明`
    - 在`maven_03_dao`的pom.xml，在引入`maven_02_pojo`的时候，添加`optional`
```xml
<dependency>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_02_pojo</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--可选依赖是隐藏当前工程所依赖的资源，隐藏后对应资源将不具有依赖传递-->  
    <optional>true</optional>  
</dependency>
```

- `方案二：`排除依赖

	- 排除依赖指主动断开依赖的资源，被排除的资源无需指定版本—-`不需要`
	- 前面我们已经通过可选依赖实现了阻断`maven_02_pojo`的依赖传递，对于排除依赖，则指的是已经有依赖的事实，也就是说`maven_01_ssm`项目中已经通过依赖传递用到了`maven_02_pojo`，此时我们需要做的是将其进行排除，所以接下来需要修改`maven_01_ssm`的pom.xml
```xml
<dependency>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_03_dao</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--排除依赖是隐藏当前资源对应的依赖关系-->  
    <exclusions>  
        <!--这里可以排除多个依赖，只要你有需求-->  
        <exclusion>  
            <groupId>com.blog</groupId>  
            <artifactId>maven_02_pojo</artifactId>  
        </exclusion>  
    </exclusions>  
</dependency>
```

介绍完这两种方式后，简单来梳理下，就是

- `A依赖B，B依赖C`，`C`通过依赖传递会被`A`使用到，现在要想办法让`A`不去依赖`C`
- 可选依赖是在B上设置`<optional>`，`A`不知道有`C`的存在，
- 排除依赖是在A上设置`<exclusions>`，`A`知道有`C`的存在，主动将其排除掉。
# 三、聚合与继承

我们的项目已经从以前的单模块，变成了现在的多模块开发。项目一旦变成了多模块开发以后，就会引发一些问题，在这一节中我们主要会学习两个内容`聚合`和`继承`，用这两个知识来解决下分模块后的一些问题。
## 1、聚合

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212619169.png)



- 分模块开发后，需要将这四个项目都安装到本地仓库，目前我们只能通过项目Maven面板的install来安装，并且需要安装四个，如果我们的项目足够多，那一个个install也挺麻烦的
- 如果四个项目都已经安装成功，当ssm_pojo发生变化后，我们就得将ssm_pojo重新安装到maven仓库，但是为了确保我们对ssm_pojo的修改不会影响到其他模块（比如我们将pojo类中的一个属性删除，如果其他模块调用了这个属性，那必然报错），我们需要对所有模块重新编译，看看有没有问题。然后还需要将所有模块再install一遍

项目少的话还好，但是如果项目多的话，一个个操作项目就容易出现漏掉或重复操作的问题，所以我们就像能不能抽取一个项目，把所有的项目管理起来，以后再想操作这些项目，做需要操作我们抽取的这个项目，这样就省事儿多了

这就要用到我们接下来讲的`聚合`了

- 所谓聚合：将多个模块组织成一个整体，同时进行项目构建的过程称为聚合
- 聚合工程：通常是一个不具有业务功能的`空`工程
- 作用：使用聚合工程可以将多个工程编组，通过对聚合工程的构建，实现对所包含的所有模块进行同步构建
    - 当工程中某个模块发生更新后，必须保障工程中与更新模块关联的模块同步更新，此时就可以使用聚合工程来解决批量模块同步构建的问题

具体实现步骤如下：

- `步骤一：`创建一个空的maven项目
- `步骤二：`将项目打包方式改为`pom`
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.blog</groupId>  
    <artifactId>maven_00_parent</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--设置打包方式-->  
    <packaging>pom</packaging>  
  
</project>
```

说明:项目的打包方式，我们接触到的有三种，分别是

- jar:默认情况，说明该项目为java项目
- war:说明该项目为web项目
- pom:说明该项目为聚合或继承(后面会讲)项目

- `步骤三：`pom.xml添加所要管理的项目
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.blog</groupId>  
    <artifactId>maven_00_parent</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--设置打包方式-->  
    <packaging>pom</packaging>  
  
    <!--设置管理的项目名称-->  
    <modules>  
        <module>../maven_01_ssm</module>  
        <module>../maven_02_pojo</module>  
        <module>../maven_03_dao</module>  
    </modules>  
  
</project>
```

- `步骤四：`使用聚合统一管理项目  
	在maven面板上点击compile，会发现所有受管理的项目都会被执行编译，这就是聚合工程的作用![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212624276.png)


	说明：聚合工程管理的项目在进行运行的时候，会按照项目与项目之间的依赖关系来自动决定执行的顺序和配置的顺序无关。  
	虽然我们配置的顺序是`123`，但是执行的时候按照依赖关系编译是`231`

那么聚合的知识我们就讲解完了，最后总结一句话就是，聚合工程主要是用来管理项目。
## 2、继承

我们已经完成了使用聚合工程去管理项目，聚合工程进行某一个构建操作，其他被其管理的项目也会执行相同的构建操作。那么接下来，我们再来分析下，多模块开发存在的另外一个问题，`重复配置`的问题，我们先来看张图:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212629916.png)


- `spring-webmvc`、`spring-jdbc`在三个项目模块中都有出现，这样就出现了重复的内容
- `spring-test`只在ssm_crm和ssm_goods中出现，而在ssm_order中没有，这里是部分重复的内容
- 我们使用的spring版本目前是`5.2.10.RELEASE`，假如后期要想升级spring版本，所有跟Spring相关jar包都得被修改，涉及到的项目越多，维护成本越高

面对上面这些问题，我们就得用到接下来要学习的`继承`

- 所谓继承：描述的是两个工程间的关系，与java中的继承类似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承
- 作用：
    - `简化配置`
    - `减少版本冲突`

接下来，我们到程序中去看看继承该如何实现

- `步骤一：`创建一个空的Maven项目并将其打包方式设置为pom
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.blog</groupId>  
    <artifactId>maven_00_parent</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--设置打包方式-->  
    <packaging>pom</packaging>  
      
</project>
```

- `步骤二：`在子工程中设置其父工程
```xml
<!--配置当前工程继承自parent工程-->  
<parent>  
    <artifactId>maven_00_parent</artifactId>  
    <groupId>com.blog</groupId>  
    <version>1.0-SNAPSHOT</version>  
    <!--配置父项目的pom.xml路径-->  
    <relativePath>../maven_00_parent/pom.xml</relativePath>  
</parent>
```

- `步骤三：`优化子项目共有依赖导入问题

1. 将子项目共同使用的jar包都抽取出来，维护在父项目的pom.xml中
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.blog</groupId>  
    <artifactId>maven_00_parent</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <!--设置打包方式-->  
    <packaging>pom</packaging>  
  
    <!--设置管理的项目名称-->  
    <modules>  
        <module>../maven_01_ssm</module>  
        <module>../maven_02_pojo</module>  
        <module>../maven_03_dao</module>  
    </modules>  
  
    <dependencies>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-webmvc</artifactId>  
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
            <groupId>javax.servlet</groupId>  
            <artifactId>javax.servlet-api</artifactId>  
            <version>3.1.0</version>  
            <scope>provided</scope>  
        </dependency>  
  
        <dependency>  
            <groupId>com.fasterxml.jackson.core</groupId>  
            <artifactId>jackson-databind</artifactId>  
            <version>2.9.0</version>  
        </dependency>  
  
    </dependencies>  
</project>
```

2. 删除子项目中已经被抽取到父项目的pom.xml中的jar包
	- 删除完后，你会发现父项目中有依赖对应的jar包，子项目虽然已经将重复的依赖删除掉了，但是刷新的时候，子项目中所需要的jar包依然存在。
	- 当项目的`<parent>`标签被移除掉，会发现多出来的jar包依赖也会随之消失。

3. 在父项目中修改jar包的版本，刷新后，子项目中的jar包版本也随之变化

- 那么现在我们就可以解决了刚才提到的第一个问题，将子项目中的公共jar包抽取到父工程中进行统一添加依赖，这样做的可以简化配置，并且当父工程中所依赖的jar包版本发生变化，所有子项目中对应的jar包版本也会跟着更新。
    
- `步骤四：`优化子项目依赖版本问题  
    如果把所有用到的jar包都管理在父项目的pom.xml，看上去更简单些，但是这样就会导致有很多项目引入了过多自己不需要的jar包。如上面看到的这张图:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212635623.png)


	如果把所有的依赖都放在了父工程中进行统一维护，就会导致ssm_order项目中多引入了`spring-test`的jar包，如果这样的jar包过多的话，对于ssm_order来说也是一种”负担”。
    

那针对于这种部分项目有的jar包，我们该如何管理优化呢?

1. 在父工程中的pom.xml中定义依赖管理
```xml
<!--定义依赖管理-->  
<dependencyManagement>  
    <dependencies>  
        <dependency>  
            <groupId>junit</groupId>  
            <artifactId>junit</artifactId>  
            <version>4.12</version>  
            <scope>test</scope>  
        </dependency>  
    </dependencies>  
</dependencyManagement>
```

2. 将maven_01_ssm的pom.xml中的junit依赖删除掉，刷新Maven  
    刷新后，在maven_01_ssm项目中找不到junit依赖，所以我们得出一个结论  
    `<dependencyManagement>`标签不真正引入jar包，而是配置可供子项目选择的jar包依赖  
    子项目要想使用它所提供的这些jar包，需要自己添加依赖，并且不需要指定`<version>`

3. 在在maven_01_ssm的pom.xml添加junit的依赖
```xml
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <scope>test</scope>  
</dependency>
```

注意：这里就不需要添加版本了，这样做的好处就是当父工程`dependencyManagement`标签中的版本发生变化后，子项目中的依赖版本也会跟着发生变化

4. 在maven_03_dao的pom.xml添加junit的依赖
```xml
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <scope>test</scope>  
</dependency>
```

这个时候，maven_01_ssm和maven_03_dao这两个项目中的junit版本就会跟随着父项目中的标签`dependencyManagement`中junit的版本发生变化而变化。不需要junit的项目就不需要添加对应的依赖即可（maven_02_pojo中就没添加）

至此继承就已经学习完了，总结来说，继承可以帮助做两件事

- 将所有项目公共的jar包依赖提取到父工程的pom.xml中，子项目就可以不用重复编写，简化开发
- 将所有项目的jar包配置到父工程的`dependencyManagement`标签下，实现版本管理，方便维护
    - `dependencyManagement`标签不真正引入jar包，只是管理jar包的版本
    - 子项目在引入的时候，只需要指定groupId和artifactId，不需要加version
    - 当`dependencyManagement`标签中jar包版本发生变化，所有子项目中有用到该jar包的地方对应的版本会自动随之更新

最后总结一句话就是，`父工程主要是用来快速配置依赖jar包和管理项目中所使用的资源`。

- `小结`
- 继承的实现步骤:
- 创建Maven模块，设置打包类型为pom
```xml
<packaging>pom</packaging>
```
- 在父工程的pom文件中配置依赖关系(子工程将沿用父工程中的依赖关系),一般只抽取子项目中公有的jar包
```xml
<dependencies>  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-webmvc</artifactId>  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
    ...  
</dependencies>
```
- 在父工程中配置子工程中可选的依赖关系
```xml
<dependencyManagement>  
    <dependencies>  
        <dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>druid</artifactId>  
            <version>1.1.16</version>  
        </dependency>  
    </dependencies>  
    ...  
</dependencyManagement>
```
- 在子工程中配置当前工程所继承的父工程
```xml
<!--定义该工程的父工程-->  
<parent>  
    <groupId>com.blog</groupId>  
    <artifactId>maven_01_parent</artifactId>  
    <version>1.0-RELEASE</version>  
    <!--填写父工程的pom文件,可以不写-->  
    <relativePath>../maven_01_parent/pom.xml</relativePath>  
</parent>
```
- 在子工程中配置使用父工程中可选依赖的坐标
```xml
<dependencies>  
    <dependency>  
        <groupId>com.alibaba</groupId>  
        <artifactId>druid</artifactId>  
    </dependency>  
</dependencies>
```

注意事项:

1. 子工程中使用父工程中的可选依赖时，仅需要提供群组id和项目id，无需提供版本，版本由父工程统一提供，避免版本冲突
2. 子工程中还可以定义父工程中没有定义的依赖关系,只不过不能被父工程进行版本统一管理。
## 3、区别

聚合与继承分别的作用:

- `聚合用于快速构建项目，对项目进行管理`
- `继承用于快速配置和管理子项目中所使用jar包的版本`

聚合和继承的相同点:

- 聚合与继承的pom.xml文件打包方式均为pom，可以将两种关系制作到同一个pom文件中
- 聚合与继承均属于设计型模块，并无实际的模块内容

聚合和继承的不同点:

- 聚合是在当前模块中配置关系，聚合可以感知到参与聚合的模块有哪些
- 继承是在子模块中配置关系，父模块无法感知哪些子模块继承了自己
# 四、属性

在这一小节，我们将学习两个内容，分别是

- 属性
- 版本管理

属性中会继续解决分模块开发项目存在的问题，版本管理主要是认识下当前主流的版本定义方式。
## 1、属性

### 1.1 问题分析

前面我们已经在父工程中的dependencyManagement标签中对项目中所使用的jar包版本进行了统一的管理，但是如果在标签中有如下的内容:
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
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
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.5.6</version>  
</dependency>
```

如果我们现在想更新Spring的版本，就会发现我们依然需要更新多个jar包的版本，这样的话还是有可能出现漏改导致程序出问题，而且改起来也是比较麻烦。  
问题清楚后，我们需要解决的话，就可以参考咱们java基础所学习的变量，声明一个变量，在其他地方使用该变量，当变量的值发生变化后，所有使用变量的地方也会跟着变化  
例如
```java
String spring_version = "5.2.10.RELEASE";
```

然后将依赖的版本号替换成`spring_version`
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>spring_version</version>  
</dependency>
```
### 1.2 解决步骤

`步骤一：`父工程中定义属性
```xml
<properties>  
        <spring.version>5.2.10.RELEASE</spring.version>  
        <mybatis.version>3.5.6</mybatis.version>  
</properties>
```

- `步骤二：`修改依赖的version
```xml
 <dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>${spring.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>${spring.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>${spring.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>${mybatis.version}</version>  
</dependency>
```

此时，我们只需要更新父工程中properties标签中所维护的jar包版本，所有子项目中的版本也就跟着更新。当然除了将spring相关版本进行维护，我们可以将其他的jar包版本也进行抽取，这样就可以对项目中所有jar包的版本进行统一维护

说明：使用`properties`标签来定义属性，在`properties`标签内自定义标签名当做属性名，自定义标签内的值即为属性值  
例如：`<spring.version>5.2.10.RELEASE</spring.version>`，属性名为`spring.version`，属性值为`5.2.10.RELEASE`  
在其他地方引用变量时用`${变量名}`
## 2、配置文件加载属性

Maven中的属性我们已经介绍过了，现在也已经能够通过Maven来集中管理Maven中依赖jar包的版本。但是又有新的需求，就是想让Maven对于属性的管理范围能更大些，比如我们之前项目中的`jdbc.properties`，这个配置文件中的属性，能不能也来让Maven进行管理呢?

答案是肯定的，具体的实现步骤如下

- `步骤一：`父工程定义属性
```xml
<properties>  
    <spring.version>5.2.10.RELEASE</spring.version>  
    <mybatis.version>3.5.6</mybatis.version>  
    <jdbc.driver>com.mysql.jdbc.Driver</jdbc.driver>  
    <jdbc.url>jdbc:mysql://localhost:13306/ssm_db?useSSL=false</jdbc.url>  
    <jdbc.username>root</jdbc.username>  
    <jdbc.password>PASSWORD</jdbc.password>  
</properties>
```

- `步骤二：`jdbc.properties文件中引用属性
```properties
jdbc.driver=${jdbc.driver}  
jdbc.url=${jdbc.url}  
jdbc.username=${jdbc.username}  
jdbc.password=${jdbc.password}
```

`步骤三：`设置maven过滤文件范围  
	直接在properties中引用属性，看起来怪怪的，properties怎么能直接用到maven中配置的属性呢？  
	所以我们还需要来配置一下，让`maven_01_ssm/src/main/resources`目录下的`jdbc.properties`文件可以解析`${}`
```xml
<build>  
    <resources>  
        <!--设置资源目录-->  
        <resource>  
            <directory>../maven_01_ssm/src/main/resources</directory>  
            <!--设置能够解析${}，默认是false -->  
            <filtering>true</filtering>  
        </resource>  
    </resources>  
</build>
```

- `步骤四：`测试是否生效  
    测试的时候，只需要将maven_01_ssm项目进行打包，然后在本地仓库观察打包结果中最终生成的内容是否为Maven中配置的内容。
    
- `存在的问题`  
    如果不只是`maven_01_ssm`项目需要有属性被父工程管理，如果还有多个项目需要配置，该如何实现呢?

- `方式一`
```xml
<build>  
    <resources>  
        <!--设置资源目录，并设置能够解析${}-->  
        <resource>  
            <directory>../maven_01_ssm/src/main/resources</directory>  
            <filtering>true</filtering>  
        </resource>  
        <resource>  
            <directory>../maven_02_pojo/src/main/resources</directory>  
            <filtering>true</filtering>  
        </resource>  
        ...  
    </resources>  
</build>
```
可以一个一个配，但是项目足够多的话，这样还是比较繁琐的

- `方式二`
```xml
<build>  
    <resources>  
        <!--  
            ${project.basedir}: 当前项目所在目录,子项目继承了父项目，  
            相当于所有的子项目都添加了资源目录的过滤  
        -->  
        <resource>  
            <directory>${project.basedir}/src/main/resources</directory>  
            <filtering>true</filtering>  
        </resource>  
    </resources>  
</build>
```
说明：如果打包过程中出现错误`Error assembling WAR: webxml attribute is required`  
原因就是Maven发现你的项目为web项目，就会去找web项目的入口web.xml(配置文件配置的方式)，发现没有找到，就会报错。

- `解决方案1：`在maven_02_ssm项目的`src\main\webapp\WEB-INF\`添加一个web.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"  
         version="3.1">  
</web-app>
```

- `解决方案2:` 配置maven打包war时，忽略web.xml检查
```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-war-plugin</artifactId>  
            <version>3.2.3</version>  
            <configuration>  
                <failOnMissingWebXml>false</failOnMissingWebXml>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>
```

上面我们所使用的都是Maven的自定义属性，除了`${project.basedir}`,它属于Maven的内置系统属性。

在Maven中的属性分为:

- 自定义属性（常用）
- 内置属性
- Setting属性
- Java系统属性
- 环境变量属性

|属性分类|引用格式|示例|
|---|---|---|
|自定义属性|${自定义属性名}|${spring.vension}|
|内置属性|${内置属性名}|${basedir}、${version}|
|setting属性|${setting.属性名}|${settings.localRepository}|
|ava系统属性|${系统属性分类.系统属性名}|${user.home}|
|环境变量属性|${env.环境变量属性名}|${env.JAVA_HOME}|
## 3、版本管理

关于这个版本管理解决的问题是，在Maven创建项目和引用别人项目的时候，我们都看到过如下内容:
```xml
<groupId>com.blog</groupId>  
<artifactId>maven_00_parent</artifactId>  
<version>1.0-SNAPSHOT</version>  
  
<groupId>org.springframework</groupId>  
<artifactId>spring-jdbc</artifactId>  
<version>5.2.10.RELEASE</version>
```

这里面有两个单词，SNAPSHOT和RELEASE，它们所代表的含义是什么呢?

- `SNAPSHOT（快照版本）`
    - 项目开发过程中临时输出的版本，称为快照版本
    - 快照版本会随着开发的进展不断更新
- `RELEASE（发布版本）`
    - 项目开发到一定阶段里程碑后，向团队外部发布较为稳定的版本，这种版本所对应的构件文件是稳定的
    - 即便进行功能的后续开发，也不会改变当前发布版本内容，这种版本称为发布版本

除了上面的工程版本，我们还经常能看到一些发布版本：

- `alpha版：内测版`，bug多不稳定内部版本不断添加新功能
- `beta版：公测版`，不稳定(比alpha稳定些)，bug相对较多不断添加新功能，相信你更新QQ的时候，一定见过QQ的beta测试版
- `纯数字版`

对于这些版本，我们简单认识下即可
# 五、多环境配置与应用

这一小节，主要会讲两个内容，分别是`多环境开发`和`跳过测试`
## 1、多环境开发

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212644461.png)



- 我们平常都是在自己的开发环境进行开发
- 当开发完成后，需要把开发的功能部署到测试环境供测试人员进行测试使用
- 等测试人员测试通过后，我们会将项目部署到生成环境上线使用。
- 这个时候就有一个问题是，不同环境的配置是不相同的，如不可能让三个环境都用一个数据库，所以就会有三个数据库的url配置，
- 我们在项目中如何配置?
- 要想实现不同环境之间的配置切换又该如何来实现呢?

maven提供配置多种环境的设定，帮助开发者在使用过程中快速切换环境。具体实现步骤如下

- `步骤一：`父工程配置多个环境，并指定默认激活环境
```xml
<profiles>  
    <!--开发环境-->  
    <profile>  
        <id>env_dep</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>  
        </properties>  
        <!--设定是否为默认环境-->  
        <activation>  
            <activeByDefault>true</activeByDefault>  
        </activation>  
    </profile>  
    <!--生产环境-->  
    <profile>  
        <id>env_pro</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.2.2.2:3306/ssm_db</jdbc.url>  
        </properties>  
    </profile>  
    <!--测试环境-->  
    <profile>  
        <id>env_test</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.3.3.3:3306/ssm_db</jdbc.url>  
        </properties>  
    </profile>  
</profiles>
```

- `步骤二：`执行install查看env_dep环境是否生效  
    在你本地仓库找到打包的war包，看看jdbc.properties配置文件中的url是否为`jdbc:mysql://127.1.1.1:3306/ssm_db`

- `步骤三：`切换默认环境为生产环境
```xml
<profiles>  
    <!--开发环境-->  
    <profile>  
        <id>env_dep</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>  
        </properties>  
  
    </profile>  
    <!--生产环境-->  
    <profile>  
        <id>env_pro</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.2.2.2:3306/ssm_db</jdbc.url>  
        </properties>  
        <!--设定是否为默认环境-->  
        <activation>  
            <activeByDefault>true</activeByDefault>  
        </activation>  
    </profile>  
    <!--测试环境-->  
    <profile>  
        <id>env_test</id>  
        <properties>  
            <jdbc.url>jdbc:mysql://127.3.3.3:3306/ssm_db</jdbc.url>  
        </properties>  
    </profile>  
</profiles>
```

- `步骤四：`执行install并查看env_pro环境是否生效  
    查看到的结果为`jdbc:mysql://127.2.2.2:3306/ssm_db`  
    虽然已经能够实现不同环境的切换，但是每次切换都是需要手动修改，如何来实现在不改变代码的前提下完成环境的切换呢?

- `步骤五：`命令行实现环境切换  
    在命令后加上环境id`mvn install -P env_test`

- `步骤六：`执行安装并查看env_test环境是否生效  ![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212653572.png)


    查看到的结果为`jdbc:mysql://127.3.3.3:3306/ssm_db`

所以总结来说，对于多环境切换只需要两步即可:

- 父工程中定义多环境
```xml
<profiles>  
<profile>  
    <id>环境名称</id>  
        <properties>  
        <key>value</key>  
        </properties>  
        <activation>  
        <activeByDefault>true</activeByDefault>  
        </activation>  
    </profile>  
    ...  
</profiles>
```

- 使用多环境(构建过程)
```plaintext
mvn 指令 -P 环境定义ID
```
## 2、跳过测试

前面在执行`install`指令的时候，Maven都会按照顺序从上往下依次执行，每次都会执行`test`,

对于`test`来说有它存在的意义，

- 可以确保每次打包或者安装的时候，程序的正确性，假如测试已经通过在我们没有修改程序的前提下再次执行打包或安装命令，由于顺序执行，测试会被再次执行，就有点耗费时间了。
- 功能开发过程中有部分模块还没有开发完毕，测试无法通过，但是想要把其中某一部分进行快速打包，此时`由于测试环境失败就会导致打包失败`。

遇到上面这些情况的时候，我们就想`跳过测试执行下面的构建命令`，具体实现方式有很多：

- `方式一：`IDEA工具实现跳过测试  
    IDEA的maven面板上有一个按钮，点击之后可以跳过测试，不过此种方式会跳过所有的测试，如果我们想更精细的控制哪些跳过，哪些不跳过，那么就需要使用配置插件的方式来完成了![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212656689.png)



- `方式二：`配置插件实现跳过测试  
    在父工程中的pom.xml中添加测试插件配置
```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-surefire-plugin</artifactId>  
            <version>2.12.4</version>  
            <configuration>  
                <skipTests>false</skipTests>  
                <!--排除掉不参与测试的内容-->  
                <excludes>  
                    <exclude>**/BookServiceTest.java</exclude>  
                </excludes>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>
```
- `skipTests：`如果为true，则跳过所有测试，如果为false，则不跳过测试  
    `excludes：`哪些测试类不参与测试，即排除，针对skipTests为false来设置的  
    `includes：`哪些测试类要参与测试，即包含,针对skipTests为true来设置的
    
- `方式三：`命令行跳过测试  
    使用Maven的命令行，`mvn 指令 -D skipTests`

注意事项:

- 执行的项目构建指令必须包含测试生命周期，否则无效果。例如执行compile生命周期，不经过test生命周期。
- 该命令可以不借助IDEA，直接使用cmd命令行进行跳过测试，需要注意的是cmd要在pom.xml所在目录下进行执行。

# 六、私服

## 1、私服简介

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212701054.png)



1. 张三负责ssm_crm的开发，自己写了一个ssm_pojo模块，要想使用直接将ssm_pojo安装到本地仓库即可
2. 李四负责ssm_order的开发，需要用到张三所写的ssm_pojo模块，这个时候如何将张三写的ssm_pojo模块交给李四呢?
3. 如果直接拷贝，那么团队之间的jar包管理会非常混乱而且容器出错，这个时候我们就想能不能将写好的项目上传到中央仓库，谁想用就直接联网下载即可
4. Maven的中央仓库不允许私人上传自己的jar包,那么我们就得换种思路，自己搭建一个类似于中央仓库的东西，把自己的内容上传上去，其他人就可以从上面下载jar包使用
5. 这个类似于中央仓库的东西就是我们接下来要学习的私服

所以到这就有两个概念，一个是私服，一个是中央仓库

- `私服:`公司内部搭建的用于存储Maven资源的服务器
- `远程仓库:`Maven开发团队维护的用于存储Maven资源的服务器

所以说:

- `私服是一台独立的服务器，用于解决团队内部的资源共享与资源同步问题`

搭建Maven私服的方式有很多，我们来介绍其中一种使用量比较大的实现方式:

- Nexus
    - Sonatype公司的一款maven私服产品
    - 下载地址：[https://help.sonatype.com/repomanager3/download](https://help.sonatype.com/repomanager3/download)

关于nexus的下载安装和初次登录，这里就不介绍了
- 下载解压两个文件夹即可![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212709941.png)


- 启动Nexus，进入nexus中的bin目录下，打开cmd![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212711636.png)


- 出现如下文字显示，即为启动成功，打开浏览器，输入如下地址![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212720596.png)


- 点击登录按钮，根据提示输入账户和密码![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212722880.png)


- 修改密码为abc123，并允许匿名访问![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212725375.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212731212.png)


- 总结![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212738539.png)


## 2、私服仓库分类

私服资源操作流程分析:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212755794.png)


1. 在没有私服的情况下，我们自己创建的服务都是安装在Maven的本地仓库中
2. 私服中也有仓库，我们要把自己的资源上传到私服，最终也是放在私服的仓库中
3. 其他人要想使用你所上传的资源，就需要从私服的仓库中获取
4. 当我们要使用的资源不是自己写的，是远程中央仓库有的第三方jar包，这个时候就需要从远程中央仓库下载，每个开发者都去远程中央仓库下速度比较慢(中央仓库服务器在国外)
5. 私服就再准备一个仓库，用来专门存储从远程中央仓库下载的第三方jar包，第一次访问没有就会去远程中央仓库下载，下次再访问就直接走私服下载
6. 前面在介绍版本管理的时候提到过有`SNAPSHOT`和`RELEASE`，如果把这两类的都放到同一个仓库，比较混乱，所以私服就把这两个种jar包放入不同的仓库
7. 上面我们已经介绍了有三种仓库，一种是存放`SNAPSHOT`的，一种是存放`RELEASE`还有一种是存放从远程仓库下载的第三方jar包，那么我们在获取资源的时候要从哪个仓库种获取呢?
8. `为了方便获取，我们将所有的仓库编成一个组，我们只需要访问仓库组去获取资源`

---
所有私服仓库总共分为三大类:

1. 宿主仓库hosted
    - `保存无法从中央仓库获取的资源`
    - `自主研发`
    - `第三方非开源项目`,比如Oracle,因为是付费产品，所以中央仓库没有
2. 代理仓库proxy
    - 代理远程仓库，通过nexus访问其他公共仓库，例如中央仓库
3. 仓库组group
    - 将若干个仓库组成一个群组，简化配置
    - 仓库组不能保存资源，属于设计型仓库

|仓库类别|英文名称|功能|关联操作|
|---|---|---|---|
|宿主仓库|hosted|保存自主研发+第三方资源|上传|
|代理仓库|proxy|代理连接中央仓库|下载|
|仓库组|group|为仓库编组简化下载操作|下载|
## 3、 本地仓库访问私服配置

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212801118.png)


- 我们通过IDEA将开发的模块上传到私服，中间是要经过本地Maven的
- 本地Maven需要知道私服的访问地址以及私服访问的用户名和密码
- 私服中的仓库很多，Maven最终要把资源上传到哪个仓库?
- Maven下载的时候，又需要携带用户名和密码到私服上找对应的仓库组进行下载，然后再给IDEA
- 上面所说的这些内容，我们需要在本地Maven的配置文件`settings.xml`中进行配置。

---

- `步骤一：`私服上配置仓库  
    新建两个仓库，type选hosted，version policy 一个选release，一个选snapshot![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212808214.png)


    
- `步骤二：`配置本地Maven对私服的访问权限![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212814075.png)


```xml
<server>  
    <!--id就是刚刚创建的仓库名-->  
    <id>blog-snapshot</id>  
    <username>admin</username>  
    <password>abc123</password>  
</server>  
<server>  
    <id>blog-release</id>  
    <username>admin</username>  
    <password>7d1f6527-cf26-4be3-974a-de177d75cb87</password>  
</server>
```

- `步骤三：`配置私服的访问路径
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212822298.png)


```xml
<mirror>  
		<!--配置仓库组的ID-->
	  <id>maven-public</id>  
	  <!--私服仓库组maven-public的访问路径--> 
	  <url>http://localhost:8081/repository/maven-public/</url>
	  <!--*代表所有内容都从私服获取-->
	  <mirrorOf>*</mirrorOf>          
</mirror>
```

最后记得将新创建的两个仓库加入到maven-public的成员中，至此本地仓库就能与私服进行交互了
## 4、私服资源的下载和上传

本地仓库与私服已经建立了连接，接下来我们就需要往私服上上传资源和下载资源，具体的实现步骤如下

- `步骤一：`配置工程上传私服的具体位置![[Pasted image 20230901021116.png]]
```xml
<distributionManagement>  
    <repository>  
        <id>blog-release</id>  
        <url>http://localhost:8081/repository/blog-release/</url>  
    </repository>  
    <snapshotRepository>  
        <id>blog-snapshot</id>  
        <url>http://localhost:8081/repository/blog-snapshot/</url>  
    </snapshotRepository>  
</distributionManagement>
```

- `步骤二：`发布资源到私服  
    maven面板中运行`deploy`，或者执行maven命令`mvn deploy`

注意:

- 要发布的项目都需要配置`distributionManagement`标签，要么在自己的pom.xml中配置，要么在其父项目中配置，然后子项目中继承父项目即可。
- 如果报401错误，尝试将maven的setting.xml文件复制到`C:\Users\username\.m2`目录下，然后在重新进行deploy

发布成功，在私服中就能看到了![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922212833396.png)



现在发布是在blog-snapshot仓库中，如果`想发布到blog-release仓库中就需要将项目pom.xml中的version修改成RELEASE即可`。
```xml
<groupId>com.blog</groupId>  
<artifactId>maven_00_parent</artifactId>  
<!--<version>1.0-SNAPSHOT</version>-->  
<version>1.0-RELEASE</version>
```

如果私服中没有对应的jar，会去中央仓库下载，速度很慢。可以配置让私服去阿里云中下载依赖。  
修改maven-central的Remote storage为`http://maven.aliyun.com/nexus/content/groups/public`

至此私服的搭建就已经完成，相对来说有点麻烦，但是步骤都比较固定，后期忘了就常回来看