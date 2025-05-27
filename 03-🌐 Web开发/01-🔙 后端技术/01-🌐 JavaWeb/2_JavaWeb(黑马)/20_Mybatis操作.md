---
文章标题: "[[20_Mybatis操作]]" 
文章作者: Dakkk
文章概要: |
  介绍MyBatis操作的完整教程，涵盖基础CRUD操作、XML配置、动态SQL和高级扩展等技术要点
tags:
- "MyBatis"
- "CRUD操作"
- "动态SQL"
- "XML配置"
- "预编译SQL"
- "PageHelper分页"
- "逆向工程"
- "ORM框架"
相关文章:
- "[[2_Mybatis使用xml时，接口形参是否使用@Param问题]]"
- "[[11_反射]]"
- "[[5_📕Spring Boot 整合 Logback 日志]]"
- "[[5_技术二面（华为Od）]]"
- "[[0_参考文章索引]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/20_Mybatis操作.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:05:50
---

## 1 Mybatis基础操作

学习完mybatis入门后，我们继续学习mybatis基础操作。
### 1.1 需求

需求说明：

- 根据资料中提供的《tlias智能学习辅助系统》页面原型及需求，完成员工管理的需求开发。

![image-20221210180155700|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/19798e14e9d1d2aea10a8fcf1b004232.png) 

![image-20221210180343288|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ad60b01b9bb92ebc623bf588148b763c.png)

![image-20221210180515206|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/806277eb5db9b320c49b7990dd557a74.png)

通过分析以上的页面原型和需求，我们确定了功能列表：

1. 查询
   - 根据主键ID查询
   - 条件查询

2. 新增
3. 更新
4. 删除
   - 根据主键ID删除
   - 根据主键ID批量删除

### 1.2 准备

实施前的准备工作：

1. 准备数据库表
2. 创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）
3. application.properties中引入数据库连接信息
4. 创建对应的实体类 Emp（实体类属性采用驼峰命名）
5. 准备Mapper接口 EmpMapper

**准备数据库表**

~~~mysql
-- 部门管理
create table dept
(
    id          int unsigned primary key auto_increment comment '主键ID',
    name        varchar(10) not null unique comment '部门名称',
    create_time datetime    not null comment '创建时间',
    update_time datetime    not null comment '修改时间'
) comment '部门表';
-- 部门表测试数据
insert into dept (id, name, create_time, update_time)
values (1, '学工部', now(), now()),
       (2, '教研部', now(), now()),
       (3, '咨询部', now(), now()),
       (4, '就业部', now(), now()),
       (5, '人事部', now(), now());


-- 员工管理
create table emp
(
    id          int unsigned primary key auto_increment comment 'ID',
    username    varchar(20)      not null unique comment '用户名',
    password    varchar(32) default '123456' comment '密码',
    name        varchar(10)      not null comment '姓名',
    gender      tinyint unsigned not null comment '性别, 说明: 1 男, 2 女',
    image       varchar(300) comment '图像',
    job         tinyint unsigned comment '职位, 说明: 1 班主任,2 讲师, 3 学工主管, 4 教研主管, 5 咨询师',
    entrydate   date comment '入职时间',
    dept_id     int unsigned comment '部门ID',
    create_time datetime         not null comment '创建时间',
    update_time datetime         not null comment '修改时间'
) comment '员工表';
-- 员工表测试数据
INSERT INTO emp (id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)
VALUES 
(1, 'jinyong', '123456', '金庸', 1, '1.jpg', 4, '2000-01-01', 2, now(), now()),
(2, 'zhangwuji', '123456', '张无忌', 1, '2.jpg', 2, '2015-01-01', 2, now(), now()),
(3, 'yangxiao', '123456', '杨逍', 1, '3.jpg', 2, '2008-05-01', 2, now(), now()),
(4, 'weiyixiao', '123456', '韦一笑', 1, '4.jpg', 2, '2007-01-01', 2, now(), now()),
(5, 'changyuchun', '123456', '常遇春', 1, '5.jpg', 2, '2012-12-05', 2, now(), now()),
(6, 'xiaozhao', '123456', '小昭', 2, '6.jpg', 3, '2013-09-05', 1, now(), now()),
(7, 'jixiaofu', '123456', '纪晓芙', 2, '7.jpg', 1, '2005-08-01', 1, now(), now()),
(8, 'zhouzhiruo', '123456', '周芷若', 2, '8.jpg', 1, '2014-11-09', 1, now(), now()),
(9, 'dingminjun', '123456', '丁敏君', 2, '9.jpg', 1, '2011-03-11', 1, now(), now()),
(10, 'zhaomin', '123456', '赵敏', 2, '10.jpg', 1, '2013-09-05', 1, now(), now()),
(11, 'luzhangke', '123456', '鹿杖客', 1, '11.jpg', 5, '2007-02-01', 3, now(), now()),
(12, 'hebiweng', '123456', '鹤笔翁', 1, '12.jpg', 5, '2008-08-18', 3, now(), now()),
(13, 'fangdongbai', '123456', '方东白', 1, '13.jpg', 5, '2012-11-01', 3, now(), now()),
(14, 'zhangsanfeng', '123456', '张三丰', 1, '14.jpg', 2, '2002-08-01', 2, now(), now()),
(15, 'yulianzhou', '123456', '俞莲舟', 1, '15.jpg', 2, '2011-05-01', 2, now(), now()),
(16, 'songyuanqiao', '123456', '宋远桥', 1, '16.jpg', 2, '2010-01-01', 2, now(), now()),
(17, 'chenyouliang', '123456', '陈友谅', 1, '17.jpg', NULL, '2015-03-21', NULL, now(), now());
~~~



**创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）**

![image-20221210182008131|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57e79c2be381d00e1d4ca6c170a73e7d.png)



**application.properties中引入数据库连接信息**

> 提示：可以把之前项目中已有的配置信息复制过来即可

~~~properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=1234
~~~



**创建对应的实体类Emp（实体类属性采用驼峰命名）**

~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Short gender;
    private String image;
    private Short job;
    private LocalDate entrydate;     //LocalDate类型对应数据表中的date类型
    private Integer deptId;
    private LocalDateTime createTime;//LocalDateTime类型对应数据表中的datetime类型
    private LocalDateTime updateTime;
}
~~~



**准备Mapper接口：EmpMapper**

~~~java
/*@Mapper注解：表示当前接口为mybatis中的Mapper接口
  程序运行时会自动创建接口的实现类对象(代理对象)，并交给Spring的IOC容器管理
*/
@Mapper
public interface EmpMapper {

}
~~~

完成以上操作后，项目工程结构目录如下：

![image-20221210182500817|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/421ed966754a2e8efdbd22b435865275.png)

### 1.3 删除

#### 1.3.1 功能实现

页面原型：

![image-20221210183336095|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/040176a66f21c1ac71264af430fdc627.png)

> 当我们点击后面的"删除"按钮时，前端页面会给服务端传递一个参数，也就是该行数据的ID。 我们接收到ID后，根据ID删除数据即可。

**功能：根据主键删除数据**

- SQL语句

~~~mysql
-- 删除id=17的数据
delete from emp where id = 17;
~~~

> Mybatis框架让程序员更关注于SQL语句

- 接口方法

~~~java
@Mapper
public interface EmpMapper {
    
    //@Delete("delete from emp where id = 17")
    //public void delete();
    //以上delete操作的SQL语句中的id值写成固定的17，就表示只能删除id=17的用户数据
    //SQL语句中的id值不能写成固定数值，需要变为动态的数值
    //解决方案：在delete方法中添加一个参数(用户id)，将方法中的参数，传给SQL语句
    
    /**
     * 根据id删除数据
     * @param id    用户id
     */
    @Delete("delete from emp where id = #{id}")//使用#{key}方式获取方法中的参数值
    public void delete(Integer id);
    
}
~~~

> @Delete注解：用于编写delete操作的SQL语句

> `如果mapper接口方法形参只有一个普通类型的参数，#{…} 里面的属性名可以随便写`，如：#{id}、#{value}。但是建议保持名字一致。

- 测试
  - 在单元测试类中通过@Autowired注解注入EmpMapper类型对象

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired //从Spring的IOC容器中，获取类型是EmpMapper的对象并注入
    private EmpMapper empMapper;

    @Test
    public void testDel(){
        //调用删除方法
        empMapper.delete(16);
    }

}
~~~

#### 1.3.2 日志输入！

在Mybatis当中我们可以借助日志，查看到sql语句的执行、执行传递的参数以及执行结果。具体操作如下：

1. 打开application.properties文件

2. 开启mybatis的日志，并指定输出到控制台

```properties
#指定mybatis输出日志的位置, 输出控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

开启日志之后，我们再次运行单元测试，可以看到在控制台中，输出了以下的SQL语句信息：

![image-20220901164225644|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5283f22a5a73c0b6950fd8ab3e0287c.png) 

> 但是我们发现输出的SQL语句：delete from emp where id = ?，`我们输入的参数16并没有在后面拼接，id的值是使用?进行占位。那这种SQL语句我们称为预编译SQL`。

#### 1.3.3 预编译SQL

##### 1.3.3.1 介绍

预编译SQL有两个优势：

1. 性能更高
2. 更安全(防止SQL注入)

![image-20221210202222206|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/add9b6f5d488db41b3837da9128ca5a5.png)

> 性能更高：预编译SQL，编译一次之后会将编译后的SQL语句缓存起来，后面再次执行这条语句时，不会再次编译。（只是输入的参数不同）
>
> 更安全(防止SQL注入)：将敏感字进行转义，保障SQL的安全性。


##### 1.3.3.2 SQL注入

SQL注入：是通过操作输入的数据来修改事先定义好的SQL语句，以达到执行代码对服务器进行攻击的方法。

> 由于没有对用户输入进行充分检查，而SQL又是拼接而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，也可以完成恶意攻击。

**测试1：使用资料中提供的程序，来验证SQL注入问题**

![image-20221210205419634|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fc65e8779612cfd30a04d7fd89f812e6.png)

第1步：进入到DOS

![image-20221211124744203|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/54e2490c9385a62da3a723c5e59a6fa7.png)

![image-20221211124840720|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78e231064277bb3a1d5abb81dbaf3181.png)

第2步：执行以下命令，启动程序

~~~powershell
#启动存在SQL注入的程序
java -jar sql_Injection_demo-0.0.1-SNAPSHOT.jar 
~~~

![image-20221210211605231|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4794a2de7a8e3b3d48db29cb667170d7.png)

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88add35cd72d7a05938ac07f68e12c1f.png)

发现竟然能够登录成功：

![image-20221210212511915|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c46a75641ac73f06909b4a2bd5a34bb.png)



以上操作为什么能够登录成功呢？

- 由于没有对用户输入内容进行充分检查，而SQL又是字符串拼接方式而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，从而完成恶意攻击。

![image-20221210213311518|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/402d10e7d2f2674f9c2a0787116c2b88.png)

> ![image-20221210214431228|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a623e6e1dad9fa52071bfcaff8aed82.png)
>
> 用户在页面提交数据的时候人为的添加一些特殊字符，使得sql语句的结构发生了变化，最终可以在没有用户名或者密码的情况下进行登录。



**测试2：使用资料中提供的程序，来验证SQL注入问题**

第1步：进入到DOS

第2步：执行以下命令，启动程序：

~~~powershell
#启动解决了SQL注入的程序
java -jar sql_prepared_demo-0.0.1-SNAPSHOT.jar
~~~

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88add35cd72d7a05938ac07f68e12c1f.png)

发现无法登录：

![image-20221211125751981|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c31667a057a2b6e6e7ba329a06c2fc70.png)

以上操作SQL语句的执行：

![image-20221211130011973|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e19b5f9dfc69fcecedbf7505ceb3f56.png)

> 把整个`' or '1'='1`作为一个完整的参数，赋值给第2个问号（`' or '1'='1`进行了转义，只当做字符串使用）



##### 1.3.3.3 参数占位符

在Mybatis中提供的参数占位符有两种：${...} 、#{...}

- #{...}
  - 执行SQL时，会将#{…}替换为?，生成预编译SQL，会自动设置参数值
  - 使用时机：参数传递，都使用#{…}

- ${...}
  - 拼接SQL。直接将参数拼接在SQL语句中，存在SQL注入问题
  - 使用时机：如果对表名、列表进行动态设置时使用

> 注意事项：在项目开发中，建议使用#{...}，生成预编译SQL，防止SQL注入安全。

### 1.4 新增

功能：新增员工信息

![image-20221211134239610|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a231f62269fcd703739e0cbad8a39dd2.png)

#### 1.4.1 基本新增

员工表结构：

![image-20221211134746319|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/629c20b2deb886482ca0340f409b7681.png)

SQL语句：

```sql
insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values ('songyuanqiao','宋远桥',1,'1.jpg',2,'2012-10-09',2,'2022-10-01 10:00:00','2022-10-01 10:00:00');
```

接口方法：

```java
@Mapper
public interface EmpMapper {

    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
```

> 说明：#{...} 里面写的名称是对象的属性名

测试类：

```java
import com.itheima.mapper.EmpMapper;
import com.itheima.pojo.Emp;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.time.LocalDate;
import java.time.LocalDateTime;

@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("tom");
        emp.setName("汤姆");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);
    }
}

