
## 1、事务是什么

多个操作被看做一个整体，即事务，事务通常具备原子性。
## 2、Multi事务

Redis原生有Multi命令，可以开启事务，我们看一下Redis这段官方说明：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eff993f935716738090d6744b2d65177.png)

原生事务其实是由 `MULTI、EXEC、DISCARD、WATCH` 这四个命令配合完成的。
### 2.1 如何操作

步骤一：MULTI 即开启事务
```shell
127.0.0.1:6379> multi
OK
```

步骤二：此时执行命令，都会到Redis队列里，但不会真正执行
```shell
127.0.0.1:6379> set tx1 1
QUEUED
127.0.0.1:6379> set tx2 2
QUEUED
```

步骤三：实际执行
```shell
127.0.0.1:6379> exec
1) OK
2) OK
```

步骤三：放弃事务
```shell
127.0.0.1:6379> discard
OK
127.0.0.1:6379> GET tx1
null
```

至于discard，很好理解，multi之后，exec之前，都可以放弃。状态流程图如下：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/65c06eb1a8282227a5e5c8219171a837.png)
### 2.2 Watch是干啥的

watch用来提前观察数据，具体来说，它用于监视一个（或多个）key，如果在事务执行之前这个（或这些）key被其他命令所改动，那么事务将被打断。

所以watch命令可以决定事务是执行还是回滚，一般操作是：在multi命令之前使用watch命令监控某些键值对，然后使用multi命令开启事务，执行各类对数据结构进行操作的命令，这些命令会加入先入先出队列中。

当redis使用exec命令执行事务的时候，首先会比较被watch的键值对有没有发生变化，如果产生变化回滚事务，没有变化执行事务中的命令，无论事务执行与否，最终都会取消执行事务之前的watch命令。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84216bb6b63375afe5cd459d83304ca1.png)

### 2.3 原理

Redis服务存储了一个结构体，里面包含了队列里的命令列表，先入先出。

执行的时候顺序执行。
### 2.4 Multi事务具备原子性吗？

All the commands in a transaction are serialized and executer sequentially

不具备，只是通过单线程的特性，让其他操作切不进来。

但是中途如果因为崩溃，或自己有问题，还是可能只做了一半。

我们来看看这个case：

首先，我们创造两个value为string的key
```shell
127.0.0.1:6379> set tx1 tx1
OK
127.0.0.1:6379> set tx2 tx2
OK
127.0.0.1:6379> get tx1
"tx1"
127.0.0.1:6379> get tx2
"tx2"
```

开启事务：
```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set tx1 txone
QUEUED
127.0.0.1:6379> lpop tx1
QUEUED
127.0.0.1:6379> set tx2 txtwo
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get tx1
"txone"
127.0.0.1:6379> get tx2
"txtwo"
```

中间的失败了，前后依然是成功的

### 2.5 Multi三宗罪

弱原子
1. 事务开启后，每个命令其实都是一次调用，浪费资源；
2. 因为这些命令执行之间是有时间的，还需要watch提前来观察，这也太难用了
3. 通常来说失败了会终端后续流程，而multi失败会继续，这容易让人疑惑

multi可以说是Redis关于事务的过渡方案，无论是原子性、功能性、易用性都比较差，事实上，也很少会有团队在生产环境使用multi

事实上，Redis在2.6版本之后，引入了Lua做事务，用官方的话来说Lua会更简单、更快。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bde17e943c2f0fe0f72a50d9d1afa495.png)

## 3、Lua做事务

### 3.1 Lua是什么？

Lua是一种用标准C语言编写的轻量的脚本语言，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

可以看到Lua的目标就是为应用程序提供扩展。

### 3.2 Reids和Lua

Redis 是2.6版本通过内嵌 支持 Lua环境。执行脚本的常用命令为EVAL

Redis因为是单线程操作，处理过程中，是不会被打断并切换到其他处理，所以Redis执行Lua，不出异常的情况下，也不会被打断

### 3.3 操作Lua

不传参例子：
```shell
127.0.0.1:6379> eval "return {redis.call('set','ka','a'),redis.call('set','kb','b')}" 0
1) OK
2) OK
```

传参例子：
```shell
127.0.0.1:6379> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

### 3.4 Lua事务分析

Lua执行一半失败了，会怎样？会回滚吗？还是会中断？

看下面的例子，ka更新为a1了，kb没更新为b1。说明`没回滚，中断`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c3229dbcebc648aeca1c507b711bf1f9.png)

讲到这里，我们来对比下原生的Multi事务：
- 可以编写if else这种选择逻辑
- 事务中间如果失败，会中断后续执行
- 使用方便，开启Lua就可以执行事务，不像Multi还需要Watch来保证执行开始时状态未改变

### 3.5 Lua在Redis中主要做什么

使用事务场景都可以用Lua

比较常见的就是分布式锁和秒杀等场景等。