## 1 MQ引言

### 1.1 什么是MQ

- MQ(Message Queue) : 翻译为`消息队列`,通过典型的生产者和消费者模型,<font color="#00b050">生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息</font>。

- 因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入,`轻松的实现系统间解耦`。别名为`消息中间件` ,通过利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ccc96bb1a51708d69d90a83b2a0d1930.png)


### 1.2 为什么要用MQ

(1)`异步解耦`

场景：双11是购物狂节,用户下单后,订单系统需要通知库存系统,传统的做法就是订单系统调用库存系统的接口.

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2114ca887281d2cdd9dc97d3fbae1fd.png)



这种做法有一个缺点:
- `当库存系统出现故障时,订单就会失败`。
- 订单系统和库存系统高耦合. 
- 引入消息队列 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13614ddcc696a00589f114f8b5585653.png)



- 订单系统:创建订单并存入数据库,发送消息给库存系统。

- 库存系统:订阅订单系统的消息,获取下单信息,进行库操作。

就算库存系统出现故障,消息队列也能保证消息的可靠投递,不会导致消息丢失。

 (2)`削峰填谷`

一般在秒杀活动中应用广泛

场景:秒杀活动，一般会因为流量过大，导致应用挂掉,一般可以在应用中加入消息队列。   

作用:   
1. 可以控制活动人数，超过此一定阀值的订单直接丢弃

2. 可以缓解短时间的高流量压垮应用(应用程序按自己的最大处理能力获取订单) 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/62ca7a52f2ddbb004b578c5008f7ddcc.png)



### 1.3 MQ有哪些

目前常用的有`RocketMQ`，`Kafka`，`ActiveMQ`等；

`RocketMQ`是阿里开源的消息中间件，它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0242000abfad108d5e7438b398de342.png)



|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|产品|客户端SDK|协议和规范|顺序消息|预定消息|批量消息|广播消息|消息过滤器|服务器触发重新投递|消息存储|消息追溯|消息优先级|高可用性和故障转移|消息跟踪|配置|管理和操作工具|
|ActiveMQ|Java、.NET、C++ 等。|推送模型，支持OpenWire、STOMP、AMQP、MQTT、JMS|独占消费者或独占队列可以确保订购|支持的|不支持|支持的|支持的|不支持|使用 JDBC 和高性能日志支持非常快速的持久化，例如 levelDB、kahaDB|支持的|支持的|支持，取决于存储，如果使用 levelDB 则需要 ZooKeeper 服务器|不支持|默认配置为低级，用户需要优化配置参数|支持的|
|KFK|Java、Scala 等|拉模式，支持TCP|确保分区内的消息排序|不支持|支持，带有异步生产者|不支持|支持，可以使用Kafka Streams过滤消息|不支持|高性能文件存储|支持的偏移指示|不支持|支持，需要 ZooKeeper 服务器|不支持|Kafka 使用键值对格式进行配置。这些值可以从文件或以编程方式提供。|支持，使用终端命令公开核心指标|
|RocketMQ|Java、C++、Go|拉模型，支持TCP、JMS、OpenMessaging|确保消息的严格排序，并且可以优雅地横向扩展|支持的|支持，同步模式避免消息丢失|支持的|支持，基于 SQL92 的属性过滤器表达式|支持的|高性能和低延迟的文件存储|支持的时间戳和偏移量两个表示|不支持|受支持的主从模式，无需其他套件|支持的|开箱即用，用户只需要注意几个配置|支持的、丰富的 web 和终端命令来公开核心指标|

  
## 2 RocketMQ基础理论

### 2.1 RocketMQ

RocketMQ是阿里巴巴开源MQ中间件，使用Java语言开发，在阿里内部，RocketMQ承接了例如“双11”等高并发场景的消息流转，能够`处理万亿级别的消息`。

### 2.2 各角色介绍

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/381e700836643cafee598a05eb008686.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9f85323f243b1b52836dea0e29003bdb.png)



NameServer：注册中心，管理生产者、消费者和Broker

Broker：暂存和传输消息

Message Queue：消息队列，消息在Broker中的逻辑存储位置

