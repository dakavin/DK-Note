
到目前为止，已经将登录的静态页面完成了。接下来，我们将转入后端开发，将登录相关的接口搞定。因为登录功能涉及到从数据库中查询用户名、密码，所以，这小节中，我们需要先将持久层框架 Mybatis Plus 整合到项目中来。

## 1 什么是 MyBatis Plus？

**MyBatis Plus （简称 MP） 是一款持久层框架**，说白话就是一款操作数据库的框架。它是**一个 MyBatis 的增强工具**，就像 iPhone手机一般都有个 plus 版本一样，它在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

MyBatis Plus 的愿景是成为 MyBatis 最好的搭档，就像魂斗罗中的 1P、2P，基友搭配，效率翻倍。

## 2 MyBatis Plus 的优势

- **快速开发**：MyBatis Plus 提供了一系列的便捷功能，如自动生成 SQL 语句、通用 Mapper 等，使数据库操作更加高效，能够节省开发时间。
- **更少的配置**： Spring Boot 已经为我们提供了很多默认的配置，整合 MyBatis Plus 时只需少量的配置，减少了繁琐的配置步骤。
- **内置分页插件**：MyBatis Plus 内置了分页插件，无需额外的代码，就可以轻松实现分页查询。
- **更好的支持**： MyBatis Plus 在社区中有较广泛的使用，拥有活跃的维护者和开发者，您可以轻松找到解决方案和文档。

## 3 新建测试库与表

### 3.1 建库

在数据库上右键，新建一个名为 `dkblog` 的数据库：

注意，字符集设定为 `utfmb4`, 让数据库支持 `emojis` 表情符号：

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a84cbc2536079ba1e62cc4972cdcc7b2.png)
### 3.2 建表

