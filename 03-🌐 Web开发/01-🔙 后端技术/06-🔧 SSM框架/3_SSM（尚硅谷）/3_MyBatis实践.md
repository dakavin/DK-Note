## 1 Mybatis简介

### 1.1 简介

<https://mybatis.org/mybatis-3/zh/index.html>

MyBatis最初是Apache的一个开源项目iBatis, 2010年6月这个项目由Apache Software Foundation迁移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于2013年11月迁移到Github。

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。`MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/986017abc87189f94977a8e75faf1a58.png)



> 社区会持续更新开源项目，版本会不断变化，我们不必每个小版本都追，关注重大更新的大版本升级即可。

本课程使用：3.5.11版本

### 1.2 持久层框架对比

-   JDBC
    -   SQL 夹杂在Java代码中耦合度高，导致硬编码内伤
    -   维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见
    -   代码冗长，开发效率低
-   Hibernate 和 JPA
    -   操作简便，开发效率高
    -   程序中的长难复杂 SQL 需要绕过框架
    -   内部自动生成的 SQL，不容易做特殊优化
    -   基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。
    -   反射操作太多，导致数据库性能下降
-   MyBatis
    -   轻量级，性能出色
    -   SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
    -   开发效率稍逊于 Hibernate，但是完全能够接收

`开发效率：Hibernate>Mybatis>JDBC`

`运行效率：JDBC>Mybatis>Hibernate`

### 1.3 快速入门（基于Mybatis3方式）

1.  准备数据模型
    ```sql
    CREATE DATABASE `mybatis-example`;

    USE `mybatis-example`;

    CREATE TABLE `t_emp`(
      emp_id INT AUTO_INCREMENT,
      emp_name CHAR(100),
      emp_salary DOUBLE(10,5),
      PRIMARY KEY(emp_id)
    );

    INSERT INTO `t_emp`(emp_name,emp_salary) VALUES("tom",200.33);
    INSERT INTO `t_emp`(emp_name,emp_salary) VALUES("jerry",666.66);
    INSERT INTO `t_emp`(emp_name,emp_salary) VALUES("andy",777.77);
    ```
2.  项目搭建和准备
    1.  项目搭建
        ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f25d2c14161c0846e48255a342dd8be5.png)
    2.  依赖导入
        pom.xml
        ```xml
        <dependencies>
          <!-- mybatis依赖 -->
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.5.11</version>
          </dependency>

          <!-- MySQL驱动 mybatis底层依赖jdbc驱动实现,本次不需要导入连接池,mybatis自带! -->
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>8.0.25</version>
          </dependency>

          <!--junit5测试-->
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter-api</artifactId>
              <version>5.3.1</version>
          </dependency>
        </dependencies>
        ```

	3.  实体类准备
        ```java
        public class Employee {
            private Integer empId;
            private String empName;
            private Double empSalary;
            //getter | setter
        }
        ```

3.  准备Mapper接口和MapperXML文件

    MyBatis 框架下，SQL语句编写位置发生改变，从原来的Java类，`改成XML或者注解定义`！

    推荐在XML文件中编写SQL语句，让用户能更专注于 SQL 代码，不用关注其他的JDBC代码。

    如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码！！

    `一般编写SQL语句的文件命名：XxxMapper.xml  Xxx一般取表名！！`

    Mybatis 中的 Mapper 接口相当于以前的 Dao。但是区别在于，Mapper 仅仅只是建接口即可，我们不需要提供实现类，具体的SQL写到对应的Mapper文件，该用法的思路如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/907cd314a1187f72bc7a36214b3d5482.png)


    1.  定义mapper接口
        包：com.atguigu.mapper
        ```java
/**
* t_emp表对应数据库SQL语句映射接口!
*    接口只规定方法,参数和返回值!
*    mapper.xml中编写具体SQL语句!
*/
public interface EmployeeMapper {
	/**
	 * 根据员工id查询员工数据方法
	 * @param empId  员工id
	 * @return 员工实体对象
	 */
	Employee selectEmployee(Integer empId);

}
        ```

	2.  定义mapper xml
        位置： resources/mappers/EmployeeMapper.xml
        ```xml
<!-- namespace等于mapper接口类的全限定名,这样实现对应 -->
<mapper namespace="com.atguigu.mapper.EmployeeMapper">

<!-- 查询使用 select标签
		id = 方法名
		resultType = 返回值类型
		标签内编写SQL语句
 -->
<select id="selectEmployee"resultType="com.atguigu.pojo.Employee">
	<!-- #{empId}代表动态传入的参数,并且进行赋值!后面详细讲解 -->
	select emp_id empId,emp_name empName, emp_salary empSalary from t_emp where emp_id = #{empId}
</select>
</mapper>
        ```
        **注意：**
        -   `方法名和SQL的id一致`
        -   `方法返回值和resultType一致`
        -   `方法的参数和SQL的参数一致`
        -   `接口的全类名和映射配置文件的名称空间一致`

4.  准备MyBatis配置文件

    mybatis框架配置文件： 数据库连接信息，性能配置，mapper.xml配置等！

    习惯上命名为 `mybatis-config.xml`，这个文件名仅仅只是建议，并非强制要求。将来整合 Spring 之后，这个配置文件可以省略，所以大家操作时可以直接复制、粘贴。
    ```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

  <!-- environments表示配置Mybatis的开发环境，可以配置多个环境，在众多具体环境中，使用default属性指定实际运行时使用的环境。default属性的取值是environment标签的id属性的值。 -->
  <environments default="development">
	<!-- environment表示配置Mybatis的一个具体的环境 -->
	<environment id="development">
	  <!-- Mybatis的内置的事务管理器 -->
	  <transactionManager type="JDBC"/>
	  <!-- 配置数据源 -->
	  <dataSource type="POOLED">
		<!-- 建立数据库连接的具体信息 -->
		<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://localhost:3306/mybatis-example"/>
		<property name="username" value="root"/>
		<property name="password" value="root"/>
	  </dataSource>
	</environment>
  </environments>

  <mappers>
	<!-- Mapper注册：指定Mybatis映射文件的具体位置 -->
	<!-- mapper标签：配置一个具体的Mapper映射文件 -->
	<!-- resource属性：指定Mapper映射文件的实际存储位置，这里需要使用一个以类路径根目录为基准的相对路径 -->
	<!--    对Maven工程的目录结构来说，resources目录下的内容会直接放入类路径，所以这里我们可以以resources目录为基准 -->
	<mapper resource="mappers/EmployeeMapper.xml"/>
  </mappers>

</configuration>
    ```

5.  运行和测试
    ```java
/**
 * projectName: com.atguigu.test
 *
 * description: 测试类
 */
public class MyBatisTest {

