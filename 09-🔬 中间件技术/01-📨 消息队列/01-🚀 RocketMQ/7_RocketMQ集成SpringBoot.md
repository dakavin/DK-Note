---
文章标题: "[[7_RocketMQ集成SpringBoot]]" 
文章作者: Dakkk
文章概要: |
  本文详细讲解了RocketMQ与SpringBoot的集成，涵盖了生产者、消费者的搭建与消息收发，并深入演示了同步、异步、单向、延迟、顺序、事务等多种消息发送模式，以及Tag/Key过滤和负载均衡/广播两种消费模式。
tags:
- "RocketMQ"
- "Spring Boot"
- "消息队列"
- "消息生产者"
- "消息消费者"
- "消息模式"
- "消息过滤"
- "分布式系统"
相关文章:
- "[[0_项目介绍]]"
- "[[秒杀+分布式锁项目]]"
- "[[0_导读]]"
- "[[1_搭建多模块工程（Spring Initializr）]]"
- "[[1_项目简介]]"
文章分类: "🔬 中间件技术"
文章路径: "09-🔬 中间件技术/01-📨 消息队列/01-🚀 RocketMQ/7_RocketMQ集成SpringBoot.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:16:27
---

## 1 集成SpringBoot

### 1.1 搭建rocketmq-producer（消息生产者）

- `步骤一：创建一个springboot项目`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea1a369c40fc47b65748a9c172024996.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea1a369c40fc47b65748a9c172024996.png)

- `步骤二：另外添加项目需要的RocketMQ依赖`

```xml
<!-- rocketmq的依赖 -->  
<dependency>  
    <groupId>org.apache.rocketmq</groupId>  
    <artifactId>rocketmq-spring-boot-starter</artifactId>  
    <version>2.2.2</version>  <!-- 对应rocketmq的版本为4.9.3 -->
</dependency>
```

- `步骤三：修改配置文件application.yml`

```yml
spring:
    application:
        name: rocketmq-producer

rocketmq:
    name-server: 127.0.0.1:9876     # rocketMq的nameServer地址
    producer:
        group: powernode-group        # 生产者组别
        send-message-timeout: 3000  # 消息发送的超时时间
        retry-times-when-send-async-failed: 2  # 异步消息发送失败重试次数
        max-message-size: 4194304       # 消息的最大长度
```

- `步骤四：在测试类里面测试发送消息`

往powernode主题里面发送一个简单的字符串消息

```java
/**  
 * 注入rocketMQTemplate，我们使用它来操作mq  
 */
 @Autowired  
private RocketMQTemplate rocketMQTemplate;  
  
/**  
 * 测试发送简单的消息  
 * @throws Exception  
 */
 @Test  
public void testSimpleMsg() throws Exception {  
    // 往powernode的主题里面发送一个简单的字符串消息  
    SendResult sendResult = rocketMQTemplate.syncSend("powernode", "我是一个简单的消息");  
    // 拿到消息的发送状态  
    System.out.println(sendResult.getSendStatus());  
    // 拿到消息的id  
    System.out.println(sendResult.getMsgId());  
}
```

运行后查看控制台

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb3a082c27ef6b4331383f956437efab.png)

- `步骤五：查看rocketMq的控制台`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/47ef36cf40c3984d6ccc629537f38f12.png)

查看消息细节

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f813180afe74a0dfb22031eca9d3b6fc.png)

### 1.2 搭建rocketmq-consumer（消息消费者）

- `步骤一：创建一个springboot项目`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/55542a497c5aea419b51bb2e793c4d9e.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c7b4f20d54f30688d61a01a879545ed.png)

- `步骤二：另外添加项目需要的RocketMQ依赖`
- 项目完整的pom.xml文件如下

```xml
<!-- rocketmq的依赖 -->  
<dependency>  
    <groupId>org.apache.rocketmq</groupId>  
    <artifactId>rocketmq-spring-boot-starter</artifactId>  
    <version>2.2.2</version>  <!-- 对应rocketmq的版本为4.9.3 -->
</dependency>
```

- `步骤三：修改配置文件application.yml`

```yml
spring:
    application:
        name: rocketmq-consumer

rocketmq:
    name-server: 127.0.0.1:9876
```

- 一个boot项目，如果不在yml配置文件中写出消费者组名，那么就可以写很多个消费者程序

- 一般在开发中，一个boot项目只对应一个消费者

- `步骤四：添加一个监听的类SimpleMsgListener`

消费者要消费消息，就添加一个监听

