## 1 RocketMQ重试机制

### 1.1 生产者重试

// 失败的情况重发3次

`producer.setRetryTimesWhenSendFailed(3);`

// 消息在1S内没有发送成功，就会重试

`producer.send(msg, 1000);`


- 一般生产者发送消息的重试我们一般不会去讨论
- 因为发送消息到mq中，出现重试是因为与mq连接失败，但是一般不会连接失败

### 1.2 消费者重试

在消费者放`return ConsumeConcurrentlyStatus.RECONSUME_LATER` 或者 `return null` 后就会执行重试

`重试的时间间隔和延迟的时间间隔一致的`，所以默认是重试16次的

上图代码中说明了，我们`在实际生产过程中，一般重试3-5次，如果还没有消费成功，则可以把消息签收了，通知人工等处理`

- 总结：
	- 自定义重试次数：`consumer.setMaxReconsumerTimes(次数)`
		- 注意：第一次不是重试，重试几次过后，重新设置了重试次数小于之前重试的次数，就不会再次重试了
		- 获取重试次数：`getReconsumeTimes`
		- 一般重试的次数，5次就够了
	- 重试失败后的消息默认处理方式：这个消息称为死信消息，进入死信主题的队列中
		- 并发模式下（16次）、顺序模式下（int最大值次）
		- 死信主题为：`%DLQ%消费者组名称`，里面只有1个队列
	- 消息处理失败的解决方案：
		- 方案一：直接监听死信主题中的消息，执行人工处理，保存mysql或文件的业务逻辑，但是如果有多个主题，就会有多个死信主题，不方便
		- 方案二：在正常的监听逻辑中，catch正常业务代码的报错，然后再重试的时候，判断重试次数，超过了重试次数，catch中执行死信的处理逻辑

```java
/**  
 * 测试消费者  
 *  
 * @throws Exception  
 */@Test  
public void testConsumer() throws Exception {  
    // 创建默认消费者组  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr("localhost:9876");  
    // 订阅一个主题来消费   *表示没有过滤参数 表示这个主题的任何消息  
    consumer.subscribe("TopicTest", "*");  
    // 注册一个消费监听  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,  
                                                        ConsumeConcurrentlyContext context) {  
            try {  
                // 这里执行消费的代码  
                System.out.println(Thread.currentThread().getName() + "----" + msgs);  
                // 这里制造一个错误  
                int i = 10 / 0;  
            } catch (Exception e) {  
                // 出现问题 判断重试的次数  
                MessageExt messageExt = msgs.get(0);  
                // 获取重试的次数 失败一次消息中的失败次数会累加一次  
                int reconsumeTimes = messageExt.getReconsumeTimes();  
                if (reconsumeTimes >= 3) {  
                    // 则把消息确认了，可以将这条消息记录到日志或者数据库 通知人工处理  
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
                } else {  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;  
                }  
            }  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

## 2 RocketMQ死信消息

当消费重试到达阈值以后，消息不会被投递给消费者了，而是进入了死信队列

当一条消息初次消费失败，RocketMQ会自动进行消息重试，达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息。此时，该消息不会立刻被丢弃，而是将其发送到该消费者对应的特殊队列中，这类消息称为死信消息（Dead-Letter Message），存储死信消息的特殊队列称为死信队列（Dead-Letter Queue），死信队列是死信Topic下分区数唯一的单独队列。如果产生了死信消息，那对应的ConsumerGroup的死信Topic名称为%DLQ%ConsumerGroupName，死信队列的消息将不会再被消费。可以利用RocketMQ Admin工具或者RocketMQ Dashboard上查询到对应死信消息的信息。我们也可以去监听死信队列，然后进行自己的业务上的逻辑

### 2.1 消息生产者

```java
@Test  
public void testDeadMsgProducer() throws Exception {  
    DefaultMQProducer producer = new DefaultMQProducer("dead-group");  
    producer.setNamesrvAddr("localhost:9876");  
    producer.start();  
    Message message = new Message("dead-topic", "我是一个死信消息".getBytes());  
    producer.send(message);  
    producer.shutdown();  
}
```

### 2.2 消息消费者

```java
@Test  
public void testDeadMsgConsumer() throws Exception {  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("dead-group");  
    consumer.setNamesrvAddr("localhost:9876");  
    consumer.subscribe("dead-topic", "*");  
    // 设置最大消费重试次数 2 次  
    consumer.setMaxReconsumeTimes(2);  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
            System.out.println(msgs);  
            // 测试消费失败  
            return ConsumeConcurrentlyStatus.RECONSUME_LATER;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

### 2.3 死信消费者

注意权限问题

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/257a1f623d44913159bc6ee6bf4d8f02.png)

