---
文章标题: "[[9_Stream(不重要，基本不考)]]" 
文章作者: Dakkk
文章概要: |
  介绍Redis 5.0新增的Stream数据类型，是具有持久化能力的轻量级消息队列，支持生产消费场景和消费者组功能
tags:
- "Redis"
- "Stream"
- "消息队列"
- "持久化"
- "消费者组"
- "XADD"
- "XREADGROUP"
- "数据结构"
相关文章:
- "[[1_持久化基础面试题]]"
- "[[1_Redis内存面试与分析]]"
- "[[1_String面试与分析]]"
- "[[2_List面试与分析]]"
- "[[2_RDB面试与分析]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/02-🔴 Redis/03-🎓 训练营笔记/2_Redis对象/9_Stream(不重要，基本不考).md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐ 基础知识
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:57
---

## 1 是什么？

Stream是Redis 5.0版本新增加的操作对象。

Stream可以看做一个`拥有持久化能力的轻量级消息队列`。

## 2 使用场景

Stream可以作为轻量级队列，可以支持生成消费场景，可以算是Redis中相对于完善的消息队列了。

## 3 常用操作

先介绍一个概念，流ID。

`流ID：可以独一无二地标志一个Stream。`

### 3.1 写操作

#### 3.1.1 XADD

语法：XADD key ID(`流ID，可以用*自动生成`) field string [field string ...]
功能：向Stream中添加数据，如果没有就新建一个Stream，返回值是一个流ID，如果输入时是自定义流ID，返回值与之相同，如果用`*`自动生成，返回的就是自动生成的流ID。

例子1：
```shell
127.0.0.1:6379> XADD strmniuniu * f1 msg1 f2 msg2
"1697812716831-0"
127.0.0.1:6379> XADD strmniuniu * f3 msg3 f4 msg4
"1697812726712-0"
```

可以看到，Redis自动生成的流ID由两部分组成：`毫秒时间`和`序列号`

上面的例子中，1697812716831就是毫秒时间，0是64位序列号，如果同一毫秒的并发请求，序列号就会递增，如果因为机器时间跳跃导致毫秒时间比之前更早，就使用最近的一条流ID时间。这种情况下，那么会以前最近的流ID毫秒。

例子2：
如果由于某些原因，用户需要自定义流ID，就像前面所说的，XADD命令可以带上一个显示的ID：
```shell
 XADD strmniuniu 123456 f5 msg5
(error) ERR The ID specified in XADD is equal or smaller than the target stream top item
```

可以看到，因为之前我们用`*`自动生成了两条记录，现在随手写的123456没之前记录的毫秒时间大，就插入不进去了。

那我们换一个新的流来测试自定义流ID：
```shell
127.0.0.1:6379> XADD strmmart 123456 f5 msg5
"123456-0"
```

可以看到，同样的语句在新的流就能成功。

如果非要在已经有自动生成流ID数据的流中插入自定义流ID，我们需要业务调整自定义ID不小于之前的流ID，实际上，尽量不要混用自动生成ID和自定义ID，不然自定义ID还需要额外关注默认ID生成的数据，相互耦合。

Redis Streams支持按ID进行范围查询。由于ID与生成的时间相关，因此可以很容易地按时间范围进行查询。我们后面讲到XRANGE命令时，很快就能明白这一点。

### 3.2 读操作

#### 3.2.1 XLEN

语法：XLEN key
功能：返回流数目

```shell
127.0.0.1:6379> XLEN strmniuniu
(integer) 2
```

#### 3.2.2 XRANGE

语法：XRANGE key start end [COUNT count]
功能：范围查询流信息

```shell
127.0.0.1:6379> XRANGE strmniuniu - +
1) 1) "1697812716831-0"
   2) 1) "f1"
      2) "msg1"
      3) "f2"
      4) "msg2"
2) 1) "1697812726712-0"
   2) 1) "f3"
      2) "msg3"
      3) "f4"
      4) "msg4"
```

### 3.3 群组操作

除了常规的读写操作，Stram还支持按群组操作，这里单独列出来。如果了解过Kafka的同学，应该知道Kafka也有消费者组的概念，虽然实现南辕北辙，但逻辑上很相似。

Stream群组具备如下特性：
1. `群组里可以有多个成员，成员名称由消费方自定义，组内唯一即可`。
2. `同个群组共享消息，消息被其中一个消费者消费之后，其他消费者不会再重复消费`。
3. 不在群组内的客户端，也可以通过XREAD命令来和群组一起消费，即群组和非群组可以混用，这个时候其实把不在群组的客户端理解为一个单独的群组。

群组最核心的操作有3个
- XGROUP：创建、摧毁或管理消费者组
- XREADGROUP：通过消费者组从一个Stream读取
- XACK：允许消费者将待处理消息标记为已正确处理的命令

#### 3.3.1 创建消费群组

语法：XGROUP CREATE key groupname id-or-$
功能：创建一个新的消费群组，成功返回OK

```shell
127.0.0.1:6379> XGROUP CREATE strmniuniu group1 0
OK
127.0.0.1:6379> XGROUP CREATE strmniuniu group2 0
OK
```

#### 3.3.2 消费数据

语法：XREADGROUP GROUP ${group_id} ${member_id} COUNT ${num} STREAMS ${stream_id}
- group_id：群组id，也就是群组名字
- member_id：群组成员id
- num：消费几条数据
- stream_id：stream流id
功能：消费流中的数据

```shell
127.0.0.1:6379> XREADGROUP GROUP group1 cat COUNT 2 STREAMS strmniuniu >
1) 1) "strmniuniu"
   2) 1) 1) "1697812716831-0"
         2) 1) "f1"
            2) "msg1"
            3) "f2"
            4) "msg2"
      2) 1) "1697812726712-0"
         2) 1) "f3"
            2) "msg3"
            3) "f4"
            4) "msg4"
```

再消费一次：
```shell
127.0.0.1:6379> XREADGROUP GROUP group1 cat COUNT 2 STREAMS strmniuniu >
(nil)
```

注意，在我们的case里只有2条数据，所以第一次执行完之后，同群组就消费不到了。不同群组呢？还是可以的：
```shell
127.0.0.1:6379> XREADGROUP GROUP group2 cat COUNT 2 STREAMS strmniuniu >
1) 1) "strmniuniu"
   2) 1) 1) "1697812716831-0"
         2) 1) "f1"
            2) "msg1"
            3) "f2"
            4) "msg2"
      2) 1) "1697812726712-0"
         2) 1) "f3"
            2) "msg3"
            3) "f4"
            4) "msg4"
```

## 4 总结

Stream的消费模型和Kafka的群组概念上挺相似，它弥补了Redis Pub/Sub不能持久化消息的缺陷。

但是一些功能和实现上，他们还是不同的，Redis作者的还有另一个开源项目Disque，这个项目最终可以说是没做起来，Stream可以说是弥补了部分遗憾。

当然，Stream不是面试重点，工作中用得也不多，适当了解即可。