---
文章标题: "[[10_📕Spring Boot 实现全局异常管理]]" 
文章作者: Dakkk
文章概要: |
  文章阐述了Spring Boot全局异常管理。通过自定义业务异常、错误码枚举，并结合`@ControllerAdvice`和`@ExceptionHandler`，实现统一捕获和处理业务及系统异常。这确保了接口响应格式一致性，提升了代码整洁性与可维护性。
文章标签:
- "Spring Boot"
- "全局异常处理"
- "ControllerAdvice"
- "ExceptionHandler"
- "自定义异常"
- "错误码"
- "异常管理"
相关文章:
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_idea创建不了spring2.X版本，无法使用JDK8]]"
- "[[1_Java学习路线]]"
- "[[1_Java异常处理通关指南]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/10_📕Spring Boot 实现全局异常管理.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:37
修改时间: 2025-05-27 00:52:55
---

## 1 前言

在开发大型 Web 应用时，统一的异常处理是保持代码整洁和维护性的关键所在。Spring Boot 提供了方便的方法来帮助我们实现全局的异常管理。在本小节中，将带着大家学习如何在 Spring Boot 中实现全局的异常处理。

![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3975bd87ce80d564fff5e81febf72c99.png)
## 2 为什么需要全局异常管理？

以下是一段没有全局异常管理的示例代码：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28b398aaf765d0792fbde3ae1e34cbd4.png)

在没有全局异常管理的情况下，每个控制器或 `service` 服务中可能都会有各种 `try-catch` 代码块来捕获和处理异常。这样会导致以下问题：

- **代码重复**：相同的异常处理逻辑会在多处重复出现。
- **不一致的响应格式**：不同的开发人员可能采用不同的方式来处理同一种异常，从而导致响应格式不统一。
- **增加维护成本**：未来对异常处理逻辑的任何更改都需要在多个地方进行修改。
- **可读性差**：`try-catch` 块使得主要的业务逻辑被异常处理代码包围，这可能会让代码的主要逻辑不够明显，降低代码的可读性。

通过实现全局异常管理，我们可以避免上述问题，确保应用在出现异常时始终有一致和统一的行为。
## 3 自定义业务异常

在企业级应用中，**除了系统异常，很多时候我们还需要处理业务异常**。与系统异常不同，业务异常通常表示一个预期的、合理的错误，例如用户输入的数据无效、所请求的资源不存在等。我们需要区分这些异常，并为它们给调用者提供友好的错误提示。通常来说，比较推荐的做法是，将自定义业务异常整合到全局异常管理中，使其更加统一且易于维护。
### 3.1 自定义一个基础异常接口

首先，在 `blog-module-common` 模块中新建 `exception` 包，用于统一放置和异常相关的代码。然后，创建一个 `BaseExceptionInterface` 基础异常接口，方便后面做拓展，代码如下：
```java
public interface BaseExceptionInterface {  
    /**  
     * 获取业务异常码  
     * @return 业务异常码  
     */  
    String getErrorCode();  
  
    /**  
     * 获取业务异常信息  
     * @return 业务异常信息  
     */  
    String getErrorMessage();  
}
```
### 3.2 自定义错误码枚举

```java
@Getter  
@AllArgsConstructor  
public enum ResponseCodeEnum implements BaseExceptionInterface {  
    // 通用异常状态码  
    SYSTEM_ERROR("10000","出错啦，后台小哥正在努力修复中。。。\uD83E\uDD23"),  
      
    // 业务异常状态码  
    PRODUCT_NOT_FOUND("20000","该业务展示不存在哈\uD83E\uDD2A（测试使用）");  
      
      
    // 异常码  
    private String errorCode;  
    //错误信息  
    private String errorMessage;  
}
```

在上述代码中，我们定义了**异常码**和**错误信息**两个字段，并实现了 `BaseExceptionInterface` 接口，同时，还定义了一个 `SYSTEM_ERROR` 枚举值，它属于系统通用异常，通常是未知的异常，为了对调用者表示友好性，统一提示。

