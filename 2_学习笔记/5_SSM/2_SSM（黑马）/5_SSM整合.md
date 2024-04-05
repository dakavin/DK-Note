## 1 SSM整合

前面我们已经把`Mybatis`、`Spring`和`SpringMVC`三个框架进行了学习，那现在我们把这三个框架整合在一起，来完成我们的业务功能开发，具体如何来整合，我们一步步来学习。
### 1.1 流程分析

1. 创建工程
    - 创建一个Maven的web工程
    - pom.xml添加SSM需要的依赖jar包
    - 编写Web项目的入口配置类，实现`AbstractAnnotationConfigDispatcherServletInitializer`重写以下方法
        - `getRootConfigClasses()` ：返回Spring的配置类 —> 需要`SpringConfig`配置类
        - `getServletConfigClasses()` ：返回SpringMVC的配置类 —> 需要`SpringMvcConfig`配置类
        - `getServletMappings()` : 设置SpringMVC请求拦截路径规则
        - `getServletFilters()` ：设置过滤器，解决POST请求中文乱码问题

2. SSM整合(<font color="#00b050">重点是各个配置的编写</font>)
    - `SpringConfig`
        - 标识该类为配置类，使用`@Configuration`
        - 扫描`Service`所在的包，使用`@ComponentScan`
        - 在`Service`层要管理事务，使用`@EnableTransactionManagement`
        - 读取外部的`properties`配置文件，使用`@PropertySource`
        - 整合`Mybatis`需要引入Mybatis相关配置类，使用`@Import`
            - 第三方数据源配置类 `JdbcConfig`
            - 构建DataSource数据源，DruidDataSouroce，需要注入数据库连接四要素，使用`@Bean`、`@Value`
            - 构建平台事务管理器，DataSourceTransactionManager，使用`@Bean`
            - Mybatis配置类 `MybatisConfig`
            - 构建`SqlSessionFactoryBean`并设置别名扫描与数据源，使用`@Bean`
            - 构建`MapperScannerConfigurer`并设置DAO层的包扫描
    - `SpringMvcConfig`
        - 标识该类为配置类，使用`@Configuratio`
        - 扫描`Controller`所在的包，使用`@ComponentScan`
        - 开启SpringMVC注解支持，使用`@EnableWebMvc`

3. 功能模块(与具体的业务模块有关)
    - 创建数据库表
    - 根据数据库表创建对应的模型类
    - 通过Dao层完成数据库表的增删改查(接口+自动代理)
    - 编写`Service`层(Service接口+实现类)
        - `@Service`
        - `@Transactional`
        - 整合Junit对业务层进行单元测试
            - `@RunWith`
            - `@ContextConfiguration`
            - `@Test`
    - 编写`Controller`层
        - 接收请求 `@RequestMapping`、`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`
        - 接收数据 简单、POJO、嵌套POJO、集合、数组、JSON数据类型
            - `@RequestParam`
            - `@PathVariable`
            - `@RequestBody`
        - 转发业务层
            - `@Autowired`
        - 响应结果
            - `@ResponseBody`
### 1.2 整合配置

分析完毕之后，我们就一步步来完成我们的SSM整合

- `步骤一：`创建Maven的web项目
- `步骤二：`导入坐标
```xml
<dependencies>  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-webmvc</artifactId>  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-jdbc</artifactId>  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-test</artifactId>  
        <version>5.2.10.RELEASE</version>  
    </dependency>  
  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis</artifactId>  
        <version>3.5.6</version>  
    </dependency>  
  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis-spring</artifactId>  
        <version>1.3.0</version>  
    </dependency>  
  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>5.1.46</version>  
    </dependency>  
  
    <dependency>  
        <groupId>com.alibaba</groupId>  
        <artifactId>druid</artifactId>  
        <version>1.1.16</version>  
    </dependency>  
  
    <dependency>  
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.12</version>  
        <scope>test</scope>  
    </dependency>  
  
    <dependency>  
        <groupId>javax.servlet</groupId>  
        <artifactId>javax.servlet-api</artifactId>  
        <version>3.1.0</version>  
        <scope>provided</scope>  
    </dependency>  
  
    <dependency>  
        <groupId>com.fasterxml.jackson.core</groupId>  
        <artifactId>jackson-databind</artifactId>  
        <version>2.9.0</version>  
    </dependency>  
</dependencies>
```

- `步骤三：`创建项目包结构
    
    - `com.blog.config`目录存放的是相关的配置类
    - `com.blog.controller`编写的是Controller类
    - `com.blog.dao`存放的是Dao接口，因为使用的是Mapper接口代理方式，所以没有实现类包
    - `com.blog.service`存的是Service接口，`com.blog.service.impl`存放的是Service实现类
    - `resources`:存入的是配置文件，如Jdbc.properties
    - `webapp`:目录可以存放静态资源
    - `test/java`:存放的是测试类

- `步骤四：`创建jdbc.properties
```java
jdbc.driver=com.mysql.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:13306/ssm_db?useSSL=false  
jdbc.username=root  
jdbc.password=PASSWORD.
```

- `步骤五：`创建JdbcConfig配置类
```java
public class JdbcConfig {  
    @Value("${jdbc.driver}")  
    private String driver;  
    @Value("${jdbc.url}")  
    private String url;  
    @Value("${jdbc.username}")  
    private String username;  
    @Value("${jdbc.password}")  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(username);  
        dataSource.setPassword(password);  
        return dataSource;  
    }  
  
    @Bean  
    public PlatformTransactionManager platformTransactionManager(DataSource dataSource){  
        DataSourceTransactionManager ds = new DataSourceTransactionManager();  
        ds.setDataSource(dataSource);  
        return ds;  
    }  
}
```

- `步骤六：`创建MyBatisConfig配置类
```java
public class MyBatisConfig {  
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){  
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();  
        factoryBean.setDataSource(dataSource);  
        factoryBean.setTypeAliasesPackage("com.blog.domain");  
        return factoryBean;  
    }  
  
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer(){  
        MapperScannerConfigurer msc = new MapperScannerConfigurer();  
        msc.setBasePackage("com.blog.dao");  
        return msc;  
    }  
}
```

- `步骤七：`创建SpringConfig配置类
```java
@Configuration  
@ComponentScan("com.blog.service")  
@PropertySource("jdbc.properties")  
@EnableTransactionManagement  
@Import({JdbcConfig.class, MyBatisConfig.class})  
public class SpringConfig {  
}
```

- `步骤八：`创建SpringMvcConfig配置类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- `步骤九：`创建ServletContainersInitConfig配置类
```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[]{SpringConfig.class};  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    @Override  
    protected Filter[] getServletFilters() {  
        CharacterEncodingFilter filter = new CharacterEncodingFilter();  
        filter.setEncoding("utf-8");  
        return new Filter[]{filter};  
    }  
}
```
### 1.3 功能模块开发

- 需求：对表tbl_book进行新增、修改、删除、根据ID查询和查询所有

- `步骤一：`创建数据库及表
```sql
create database ssm_db;  
use ssm_db;  
create table tbl_book  
(  
    id          int primary key auto_increment,  
    type        varchar(20),  
    `name`      varchar(50),  
    description varchar(255)  
);  
  
insert into `tbl_book`(`id`, `type`, `name`, `description`)  
values (1, '计算机理论', 'Spring实战 第五版', 'Spring入门经典教程，深入理解Spring原理技术内幕'),  
       (2, '计算机理论', 'Spring 5核心原理与30个类手写实践', '十年沉淀之作，手写Spring精华思想'),  
       (3, '计算机理论', 'Spring 5设计模式', '深入Spring源码刨析Spring源码中蕴含的10大设计模式'),  
       (4, '计算机理论', 'Spring MVC+Mybatis开发从入门到项目实战',  
        '全方位解析面向Web应用的轻量级框架，带你成为Spring MVC开发高手'),  
       (5, '计算机理论', '轻量级Java Web企业应用实战', '源码级刨析Spring框架，适合已掌握Java基础的读者'),  
       (6, '计算机理论', 'Java核心技术 卷Ⅰ 基础知识(原书第11版)',  
        'Core Java第11版，Jolt大奖获奖作品，针对Java SE9、10、11全面更新'),  
       (7, '计算机理论', '深入理解Java虚拟机', '5个纬度全面刨析JVM,大厂面试知识点全覆盖'),  
       (8, '计算机理论', 'Java编程思想(第4版)', 'Java学习必读经典，殿堂级著作！赢得了全球程序员的广泛赞誉'),  
       (9, '计算机理论', '零基础学Java(全彩版)', '零基础自学编程的入门图书，由浅入深，详解Java语言的编程思想和核心技术'),  
       (10, '市场营销', '直播就这么做:主播高效沟通实战指南', '李子柒、李佳奇、薇娅成长为网红的秘密都在书中'),  
       (11, '市场营销', '直播销讲实战一本通', '和秋叶一起学系列网络营销书籍'),  
       (12, '市场营销', '直播带货:淘宝、天猫直播从新手到高手', '一本教你如何玩转直播的书，10堂课轻松实现带货月入3W+');
```

- `步骤二：`编写模型类
```java
public class Book {  
    private Integer id;  
    private String type;  
    private String name;  
    private String description;  
      
    public Integer getId() {  
        return id;  
    }  
  
    public void setId(Integer id) {  
        this.id = id;  
    }  
  
    public String getType() {  
        return type;  
    }  
  
    public void setType(String type) {  
        this.type = type;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public String getDescription() {  
        return description;  
    }  
  
    public void setDescription(String description) {  
        this.description = description;  
    }  
  
    @Override  
    public String toString() {  
        return "Book{" +  
                "id=" + id +  
                ", type='" + type + '\'' +  
                ", name='" + name + '\'' +  
                ", description='" + description + '\'' +  
                '}';  
    }  
}
```