Message：就是要传输的信息。

Producer：消息的发送者

Consumer：消息接收者，分为`拉取型消费者`和`推送型消费者`，支持`集群消费`和`广播消费`

Topic：对消息进行分类；从而实现发消息给指定的消费者；还可以设置即`tag 二级topic`

### 2.3 发送消息的策略

RocketMQ生产者`发送消息采取的是轮询制`，消息以轮询的方式发送至topic下的Message Queue中。`一个topic默认是4个消息队列`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae9dee06312f64c16768068dc95855ab.png)



### 2.4 拉取型消费者和推送型消费者

- 拉取型消费者（Pull Consumer）`主动`从消息服务器拉取信息，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。

- `推送型消费者`（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但从实现上看还是从消息服务器中拉取消息，不同于 Pull 的是 Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。`开发中都是使用这种方式`

### 2.5 集群消息和广播消息

`集群消息`

1）在集群消息模式下，每条消息`只需要投递到订阅这个topic的Consumer Group下的一个实例即可`。

- 图例为：有6个消息队列，会将这6个消息队列，平均分摊到3个消费者中（3个消费者组成消费者集群）
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f410a14ead2d0aea364f51cae77ccbbc.png)



`广播消息模式`

就是在consumer分配queue的时候，`所有consumer都分到所有的queue`。

- 图例为：有6个消息队列，3个消费者，每个消息队列都会发给每个消费者
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ef0d92293f10b40bb75d63f29812012.png)

### 2.6 Group

分为ProducerGroup，ConsumerGroup，代表某一类的生产者和消费者，一般来说同一个服务可以作为Group，同一个Group一般来说发送和消费的消息都是一样的。

`注意：某一topic下的消息可以被订阅了该topic的不同Group中的消费者消费`

### 2.7 Message Queue（也可以称为ConsumerQueue）

**Message Queue** :即消息队列,下文简称queue.

一个 Topic 下可以设置多个消息队列，发送消息时执行该消息的 Topic ，RocketMQ 会轮询该 Topic 下的所有队列将消息发出去。`一个topic默认是4个消息队列`。

可以通过如下代码修改topic默认队列数量`rocketMQTemplate.getProducer().setDefaultTopicQueueNums(8);`

### 2.8 消息的类型

1、`同步消息`：发送消息后等待Broker的响应结果，重要的消息通知，短信通知。

2、`异步消息`：发送端不等待Broker的响应，通过异步回调来处理消息发送结果

3、`单向消息`：用在不关心发送结果的场景，例如日志发送。

4、`顺序消息`：对于指定的一个Topic，生产者按照一定的先后顺序发布消息；消费者按照既定的先后顺序订阅消息，即先发布的消息一定会先被客户端接收到。分为`全局顺序消息`和`分区顺序消息`

5、`延时消息`：1h后去检查这个订单的状态

6、`事务消息`：本地事务和消息发送放到一个事务中，要么一起成功要么一起失败

## 3 RocketMQ 安装

### 3.1 3.1启动本地RocketMQ

1、`复制软件到你电脑常用软件安装目录`     

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/08a0c676b475ac945565c831c0b2cf3e.png)



注意：软件所在目录不能有空格

2、`设置系统环境变量`

注意：要先配置JDK环境变量，路径中也不能有空格。
注意：`本版本需要JDK版本为8`

计算机——右键属性——高级系统设置——环境变量——新建，变量名：ROCKETMQ_HOME，变量值为RocketMQ的解压目录。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9a55d688fe5da3ebc78bb843371af28c.png)



并添加进path变量下，这样就可以在任意路径找到我们的启动命令

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/66a6fdbf2eabb771f48b1967be74065e.png)



3、`启动nameServer`

NameServer的主要功能是为整个MQ集群提供服务协调与治理，具体就是记录维护Topic、Broker的信息，及监控Broker的运行状态。为client提供路由能力（类似于redis的redis-server）

双击start mqnamesrv.cmd ，成功后会弹出提示框，

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/166355dcaf89f5a572b8522454c30d82.png)



 4、`启动BROKER`

在cmd命令框下执行：

