
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
	
- 报错1：[问题1](bug处理/2_Part2-BUG.md#问题1)
- 报错2：[问题2](bug处理/2_Part2-BUG.md#问题2)
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
## 4 注册功能
### 4.1 详细设计！！

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

### 4.2 接口逻辑开发

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
### 4.3 单元测试

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
### 4.4 快速页面开发（前端）

### 4.5 表单组件使用（前端）


### 4.6 API接口测试

## 5 登录功能

### 5.1 详细设计！！


### 5.2 登录态管理（前端）

### 5.3 请求库的使用（前端）

### 5.4 页面开发与验证（前端）

### 5.5 登录态管理（Cookie & Session）

### 5.6 接口开发及测试

### 5.7 前后端联调

## 6 代理知识

### 6.1 正向代理

### 6.2 方向代理

### 6.3 本地开启代理

## 7 用户管理功能

### 7.1 前端

### 7.2 后端

## 8 用户注销功能

### 8.1 前端

### 8.2 后端