* `类上添加注解@Component和@RocketMQMessageListener  `

* `@RocketMQMessageListener`(topic = "powernode", consumerGroup = "powernode-group")  

* topic指定消费的主题，consumerGroup指定消费组,一个主题可以有多个消费者组,一个消息可以被多个不同的组的消费者都消费 

* 实现RocketMQListener接口，注意泛型的使用，可以为具体的类型，`如果想拿到消息的其他参数可以写成MessageExt`

```java
@Component  
@RocketMQMessageListener(topic = "bootTestTopic",consumerGroup = "test-boot-consumer-group")  
public class SimpleMsgListener implements RocketMQListener<MessageExt> {  
    /**  
     * 这个方法就是消费者方法  
     * 如果泛型定义了固定的类型，那么消息体就是我们的参数  
     * 如果想要获取消息的其他参数/指标，我们可以将泛型定义为MessageExt类    
     */  
    @Override  
    public void onMessage(MessageExt msg) {  
        System.out.println(msg);  
    }  
}
```

- `步骤五：启动rocketmq-consumer`

查看控制台，发现我们已经监听到消息了

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0d8831b175aed5dc498ca6654788bdb4.png)

## 2 发送对象和集合消息

我们接着在上面项目里面做

### 2.1 发送对象消息

`主要是监听的时候泛型中写对象的类型即可`


- `步骤一：修改rocketmq-producer添加一个Order类`

```java
/**  
 * 订单对象  
 */  
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class Order {  
    /**  
     * 订单号  
     */  
    private String orderId;  
  
    /**  
     * 订单名称  
     */  
    private String orderName;  
  
    /**  
     * 订单价格  
     */  
    private Double price;  
  
    /**  
     * 订单号创建时间  
     */  
    private Date createTime;  
  
    /**  
     * 订单描述  
     */  
    private String desc;  
  
}
```

- `步骤二：修改rocketmq-producer添加一个单元测试`

```java
/**  
 * 测试发送对象消息  
 *  
 * @throws Exception  
 */@Test  
public void testObjectMsg() throws Exception {  
    Order order = new Order();  
    order.setOrderId(UUID.randomUUID().toString());  
    order.setOrderName("我的订单");  
    order.setPrice(998D);  
    order.setCreateTime(new Date());  
    order.setDesc("加急配送");  
    // 往powernode-obj主题发送一个订单对象  
    rocketMQTemplate.syncSend("powernode-obj", order);  
}
```

- `步骤三：发送此消息`

- `步骤四：修改rocketmq-consumer也添加一个Order类（拷贝过来）`

- `步骤五：修改rocketmq-consumer添加一个ObjMsgListener`

```java
/**  
 * 创建一个对象消息的监听  
 * 1.类上添加注解@Component和@RocketMQMessageListener  
 * 2.实现RocketMQListener接口，注意泛型的使用  
 */  
@Component  
@RocketMQMessageListener(topic = "powernode-obj", consumerGroup = "powernode-obj-group")  
public class ObjMsgListener implements RocketMQListener<Order> {  
  
    /**  
     * 消费消息的方法  
     *  
     * @param message  
     */  
    @Override  
    public void onMessage(Order message) {  
        System.out.println(message);  
    }  
}
```

- `步骤六：重启rocketmq-consumer后查看控制台`

对象消息已经监听到了

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/609c71502e6c49ed8f2c6aeef56e4a15.png)

### 2.2 发送集合消息

和对象消息同理，创建一个Order的集合，发送出去，监听方注意修改泛型中的类型为Object即可，这里就不做重复演示了

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac35e6e898b77eab7fc1131d2487e323.png)

## 3 发送不同消息模式

### 3.1 发送同步消息

理解为：消息由消费者发送到broker后，会得到一个确认，是具有可靠性的

这种可靠性同步地发送方式使用的比较广泛，比如：重要的消息通知，短信通知等。

我们在上面的快速入门中演示的消息，就是同步消息，即

`rocketMQTemplate.syncSend()`

`rocketMQTemplate.send()`

`rocketMQTemplate.convertAndSend()`

这三种发送消息的方法，底层都是调用syncSend，发送的是同步消息

### 3.2 发送异步消息

`rocketMQTemplate.asyncSend()`

- 步骤一：修改rocketmq-producer添加一个单元测试

