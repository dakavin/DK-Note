---
文章标题: "[[3_使用PreparedStatement实现CRUD操作]]" 
文章作者: Dakkk
文章概要: |
  详细介绍PreparedStatement实现数据库CRUD操作，对比Statement弊端，展示预编译SQL语句的优势和防SQL注入机制，包含完整代码示例。
tags:
- "PreparedStatement"
- "JDBC"
- "SQL注入"
- "数据库操作"
- "CRUD"
- "ResultSet"
- "预编译语句"
- "Java数据库编程"
相关文章:
- "[[5_批量插入]]"
- "[[9_Apache-DBUtils实现CRUD操作]]"
- "[[0_数据库操作汇总]]"
- "[[10_文章分类：删除功能开发]]"
- "[[2_📕文章分类：新增接口开发]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/1_JDBC（尚硅谷）/3_使用PreparedStatement实现CRUD操作.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:00:31
---


- CRUD是指 **Create（创建）、Retrieve（读取）、Update（更新）和Delete（删除）** 这四个操作，是指在数据库或系统中对数据进行基本操作的常用术语

## 1 操作和访问数据库

- 数据库连接被用于向数据库服务器发送命令和 SQL 语句，并接受数据库服务器返回的结果。`其实一个数据库连接就是一个Socket连接`。

- 在 java.sql 包中`有 3 个接口分别定义了对数据库的调用`的不同方式：
	 -` Statement`：用于`执行静态 SQL 语句`并返回它所生成结果的对象。 
	- `PrepatedStatement`：SQL 语句被`预编译并存储在此对象`中，可以使用此对象`多次高效地执行该语句`。 
	- `CallableStatement`：用于执行 SQL 存储过程


## 2 使用Statement操作数据表的弊端

- 通过调用 Connection 对象的 createStatement() 方法创建该对象。该对象用于执行静态的 SQL 语句，并且返回执行结果。

- Statement 接口中定义了下列方法用于执行 SQL 语句：

  ```mysql
  int excuteUpdate(String sql)：执行更新操作INSERT、UPDATE、DELETE
  ResultSet executeQuery(String sql)：执行查询操作SELECT
  ```

- 但是使用Statement操作数据表存在弊端：

  - **问题一：存在拼串操作，繁琐**
  - **问题二：存在SQL注入问题**

- SQL 注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的 SQL 语句段或命令(如：SELECT user, password FROM user_table WHERE user='a' OR 1 = ' AND password = ' OR '1' = '1') ，从而利用系统的 SQL 引擎完成恶意行为的做法。

- 对于 Java 而言，要防范 SQL 注入，只要用`PreparedStatement(从Statement扩展而来) 取代 Statement 就可以了`。

- 代码演示：

```java
public class StatementTest {  
  
    // 使用Statement的弊端：需要拼写sql语句，并且存在SQL注入的问题  
    @Test  
    public void testLogin() {  
        Scanner scanner = new Scanner(System.in);  
        System.out.print("请输入用户名：");  
        String user = scanner.next();  
        System.out.print("请输入用户密码：");  
        String password = scanner.next();  
  
        String sql = "SELECT user,password FROM user_table WHERE user = '" + user + "' AND password = '" + password + "'";  
  
      User returnUser = get(sql, User.class);  
      if (returnUser != null)  
         System.out.println("登录成功");  
      else         System.out.println("登录失败");  
   }  
  
    // 使用Statement实现对数据表的查询操作  
    public <T> T get(String sql, Class<T> clazz) {  
        T t = null;  
  
        Connection conn = null;  
        Statement st = null;  
        ResultSet rs = null;  
        try {  
            // 1.加载配置文件  
            InputStream is = StatementTest.class.getClassLoader().getResourceAsStream("jdbc.properties");  
            Properties pros = new Properties();  
            pros.load(is);  
  
            // 2.读取配置信息  
            String user = pros.getProperty("user");  
            String password = pros.getProperty("password");  
            String url = pros.getProperty("url");  
            String driverClass = pros.getProperty("driverClass");  
  
            // 3.加载驱动  
            Class.forName(driverClass);  
  
            // 4.获取连接  
            conn = DriverManager.getConnection(url, user, password);  
  
            st = conn.createStatement();  
  
            rs = st.executeQuery(sql);  
  
            // 获取结果集的元数据  
            ResultSetMetaData rsmd = rs.getMetaData();  
  
            // 获取结果集的列数  
            int columnCount = rsmd.getColumnCount();  
  
            if (rs.next()) {  
  
                t = clazz.newInstance();  
  
                for (int i = 0; i < columnCount; i++) {  
                    // //1. 获取列的名称  
                    // String columnName = rsmd.getColumnName(i+1);  
  
                    // 1. 获取列的别名  
                    String columnName = rsmd.getColumnLabel(i + 1);  
  
                    // 2. 根据列名获取对应数据表中的数据  
                    Object columnVal = rs.getObject(columnName);  
  
                    // 3. 将数据表中得到的数据，封装进对象  
                    Field field = clazz.getDeclaredField(columnName);  
                    field.setAccessible(true);  
                    field.set(t, columnVal);  
                }  
                return t;  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally {  
            // 关闭资源  
            if (rs != null) {  
                try {  
                    rs.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
            if (st != null) {  
                try {  
                    st.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
  
            if (conn != null) {  
                try {  
                    conn.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
  
        return null;  
    }  
  
}
```