`start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true`

成功后会弹出提示框 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/467438864424c3e93946ed2eec798a15.png)



autoCreateTopicEnable=true加了这个属性是为了省去在控制台手动创建topic才能发消息的现象。

5、`在idea中运行rocketmq-console`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8bbd4789b3ae8bd1fa6d29afd6a1f1b2.png)



rocketmq-console这个应用运行时可能会报一些统计数据失败的错误，可以直接忽视，原因时，内部的定时任务统计数据时，topic和group创建后还没有收发过消息，导致获取统计数据失拜

### 3.2 web管理界面介绍

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e0dd709b263f94cd220bc23523327eab.png)



## 4  SpringBoot中使用RocketMQ

### 4.1 RocketMQ 发送同步、异步和单向

引入依赖

```xml
<dependency>  
    <groupId>org.apache.rocketmq</groupId>  
    <artifactId>rocketmq-spring-boot-starter</artifactId>  
    <version>2.2.0</version>  
</dependency>
```

自动配置模板类RocketMQTemplate

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9a254ffc3975613fb26810653594557a.png)



-  RocketMQTemplate提供了syncSend、asyncSend、sendOneWay、syncSendOrderly 、sendOneWayOrderly、sendMessageInTransaction等一系列方法

添加如下配置

```properties
rocketmq.name-server=127.0.0.1:9876  
rocketmq.producer.group=producer-group
```

#### 4.1.1 消息发送

```java
@SpringBootTest  
@Slf4j  
class Class30RocketMqProducerApplicationTests {  
    @Autowired  
    RocketMQTemplate rocketMQTemplate;  
  
    //发送同步消息  
    @Test  
    void syncSendMessageTest() {  
        for (int i = 0; i < 8; i++) {  
            //指定topic、tag  
            //注意Message的类是org.springframework.messaging.Message  
            Message msg = MessageBuilder.withPayload("测试同步消息"+i).build();  
//            rocketMQTemplate.getProducer().setDefaultTopicQueueNums(1);  
            SendResult sendResult =  
                    rocketMQTemplate.syncSend("topic1", msg,5000);  
  
            log.info("synSendResult:"+sendResult);  
  
        }  
    }  
  
    //发送异步消息  
    @Test  
    void asyncSendMessageTest() throws InterruptedException {  
        for (int i = 0; i < 8; i++) {  
            //指定topic、tag  
            //注意Message的类是org.springframework.messaging.Message  
            Message msg = MessageBuilder.withPayload("测试异步消息"+i).build();  
//            rocketMQTemplate.getProducer().setDefaultTopicQueueNums(1);  
            rocketMQTemplate.asyncSend("topic2", msg, new SendCallback() {  
                // 实现消息发送成功的后续处理  
                @Override  
                public void onSuccess(SendResult sendResult) {  
                    log.info("synSendResult:"+sendResult);  
                }  
                // 实现消息发送失败的后续处理  
                @Override  
                public void onException(Throwable throwable) {  
                    log.info("synSendResult:"+throwable);  
                }  
            });  
        }  
  
        //方法执行的很快，所以不会有信息显示  
        Thread.sleep(5000);  
    }  
  
    //发生单向消息  
    @Test  
    public void syncSendMessageOneWayTest(){  
        for (int i = 0; i < 5; i++) {  
            Message msg = MessageBuilder.withPayload("测试对象消息"+i).build();  
            rocketMQTemplate.sendOneWay("topic3",msg);  
        }  
//        for(int k=5;k<10;k++){  
//            Message msg = MessageBuilder.withPayload("测试同步消息"+k).build();  
//            rocketMQTemplate.sendOneWay("topic1", msg);  
//        }  
    }  
  
}
```

说明：因为之前启动Broker的时候指定了autoCreateTopicEnable=true，所以发送消息的时候，如果topic不存在则会新建该topic，此时可以通过来修改该topic下的queue数:

rocketMQTemplate.getProducer().setDefaultTopicQueueNums(8)

#### 4.1.2 消息消费

创建一个RocketMQListener

