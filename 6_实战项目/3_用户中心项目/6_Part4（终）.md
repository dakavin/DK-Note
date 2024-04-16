
## 1 用户注销功能

### 1.1 后端

**先写业务层（service）**

**UserService接口的思路**
- 注销成功，向前端返回一个成功的标识，我们先返回一个int类型的数据，方法名叫userLogout
- 那么方法需要接受什么参数呢？记得用户登录成功的时候，我们将用户保存在了session中不？
	- 所以，用户注销的时候，我们将session中的用户移除掉即可，而session是提供HttpServletRequest获取的
	- 所以方法接受的参数为 HttpServletRequest

**UserServiceImpl实现类中重写userLogout方法的思路**
- 通过HttpServletRequest获取到请求对象，然后请求对象获取session
- 通过session的removeAttribute方法将之前保存的用户态（USER_LOGIN_STATE）删除
- 注销成功直接返回1

---

**再写控制层（controller）**

**UserController类中编写userLogout方法的思路**
- 使用POST，url为：/logout
- 方法的参数肯定是HttpServletRequest
- 判断方法的参数是否为空，为空则直接返回null
- 不为空，则调用userService的userLogout方法即可

---

**具体代码如下：**

- **UserService**
```java
/**  
 * 用户注销方法  
 * @param req 获取连接对象  
 * @return 返回 1-注销成功  
 */  
int userLogout(HttpServletRequest req);
```

- **UserServiceImpl**
```java
@Override  
public int userLogout(HttpServletRequest req) {  
    req.getSession().removeAttribute(USER_LOGIN_STATE);  
    return 1;  
}
```

- **UserContoller**
```java
@PostMapping("/logout")  
public Integer userLogout(HttpServletRequest req){  
    if(req==null){  
        return null;  
    }  
    return userService.userLogout(req);  
}
```

### 1.2 前端

此时我们的前端还**没有对接到后端的用户注销接口**，如何实现呢？

**思路：**
- 先到页面触发注销的位置，看看是在哪里写的注销，发现这个注销是写在导航条上的，导航条是页面公有的一个组件，所以导航条应该是在components目录下，然后发现有一个RightContent目录，里面有一个AvatarDropDown头像的下拉菜单
  ![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4409b6148b55f2a88a56177a44651f25.png)
- 很显然我们修改这个tsx文件即可，那我们找一找==文件内那个方法对接后端接口==的吧![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c65e77023e7c214a346bde4efebf4b87.png)
- 我们直接点击，outLogin()，发现来到了退出登录的接口，那我们对这个接口进行修改一下（对接后端接口）
  ![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f68f3377d301ca5e461af4f863d533f3.png)
- 测试一下，刷新页面，然后点击退出登录，成功退出并跳转登录页面，响应值为1
  ![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e46e36df7095092ee9c1ec899f50453a.png)
用户注销完成啦！
## 2 用户注册优化（增加邀请码校验）

之前的用户注册页面，只让用户填了账号密码，我们现在要补充一些逻辑来校验这个用户必须是由邀请码的用户，邀请码我们可以先内置为一个常量数组（INVITE_CODE）中

### 2.1 后端

**数据库补充inviteCode字段**
- 先在用户表中补充一个字段，直接增加一个邀请码 inviteCode
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/876e14e5debf3012ca633acc2aaec88f.png)
- 现在表结构发生变化，我们重新生成一下对象（==尽量不要频繁修改表结构==）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/06ff5ef6aa03d5c4ff3d76ca5033d8de.png)

- 然后把这新生成的User中的星球编号字段复制，粘贴到model.domain包下的User中
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4a9c4b892e9ae40bfc9daa04e6ed48e9.png)

- 把resources目录下的mapper目录中的UserMapper.xml文件修改一下
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dc6ec4a85cb878cd6d89178e8fe8eff2.png)
- 然后删除生成的generator包

---

**编写邀请码代码**

- 修改UserServiceImpl中，用户加密的时候，将邀请码返回
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b970392b8e3620d1f8e36c3f47534d2e.png)
- 给注册请求类UserRegisterRequest补充一个星球编号的参数
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/019c6430e6e27267389ebc7b72692f4d.png)

- 修改UserService接口和UserServiceImpl实现类的注册方法
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/546397f18191ef0a89d2705a8bd51916.png)

- 在constant包中编写邀请码枚举类InviteCodeEnum
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8e583436ce51ab250b4be3f724373c6.png)

- 继续修改UserServiceImpl中的注册方法，添加邀请码校验（是否是合法的邀请码），并将邀请码保存在数据库
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5b1c081e83a436f09f47003c50daa52.png)

