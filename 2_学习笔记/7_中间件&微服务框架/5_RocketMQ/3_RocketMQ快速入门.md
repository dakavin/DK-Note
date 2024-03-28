
RocketMQ提供了发送多种发送消息的模式，例如`同步消息，异步消息，顺序消息，延迟消息，事务消息`等，我们一一学习
## 1.1 消息发送和监听的流程

我们`先搞清楚消息发送和监听的流程`，然后我们在开始敲代码
### 1.1.1 消息生产者

1. 创建消息生产者producer，并制定生产者组名

2. 指定Nameserver地址

3. 启动producer

4. 创建消息对象，指定主题Topic、Tag和消息体等

5. 发送消息

6. 关闭生产者producer

### 1.1.2 消息消费者

1. 创建消费者consumer，制定消费者组名

2. 指定Nameserver地址

3. 创建监听订阅主题Topic和Tag等

4. 处理消息

5. 启动消费者consumer

## 1.2 搭建Rocketmq-demo

### 1.2.1 加入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-client</artifactId>
        <version>4.9.2</version>
        <!--docker的用下面这个版本-->
		<version>4.4.0</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
    </dependency>
</dependencies>
```

`注意依赖要和RocketMQ安装的版本一致`

### 1.2.2 编写生产者

```java
//发消息，测试生产者  
@Test  
void testProducer() throws Exception {  
    // 1. 创建默认的生产者组  
    DefaultMQProducer producer = new DefaultMQProducer("test-producer-group");  
    // 2. 设置nameServer的地址  
    producer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 3. 启动  
    producer.start();  
    // 4. 创建一个消息  
    Message message = new Message("testTopic","我是一个测试消息".getBytes());  
    // 5. 发送消息  
    SendResult sendResult = producer.send(message);  
    System.out.println(sendResult.getSendStatus());  
    // 6. 关闭  
    producer.shutdown();  
}
```

- 细节



### 1.2.3 编写消费者

```java
//接收消息，测试消费者  
@Test  
public void testConsumer() throws Exception{  
    // 1. 创建默认的消费者组  
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test-consumer-group");  
    // 2. 设置nameServer的地址  
    consumer.setNamesrvAddr(NAME_SRV_ADDR);  
    // 3. 订阅一个主题 * 表示订阅这个主题的所有消息，后期会有消息过滤  
    consumer.subscribe("testTopic","*");  
    // 4. 注册一个消费监听 MessageListenerConcurrently 是多线程消费（异步的），默认20个线程，可以参看consumer.setConsumeThreadMax()  
    consumer.registerMessageListener(new MessageListenerConcurrently() {  
        @Override  
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {  
            // 这个就是消费的方法(业务处理)  
            System.out.println("我是消费者");  
            System.out.println(list.get(0).toString());  
            byte[] body = list.get(0).getBody();  
            System.out.println("消息内容："+new String(list.get(0).getBody()));  
            System.out.println("消费的上下文 = " + consumeConcurrentlyContext);  
            // 返回值（返回消费的处理情况）  
            // CONSUME_SUCCESS成功，消息会从mq中出队  
            // RECONSUME_LATER（报错/null），失败，消息会重新回到消息队列，过一会再次投递出来，给当前消费者或其他消费者消费  
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS; //表示成功消费  
        }  
    });  
  
    // 5. 启动  
    consumer.start();  
    // 6. 挂起jvm  
    System.in.read();  
}
```

### 1.2.4 细节

- 生产者和消费者为什么写组名？
	- `因为消费者组必须保证消息的订阅和接收一致性`
	- 消费者组内的消费者，订阅关系必须保持一致，即组内的多个消费者，只能消费相同的主题，而不是a消费者消费topic1，b消费者消费topic2，否则会导致消息的紊乱
	- 另外一方面，加入一个topic有多个消费者组订阅，topic中的信息，是按组为单位进行投递的，每个组会有一份消息
	- `生产者组限制没那么多`

- 查看面板可以发现，`一个Topic中默认有4个队列`，那么消费者如何消费的呢？
	- 首先生产者`发送消息的机制是轮询`，即每个队列都有机会获得消息
	- 负载均衡模式
		- 一个消费者：会消费4个队列的消息
		- 二个消费者：每个消费者对应2个队列的消息
		- 三个消费者：有一个消费者对应2个队列的消息，剩余2个消费者对应1个队列的消息
		- 四个消费者：每个消费者对应1个队列的消息
		- 五个和多个消费者：第5个和其他消费者都会空闲，永远收不到消息
		- 所以，我们要求`broker中的队列数要大于消费者组中的消费者数量`
	- 广播模式
		- 每个消费者都会获得所有的消息


- 因为我们测试，消费者组只有一个消费者，所以4个默认队列的消费者都是同一个

- `代理者位点`：代理端上累计保存的消息数量，即每加入一个消息，代理者位点就+1
- `消费者位点`：已经消费的消息数量，即每消费一个消息，消费者位点就+1
- `差值`：未被消费的消息数量