	@Test
	public void testSelectEmployee() throws IOException {

		// 1.创建SqlSessionFactory对象
		// ①声明Mybatis全局配置文件的路径
		String mybatisConfigFilePath = "mybatis-config.xml";

		// ②以输入流的形式加载Mybatis配置文件
		InputStream inputStream = Resources.getResourceAsStream(mybatisConfigFilePath);

		// ③基于读取Mybatis配置文件的输入流创建SqlSessionFactory对象
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

		// 2.使用SqlSessionFactory对象开启一个会话
		SqlSession session = sessionFactory.openSession();

		// 3.根据EmployeeMapper接口的Class对象获取Mapper接口类型的对象(动态代理技术)
		EmployeeMapper employeeMapper = session.getMapper(EmployeeMapper.class);

		// 4. 调用代理类方法既可以触发对应的SQL语句
		Employee employee = employeeMapper.selectEmployee(1);

		System.out.println("employee = " + employee);

		// 4.关闭SqlSession
		session.commit(); //提交事务 [DQL不需要,其他需要]
		session.close(); //关闭会话

	}
}
    ```
    说明：
    -   SqlSession：代表Java程序和数据库之间的会话。（HttpSession是Java程序和浏览器之间的会话）
    -   SqlSessionFactory：是“生产”SqlSession的“工厂”。
    -   工厂模式：如果创建某一个对象，使用的过程基本固定，那么我们就可以把创建这个对象的相关代码封装到一个“工厂类”中，以后都使用这个工厂类来“生产”我们需要的对象。

6.  SqlSession和HttpSession区别
    -   HttpSession：工作在Web服务器上，属于表述层。
        -   代表浏览器和Web服务器之间的会话。
    -   SqlSession：不依赖Web服务器，属于持久化层。
        -   代表Java程序和数据库之间的会话。
            ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/195b69b5f3033cac949d728fe93e2735.png)


## 2 MyBatis基本使用

### 2.1 向SQL语句传参

#### 2.1.1 **mybatis日志输出配置**

mybatis配置文件设计标签和顶层结构如下：

-   configuration（配置）
    -   [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties "properties（属性）")
    -   [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings "settings（设置）")
    -   [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases "typeAliases（类型别名）")
    -   [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers "typeHandlers（类型处理器）")
    -   [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory "objectFactory（对象工厂）")
    -   [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins "plugins（插件）")
    -   [environments（环境配置）](https://mybatis.org/mybatis-3/zh/configuration.html#environments "environments（环境配置）")
        -   environment（环境变量）
            -   transactionManager（事务管理器）
            -   dataSource（数据源）
    -   [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider "databaseIdProvider（数据库厂商标识）")
    -   [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers "mappers（映射器）")

我们可以在mybatis的配置文件使用**settings标签**设置，输出运过程SQL日志！

通过查看日志，我们可以判定`#{}` 和 `${}`的输出效果！

settings设置项：

| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| LOG4J（3.5.9 起废弃） \| LOG4J2 \| JDK\_LOGGING \| COMMONS\_LOGGING \| STDOUT\_LOGGING \| NO\_LOGGING | 未设置 |
| ------- | ------------------------------- | --------------------------------------------------------------------------------------------------------- | --- |

日志配置：
```xml
<settings>
  <!-- SLF4J 选择slf4j输出！ -->
  <setting name="logImpl" value="SLF4J"/>
</settings>
```

#### 2.1.2 #{}形式

Mybatis会将SQL语句中的#{}转换为问号占位符。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6d0b39ba7dc439f73e35b441e45598bf.png)


#### 2.1.3 ${}形式

\${}形式传参，底层Mybatis做的是字符串拼接操作。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/38a7c2e29597b1697be4dff7322a16fd.png)

通常不会采用`${}`的方式传值。

一个特定的适用场景是：通过Java程序动态生成数据库表，表名部分需要Java程序通过参数传入；而JDBC对于表名部分是不能使用问号占位符的，此时可以使用

结论：`实际开发中，能用#{}实现的，肯定不用${}。`

特殊情况： 动态的不是值，是列名或者关键字，需要使用\${}拼接

```java
//注解方式传入参数！！
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, 
                                @Param("value") String value);
```

### 2.2 数据输入！！

#### 2.2.1 Mybatis总体机制概括

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7d6b49e2ec660dc9536384a19b335612.png)



#### 2.2.2 概念说明

这里数据输入具体是指上层方法（例如Service方法）调用Mapper接口时，数据传入的形式。

-   `简单类型：只包含一个值的数据类型`
    -   基本数据类型：int、byte、short、double、……
    -   基本数据类型的包装类型：Integer、Character、Double、……
    -   字符串类型：String
-   `复杂类型：包含多个值的数据类型`
    -   实体类类型：Employee、Department、……
    -   集合类型：List、Set、Map、……
    -   数组类型：int\[]、String\[]、……
    -   复合类型：List\<Employee>、实体类中包含集合……

#### 2.2.3 单个简单类型参数

Mapper接口中抽象方法的声明

```java
Employee selectEmployee(Integer empId);
```

SQL语句

```xml
<select id="selectEmployee" resultType="com.atguigu.mybatis.entity.Employee">
  select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{empId}
</select>
```

> 单个简单类型参数，在#{}中可以随意命名，但是没有必要。通常还是使用和接口方法参数同名。

#### 2.2.4 📕实体类类型参数

Mapper接口中抽象方法的声明

```java
int insertEmployee(Employee employee);
```

SQL语句

```xml
<insert id="insertEmployee">
  insert into t_emp(emp_name,emp_salary) values(#{empName},#{empSalary})
</insert>
```

对应关系

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bc3941f0a286d082167056bece4cd6e7.png)



结论

Mybatis会根据#{}中传入的数据，加工成getXxx()方法，通过反射在实体类对象中调用这个方法，从而获取到对应的数据。填充到#{}解析后的问号占位符这个位置。

#### 2.2.5 零散的简单类型数据

零散的多个简单类型参数，如果没有特殊处理，那么Mybatis无法识别自定义名称：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/97a1bf058385b1d08016c725bda8790f.png)



Mapper接口中抽象方法的声明
```java
int updateEmployee(@Param("empId") Integer empId,@Param("empSalary") Double empSalary);
```

SQL语句

```xml
<update id="updateEmployee">
  update t_emp set emp_salary=#{empSalary} where emp_id=#{empId}
</update>
```

对应关系

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9f0def9ed9d06f23cc74bceff0bee1e7.png)

#### 2.2.6 Map类型参数

Mapper接口中抽象方法的声明

```java
int updateEmployeeByMap(Map<String, Object> paramMap);
```

SQL语句

```xml
<update id="updateEmployeeByMap">

  update t_emp set emp_salary=#{empSalaryKey} where emp_id=#{empIdKey}

</update>
```

junit测试

```java
private SqlSession session;
//junit5会在每一个@Test方法前执行@BeforeEach方法
@BeforeEach
public void init() throws IOException {
    session = new SqlSessionFactoryBuilder()
            .build(
                    Resources.getResourceAsStream("mybatis-config.xml"))
            .openSession();
}

@Test
public void testUpdateEmpNameByMap() {
  EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
  Map<String, Object> paramMap = new HashMap<>();
  paramMap.put("empSalaryKey", 999.99);
  paramMap.put("empIdKey", 5);
  int result = mapper.updateEmployeeByMap(paramMap);
  log.info("result = " + result);
}

//junit5会在每一个@Test方法后执行@@AfterEach方法
@AfterEach
public void clear() {
    session.commit();
    session.close();
}

```

**对应关系**

`#{}中写Map中的key`

**使用场景**

有很多零散的参数需要传递，但是没有对应的实体类类型可以使用。使用@Param注解一个一个传入又太麻烦了。所以都封装到Map中。

### 2.3 数据输出！！

#### 2.3.1 输出概述

数据输出总体上有两种形式：
-   增删改操作返回的受影响行数：直接使用 int 或 long 类型接收即可
-   查询操作的查询结果

我们需要做的是，指定查询的输出数据类型即可！

并且`插入场景下，实现主键数据回显示！`

#### 2.3.2 单个简单类型

Mapper接口中的抽象方法

```java
int selectEmpCount();
```

SQL语句

```xml
<select id="selectEmpCount" resultType="int">
  select count(*) from t_emp
</select>
```

> Mybatis 内部给常用的数据类型设定了很多别名。 以 int 类型为例，可以写的名称有：int、integer、Integer、java.lang.Integer、Int、INT、INTEGER 等等。

junit测试

```java
@Test

public void testEmpCount() {

  EmployeeMapper employeeMapper = session.getMapper(EmployeeMapper.class);

  int count = employeeMapper.selectEmpCount();

  log.info("count = " + count);

}
```

**细节解释：**

select标签，通过resultType指定查询返回值类型！