```java
@Test  
public void testDeadMq() throws  Exception{  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("dead-group");  
    consumer.setNamesrvAddr("localhost:9876");  
    // 消费重试到达阈值以后，消息不会被投递给消费者了，而是进入了死信队列  
    // 队列名称 默认是 %DLQ% + 消费者组名  
    consumer.subscribe("%DLQ%dead-group", "*");  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
            System.out.println(msgs);  
            // 处理消息 签收了  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

### 2.4 控制台显示

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/49bad0ae5a56aa43f5e0aaa9c21bdaae.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5f89754e0f5ef8463b0bbd546b4eb3e4.png)

## 3 RocketMQ消息重复消费问题【重点】

### 3.1 为什么会出现重复消费问题呢？

BROADCASTING(广播)模式下，所有注册的消费者都会消费，`而这些消费者通常是集群部署的一个个微服务，这样就会多台机器重复消费`，当然这个是根据需要来选择。

CLUSTERING（负载均衡）模式下，如果`一个topic被多个consumerGroup消费，也会重复消费`。

即使是在CLUSTERING模式下，同一个consumerGroup下，一个队列只会分配给一个消费者，看起来好像是不会重复消费。
但是，有个`特殊情况`：

- `情况一`：一个消费者新上线后，同组的所有消费者要`重新负载均衡`（反之一个消费者掉线后，也一样）。一个队列所对应的新的消费者要获取之前消费的offset（偏移量，也就是消息消费的点位），`此时之前的消费者可能已经消费了一条消息，但是并没有把offset提交给broker，那么新的消费者可能会重新消费一次`。

- `情况二`：虽然orderly模式是前一个消费者先解锁，后一个消费者加锁再消费的模式，比起concurrently要严格了，但是`加锁的线程和提交offset的线程不是同一个，所以还是会出现极端情况下的重复消费`

- `情况三`：还有在发送批量消息的时候，会被当做一条消息进行处理，那么如果批量消息中有一条业务处理成功，其他失败了，还是会被重新消费一次。

- `解决方式`：那么如果在CLUSTERING（负载均衡）模式下，并且在同一个消费者组中，不希望一条消息被重复消费，改怎么办呢？我们可以想到`去重操作，找到消息唯一的标识，可以是msgId也可以是你自定义的唯一的key，这样就可以去重了`

### 3.2 解决方案

使用去重方案解决，例如`将消息的唯一标识存起来，然后每次消费之前先判断是否存在这个唯一标识`，如果存在则不消费，如果不存在则消费，并且消费以后将这个标记保存。

想法很好，但是消息的体量是非常大的，可能在生产环境中会到达上千万甚至上亿条，那么我们该`如何选择一个容器来保存所有消息的标识`，并且又可以快速的判断是否存在呢？

可以选择 `redis容器` 或 `mysql容器` 或 `布隆过滤`

### 3.3 方案一：mysql

方案一：`mysql容器`
- `我们可以抓住幂等性来控制消息的重复消费`
- 先建立一个去重表，因为针对唯一id的索引的插入的默认加锁的
- 当我们获取消息队列的消息后，获取消息的key，这个key对应去重表中的唯一索引
- 插入成功，执行业务
- 插入失败，则返回
- 这样就可以避免消息的重复消费问题

- 为什么是直接插入，而不是先查询key，再判断存不存在呢？
	- 因为先查询，后判断，这个操作存在原子性问题
	- 假如高并发的情况下，多个线程都通过了查询操作（mysql中查询是不加写锁的），那么就都会进行判断，最后产生高并发问题
	- 所以我们直接插入（mysql插入是加写锁的），插入的过程是有序的，通过插入的结果，来判断key是否存在
	- 如果插入失败，说明消息来过了，我们可以直接签收消息

- 如果插入数据的业务，后面的代码逻辑出错怎么办？
	- 会触发重试机制
	- 我们先要在catch中，将去重表中的这个key的记录删除

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2bba331b7ce72d4b839302739fb50214.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9dbbe176fef458ced26f7dd731819a39.png)



#### 3.3.1 创建表

- `设计key的唯一索引`

```mysql
CREATE TABLE `order_oper_log` (
  `id` int NOT NULL AUTO_INCREMENT,
  `type` int DEFAULT NULL,
  `order_sn` varchar(255) DEFAULT NULL,
  `user` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `order_sn` (`order_sn`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
#### 3.3.2 导入依赖

```xml
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>8.0.20</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-jdbc</artifactId>  
</dependency>
```

#### 3.3.3 测试

- 测试的逻辑是 发送两条消息一样，key一样的消息
- 因为，通过增加consumer来改变队列的消息发布，不好实现

```java
@SpringBootTest  
public class MysqlType {  
    @Autowired  
    JdbcTemplate jdbcTemplate;  
    @Test  
    public void testRepeatProducer() throws Exception {  
        DefaultMQProducer producer = new DefaultMQProducer("repeat-producer-group");  
        producer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
        producer.start();  
  
        String key = UUID.randomUUID().toString();  
        System.out.println("key = " + key);  
        //测试，发送两条key一样的消息  
        Message m1 = new Message("repeatTopic", null, key, "扣减库存-1".getBytes());  
        Message m2 = new Message("repeatTopic", null, key, "扣减库存-1".getBytes());  
  
        producer.send(m1);  
        producer.send(m2);  
        System.out.println("发送成功");  
        producer.shutdown();  
    }  
  
    @Test  
    public void testRepeatConsumer() throws Exception {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("repeat-consumer-group");  
        consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
        consumer.subscribe("repeatTopic","*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
            @Override  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
                String msg = new String(list.get(0).getBody());  
                String key = list.get(0).getKeys();  
                //插入数据库，因为我们给key加了唯一索引  
                String sql = "insert into order_oper_log(type,order_sn,user) values(1,?,'wangwu')";  
                int update = jdbcTemplate.update(sql, key);  
                // 新增 要么成功 要么报错  
                // 修改 要么成功 要么返回0 要么报错  
                // 处理业务逻辑  
                System.out.println(msg);  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
            }  
        });  
        consumer.start();  
        System.in.read();  
    }  
}
```

- 注意，使用了JdbcTemplate，异步执行，其他线程执行sql的时候，在控制台面板不会报错，会显示不出来
- 我们直接使用原生的方式
```java
@SpringBootTest  
public class MysqlType {  
    @Autowired  
    JdbcTemplate jdbcTemplate;  
    @Test  
    public void testRepeatProducer() throws Exception {  
        DefaultMQProducer producer = new DefaultMQProducer("repeat-producer-group");  
        producer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
        producer.start();  
  
        String key = UUID.randomUUID().toString();  
        System.out.println("key = " + key);  
        //测试，发送两条key一样的消息  
        Message m1 = new Message("repeatTopic", null, key, "扣减库存-1".getBytes());  
        Message m2 = new Message("repeatTopic", null, key, "扣减库存-1".getBytes());  
  
        producer.send(m1);  
        producer.send(m2);  
        System.out.println("发送成功");  
        producer.shutdown();  
    }  
  
    @Test  
    public void testRepeatConsumer() throws Exception {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("repeat-consumer-group");  
        consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
        consumer.subscribe("repeatTopic","*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
            @Override  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
                String msg = new String(list.get(0).getBody());  
                String key = list.get(0).getKeys();  
  
                //插入数据库，因为我们给key加了唯一索引  
                String sql = "insert into order_oper_log(type,order_sn,user) values(1,?,'wangwu')";  
  
                Connection connection = null;  
                int update = 0;  
                try {  
                    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydata?serverTimezone=GMT%2B8","root","abc123");  
                    PreparedStatement ps = connection.prepareStatement(sql);  
                    //填充占位符  
                    ps.setObject(1,key);  
                    //执行  
                    update = ps.executeUpdate();  
                } catch (SQLException e) {  
                    if (e instanceof SQLIntegrityConstraintViolationException){  
                        System.out.println(Thread.currentThread().getName()+"唯一索引异常");  
                        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS; 
                    }  
                    throw new RuntimeException(e);  
                }  
  
  //处理业务逻辑
                System.out.println(Thread.currentThread().getName()+update);  
  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
            }  
        });  
        consumer.start();  
        System.in.read();  
    }  
}
```

因为，消息出现了异常，我们没有确认异常后的消息处理方式，所以触发了重试机制

添加了上述一段话后
### 3.4 方案二：布隆过滤

我们还可以`选择布隆过滤器(BloomFilter)`

布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难（哈希冲突）。

在hutool的工具中我们可以直接使用，当然你自己使用redis的bitmap类型手写一个也是可以的 [https://hutool.cn/docs/#/bloomFilter/%E6%A6%82%E8%BF%B0](https://hutool.cn/docs/#/bloomFilter/%E6%A6%82%E8%BF%B0)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2a1fd84c01a344adb0fee94b225d8cc.png)

#### 3.4.1 测试生产者

```java
@Test  
public void testRepeatProducer() throws Exception {  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("test-group");  
    // 设置nameServer地址  
    producer.setNamesrvAddr("localhost:9876");  
    // 启动实例  
    producer.start();  
    // 我们可以使用自定义key当做唯一标识  
    String keyId = UUID.randomUUID().toString();  
    System.out.println(keyId);  
    Message msg = new Message("TopicTest", "tagA", keyId, "我是一个测试消息".getBytes());  
    SendResult send = producer.send(msg);  
    System.out.println(send);  
    // 关闭实例  
    producer.shutdown();  
}
```

#### 3.4.2 添加hutool的依赖

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.11</version>
</dependency>
```

#### 3.4.3 测试消费者

```java
/**  
 * 在boot项目中可以使用@Bean在整个容器中放置一个单利对象  
 */  
public static BitMapBloomFilter bloomFilter = new BitMapBloomFilter(100);  
  
@Test  
public void testRepeatConsumer() throws Exception {  
    // 创建默认消费者组  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");  
    consumer.setMessageModel(MessageModel.BROADCASTING);  
    // 设置nameServer地址  
    consumer.setNamesrvAddr("localhost:9876");  
    // 订阅一个主题来消费   表达式，默认是*  
    consumer.subscribe("TopicTest", "*");  
    // 注册一个消费监听 MessageListenerConcurrently是并发消费  
    // 默认是20个线程一起消费，可以参看 consumer.setConsumeThreadMax()    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,  
                                                        ConsumeConcurrentlyContext context) {  
            // 拿到消息的key  
            MessageExt messageExt = msgs.get(0);  
            String keys = messageExt.getKeys();  
            // 判断是否存在布隆过滤器中  
            if (bloomFilter.contains(keys)) {  
                // 直接返回了 不往下处理业务  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
            }  
            // 这个处理业务，然后放入过滤器中  
            // do sth...  
            bloomFilter.add(keys);  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```
