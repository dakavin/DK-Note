
## 1 前言

在后端业务中，`对每次请求的入参、被请求类、方法，以及出参、执行耗时等信息进行日志打印，是很有必要的`，有了这些信息，当某个接口出现问题时，可以帮助我们快速完成问题的追踪。那么，Spring Boot 中要如何实现呢? 本小节中，就带着大家通过自定义注解和切面编程（AOP）的方法，轻松实现此功能。
## 2 什么是自定义注解（Custom Annotations）？

Java 注解是从 Java 5 开始引入的，它为我们提供了一种元编程的方法，允许我们在不改变代码逻辑的情况下**为代码添加元数据**。**这些元数据可以在编译时或运行时通过反射被访问**。

自定义注解就是用户定义的，用于为代码提供元数据的注解。例如，本小节中自定义的 `@ApiOperationLog` 注解，它用来表示一个方法在执行时需要被记录日志。
## 3 什么是AOP（面向切面编程）？

AOP（Aspect-Oriented Programming，面向切面编程）是一个编程范式，它提供了一种能力，让开发者能够模块化跨多个对象的横切关注点（例如日志、事务管理、安全等）。

主要概念包括：
- **切点 (Pointcuts)**: 定义在哪里应用切面（即在哪里插入横切关注点的代码）。
- **通知 (Advices)**: 定义在特定切点上要执行的代码。常见的通知类型有：前置通知、后置通知、环绕通知等。
- **切面 (Aspects)**: 切面将切点和通知结合起来，定义了在何处和何时应用特定的逻辑。

例如，使用AOP，我们可以为所有使用 `@ApiOperationLog` 注解的方法自动添加日志逻辑，而不需要在每个方法中手动添加。
## 4 实战

### 4.1 添加依赖

在父项目 `blog-springboot` 中的 `pom.xml` 文件中，添加 `jackson` 工具，它用于将出入参转为 `json` 字符串：
```xml
<jackson.version>2.15.2</jackson.version>

<!-- Jackson -->  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>${jackson.version}</version>  
</dependency>
```

因为日志切面属于前台、后台管理接口通用的功能，所以和该功能相关代码可以统一放置于 `blog-module-common` 模块中。

打开 `blog-module-common` 模块中的 `pom.xml` , 引用具体依赖：
```xml
<!-- AOP 切面 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-aop</artifactId>  
</dependency>  
  
<!-- Jackson -->  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
</dependency>
```

### 4.2 自定义注解

在 `blog-module-common` 通用模块下，新建一个名为 `aspect` 的包，用于放置切面相关的功能类，接着，在其中创建一个名为 `ApiOperationLog` 的自定义注解：
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.METHOD})  
@Documented  
public @interface ApiOperationLog {  
    /**  
     * API功能描述  
     * @return 返回API功能描述  
     */  
    String description() default "";  
}
```

元注解说明：
- `@Retention(RetentionPolicy.RUNTIME)`： 这个元注解用于指定注解的保留策略，即注解在何时生效。`RetentionPolicy.RUNTIME` 表示该注解将在运行时保留，这意味着它可以通过反射在运行时被访问和解析。
- `@Target({ElementType.METHOD})`： 这个元注解用于指定注解的目标元素，即可以在哪些地方使用这个注解。`ElementType.METHOD` 表示该注解只能用于方法上。这意味着您只能在方法上使用这个特定的注解。
- `@Documented`： 这个元注解用于指定被注解的元素是否会出现在生成的Java文档中。如果一个注解使用了 `@Documented`，那么在生成文档时，被注解的元素及其注解信息会被包含在文档中。这可以帮助文档生成工具（如 JavaDoc）在生成文档时展示关于注解的信息。
### 4.3 日志切面

**aspectj 注解复习**

在配置 AOP 切面之前，我们需要了解下 `aspectj` 相关注解的作用：
- **@Aspect**：声明该类为一个切面类；
- **@Pointcut**：定义一个切点，后面跟随一个表达式，表达式可以定义为切某个注解，也可以切某个 package 下的方法；

切点定义好后，就是围绕这个切点做文章了：
- **@Before**: 在切点之前，织入相关代码；
- **@After**: 在切点之后，织入相关代码;
- **@AfterReturning**: 在切点返回内容后，织入相关代码，一般用于对返回值做些加工处理的场景；
- **@AfterThrowing**: 用来处理当织入的代码抛出异常后的逻辑处理;
- **@Around**: 环绕，可以在切入点前后织入代码，并且可以自由的控制何时执行切点；

---

**创建 JSON 工具类**

在定义日志切面之前，我们先来创建一个 JSON 工具类，这在日志切面中打印出入参为 JSON 字符串会用到。在 `blog-module-common` 通用模块下，创建一个 `utils` 包，用于统一放置工具类相关，然后，新建一个名为 `JsonUtil` 的工具类， 代码如下：
```java
package com.dakkk.blog.common.utils;  
  
