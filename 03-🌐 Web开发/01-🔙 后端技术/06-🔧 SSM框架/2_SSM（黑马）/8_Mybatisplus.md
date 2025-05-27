---
文章标题: "[[8_Mybatisplus]]" 
文章作者: Dakkk
文章概要: |
  介绍基于SpringBoot集成MyBatisPlus的全面开发教程，包括基础CRUD操作、条件查询、分页、主键生成策略、逻辑删除、乐观锁和代码生成器等高级特性的详细实现。
tags:
- "MyBatisPlus"
- "SpringBoot"
- "CRUD"
- "条件查询"
- "分页"
- "主键策略"
- "逻辑删除"
- "乐观锁"
- "代码生成器"
相关文章:
- "[[0_导读]]"
- "[[0_主要内容和学习目标]]"
- "[[10_首页文章分页数据渲染]]"
- "[[10_文章分类：删除功能开发]]"
- "[[11_标签-文章列表页开发]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/2_SSM（黑马）/8_Mybatisplus.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:12:43
---

## 1 本文摘要

- 基于MyBatisPlus完成标准Dao的增删改查功能
- 掌握MyBatisPlus中的分页及条件查询构建
- 掌握主键ID的生成策略
- 了解MyBatisPlus的代码生成器
## 2 MyBatisPlus入门案例与简介

由于我们之前有MyBatis开发的基础，所以我们现在直接来一个入门案例，来体现MyBatisPlus的强大之处
### 2.1 入门案例

- MyBatisPlus（简称MP）是基于MyBatis框架基础上开发的增强型工具，旨在简化开发，提高效率
- 开发方式
    - 基于MyBatis使用MyBatisPlus
    - 基于Spring使用MyBatisPlus
    - ==基于SpringBoot使用MyBatisPlus(重点)==

由于我们刚刚才学完SpringBoot，所以我们现在直接使用SpringBoot来构建项目，官网的快速开始也是直接用的SpringBoot

- `步骤一：`创建数据库和表  
    数据库随便挑一个自己喜欢的吧
```sql
CREATE TABLE user (  
    id bigint(20) primary key auto_increment,  
    name varchar(32) not null,  
    password  varchar(32) not null,  
    age int(3) not null ,  
    tel varchar(32) not null  
);  
insert into user values(1,'Tom','tom',3,'18866668888');  
insert into user values(2,'Jerry','jerry',4,'16688886666');  
insert into user values(3,'Jock','123456',41,'18812345678');  
insert into user values(4,'略略略','nigger',15,'4006184000');
```
- `步骤二：`创建SpringBoot工程  
    只需要勾选MySQL，不用勾选MyBatis了

- `步骤三：`补全依赖  
    导入德鲁伊和MyBatisPlus的坐标
```xml
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-boot-starter</artifactId>  
    <version>3.4.1</version>  
</dependency>  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.16</version>  
</dependency>
```

- `步骤四：`编写数据库连接四要素  
	还是将application的后缀名改为yml，以后配置都是用yml来配置  
	注意要设置一下时区，不然可能会报错（指高版本的mysql）
```yml
spring:  
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: YOUSONOFABTICH.  
  
# mybatis的日志信息  
mybatis-plus:  
  configuration:  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

- `步骤五：`根据数据表来创建对应的模型类  
	注意id是Long类型，至于为什么是Long，接着往下看
```java
@Data
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- `步骤六：`创建dao接口  
```java
@Mapper  
public interface UserDao extends BaseMapper<User>{  
}
```
只需要在类上方加一个`@Mapper`注解，同时继承`BaseMapper<>`，泛型写创建的模型类的类型  

- `步骤七：`赶快来测试！！  
	SpringBoot的测试类也是简单的一批，只需要一个`@SpringBootTest`注解就能完成（创建SpringBoot工程的时候已经帮我们自动弄好了）  
	测试类里需要什么东西就用`@Autowired`自动装配，测试方法上用`@Test`注解
```java
@SpringBootTest  
class MybatisplusApplicationTests {  
  
    @Autowired  
    private UserDao userDao;  
  
    @Test  
    void contextLoads() {  
        List<User> users = userDao.selectList(null);  
        for (User b : users) {  
            System.out.println(b);  
        }  
    }  
}
```

selectList() 方法的参数为 MP 内置的条件封装器 Wrapper，所以不填写就是无任何条件

现在运行测试方法，看看控制台![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36b3aa4b3995574da5aa679db3d76719.png)


原理，慢慢往后看
### 2.2 MyBatis-Plus简介

MyBatisPlus的官网为:[https://mp.baomidou.com/](https://mp.baomidou.com/) ，没错就是个拼音，苞米豆，因为域名被抢注了，但是粉丝也捐赠了一个 [https://mybatis.plus](https://mybatis.plus/) 域名

MP旨在成为MyBatis的最好搭档，而不是替换掉MyBatis，从名称上来看也是这个意思，一个MyBatis的plus版本，在原有的MyBatis上做增强，其底层仍然是MyBatis的东西，所以我们当然也可以在MP中写MyBatis的内容

对于MP的深入学习，可以多看看官方文档，锻炼自己自学的能力，毕竟不是所有知识都有像这样的网课，更多的还是自己看文档，挖源码。

MP的特性：

- `无侵入：`只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- `损耗小：`启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- `强大的 CRUD 操作：`内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- `支持 Lambda 形式调用`：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- `支持主键自动生成：`支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- `支持 ActiveRecord 模式：`支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- `支持自定义全局通用操作：`支持全局通用方法注入（ Write once, use anywhere ）
- `内置代码生成器：`采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- `内置分页插件：`基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- `分页插件支持多种数据库：`支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- `内置性能分析插件：`可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- `内置全局拦截插件：`提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
### 2.3 小结

SpringBoot集成MyBatisPlus非常的简单，只需要导入`MyBatisPlus`的坐标，然后令dao类继承`BaseMapper`，写上泛型，类上方加`@Mapper`注解

可能存在的疑问：
- 我甚至都没写在哪个表里查，为什么能自动识别是在我刚刚创建的表里查？
    - 注意我们创建的表，和对应的模型类，是同一个名，默认情况是在同名的表中查找
- 那我要是表明和模型类的名不一样，那咋整？
    - 在模型类的上方加上`@TableName`注解
        - 例如数据表叫`tb_user`但数据类叫`User`，那么就在User类上加`@TableName("tb_user")`注解
## 3 标准数据层开发

### 3.1 标准的CRUD使用

先来看看MP给我们提供了哪些方法

|功能|自定义接口|MP接口|
|---|---|---|
|新增|boolean save(T t)|int insert(T t)|
|删除|boolean delete(int id)|int deleteById(Serializable id)|
|修改|boolean update(T t)|int updateById(T t)|
|根据id查询|T getById(int id)|T selectById(Serializable id)|
|查询全部|List\<T> getAll()|List\<T> selectList()|
|分页查询|PageInfo\<T> getAll(int page,int size)|IPage\<T> selectPage(IPage\<T> page)|
|按条件查询|List\<T> getAll(Condition condition)|IPage\<T> selectPage(Wrapper\<T>queryWrapper)|

我们就按照上表中的方法来演示
### 3.2 新增

```java
int insert(T t)
```

参数类型是泛型，也就是我们当初继承BaseMapper的时候，填的泛型，返回值是int类型，0代表添加失败，1代表添加成功
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Seto");  
    user.setAge(23);  
    user.setTel("4005129421");  
    user.setPassword("MUSICIAN");  
    userDao.insert(user);  
}
```

随便写一个User的数据，运行程序，然后去数据库看看新增是否成功![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2b508629ca38119f114dfbc2980a53c.png)
这个主键自增id看着有点奇怪，但现在你知道为什么要将id设为long类型了吧
### 3.3 删除

```java
int deleteByIds(Serializable id);
```

- 参数类型为什么是一个序列化类`Serializable`
    - 通过查看String的源码，你会发现String实现了Serializable接口，而且Number类也实现了Serializable接口
    - Number类又是Float，Double，Long等类的父类
    - 那现在能作为主键的数据类型，都已经是Serializable类型的子类了
    - MP使用Serializable类型当做参数类型，就好比我们用Object类型来接收所有类型一样
- 返回值类型是int
    - 数据删除成功返回1
    - 未删除数据返回0。
- 那下面我们就来删除刚刚添加的数据，注意末尾加个L
```java
@Test  
void testDelete(){  
    userDao.deleteById(1572364408896622593L);  
}
```

删除完毕之后，刷新数据库，看看是否删除成功
### 3.4 修改

```java
int updateById(T t);
```

- T:泛型，需要修改的数据内容，注意因为是根据ID进行修改，所以传入的对象中需要有ID属性值
- int:返回值
    - 修改成功后返回1
    - 未修改数据返回0

```java
@Test  
void testUpdate(){  
    User user = new User();  
    user.setId(1L);  
    user.setName("Alen");  
    userDao.updateById(user);  
}
```
==修改功能只修改指定的字段，未指定的字段保持原样==
### 3.5 根据ID查询

```java
T selectById (Serializable id)
```

- Serializable：参数类型,主键ID的值
- T:根据ID查询只会返回一条数据

```java
@Test  
void testSelectById(){  
    User user = userDao.selectById(1);  
    System.out.println(user);  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3feb58c50ee830be4c77ba995d26183f.png)


### 3.6 查询全部

```java
List<T> selectList(Wrapper<T> queryWrapper)
```

Wrapper：用来构建条件查询的条件，目前我们没有可直接传为Null

```java
@Test  
void testSelectAll() {  
    List<User> users = userDao.selectList(null);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c0a9eef85147fba46f50bb14f5752bf.png)
- 方法都测试完了，那你们有没有想过，这些方法都是谁提供的呢？
    - 想都不用想，肯定是我们当初继承的`BaseMapper`，里面的方法还有很多，后面我们再慢慢学习
### 3.7 Lombok

- 代码写到这，我们发现之前的dao接口，都不用我们自己写了，只需要继承BaseMapper，用他提供的方法就好了
- 但是现在我还想偷点懒，毕竟懒是第一生产力，之前我们手写模型类的时候，创建好对应的属性，然后用IDEA的Alt+Insert快捷键，快速生成get和set方法，toSring，各种构造器（有需要的话）等
- U1S1项目做这么久，写模型类都给我写烦了，有没有更简单的方式呢？
    - 答案当然是有的，可以使用Lombok，一个Java类库，提供了一组注解，来简化我们的POJO模型类开发

具体步骤如下
- `步骤一：`添加Lombok依赖
```xml
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <!--<version>1.18.12</version>-->  
</dependency>
```
版本不用谢，SpringBoot中已经管理了lombok的版本

- `步骤二：`在模型类上添加注解
- Lombok常见的注解有:
	- `@Setter:`为模型类的属性提供setter方法
	- `@Getter:`为模型类的属性提供getter方法
	- `@ToString:`为模型类的属性提供toString方法
	- `@EqualsAndHashCode:`为模型类的属性提供equals和hashcode方法
	- `@Data:`是个组合注解，包含上面的注解的功能
	- `@NoArgsConstructor:`提供一个无参构造函数
	- `@AllArgsConstructor:`提供一个包含所有参数的构造函数
```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

说明:Lombok只是简化模型类的编写，我们之前的方法也能用  
例如你有特殊的构造器需求，只想要name和password这两个参数，那么可以手写一个
```java
public User(String name, String password) {  
    this.name = name;  
    this.password = password;  
}
```
### 3.8 分页功能

基础的增删改查功能就完成了，现在我们来进行分页功能的学习
```java
IPage<T> selectPage(IPage<T> page, Wrapper<T> queryWrapper)
```

- IPage用来构建分页查询条件
- Wrapper：用来构建条件查询的条件，暂时没有条件可以传一个null
- 返回值IPage是什么意思，后面我们会说明

具体的使用步骤如下

