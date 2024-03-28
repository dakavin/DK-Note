

## 一、MySQL BLOB类型

- MySQL中，`BLOB是一个二进制大型对象`，是一个可以存储大量数据的容器，它能容纳不同大小的数据。
- `插入BLOB类型的数据必须使用PreparedStatement`，因为BLOB类型的数据无法使用字符串拼接写的。

- MySQL的四种BLOB类型(除了在存储的最大信息量上不同外，他们是等同的)

- 实际使用中根据需要存入的数据大小定义不同的BLOB类型。
- 需要注意的是：`如果存储的文件过大，数据库的性能会下降`。
- 如果在指定了相关的Blob类型以后，还报错：xxx too large，那么在mysql的安装目录下，找my.ini文件加上如下的配置参数： **max_allowed_packet=16M**。同时注意：修改了my.ini文件之后，需要重新启动mysql服务。

## 二、向数据表中插入大数据类型

```java
//1.实现数据表customers的Blob插入操作  
@Test  
public void testInsert(){  
    Connection con = null;  
    PreparedStatement ps = null;  
    try {  
        con = JDBCUtils.getConnection();  
        String sql = "INSERT INTO customers(name,email,birth,photo) VALUES(?,?,?,?)";  
        ps = con.prepareStatement(sql);  
        ps.setObject(1,"屠天压");  
        ps.setObject(2,"tutian@216.com");  
        ps.setObject(3,"1998-6-29");  
        FileInputStream is = new FileInputStream("C:\\Users\\mikey\\Desktop\\壁纸\\1.jpg");  
        ps.setBlob(4, is);  
  
        int i = ps.executeUpdate();  
        if (i>0)  
            System.out.println("插入成功");  
        else            
	        System.out.println("插入失败");  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    } finally {  
        JDBCUtils.updateClose(con,ps);  
    }  
}

```



## 三、修改数据表中的Blob类型字段

```java
//2.实现数据表customers的Blob替换操作  
@Test  
public void testReplace(){  
    Connection con = null;  
    PreparedStatement ps = null;  
    try {  
        con = JDBCUtils.getConnection();  
        String sql = "UPDATE customers SET photo = ? WHERE id = ?";  
        ps = con.prepareStatement(sql);  
  
        FileInputStream is = new FileInputStream("C:\\Users\\mikey\\Desktop\\graphics.jpg");  
        ps.setBlob(1, is);  
        ps.setObject(2,23);  
        int i = ps.executeUpdate();  
        if (i>0)  
            System.out.println("修改成功");  
        else            
	        System.out.println("修改失败");  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    } finally {  
        JDBCUtils.updateClose(con,ps);  
    }  
}
```



## 四、 从数据表中读取大数据类型

```java
//3.实现数据表customers的Blob获取操作  
@Test  
public void testGetBlob(){  
    Connection con = null;  
    PreparedStatement ps = null;  
    ResultSet rs = null;  
    InputStream is = null;  
    FileOutputStream fos = null;  
    try {  
        con = JDBCUtils.getConnection();  
        String sql = "SELECT photo FROM customers WHERE id = ?";  
        ps = con.prepareStatement(sql);  
        ps.setObject(1,23);  
  
        rs = ps.executeQuery();  
        if (rs.next()){  
            is = rs.getBlob("photo").getBinaryStream();  
            fos = new FileOutputStream("3.jpg");  
            byte[] buffer = new byte[1024];  
            int len;  
            while ((len = is.read(buffer))!=-1){  
                fos.write(buffer,0,len);  
            }  
        }  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    } finally {  
        JDBCUtils.queryClose(con,ps,rs);  
        try {  
            if (fos!=null)  
                fos.close();  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
        try {  
            if (is!=null)  
                is.close();  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

