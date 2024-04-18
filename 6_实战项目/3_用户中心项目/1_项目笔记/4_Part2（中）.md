
## 1 数据库表设计

**什么是设计数据库表？**
1. 有哪些表？
2. 有那些字段？
3. 字段的类型？
4. 字段是否需要索引？
5. 表与表之间是否存在关联？

**设计用户表**
1. id：用户ID、主键（非空且唯一）、bitint、主键默认是主键索引
2. userName：用户名、varchar
3. account：登录账号
4. avatarUrl：头像、varchar
5. gender：性别、tinyint
	- ==性别是否需要加索引？==
	- 性别加上索引的话，也是普通索引（非聚簇索引），重复字段表较多，查询起来效率也不高
	- 用技术的话，就是区分度（散列性）不高，可以使用MySQL语句`select count(dintinct gender)/count(*) from user`来计算区分度，一般超过33%才算是比较高效的索引
6. userPassword：密码、varchar
7. phone：电话、varchar
8. email：邮箱、varchar
9. userStatus：是否有效（0-正常 1-）、tinyint
10. createTime：创建时间（数据插入）、datetime
11. updateTime：更新时间（数据更新）、datetime
12. isDelete：是否删除（逻辑删除 0 1）、tinyint

**使用IDEA中的工具创建user表**
- 连接数据库之后，在上方的`jump to query console`进入mysql语句编写界面
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/501253000a6aeb13bf2e73d9dee34ae6.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/86d6399bb01df1a3fecc0983f0775502.png)


- 或者直接使用图形化工具来创建表
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4266082a17b9621670430330f17fa6ae.png)


## 2 测试表和Mybatis-plus

**代码生成器的使用**
- 使用插件`mybatisX`
  ![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/44b268dce7102f3c33c139f15998fb96.png)

- 右键数据库的表格，使用mybatisX一键生成代码 
  ![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b41982e705138a89de1c854c5dded052.png)
- 第一个窗口的选择
  ![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aabf80e3aa85113745ec1ec2f5c0b98c.png)
- 第二个窗口的选择
  ![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/deae02996a06e7e35b14d113c9834b83.png)

- 如果生成的代码没有问题，将generator中的代码复制到对应的文件夹中即可！

- **注意：mp可以自定义逻辑删除字段，在domain的类对应的属性使用@TableLogic注解即可**
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a027f1979e1df829f7d941ee5d81761a.png)

**小demo测试：**
- ==编写测试类的时候，测试那个类，就需要个java中对应的包名一致==

- 在对应的userServcie，按下`alt+enter`
![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6770ae75db73d5dbe9d76eb6659694c2.png)

- 看个人习惯选择 junit4 还是 junit5 ， junit4的断言方法会比较多一点
![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88f67ec5e03cc4e12ed4c47021947c59.png)

- 点击ok后，就会在test目录下，创建对应的测试类
	- 这个测试类需要我们自己加上@SpringbootTest注解

**测试UserService这个接口**
- 先测试UserService能不能使用，测试代码如下：
	```java
	@SpringBootTest  
	class UserServiceTest {  
	    @Resource  
	    private UserService userService;  
	    @Test  
	    void testAddUser(){  
	        User user = new User();  
	        user.setUsername("testdakkk");  
	        user.setUserAccount("123");  
	        user.setAvatarUrl("https://www.xxxxxxx.com");  
	        user.setGender(0);  
	        user.setUserPassword("xxx");  
	        user.setPhone("123");  
	        user.setEmail("456");  
	        boolean result = userService.save(user);  
	        System.out.println(user.getId());  
	        Assertions.assertTrue(result);  
	    }  
	}
	```
	