```

> 日志输出：
>
> ![image-20221211140222240|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/253406995d6951eeeb301e5680251eab.png)

#### 1.4.2 主键返回！

概念：在数据添加成功后，需要获取插入数据库数据的主键。

> 如：添加套餐数据时，还需要维护套餐菜品关系表数据。
>
> ![image-20221211150353385|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e510a6797ab904811bc184dbe65bdfda.png)
>
> 业务场景：在前面讲解到的苍穹外卖菜品与套餐模块的表结构，菜品与套餐是多对多的关系，一个套餐对应多个菜品。既然是多对多的关系，是不是有一张套餐菜品中间表来维护它们之间的关系。
>
> ![image-20221212093655389|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8651f3ec7883a2a5abb8ebbee15e826.png)
>
> 在添加套餐的时候，我们需要在界面当中来录入套餐的基本信息，还需要来录入套餐与菜品的关联信息。这些信息录入完毕之后，我们一点保存，就需要将套餐的信息以及套餐与菜品的关联信息都需要保存到数据库当中。其实具体的过程包括两步，首先第一步先需要将套餐的基本信息保存了，接下来第二步再来保存套餐与菜品的关联信息。套餐与菜品的关联信息就是往中间表当中来插入数据，来维护它们之间的关系。而中间表当中有两个外键字段，一个是菜品的ID，就是当前菜品的ID，还有一个就是套餐的ID，而这个套餐的 ID 指的就是此次我所添加的套餐的ID，所以我们在第一步保存完套餐的基本信息之后，就需要将套餐的主键值返回来供第二步进行使用。这个时候就需要用到主键返回功能。

那要如何实现在插入数据之后返回所插入行的主键值呢？

- 默认情况下，执行插入操作时，是不会主键值返回的。如果我们想要拿到主键值，需要在Mapper接口中的方法上添加一个Options注解，并在注解中指定属性useGeneratedKeys=true和keyProperty="实体类属性名"


主键返回代码实现：

~~~java
@Mapper
public interface EmpMapper {
    
    //会自动将生成的主键值，赋值给emp对象的id属性
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
~~~

测试：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("jack");
        emp.setName("杰克");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);

        System.out.println(emp.getDeptId());
    }
}
~~~


### 1.5 更新

功能：修改员工信息

![image-20221212095605863|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b853a4201749f9805efa0f73e1cd9fba.png)

> 点击"编辑"按钮后，会查询所在行记录的员工信息，并把员工信息回显在修改员工的窗体上(下个知识点学习)
>
> 在修改员工的窗体上，可以修改的员工数据：用户名、员工姓名、性别、图像、职位、入职日期、归属部门
>
> 思考：在修改员工数据时，要以什么做为条件呢？
>
> 答案：员工id

SQL语句：

```sql
update emp set username = 'linghushaoxia', name = '令狐少侠', gender = 1 , image = '1.jpg' , job = 2, entrydate = '2012-01-01', dept_id = 2, update_time = '2022-10-01 12:12:12' where id = 18;
```

接口方法：

```java
@Mapper
public interface EmpMapper {
    /**
     * 根据id修改员工信息
     * @param emp
     */
    @Update("update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}, job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}")
    public void update(Emp emp);
    
}
```

测试类：

```java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testUpdate(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(23);
        emp.setUsername("songdaxia");
        emp.setPassword(null);
        emp.setName("老宋");
        emp.setImage("2.jpg");
        emp.setGender((short)1);
        emp.setJob((short)2);
        emp.setEntrydate(LocalDate.of(2012,1,1));
        emp.setCreateTime(null);
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(2);
        //调用方法，修改员工数据
        empMapper.update(emp);
    }
}
```


### 1.6 查询

#### 1.6.1 根据ID查询

在员工管理的页面中，当我们进行更新数据时，会点击 “编辑” 按钮，然后此时会发送一个请求到服务端，会根据Id查询该员工信息，并将员工数据回显在页面上。

![image-20221212101331292|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a895acf69bfab64c8e146f71c83e3537.png) 

SQL语句：

~~~mysql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp;
~~~

接口方法：

~~~java
@Mapper
public interface EmpMapper {
    @Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
    public Emp getById(Integer id);
}
~~~

测试类：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testGetById(){
        Emp emp = empMapper.getById(1);
        System.out.println(emp);
    }
}
~~~

> 执行结果：
>
> ![image-20221212103004961|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04a630255727b8be38cdd47dbe3297bb.png)
>
> 而在测试的过程中，我们会发现有几个字段(deptId、createTime、updateTime)是没有数据值的


#### 1.6.2 数据封装！

我们看到查询返回的结果中大部分字段是有值的，但是deptId，createTime，updateTime这几个字段是没有值的，而数据库中是有对应的字段值的，这是为什么呢？

![image-20221212103124490|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0a4ec7c0a174ca528ece134d8c14377.png)

原因如下： 

- 实体类属性名和数据库表查询返回的字段名一致，mybatis会自动封装。
- 如果实体类属性名和数据库表查询返回的字段名不一致，不能自动封装。



 解决方案：

1. 起别名（数据库名称起别名，为代码属性名称）
2. 结果映射
3. 开启驼峰命名



**起别名**：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样

```java
@Select("select id, username, password, name, gender, image, job, entrydate, " +
        "dept_id AS deptId, create_time AS createTime, update_time AS updateTime " +
        "from emp " +
        "where id=#{id}")
public Emp getById(Integer id);
```

> 再次执行测试类：
>
> ![image-20221212111027396|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12e1f6f64418be23bab725d4a2cd568a.png)



**手动结果映射**：通过 @Results及@Result 进行手动结果映射

```java
@Results({@Result(column = "dept_id", property = "deptId"),
          @Result(column = "create_time", property = "createTime"),
          @Result(column = "update_time", property = "updateTime")})
@Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
public Emp getById(Integer id);
```

@Results源代码：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Results {
String id() default "";

Result[] value() default {};  //Result类型的数组
}
```

@Result源代码：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Repeatable(Results.class)
public @interface Result {
boolean id() default false;//表示当前列是否为主键（true:是主键）

String column() default "";//指定表中字段名

String property() default "";//指定类中属性名

Class<?> javaType() default void.class;

JdbcType jdbcType() default JdbcType.UNDEFINED;

Class<? extends TypeHandler> typeHandler() default UnknownTypeHandler.class;

One one() default @One;

Many many() default @Many;
}
```

**开启驼峰命名(推荐)**：如果字段名与属性名符合驼峰命名规则，mybatis会自动通过驼峰命名规则映射

> 驼峰命名规则：   abc_xyz    =>   abcXyz
>
> - 表中字段名：abc_xyz
> - 类中属性名：abcXyz

```properties
# 在application.properties中添加：
mybatis.configuration.map-underscore-to-camel-case=true
```

> 要使用驼峰命名前提是 实体类的属性 与 数据库表中的字段名严格遵守驼峰命名。

#### 1.6.3 条件查询！

在员工管理的列表页面中，我们需要根据条件查询员工信息，查询条件包括：姓名、性别、入职时间。 

![image-20221212113422924|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fad8f9e1151bc51147f598ea69577a8.png)

通过页面原型以及需求描述我们要实现的查询：

- 姓名：要求`支持模糊匹配`
- 性别：要求`精确匹配`
- 入职时间：要求`进行范围查询`
- 根据最后修改时间进行`降序排序`

SQL语句：

```sql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time 
from emp 
where name like '%张%' 
      and gender = 1 
      and entrydate between '2010-01-01' and '2020-01-01 ' 
