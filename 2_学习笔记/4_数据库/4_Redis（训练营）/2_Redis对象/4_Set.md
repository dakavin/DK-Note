## 1、是什么？

Redis的Set是一个不重复、无序的字符串集合，这里额外说明一下，如果是INTSET编码的时候其实是有序的，不过一般不应该依赖这个，整体还是看成无序来用比较好。

## 2、适用场景

适用于无序集合场景，比如某个用户关注了哪些公众号，这些信息就可以放进一个集合，Set还提供了查交集、并集的功能，可以很方便地实现共同关注的能力。

## 3、常用操作

我们还是从创建、查询、更新、删除这几个基本操作来了解Set。

创建：产生一个Set对象，可以使用SADD创建。

查询：Set的查询十分丰富，SISMEMBER可以查询元素是否存在；SCARD、SMEMBERS、SSCAN可以查询集合元素数据；SINTER、SUNION、SDIFF可以对多个集合查交集并集和差异；

更新：可以使用SADD增加元素，SREM删除元素

删除：和其他对象一样，DEL可以删除一个SET对象

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017195615.png)

### 3.1 写操作

#### 3.1.1 SADD

语法：SADD key member [member ...]
功能：添加元素，返回值为成功添加了几个元素

下面，我们先来创建一个名为setniuniu的集合，初始携带aa，bb，cc三个元素：
```shell
SADD setniuniu aa bb cc
(integer) 3
```

当前setniuniu中的元素有：“aa”、“bb”、“cc”
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017200207.png)

为setniuniu添加元素：
```shell
SADD setniuniu 11 22 33
(integer) 3
```

当前setniuniu中的元素有：“aa”、“bb”、“cc”、“11”、“22”、“33”![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017200321.png)
#### 3.1.2 SREM

语法：SREM key member [member ...]
功能：删除元素，返回值为成功删除了几个元素

```shell
SREM setniuniu 33
(integer) 1
```

删除完成之后，当前setniuniu中的元素有：“aa”、“bb”、“cc”、“11”、“33”![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017200435.png)
### 3.2 读操作

我们按照以下状态来演示：
setniuniu：[“aa”、“bb”、“cc”、“11”、“22”、“33”]
setmart：[“aa”、“bb”、“ff”]

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017200528.png)

#### 3.2.1 SISMEMBER

语法：SISMEMBER key member
功能：查询元素是否存在

```shell
SISMEMBER setniuniu aa
(integer) 1

SISMEMBER setniuniu ff
(integer) 0
```

#### 3.2.2 SCARD

语法：SCARD key
功能：查询集合元素个数

```shell
SCARD setniuniu
(integer) 6
```

#### 3.2.3 SMEMBERS

语法：SMEMBERS key
功能：查看集合的所有元素(注意是无序的)

```shell
SMEMBERS setniuniu
1) "aa"
2) "22"
3) "bb"
4) "33"
5) "cc"
6) "11"
```

#### 3.2.4 SSCAN

语法：`SSCAN key cursor [MATCH pattern] [COUNT count]`
功能：查看集合元素，可以理解为指定游标进行查询，可以指定个数，默认个数为10

例子1：从0号位置开始查询，默认10个
```shell
127.0.0.1:6379> SSCAN setniuniu 0
1) "0"
2) 1) "cc"
   2) "33"
   3) "bb"
   4) "11"
   5) "22"
   6) "aa"
```

例子2：使用MATCH模糊查询
```shell
127.0.0.1:6379> SSCAN setniuniu 0 MATCH 1*
1) "0"
2) 1) "11"
127.0.0.1:6379> SSCAN setniuniu 0 MATCH 2*
1) "0"
2) 1) "22"
127.0.0.1:6379>
```

(参考文章)[Redis中Scan命令的基本用法](https://cloud.tencent.com/developer/article/1554983)]

#### 3.2.5 SINTER（交集）

语法：SINTER key [key ...]
功能：返回在第一个集合里，同时在后面所有集合都存在的元素。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017203618.png)

```shell
127.0.0.1:6379> SINTER setniuniu setmart
1) "bb"
2) "aa"
```
#### 3.2.6 SUION

