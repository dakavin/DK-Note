---
文章标题: "[[4_SpringBoot异步、定时任务和方法重试]]" 
文章作者: Dakkk
文章概要: |
  AI未能生成概要。
tags:
  - "未分类"
相关文章: 暂无 🤷
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/1_其他内容补充/4_SpringBoot异步、定时任务和方法重试.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 SpringBoot异步方法       

### 1.1 异步和同步

“异步”对应的是“同步”

`同步`：指程序按照定义顺序依次执行，每一行程序都必须等待上一行程序执行完成之后才能执行；

`异步`：调用指程序在顺序执行时，不等待异步调用的语句返回结果就执行后面的程序。

如下就是同步方法的示例：        

```java
@Service  
@Slf4j  
public class MyServiceImpl implements MyService {  
  
  
    public void A() throws InterruptedException {  
        log.info("A方法开始执行");  
        B();  
        C();  
        log.info("A方法开始结束");  
    }  
  
    public void B() throws InterruptedException {  
        log.info("B方法开始执行");  
        Thread.sleep(5000);  
        log.info("B执行结束");  
    }  
  
    public void C() {  
        log.info("C方法开始执行");  
        log.info("C执行结束");  
    }  
  
}
```

- 执行结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/917b2c498edef71b2db284dcaa1f32e8.png)

如果异步就不用等方法执行完了。比如此处如果B方法是异步，那么A方法不用等B方法执行完就去执行C方法
### 1.2 SpringBoot异步方法  
  

1、启动类里面使用@EnableAsync注解开启功能，自动扫描

```java
@SpringBootApplication  
@EnableAsync  
public class SpringbootApplication {
```

2、定义异步方法类并使用@Component等注解标注，在类上或方法上加@Async注解。

```java
@Component  
@Async //可放在类上，也可在方法上  
public class AsyncTask {  
  
    public void B() throws InterruptedException {  
        System.out.println("B方法开始执行");  
        Thread.sleep(5000);  
        System.out.println("B执行结束");  
    }  
}
```

注意点：

1、同步方法想调异步方法必须要把异步方法单独封装到一个Bean中。

2、`异步方法使用注解@Async的返回值只能为void或者Future`

### 1.3 异步方法返回值—Future

Future接口，方法实际返回的是Future的实现类AsyncResult  

Future的相关方法
	`isDone()`   //判断任务是否完成
	`get() `    //获取任务最终结果，`这是一个阻塞方法`，会等待任务执行好才会执行后面的代码
	`get(long timeout, TimeUnit unit)`   //有等待时常的get方法，等待时间到了后仍然没有计算完成，则抛异常

```java
public Future<String> B() throws InterruptedException {  
    System.out.println("B方法开始执行");  
    Thread.sleep(5000);  
    System.out.println("B执行结束");  
    return new AsyncResult<String>("B");  
}
```

```java
@Service  
@Slf4j  
public class MyServiceImpl implements MyService {  
  
    @Autowired  
    MyAsyncService myAsyncService;  
    @Override  
    public void A() throws InterruptedException {  
        log.info("A方法开始执行了");  
//        B();  
        Future<String> future = myAsyncService.B();  
        //1. 测试isDone方法  
//        Thread.sleep(3000);  
//        boolean done = future.isDone();  
//        log.info("异步方法执行是否完成了：{}",done);  
        //2. 测试get方法  
//        String s = null;  
//        try {  
//            s = future.get();  
//        } catch (ExecutionException e) {  
//            e.printStackTrace();  
//        }  
//        log.info("异步方法的返回值：{}",s);  
        //3. 测试等待时长的get方法  
        String s = null;  
        try {  
            s = future.get(1, TimeUnit.SECONDS);  
        } catch (Exception e) {  
            e.printStackTrace();  
        }   
log.info("异步方法的返回值：{}",s);  
        C();  
        log.info("A方法执行结束");  
    }  
  
    @Override  
    public void B() throws InterruptedException {  
        log.info("B方法开始执行");  
        Thread.sleep(5000);  
        log.info("B执行结束");  
    }  
  
    @Override  
    public void C() throws InterruptedException {  
        log.info("C方法开始执行");  
        log.info("C执行结束");  
    }  
}
```