```java
@Test  
void testAsync() throws IOException {  
    // 发送异步消息，发送完以后会有一个异步通知，异步消息的方法没有返回值  
    rocketMQTemplate.asyncSend("bootTestTopic", "boot的异步消息", new SendCallback() {  
        @Override  
        public void onSuccess(SendResult sendResult) {  
            //消息发送成功的回调通知  
            System.out.println("发送成功");  
        }  
  
        @Override  
        public void onException(Throwable throwable) {  
            //消息发送失败的回调通知  
            System.out.println("发送失败");  
        }  
    });  
    //异步消息发送的很快，所以要挂起  
    System.out.println("我先执行");  
    System.in.read();  
}
```

- `步骤二：运行查看控制台效果`

谁先发送打印在前面

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2fef9b3449b699803dc3d93d3aa250df.png)

### 3.3 发送单向消息

这种方式主要用在不关心发送结果的场景，这种方式吞吐量很大，但是存在消息丢失的风险，例如日志信息的发送

- `步骤：修改rocketmq-producer添加一个单元测试`

```java
//测试单向消息  
@Test  
void testOneway() {  
    // 单向消息也没有返回值和结果  
    rocketMQTemplate.sendOneWay("bootTestTopic", "boot的单向消息");  
}
```

### 3.4 发送延迟消息

- `步骤一：修改rocketmq-producer添加一个单元测试`

- 注意，这个方法不能直接放一个Object对象进去，只能先放一个Message对象进去

```java
//测试延时消息  
@Test  
void testDelay() {  
    // 构建消息对象  
    Message<String> msg = MessageBuilder.withPayload("boot的延迟消息").build();  
    // 发送一个延时消息，延迟等级为2级，也就是10s后被监听消费  
    // 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h  
    SendResult sendResult = rocketMQTemplate.syncSend("bootTestTopic", msg,2000,3);  
    System.out.println(LocalDateTime.now()+"发送消息的状态："+sendResult.getSendStatus());  
}
```

- `步骤二：运行后，查看消费者端，过了10s才被消费`

这里注意的是RocketMQ不支持任意时间的延时

只支持以下几个固定的延时等级，等级1就对应1s，以此类推，最高支持2h延迟

private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";

### 3.5 发送顺序消息

- `步骤一：使用Order表中的一个唯一区分字段`

```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class Order {  
    private String orderSn;  
    private String desc;  
    private Integer userId;  
}
```

- `步骤二：修改rocketmq-producer添加一个单元测试`

```java
// 顺序消费  
// 发送者：将相同的一组消息，发到同一个队列中去  
// 消费者：需要单线程消费，即不是并发消费的  
@Test  
void testOrder() {  
    List<Order> orders = Arrays.asList(  
            new Order(UUID.randomUUID().toString().substring(0, 5), "张三的下订单", 1),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "张三的发短信", 1),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "张三的物流", 1),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "张三的签收", 1),  
  
            new Order(UUID.randomUUID().toString().substring(0, 5), "李四的下订单",  2),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "李四的发短信",  2),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "李四的物流",  2),  
            new Order(UUID.randomUUID().toString().substring(0, 5), "李四的签收",  2)  
    );  
  
    // 我们控制流程为 下订单->发短信->物流->签收  hash的值为seq，也就是说 seq相同的会放在同一个队列里面，顺序消费  
  
    orders.forEach(order -> {  
        rocketMQTemplate.syncSendOrderly("bootTestTopic",order.getDesc(),order.getUserId().toString());  
    });  
}
```

- `步骤三：发送消息`

- `步骤四：修改rocketmq-consumer的ObjMsgListener`

 * consumeMode 指定消费类型  
	 * `CONCURRENTLY` 并发消费  
	 * `ORDERLY` 顺序消费 messages orderly. one queue, one thread  

```java
@Component  
@RocketMQMessageListener(topic = "bootTestTopic",consumerGroup = "test-boot-consumer-group"  
,consumeMode = ConsumeMode.CONCURRENTLY)  
public class SimpleMsgListener implements RocketMQListener<MessageExt> {  
    /**  
     * 这个方法就是消费者方法  
     * 如果泛型定义了固定的类型，那么消息体就是我们的参数  
     * 如果想要获取消息的其他参数/指标，我们可以将泛型定义为MessageExt类  
     *  
     * 没有报错，就是签收了消息  
     * 报错，就是拒收，就会去尝试重试  
     *   
     * consumeMode  
     * ConsumeMode.ORDERLY          顺序（单线程）消费  
     * ConsumeMode.CONCURRENTLY     并发消费  
     */  
    @Override  
    public void onMessage(MessageExt msg) {  
        String time = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss"));  
        System.out.println(Thread.currentThread().getName());  
        System.out.println(time+":---->  "+new String(msg.getBody()));  
    }  
}
```

