# 一、发送同步消息【重点】

上面的快速入门就是发送同步消息，发送过后会有一个返回值，也就是mq服务器接收到消息后返回的一个确认，这种方式非常安全，但是性能上并没有这么高，而且在mq集群中，也是要等到所有的从机都复制了消息以后才会返回，所以针对重要的消息可以选择这种方式

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f013dd07f0dac0bf43734c982a1df13f.png)

- 同步消息的特点，`生产者发送消息后会等待消息发送成功的确认`
	- 单mq服务器接到消息后会返回确认
	- 集群mq服务器，等到所有的从机都复制了消息后才返回确认
	- 非常安全
	- `针对主要的消息可以选择`
	- `性能上比较差`

# 二、发送异步消息【重点】

异步消息通常用在`对响应时间敏感的业务场景`，即发送端不能容忍长时间地等待Broker的响应。发送完以后会有一个异步消息通知

`生产者发送消息后不会等待消息发送成功的确认，确认通过异步的方式发回给生产者`
## 2.1 异步消息生产者

`SendCallback`是RocketMQ中的一个接口，用于发送消息后异步回调通知发送结果。

```java
@Test  
public void testAsyncProducer() throws Exception{  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("async-producer-group");  
    // 设置nameServer地址  
    producer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 启动实例  
    producer.start();  
    Message msg = new Message("asyncTopic","我是一个异步消息".getBytes());  
    // 发送异步的消息  
    producer.send(msg, new SendCallback() {  
        @Override  
        public void onSuccess(SendResult sendResult) {  
            System.out.println("发送成功");  
        }  
  
        @Override  
        public void onException(Throwable throwable) {  
            System.out.println("发送失败");  
        }  
    });  
    System.out.println("看看谁先执行，异步的，我先执行");  
    // 挂起jvm , 因为回调是异步的，不然测试不出来  
    System.in.read();  
    // 关闭生产者  
    producer.shutdown();  
}
```

## 2.2 异步消息消费者

```java
@Test  
public void testAsyncConsumer() throws Exception{  
    // 创建默认的消费者  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("async-consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 订阅一个主题来消费  
    consumer.subscribe("asyncTopic","*");  
    // 注册一个消费监听，并发消费  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            // 这里执行消费的代码， 默认是多线程消费  
            System.out.println(Thread.currentThread().getName() + "---" + new String(list.get(0).getBody()));  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

# 三、发送单向消息【一般】

这种方式`主要用在不关心发送结果的场景`
这种方式`吞吐量很大，但是存在消息丢失的风险`
例如`日志信息的发送`
## 3.1 单向消息生产者

- `方法sendOneway（message）`
```java
@Test  
public void testOneWayProducer() throws Exception{  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("oneway-consumer-group");  
    // 注册服务中心  
    producer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 启动生产者  
    producer.start();  
    Message msg = new Message("onewayTopic","*","这是一条单向消息".getBytes());  
    // 发送单向消息  
    producer.sendOneway(msg);  
    // 关闭实例  
    producer.shutdown();  
}
```
## 3.2 单向消息消费者

消费者和上面异步消费者一样

# 四、发送延迟消息【重点】

消息放入mq后，`过一段时间，才会被监听到，然后消费`

比如下订单业务，提交了一个订单就可以发送一个延时消息，30min后去检查这个订单的状态，如果还是未付款就取消订单释放库存
## 4.1 延迟消息生产者

- `使用Message的setDelayLevel方法`
- messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```java
@Test  
public void testDelayProducer() throws Exception{  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("delay-producer-group");  
    // 设置nameServer地址  
    producer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 启动实例  
    producer.start();  
    Message msg = new Message("delayTopic","*","这是一条延时消息".getBytes());  
    // 设定一个延迟等级  
    // messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h  
    msg.setDelayTimeLevel(2);  
    // 发送消息  
    producer.send(msg);  
    System.out.println("发送时间为---"+LocalDateTime.now());  
    // 关闭实例  
    producer.shutdown();  
}
```

## 4.2 延迟消息消费者

```java
@Test  
public void testDelayConsumer() throws Exception{  
    // 创建默认的消费者  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("delay-consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 订阅一个主题来消费  
    consumer.subscribe("delayTopic","*");  
    // 注册一个消费监听，并发消费  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            // 这里执行消费的代码， 默认是多线程消费  
            System.out.println("收到消息了，收到的时间"+LocalDateTime.now());  
            System.out.println(Thread.currentThread().getName() + "---" + new String(list.get(0).getBody()));  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

这里注意的是`RocketMQ不支持任意时间的延时`

只支持以下几个固定的延时等级，等级1就对应1s，以此类推，最高支持2h延迟

private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";

注意：时间是有一点点误差的

# 五、发送顺序消息【一般】

消息有序指的是可以`按照消息的发送顺序来消费(FIFO)`
RocketMQ可以严格的保证消息有序，可以分为：`分区有序`或者`全局有序`

可能大家会有疑问，mq不就是FIFO吗？

rocketMq的broker的机制，导致了rocketMq会有这个问题
即发送的消息被不同的queue所获取，那么`消费者在并发模式下，消费消息的时候，所获取的消息并不是有序的`

消费者消费的模式还有 `并发消费模式` 和 `顺序消费模式(单线程)`
`注意和 集群消费机制 和 广播消费机制的区别`

因为一个broker中对应了四个queue

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c95edfcd162eb8e7f3a1d58e6fc14c5c.png)