order by update_time desc;
```

接口方法：

- 方式一

```java
@Mapper
public interface EmpMapper {
    @Select("select * from emp " +
            "where name like '%${name}%' " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);
}
```

> ![image-20221212115149151|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/758860817c526a380d34accd26dc5837.png)
>
> 以上方式注意事项：
>
> 1. 方法中的形参名和SQL语句中的参数占位符名保持一致
>
> 2. 模糊查询使用${...}进行字符串拼接，这种方式呢，由于是字符串拼接，并不是预编译的形式，所以效率不高、且存在sql注入风险。



- 方式二（解决SQL注入风险）
  - `使用MySQL提供的字符串拼接函数：concat('%' , '关键字' , '%')`

~~~java
@Mapper
public interface EmpMapper {

    @Select("select * from emp " +
            "where name like concat('%',#{name},'%') " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);

}

~~~

> 执行结果：生成的SQL都是预编译的SQL语句（性能高、安全）
>
> ![image-20221212120006242|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4440f6deea5397a5a566d2003c1e4b5.png)


#### 1.6.4 参数名说明

在上面我们所编写的条件查询功能中，我们需要保证接口中方法的形参名和SQL语句中的参数占位符名相同。

> 当方法中的形参名和SQL语句中的占位符参数名不相同时，就会出现以下问题：
>
> ![image-20221212150611796|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b864bb3e4655a48725fcbc41e92ccdc.png)



参数名在不同的SpringBoot版本中，处理方案还不同：

- 在springBoot的2.x版本（保证参数名一致）

![image-20221212151156273|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5290c97926460a92da8c1ae3c1e7e1f2.png)

> springBoot的父工程对compiler编译插件进行了默认的参数parameters配置，使得在编译时，会在生成的字节码文件中保留原方法形参的名称，所以#{…}里面可以直接通过形参名获取对应的值
>
> ![image-20221212151411154|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e0b716d80a78f33d8667fdeb9a30143.png)



- 在springBoot的1.x版本/单独使用mybatis（使用@Param注解来指定SQL语句中的参数名）

![image-20221212151628715|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2129c97a9fceb9a5975e5620450786e9.png)

> 在编译时，生成的字节码文件当中，不会保留Mapper接口中方法的形参名称，而是使用var1、var2、...这样的形参名字，此时要获取参数值时，就要通过@Param注解来指定SQL语句中的参数名
>
> ![image-20221212151736274|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5db0b2e29e1c7f733878dc4e8492b6d8.png)

## 2 Mybatis的XML配置文件

Mybatis的开发有两种方式：

1. 注解
2. XML

### 2.1 XML配置文件规范

使用Mybatis的注解方式，主要是来完成一些简单的增删改查功能。

`如果需要实现复杂的SQL功能，建议使用XML来配置映射语句，也就是将SQL语句写在XML配置文件中`。

在Mybatis中使用XML映射文件方式开发，需要符合一定的规范：

1. `XML映射文件的名称与Mapper接口名称一致`，并且将`XML映射文件和Mapper接口放置在相同包下（同包同名）`

2. XML映射文件的namespace属性为Mapper接口全限定名一致

3. XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。

![image-20221212153529732|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fa08cba9bcbc7cfb3e9748672ace019a.png)

> \<select>标签：就是用于编写select查询语句的。
>
> - resultType属性，指的是查询返回的单条记录所封装的类型。


### 2.2 XML配置文件实现

第1步：创建XML映射文件
- `切记包名要用 / 分隔`

![image-20221212154908306|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f7c425f8e68475c6b4408fa320f50ae.png)

![image-20221212155304635|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/818dd955aecc74002c0411573d8fb7d2.png)

![image-20221212155544404|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/002eb7a5a703b5c167917e46c149e4d0.png)



第2步：编写XML映射文件

> xml映射文件中的dtd约束，直接从mybatis官网复制即可

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="">
 
</mapper>
~~~



配置：XML映射文件的namespace属性为Mapper接口全限定名

![image-20221212160316644|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab13794272b3ed5e2ea4b507fad324ef.png)

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

</mapper>
~~~



配置：XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致

![image-20221212163528787|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9e3a7a547d45e9cb3c972fae846c51c.png)

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--查询操作-->
    <select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where name like concat('%',#{name},'%')
              and gender = #{gender}
              and entrydate between #{begin} and #{end}
        order by update_time desc
    </select>
</mapper>
~~~

- `注意：参数前要使用@Param注解`
```java
public List<Emp> list(@Param("name") String name, @Param("gender")Short gender,  
                      @Param("begin")LocalDate begin, @Param("end")LocalDate end);
```

> 运行测试类，执行结果：
>
> ![image-20221212163719534|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/662a1b47b5048569359959e91bc7aa15.png)

### 2.3 MybatisX的使用！

MybatisX是一款基于IDEA的快速开发Mybatis的插件，为效率而生。

MybatisX的安装：

![image-20221213120923252|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea5040d4f6d54fe4bf9d7e16c56a0a4d.png)

可以通过MybatisX快速定位：

![image-20221213121521406|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb6dc20183e837287abc239c51e70b93.png)

> MybatisX的使用在后续学习中会继续分享



学习了Mybatis中XML配置文件的开发方式了，大家可能会存在一个疑问：到底是使用注解方式开发还是使用XML方式开发？

> 官方说明：https://mybatis.net.cn/getting-started.html
>
> ![image-20220901173948645|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9fa59300569a4b6569ae56f8dd3d0fc.png) 

**结论：** 使用Mybatis的注解，主要是来完成一些简单的增删改查功能。如果需要实现复杂的SQL功能，建议使用XML来配置映射语句。

## 3 Mybatis动态SQL！

### 3.1 什么是动态SQL

在页面原型中，列表上方的条件是动态的，是可以不传递的，也可以只传递其中的1个或者2个或者全部。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ab6ac23be82e80b81bc0f435a79d5de.png)

![image-20220901173203491|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/833fee0ee0b782c8960a8c18d2d627cc.png)

而在我们刚才编写的SQL语句中，我们会看到，我们将三个条件直接写死了。 如果页面只传递了参数姓名name 字段，其他两个字段 性别 和 入职时间没有传递，那么这两个参数的值就是null。

此时，执行的SQL语句为：

![image-20220901173431554|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c22e18cc3a5ae5b3ab9e0398fa64fec.png) 

这个查询结果是不正确的。正确的做法应该是：`传递了参数，再组装这个查询条件；如果没有传递参数，就不应该组装这个查询条件`。

比如：如果姓名输入了"张", 对应的SQL为:

```sql
select *  from emp where name like '%张%' order by update_time desc;
```


如果姓名输入了"张",，性别选择了"男"，则对应的SQL为:

```sql
select *  from emp where name like '%张%' and gender = 1 order by update_time desc;
```

SQL语句会随着用户的输入或外部条件的变化而变化，我们称为：**动态SQL**。

![image-20221213122623278|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2c53dbe94fafa4288e2e3ae556f128f2.png)

在Mybatis中提供了很多实现动态SQL的标签，我们学习Mybatis中的动态SQL就是掌握这些动态SQL标签。

### 3.2 动态SQL-if

`<if>`：用于判断条件是否成立。使用test属性进行条件判断，如果条件为true，则拼接SQL。

~~~xml
<if test="条件表达式">
   要拼接的sql语句
</if>
~~~

接下来，我们就通过`<if>`标签来改造之前条件查询的案例。

#### 3.2.1 条件查询

示例：把SQL语句改造为动态SQL方式

- 原有的SQL语句

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where name like concat('%',#{name},'%')
              and gender = #{gender}
              and entrydate between #{begin} and #{end}
        order by update_time desc
</select>
~~~

- 动态SQL语句

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where
    
             <if test="name != null">
                 name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
    
        order by update_time desc
</select>
~~~

测试方法：

~~~java
@Test
public void testList(){
    //性别数据为null、开始时间和结束时间也为null
    List<Emp> list = empMapper.list("张", null, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

> 执行的SQL语句： 
>
> ![image-20221213140353285|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4eae65c7dfccf561543b38f5d7c1566.png)


下面呢，我们修改测试方法中的代码，再次进行测试，观察执行情况：

~~~java
@Test
public void testList(){
    //姓名为null
    List<Emp> list = empMapper.list(null, (short)1, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

执行结果：

![image-20221213141139015|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34b2856f312a50e80148996767ca1c6e.png) 

![image-20221213141253355|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4be7fa3ff745e0b0e621f126237def5.png) 


再次修改测试方法中的代码，再次进行测试：

~~~java
@Test
public void testList(){
    //传递的数据全部为null
    List<Emp> list = empMapper.list(null, null, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

执行的SQL语句：

![image-20221213143854434|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fda2c6cb305d00cafdf8460a471df8a4.png)

以上问题的解决方案：使用`<where>`标签代替SQL语句中的where关键字

- `<where>`只会在子元素有内容的情况下才插入where子句，而且会自动去除子句的开头的AND或OR

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        <where>
             <!-- if做为where标签的子元素 -->
             <if test="name != null">
                 and name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
        </where>
        order by update_time desc
</select>
~~~

测试方法：

~~~java
@Test
public void testList(){
    //只有性别
    List<Emp> list = empMapper.list(null, (short)1, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

> 执行的SQL语句：
>
> ![image-20221213141909455|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5122017261349a9bc04286f9c860ec68.png)

#### 3.2.2 更新员工

案例：完善更新员工功能，修改为动态更新员工数据信息

- 动态更新员工信息，如果更新时传递有值，则更新；如果更新时没有传递值，则不更新
- 解决方案：动态SQL

修改Mapper接口：

~~~java
@Mapper
public interface EmpMapper {
    //删除@Update注解编写的SQL语句
    //update操作的SQL语句编写在Mapper映射文件中
    public void update(Emp emp);
}
~~~

修改Mapper映射文件：

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--更新操作-->
    <update id="update">
        update emp
        set
            <if test="username != null">
                username=#{username},
            </if>
            <if test="name != null">
                name=#{name},
            </if>
            <if test="gender != null">
                gender=#{gender},
            </if>
            <if test="image != null">
                image=#{image},
            </if>
            <if test="job != null">
                job=#{job},
            </if>
            <if test="entrydate != null">
                entrydate=#{entrydate},
            </if>
            <if test="deptId != null">
                dept_id=#{deptId},
            </if>
            <if test="updateTime != null">
                update_time=#{updateTime}
            </if>
        where id=#{id}
    </update>

</mapper>
~~~

测试方法：

~~~java
@Test
public void testUpdate2(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(20);
        emp.setUsername("Tom111");
        emp.setName("汤姆111");

        emp.setUpdateTime(LocalDateTime.now());

        //调用方法，修改员工数据
        empMapper.update(emp);
}
~~~

> 执行的SQL语句：
>
> ![image-20221213152533851|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ca5f34e3b327bf33db0d4e76f0830e2f.png)



再次修改测试方法，观察SQL语句执行情况：

~~~java
@Test
public void testUpdate2(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(20);
        emp.setUsername("Tom222");
      
        //调用方法，修改员工数据
        empMapper.update(emp);
}
~~~

> 执行的SQL语句：
>
> ![image-20221213152850322|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79a68bd847c0e31d3421bede4c635237.png)

以上问题的解决方案：使用`<set>`标签代替SQL语句中的set关键字

- `<set>`：动态的在SQL语句中插入set关键字，并会删掉额外的逗号。（用于update语句中）

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--更新操作-->
    <update id="update">
        update emp
        <!-- 使用set标签，代替update语句中的set关键字 -->
        <set>
            <if test="username != null">
                username=#{username},
            </if>
            <if test="name != null">
                name=#{name},
            </if>
            <if test="gender != null">
                gender=#{gender},
            </if>
            <if test="image != null">
                image=#{image},
            </if>
            <if test="job != null">
                job=#{job},
            </if>
            <if test="entrydate != null">
                entrydate=#{entrydate},
            </if>
            <if test="deptId != null">
                dept_id=#{deptId},
            </if>
            <if test="updateTime != null">
                update_time=#{updateTime}
            </if>
        </set>
        where id=#{id}
    </update>
</mapper>
~~~

> 再次执行测试方法，执行的SQL语句：
>
> ![image-20221213153329553|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8076d35903868eba0630d959e8de696.png)


**小结**

- `<if>`

  - 用于判断条件是否成立，如果条件为true，则拼接SQL

  - 形式：

    ~~~xml
    <if test="name != null"> … </if>
    ~~~

- `<where>`

  - where元素只会在子元素有内容的情况下才插入where子句，而且会自动去除子句的开头的AND或OR

- `<set>`

  - 动态地在行首插入 SET 关键字，并会删掉额外的逗号。（用在update语句中）

### 3.3 动态SQL-foreach

案例：员工删除功能（既支持删除单条记录，又支持批量删除）

![image-20220901181751004|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/efb157b491c0685d58e8928dea9b05dc.png) 

SQL语句：

~~~mysql
delete from emp where id in (1,2,3);
~~~

Mapper接口：

~~~java
@Mapper
public interface EmpMapper {
    //批量删除
    public void deleteByIds(List<Integer> ids);
}
~~~

XML映射文件：

- 使用`<foreach>`遍历deleteByIds方法中传递的参数ids集合

~~~xml
<foreach collection="集合名称" item="集合遍历出来的元素/项" separator="每一次遍历使用的分隔符" 
         open="遍历开始前拼接的片段" close="遍历结束后拼接的片段">
</foreach>
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">
    <!--删除操作-->
    <delete id="deleteByIds">
        delete from emp where id in
        <foreach collection="ids" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>
    </delete>
</mapper> 
~~~

> ![image-20221213165710141|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/76a9515cff2becf13fd0130d196e4663.png)

> 执行的SQL语句：
>
> ![image-20221213164957636|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f779f4112b493c3ee904547aa3b8eab.png)

### 3.4 动态SQL-sql&include

问题分析：

- 在xml映射文件中配置的SQL，有时可能会存在很多重复的片段，此时就会存在很多冗余的代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/52de06d43d40d73fb5e6d1b68b460ea0.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8bf82efbde3ed134448d983431e97f54.png)

我们可以对重复的代码片段进行抽取，将其通过`<sql>`标签封装到一个SQL片段，然后再通过`<include>`标签进行引用。

- `<sql>`：定义可重用的SQL片段

- `<include>`：通过属性refid，指定包含的SQL片段

![image-20221213171244796|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0b220335c1b7a4ceca54007d7153fb3.png)

SQL片段： 抽取重复的代码

```xml
<sql id="commonSelect">
 	select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp
</sql>
```

然后通过`<include>` 标签在原来抽取的地方进行引用。操作如下：

```xml
<select id="list" resultType="com.itheima.pojo.Emp">
    <include refid="commonSelect"/>
    <where>
        <if test="name != null">
            name like concat('%',#{name},'%')
        </if>
        <if test="gender != null">
            and gender = #{gender}
        </if>
        <if test="begin != null and end != null">
            and entrydate between #{begin} and #{end}
        </if>
    </where>
    order by update_time desc
</select>
```



## 4 MyBatis高级扩展

### 4.1 Mapper批量映射优化

1.  需求

    Mapper 配置文件很多时，在全局配置文件中一个一个注册太麻烦，希望有一个办法能够一劳永逸。
    
2.  配置方式

    `Mybatis 允许在指定 Mapper 映射文件时，只指定其所在的包`：
    ```xml
    <mappers>
        <package name="com.atguigu.mapper"/>
    </mappers>
    ```
    此时这个包下的所有 Mapper 配置文件将被自动加载、注册，比较方便。

3.  资源创建要求

-   Mapper 接口和 Mapper 配置文件名称一致
    -   Mapper 接口：EmployeeMapper.java
    -   Mapper 配置文件：EmployeeMapper.xml
-   Mapper 配置文件放在 Mapper 接口所在的包内
    -   可以将mapperxml文件放在mapper接口所在的包！
    -   可以在sources下创建mapper接口包一致的文件夹结构存放mapperxml文件

        ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/99a3d30c6ef5243b0305655238f3a319.png)


### 4.2 插件和分页插件PageHelper

#### 4.2.1 插件机制和PageHelper插件介绍

MyBatis 对插件进行了标准化的设计，并提供了一套可扩展的插件机制。插件可以在用于语句执行过程中进行拦截，并允许通过自定义处理程序来拦截和修改 SQL 语句、映射语句的结果等。

具体来说，MyBatis 的插件机制包括以下三个组件：

1.  `Interceptor`（拦截器）：定义一个拦截方法 `intercept`，该方法在执行 SQL 语句、执行查询、查询结果的映射时会被调用。
2.  `Invocation`（调用）：实际上是对被拦截的方法的封装，封装了 `Object target`、`Method method` 和 `Object[] args` 这三个字段。
3.  `InterceptorChain`（拦截器链）：对所有的拦截器进行管理，包括将所有的 Interceptor 链接成一条链，并在执行 SQL 语句时按顺序调用。

插件的开发非常简单，只需要实现 Interceptor 接口，并使用注解 `@Intercepts` 来标注需要拦截的对象和方法，然后在 MyBatis 的配置文件中添加插件即可。

`PageHelper 是 MyBatis 中比较著名的分页插件`，它提供了多种分页方式（例如 MySQL 和 Oracle 分页方式），支持多种数据库，并且使用非常简单。下面就介绍一下 PageHelper 的使用方式。

<https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md#如何配置数据库方言>

#### 4.2.2 PageHelper插件使用

1.  pom.xml引入依赖
    ```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.1.11</version>
</dependency>
    ```

2.  mybatis-config.xml配置分页插件
    在 MyBatis 的配置文件中添加 PageHelper 的插件：
    ```xml
<plugins>
	<plugin interceptor="com.github.pagehelper.PageInterceptor">
		<property name="helperDialect" value="mysql"/>
	</plugin>
</plugins>
    ```
    其中，`com.github.pagehelper.PageInterceptor` 是 PageHelper 插件的名称，`dialect` 属性用于指定数据库类型（支持多种数据库）

3.  页插件使用
    在查询方法中使用分页：
    ```java
    @Test
    public void testTeacherRelationshipToMulti() {

        TeacherMapper teacherMapper = session.getMapper(TeacherMapper.class);

        PageHelper.startPage(1,2);
        // 查询Customer对象同时将关联的Order集合查询出来
        List<Teacher> allTeachers = teacherMapper.findAllTeachers();
    //
        PageInfo<Teacher> pageInfo = new PageInfo<>(allTeachers);

        System.out.println("pageInfo = " + pageInfo);
        long total = pageInfo.getTotal(); // 获取总记录数
        System.out.println("total = " + total);
        int pages = pageInfo.getPages();  // 获取总页数
        System.out.println("pages = " + pages);
        int pageNum = pageInfo.getPageNum(); // 获取当前页码
        System.out.println("pageNum = " + pageNum);
        int pageSize = pageInfo.getPageSize(); // 获取每页显示记录数
        System.out.println("pageSize = " + pageSize);
        List<Teacher> teachers = pageInfo.getList(); //获取查询页的数据集合
        System.out.println("teachers = " + teachers);
        teachers.forEach(System.out::println);
    }
    ```

- `注意：不能将2条查询语句得到的结果放在一个分页区`

### 4.3 逆向工程和MybatisX插件

#### 4.3.1 ORM思维介绍

`ORM（Object-Relational Mapping，对象-关系映射）`是一种将数据库和面向对象编程语言中的对象之间进行转换的技术。它将对象和关系数据库的概念进行映射，最后我们就可以通过方法调用进行数据库操作!!

最终: **让我们可以使用面向对象思维进行数据库操作！！！**

**ORM 框架通常有半自动和全自动两种方式。**

-   `半自动 ORM` 通常需要程序员手动编写 SQL 语句或者配置文件，将实体类和数据表进行映射，还需要手动将查询的结果集转换成实体对象。
-   `全自动 ORM` 则是将实体类和数据表进行自动映射，使用 API 进行数据库操作时，ORM 框架会自动执行 SQL 语句并将查询结果转换成实体对象，程序员无需再手动编写 SQL 语句和转换代码。

**下面是半自动和全自动 ORM 框架的区别：**

1.  `映射方式`：半自动 ORM 框架需要程序员`手动指定实体类和数据表之间的映射关系`，通常使用 XML 文件或注解方式来指定；全自动 ORM 框架则可以`自动进行实体类和数据表的映射，无需手动干预`。

2.  `查询方式`：半自动 ORM 框架通常需要程序员`手动编写 SQL 语句并将查询结果集转换成实体对象`；全自动 ORM 框架可以`自动组装 SQL 语句、执行查询操作`，并将查询结果转换成实体对象。

3.  `性能`：由于半自动 ORM 框架需要手动编写 SQL 语句，因此程序员必须对 SQL 语句和数据库的底层知识有一定的了解，才能编写高效的 SQL 语句；而全自动 ORM 框架通过自动优化生成的 SQL 语句来提高性能，程序员无需进行优化。

4.  `学习成本`：半自动 ORM 框架需要程序员手动编写 SQL 语句和映射配置，要求程序员具备较高的数据库和 SQL 知识；全自动 ORM 框架可以自动生成 SQL 语句和映射配置，程序员无需了解过多的数据库和 SQL 知识。

常见的半自动 ORM 框架包括 MyBatis 等；常见的全自动 ORM 框架包括 Hibernate、Spring Data JPA、MyBatis-Plus 等。

#### 4.3.2 逆向工程

`MyBatis 的逆向工程是一种自动化生成持久层代码和映射文件的工具`，它可以根据数据库表结构和设置的参数生成对应的实体类、Mapper.xml 文件、Mapper 接口等代码文件，简化了开发者手动生成的过程。`逆向工程使开发者可以快速地构建起 DAO 层`，并快速上手进行业务开发。

MyBatis 的逆向工程有两种方式：通过 `MyBatis Generator 插件`实现和通过 Maven 插件实现。无论是哪种方式，逆向工程一般需要指定一些配置参数，例如数据库连接 URL、用户名、密码、要生成的表名、生成的文件路径等等。

总的来说，MyBatis 的逆向工程为程序员提供了一种方便快捷的方式，能够快速地生成持久层代码和映射文件，是半自动 ORM 思维像全自动发展的过程，提高程序员的开发效率。

**注意：逆向工程只能生成单表crud的操作，多表查询依然需要我们自己编写！**

#### 4.3.3 逆向工程插件MyBatisX使用

MyBatisX 是一个 MyBatis 的代码生成插件，可以通过简单的配置和操作快速生成 MyBatis Mapper、pojo 类和 Mapper.xml 文件。下面是使用 MyBatisX 插件实现逆向工程的步骤：

1.  安装插件：

    在 IntelliJ IDEA 中打开插件市场，搜索 MyBatisX 并安装。
2.  使用 IntelliJ IDEA连接数据库
    -   连接数据库![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fddd7d66c7c5586aa9ff4b81a9f45452.png)


    -   填写信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5675681efa73e586fab2c311bdcbfd84.png)


    -   展示库表![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8ea0be52476aabc12312b2078d30b8c.png)


    -   逆向工程使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f34ee2a18a610b8198c9105eca6dcc1.png)![[image_KXYfK5CQd-.png]]

3.  查看生成结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b87fd3db111fc94fa6a0b2972a5b8c61.png)


4.  逆向工程案例使用

    正常使用即可，自动生成单表的crud方法！
    ```java
    package com.atguigu.mapper;

    import com.atguigu.pojo.User;

    /**
    * @author Jackiechan
    * @description 针对表【user】的数据库操作Mapper
    * @createDate 2023-06-02 16:55:32
    * @Entity com.atguigu.pojo.User
    */
    public interface UserMapper {

        int deleteByPrimaryKey(Long id);

        int insert(User record);

        int insertSelective(User record);

        User selectByPrimaryKey(Long id);

        int updateByPrimaryKeySelective(User record);

        int updateByPrimaryKey(User record);

    }

    ```

## 5 MyBatis总结

| 核心点           | 掌握目标                                    |
| ------------- | --------------------------------------- |
| mybatis基础     | 使用流程, 参数输入,#{} \${},参数输出                |
| mybatis多表     | 实体类设计,resultMap多表结果映射                   |
| `mybatis动态语句` | `Mybatis动态语句概念, where , if , foreach标签` |
| mybatis扩展     | Mapper批量处理,分页插件,逆向工程                    |

## 1 Mybatis基础操作

学习完mybatis入门后，我们继续学习mybatis基础操作。
### 1.1 需求

需求说明：

- 根据资料中提供的《tlias智能学习辅助系统》页面原型及需求，完成员工管理的需求开发。

![image-20221210180155700|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/19798e14e9d1d2aea10a8fcf1b004232.png) 

![image-20221210180343288|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ad60b01b9bb92ebc623bf588148b763c.png)

![image-20221210180515206|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/806277eb5db9b320c49b7990dd557a74.png)

通过分析以上的页面原型和需求，我们确定了功能列表：

1. 查询
   - 根据主键ID查询
   - 条件查询

2. 新增
3. 更新
4. 删除
   - 根据主键ID删除
   - 根据主键ID批量删除

### 1.2 准备

实施前的准备工作：

1. 准备数据库表
2. 创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）
3. application.properties中引入数据库连接信息
4. 创建对应的实体类 Emp（实体类属性采用驼峰命名）
5. 准备Mapper接口 EmpMapper

**准备数据库表**

~~~mysql
-- 部门管理
create table dept
(
    id          int unsigned primary key auto_increment comment '主键ID',
    name        varchar(10) not null unique comment '部门名称',
    create_time datetime    not null comment '创建时间',
    update_time datetime    not null comment '修改时间'
) comment '部门表';
-- 部门表测试数据
insert into dept (id, name, create_time, update_time)
values (1, '学工部', now(), now()),
       (2, '教研部', now(), now()),
       (3, '咨询部', now(), now()),
       (4, '就业部', now(), now()),
       (5, '人事部', now(), now());


-- 员工管理
create table emp
(
    id          int unsigned primary key auto_increment comment 'ID',
    username    varchar(20)      not null unique comment '用户名',
    password    varchar(32) default '123456' comment '密码',
    name        varchar(10)      not null comment '姓名',
    gender      tinyint unsigned not null comment '性别, 说明: 1 男, 2 女',
    image       varchar(300) comment '图像',
    job         tinyint unsigned comment '职位, 说明: 1 班主任,2 讲师, 3 学工主管, 4 教研主管, 5 咨询师',
    entrydate   date comment '入职时间',
    dept_id     int unsigned comment '部门ID',
    create_time datetime         not null comment '创建时间',
    update_time datetime         not null comment '修改时间'
) comment '员工表';
-- 员工表测试数据
INSERT INTO emp (id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)
VALUES 
(1, 'jinyong', '123456', '金庸', 1, '1.jpg', 4, '2000-01-01', 2, now(), now()),
(2, 'zhangwuji', '123456', '张无忌', 1, '2.jpg', 2, '2015-01-01', 2, now(), now()),
(3, 'yangxiao', '123456', '杨逍', 1, '3.jpg', 2, '2008-05-01', 2, now(), now()),
(4, 'weiyixiao', '123456', '韦一笑', 1, '4.jpg', 2, '2007-01-01', 2, now(), now()),
(5, 'changyuchun', '123456', '常遇春', 1, '5.jpg', 2, '2012-12-05', 2, now(), now()),
(6, 'xiaozhao', '123456', '小昭', 2, '6.jpg', 3, '2013-09-05', 1, now(), now()),
(7, 'jixiaofu', '123456', '纪晓芙', 2, '7.jpg', 1, '2005-08-01', 1, now(), now()),
(8, 'zhouzhiruo', '123456', '周芷若', 2, '8.jpg', 1, '2014-11-09', 1, now(), now()),
(9, 'dingminjun', '123456', '丁敏君', 2, '9.jpg', 1, '2011-03-11', 1, now(), now()),
(10, 'zhaomin', '123456', '赵敏', 2, '10.jpg', 1, '2013-09-05', 1, now(), now()),
(11, 'luzhangke', '123456', '鹿杖客', 1, '11.jpg', 5, '2007-02-01', 3, now(), now()),
(12, 'hebiweng', '123456', '鹤笔翁', 1, '12.jpg', 5, '2008-08-18', 3, now(), now()),
(13, 'fangdongbai', '123456', '方东白', 1, '13.jpg', 5, '2012-11-01', 3, now(), now()),
(14, 'zhangsanfeng', '123456', '张三丰', 1, '14.jpg', 2, '2002-08-01', 2, now(), now()),
(15, 'yulianzhou', '123456', '俞莲舟', 1, '15.jpg', 2, '2011-05-01', 2, now(), now()),
(16, 'songyuanqiao', '123456', '宋远桥', 1, '16.jpg', 2, '2010-01-01', 2, now(), now()),
(17, 'chenyouliang', '123456', '陈友谅', 1, '17.jpg', NULL, '2015-03-21', NULL, now(), now());
~~~



**创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）**

![image-20221210182008131|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57e79c2be381d00e1d4ca6c170a73e7d.png)



**application.properties中引入数据库连接信息**

> 提示：可以把之前项目中已有的配置信息复制过来即可

~~~properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=1234
~~~



**创建对应的实体类Emp（实体类属性采用驼峰命名）**

~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Short gender;
    private String image;
    private Short job;
    private LocalDate entrydate;     //LocalDate类型对应数据表中的date类型
    private Integer deptId;
    private LocalDateTime createTime;//LocalDateTime类型对应数据表中的datetime类型
    private LocalDateTime updateTime;
}
~~~



**准备Mapper接口：EmpMapper**

~~~java
/*@Mapper注解：表示当前接口为mybatis中的Mapper接口
  程序运行时会自动创建接口的实现类对象(代理对象)，并交给Spring的IOC容器管理
*/
@Mapper
public interface EmpMapper {

}
~~~

完成以上操作后，项目工程结构目录如下：

![image-20221210182500817|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/421ed966754a2e8efdbd22b435865275.png)

### 1.3 删除

#### 1.3.1 功能实现

页面原型：

![image-20221210183336095|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/040176a66f21c1ac71264af430fdc627.png)

> 当我们点击后面的"删除"按钮时，前端页面会给服务端传递一个参数，也就是该行数据的ID。 我们接收到ID后，根据ID删除数据即可。

**功能：根据主键删除数据**

- SQL语句

~~~mysql
-- 删除id=17的数据
delete from emp where id = 17;
~~~

> Mybatis框架让程序员更关注于SQL语句

- 接口方法

~~~java
@Mapper
public interface EmpMapper {
    
    //@Delete("delete from emp where id = 17")
    //public void delete();
    //以上delete操作的SQL语句中的id值写成固定的17，就表示只能删除id=17的用户数据
    //SQL语句中的id值不能写成固定数值，需要变为动态的数值
    //解决方案：在delete方法中添加一个参数(用户id)，将方法中的参数，传给SQL语句
    
    /**
     * 根据id删除数据
     * @param id    用户id
     */
    @Delete("delete from emp where id = #{id}")//使用#{key}方式获取方法中的参数值
    public void delete(Integer id);
    
}
~~~

> @Delete注解：用于编写delete操作的SQL语句

> `如果mapper接口方法形参只有一个普通类型的参数，#{…} 里面的属性名可以随便写`，如：#{id}、#{value}。但是建议保持名字一致。

- 测试
  - 在单元测试类中通过@Autowired注解注入EmpMapper类型对象

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired //从Spring的IOC容器中，获取类型是EmpMapper的对象并注入
    private EmpMapper empMapper;

    @Test
    public void testDel(){
        //调用删除方法
        empMapper.delete(16);
    }

}
~~~