## 3 PreparedStatement的使用

### 3.1 PreparedStatement介绍

- 可以通过调用 Connection 对象的 **preparedStatement(String sql)** 方法获取 PreparedStatement 对象

- **PreparedStatement 接口是 Statement 的子接口，它表示一条预编译过的 SQL 语句**

- PreparedStatement 对象所代表的 SQL 语句中的参数用问号(?)来表示，调用 PreparedStatement 对象的 setXxx() 方法来设置这些参数. setXxx() 方法有两个参数，第一个参数是要设置的 SQL 语句中的参数的索引(从 1 开始)，第二个是设置的 SQL 语句中的参数的值

### 3.2 PreparedStatement vs Statement

- 代码的可读性和可维护性。

- **PreparedStatement 能最大可能提高性能：**
  - DBServer会对**预编译**语句提供性能优化。因为预编译语句有可能被重复调用，所以<u>语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。</u>
  - 在statement语句中,即使是相同操作但因为数据内容不一样,所以整个语句本身不能匹配,没有缓存语句的意义.事实是没有数据库会对普通语句编译后的执行代码缓存。这样<u>每执行一次都要对传入的语句编译一次。</u>
  - (语法检查，语义检查，翻译成二进制命令，缓存)

- PreparedStatement 可以防止 SQL 注入 

### 3.3 Java与SQL对应数据类型转换表

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

### 3.4 使用PreparedStatement实现增、删、改操作

```java
//通用的增删改操作  
public void update(String sql,Object ... args){  
    Connection con = null;  
    PreparedStatement ps = null;  
    try {  
        //1.获取连接  
        con = JDBCUtils.getConnection();  
        //2.预编译sql语句，获取PerparedStatement的实例  
        ps = con.prepareStatement(sql);  
        //3.填充占位符  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
        //4.执行  
        ps.execute();  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    } finally {  
        //5.关闭资源  
        JDBCUtils.closeResource(con,ps);  
    }  
}
```


### 3.5 使用PreparedStatement实现查询操作


- `针对一个表customers表，的通用查询操作`
	- 重点1：获取结果集
	- 重点2：调用结果集的next方法，获得每一行数据，while
	- 重点3：`使用结果集获得元数据和列植`
	- 重点4：`使用元数据获得列数和列名`
```java
public Customer queryForCustomers(String sql,Object ... args)  {  
    Connection con = null;  
    PreparedStatement ps = null;  
    ResultSet resultSet = null;  
    try {  
        con = JDBCUtils.getConnection();  
  
        ps = con.prepareStatement(sql);  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
  
        resultSet = ps.executeQuery();  
        //获取结果集的元数据  
        ResultSetMetaData metaData = resultSet.getMetaData();  
        //通过元数据获取结果集中的列数  
        int columnCount = metaData.getColumnCount();  
  
        if (resultSet.next()){  
            Customer customer = new Customer();  
            //处理结果集一行数据的每一个列  
            for (int i = 0; i < columnCount; i++) {  
                //获取列值  
                Object columnValue = resultSet.getObject(i + 1);  
                //获取每个列的列名，注意无法获取到别名
                //String columnName = metaData.getColumnName(i + 1);  
                //使用另外一个方法
                String columnName = metaData.getColumnLabel(i + 1); 
                //给customer的某个属性赋值  
                Field field = Customer.class.getDeclaredField(columnName);  
                field.setAccessible(true);  
                field.set(customer,columnValue);  
            }  
            return customer;  
        }  
    } catch (Exception e) {  
        e.printStackTrace();  
    } finally {  
        JDBCUtils.closeResource(con,ps,resultSet);  
    }  
    return null;  
}
```


- 针对于不同表的通用查询
- 不同表
	- 形参列表多一个，对象所在类的Class对象，且\<T>代泛型
	- 对应的实例的生成也要通过反射建立
- 返回多条记录
	- 返回值数组