顺序消费的原理解析，在默认的情况下消息发送会采取Round Robin轮询方式把消息发送到不同的queue(分区队列)；而消费消息的时候从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。

但是如果`1. 控制发送的顺序消息只依次发送到同一个queue中`，`2. 消费的时候只从这个queue上依次拉取`，则就保证了顺序。

当发送和消费参与的queue只有一个，则是`全局有序`；
如果多个queue参与，则为`分区有序`，
即相对每个queue，消息都是有序的。

下面用订单进行分区有序的示例。一个订单的顺序流程是：下订单、发短信通知、物流、签收。订单顺序号相同的消息会被先后发送到同一个队列中，消费时，同一个顺序获取到的肯定是同一个队列。

## 5.1 场景分析

模拟一个订单的发送流程，创建两个订单，发送的消息分别是

订单号111 消息流程 下订单->物流->签收

订单号112 消息流程 下订单->物流->拒收

## 5.2 创建一个订单对象

```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class MsgModel {  
    private String orderSn;  
    private Integer userId;  
    private String desc; //下单 短信 物流  
  
    // xxxx  
}
```

## 5.3 顺序消息生产者


- 发送消息的时候，使用send方法的上述参数来传参
	- msg：需要传递的消息
	- 队列选择器：复写其中的方法，返回队列的id号即可
	- arg参数：传递到上面复写方法的Object o这个参数去的

```java
private List<MsgModel> msgModels = Arrays.asList(  
        new MsgModel("qwer",1,"下单"),  
        new MsgModel("qwer",1,"短信"),  
        new MsgModel("qwer",1,"物流"),  
  
        new MsgModel("zxcv",2,"下单"),  
        new MsgModel("zxcv",2,"短信"),  
        new MsgModel("zxcv",2,"物流")  
);

@Test  
public void testOrderlyTProducer() throws Exception{  
    DefaultMQProducer producer = new DefaultMQProducer("order-producer-group");  
    producer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
    producer.start();  
    //发送顺序消息，发送时要确保有序，并且同一个用户发到同一个队列中  
    msgModels.forEach(msgModel -> {  
        Message msg = new Message("orderTopic",msgModel.toString().getBytes());  
        try {  
            // 发 相同的订单号 到相同的队列  
            producer.send(msg, new MessageQueueSelector() {  
                @Override  
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {  
                    // 选择队列  
                    int hashCode = o.toString().hashCode();  
                    // 取模，不同的id获得的hashCode和队列数取模，是不一致的  
                    // 可以完成轮询的效果  
                    int i = hashCode % list.size();  
                    return list.get(i);  
                }  
            },msgModel.getOrderSn());  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
    });  
  
    producer.shutdown();  
}
```
## 5.4 顺序消息消费者，测试时等一会即可有延迟

