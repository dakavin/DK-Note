---
文章标题: "[[7_Springboot]]" 
文章作者: Dakkk
文章概要: |
  介绍SpringBoot框架基础知识，包括快速入门、配置文件使用、多环境配置、整合JUnit和MyBatis，并通过SSM整合案例演示SpringBoot简化Spring开发的实践。
tags:
- "SpringBoot"
- "Spring"
- "Java"
- "MyBatis"
- "Maven"
- "配置管理"
- "微服务"
- "SSM"
相关文章:
- "[[11_反射]]"
- "[[2_Maven和Springboot下多环境的区别]]"
- "[[1_Java学习路线]]"
- "[[1_mapstruct 和 lombok 的冲突问题]]"
- "[[2_后端封装 MD 转换 HTML 工具类]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/2_SSM（黑马）/7_Springboot.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---


## 1 本文摘要：

- 掌握基于SpringBoot框架的程序开发步骤
- 熟练使用SpringBoot配置信息修改服务器配置
- 基于SpringBoot的完成SSM整合项目开发
## 2 SpringBoot简介

SpringBoot是由Pivotal团队提供的全新框架，其设计目的是用来`简化`Spring应用的`初始搭建`以及`开发过程`。  

使用了Spring框架后已经简化了我们的开发，而SpringBoot又是对Spring开发进行简化的，可想而知SpringBoot使用的简单及广泛性。  

既然SpringBoot是用来简化Spring开发的，那我们就先回顾一下，以SpringMVC开发为例

1. 创建一个maven工程，并在pom.xml中导入所需依赖的坐标
```xml
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
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.12</version>  
        <scope>test</scope>  
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
```

2. 编写web3.0的配置类
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[]{SpringConfig.class};  
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
```

3. 编写SpringMvc配置类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
public class SpringMvcConfig implements WebMvcConfigurer {  
  
}
```

4. 编写Controller类
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private BookService bookService;  
  
    @PostMapping  
    public boolean save(@RequestBody Book book) {  
        return bookService.save(book);  
    }  
  
    @PutMapping  
    public boolean update(@RequestBody Book book) {  
        return bookService.update(book);  
    }  
  
    @DeleteMapping("/{id}")  
    public boolean delete(@PathVariable Integer id) {  
        return bookService.delete(id);  
    }  
  
    @GetMapping("/{id}")  
    public Book getById(@PathVariable Integer id) {  
        return bookService.getById(id);  
    }  
  
    @GetMapping  
    public List<Book> getAll() {  
        return bookService.getAll();  
    }  
}
```

从上面的 `SpringMVC` 程序开发可以看到，前三步都是在搭建环境，而且这三步基本都是固定的。`SpringBoot` 就是对这三步进行简化了。接下来我们通过一个入门案例来体现 `SpingBoot` 简化 `Spring` 开发。
### 2.1 SpringBoot快速入门

#### 2.1.1 开发步骤

`SpringBoot` 开发起来特别简单，分为如下几步：
- 创建新模块，选择Spring初始化，并配置模块相关基础信息
- 选择当前模块需要使用的技术集
- 开发控制器类
- 运行自动生成的Application类  
    知道了 `SpringBoot` 的开发步骤后，下面我们进行具体的操作

- `步骤一：`创建新模块  
    在IDEA下创建一个新模块，选择Spring Initializr，用来创建SpringBoot工程![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c053ef124e66d6d6ef51379eecb4ab59.png)


    选中 `Web`，然后勾选 `Spring Web`，由于我们需要开发一个 `web` 程序，使用到了 `SpringMVC` 技术，所以按照下图红框进行勾选![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dced1d7db76bdad1a2e0960e36898304.png)


    最后点击创建，就大功告成了，经过以上步骤后就创建了如下结构的模块，它会帮我们自动生成一个 `Application` 类，而该类一会再启动服务器时会用到

	<font color="#00b050">注意：</font>
	1. 在创建好的工程中不需要创建配置类
	2. 创建好的项目会自动生成其他的一些文件，而这些文件目前对我们来说没有任何作用，所以可以将这些文件删除。
	3. 可以删除的目录和文件如下：
	    - `.mvn`
	    - `.gitignore`
	    - `HELP.md`
	    - `mvnw`
	    - `mvnw.cmd`

- `步骤二：`创建Controller  
	在`com.blog.controller`包下创建BookController，代码如下
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id) {  
        System.out.println("get id ==> " + id);  
        return "hello,spring boot!";  
    }  
}
```

- `步骤三：`启动服务器  
	运行 `SpringBoot` 工程不需要使用本地的 `Tomcat` 和 插件，只运行项目 `com.blog` 包下的 `Application` 类，我们就可以在控制台看出如下信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0410db728d898447e5a55dd9f1b8b4ed.png)


- `步骤四：`进行测试  
    依旧是使用PostMan来测试，发送GET请求访问`localhost:8080/books/9527`  
    可以看到响应回来的结果`hello,spring boot!`  
    同时控制台也输出了`get id ==> 9527`

---

通过上面的入门案例我们可以看到使用 `SpringBoot` 进行开发，使整个开发变得很简单，那它是如何做到的呢？

- 要研究这个问题，我们需要看看 `Application` 类和 `pom.xml` 都书写了什么。先看看 `Applicaion` 类，该类内容如下：
```java
@SpringBootApplication  
public class Application {  
    public static void main(String[] args) {  
        SpringApplication.run(Application.class, args);  
    }  
}
```
- 这个类中的东西很简单，就在类上添加了一个 `@SpringBootApplication` 注解，而在主方法中就一行代码。我们在启动服务器时就是执行的该类中的主方法。

---

- 再看看 `pom.xml` 配置文件中的内容
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <!--指定了一个父工程，父工程中的东西在该工程中可以继承过来使用-->  
    <parent>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.7.3</version>  
        <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
  
    <groupId>com.blog</groupId>  
    <artifactId>springboot_01_quickstart</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
      
    <!--JDK 的版本-->  
    <properties>  
        <java.version>1.8</java.version>  
    </properties>  
    <dependencies>  
        <!--该依赖就是我们在创建 SpringBoot 工程勾选的那个 Spring Web 产生的-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <!--这个是单元测试的依赖，我们现在没有进行单元测试，所以这个依赖现在可以没有-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
    </dependencies>  
  
    <build>  
        <plugins>  
            <!--这个插件是在打包时需要的，而这里暂时还没有用到-->  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>  
        </plugins>  
    </build>  
  