#### 1.3.2 日志输入！

在Mybatis当中我们可以借助日志，查看到sql语句的执行、执行传递的参数以及执行结果。具体操作如下：

1. 打开application.properties文件

2. 开启mybatis的日志，并指定输出到控制台

```properties
#指定mybatis输出日志的位置, 输出控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

开启日志之后，我们再次运行单元测试，可以看到在控制台中，输出了以下的SQL语句信息：

![image-20220901164225644|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5283f22a5a73c0b6950fd8ab3e0287c.png) 

> 但是我们发现输出的SQL语句：delete from emp where id = ?，`我们输入的参数16并没有在后面拼接，id的值是使用?进行占位。那这种SQL语句我们称为预编译SQL`。

#### 1.3.3 预编译SQL

##### 1.3.3.1 介绍

预编译SQL有两个优势：

1. 性能更高
2. 更安全(防止SQL注入)

![image-20221210202222206|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/add9b6f5d488db41b3837da9128ca5a5.png)

> 性能更高：预编译SQL，编译一次之后会将编译后的SQL语句缓存起来，后面再次执行这条语句时，不会再次编译。（只是输入的参数不同）
>
> 更安全(防止SQL注入)：将敏感字进行转义，保障SQL的安全性。


##### 1.3.3.2 SQL注入

SQL注入：是通过操作输入的数据来修改事先定义好的SQL语句，以达到执行代码对服务器进行攻击的方法。

> 由于没有对用户输入进行充分检查，而SQL又是拼接而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，也可以完成恶意攻击。

**测试1：使用资料中提供的程序，来验证SQL注入问题**

![image-20221210205419634|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fc65e8779612cfd30a04d7fd89f812e6.png)

第1步：进入到DOS

![image-20221211124744203|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/54e2490c9385a62da3a723c5e59a6fa7.png)

![image-20221211124840720|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78e231064277bb3a1d5abb81dbaf3181.png)

第2步：执行以下命令，启动程序

~~~powershell
#启动存在SQL注入的程序
java -jar sql_Injection_demo-0.0.1-SNAPSHOT.jar 
~~~

![image-20221210211605231|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4794a2de7a8e3b3d48db29cb667170d7.png)

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88add35cd72d7a05938ac07f68e12c1f.png)

发现竟然能够登录成功：

![image-20221210212511915|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c46a75641ac73f06909b4a2bd5a34bb.png)



以上操作为什么能够登录成功呢？

- 由于没有对用户输入内容进行充分检查，而SQL又是字符串拼接方式而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，从而完成恶意攻击。

![image-20221210213311518|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/402d10e7d2f2674f9c2a0787116c2b88.png)

> ![image-20221210214431228|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a623e6e1dad9fa52071bfcaff8aed82.png)
>
> 用户在页面提交数据的时候人为的添加一些特殊字符，使得sql语句的结构发生了变化，最终可以在没有用户名或者密码的情况下进行登录。



**测试2：使用资料中提供的程序，来验证SQL注入问题**

第1步：进入到DOS

第2步：执行以下命令，启动程序：

~~~powershell
#启动解决了SQL注入的程序
java -jar sql_prepared_demo-0.0.1-SNAPSHOT.jar
~~~

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88add35cd72d7a05938ac07f68e12c1f.png)

发现无法登录：

![image-20221211125751981|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c31667a057a2b6e6e7ba329a06c2fc70.png)

以上操作SQL语句的执行：

![image-20221211130011973|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e19b5f9dfc69fcecedbf7505ceb3f56.png)

> 把整个`' or '1'='1`作为一个完整的参数，赋值给第2个问号（`' or '1'='1`进行了转义，只当做字符串使用）



##### 1.3.3.3 参数占位符

在Mybatis中提供的参数占位符有两种：${...} 、#{...}

- #{...}
  - 执行SQL时，会将#{…}替换为?，生成预编译SQL，会自动设置参数值
  - 使用时机：参数传递，都使用#{…}

- ${...}
  - 拼接SQL。直接将参数拼接在SQL语句中，存在SQL注入问题
  - 使用时机：如果对表名、列表进行动态设置时使用

> 注意事项：在项目开发中，建议使用#{...}，生成预编译SQL，防止SQL注入安全。

### 1.4 新增

功能：新增员工信息

![image-20221211134239610|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a231f62269fcd703739e0cbad8a39dd2.png)

#### 1.4.1 基本新增

员工表结构：

![image-20221211134746319|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/629c20b2deb886482ca0340f409b7681.png)

SQL语句：

```sql
insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values ('songyuanqiao','宋远桥',1,'1.jpg',2,'2012-10-09',2,'2022-10-01 10:00:00','2022-10-01 10:00:00');
```

接口方法：

```java
@Mapper
public interface EmpMapper {

    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
```

> 说明：#{...} 里面写的名称是对象的属性名

测试类：

```java
import com.itheima.mapper.EmpMapper;
import com.itheima.pojo.Emp;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.time.LocalDate;
import java.time.LocalDateTime;

@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("tom");
        emp.setName("汤姆");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);
    }
}

