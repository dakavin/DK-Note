---
文章标题: "[[13_📕自定义 Jackson 序列化、反序列化，支持 Java 8 日期新特性]]" 
文章作者: Dakkk
文章概要: |
  文章指导如何配置Spring Boot的Jackson，使其支持Java 8新的日期时间API（如`LocalDateTime`）的序列化与反序列化。通过自定义`ObjectMapper`和`JavaTimeModule`，有效解决了旧日期API的痛点，提供了详细配置和测试案例。
文章标签:
- "Jackson"
- "Java 8 Date API"
- "序列化"
- "反序列化"
- "Spring Boot"
- "LocalDateTime"
- "ObjectMapper"
- "JavaTimeModule"
相关文章:
- "[[6_📕Spring Boot 自定义注解，实现 API 请求日志切面]]"
- "[[9_📕Spring Boot 自定义响应工具类]]"
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_idea创建不了spring2.X版本，无法使用JDK8]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/13_📕自定义 Jackson 序列化、反序列化，支持 Java 8 日期新特性.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:37
修改时间: 2025-05-27 00:52:55
---

Java 8 中，引入了新的时间和日期 API，相比较老旧的 `Date` , 它使用起来更加便捷，用过的都懂。本小节中，将带着大家自定义 Spring Boot 内置的 JSON 框架 Jackson，让出入参支持序列化、反序列化 Java 8 中新的日期 API : `LocalDateTime`、`LocalDate`、`LocalTime`, 废话不多说，开整~
## 1 为什么要用Java8新的日期API

假设有个需求，需要对当前日期进行 `加 7 天` 操作，我们来看看使用 `Date` 要如何计算，示例代码如下：
```java
@Test  
public void testDate(){  
    Date today = new Date();  
    System.out.println("今天的日期：" + today);  
  
    Calendar calendar = Calendar.getInstance();  
    calendar.setTime(today);  
  
    calendar.add(Calendar.DATE,7);  
  
    Date sevenDaysLater = calendar.getTime();  
  
    System.out.println("7天后的日期：" + sevenDaysLater);  
}
```

再来看看使用新的日期 `LocalDate` 完成同样功能的示例代码：
```java
@Test  
public void testNewDate(){  
    LocalDate today = LocalDate.now();  
    System.out.println("今天的日期：" + today);  
  
    LocalDate sevenDaysLater = today.plusDays(7L);  
    System.out.println("7天后的日期：" + sevenDaysLater);  
}
```

看完两者之间的对比，你就应该知道，为什么要推荐使用 Java 8 新的日期 API 了。
## 2 自定义Jackson配置类

由于 Spring Boot 内置使用的就是 `Jackson` JSON 框架，所以，无需引入新的依赖，仅需添加自定义配置类即可，让其支持新的日期 `API`。

在 `weblog-module-common` 模块中，新建 `config` 配置包，并创建 `JacksonConfig` 配置类，代码如下：
```java
@Configuration
public class JacksonConfig {  
    @Bean  
    public ObjectMapper objectMapper(){  
        // 初始化一个 ObjectMapper对象，用于自定义 Jackson 的行为  
        ObjectMapper objectMapper = new ObjectMapper();  
  
        // 忽略未知字段（前端有传入某个字段，但是后端未定义接受该字段值，则一律忽略掉）  
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,false);  
  
        // JavaTimeModule 用于指定时间相关的序列化和反序列化规则  
        JavaTimeModule javaTimeModule = new JavaTimeModule();  
  
        // 支持 LocalDateTime、LocalDate、LocalTime  
        javaTimeModule.addSerializer(LocalDateTime.class,new LocalDateTimeSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));  
        javaTimeModule.addDeserializer(LocalDateTime.class,new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));  
        javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd")));  
        javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd")));  
        javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern("HH:mm:ss")));  
        javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern("HH:mm:ss")));  
          
        objectMapper.registerModule(javaTimeModule);  
          
        // 设置时区  
        objectMapper.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));  
          
        // 设置凡是为 null 的字段，返参中均不返回，根据项目需要确定是否开启  
        // objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);  
        return objectMapper;  
    }  
}
```

上述代码中，我们定义了一个 `ObjectMapper` Bean，它用于自定义 `Jackson` 的行为，然后，通过 `JavaTimeModule` 添加了 `LocalDateTime、LocalDate、LocalTime` 的序列化、反序列化规则，搞定这些后，再设置时区、以及序列化中是否包含值为 `null` 的字段。仅仅数行代码，我们就支持了新的日期 `API`， 是不是很简单。
## 3 清除applicaiton.yml中无用的jackson配置

之前在搭建 `weblog` 多模块项目时，有针对 `jackson` 添加了一些配置，如今已经不再需要。后面，我们将统一使用新日期 `API`。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2b66b230ff21eae64f52d93bd51375ec.png)

## 4 测试一波

接下来，我们来测试一下是否真的支持了新的日期 `API`。在 `User` 对象中，添加如下三个字段，用以测试三种类型的日期类是否能够正常序列化、反序列化：
```java
// 创建时间  
private LocalDateTime createTime;  
// 更新日期  
private LocalDate updateTime;  
// 时间  
private LocalTime time;
```

修改 `TestController` 中的 `/test` 接口：
```java
public Response test(@RequestBody @Validated User user) {  
    // 打印入参  
    log.info(JsonUtil.toJsonString(user));  
  
    // 设置三种日期字段值  
    user.setCreateTime(LocalDateTime.now());  
    user.setUpdateTime(LocalDate.now());  
    user.setTime(LocalTime.now());  
  
    return Response.success(user);  
}
```

重启项目，请求 `/test` 接口，看下接口是否能够正常被请求：
```json
{
  "age": 20,
  "createTime": "2024-04-18 14:13:23",
  "email": "21654@qq.com",
  "sex": 1,
  "time": "14:13:23",
  "updateTime": "2024-04-18",
  "username": "zhangsan"
}
```

我们设置了类型为新日期的三个字段的值，若自定义 `Jackson` 生效了，则能够被正常序列化，则日志中会正常打印 `user` 对象参数：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cc8e50a22862bc502317d3d375f5b358.png)

再来看看出参，也是正常的：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/160fea7e5e97a2958532a01dfb78a865.png)

OK, 到这里，让 `Jackson` 支持 `LocalDateTime` 等新的日期 `API` 就完成了。
## 5 结语

本小节中，带着大家自定义了 Jackson 序列化、反序列化，让其支持了 Java 8 新的日期 API , 这些日期新特性相比较 `Date`, 使用起来会更加方便，在后面小节的学习中，你就能切身体会到了。