除了通用异常外就是业务异常了, 这里定义了一个供测试使用的 `PRODUCT_NOT_FOUND` 枚举值，表示产品不存在，后面会演示如何使用。
### 3.3 自定义业务异常BizException

然后，创建 `BizException` ， `Biz` 是业务英文 `Business` 的缩写:
```java
@Getter  
@Setter  
public class BizException extends RuntimeException{  
    // 异常码  
    private String errorCode;  
    // 异常信息  
    private String errorMessage;  
      
    public BizException(BaseExceptionInterface baseExceptionInterface){  
        this.errorCode = baseExceptionInterface.getErrorCode();  
        this.errorMessage = baseExceptionInterface.getErrorMessage();  
    }  
}
```

我们让 `BizException` 继承自运行时异常 `RuntimeException`, 同时定义了两个基本字段：

- `errorCode` : 异常码；
- `errorMessage`: 错误信息，用于提供给调用者；

另外，还定义了一个构造器，入参是前面创建的 `BaseExceptionInterface`，通过它可以方便的构造一个业务异常。
## 4 使用@ControllerAdvice

Spring Boot 提供了 `@ControllerAdvice` 注解，它允许我们定义一个全局的异常处理类。这个类可以捕获应用中抛出的所有异常，并根据需要进行处理。
### 4.1 添加依赖

在 `blog-module-common` 模块中的 `pom.xml` 中添加如下依赖，因为 `@ControllerAdvice` 注解在这个依赖中：
```xml
<!-- 使用springweb中的@ControllerAdvide注解 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>
```
### 4.2 拓展响应工具类

在 `Response` 响应工具类中，添加一个新的方法：
```java
/**  
 * 通用响应异常方法3  
 * @param bizException 业务异常  
 * @return 包含业务异常的异常码和异常信息的response对象  
 * @param <T> 指定response对象的data属性的数据类型  
 */  
public static <T> Response<T> fail(BizException bizException){  
    Response<T> response = new Response<>();  
    response.setSuccess(false);  
    response.setErrorCode(bizException.getErrorCode());  
    response.setMessage(bizException.getMessage());  
    return response;  
}
```

可以看到入参是 `BizException` 自定义业务异常，等下在全局异常处理类中会用到它。
### 4.3 创建全局异常处理类

在 `exception` 包下，创建全局异常处理类 `GlobalExceptionHandler` ，代码如下：

```java
@ControllerAdvice  
@Slf4j  
public class GlobalExceptionHandler {  
    /**  
     * 捕获自定义异常  
     * @param request 请求对象  
     * @param e 自定义异常对象  
     * @return Response对象（errorCode和errorMessage来源于自定义异常）  
     */  
    @ExceptionHandler({BizException.class})  
    @ResponseBody  
    public Response<Object> handleBizException(HttpServletRequest request, BizException e) {  
        log.warn("{} request fail , errorCode:{} , errorMessage:{}",request.getRequestURI(),e.getErrorMessage(),e.getErrorMessage());  
        return Response.fail(e);  
    }  
}
```

上述代码中，通过 `@ControllerAdvice` 注解，我们将 `GlobalExceptionHandler` 声明为了全局异常处理类。在其中，定义了一个 `handleBizException()` 方法，并通过 `@ExceptionHandler` 注解指定只捕获 `BizException` 自定义业务异常。然后，打印了相关错误日志，并组合了统一的响应格式返回。
### 4.4 试试好不好使

完成上面工作后，我们在 `TestConroller` 中的 `/test` 接口中，手动抛一个自定义业务异常，看看能不能被处理器捕获，并返回统一的出参格式：
```java
@PostMapping("/test")  
@ApiOperationLog(description = "测试接口")  
public Response test(@RequestBody @Validated User user, BindingResult bindingResult) {  
    // 手动抛异常，入参是前面定义好的异常码枚举，返回统一交给全局处理器搞定  
    throw new BizException(ResponseCodeEnum.PRODUCT_NOT_FOUND);  
}
```