## 2 SpringBoot定时任务

定时任务：就是`定个时间，让某个任务在那个时间执行  `

类似我们的设置闹钟一样，可以设置什么时候响，隔几分钟响一次等等   

实际案例：

1、凌晨2点进行数据备份

2、每分钟检测超时订单（超过30分钟未支付的订单），并取消该订单

### 2.1 操作步骤   

1、在主启动类加上这个注解 @EnableScheduling：

```java
@SpringBootApplication  
@EnableScheduling  
public class SpringbootApplication {  
  
    public static void main(String[] args)  
    {  
        SpringApplication.run(SpringbootdemoApplication.class, args);  
    }  
  
}
```

2、然后编写定时任务类，在中的方法上加上@Scheduled。

常见的三种方式：

@Scheduled(fixedDelay=2000)：上一次`执行完毕`时间点后2秒再次执行

@Scheduled(fixedRate=2000)：上一次`开始执行`时间点后2秒再次执行

`@Scheduled(cron="* * * * * ?")：按cron规则执行`。

```java
@Component  
@Slf4j  
public class MyScheduledTask {  
    //定义定时方法的初始执行次数  
    private int fixedDelayCount = 1;  
    private int fixedRateCount = 1;  
    private int cronCount = 1;  
  
    //表示当前方法执行完毕的5s后，Spring会再次调用该方法  
    @Scheduled(fixedDelay = 5000)  
    public void testFixDelay(){  
        log.info("===fixedDelay:第{}次执行方法",fixedDelayCount++);  
        try {  
            Thread.sleep(2000);  
            //方法的再次执行时机是7s  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        log.info("===fixedDelay：执行结束");  
    }  
  
    //fixedRate = 1000表示当前方法开始执行5000ms后，Spring会再次调用该方法  
    @Scheduled(fixedRate = 5000)
    public void testFixedRate() {  
        log.info("===fixedRate: 第{}次执行方法", fixedRateCount++);  
        try {  
            Thread.sleep(2000);  
            //方法的再次执行时机，不是7s了，而是5s  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        log.info("===fixRate：执行结束");  
    }  
  
    //根据cron表达式确定定时规则  
    @Scheduled(cron = "* * * * * ?")  
    // 表示 任何 秒 分 时 日期 月份 和 年份都会执行  
    public void testCron() {  
        log.info("===cron: 第{}次执行方法", cronCount++);  
    }  
}
```

`Cron表达式`

从左到右共7位（用空格隔开，年份可省略）：

`秒   分   小时   日期(月中日)   月份  星期（周中日）   年份`

`Seconds`
秒：数字0－59

`Minutes`
分：数字0－59

`Hours`
时 ：数字0-23

`Day-of-Month`
月中的几号 ：可以用数字1-31 中的任一一个值，但要注意一些特别的月份

`Month`
一年中的几月：可以用0-11 或用字符串 “JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV and DEC” 表示

`Day-of-Week`
每周：数字1-7（1 ＝ 星期日），或用字符口串“SUN, MON, TUE, WED, THU, FRI and SAT”

|字段名       |          允许的值                   |     允许的特殊字符  |
|---|---|---|
|秒              |             0-59                      |         , - * /     |
|分              |             0-59                        |       , - * /   |
|小时            |           0-23                       |        , - * /  |
|日               |           1-31                           |     , - * ? / L W C  |
|月               |       1-12 or JAN-DEC          |       , - * /  |
|周几            |       1-7 or SUN-SAT             |      , - * ? / L C #  |
|年 (可选字段)    |   empty, 1970-2099       |     , - * /|
`* `：代表整个时间段.
`/ `：表示每多长时间执行一次
	`0/15`表示每隔15分钟执行一次,“0”表示为从“0”分开始；
	`3/20`表示每隔20分钟执行一次，“3”表示从第3分钟开始执行
`?` ：表示每月的某一天，或第几周的某一天
`L` ：
	“`6L`”表示“每月的最后一个星期五”