- 报错1：[问题1](0_bug处理/2_Part2-BUG.md#问题1)
- 报错2：[问题2](0_bug处理/2_Part2-BUG.md#问题2)
## 3 规整项目目录

创建一些用到的文件夹
- controller
- mapper
- model
	- domain
- service
- utils
- 。。。。。

其他暂时没有用的文件先删除！

## 4 后端实现
### 4.1 注册功能
#### 4.1.1 详细设计！！

1. 用户在前端输入账户和密码、以及校验码（todo）
2. 校验用户的账户、密码、校验密码是否符合需求
	- 数据不能为空
	- 账户的话不小于4位（可以使用正则表达式）
	- 密码不小于8位（可以使用正则表达式）
	- 账户不能重复！
	- 账户不包含特殊字符
	- 密码和校验密码相同
3. 对密码进行加密（==千万不能明文保存到数据库==）
4. 向数据库插入用户数据

#### 4.1.2 接口逻辑开发

**编写UserService接口 和 UserServiceImpl实现类**
- 使用了lang3的依赖，便于快速判断多个值是否为null
- 校验账户特殊字符：
	- [java正则表达式处理特殊字符_java校验账户名特殊字符的正则表达式-CSDN博客](https://blog.csdn.net/YaoWu_Zhou/article/details/107974793)
	- [Java过滤特殊字符的正则表达式 - 精神领袖 - 博客园 (cnblogs.com)](https://www.cnblogs.com/yasong-zhang/p/4916462.html)

**UserService接口**
```java
/**  
 * 用户注册功能  
 * @param userAccont 用户账户  
 * @param userPassword 用户密码  
 * @param checkPassword 校验密码  
 * @return 新用户id  
 */    
 long userRegister(String userAccont,String userPassword,String checkPassword); 
```

**UserServiceImpl实现类**中的登录方法
```java
@Override  
public long userRegister(String userAccont, String userPassword, String checkPassword) {  
    // 1.校验  
    // 输入内容不能为空  
    if (StringUtils.isAnyBlank(userAccont, userPassword, checkPassword)) {  
        return -1;  
    }  
    if (userAccont.length() < 4){  
        return -1;  
    }  
    if(userPassword.length()<8||checkPassword.length()<8){  
        return -1;  
    }  
    // 账户不包含特殊字符(放在账户不能重复前面)  
    String regEx="\\pP|\\pS|\\s+";  
    Matcher matcher = Pattern.compile(regEx).matcher(userAccont);  
    if (matcher.find()){  
        return -1;  
    }  
    //密码和校验密码相同  
    if(!userPassword.equals(checkPassword)){  
        return -1;  
    }  
    // 账户不能重复  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.eq("userAccount",userAccont);  
    long count = this.count(qw);  
    if(count >0){  
        return -1;  
    }  
  
    //2.密码加密(使用spring自带的工具库进行加密)  
    final String SALT = "dakkk";  
    String encryPassword = DigestUtils.md5DigestAsHex((SALT+userPassword).getBytes());  
    //3. 插入数据  
    User user = new User();  
    user.setUserAccount(userAccont);  
    user.setUserPassword(userPassword);  
    boolean saveResult = this.save(user);  
    return saveResult?user.getId():-1;  
}
```

==补充：查询用户是否已经存在这个方法，放在最后，前面判断都ok了，再去数据库查，减少数据库压力==
#### 4.1.3 单元测试

**特别注意，测试的顺序要按照实现类的逻辑来进行，否则容易出错**

出错了也不怕，多debug检查一下代码

单元测试方法代码如下：
```java
@Test  
void userRegister() {  
    //校验密码等不能为空  
    String userAccount = "mikey";  
    String userPassword = "";  
    String checkPassword = "123456";  
    long res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    //校验账户不能小于4位  
    userAccount="da";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    //校验密码不能小于8位  
    userAccount="mikey";  
    userPassword="123456";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    //校验账户不能包含特殊字符  
    userAccount="da  kkk";  
    userPassword = "12345678";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    //校验 密码和校验密码需要相同  
    checkPassword = "123456789";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    //校验 用户不能重复  
    userAccount = "123456";  
    checkPassword = "12345678";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertEquals(-1,res);  
  
    // 最后一次成功  
    userAccount="dakkk";  
    res = userService.userRegister(userAccount, userPassword, checkPassword);  
    Assertions.assertTrue(res>0);  
}
```

- UserController类
```java
@RestController  
@RequestMapping("/user")  
public class UserController {  
    @Resource  
    private UserService userService;  
  
    @PostMapping("register")  
    public Long userRegister(@RequestBody UserRegisterRequest userRegisterRequest){  
        if (userRegisterRequest == null){  
            return null;  
        }  
  
        String userAccont = userRegisterRequest.getUserAccont();  
        String userPassword = userRegisterRequest.getUserPassword();  
        String checkPassword = userRegisterRequest.getCheckPassword();  
  
        if(StringUtils.isAnyBlank(userAccont, userPassword, checkPassword)){  
            return null;  
        }  
  
        return userService.userRegister(userAccont, userPassword, checkPassword);  
    }  
```

- 在这个方法中，使用了一个封装接受前端参数的类
```java
@Data  
public class UserLoginRequest implements Serializable {  
    private static final long serialVersionUID = -3081732840333386033L;  
    private String userAccont;  
    private String userPassword;  
}
```

### 4.2 登录功能

#### 4.2.1 详细设计！！

**登录接口**
- 接受参数：用户账户、密码
- 请求类型：POST
	- 请求参数很长，或无法预料请求参数的时候，不要用GET
- 请求体：JSON
- 返回值：用户信息（脱敏）

**逻辑**
1. 校验用户账户和密码是否合法
	- 非空
	- 账户长度 不小于 4位
	- 密码长度 不小于 8位
	- 账户不包含特殊字符
2. 校验密码是否输入正确，和数据库中的密文密码对比
3. 记录用户的登录态（session&cookie），存到服务器上
	- 用后端SpringBoot框架封装的服务器 tomcat去记录）
4. 返回用户信息（脱敏）

#### 4.2.2 登录态管理（Cookie & Session）

1. 连接服务器端后，得到一个session1状态，返回给前端
2. 登录成功后，得到了登录成功的session，返回给前端一个设置cookie的命令
3. 前端接收到后端的命令后，设置cookie，保存到用户的浏览器中
4. 前端再次请求后端的时候（相同的域名），在请求头中带上cookie去请求
5. 后端拿到前端传过来的cookie，找到对应的session1
6. 再从session1中可以取出其中存储的属性（用户登录信息，登录名等）
#### 4.2.3 接口开发及测试

**接口开发：**
- UserService 接口
```java
/**  
 * 用户登录  
 * @param userAccont 用户账户  
 * @param userPassword 用户密码  
 * @return 脱敏后的用户信息  
 */  
User doLogin(String userAccont, String userPassword, HttpServletRequest req);
```

- UserServiceImpl实现类
```java
@Override  
public User doLogin(String userAccont, String userPassword, HttpServletRequest req) {  
    // 1.校验  
    // 输入内容不能为空  
    if (StringUtils.isAnyBlank(userAccont, userPassword)) {  
        //todo 修改为自定义异常  
        return null;  
    }  
    if (userAccont.length() < 4){  
        return null;  
    }  
    if(userPassword.length()<8){  
        return null;  
    }  
    // 账户不包含特殊字符(放在账户不能重复前面)  
    String regEx="\\pP|\\pS|\\s+";  
    Matcher matcher = Pattern.compile(regEx).matcher(userAccont);  
    if (matcher.find()){  
        return null;  
    }  
    // 账户不能重复  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.eq("userAccount",userAccont);  
    long count = this.count(qw);  
    if(count >0){  
        return null;  
    }  
  
    //2.密码加密(使用spring自带的工具库进行加密)  
    String encryPassword = DigestUtils.md5DigestAsHex((SALT+userPassword).getBytes());  
  
    //3.查询用户是否存在  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.eq(User::getUserAccount,userAccont);  
    lqw.eq(User::getUserPassword,encryPassword);  
    User loginUser = this.getOne(lqw);  
  
    //todo 可以添加用户限流，如果用户在单ip下登录次数过多  
  
    //4. 判断数据库中是否存在该用户  
    if(null==loginUser){  
        log.info("user login failed,userAccount connot matched!");  
        return null;  
    }  
  
    //5.用户信息脱敏（用的多的话，可以抽取为独立的方法）
    User safetyUser = new User();  
    safetyUser.setId(loginUser.getId());  
    safetyUser.setUsername(loginUser.getUsername());  
    safetyUser.setUserAccount(loginUser.getUserAccount());  
    safetyUser.setAvatarUrl(loginUser.getAvatarUrl());  
    safetyUser.setGender(loginUser.getGender());  
    safetyUser.setPhone(loginUser.getPhone());  
    safetyUser.setEmail(loginUser.getEmail());  
    safetyUser.setUserStatus(loginUser.getUserStatus());  
    safetyUser.setCreateTime(loginUser.getCreateTime());  
  
    //6.记录用户的登录态(使用request)  
    HttpSession session = req.getSession();  
    //这里使用了全局常量
    session.setAttribute(USER_LOGIN_STATE,safetyUser);  
  
    return safetyUser;  
}
```
- service层是对业务逻辑的校验（有可能被controller之外的类调用）

- UserController类
```java
@RestController  
@RequestMapping("/user")  
public class UserController {   
    @PostMapping("login")  
    public User userLogin(@RequestBody UserLoginRequest userLoginRequest, HttpServletRequest req){  
        if (userLoginRequest == null){  
            return null;  
        }  
  
        String userAccont = userLoginRequest.getUserAccont();  
        String userPassword = userLoginRequest.getUserPassword();  
  
        if(StringUtils.isAnyBlank(userAccont, userPassword)){  
            return null;  
        }  
  
        return userService.userLogin(userAccont,userPassword, req);  
    }  
}
```
- controller层一般倾向于请求参数本身的校验，不涉及业务逻辑本身（越少越好）
- [问题3](0_bug处理/2_Part2-BUG.md#问题3)

- 在这个方法中，使用了一个封装接受前端参数的类
```java
@Data  
public class UserRegisterRequest implements Serializable {  
    private static final long serialVersionUID = 8628383983087108526L;  
    private String userAccont;  
    private String userPassword;  
    private String checkPassword;  
}
```
- [问题4](0_bug处理/2_Part2-BUG.md#问题4)

**测试**
- 先启动启动类
- 然后在IDEA的上方Tools中选择HTTP Client选择Create Request in HTTP Client，生成一个测试服务
- ==测试注册接口==（**注意debug测试，F8跳过，F9跳到下一个断点**）
```http
POST http://localhost:8080/user/login  
Content-Type: application/json  
  
{  
  "userAccont": "zhangsan1",  
  "userPassword": "123456789"  
}
```

- ==测试登录接口==
```http
POST http://localhost:8080/user/register  
Content-Type: application/json  
  
{  
  "userAccont": "zhangsan1",  
  "userPassword": "123456789",  
  "checkPassword": "123456789"  
}
```

- **注意：** 需要屏蔽test 参考[Springboot Maven打包跳过测试的五种方式总结 -Dmaven.test.skip=true_springboot打包跳过test-CSDN博客](https://blog.csdn.net/Galaxy_0/article/details/136526664#:~:text=%E6%89%93%E5%BC%80%E9%85%8D%E7%BD%AE%EF%BC%8C%E6%89%BE%E5%88%B0%20Build%2CExxcution%2CDeployment%20%E2%80%93%3E%20Maven%20Tools%20%E2%80%93%3E%20Maven%20%E2%80%93%3E,%E2%80%93%3E%20Runner%20%EF%BC%8C%E5%9C%A8%20Properties%20%E4%B8%AD%E5%8B%BE%E9%80%89%20Skip%20Test%20%E9%80%89%E9%A1%B9%E3%80%82)

### 4.3 用户管理和注销功能（管理员）


**注意：必须鉴权，即是否为管理员，不是管理员不能使用下面的接口**
- 给user表加一个字段：role
- 同时在User实体对象中增加属性：role
- 在UserMapper.xml文件中，标签resultMap也要加上role，不然MP无法读取到role
- ==因为添加了一个字段，所以很多文件都需要修改，不止上述内容，还有其他内容要多检查一下==

**接口设计**
1. 查询用户
	- 允许根据用户名查询
2. 删除用户


**UserController代码**
```mysql
@RestController  
@RequestMapping("/user")  
public class UserController {  
    @Resource  
    private UserService userService;  

    //查询用户
    @GetMapping("/search")  
    public List<User> searchUsers(String username,HttpServletRequest req) {  
        //鉴权  
        if(!isAdmin(req)){  
            return new ArrayList<>();  
        }  
  
        LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
        if (StringUtils.isNotBlank(username)) {  
            lqw.like(User::getUsername, username);  
        }  
        List<User> userList = userService.list(lqw);  
        //返回的信息要脱密  
        return userList.stream().map(user->userService.getSafetyUser(user)).collect(Collectors.toList());  
    }  
  
    //删除用户
    @PostMapping("/delete")  
    public Boolean deleteUser(@RequestBody long id,HttpServletRequest req) {  
        //鉴权  
        if(!isAdmin(req)){  
            return false;  
        }  
  
        if (id <= 0) {  
            return false;  
        }  
        return userService.removeById(id);  
    }  
  
    /**  
     * 用户鉴权方法  
     * @param req 用户的请求  
     * @return 用户是否为管理员，是则为true  
     */    
     private boolean isAdmin(HttpServletRequest req){  
        //鉴权  
        HttpSession session = req.getSession();  
        Object userObj = session.getAttribute(USER_LOGIN_STATE);  
        User user = (User) userObj;  
        //注意空指针异常  
        return user != null && user.getUserRole() == ADMIN_ROLE;  
    }  
}
```

- 将我们之前与用户相关的常量封装在UserConstant这个类中
```java
public interface UserConstant {  
    /**  
     * 用户登录态的键  
     */  
    String USER_LOGIN_STATE = "userLoginState";  
  
    // ----- 权限 -----  
    /**  
     * 默认权限  
     */  
    int DEFAULT_ROLE = 0;  
    /**  
     * 管理员权限  
     */  
    int ADMIN_ROLE = 1;  
}
```

**接口的测试**
- 注意：
	- 设置了session后，会在用户的浏览器中保留下一个cookie，这个cookie就会对应服务器中该设置的session
	- 如果我们没有设置session的有效期（可以在yml文件中设置），默认是30min

- 使用IDEA自带的HTTP Client进行测试，代码如下
```http
###  
POST http://localhost:8080/user/login  
Content-Type: application/json  
  
{  
  "userAccont": "zhangsan1",  
  "userPassword": "123456789",  
  "checkPassword": "123456789"  
}  
  
  
###  
GET http://localhost:8080/user/search/?username=test  
  
  
###  
POST http://localhost:8080/user/delete  
Content-Type: application/json  
  
1
```
- 上述的`1`，是因为我们在删除的方法中，接受的参数是@RequestBody的，所以使用这个，==不需要加{}==

## 5 前端实现
### 5.1 快速页面开发

问题：使用`start:dev`脚本启动项目无法启动，和part1启动start脚本问题一样，参考[问题2](0_bug处理/1_Part1-BUG.md#问题2)

**开始删除登录页面的东西**：`src/pages/user/Login/index.tsx` 文件
- `ctrl + shift + - ` 一键折叠所有代码，然后一级级的展开代码

- ==先修改Footer==：ctrl + 鼠标左键选择Footer这个标签，然后对应格式随意修改
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7d711cbe442586dc595fda03be81d327.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/933fc581e70624cda544a61646f84131.png)
- 创建constant常量包，来保存logo图标的地址，以后一些常用的素材（链接），也可以放到这里
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/546efe41510fe61656571c1773bc00b3.png)


- 继续修改`index.tsx`
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/662dd9137dddc841e70e10f64d10b7e1.png)
- ==具体删除和修改的代码==如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f48c869a240e18232afdafe39cb45a5e.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f190eb87c6530ab5fd8adf81938d357.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12b1a7fe92c391e9541d1b4158831d15.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ec7c1216a37bf93a68254f651a5a0339.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2601cf8454305fe97725b656dfc06d64.png)
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/68edd4670021afc99f4c357c6b8a7b52.png)

- 使用`ctrl + r `快捷键全局替换==用户名==为==账号（userAccount）== 

### 5.2 前后端联调

前端需要向后端发送请求
- 前端使用ajax来异步请求后端（最初的技术）
- 后来又axios封装了ajax
- 然后在我们这个前端界面中request又封装了一次（ant design项目）

### 5.3 登录页面修改

我们==看一下登录页面有哪些方法==，可以在里面看到有个` onFinish`方法，这就是表单提交的方法
- 方法的传递如下图：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a510f0204166f0823d4b3d4493f8886c.png)
- 使用快捷键 `shift+f6`修改LoginParams中的值（可以一键重构），与后端对应即可

- 继续看`handleSubmit`的方法，看到有一个`login`方法，进入看一下，追踪源码，可以看到方法的来源，来源于umi
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb02c294f02b32fbd6334c493eb07234.png)
- 我们需要找到前端项目的全局配置，及url的前缀
	- 追踪request源码：用到了umi的插件，requestConfig 是一个配置
	- ==所以我们可以想一想，去UmiJs官网，看看有没有request的说明==，UmiJs官网看了一遍
	- 看一看config目录下有没有，也没有
	- 再去ant design pro官网看一看，发现了
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/70380b8f86458d9d7ad5faa98fc3ea32.png)
		- 我们需要在 `app.tsx` 中使用上述代码，实现全局url前缀（==注意是app.tsx文件==）
		- 复制到文件中后，点进入看看，有什么参数
		  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8132ce36d25845347b2c7b9471ceb042.png)
	- 最后使用 prefix设置前缀
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e256d78007aab43a3e95d659a14bd5e5.png)
	- 后期上线环境下的前缀修改，后面再讲

