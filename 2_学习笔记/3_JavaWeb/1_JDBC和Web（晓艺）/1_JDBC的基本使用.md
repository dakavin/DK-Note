## 目录

- [1 JDBC介绍](#1%20JDBC%E4%BB%8B%E7%BB%8D)
- [2 实战案例](#2%20%E5%AE%9E%E6%88%98%E6%A1%88%E4%BE%8B)
	- [2.1 开发步骤](#2.1%20%E5%BC%80%E5%8F%91%E6%AD%A5%E9%AA%A4)
	- [2.2 重要对象](#2.2%20%E9%87%8D%E8%A6%81%E5%AF%B9%E8%B1%A1)

## 1 JDBC介绍

- 概念： Java DataBase Connectivity （Java数据库连接）

- 功能：
	- 连接数据库
	- 发送并执行SQL语句
	- 获得处理结果

- 本质：
	- 其实是Java官方定义的一套操作所有关系型数据库的接口。各个数据库厂商区实现这套接口，提供数据库驱动jar包（实现类）
	- 我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922073757245.png)



## 2 实战案例

### 2.1 开发步骤

先将数据库驱动jar包加到项目依赖中，然后执行后续步骤

1. 注册驱动：主要告诉JVM，我们的程序将要使用哪一种数据库
```java
Class.forName("com.mysql.cj.jdbc.Driver");
//注意的是，加载驱动时会默认完成驱动的注册工作
```

2. 获取连接：使用JDBC中的类，获得数据库的连接对象
```java
String url = "jdbc:mysql://localhost:3306/test";
String user = "root";
String password = "abc123";
Connection con = DriverManager.getConnection(url,user,password);
```

3. 通过Connection的对象，获取Statement或PreparedStatement接口的实现类
```java
Statement statement = con.createStatement();

PreparedStatement ps = con.preparedStatement(sql);

String sql = "SELECT * FROM product";
```

4. 执行SQL语句：使用执行者对象，向数据库中执行SQL语句（增删改查）
```java
ResultSet rs = statement.executeQuery(sql);
ResultSet rs = ps.executeQuery();
```

5. 处理结果
```java
while (resultSet.next()) {

        int idStr = resultSet.getInt("id");

        String nameStr = resultSet.getString("name");

        double priceStr = resultSet.getDouble("price");

        String markStr = resultSet.getString("mark");

        System.out.println(idStr + "--" + nameStr + "--" + markStr);

       }
```

6. 释放对象：对象占用内存资源和cpu资源，关闭顺序为先打开的最后关闭（玩套娃一样）

```java
resultSet.close();

statement.close();

connection.close();
```

### 2.2 重要对象

- `DriverManager：驱动管理对象`
	- 功能：获取数据库连接：
	- 方法：static Connection getConnection(String url, String user, String password)
	- 参数：
		- url：指定连接的路径
		- 语法：jdbc:mysql://ip地址(域名):端口号/数据库名称
		- 例子：jdbc:mysql://localhost:3306/db3
		- user：用户名
		- password：密码

- `Connection：数据库连接对象`

- 功能：创建执行器
Statement createStatement()
PreparedStatement prepareStatement(String sql) 

-  `Statement： sql执行器`
	- 功能：
		1.  int executeUpdate(String sql) ：执行DML（insert、update、delete）语句、DDL(create，alter、drop)语句，返回值为影响的行数
		2.  ResultSet executeQuery(String sql)  ：执行DQL（select)句
		3.  boolean execute(String sql) ：可以执行任意的sql，不建议使用，执行select语句会返回true，其它会返回false

-  `PreparedStatement：预编译的sql的对象`
- 相比Statement，PreparedStatement的优点：
	- a、性能更好，PreparedStatement中参数使用?作为占位符，编译后会被缓存下来，下次调用相同的预编译语句时不需要重新编译，只需传入对应参数就行，参数使用?作为占位符。
	
	- b、可以解决SQL注入问题：preparedStatement不是将参数简单拼凑成sql，而是做了一些预处理，将参数转换为string，两端加单引号，将参数内的一些特殊字符（换行，单双引号，斜杠等）做转义处理。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922073806432.png)



- `ResultSet：结果集对象，封装查询结果`

- boolean next(): ResultSet里的游标向下移动一行（从第0行开始移动），并判断当前行是是否有数据，如果有数据则返回true，如果没数据则返回false

- getXxx(参数):获取数据

- Xxx：代表数据类型   如： int getInt() ,   String getString()

- 参数：

1. int：代表列的编号,从1开始   如： getInt(1)

2. String：代表列名称。 如： getString("balance")

ResultSet使用步骤：

1. 游标从第0行向下移动一行,并判断是否有数据

2．获取数据

3. 继续下一循环
```java
   while(rs.next()){
//获取数据
}
```