`W`：表示为最近工作日
	如“15W”放在每月（day-of-month）字段上表示为“到本月15日最近的工作日”
`#`：是用来指定“的”每月第n个工作日
	"6#3"或者"FRI#3":在每周（day-of-week）中表示“每月第三个星期五”

`问号(?)`就是用来对日期和星期字段做互斥的，`问号(?)`的作用是指明该字段‘没有特定的值’，`星号(*)`和其它值，比如数字，都是给该字段指明特定的值，而`星号(*`)代表所有，在天时表示每一天。

“?”字符：表示不确定的值
“,”字符：指定数个值
“-”字符：指定一个值的范围
“/”字符：指定一个值的增加幅度。n/m表示从n开始，每次增加m
“L”字符：用在日表示一个月中的最后一天，用在周表示该月最后一个星期X
“W”字符：指定离给定日期最近的工作日(周一到周五)
“#”字符：表示该月第几个周X。6#3表示该月第3个周五


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4b44a1408613b85d0ca45230b77ecfa9.png)


（1）`*`：表示`匹配该域的任意值`。假如在Minutes域使用*, 即表示每分钟都会触发事件。

（2）`-`：表示`范围`。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次

（3）`/`：表示起始时间开始触发，然后每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次.

（4）`?`：只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13 13 15 20 * ?, 其中最后一位只能用？，而不能使用*，如果使用`表示不管星期几都会触发，实际上并不是这样`。

`常用表达式例子`

　　（1）0 0 2 1 * ? *   表示在每月的1日的凌晨2点调整任务

　　（2）0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业

　　（3）0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作

　　（4）0 0 10,14,16 * * ?   每天上午10点，下午2点，4点

　　（5）0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时

　　（6）0 0 12 ? * WED    表示每个星期三中午12点

　　（7）0 0 12 * * ?   每天中午12点触发

　　（8）0 15 10 ? * *    每天上午10:15触发

　　（9）0 15 10 * * ?     每天上午10:15触发

　　（10）0 15 10 * * ? *    每天上午10:15触发

　　（11）0 15 10 * * ? 2005    2005年的每天上午10:15触发

　　（12）0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发

　　（13）0 0/5 14 * * ?    在每天下午2点到下午2:55期间的每5分钟触发

　　（14）0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发

　　（15）0 0-5 14 * * ?    在每天下午2点到下午2:05期间的每1分钟触发

　　（16）0 10,44 14 ? 3 WED    每年三月的星期三的下午2:10和2:44触发

　　（17）0 15 10 ? * MON-FRI    周一至周五的上午10:15触发

　　（18）0 15 10 15 * ?    每月15日上午10:15触发

　　（19）0 15 10 L * ?    每月最后一日的上午10:15触发

　　（20）0 15 10 ? * 6L    每月的最后一个星期六上午10:15触发

　　（21）0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期六上午10:15触发

　　（22）0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发

### 2.2 默认情况下多个定时任务是串行执行的

看到控制台输出的结果，所有的定时任务都是通过一个线程来处理的，是因为在springboot中，默认情况下，定时任务的配置中`设定了一SingleThreadScheduledExecutor，是一个可以周期性执行任务的单线程线程池`。

这样会造成一个严重的问题：`某个定时任务会阻塞另外一个定时任务`。

要想`解决不同定时任务之间的相互影响，就是得引入线程池`。

```java
//fixedDelay = 1000表示当前方法执行完毕1000ms后，Spring会再次调用该方法  
@Scheduled(fixedDelay = 1000)  
public void testFixDelay() throws InterruptedException {  
	logger.info("===fixedDelay: 第{}次执行方法，线程：{}", fixedDelayCount++,Thread.currentThread().getName());  
//        Thread.sleep(2000);  
}  
  
//fixedRate = 1000表示当前方法开始执行1000ms后，Spring会再次调用该方法  
@Scheduled(fixedRate = 1000)  
public void testFixedRate() throws InterruptedException {  
	logger.error("===fixedRate: 第{}次执行方法，线程：{}", fixedRateCount++,Thread.currentThread().getName());  
//        Thread.sleep(2000);  
}  
  
//根据cron表达式确定定时规则  
@Scheduled(cron = "* * * * * ?")  
public void testCron() throws InterruptedException {  
	logger.error("===cron: 第{}次执行方法，线程：{}", cronCount++,Thread.currentThread().getName());  
//        Thread.sleep(2000);  
}  

```