</project>
```

我们代码之所以能简化，就是因为指定的父工程和 `Spring Web` 依赖实现的。具体的我们后面在聊。
#### 2.1.2 对比

做完 `SpringBoot` 的入门案例后，接下来对比一下 `Spring` 程序和 `SpringBoot` 程序。

|类/配置文件|Spring|SpringBoot|
|---|---|---|
|pom文件中的坐标|手工添加|勾选添加|
|web3.e配置类|手工制作|无|
|Spring/SpringMVC配置类|手工制作|无|
|控制器|手工制作|手工制作|

- 坐标  
    `Spring` 程序中的坐标需要自己编写，而且坐标非常多  
    `SpringBoot` 程序中的坐标是我们在创建工程时进行勾选自动生成的
- web3.0配置类  
    `Spring` 程序需要自己编写这个配置类。这个配置类我们之前编写过，肯定感觉很复杂  
    `SpringBoot` 程序不需要我们自己书写
- 配置类  
    `Spring/SpringMVC` 程序的配置类需要自己书写。而 `SpringBoot` 程序则不需要书写。

<font color="#00b050">注意</font>：基于Idea的 `Spring Initializr` 快速构建 `SpringBoot` 工程时需要联网。
#### 2.1.3 官网构建工程

在入门案例中之所以能快速构建 `SpringBoot` 工程，是因为 `Idea` 使用了官网提供了快速构建 `SpringBoot` 工程的组件实现的。  

首先进入SpringBoot官网 [https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot) ，拉到页面最下方，会有一个`Quickstart your project`  

然后点击`Spring Initializr`超链接，就会跳转到如下页面，构建工程的步骤与我们在IDEA中几乎没什么区别![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a792e4e059ace604b8d526e0ec5e611.png)


点击`GENERATE`，就可以生成工程并下载到本地了，打开下载好的压缩包，可以看到工程的内容与IDEA生成的一模一样。  

通过上面官网的操作，我们知道 `Idea` 中快速构建 `SpringBoot` 工程其实就是使用的官网的快速构建组件，那以后即使没有 `Idea` 也可以使用官网的方式构建 `SpringBoot` 工程。
#### 2.1.4 SpringBoot工程快速启动

- 问题引入  
    以后我们和前端开发人员协同开发，而前端开发人员需要测试前端程序就需要后端开启服务器，这就受制于后端开发人员。为了摆脱这个受制，前端开发人员尝试着在自己电脑上安装 `Tomcat` 和 `Idea` ，在自己电脑上启动后端程序，这显然不现实。  
    我们后端可以将 `SpringBoot` 工程打成 `jar` 包，该 `jar` 包运行不依赖于 `Tomcat` 和 `Idea` 这些工具也可以正常运行，只是这个 `jar` 包在运行过程中连接和我们自己程序相同的 `Mysql` 数据库即可，这样就可以解决这个问题。

- 那现在问题就是如何打包  
    由于我们在构建 `SpringBoot` 工程时已经在 `pom.xml` 中配置了如下插件
```xml
<plugin>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-maven-plugin</artifactId>  
</plugin>
```

所以我们只需要使用 `Maven` 的 `package` 指令打包就会在 `target` 目录下生成对应的 `Jar` 包。

<font color="#00b050">注意</font>：该插件必须配置，不然打好的 `jar` 包也是有问题的。

- 启动  
	进入 `jar` 包所在位置，在 `命令提示符` 中输入如下命令
```plaintext
java -jar springboot_01_quickstart-0.0.1-SNAPSHOT.jar
```

执行上述命令就可以看到 `SpringBoot` 运行的日志信息，同时使用PostMan发送GET请求访问`localhost:8080/books/9527`，也可以正常输出`get id ==> 9527`![[![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/490372a80ba44217b129ab175698ed96.png)
### 2.2 SpringBoot概述

`SpringBoot` 是由Pivotal团队提供的全新框架，其设计目的是用来`简化`Spring应用的`初始搭建`以及`开发过程`

原始 `Spring` 环境搭建和开发存在以下问题

- 配置繁琐
- 依赖设置繁琐  
    `SpringBoot` 程序优点恰巧就是针对 `Spring` 的缺点
- 自动配置。这个是用来解决 `Spring` 程序配置繁琐的问题
- 起步依赖。这个是用来解决 `Spring` 程序依赖设置繁琐的问题
- 辅助功能（内置服务器，…）。我们在启动 `SpringBoot` 程序时既没有使用本地的 `tomcat` 也没有使用 `tomcat` 插件，而是使用 `SpringBoot` 内置的服务器。

接下来我们来说一下 `SpringBoot` 的起步依赖
#### 2.2.1 起步依赖

我们使用 `Spring Initializr` 方式创建的 `Maven` 工程的的 `pom.xml` 配置文件中自动生成了很多包含 `starter` 的依赖

```xml
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <!--                      ↓↓↓              -->  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>2.7.3</version>  
    <relativePath/>   
</parent>  
<groupId>com.blog</groupId>  
<artifactId>springboot_01_quickstart</artifactId>  
<version>0.0.1-SNAPSHOT</version>  
<properties>  
    <java.version>1.8</java.version>  
</properties>  
<dependencies>  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <!--                      ↓↓↓              -->  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <!--                      ↓↓↓              -->  
        <artifactId>spring-boot-starter-test</artifactId>  
        <scope>test</scope>  
    </dependency>  
</dependencies>
```

这些依赖就是启动依赖，接下来我们探究一下它是如何实现的。
##### 2.2.1.1 探索父工程

从上面的文件中可以看到指定了一个父工程
```xml
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>2.7.3</version>  
    <relativePath/>  
</parent>
```

我们进入到父工程，发现父工程中又指定了一个父工程
```xml
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-dependencies</artifactId>  
    <version>2.7.3</version>  
</parent>
```

再进入到该父工程中，在该工程中我们可以看到配置内容结构如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e15e8372469c750893a0fd59c7620cae.png)


- `properties` 标签中定义了各个技术软件依赖的版本，避免了我们在使用不同软件技术时考虑版本的兼容问题。
```xml
<activemq.version>5.16.5</activemq.version>  
<antlr2.version>2.7.7</antlr2.version>  
<appengine-sdk.version>1.9.98</appengine-sdk.version>  
<artemis.version>2.19.1</artemis.version>  
<aspectj.version>1.9.7</aspectj.version>  
<assertj.version>3.22.0</assertj.version>  
<atomikos.version>4.0.6</atomikos.version>  
<awaitility.version>4.2.0</awaitility.version>  
<build-helper-maven-plugin.version>3.3.0</build-helper-maven-plugin.version>  
<byte-buddy.version>1.12.13</byte-buddy.version>  
  
···
```

- `dependencyManagement` 标签是进行依赖版本锁定，但是并没有导入对应的依赖；如果我们工程需要那个依赖只需要引入依赖的 `groupid` 和 `artifactId` 不需要定义 `version`。
```xml
<dependency>  
    <groupId>org.apache.activemq</groupId>  
    <artifactId>activemq-amqp</artifactId>  
    <version>${activemq.version}</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.activemq</groupId>  
    <artifactId>activemq-blueprint</artifactId>  
    <version>${activemq.version}</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.activemq</groupId>  
    <artifactId>activemq-broker</artifactId>  
    <version>${activemq.version}</version>  
</dependency>  
  
···
```

- 而 `build` 标签中也对插件的版本进行了锁定
```xml
<plugin>  
    <groupId>org.codehaus.mojo</groupId>  
    <artifactId>build-helper-maven-plugin</artifactId>  
    <version>${build-helper-maven-plugin.version}</version>  
</plugin>  
<plugin>  
    <groupId>org.flywaydb</groupId>  
    <artifactId>flyway-maven-plugin</artifactId>  
    <version>${flyway.version}</version>  