- `步骤一：`调用方法传入参数获取返回值
```java
@Test  
void testSelectPage() {  
    IPage<User> page = new Page<>(1, 3);  
    userDao.selectPage(page, null);  
    System.out.println("当前页码" + page.getCurrent());  
    System.out.println("本页条数" + page.getSize());  
    System.out.println("总页数" + page.getPages());  
    System.out.println("总条数" + page.getTotal());  
    System.out.println(page.getRecords());  
}
```

- `步骤二：`设置分页拦截器
```java
//注意要被扫描到
@Configuration
public class MybatisPlusConfig {  
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor(){  
        MybatisPlusInterceptor myInterceptor = new MybatisPlusInterceptor();  
        myInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());  
        return myInterceptor;  
    }  
}
```

- `步骤三：`运行测试程序  
	运行程序，结果如下，符合我们的预期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48f0f5a5ef7d4d12e08f7dfb86cfc781.png)

## 4 DQL编程控制

增删改查四个操作中，查询是非常重要的也是非常复杂的操作，这部分我们主要学习的内容有:

- 条件查询方式
- 查询投影
- 查询条件设定
- 字段映射与表名映射
### 4.1 条件查询

#### 4.1.1 条件查询的类

- MP将复杂的SQL查询语句都做了封装，使用编程的方式来完成查询条件的组合
- 之前我们在写CRUD时，都看到了一个Wrapper类，我们当初都是赋一个null值，但其实这个类就是用来查询的
#### 4.1.2 构建条件查询

- `QueryWrapper`  
	小于用lt，大于用gt  
	回想之前我们在html页面中，如果需要用到小于号或者大于号，需要用对应的html实体来替换  
	小于号的实体是 `&lt;`，大于号的实体是`&gt;`
```java
@Test  
void testQueryWrapper(){  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    //条件为 age字段小于18  
    qw.lt("age",18);  
    List<User> userList = userDao.selectList(qw);  
    System.out.println(userList);  
}
```

运行测试方法，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e24f7d636af92257de4422a67580670e.png)
这种方法有个弊端，那就是==字段名是字符串类型，没有提示信息和自动补全==，如果写错了，那就查不出来

- `QueryWrapper`的基础上，使用`lambda`
```java
@Test  
void testQueryWrapper(){  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.lambda().lt(User::getAge,18);  
    List<User> userList = userDao.selectList(qw);  
    System.out.println(userList);  
}
```
`ser::getAget`,为lambda表达式中的，`类名::方法名`
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b933a6a63feb3913540fc8f791b553fb.png)


- `LambdaQueryWrapper`  
	方式二解决了方式一的弊端，但是要多些一个lambda()，那方式三就来解决方式二的弊端，使用LambdaQueryWrapper，就可以不写lambda()
```java
@Test  
void testQueryWrapper(){  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.lt(User::getAge,18);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```
#### 4.1.3 多条件查询

上面三种都是单条件的查询，那我们现在想进行多条件的查询，该如何编写代码呢？  

需求：查询表中年龄在10~30岁的用户信息
```java
@Test  
void testQueryWrapper(){  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    //大于10  
    lqw.gt(User::getAge,10);  
    //小于30  
    lqw.lt(User::getAge,30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```

- 构建多条件的时候，我们还可以使用链式编程
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.gt(User::getAge, 10).lt(User::getAge, 30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```

- 可能存在的疑问
    - MP怎么就知道你这俩条件是AND的关系呢，那我要是想用OR的关系，该咋整
- 解答
    - ==默认就是AND的关系，如果需要OR关系，用or()链接就可以了==
```java
lqw.gt(User::getAge, 10).or().lt(User::getAge, 30);
```

需求：查询年龄小于10，或者年龄大于30的用户信息
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.lt(User::getAge, 10).or().gt(User::getAge, 30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```
#### 4.1.4 null值的判定

- 我们在做条件查询的时候，一般都会有很多条件供用户查询
- 这些条件用户可以选择用，也可以选择不用
- 之前我们是通过动态SQL来实现的
```xml
<select id="selectByPageAndCondition" resultMap="brandResultMap">  
    select *  
    from tb_brand  
    <where>  
        <if test="brand.brandName != null and brand.brandName != '' ">  
            and  brand_name like #{brand.brandName}  
        </if>  
  
        <if test="brand.companyName != null and brand.companyName != '' ">  
            and  company_name like #{brand.companyName}  
        </if>  
  
        <if test="brand.status != null">  
            and  status = #{brand.status}  
        </if>  
    </where>  
    limit #{begin} , #{size}  
</select>
```

那现在我们来试试在MP里怎么写
	需求:查询数据库表中，根据输入年龄范围来查询符合条件的记录  
	用户在输入值的时候，  
	​如果只输入第一个框，说明要查询大于该年龄的用户  
	​如果只输入第二个框，说明要查询小于该年龄的用户  
	如果两个框都输入了，说明要查询年龄在两个范围之间的用户

- 问题一：后台如果想接收前端的两个数据，该如何接收?
	- 我们可以使用两个简单数据类型，也可以使用一个模型类，但是User类中目前只有一个age属性
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- 使用一个age属性，如何去接收页面上的两个值呢?这个时候我们有两个解决方案
	- 方案一：添加属性age2,这种做法可以但是会影响到原模型类的属性内容
	- 方案二：新建一个模型类,让其继承User类，并在其中添加age2属性，UserQuery在拥有User属性后同时添加了age2属性
```java
@Data  
@TableName("tb_user")  
public class UserQuery extends User{  
    private Integer age2;  
}
```

- 环境准备好后，我们来实现下刚才的需求：
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    UserQuery uq = new UserQuery();  
    uq.setAge(10);  
    uq.setAge2(30);  
    if (null != uq.getAge()) {  
        lqw.gt(User::getAge, uq.getAge());  
    }  
    if (null != uq.getAge2()) {  
        lqw.lt(User::getAge, uq.getAge2());  
    }  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    }  
}
```
- 上面的写法可以完成条件为非空的判断，但是问题很明显，如果条件多的话，每个条件都需要判断，代码量就比较大，来看MP给我们提供的简化方式

- lt还有一个重载的方法，当condition为true时，添加条件，为false时，不添加条件
```java
public Children lt(boolean condition, R column, Object val) {  
    return this.addCondition(condition, column, SqlKeyword.LT, val);  
}
```

- 故我们可以把if的判断操作，放到lt和gt方法中当做参数来写
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    UserQuery uq = new UserQuery();  
    uq.setAge(10);  
    uq.setAge2(30);  
    lqw.gt(null != uq.getAge(), User::getAge, uq.getAge())  
       .lt(null != uq.getAge2(), User::getAge, uq.getAge2());  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    } 
}
```
### 4.2 查询投影

#### 4.2.1 查询指定字段

目前我们在查询数据的时候，什么都没有做默认就是查询表中所有字段的内容，我们所说的查询投影即不查询所有字段，只查询出指定内容的数据。

具体如何来实现?
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.select(User::getName,User::getName);  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    }  
}
```

select(…)方法用来设置查询的字段列，可以设置多个
```java
lqw.select(User::getName,User::getName);
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e73e7d15904985ce124435063e81fc6.png)


如果使用的不是lambda，就需要手动指定字段
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("name", "age");  
    for (User user : userDao.selectList(qw)) {  
        System.out.println(user);  
    }  
}
```
#### 4.2.2 聚合查询

需求:聚合函数查询，完成count、max、min、avg、sum的使用

- count:总记录数
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("count(*) as count");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- max:最大值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("max(age) as maxAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- min:最小值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("min(age) as minAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- avg:平均值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("avg(age) as avgAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- sum:求和
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("sum(age) as sumAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
#### 4.2.3 分组查询

```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("max(age) as maxAge");  
    qw.groupBy("tel");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4074e0a082dda50e524590723e955a0d.png)


注意：
- ==聚合与分组查询，无法使用lambda表达式来完成==
- MP只是对MyBatis的增强，如果MP实现不了，我们可以直接在DAO接口中使用MyBatis的方式实现
### 4.3 查询条件

前面我们只使用了lt()和gt(),除了这两个方法外，MP还封装了很多条件对应的方法

- 范围匹配（> 、 = 、between）
- 模糊匹配（like）
- 空判定（null）
- 包含性匹配（in）
- 分组（group）
- 排序（order）
- ……
#### 4.3.1 等值查询

需求:根据用户名和密码查询用户信息
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.eq(User::getName,"Seto").eq(User::getPassword,"MUSICIAN");  
    User user = userDao.selectOne(qw);  
    System.out.println(user);  
}
```

- eq()： 相当于 `=`,对应的sql语句为
```sql
SELECT * FROM tb_user WHERE name = 'seto' AND password = 'MUSICIAN';
```
- selectList：查询结果为多个或者单个
- selectOne:查询结果为单个
#### 4.3.2 范围查询

需求:对年龄进行范围查询，使用lt()、le()、gt()、ge()、between()进行范围查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.between(User::getAge,10,30);  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

- gt():大于(>)
- ge():大于等于(>=)
- lt():小于(<)
- lte():小于等于(<=)
- between():between ? and ?
#### 4.3.3 模糊查询

需求:查询表中name属性的值以`J`开头的用户信息,使用like进行模糊查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.likeRight(User::getName,"J");  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

- like():前后加百分号,如 %J%，相当于包含J的name
- likeLeft():前面加百分号,如 %J，相当于J结尾的name
- likeRight():后面加百分号,如 J%，相当于J开头的name

需求:查询表中name属性的值包含`e`的用户信息,使用like进行模糊查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.like(User::getName,"e");  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```
#### 4.3.4 排序查询

需求:查询所有数据，然后按照age降序
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    /**  
        * condition ：条件，返回boolean，  
		当condition为true，进行排序，如果为false，则不排序  
        * isAsc:是否为升序，true为升序，false为降序  
        * columns：需要操作的列  
    */  
    qw.orderBy(true,false,User::getAge);  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```
遇到想用的功能，先自己用一个试试，方法名和形参名都很见名知意，遇到不确定的用法，再去官方文档查阅资料
### 4.4 映射匹配兼容性

在上面的案例中，我们做查询的时候，数据表中的字段名与模型类中的属性名一致，查询的时候没有问题，那么问题就来了

- `问题一：`表字段与模型类编码属性不一致
    - 当表的列名和模型类的属性名发生不一致，就会导致数据封装不到模型对象，这个时候就需要其中一方做出修改，那如果前提是两边都不能改又该如何解决?
    - MP给我们提供了一个注解`@TableField`,使用该注解可以实现模型类属性名和表的列名之间的映射关系
    - 例如表中密码字段为`pwd`，而模型类属性名为`password`，那我们就可以用`@TableField`注解来实现他们之间的映射关系
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField("pwd")  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- `问题二：`编码中添加了数据库中未定义的属性
	- 当模型类中多了一个数据库表不存在的字段，就会导致生成的sql语句中在select的时候查询了数据库不存在的字段，程序运行就会报错，错误信息为:`Unknown column '多出来的字段名称' in 'field list'`
	- 具体的解决方案用到的还是`@TableField`注解，它有一个属性叫`exist`，设置该字段是否在数据库表中存在，如果设置为false则不存在，生成sql语句查询的时候，就不会再查询该字段了。
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField("pwd")  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

- `问题三：`采用默认查询开放了更多的字段查看权限
	- 查询表中所有的列的数据，就可能把一些敏感数据查询到返回给前端，这个时候我们就需要限制哪些字段默认不要进行查询。解决方案是`@TableField`注解的一个属性叫`select`，该属性设置默认是否需要查询该字段的值，true(默认值)表示默认查询该字段，false表示默认不查询该字段。
	- 例如像密码这种的敏感字段，不应该查询出来作为JSON返回给前端，不安全
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField(value = "pwd",select = false)  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

知识点：`@TableField`

|名称|@TableField|
|---|---|
|类型|属性注解|
|位置|模型类属性定义上方|
|作用|设置当前属性对应的数据库表中的字段关系|
|相关属性|value(默认)：设置数据库表字段名称  <br>exist:设置属性在数据库表字段中是否存在，默认为true，此属性不能与value合并使用  <br>select:设置属性是否参与查询，此属性与select()映射配置不冲突|
- `问题四：`表名与编码开发设计不同步
    - 这个问题其实我们在一开始就解决过了，现在再来回顾一遍
    - 该问题主要是表的名称和模型类的名称不一致，导致查询失败，这个时候通常会报如下错误信息`Table 'databaseName.tableNaem' doesn't exist`
    - 解决方案是使用MP提供的另外一个注解`@TableName`来设置表与模型类之间的对应关系。