- `步骤三：`编写Dao接口
```java
public interface BookDao {  
    @Insert("insert into tbl_book values (null, #{type}, #{name}, #{description})")  
    void save(Book book);  
  
    @Update("update tbl_book set type=#{type}, `name`=#{name}, `description`=#{description} where id=#{id}")  
    void update(Book book);  
  
    @Delete("delete from tbl_book where id=#{id}")  
    void delete(Integer id);  
  
    @Select("select * from tbl_book where id=#{id}")  
    void getById(Integer id);  
  
    @Select("select * from tbl_book")  
    void getAll();  
}
```

- `步骤四：`编写Service接口及其实现类
```java
@Transactional  
public interface BookService {  
    /**  
     * 保存  
     * @param book  
     * @return  
     */  
    boolean save(Book book);  
  
    /**  
     * 修改  
     * @param book  
     * @return  
     */  
    boolean update(Book book);  
  
    /**  
     * 按id删除  
     * @param id  
     * @return  
     */  
    boolean delete(Integer id);  
  
    /**  
     * 按id查询  
     * @param id  
     * @return  
     */  
    Book getById(Integer id);  
  
    /**  
     * 查询所有  
     * @return  
     */  
    List<Book> getAll();  
}

@Service  
public class BookServiceImpl implements BookService {  
  
    @Autowired  
    private BookDao bookDao;  
  
    public boolean save(Book book) {  
        bookDao.save(book);  
        return true;  
    }  
  
    public boolean update(Book book) {  
        bookDao.update(book);  
        return true;  
    }  
  
    public boolean delete(@PathVariable Integer id) {  
        bookDao.delete(id);  
        return true;  
    }  
  
    public Book getById(Integer id) {  
        return bookDao.getById(id);  
    }  
  
    public List<Book> getAll() {  
        return bookDao.getAll();  
    }  
}
```

- `步骤五：`编写Controller类
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private BookService bookService;  
  
    @PostMapping  
    public boolean save(@RequestBody Book book) {  
        return bookService.save(book);  
    }  
  
    @PutMapping  
    public boolean update(@RequestBody Book book) {  
        return bookService.update(book);  
    }  
  
    @DeleteMapping("/{id}")  
    public boolean delete(@PathVariable Integer id) {  
        return bookService.delete(id);  
    }  
  
    @GetMapping("/{id}")  
    public Book getById(@PathVariable Integer id) {  
        return bookService.getById(id);  
    }  
  
    @GetMapping  
    public List<Book> getAll() {  
        return bookService.getAll();  
    }  
}
```
### 1.4 单元测试

- `步骤一：`新建测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class BookServiceTest {  
}
```

- `步骤二：`注入Servic
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class BookServiceTest {  
    @Autowired  
    private BookService bookService;  
}
```

- `步骤三：`编写测试方法
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class BookServiceTest {  
  
    @Autowired  
    private BookService bookService;  
  
    @Test  
    public void testGetById() {  
        Book book = bookService.getById(1);  
        System.out.println(book);  
    }  
  
    @Test  
    public void testGetAll() {  
        for (Book book : bookService.getAll()) {  
            System.out.println(book);  
        }  
    }  
}
```
### 1.5 PostMan测试

- `新增`  
	发送`POST`请求与数据，访问`localhost:8080/books`
```json
{  
	"type":"类别测试数据",  
    "name":"书名测试数据",  
    "description":"描述测试数据"  
}
```
数据库中能看到新增的数据
    
- `修改`  
    发送`PUT`请求与数据，访问`localhost:8080/books`
```json
{  
    "id":"13",  
	"type":"类别测试数据",  
    "name":"书名测试数据9527",  
    "description":"描述测试数据"  
}
```
数据库中能看到修改后的数据
    
- `删除`  
    发送DELETE请求，访问`localhost:8080/books/13`  
    数据库中能看到id为13的数据不见了
    
- `查询单个`  
    发送`GET`请求，访问`localhost:8080/books/1`  
    可以查询到ID为1的数据，在PostMan中表现为JSON数据
```json
{  
    "id": 1,  
    "type": "计算机理论",  
    "name": "Spring实战 第五版",  
    "description": "Spring入门经典教程，深入理解Spring原理技术内幕"  
}
```

- `查询所有`  
    发送`GET`请求，访问`localhost:8080/books`  
    PostMan中以JSON对象数组的形式显示了数据库中的所有数据

## 2 统一结果封装

### 2.1 表现层与前端数据传输协议定义

SSM整合以及功能模块开发完成后，接下来我们在上述案例的基础上，分析一下有哪些问题需要我们解决。  

首先第一个问题是：
- 在Controller层增删改操作完成后，返回给前端的是boolean类型的数据
- 在Controller层查询单个，返回给前端的是对象
```json
{  
    "id": 1,  
    "type": "计算机理论",  
    "name": "Spring实战 第五版",  
    "description": "Spring入门经典教程，深入理解Spring原理技术内幕"  
}
```
- 在Controller层查询所有，返回给前端的是集合对象
```json
[  
    {  
        "id": 1,  
        "type": "计算机理论",  
        "name": "Spring实战 第五版",  
        "description": "Spring入门经典教程，深入理解Spring原理技术内幕"  
    },  
    {  
        "id": 2,  
        "type": "计算机理论",  
        "name": "Spring 5核心原理与30个类手写实践",  
        "description": "十年沉淀之作，手写Spring精华思想"  
    },  
    ...  
]
```

- 目前我们就已经有`三种数据类型`返回给前端了，随着业务的增长，我们需要返回的数据类型就会`越来越多`。那么前端开发人员在解析数据的时候就比较`凌乱`了，所以对于前端来说，如果后端能返回一个`统一的数据结果`，前端在解析的时候就可以按照一种方式进行解析，开发就会变得更加简单
    
- 所以现在我们需要解决的问题就是`如何将返回的结果数据进行统一`，具体如何来做，大体思路如下
    
    - 为了`封装返回的结果数据`：创建结果模型类，封装数据到`data`属性中
        - 我们可以设置data的数据类型为`Object`，这样data中就可以放任意的结果类型了，包括但不限于上面的`boolean`、`对象`、`集合对象`
    - 为了封装返回的数据是`何种操作`，以及是否`操作成功`：封装操作结果到`code`属性中
        - 例如增删改操作返回的都是`true`，那我们怎么分辨这个`true`到底是`增`还是`删`还是`改`呢？我们就通过这个`code`来区分
    - 操作失败后，需要封装`返回错误信息`提示给用户：封装特殊消息到message(msg)属性中
        - 例如查询或删除的目标不存在，会返回null，那么此时我们需要提示`查询/删除的目标不存在，请重试！`

- 那么之前的三种返回方式就可以变为如下形式
- `boolean`
	code的规则可以自己定  
	这里前三位是固定的  
	第四位表示不同的操作  
	末位表示成功/失败，1成功，0失败
```json
{  
    "code":20011,  
    "data":true  
}
```
- `对象`
	这里的末尾是0，表示失败操作  
	第四位是2，区别于上面的1，表示是不同的操作类型  
	msg给用户提示信息，不是必有项
```json
{  
    "code":20020,  
    "data":null,  
    "msg":"数据查询失败，请重试！"  
}
```
- `对象集合`
	这里末尾操作是1，表示成功操作  
	data中显示的是对象集合  
	没有msg
```json
{  
    "code":20031,  
    "data":[  
        {  
            "id":1,  
            "type":"计算机理论",  
            "name":"Spring实战 第5版",  
            "description":"Spring入门经典教程"  
        },  
        {  
            "id":2,  
            "type":"计算机理论",  
            "name":"Spring 5核心原理与30个类手写实战",  
            "description":"十年沉淀之作"  
        }  
    ]  
}
```

- 根据分析，我们可以设置统一数据返回结果类
```java
public class Result{  
	private Object data;  
	private Integer code;  
	private String msg;  
}
```

- 注意：Result类名及类中的字段并不是固定的，可以根据需要自行增减提供若干个构造方法，方便操作。
### 2.2 表现层与前端数据传输协议实现

前面我们已经分析了如何封装返回结果数据，现在我们来具体实现一下

对于结果封装，我们应该是在表现层进行处理，所以我们把结果类放在controller包下，当然你也可以放在domain包，这个都是可以的，具体如何实现结果封装，具体的步骤如下

- `步骤一：`创建Result类
```java
public class Result {  
    //描述统一格式中的编码，用于区分操作，可以简化配置0或1表示成功失败  
    private Integer code;  
    //描述统一格式中的数据  
    private Object data;  
    //描述统一格式中的消息，可选属性  
    private String msg;  
  
    public Result() {  
    }  
  
    //构造器可以根据自己的需要来编写  
    public Result(Integer code, Object data) {  
        this.code = code;  
        this.data = data;  
    }  
  
    public Result(Integer code, Object data, String msg) {  
        this.code = code;  
        this.data = data;  
        this.msg = msg;  
    }  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public Object getData() {  
        return data;  
    }  
  
    public void setData(Object data) {  
        this.data = data;  
    }  
  
    public String getMsg() {  
        return msg;  
    }  
  
    public void setMsg(String msg) {  
        this.msg = msg;  
    }  
  
    @Override  
    public String toString() {  
        return "Result{" +  
                "code=" + code +  
                ", data=" + data +  
                ", msg='" + msg + '\'' +  
                '}';  
    }  
}
```

