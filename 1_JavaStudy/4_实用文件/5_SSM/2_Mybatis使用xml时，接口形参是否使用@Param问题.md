
`还是有bug，使用mybatis2.3.0版本，在动态查询语句的时候，必须使用@Param注解，而在动态更新语句的时候，就不需要使用@Param注解`

有一些小伙伴觉得 MyBatis 只有方法中存在多个参数的时候，才需要添加 @Param 注解，其实这个理解是不准确的。即使 MyBatis 方法只有一个参数，也可能会用到 @Param 注解。

但是，在你总结出规律之前，你可能会觉得莫名其妙，有的时候一个参数明明不用添加 @Param 注解，有的时候，却需要添加，不添加会报错。

有的人会觉得这是 MyBatis 各个版本差异的锅，不可否认，MyBatis 发展很快，不同版本之间的差异还挺明显的，不过这个加不加 @Param 注解的问题，却并不是版本的锅！今天松哥就和大家来聊一聊这个问题，到底哪些情况下需要添加 @Param 注解。

首先，如下几个需要添加 @Param 注解的场景，相信大家都已经有共识了：

- 第一种：方法有多个参数，需要 @Param 注解

例如下面这样：

```java
@Mapper
public interface UserMapper {
 Integer insert(@Param("username") String username, @Param("address") String address);
}
```

对应的 XML 文件如下：

```xml
<insert id="insert" parameterType="org.javaboy.helloboot.bean.User">
    insert into user (username,address) values (#{username},#{address});
</insert>
```

这是最常见的需要添加 @Param 注解的场景。

- 第二种：方法参数要取别名，需要 @Param 注解

当需要给参数取一个别名的时候，我们也需要 @Param 注解，例如方法定义如下：

```java
@Mapper
public interface UserMapper {
 User getUserByUsername(@Param("name") String username);
}
```

对应的 XML 定义如下：

```xml
<select id="getUserByUsername" parameterType="org.javaboy.helloboot.bean.User">
    select * from user where username=#{name};
</select>
```

老实说，这种需求不多，费事。

- 第三种：XML 中的 SQL 使用了 $ ，那么参数中也需要 @Param 注解

$ 会有注入漏洞的问题，但是有的时候你不得不使用 $ 符号，例如要传入列名或者表名的时候，这个时候必须要添加 @Param 注解，例如：

```java
@Mapper
public interface UserMapper {
 List<User> getAllUsers(@Param("order_by")String order_by);
}
```

对应的 XML 定义如下：

```xml
<select id="getAllUsers" resultType="org.javaboy.helloboot.bean.User">
    select * from user
 <if test="order_by!=null and order_by!=''">
        order by ${order_by} desc
 </if>
</select>
```

前面这三种，都很容易懂，相信很多小伙伴也都懂，除了这三种常见的场景之外，还有一个特殊的场景，经常被人忽略。

- 第四种，那就是动态 SQL ，如果在动态 SQL 中使用了参数作为变量，那么也需要 @Param 注解，即使你只有一个参数。

**如果我们在动态 SQL 中用到了 参数作为判断条件，那么也是一定要加 @Param 注解的**，例如如下方法：

```java
@Mapper
public interface UserMapper {
 List<User> getUserById(@Param("id")Integer id);
}
```

定义出来的 SQL 如下：

```xml
<select id="getUserById" resultType="org.javaboy.helloboot.bean.User">
    select * from user
 <if test="id!=null">
        where id=#{id}
 </if>
</select>
```