</plugin>  
<plugin>  
    <groupId>pl.project13.maven</groupId>  
    <artifactId>git-commit-id-plugin</artifactId>  
    <version>${git-commit-id-plugin.version}</version>  
</plugin>  
  
···
```
看完了父工程中 `pom.xml` 的配置后不难理解我们工程的的依赖为什么都没有配置 `version`。
##### 2.2.1.2 探索依赖

在我们创建的工程中的pom.xml中配置了如下依赖
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>
```

进入到该依赖，查看pom.xml的依赖，会发现它引入了如下依赖
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-tomcat</artifactId>  
    <version>2.7.3</version>  
    <scope>compile</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-web</artifactId>  
    <version>5.3.22</version>  
    <scope>compile</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.3.22</version>  
    <scope>compile</scope>  
</dependency>
```

里面的引入了 `spring-web` 和 `spring-webmvc` 的依赖，这就是为什么我们的工程中没有依赖这两个包还能正常使用 `springMVC` 中的注解的原因。  

而依赖 `spring-boot-starter-tomcat` ，从名字基本能确认内部依赖了 `tomcat`，所以我们的工程才能正常启动。  

结论：**以后需要使用技术，只需要引入该技术对应的起步依赖即可**
##### 2.2.1.3 小结

- starter
    - `SpringBoot` 中常见项目名称，定义了当前项目使用的所有项目坐标，以达到减少依赖配置的目的
- parent
    - 所有 `SpringBoot` 项目要继承的项目，定义了若干个坐标版本号（依赖管理，而非依赖），以达到减少依赖冲突的目的
- 实际开发
    - 使用任意坐标时，仅书写GAV中的G和A，V由SpringBoot提供
        - G：groupid
        - A：artifactId
        - V：version
    - 如发生坐标错误，再指定version（要小心版本冲突）
#### 2.2.2 程序启动

创建的每一个 `SpringBoot` 程序时都包含一个类似于下面的类，我们将这个类称作引导类
```java
@SpringBootApplication  
public class Springboot01QuickstartApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(Springboot01QuickstartApplication.class, args);  
    }  
}
```

<font color="#00b050">注意</font>：
- `SpringBoot` 在创建项目时，采用jar的打包方式
- `SpringBoot` 的引导类是项目的入口，运行 `main` 方法就可以启动项目  
    因为我们在 `pom.xml` 中配置了 `spring-boot-starter-web` 依赖，而该依赖通过前面的学习知道它依赖 `tomcat` ，所以运行 `main` 方法就可以使用 `tomcat` 启动咱们的工程。
#### 2.2.3 切换web服务器

现在我们启动工程使用的是 `tomcat` 服务器，那能不能不使用 `tomcat` 而使用 `jetty` 服务器。而要切换 `web` 服务器就需要将默认的 `tomcat` 服务器给排除掉，怎么排除呢？需要用到我们前面学的知识`排除依赖`，使用 `exclusion` 标签
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <exclusions>  
        <exclusion>  
            <artifactId>spring-boot-starter-tomcat</artifactId>  
            <groupId>org.springframework.boot</groupId>  
        </exclusion>  
    </exclusions>  
</dependency>
```

然后还要引入 `jetty` 服务器。
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-jetty</artifactId>  
</dependency>
```

Jetty比Tomcat更轻量级，可拓展性更强（相对于Tomcat），谷歌应用引擎（GAE）已全面切换为Jetty

接下来再次运行引导类，在日志信息中就可以看到使用的是jetty服务器![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/215fb80485454ba91216b092341c07f1.png)
小结：通过切换服务器，我们不难发现在使用 `SpringBoot` 换技术时只需要导入该技术的`起步依赖`即可。
## 3 配置文件

### 3.1 配置文件格式

我们现在启动服务器默认的端口号是 `8080`，访问路径可以书写为  
[http://localhost:8080/books/1](http://localhost:8080/books/1)

在线上环境我们还是希望将端口号改为 `80`，这样在访问的时候就可以不写端口号了，如下  
[http://localhost/books/1](http://localhost/books/1)

而 `SpringBoot` 程序如何修改呢？`SpringBoot` 提供了多种属性配置方式

- `application.properties`
```properties
server.port=80
```

- `application.yml`
```yaml
server:  
	port: 81
```

- `application.yaml`
```yaml
server:  
	port: 82
```

<font color="#00b050">注意</font>：`SpringBoot` 程序的配置文件名必须是 `application` ，只是后缀名不同而已。
#### 3.1.1 不同配置文件演示

`application.properties`配置文件

- 配置文件必须放在 `resources` 目录下，而该目录下有一个名为 `application.properties` 的配置文件（SpringBoot已经为我们提供好了），我们就可以在该配置文件中修改端口号，在该配置文件中书写 `port` ，`Idea` 就会补全提示

- `application.properties` 配置文件内容如下：
```properties
server.port=80
```

- 启动服务，会在控制台打印出日志信息，从日志信息中可以看到绑定的端口号已经修改了
  ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/94a340e801860c5b3e13bc0fb35d33db.png)
---

`application.yml`配置文件
- 删除 `application.properties` 配置文件中的内容。在 `resources` 下创建一个名为 `application.yml` 的配置文件，在该文件中书写端口号的配置项，格式如下：
```yml
server:  
  port: 81
```

<font color="#00b050">注意</font>： 在`:`后，数据前一定要加空格。
- 启动服务，可以在控制台看到绑定的端口号是 `81`

--- 

- `application.yaml`配置文件
- 删除 `application.yml` 配置文件和 `application.properties` 配置文件内容，然后在 `resources` 下创建名为 `application.yaml` 的配置文件，配置内容和后缀名为 `yml` 的配置文件中的内容相同，只是使用了不同的后缀名而已
- `application.yaml` 配置文件内容如下：
```yml
server:  
  port: 82
```

- 启动服务，在控制台可以看到绑定的端口号![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0881ae0c0789ee5ecce0a8a639659278.png)

#### 3.1.2 三种配置文件的优先级

在三种配合文件中分别配置不同的端口号，启动服务查看绑定的端口号。用这种方式就可以看到哪个配置文件的优先级更高一些

- `application.properties` 文件内容如下：
```properties
server.port=80
```

- `application.yml` 文件内容如下：
```yaml
server:  
port: 81
```

- `application.yaml` 文件内容如下：
```yaml
server:  
port: 82
```

- 启动服务，在控制台可以看到使用的端口号是 `80`。说明 `application.properties` 的优先级最高

- 注释掉 `application.properties` 配置文件内容。再次启动服务，在控制台可以看到使用的端口号是 `81`，说明 `application.yml` 配置文件为第二优先级。

- 从上述的验证结果可以确定三种配置文件的优先级是：`application.properties` > `application.yml` > `application.yaml`

注意：
- `SpringBoot` 核心配置文件名为 `application`
- `SpringBoot` 内置属性过多，且所有属性集中在一起修改，在使用时，通过代码补全+关键字修改属性
- 例如要设置日志的级别时，可以在配置文件中书写 `logging`，就会提示出来。配置内容如下
```yml
logging:  
  level:  
    root: info