- `步骤二：`定义返回码Code类
```java
public class Code {  
    public static final Integer SAVE_OK = 20011;  
    public static final Integer UPDATE_OK = 20021;  
    public static final Integer DELETE_OK = 20031;  
    public static final Integer GET_OK = 20041;  
      
    public static final Integer SAVE_ERR = 20010;  
    public static final Integer UPDATE_ERR = 20020;  
    public static final Integer DELETE_ERR = 20030;  
    public static final Integer GET_ERR = 20040;  
}
```

- 注意：code类中的常量设计也不是固定的，可以根据需要自行增减，例如将查询再进行细分为`GET_OK`，`GET_ALL_OK`，`GET_PAGE_OK`等。

- `步骤三：`修改Controller类的返回值
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
    @Autowired  
    private BookService bookService;  
  
    @PostMapping  
    public Result save(@RequestBody Book book) {  
        boolean flag = bookService.save(book);  
        return new Result(flag ? Code.SAVE_OK : Code.SAVE_ERR, flag);  
    }  
  
    @PutMapping  
    public Result update(@RequestBody Book book) {  
        boolean flag = bookService.update(book);  
        return new Result(flag ? Code.UPDATE_OK : Code.UPDATE_ERR, flag);  
    }  
  
    @DeleteMapping("/{id}")  
    public Result delete(@PathVariable Integer id) {  
        boolean flag = bookService.delete(id);  
        return new Result(flag ? Code.DELETE_OK : Code.DELETE_ERR, flag);  
    }  
  
    @GetMapping("/{id}")  
    public Result getById(@PathVariable Integer id) {  
        Book book = bookService.getById(id);  
        Integer code = book == null ? Code.GET_ERR : Code.GET_OK;  
        String msg = book == null ? "数据查询失败，请重试！" : "";  
        return new Result(code, book, msg);  
    }  
  
    @GetMapping  
    public Result getAll() {  
        List<Book> bookList = bookService.getAll();  
        Integer code = bookList == null ? Code.GET_ERR : Code.GET_OK;  
        String msg = bookList == null ? "数据查询失败，请重试！" : "";  
        return new Result(code, bookList, msg);  
    }  
}
```

- `步骤四：`启动服务测试  
## 3 统一异常处理

### 3.1 问题描述

在学习这部分知识之前，我们先来演示一个效果，修改`BookController`的`getById()`方法，手写一个异常

```java
@GetMapping("/{id}")  
public Result getById(@PathVariable Integer id) {  
    //当id为1的时候，手动添加了一个错误信息  
    if (id == 1){  
        int a = 1 / 0;  
    }  
    Book book = bookService.getById(id);  
    Integer code = book == null ? Code.GET_ERR : Code.GET_OK;  
    String msg = book == null ? "数据查询失败，请重试！" : "";  
    return new Result(code, book, msg);  
}
```

重新启动服务器，使用`PostMan`发送请求，当传入的`id为1`时，会出现如下效果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14005fd315cdf3b79264df920391927d.png)


前端接收到这个信息后，和我们之前约定的格式不一致，怎么解决呢？  
在解决问题之前，我们先来看一下异常的种类，以及出现异常的原因：

- 框架内部抛出的异常：因`使用不合规`导致
- 数据层抛出的异常：因使用`外部服务器故障`导致（例如：服务器访问超时）
- 业务层抛出的异常：因`业务逻辑书写错误`导致（例如：遍历业务书写操作，导致索引越界异常等）
- 表现层抛出的异常：因`数据收集`、`校验`等规则导致（例如：不匹配的数据类型间转换导致异常）
- 工具类抛出的异常：因工具类`书写不严谨`，`健壮性不足`导致（例如：必要时放的连接，长时间未释放等）

了解完上面这些出现`异常的位置`，我们发现，在我们开发的`任何一个位置`都可能会出现异常，而且这些异常是`不能避免的`，所以我们就需要对这些异常来`进行处理`。

`思考`

1. 各个层级均出现异常，那么异常处理代码要写在哪一层？
    - 所有的异常均抛出到表现层进行处理
2. 异常的种类很多，表现层如何将所有的异常都处理到呢？
    - 异常分类
3. 表现层处理异常，每个方法中单独书写，代码书写两巨大，且意义不强，如何解决呢？
    - AOP

对于上面这些问题以及解决方案，SpringMVC已经为我们提供了一套了：

- 异常处理器：
    - 集中的、统一的处理项目中出现的异常
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
    @ExceptionHandler(Exception.class)  
    public Result doException(Exception ex) {  
        return new Result(666, null);  
    }  
}
```
### 3.2 异常处理器的使用

- `步骤一：`创建异常处理器类
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
    @ExceptionHandler(Exception.class)  
    public void doException(Exception ex) {  
        System.out.println("嘿嘿，逮到一个异常~");  
    }  
}
```

- 注意：要`确保SpringMvcConfig能够扫描到异常处理器类`

- `步骤二：`让程序抛出异常
```java
@GetMapping("/{id}")  
public Result getById(@PathVariable Integer id) {  
    //当id为1的时候，手动添加了一个错误信息  
    if (id == 1){  
        int a = 1 / 0;  
    }  
    Book book = bookService.getById(id);  
    Integer code = book == null ? Code.GET_ERR : Code.GET_OK;  
    String msg = book == null ? "数据查询失败，请重试！" : "";  
    return new Result(code, book, msg);  
}
```

- `步骤三：`使用`PostMan`发送`GET`请求访问`localhost:8080/books/1`  
	控制台输出如下，说明异常已经被拦截，且执行了`doException()`方法![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/89ac5fc2b427822503ef42e60ae920f7.png)


但是现在没有返回数据给前端，为了统一返回结果，我们继续修改异常处理器类
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
    @ExceptionHandler(Exception.class)  
    public Result doException(Exception ex) {  
        System.out.println("嘿嘿，逮到一个异常~");  
        return new Result(666, null, "嘿嘿，逮到一个异常~");  
    }  
}
```

重启服务器，使用PostMan发送请求，此时就能接收到结果了
```json
{  
    "code": 666,  
    "data": null,  
    "msg": "嘿嘿，逮到一个异常~"  
}
```

知识点：`@RestControllerAdvice`  

说明：此注解自带`@ResponseBody`注解与`@Component`注解，具备对应的功能

|名称|@RestControllerAdvice|
|---|---|
|类型|类注解|
|位置|Rest风格开发的控制器增强类定义上方|
|作用|为Rest风格开发的控制器类做增强|

知识点：`@ExceptionHandler`  

说明：此类方法可以根据处理的异常不同，制作多个方法分别处理对应的异常

|名称|@ExceptionHandler|
|---|---|
|类型|方法注解|
|位置|专用于异常处理的控制器方法上方|
|作用|设置指定异常的处理方案，功能等同于控制器方法，  <br>出现异常后终止原始控制器执行,并转入当前方法执行|
### 3.3 项目异常处理方案

异常处理器我们已经能够使用了，那么我们如何在项目中来处理异常呢?
#### 3.3.1 异常分类

因为异常的种类有很多，如果每一个异常都对应一个`@ExceptionHandler`，那得写多少个方法来处理各自的异常，所以我们在处理异常之前，需要对异常进行一个分类:

- `业务异常`（BusinessException）
    - 规范的用户行为产生的异常
        - 用户在页面输入内容的时候未按照指定格式进行数据填写，如在年龄框输入的是字符串
    - 不规范的用户行为操作产生的异常
        - 如用户手改URL，故意传递错误数据`localhost:8080/books/略略略`
- `系统异常`（SystemException）
    - 项目运行过程中可预计，但无法避免的异常
        - 如服务器宕机
- `其他异常`（Exception）
    - 编程人员未预期到的异常
        - 如：系统找不到指定文件

将异常分类以后，针对不同类型的异常，要提供具体的解决方案
#### 3.3.2 异常解决方案

- `业务异常`（BusinessException）
    - 发送对应消息传递给用户，提醒规范操作
        - 大家常见的就是提示用户名已存在或密码格式不正确等
- `系统异常`（SystemException）
    - 发送固定消息传递给用户，安抚用户
        - 系统繁忙，请稍后再试
        - 系统正在维护升级，请稍后再试
        - 系统出问题，请联系系统管理员等
    - 发送特定消息给运维人员，提醒维护
        - 可以发送短信、邮箱或者是公司内部通信软件
    - 记录日志
        - 发消息给运维和记录日志对用户来说是不可见的，属于后台程序
- `其他异常`（Exception）
    - 发送固定消息传递给用户，安抚用户
    - 发送特定消息给编程人员，提醒维护（纳入预期范围内）
        - 一般是程序没有考虑全，比如未做非空校验等
    - 记录日志
#### 3.3.3 具体实现

- 思路:
    1. 先通过自定义异常，完成BusinessException和SystemException的定义
    2. 将其他异常包装成自定义异常类型
    3. 在异常处理器类中对不同的异常进行处理