```java
@Component  
@Slf4j  
@RocketMQMessageListener(topic = "topic1",consumerGroup = "consumer-group1")  
public class Consumer1 implements RocketMQListener<String> {  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer1接收到消息："+message);  
    }  
}
```

@RocketMQMessageListener注解的参数

1. `consumerGroup` ：消费者分组，自定义即可

2. `topic` 主题：表示这个消费者，监听那个topic的消息

3. `selectorExpression`  二次过滤表达式

	默认值 ` * `  ，标识订阅所有消息，可以自定义，格式如：“tag1”，那么该消费者只会消费该topic下Tag为tag1的消息。

4. `consumeMode` 消费模式

	默认值 ConsumeMode.CONCURRENTLY 并行处理![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2bd36124fd4bf9c94115e64c33fa8bb.png)



	ConsumeMode.ORDERLY 按顺序处理
	一个线程对应一个消息队列

5. `messageModel` 消息模型

	默认值 MessageModel.CLUSTERING 集群

	也可以选MessageModel.BROADCASTING 广播

##### 4.1.2.1 集群消息模式

MessageModel.CLUSTERING:多个消费者共同消费队列消息，每个消费者处理的消息不同（本质上是每个消费者分配的队列不同）

```java
@RocketMQMessageListener(topic = "topic1",messageModel = MessageModel.CLUSTERING,consumerGroup = "consumer-group1")  
public class Consumer1 implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer1接收到消息："+message);  
    }  
}
```

##### 4.1.2.2 广播消息模式

MessageModel.BROADCASTING:消费者采用广播的方式消费消息，每个消费者消费的消息都是相同的

```java
@RocketMQMessageListener(topic = "topic1",messageModel = MessageModel.BROADCASTING,consumerGroup = "consumer-group1")  
public class Consumer1 implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer1接收到消息："+message);  
    }  
}
```

##### 4.1.2.3 并发消费模式

consumeMode = ConsumeMode.CONCURRENTLY，多线程的方式来消费不同消息队列中的消息

```java
@Slf4j  
@Component  
@RocketMQMessageListener(topic = "topic1",consumeMode = ConsumeMode.CONCURRENTLY, consumerGroup = "consumer-group1")  
public class Consumer1 implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer1接收到消息："+message);  
    }  
}
```

##### 4.1.2.4 顺序消费模式

consumeMode = ConsumeMode.ORDERLY：一个线程对应一个消息队列。

```java
@Slf4j  
@Component  
@RocketMQMessageListener(topic = "topic1",consumeMode = ConsumeMode.ORDERLY, consumerGroup = "consumer-group1")  
public class Consumer1 implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer1接收到消息："+message);  
    }  
}
```
#### 4.1.3 消息的可靠性投递

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/31934096a6d101b747f2e9903e0c5bed.png)



要想保证消息的可靠型投递，无非保证如下3个阶段的正常执行即可。

1.     生产者将消息成功投递到broker

2.     broker将投递过程的消息持久化下来

3.     消费者能从broker消费到消息

下面分别从上述3个阶段来分析：

1、`发送阶段`

（1）、发送成功

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1262481dcfd692d28e707959b0d18917.png)



（2）、发送失败

Producer程序向Broker发送消息时没有成功（比如网络抖动或broker宕机），此时`Producer会自动进行重试`，重试默认次数如下：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/911c3efe5de1094dc955294359c24aae.png)



也可以通过如下方式修改：

```java
rocketMQTemplate.getProducer().setRetryTimesWhenSendFailed(xx);
rocketMQTemplate.getProducer().setRetryTimesWhenSendAsyncFailed(xx);
```

2、`Broker存储消息阶段`

「异步刷盘」：消息被`写入内存的PAGECACHE，返回写成功状态`，当内存里的消息量积累到一定程度时，统一触发写磁盘操作，快速写入 。吞吐量高，当磁盘损坏时，会丢失消息

「同步刷盘」：消息`写入内存的PAGECACHE后，立刻通知刷盘线程刷盘`，然后等待刷盘完成，刷盘线程执行完成后唤醒等待的线程，给应用返回消息写成功的状态。吞吐量低，但不会造成消息丢失