- 继续修改UserController控制类中的userRegister方法（注册接口）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/473e8e0cfdb87b130e892c2057b93545.png)


- 修改一下之前的UserServiceTest测试类中的userRegister方法
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce31129146bed47d1c777ba5aebdc553.png)

- 开始测试，发现到最后一步的断言报错，检查原因：发现邀请码判断逻辑出错，修改为下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/85fcf0abe4e0682383254b21b51b7866.png)

- 好啦，后端开发完成，我们继续开发前端
### 2.2 前端

前端注册页面需要补充一个输入框，用于输入邀请码，找到注册页面，然后复制账号框，并修改
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b707545ba22c4fe179163bd726cdbd51.png)

查看注册表单的提交注册方法register，修改前端接收后端的参数模版API.RegisterParams，添加一个邀请码inviteCode
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/69202b438f489b970b2c3de917503bef.png)

修改返回用户的信息中，添加用户的邀请码（typings.d.ts文件），并在查询用户表格中添加邀请码选项（UserManage/index.tsx）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42241500ec1a72edb0b2a16a54cab3d5.png)

测试，注册页面注册，发现无法注册，DEBUG一下，发现**邀请码逻辑出问题了**，如图所示![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30abdee2bdf09fed7904a27a045851ee.png)
修改BUG，代码如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/af30210413f2fbd4884897191c2020b9.png)

继续测试注册页面，此时可以正常注册！

## 3 后端优化

### 3.1 通用返回对象

**为什么要做一个通用返回对象?**
- 如果后端直接返回一个对象/数值给前端，当这个数据出现问题了
	- 后端处理报错，查不到用户
	- 前端不可以区分的话，也不知道为什么会报错
- 例如1：比如说一个测试，返回了有6项列表，假如后台返回一个空数组，前端可能也不会意识到它是错误的
- 例如2：如果说后台因为一些异常，我们强制给它返回空数组，当前端不知道对不对，可能前端会认为这本来就是后台传过来的一个值
- 所以我们==需要定义一个前后端通用的对象==

---

**开始创建全局通用返回对象**
- 后端新建一个common包（在这个包内存放一些公用的类）

- 在这个包下创建一个BaseResponse类，类中的具体代码如下
```java
/*
 * Description: 通用返回类
 */ 
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class BaseResponse<T> implements Serializable {  
    private static final long serialVersionUID = 3465127617799476957L;  
    private int code;  
    private T data;  
    private String msg;  
  
    public BaseResponse(int code, T data) {  
        this(code,data,"");  
    }  
}
```

- 在这个包下创建一个工具列ResultUtil，类中具体的代码如下
```java
/*
 * Description: 返回工具类  
 */  
public class ResultUtil {  
    public static <T> BaseResponse<T> success(T data){  
        return new BaseResponse<>(0,data,"ok");  
    }  
}
```

- 将UserController中所有方法，返回成功对象，进行修改
	- usesrRegister注册方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cca635682cc71ce9ccc0e6786699ab12.png)
	- getCurrentUser获取当前用户方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1d3bb8a3774954853507454896f5142.png)
	- userLogin用户登录方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/efd2152338ce9916701e8f78fd046909.png)
	- searchUsers用户搜索方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e27ff87e25e4dd58f5bcab0380e0c93a.png)
	- deleteUser用户删除方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bcafb2bc98487cae6073306fb15450d7.png)
	- userLogout用户注销方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f065ab452326a0f7fc902bc8473c0a8f.png)
	- isAdmin用户鉴权方法，不是对外的方法，不需要统一包装

---

我们上面已经完成了成功的返回结果包装，那么对于失败/异常的结果如何包装呢，我们往下看

**自定义错误码**
- 写一个枚举值，在common包下创建ErrorCode异常代码枚举类
```java
/*
 * Description: 错误码  
 */  
public enum ErrorCode {  
  
    SUCCESS(0,"ok",""),  
    PARAMS_ERROR(40000,"请求参数错误",""),  
    NULL_ERROR(40001,"请求数据为空",""),  
    NOT_LOGIN(40002,"未登录",""),  
    NO_AUTH(40003,"无权限","");  
  
    private final int code;  
    private final String message;  
    private final String description;  
  
    ErrorCode(int code, String message, String description) {  
        this.code = code;  
        this.message = message;  
        this.description = description;  
    } 
    public int getCode() {  
    return code;  
}  
  
	public String getMessage() {  
	    return message;  
	}  
  
	public String getDescription() {  
	    return description;  
	}
}
```