- `步骤一：`自定义异常类
```java
public class SystemException extends RuntimeException {  
    private Integer code;  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public SystemException() {  
    }  
  
    public SystemException(Integer code) {  
        this.code = code;  
  
    }  
  
    public SystemException(Integer code, String message) {  
        super(message);  
        this.code = code;  
    }  
  
    public SystemException(Integer code, String message, Throwable cause) {  
        super(message, cause);  
        this.code = code;  
    }  
}


public class BusinessException extends RuntimeException{  
    private Integer code;  
  
    public Integer getCode() {  
        return code;  
    }  
  
    public void setCode(Integer code) {  
        this.code = code;  
    }  
  
    public BusinessException() {  
    }  
  
    public BusinessException(Integer code) {  
        this.code = code;  
  
    }  
  
    public BusinessException(Integer code, String message) {  
        super(message);  
        this.code = code;  
    }  
  
    public BusinessException(Integer code, String message, Throwable cause) {  
        super(message, cause);  
        this.code = code;  
    }  
}
```
说明：
- 让自定义异常类继承`RuntimeException`的好处是，后期在抛出这两个异常的时候，就不用在`try..catch..`或`throws`了
- 自定义异常类中添加`code`属性的原因是为了更好的区分异常是来自哪个业务的

- `步骤二：`将其他异常包成自定义异常  
	假如在`BookServiceImpl`的`getById`方法抛异常了，该如何来包装呢?
具体的包装方式有：
- 方式一：`try{}catch(){}`在catch中重新throw我们自定义异常即可。
- 方式二：直接`throw`自定义异常即可
```java
public Book getById(Integer id) {  
    //模拟业务异常，包装成自定义异常  
    if(id == 1){  
        throw new BusinessException(Code.BUSINESS_ERR,"你别给我乱改URL噢");  
    }  
    //模拟系统异常，将可能出现的异常进行包装，转换成自定义异常  
    try{  
        int i = 1/0;  
    }catch (Exception e){  
        throw new SystemException(Code.SYSTEM_TIMEOUT_ERR,"服务器访问超时，请重试!",e);  
    }  
    return bookDao.getById(id);  
}
```

上面为了使`code`看着更专业些，我们在Code类中再新增需要的属性
```java
public class Code {  
    public static final Integer SAVE_OK = 20011;  
    public static final Integer UPDATE_OK = 20021;  
    public static final Integer DELETE_OK = 20031;  
    public static final Integer GET_OK = 20041;  
  
    public static final Integer SAVE_ERR = 20010;  
    public static final Integer UPDATE_ERR = 20020;  
    public static final Integer DELETE_ERR = 20030;  
    public static final Integer GET_ERR = 20040;  
  
    public static final Integer SYSTEM_ERR = 50001;  
    public static final Integer SYSTEM_TIMEOUT_ERR = 50002;  
    public static final Integer SYSTEM_UNKNOW_ERR = 59999;  
  
    public static final Integer BUSINESS_ERR = 60001;  
}
```

- `步骤三：`处理器类中处理自定义异常
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
    @ExceptionHandler(SystemException.class)  
    public Result doSystemException(SystemException ex) {  
        return new Result(ex.getCode(), null, ex.getMessage());  
    }  
  
    @ExceptionHandler(BusinessException.class)  
    public Result doBusinessException(BusinessException ex) {  
        return new Result(ex.getCode(), null, ex.getMessage());  
    }  
  
    @ExceptionHandler(Exception.class)  
    public Result doException(Exception ex) {  
        return new Result(Code.SYSTEM_UNKNOW_ERR, null, "系统繁忙，请稍后再试！");  
    }  
}
```

- `步骤四：`运行程序  
	根据ID查询，如果传入的参数为1，会报`BusinessException`，错误信息应为`你别给我乱改URL噢`
```json
{  
    "code": 60001,  
    "data": null,  
    "msg": "你别给我乱改URL噢"  
}
```

如果传入的是其他参数，会报`SystemException`，错误信息应为`服务器访问超时，请重试!`
```json
{  
    "code": 50002,  
    "data": null,  
    "msg": "服务器访问超时，请重试!"  
}
```

那么对于异常我们就已经处理完成了，不管后台哪一层抛出异常，都会以我们与前端约定好的方式进行返回，前端只需要把信息获取到，根据返回的正确与否来展示不同的内容即可。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ff84d487d09abfa0f6b0b033b6ab61c.png)


## 4 前后台协议联调

### 4.1 环境准备

- 导入提供好的前端页面，如果想自己写页面，也可以用element-ui，有空了考虑考虑
- 由于添加了静态资源，SpringMVC会拦截，所以需要在对静态资源放行
    - 新建SpringMVCSupport类，继承`WebMvcConfigurationSupport`，并重写`addResourceHandlers()`方法
```java
@Configuration  
public class SpringMvcSupport extends WebMvcConfigurationSupport {  
    @Override  
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {  
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");  
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");  
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");  
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");  
    }  
}
```

- 同时也需要让SpringMvcConfig扫描到我们的配置类
```java
@Configuration  
@ComponentScan({"com.blog.controller","com.blog.config"})  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

现在我们打开浏览器，访问`http://localhost:8080/pages/books.html`，应该是可以正常看到页面的
### 4.2 页面分析

在完成增删改查操作之前，我们先来看看给我们提供的页面源码

- `源码`
```html
<!DOCTYPE html>  
  
<html>  
<head>  
    <!-- 页面meta -->  
    <meta charset="utf-8">  
    <title>SpringMVC案例</title>  
    <!-- 引入样式 -->  
    <link rel="stylesheet" href="../plugins/elementui/index.css">  
    <link rel="stylesheet" href="../plugins/font-awesome/css/font-awesome.min.css">  
    <link rel="stylesheet" href="../css/style.css">  
</head>  
  
<body class="hold-transition">  
  
<div id="app">  
  
    <div class="content-header">  
        <h1>图书管理</h1>  
    </div>  
  
    <div class="app-container">  
        <div class="box">  
            <div class="filter-container">  
                <el-input placeholder="图书名称" style="width: 200px;" class="filter-item"></el-input>  
                <el-button class="dalfBut">查询</el-button>  
                <el-button type="primary" class="butT" @click="openSave()">新建</el-button>  
            </div>  
  
            <el-table size="small" current-row-key="id" :data="dataList" stripe highlight-current-row>  
                <el-table-column type="index" align="center" label="序号"></el-table-column>  
                <el-table-column prop="type" label="图书类别" align="center"></el-table-column>  
                <el-table-column prop="name" label="图书名称" align="center"></el-table-column>  
                <el-table-column prop="description" label="描述" align="center"></el-table-column>  
                <el-table-column label="操作" align="center">  
                    <template slot-scope="scope">  
                        <el-button type="primary" size="mini">编辑</el-button>  
                        <el-button size="mini" type="danger">删除</el-button>  
                    </template>  
                </el-table-column>  
            </el-table>  
  
            <div class="pagination-container">  
                <el-pagination  
                        class="pagiantion"  
                        @current-change="handleCurrentChange"  
                        :current-page="pagination.currentPage"  
                        :page-size="pagination.pageSize"  
                        layout="total, prev, pager, next, jumper"  
                        :total="pagination.total">  
                </el-pagination>  
            </div>  
  
            <!-- 新增标签弹层 -->  
            <div class="add-form">  
                <el-dialog title="新增图书" :visible.sync="dialogFormVisible">  
                    <el-form ref="dataAddForm" :model="formData" :rules="rules" label-position="right"  
                             label-width="100px">  
                        <el-row>  
                            <el-col :span="12">  
                                <el-form-item label="图书类别" prop="type">  
                                    <el-input v-model="formData.type"/>  
                                </el-form-item>  
                            </el-col>  
                            <el-col :span="12">  
                                <el-form-item label="图书名称" prop="name">  
                                    <el-input v-model="formData.name"/>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                        <el-row>  
                            <el-col :span="24">  
                                <el-form-item label="描述">  
                                    <el-input v-model="formData.description" type="textarea"></el-input>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                    </el-form>  
                    <div slot="footer" class="dialog-footer">  
                        <el-button @click="dialogFormVisible = false">取消</el-button>  
                        <el-button type="primary" @click="saveBook()">确定</el-button>  
                    </div>  
                </el-dialog>  
            </div>  
  
        </div>  
    </div>  
</div>  
</body>  
  
<!-- 引入组件库 -->  
<script src="../js/vue.js"></script>  
<script src="../plugins/elementui/index.js"></script>  
<script type="text/javascript" src="../js/jquery.min.js"></script>  
<script src="../js/axios-0.18.0.js"></script>  
  
<script>  
    var vue = new Vue({  
  
        el: '#app',  
  
        data: {  
            dataList: [],//当前页要展示的分页列表数据  
            formData: {},//表单数据  
            dialogFormVisible: false,//增加表单是否可见  
            dialogFormVisible4Edit: false,//编辑表单是否可见  
            pagination: {},//分页模型数据，暂时弃用  
        },  
  
        //钩子函数，VUE对象初始化完成后自动执行  
        created() {  
  
        },  
  
        methods: {  
            // 重置表单  
            resetForm() {  
  
            },  
  
            // 弹出添加窗口  
            openSave() {  
  
            },  
  
            //添加  
            saveBook() {  
  
            },  
  
            //主页列表查询  
            getAll() {  
  
            },  
  
        }  
    })  
</script>  
</html>
```

- `页头部分`
	这部分有一个`input输入框`，一个`查询按钮`和一个`新建按钮`  
	由于本案例没有查询业务，故未给查询按钮绑定事件  
	而新建按钮则绑定了一个`openSave()`函数，后面我们会分析
```html
<div class="filter-container">  
    <el-input placeholder="图书名称" style="width: 200px;" class="filter-item"></el-input>  
    <el-button class="dalfBut">查询</el-button>  
    <el-button type="primary" class="butT" @click="openSave()">新建</el-button>  
</div>
```

