# 一、如何解决消息堆积问题？

` 一般认为单条队列消息差值>=10w时 算堆积问题`

默认情况下，一条队列的差值，官方给的最大值是30w
## 1.1 什么情况下会出现堆积

- `原因一：生产太快了`

	- 解决一：生产方可以做业务限流

	- 解决二：增加消费者数量，但是消费者数量<=队列数量，适当的设置最大的消费线程数量(   根据  ` IO(2n)`  /  `CPU(n+1)`   )
		- IO密集型：线程数量=cpu线程数x2
		- CPU密集型：现场数量=cpu线程数+1
		- cpu线程数：通过`Runtime.getRuntime().availableProcessors()`获取

	- 解决三：`动态扩容队列数量`，从而增加消费者数量
		- 注意缩容队列的时候，要等到队列中的消息都消费完成后再继续缩容

- `原因二：消费者消费出现问题`

	- 解决一：排查消费者程序的问题

- 如果堆积的消息不想要了
	- `跳过堆积`：选择一个组 跳过堆积以后 这个组里面的的所有都不会被消费了

	- `重置消费点位`：将一个组的消费节点 设置在之前的某一个时间点上去 从这个时间点开始往后消费，是一个补救的操作，可以再次消费消息

# 二、如何确保消息不丢失？


![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922232226487.png)



要想保证消息的可靠型投递，无非保证如下3个阶段的正常执行即可。

1.     生产者将消息成功投递到broker

2.     broker将投递过程的消息持久化下来

3.     消费者能从broker消费到消息

下面分别从上述3个阶段来分析：

1、`发送阶段`

（1）、发送成功

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922232228590.png)



（2）、发送失败

Producer程序向Broker发送消息时没有成功（比如网络抖动或broker宕机），此时`Producer会自动进行重试`，重试默认次数如下：

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922232232439.png)



也可以通过如下方式修改：

```java
rocketMQTemplate.getProducer().setRetryTimesWhenSendFailed(xx);
rocketMQTemplate.getProducer().setRetryTimesWhenSendAsyncFailed(xx);
```

2、`Broker存储消息阶段`

「异步刷盘」：消息被`写入内存的PAGECACHE，返回写成功状态`，当内存里的消息量积累到一定程度时，统一触发写磁盘操作，快速写入 。吞吐量高，当磁盘损坏时，会丢失消息

异步刷盘丢失数据解决方式：发送数据的时候，可以 ① log记录在文件；②mysql记录数据库



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



- 总结


- `步骤一`：生产者使用同步发送模式 ，收到mq的返回确认以后  `顺便往自己的数据库里面写`

msgId status(0) time

- `步骤二`：消费者消费以后 修改数据这条消息的状态 = 1

1和2步骤的思路图：发的时候记录状态为1，收到并处理完成记录状态为2
这样检查mysql中，状态仍为1的数据（很久），可以确定那些数据丢失了

- `步骤三`：写一个定时任务 间隔两天去查询数据  如果有status = 0 and time < day-2

- `步骤四`：将mq的刷盘机制设置为同步刷盘

- `步骤五`：使用集群模式 ，搞主备模式，将消息持久化在不同的硬件上

- `步骤六`：可以开启mq的trace机制，消息跟踪机制

	1. 在broker.conf中开启消息追踪，设置traceTopicEnable=true
	
	2. 重启broker即可
	
	3. 生产者配置文件开启消息轨迹，设置enable-msg-trace: true

	4. 消费者开启消息轨迹功能，可以给单独的某一个消费者开启，设置enableMsgTrace = true

	在rocketmq的面板中可以查看消息轨迹

	默认会将消息轨迹的数据存在 RMQ_SYS_TRACE_TOPIC 主题里面

# 三、安全

1. `开启acl的控制` 在broker.conf中开启aclEnable=true![[Pasted image 20230917010928.png]]

2. `配置账号密码` 修改plain_acl.yml
	- 到这步为止，我们给broker配置了账户和密码
	- 我们还需要个dashboard配置mq服务的账户和密码


3. `修改控制面板的配置文件` 放开52/53行 把49行改为true 上传到服务器的jar包平级目录下即可![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922232309740.png)
- 注意，因为是jar包，所以我们可以在window中解压后，修改完配置，将配置文件放在和jar包平级的目录下即可