知识点：`@TableName`

|名称|@TableName|
|---|---|
|类型|类注解|
|位置|模型类定义上方|
|作用|设置当前类对应于数据库表关系|
|相关属性|value(默认)：设置数据库表名称|
## 5 DML编程控制

查询相关的操作我们已经介绍完了，紧接着我们需要对另外三个，增删改进行内容的讲解。挨个来说明下，首先是新增(insert)中的内容。
### 5.1 id生成策略控制

前面我们在新增数据的时候，主键ID是一个很长的Long类型，我们现在想要主键按照数据表字段进行自增长，在解决这个问题之前，我们先来分析一下ID的生成策略

- 不同的表，应用不同的id生成策略
    - 日志：自增（1 2 3 4）
    - 购物订单：特殊规则（线下购物发票，下次可以留意一下）
    - 外卖订单：关联地区日期等信息（这个我熟，举个例子10 04 20220921 13 14，例如10表示北京市，04表示朝阳区，20220921表示日期等）
    - 关系表：可以省略ID
    - ……
- 不同的业务采用的ID生成方式应该是不一样的，那么在MP中都提供了哪些主键生成策略，以及我们该如何进行选择?
    - 在这里我们又需要用到MP的一个注解叫`@TableId`

知识点：`@TableId`

|名称|@TableId|
|---|---|
|类型|属性注解|
|位置|模型类中用于表示主键的属性定义上方|
|作用|设置当前类中主键属性的生成策略|
|相关属性|value(默认)：设置数据库表主键名称  <br>type:设置主键属性的生成策略，值查照IdType的枚举值|
#### 5.1.1 AUTO策略

- `步骤一：`设置生成策略为AUTO
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.AUTO)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```
- `步骤二：`设置自动增量为5，将4之后的数据都删掉，防止影响我们的结果
- `步骤三：`运行新增方法
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```

会发现，新增成功，并且主键id也是从5开始

我们进入源码来看看还有什么生成策略
```java
public enum IdType {  
    AUTO(0),  
    NONE(1),  
    INPUT(2),  
    ASSIGN_ID(3),  
    ASSIGN_UUID(4),  
    /** @deprecated */  
    @Deprecated  
    ID_WORKER(3),  
    /** @deprecated */  
    @Deprecated  
    ID_WORKER_STR(3),  
    /** @deprecated */  
    @Deprecated  
    UUID(4);  
  
    private final int key;  
  
    private IdType(int key) {  
        this.key = key;  
    }  
  
    public int getKey() {  
        return this.key;  
    }  
}
```

- NONE: 不设置id生成策略
- INPUT:用户手工输入id
- ASSIGN_ID:雪花算法生成id(可兼容数值型与字符串型)
- ASSIGN_UUID:以UUID生成算法作为id生成策略
- 其他的几个策略均已过时，都将被`ASSIGN_ID`和`ASSIGN_UUID`代替掉。

拓展:  
分布式ID是什么?
- 当数据量足够大的时候，一台数据库服务器存储不下，这个时候就需要多台数据库服务器进行存储
- 比如订单表就有可能被存储在不同的服务器上
- 如果用数据库表的自增主键，因为在两台服务器上所以会出现冲突
- 这个时候就需要一个全局唯一ID,这个ID就是分布式ID。
#### 5.1.2 INPUT策略

`步骤一：`将ID生成策略改为INPUT
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.INPUT)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

`步骤二：`运行新增方法  
	注意这里需要手动设置ID了
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setId(6L);  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```
查看数据库，ID确实是我们设置的值
#### 5.1.3 ASSIGN_ID策略

`步骤一：`设置生成策略为`ASSIGN_ID`
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_ID)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

`步骤二：`运行新增方法  
	这里就不要手动设置ID了
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```

查看结果，生成的ID就是一个Long类型的数据，生成ID时，使用的是雪花算法  
雪花算法(SnowFlake),是Twitter官方给出的算法实现 是用Scala写的。其生成的结果是一个64bit大小整数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88f6c2e7948c5ab5f13d2911adb1adb6.png)


1. 1bit,不用,因为二进制中最高位是符号位，1表示负数，0表示正数。生成的id一般都是用整数，所以最高位固定为0。
2. 41bit-时间戳，用来记录时间戳，毫秒级
3. 10bit-工作机器id，用来记录工作机器id,其中高位5bit是数据中心ID其取值范围0-31，低位5bit是工作节点ID其取值范围0-31，两个组合起来最多可以容纳1024个节点
4. 序列号占用12bit，每个节点每毫秒，从0开始不断累加，最多可以累加到4095，一共可以产生4096个ID
#### 5.1.4 ASSIGN_UUID策略

- `步骤一：`设置生成策略为ASSIGN_UUID
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_UUID)  
    private String id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

- `步骤二：`修改表的主键类型  
    主键类型设置为varchar，长度要大于32，因为UUID生成的主键为32位，如果长度小的话就会导致插入失败。
- `步骤三：`运行新增方法
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```
#### 5.1.5 ID生成策略对比

介绍完了这些主键ID的生成策略，那么以后我们开发用哪个呢？

- NONE：不设置ID生成策略，MP不自动生成，约定于INPUT，所以这两种方式都需要用户手动设置（SET方法），但是手动设置的第一个问题就是容易出错，加了相同的ID造成主键冲突，为了保证主键不冲突就得做很多判定，实现起来较为复杂
- AUTO：数据库ID自增，这种策略适合在数据库服务器只有一台的情况下使用，不可作为分布式ID使用
- ASSIGN_UUID：可以在分布式的情况下使用，而且能够保证ID唯一，但是声称的主键是32位的字符串，长度过长占用空间，而且不能排序，查询性能也慢
- ASSIGN_ID：可以在分布式的情况下使用，生成的是Long类型的数字，可以排序，性能也高，但是生成的策略与服务器时间有关，如果修改了系统时间，也有可能出现重复的主键
- 综上所述，每一种主键的策略都有自己的优缺点，根据自己的项目业务需求的实际情况来使用，才是最明智的选择
#### 5.1.6 简化配置

- 模型类主键策略设置  
	如果要在项目中的每一个模型类上都需要使用相同的生成策略，比如你有Book表，User表，Student表，Score表等好多个表，如果你每一个表的主键生成策略都是ASSIGN_ID，那我们就可以用yml配置文件来简化开发，不用在每一个表的id上都加上`@TableId(type = IdType.ASSIGN_ID)`

```yml
mybatis-plus:  
  global-config:  
    db-config:  
      id-type: assign_id
```

- 数据库表与模型类的映射关系  
	MP会默认将模型类的类名名首字母小写作为表名使用，假如数据库表的名称都以`tb_`开头，那么我们就需要将所有的模型类上添加`@TableName("tb_TABLENAME")`，这样做很繁琐，有没有更简单的方式呢？

- 我们可以在配置文件中设置表的前缀
```yml
mybatis-plus:  
global-config:  
    db-config:  
    id-type: assign_id  
    table-prefix: tb_
```

设置表的前缀内容，这样MP就会拿 `tb_`加上模型类的首字母小写，就刚好组装成数据库的表名（前提是你的表名得规范命名，别瞎起花里胡哨的名）。将User类的`@TableName`注解去掉，再次运行新增方法
```java
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_ID)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```
## 6 多记录操作

这部分其实没有新内容，MP已经提供好了针对多记录的删除和查询，我们自己多看看API就好了

需求:根据传入的id集合将数据库表中的数据删除掉
```java
@Test  
void testDeleteByIds(){  
    ArrayList<Long> list = new ArrayList<>();  
    list.add(1572543345085964289L);  
    list.add(1572554951983460354L);  
    list.add(1572555035978534913L);  
    userDao.deleteBatchIds(list);  
}
```
执行成功后，数据库表中的数据就会按照指定的id进行删除。上面三个数据是我之前新增插入的，可以随便换成数据库中有的id

需求：根据传入的ID集合查询用户信息
```java
@Test  
void testSelectByIds() {  
    ArrayList<Long> list = new ArrayList<>();  
    list.add(1L);  
    list.add(2L);  
    list.add(3L);  
    for (User user : userDao.selectBatchIds(list)) {  
        System.out.println(user);  
    }  
}
```
## 7 逻辑删除

逻辑删除是删除操作中比较重要的一部分，先来讲个案例![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cca88731e73303e0e1b21ccaf0c59ab.png)
- 这是一个员工和其所办理的合同表，一个员工可以办理多张合同表
- 员工ID为1的张业绩，办理了三个合同，但是她现在想离职跳槽了，我们需要将员工表中的数据进行删除，执行DELETE操作
- 如果表在设计的时候有主外键关系，那么同时也要将合同表中的张业绩的数据删掉![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6f0e638225b86054caa2772b52ed0fe2.png)


- 后来公司要统计今年的总业绩，发现这数据咋对不上呢，业绩这么少，原因是张业绩办理的合同信息被删掉了
- 如果只删除员工，却不删除员工对应的合同表数据，那么合同的员工编号对应的员工信息不存在，那么就会产生垃圾数据，出现无主合同，根本不知道有张业绩这个人的存在
- 经过我们的分析之后，我们不应该将表中的数据删除掉，得留着，但是又得把离职的人和在职的人区分开，这样就解决了上述问题![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/459fd2c45a95c5682eae8555319eafd8.png)
- 区分的方式，就是在员工表中添加一列`deleted`，如果为0说明在职员工，如果离职则将其改完1，（0和1所代表的含义是可以自定义的）

所以对于删除操作业务问题来说有:
- 物理删除:业务数据从数据库中丢弃，执行的是delete操作
- 逻辑删除:为数据设置是否可用状态字段，删除时设置状态字段为不可用状态，数据保留在数据库中，执行的是update操作

MP中逻辑删除具体该如何实现?
- `步骤一：`修改数据库表，添加`deleted`列  
    字段名任意，类型int，长度1，默认值0（个人习惯，你随便）
- `步骤二：`实体类添加属性  
    还得修改对应的pojo类，增加delete属性（属性名也任意，对不上用`@TableField`来添加映射关系  
    标识新增的字段为逻辑删除字段，使用`@TableLogic`
```java
//表名前缀和id生成策略在yml配置文件写了  
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    //新增delete属性  
    //value为正常数据的值（在职），delval为删除数据的值（离职）  
    @TableLogic(value = "0",delval = "1")  
    private Integer deleted;  
}
```

`步骤三：`运行删除方法  
	没有就自己写一个呗
```java
@Test  
void testLogicDelete(){  
    userDao.deleteById(1);  
}
```

- 从测试结果来看，逻辑删除最后走的是update操作，执行的是`UPDATE tb_user SET deleted=1 WHERE id=? AND deleted=0`，会将指定的字段修改成删除状态对应的值。
    
- 思考：逻辑删除，对查询有没有影响呢?
    - 执行查询操作
```java
@Test  
void testSelectAll() {  
    for (User user : userDao.selectList(null)) {  
        System.out.println(user);  
    }  
}
```

从日志中可以看到执行的SQL语句如下，WHERE条件中，规定只查询deleted字段为0的数据
```sql
SELECT id,name,password,age,tel,deleted FROM tb_user WHERE deleted=0
```

- 如果还是想把已经删除的数据都查询出来该如何实现呢?
```java
@Mapper  
public interface UserDao extends BaseMapper<User> {  
    //查询所有数据包含已经被删除的数据  
    @Select("select * from tb_user")  
    public List<User> selectAll();  
}
```

如果每个表都要有逻辑删除，那么就需要在每个模型类的属性上添加`@TableLogic`注解，如何优化?

- 在配置文件中添加全局配置，如下
```yml
mybatis-plus:  
    global-config:  
    db-config:  
        # 逻辑删除字段名  
        logic-delete-field: deleted  
        # 逻辑删除字面值：未删除为0  
        logic-not-delete-value: 0  
        # 逻辑删除字面值：删除为1  
        logic-delete-value: 1