import com.fasterxml.jackson.core.JsonProcessingException;  
import com.fasterxml.jackson.databind.ObjectMapper;  
import lombok.extern.slf4j.Slf4j;  
  
/**  
 * ClassName: JsonUtil * Package: com.dakkk.blog.common.utils * * @Create 2024/4/17 4:52 * @Author dakkk * Description: JSON工具类  
 */  
@Slf4j  
public class JsonUtil {  
    /**  
     * jackson主要对象，用于操作对象和json字符串  
     */  
    public static final ObjectMapper INSTANCE = new ObjectMapper();  
  
    /**  
     * 将obj转换为json字符串  
     * @param obj obj对象  
     * @return json字符串  
     */  
    public static String toJsonString(Object obj){  
        try {  
            return INSTANCE.writeValueAsString(obj);  
        } catch (JsonProcessingException e) {  
            return obj.toString();  
        }  
    }  
}
```

上面的代码中，我们使用了 Spring Boot 内置的 JSON 工具`Jackson` ， 同时，创建了一个静态的 `ObjectMapper` 类，并写个一个 `toJsonString` 方法，用于将传入的对象打印成 JSON 字符串。

---

**定义日志切面类**

工具类搞定后，在 `aspect` 包下，新建切面类 `ApiOperationLogAspect` , 代码如下，附有详细注释：
```java
package com.dakkk.blog.common.aspect;  
  
import com.dakkk.blog.common.utils.JsonUtil;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Pointcut;  
import org.aspectj.lang.reflect.MethodSignature;  
import org.slf4j.MDC;  
import org.springframework.stereotype.Component;  
  
import java.lang.reflect.Method;  
import java.util.Arrays;  
import java.util.Locale;  
import java.util.UUID;  
import java.util.function.Function;  
import java.util.stream.Collectors;  
  
@Aspect  
@Component  
@Slf4j  
public class ApiOperationLogAspect {  
    /**  
     * 自定义 @ApiOperationLog 注解为切入点，凡是使用该注解的方法  
     * 都会被设置环绕通知，可以实现其他四种通知类型的作用  
     */  
    @Pointcut("@annotation(com.dakkk.blog.common.aspect.ApiOperationLog)")  
    public void apiOperationLog() {  
    }  
    /**  
     * @param joinPoint  
     * @return  
     * @throws Throwable  
     */  
    @Around("apiOperationLog()")  
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {  
        try {  
            // 请求开始的时间  
            long statrTime = System.currentTimeMillis();  
  
            // MDC  
            MDC.put("traceID", UUID.randomUUID().toString());  
  
            // 获取被请求的类和方法  
            String className = joinPoint.getTarget().getClass().getSimpleName();  
            String methodName = joinPoint.getSignature().getName();  
  
            // 获取请求入参  
            Object[] args = joinPoint.getArgs();  
            // 请求入参转为 JSON 字符串  
            // 将数组args转换为流，然后使用map()方法将每个元素转换为JSON字符串  
            // 最后，使用collect(Collectors.joining(", "))将转换后的字符串用逗号分隔拼接起来  
            String argsJsonStr = Arrays.stream(args).map(toJsonStr()).collect(Collectors.joining(","));  
  
            // 功能描述信息  
            String description = getApiOperationLogDescription(joinPoint);  
  
            // 打印请求相关参数  
            log.info("==== 请求开始：[{}]，入参：{}，请求类：{}，请求方法：{} ====================",description,argsJsonStr,className,methodName);  
              
            // 执行切入点方法，获得出参对象  
            Object result = joinPoint.proceed();  
              
            // 执行耗时  
            long executionTime = System.currentTimeMillis() - statrTime;  
              
            // 打印出参相关信息  
            log.info("==== 请求结束：[{}]，耗时：{} ms，出参：{} ====================",description,executionTime,JsonUtil.toJsonString(result));  
              
            // 返回原方法返回值  
            return result;  
        } finally {  
            MDC.clear();  
        }  
    }  
  