- `表格部分`
	表格绑定的Model是`dataList`  
	id不需要在页面显示，图书`类型`/`名称`/`描述`分别绑定了`type`/`name`/`description`属性  
	编辑和删除按钮暂时还没有绑定点击事件，待会儿我们自己来做
```html
<el-table size="small" current-row-key="id" :data="dataList" stripe highlight-current-row>  
    <el-table-column type="index" align="center" label="序号"></el-table-column>  
    <el-table-column prop="type" label="图书类别" align="center"></el-table-column>  
    <el-table-column prop="name" label="图书名称" align="center"></el-table-column>  
    <el-table-column prop="description" label="描述" align="center"></el-table-column>  
    <el-table-column label="操作" align="center">  
        <template slot-scope="scope">  
            <el-button type="primary" size="mini">编辑</el-button>  
            <el-button size="mini" type="danger">删除</el-button>  
        </template>  
    </el-table-column>  
</el-table>
```

- `分页`
	案例中没有涉及到分页的相关内容，这里就不分析了
```html
<div class="pagination-container">  
    <el-pagination  
        class="pagiantion"  
        @current-change="handleCurrentChange"  
        :current-page="pagination.currentPage"  
        :page-size="pagination.pageSize"  
        layout="total, prev, pager, next, jumper"  
        :total="pagination.total">  
    </el-pagination>  
</div>
```

- `新增按钮弹出框`
	当点击新增按钮时的对话框，在JavaWeb课程的最后项目中已经用过了，  
	第二行的`dialogFormVisible`控制该dialog是否显示  
	第三行的`:model="formData"`表示该对话框绑定模型的是`formData`  
	最后还有`取消`和`确定`两个按钮，  
	取消按钮绑定点击事件为`dialogFormVisible = false`，也就是关闭对话框  
	确定按钮绑定的是`saveBook()`，见名知意，当我们点击确定按钮时，图书就执行新增函数
```html
<!-- 新增标签弹层 -->  
<div class="add-form">  
    <!--dialogFormVisible属性控制dialog是否显示-->  
    <el-dialog title="新增图书" :visible.sync="dialogFormVisible">  
        <!--绑定的模型是fromData-->  
        <el-form ref="dataAddForm" :model="formData" :rules="rules" label-position="right"  
                    label-width="100px">  
            <el-row>  
                <el-col :span="12">  
                    <el-form-item label="图书类别" prop="type">  
                        <el-input v-model="formData.type"/>  
                    </el-form-item>  
                </el-col>  
                <el-col :span="12">  
                    <el-form-item label="图书名称" prop="name">  
                        <el-input v-model="formData.name"/>  
                    </el-form-item>  
                </el-col>  
            </el-row>  
            <el-row>  
                <el-col :span="24">  
                    <el-form-item label="描述">  
                        <el-input v-model="formData.description" type="textarea"></el-input>  
                    </el-form-item>  
                </el-col>  
            </el-row>  
        </el-form>  
        <div slot="footer" class="dialog-footer">  
            <el-button @click="dialogFormVisible = false">取消</el-button>  
            <el-button type="primary" @click="saveBook()">确定</el-button>  
        </div>  
    </el-dialog>  
</div>
```

- `data`
	下面我们来看看data区都提供了什么模型  
	`dataList`：我们主体的表格就绑定的是它  
	`formData`：新增按钮弹出的对话框绑定的就是它  
	`dialogFormVisible`：控制增加表单是否课件  
	`dialogFormVisible4Edit`：控制编辑表单是否可见
```js
data: {  
    dataList: [],//当前页要展示的分页列表数据  
    formData: {},//表单数据  
    dialogFormVisible: false,//增加表单是否可见  
    dialogFormVisible4Edit: false,//编辑表单是否可见  
    pagination: {},//分页模型数据，暂时弃用  
}
```

- `钩子函数`
	不出意外的话，当VUE初始化之后，我们就该展示数据了，调用`getAll()`函数
```js
//钩子函数，VUE对象初始化完成后自动执行  
created() {  
  
}
```

- `methods`
	最后就是一些函数，目测是不够的，那我们自己根据需求再写点
	- 弹出编辑窗口
	- 编辑
	- 删除
```js
methods: {  
    // 重置表单  
    resetForm() {  
  
    },  
  
    // 弹出添加窗口  
    openSave() {  
  
    },  
  
    //添加  
    saveBook() {  
  
    },  
  
    //主页列表查询  
    getAll() {  
  
    },  
}
```

- `DIY`
	由于给我们准备好的前端页面有点问题，所以现在我们需要自己来改造  
	首先`编辑`和`删除`按钮，我们需要为其绑定事件
```html
<el-button type="primary" size="mini" @click="openEdit(scope.row)">编辑</el-button>  
<el-button size="mini" type="danger" @click="deleteBook(scope.row)">删除</el-button>
```

那么编辑对应的表单也需要我们来准备，照着新增表单的改改就得了，确定按钮绑定了一个`handleEdit()`函数
```html
<!-- 编辑标签弹层 -->  
<div class="add-form">  
    <el-dialog title="编辑检查项" :visible.sync="dialogFormVisible4Edit">  
        <el-form ref="dataEditForm" :model="formData" :rules="rules" label-position="right" label-width="100px">  
            <el-row>  
                <el-col :span="12">  
                    <el-form-item label="图书类别" prop="type">  
                        <el-input v-model="formData.type"/>  
                    </el-form-item>  
                </el-col>  
                <el-col :span="12">  
                    <el-form-item label="图书名称" prop="name">  
                        <el-input v-model="formData.name"/>  
                    </el-form-item>  
                </el-col>  
            </el-row>  
            <el-row>  
                <el-col :span="24">  
                    <el-form-item label="描述">  
                        <el-input v-model="formData.description" type="textarea"></el-input>  
                    </el-form-item>  
                </el-col>  
            </el-row>  
        </el-form>  
        <div slot="footer" class="dialog-footer">  
            <el-button @click="dialogFormVisible4Edit = false">取消</el-button>  
            <el-button type="primary" @click="handleEdit()">确定</el-button>  
        </div>  
    </el-dialog>  
</div>
```

- `最终页面`
```html
<!DOCTYPE html>  
  
<html>  
<head>  
    <!-- 页面meta -->  
    <meta charset="utf-8">  
    <title>SpringMVC案例</title>  
    <!-- 引入样式 -->  
    <link rel="stylesheet" href="../plugins/elementui/index.css">  
    <link rel="stylesheet" href="../plugins/font-awesome/css/font-awesome.min.css">  
    <link rel="stylesheet" href="../css/style.css">  
</head>  
  
<body class="hold-transition">  
  
<div id="app">  
  
    <div class="content-header">  
        <h1>图书管理</h1>  
    </div>  
  
    <div class="app-container">  
        <div class="box">  
            <div class="filter-container">  
                <el-input placeholder="图书名称" style="width: 200px;" class="filter-item"></el-input>  
                <el-button class="dalfBut">查询</el-button>  
                <el-button type="primary" class="butT" @click="openSave()">新建</el-button>  
            </div>  
  
            <el-table size="small" current-row-key="id" :data="dataList" stripe highlight-current-row>  
                <el-table-column type="index" align="center" label="序号"></el-table-column>  
                <el-table-column prop="type" label="图书类别" align="center"></el-table-column>  
                <el-table-column prop="name" label="图书名称" align="center"></el-table-column>  
                <el-table-column prop="description" label="描述" align="center"></el-table-column>  
                <el-table-column label="操作" align="center">  
                    <template slot-scope="scope">  
                        <el-button type="primary" size="mini" @click="openEdit(scope.row)">编辑</el-button>  
                        <el-button size="mini" type="danger" @click="deleteBook(scope.row)">删除</el-button>  
                    </template>  
                </el-table-column>  
            </el-table>  
  
            <div class="pagination-container">  
                <el-pagination  
                        class="pagiantion"  
                        @current-change="handleCurrentChange"  
                        :current-page="pagination.currentPage"  
                        :page-size="pagination.pageSize"  
                        layout="total, prev, pager, next, jumper"  
                        :total="pagination.total">  
                </el-pagination>  
            </div>  
  
            <!-- 新增标签弹层 -->  
            <div class="add-form">  
                <el-dialog title="新增图书" :visible.sync="dialogFormVisible">  
                    <el-form ref="dataAddForm" :model="formData" :rules="rules" label-position="right"  
                             label-width="100px">  
                        <el-row>  
                            <el-col :span="12">  
                                <el-form-item label="图书类别" prop="type">  
                                    <el-input v-model="formData.type"/>  
                                </el-form-item>  
                            </el-col>  
                            <el-col :span="12">  
                                <el-form-item label="图书名称" prop="name">  
                                    <el-input v-model="formData.name"/>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                        <el-row>  
                            <el-col :span="24">  
                                <el-form-item label="描述">  
                                    <el-input v-model="formData.description" type="textarea"></el-input>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                    </el-form>  
                    <div slot="footer" class="dialog-footer">  
                        <el-button @click="dialogFormVisible = false">取消</el-button>  
                        <el-button type="primary" @click="saveBook()">确定</el-button>  
                    </div>  
                </el-dialog>  
            </div>  
  
        </div>  
  
        <!-- 编辑标签弹层 -->  
        <div class="add-form">  
            <el-dialog title="编辑检查项" :visible.sync="dialogFormVisible4Edit">  
                <el-form ref="dataEditForm" :model="formData" :rules="rules" label-position="right" label-width="100px">  
                    <el-row>  
                        <el-col :span="12">  
                            <el-form-item label="图书类别" prop="type">  
                                <el-input v-model="formData.type"/>  
                            </el-form-item>  
                        </el-col>  
                        <el-col :span="12">  
                            <el-form-item label="图书名称" prop="name">  
                                <el-input v-model="formData.name"/>  
                            </el-form-item>  
                        </el-col>  
                    </el-row>  
                    <el-row>  
                        <el-col :span="24">  
                            <el-form-item label="描述">  
                                <el-input v-model="formData.description" type="textarea"></el-input>  
                            </el-form-item>  
                        </el-col>  
                    </el-row>  
                </el-form>  
                <div slot="footer" class="dialog-footer">  
                    <el-button @click="dialogFormVisible4Edit = false">取消</el-button>  
                    <el-button type="primary" @click="handleEdit()">确定</el-button>  
                </div>  
            </el-dialog>  
        </div>  
    </div>  
</div>  
</div>  
</body>  
  
<!-- 引入组件库 -->  
<script src="../js/vue.js"></script>  
<script src="../plugins/elementui/index.js"></script>  
<script type="text/javascript" src="../js/jquery.min.js"></script>  
<script src="../js/axios-0.18.0.js"></script>  
  
<script>  
    var vue = new Vue({  
  
        el: '#app',  
  
        data: {  
            dataList: [],//当前页要展示的分页列表数据  
            formData: {},//表单数据  
            dialogFormVisible: false,//增加表单是否可见  
            dialogFormVisible4Edit: false,//编辑表单是否可见  
            pagination: {},//分页模型数据，暂时弃用  
        },  
  
        //钩子函数，VUE对象初始化完成后自动执行  
        created() {  
  
        },  
  
        methods: {  
            // 重置表单  
            resetForm() {  
  
            },  
  
            // 弹出添加窗口  
            openSave() {  
                  
            },  
  
            //添加  
            saveBook() {  
  
            },  
  
            //主页列表查询  
            getAll() {  
  
            },  
            openEdit(row){  
  
            },  
            deleteBook(row){  
  
            },  
            handleEdit(){  
  
            }  
        }  
    })  
</script>  
</html>
```
### 4.3 列表功能