```java
public <T> List<T> getForList(Class<T> clazz, String sql, Object ... args){  
    Connection con = null;  
    PreparedStatement ps = null;  
    ResultSet resultSet = null;  
    try {  
        con = JDBCUtils.getConnection();  
  
        ps = con.prepareStatement(sql);  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
  
        resultSet = ps.executeQuery();  
        //获取结果集的元数据  
        ResultSetMetaData metaData = resultSet.getMetaData();  
  
        //通过元数据获取结果集中的列数  
        int columnCount = metaData.getColumnCount();  
  
        //创建集合对象  
        ArrayList<T> list = new ArrayList<>();  
  
        while (resultSet.next()){  
            T t = clazz.getDeclaredConstructor().newInstance();  
            //处理结果集一行数据的每一个列  
            for (int i = 0; i < columnCount; i++) {  
                //获取列值  
                Object columnValue = resultSet.getObject(i + 1);  
                //获取每个列的列名  
                String columnName = metaData.getColumnLabel(i + 1);  
                //给t的某个属性赋值  
                Field field = clazz.getDeclaredField(columnName);  
                field.setAccessible(true);  
                field.set(t,columnValue);  
            }  
            list.add(t);  
        }  
        return list;  
    } catch (Exception e) {  
        e.printStackTrace();  
    } finally {  
        JDBCUtils.closeResource(con,ps,resultSet);  
    }  
    return null;  
}
```

> 说明：使用PreparedStatement实现的查询操作可以替换Statement实现的查询操作，解决Statement拼串和SQL注入问题。
>

## 4 ResultSet与ResultSetMetaData

### 4.1 ResultSet

- 查询需要调用PreparedStatement 的 executeQuery() 方法，查询结果是一个ResultSet 对象

- ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商提供实现
- ResultSet 返回的实际上就是一张数据表。有一个指针指向数据表的第一条记录的前面。

- ResultSet 对象维护了一个指向当前数据行的**游标**，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的 next() 方法移动到下一行。调用 next()方法检测下一行是否有效。若有效，该方法返回 true，且指针下移。相当于Iterator对象的 hasNext() 和 next() 方法的结合体。
- 当指针指向一行时, 可以通过调用 getXxx(int index) 或 getXxx(int columnName) 获取每一列的值。

  - 例如: getInt(1), getString("name")
  - **注意：Java与数据库交互涉及到的相关Java API中的索引都从1开始。**

- ResultSet 接口的常用方法：
  - boolean next()

  - getString()
  - …

  ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2c4ff8b12bc1aa8d911d181122f28dcd.png)



### 4.2 ResultSetMetaData

- 可用于获取关于 ResultSet 对象中列的类型和属性信息的对象

- ResultSetMetaData meta = rs.getMetaData();
  - **getColumnName**(int column)：获取指定列的名称
  - **getColumnLabel**(int column)：获取指定列的别名
  - **getColumnCount**()：返回当前 ResultSet 对象中的列数。 

  - getColumnTypeName(int column)：检索指定列的数据库特定的类型名称。 
  - getColumnDisplaySize(int column)：指示指定列的最大标准宽度，以字符为单位。 
  - **isNullable**(int column)：指示指定列中的值是否可以为 null。 

  -  isAutoIncrement(int column)：指示是否自动为指定列进行编号，这样这些列仍然是只读的。 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/73fa57d73aed8c46047790717cca4d9c.png)



**问题1：得到结果集后, 如何知道该结果集中有哪些列 ？ 列名是什么？**

​     需要使用一个描述 ResultSet 的对象， 即 ResultSetMetaData

**问题2：关于ResultSetMetaData**

1. **如何获取 ResultSetMetaData**： 调用 ResultSet 的 getMetaData() 方法即可
2. **获取 ResultSet 中有多少列**：调用 ResultSetMetaData 的 getColumnCount() 方法
3. **获取 ResultSet 每一列的列的别名是什么**：调用 ResultSetMetaData 的getColumnLabel() 方法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b4da63f397834753af4f4d5461ba943.png)



## 5 资源的释放

- 释放ResultSet, Statement,Connection。
- 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统宕机。Connection的使用原则是**尽量晚创建，尽量早的释放。**
- 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。


## 6 JDBC API小结

- 两种思想
  - 面向接口编程的思想

  - ORM思想(object relational mapping)
    - 一个数据表对应一个java类
    - 表中的一条记录对应java类的一个对象
    - 表中的一个字段对应java类的一个属性

  > sql是需要结合列名和表的属性名来写。注意起别名。

- 两种技术
  - JDBC结果集的元数据：ResultSetMetaData
    - 获取列数：getColumnCount()
    - 获取列的别名：getColumnLabel()
  - 通过反射，创建指定类的对象，获取指定的属性并赋值