`可以开启同步刷盘，保证消息不丢失`

3、`消息消费阶段`

（1）、消费成功

如果onMessage方法没有抛异常，则是消费成功，会告诉Broker消费成功（ack）

（2）、消费失败

如果onMessage方法抛异常，则是消费失败，会告诉Broker消费失败

如果是`集群消息`，那么会触发消费重试策略，重试最多16次，重试时间间隔如下

<font color="#00b050">10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h</font>

同一条消息失败一次将延时等级提高一次，然后再放到重试队列，重试队列就是名字为 “`%RETRY%消费组名称`” 的topic对应的重试队列中，从而被消费者再次消费

`如果16次还搞不定，那么会进入死信队列`，即topic名字为”`%DLQ%消费组名称`”的消息队列中

可以在控制台查看死信队列，进行处理

### 4.2 顺序消息

消息有序指的是可以按照消息的发送顺序来消费(FIFO)。RocketMQ可以严格的保证消息有序，可以分为`分区有序`或者`全局有序`。

全局有序：`发送和消费参与的queue只有一个`

分区有序：
	如果多个queue参与，将消息按类别发送到不同的queue，`同一类别的消息发到同一个queue`


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e48b8383ec8dc1b298307b33028e839.png)


两种情况下消费者都是一个queue只使用一个线程的方式来保证顺序消费。

#### 4.2.1 顺序消息生产

```java
//发送顺序消息  
@Test  
public void syncSendMessageOrderTest(){  
    for (int i = 0; i < 4; i++) {  
        Message msg = MessageBuilder.withPayload("测试顺序消息"+i).build();  
        //加上下面一句就是全局顺序消息的发送方式，只有一个queue
        rocketMQTemplate.getProducer().setDefaultTopicQueueNums(1); 
        SendResult sendResult =  
                rocketMQTemplate.syncSendOrderly("topic2", msg, "订单A");  
        log.info("sendResult" + sendResult);  
    }  
}
```

#### 4.2.2 顺序消费消息

指定消费模式为ConsumeMode.ORDERLY

```java
@Component  
@Slf4j  
@RocketMQMessageListener(topic = "topic2",  
        messageModel = MessageModel.CLUSTERING,  
        consumeMode = ConsumeMode.ORDERLY,  
        consumerGroup = "consumer-group2")  
public class Consumer2 implements RocketMQListener<String> {  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer2接收到消息："+message);  
    }  
}
```

### 4.3 延时消息

比如电商里，提交了一个订单就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就取消订单释放库存。

#### 4.3.1 发送延时消息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71813d208baaedde1503db11a5902312.png)



现在RocketMq并不支持任意时间的延时，需要设置几个固定的延时等级，从1s到2h分别对应着等级1到18

<font color="#00b050">1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h</font>

```java
//发送延时消息  
@Test  
public void sendDelayMessageTest(){  
    for (int i = 0; i < 1; i++) {  
        Message msg = MessageBuilder.withPayload("测试延时消息"+i).build();  
        SendResult sendResult =  
                rocketMQTemplate.syncSend("topic3", msg, 5000, 2);  
        log.info("sendResult" + sendResult);  
    }  
}
```

#### 4.3.2 消费消息

```java
@Component  
@Slf4j  
@RocketMQMessageListener(topic = "topic3",  
        consumerGroup = "consumer-group3")  
public class Consumer3 implements RocketMQListener<String> {  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer3接收到消息："+message);  
    }  
}
```
#### 4.3.3 延时消息原理

`其实就是，Broker收到了消息后延时投递到消费者能看到的queue中`。

1） 缓存延时消息  
替换主题SCHEDULE_TOPIC_XXX，不进行投递  
2）`18个Queue对应18个延时级别 ` 
3）根据延时级别放入对应的队列  
4）每个队列创建定时任务进行调度  
5）恢复到期消息重新投递到真实的Topic路由的消息队列中

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e268608485f8b1ddfc1dca27b06bf44e.png)


  
### 4.4 批量消息

批量发送消息能显著提高传递小消息的性能。限制是这些批量消息应该有`相同的topic`，而且`不能是延时消息`。此外，这一批消息的`总大小不应超过4MB`。