需求：页面加载完后发送异步请求到后台获取列表数据进行展示

1. 找到页面的钩子函数，`created()`
2. `created()`方法中调用了`this.getAll()`方法
3. 在getAll()方法中使用axios发送异步请求从后台获取数据
4. 访问的路径为`http://localhost/books`
5. 返回数据

那么修改getAll()方法  

`res.data`表示获取的`Result`对象，而`Result`对象的`data`属性才是真正的数据  
也就是将`Rusult.data`赋给了`this.dataList`

```js
getAll() {  
    axios.get("/books").then((res)=>{  
        this.dataList = res.data.data;  
    })  
}
```

在钩子函数中直接调用`getAll()`即可
```js
created() {  
    this.getAll();  
}
```

那么现在重启服务器，打开浏览器访问`http://localhost:8080/pages/books.html`，表格中可以正常显示数据了
### 4.4 添加功能

需求：完成图片的新增功能模块

1. 找到页面上的`新建`按钮，按钮上绑定了`@click="openSave()"`方法
2. 在method中找到`openSave`方法，方法中打开新增面板
3. 新增面板中找到`确定`按钮,按钮上绑定了`@click="saveBook()"`方法
4. 在method中找到`saveBook`方法
5. 在方法中发送请求和数据，响应成功后将新增面板关闭并重新查询数据

- `openSave`打开新增面板
```js
openSave() {  
    this.dialogFormVisible = true;  
}
```

- `saveBook`方法发送异步请求并携带数据
```js
saveBook () {  
    //发送ajax请求  
    axios.post("/books",this.formData).then((res)=>{  
        this.dialogFormVisible = false;  
        this.getAll();  
    });  
}
```
### 4.5 添加功能状态处理

基础的新增功能已经完成，但是还有一些问题需要解决下:  

需求：新增成功是关闭面板，重新查询数据，那么新增失败以后该如何处理?

1. 在handlerAdd方法中根据后台返回的数据来进行不同的处理
2. 如果后台返回的是成功，则提示成功信息，并关闭面板
3. 如果后台返回的是失败，则提示错误信息

- 修改前端页面
```js
saveBook() {  
    axios.post("/books",this.formData).then((res)=>{  
        //20011是成功的状态码，成功之后就关闭对话框，并显示添加成功  
        if (res.data.code == 20011){  
            this.dialogFormVisible = false;  
            this.$message.success("添加成功")  
        //20010是失败的状态码，失败后给用户提示信息  
        }else if(res.data.code == 20010){  
            this.$message.error("添加失败");  
        //如果前两个都不满足，那就是SYSTEM_UNKNOW_ERR，未知异常了，显示未知异常的错误提示信息安抚用户情绪  
        }else {  
            this.$message.error(res.data.msg);  
        }  
    }).finally(()=>{  
        this.getAll();  
    })  
}
```

- 后台返回操作结果，将Dao层的增删改方法返回值从`void`改成`int`  
	如果添加失败，int值为0，添加成功则int值为显示受影响的行数
```java
public interface BookDao {  
    @Insert("insert into tbl_book values (null, #{type}, #{name}, #{description})")  
    int save(Book book);  
  
    @Update("update tbl_book set type=#{type}, `name`=#{name}, `description`=#{description} where id=#{id}")  
    int update(Book book);  
  
    @Delete("delete from tbl_book where id=#{id}")  
    int delete(Integer id);  
  
    @Select("select * from tbl_book where id=#{id}")  
    Book getById(Integer id);  
  
    @Select("select * from tbl_book")  
    List<Book> getAll();  
}
```

- 在BookServiceImpl中，增删改方法根据DAO的返回值来决定返回true/false  
	如果受影响的行大于0，则添加成功，否则添加失败
```java
@Service  
public class BookServiceImpl implements BookService {  
  
    @Autowired  
    private BookDao bookDao;  
  
    public boolean save(Book book) {  
        return bookDao.save(book) > 0;  
    }  
  
    public boolean update(Book book) {  
        return bookDao.update(book) > 0;  
    }  
  
    public boolean delete(Integer id) {  
       return bookDao.delete(id) > 0;  
    }  
  
    public Book getById(Integer id) {  
        return bookDao.getById(id);  
    }  
  
    public List<Book> getAll() {  
        return bookDao.getAll();  
    }  
}
```

处理完新增后，会发现新增还存在一个问题，  
新增成功后，再次点击`新增`按钮会发现之前的数据还存在，这个时候就需要在新增的时候将表单内容清空。
```js
// 重置表单  
resetForm() {  
    this.formData = {};  
}  
  
// 弹出添加窗口  
openSave() {  
    this.dialogFormVisible = true;  
    //每次弹出表单的时候，都重置一下数据  
    this.resetForm();  
}
```
### 4.6 修改功能

需求:完成图书信息的修改功能

1. 找到页面中的`编辑`按钮，该按钮绑定了`@click="openEdit(scope.row)"`
2. 在method的`openEdit`方法中发送异步请求根据ID查询图书信息
3. 根据后台返回的结果，判断是否查询成功
    - 如果查询成功打开修改面板回显数据，如果失败提示错误信息
4. 修改完成后找到修改面板的`确定`按钮，该按钮绑定了`@click="handleEdit()"`
5. 在method的`handleEdit`方法中发送异步请求提交修改数据
6. 根据后台返回的结果，判断是否修改成功
    - 如果成功提示错误信息，关闭修改面板，重新查询数据，如果失败提示错误信息

- scope.row代表的是当前行的行数据，也就是说,scope.row就是选中行对应的json数据，如下:
```json
{  
    "id": 1,  
    "type": "计算机理论",  
    "name": "Spring实战 第五版",  
    "description": "Spring入门经典教程，深入理解Spring原理技术内幕"  
}
```

- 修改openEdit()方法
```js
openEdit(row) {  
    axios.get("/books/" + row.id).then((res) => {  
        if (res.data.code == 20041) {  
            this.formData = res.data.data;  
            this.dialogFormVisible4Edit = true;  
        } else {  
            this.$message.error(res.data.msg);  
        }  
    });  
}
```

- 修改`handleUpdate`方法
```js
handleEdit() {  
    axios.put("/books", this.formData).then((res) => {  
        if (res.data.code == 20021) {  
            this.dialogFormVisible4Edit = false;  
            this.$message.success("修改成功")  
        } else if (res.data.code == 20020) {  
            this.$message.error("修改失败")  
        } else {  
            this.$message.error(res.data.msg);  
        }  
    }).finally(() => {  
        this.getAll();  
    });  
}
```
### 4.7 删除功能

需求:完成页面的删除功能。

1. 找到页面的删除按钮，按钮上绑定了`@click="delete(scope.row)"`
2. method的`delete`方法弹出提示框
3. 用户点击取消，提示操作已经被取消。
4. 用户点击确定，发送异步请求并携带需要删除数据的主键ID
5. 根据后台返回结果做不同的操作
    - 如果返回成功，提示成功信息，并重新查询数据
    - 如果​返回失败，提示错误信息，并重新查询数据