## 7 使用PreparedStatement的好处

1. 使用了占位符的方式，解决了Statement中存在的拼串操作；
2. 使用的是预编译方式，避免了直接SQL注入导致逻辑变化；
3. 使用占位符，可以操作Blob的数据；
4. 可以实现更高效的批量操作；

- 补充代码，日期的插入问题
```java
@Test  
public void test2(){  
    String sql = "INSERT INTO `order`(order_name,order_date) VALUES(?,?)";  
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");  
    LocalDate date = LocalDate.parse("1997-05-30",dateTimeFormatter);  
    Object[] args = {"DD",date};  
    update(sql,args);  
}
```



***

## 8 章节练习

**练习题1：从控制台向数据库的表customers中插入一条数据，表结构如下：**

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d30198453bbf15c837922f0e503c91e6.png)



```java
public void test1(){  
    String sql = "INSERT INTO customers1(cust_id,cust_name,email,birth) VALUES(?,?,?,?)";  
    Scanner scan = new Scanner(System.in);  
    System.out.print("请输入id：");  
    int id = Integer.parseInt(scan.next());  
    System.out.print("请输入name：");  
    String name = scan.next();  
    System.out.print("请输入email：");  
    String email = scan.next();  
    System.out.print("请输入birth：");  
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");  
    LocalDate date = LocalDate.parse(scan.next());  
  
    PreparedStatementTest.update(sql,id,name,email,date);  
    System.out.println("插入成功");  
    scan.close();  
}
```

**练习题2：创立数据库表 examstudent，表结构如下：**

向数据表中添加如下数据：

**代码实现1：插入一个新的student 信息**

- 和上题一致！！！
```java
@Test  
public void test2(){  
    String sql = "INSERT INTO examstudent(type,id_card,exam_card,student_name,location,grade) VALUES(?,?,?,?,?,?)";  
    System.out.println("请输入考生的详细信息");  
    Scanner scan = new Scanner(System.in);  
    System.out.print("请输入type：");  
    int type = Integer.parseInt(scan.next());  
    System.out.print("请输入id_card：");  
    String idCard = scan.next();  
    System.out.print("请输入exam_card：");  
    String examCard = scan.next();  
    System.out.print("请输入student_name：");  
    String studentName = scan.next();  
    System.out.print("请输入location：");  
    String location = scan.next();  
    System.out.print("请输入grade：");  
    int grade = Integer.parseInt(scan.next());  
    PreparedStatementTest.update(sql,type,idCard,examCard,studentName,location,grade);  
    System.out.println("信息录入成功!");  
    scan.close();  
}
```

请输入考生的详细信息

Type: 
IDCard:
ExamCard:
StudentName:
Location:
Grade:

信息录入成功!

**代码实现2：在 IDEA中建立 java 程序：输入身份证号或准考证号可以查询到学生的基本信息。结果如下：**

```java
@Test  
public void test3(){  
    System.out.println("请选择您要输入的类型：\na:准考证号\nb:身份证号");  
    Scanner scan = new Scanner(System.in);  
    char choose = scan.next().charAt(0);  
    switch (choose){  
        case 'a':  
            String sqlForExamCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE exam_card = ?";  
            System.out.println("请输入准考证号：");  
            String examCard = scan.next();  
            ArrayList<Student> list1 = PreparedStatementTest.query(Student.class, sqlForExamCard, examCard);  
            Student student1 = list1.get(0);  
            System.out.println(student1.toString1());  
            break;        
		case 'b':  
            String sqlForIdCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE id_card = ?";  
            System.out.println("请输入身份证号：");  
            String idCard = scan.next();  
            ArrayList<Student> list2 = PreparedStatementTest.query(Student.class, sqlForIdCard, idCard);  
            Student student2 = list2.get(0);  
            System.out.println(student2.toString1());  
            break;    }  
    scan.close();  
}
```

**代码实现3：完成学生信息的删除功能**

```java
@Test  
public void test4(){  
    String sqlForExamCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE exam_card = ?";  
    System.out.println("请输入学生的考号：");  
    Scanner scan = new Scanner(System.in);  
    String examCard = scan.next();  
    ArrayList<Student> list1 = PreparedStatementTest.query(Student.class, sqlForExamCard, examCard);  
    if (list1.size()==0)  
        System.out.println("查无此人！请重新进入程序");  
    else{  
        String sql = "DELETE FROM examstudent WHERE exam_card = ?";  
        PreparedStatementTest.update(sql,examCard);  
        System.out.println("删除成功");  
    }  
}  


//test4的优化  
@Test  
public void test5(){  
    System.out.println("请输入学生的考号：");  
    Scanner scan = new Scanner(System.in);  
    String examCard = scan.next();  
    String sql = "DELETE FROM examstudent WHERE exam_card = ?";  
    int i = PreparedStatementTest.update1(sql, examCard);  
    if (i>0)  
        System.out.println("删除成功");  
    else        System.out.println("查无此人，请重新输入");  
}
```

