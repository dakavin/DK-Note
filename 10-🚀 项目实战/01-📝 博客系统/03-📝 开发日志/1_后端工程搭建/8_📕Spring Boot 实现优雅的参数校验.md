---
文章标题: "[[8_📕Spring Boot 实现优雅的参数校验]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了Spring Boot中如何优雅实现参数校验。从手动校验痛点出发，详细讲解并演示了利用JSR 380 (Bean Validation) 注解进行声明式校验，包括依赖引入、实体类与Controller集成，大幅提升了入参验证的简洁性和可维护性。
文章标签:
- "Spring Boot"
- "参数校验"
- "JSR 380"
- "Bean Validation"
- "Web开发"
- "Java"
- "后端开发"
相关文章:
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_Java学习路线]]"
- "[[3_实战web服务]]"
- "[[0_导论]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/8_📕Spring Boot 实现优雅的参数校验.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:36
修改时间: 2024-04-21 21:50:15
---

## 1 前言

在日常的 Web 开发中，请求参数校验是一个非常基础且重要的环节。通过校验，我们可以确保每次接口请求中，入参的数据是有效、安全且合规的，避免数据库中出现脏数据。
## 2 手动校验参数

原始的手动校验参数代码如下：
```java
@PostMapping("/test")  
@ApiOperationLog(description = "测试接口")  
public ResponseEntity<String> test(@RequestBody User user) {  
    // 参数校验  
    if (user.getUsername() == null || user.getUsername().trim().isEmpty()) {  
        return ResponseEntity.badRequest().body("姓名不能为空");  
    }  
    if (user.getAge() <18 || user.getAge()>100){  
        return ResponseEntity.badRequest().body("年龄必须在18到100之间");  
    }  
    if (user.getEmail()==null||!isValidEmail(user.getEmail())){  
        return ResponseEntity.badRequest().body("邮箱格式不正确");  
    }  
    // 更多的校验  
    // 返参  
    return  ResponseEntity.ok("参数没有任何问题");  
}
```

这还是只有 3 个字段需要校验的情况，如果更多呢？一堆的 `if else` 是不是又臭又长！有没有什么优雅的解决方案呢？
## 3 JSR380参数校验注解

Spring Boot 提供了简洁的方法，让我们能够利用 Java 校验 API (JSR 380) 中定义的注解进行参数校验。JSR 380，也被称为 Bean Validation 2.0，是 Java Bean 验证规范的一个版本。该规范定义了一系列注解，用于验证 Java Bean 对象的属性，确保它们满足某些条件或限制。

以下是 JSR 380 中提供的主要验证注解及其描述：
1. `@NotNull:` 验证对象值不应为 null。
2. `@AssertTrue:` 验证布尔值是否为 true。
3. `@AssertFalse:` 验证布尔值是否为 false。
4. `@Min(value):` 验证数字是否不小于指定的最小值。
5. `@Max(value):` 验证数字是否不大于指定的最大值。
6. `@DecimalMin(value):` 验证数字值（可以是浮点数）是否不小于指定的最小值。
7. `@DecimalMax(value):` 验证数字值（可以是浮点数）是否不大于指定的最大值。
8. `@Positive:` 验证数字值是否为正数。
9. `@PositiveOrZero:` 验证数字值是否为正数或零。
10. `@Negative:` 验证数字值是否为负数。
11. `@NegativeOrZero:` 验证数字值是否为负数或零。
12. `@Size(min, max):` 验证元素（如字符串、集合或数组）的大小是否在给定的最小值和最大值之间。
13. `@Digits(integer, fraction):` 验证数字是否在指定的位数范围内。例如，可以验证一个数字是否有两位整数和三位小数。
14. `@Past:` 验证日期或时间是否在当前时间之前。
15. `@PastOrPresent:` 验证日期或时间是否在当前时间或之前。
16. `@Future:` 验证日期或时间是否在当前时间之后。
17. `@FutureOrPresent:` 验证日期或时间是否在当前时间或之后。
18. `@Pattern(regexp):` 验证字符串是否与给定的正则表达式匹配。
19. `@NotEmpty:` 验证元素（如字符串、集合、Map 或数组）不为 null，并且其大小/长度大于0。
20. `@NotBlank:` 验证字符串不为 null，且至少包含一个非空白字符。
21. `@Email:` 验证字符串是否符合有效的电子邮件格式。