- `步骤五：重启rocketmq-consumer`

查看控制台，消息按照我们的放入顺序进行消费了

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b0cf5cd10c36ee41cabf18878dc7b61b.png)

### 3.6 发送事务消息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58dbb098540209debded2bcb1c369c33.png)

- `步骤一：修改rocketmq-producer添加一个单元测试
`
```java
/**  
 * 测试事务消息  
 * 默认是sync（同步的）  
 * 事务消息会有确认和回查机制  
 * 事务消息都会走到同一个监听回调里面，所以我们需要使用tag或者key来区分过滤  
 *  
 * @throws Exception  
 */@Test  
public void testTrans() throws Exception {  
    // 构建消息体  
    Message<String> message = MessageBuilder.withPayload("这是一个事务消息").build();  
    // 发送事务消息（同步的） 最后一个参数才是消息主题  
    TransactionSendResult transaction = rocketMQTemplate.sendMessageInTransaction("powernode", message, "消息的参数");  
    // 拿到本地事务状态  
    System.out.println(transaction.getLocalTransactionState());  
    // 挂起jvm，因为事务的回查需要一些时间  
    System.in.read();  
}
```

- `步骤二：修改rocketmq-producer添加一个本地事务消息的监听（半消息）`

```java
/**  
 * 事务消息的监听与回查  
 * 类上添加注解@RocketMQTransactionListener 表示这个类是本地事务消息的监听类  
 * 实现RocketMQLocalTransactionListener接口  
 * 两个方法为执行本地事务，与回查本地事务  
 */  
@Component  
@RocketMQTransactionListener(corePoolSize = 4,maximumPoolSize = 8) 
public class TmMsgListener implements RocketMQLocalTransactionListener {  
  
  
    /**  
     * 执行本地事务，这里可以执行一些业务  
     * 比如操作数据库，操作成功就return RocketMQLocalTransactionState.COMMIT;  
     * 可以使用try catch来控制成功或者失败;  
     */  
    @Override  
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {  
        // 拿到消息参数  
        System.out.println(arg);  
        // 拿到消息头  
        System.out.println(msg.getHeaders());  
        // 返回状态COMMIT,UNKNOWN  
        return RocketMQLocalTransactionState.UNKNOWN;  
    }  
  
    /**  
     * 回查本地事务，只有上面的执行方法返回UNKNOWN时，才执行下面的方法 默认是1min回查  
     * 此方法为回查方法，执行需要等待一会  
     * xxx.isSuccess()  这里可以执行一些检查的方法  
     * 如果返回COMMIT，那么本地事务就算是提交成功了，消息就会被消费者看到  
     */  
    @Override  
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {  
        System.out.println(msg);  
        return RocketMQLocalTransactionState.COMMIT;  
    }  
}
```

- `步骤三：测试发送事务，建议断点启动`

1.  消息会先到事务监听类的执行方法，

2.  如果返回状态为COMMIT，则消费者可以直接监听到

3.  如果返回状态为ROLLBACK，则消息发送失败，直接回滚

4.  如果返回状态为UNKNOW，则过一会会走回查方法

5.  如果回查方法返回状态为UNKNOW或者ROLLBACK，则消息发送失败，直接回滚

6.  如果回查方法返回状态为COMMIT，则消费者可以直接监听到

## 4 消息过滤

### 4.1 tag过滤（常在消费者端过滤）

我们从源码注释得知，`tag带在主题后面用：来携带`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8b5d0fff19687ca82e4d91c405690986.png)

我们往下去看源码，在

org.apache.rocketmq.spring.support.RocketMQUtil 的getAndWrapMessage方法里面看到了具体细节，我们也知道了keys在消息头里面携带

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/211567acf5413042174f382db8e18154.png)


- `步骤一：修改rocketmq-producer添加一个单元测试`

```java
/**  
 * 发送一个带tag的消息  
 *  
 * @throws Exception  
 */@Test  
public void testTagMsg() throws Exception {  
    // 发送一个tag为java的数据  
    rocketMQTemplate.syncSend("powernode-tag:java", "我是一个带tag的消息");  
}
```

- `步骤二：发送消息`

- `步骤三：修改rocketmq-consumer添加一个TagMsgListener`

 * 类上添加注解@Component和@RocketMQMessageListener  

 * `selectorType = SelectorType.TAG`,  指定使用tag过滤。
 * 也可以使用sql92 需要在配置文件broker.conf中开启enbalePropertyFilter=true，一般不使用 
 * selectorExpression = "java"     表达式，默认是*,支持"tag1 || tag2 || tag3"  