***



- CRUD是指 **Create（创建）、Retrieve（读取）、Update（更新）和Delete（删除）** 这四个操作，是指在数据库或系统中对数据进行基本操作的常用术语

## 1 操作和访问数据库

- 数据库连接被用于向数据库服务器发送命令和 SQL 语句，并接受数据库服务器返回的结果。`其实一个数据库连接就是一个Socket连接`。

- 在 java.sql 包中`有 3 个接口分别定义了对数据库的调用`的不同方式：
	 -` Statement`：用于`执行静态 SQL 语句`并返回它所生成结果的对象。 
	- `PrepatedStatement`：SQL 语句被`预编译并存储在此对象`中，可以使用此对象`多次高效地执行该语句`。 
	- `CallableStatement`：用于执行 SQL 存储过程


## 2 使用Statement操作数据表的弊端

- 通过调用 Connection 对象的 createStatement() 方法创建该对象。该对象用于执行静态的 SQL 语句，并且返回执行结果。

- Statement 接口中定义了下列方法用于执行 SQL 语句：

  ```mysql
  int excuteUpdate(String sql)：执行更新操作INSERT、UPDATE、DELETE
  ResultSet executeQuery(String sql)：执行查询操作SELECT
  ```

- 但是使用Statement操作数据表存在弊端：

  - **问题一：存在拼串操作，繁琐**
  - **问题二：存在SQL注入问题**

- SQL 注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的 SQL 语句段或命令(如：SELECT user, password FROM user_table WHERE user='a' OR 1 = ' AND password = ' OR '1' = '1') ，从而利用系统的 SQL 引擎完成恶意行为的做法。

- 对于 Java 而言，要防范 SQL 注入，只要用`PreparedStatement(从Statement扩展而来) 取代 Statement 就可以了`。

- 代码演示：

```java
public class StatementTest {  
  
    // 使用Statement的弊端：需要拼写sql语句，并且存在SQL注入的问题  
    @Test  
    public void testLogin() {  
        Scanner scanner = new Scanner(System.in);  
        System.out.print("请输入用户名：");  
        String user = scanner.next();  
        System.out.print("请输入用户密码：");  
        String password = scanner.next();  
  
        String sql = "SELECT user,password FROM user_table WHERE user = '" + user + "' AND password = '" + password + "'";  
  
      User returnUser = get(sql, User.class);  
      if (returnUser != null)  
         System.out.println("登录成功");  
      else         System.out.println("登录失败");  
   }  
  
    // 使用Statement实现对数据表的查询操作  
    public <T> T get(String sql, Class<T> clazz) {  
        T t = null;  
  
        Connection conn = null;  
        Statement st = null;  
        ResultSet rs = null;  
        try {  
            // 1.加载配置文件  
            InputStream is = StatementTest.class.getClassLoader().getResourceAsStream("jdbc.properties");  
            Properties pros = new Properties();  
            pros.load(is);  
  
            // 2.读取配置信息  
            String user = pros.getProperty("user");  
            String password = pros.getProperty("password");  
            String url = pros.getProperty("url");  
            String driverClass = pros.getProperty("driverClass");  
  
            // 3.加载驱动  
            Class.forName(driverClass);  
  
            // 4.获取连接  
            conn = DriverManager.getConnection(url, user, password);  
  
            st = conn.createStatement();  
  
            rs = st.executeQuery(sql);  
  
            // 获取结果集的元数据  
            ResultSetMetaData rsmd = rs.getMetaData();  
  
            // 获取结果集的列数  
            int columnCount = rsmd.getColumnCount();  
  
            if (rs.next()) {  
  
                t = clazz.newInstance();  
  
                for (int i = 0; i < columnCount; i++) {  
                    // //1. 获取列的名称  
                    // String columnName = rsmd.getColumnName(i+1);  
  
                    // 1. 获取列的别名  
                    String columnName = rsmd.getColumnLabel(i + 1);  
  
                    // 2. 根据列名获取对应数据表中的数据  
                    Object columnVal = rs.getObject(columnName);  
  
                    // 3. 将数据表中得到的数据，封装进对象  
                    Field field = clazz.getDeclaredField(columnName);  
                    field.setAccessible(true);  
                    field.set(t, columnVal);  
                }  
                return t;  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally {  
            // 关闭资源  
            if (rs != null) {  
                try {  
                    rs.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
            if (st != null) {  
                try {  
                    st.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
  
            if (conn != null) {  
                try {  
                    conn.close();  
                } catch (SQLException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
  
        return null;  
    }  
  
}
```