- 修改`delete`方法
```js
deleteBook(row) {  
    this.$confirm("此操作永久删除当前数据，是否继续？","提示",{  
        type:'info'  
    }).then(()=> {  
        axios.delete("/books/" + row.id).then((res) => {  
            if (res.data.code == 20031) {  
                this.$message.success("删除成功")  
            } else if (res.data.code == 20030) {  
                this.$message.error("删除失败")  
            }  
        }).finally(() => {  
            this.getAll();  
        });  
    }).catch(()=>{  
        this.$message.info("取消删除操作")  
    })  
}
```

至此增删改操作就都完成了，完整的前端代码如下
```html
<!DOCTYPE html>  
  
<html>  
<head>  
    <!-- 页面meta -->  
    <meta charset="utf-8">  
    <title>SpringMVC案例</title>  
    <!-- 引入样式 -->  
    <link rel="stylesheet" href="../plugins/elementui/index.css">  
    <link rel="stylesheet" href="../plugins/font-awesome/css/font-awesome.min.css">  
    <link rel="stylesheet" href="../css/style.css">  
</head>  
  
<body class="hold-transition">  
  
<div id="app">  
  
    <div class="content-header">  
        <h1>图书管理</h1>  
    </div>  
  
    <div class="app-container">  
        <div class="box">  
            <div class="filter-container">  
                <el-input placeholder="图书名称" style="width: 200px;" class="filter-item"></el-input>  
                <el-button class="dalfBut">查询</el-button>  
                <el-button type="primary" class="butT" @click="openSave()">新建</el-button>  
            </div>  
  
            <el-table size="small" current-row-key="id" :data="dataList" stripe highlight-current-row>  
                <el-table-column type="index" align="center" label="序号"></el-table-column>  
                <el-table-column prop="type" label="图书类别" align="center"></el-table-column>  
                <el-table-column prop="name" label="图书名称" align="center"></el-table-column>  
                <el-table-column prop="description" label="描述" align="center"></el-table-column>  
                <el-table-column label="操作" align="center">  
                    <template slot-scope="scope">  
                        <el-button type="primary" size="mini" @click="openEdit(scope.row)">编辑</el-button>  
                        <el-button size="mini" type="danger" @click="deleteBook(scope.row)">删除</el-button>  
                    </template>  
                </el-table-column>  
            </el-table>  
  
  
            <!-- 新增标签弹层 -->  
            <div class="add-form">  
                <el-dialog title="新增图书" :visible.sync="dialogFormVisible">  
                    <el-form ref="dataAddForm" :model="formData" label-position="right"  
                             label-width="100px">  
                        <el-row>  
                            <el-col :span="12">  
                                <el-form-item label="图书类别" prop="type">  
                                    <el-input v-model="formData.type"/>  
                                </el-form-item>  
                            </el-col>  
                            <el-col :span="12">  
                                <el-form-item label="图书名称" prop="name">  
                                    <el-input v-model="formData.name"/>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
  
                        <el-row>  
                            <el-col :span="24">  
                                <el-form-item label="描述">  
                                    <el-input v-model="formData.description" type="textarea"></el-input>  
                                </el-form-item>  
                            </el-col>  
                        </el-row>  
                    </el-form>  
                    <div slot="footer" class="dialog-footer">  
                        <el-button @click="dialogFormVisible = false">取消</el-button>  
                        <el-button type="primary" @click="saveBook()">确定</el-button>  
                    </div>  
                </el-dialog>  
            </div>  
  
        </div>  
  
        <!-- 编辑标签弹层 -->  
        <div class="add-form">  
            <el-dialog title="编辑检查项" :visible.sync="dialogFormVisible4Edit">  
                <el-form ref="dataEditForm" :model="formData" label-position="right" label-width="100px">  
                    <el-row>  
                        <el-col :span="12">  
                            <el-form-item label="图书类别" prop="type">  
                                <el-input v-model="formData.type"/>  
                            </el-form-item>  
                        </el-col>  
                        <el-col :span="12">  
                            <el-form-item label="图书名称" prop="name">  
                                <el-input v-model="formData.name"/>  
                            </el-form-item>  
                        </el-col>  
                    </el-row>  
                    <el-row>  
                        <el-col :span="24">  
                            <el-form-item label="描述">  
                                <el-input v-model="formData.description" type="textarea"></el-input>  
                            </el-form-item>  
                        </el-col>  
                    </el-row>  
                </el-form>  
                <div slot="footer" class="dialog-footer">  
                    <el-button @click="dialogFormVisible4Edit = false">取消</el-button>  
                    <el-button type="primary" @click="handleEdit()">确定</el-button>  
                </div>  
            </el-dialog>  
        </div>  
  
    </div>  
  
</div>  
</div>  
</body>  
  
<!-- 引入组件库 -->  
<script src="../js/vue.js"></script>  
<script src="../plugins/elementui/index.js"></script>  
<script type="text/javascript" src="../js/jquery.min.js"></script>  
<script src="../js/axios-0.18.0.js"></script>  
  
<script>  
    var vue = new Vue({  
  
        el: '#app',  
  
        data: {  
            dataList: [],//当前页要展示的分页列表数据  
            formData: {},//表单数据  
            dialogFormVisible: false,//增加表单是否可见  
            dialogFormVisible4Edit: false,//编辑表单是否可见  
            pagination: {},//分页模型数据，暂时弃用  
        },  
  
        //钩子函数，VUE对象初始化完成后自动执行  
        created() {  
            this.getAll();  
        },  
  
        methods: {  
            // 重置表单  
            resetForm() {  
                this.formData = {};  
            },  
  
            // 弹出添加窗口  
            openSave() {  
                this.dialogFormVisible = true;  
                this.resetForm();  
            },  
  
            //添加  
            saveBook() {  
                axios.post("/books", this.formData).then((res) => {  
                    if (res.data.code == 20011) {  
                        this.dialogFormVisible = false;  
                        this.$message.success("添加成功")  
                    } else if (res.data.code == 20010) {  
                        this.$message.error("添加失败");  
                    } else {  
                        this.$message.error(res.data.msg);  
                    }  
                }).finally(() => {  
                    this.getAll();  
                })  
            },  
  
            //主页列表查询  
            getAll() {  
                axios.get("/books").then((res) => {  
                    this.dataList = res.data.data;  
                })  
            },  
            // 弹出编辑页面，调用查询，将查询到的数据赋给formData，这样就能回显数据了  
            openEdit(row) {  
                axios.get("/books/" + row.id).then((res) => {  
                    if (res.data.code == 20041) {  
                        this.formData = res.data.data;  
                        this.dialogFormVisible4Edit = true;  
                    } else {  
                        this.$message.error(res.data.msg);  
                    }  
                });  
            },  
            // 修改  
            handleEdit() {  
                axios.put("/books", this.formData).then((res) => {  
                    if (res.data.code == 20021) {  
                        this.dialogFormVisible4Edit = false;  
                        this.$message.success("修改成功")  
                    } else if (res.data.code == 20020) {  
                        this.$message.error("修改失败")  
                    } else {  
                        this.$message.error(res.data.msg);  
                    }  
                }).finally(() => {  
                    this.getAll();  
                });  
            },  
            // 删除操作  
            deleteBook(row) {  
                this.$confirm("此操作永久删除当前数据，是否继续？", "提示", {  
                    type: 'info'  
                }).then(() => {  
                    axios.delete("/books/" + row.id).then((res) => {  
                        if (res.data.code == 20031) {  
                            this.$message.success("删除成功")  
                        } else if (res.data.code == 20030) {  
                            this.$message.error("删除失败")  
                        }  
                    }).finally(() => {  
                        this.getAll();  
                    });  
                }).catch(() => {  
                    this.$message.info("取消删除操作")  
                })  
            }  
             
        }  
    })  
</script>  
</html>
```

## 5 拦截器

### 5.1 拦截器概念

在学习拦截器的概念之前，我们先看一张图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bee1f4a99bc6e3bdd612f02592c856d2.png)


1. 浏览器发送一个请求，会先到Tomcat服务器的web服务器
2. Tomcat服务器接收到请求后，会先去判断请求的是`静态资源`还是`动态资源`
3. 如果是静态资源，会直接到Tomcat的项目部署目录下直接访问
4. 如果是动态资源，就需要交给项目的后台代码进行处理
5. 在找到具体的方法之前，我们可以去配置过滤器（可以配置多个），按照顺序进行执行（在这里就可以进行权限校验）
6. 然后进入到中央处理器（SpringMVC中的内容），SpringMVC会根据配置的规则进行拦截
7. 如果满足规则，则进行处理，找到其对应的`Controller`类中的方法进行之星，完成后返回结果
8. 如果不满足规则，则不进行处理
9. 这个时候，如果我们需要在每个Controller方法执行的前后添加业务，具体该如何来实现？
    - 这个就是拦截器要做的事

- 拦截器（Interceptor）是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制器方法的执行
    
    - `作用：`
        - 在指定的方法调用前后执行预先设定的代码
        - 阻止原始方法的执行
    - `总结：`拦截器就是用来作增强
- 但是这个拦截器貌似跟我们之前学的过滤器很像啊，不管是从作用上来看还是从执行顺序上来看
    
    - 那么拦截器和过滤器之间的区别是什么呢？
        - `归属不同：`Filter属于Servlet技术，而Interceptor属于SpringMVC技术
        - `拦截内容不同：`Filter对所有访问进行增强，Interceptor仅对SpringMVC的访问进行增强

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e7738078e6e2558d7ebdcd040365cea0.png)


### 5.2 拦截器入门案例

#### 5.2.1 环境准备