除了上述的标准注解，JSR 380 也支持开发者定义和使用自己的自定义验证注解。此外，这个规范还提供了一系列的APIs和工具，用于执行验证和处理验证结果。大部分现代Java框架（如 Spring 和 Jakarta EE）都与 JSR 380 兼容，并支持其验证功能。
## 4 实战
### 4.1 引入依赖

首先，我们需要在 `weblog-web` 模块中的 `pom.xml` 文件添加参数校验依赖：
```xml
<!-- 引入JSR380 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```
### 4.2 实体类参数校验

为了测试日志切面，我们已经新增了一个用户 `User` 实体类，这次再多添加两个字段，用于测试参数校验，代码如下：
```java
@Data  
public class User {  
    // 用户名  
    @NotBlank(message = "用户名不能为空")  
    private String username;  
  
    //性别  
    @NotNull(message = "性别不能为空")  
    private Integer sex;  
  
    // 年龄  
    @NotNull(message = "年龄不能为空")  
    @Min(value = 18,message = "年龄必须大于等于 18")  
    @Max(value = 100,message = "年龄必须小于等于 100")  
    private Integer age;  
  
    //邮箱  
    @NotBlank(message = "邮箱不能为空")  
    @Email(message = "邮箱格式不正确")  
    private String email;  
}
```

`message` 属性用于指定提示信息；
### 4.3 Controller参数校验

针对每个字段的校验注解添加完成后，还需要在 `controller` 层进行捕获，并将错误信息返回。编辑 `TestController` 类，代码如下：
```java
@RestController  
@Slf4j  
public class TestController {  
    @PostMapping("/test")  
    @ApiOperationLog(description = "测试接口")  
    public ResponseEntity<String> test(@RequestBody @Validated User user,BindingResult bindingResult) {  
        // 是否存在校验错误  
        if (bindingResult.hasErrors()){  
            // 获取校验不通过字段的提升信息  
            String errorMsg = bindingResult.getFieldErrors()  
                    .stream()  
                    .map(FieldError::getDefaultMessage)  
                    .collect(Collectors.joining(","));  
            return ResponseEntity.badRequest().body(errorMsg);  
        }  
        // 返参  
        return  ResponseEntity.ok("参数没有任何问题");  
    }  
}
```

解释一下上面代码中的关键部分：
- 📕   `@Validated`: 告诉 Spring 需要对 `User` 对象执行校验;
- `BindingResult` : 验证的结果对象，其中包含所有验证错误信息；
### 4.4 测试一下效果

使用 Postman 工具请求 `/test` 接口测试一下，请求入参如下：
```json
// 入参正确
{
    "username": "dakkk",
    "sex": 1,
    "age": 32,
    "email": "123124@qq.com"
}

// 入参错误
{
    "username": "",
    "sex": null,
    "age": 120,
    "email": "123124qq.com"
}
```

测试结果如图所示
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4d598e3b165c9d4e13fa2c9ca4462699.png)
## 5 还不够优雅？

虽然实现了以添加注解的方式搞定了参数校验功能，但是 `controller` 层的代码也太不优雅美观了：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf7fe030a37843d78781f78e950ca2b6.png)

总不能每次定义一个接口，都要写上面这么一坨吧！_别急，后面小节中，我们将通过自定义响应工具类 + 全局异常管理来干掉这一坨丑陋的代码，无需再手动返回。_
## 6 结语

本小节中，我们在 `blog` 项目中搞定了 1.0 版本的参数校验功能，但是它还不完美，后面小节中，我们继续优化它，实现企业级的终极解决方案。