#### 4.4.1 发送批量消息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1dbfd9b2b6bed6d1ad276e7b87225232.png)



样例如下：

```java
//发送批量消息  
@Test  
public void sendBatchMessageTest(){  
    //指定topic，tag  
    Message msg1 = MessageBuilder.withPayload("测试批量消息1").build();  
    Message msg2 = MessageBuilder.withPayload("测试批量消息2").build();  
    Message msg3 = MessageBuilder.withPayload("测试批量消息2").build();  
    List<Message> messageList = new ArrayList<>();  
    messageList.add(msg1);  
    messageList.add(msg2);  
    messageList.add(msg3);  
    SendResult sendResult = rocketMQTemplate.syncSend("topic4", messageList);  
    log.info("sendResult: "+sendResult);  
}
```

注意：

1、同一次发送的批量消息会发到相同的queue

2、如果消息的总长度可能大于4MB时，需要把消息进行分割

#### 4.4.2 消费消息

```java
@Component  
@Slf4j  
@RocketMQMessageListener(topic = "topic4",  
        consumerGroup = "consumer-group4")  
public class Consumer4 implements RocketMQListener<String> {  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer4接收到消息："+message);  
    }  
}
```

### 4.5 事务消息

#### 4.5.1 事务消息原理

`本地事务`和`消息发送`放到一个事务中，要么一起成功要么一起失败

`这个本地事务就是自定义的业务逻辑`。      

事务消息的基本原理：

先发一个半成品消息到Broker，暂存在`半消息队列`中（`消费者看不到这个队列`）

1、如果`本地事务执行成功`，那么`半消息会被转投递到consumer queue`

2、如果`本地事务执行失败`，那么`半消息会被删除`

3、如果`本地事务执行结果未知`，那么会`触发回查机制`，可以定义回查的逻辑来决定是否转投消息到consumer queue，回查默认时间间隔是1分钟，默认回查15次，回查15次仍未成功然后会丢弃该消息。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5dc68e2faeb27c985c5455ce053c3011.png)

#### 4.5.2 事务消息生产者


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e3f9a2416d35818c6abbf687c09f2de8.png)



示例：

第一步：写发送消息的方法

```java
//发送事务消息  
@Test  
public void sendTransactionMessageTest() throws InterruptedException {  
    Message msg = MessageBuilder.withPayload("测试事务消息").build();  
    SendResult sendResult = rocketMQTemplate.sendMessageInTransaction("topic5", msg, null);  
    log.info("sendResult" + sendResult);  
}
```

注意：`事务消息不支持延时消息和批量消息`。

第二步：定义一个本地事务监听器TransactionalListener

```java
@Slf4j  
@Component  
@RocketMQTransactionListener  
public class TransactionalListener implements RocketMQLocalTransactionListener {  
  
  
    @Override  
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {  
        log.info("执行本地事務");  
        //可以拿到消息的内容  
        String str = new String((byte[]) msg.getPayload());  
        return RocketMQLocalTransactionState.UNKNOWN;  
    }  
  
    @Override  
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {  
        log.info("消息回查：");  
        return RocketMQLocalTransactionState.UNKNOWN;  
    }  
}
```

本地事务执行的结果：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c5917892d6c26b7966da411b63838de.png)



#### 4.5.3 事务消息消费者

无需特殊配置，

```java
@Slf4j  
@Component  
@RocketMQMessageListener(topic = "topic5", consumerGroup = "consumer-group5")  
public class Consumer5 implements RocketMQListener<String> {  
  
    @Override  
    public void onMessage(String message) {  
        log.info("Consumer5接收到消息："+message);  
    }  
}
```

### 4.6 如何避免消息被重复消费

消息重复的场景

1）发送时消息重复

当生产者成功发送消息到Broker，但是由于网络原因导致生产者没有收到Broker的确认应答，生产者会重发消息

2）消费时消息重复

消息者消费该消息后，未能将消费成功反馈给Broker，会触发重试机制。

答案：`做唯一性校验，相同的业务唯一ID的消息只消费一次`