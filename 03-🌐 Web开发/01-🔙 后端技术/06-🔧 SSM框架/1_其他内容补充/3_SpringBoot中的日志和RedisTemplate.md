
## 1 日志        

Springboot项目，`默认日志组件为logback`，且已经整合对应的logback包，因此不再需要额外的jar。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/225ad0b9a88c087e13955e43ec147c25.png)


- 从上图可以看到，日志输出内容元素具体如下：

	- 时间日期：精确到毫秒
	
	- `日志级别：ERROR, WARN, INFO, DEBUG 或 TRACE`
	
	- 进程ID
	
	- 分隔符：— 标识实际日志的开始
	
	- 线程名：方括号括起来（可能会截断控制台输出）
	
	- Logger名：通常使用源代码的类名
	
	- 日志内容

### 1.1 日志级别

就是为了对日志信息就行分类化管理。

TRACE < `DEBUG` < ***INFO*** < `WARN` < ***ERROR*** < FATAL。

如果`指定了某个级别，那么低于改级别的信息都不会输出`。

比如将日志级别设置为 WARN ，则低于 WARN 的信息都不会输出。 

`Spring Boot中默认日志级别就是INFO`

- 所以，一般会将sql的执行相关内容的日志，设置为debug级别![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8278df3df0c269fba8421137b4c4d02.png)
```properties
#日志级别 root表示所有包，也可以单独配置具体包  
logging.level.root=info
```

### 1.2 日志使用方式-- Logger

```java
Logger logger = LoggerFactory.getLogger(MapperController.class);

//一般程序运行正常的情况下
logger.info(需要打印的日志信息);

//在catch到异常的时候
logger.error(错误信息);
```

- 注意的是，在那个类创建的logger对象，形参就填那个类的类对象
- 然后使用logger对象的各个方法即可！！！

### 1.3 日志使用方式--@Slf4j注解

1、添加lombok依赖以及idea安装lombok插件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bed29da5f0c79e8f0c26983529acc55d.png)

```xml
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <optional>true</optional>  
</dependency>
```

`<optional>true</optional>`表示依赖不传递，如B项目依赖本项目，不会把该依赖传递给B。

2、添加@Slf4j注解并在代码中写出日志打印语句

- 方式一：使用拼接字符串
```java
log.info("dongTaiQuery的入参是：{}" + user);
```

- 方式二：使用大括号占位符
```java
log.info("dongTaiQuery的入参是：{}", user);
```

### 1.4 日志打印到文件

```properties
#日志打印到文件  路径需要写全
logging.file.name=D:/logs/my.log
```

## 2 RedisTemplate  

### 2.1 添加redis的起步依赖

```xml
<!-- 配置使用redis启动器 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>
```
### 2.2 配置redis的连接信息

```properties
#Redis
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

- `启动redis`
### 2.3 引入RedisTemplate工具类RedisUtil

```java
@Component  
public class RedisUtil {  
  
    @Autowired  
    private RedisTemplate<Object, Object> redisTemplate;  
  
    /**  
     * 设置 String 类型 key-value    
     * @param key  
     * @param value  
     */  
    public void set(String key, String value) {  
        redisTemplate.opsForValue().set(key, value);  
    }  
  
    /**  
     * 获取 String 类型 key-value  
     * @param key  
     * @return  
     */  
    public String get(String key) {  
        return (String) redisTemplate.opsForValue().get(key);  
    }  
  
    /**  
     * 设置 String 类型 key-value 并添加过期时间 (毫秒单位)  
     *     
     * @param key  
     * @param value  
     * @param time  过期时间,毫秒单位  
     */  
    public void setForTimeMS(String key, String value, long time) {  
        redisTemplate.opsForValue().set(key, value, time, TimeUnit.MILLISECONDS);  
    }  
  
    /**  
     * 设置分布式锁，获取锁的时间为300ms  
     */    public Boolean lock(String key, String value, long expireTime) {  
        Long start = System.currentTimeMillis();  
        //自旋，在一定时间内获取锁，超时则返回错误  
        for (; ; ) {  
            //Set命令返回OK，则证明获取锁成功  
            Boolean ret = redisTemplate.opsForValue().setIfAbsent(key, value, expireTime,  
                    TimeUnit.SECONDS);  
            if (ret) {  
                return true;  
            }  
            //否则循环等待，在timeout时间内仍未获取到锁，则获取失败  
            long end = System.currentTimeMillis() - start;  
            if (end >= 300) {  
                return false;  
            }  
        }  
    }  
  
    /**  
     * 删除 key-value  
     *     * @param key  
     * @return  
     */  
    public boolean delete(String key) {  
        return redisTemplate.delete(key);  
    }  
  
//省略其它方法
```

### 2.4 在需要用redis的地方注入RedisUtil的对象即可。

案例：查询全部用户方法使用redis缓存来提高性能

1、先查redis，有则直接返回redis存储的结果，

2、redis无则查询数据库，并将结果存储到redis中，供下次查询使用

### 2.5 配置redis自定义JSON序列化

```java

@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        // 创建模板
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 设置序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // key和 hashKey采用 string序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // value和 hashValue采用 JSON序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
```