```
### 3.2 yaml格式

上面讲了三种不同类型的配置文件，而 `properties` 类型的配合文件之前我们学习过，接下来我们重点学习 `yaml` 类型的配置文件。  

YAML（YAML Ain’t Markup Language），一种数据序列化格式。这种格式的配置文件在近些年已经占有主导地位，那么这种配置文件和前期使用的配置文件是有一些优势的，我们先看之前使用的配置文件。

- 最开始我们使用的是 `xml` ，格式如下：
```xml
<enterprise>  
    <name>Helsing</name>  
    <age>16</age>  
    <tel>400-957-241</tel>  
</enterprise>
```

- 而 `properties` 类型的配置文件如下
```properties
enterprise.name=Helsing  
enterprise.age=16  
enterprise.tel=400-957-241
```

- `yaml` 类型的配置文件内容如下
```yaml
enterprise:  
  name: Helsing  
  age: 16  
  tel: 400-957-241
```

- 通过对比，我们得出yaml的优点有：
    - 容易阅读
        - `yaml` 类型的配置文件比 `xml` 类型的配置文件更容易阅读，结构更加清晰
    - 容易与脚本语言交互（暂时还体会不到，后面会了解）
    - 以数据为核心，重数据轻格式
        - `yaml` 更注重数据，而 `xml` 更注重格式
- YAML 文件扩展名：
    - `.yml` (主流)
    - `.yaml`

上面两种后缀名都可以，以后使用更多的还是 `yml` 的。

---

yml语法规则
- 大小写敏感
- 属性层级关系使用多行描述，每行结尾使用冒号结束
- 使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用Tab键）
    - 空格的个数并不重要，只要保证同层级的左侧对齐即可。
- 属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔）

- `#表示注释`
    核心规则：数据前面要加空格与冒号隔开

- 数组数据在数据书写位置的下方使用减号作为数据开始符号，每行书写一个数据，减号与数据间空格分隔，例如
```yaml
enterprise:  
  name: Helsing  
  age: 16  
  tel: 400-957-241  
  subject:  
    - Java  
    - Python  
    - C#
```
### 3.3 yaml配置文件数据读取

#### 3.3.1 环境准备

- 修改`resource`目录下的`application.yml`配置文件
```yml
lesson: SpringBoot  
  
server:  
  port: 80  
  
enterprise:  
  name: Helsing  
  age: 16  
  tel: 400-957-241  
  subject:  
    - Java  
    - Python  
    - C#
```

- 在`com.blog.domain`包下新建一个Enterprise类，用来封装数据
```java
@Data
public class Enterprise {  
    private String name;  
    private int age;  
    private String tel;  
    private String[] subject;   
}
```
#### 3.3.2 读取配置文件

`方式一：`使用 @Value注解

- 使用 `@Value("表达式")` 注解可以从配合文件中读取数据，注解中用于读取属性名引用方式是：`${一级属性名.二级属性名……}`
- 我们可以在 `BookController` 中使用 `@Value` 注解读取配合文件数据，如下
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Value("${lesson}")  
    private String lesson;  
    @Value("${server.port}")  
    private Integer port;  
    @Value("${enterprise.subject[0]}")  
    private String subject_0;  
  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id) {  
        System.out.println(lesson);  
        System.out.println(port);  
        System.out.println(subject_0);  
        return "hello , spring boot!";  
    }  
}
```
- 使用PostMan发送请求，控制台输出如下，成功获取到了数据
  ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6415f5d3dce50a6455d41dfad1ac2ead.png)

`方式二：`使用`Environment`对象

- 上面方式读取到的数据特别零散，`SpringBoot` 还可以使用 `@Autowired` 注解注入 `Environment` 对象的方式读取数据。这种方式 `SpringBoot` 会将配置文件中所有的数据封装到 `Environment` 对象中，如果需要使用哪个数据只需要通过调用 `Environment` 对象的 `getProperty(String name)` 方法获取。具体代码如下
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private Environment environment;  
  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id) {  
        System.out.println(environment.getProperty("lesson"));  
        System.out.println(environment.getProperty("enterprise.name"));  
        System.out.println(environment.getProperty("enterprise.subject[1]"));  
        return "hello , spring boot!";  
    }  
}
```
- 使用PostMan发送请求，控制台输出如下，成功获取到了数据
- <font color="#00b050">注意</font>：这种方式在开发中很少用，因为框架内含大量数据


`方式三：`使用自定义对象

- `SpringBoot` 还提供了将配置文件中的数据封装到我们自定义的实体类对象中的方式。具体操作如下：
    - 将实体类 `bean` 的创建交给 `Spring` 管理。
        - 在类上添加 `@Component` 注解
    - 使用 `@ConfigurationProperties` 注解表示加载配置文件
        - 在该注解中也可以使用 `prefix` 属性指定只加载指定前缀的数据
    - 在 `BookController` 中进行注入
- 具体代码如下
    - `Enterprise` 实体类内容如下：
```java
@Component  
@ConfigurationProperties(prefix = "enterprise")
@Data
public class Enterprise {  
    private String name;  
    private int age;  
    private String tel;  
    private String[] subject;    
}
```

- BookController内容如下
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private Enterprise enterprise;  
  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id) {  
        System.out.println(enterprise);  
        System.out.println(enterprise.getAge());  
        System.out.println(enterprise.getName());  
        System.out.println(enterprise.getTel());  
        return "hello , spring boot!";  
    }  
}
```
- 使用PostMan发送请求，控制台输出如下，成功获取到了数据
  ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab1c7e9c55e57f229c59f049da9e94e3.png)

可能遇到的问题：
- 在Enterprise实体类上遇到`Spring Boot Configuration Annotation Processor not configured`警告提示

解决方案
- 在`pom.xml`中添加如下依赖即可
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-configuration-processor</artifactId>  
    <optional>true</optional>  
</dependency>
```
### 3.4 多环境配置

以后在工作中，对于开发环境、测试环境、生产环境的配置肯定都不相同，比如我们开发阶段会在自己的电脑上安装 `mysql` ，连接自己电脑上的 `mysql` 即可，但是项目开发完毕后要上线就需要该配置，将环境的配置改为线上环境的。  

来回的修改配置会很麻烦，而 `SpringBoot` 给开发者提供了多环境的快捷配置，需要切换环境时只需要改一个配置即可。不同类型的配置文件多环境开发的配置都不相同，接下来对不同类型的配置文件进行说明
#### 3.4.1 yaml文件

在 `application.yml` 中使用 `---` 来分割不同的配置，内容如下
```yml
# 开发环境  
spring:  
  profiles: dev # 给开发环境取的名  
server:  
  port: 80  
---  
# 生产环境  
spring:  
  profiles: pro # 给生产环境取的名  
server:  
  port: 81  
---  
# 测试环境  
spring:  
  profiles: test # 给测试环境起的名  
server:  
  port: 82
```

