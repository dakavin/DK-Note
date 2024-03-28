
## 1、客户端与服务端交互

客户端发送请求，服务端接收并处理请求，最后返回回包。假设我们要做多次操作，那实际执行会是这样：

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231024202839.png)

可以看到，这种客户端服务器交互模式下，我们需要一条命令结束了，再执行另一个命令，也就是说，多客户端可以并发，但是单客户端是同步的，而`一次请求的时间，其实不光受Redis本身处理性能影响，很大程度还会受网络波动影响，在客户端大量发包场景，很可能会因为波动造成性能瓶颈
`
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231024203014.png)

所以，有没有一种技术，能够让客户端可以将多条命令一起传递给服务端呢？
## 2、Pipeline技术

Pipeline，即管道技术，顾名思义，是将请求打包在一起，通过管道传递。

Pipeline是很成熟的技术，在很多其他组件中也有出现，比如HTTP2.0，比如ETCD，以及很多POP3协议。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231024203150.png)

Pipeline的本质，是将请求在客户端打包，然后一次发送给服务端，服务端处理完成之后，会将结果存起来，等Pipeline中所有命令都完成了，再一起回包，这样可以节约很多网络交互时间。

## 3、Pipeline的代价

Pipeline可以让多个请求一起打包发给服务端，这样看起来，请求都走Pipeline模式是一个好的选择？实际不是的，任何方案都不可能只有优点、没有缺点。Pipeline也是会付出"代价"的：
1. 当使用Pipeline，服务端不得不为回包排位，`同一时间会占用更多内存`，我们一直说，Redis内存是寸土寸金，所以除非特定场景，不然也没有必要强行使用Pipeline
2. `单个命令的时间会变`长，原因是需要等待Pipeline中其他命令的完成。

## 4、Pipeline注意事项

1. Pipeline的命令不是原子的，因为实际上Redis还是一条一条去执行
2. Pipeline包含的命令也不宜过多，Redis内存寸土寸金
3. Pipeline只会在一个Redis节点上执行，这个很好理解，因为其实客户端是发送了一次请求，并不会分发到多个节点上。

## 5、适用场景

适用于一个流程中多次操作Redis创建，比如一个线程需要更新100个学生的分数，这时候可以用Pipeline打包执行。

## 6、总结

Redis支持Pipeline模式来批量发送数据给服务端，这样的好处是节省了网络时间，缺点是服务端会耗费更多内存，以及单个命令的时间会被整体拖慢。

生产中，不建议过度使用Pipeline，只有在真正需要的时候，比如在一个线程需要更新多个数据的时候，就比较合适。