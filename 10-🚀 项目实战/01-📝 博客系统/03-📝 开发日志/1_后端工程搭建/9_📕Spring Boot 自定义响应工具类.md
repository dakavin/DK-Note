
在开发 RESTful API 时，为了保持响应结构的一致性，公司内部一般都有标准化的响应格式。这不仅可以提高代码的可维护性，还可以帮助前端开发者更容易地处理和解析 API 响应。在本小节中，将展示如何在 Spring Boot 中创建一个自定义响应参数工具类，然后使用它进行返参给前端。
## 1 设计响应模型

在前后端分离项目中，前端和后端一般都是以 JSON 格式进行数据交互。而在数据格式设计上，每个公司基本大同小异，习惯这样设计，如下：
### 1.1 接口执行成功返参格式

```json
{
	"success": true,
	"data": null
}
```

- `success` : 是否请求成功，布尔型，`true` 表示接口请求成功，`false` 表示执行失败；
- `data` : 服务端响应数据，对象类型；
### 1.2 接口执行异常返参格式

```java
{
	"success": false,
	"errorCode": "10000",
	"message": "用户名不能为空"
}
```

- `message`: 服务端响应消息，字符串类型，当 `success` 为 `false` 时，此字段才会不为空，用于后端返回失败的原因，方便前端弹出提示消息；
- `errorCode`: 异常码，字符串类型，微服务中用的比较多，通常格式为服务的唯一标识 + 异常码进行返回，如 `QMS100000`, 这样做的好处是，当发生问题时，用于快速锁定是链路上的哪个服务出现了问题。因为本项目是单体项目，其实用处不大，小伙伴们知道有这样的规范就行;
## 2 创建响应参数工具类

确定了格式以后，我们开始着手写响应参数工具类。在 `blog-module-common` 模块下新建 `utils` 包，然后创建 `Response` 工具类：
```java
@Data  
public class Response<T> implements Serializable {  
  
    private static final long serialVersionUID = -4424251335216902416L;  
    // 是否成功，默认为 true    
    private boolean success = true;  
    // 响应消息  
    private String message;  
    // 响应异常时的异常码  
    private String errorCode;  
    // 响应数据  
    private T Data;  
  
    /**  
     * 通用响应成功方法1  
     * @return 没有data的response  
     * @param <T> 用于指定响应中data的类型，此构造函数用不上  
     */  
    public static <T> Response<T> success(){  
        Response<T> response = new Response<>();  
        return response;  
    }  
  
    /**  
     * 通用响应成功方法2  
     * @param data 成功响应的数据  
     * @return 携带data的response  
     * @param <T> 用于指定响应中data的类型  
     */  
    public static <T> Response<T> success(T data){  
        Response<T> response = new Response<>();  
        response.setData(data);  
        return response;  
    }  
  
    /**  
     * 通用响应异常方法1  
     * @param errorMessage 响应异常信息  
     * @return 携带errorMessage的response  
     * @param <T> 用于指定响应中data的类型  
     */  
    public static <T> Response<T> fail(String errorMessage){  
        Response<T> response = new Response<>();  
        response.setSuccess(false);  
        response.setMessage(errorMessage);  
        return response;  
    }  
  
    /**  
     * 通用响应异常方法2  
     * @param errorCode 响应异常码  
     * @param errorMessage 响应异常信息  
     * @return 携带errorCode和errorMessage的response  
     * @param <T> 用于指定响应中data的类型  
     */  
    public static <T> Response<T> fail(String errorCode,String errorMessage){  
        Response<T> response = new Response<>();  
        response.setSuccess(false);  
        response.setErrorCode(errorCode);  
        response.setMessage(errorMessage);  
        return response;  
    }  
}
```

按照之前定义好的 JSON 格式，我们定义了 4 个字段，并添加了 `success()` 和 `fail()` 方法，分别用于快速生成执行成功的响应参数，和执行失败的响应参数，重载了的多个方法，用于适配不同场景。
## 3 在控制器中使用

有了 `Response` 工具类，再配合 Spring Boot 的 `@RestController` 或者 `@ResponseBody` 注解， 就可以快速生成 JSON 格式的响应数据了。重写前面小节中 `TestController` 中 `/test` 接口的返参：
```java
@RestController  
@Slf4j  
public class TestController {  
    @PostMapping("/test")  
    @ApiOperationLog(description = "测试接口")  
    public Response test(@RequestBody @Validated User user, BindingResult bindingResult) {  
        // 是否存在校验错误  
        if (bindingResult.hasErrors()) {  
            // 获取校验不通过字段的提升信息  
            String errorMsg = bindingResult.getFieldErrors()  
                    .stream()  
                    .map(FieldError::getDefaultMessage)  
                    .collect(Collectors.joining(","));  
            return Response.fail(errorMsg);  
        }  
        // 返参  
        return Response.success();  
    }  
}
```
## 4 扩展响应工具类

上面的工具类功能相对基础。随着后续功能的增加，我们还需要添加更多的辅助方法，例如支持传入异常枚举、分页数据等。这些功能，等后面小节中我们再一一实现它们。本小节暂时不做讲解。
## 5 测试看下效果

重启项目，通过 Postman 工具对 `/test` 进行 Post 请求，看下现在的接口是否是以 JSON 格式数据进行返回了。

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dccacb055653e3305635425a3426e85a.png)
## 6 美化：不返回null值的字段

细心的小伙伴应该发现了，在接口的返参中，有很多 `null` 值的字段也返回了，有木有办法自动去掉呢？当然是可以的，只需在 `applicaiton.yml` 文件中对 `jackson` 添加相关配置即可，如下：
```yml
spring:   
    jackson:  
        # 设置后台对于参数为null的值，不返回  
        default-property-inclusion: non_null  
```

开始测试
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/32919e253773b7629e5450c9bd0a8858.png)

可以看到，值为 `null` 的字段全都没有返回了, 简洁很多。要说不返回 `null` 值的字段的好处，可以省一些流量，但是不方便的地方是，前端在获取字段值时，需要先判断字段是否存在，是否开启该功能小伙伴们可自行取舍。

> 提示：`blog` 这个项目中，依然会返回 `null` 值，因为本身这个项目访问量不会很大，更多的是讲究前端开发更方便一点。
## 7 总结

标准化的 API 响应有助于维护一致性，并提高前后端开发的效率。通过创建自定义的响应工具类，我们可以确保应用程序的每个响应都遵循相同的格式，从而提供更好的开发和用户体验。