执行如下建表语句：
```mysql
CREATE TABLE `t_user` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `username` varchar(60) NOT NULL COMMENT '用户名',
  `password` varchar(60) NOT NULL COMMENT '密码',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '最后一次更新时间',
  `is_deleted` tinyint(2) NOT NULL DEFAULT '0' COMMENT '逻辑删除：0：未删除 1：已删除',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uk_username` (`username`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

## 4 开始整合

数据库、表都准备好后，我们开始在项目中整合持久层框架 Mybatis Plus。

### 4.1 添加依赖

在父项目 `weblog-springboot` 的 `pom.xml` 文件中，声明 MP 的依赖版本号：

```xml
<!-- 版本号统一管理 -->
<properties>
		// 省略...
		<mybatis-plus.version>3.5.1</mybatis-plus.version>
</properties>

		<!-- 统一依赖管理 -->
<dependencyManagement>
		<dependencies>
				// 省略...

				<!-- Mybatis Plus -->
				<dependency>
						<groupId>com.baomidou</groupId>
						<artifactId>mybatis-plus-boot-starter</artifactId>
						<version>${mybatis-plus.version}</version>
				</dependency>

		</dependencies>
</dependencyManagement>
```

然后，在 `weblog-module-common` 模块的 `pom.xml` 文件中，引入 MP 和 MySQL 依赖：

```xml
<!-- Mybatis Plus -->
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>

<!-- mysql 依赖 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 4.2 配置文件

编辑 `applicaiton-dev.yml` 文件，添加数据库链接相关的配置，包含连接池的配置：

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/weblog?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&zeroDateTimeBehavior=convertToNull
    username: root
    password: 123456
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      auto-commit: true
      idle-timeout: 30000
      pool-name: Weblog-HikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
```

**解释一下各个配置的含义和作用：**
- `spring.datasource.driver-class-name`: 指定数据库驱动类的完整类名，这里使用的是 MySQL 数据库的驱动类。
- `spring.datasource.url`: 数据库连接的URL，指向本地的MySQL数据库，端口是3306，数据库名是`weblog`，同时配置了一系列参数，如使用Unicode编码、字符编码为UTF-8、自动重连、不使用SSL、对零时间进行转换等。
- `spring.datasource.username`: 数据库用户名，这里使用的是`root`。
- `spring.datasource.password`: 数据库密码，这里使用的是`123456`。

数据库链接池这块，我们**使用的 Spring Boot 默认的 HikariCP**，它是一个高性能的连接池实现 , 同时，它号称是==速度最快的连接池==：
- `spring.datasource.hikari.minimum-idle`: Hikari连接池中最小空闲连接数。
- `spring.datasource.hikari.maximum-pool-size`: Hikari连接池中允许的最大连接数。
- `spring.datasource.hikari.auto-commit`: 连接是否自动提交事务。
- `spring.datasource.hikari.idle-timeout`: 连接在连接池中闲置的最长时间，超过这个时间会被释放。
- `spring.datasource.hikari.pool-name`: 连接池的名字。
- `spring.datasource.hikari.max-lifetime`: 连接在连接池中的最大存活时间，超过这个时间会被强制关闭。
- `spring.datasource.hikari.connection-timeout`: 获取连接的超时时间。
- `spring.datasource.hikari.connection-test-query`: 用于测试连接是否可用的SQL查询，这里使用的是`SELECT 1`，表示执行这个查询来测试连接。

然后，在 `weblog-module-common` 模块中的 `config` 包下，新建一个 `MybatisPlusConfig` 配置文件，代码如下：
```java
@Configuration
@MapperScan("com.quanxiaoha.weblog.common.domain.mapper")
	public class MybatisPlusConfig {
}
```

- `@Configuration` : 此注解声明该类为配置类；
- `@MapperScan` : 扫描 MP 的 `mapper` 接口存放位置。PS: 数据库相关的代码，我们统一放置在 `/domain` 这个包中，格式如下：
    
    ![](https://img.quanxiaoha.com/quanxiaoha/169269468935130)
    
    - `dos` : 根据阿里的开发规范，统一将数据库对应的实体类命名为 `xxxDO` 这种形式，统一存放此包下。
    - `mapper` : 统一放置 `mapper` 接口文件；

## 5 实体类

在 `/dos` 包中，新建一个 `UserDo` 类，字段和数据库中的字段通过转驼峰的形式对应一一对应起来，MP 框架会默认通过这种规则将字段光联在一起，内容如下：

```java
@TableName(value ="t_user")  
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Builder  
public class UserDo implements Serializable {  
    /**  
     * id     
     * */    
    @TableId(type = IdType.AUTO)  
    private Long id;  
  
    /**  
     * 用户名  
     */  
    private String username;  
  
    /**  
     * 密码  
     */  
    private String password;  
  
    /**  
     * 创建时间  
     */  
    private Date createTime;  
  
    /**  
     * 最后一次更新时间  
     */  
    private Date updateTime;  
  
    /**  
     * 逻辑删除：0：未删除 1：已删除  
     * 注意类型是Boolean的！！！
     */  
    @TableLogic  
    private Boolean isDeleted;  
  
    @TableField(exist = false)  
    private static final long serialVersionUID = 1L;  
}
```

## 6 新建 Mapper 接口

在 `mapper`包中，创建一个 `UserMapper` 接口，代码如下：
```java
public interface UserMapper extends BaseMapper<UserDO> {
}
```

至此，用于操作数据的前置代码都搞定了。

## 7 新增一条用户记录

接下来，我们通过单元测试，往数据库中添加一个测试记录，看看能否新增成功。在 `weblog-web` 模块中的单元测试类中，新增一个测试方法，代码如下：

```java
@Autowired  
private UserMapper userMapper;  
@Test  
public void insertTest(){  
    UserDO userDO = UserDO.builder()  
            .username("zhangsan")  
            .password("123456")  
            .createTime(new Date())  
            .updateTime(new Date())  
            .isDeleted(false)  
            .build();  
    userMapper.insert(userDO);  
}
```

运行该测试方法，看看表中是否被成功插入一条数据：

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/000c00fe62501cc2d628d78b33659084.png)

可以看到，数据插入成功了。

## 8 结语

至此，本小节整合 Mabatis Plus 持久层框架就成功了，有了持久层框架，能够帮助开发人员更方便地进行数据库操作。同时，MyBatis Plus 的强大功能和 Spring Boot 的自动化配置使得整合变得非常简单，节省了开发人员的时间和精力。