上面配置中 `spring.profiles` 是用来给不同的配置起名字的。而如何告知 `SpringBoot` 使用哪段配置呢？可以使用如下配置来启用都一段配置
```yml
# 设置启用的环境  
spring:  
  profiles:  
    active: dev  # 表示使用的是开发环境的配置
```

综上所述，`application.yml` 配置文件内容如下
```yml
spring:  
  profiles:  
    active: dev  

# 以三个-来区分不同环境的配置文件
---  
# 开发环境  
spring:  
  profiles: dev # 给开发环境取的名  
server:  
  port: 80  
---  
# 生产环境  
spring:  
  profiles: pro # 给生产环境取的名  
server:  
  port: 81  
---  
# 测试环境  
spring:  
  profiles: test # 给测试环境起的名  
server:  
  port: 82
```

<font color="#00b050">注意</font>：在上面配置中给不同配置起名字的 `spring.profiles` 配置项已经过时。最新用来起名字的配置项是
```yml
# 开发环境  
spring:  
  config:  
    activate:  
      on-profile: dev # 给开发环境取的名
```


#### 3.4.2 properties文件

`properties` 类型的配置文件配置多环境需要`定义不同的配置文件`

- `application-dev.properties` 是开发环境的配置文件。我们在该文件中配置端口号为 `80`
```properties
server.port=80
```

- `application-test.properties` 是测试环境的配置文件。我们在该文件中配置端口号为 `81`
```properties
server.port=81
```

- `application-pro.properties` 是生产环境的配置文件。我们在该文件中配置端口号为 `82`
```properties
server.port=82
```

- `SpringBoot` 只会默认加载名为 `application.properties` 的配置文件，所以需要在 `application.properties` 配置文件中设置启用哪个配置文件，配置如下:
```properties
spring.profiles.active=pro
```
#### 3.4.3 命令行启动参数设置

使用 `SpringBoot` 开发的程序以后都是打成 `jar` 包，通过 `java -jar xxx.jar` 的方式启动服务的。那么就存在一个问题，如何切换环境呢？因为配置文件打到的jar包中了。

我们知道 `jar` 包其实就是一个压缩包，可以解压缩，然后修改配置，最后再打成jar包就可以了。这种方式显然有点麻烦，而 `SpringBoot` 提供了在运行 `jar` 时设置开启指定的环境的方式，如下
```shell
java -jar xxx.jar --spring.profiles.active=test
```

那么这种方式能不能临时修改端口号呢？也是可以的，可以通过如下方式
```shell
java -jar xxx.jar --server.port=9421
```

当然也可以同时设置多个配置，比如即指定启用哪个环境配置，又临时指定端口，如下
```shell
java -jar xxx.jar -server.port=9421 --spring.profiles.active=pro
```

那现在命令行配置的端口号是9421，配置文件中的端口号为82，那么结果将会是多少呢？  
测试后就会发现命令行设置的端口号优先级高（也就是使用的是命令行设置的端口号），配置的优先级其实 `SpringBoot` 官网已经进行了说明，详情参见 [https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)  
如果使用了多种方式配合同一个配置项，优先级高的生效。
### 3.5 配置文件分类

有这样的场景，我们开发完毕后需要测试人员进行测试，由于测试环境和开发环境的很多配置都不相同，所以测试人员在运行我们的工程时需要临时修改很多配置，如下
```shell
java –jar springboot.jar –-spring.profiles.active=test --server.port=85 --server.servlet.context-path=/heima --server.tomcat.connection-timeout=-1 …… …… …… …… ……
```

针对这种情况，`SpringBoot` 定义了配置文件不同的放置的位置；而放在不同位置的优先级时不同的。

- `SpringBoot` 中4级配置文件放置位置：
    - 1级：classpath：application.yml
    - 2级：classpath：config/application.yml
    - 3级：file ：application.yml
    - 4级：file ：config/application.yml
        说明：级别越高的优先级越高，file指打包后的jar包所在的文件夹

下面我们来验证这个优先级
#### 3.5.1 1级和2级

1级就是resource目录下的`application.yml`，2级是在resource目录下新建一个config文件，在其中新建`application.yml`

- 1级
```yml
server:  
  port: 80
```
- 2级
```yml
server:  
  port: 81
```

启动引导类，控制台输出的为81端口
#### 3.5.2 3级和4级

先将工程打成一个jar包，进入到jar包的目录下，创建`application.yml` 配置文件，而在该配合文件中将端口号设置为 `82`  

在 `jar` 包所在位置创建 `config` 文件夹，在该文件夹下创建 `application.yml` 配置文件，而在该配合文件中将端口号设置为 `83`

- 3级
```yml
server:  
  port: 82
```
- 4级
```yml
server:  
  port: 83
```

在命令行使用以下命令运行程序
```shell
java -jar springboot_06_config_file-0.0.1-SNAPSHOT.jar
```

运行后日志信息如下，端口为83

通过这个结果可以得出一个结论 `config`下的配置文件优先于类路径下的配置文件。
## 4 SpringBoot整合Junit

首先我们来回顾一下 `Spring` 整合 `junit`
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class UserServiceTest {  
      
    @Autowired  
    private BookService bookService;  
      
    @Test  
    public void testSave(){  
        bookService.save();  
    }  
}
```

使用 `@RunWith` 注解指定运行器，使用 `@ContextConfiguration` 注解来指定配置类或者配置文件。

而 `SpringBoot` 整合 `junit` 特别简单，分为以下三步完成

- 在测试类上添加 `SpringBootTest` 注解
- 使用 `@Autowired` 注入要测试的资源
- 定义测试方法进行测试
### 4.1 环境准备

- 创建一个新的SpringBoot工程
- 在com.blog.service包下创建BookService接口
```java
public interface BookService {  
    void save();  
}
```

- 在com.blog.service.impl包下创建BookService接口的实现类，并重写其方法
```java
@Service  
public class BookServiceImpl implements BookService {  
    @Override  
    public void save() {  
        System.out.println("book service is running ..");  
    }  
}
```
### 4.2 编写测试类

在 `test/java` 下创建 `com.blog` 包，在该包下创建测试类，将 `BookService` 注入到该测试类中
```java
@SpringBootTest  
class Springboot02JunitApplicationTests {  
    @Autowired  
    private BookService bookService;  
  
    @Test  
    void contextLoads() {  
        bookService.save();  
    }  
  
}
```

运行测试方法，控制台成功输出

<font color="#00b050">注意</font>：这里的引导类所在包必须是测试类所在包及其子包。

例如：
- 引导类所在包是 `com.blog`
- 测试类所在包是 `com.blog`

如果不满足这个要求的话，就需要在使用 `@SpringBootTest` 注解时，使用 `classes` 属性指定引导类的字节码对象。如 `@SpringBootTest(classes = XxxApplication.class)`
## 5 SpringBoot整合MyBatis

### 5.1 回顾Spring整合MyBatis

之前Spring整合MyBatis时，需要定义很多配置类

- SpringConfig配置类
```java
@Configuration  
@ComponentScan("com.blog")  
@PropertySource("jdbc.properties")  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- 导入JdbcConfig配置类
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

- 导入MyBatisConfig配置类
	- 定义 `SqlSessionFactoryBean`
	- 定义映射配置
