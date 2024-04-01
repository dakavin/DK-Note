## 1、是什么？

ZSet就是有序集合，也叫Sorted Set，是一组按关联积分有序的字符串集合，这里的分数是个抽象概念，任何指标都可以抽象为分数，以满足不同场景。

积分相同的情况下，按字典序排序。

## 2、适用场景

用于需要排序集合的场景，最为经典的就是游戏排行榜。

## 3、常用操作

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30ac57ecd10a6fd0575a6da62ad1d263.png)
### 3.1 写操作

#### 3.1.1 ZADD

语法：ZADD key score member [score member]
功能：向Sorted Set增加数据，如果是已经存在的Key，则更新对应的数据

```shell
127.0.0.1:6379> ZADD zsetniuniu 100 user1 128 user2 66 user3
(integer) 3
```

扩展参数：
- XX：仅更新存在的成员，不添加新成员
- NX：不更新存在的成员，只添加新成员
- LT：更新新的分值比当前分值小的成员，不存在则新增
- GT：更新新的分值比当前分值大的成员，不存在则新增

此时集合中的有序数据如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8af08b49de4ff6c9762ac5cd89d89de7.png)
ZADD操作已存在的member就是更新它，所以不要被ZADD名字误导，其既有add也有update能力。

注意这里虽然返回值是1，表示新增member只有user4一个，更新的user3不计入返回值，但是数据是更新成功的。
```shell
127.0.0.1:6379> ZADD zsetniuniu 256 user3 512 user4
(integer) 1
// 此处为添加了多少个元素
```

此时集合中的有序数据如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/07ca0aa674cbff441c0e08f340e740ba.png)

#### 3.1.2 ZREM

语法：ZREM key member [member ...]
功能：删除ZSet中的元素

```shell
127.0.0.1:6379> ZREM zsetniuniu user4
(integer) 1
```

此时集合中的有序数据如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1969587c5f3f477a686c57fdc38241d5.png)

### 3.2 读操作

读操作我们以以下状态来进行讲解。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a2c29027e9a2bb6c977f043a3a5dfc3.png)
#### 3.2.1 ZCARD

语法：ZCARD key
功能：查看ZSet中成员总数

```shell
127.0.0.1:6379> ZCARD zsetniuniu
(integer) 3
```

#### 3.2.2 ZRANGE

语法：ZRANGE key start stop [WITHSCORES]
功能：查询从start到stop范围的ZSet数据，WITHSCORES选填，不写输出里就只有key，没有score值

```shell
127.0.0.1:6379> ZRANGE zsetniuniu 0 -1 WITHSCORES
1) "user1"
2) "100"
3) "user2"
4) "128"
5) "user3"
6) "256"
```

#### 3.2.3 ZREVRANGE

语法：ZREVRANGE key start stop [WITHSCORES]
功能：即reverse range，`从大到小遍历`，WITHSOCRES选填，不写输出里就只有key，没有score值

```shell
127.0.0.1:6379> ZREVRANGE zsetniuniu 0 -1 WITHSCORES
1) "user3"
2) "256"
3) "user2"
4) "128"
5) "user1"
6) "100"
```

#### 3.2.4 ZCOUNT

语法：ZCOUNT key min max
功能：计算min-max积分范围的成员个数

```shell
127.0.0.1:6379> ZCOUNT zsetniuniu 50 200
(integer) 2
```

#### 3.2.5 ZRANK

语法：ZRANK key member
功能：查看ZSet中member的排名索引，索引是从0开始，所以如果排第一，索引就是0

```shell
127.0.0.1:6379> ZRANK zsetniuniu user2
(integer) 1
```

#### 3.2.6 ZSCORE

语法：ZSCORE key member
功能：查询ZSet中成员的分数

```shell
127.0.0.1:6379> ZSCORE zsetniuniu user3
"256"
```

## 4、底层实现

### 4.1 编码方式

ZSet底层编码有两种，一种是ZIPLIST，一种是SKIPLIST+HASHTABLE

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/092fd5e319821f37e6f7bd534138832e.png)

ZIPLIST我们可以说很熟悉了，之前List、Hash中都有打过交道，同样在ZSet中ZIPLIST也是用于数据量比较小的时候，便于节约内存，结构如下：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4af0a16d1af8a69cd7e6cd6c48f2ef66.png)

如果满足如下规则，ZSet就用ZIPLIST编码：
1. 列表对象保存的所有`字符串对象（不包括score）长度都小于64字节`；
2. `列表对象元素个数少于128个`

两个条件任何一条都不满足，编码结构就用SKIPLIST+HASHTABLE，其中SKIPLIST也是底层数据结构，是本节才出现的新伙伴。

SKIPLIST是一种快速查找的多级链表结构，通过SKIPLIST可以快速定位数据所在。它的排名操作、范围查询性能都很高，在前面的章节，我们已经介绍过SKIPLIST，如果有遗忘，可以回头看看

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/152f8468533b8c38ece0ec7ba63a231d.png)

处理SKIPLIST，`Redis还使用了HASHTABLE来配合查询`，这样可以在O(1)时间复杂度查到成员的分数值

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d20b83f44f7a1ce63f056d2a7ecf6635.png)

`补充：`

skiplist 编码的有序集合对象使用 zset 结构作为底层实现，一个 zset 结构同时包含一个字典和一个跳表：
```C
typedef struct zset{
     //跳跃表
     zskiplist *zsl;
     //字典
     dict *dice;
} zset;
```

`字典的键`保存元素的值，`字典的值`则保存元素的分值；

跳跃表节点的 `object 属性保存元素的成员`，跳跃表节点的 `score 属性保存元素的分值`。

这两种数据结构会**通过指针来共享相同元素的成员和分值**，所以不会产生重复成员和分值，造成内存的浪费。

>**说明**：其实有序集合单独使用字典或跳跃表其中一种数据结构都可以实现，但是这里使用两种数据结构组合起来，原因是假如我们单独使用 字典，虽然能以 O(1) 的时间复杂度查找成员的分值，但是因为字典是以无序的方式来保存集合元素，所以**每次进行范围操作的时候都要进行排序**；假如我们单独使用跳跃表来实现，虽然能执行范围操作，**但是查找操作有 O(1)的复杂度变为了O(logN)**。因此**Redis使用了两种数据结构来共同实现有序集合。**

  
## 5、总结

ZSet就是有序集合，顾名思义，用于保存、查询处理有序的集合，其范围查询、成员分值查询速度都非常快。

ZSet非常适用于排名场景，比如游戏战力排行榜。ZSet底层有两种编码方式，一种是数据量比较少时的ZIPLIST，一种是查询性能更优的SKIPLIST+HASHTABLE的编码模式。