重启项目，用 Postman 请求一下 `/test` 接口，看下效果：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/151d1271b64d81d35672605deb61c32a.png)

这样，在后续的业务层开发中，如果**有相关逻辑校验等功能，需要提供给调用者提示信息的，就可以直接抛异常**就 OK 了，代码层面看上去也更加整洁、好维护。
### 4.5 支持通用异常捕获

在以上的全局异常处理器中，我们只捕获了 `BizException` 业务级的异常，通常来说，还有其他未知类型的运行时异常，可能会发生在程序执行过程中。所以也要捕获一下，给调用者一个友好的提示信息。

对 `Response` 响参工具类拓展一下方法，以支持直接传入异常码枚举：

编辑 `GlobalExceptionHandler` 类，添加如下方法：
```java
/**  
 * 通用响应异常方法4  
 * @param baseExceptionInterface 用于获取其实现类，如ResponseCodeEnum  
 * @return 包含系统异常的异常码和异常信息的response对象  
 * @param <T> 指定response对象的data属性的数据类型  
 */  
public static <T> Response<T> fail(BaseExceptionInterface baseExceptionInterface){  
    Response<T> response = new Response<>();  
    response.setSuccess(false);  
    response.setErrorCode(baseExceptionInterface.getErrorCode());  
    response.setMessage(baseExceptionInterface.getErrorMessage());  
    return response;  
}
```

编辑 `GlobalExceptionHandler` 类，添加如下方法：
```java
/**  
 * 捕获其他异常  
 * @param request 请求对象  
 * @param e 其他异常对象  
 * @return Response对象（errorCode和errorMessage来源于其他异常）  
 */  
@ExceptionHandler({Exception.class})  
@ResponseBody  
public Response<Object> handleOtherException(HttpServletRequest request, Exception e) {  
    log.error("{} request error,",request.getRequestURI(),e);  
    return Response.fail(ResponseCodeEnum.SYSTEM_ERROR);  
}
```

**看看效果**：重写一下 `TestController` 中的 `/test` 接口，主动定义一个运行时异常，分母不能为零：
```java
@PostMapping("/test5")  
@ApiOperationLog(description = "测试Exception异常响应的接口")  
public Response test5(@RequestBody @Validated User user, BindingResult bindingResult){  
    // 手动抛异常，入参是前面定义好的异常码枚举，返回统一交给全局处理器搞定  
    int i = 1/0;  
    return Response.success();  
}
```

重启项目，再次请求 `/test` 接口，看下通用类的运行时异常能不能够被正常捕获，并响应错误信息：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20ddb6549620f56355cefd32b7141628.png)

OK, 正常被捕获了，并返回了 `出错啦，后台小哥正在努力修复中...`的友好提示信息。
## 5 核心代码目录结构

本小节中，`blog` 项目需要完成的核心代码目录结构如下：
![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3975bd87ce80d564fff5e81febf72c99.png)
## 6 最后

至此，一个基本通用的异常处理器的功能就搞定了。全局异常管理不仅仅是捕获和响应异常，它还与整个系统的健壮性和用户体验紧密相关。通过自定义业务异常和全局异常处理，我们可以确保应用在面对各种错误时都能优雅地应对，为用户提供有意义的反馈，同时确保开发团队能够迅速定位并处理问题。

但是，本小节中的全局异常管理器==还有不完美的地方==，还记得前面小节 中的参数校验吗？也是需要手动获取校验错误信息，再手动返回参数，这块的代码比较通用，一坨一坨的，每次都要手动写未免太难看了。==能不能在全局异常中统一处理一下呢？==答案是肯定的，下小节中，我们将来支持一下这个功能。