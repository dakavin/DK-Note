
## 第1节   SpringBoot事务       

### 1.1 事务的回顾

#### 1、事务的概念

事务指逻辑上的⼀组操作，组成这组操作的各个单元，要么全部成功，要么全部不成功。从⽽确保了数据的准确与安全。
#### 2、事务的四⼤特性

原⼦性（Atomicity）

⼀致性（Consistency）

隔离性（Isolation）

持久性（Durability）

#### 3、事务的隔离级别

Serializable（串⾏化）

Repeatable read（可重复读）

Read committed（读已提交）

Read uncommitted（读未提交）

### 1.2 SpringBoot中事务实现方式

编程式事务：在业务代码中添加事务控制代码，这样的事务控制机制就叫做编程式事务

声明式事务：通过xml或者注解配置的⽅式达到事务控制的⽬的，叫做声明式事务

### 1.3 注解方式开启事务（@Transactional 注解）

SpringBoot中已经默认对JDBC、Mybatis开启了事务，引入这些依赖的时候，事务就默认开启，只需一个@Transactional就可以了。

`@Transactional 生效的原理`
	(1) 在spring的bean的初始化过程中，通过判断bean的方法是否标有事务注解(@Transactional)来决定受否创建代理对象。如果一个类中任意方法含有事务注解，那么这个对象就会被代理。
	(2) `核心思想就是Aop`，bean的方法就是切点，@Transactional注解对应的解释器会根据注解的信息解析出需要增强的逻辑，此处的切面逻辑类似于@Around，会在方法的前后执行一些事务相关操作（`设置隔离级别，传播行为，定义一些回滚或提交的条件等`）。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/19281736a3f62183cc272c83217ddc41.png)


`Connection和Sqlsession的关系`：

Connection： 连接数据源的物理链路,一般项目都会使用连接池来提高性能。

SqlSession： 一个与数据库的会话对象，基于数据库连接。SqlSession在执行sql语句时会根据调度策略，选取其中一个Connection。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e5255a5d996a874ecee01de1a6c3712.png)


Mybatis源码中Sqlsession的提交最终会操作Connection对象提交事务。

没使用事务时，每个sql操作都会创建Sqlsession

在没使用@Transactional注解标记方法式，SpringBoot整合Mybatis的情况下,

每个方法的每个sql操作都会创建一个与数据库的会话，即Sqlsession对象，它是基于数据库连接Connection对象的会话对象。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4554ae4fdc5258f777f119ef8d2eef31.png)


开启Sqlsession的操作日志

//开启Sqlsession的操作日志

logging.level.org.mybatis.spring.SqlSessionUtils=debug

创建测试方法：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/00044434daa26f4a9fdedc9ac2332f51.png)


访问后查看日志。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/166ec83dbadb25596bf470e830ff22a1.png)


`使用事务时，事务方法内的所有sql操作共用一个Session对象，共同提交或回滚，包括方法内容调用的非事务方法内部的sql操作`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d6a142c98ae0e84f865aa39568b93e7.png)


### 1.4 @Transactional的属性

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/26da3995274d313f817b16cdd670f067.png)

### 1.5 @Transactional的注意事项

1、@Transactional注解默认`在抛出RuntimeException或者Error时才会触发事务的回滚`，常见的非RuntimeException是不会触发事务的回滚的。我们平时做业务处理时，需要捕获异常，所以可以手动抛出RuntimeException异常或者添加rollbackFor=Exception.class(也可以指定相应异常)

常见的RuntimeException：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/26a797098b8d8c66d2769512fb245749.png)

SQLException并不是RuntimeException的子类。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33fff49cf7385100e57ba402a566dcd3.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0135a4d6b91018cbf3adb0ccc539ecaa.png)

这里大家会有个疑问？

`为什么有时候发生了SqlException也会回滚？`

原因：当我们在项目开发中加入了Spring框架以后，SQLException会被Spring捕捉后抛出其他自定义的RuntimeException

2、`只有public修饰的方法才会生效`

3、（同一个类中）`当无事务方法调用有事务的方法时事务不会生效`（a方法内通过this.b()，调用的this(被代理对象)的方法，不会增强），而主方法有事务去调用其他方法，无论被调用的方法有无事务，且是否出现异常（有异常需要能够抛出不被捕获），都触发事务。

### 1.6、配置事务的隔离级别

隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

1、`TransactionDefinition.ISOLATION_DEFAULT`：这是默认值，表示使用底层数据库的默认隔离级别。
`对于Mysql数据库就是可重复读`TransactionDefinition.ISOLATION_REPEATABLE_READ，
对于Oracle数据库而言，这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。 
对应的枚举值为：
（Isolation. DEFAULT）注意：前面两句话和课程视频中显示不同，视频中未分别对不同数据库进行分类说明。

 2、`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。对应的枚举值为：（Isolation. READ_UNCOMMITTED）

3、`TransactionDefinition.ISOLATION_READ_COMMITTED`：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。对应的枚举值为：（Isolation.READ_COMMITTED）

4、`TransactionDefinition.ISOLATION_REPEATABLE_READ`：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。对应的枚举值为：（Isolation. REPEATABLE_READ）

 5、TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。对应的枚举值为：（Isolation. SERIALIZABLE）

### 1.7、配置事务的传播行为

所谓事务的传播行为是指，`事务方法在被调用时的运行机制`。是事务方法被其它事务方法或非事务方法调用时的运行机制。

我们要研究的就是比如ServiceA的A方法调用serviceB的B方法，在两个方法不同传播行为下的运行机制：

1、`TransactionDefinition.PROPAGATION_REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则自己创建一个新的事务并运行在其中。这是`默认值（实际开发99.9%都是这种）`，对应的枚举值为：（Propagation. REQUIRED）