- 修改BaseResponse通用全局返回类
```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class BaseResponse<T> implements Serializable {  
    private static final long serialVersionUID = 3465127617799476957L;
    // code表示业务层面的错误码，不是HTTP的响应状态码  
    private int code;  
    private T data;  
    private String message;  
    private String description;  
  
    public BaseResponse(int code, T data) {  
        this(code,data,"","");  
    }  
  
    public BaseResponse(int code, T data, String message) {  
        this(code,data,message,"");  
    }  
    public BaseResponse(ErrorCode errorCode){  
        this(errorCode.getCode(),null,errorCode.getMessage(),errorCode.getDescription());  
    }  
}
```

- 完善ResultUtil全局返回结果工具类
```java
public class ResultUtil {  
    /**  
     * 成功  
     * @param data  
     * @return  
     * @param <T>  
     */  
    public static <T> BaseResponse<T> success(T data){  
        return new BaseResponse<>(0,data,"ok");  
    }  
  
    /**  
     * 失败  
     * @param errorCode  
     * @return  
     */    
     public static BaseResponse error(ErrorCode errorCode){  
        return new BaseResponse(errorCode);  
    }  
}
```

如果这样的话，UserController类就有很多接口方法判断错误的时候，就使用`ResultUtil.error()`的方法，同理，UserServiceImpl也是这样的

一个方法可能判断多次错误，都使用这个方法的话不太优雅，提示也比较别扭，而且每次都要去调这个错误返回类

**接下来我们封装全局异常处理，就是我们把所有错误放在一个地方（全局异常类）去处理**

### 3.2 全局异常类

**先在代码目录下创建一个exception包，在包中创建BusinessException自定义异常类**
- 继承RuntimeException，这样就不需要try-catch捕获
- 添加一些我们的字段（相比于RE就支持更多字段了）
- 自定义构造函数（更灵活/快捷的设置字段）

代码如下：
```java
/**  
* Description: 自定义异常类  
*/  
public class BusinessException extends RuntimeException {  
    private final int code;  
    private final String description;  
  
    public BusinessException(String message, int code, String description) {  
        super(message);  
        this.code = code;  
        this.description = description;  
    }  
  
    public BusinessException(ErrorCode errorCode) {  
        super(errorCode.getMessage());  
        this.code = errorCode.getCode();  
        this.description=errorCode.getDescription();  
    }  
  
    public BusinessException(ErrorCode errorCode,String description) {  
        super(errorCode.getMessage());  
        this.code = errorCode.getCode();  
        this.description=description;  
    }  
  
    public int getCode() {  
        return code;  
    }  
  
    public String getDescription() {  
        return description;  
    }  
}
```

---

**优化UserController类和UserServiceImpl类中的代码**
- 把所有的if，返回结果修改成
- `throw new BusinessException(对应的错误代码，对应的错误描述等);`

- 如果这个时候我们测试注册，会发现==前端接收到的是500响应码，表示后台抛出异常了==![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9898986c0b2dbb6228808324eb80f8d.png)
- 所以我们需要一个异常捕获处理器，==不让500响应给前端，而应该响应我们之前的状态码==

### 3.3 全局异常处理器

**介绍**
- 捕获代码中所有异常，并且集中进行处理

---

**创建全局异常处理器**
- 在Exception包下，创建一个GlobalExceptionHandler处理类
- 再去ErrorCode中添加一个系统内部异常的状态码
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/280f3b2506f2fab7f97140602da5f4a8.png)
- 在ResultUtils工具类中，再写三个构造函数
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33e9bd63f640298efc72d48c93322a4c.png)
- GlobalExceptionHandler处理类代码如下：
```java
@Slf4j  
@RestControllerAdvice  
public class GlobalExceptionHandler {  
    @ExceptionHandler(BusinessException.class)  
    public BaseResponse businessExceptionHandler(BusinessException e){  
        log.error("businessException:" + e.getMessage(),e);  
        return ResultUtils.error(e.getCode(),e.getMessage(),e.getDescription());  
    }  
  
    @ExceptionHandler(RuntimeException.class)  
    public BaseResponse runtimeExceptionHandler(BusinessException e){  
        log.error("runtimeException:" + e.getMessage(),e);  
        return ResultUtils.error(ErrorCode.SYSTEM_ERROR,e.getMessage(),"");  
    }  
}
```

- 重新测试一下，返回正确结果啦
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2d82380af64116580bc3c5c2ca83367.png)
## 4 前端优化

### 4.1 后端➡前端 参数统一

由于我们修改了后端接口的返回对象，所以前端接收的对象也要相对应进行修改，那我们开始吧