resultType = "全限定符 ｜ 别名 ｜ 如果是返回集合类型，写范型类型即可"

`别名问题`：

[https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases "https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases")

类型别名可为 Java 类型设置一个缩写名字。 它==仅用于 XML 配置==，意在降低冗余的全限定类名书写。例如：

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
</typeAliases>
```
当这样配置时，`Blog` 可以用在任何使用 `domain.blog.Blog` 的地方。


也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：
```xml
<typeAliases> <package name="domain.blog"/> </typeAliases>
```

每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；若有注解，则别名为其注解值。见下面的例子：
```java
@Alias("author")
public class Author {
    ...
}
```

下面是Mybatis为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ebb984b1f9f4c53c41248b05e6e6cbf5.png)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c03ef3dcbf4bb1d66f7789533c2f7db.png)


| 别名                         | 映射的类型      |
| -------------------------- | ---------- |
| \_byte                     | byte       |
| \_char (since 3.5.10)      | char       |
| \_character (since 3.5.10) | char       |
| \_long                     | long       |
| \_short                    | short      |
| \_int                      | int        |
| \_integer                  | int        |
| \_double                   | double     |
| \_float                    | float      |
| \_boolean                  | boolean    |
| string                     | String     |
| byte                       | Byte       |
| char (since 3.5.10)        | Character  |
| character (since 3.5.10)   | Character  |
| long                       | Long       |
| short                      | Short      |
| int                        | Integer    |
| integer                    | Integer    |
| double                     | Double     |
| float                      | Float      |
| boolean                    | Boolean    |
| date                       | Date       |
| decimal                    | BigDecimal |
| bigdecimal                 | BigDecimal |
| biginteger                 | BigInteger |
| object                     | Object     |
| object\[]                  | Object\[]  |
| map                        | Map        |
| hashmap                    | HashMap    |
| list                       | List       |
| arraylist                  | ArrayList  |
| collection                 | Collection |

#### 2.3.3 返回实体类对象

Mapper接口的抽象方法

```java
Employee selectEmployee(Integer empId);
```

SQL语句

```xml
<!-- 编写具体的SQL语句，使用id属性唯一的标记一条SQL语句 -->
<!-- resultType属性：指定封装查询结果的Java实体类的全类名 -->
<select id="selectEmployee" resultType="com.atguigu.mybatis.entity.Employee">

  <!-- Mybatis负责把SQL语句中的#{}部分替换成“?”占位符 -->
  <!-- 给每一个字段设置一个别名，让别名和Java实体类中属性名一致 -->
  select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{maomi}

</select>
```

通过给数据库表字段加别名，让查询结果的每一列都和Java实体类中属性对应起来。

`增加全局配置自动识别对应关系`

在 Mybatis 全局配置文件中，做了下面的配置，select语句中可以不给字段设置别名

```xml
<!-- 在全局范围内对Mybatis进行配置 -->
<settings>

  <!-- 具体配置 -->
  <!-- 从org.apache.ibatis.session.Configuration类中可以查看能使用的配置项 -->
  <!-- 将mapUnderscoreToCamelCase属性配置为true，表示开启自动映射驼峰式命名规则 -->
  <!-- 规则要求数据库表字段命名方式：单词_单词 -->
  <!-- 规则要求Java实体类属性名命名方式：首字母小写的驼峰式命名 -->
  <setting name="mapUnderscoreToCamelCase" value="true"/>

</settings>
```

#### 2.3.4 返回Map类型！！

适用于SQL查询返回的各个字段综合起来并不和任何一个现有的实体类对应，`没法封装到实体类对象中`。能够封装成实体类类型的，就不使用Map类型。

Mapper接口的抽象方法
```java
Map<String,Object> selectEmpNameAndMaxSalary();
```

SQL语句
```xml
<!-- Map<String,Object> selectEmpNameAndMaxSalary(); -->
<!-- 返回工资最高的员工的姓名和他的工资 -->
<select id="selectEmpNameAndMaxSalary" resultType="map">
  SELECT
    emp_name 员工姓名,
    emp_salary 员工工资,
    (SELECT AVG(emp_salary) FROM t_emp) 部门平均工资
  FROM t_emp WHERE emp_salary=(
    SELECT MAX(emp_salary) FROM t_emp
  )
</select>
```

junit测试

```java
@Test
public void testQueryEmpNameAndSalary() {

  EmployeeMapper employeeMapper = session.getMapper(EmployeeMapper.class);

  Map<String, Object> resultMap = employeeMapper.selectEmpNameAndMaxSalary();

  Set<Map.Entry<String, Object>> entrySet = resultMap.entrySet();

  for (Map.Entry<String, Object> entry : entrySet) {

    String key = entry.getKey();

    Object value = entry.getValue();

    log.info(key + "=" + value);

  }
}
```

#### 2.3.5 返回List类型

查询结果返回多个实体类对象，希望把多个实体类对象放在List集合中返回。`此时不需要任何特殊处理，在resultType属性中还是设置实体类类型即可`。

Mapper接口中抽象方法
```java
List<Employee> selectAll();
```

SQL语句
```xml
<!-- List<Employee> selectAll(); -->
<select id="selectAll" resultType="com.atguigu.mybatis.entity.Employee">
  select emp_id empId,emp_name empName,emp_salary empSalary
  from t_emp