```
使用yml配置文件配置了之后，就不需要在模型类上用`@TableLogic`注解了

```sql
UPDATE tb_user SET deleted=1 WHERE id=? AND deleted=0
```

知识点：`@TableLogic`

|名称|@TableLogic|
|---|---|
|类型|属性注解|
|位置|模型类中用于表示删除字段的属性定义上方|
|作用|标识该字段为进行逻辑删除的字段|
|相关属性|value：逻辑未删除值  <br>delval:逻辑删除值|
## 8 乐观锁

### 8.1 概念

在学乐观锁之前，我们还是先由一个案例来引入

- 业务并发现象带来的问题：秒杀
    - 加入有100个商品在售，为了保证每个商品只能被一个人购买，如何保证不会超买或者重复卖
    - 对于这一类的问题，其实有很多的解决方案可以使用
    - 第一个最先想到的就是锁，锁在一台服务器中是可以解决的，但是如果在多台服务器下就没办法控制，比如12306有两台服务器，再进行卖票，在两台服务器上都添加锁的话，那也有可能会在同一时刻有两个线程在卖票，还是会出现并发问题
    - 我们接下来介绍的这种方式就是针对于小型企业的解决方案，因为数据库本身的性能就是个瓶颈，如果对其并发超过2000以上的就需要考虑其他解决方案了

简单来说，乐观锁主要解决的问题是，当要更新一条记录的时候，希望这条记录没有被别人更新
### 8.2 实现思路

- 数据库表中添加`version`字段，比如默认值给个1
- 第一个线程要修改数据之前，取出记录时，获取当前的version=1
- 第二个线程要修改数据之前，取出记录时，获取当前的version=1
- 第一个线程执行更新时
    - set version = newVersion where version = oldVersion
        - newVersion = version + 1
        - oldVersion = version
- 第二个线程执行更新时
    - set version = newVersion where version = oldVersion
        - newVersion = version + 1
        - oldVersion = version
- 假如这两个线程都来更新数据，第一个和第二个线程都可能先执行
    - 假如第一个线程先执行更新，会将version改为2
        - 那么第二个线程再更新的时候，set version = 2 where version = 1，此时数据库表的version已经是2了，所以第二个线程修改失败
    - 假如第二个线程先执行更新，会将version改为2
        - 那么第一个线程再更新的时候，set version = 2 where version = 1，此时数据库表的version已经是2了，所以第一个线程修改失败
### 8.3 实现步骤

- `步骤一：`数据库表添加列  
    加一列version，长度给个11，默认值设为1
- `步骤二：`在模型类中添加对应的属性
```java
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableLogic(value = "0", delval = "1")  
    private Integer deleted;  
    @Version  
    private Integer version;  
}
```

- `步骤三：`添加乐观锁拦截器
```java
@Configuration  
public class MpConfig {  
    @Bean  
    public MybatisPlusInterceptor mpInterceptor() {  
        //1.定义Mp拦截器  
        MybatisPlusInterceptor mpInterceptor = new MybatisPlusInterceptor();  
        //2.添加乐观锁拦截器  
        mpInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());  
        return mpInterceptor;  
    }  
}
```

- `步骤四：`执行更新操作
```java
@Test  
void testUpdate(){  
    //1. 先通过要修改的数据id将当前数据查询出来  
    User user = userDao.selectById(1L);  
    //2. 修改属性  
    user.setName("Person");  
    userDao.updateById(user);  
}
```

我们传递的是1(oldVersion)，MP会将1进行加1，变成2，然后更新回到数据库中(newVersion)

大概分析完乐观锁的实现步骤以后，我们来模拟一种加锁的情况，看看能不能实现多个人修改同一个数据的时候，只能有一个人修改成功。
```java
@Test  
void testUpdate() {  
    User userA = userDao.selectById(1L); //version=1  
    User userB = userDao.selectById(1L); //version=1  
    userB.setName("Jackson");  
    userDao.updateById(userB);  //B修改完了之后，version=2  
    userA.setName("Person");  
    //A拿到的version是1，但现在的version已经是2了，那么A在执行 UPDATE ... WHERE version = 1时，就必然会失败  
    userDao.updateById(userA);    
}
```
## 9 快速开发

### 9.1 代码生成器原理分析

官方文档地址：[https://baomidou.com/pages/981406/](https://baomidou.com/pages/981406/)

通过观察我们之前写的代码，会发现其中有很多重复的内容，于是MP抽取了这些重复的地方，做成了一个模板供我们使用  
要想完成代码自动生成，我们需要有以下内容:

- 模板: MyBatisPlus提供，可以自己提供，但是麻烦，不建议
- 数据库相关配置:读取数据库获取表和字段信息
- 开发者自定义配置:手工配置，比如ID生成策略
### 9.2 代码生成器实现

- `步骤一：`创建一个Maven项目
- `步骤二：`导入对应的jar包
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.5.1</version>  
    </parent>  
    <groupId>com.blog</groupId>  
    <artifactId>mybatisplus_04_generator</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <properties>  
        <java.version>1.8</java.version>  
    </properties>  
    <dependencies>  
        <!--spring webmvc-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <!--mybatisplus-->  
        <dependency>  
            <groupId>com.baomidou</groupId>  
            <artifactId>mybatis-plus-boot-starter</artifactId>  
            <version>3.4.1</version>  
        </dependency>  
  
        <!--druid-->  
        <dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>druid</artifactId>  
            <version>1.1.16</version>  
        </dependency>  
  
        <!--mysql-->  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <scope>runtime</scope>  
        </dependency>  
  
        <!--test-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
  
        <!--lombok-->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <version>1.18.12</version>  
        </dependency>  
  
        <!--代码生成器-->  
        <dependency>  
            <groupId>com.baomidou</groupId>  
            <artifactId>mybatis-plus-generator</artifactId>  
            <version>3.4.1</version>  
        </dependency>  
  
        <!--velocity模板引擎-->  
        <dependency>  
            <groupId>org.apache.velocity</groupId>  
            <artifactId>velocity-engine-core</artifactId>  
            <version>2.3</version>  
        </dependency>  
  
    </dependencies>  
  
    <build>  
        <plugins>  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>  
        </plugins>  
    </build>  
  
</project>
```

- `步骤三：`编写引导类
```java
@SpringBootApplication  
public class Mybatisplus04GeneratorApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(Mybatisplus04GeneratorApplication.class, args);  
    }  
  
}
```

- `步骤四：`创建代码生成类
```java
public class CodeGenerator {  
    public static void main(String[] args) {  
        //1.获取代码生成器的对象  
        AutoGenerator autoGenerator = new AutoGenerator();  
  
        //设置数据库相关配置  
        DataSourceConfig dataSource = new DataSourceConfig();  
        dataSource.setDriverName("com.mysql.cj.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC");  
        dataSource.setUsername("root");  
        dataSource.setPassword("YOURPASSWORD");  
        autoGenerator.setDataSource(dataSource);  
  
        //设置全局配置  
        GlobalConfig globalConfig = new GlobalConfig();  
        globalConfig.setOutputDir(System.getProperty("user.dir")+"/项目名/src/main/java");    //设置代码生成位置  
        globalConfig.setOpen(false);    //设置生成完毕后是否打开生成代码所在的目录  
        globalConfig.setAuthor("Kyle");    //设置作者  
        globalConfig.setFileOverride(true);     //设置是否覆盖原始生成的文件  
        globalConfig.setMapperName("%sDao");    //设置数据层接口名，%s为占位符，指代模块名称  
        globalConfig.setIdType(IdType.ASSIGN_ID);   //设置Id生成策略  
        autoGenerator.setGlobalConfig(globalConfig);  
  
        //设置包名相关配置  
        PackageConfig packageInfo = new PackageConfig();  
        packageInfo.setParent("com.aaa");   //设置生成的包名，与代码所在位置不冲突，二者叠加组成完整路径  
        packageInfo.setEntity("domain");    //设置实体类包名  
        packageInfo.setMapper("dao");   //设置数据层包名  
        autoGenerator.setPackageInfo(packageInfo);  
  
        //策略设置  
        StrategyConfig strategyConfig = new StrategyConfig();  
        strategyConfig.setInclude("tb_user");  //设置当前参与生成的表名，参数为可变参数  
        strategyConfig.setTablePrefix("tb_");  //设置数据库表的前缀名称，模块名 = 数据库表名 - 前缀名  例如： User = tb_user - tb_  
        strategyConfig.setRestControllerStyle(true);    //设置是否启用Rest风格  
        strategyConfig.setVersionFieldName("version");  //设置乐观锁字段名  
        strategyConfig.setLogicDeleteFieldName("deleted");  //设置逻辑删除字段名  
        strategyConfig.setEntityLombokModel(true);  //设置是否启用lombok  
        autoGenerator.setStrategy(strategyConfig);  
        //2.执行生成操作  
        autoGenerator.execute();  
    }  
}
```
对于代码生成器中的代码内容，我们可以直接从官方文档中获取代码进行修改，`https://baomidou.com/pages/981406/`

- `步骤五：`运行程序
	运行成功后，会在当前项目中生成很多代码，代码包含`controller`,`service`，`mapper`和`entity`等
	
	至此代码生成器就已经完成工作，我们能快速根据数据库表来创建对应的类，简化我们的代码开发。
	
	初期还是不建议直接使用代码生成器，还是多自己手写几遍比较好
### 9.3 MP中Service的CRUD

回顾我们之前业务层代码的编写，编写接口和对应的实现类:
```java
public interface UserService{  
	  
}  
  
@Service  
public class UserServiceImpl implements UserService{  
  
}
```

接口和实现类有了以后，需要在接口和实现类中声明方法
```java
public interface UserService{  
	public List<User> findAll();  
}  
  
@Service  
public class UserServiceImpl implements UserService{  
    @Autowired  
    private UserDao userDao;  
      
	public List<User> findAll(){  
        return userDao.selectList(null);  
    }  
}
```

MP看到上面的代码以后就说这些方法也是比较固定和通用的，那我来帮你抽取下，所以MP提供了一个Service接口和实现类，分别是:`IService`和`ServiceImpl`,后者是对前者的一个具体实现。

以后我们自己写的Service就可以进行如下修改:
```java
public interface UserService extends IService<User>{  
	  
}  
  
@Service  
public class UserServiceImpl extends ServiceImpl<UserDao, User> implements UserService{  
  
}
```

修改以后的好处是，MP已经帮我们把业务层的一些基础的增删改查都已经实现了，可以直接进行使用。

编写测试类进行测试:
```java
@SpringBootTest  
class Mybatisplus04GeneratorApplicationTests {  
  
    @AutoWired
    private IUserService userService;  
  
    @Test  
    void testFindAll() {  
        List<User> list = userService.list();  
        System.out.println(list);  
    }  
}
```


## 1 本文摘要

- 基于MyBatisPlus完成标准Dao的增删改查功能
- 掌握MyBatisPlus中的分页及条件查询构建
- 掌握主键ID的生成策略
- 了解MyBatisPlus的代码生成器
## 2 MyBatisPlus入门案例与简介