**先修改service包下的typing.d.ts文件（演示注册的修改）**
- 在文件中添加 BaseResponse对象（通用的操作）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/247c6fbc4f1fae4e709f3d068ab2fc2d.png)
- 在api.ts文件中涉及和后台请求的方法的响应数据修改（封装）一下（==其他方法也适配一下==）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/760c0e5338fa18dab3e5ecda8cce9041.png)
- 在注册页面对方法的返回值进行获取，来执行后面的逻辑
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/11dff74da7336eedbc56acccd3aaeeaa.png)
**此时发现，如果按照上述过程，之后我们要对每一个页面都需要取data，很麻烦**

### 4.2 尝试改框架的请求响应拦截器（可以跳过）

**所以我们可以设置一个全局拦截器，帮我们把data取出来（`这里可以跳过`）**
- 我们看一下调用每一个方法的时候，都是用request方法，点进去看看
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/31a6147fb20d9e6513ed836257c0464b.png)
- 找到request.ts文件，我们把文件过一遍
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23f1ba645f2d20778a8536a37b97fd46.png)
- 发现不能配置requestConfig，那我们直接写一个responseInterceptor也行，研究一下他的结构如下：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/70c65a74a3582c25c6b191a0088f2008.png)
- 那么我们开始写吧
```ts
requestConfig.responseInterceptors = [  
  async function (response: Response, options: RequestOptionsInit) : Response | Promise<Response>{  
    const data = await response.clone().json();  
    console.log('全局响应拦截器');  
  }  
]
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c88758262c24b5b15e64a8489246a6bb.png)

- 测试一下注册，发现我们写的data就是我们需要的数据
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6b7a1a96bf2cfa5eb46dd0064eb99e62.png)
- 继续修改代码，如果响应的code为0，表示响应成功，响应只取data即可
- 这样的话前端不用做任何修改
```ts
requestConfig.responseInterceptors = [  
  async function (response: Response, options: RequestOptionsInit): Response | Promise<Response> {  
    const result = await response.clone().json();  
    if (result.code === 0){  
      return result.data;  
    }  
  }  
]
```
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f97e438ee00b6c878ecae468bc5e0953.png)
- 修改注册代码的逻辑（因为code=0的时候，我们响应的是data数据）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cda860e7e37ea15fa204fe209f166cf0.png)
- `问题：`request.ts文件属于.umi目录的，而在gitignore中忽略了这个目录，所以这样是不可取的，==即我们在这里面写的代码会被重置为原来的！==

### 4.3 自定义请求响应拦截器

**📕我们自定义一个请求拦截器！覆盖默认的request方法**
- 在src文件夹下新建一个plugins文件夹，在该文件夹中新建globalRequest.ts文件
- 复制代码，[参考网站——Ant Design Pro中Umi-request全局请求响应的处理](https://blog.csdn.net/huantai3334/article/details/116780020)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ee1f53839d088ee6a0a5cf86a9ba10fe.png)
- 删除不需要的代码，剩余代码如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5cfa8fa70263b918dd767d2b9149ae70.png)
- 定义==请求拦截器==，比如说我们每次请求的时候log一下
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14d61c3cbadc6c14ce019ad3967f0e91.png)
- 📕定义==响应拦截器==，比如说，前端请求接收到一个响应后，我们先获取到这个响应的data的值，然后通过这个data值进行判断
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e393e478cc7073196dc68a7862951d0.png)
- 在api.ts文件中引入我们自己写的request文件，而不是umi的
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a26436a213fd0c3d05eb218b153c4304.png)
- 测试一下，注册的时候输入错误的
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a1187c8bcd1fd87120a2941d9d718e95.png)
- 在测试一下，未登录进入管理员界面
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/be81ce5dbb29b5addba9814b13180c83.png)
- ok，成功完成！

---

### 4.4 小结

1. 前端对后端的返回值
	- 成功的时候，只需要data
	- 失败的时候，需要message 和 description

2. 专门创建一个对象BaseResponse来接收后端传过来的返回值，并对前对各个异步方法的返回值用该对象BaseResponse进行包装

3. 对于异步方法内具体的逻辑，我们之前是直接获取后端的返回值的，现在加了一层包装
	- 麻烦一点，就对各个方法内的逻辑重写（根据返回值的各个参数（data、code、message、description）进行逻辑处理）
	- 简单一点，直接使用**全局响应处理**

4. ==全局响应处理==
	- 逻辑：成功的时候，只需要data；失败的时候，需要message 和 description用来描述失败所在
	- 我们之前说过，这个框架底部使用的是umi组件（封装了react）
	- 所以可以参考umi官方文档中，request的使用方式来，自定义一个我们的全局响应
	- 补充：如果是axios，就参考axios官方文档