- 发送消息使用 MessageListenerOrderly 顺序模式
```java
@Test  
public void testOrderlyConsumer() throws Exception{  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("order-consumer-group");  
    consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
  
    consumer.subscribe("orderTopic","*");  
    // 选择 MessageListenerOrderly 顺序模式  
    consumer.registerMessageListener(new MessageListenerOrderly() {  
        @Override  
        public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {  
            System.out.println("线程的id：" + Thread.currentThread().getId());  
            System.out.println(new String(list.get(0).getBody()));  
            return ConsumeOrderlyStatus.SUCCESS;  
        }  
    });  
  
    consumer.start();  
    System.in.read();  
}
```

- 关于获取消息失败的情况
	- 并发模式下，会重试16次，仍然失败会把消息放入死信队列中
	- 顺序模式下，会无限重试Integer.MAX_Value


# 六、发送批量消息

Rocketmq可以一次性发送一组消息，那么这一组消息会被当做一个消息消费

注意，`这一组消息会放在同一个队列中`
## 6.1 批量消息生产者


```java
@Test  
public void testBatchProducer() throws Exception{  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("batch-producer-group");  
    // 设置nameServer  
    producer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 启动实例  
    producer.start();  
    // 创建消息的集合  
    List<Message> list = Arrays.asList(  
            new Message("batchTopic","*","我是一组消息的A消息".getBytes()),  
            new Message("batchTopic","*","我是一组消息的B消息".getBytes()),  
            new Message("batchTopic","*","我是一组消息的C消息".getBytes())  
    );  
    // 发送消息  
    producer.send(list);  
    // 关闭实例  
    producer.shutdown();  
}
```

## 6.2 批量消息消费者

```java
@Test  
public void testBatchConsumer() throws Exception{  
    // 创建默认的消费者  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("batch-consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 订阅主题  
    consumer.subscribe("batchTopic","*");  
    // 注册一个消费监听，并发监听  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            for (MessageExt msg:list){  
                System.out.println(new String(msg.getBody()));  
            }  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    // 启动实例  
    consumer.start();  
    System.in.read();  
}
```

不论是单条数据的去读，还是遍历list中的数据去读，最后还是通过多线程并发的情况下读到多个数据；
# 七、发送事务消息【不重要】

- 可以被分布式事务的Seata所替代

## 7.1 事务消息的发送流程

它可以被认为是一个`两阶段的提交消息实现`，以`确保分布式系统的最终一致性`。事务性消息确保本地事务的执行和消息的发送可以原子地执行。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ec6e700dde38522c135c981330811ca3.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f589be2de5f49a8805d90972dc9ab3a9.png)



上图说明了事务消息的大致方案，其中分为两个流程：正常事务消息的发送及提交、事务消息的补偿流程。

`事务消息发送及提交`

1.  发送消息（half消息）。

2.  服务端响应消息写入结果。

3.  根据发送结果执行本地事务（如果写入失败，此时half消息对业务不可见，本地逻辑不执行）。

4.  根据本地事务状态执行Commit或Rollback（Commit操作生成消息索引，消息对消费者可见）

`事务补偿`

1.  对没有Commit/Rollback的事务消息（pending状态的消息），从服务端发起一次“回查”

2.  Producer收到回查消息，检查回查消息对应的本地事务的状态

3.  根据本地事务状态，重新Commit或者Rollback

其中，补偿阶段用于解决消息UNKNOW或者Rollback发生超时或者失败的情况。

`事务消息状态`

事务消息共有三种状态，提交状态、回滚状态、中间状态：

TransactionStatus.CommitTransaction: 提交事务，它允许消费者消费此消息。

TransactionStatus.RollbackTransaction: 回滚事务，它代表该消息将被删除，不允许被消费。

TransactionStatus.Unknown: 中间状态，它代表需要检查消息队列来确定状态。

## 7.2 事务消息生产者