由于我们之前有MyBatis开发的基础，所以我们现在直接来一个入门案例，来体现MyBatisPlus的强大之处
### 2.1 入门案例

- MyBatisPlus（简称MP）是基于MyBatis框架基础上开发的增强型工具，旨在简化开发，提高效率
- 开发方式
    - 基于MyBatis使用MyBatisPlus
    - 基于Spring使用MyBatisPlus
    - ==基于SpringBoot使用MyBatisPlus(重点)==

由于我们刚刚才学完SpringBoot，所以我们现在直接使用SpringBoot来构建项目，官网的快速开始也是直接用的SpringBoot

- `步骤一：`创建数据库和表  
    数据库随便挑一个自己喜欢的吧
```sql
CREATE TABLE user (  
    id bigint(20) primary key auto_increment,  
    name varchar(32) not null,  
    password  varchar(32) not null,  
    age int(3) not null ,  
    tel varchar(32) not null  
);  
insert into user values(1,'Tom','tom',3,'18866668888');  
insert into user values(2,'Jerry','jerry',4,'16688886666');  
insert into user values(3,'Jock','123456',41,'18812345678');  
insert into user values(4,'略略略','nigger',15,'4006184000');
```
- `步骤二：`创建SpringBoot工程  
    只需要勾选MySQL，不用勾选MyBatis了

- `步骤三：`补全依赖  
    导入德鲁伊和MyBatisPlus的坐标
```xml
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-boot-starter</artifactId>  
    <version>3.4.1</version>  
</dependency>  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.16</version>  
</dependency>
```

- `步骤四：`编写数据库连接四要素  
	还是将application的后缀名改为yml，以后配置都是用yml来配置  
	注意要设置一下时区，不然可能会报错（指高版本的mysql）
```yml
spring:  
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot_db?serverTimezone=UTC  
    username: root  
    password: YOUSONOFABTICH.  
  
# mybatis的日志信息  
mybatis-plus:  
  configuration:  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

- `步骤五：`根据数据表来创建对应的模型类  
	注意id是Long类型，至于为什么是Long，接着往下看
```java
@Data
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- `步骤六：`创建dao接口  
```java
@Mapper  
public interface UserDao extends BaseMapper<User>{  
}
```
只需要在类上方加一个`@Mapper`注解，同时继承`BaseMapper<>`，泛型写创建的模型类的类型  

- `步骤七：`赶快来测试！！  
	SpringBoot的测试类也是简单的一批，只需要一个`@SpringBootTest`注解就能完成（创建SpringBoot工程的时候已经帮我们自动弄好了）  
	测试类里需要什么东西就用`@Autowired`自动装配，测试方法上用`@Test`注解
```java
@SpringBootTest  
class MybatisplusApplicationTests {  
  
    @Autowired  
    private UserDao userDao;  
  
    @Test  
    void contextLoads() {  
        List<User> users = userDao.selectList(null);  
        for (User b : users) {  
            System.out.println(b);  
        }  
    }  
}
```

selectList() 方法的参数为 MP 内置的条件封装器 Wrapper，所以不填写就是无任何条件

现在运行测试方法，看看控制台![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36b3aa4b3995574da5aa679db3d76719.png)


原理，慢慢往后看
### 2.2 MyBatis-Plus简介

MyBatisPlus的官网为:[https://mp.baomidou.com/](https://mp.baomidou.com/) ，没错就是个拼音，苞米豆，因为域名被抢注了，但是粉丝也捐赠了一个 [https://mybatis.plus](https://mybatis.plus/) 域名

MP旨在成为MyBatis的最好搭档，而不是替换掉MyBatis，从名称上来看也是这个意思，一个MyBatis的plus版本，在原有的MyBatis上做增强，其底层仍然是MyBatis的东西，所以我们当然也可以在MP中写MyBatis的内容

对于MP的深入学习，可以多看看官方文档，锻炼自己自学的能力，毕竟不是所有知识都有像这样的网课，更多的还是自己看文档，挖源码。

MP的特性：

- `无侵入：`只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- `损耗小：`启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- `强大的 CRUD 操作：`内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- `支持 Lambda 形式调用`：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- `支持主键自动生成：`支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- `支持 ActiveRecord 模式：`支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- `支持自定义全局通用操作：`支持全局通用方法注入（ Write once, use anywhere ）
- `内置代码生成器：`采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- `内置分页插件：`基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- `分页插件支持多种数据库：`支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- `内置性能分析插件：`可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- `内置全局拦截插件：`提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
### 2.3 小结

SpringBoot集成MyBatisPlus非常的简单，只需要导入`MyBatisPlus`的坐标，然后令dao类继承`BaseMapper`，写上泛型，类上方加`@Mapper`注解

可能存在的疑问：
- 我甚至都没写在哪个表里查，为什么能自动识别是在我刚刚创建的表里查？
    - 注意我们创建的表，和对应的模型类，是同一个名，默认情况是在同名的表中查找
- 那我要是表明和模型类的名不一样，那咋整？
    - 在模型类的上方加上`@TableName`注解
        - 例如数据表叫`tb_user`但数据类叫`User`，那么就在User类上加`@TableName("tb_user")`注解
## 3 标准数据层开发

### 3.1 标准的CRUD使用

先来看看MP给我们提供了哪些方法

|功能|自定义接口|MP接口|
|---|---|---|
|新增|boolean save(T t)|int insert(T t)|
|删除|boolean delete(int id)|int deleteById(Serializable id)|
|修改|boolean update(T t)|int updateById(T t)|
|根据id查询|T getById(int id)|T selectById(Serializable id)|
|查询全部|List\<T> getAll()|List\<T> selectList()|
|分页查询|PageInfo\<T> getAll(int page,int size)|IPage\<T> selectPage(IPage\<T> page)|
|按条件查询|List\<T> getAll(Condition condition)|IPage\<T> selectPage(Wrapper\<T>queryWrapper)|

我们就按照上表中的方法来演示
### 3.2 新增

```java
int insert(T t)
```

参数类型是泛型，也就是我们当初继承BaseMapper的时候，填的泛型，返回值是int类型，0代表添加失败，1代表添加成功
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Seto");  
    user.setAge(23);  
    user.setTel("4005129421");  
    user.setPassword("MUSICIAN");  
    userDao.insert(user);  
}
```

随便写一个User的数据，运行程序，然后去数据库看看新增是否成功![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2b508629ca38119f114dfbc2980a53c.png)
这个主键自增id看着有点奇怪，但现在你知道为什么要将id设为long类型了吧
### 3.3 删除

```java
int deleteByIds(Serializable id);
```

- 参数类型为什么是一个序列化类`Serializable`
    - 通过查看String的源码，你会发现String实现了Serializable接口，而且Number类也实现了Serializable接口
    - Number类又是Float，Double，Long等类的父类
    - 那现在能作为主键的数据类型，都已经是Serializable类型的子类了
    - MP使用Serializable类型当做参数类型，就好比我们用Object类型来接收所有类型一样
- 返回值类型是int
    - 数据删除成功返回1
    - 未删除数据返回0。
- 那下面我们就来删除刚刚添加的数据，注意末尾加个L
```java
@Test  
void testDelete(){  
    userDao.deleteById(1572364408896622593L);  
}
```

删除完毕之后，刷新数据库，看看是否删除成功
### 3.4 修改

```java
int updateById(T t);
```

- T:泛型，需要修改的数据内容，注意因为是根据ID进行修改，所以传入的对象中需要有ID属性值
- int:返回值
    - 修改成功后返回1
    - 未修改数据返回0

```java
@Test  
void testUpdate(){  
    User user = new User();  
    user.setId(1L);  
    user.setName("Alen");  
    userDao.updateById(user);  
}
```
==修改功能只修改指定的字段，未指定的字段保持原样==
### 3.5 根据ID查询

```java
T selectById (Serializable id)
```

- Serializable：参数类型,主键ID的值
- T:根据ID查询只会返回一条数据

```java
@Test  
void testSelectById(){  
    User user = userDao.selectById(1);  
    System.out.println(user);  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3feb58c50ee830be4c77ba995d26183f.png)


### 3.6 查询全部

```java
List<T> selectList(Wrapper<T> queryWrapper)
```

Wrapper：用来构建条件查询的条件，目前我们没有可直接传为Null

```java
@Test  
void testSelectAll() {  
    List<User> users = userDao.selectList(null);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c0a9eef85147fba46f50bb14f5752bf.png)
- 方法都测试完了，那你们有没有想过，这些方法都是谁提供的呢？
    - 想都不用想，肯定是我们当初继承的`BaseMapper`，里面的方法还有很多，后面我们再慢慢学习
### 3.7 Lombok

- 代码写到这，我们发现之前的dao接口，都不用我们自己写了，只需要继承BaseMapper，用他提供的方法就好了
- 但是现在我还想偷点懒，毕竟懒是第一生产力，之前我们手写模型类的时候，创建好对应的属性，然后用IDEA的Alt+Insert快捷键，快速生成get和set方法，toSring，各种构造器（有需要的话）等
- U1S1项目做这么久，写模型类都给我写烦了，有没有更简单的方式呢？
    - 答案当然是有的，可以使用Lombok，一个Java类库，提供了一组注解，来简化我们的POJO模型类开发

具体步骤如下
- `步骤一：`添加Lombok依赖
```xml
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <!--<version>1.18.12</version>-->  
</dependency>
```
版本不用谢，SpringBoot中已经管理了lombok的版本

- `步骤二：`在模型类上添加注解
- Lombok常见的注解有:
	- `@Setter:`为模型类的属性提供setter方法
	- `@Getter:`为模型类的属性提供getter方法
	- `@ToString:`为模型类的属性提供toString方法
	- `@EqualsAndHashCode:`为模型类的属性提供equals和hashcode方法
	- `@Data:`是个组合注解，包含上面的注解的功能
	- `@NoArgsConstructor:`提供一个无参构造函数
	- `@AllArgsConstructor:`提供一个包含所有参数的构造函数
```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

说明:Lombok只是简化模型类的编写，我们之前的方法也能用  
例如你有特殊的构造器需求，只想要name和password这两个参数，那么可以手写一个
```java
public User(String name, String password) {  
    this.name = name;  
    this.password = password;  
}
```
### 3.8 分页功能

基础的增删改查功能就完成了，现在我们来进行分页功能的学习
```java
IPage<T> selectPage(IPage<T> page, Wrapper<T> queryWrapper)
```

- IPage用来构建分页查询条件
- Wrapper：用来构建条件查询的条件，暂时没有条件可以传一个null
- 返回值IPage是什么意思，后面我们会说明

具体的使用步骤如下

- `步骤一：`调用方法传入参数获取返回值
```java
@Test  
void testSelectPage() {  
    IPage<User> page = new Page<>(1, 3);  
    userDao.selectPage(page, null);  
    System.out.println("当前页码" + page.getCurrent());  
    System.out.println("本页条数" + page.getSize());  
    System.out.println("总页数" + page.getPages());  
    System.out.println("总条数" + page.getTotal());  
    System.out.println(page.getRecords());  
}
```

- `步骤二：`设置分页拦截器
```java
//注意要被扫描到
@Configuration
public class MybatisPlusConfig {  
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor(){  
        MybatisPlusInterceptor myInterceptor = new MybatisPlusInterceptor();  
        myInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());  
        return myInterceptor;  
    }  
}
```

- `步骤三：`运行测试程序  
	运行程序，结果如下，符合我们的预期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48f0f5a5ef7d4d12e08f7dfb86cfc781.png)

## 4 DQL编程控制

增删改查四个操作中，查询是非常重要的也是非常复杂的操作，这部分我们主要学习的内容有:

- 条件查询方式
- 查询投影
- 查询条件设定
- 字段映射与表名映射
### 4.1 条件查询