## 3 PreparedStatement的使用

### 3.1 PreparedStatement介绍

- 可以通过调用 Connection 对象的 **preparedStatement(String sql)** 方法获取 PreparedStatement 对象

- **PreparedStatement 接口是 Statement 的子接口，它表示一条预编译过的 SQL 语句**

- PreparedStatement 对象所代表的 SQL 语句中的参数用问号(?)来表示，调用 PreparedStatement 对象的 setXxx() 方法来设置这些参数. setXxx() 方法有两个参数，第一个参数是要设置的 SQL 语句中的参数的索引(从 1 开始)，第二个是设置的 SQL 语句中的参数的值

### 3.2 PreparedStatement vs Statement

- 代码的可读性和可维护性。

- **PreparedStatement 能最大可能提高性能：**
  - DBServer会对**预编译**语句提供性能优化。因为预编译语句有可能被重复调用，所以<u>语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。</u>
  - 在statement语句中,即使是相同操作但因为数据内容不一样,所以整个语句本身不能匹配,没有缓存语句的意义.事实是没有数据库会对普通语句编译后的执行代码缓存。这样<u>每执行一次都要对传入的语句编译一次。</u>
  - (语法检查，语义检查，翻译成二进制命令，缓存)

- PreparedStatement 可以防止 SQL 注入 

### 3.3 Java与SQL对应数据类型转换表

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

### 3.4 使用PreparedStatement实现增、删、改操作

```java
//通用的增删改操作  
public void update(String sql,Object ... args){  
    Connection con = null;  
    PreparedStatement ps = null;  
    try {  
        //1.获取连接  
        con = JDBCUtils.getConnection();  
        //2.预编译sql语句，获取PerparedStatement的实例  
        ps = con.prepareStatement(sql);  
        //3.填充占位符  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
        //4.执行  
        ps.execute();  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    } finally {  
        //5.关闭资源  
        JDBCUtils.closeResource(con,ps);  
    }  
}
```


### 3.5 使用PreparedStatement实现查询操作


- `针对一个表customers表，的通用查询操作`
	- 重点1：获取结果集
	- 重点2：调用结果集的next方法，获得每一行数据，while
	- 重点3：`使用结果集获得元数据和列植`
	- 重点4：`使用元数据获得列数和列名`
```java
public Customer queryForCustomers(String sql,Object ... args)  {  
    Connection con = null;  
    PreparedStatement ps = null;  
    ResultSet resultSet = null;  
    try {  
        con = JDBCUtils.getConnection();  
  
        ps = con.prepareStatement(sql);  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
  
        resultSet = ps.executeQuery();  
        //获取结果集的元数据  
        ResultSetMetaData metaData = resultSet.getMetaData();  
        //通过元数据获取结果集中的列数  
        int columnCount = metaData.getColumnCount();  
  
        if (resultSet.next()){  
            Customer customer = new Customer();  
            //处理结果集一行数据的每一个列  
            for (int i = 0; i < columnCount; i++) {  
                //获取列值  
                Object columnValue = resultSet.getObject(i + 1);  
                //获取每个列的列名，注意无法获取到别名
                //String columnName = metaData.getColumnName(i + 1);  
                //使用另外一个方法
                String columnName = metaData.getColumnLabel(i + 1); 
                //给customer的某个属性赋值  
                Field field = Customer.class.getDeclaredField(columnName);  
                field.setAccessible(true);  
                field.set(customer,columnValue);  
            }  
            return customer;  
        }  
    } catch (Exception e) {  
        e.printStackTrace();  
    } finally {  
        JDBCUtils.closeResource(con,ps,resultSet);  
    }  
    return null;  
}
```


- 针对于不同表的通用查询
- 不同表
	- 形参列表多一个，对象所在类的Class对象，且\<T>代泛型
	- 对应的实例的生成也要通过反射建立
- 返回多条记录
	- 返回值数组