语法：SUNION key [key ...]
功能：返回所有集合的并集，集合个数大于等于2个

```shell
127.0.0.1:6379> SUNION setniuniu stemart
1) "22"
2) "11"
3) "bb"
4) "cc"
5) "33"
6) "aa"
```

#### 3.2.7 SDIFF

语法：SDIFF key [key ...]
功能：返回第一个集合有，且在后续集合中不存在的元素，集合个数大于等于2个，注意，是以第一个集合和后面比， 看第一个集合多了哪些元素

```shell
127.0.0.1:6379> SDIFF setniuniu setmart
1) "33"
2) "cc"
3) "22"
4) "11"
```

## 4、底层实现

### 4.1 编码方式

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017203843.png)

Redis出于性能和内存的综合考虑，也支持两种编码方式，如果集群元素`都是整数`，且`元素数量不超过512个`，就可以用INTSET编码，结构如下图，可以看到INTSET排列比较紧凑，内存占用少，但是查询时需要二分查找（说明该编码下实际存储的是一个有序集合）。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017204020.png)

如果不满足INTSET的条件，就需要用HASHTABLE，HASHTABLE结构如下图，可以看到其查询一个元素的性能很高，能O(1)时间就能找到一个元素是否存在。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231017204159.png)
## 5、总结

Set可以很方便的管理无序集合，还可以为多个集合求交集并集，底层编码模式有INTSET和HASHTABLE两种，INTSET对应少量整数集合下节约内存，HASHTABLE适用于需要快速定位某个元素的场景。

下一节，我们先介绍另外一种Redis对象Hash，这种对象和Set非常类似，讲完Hash之后，我们再详细分析他们都用到的底层数据结构HASHTABLE。

## 6、补充

补充：字典（Dictionary）和哈希表（HashTable）的区别

- 字典
	1. 字典是一种典型的键值对类型的数据结构，每一个元素都是由一个键值对（键key和值value）组成。
	2. 这种数据结构可以通过某个键来访问元素，所以字典也被称为映射或散列表。
	3. 字典的主要特性是根据键快速查找值，也可以自由添加和删除元素，这有点像List，但跟List不同的是，List是连续存储，直接定址的。 字典像链表，非连续存储，所以删除元素不需要移动后续元素，就没有在内存中移动后续元素的性能开销。
	4. 通过键来检索值的速度是非常快的，接近于 O(1)，这是因为 Dictionary 类是作为一个哈希表来实现的，Dictionary 没有顺序之分，这一点不同于list列表，有顺序之分。键必须是唯一的，而值不需要唯一。
	5. 字典中的键和值都是object类型，所以可以是任何类型（比如：string、int、自定义类型等等）

- 哈希表
	1. `哈希表Hashtable类实现了IDictionary接口，集合中的值也是以键值对（key/value）的形式存取的`。
	2. 哈希表，也称为散列表，在该集合中每一个元素都是由键值对（key/value）的形式存放值。
	3. 需要注意的是，key是区分大小写的。
	4. 哈希表Hashtable中的键和值为object类型，所以哈希表Hashtable支持任何类型的键值对。

- 区别
	1. 多线程区别
		- 单线程用字典，有泛型优势，读取速度快，容量利用更充分；
		- 哈希表单线程写入，多线程读取
	2. 线程安全
		- 字典非线程安全，必须人为的增加lock语句进行保护，影响效率。
		- 哈希表可以调用Synchronized()方法可以获得完全线程安全的类型。
	3. 数据插入顺序
		- 字典的排序就是按照插入的排序（注：删除节点后顺序就被打乱了），因此在需要体现顺序的情况下，字典更有效。
		- 哈希表不是按照顺序插入数据的。
	4. 索引效率
		- 字典在单线程索引数据的时候效率比较高，读取速度快，但是数据量大的时候效率会下降
		- 哈希表的索引方式是经过散列处理的，在数据量大的时候处理效率更高，所以哈希表比较适合运用在做对象缓存，树递归算法的替代等各种需要提升效率的场合。