#### 4.1.1 条件查询的类

- MP将复杂的SQL查询语句都做了封装，使用编程的方式来完成查询条件的组合
- 之前我们在写CRUD时，都看到了一个Wrapper类，我们当初都是赋一个null值，但其实这个类就是用来查询的
#### 4.1.2 构建条件查询

- `QueryWrapper`  
	小于用lt，大于用gt  
	回想之前我们在html页面中，如果需要用到小于号或者大于号，需要用对应的html实体来替换  
	小于号的实体是 `&lt;`，大于号的实体是`&gt;`
```java
@Test  
void testQueryWrapper(){  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    //条件为 age字段小于18  
    qw.lt("age",18);  
    List<User> userList = userDao.selectList(qw);  
    System.out.println(userList);  
}
```

运行测试方法，结果如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e24f7d636af92257de4422a67580670e.png)
这种方法有个弊端，那就是==字段名是字符串类型，没有提示信息和自动补全==，如果写错了，那就查不出来

- `QueryWrapper`的基础上，使用`lambda`
```java
@Test  
void testQueryWrapper(){  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.lambda().lt(User::getAge,18);  
    List<User> userList = userDao.selectList(qw);  
    System.out.println(userList);  
}
```
`ser::getAget`,为lambda表达式中的，`类名::方法名`
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b933a6a63feb3913540fc8f791b553fb.png)


- `LambdaQueryWrapper`  
	方式二解决了方式一的弊端，但是要多些一个lambda()，那方式三就来解决方式二的弊端，使用LambdaQueryWrapper，就可以不写lambda()
```java
@Test  
void testQueryWrapper(){  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.lt(User::getAge,18);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```
#### 4.1.3 多条件查询

上面三种都是单条件的查询，那我们现在想进行多条件的查询，该如何编写代码呢？  

需求：查询表中年龄在10~30岁的用户信息
```java
@Test  
void testQueryWrapper(){  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    //大于10  
    lqw.gt(User::getAge,10);  
    //小于30  
    lqw.lt(User::getAge,30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```

- 构建多条件的时候，我们还可以使用链式编程
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.gt(User::getAge, 10).lt(User::getAge, 30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```

- 可能存在的疑问
    - MP怎么就知道你这俩条件是AND的关系呢，那我要是想用OR的关系，该咋整
- 解答
    - ==默认就是AND的关系，如果需要OR关系，用or()链接就可以了==
```java
lqw.gt(User::getAge, 10).or().lt(User::getAge, 30);
```

需求：查询年龄小于10，或者年龄大于30的用户信息
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.lt(User::getAge, 10).or().gt(User::getAge, 30);  
    List<User> userList = userDao.selectList(lqw);  
    System.out.println(userList);  
}
```
#### 4.1.4 null值的判定

- 我们在做条件查询的时候，一般都会有很多条件供用户查询
- 这些条件用户可以选择用，也可以选择不用
- 之前我们是通过动态SQL来实现的
```xml
<select id="selectByPageAndCondition" resultMap="brandResultMap">  
    select *  
    from tb_brand  
    <where>  
        <if test="brand.brandName != null and brand.brandName != '' ">  
            and  brand_name like #{brand.brandName}  
        </if>  
  
        <if test="brand.companyName != null and brand.companyName != '' ">  
            and  company_name like #{brand.companyName}  
        </if>  
  
        <if test="brand.status != null">  
            and  status = #{brand.status}  
        </if>  
    </where>  
    limit #{begin} , #{size}  
</select>
```

那现在我们来试试在MP里怎么写
	需求:查询数据库表中，根据输入年龄范围来查询符合条件的记录  
	用户在输入值的时候，  
	​如果只输入第一个框，说明要查询大于该年龄的用户  
	​如果只输入第二个框，说明要查询小于该年龄的用户  
	如果两个框都输入了，说明要查询年龄在两个范围之间的用户

- 问题一：后台如果想接收前端的两个数据，该如何接收?
	- 我们可以使用两个简单数据类型，也可以使用一个模型类，但是User类中目前只有一个age属性
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- 使用一个age属性，如何去接收页面上的两个值呢?这个时候我们有两个解决方案
	- 方案一：添加属性age2,这种做法可以但是会影响到原模型类的属性内容
	- 方案二：新建一个模型类,让其继承User类，并在其中添加age2属性，UserQuery在拥有User属性后同时添加了age2属性
```java
@Data  
@TableName("tb_user")  
public class UserQuery extends User{  
    private Integer age2;  
}
```

- 环境准备好后，我们来实现下刚才的需求：
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    UserQuery uq = new UserQuery();  
    uq.setAge(10);  
    uq.setAge2(30);  
    if (null != uq.getAge()) {  
        lqw.gt(User::getAge, uq.getAge());  
    }  
    if (null != uq.getAge2()) {  
        lqw.lt(User::getAge, uq.getAge2());  
    }  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    }  
}
```
- 上面的写法可以完成条件为非空的判断，但是问题很明显，如果条件多的话，每个条件都需要判断，代码量就比较大，来看MP给我们提供的简化方式

- lt还有一个重载的方法，当condition为true时，添加条件，为false时，不添加条件
```java
public Children lt(boolean condition, R column, Object val) {  
    return this.addCondition(condition, column, SqlKeyword.LT, val);  
}
```

- 故我们可以把if的判断操作，放到lt和gt方法中当做参数来写
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    UserQuery uq = new UserQuery();  
    uq.setAge(10);  
    uq.setAge2(30);  
    lqw.gt(null != uq.getAge(), User::getAge, uq.getAge())  
       .lt(null != uq.getAge2(), User::getAge, uq.getAge2());  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    } 
}
```
### 4.2 查询投影

#### 4.2.1 查询指定字段

目前我们在查询数据的时候，什么都没有做默认就是查询表中所有字段的内容，我们所说的查询投影即不查询所有字段，只查询出指定内容的数据。

具体如何来实现?
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.select(User::getName,User::getName);  
    for (User user : userDao.selectList(lqw)) {  
        System.out.println(user);  
    }  
}
```

select(…)方法用来设置查询的字段列，可以设置多个
```java
lqw.select(User::getName,User::getName);
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e73e7d15904985ce124435063e81fc6.png)


如果使用的不是lambda，就需要手动指定字段
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("name", "age");  
    for (User user : userDao.selectList(qw)) {  
        System.out.println(user);  
    }  
}
```
#### 4.2.2 聚合查询

需求:聚合函数查询，完成count、max、min、avg、sum的使用

- count:总记录数
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("count(*) as count");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- max:最大值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("max(age) as maxAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- min:最小值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("min(age) as minAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- avg:平均值
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("avg(age) as avgAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
- sum:求和
```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("sum(age) as sumAge");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```
#### 4.2.3 分组查询

```java
@Test  
void testQueryWrapper() {  
    QueryWrapper<User> qw = new QueryWrapper<>();  
    qw.select("max(age) as maxAge");  
    qw.groupBy("tel");  
    for (Map<String, Object> selectMap : userDao.selectMaps(qw)) {  
        System.out.println(selectMap);  
    }  
}
```

控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4074e0a082dda50e524590723e955a0d.png)


注意：
- ==聚合与分组查询，无法使用lambda表达式来完成==
- MP只是对MyBatis的增强，如果MP实现不了，我们可以直接在DAO接口中使用MyBatis的方式实现
### 4.3 查询条件

前面我们只使用了lt()和gt(),除了这两个方法外，MP还封装了很多条件对应的方法

- 范围匹配（> 、 = 、between）
- 模糊匹配（like）
- 空判定（null）
- 包含性匹配（in）
- 分组（group）
- 排序（order）
- ……
#### 4.3.1 等值查询

需求:根据用户名和密码查询用户信息
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.eq(User::getName,"Seto").eq(User::getPassword,"MUSICIAN");  
    User user = userDao.selectOne(qw);  
    System.out.println(user);  
}
```

- eq()： 相当于 `=`,对应的sql语句为
```sql
SELECT * FROM tb_user WHERE name = 'seto' AND password = 'MUSICIAN';
```
- selectList：查询结果为多个或者单个
- selectOne:查询结果为单个
#### 4.3.2 范围查询

需求:对年龄进行范围查询，使用lt()、le()、gt()、ge()、between()进行范围查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.between(User::getAge,10,30);  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

- gt():大于(>)
- ge():大于等于(>=)
- lt():小于(<)
- lte():小于等于(<=)
- between():between ? and ?
#### 4.3.3 模糊查询

需求:查询表中name属性的值以`J`开头的用户信息,使用like进行模糊查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.likeRight(User::getName,"J");  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```

- like():前后加百分号,如 %J%，相当于包含J的name
- likeLeft():前面加百分号,如 %J，相当于J结尾的name
- likeRight():后面加百分号,如 J%，相当于J开头的name

需求:查询表中name属性的值包含`e`的用户信息,使用like进行模糊查询
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    qw.like(User::getName,"e");  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```
#### 4.3.4 排序查询

需求:查询所有数据，然后按照age降序
```java
@Test  
void testQueryWrapper() {  
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();  
    /**  
        * condition ：条件，返回boolean，  
		当condition为true，进行排序，如果为false，则不排序  
        * isAsc:是否为升序，true为升序，false为降序  
        * columns：需要操作的列  
    */  
    qw.orderBy(true,false,User::getAge);  
    List<User> users = userDao.selectList(qw);  
    for (User u : users) {  
        System.out.println(u);  
    }  
}
```
遇到想用的功能，先自己用一个试试，方法名和形参名都很见名知意，遇到不确定的用法，再去官方文档查阅资料
### 4.4 映射匹配兼容性

在上面的案例中，我们做查询的时候，数据表中的字段名与模型类中的属性名一致，查询的时候没有问题，那么问题就来了