```java
public <T> List<T> getForList(Class<T> clazz, String sql, Object ... args){  
    Connection con = null;  
    PreparedStatement ps = null;  
    ResultSet resultSet = null;  
    try {  
        con = JDBCUtils.getConnection();  
  
        ps = con.prepareStatement(sql);  
        for (int i = 0; i < args.length; i++) {  
            ps.setObject(i+1,args[i]);  
        }  
  
        resultSet = ps.executeQuery();  
        //获取结果集的元数据  
        ResultSetMetaData metaData = resultSet.getMetaData();  
  
        //通过元数据获取结果集中的列数  
        int columnCount = metaData.getColumnCount();  
  
        //创建集合对象  
        ArrayList<T> list = new ArrayList<>();  
  
        while (resultSet.next()){  
            T t = clazz.getDeclaredConstructor().newInstance();  
            //处理结果集一行数据的每一个列  
            for (int i = 0; i < columnCount; i++) {  
                //获取列值  
                Object columnValue = resultSet.getObject(i + 1);  
                //获取每个列的列名  
                String columnName = metaData.getColumnLabel(i + 1);  
                //给t的某个属性赋值  
                Field field = clazz.getDeclaredField(columnName);  
                field.setAccessible(true);  
                field.set(t,columnValue);  
            }  
            list.add(t);  
        }  
        return list;  
    } catch (Exception e) {  
        e.printStackTrace();  
    } finally {  
        JDBCUtils.closeResource(con,ps,resultSet);  
    }  
    return null;  
}
```

> 说明：使用PreparedStatement实现的查询操作可以替换Statement实现的查询操作，解决Statement拼串和SQL注入问题。
>

## 4 ResultSet与ResultSetMetaData

### 4.1 ResultSet

- 查询需要调用PreparedStatement 的 executeQuery() 方法，查询结果是一个ResultSet 对象

- ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商提供实现
- ResultSet 返回的实际上就是一张数据表。有一个指针指向数据表的第一条记录的前面。

- ResultSet 对象维护了一个指向当前数据行的**游标**，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的 next() 方法移动到下一行。调用 next()方法检测下一行是否有效。若有效，该方法返回 true，且指针下移。相当于Iterator对象的 hasNext() 和 next() 方法的结合体。
- 当指针指向一行时, 可以通过调用 getXxx(int index) 或 getXxx(int columnName) 获取每一列的值。

  - 例如: getInt(1), getString("name")
  - **注意：Java与数据库交互涉及到的相关Java API中的索引都从1开始。**

- ResultSet 接口的常用方法：
  - boolean next()

  - getString()
  - …

  ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2c4ff8b12bc1aa8d911d181122f28dcd.png)



### 4.2 ResultSetMetaData

- 可用于获取关于 ResultSet 对象中列的类型和属性信息的对象

- ResultSetMetaData meta = rs.getMetaData();
  - **getColumnName**(int column)：获取指定列的名称
  - **getColumnLabel**(int column)：获取指定列的别名
  - **getColumnCount**()：返回当前 ResultSet 对象中的列数。 

  - getColumnTypeName(int column)：检索指定列的数据库特定的类型名称。 
  - getColumnDisplaySize(int column)：指示指定列的最大标准宽度，以字符为单位。 
  - **isNullable**(int column)：指示指定列中的值是否可以为 null。 

  -  isAutoIncrement(int column)：指示是否自动为指定列进行编号，这样这些列仍然是只读的。 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/73fa57d73aed8c46047790717cca4d9c.png)



**问题1：得到结果集后, 如何知道该结果集中有哪些列 ？ 列名是什么？**

​     需要使用一个描述 ResultSet 的对象， 即 ResultSetMetaData

**问题2：关于ResultSetMetaData**

1. **如何获取 ResultSetMetaData**： 调用 ResultSet 的 getMetaData() 方法即可
2. **获取 ResultSet 中有多少列**：调用 ResultSetMetaData 的 getColumnCount() 方法
3. **获取 ResultSet 每一列的列的别名是什么**：调用 ResultSetMetaData 的getColumnLabel() 方法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b4da63f397834753af4f4d5461ba943.png)



## 5 资源的释放

- 释放ResultSet, Statement,Connection。
- 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统宕机。Connection的使用原则是**尽量晚创建，尽量早的释放。**
- 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。


## 6 JDBC API小结

- 两种思想
  - 面向接口编程的思想

  - ORM思想(object relational mapping)
    - 一个数据表对应一个java类
    - 表中的一条记录对应java类的一个对象
    - 表中的一个字段对应java类的一个属性

  > sql是需要结合列名和表的属性名来写。注意起别名。

- 两种技术
  - JDBC结果集的元数据：ResultSetMetaData
    - 获取列数：getColumnCount()
    - 获取列的别名：getColumnLabel()
  - 通过反射，创建指定类的对象，获取指定的属性并赋值

## 7 使用PreparedStatement的好处

1. 使用了占位符的方式，解决了Statement中存在的拼串操作；
2. 使用的是预编译方式，避免了直接SQL注入导致逻辑变化；
3. 使用占位符，可以操作Blob的数据；
4. 可以实现更高效的批量操作；