可以看出都是同一个线程

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bed135c45d2aa33d9a6da0634a9d7b10.png)

### 2.3 引入线程池

我们要做的仅仅是实现SchedulingConfigurer接口，重写configureTasks方法，注册一个自定义的线程池。

```java
@Configuration  
//所有的定时任务都放在一个线程池中，定时任务启动时使用不同的线程。  
public class ScheduleConfig implements SchedulingConfigurer {  
    @Override  
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {  
        //设定一个长度10的定时任务线程池  
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(10));  
    }  
}
```

注意：配置了多线程的线程池之后，只会让不同的定时任务运行在不同线程里。但是对`同一个定时任务还是会串行执行的`，即该任务的前一次执行会阻塞后一次执行。

要想`解决同一个定时任务自己阻塞自己下一次执行，就的使用异步定时任务`。

### 2.4 异步定时任务

使用 @Async 注解+@EnableAsync实现异步

```java
@Async  //解决同一个定时任务内部阻塞
@Scheduled(cron = "* * * * * ?")  //cron接受cron表达式，根据cron表达式确定定时规则  
public void testCron() throws InterruptedException {  
    Thread.sleep(15000);  
    logger.info("===cron: 第{}次执行方法", cronCount++);  
}
```

```java
@Scheduled(fixedRate = 1000)  
@Async //解决同一个定时任务内部阻塞  
public void testFixedRate() {  
    log.info("===fixedRate: 第{}次执行方法", fixedRateCount++);  
    try {  
        Thread.sleep(2000);  
        //方法的再次执行时机，不是7s了，而是5s  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    log.info("===fixRate：执行结束");  
}
```
## 3 SpringBoot方法重试 

### 3.1 重试机制

当我们调用一个方法时，可能由于各种原因造成第一次失败，再去尝试就成功了，这就是重试机制。

重试，在项目需求中是非常常见的，例如遇到网络波动等，要求某个接口或者是方法可以最多/最少调用几次；

### 3.2 使用步骤

1、引入响应jar

```xml
<dependency>  
    <groupId>org.springframework.retry</groupId>  
    <artifactId>spring-retry</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.6.11</version>  
</dependency>
```

2、启动类加注解@EnableRetry开启重试机制

```java
@SpringBootApplication  
@EnableRetry  
public class SpringbootApplication {  
  
    public static void main(String[] args)  
    {  
        SpringApplication.run(SpringbootdemoApplication.class, args);  
    }  
  
}
```

3、要重试的Servcie 实现类的方法上加@Retryable注解

```java
public interface RetryService {  
    String testRetryMethod(String name);  
}
```

```java
@Service  
@Slf4j  
public class RetryServiceImpl implements RetryService {  
    public int count = 1;  
  
    @Retryable()  
    @Override  
    public String testRetryMethod(String name) {  
        log.info("第{}次尝试",count++);  
        int i = 1 / 0;  
        return null;  
    }  
}
```

注意：重试方法不能被其他非重试方法调用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e564dd5a4695e1345685ca9b3bd4746.png)


因为相当于，原始的RetryService对象调用这个方法，而`原始对象是没有被AOP增强过的，所以重试方法不会生效`

默认情况下，`方法抛异常`就会重试，最大重试次数3次（含方法第一次运行）

可以在@Retryable注解中添加其它特性：

1、`设置重试次数`：

```java
@Retryable(maxAttempts = 5)
```

2、`指定重试间隔时间`：

```java
@Retryable(backoff =@Backoff(delay = 2000))
```

 **delay**：指定等待时间，没有设置的话默认为1000ms

3、`指定抛哪些异常时不重试`：

```java
@Retryable(exclude =ArithmeticException.class)
```