```java
public class MyBatisConfig {  
  
    //定义bean，SqlSessionFactoryBean，用于产生SqlSessionFactory对象  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {  
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
### 5.2 SpringBoot整合MyBatis

- 创建一个新的模块  
    注意选择技术集的时候，要勾选`MyBatis Framework`和`MySQL Driver`
- 建库建表
```sql
CREATE DATABASE springboot_db;  
USE springboot_db;  
  
CREATE TABLE tbl_book  
(  
    id          INT PRIMARY KEY AUTO_INCREMENT,  
    `type`      VARCHAR(20),  
    `name`      VARCHAR(50),  
    description VARCHAR(255)  
);  
  
INSERT INTO `tbl_book`(`id`, `type`, `name`, `description`)  
VALUES (1, '计算机理论', 'Spring实战 第五版', 'Spring入门经典教程，深入理解Spring原理技术内幕'),  
       (2, '计算机理论', 'Spring 5核心原理与30个类手写实践', '十年沉淀之作，手写Spring精华思想'),  
       (3, '计算机理论', 'Spring 5设计模式', '深入Spring源码刨析Spring源码中蕴含的10大设计模式'),  
       (4, '计算机理论', 'Spring MVC+Mybatis开发从入门到项目实战',  
        '全方位解析面向Web应用的轻量级框架，带你成为Spring MVC开发高手'),  
       (5, '计算机理论', '轻量级Java Web企业应用实战', '源码级刨析Spring框架，适合已掌握Java基础的读者'),  
       (6, '计算机理论', 'Java核心技术 卷Ⅰ 基础知识(原书第11版)',  
        'Core Java第11版，Jolt大奖获奖作品，针对Java SE9、10、11全面更新'),  
       (7, '计算机理论', '深入理解Java虚拟机', '5个纬度全面刨析JVM,大厂面试知识点全覆盖'),  
       (8, '计算机理论', 'Java编程思想(第4版)', 'Java学习必读经典，殿堂级著作！赢得了全球程序员的广泛赞誉'),  
       (9, '计算机理论', '零基础学Java(全彩版)', '零基础自学编程的入门图书，由浅入深，详解Java语言的编程思想和核心技术'),  
       (10, '市场营销', '直播就这么做:主播高效沟通实战指南', '李子柒、李佳奇、薇娅成长为网红的秘密都在书中'),  
       (11, '市场营销', '直播销讲实战一本通', '和秋叶一起学系列网络营销书籍'),  
       (12, '市场营销', '直播带货:淘宝、天猫直播从新手到高手', '一本教你如何玩转直播的书，10堂课轻松实现带货月入3W+');
```

- 定义实体类
```java
@Data
public class Book {  
    private Integer id;  
    private String type;  
    private String name;  
    private String description;   
}
```

- 定义dao接口  
	在com.blog.dao包下定义BookDao接口
```java
public interface BookDao {  
    @Select("select * from tbl_book where id = #{id}")  
    Book getById(Integer id);  
}
```

- 定义测试类
```java
@SpringBootTest  
class Springboot03MybatisApplicationTests {  
    @Autowired  
    private BookDao bookDao;  
    @Test  
    void contextLoads() {  
        Book book = bookDao.getById(1);  
        System.out.println(book);  
    }  
}
```

- 编写配置
```yml
spring:  
  datasource:  
    driver-class-name: com.mysql.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: PASSWORD
```

- 测试  
	运行测试方法，会报错`No qualifying bean of type 'com.blog.dao.BookDao'`，没有类型为“com.blog.dao.BookDao”的限定bean  
	为什么会出现这种情况呢？之前我们在配置MyBatis时，配置了如下内容
```java
@Bean  
public MapperScannerConfigurer mapperScannerConfigurer() {  
    MapperScannerConfigurer msc = new MapperScannerConfigurer();  
    msc.setBasePackage("com.blog.dao");  
    return msc;  
}
```

`Mybatis` 会扫描接口并创建接口的代码对象交给 `Spring` 管理，但是现在并没有告诉 `Mybatis` 哪个是 `dao` 接口。

而我们要解决这个问题需要在`BookDao` 接口上使用 `@Mapper` ，`BookDao` 接口修改为
```java
@Mapper  
public interface BookDao {  
    @Select("select * from tbl_book where id = #{id}")  
    Book getById(Integer id);  
}
```

注意：  
`SpringBoot` 版本低于2.4.3(不含)，Mysql驱动版本大于8.0时，需要在url连接串中配置时区 `jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC`，或在MySQL数据库端配置时区解决此问题

- 使用Druid数据源  
	现在我们并没有指定数据源，`SpringBoot` 有默认的数据源，我们也可以指定使用 `Druid` 数据源，按照以下步骤实现
```xml
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.12</version>  
</dependency>
```

- 在 `application.yml` 修改配置文件配置  
	可以通过 `spring.datasource.type` 来配置使用什么数据源。配置文件内容可以改进为
```yml
spring:  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: PASSWORD.  
    type: com.alibaba.druid.pool.DruidDataSource
```
## 6 案例

`SpringBoot` 到这就已经学习完毕，接下来我们将学习 `SSM` 时做的三大框架整合的案例用 `SpringBoot` 来实现一下。先将之前做的SSM整合的代码拷贝过来，修改成SpringBoot的，之后再自己手动敲一遍SpringBoot的全流程，就当复盘了。
### 6.1 创建工程

创建一个新的SpringBoot工程，注意要勾选`Spring Web`，`MyBatis Framework`和`MySQL Driver`  
由于我们工程中使用到了 `Druid` ，所以需要导入 `Druid` 的坐标
### 6.2 代码拷贝

将之前的ssm整合工程的代码拷贝过来，将`com.blog`包下的所有内容拷贝过来，放在对应的位置即可。  
需要修改的内容如下：

- 将`com.blog.config`包直接删掉，SpringBoot并不需要这些配置类
- 在dao包下的接口上，加上`@Mapper`注解
```java
@Mapper  
public interface BookDao {  
    @Insert("insert into tbl_book values (null, #{type}, #{name}, #{description})")  
    int save(Book book);  
  
    @Update("update tbl_book set type=#{type}, `name`=#{name}, `description`=#{description} where id=#{id}")  
    int update(Book book);  
  
    @Delete("delete from tbl_book where id=#{id}")  
    int delete(Integer id);  
  
    @Select("select * from tbl_book where id=#{id}")  
    Book getById(Integer id);  
  
    @Select("select * from tbl_book")  
    List<Book> getAll();  
}
```

- 将测试类也修改为SpringBoot的
```java
import com.blog.domain.Book;  
import org.junit.jupiter.api.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
  
@SpringBootTest  
public class BookServiceTest {  
  
    @Autowired  
    private BookService bookService;  
  
    @Test  
    public void testGetById() {  
        Book book = bookService.getById(1);  
        System.out.println(book);  
    }  
  
    @Test  
    public void testGetAll() {  
        for (Book book : bookService.getAll()) {  
            System.out.println(book);  
        }  
    }  
}
```
### 6.3 配置文件

在application.yml配置文件中配置如下内容

- 服务的端口号（设为80，这样我们就不用写了）
- 连接数据库的信息（数据库连接四要素）
- 数据源（德鲁伊）
```yml
server:  
  port: 80  