- `问题一：`表字段与模型类编码属性不一致
    - 当表的列名和模型类的属性名发生不一致，就会导致数据封装不到模型对象，这个时候就需要其中一方做出修改，那如果前提是两边都不能改又该如何解决?
    - MP给我们提供了一个注解`@TableField`,使用该注解可以实现模型类属性名和表的列名之间的映射关系
    - 例如表中密码字段为`pwd`，而模型类属性名为`password`，那我们就可以用`@TableField`注解来实现他们之间的映射关系
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField("pwd")  
    private String password;  
    private Integer age;  
    private String tel;  
}
```

- `问题二：`编码中添加了数据库中未定义的属性
	- 当模型类中多了一个数据库表不存在的字段，就会导致生成的sql语句中在select的时候查询了数据库不存在的字段，程序运行就会报错，错误信息为:`Unknown column '多出来的字段名称' in 'field list'`
	- 具体的解决方案用到的还是`@TableField`注解，它有一个属性叫`exist`，设置该字段是否在数据库表中存在，如果设置为false则不存在，生成sql语句查询的时候，就不会再查询该字段了。
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField("pwd")  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

- `问题三：`采用默认查询开放了更多的字段查看权限
	- 查询表中所有的列的数据，就可能把一些敏感数据查询到返回给前端，这个时候我们就需要限制哪些字段默认不要进行查询。解决方案是`@TableField`注解的一个属性叫`select`，该属性设置默认是否需要查询该字段的值，true(默认值)表示默认查询该字段，false表示默认不查询该字段。
	- 例如像密码这种的敏感字段，不应该查询出来作为JSON返回给前端，不安全
```java
@TableName("tb_user")  
@Data  
public class User {  
    private Long id;  
    private String name;  
    @TableField(value = "pwd",select = false)  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

知识点：`@TableField`

|名称|@TableField|
|---|---|
|类型|属性注解|
|位置|模型类属性定义上方|
|作用|设置当前属性对应的数据库表中的字段关系|
|相关属性|value(默认)：设置数据库表字段名称  <br>exist:设置属性在数据库表字段中是否存在，默认为true，此属性不能与value合并使用  <br>select:设置属性是否参与查询，此属性与select()映射配置不冲突|
- `问题四：`表名与编码开发设计不同步
    - 这个问题其实我们在一开始就解决过了，现在再来回顾一遍
    - 该问题主要是表的名称和模型类的名称不一致，导致查询失败，这个时候通常会报如下错误信息`Table 'databaseName.tableNaem' doesn't exist`
    - 解决方案是使用MP提供的另外一个注解`@TableName`来设置表与模型类之间的对应关系。

知识点：`@TableName`

|名称|@TableName|
|---|---|
|类型|类注解|
|位置|模型类定义上方|
|作用|设置当前类对应于数据库表关系|
|相关属性|value(默认)：设置数据库表名称|
## 5 DML编程控制

查询相关的操作我们已经介绍完了，紧接着我们需要对另外三个，增删改进行内容的讲解。挨个来说明下，首先是新增(insert)中的内容。
### 5.1 id生成策略控制

前面我们在新增数据的时候，主键ID是一个很长的Long类型，我们现在想要主键按照数据表字段进行自增长，在解决这个问题之前，我们先来分析一下ID的生成策略

- 不同的表，应用不同的id生成策略
    - 日志：自增（1 2 3 4）
    - 购物订单：特殊规则（线下购物发票，下次可以留意一下）
    - 外卖订单：关联地区日期等信息（这个我熟，举个例子10 04 20220921 13 14，例如10表示北京市，04表示朝阳区，20220921表示日期等）
    - 关系表：可以省略ID
    - ……
- 不同的业务采用的ID生成方式应该是不一样的，那么在MP中都提供了哪些主键生成策略，以及我们该如何进行选择?
    - 在这里我们又需要用到MP的一个注解叫`@TableId`

知识点：`@TableId`

|名称|@TableId|
|---|---|
|类型|属性注解|
|位置|模型类中用于表示主键的属性定义上方|
|作用|设置当前类中主键属性的生成策略|
|相关属性|value(默认)：设置数据库表主键名称  <br>type:设置主键属性的生成策略，值查照IdType的枚举值|
#### 5.1.1 AUTO策略

- `步骤一：`设置生成策略为AUTO
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.AUTO)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```
- `步骤二：`设置自动增量为5，将4之后的数据都删掉，防止影响我们的结果
- `步骤三：`运行新增方法
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```

会发现，新增成功，并且主键id也是从5开始

我们进入源码来看看还有什么生成策略
```java
public enum IdType {  
    AUTO(0),  
    NONE(1),  
    INPUT(2),  
    ASSIGN_ID(3),  
    ASSIGN_UUID(4),  
    /** @deprecated */  
    @Deprecated  
    ID_WORKER(3),  
    /** @deprecated */  
    @Deprecated  
    ID_WORKER_STR(3),  
    /** @deprecated */  
    @Deprecated  
    UUID(4);  
  
    private final int key;  
  
    private IdType(int key) {  
        this.key = key;  
    }  
  
    public int getKey() {  
        return this.key;  
    }  
}
```

- NONE: 不设置id生成策略
- INPUT:用户手工输入id
- ASSIGN_ID:雪花算法生成id(可兼容数值型与字符串型)
- ASSIGN_UUID:以UUID生成算法作为id生成策略
- 其他的几个策略均已过时，都将被`ASSIGN_ID`和`ASSIGN_UUID`代替掉。

拓展:  
分布式ID是什么?
- 当数据量足够大的时候，一台数据库服务器存储不下，这个时候就需要多台数据库服务器进行存储
- 比如订单表就有可能被存储在不同的服务器上
- 如果用数据库表的自增主键，因为在两台服务器上所以会出现冲突
- 这个时候就需要一个全局唯一ID,这个ID就是分布式ID。
#### 5.1.2 INPUT策略

`步骤一：`将ID生成策略改为INPUT
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.INPUT)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

`步骤二：`运行新增方法  
	注意这里需要手动设置ID了
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setId(6L);  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```
查看数据库，ID确实是我们设置的值
#### 5.1.3 ASSIGN_ID策略

`步骤一：`设置生成策略为`ASSIGN_ID`
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_ID)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

`步骤二：`运行新增方法  
	这里就不要手动设置ID了
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```

查看结果，生成的ID就是一个Long类型的数据，生成ID时，使用的是雪花算法  
雪花算法(SnowFlake),是Twitter官方给出的算法实现 是用Scala写的。其生成的结果是一个64bit大小整数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88f6c2e7948c5ab5f13d2911adb1adb6.png)


1. 1bit,不用,因为二进制中最高位是符号位，1表示负数，0表示正数。生成的id一般都是用整数，所以最高位固定为0。
2. 41bit-时间戳，用来记录时间戳，毫秒级
3. 10bit-工作机器id，用来记录工作机器id,其中高位5bit是数据中心ID其取值范围0-31，低位5bit是工作节点ID其取值范围0-31，两个组合起来最多可以容纳1024个节点
4. 序列号占用12bit，每个节点每毫秒，从0开始不断累加，最多可以累加到4095，一共可以产生4096个ID
#### 5.1.4 ASSIGN_UUID策略

- `步骤一：`设置生成策略为ASSIGN_UUID
```java
@TableName("tb_user")  
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_UUID)  
    private String id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableField(exist = false)  
    private Integer online;  
}
```

- `步骤二：`修改表的主键类型  
    主键类型设置为varchar，长度要大于32，因为UUID生成的主键为32位，如果长度小的话就会导致插入失败。
- `步骤三：`运行新增方法
```java
@Test  
void testInsert(){  
    User user = new User();  
    user.setName("Helsing");  
    user.setAge(531);  
    user.setPassword("HELL_SING");  
    user.setTel("4006669999");  
    userDao.insert(user);  
}
```
#### 5.1.5 ID生成策略对比

介绍完了这些主键ID的生成策略，那么以后我们开发用哪个呢？

- NONE：不设置ID生成策略，MP不自动生成，约定于INPUT，所以这两种方式都需要用户手动设置（SET方法），但是手动设置的第一个问题就是容易出错，加了相同的ID造成主键冲突，为了保证主键不冲突就得做很多判定，实现起来较为复杂
- AUTO：数据库ID自增，这种策略适合在数据库服务器只有一台的情况下使用，不可作为分布式ID使用
- ASSIGN_UUID：可以在分布式的情况下使用，而且能够保证ID唯一，但是声称的主键是32位的字符串，长度过长占用空间，而且不能排序，查询性能也慢
- ASSIGN_ID：可以在分布式的情况下使用，生成的是Long类型的数字，可以排序，性能也高，但是生成的策略与服务器时间有关，如果修改了系统时间，也有可能出现重复的主键
- 综上所述，每一种主键的策略都有自己的优缺点，根据自己的项目业务需求的实际情况来使用，才是最明智的选择
#### 5.1.6 简化配置

- 模型类主键策略设置  
	如果要在项目中的每一个模型类上都需要使用相同的生成策略，比如你有Book表，User表，Student表，Score表等好多个表，如果你每一个表的主键生成策略都是ASSIGN_ID，那我们就可以用yml配置文件来简化开发，不用在每一个表的id上都加上`@TableId(type = IdType.ASSIGN_ID)`

```yml
mybatis-plus:  
  global-config:  
    db-config:  
      id-type: assign_id
```

- 数据库表与模型类的映射关系  
	MP会默认将模型类的类名名首字母小写作为表名使用，假如数据库表的名称都以`tb_`开头，那么我们就需要将所有的模型类上添加`@TableName("tb_TABLENAME")`，这样做很繁琐，有没有更简单的方式呢？

- 我们可以在配置文件中设置表的前缀
```yml
mybatis-plus:  
global-config:  
    db-config:  
    id-type: assign_id  
    table-prefix: tb_
```

设置表的前缀内容，这样MP就会拿 `tb_`加上模型类的首字母小写，就刚好组装成数据库的表名（前提是你的表名得规范命名，别瞎起花里胡哨的名）。将User类的`@TableName`注解去掉，再次运行新增方法
```java
@Data  
public class User {  
    @TableId(type = IdType.ASSIGN_ID)  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
}
```
## 6 多记录操作

这部分其实没有新内容，MP已经提供好了针对多记录的删除和查询，我们自己多看看API就好了

需求:根据传入的id集合将数据库表中的数据删除掉
```java
@Test  
void testDeleteByIds(){  
    ArrayList<Long> list = new ArrayList<>();  
    list.add(1572543345085964289L);  
    list.add(1572554951983460354L);  
    list.add(1572555035978534913L);  
    userDao.deleteBatchIds(list);  
}
```
执行成功后，数据库表中的数据就会按照指定的id进行删除。上面三个数据是我之前新增插入的，可以随便换成数据库中有的id

需求：根据传入的ID集合查询用户信息
```java
@Test  
void testSelectByIds() {  
    ArrayList<Long> list = new ArrayList<>();  
    list.add(1L);  
    list.add(2L);  
    list.add(3L);  
    for (User user : userDao.selectBatchIds(list)) {  
        System.out.println(user);  
    }  
}
```
## 7 逻辑删除

逻辑删除是删除操作中比较重要的一部分，先来讲个案例![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cca88731e73303e0e1b21ccaf0c59ab.png)
- 这是一个员工和其所办理的合同表，一个员工可以办理多张合同表
- 员工ID为1的张业绩，办理了三个合同，但是她现在想离职跳槽了，我们需要将员工表中的数据进行删除，执行DELETE操作
- 如果表在设计的时候有主外键关系，那么同时也要将合同表中的张业绩的数据删掉![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6f0e638225b86054caa2772b52ed0fe2.png)


- 后来公司要统计今年的总业绩，发现这数据咋对不上呢，业绩这么少，原因是张业绩办理的合同信息被删掉了
- 如果只删除员工，却不删除员工对应的合同表数据，那么合同的员工编号对应的员工信息不存在，那么就会产生垃圾数据，出现无主合同，根本不知道有张业绩这个人的存在
- 经过我们的分析之后，我们不应该将表中的数据删除掉，得留着，但是又得把离职的人和在职的人区分开，这样就解决了上述问题![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/459fd2c45a95c5682eae8555319eafd8.png)
- 区分的方式，就是在员工表中添加一列`deleted`，如果为0说明在职员工，如果离职则将其改完1，（0和1所代表的含义是可以自定义的）

所以对于删除操作业务问题来说有:
- 物理删除:业务数据从数据库中丢弃，执行的是delete操作
- 逻辑删除:为数据设置是否可用状态字段，删除时设置状态字段为不可用状态，数据保留在数据库中，执行的是update操作

MP中逻辑删除具体该如何实现?
- `步骤一：`修改数据库表，添加`deleted`列  
    字段名任意，类型int，长度1，默认值0（个人习惯，你随便）
- `步骤二：`实体类添加属性  
    还得修改对应的pojo类，增加delete属性（属性名也任意，对不上用`@TableField`来添加映射关系  
    标识新增的字段为逻辑删除字段，使用`@TableLogic`
```java
//表名前缀和id生成策略在yml配置文件写了  
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    //新增delete属性  
    //value为正常数据的值（在职），delval为删除数据的值（离职）  
    @TableLogic(value = "0",delval = "1")  
    private Integer deleted;  
}
```

`步骤三：`运行删除方法  
	没有就自己写一个呗
```java
@Test  
void testLogicDelete(){  
    userDao.deleteById(1);  
}
```

- 从测试结果来看，逻辑删除最后走的是update操作，执行的是`UPDATE tb_user SET deleted=1 WHERE id=? AND deleted=0`，会将指定的字段修改成删除状态对应的值。
    
- 思考：逻辑删除，对查询有没有影响呢?
    - 执行查询操作
```java
@Test  
void testSelectAll() {  
    for (User user : userDao.selectList(null)) {  
        System.out.println(user);  
    }  
}
```