</select>
```

junit测试
```java
@Test
public void testSelectAll() {
  EmployeeMapper employeeMapper = session.getMapper(EmployeeMapper.class);
  List<Employee> employeeList = employeeMapper.selectAll();
  for (Employee employee : employeeList) {
    log.info("employee = " + employee);
  }
}
```

#### 2.3.6 返回主键值

1.  **自增长类型主键**

    Mapper接口中的抽象方法
    ```java
    int insertEmployee(Employee employee);
    ```
    SQL语句
    ```xml
    <!-- int insertEmployee(Employee employee); -->
    <!-- useGeneratedKeys属性字面意思就是“使用生成的主键” -->
    <!-- keyProperty属性可以指定主键在实体类对象中对应的属性名，Mybatis会将拿到的主键值存入这个属性 -->
    <insert id="insertEmployee" useGeneratedKeys="true" keyProperty="empId">
      insert into t_emp(emp_name,emp_salary)
      values(#{empName},#{empSalary})
    </insert>
    ```
    junit测试
    ```java
    @Test
    public void testSaveEmp() {
      EmployeeMapper employeeMapper = session.getMapper(EmployeeMapper.class);
      Employee employee = new Employee();
      employee.setEmpName("john");
      employee.setEmpSalary(666.66);
      employeeMapper.insertEmployee(employee);
      log.info("employee.getEmpId() = " + employee.getEmpId());
    }
    ```
    注意
    Mybatis是将自增主键的值设置到实体类对象中，而不是以Mapper接口方法返回值的形式返回。

2.  **非自增长类型主键**
    而对于不支持自增型主键的数据库（例如 Oracle）或者字符串类型主键，则可以使用 selectKey 子元素：selectKey 元素将会首先运行，id 会被设置，然后插入语句会被调用！
    ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48bf0a1852a4c361fbde5bf752420906.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7f7f4dc89638ea4d7f222c17f6fdf13.png)



    使用 `selectKey` 帮助插入UUID作为字符串类型主键示例：
    ```xml
    <insert id="insertUser" parameterType="User">
        <selectKey keyProperty="id" resultType="java.lang.String"
            order="BEFORE">
            SELECT UUID() as id
        </selectKey>
        INSERT INTO user (id, username, password) 
        VALUES (
            #{id},
            #{username},
            #{password}
        )
    </insert>
    ```
    在上例中，我们定义了一个 `insertUser` 的插入语句来将 `User` 对象插入到 `user` 表中。我们使用 `selectKey` 来查询 UUID 并设置到 `id` 字段中。

    通过 `keyProperty` 属性来指定查询到的 UUID 赋值给对象中的 `id` 属性，而 `resultType` 属性指定了 UUID 的类型为 `java.lang.String`。

    需要注意的是，我们将 `selectKey` 放在了插入语句的前面，这是因为 MySQL 在 `insert` 语句中只支持一个 `select` 子句，而 `selectKey` 中查询 UUID 的语句就是一个 `select` 子句，因此我们需要将其放在前面。

    最后，在将 `User` 对象插入到 `user` 表中时，我们直接使用对象中的 `id` 属性来插入主键值。

    使用这种方式，我们可以方便地插入 UUID 作为字符串类型主键。当然，还有其他插入方式可以使用，如使用Java代码生成UUID并在类中显式设置值等。需要根据具体应用场景和需求选择合适的插入方式。

#### 2.3.7 实体类属性和数据库字段对应关系

1.  别名对应

    `将字段的别名设置成和实体类属性一致`。
    ```xml
    <!-- 编写具体的SQL语句，使用id属性唯一的标记一条SQL语句 -->
    <!-- resultType属性：指定封装查询结果的Java实体类的全类名 -->
    <select id="selectEmployee" resultType="com.atguigu.mybatis.entity.Employee">

      <!-- Mybatis负责把SQL语句中的#{}部分替换成“?”占位符 -->
      <!-- 给每一个字段设置一个别名，让别名和Java实体类中属性名一致 -->
      select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{maomi}

    </select>
    ```
    > 关于实体类属性的约定：
    > getXxx()方法、setXxx()方法把方法名中的get或set去掉，首字母小写。

2.  `全局配置自动识别驼峰式命名规则`

    在Mybatis全局配置文件加入如下配置：
    ```xml
    <!-- 使用settings对Mybatis全局进行设置 -->
    <settings>

      <!-- 将xxx_xxx这样的列名自动映射到xxXxx这样驼峰式命名的属性名 -->
      <setting name="mapUnderscoreToCamelCase" value="true"/>

    </settings>
    ```
    SQL语句中可以不使用别名
    ```xml
    <!-- Employee selectEmployee(Integer empId); -->
    <select id="selectEmployee" resultType="com.atguigu.mybatis.entity.Employee">

      select emp_id,emp_name,emp_salary from t_emp where emp_id=#{empId}

    </select>
    ```

3.  `使用resultMap`

    使用resultMap标签定义对应关系，再在后面的SQL语句中引用这个对应关系
    ```xml
    <!-- 专门声明一个resultMap设定column到property之间的对应关系 -->
    <resultMap id="selectEmployeeByRMResultMap" type="com.atguigu.mybatis.entity.Employee">

      <!-- 使用id标签设置主键列和主键属性之间的对应关系 -->
      <!-- column属性用于指定字段名；property属性用于指定Java实体类属性名 -->
      <id column="emp_id" property="empId"/>

      <!-- 使用result标签设置普通字段和Java实体类属性之间的关系 -->
      <result column="emp_name" property="empName"/>

      <result column="emp_salary" property="empSalary"/>

    </resultMap>

    <!-- Employee selectEmployeeByRM(Integer empId); -->
    <select id="selectEmployeeByRM" resultMap="selectEmployeeByRMResultMap">

      select emp_id,emp_name,emp_salary from t_emp where emp_id=#{empId}

    </select>
    ```

### 2.4 CRUD强化练习

1.  准备数据库数据

    首先，我们需要准备一张名为 `user` 的表。该表包含字段 id（主键）、username、password。创建SQL如下：
    ```sql
    CREATE TABLE `user` (
      `id` INT(11) NOT NULL AUTO_INCREMENT,
      `username` VARCHAR(50) NOT NULL,
      `password` VARCHAR(50) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
    ```
2.  实体类准备

    接下来，我们需要定义一个实体类 `User`，来对应 user 表的一行数据。
    ```sql
    @Data //lombok
    public class User {
      private Integer id;
      private String username;
      private String password;
    }
    ```
3.  Mapper接口定义

    定义一个 Mapper 接口 `UserMapper`，并在其中添加 user 表的增、删、改、查方法。
    ```java
    public interface UserMapper {
      
      int insert(User user);

      int update(User user);

      int delete(Integer id);

      User selectById(Integer id);

      List<User> selectAll();
    }
    ```
4.  MapperXML编写

    在 resources /mappers目录下创建一个名为 `UserMapper.xml` 的 XML 文件，包含与 Mapper 接口中相同的五个 SQL 语句，并在其中，将查询结果映射到 `User` 实体中。
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- namespace等于mapper接口类的全限定名,这样实现对应 -->
    <mapper namespace="com.atguigu.mapper.UserMapper">
      <!-- 定义一个插入语句，并获取主键值 -->
      <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user(username, password)
                    VALUES(#{username}, #{password})
      </insert>
      
      <update id="update">
        UPDATE user SET username=#{username}, password=#{password}
        WHERE id=#{id}
      </update>
      
      <delete id="delete">
        DELETE FROM user WHERE id=#{id}
      </delete>
      <!-- resultType使用user别名，稍后需要配置！-->
      <select id="selectById" resultType="user">
        SELECT id, username, password FROM user WHERE id=#{id}
      </select>
      
      <!-- resultType返回值类型为集合，所以只写范型即可！ -->
      <select id="selectAll" resultType="user">
        SELECT id, username, password FROM user
      </select>
      
    </mapper>

    ```
5.  MyBatis配置文件

    位置：resources: mybatis-config.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- 开启驼峰式映射-->
            <setting name="mapUnderscoreToCamelCase" value="true"/>
            <!-- 开启logback日志输出-->
            <setting name="logImpl" value="SLF4J"/>
        </settings>

        <typeAliases>
            <!-- 给实体类起别名 -->
            <package name="com.atguigu.pojo"/>
        </typeAliases>

        <!-- environments表示配置Mybatis的开发环境，可以配置多个环境，在众多具体环境中，使用default属性指定实际运行时使用的环境。default属性的取值是environment标签的id属性的值。 -->
        <environments default="development">
            <!-- environment表示配置Mybatis的一个具体的环境 -->
            <environment id="development">
                <!-- Mybatis的内置的事务管理器 -->
                <transactionManager type="JDBC"/>
                <!-- 配置数据源 -->
                <dataSource type="POOLED">
                    <!-- 建立数据库连接的具体信息 -->
                    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/mybatis-example"/>
                    <property name="username" value="root"/>
                    <property name="password" value="root"/>
                </dataSource>
            </environment>
        </environments>

        <mappers>
            <!-- Mapper注册：指定Mybatis映射文件的具体位置 -->
            <!-- mapper标签：配置一个具体的Mapper映射文件 -->
            <!-- resource属性：指定Mapper映射文件的实际存储位置，这里需要使用一个以类路径根目录为基准的相对路径 -->
            <!--    对Maven工程的目录结构来说，resources目录下的内容会直接放入类路径，所以这里我们可以以resources目录为基准 -->
            <mapper resource="mappers/UserMapper.xml"/>
        </mappers>

    </configuration>
    ```
6.  效果测试
    ```java
    package com.atguigu.test;

    import com.atguigu.mapper.UserMapper;
    import com.atguigu.pojo.User;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    import org.junit.jupiter.api.AfterEach;
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Test;

    import java.io.IOException;
    import java.util.List;

    /**
     * projectName: com.atguigu.test
     */
    public class MyBatisTest {

        private SqlSession session;
        // junit会在每一个@Test方法前执行@BeforeEach方法

        @BeforeEach
        public void init() throws IOException {
            session = new SqlSessionFactoryBuilder()
                    .build(
                            Resources.getResourceAsStream("mybatis-config.xml"))
                    .openSession();
        }

        @Test
        public void createTest() {
            User user = new User();
            user.setUsername("admin");
            user.setPassword("123456");
            UserMapper userMapper = session.getMapper(UserMapper.class);
            userMapper.insert(user);
            System.out.println(user);
        }

        @Test
        public void updateTest() {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            User user = userMapper.selectById(1);
            user.setUsername("root");
            user.setPassword("111111");
            userMapper.update(user);
            user = userMapper.selectById(1);
            System.out.println(user);
        }

        @Test
        public void deleteTest() {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            userMapper.delete(1);
            User user = userMapper.selectById(1);
            System.out.println("user = " + user);
        }

        @Test
        public void selectByIdTest() {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            User user = userMapper.selectById(1);
            System.out.println("user = " + user);
        }

        @Test
        public void selectAllTest() {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            List<User> userList = userMapper.selectAll();
            System.out.println("userList = " + userList);
        }

        // junit会在每一个@Test方法后执行@@AfterEach方法
        @AfterEach
        public void clear() {
            session.commit();
            session.close();
        }
    }

    ```

### 2.5 mapperXML标签总结

MyBatis 的真正强大在于它的语句映射，这是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 致力于减少使用成本，让用户能更专注于 SQL 代码。

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

-   `insert` – 映射插入语句。
-   `update` – 映射更新语句。
-   `delete` – 映射删除语句。
-   `select` – 映射查询语句。

**select标签：**

MyBatis 在查询和结果映射做了相当多的改进。一个简单查询的 select 元素是非常简单：

```xml
<select id="selectPerson" 
resultType="hashmap" resultMap="自定义结构"> SELECT * FROM PERSON WHERE ID = #{id} </select>
```

这个语句名为 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：#{id}  \${key}

MyBatis 创建一个预处理语句（PreparedStatement）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：

```java
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

select 元素允许你配置很多属性来配置每条语句的行为细节：

| 属性              | 描述                                                                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------- |
| `id`            | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                       |
| `resultType`    | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。                 |
| `resultMap`     | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。          |
| `timeout`       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                        |
| `statementType` | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |

**insert, update 和 delete标签：**

数据变更语句 insert，update 和 delete 的实现非常接近：

```xml
<insert
  id="insertAuthor"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  statementType="PREPARED"
  timeout="20">
```

| 属性                 | 描述                                                                                                                                             |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`               | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                                                      |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                                                       |
| `statementType`    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。                                |
| `useGeneratedKeys` | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。         |
| `keyProperty`      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `keyColumn`        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。                                       |

## 3 MyBatis多表映射

### 3.1 多表映射概念

1.  **多表查询结果映射思路**

    上面课程中，我全面讲解了单表的mybatis操作！但是`开发中更多的是多表查询需求`，这种情况我们如何让进行处理？

	通常我们是一个表对应一个对象，但是一个对象中可能存在另外一个对象，而另外一个对象涉及到另外一个表，所以我们需要多表查询
		1. 多表查询的sql语句我们编写
		2. 自己设计存储数据的实体类
		3. 即自己定义结果集映射（ResultMap）

    MyBatis 思想是：数据库不可能永远是你所想或所需的那个样子。 我们希望每个数据库都具备良好的第三范式或 BCNF 范式，可惜它们并不都是那样。 如果能有一种数据库映射模式，完美适配所有的应用程序查询需求，那就太好了，而 ResultMap 就是 MyBatis 就是完美答案。

    官方例子：我们如何映射下面这个语句？
    ```xml
    <!-- 非常复杂的语句 -->
    <select id="selectBlogDetails" resultMap="detailedBlogResultMap">
      select
           B.id as blog_id,
           B.title as blog_title,
           B.author_id as blog_author_id,
           A.id as author_id,
           A.username as author_username,
           A.password as author_password,
           A.email as author_email,
           A.bio as author_bio,
           A.favourite_section as author_favourite_section,
           P.id as post_id,
           P.blog_id as post_blog_id,
           P.author_id as post_author_id,
           P.created_on as post_created_on,
           P.section as post_section,
           P.subject as post_subject,
           P.draft as draft,
           P.body as post_body,
           C.id as comment_id,
           C.post_id as comment_post_id,
           C.name as comment_name,
           C.comment as comment_text,
           T.id as tag_id,
           T.name as tag_name
      from Blog B
           left outer join Author A on B.author_id = A.id
           left outer join Post P on B.id = P.blog_id
           left outer join Comment C on P.id = C.post_id
           left outer join Post_Tag PT on PT.post_id = P.id
           left outer join Tag T on PT.tag_id = T.id
      where B.id = #{id}
    </select>
    ```
    
    你可能想把它映射到一个智能的对象模型，这个对象表示了一篇博客，它由某位作者所写，有很多的博文，每篇博文有零或多条的评论和标签。 我们先来看看下面这个完整的例子，它是一个非常复杂的结果映射（假设作者，博客，博文，评论和标签都是类型别名）。 虽然它看起来令人望而生畏，但其实非常简单。
    ```java
    <!-- 非常复杂的结果映射 -->
    <resultMap id="detailedBlogResultMap" type="Blog">
      <constructor>
        <idArg column="blog_id" javaType="int"/>
      </constructor>
      <result property="title" column="blog_title"/>
      <association property="author" javaType="Author">
        <id property="id" column="author_id"/>
        <result property="username" column="author_username"/>
        <result property="password" column="author_password"/>
        <result property="email" column="author_email"/>
        <result property="bio" column="author_bio"/>
        <result property="favouriteSection" column="author_favourite_section"/>
      </association>
      <collection property="posts" ofType="Post">
        <id property="id" column="post_id"/>
        <result property="subject" column="post_subject"/>
        <association property="author" javaType="Author"/>
        <collection property="comments" ofType="Comment">
          <id property="id" column="comment_id"/>
        </collection>
        <collection property="tags" ofType="Tag" >
          <id property="id" column="tag_id"/>
        </collection>
      </collection>
    </resultMap>
    ```
    你现在可能看不懂，接下来我们要学习将多表查询结果使用ResultMap标签映射到实体类对象上！

  **我们的学习目标：**
- `多表查询语句使用`
- `多表结果承接实体类设计`
- `使用ResultMap完成多表结果映射`

2.  **实体类设计方案**
- 多表关系回顾：（双向查看）
    -   一对一
        夫妻关系，人和身份证号
    -   一对多| 多对一
        用户和用户的订单，锁和钥匙
    -   多对多
        老师和学生，部门和员工
  
        实体类设计关系(查询)：（`单向查看`）
    -   对一 ： 夫妻一方对应另一方，订单对应用户都是对一关系

        实体类设计：对一关系下，类中只要包含单个对方对象类型属性即可！

例如：
```java
public class Customer {
  private Integer customerId;
  private String customerName;
}

public class Order {
  private Integer orderId;
  private String orderName;
  private Customer customer;// 体现的是对一的关系
}  
```

-   对多: 用户对应的订单，讲师对应的学生或者学生对应的讲师都是对多关系：
	实体类设计：对多关系下，类中只要包含对方类型集合属性即可！
```java
public class Customer {
private Integer customerId;
private String customerName;
private List<Order> orderList;// 体现的是对多的关系
}

public class Order {
private Integer orderId;
private String orderName;
private Customer customer;// 体现的是对一的关系
}
//查询客户和客户对应的订单集合  不要管!
```
多表结果实体类设计小技巧：
- `对一，属性中包含对方对象`
- `对多，属性中包含对方对象集合`

只有真实发生多表查询时，才需要设计和修改实体类，否则不提前设计和修改实体类！

无论多少张表联查，实体类设计都是两两考虑!

在查询映射的时候，只需要关注本次查询相关的属性！例如：查询订单和对应的客户，就不要关注客户中的订单集合！

3.  **多表映射案例准备**

数据库：
```sql
CREATE TABLE `t_customer` (`customer_id` INT NOT NULL AUTO_INCREMENT, `customer_name` CHAR(100), PRIMARY KEY (`customer_id`) );

CREATE TABLE `t_order` ( `order_id` INT NOT NULL AUTO_INCREMENT, `order_name` CHAR(100), `customer_id` INT, PRIMARY KEY (`order_id`) ); 

INSERT INTO `t_customer` (`customer_name`) VALUES ('c01');

INSERT INTO `t_order` (`order_name`, `customer_id`) VALUES ('o1', '1');
INSERT INTO `t_order` (`order_name`, `customer_id`) VALUES ('o2', '1');
INSERT INTO `t_order` (`order_name`, `customer_id`) VALUES ('o3', '1'); 
```

实际开发时，`一般在开发过程中，不给数据库表设置外键约束`。
原因是避免调试不方便。
一般是功能开发完成，再加外键约束检查是否有bug。

实体类设计：
稍后会进行订单关联客户查询，也会进行客户关联订单查询，所以在这先练习设计
```java
@Data
public class Customer {

private Integer customerId;
private String customerName;
private List<Order> orderList;// 体现的是对多的关系

}  

@Data
public class Order {
private Integer orderId;
private String orderName;
private Customer customer;// 体现的是对一的关系
}  

```

### 3.2 对一映射

1.  需求说明
    根据ID查询订单，以及订单关联的用户的信息！
2.  OrderMapper接口
    ```java
    public interface OrderMapper {
      Order selectOrderWithCustomer(Integer orderId);
    }
    ```
3.  OrderMapper.xml配置文件
    ```xml
    <!-- 创建resultMap实现“对一”关联关系映射 -->
    <!-- id属性：通常设置为这个resultMap所服务的那条SQL语句的id加上“ResultMap” -->
    <!-- type属性：要设置为这个resultMap所服务的那条SQL语句最终要返回的类型 -->
    <resultMap id="selectOrderWithCustomerResultMap" type="order">

      <!-- 先设置Order自身属性和字段的对应关系 -->
      <!-- id:代表主键 -->
      <!-- result:代表其他列 -->
      <id column="order_id" property="orderId"/>

      <result column="order_name" property="orderName"/>

      <!-- 使用association标签配置“对一”关联关系 -->
      <!-- property属性：在Order类中的属性名 -->
      <!-- javaType属性：该属性的类型 -->
      <association property="customer" javaType="customer">

        <!-- 配置Customer类的属性和字段名之间的对应关系 -->
        <id column="customer_id" property="customerId"/>
        <result column="customer_name" property="customerName"/>

      </association>

    </resultMap>

    <!-- Order selectOrderWithCustomer(Integer orderId); -->
    <select id="selectOrderWithCustomer" resultMap="selectOrderWithCustomerResultMap">

      SELECT order_id,order_name,c.customer_id,customer_name
      FROM t_order o
      LEFT JOIN t_customer c
      ON o.customer_id=c.customer_id
      WHERE o.order_id=#{orderId}

    </select>
    ```
    对应关系可以参考下图：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3f6a58891a99a77c0a7d713b193f2685.png)
    
4.  Mybatis全局注册Mapper文件
    ```xml
    <!-- 注册Mapper配置文件：告诉Mybatis我们的Mapper配置文件的位置 -->
    <mappers>

      <!-- 在mapper标签的resource属性中指定Mapper配置文件以“类路径根目录”为基准的相对路径 -->
      <mapper resource="mappers/OrderMapper.xml"/>

    </mappers>
    ```
5.  junit测试程序
    ```java
    @Slf4j
    public class MyBatisTest {

        private SqlSession session;
        // junit会在每一个@Test方法前执行@BeforeEach方法

        @BeforeEach
        public void init() throws IOException {
            session = new SqlSessionFactoryBuilder()
                    .build(
                            Resources.getResourceAsStream("mybatis-config.xml"))
                    .openSession();
        }

        @Test
        public void testRelationshipToOne() {
        
          OrderMapper orderMapper = session.getMapper(OrderMapper.class);
          // 查询Order对象，检查是否同时查询了关联的Customer对象
          Order order = orderMapper.selectOrderWithCustomer(2);
          log.info("order = " + order);
        
        }

        // junit会在每一个@Test方法后执行@@AfterEach方法
        @AfterEach
        public void clear() {
            session.commit();
            session.close();
        }
    }
    ```
6.  关键词

   ` 在“对一”关联关系中，我们的配置比较多，但是关键词就只有：association和javaType`

### 3.3 对多映射

1.  需求说明

    查询客户和客户关联的订单信息！
2.  CustomerMapper接口
    ```java
public interface CustomerMapper {
  Customer selectCustomerWithOrderList(Integer customerId);
}
    ```
3.  CustomerMapper.xml文件
    ```xml
    <!-- 配置resultMap实现从Customer到OrderList的“对多”关联关系 -->
    <resultMap id="selectCustomerWithOrderListResultMap"

      type="customer">

      <!-- 映射Customer本身的属性 -->
      <id column="customer_id" property="customerId"/>

      <result column="customer_name" property="customerName"/>

      <!-- collection标签：映射“对多”的关联关系 -->
      <!-- property属性：在Customer类中，关联“多”的一端的属性名 -->
      <!-- ofType属性：集合属性中元素的类型 -->
      <collection property="orderList" ofType="order">

        <!-- 映射Order的属性 -->
        <id column="order_id" property="orderId"/>

        <result column="order_name" property="orderName"/>

      </collection>

    </resultMap>

    <!-- Customer selectCustomerWithOrderList(Integer customerId); -->
    <select id="selectCustomerWithOrderList" resultMap="selectCustomerWithOrderListResultMap">
      SELECT c.customer_id,c.customer_name,o.order_id,o.order_name
      FROM t_customer c
      LEFT JOIN t_order o
      ON c.customer_id=o.customer_id
      WHERE c.customer_id=#{customerId}
    </select>
    ```
    对应关系可以参考下图：

    ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a017af100c6b63a01fafb5c681a25f7.png)


4.  Mybatis全局注册Mapper文件
    ```xml
    <!-- 注册Mapper配置文件：告诉Mybatis我们的Mapper配置文件的位置 -->
    <mappers>
      <!-- 在mapper标签的resource属性中指定Mapper配置文件以“类路径根目录”为基准的相对路径 -->
      <mapper resource="mappers/OrderMapper.xml"/>
      <mapper resource="mappers/CustomerMapper.xml"/>
    </mappers>
    ```
5.  junit测试程序
    ```java
    @Test
    public void testRelationshipToMulti() {

      CustomerMapper customerMapper = session.getMapper(CustomerMapper.class);
      // 查询Customer对象同时将关联的Order集合查询出来
      Customer customer = customerMapper.selectCustomerWithOrderList(1);
      log.info("customer.getCustomerId() = " + customer.getCustomerId());
      log.info("customer.getCustomerName() = " + customer.getCustomerName());
      List<Order> orderList = customer.getOrderList();
      for (Order order : orderList) {
        log.info("order = " + order);
      }
    }
    ```
6.  关键词

    `在“对多”关联关系中，同样有很多配置，但是提炼出来最关键的就是：“collection”和“ofType”`

### 3.4 多表映射总结

#### 3.4.1 多表映射优化

| setting属性           | 属性含义                                                                                              | 可选值                 | 默认值     |
| ------------------- | ------------------------------------------------------------------------------------------------- | ------------------- | ------- |
| autoMappingBehavior | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL | PARTIAL |

我们可以`将autoMappingBehavior设置为full`,进`行多表resultMap映射的时候`，可以省略符合列和属性命名映射规则（列名=属性名，或者开启驼峰映射也可以自定映射）的result标签！

修改mybati-sconfig.xml:
```xml
<!--开启resultMap自动映射 -->
<setting name="autoMappingBehavior" value="FULL"/>
```

修改teacherMapper.xml
```xml
<resultMap id="teacherMap" type="teacher">
    <id property="tId" column="t_id" />
    <!-- 开启自动映射,并且开启驼峰式支持!可以省略 result!-->
<!--        <result property="tName" column="t_name" />-->
    <collection property="students" ofType="student" >
        <id property="sId" column="s_id" />
<!--            <result property="sName" column="s_name" />-->
    </collection>
</resultMap>
```

#### 3.4.2 多表映射总结

| 关联关系 | 配置项关键词                              | 所在配置文件和具体位置              |
| ---- | ----------------------------------- | ------------------------ |
| 对一   | association标签/javaType属性/property属性 | Mapper配置文件中的resultMap标签内 |
| 对多   | collection标签/ofType属性/property属性    | Mapper配置文件中的resultMap标签内 |

## 4 MyBatis动态语句

### 4.1 动态语句需求和简介

经常遇到很多按照很多查询条件进行查询的情况，比如智联招聘的职位搜索等。其中经常出现很多条件不取值的情况，在后台应该如何完成最终的SQL语句呢？

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dc8503037d06fe2ec81b42ea37112435.png)



`动态 SQL 是 MyBatis 的强大特性之一`。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 `OGNL 的表达式`，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。
### 4.2 if和where标签

使用动态 SQL 最常见情景是根据条件包含 where  / if 子句的一部分。比如：

```xml
<!-- List<Employee> selectEmployeeByCondition(Employee employee); -->
<select id="selectEmployeeByCondition" resultType="employee">
    select emp_id,emp_name,emp_salary from t_emp
    <!-- where标签会自动去掉“标签体内前面多余的and/or” -->
    <where>
        <!-- 使用if标签，让我们可以有选择的加入SQL语句的片段。这个SQL语句片段是否要加入整个SQL语句，就看if标签判断的结果是否为true -->
        <!-- 在if标签的test属性中，可以访问实体类的属性，不可以访问数据库表的字段 -->
        <if test="empName != null">
            <!-- 在if标签内部，需要访问接口的参数时还是正常写#{} -->
            or emp_name=#{empName}
        </if>
        <if test="empSalary &gt; 2000">
            or emp_salary>#{empSalary}
        </if>
        <!--
         第一种情况：所有条件都满足 WHERE emp_name=? or emp_salary>?
         第二种情况：部分条件满足 WHERE emp_salary>?
         第三种情况：所有条件都不满足 没有where子句
         -->
    </where>
</select>
```

### 4.3 set标签

```xml
<!-- void updateEmployeeDynamic(Employee employee) -->
<update id="updateEmployeeDynamic">
    update t_emp
    <!-- set emp_name=#{empName},emp_salary=#{empSalary} -->
    <!-- 使用set标签动态管理set子句，并且动态去掉两端多余的逗号 -->
    <set>
        <if test="empName != null">
            emp_name=#{empName},
        </if>
        <if test="empSalary &lt; 3000">
            emp_salary=#{empSalary},
        </if>
    </set>
    where emp_id=#{empId}
    <!--
         第一种情况：所有条件都满足 SET emp_name=?, emp_salary=?
         第二种情况：部分条件满足 SET emp_salary=?
         第三种情况：所有条件都不满足 update t_emp where emp_id=?
            没有set子句的update语句会导致SQL语法错误
     -->
</update>
```

### 4.4 trim标签(了解)

使用trim标签控制条件部分两端是否包含某些字符

-   prefix属性：指定要动态添加的前缀
-   suffix属性：指定要动态添加的后缀
-   prefixOverrides属性：指定要动态去掉的前缀，使用“|”分隔有可能的多个值
-   suffixOverrides属性：指定要动态去掉的后缀，使用“|”分隔有可能的多个值

```xml
<!-- List<Employee> selectEmployeeByConditionByTrim(Employee employee) -->
<select id="selectEmployeeByConditionByTrim" resultType="com.atguigu.mybatis.entity.Employee">
    select emp_id,emp_name,emp_age,emp_salary,emp_gender
    from t_emp
    
    <!-- prefix属性指定要动态添加的前缀 -->
    <!-- suffix属性指定要动态添加的后缀 -->
    <!-- prefixOverrides属性指定要动态去掉的前缀，使用“|”分隔有可能的多个值 -->
    <!-- suffixOverrides属性指定要动态去掉的后缀，使用“|”分隔有可能的多个值 -->
    <!-- 当前例子用where标签实现更简洁，但是trim标签更灵活，可以用在任何有需要的地方 -->
    <trim prefix="where" suffixOverrides="and|or">
        <if test="empName != null">
            emp_name=#{empName} and
        </if>
        <if test="empSalary &gt; 3000">
            emp_salary>#{empSalary} and
        </if>
        <if test="empAge &lt;= 20">
            emp_age=#{empAge} or
        </if>
        <if test="empGender=='male'">
            emp_gender=#{empGender}
        </if>
    </trim>
</select>
```

### 4.5 choose/when/otherwise标签

在多个分支条件中，仅执行一个。

-   从上到下依次执行条件判断
-   遇到的第一个满足条件的分支会被采纳
-   被采纳分支后面的分支都将不被考虑
-   如果所有的when分支都不满足，那么就执行otherwise分支

```xml
<!-- List<Employee> selectEmployeeByConditionByChoose(Employee employee) -->
<select id="selectEmployeeByConditionByChoose" resultType="com.atguigu.mybatis.entity.Employee">
    select emp_id,emp_name,emp_salary from t_emp
    where
    <choose>
        <when test="empName != null">emp_name=#{empName}</when>
        <when test="empSalary &lt; 3000">emp_salary &lt; 3000</when>
        <otherwise>1=1</otherwise>
    </choose>
    
    <!--
     第一种情况：第一个when满足条件 where emp_name=?
     第二种情况：第二个when满足条件 where emp_salary < 3000
     第三种情况：两个when都不满足 where 1=1 执行了otherwise
     -->
</select>
```

### 4.6 foreach标签

**基本用法**

用批量插入举例![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0d22ae393984f417780aacf84d316909.png)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b2aeecfa4197300b4c9cd5600cfe8c97.png)


```xml
<!--
    collection属性：要遍历的集合
    item属性：遍历集合的过程中能得到每一个具体对象，在item属性中设置一个名字，将来通过这个名字引用遍历出来的对象
    separator属性：指定当foreach标签的标签体重复拼接字符串时，各个标签体字符串之间的分隔符
    open属性：指定整个循环把字符串拼好后，字符串整体的前面要添加的字符串
    close属性：指定整个循环把字符串拼好后，字符串整体的后面要添加的字符串
    index属性：这里起一个名字，便于后面引用
        遍历List集合，这里能够得到List集合的索引值
        遍历Map集合，这里能够得到Map集合的key
 -->
<foreach collection="empList" item="emp" separator="," open="values" index="myIndex">
    <!-- 在foreach标签内部如果需要引用遍历得到的具体的一个对象，需要使用item属性声明的名称 -->
    (#{emp.empName},#{myIndex},#{emp.empSalary},#{emp.empGender})
</foreach>
```

**批量更新时需要注意**

上面批量插入的例子本质上是一条SQL语句，而实现批量更新则需要多条SQL语句拼起来，用分号分开。也就是一次性发送多条SQL语句让数据库执行。此`时需要在数据库连接信息的URL地址中设置`：

```.properties
atguigu.dev.url=jdbc:mysql:///mybatis-example?allowMultiQueries=true
```

对应的foreach标签如下：

```xml
<!-- int updateEmployeeBatch(@Param("empList") List<Employee> empList) -->
<update id="updateEmployeeBatch">
    <foreach collection="empList" item="emp" separator=";">
        update t_emp set emp_name=#{emp.empName} where emp_id=#{emp.empId}
    </foreach>
</update>
```

**关于foreach标签的collection属性**

如果没有给接口中List类型的参数使用@Param注解指定一个具体的名字，那么在collection属性中默认可以使用collection或list来引用这个list集合。这一点可以通过异常信息看出来：

```xml
Parameter 'empList' not found. Available parameters are [arg0, collection, list]
```

在实际开发中，为了避免隐晦的表达造成一定的误会，`建议使用@Param注解明确声明变量的名称，然后在foreach标签的collection属性中按照@Param注解指定的名称来引用传入的参数`

### 4.7 sql片段

**抽取重复的SQL片段**

```xml
<!-- 使用sql标签抽取重复出现的SQL片段 -->
<sql id="mySelectSql">
    select emp_id,emp_name,emp_age,emp_salary,emp_gender from t_emp
</sql>
```

引用已抽取的SQL片段

```xml
<!-- 使用include标签引用声明的SQL片段 -->
<include refid="mySelectSql"/>
```

## 5 MyBatis高级扩展

### 5.1 Mapper批量映射优化

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

        ![image-20230922214828609.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e58a0ebbe49efb0549651d94ec5936eb.png)
        ![image-20230922214839215.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/99a3d30c6ef5243b0305655238f3a319.png)



### 5.2 插件和分页插件PageHelper

#### 5.2.1 插件机制和PageHelper插件介绍

MyBatis 对插件进行了标准化的设计，并提供了一套可扩展的插件机制。插件可以在用于语句执行过程中进行拦截，并允许通过自定义处理程序来拦截和修改 SQL 语句、映射语句的结果等。

具体来说，MyBatis 的插件机制包括以下三个组件：

1.  `Interceptor`（拦截器）：定义一个拦截方法 `intercept`，该方法在执行 SQL 语句、执行查询、查询结果的映射时会被调用。
2.  `Invocation`（调用）：实际上是对被拦截的方法的封装，封装了 `Object target`、`Method method` 和 `Object[] args` 这三个字段。
3.  `InterceptorChain`（拦截器链）：对所有的拦截器进行管理，包括将所有的 Interceptor 链接成一条链，并在执行 SQL 语句时按顺序调用。

插件的开发非常简单，只需要实现 Interceptor 接口，并使用注解 `@Intercepts` 来标注需要拦截的对象和方法，然后在 MyBatis 的配置文件中添加插件即可。

`PageHelper 是 MyBatis 中比较著名的分页插件`，它提供了多种分页方式（例如 MySQL 和 Oracle 分页方式），支持多种数据库，并且使用非常简单。下面就介绍一下 PageHelper 的使用方式。

<https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md#如何配置数据库方言>

#### 5.2.2 PageHelper插件使用

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

### 5.3 逆向工程和MybatisX插件

#### 5.3.1 ORM思维介绍

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

#### 5.3.2 逆向工程

`MyBatis 的逆向工程是一种自动化生成持久层代码和映射文件的工具`，它可以根据数据库表结构和设置的参数生成对应的实体类、Mapper.xml 文件、Mapper 接口等代码文件，简化了开发者手动生成的过程。`逆向工程使开发者可以快速地构建起 DAO 层`，并快速上手进行业务开发。

MyBatis 的逆向工程有两种方式：通过 `MyBatis Generator 插件`实现和通过 Maven 插件实现。无论是哪种方式，逆向工程一般需要指定一些配置参数，例如数据库连接 URL、用户名、密码、要生成的表名、生成的文件路径等等。

总的来说，MyBatis 的逆向工程为程序员提供了一种方便快捷的方式，能够快速地生成持久层代码和映射文件，是半自动 ORM 思维像全自动发展的过程，提高程序员的开发效率。

**注意：逆向工程只能生成单表crud的操作，多表查询依然需要我们自己编写！**

#### 5.3.3 逆向工程插件MyBatisX使用

MyBatisX 是一个 MyBatis 的代码生成插件，可以通过简单的配置和操作快速生成 MyBatis Mapper、pojo 类和 Mapper.xml 文件。下面是使用 MyBatisX 插件实现逆向工程的步骤：

1.  安装插件：

    在 IntelliJ IDEA 中打开插件市场，搜索 MyBatisX 并安装。
2.  使用 IntelliJ IDEA连接数据库
    -   连接数据库![image-20230922214905276.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fddd7d66c7c5586aa9ff4b81a9f45452.png)



    -   填写信息![image-20230922214912923.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5675681efa73e586fab2c311bdcbfd84.png)

    -   展示库表![image-20230922214917012.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8ea0be52476aabc12312b2078d30b8c.png)



    -   逆向工程使用![image-20230922214921795.png|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f34ee2a18a610b8198c9105eca6dcc1.png)
![image-20230922214928981.png|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04a8d46e1e3d294f3b3f3db3cc2978ec.png)


3.  查看生成结果![image-20230922214955291.png|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b87fd3db111fc94fa6a0b2972a5b8c61.png)

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


## 6 MyBatis缓存

### 6.1 简介

- **什么是缓存【Cache】：**
    - 存在内存中的临时数据！
    - 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用了从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题
  
- **为什么使用缓存？**
    - 减少和数据库的交互次数，较少系统开销，提高系统效率

- **什么样的数据能使用缓存？**
    - 经常查询而且不经常改变的数据
### 6.2 Mybatis缓存

- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。
    
- MyBatis系统中默认定义了两级缓存：**一级缓存**和**二级缓存**
    - 默认情况下，只有一级缓存开启。(SqlSession级别的缓存，也称为本地缓存)
    - 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
    - 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

### 6.3 一级缓存

- 一级缓存也叫本地缓存：
- 与数据库同一次会话期间查询到的数据会放在本地缓存中。
- 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库；

**缓存失效的情况：**
1. 查询不同的东西
2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存！
3. 查询不同的Mapper.xml
4. 手动清理缓存！

==**`一级缓存默认是开启的，只在一次SqlSession中有效，也就是拿到连接到关闭连接这个区间(相当于一个用户不断查询相同的数据，比如不断刷新)`**==

一级缓存就是一个map！
### 6.4 二级缓存

- 二级缓存也叫全局缓存，==一级缓存作用域太低了，所以诞生了二级缓存==
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存：
- 工作机制
    - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
    - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中
    - 新的会话查询信息，就可以从二级缓存中获取内容
    - 不同的mapper查出的数据会放在自己对应的缓存(map)中

开启全局缓存
```xml
<!--        开启全局缓存-->
<setting name="cacheEnabled" value="true"/>
```

mapper文件添加二级缓存
```xml
<!--    在当前Mapper.xml中使用二级缓存-->
<cache  eviction="FIFO"
	flushInterval="60000"
	size="512"
	readOnly="true"/>
<!--    没有参数也可以用-->
<cache/>
```

小结：
- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会先放在一级缓存中
- 只有当会话提交，或者关闭的时候才会提交到二级缓存中
### 6.5 缓存原理

缓存顺序：
1. 先看二级缓存中有没有
2. 再看一级缓存中有没有
3. 查询数据库

**注：一二级缓存都没有，查询数据库，查询后将数据放入一级缓存**

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e0091e47809f0bb8226dbd086ba27972.png)

### 6.6 自定义缓存—ehcache

有需要，自行了解

后面我们主要使用Redis做旁路缓存
## 7 MyBatis总结

| 核心点         | 掌握目标                                  |
| ----------- | ------------------------------------- |
| mybatis基础   | 使用流程, 参数输入,#{} \${},参数输出              |
| mybatis多表   | 实体类设计,resultMap多表结果映射                 |
| `mybatis动态语句` | `Mybatis动态语句概念, where , if , foreach标签` |
| mybatis扩展   | Mapper批量处理,分页插件,逆向工程                  |
