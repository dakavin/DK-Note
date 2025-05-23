
在构建任何应用程序时，良好的日志管理都是必不可少的。**日志可以帮助我们监控、调试和跟踪代码的运行情况。** 在本小节中，小哈将介绍如何在 Spring Boot 中整合 Logback 这个流行的日志管理框架。
## 1 什么是 Logback？

Logback 是日志框架 SLF4J 的一个实现，它被设计用来替代 `log4j`。Logback 提供了更高的性能，更丰富的日志功能和更好的配置选项。

## 2 为什么要用它？

**在 Spring Boot 中，Logback 是默认的日志实现**，至于官方为何用它作为默认日志组件，有以下几个原因：

- 1. **性能**：Logback 在性能上超越了许多其他的日志实现，==尤其是在高并发环境下==。

- 2. **灵活性**：Logback 提供了高度灵活的日志配置方式，支持从 XML、Groovy 以及编程式的方式进行配置。

- 3. **功能丰富**：除了基本的日志功能，Logback 还提供了如日志归档、日志级别动态修改、事件监听等高级功能。

- 4. **与 SLF4J 集成**：SLF4J 是一个日志门面（facade），使得应用程序可以在运行时更换日志实现。Logback 作为 SLF4J 的一个原生实现，可以无缝地与其集成。

- 5. **与 Spring Boot 的自动配置集成**：Spring Boot 提供了对 Logback 的自动配置，这意味着开发者无需手动配置 Logback，只需提供一个简单的配置文件即可。

## 3 引入依赖

由于 **Spring Boot 默认使用 Logback**，所以当你在 `pom.xml` 中加入 `spring-boot-starter-web` 依赖时，它会自动包含 Logback 相关依赖，无需额外添加：
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>
```

## 4 自定义 Logback配置

为了满足特定的日志需求，我们通常会自定义 Logback 配置。这里需要注意，配置文件我们统一放置在 `blog-web` 模块中，方便统一管理。然后在 `src/main/resources` 目录下，创建一个名为 `logback-blog.xml` 的文件。

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e87b1f3afab07e1ca509039bd9bb99ef.png)

内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration >  
    <jmxConfigurator/>  
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />  
  
    <!-- 应用名称，scope表示这个属性的作用域是上下文（全局） -->  
    <property scope="context" name="appName" value="blog" />  
    <!-- 指定日志输出路径、日志名称前缀、${appName}为引用vlog这个值-->  
    <property name="LOG_FILE" value="/app/blog/logs/${appName}.%d{yyyy-MM-dd}"/>  
    <!-- 指定日志文件的格式：格式化输出：%d 表示日期，%thread 表示线程名，%-5level：级别从左显示 5 个字符宽度 %errorMessage：日志消息，%n 是换行符-->  
    <property name="FILE_LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>  
    <!--<property name="CONSOLE_LOG_PATTERN" value="${FILE_LOG_PATTERN}"/>-->  
  
    <!-- 按照每天生成日志文件 --> 
    <!--  “appender” 是指将日志消息发送到指定目的的组件或插件 -->
    <!-- class属性决定appender输出的位置是文件 -->  
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!-- 日志文件输出的文件名 `%i` 表示滚动编号 -->  
            <FileNamePattern>${LOG_FILE}-%i.log</FileNamePattern>  
            <!-- 日志文件保留天数 -->  
            <MaxHistory>30</MaxHistory>  
            <!-- 定义了文件命名和触发策略，该类表示同时基于文件大小和时间触发滚动 -->  
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
	            <!-- 日志文件最大的大小，达到这个大小时会触发滚动 --> 
                <maxFileSize>10MB</maxFileSize>  
            </TimeBasedFileNamingAndTriggeringPolicy>  
        </rollingPolicy> 
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <!-- 指定了日志的输出格式，使用了之前定义的FILE_LOG_PATTERN属性 -->  
            <pattern>${FILE_LOG_PATTERN}</pattern>  
        </encoder>  
    </appender>  
  
    <!-- dev 环境（仅输出到控制台） -->  
    <springProfile name="dev"> 
	    <!-- 这个文件会配置控制台输出的日志格式、颜色等 -->
        <include resource="org/springframework/boot/logging/logback/console-appender.xml" />  
        <!-- 意味着只有 INFO 级别及以上的日志消息会记录 -->
        <!-- 只有 INFO、WARN、ERROR、FATAL 等级别的消息会被输出 -->
        <!-- 而 DEBUG 和 TRACE 等级别的消息将被忽略 -->
        <root level="INFO">  
	        <!-- 将名为 “CONSOLE” 的 appender 关联到根日志记录器 -->
	        <!-- 根日志记录器的日志消息将会被发送到控制台输出 -->
            <appender-ref ref="CONSOLE" />  
        </root>  
    </springProfile>  
  
    <!-- prod 环境（仅输出到文件中） -->  
    <springProfile name="prod">  
        <include resource="org/springframework/boot/logging/logback/console-appender.xml" />  
        <root level="INFO">  
            <appender-ref ref="FILE" />  
        </root>  
    </springProfile>  
</configuration>
```

因为打印日志到文件只需要在生产环境开启就行了，所以，使日志生效的配置放到 `application-prod.yml` 文件中就行了：
```yml
# ================================================================= 
# log 日志  
#================================================================== 
logging:  
    config: classpath:logback-blog.xml
```
## 5 打印日志

为了测试一下日志是否能够正常打印，我们在单元测试包下的 `WeblogWebApplicationTests` 类中：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f27b88695b59d3970c53d01200398e1.png)

新建一个 `testLog()` 测试方法，同时添加 Lombok 的 `@Slf4j` 日志注解，它可以帮助我们自动的生成日志实例，示例代码如下：
```java
@SpringBootTest  
@Slf4j  
class BlogWebApplicationTests {      
    @Test  
    public void testLog(){  
        log.info("这是一行 info 级别的日志");  
        log.warn("这是一行 warn 级别的日志");  
        log.error("这是一行 error 级别的日志");  
          
        // 占位符测试  
        String author = "dakkk";  
        log.info("这是一行带有占位符日志，作者:{}",author);  
    }  
}
```

### 5.1 控制台日志

运行该测试方法，若控制台日志输出如下，表示日志组件运行正常运行，在后续的功能开发中，我们==会频繁地使用到它==：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d25952d7c9cfca8011964d826f4c0b2f.png)

### 5.2 日志输出到文件

接下来，我们再验证一下生产环境是否能够使文件输出到文件中。先将 `application.yml` 的环境改为 `prod` ，一旦激活 `prod`, 则日志将被输出到文件中:
```yml
spring:  
    profiles:  
        # 默认激活 dev 环境  
        active: prod
```

用的 Windows 系统做的演示，还需要修改 `logback-weblog.xml` , 将日志输出路径自定义为 Windows 路径：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0aeea2ee31dbb86850860f9fa7ef9953.png)

> 测试完成后，记得将 `profile` 重新改回 `dev` 环境，以及路径改回 Linux 格式路径。

重新运行该单元测试，查看该路径下是否正确输出日志文件：

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8fdd4b330eae1b2d8169fee73796e151.png)

可以看到日志文件输出正常。至此，Spring Boot 整合 Logback 日志就算搞定啦~