2、`TransactionDefinition.PROPAGATION_REQUIRES_NEW`：`创建一个新的事务并运行在这个新事务中`，如果当前存在事务，则把当前事务挂起。对应的枚举值为：（Propagation. REQUIRES_NEW）

3、`TransactionDefinition.PROPAGATION_SUPPORTS`：`如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行`（源码叙述了此处仍然会共享事务的资源比如同一个Sqlsession，每个sql执行完自动提交，如果发生了错误，之前的不会回滚）。 对应的枚举值为：（Propagation. SUPPORTS）

4、`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行（此处仍然会共享事务的资源比如同一个Sqlsession，每个sql执行完自动提交，如果发生了错误，之前的不会回滚），如果当前存在事务，则把当前事务挂起。对应的枚举值为：（Propagation. NOT_SUPPORTED）

 5、`TransactionDefinition.PROPAGATION_NEVER`：不可以存在与于别的事务中，若调用方法有事务，则抛出异常。对应的枚举值为：（Propagation. NEVER）

6、`TransactionDefinition.PROPAGATION_MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。对应的枚举值为：（Propagation. MANDATORY）

 7、`TransactionDefinition.PROPAGATION_NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，嵌套事务可以看成外层事务的子事务，子事务可以单独提交或回滚，如果外层事务回滚，子事务也会回滚，`但是子事务回滚，外层事务不一定回滚`；如果当前没有事务，则该等价于TransactionDefinition.PROPAGATION_REQUIRED。对应的枚举值为：（Propagation. NESTED）

补充知识点：

·   `NESTED和REQUIRES_NEW的区别`

REQUIRES_NEW是新建一个事务并且新开启的这个事务与原有事务无关，而NESTED则是当前存在事务时（我们把当前事务称之为父事务）会开启一个嵌套事务（称之为一个子事务）。  
在NESTED情况下父事务回滚时，子事务也会回滚，而在REQUIRES_NEW情况下，原有事务回滚，不会影响新开启的事务。

·   `NESTED和REQUIRED的区别`

REQUIRED情况下，调用方存在事务时，则被调用方和调用方使用同一事务，那么被调用方出现异常时，由于共用一个事务，所以无论调用方是否catch其异常，事务都会回滚  
而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，`这样只有子事务回滚，父事务不受影响`

## 第2节    Mybatis的缓存

### 2.1 一级缓存

`一级缓存是基于SqlSession`，一个 SqlSession执行完一条查询，会将结果保存起来，当该SqlSession再次执行相同的查询的语句时就会直接返回缓存的结果，不会再次去查数据库。

验证：可以在标记了@Transactional注解的方法中，两次执行相同的查询语句，因为事务方法会共用一个SqlSession，所以第二次不会再执行sql。

注意：springBoot中，非事务方法，每次sql操作会创建不同的SqlSession，`所以除了事务方法一级缓存会生效，其余情况都不会生效`

### 2.2 二级缓存

二级缓存的原理和一级缓存原理一样，第一次查询，会将数据放入缓存中，然后第二次查询则会直接去缓存中取。但是`一级缓存是基于SqlSession`的，而`二级缓存是基于namespace的`（不同的namespace缓存内容是互不相干的），也就是说多个SqlSession可以共享一个mapper中的二级缓存区域，并且如果两个mapper的namespace相同，即使是两个mapper,那么这两个mapper中执行sql查询到的数据也将存在相同的二级缓存区域中。

二级缓存使用步骤：

1、二级缓存是默认关闭的，需要手动配置开启。

在Mapper.XMl映射文件中开启二级缓存。
`<cache></cache>`

2、开启了二级缓存后，`还需要将要缓存的Entity实现Serializable接口`

补充知识点：

`序列化：内存中的对象转化成输出流然写到磁盘上`，

java.io.ObjectOutputStream代表对象输出流，它的writeObject(Object obj)方法可对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中。

`反序列化：磁盘上读取输入流，然后转化成java对象。`　　

java.io.ObjectInputStream代表对象输入流，它的readObject()方法从一个源输入流中读取字节序列，再把它们反序列化为一个对象，并将其返回。

3、并且要使用idea为该Entity类添加序列化ID

序列化ID它决定着是否能够成功反序列化，在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地实体类中的serialVersionUID进行比较如果相同则认为是一致的，便可以进行反序列化

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e4cd0761ca87b59bfaf192cf6087430.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f76f4e76f0f5d48437ac10aa62d44f14.png)

然后选中实体类名称 使用快捷键 Alt + Enter（回车）即可添加序列化id

验证二级缓存的作用：在非事务方法中执行相同的两次查询。因为非事务方法中两次查询时不同SqlSession，所以此时一级缓存并不生效，我们第二次的结果其实是来自于二级缓存。

`高频面试知识点`：

1、二级缓存什么时候生效(就是说一级缓存的结果什么时候移到二级缓存)？

SQLsession执行了commit或close

2、二级缓存什么时候失效？

测试执行update、add、delete操作，二级缓存数据清空

3、一级缓存和二级缓存执行顺序

先查二级缓存，再查一级缓存，再查数据库；