- 补充代码，日期的插入问题
```java
@Test  
public void test2(){  
    String sql = "INSERT INTO `order`(order_name,order_date) VALUES(?,?)";  
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");  
    LocalDate date = LocalDate.parse("1997-05-30",dateTimeFormatter);  
    Object[] args = {"DD",date};  
    update(sql,args);  
}
```



***

## 8 章节练习

**练习题1：从控制台向数据库的表customers中插入一条数据，表结构如下：**

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d30198453bbf15c837922f0e503c91e6.png)



```java
public void test1(){  
    String sql = "INSERT INTO customers1(cust_id,cust_name,email,birth) VALUES(?,?,?,?)";  
    Scanner scan = new Scanner(System.in);  
    System.out.print("请输入id：");  
    int id = Integer.parseInt(scan.next());  
    System.out.print("请输入name：");  
    String name = scan.next();  
    System.out.print("请输入email：");  
    String email = scan.next();  
    System.out.print("请输入birth：");  
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");  
    LocalDate date = LocalDate.parse(scan.next());  
  
    PreparedStatementTest.update(sql,id,name,email,date);  
    System.out.println("插入成功");  
    scan.close();  
}
```

**练习题2：创立数据库表 examstudent，表结构如下：**

向数据表中添加如下数据：

**代码实现1：插入一个新的student 信息**

- 和上题一致！！！
```java
@Test  
public void test2(){  
    String sql = "INSERT INTO examstudent(type,id_card,exam_card,student_name,location,grade) VALUES(?,?,?,?,?,?)";  
    System.out.println("请输入考生的详细信息");  
    Scanner scan = new Scanner(System.in);  
    System.out.print("请输入type：");  
    int type = Integer.parseInt(scan.next());  
    System.out.print("请输入id_card：");  
    String idCard = scan.next();  
    System.out.print("请输入exam_card：");  
    String examCard = scan.next();  
    System.out.print("请输入student_name：");  
    String studentName = scan.next();  
    System.out.print("请输入location：");  
    String location = scan.next();  
    System.out.print("请输入grade：");  
    int grade = Integer.parseInt(scan.next());  
    PreparedStatementTest.update(sql,type,idCard,examCard,studentName,location,grade);  
    System.out.println("信息录入成功!");  
    scan.close();  
}
```

请输入考生的详细信息

Type: 
IDCard:
ExamCard:
StudentName:
Location:
Grade:

信息录入成功!

**代码实现2：在 IDEA中建立 java 程序：输入身份证号或准考证号可以查询到学生的基本信息。结果如下：**

```java
@Test  
public void test3(){  
    System.out.println("请选择您要输入的类型：\na:准考证号\nb:身份证号");  
    Scanner scan = new Scanner(System.in);  
    char choose = scan.next().charAt(0);  
    switch (choose){  
        case 'a':  
            String sqlForExamCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE exam_card = ?";  
            System.out.println("请输入准考证号：");  
            String examCard = scan.next();  
            ArrayList<Student> list1 = PreparedStatementTest.query(Student.class, sqlForExamCard, examCard);  
            Student student1 = list1.get(0);  
            System.out.println(student1.toString1());  
            break;        
		case 'b':  
            String sqlForIdCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE id_card = ?";  
            System.out.println("请输入身份证号：");  
            String idCard = scan.next();  
            ArrayList<Student> list2 = PreparedStatementTest.query(Student.class, sqlForIdCard, idCard);  
            Student student2 = list2.get(0);  
            System.out.println(student2.toString1());  
            break;    }  
    scan.close();  
}
```

**代码实现3：完成学生信息的删除功能**

```java
@Test  
public void test4(){  
    String sqlForExamCard = "SELECT flow_id flowId,type,id_card idCard,exam_card examCard,student_name studentName,location,grade FROM examstudent WHERE exam_card = ?";  
    System.out.println("请输入学生的考号：");  
    Scanner scan = new Scanner(System.in);  
    String examCard = scan.next();  
    ArrayList<Student> list1 = PreparedStatementTest.query(Student.class, sqlForExamCard, examCard);  
    if (list1.size()==0)  
        System.out.println("查无此人！请重新进入程序");  
    else{  
        String sql = "DELETE FROM examstudent WHERE exam_card = ?";  
        PreparedStatementTest.update(sql,examCard);  
        System.out.println("删除成功");  
    }  
}  


//test4的优化  
@Test  
public void test5(){  
    System.out.println("请输入学生的考号：");  
    Scanner scan = new Scanner(System.in);  
    String examCard = scan.next();  
    String sql = "DELETE FROM examstudent WHERE exam_card = ?";  
    int i = PreparedStatementTest.update1(sql, examCard);  
    if (i>0)  
        System.out.println("删除成功");  
    else        System.out.println("查无此人，请重新输入");  
}
```

***

