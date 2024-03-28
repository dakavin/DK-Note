## 目录

- [1 JDBC事务](#1%20JDBC%E4%BA%8B%E5%8A%A1)
- [2 JDBC工具类](#2%20JDBC%E5%B7%A5%E5%85%B7%E7%B1%BB)
- [3 JDBC连接池](#3%20JDBC%E8%BF%9E%E6%8E%A5%E6%B1%A0)
	- [3.1 连接池概念：](#3.1%20%E8%BF%9E%E6%8E%A5%E6%B1%A0%E6%A6%82%E5%BF%B5%EF%BC%9A)

## 1 JDBC事务

- 案例：转账

小明账户余额  1000

小芳账户余额     1000

小明给小芳转500元，正常情况下需要执行sql1和sql2

sql1 :  update 表 set 小明的余额=500 where name='小明'

`执行完sql1还没来得及执行sql2，发生错误或服务器宕机，`

sql2  : update 表 set 小芳的余额=1500 where name='小芳'

此时的结果就变成了

小明账户余额  500

小芳账户余额     1000

`少了500去哪里了？`

`为了解决这个问题，就要用到事务，将sql1和sql2放在同一个事务中，要么全部成功，要么全部失败`

在我们的java项目中该怎么操作呢？

`那就用到了jdbc来操作事务
`
**1.**   Jdbc事务的概念：

使用jdbc来维持一批sql语句的事务特性，本质上还是依赖于`mysql的事务`

**2.**   操作步骤：

1）       关闭事务自动提交：

connection. setAutoCommit(boolean autoCommit) ：调用该方法设置参数为false

2）       执行sql

3）       提交事务：commit()

当所有sql都执行完提交事务

4）       回滚事务：rollback()

中途发生异常，在catch中回滚事务

## 2 JDBC工具类

**1.**   工具类概念：

供其它业务核心类调用的具有某个基础功能的普通类，比如创建随机id、校验日期和管理数据库连接工具类等，本质上是从多个业务类代码中抽离了公共的部分。

**2.**   定义JDBC工具类步骤：
                  1）. 定义一个类 JDBCUtils
                  2）. 提供静态代码块加载配置文件，初始化连接池对象
                  3）. 提供方法
                       a. 获取连接方法：
                       b. 释放资源

示例：

```java
public class JDBCUtils {
    private static final String url = "jdbc:mysql://localhost:3306/lesson10";
    private static final String user = "root";
    private static final String password = "123456";
    static {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    //获取连接
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, user, password);
    }
    //释放资源
    public static void close(ResultSet rs, PreparedStatement pstmt, Connection conn) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();//归还连接
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 3 JDBC连接池

### 3.1 连接池概念：

- 其实就是一个`容器(集合)`，存放数据库连接(connection)的容器。`当系统初始化好后，容器被创建，容器中会申请一些连接对象，当用户来访问数据库时，从容器中获取连接对象并占用该connection对象，用户访问完之后，会释放该`connection对象（置为空闲状态）。

     ![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

例：程序初始化了5个connection对象，我拿走一个用，那么池子还剩4个空闲对象，等我用完释放，那么池子connection空闲对象数量又变为5个。

我们即可以给连接池设置初始connection数量，也可以设置最大connection数量，这样当池子里的空闲connection不够用了，就会继续创建直到达到最大connection数量。

- `优点`：
1. 资源重用: 避免了频繁创建，释放连接引起的大量性能开销
2. 反应速度更快：初始化过程中，往往已经创建了若干数据库连接备用
3. 统一的连接管理：限制应用最大连接数，超时设置

- 实例：

Druid连接池—目前最热门的连接池，由阿里巴巴开发

步骤：

1）. `导入jar包` druid-1.0.9.jar和commons-logging-1.2.jar

mysql-connector-java-8.0.20.jar

2）. `定义配置文件`：

druid.properties，放在src目录下，内容如下：

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

3）. `加载配置文件`，将属性读入Properties集合。

InputStream inputStream= DruidDemo1.class.getClassLoader().getResour

ceAsStream("druid.properties");

properties.load(inputStream);

4）. `获取数据库连接池对象(又称为数据源)`：

DataSource dataSe = DruidDataSourceFactory.createDataSource(properties);

5）. `获取连接`：getConnection

Connection conn = dataSe.getConnection();