从日志中可以看到执行的SQL语句如下，WHERE条件中，规定只查询deleted字段为0的数据
```sql
SELECT id,name,password,age,tel,deleted FROM tb_user WHERE deleted=0
```

- 如果还是想把已经删除的数据都查询出来该如何实现呢?
```java
@Mapper  
public interface UserDao extends BaseMapper<User> {  
    //查询所有数据包含已经被删除的数据  
    @Select("select * from tb_user")  
    public List<User> selectAll();  
}
```

如果每个表都要有逻辑删除，那么就需要在每个模型类的属性上添加`@TableLogic`注解，如何优化?

- 在配置文件中添加全局配置，如下
```yml
mybatis-plus:  
    global-config:  
    db-config:  
        # 逻辑删除字段名  
        logic-delete-field: deleted  
        # 逻辑删除字面值：未删除为0  
        logic-not-delete-value: 0  
        # 逻辑删除字面值：删除为1  
        logic-delete-value: 1
```
使用yml配置文件配置了之后，就不需要在模型类上用`@TableLogic`注解了

```sql
UPDATE tb_user SET deleted=1 WHERE id=? AND deleted=0
```

知识点：`@TableLogic`

|名称|@TableLogic|
|---|---|
|类型|属性注解|
|位置|模型类中用于表示删除字段的属性定义上方|
|作用|标识该字段为进行逻辑删除的字段|
|相关属性|value：逻辑未删除值  <br>delval:逻辑删除值|
## 8 乐观锁

### 8.1 概念

在学乐观锁之前，我们还是先由一个案例来引入

- 业务并发现象带来的问题：秒杀
    - 加入有100个商品在售，为了保证每个商品只能被一个人购买，如何保证不会超买或者重复卖
    - 对于这一类的问题，其实有很多的解决方案可以使用
    - 第一个最先想到的就是锁，锁在一台服务器中是可以解决的，但是如果在多台服务器下就没办法控制，比如12306有两台服务器，再进行卖票，在两台服务器上都添加锁的话，那也有可能会在同一时刻有两个线程在卖票，还是会出现并发问题
    - 我们接下来介绍的这种方式就是针对于小型企业的解决方案，因为数据库本身的性能就是个瓶颈，如果对其并发超过2000以上的就需要考虑其他解决方案了

简单来说，乐观锁主要解决的问题是，当要更新一条记录的时候，希望这条记录没有被别人更新
### 8.2 实现思路

- 数据库表中添加`version`字段，比如默认值给个1
- 第一个线程要修改数据之前，取出记录时，获取当前的version=1
- 第二个线程要修改数据之前，取出记录时，获取当前的version=1
- 第一个线程执行更新时
    - set version = newVersion where version = oldVersion
        - newVersion = version + 1
        - oldVersion = version
- 第二个线程执行更新时
    - set version = newVersion where version = oldVersion
        - newVersion = version + 1
        - oldVersion = version
- 假如这两个线程都来更新数据，第一个和第二个线程都可能先执行
    - 假如第一个线程先执行更新，会将version改为2
        - 那么第二个线程再更新的时候，set version = 2 where version = 1，此时数据库表的version已经是2了，所以第二个线程修改失败
    - 假如第二个线程先执行更新，会将version改为2
        - 那么第一个线程再更新的时候，set version = 2 where version = 1，此时数据库表的version已经是2了，所以第一个线程修改失败
### 8.3 实现步骤

- `步骤一：`数据库表添加列  
    加一列version，长度给个11，默认值设为1
- `步骤二：`在模型类中添加对应的属性
```java
@Data  
public class User {  
    private Long id;  
    private String name;  
    private String password;  
    private Integer age;  
    private String tel;  
    @TableLogic(value = "0", delval = "1")  
    private Integer deleted;  
    @Version  
    private Integer version;  
}
```

- `步骤三：`添加乐观锁拦截器
```java
@Configuration  
public class MpConfig {  
    @Bean  
    public MybatisPlusInterceptor mpInterceptor() {  
        //1.定义Mp拦截器  
        MybatisPlusInterceptor mpInterceptor = new MybatisPlusInterceptor();  
        //2.添加乐观锁拦截器  
        mpInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());  
        return mpInterceptor;  
    }  
}
```

- `步骤四：`执行更新操作
```java
@Test  
void testUpdate(){  
    //1. 先通过要修改的数据id将当前数据查询出来  
    User user = userDao.selectById(1L);  
    //2. 修改属性  
    user.setName("Person");  
    userDao.updateById(user);  
}
```

我们传递的是1(oldVersion)，MP会将1进行加1，变成2，然后更新回到数据库中(newVersion)

大概分析完乐观锁的实现步骤以后，我们来模拟一种加锁的情况，看看能不能实现多个人修改同一个数据的时候，只能有一个人修改成功。
```java
@Test  
void testUpdate() {  
    User userA = userDao.selectById(1L); //version=1  
    User userB = userDao.selectById(1L); //version=1  
    userB.setName("Jackson");  
    userDao.updateById(userB);  //B修改完了之后，version=2  
    userA.setName("Person");  
    //A拿到的version是1，但现在的version已经是2了，那么A在执行 UPDATE ... WHERE version = 1时，就必然会失败  
    userDao.updateById(userA);    
}
```
## 9 快速开发

### 9.1 代码生成器原理分析

官方文档地址：[https://baomidou.com/pages/981406/](https://baomidou.com/pages/981406/)

通过观察我们之前写的代码，会发现其中有很多重复的内容，于是MP抽取了这些重复的地方，做成了一个模板供我们使用  
要想完成代码自动生成，我们需要有以下内容:

- 模板: MyBatisPlus提供，可以自己提供，但是麻烦，不建议
- 数据库相关配置:读取数据库获取表和字段信息
- 开发者自定义配置:手工配置，比如ID生成策略
### 9.2 代码生成器实现

- `步骤一：`创建一个Maven项目
- `步骤二：`导入对应的jar包
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.5.1</version>  
    </parent>  
    <groupId>com.blog</groupId>  
    <artifactId>mybatisplus_04_generator</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <properties>  
        <java.version>1.8</java.version>  
    </properties>  
    <dependencies>  
        <!--spring webmvc-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <!--mybatisplus-->  
        <dependency>  
            <groupId>com.baomidou</groupId>  
            <artifactId>mybatis-plus-boot-starter</artifactId>  
            <version>3.4.1</version>  
        </dependency>  
  
        <!--druid-->  
        <dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>druid</artifactId>  
            <version>1.1.16</version>  
        </dependency>  
  
        <!--mysql-->  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <scope>runtime</scope>  
        </dependency>  
  
        <!--test-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
  
        <!--lombok-->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <version>1.18.12</version>  
        </dependency>  
  
        <!--代码生成器-->  
        <dependency>  
            <groupId>com.baomidou</groupId>  
            <artifactId>mybatis-plus-generator</artifactId>  
            <version>3.4.1</version>  
        </dependency>  
  
        <!--velocity模板引擎-->  
        <dependency>  
            <groupId>org.apache.velocity</groupId>  
            <artifactId>velocity-engine-core</artifactId>  
            <version>2.3</version>  
        </dependency>  
  
    </dependencies>  
  
    <build>  
        <plugins>  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>  
        </plugins>  
    </build>  
  
</project>
```

- `步骤三：`编写引导类
```java
@SpringBootApplication  
public class Mybatisplus04GeneratorApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(Mybatisplus04GeneratorApplication.class, args);  
    }  
  
}
```

- `步骤四：`创建代码生成类
```java
public class CodeGenerator {  
    public static void main(String[] args) {  
        //1.获取代码生成器的对象  
        AutoGenerator autoGenerator = new AutoGenerator();  
  
        //设置数据库相关配置  
        DataSourceConfig dataSource = new DataSourceConfig();  
        dataSource.setDriverName("com.mysql.cj.jdbc.Driver");  
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC");  
        dataSource.setUsername("root");  
        dataSource.setPassword("YOURPASSWORD");  
        autoGenerator.setDataSource(dataSource);  
  
        //设置全局配置  
        GlobalConfig globalConfig = new GlobalConfig();  
        globalConfig.setOutputDir(System.getProperty("user.dir")+"/项目名/src/main/java");    //设置代码生成位置  
        globalConfig.setOpen(false);    //设置生成完毕后是否打开生成代码所在的目录  
        globalConfig.setAuthor("Kyle");    //设置作者  
        globalConfig.setFileOverride(true);     //设置是否覆盖原始生成的文件  
        globalConfig.setMapperName("%sDao");    //设置数据层接口名，%s为占位符，指代模块名称  
        globalConfig.setIdType(IdType.ASSIGN_ID);   //设置Id生成策略  
        autoGenerator.setGlobalConfig(globalConfig);  
  
        //设置包名相关配置  
        PackageConfig packageInfo = new PackageConfig();  
        packageInfo.setParent("com.aaa");   //设置生成的包名，与代码所在位置不冲突，二者叠加组成完整路径  
        packageInfo.setEntity("domain");    //设置实体类包名  
        packageInfo.setMapper("dao");   //设置数据层包名  
        autoGenerator.setPackageInfo(packageInfo);  
  
        //策略设置  
        StrategyConfig strategyConfig = new StrategyConfig();  
        strategyConfig.setInclude("tb_user");  //设置当前参与生成的表名，参数为可变参数  
        strategyConfig.setTablePrefix("tb_");  //设置数据库表的前缀名称，模块名 = 数据库表名 - 前缀名  例如： User = tb_user - tb_  
        strategyConfig.setRestControllerStyle(true);    //设置是否启用Rest风格  
        strategyConfig.setVersionFieldName("version");  //设置乐观锁字段名  
        strategyConfig.setLogicDeleteFieldName("deleted");  //设置逻辑删除字段名  
        strategyConfig.setEntityLombokModel(true);  //设置是否启用lombok  
        autoGenerator.setStrategy(strategyConfig);  
        //2.执行生成操作  
        autoGenerator.execute();  
    }  
}
```
对于代码生成器中的代码内容，我们可以直接从官方文档中获取代码进行修改，`https://baomidou.com/pages/981406/`

- `步骤五：`运行程序
	运行成功后，会在当前项目中生成很多代码，代码包含`controller`,`service`，`mapper`和`entity`等
	
	至此代码生成器就已经完成工作，我们能快速根据数据库表来创建对应的类，简化我们的代码开发。
	
	初期还是不建议直接使用代码生成器，还是多自己手写几遍比较好
### 9.3 MP中Service的CRUD

回顾我们之前业务层代码的编写，编写接口和对应的实现类:
```java
public interface UserService{  
	  
}  
  
@Service  
public class UserServiceImpl implements UserService{  
  
}
```

接口和实现类有了以后，需要在接口和实现类中声明方法
```java
public interface UserService{  
	public List<User> findAll();  
}  
  
@Service  
public class UserServiceImpl implements UserService{  
    @Autowired  
    private UserDao userDao;  
      
	public List<User> findAll(){  
        return userDao.selectList(null);  
    }  
}
```

MP看到上面的代码以后就说这些方法也是比较固定和通用的，那我来帮你抽取下，所以MP提供了一个Service接口和实现类，分别是:`IService`和`ServiceImpl`,后者是对前者的一个具体实现。

以后我们自己写的Service就可以进行如下修改:
```java
public interface UserService extends IService<User>{  
	  
}  
  
@Service  
public class UserServiceImpl extends ServiceImpl<UserDao, User> implements UserService{  
  
}
```

修改以后的好处是，MP已经帮我们把业务层的一些基础的增删改查都已经实现了，可以直接进行使用。

编写测试类进行测试:
```java
@SpringBootTest  
class Mybatisplus04GeneratorApplicationTests {  
  
    @AutoWired
    private IUserService userService;  
  
    @Test  
    void testFindAll() {  
        List<User> list = userService.list();  
        System.out.println(list);  
    }  
}
```