```java
/**  
 * TransactionalMessageCheckService的检测频率默认1分钟，可通过在broker.conf文件中设置transactionCheckInterval的值来改变默认值，单位为毫秒。  
 * 从broker配置文件中获取transactionTimeOut参数值。  
 * 从broker配置文件中获取transactionCheckMax参数值，表示事务的最大检测次数，如果超过检测次数，消息会默认为丢弃，即回滚消息。  
 *  
 * @throws Exception  
 */@Test  
public void testTransactionProducer() throws Exception {  
    // 创建一个事务消息生产者  
    TransactionMQProducer producer = new TransactionMQProducer("test-group");  
    producer.setNamesrvAddr("localhost:9876");  
    // 设置事务消息监听器  
    producer.setTransactionListener(new TransactionListener() {  
        // 这个是执行本地业务方法  
        @Override  
        public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {  
            System.out.println(new Date());  
            System.out.println(new String(msg.getBody()));  
            // 这个可以使用try catch对业务代码进行性包裹  
            // COMMIT_MESSAGE 表示允许消费者消费该消息  
            // ROLLBACK_MESSAGE 表示该消息将被删除，不允许消费  
            // UNKNOW表示需要MQ回查才能确定状态 那么过一会 代码会走下面的checkLocalTransaction(msg)方法  
            return LocalTransactionState.UNKNOW;  
        }  
  
        // 这里是回查方法 回查不是再次执行业务操作，而是确认上面的操作是否有结果  
        // 默认是1min回查 默认回查15次 超过次数则丢弃打印日志 可以通过参数设置  
        // transactionTimeOut 超时时间  
        // transactionCheckMax 最大回查次数  
        // transactionCheckInterval 回查间隔时间单位毫秒  
        // 触发条件  
        // 1.当上面执行本地事务返回结果UNKNOW时,或者下面的回查方法也返回UNKNOW时 会触发回查  
        // 2.当上面操作超过20s没有做出一个结果，也就是超时或者卡主了，也会进行回查  
        @Override  
        public LocalTransactionState checkLocalTransaction(MessageExt msg) {  
            System.err.println(new Date());  
            System.err.println(new String(msg.getBody()));  
            // 这里  
            return LocalTransactionState.UNKNOW;  
        }  
    });  
    producer.start();  
    Message message = new Message("TopicTest2", "我是一个事务消息".getBytes());  
    // 发送消息  
    producer.sendMessageInTransaction(message, null);  
    System.out.println(new Date());  
    System.in.read();  
}
```

## 7.3 事务消息消费者

```java
@Test  
public void testTransactionConsumer() throws Exception {  
    // 创建默认消费者组  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr("localhost:9876");  
    // 订阅一个主题来消费   *表示没有过滤参数 表示这个主题的任何消息  
    consumer.subscribe("TopicTest2", "*");  
    // 注册一个消费监听 MessageListenerConcurrently是并发消费  
    // 默认是20个线程一起消费，可以参看 consumer.setConsumeThreadMax()    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,  
                                                        ConsumeConcurrentlyContext context) {  
            // 这里执行消费的代码 默认是多线程消费  
            System.out.println(Thread.currentThread().getName() + "----" + new String(msgs.get(0).getBody()));  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

## 7.4 测试结果

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35f72ca649b73564c88536092e4bbe1f.png)
# 八、发送带标签的消息，消息过滤

Rocketmq提供消息过滤功能，通过tag或者key进行区分

我们往一个主题里面发送消息的时候，根据业务逻辑，可能需要区分，比如带有tagA标签的被A消费，带有tagB标签的被B消费，还有在事务监听的类里面，只要是事务消息都要走同一个监听，我们也需要通过过滤才区别对待

## 8.1 标签消息生产者

```java
@Test  
public void testTagProducer() throws Exception {  
    DefaultMQProducer producer = new DefaultMQProducer("tag-producer-group");  
    producer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
    producer.start();  
    Message msg1 = new Message("tagTopic", "vip1", "vip1的文章".getBytes());  
    Message msg2 = new Message("tagTopic", "vip2", "vip2的文章".getBytes());  
    producer.send(msg1);  
    producer.send(msg2);  
    System.out.println("发送成功");  
    producer.shutdown();  
}
```

## 8.2 标签消息消费者

```java
//注意订阅关系一致性,tag不一致就不能是一个consumer-group了  
@Test  
public void testTagConsumer1() throws Exception {  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("tag-consumer-group-1");  
    consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
    consumer.subscribe("tagTopic","vip1");  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            System.out.println("我是vip1的消费者,我正在消费消息"+new String(list.get(0).getBody()));  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
  
    consumer.start();  
    System.in.read();  
  
}  
  