spring:  
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: PASSWORD.
```
### 6.4 静态资源

在 `SpringBoot` 程序中是没有 `webapp` 目录的，那么在 `SpringBoot` 程序中静态资源需要放在什么位置呢？  

静态资源需要放在 `resources` 下的 `static` 下  

那我们再配置一个默认页面，跳转到我们的增删改页面，新建`index.html`，写入以下内容
```html
<script>  
    document.location.href="/pages/books.html"  
</script>
```

这样当我们在浏览器输入localhost，然后直接按回车，就能直接跳转到增删改的页面了，如果以后我们需要频繁测试某一个页面，也可以将上述代码中的地址换为我们要测试的地址，这样就不用老手敲地址了

那么至此，将之前的ssm整合，改为springboot的工作，就完成了
## 7 复盘

### 7.1 流程分析

1. 创建工程
    - 创建一个SpringBoot工程
    - 需要勾选`Spring Web`，`MyBatis Framework`和`MySQL Driver`
2. SpringBoot整合
    - 整合MyBatis
        - 添加Druid数据源依赖
        - 编写数据库配置文件（application.yml），配置数据库连接四要素
        - 对于Dao层的包扫描，使用`@Mapper`注解
    - 整合Junit
        - 使用`@SpringBootTest`注解
3. 功能模块
    - 创建数据库和表
    - 根据数据表来创建对应的模型类
    - 通过`Dao`层完成数据库的增删改
    - 编写`Service`层（Service接口+实现类）
    - 编写`Controller`层
        - 接收请求 `@RequestMapping`、`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`
        - 接收数据 简单类型、POJO类型、嵌套POJO类型、数组类型、JSON数据类型
            - `@RequestParam`
            - `@PathVariable`
            - `@RequestBody`
        - 转发业务层
            - 使用`@Autowired`自动装配
        - 响应结果
            - `@ResponseBody`
### 7.2 整合配置

分析完毕之后，那我们现在就开始来完成SpringBoot的整合

- `步骤一：`创建一个SpringBoot工程  
    注意要勾选`Spring Web`，`MyBatis Framework`和`MySQL Driver`
- `步骤二：`创建项目包结构
    - `com.blog.controller` 编写Controller类
    - `com.blog.dao` 存放的是Dao层的接口，注意要使用`@Mapper`注解
    - `com.blog.service` 存放的是Service层接口，
    - `com.blog.service.impl` 存放的是Service的实现类
    - `com.blog.domain` 存放的是pojo类
    - `resources/static` 存放静态资源HTML，CSS，JS等
    - `test/java` 存放测试类
- `步骤三：`编写application.yml  
    导入Druid的坐标，并在配置文件中编写数据库连接四要素
```yml
server:  
  port: 80  
spring:  
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: YOURPASSWORD.
```
### 7.3 功能模块开发

- `步骤一：`创建数据库和表
```sql
create database springboot_db;  
use springboot_db;  
create table tbl_book  
(  
    id          int primary key auto_increment,  
    type        varchar(20),  
    `name`      varchar(50),  
    description varchar(255)  
);  
  
insert into `tbl_book`(`id`, `type`, `name`, `description`)  
values (1, '计算机理论', 'Spring实战 第五版', 'Spring入门经典教程，深入理解Spring原理技术内幕'),  
       (2, '计算机理论', 'Spring 5核心原理与30个类手写实践', '十年沉淀之作，手写Spring精华思想'),  
       (3, '计算机理论', 'Spring 5设计模式', '深入Spring源码刨析Spring源码中蕴含的10大设计模式'),  
       (4, '计算机理论', 'Spring MVC+Mybatis开发从入门到项目实战',  
        '全方位解析面向Web应用的轻量级框架，带你成为Spring MVC开发高手'),  
       (5, '计算机理论', '轻量级Java Web企业应用实战', '源码级刨析Spring框架，适合已掌握Java基础的读者'),  
       (6, '计算机理论', 'Java核心技术 卷Ⅰ 基础知识(原书第11版)',  
        'Core Java第11版，Jolt大奖获奖作品，针对Java SE9、10、11全面更新'),  
       (7, '计算机理论', '深入理解Java虚拟机', '5个纬度全面刨析JVM,大厂面试知识点全覆盖'),  
       (8, '计算机理论', 'Java编程思想(第4版)', 'Java学习必读经典，殿堂级著作！赢得了全球程序员的广泛赞誉'),  
       (9, '计算机理论', '零基础学Java(全彩版)', '零基础自学编程的入门图书，由浅入深，详解Java语言的编程思想和核心技术'),  
       (10, '市场营销', '直播就这么做:主播高效沟通实战指南', '李子柒、李佳奇、薇娅成长为网红的秘密都在书中'),  
       (11, '市场营销', '直播销讲实战一本通', '和秋叶一起学系列网络营销书籍'),  
       (12, '市场营销', '直播带货:淘宝、天猫直播从新手到高手', '一本教你如何玩转直播的书，10堂课轻松实现带货月入3W+');
```

- `步骤二：`编写模型类
```java
@Data
public class Book {  
    private Integer id;  
    private String type;  
    private String name;  
    private String description;  
}
```

- `步骤三：`编写dao接口  
	注意使用`@Mapper`注解
```java
@Mapper  
public interface BookDao {  
    @Select("select * from tbl_book where id=#{id}")  
    Book getById(Integer id);  
  
    @Select("select * from tbl_book")  
    List<Book> getAll();  
  
    @Update("update tbl_book set type=#{type}, `name`=#{name}, `description`=#{description} where id=#{id}")  
    int update(Book book);  
  
    @Delete("delete from tbl_book where id=#{id}")  
    int delete(Integer id);  
  
    @Insert("insert into tbl_book values (null, #{type}, #{name}, #{description})")  
    int save(Book book);  
}
```

- `步骤四：`编写service接口及其实现类
```java
public interface BookService {  
    boolean save(Book book);  
  
    boolean update(Book book);  
  
    boolean delete(Integer id);  
  
    Book getById(Integer id);  
  
    List<Book> getAll();  
}

@Service  
public class BookServiceImpl implements BookService {  
  
    @Autowired  
    private BookDao bookDao;  
  
    @Override  
    public boolean save( Book book) {  
        int cnt = bookDao.save(book);  
        return cnt > 0;  
    }  
  
    @Override  
    public boolean update(Book book) {  
        int cnt = bookDao.update(book);  
        return cnt>0;  
    }  
  
    @Override  
    public boolean delete(Integer id) {  
        return bookDao.delete(id) > 0;  
    }  
  
    @Override  
    public Book getById(Integer id) {  
        return bookDao.getById(id);  
    }  
  