```java 
@Component  
@RocketMQMessageListener(topic = "powernode-tag",  
        consumerGroup = "powernode-tag-group",  
        selectorType = SelectorType.TAG,  
        selectorExpression = "java"  
)  
public class TagMsgListener implements RocketMQListener<String> {  
  
    /**  
     * 消费消息的方法  
     *  
     * @param message  
     */  
    @Override  
    public void onMessage(String message) {  
        System.out.println(message);  
    }  
}
```

- `步骤四：重启rocketmq-consumer查看控制台`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1411e05ceabf815692d3ad485499f135.png)

### 4.2 Key过滤（可以在事务监听的类里面区分）

- `步骤一：修改rocketmq-producer添加一个单元测试`

- `setHeader(RocketMQHeaders.KEYS, "spring")`，固定写法

```java
/**  
 * 发送一个带key的消息,我们使用事务消息 打断点查看消息头  
 *  
 * @throws Exception  
 */@Test  
public void testKeyMsg() throws Exception {  
    // 发送一个key为spring的事务消息  
    Message<String> message = MessageBuilder.withPayload("我是一个带key的消息")  
            .setHeader(RocketMQHeaders.KEYS, "spring")  
            .build();  
    rocketMQTemplate.sendMessageInTransaction("powernode", message, "我是一个带key的消息");  
}
```

- `步骤二：断点发送这个消息，查看事务里面消息头`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9cea60684db20157f1a11c77ff9aaf64.png)



我们在mq的控制台也可以看到

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29c869a9735190872ac00f851aa7c91c.png)



## 5 消息消费两种模式

`Rocketmq消息消费的模式分为两种：负载均衡模式和广播模式`

负载均衡模式表示多个消费者交替消费同一个主题里面的消息

广播模式表示每个每个消费者都消费一遍订阅的主题的消息

- 步骤一：再搭建一个消费者rocketmq-consumer-b，依赖和配置文件和rocketmq-consumer一致，记住端口修改一下，避免占用

- 步骤二：rocketmq-consumer-b添加一个监听

```java
/**  
 * messageModel  指定消息消费的模式  
 *      CLUSTERING 为负载均衡模式  
 *      BROADCASTING 为广播模式  
 */  
@Component  
@RocketMQMessageListener(topic = "powernode",  
        consumerGroup = "powernode-group",  
        messageModel = MessageModel.CLUSTERING  
)  
public class ConsumerBListener implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        System.out.println(message);  
    }  
}
```

- 步骤三：修改rocketmq-consumer的SimpleMsgListener

```java
/**  
 * 创建一个简单消息的监听  
 * 1.类上添加注解@Component和@RocketMQMessageListener  
 * * @RocketMQMessageListener(topic = "powernode", consumerGroup = "powernode-group")  
 * topic指定消费的主题，consumerGroup指定消费组,一个主题可以有多个消费者组,一个消息可以被多个不同的组的消费者都消费  
 * 2.实现RocketMQListener接口，注意泛型的使用  
 */  
@Component  
@RocketMQMessageListener(topic = "powernode",  
        consumerGroup = "powernode-group",  
        messageModel = MessageModel.CLUSTERING)  
public class SimpleMsgListener implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        System.out.println(new Date());  
        System.out.println(message);  
    }  
}
```

- 步骤四：启动两个消费者

- 步骤五：在生产者里面添加一个单元测试并且运行

```java
/**  
 * 测试消息消费的模式  
 *  
 * @throws Exception  
 */@Test  
public void testMsgModel() throws Exception {  
    for (int i = 0; i < 10; i++) {  
        rocketMQTemplate.syncSend("powernode", "我是消息" + i);  
    }  
}
```

- 步骤六：查看两个消费者的控制台，发现是负载均衡的模式

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce1940a69582013d85bbf316294bc6ce.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/226362765a12e4246c17832d9df1ed98.png)




- 步骤七：修改两个消费者的模式为BROADCASTING

重启测试，结果是广播模式，每个消费者都消费了这些消息

项目中` 一般部署多台机器  消费者 2  -  3`   根据业务可以选择具体的模式来配置

- 总结：
	- `集群模式`：队列会被消费者分摊，队列数量>=消费者数量，消费的消费位点，mq服务器会记录处理
	- 广播模式：消息会被每个消费者处理一次，mq服务器不会记录消费位点，也不会重试