```

> 日志输出：
>
> ![image-20221211140222240|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/253406995d6951eeeb301e5680251eab.png)

#### 1.4.2 主键返回！

概念：在数据添加成功后，需要获取插入数据库数据的主键。

> 如：添加套餐数据时，还需要维护套餐菜品关系表数据。
>
> ![image-20221211150353385|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e510a6797ab904811bc184dbe65bdfda.png)
>
> 业务场景：在前面讲解到的苍穹外卖菜品与套餐模块的表结构，菜品与套餐是多对多的关系，一个套餐对应多个菜品。既然是多对多的关系，是不是有一张套餐菜品中间表来维护它们之间的关系。
>
> ![image-20221212093655389|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8651f3ec7883a2a5abb8ebbee15e826.png)
>
> 在添加套餐的时候，我们需要在界面当中来录入套餐的基本信息，还需要来录入套餐与菜品的关联信息。这些信息录入完毕之后，我们一点保存，就需要将套餐的信息以及套餐与菜品的关联信息都需要保存到数据库当中。其实具体的过程包括两步，首先第一步先需要将套餐的基本信息保存了，接下来第二步再来保存套餐与菜品的关联信息。套餐与菜品的关联信息就是往中间表当中来插入数据，来维护它们之间的关系。而中间表当中有两个外键字段，一个是菜品的ID，就是当前菜品的ID，还有一个就是套餐的ID，而这个套餐的 ID 指的就是此次我所添加的套餐的ID，所以我们在第一步保存完套餐的基本信息之后，就需要将套餐的主键值返回来供第二步进行使用。这个时候就需要用到主键返回功能。

那要如何实现在插入数据之后返回所插入行的主键值呢？

- 默认情况下，执行插入操作时，是不会主键值返回的。如果我们想要拿到主键值，需要在Mapper接口中的方法上添加一个Options注解，并在注解中指定属性useGeneratedKeys=true和keyProperty="实体类属性名"


主键返回代码实现：

~~~java
@Mapper
public interface EmpMapper {
    
    //会自动将生成的主键值，赋值给emp对象的id属性
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
~~~

测试：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("jack");
        emp.setName("杰克");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);

        System.out.println(emp.getDeptId());
    }
}
~~~


### 1.5 更新

功能：修改员工信息

![image-20221212095605863|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b853a4201749f9805efa0f73e1cd9fba.png)

> 点击"编辑"按钮后，会查询所在行记录的员工信息，并把员工信息回显在修改员工的窗体上(下个知识点学习)
>
> 在修改员工的窗体上，可以修改的员工数据：用户名、员工姓名、性别、图像、职位、入职日期、归属部门
>
> 思考：在修改员工数据时，要以什么做为条件呢？
>
> 答案：员工id

SQL语句：

```sql
update emp set username = 'linghushaoxia', name = '令狐少侠', gender = 1 , image = '1.jpg' , job = 2, entrydate = '2012-01-01', dept_id = 2, update_time = '2022-10-01 12:12:12' where id = 18;
```

接口方法：

```java
@Mapper
public interface EmpMapper {
    /**
     * 根据id修改员工信息
     * @param emp
     */
    @Update("update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}, job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}")
    public void update(Emp emp);
    
}
```

测试类：

```java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testUpdate(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(23);
        emp.setUsername("songdaxia");
        emp.setPassword(null);
        emp.setName("老宋");
        emp.setImage("2.jpg");
        emp.setGender((short)1);
        emp.setJob((short)2);
        emp.setEntrydate(LocalDate.of(2012,1,1));
        emp.setCreateTime(null);
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(2);
        //调用方法，修改员工数据
        empMapper.update(emp);
    }
}
```


### 1.6 查询

#### 1.6.1 根据ID查询

在员工管理的页面中，当我们进行更新数据时，会点击 “编辑” 按钮，然后此时会发送一个请求到服务端，会根据Id查询该员工信息，并将员工数据回显在页面上。

![image-20221212101331292|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a895acf69bfab64c8e146f71c83e3537.png) 

SQL语句：

~~~mysql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp;
~~~

接口方法：

~~~java
@Mapper
public interface EmpMapper {
    @Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
    public Emp getById(Integer id);
}
~~~

测试类：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testGetById(){
        Emp emp = empMapper.getById(1);
        System.out.println(emp);
    }
}
~~~

> 执行结果：
>
> ![image-20221212103004961|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04a630255727b8be38cdd47dbe3297bb.png)
>
> 而在测试的过程中，我们会发现有几个字段(deptId、createTime、updateTime)是没有数据值的


#### 1.6.2 数据封装！

我们看到查询返回的结果中大部分字段是有值的，但是deptId，createTime，updateTime这几个字段是没有值的，而数据库中是有对应的字段值的，这是为什么呢？

![image-20221212103124490|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0a4ec7c0a174ca528ece134d8c14377.png)

原因如下： 

- 实体类属性名和数据库表查询返回的字段名一致，mybatis会自动封装。
- 如果实体类属性名和数据库表查询返回的字段名不一致，不能自动封装。



 解决方案：

1. 起别名（数据库名称起别名，为代码属性名称）
2. 结果映射
3. 开启驼峰命名



**起别名**：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样

```java
@Select("select id, username, password, name, gender, image, job, entrydate, " +
        "dept_id AS deptId, create_time AS createTime, update_time AS updateTime " +
        "from emp " +
        "where id=#{id}")
public Emp getById(Integer id);
```

> 再次执行测试类：
>
> ![image-20221212111027396|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12e1f6f64418be23bab725d4a2cd568a.png)



**手动结果映射**：通过 @Results及@Result 进行手动结果映射

```java
@Results({@Result(column = "dept_id", property = "deptId"),
          @Result(column = "create_time", property = "createTime"),
          @Result(column = "update_time", property = "updateTime")})
@Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
public Emp getById(Integer id);
```

@Results源代码：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Results {
String id() default "";

Result[] value() default {};  //Result类型的数组
}
```

@Result源代码：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Repeatable(Results.class)
public @interface Result {
boolean id() default false;//表示当前列是否为主键（true:是主键）

String column() default "";//指定表中字段名

String property() default "";//指定类中属性名

Class<?> javaType() default void.class;

JdbcType jdbcType() default JdbcType.UNDEFINED;

Class<? extends TypeHandler> typeHandler() default UnknownTypeHandler.class;

One one() default @One;

Many many() default @Many;
}
```

**开启驼峰命名(推荐)**：如果字段名与属性名符合驼峰命名规则，mybatis会自动通过驼峰命名规则映射

> 驼峰命名规则：   abc_xyz    =>   abcXyz
>
> - 表中字段名：abc_xyz
> - 类中属性名：abcXyz

```properties
# 在application.properties中添加：
mybatis.configuration.map-underscore-to-camel-case=true
```

> 要使用驼峰命名前提是 实体类的属性 与 数据库表中的字段名严格遵守驼峰命名。

#### 1.6.3 条件查询！

在员工管理的列表页面中，我们需要根据条件查询员工信息，查询条件包括：姓名、性别、入职时间。 

![image-20221212113422924|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fad8f9e1151bc51147f598ea69577a8.png)

通过页面原型以及需求描述我们要实现的查询：

- 姓名：要求`支持模糊匹配`
- 性别：要求`精确匹配`
- 入职时间：要求`进行范围查询`
- 根据最后修改时间进行`降序排序`

SQL语句：

```sql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time 
from emp 
where name like '%张%' 
      and gender = 1 
      and entrydate between '2010-01-01' and '2020-01-01 ' 