    /**  
     * 获取切入点方法上@ApiOperationLog注解的description属性  
     * @param joinPoint 切入点  
     * @return @ApiOperationLog注解的description属性  
     */  
    private String getApiOperationLogDescription(ProceedingJoinPoint joinPoint){  
        // 从 joinPoint 获取 MethodSignature        MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
  
        // 使用 MethodSignature 获取当前被注解的 Method        Method method = signature.getMethod();  
  
        // 从Method 中提取 ApiOperationLog 注解  
        ApiOperationLog apiOperationLog = method.getAnnotation(ApiOperationLog.class);  
  
        // 从 ApiOperationLog 注解中获取 description 属性  
        return apiOperationLog.description();  
    }  
  
    /**  
     * 返回一个function，x=obj，y=string ： y = f(x)  
     * @return 返回obj的json字符串  
     */  
    private Function<Object,String> toJsonStr(){  
        // return arg -> JsonUtil.toJsonString(arg);  
        return JsonUtil::toJsonString;  
    }  
  
    // private Function<Object,String> toJsonStr(){  
    //     return new Function<Object, String>() {    
    //         @Override    
    //         public String apply(Object arg) {    
    //             return JsonUtil.toJsonString(arg);    
    //         }    
    //     };    
    // }
}
```

功能核心代码完成后，结构如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c9c6168a0c176bfc7c5871e17541174.png)

### 4.4 添加包扫描

在启动类 `WeblogWebApplication` 中，手动添加包扫描 `@ComponentScan` , 指定扫描 `com.quanxiaoha.weblog` 包下面的所有类:
## 5 新增测试接口

下面我们新建一个测试接口，并指定出入参，看看日志切面是否能够正常运行。在 `weblog-module-web` 模块中，新建 `controller` 包用于统一存放接口定义，另外，再定义一个 `model` 模型包，用于放置 pojo 对象：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4e9e0a9af9aa3375189b18ff2470ff5.png)

新增一个 `User` 对象类，代码如下：
```java
@Data  
public class User {  
    private String username;  
    private Integer sex;  
}
```

新增 `TestController` 类，定义一个 POST 格式，路径为 `/test` 接口，代码如下：
```java
@RestController  
@Slf4j  
public class TestController {  
    @PostMapping("/test")  
    @ApiOperationLog(description = "测试接口")  
    public User test(@RequestBody User user){  
        return user;  
    }  
}
```
## 6 测试

接下来，我们重启 `blog` 项目，并使用 Postman 工具以 json 的方式请求 `/test` 接口：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0d8a889477816d0e9c7cc858c6ba47a.png)

请求参数如下：
```json
{
    "username":"zhangsan",
    "sex":20
}
```

查看控制台日志：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d94614a843f057d68f7ff1a421e46f7c.png)

可以看到正确打印了两条日志：
- 请求开始日志：请求接口的描述信息，入参的 Json 数据，被请求类、方法；
- 请求结束日志：请求接口出参 Json 数据，以及执行耗时；
## 7 小结

本小节中，我们对 `blog` 项目添加了一个自定义注解 `@ApiOperationLog` , 以及日志切面用于打印出入参信息等相关数据。它对于当接口发生异常时，可以很方便的帮助我们追踪案发现场的相关信息。