    @Override  
    public List<Book> getAll() {  
        return bookDao.getAll();  
    }  
}
```

- `步骤五：`编写Controller类  
	注意响应pojo类型要加`@RequestBody`注解，某人忘加了，调试了半天
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private BookService bookService;  
  
    @GetMapping("/{id}")  
    public Book getById(@PathVariable Integer id) {  
        return bookService.getById(id);  
    }  
  
    @GetMapping  
    public List<Book> getAll() {  
        return bookService.getAll();  
    }  
  
    @PostMapping  
    public boolean save(@RequestBody Book book) {  
        return bookService.save(book);  
    }  
  
    @PutMapping  
    public boolean update(@RequestBody Book book) {  
        return bookService.update(book);  
    }  
  
    @DeleteMapping("/{id}")  
    public boolean delete(@PathVariable Integer id) {  
        return bookService.delete(id);  
    }  
}
```

- `步骤六：`使用PostMan进行测试  
    将增删改查全部测试完毕之后，就可以继续往下做了
### 7.4 统一结果封装

- 创建一个返回结果类  
	我这里暂时只需要返回的结果，状态码和异常信息，如果还有别的需求，可以自行删改
```java
public class Result {  
    private Object data;  
    private Integer code;  
    private String msg;  
  
    public Object getData() {  
        return data;  
    }  
  
    public void setData(Object data) {  
        this.data = data;  
    }  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public String getMsg() {  
        return msg;  
    }  
  
    public void setMsg(String msg) {  
        this.msg = msg;  
    }  
  
    @Override  
    public String toString() {  
        return "Result{" +  
                "data=" + data +  
                ", code=" + code +  
                ", msg='" + msg + '\'' +  
                '}';  
    }  
}
```

- 定义状态码Code类  
	状态码也可以根据自己的需求来自定义
```java
public class Code {  
    public static final Integer SAVE_OK = 20011;  
    public static final Integer SAVE_ERR = 20010;  
  
    public static final Integer UPDATE_OK = 20021;  
    public static final Integer UPDATE_ERR = 20020;  
  
    public static final Integer DELETE_OK = 20031;  
    public static final Integer DELETE_ERR = 20030;  
  
    public static final Integer GET_OK = 20041;  
    public static final Integer GET_ERR = 20040;  
  
    public static final Integer SYSTEM_ERR = 50001;  
    public static final Integer SYSTEM_TIMEOUT_ERR = 50002;  
    public static final Integer SYSTEM_UNKNOW_ERR = 59999;  
  
    public static final Integer BUSINESS_ERR = 60001;  
}
```

- 修改Controller类的返回值
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private BookService bookService;  
  
    @GetMapping("/{id}")  
    public Result getById(@PathVariable Integer id) {  
        Book book = bookService.getById(id);  
        Integer code = book == null ? Code.GET_ERR : Code.GET_OK;  
        String msg = book == null ? "数据查询失败，请重试！" : "";  
        return new Result(code, book, msg);  
    }  
  
    @GetMapping  
    public Result getAll() {  
        List<Book> books = bookService.getAll();  
        Integer code = books == null ? Code.GET_ERR : Code.GET_OK;  
        String msg = books == null ? "数据查询失败，请重试！" : "";  
        return new Result(code, books, msg);  
    }  
  
    @PostMapping  
    public Result save(@RequestBody Book book) {  
        boolean flag = bookService.save(book);  
        return new Result(flag ? Code.SAVE_OK : Code.SAVE_ERR, flag);  
    }  
  
    @PutMapping  
    public Result update(@RequestBody Book book) {  
        boolean flag = bookService.update(book);  
        return new Result(flag ? Code.UPDATE_OK : Code.UPDATE_ERR, flag);  
    }  
  
    @DeleteMapping("/{id}")  
    public Result delete(@PathVariable Integer id) {  
        boolean flag = bookService.delete(id);  
        return new Result(flag ? Code.DELETE_OK : Code.DELETE_ERR, flag);  
    }  
}
```

- 如果不放心自己改的对不对，可以继续用PostMan再将每个方法测试一遍

### 7.5 统一异常处理

- 将异常进行分类  
	这里只将其划分为了业务异常和系统异常  
	在com.blog.exception包下新建两个异常类
```java
public class BusinessException extends RuntimeException {  
    private Integer code;  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public BusinessException(Integer code) {  
        super();  
        this.code = code;  
    }  
  
    public BusinessException(Integer code, String message) {  
        super(message);  
        this.code = code;  
    }  
  
    public BusinessException(Integer code, String message, Throwable cause) {  
        super(message, cause);  
        this.code = code;  
    }  
}

public class SystemException extends RuntimeException {  
    private Integer code;  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public SystemException(Integer code) {  
        this.code = code;  
    }  
  
    public SystemException(Integer code, String message) {  
        super(message);  
        this.code = code;  
    }  
  
    public SystemException(Integer code, String message, Throwable cause) {  
        super(message, cause);  
        this.code = code;  
    }  
}
```

- 同时再增加几个状态码
```java
public static final Integer SYSTEM_ERR = 50001;  
public static final Integer SYSTEM_TIMEOUT_ERR = 50002;  
public static final Integer SYSTEM_UNKNOW_ERR = 59999;  
public static final Integer BUSINESS_ERR = 60001;
```

- 编写自定义异常处理类
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
  
    //处理系统异常  
    @ExceptionHandler(SystemException.class)  
    public Result doSystemException(SystemException exception){  
        return new Result(exception.getCode(),null,exception.getMessage());  
    }  
  
    //处理业务异常  
    @ExceptionHandler(BusinessException.class)  
    public Result doBusinessException(BusinessException exception){  
        return new Result(exception.getCode(),null,exception.getMessage());  
    }  
  
    //处理未知异常  
    @ExceptionHandler(Exception.class)  
    public Result doException(Exception exception){  
        return new Result(Code.SYSTEM_UNKNOW_ERR,null,"系统繁忙，请稍后再试");  
    }  
}
```

- 测试异常  
	可以getById方法中来进行测试，当id为1时，错误码为`BUSINESS_ERR`，错误信息为`不让你瞅`，当查询其他id时，均为`SYSTEM_UNKNOW_ERR`，错误提示信息为`服务器访问超时，请稍后再试`
```java
@Service  
public class BookServiceImpl implements BookService {  
  
    @Autowired  
    private BookDao bookDao;  
  
    @Override  
    public boolean save(Book book) {  
        int cnt = bookDao.save(book);  
        return cnt > 0;  
    }  
  
    @Override  
    public boolean update(Book book) {  
        int cnt = bookDao.update(book);  
        return cnt > 0;  
    }  
  
    @Override  
    public boolean delete(Integer id) {  
        return bookDao.delete(id) > 0;  
    }  
  
    @Override  
    public Book getById(Integer id) {  
        if (id == 1){  
            throw new BusinessException(Code.BUSINESS_ERR,"不让你瞅");  
        }  
        try {  
            int a = 1 / 0;  
        } catch (Exception e) {  
            throw new SystemException(Code.SYSTEM_UNKNOW_ERR, "服务器访问超时，请稍后再试");  
        }  
        return bookDao.getById(id);  
    }  
  
    @Override  
    public List<Book> getAll() {  
        return bookDao.getAll();  
    }  
}
```
### 7.6 前端内容

前端内容并没有变化，有空再回来重新写一遍页面吧