- 如果实在找到不前缀，直接在url中写完整的后端请求路径
	- 或者，在request所在的文件中定义一个全局前缀`const BASE_PREFIX process.env ? 'http://localhost:8080' : 'https:/xxx' ;`

### 5.4 登录API接口测试

根据上述的设置，我们已经完成url前缀的设置了

回到`api.ts`文件中，我们修改登录的api
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cea6f1e707670df20fa91347fa6ebbc9.png)

使用网页进行测试，请求网址没有错误！
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d03870de20a5412735b05e55057b610.png)

**注意：跨域问题➡协议，ip，端口其中任何一个不同，就会产生跨域问题，即无法连接另外一端的服务**

我们使用代理服务器，修改config目录下的proxy.ts文件，修改如下图：（==不能使用localhost==）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/41034f9ee2ebf1cd1234dea071c631ce.png)
在`app.tsx`文件中，设置全局url前缀（==可以在api.ts文件中每个url都加api前缀也行==）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d302976fe04cdc72f0ad5d4dfc146480.png)
在 `api.ts`文件中，登录接口不用动
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/65c675a7b90cc5f5cb9932183a95dcb8.png)
回到后端项目，设置全局前缀
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4bba0c907eb1678b8e301d471964358f.png)

在`api.ts`文件中，修改登录页面中的 username 和 password 为我们后台的参数
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/169a5affaa35416683505a49c242fd1a.png)

在 `api.ts`文件中，添加密码校验规则，具体参数参考ant design官网
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78e46da3d7e935a567da905f1f32f7ee.png)

在 `api.ts`文件中，修改用户登录方法的校验
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/812ea541ca157ff832735e499c686027.png)


开启前后端，后端登录的代码打上断点，前端开始登录，发现进入断点了，然后不断跳过断点，发现可以登录成功即可（账号密码要写数据库中对的哈！）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/39c47a847fc64f10a317a954493d546e.png)
成功！

## 6 代理知识

### 6.1 正向代理

替客户端向服务器发送请求
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d6176c313702e34c29c5645015e9e935.png)

### 6.2 反向代理

替服务器接受请求
- 加入你有三台服务器，每台服务器的url地址不一样
- 那么用户访问的时候，肯定不会根据你不同的url来访问不同的服务器
- 如何解决呢？**使用代理**
	- 用户访问代理的url，代理通过某种方式将请求（按照你的需求）转发到各个服务器
	- 这就叫反向代理

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/775bf1573b9c7588b76a38f0aa6034a7.png)

### 6.3 本地开启代理

Nginx服务器

Nodejs服务器

ant design服务器