order by update_time desc;
```

接口方法：

- 方式一

```java
@Mapper
public interface EmpMapper {
    @Select("select * from emp " +
            "where name like '%${name}%' " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);
}
```

> ![image-20221212115149151|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/758860817c526a380d34accd26dc5837.png)
>
> 以上方式注意事项：
>
> 1. 方法中的形参名和SQL语句中的参数占位符名保持一致
>
> 2. 模糊查询使用${...}进行字符串拼接，这种方式呢，由于是字符串拼接，并不是预编译的形式，所以效率不高、且存在sql注入风险。



- 方式二（解决SQL注入风险）
  - `使用MySQL提供的字符串拼接函数：concat('%' , '关键字' , '%')`

~~~java
@Mapper
public interface EmpMapper {

    @Select("select * from emp " +
            "where name like concat('%',#{name},'%') " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);

}

~~~

> 执行结果：生成的SQL都是预编译的SQL语句（性能高、安全）
>
> ![image-20221212120006242|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4440f6deea5397a5a566d2003c1e4b5.png)


#### 1.6.4 参数名说明

在上面我们所编写的条件查询功能中，我们需要保证接口中方法的形参名和SQL语句中的参数占位符名相同。

> 当方法中的形参名和SQL语句中的占位符参数名不相同时，就会出现以下问题：
>
> ![image-20221212150611796|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b864bb3e4655a48725fcbc41e92ccdc.png)



参数名在不同的SpringBoot版本中，处理方案还不同：

- 在springBoot的2.x版本（保证参数名一致）

![image-20221212151156273|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5290c97926460a92da8c1ae3c1e7e1f2.png)

> springBoot的父工程对compiler编译插件进行了默认的参数parameters配置，使得在编译时，会在生成的字节码文件中保留原方法形参的名称，所以#{…}里面可以直接通过形参名获取对应的值
>
> ![image-20221212151411154|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e0b716d80a78f33d8667fdeb9a30143.png)



- 在springBoot的1.x版本/单独使用mybatis（使用@Param注解来指定SQL语句中的参数名）

![image-20221212151628715|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2129c97a9fceb9a5975e5620450786e9.png)

> 在编译时，生成的字节码文件当中，不会保留Mapper接口中方法的形参名称，而是使用var1、var2、...这样的形参名字，此时要获取参数值时，就要通过@Param注解来指定SQL语句中的参数名
>
> ![image-20221212151736274|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5db0b2e29e1c7f733878dc4e8492b6d8.png)

## 2 Mybatis的XML配置文件

Mybatis的开发有两种方式：

1. 注解
2. XML

### 2.1 XML配置文件规范

使用Mybatis的注解方式，主要是来完成一些简单的增删改查功能。

`如果需要实现复杂的SQL功能，建议使用XML来配置映射语句，也就是将SQL语句写在XML配置文件中`。

在Mybatis中使用XML映射文件方式开发，需要符合一定的规范：

1. `XML映射文件的名称与Mapper接口名称一致`，并且将`XML映射文件和Mapper接口放置在相同包下（同包同名）`

2. XML映射文件的namespace属性为Mapper接口全限定名一致

3. XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。

![image-20221212153529732|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fa08cba9bcbc7cfb3e9748672ace019a.png)

> \<select>标签：就是用于编写select查询语句的。
>
> - resultType属性，指的是查询返回的单条记录所封装的类型。


### 2.2 XML配置文件实现

第1步：创建XML映射文件
- `切记包名要用 / 分隔`

![image-20221212154908306|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f7c425f8e68475c6b4408fa320f50ae.png)

![image-20221212155304635|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/818dd955aecc74002c0411573d8fb7d2.png)

![image-20221212155544404|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/002eb7a5a703b5c167917e46c149e4d0.png)



第2步：编写XML映射文件

> xml映射文件中的dtd约束，直接从mybatis官网复制即可

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="">
 
</mapper>
~~~



配置：XML映射文件的namespace属性为Mapper接口全限定名

![image-20221212160316644|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab13794272b3ed5e2ea4b507fad324ef.png)

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

</mapper>
~~~



配置：XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致

![image-20221212163528787|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9e3a7a547d45e9cb3c972fae846c51c.png)

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--查询操作-->
    <select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where name like concat('%',#{name},'%')
              and gender = #{gender}
              and entrydate between #{begin} and #{end}
        order by update_time desc
    </select>
</mapper>
~~~

- `注意：参数前要使用@Param注解`
```java
public List<Emp> list(@Param("name") String name, @Param("gender")Short gender,  
                      @Param("begin")LocalDate begin, @Param("end")LocalDate end);
```

> 运行测试类，执行结果：
>
> ![image-20221212163719534|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/662a1b47b5048569359959e91bc7aa15.png)

### 2.3 MybatisX的使用！

MybatisX是一款基于IDEA的快速开发Mybatis的插件，为效率而生。

MybatisX的安装：

![image-20221213120923252|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea5040d4f6d54fe4bf9d7e16c56a0a4d.png)

可以通过MybatisX快速定位：

![image-20221213121521406|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb6dc20183e837287abc239c51e70b93.png)

> MybatisX的使用在后续学习中会继续分享



学习了Mybatis中XML配置文件的开发方式了，大家可能会存在一个疑问：到底是使用注解方式开发还是使用XML方式开发？

> 官方说明：https://mybatis.net.cn/getting-started.html
>
> ![image-20220901173948645|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9fa59300569a4b6569ae56f8dd3d0fc.png) 

**结论：** 使用Mybatis的注解，主要是来完成一些简单的增删改查功能。如果需要实现复杂的SQL功能，建议使用XML来配置映射语句。

## 3 Mybatis动态SQL！

### 3.1 什么是动态SQL

在页面原型中，列表上方的条件是动态的，是可以不传递的，也可以只传递其中的1个或者2个或者全部。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ab6ac23be82e80b81bc0f435a79d5de.png)

![image-20220901173203491|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/833fee0ee0b782c8960a8c18d2d627cc.png)

而在我们刚才编写的SQL语句中，我们会看到，我们将三个条件直接写死了。 如果页面只传递了参数姓名name 字段，其他两个字段 性别 和 入职时间没有传递，那么这两个参数的值就是null。

此时，执行的SQL语句为：

![image-20220901173431554|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c22e18cc3a5ae5b3ab9e0398fa64fec.png) 

这个查询结果是不正确的。正确的做法应该是：`传递了参数，再组装这个查询条件；如果没有传递参数，就不应该组装这个查询条件`。

比如：如果姓名输入了"张", 对应的SQL为:

```sql
select *  from emp where name like '%张%' order by update_time desc;
```


如果姓名输入了"张",，性别选择了"男"，则对应的SQL为:

```sql
select *  from emp where name like '%张%' and gender = 1 order by update_time desc;
```

SQL语句会随着用户的输入或外部条件的变化而变化，我们称为：**动态SQL**。

![image-20221213122623278|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2c53dbe94fafa4288e2e3ae556f128f2.png)

在Mybatis中提供了很多实现动态SQL的标签，我们学习Mybatis中的动态SQL就是掌握这些动态SQL标签。

### 3.2 动态SQL-if

`<if>`：用于判断条件是否成立。使用test属性进行条件判断，如果条件为true，则拼接SQL。

~~~xml
<if test="条件表达式">
   要拼接的sql语句
</if>
~~~

接下来，我们就通过`<if>`标签来改造之前条件查询的案例。

#### 3.2.1 条件查询

示例：把SQL语句改造为动态SQL方式

- 原有的SQL语句

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where name like concat('%',#{name},'%')
              and gender = #{gender}
              and entrydate between #{begin} and #{end}
        order by update_time desc
</select>
~~~

- 动态SQL语句

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        where
    
             <if test="name != null">
                 name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
    
        order by update_time desc
</select>
~~~

测试方法：

~~~java
@Test
public void testList(){
    //性别数据为null、开始时间和结束时间也为null
    List<Emp> list = empMapper.list("张", null, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

> 执行的SQL语句： 
>
> ![image-20221213140353285|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4eae65c7dfccf561543b38f5d7c1566.png)


下面呢，我们修改测试方法中的代码，再次进行测试，观察执行情况：

~~~java
@Test
public void testList(){
    //姓名为null
    List<Emp> list = empMapper.list(null, (short)1, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

执行结果：

![image-20221213141139015|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34b2856f312a50e80148996767ca1c6e.png) 

![image-20221213141253355|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4be7fa3ff745e0b0e621f126237def5.png) 


再次修改测试方法中的代码，再次进行测试：

~~~java
@Test
public void testList(){
    //传递的数据全部为null
    List<Emp> list = empMapper.list(null, null, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

执行的SQL语句：

![image-20221213143854434|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fda2c6cb305d00cafdf8460a471df8a4.png)

以上问题的解决方案：使用`<where>`标签代替SQL语句中的where关键字

- `<where>`只会在子元素有内容的情况下才插入where子句，而且会自动去除子句的开头的AND或OR

~~~xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        <where>
             <!-- if做为where标签的子元素 -->
             <if test="name != null">
                 and name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
        </where>
        order by update_time desc
</select>
~~~

测试方法：

~~~java
@Test
public void testList(){
    //只有性别
    List<Emp> list = empMapper.list(null, (short)1, null, null);
    for(Emp emp : list){
        System.out.println(emp);
    }
}
~~~

> 执行的SQL语句：
>
> ![image-20221213141909455|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5122017261349a9bc04286f9c860ec68.png)

#### 3.2.2 更新员工

案例：完善更新员工功能，修改为动态更新员工数据信息

- 动态更新员工信息，如果更新时传递有值，则更新；如果更新时没有传递值，则不更新
- 解决方案：动态SQL

修改Mapper接口：

~~~java
@Mapper
public interface EmpMapper {
    //删除@Update注解编写的SQL语句
    //update操作的SQL语句编写在Mapper映射文件中
    public void update(Emp emp);
}
~~~

修改Mapper映射文件：

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--更新操作-->
    <update id="update">
        update emp
        set
            <if test="username != null">
                username=#{username},
            </if>
            <if test="name != null">
                name=#{name},
            </if>
            <if test="gender != null">
                gender=#{gender},
            </if>
            <if test="image != null">
                image=#{image},
            </if>
            <if test="job != null">
                job=#{job},
            </if>
            <if test="entrydate != null">
                entrydate=#{entrydate},
            </if>
            <if test="deptId != null">
                dept_id=#{deptId},
            </if>
            <if test="updateTime != null">
                update_time=#{updateTime}
            </if>
        where id=#{id}
    </update>

</mapper>
~~~

测试方法：

~~~java
@Test
public void testUpdate2(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(20);
        emp.setUsername("Tom111");
        emp.setName("汤姆111");

        emp.setUpdateTime(LocalDateTime.now());

        //调用方法，修改员工数据
        empMapper.update(emp);
}
~~~

> 执行的SQL语句：
>
> ![image-20221213152533851|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ca5f34e3b327bf33db0d4e76f0830e2f.png)



再次修改测试方法，观察SQL语句执行情况：

~~~java
@Test
public void testUpdate2(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(20);
        emp.setUsername("Tom222");
      
        //调用方法，修改员工数据
        empMapper.update(emp);
}
~~~

> 执行的SQL语句：
>
> ![image-20221213152850322|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79a68bd847c0e31d3421bede4c635237.png)

以上问题的解决方案：使用`<set>`标签代替SQL语句中的set关键字

- `<set>`：动态的在SQL语句中插入set关键字，并会删掉额外的逗号。（用于update语句中）

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!--更新操作-->
    <update id="update">
        update emp
        <!-- 使用set标签，代替update语句中的set关键字 -->
        <set>
            <if test="username != null">
                username=#{username},
            </if>
            <if test="name != null">
                name=#{name},
            </if>
            <if test="gender != null">
                gender=#{gender},
            </if>
            <if test="image != null">
                image=#{image},
            </if>
            <if test="job != null">
                job=#{job},
            </if>
            <if test="entrydate != null">
                entrydate=#{entrydate},
            </if>
            <if test="deptId != null">
                dept_id=#{deptId},
            </if>
            <if test="updateTime != null">
                update_time=#{updateTime}
            </if>
        </set>
        where id=#{id}
    </update>
</mapper>
~~~

> 再次执行测试方法，执行的SQL语句：
>
> ![image-20221213153329553|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8076d35903868eba0630d959e8de696.png)


**小结**

- `<if>`

  - 用于判断条件是否成立，如果条件为true，则拼接SQL

  - 形式：

    ~~~xml
    <if test="name != null"> … </if>
    ~~~

- `<where>`

  - where元素只会在子元素有内容的情况下才插入where子句，而且会自动去除子句的开头的AND或OR

- `<set>`

  - 动态地在行首插入 SET 关键字，并会删掉额外的逗号。（用在update语句中）

### 3.3 动态SQL-foreach

案例：员工删除功能（既支持删除单条记录，又支持批量删除）

![image-20220901181751004|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/efb157b491c0685d58e8928dea9b05dc.png) 

SQL语句：

~~~mysql
delete from emp where id in (1,2,3);
~~~

Mapper接口：

~~~java
@Mapper
public interface EmpMapper {
    //批量删除
    public void deleteByIds(List<Integer> ids);
}
~~~

XML映射文件：

- 使用`<foreach>`遍历deleteByIds方法中传递的参数ids集合

~~~xml
<foreach collection="集合名称" item="集合遍历出来的元素/项" separator="每一次遍历使用的分隔符" 
         open="遍历开始前拼接的片段" close="遍历结束后拼接的片段">
</foreach>
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">
    <!--删除操作-->
    <delete id="deleteByIds">
        delete from emp where id in
        <foreach collection="ids" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>
    </delete>
</mapper> 
~~~

> ![image-20221213165710141|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/76a9515cff2becf13fd0130d196e4663.png)

> 执行的SQL语句：
>
> ![image-20221213164957636|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f779f4112b493c3ee904547aa3b8eab.png)

### 3.4 动态SQL-sql&include

问题分析：

- 在xml映射文件中配置的SQL，有时可能会存在很多重复的片段，此时就会存在很多冗余的代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/52de06d43d40d73fb5e6d1b68b460ea0.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8bf82efbde3ed134448d983431e97f54.png)

我们可以对重复的代码片段进行抽取，将其通过`<sql>`标签封装到一个SQL片段，然后再通过`<include>`标签进行引用。

- `<sql>`：定义可重用的SQL片段

- `<include>`：通过属性refid，指定包含的SQL片段

![image-20221213171244796|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0b220335c1b7a4ceca54007d7153fb3.png)

SQL片段： 抽取重复的代码

```xml
<sql id="commonSelect">
 	select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp
</sql>
```

然后通过`<include>` 标签在原来抽取的地方进行引用。操作如下：

```xml
<select id="list" resultType="com.itheima.pojo.Emp">
    <include refid="commonSelect"/>
    <where>
        <if test="name != null">
            name like concat('%',#{name},'%')
        </if>
        <if test="gender != null">
            and gender = #{gender}
        </if>
        <if test="begin != null and end != null">
            and entrydate between #{begin} and #{end}
        </if>
    </where>
    order by update_time desc
</select>
```



## 4 MyBatis高级扩展

### 4.1 Mapper批量映射优化

1.  需求

    Mapper 配置文件很多时，在全局配置文件中一个一个注册太麻烦，希望有一个办法能够一劳永逸。
    
2.  配置方式

    `Mybatis 允许在指定 Mapper 映射文件时，只指定其所在的包`：
    ```xml
    <mappers>
        <package name="com.atguigu.mapper"/>
    </mappers>
    ```
    此时这个包下的所有 Mapper 配置文件将被自动加载、注册，比较方便。

3.  资源创建要求

-   Mapper 接口和 Mapper 配置文件名称一致
    -   Mapper 接口：EmployeeMapper.java
    -   Mapper 配置文件：EmployeeMapper.xml
-   Mapper 配置文件放在 Mapper 接口所在的包内
    -   可以将mapperxml文件放在mapper接口所在的包！
    -   可以在sources下创建mapper接口包一致的文件夹结构存放mapperxml文件

        ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/99a3d30c6ef5243b0305655238f3a319.png)


### 4.2 插件和分页插件PageHelper

#### 4.2.1 插件机制和PageHelper插件介绍

MyBatis 对插件进行了标准化的设计，并提供了一套可扩展的插件机制。插件可以在用于语句执行过程中进行拦截，并允许通过自定义处理程序来拦截和修改 SQL 语句、映射语句的结果等。

具体来说，MyBatis 的插件机制包括以下三个组件：

1.  `Interceptor`（拦截器）：定义一个拦截方法 `intercept`，该方法在执行 SQL 语句、执行查询、查询结果的映射时会被调用。
2.  `Invocation`（调用）：实际上是对被拦截的方法的封装，封装了 `Object target`、`Method method` 和 `Object[] args` 这三个字段。
3.  `InterceptorChain`（拦截器链）：对所有的拦截器进行管理，包括将所有的 Interceptor 链接成一条链，并在执行 SQL 语句时按顺序调用。

插件的开发非常简单，只需要实现 Interceptor 接口，并使用注解 `@Intercepts` 来标注需要拦截的对象和方法，然后在 MyBatis 的配置文件中添加插件即可。

`PageHelper 是 MyBatis 中比较著名的分页插件`，它提供了多种分页方式（例如 MySQL 和 Oracle 分页方式），支持多种数据库，并且使用非常简单。下面就介绍一下 PageHelper 的使用方式。

<https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md#如何配置数据库方言>

#### 4.2.2 PageHelper插件使用

1.  pom.xml引入依赖
    ```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.1.11</version>
</dependency>
    ```

2.  mybatis-config.xml配置分页插件
    在 MyBatis 的配置文件中添加 PageHelper 的插件：
    ```xml
<plugins>
	<plugin interceptor="com.github.pagehelper.PageInterceptor">
		<property name="helperDialect" value="mysql"/>
	</plugin>
</plugins>
    ```
    其中，`com.github.pagehelper.PageInterceptor` 是 PageHelper 插件的名称，`dialect` 属性用于指定数据库类型（支持多种数据库）

3.  页插件使用
    在查询方法中使用分页：
    ```java
    @Test
    public void testTeacherRelationshipToMulti() {

        TeacherMapper teacherMapper = session.getMapper(TeacherMapper.class);

        PageHelper.startPage(1,2);
        // 查询Customer对象同时将关联的Order集合查询出来
        List<Teacher> allTeachers = teacherMapper.findAllTeachers();
    //
        PageInfo<Teacher> pageInfo = new PageInfo<>(allTeachers);

        System.out.println("pageInfo = " + pageInfo);
        long total = pageInfo.getTotal(); // 获取总记录数
        System.out.println("total = " + total);
        int pages = pageInfo.getPages();  // 获取总页数
        System.out.println("pages = " + pages);
        int pageNum = pageInfo.getPageNum(); // 获取当前页码
        System.out.println("pageNum = " + pageNum);
        int pageSize = pageInfo.getPageSize(); // 获取每页显示记录数
        System.out.println("pageSize = " + pageSize);
        List<Teacher> teachers = pageInfo.getList(); //获取查询页的数据集合
        System.out.println("teachers = " + teachers);
        teachers.forEach(System.out::println);
    }
    ```

- `注意：不能将2条查询语句得到的结果放在一个分页区`

### 4.3 逆向工程和MybatisX插件

#### 4.3.1 ORM思维介绍

`ORM（Object-Relational Mapping，对象-关系映射）`是一种将数据库和面向对象编程语言中的对象之间进行转换的技术。它将对象和关系数据库的概念进行映射，最后我们就可以通过方法调用进行数据库操作!!

最终: **让我们可以使用面向对象思维进行数据库操作！！！**

**ORM 框架通常有半自动和全自动两种方式。**

-   `半自动 ORM` 通常需要程序员手动编写 SQL 语句或者配置文件，将实体类和数据表进行映射，还需要手动将查询的结果集转换成实体对象。
-   `全自动 ORM` 则是将实体类和数据表进行自动映射，使用 API 进行数据库操作时，ORM 框架会自动执行 SQL 语句并将查询结果转换成实体对象，程序员无需再手动编写 SQL 语句和转换代码。

**下面是半自动和全自动 ORM 框架的区别：**

1.  `映射方式`：半自动 ORM 框架需要程序员`手动指定实体类和数据表之间的映射关系`，通常使用 XML 文件或注解方式来指定；全自动 ORM 框架则可以`自动进行实体类和数据表的映射，无需手动干预`。

2.  `查询方式`：半自动 ORM 框架通常需要程序员`手动编写 SQL 语句并将查询结果集转换成实体对象`；全自动 ORM 框架可以`自动组装 SQL 语句、执行查询操作`，并将查询结果转换成实体对象。

3.  `性能`：由于半自动 ORM 框架需要手动编写 SQL 语句，因此程序员必须对 SQL 语句和数据库的底层知识有一定的了解，才能编写高效的 SQL 语句；而全自动 ORM 框架通过自动优化生成的 SQL 语句来提高性能，程序员无需进行优化。

4.  `学习成本`：半自动 ORM 框架需要程序员手动编写 SQL 语句和映射配置，要求程序员具备较高的数据库和 SQL 知识；全自动 ORM 框架可以自动生成 SQL 语句和映射配置，程序员无需了解过多的数据库和 SQL 知识。

常见的半自动 ORM 框架包括 MyBatis 等；常见的全自动 ORM 框架包括 Hibernate、Spring Data JPA、MyBatis-Plus 等。

#### 4.3.2 逆向工程

`MyBatis 的逆向工程是一种自动化生成持久层代码和映射文件的工具`，它可以根据数据库表结构和设置的参数生成对应的实体类、Mapper.xml 文件、Mapper 接口等代码文件，简化了开发者手动生成的过程。`逆向工程使开发者可以快速地构建起 DAO 层`，并快速上手进行业务开发。

MyBatis 的逆向工程有两种方式：通过 `MyBatis Generator 插件`实现和通过 Maven 插件实现。无论是哪种方式，逆向工程一般需要指定一些配置参数，例如数据库连接 URL、用户名、密码、要生成的表名、生成的文件路径等等。

总的来说，MyBatis 的逆向工程为程序员提供了一种方便快捷的方式，能够快速地生成持久层代码和映射文件，是半自动 ORM 思维像全自动发展的过程，提高程序员的开发效率。

**注意：逆向工程只能生成单表crud的操作，多表查询依然需要我们自己编写！**

#### 4.3.3 逆向工程插件MyBatisX使用

MyBatisX 是一个 MyBatis 的代码生成插件，可以通过简单的配置和操作快速生成 MyBatis Mapper、pojo 类和 Mapper.xml 文件。下面是使用 MyBatisX 插件实现逆向工程的步骤：

1.  安装插件：

    在 IntelliJ IDEA 中打开插件市场，搜索 MyBatisX 并安装。
2.  使用 IntelliJ IDEA连接数据库
    -   连接数据库![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fddd7d66c7c5586aa9ff4b81a9f45452.png)


    -   填写信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5675681efa73e586fab2c311bdcbfd84.png)


    -   展示库表![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8ea0be52476aabc12312b2078d30b8c.png)


    -   逆向工程使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f34ee2a18a610b8198c9105eca6dcc1.png)![[image_KXYfK5CQd-.png]]

3.  查看生成结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b87fd3db111fc94fa6a0b2972a5b8c61.png)


4.  逆向工程案例使用

    正常使用即可，自动生成单表的crud方法！
    ```java
    package com.atguigu.mapper;

    import com.atguigu.pojo.User;

    /**
    * @author Jackiechan
    * @description 针对表【user】的数据库操作Mapper
    * @createDate 2023-06-02 16:55:32
    * @Entity com.atguigu.pojo.User
    */
    public interface UserMapper {

        int deleteByPrimaryKey(Long id);

        int insert(User record);

        int insertSelective(User record);

        User selectByPrimaryKey(Long id);

        int updateByPrimaryKeySelective(User record);

        int updateByPrimaryKey(User record);

    }

    ```

## 5 MyBatis总结

| 核心点           | 掌握目标                                    |
| ------------- | --------------------------------------- |
| mybatis基础     | 使用流程, 参数输入,#{} \${},参数输出                |
| mybatis多表     | 实体类设计,resultMap多表结果映射                   |
| `mybatis动态语句` | `Mybatis动态语句概念, where , if , foreach标签` |
| mybatis扩展     | Mapper批量处理,分页插件,逆向工程                    |