- 创建一个Web的Maven项目
- 导入坐标
```xml
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>2.9.0</version>  
</dependency>
```

- 创建对应的配置类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
public class SpringMvcConfig {  
}

public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class[0];  
    }  
  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[]{SpringMvcConfig.class};  
    }  
  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    @Override  
    protected Filter[] getServletFilters() {  
        CharacterEncodingFilter filter = new CharacterEncodingFilter();  
        filter.setEncoding("utf-8");  
        return new Filter[]{filter};  
    }  
}
```

- 编写Controller类
```java
@RestController  
@RequestMapping("/books")  
public class BookController {  
  
    @PostMapping  
    public String save(@RequestBody Book book){  
        System.out.println("book save .." + book);  
        return "{'module':'book save'}";  
    }  
  
    @DeleteMapping("/{id}")  
    public String delete(@PathVariable Integer id){  
        System.out.println("book delete .." + id);  
        return "{'module':'book delete'}";  
    }  
  
    @PutMapping  
    public String update(@RequestBody Book book){  
        System.out.println("book update .." + book);  
        return "{'module':'book update'}";  
    }  
  
    @GetMapping("/{id}")  
    public String getById(@PathVariable Integer id){  
        System.out.println("book getById .." + id);  
        return "{'module':'book getById'}";  
    }  
  
    @GetMapping  
    public String getAll(){  
        System.out.println("book getAll ..");  
        return "{'module':'book getAll'}";  
    }  
}
```

- 使用PostMan测试，没有问题的话就可以继续往下看了
#### 5.2.2 拦截器开发

- `步骤一：`创建拦截器类  
	在`com.blog.controller.interceptor`下创建`ProjectInterceptor`类，实现`HandlerInterceptor`接口，并重写其中的三个方法
```java
@Component  
//注意当前类必须受Spring容器控制  
public class ProjectInterceptor implements HandlerInterceptor {  
  
    //原始方法调用前执行的内容  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
        System.out.println("preHandle");  
        return true;  
    }  
  
    //原始方法调用后执行的内容  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
        System.out.println("postHandle");  
    }  
  
    //原始方法调用完成后执行的内容  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
        System.out.println("afterCompletion");  
    }  
}
```

- `步骤二：`配置拦截器类
```java
public class SpringMvcSupport extends WebMvcConfigurationSupport {  
    @Autowired  
    private ProjectInterceptor projectInterceptor;  
  
    @Override  
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {  
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");  
    }  
  
    @Override  
    protected void addInterceptors(InterceptorRegistry registry) {  
        //配置拦截器,拦截路径是/books，只会拦截/books，拦截不到/books/1  
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books");  
    }  
}
```

- `步骤三：`SpringMvc添加SpringMvcSupport包扫描
```java
@Configuration  
@ComponentScan({"com.blog.controller", "com.blog.config"})  
@EnableWebMvc  
public class SpringMvcConfig {  
}
```

- `步骤四：`运行程序测试

	- 使用PostMan发送请求给`localhost:8080/books`，控制台输出如下，说明已经成功拦截![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/edf171d6547cb70110700e05ee447085.png)


	- 使用PostMan发送请求给`localhost:8080/books/9527`，控制台输出如下，说明没有拦截，若想拦截，则继续修改拦截器的拦截规则![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d37ec30d211c8df1c1005de0af04873c.png)


- `步骤五：`修改拦截器拦截规则
```java
@Configuration  
public class SpringMvcSupport extends WebMvcConfigurationSupport {  
    @Autowired  
    private ProjectInterceptor projectInterceptor;  
  
    @Override  
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {  
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");  
    }  
  
    @Override  
    protected void addInterceptors(InterceptorRegistry registry) {  
        //配置拦截器，查看源码发现，参数是个可变形参，可以设置任意多个拦截路径  
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books","/books/*");  
    }  
}
```

- 此时我们再次使用PostMan发送请求给`localhost:8080/books/9527`，控制台输出如下，说明已经成功拦截![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8d5f8e2b26a4412e4c20d01bae406354.png)

- 最后说一件事，就是拦截器中的`preHandler`方法，如果返回true，则代表放行，会执行原始`Controller`类中要请求的方法，如果返回`false`，则代表拦截，后面的就不会再执行了。

- `步骤六：`简化SpringMvcSupport的编写  
    我们可以让`SpringMvcConfig`类实现`WebMvcConfigurer`接口，然后直接在`SpringMvcConfig`中写`SpringMvcSupport`的东西，这样我们就不用再写`SpringMvcSupport`类了，全都在`SpringMvcConfig`中写
```java
@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
//实现WebMvcConfigurer接口可以简化开发，但具有一定的侵入性  
public class SpringMvcConfig implements WebMvcConfigurer {  
    @Autowired  
    private ProjectInterceptor projectInterceptor;  
  
    public void addResourceHandlers(ResourceHandlerRegistry registry) {  
        //对静态资源放行  
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");  
    }  
  
    public void addInterceptors(InterceptorRegistry registry) {  
        //配置拦截器  
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books", "/books/*");  
    }  
}
```

- 最后我们来看下拦截器的执行流程![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6765365f6154fff0faa170e06247eac5.png)


- 当有拦截器后，请求会先进入`preHandle`方法，
    - 如果方法返回`true`，则放行继续执行后面的handle(Controller的方法)和后面的方法
    - 如果返回`false`，则直接跳过后面方法的执行。
### 5.3 拦截器参数

#### 5.3.1 前置处理方法

原始方法之前运行preHandle
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
    System.out.println("preHandle");  
    return true;  
}
```

- `request:`请求对象
- `response:`响应对象
- `handler:`被调用的处理器对象，本质上是一个方法对象，对反射中的Method对象进行了再包装

使用request对象可以获取请求数据中的内容，如获取请求头的`Content-Type`
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
    String name = request.getHeader("Content-Type");  
    System.out.println("preHandle.." + name);  
    return true;  
}
```

控制台输出如下，成功输出了Content-Type`application/json`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb6e20aafa6a93faa9271b8738faff7e.png)


使用handler参数，可以获取方法的相关信息
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
    HandlerMethod handlerMethod = (HandlerMethod) handler;  
    String methodName = handlerMethod.getMethod().getName();  
    System.out.println("preHandle.." + methodName);  
    return true;  
}
```

控制台输出如下，成功输出了方法名`save`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d142edef142f81513ec8a21f93980803.png)


#### 5.3.2 后置处理方法

原始方法运行后运行，如果原始方法被拦截，则不执行
```java
//原始方法调用后执行的内容  
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
    System.out.println("postHandle");  
}
```

前三个参数和上面的是一致的。  
`modelAndView:`如果处理器执行完成具有返回结果，可以读取到对应数据与页面信息，并进行调整  
因为我们现在都是返回json数据，所以该参数的使用率不高。
#### 5.3.3 完成处理方法

拦截器最后执行的方法，无论原始方法是否执行
```java
//原始方法调用完成后执行的内容  
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
    System.out.println("afterCompletion");  
}
```

前三个参数与上面的是一致的。  
`ex:`如果处理器执行过程中出现异常对象，可以针对异常情况进行单独处理  
因为我们现在已经有全局异常处理器类，所以该参数的使用率也不高。  
这三个方法中，最常用的是`preHandle`，在这个方法中可以通过返回值来决定是否要进行放行，我们可以把业务逻辑放在该方法中，如果满足业务则返回`true`放行，不满足则返回`false`拦截。
### 5.4 拦截器链配置

目前，我们在项目中只添加了一个拦截器，如果有多个，该如何配置?配置多个后，执行顺序是什么?
#### 5.4.1 配置多个拦截器

- `步骤一：`创建拦截器类  
	直接复制一份改个名，改个输出语句
```java
@Component  
//注意当前类必须受Spring容器控制  
public class ProjectInterceptor2 implements HandlerInterceptor {  
  
    //原始方法调用前执行的内容  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
        System.out.println("preHandle..222");  
        return true;  
    }  
  
    //原始方法调用后执行的内容  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
        System.out.println("postHandle..222");  
    }  
  
    //原始方法调用完成后执行的内容  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
        System.out.println("afterCompletion..222");  
    }  
}
```

- `步骤二：`配置拦截器类
```java
@Configuration  
@ComponentScan("com.blog.controller")  
@EnableWebMvc  
public class SpringMvcConfig implements WebMvcConfigurer {  
    @Autowired  
    private ProjectInterceptor projectInterceptor;  
    @Autowired  
    private ProjectInterceptor2 projectInterceptor2;  
  
    public void addInterceptors(InterceptorRegistry registry) {  
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books", "/books/*");  
        registry.addInterceptor(projectInterceptor2).addPathPatterns("/books", "/books/*");  
    }  
}
```

重新启动服务器，使用PostMan发送请求，控制台输出如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d9eebccc29bc10ba1d0df842f61edf23.png)


- 当配置多个拦截器时，形成拦截器链
- 拦截器链的运行顺序参照拦截器添加顺序为准
- 当拦截器中出现对原始处理器的拦截，后面的拦截器均终止运行
- 当拦截器运行中断，仅运行配置在前面的拦截器的afterCompletion操作
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7edd446b22d157e2214647dfd738fd2.png)


- `preHandle：`与配置顺序相同，必定运行
- `postHandle:`与配置顺序相反，可能不运行
- `afterCompletion:`与配置顺序相反，可能不运行。

## 6 完结

- 至此SSM的学习就告一段落了，有空再回来复盘一遍，要继续学Maven高级、SpringBoot和MyBatis-Plus了