@Test  
public void testTagConsumer2() throws Exception {  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("tag-consumer-group-2");  
    consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR);  
    consumer.subscribe("tagTopic","vip1 || vip2");  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            System.out.println("我是vip2的消费者,我正在消费消息"+new String(list.get(0).getBody()));  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
  
    consumer.start();  
    System.in.read();  
  
}
```

## 8.3 什么时候该用 Topic，什么时候该用 Tag？

总结：不同的业务应该使用不同的Topic如果是相同的业务里面有不同表的表现形式，那么我们要使用tag进行区分

可以从以下几个方面进行判断：

1. `消息类型是否一致`：如普通消息、事务消息、定时（延时）消息、顺序消息，<font color="#00b050">不同的消息类型使用不同的 Topic，无法通过 Tag 进行区分</font>。

2. `业务是否相关联`：没有直接关联的消息，如淘宝交易消息，京东物流消息使用不同的 Topic 进行区分；而同样是天猫交易消息，电器类订单、女装类订单、化妆品类订单的消息可以用 Tag 进行区分。

3. `消息优先级是否一致`：如同样是物流消息，盒马必须小时内送达，天猫超市 24 小时内送达，淘宝物流则相对会慢一些，<font color="#00b050">不同优先级的消息用不同的 Topic 进行区分</font>。

4. `消息量级是否相当`：有些业务消息虽然量小但是实时性要求高，如果跟某些万亿量级的消息使用同一个 Topic，则有可能会因为过长的等待时间而“饿死”，此时需要<font color="#00b050">将不同量级的消息进行拆分，使用不同的 Topic</font>。

`总的来说`，针对消息分类，您可以选择创建多个 Topic，或者在同一个 Topic 下创建多个 Tag。但通常情况下，`不同的 Topic 之间的消息没有必然的联系`，而 `Tag 则用来区分同一个 Topic 下相互关联的消息`，例如全集和子集的关系、流程先后的关系。

# 九、消息的Key【重点】

在rocketmq中的消息，`默认会有一个messageId当做消息的唯一标识`，我们`也可以给消息携带一个key`，`用作唯一标识或者业务标识`，包括在控制面板查询的时候也可以使用messageId或者key来进行查询

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/21f7971766becda32adbcacf9ae01fe1.png)

## 9.1 带key消息生产者

```java
@Test  
public void testKeyProducer() throws Exception {  
    // 创建默认的生产者  
    DefaultMQProducer producer = new DefaultMQProducer("test-group");  
    // 设置nameServer地址  
    producer.setNamesrvAddr("localhost:9876");  
    // 启动实例  
    producer.start();  
    Message msg = new Message("TopicTest","tagA","key", "我是一个带标记和key的消息".getBytes());  
    SendResult send = producer.send(msg);  
    System.out.println(send);  
    // 关闭实例  
    producer.shutdown();  
}
```

## 9.2 带key消息消费者

```java
@Test  
public void testKeyConsumer() throws Exception {  
    // 创建默认消费者组  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");  
    // 设置nameServer地址  
    consumer.setNamesrvAddr("localhost:9876");  
    // 订阅一个主题来消费   表达式，默认是*,支持"tagA || tagB || tagC" 这样或者的写法 只要是符合任何一个标签都可以消费  
    consumer.subscribe("TopicTest", "tagA || tagB || tagC");  
    // 注册一个消费监听 MessageListenerConcurrently是并发消费  
    // 默认是20个线程一起消费，可以参看 consumer.setConsumeThreadMax()    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,  
                                                        ConsumeConcurrentlyContext context) {  
            // 这里执行消费的代码 默认是多线程消费  
            System.out.println(Thread.currentThread().getName() + "----" + new String(msgs.get(0).getBody()));  
            System.out.println(msgs.get(0).getTags());  
            System.out.println(msgs.get(0).getKeys());  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
        }  
    });  
    consumer.start();  
    System.in.read();  
}
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8dc44f36af475bdca0d1574474ee606.png)

