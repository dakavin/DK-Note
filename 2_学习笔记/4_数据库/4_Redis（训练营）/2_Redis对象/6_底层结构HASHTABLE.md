
## 1、HASHTABLE概述

HASHTABLE，可以想象成目录。要翻看什么内容，直接通过目录能找到页数，翻过去看。如果没有目录，我们需要一页一页往后翻，效率很低。

在计算机世界里，HASHTABLE就扮演这样一个快速索引的角色，通过HASHTABLE我们可以只用O(1)时间复杂度就能快速找到key对应的value

## 2、HASHTABLE结构

Redis中HASHTABLE的结构如下：

```C
// from Redis 5.0.5
/* This is our hash table structre. */
typedef struct dictht{
	dictEntry **table;
	unsigned long size;
	unsigned long sizemask;
	unsigned long used;
} dictht;
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6d5d783bfd0090119f1f5ba69e82031c.png)

最外层是一个封装的dictht结构，其中字段含义如下：

- table：指向实际hash存储。存储可以看做一个数组，所以是`*table`的表示，在C语言中`*table`可以表示一个数组。

- size：哈希表大小。实际就是dictEntry有多少元素空间。

- sizemask：哈希表大小的掩码表示，总数等于size-1。这个属性和哈希值一起决定一个键应该被放到table数组的哪个索引上面，规则`Index = hash&sizemask`  
	- 联想HashMap的put方法中，确定元素索引 `hash&(n-1)`

- used：表示已经使用的节点数量。通过这个字段可以很方便地查询到目前HASHTABLE元素总量。

## 3、Hash表渐进式扩容

渐进式扩容顾名思义就是一点一点慢慢扩容，而不是一股脑直接做完，那具体流程是怎么样的呢？

其实为了实现渐进式扩容，Redis中没有直接把dictht暴露给上层，而是再封装了一层：
```C
// from Redis 5.0.5
typedef struct dict{
	dictType *type;
	void *privdata;
	dictht th[2];
	long rehashidx; /* rehashing not in progress if rehashidx == -1 */
	unsigned long iterators; /* number of iterators currently running */
} dict;
```

可以看到dict结构里面，包含了2个dictht结构，也就是2个HASHTABLE结构。dictEntry是链表结构，也就是用拉链法解决Hash冲突，用的是头插法；

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/334f4c7426f6b6b7a261e90575ad77f6.png)

实际上平常使用的就是一个HASHTABLE，`在触发扩容之后，就会两个HASHTABLE同时使用`，详细过程是这样的：

当先字典添加元素时，发现需要扩容就会进行Rehash。Rehash的流程大概分为三步：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36f0ccb7b159f5de5d64c20a1266bf5b.png)

- 第一步：`为新Hash表ht[1]分配空间`。下表大小为第一个大于等于原表2倍used的2次方幂。举个例子，原表如果used=500，2倍就是1000，那第一个大于1000的2次方幂则为1024。此时字典同时持有ht[0]和ht[1]两个哈希表。`字典的偏移索引从静默状态-1，设置为0，表示Rehash工作正式开始`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a994b9d50cbfc7d6ab723d4e7e660c54.png)
- 第二步：`迁移ht[0]数据到ht[1]`。在Rehash进行期间，每次对字典执行增删改查操作，程序会`顺带迁移当前rehashidx(索引)在ht[0]上的对应的数据`，`并更新偏移索引`。与此同时，部分情况周期函数也会进行迁移，详情看评论。这里有一个疑问，如果rehashinx刚好在一个已删除的空位置上，是直接返回，还是尝试往下找。我们直接看dictRehash函数的片段代码![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0fe8fd8878c84558693370e5e29e3af2.png)
- 第三步：随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被Rehash到ht[1]，此时再将ht[0]和ht[1]指针对象互换，同时把偏移索引的值设为-1，表示Rehash操作已完成。这个事情也是在Rehash函数做的，每次迁移完一个元素，会检查下是否完成了整个迁移![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f385204e9f062e0e35b1779a68e850f.png)
对于HASHTABLE的增删改查操作：
- 查询：ht[0]先查，查到了就break，不然查ht[1]
- 增加：向ht[1]增加
- 删除：和查询一样
- 更新：和查询一样

`注意：增删改查操作，和数据迁移操作，是两个独立的操作，迁移是顺带的！！！`


`总结一下，渐进式扩容的核心就是操作时顺带迁移。`

将了扩容是怎么做的，大家肯定会问，到底啥时候会扩容呢？那么接下来我们就讲讲扩容的时机。

## 4、扩容时机

Redis提出了一个负载因子的概念，负载因子表示目前Redis HASHTABLE的负载情况，是游刃有余，还是不堪重负了。我们设负载因子为k，那么 `k=ht[0].used / ht[0].size` ，也就是使用空间和总空间大小的比例。

Redis会根据负载因子的情况来扩容：
1. 负载因子大于等于1，说明此时空间已经非常紧张。新数据是在链表上叠加的，越来越多的数据其实无法再O(1)时间复杂度找到，还需要遍历一次链表，如果此时服务器没有执行BGSAVE或BGREWRITEAOF这两个命令，就会发生扩容。复制命令对Redis的影响我们后面再原理篇再讲。
2. 负载因子大于5，这时候说明HASHTABLE真的已经不堪重负了，此时及时是有复制命令在进行，也要就是Rehash扩容。

## 5、缩容

扩容是数据太多存不下了，那如果太富裕呢，比如原来可能数据比较多，发送了扩容，但后面数据不再需要，被删除了，此时多余的空间就是一种浪费。

缩容过程其实和扩容是相似的，也是渐进式缩容，这里不再赘述。

同样的，Redis还是用负载因子来控制什么时候缩容：
- 当负载因子小于0.1，即负载率小于10%，此时进行缩容，下表大小为第一个大于等于原表used的2次方幂。当然如果有BGSAVE或BGREWRITEAOF这两个复制命令，缩容也会受影响，不会进行。

## 6、对比HashMap

### 6.1 扩容过程对比

1. 在dict数据结构，渐进式扩容时，通过rehashidx++，来将ht[0].table[rehashidx] 每个桶位元素 逐渐 移到 ht[1].table的每个桶位去，新桶位的索引计算如下：
	- 桶位没有元素：在该桶位最多查找n*10次，n一般是1
	- 桶位只有一个元素：dictHashKey(d, de->key) & d->ht[1].sizemask; （field的哈希值 & 新表的长度-1）
	- 桶位是链表结构：如上计算新桶位的索引，然后遍历链表，将其放入
    

2. 在HashMap中，通过resize方法，通过遍历table所有元素，一次性将元素移动到新table中，，新桶位的计算如下：
	- 桶位没有元素：不做处理
	- 桶位只有一个元素：key的哈希值 & 新表的长度-1
	- 桶位是链表结构：将链表中的元素区分高位和低位
		1. 高位：key的哈希值 & 旧table长度 = 0，链表元素放在索引不变的桶位
		2. 低位：key的哈希值 & 旧table长度 != 0，链表元素放在（当前索引+旧table长度）的桶位
	- 桶位是红黑树：将原来的红黑树拆解，放到新table上（略）

### 6.2 扩容和缩容时机对比

1. HashMap
	- 扩容：
		- 内置数组为空，或数组长度为0时，会对table进行一次初始化
		- 当添加元素所需容量`size>threshold = loadfactor*table.length` ，发生扩容
		- 堆上的链表长度大于8，但是数组长度小于64，发生扩容
			- 如果此时数组长度达到64，链表转化为红黑树
		- 数组扩容后，table数组容量不会缩容（go语言有缩容）
	- 缩容：
		- 红黑树上节点小于6的时候，会转化为链表

2. dict
	- 扩容：
		- dictht.used / dictht.size > 1 ，且 没有BGSAVE或BGREWRITEOF，会发生扩容
		- dictht.used / dictht.size > 5 ，无论是否BGSAVE或BGREWRITEOF，都会发生扩容
	- 缩容：
		- dictht.used / dictht.size < 0.1 ，且 没有BGSAVE或BGREWRITEOF，会发生缩容

## 6、总结

HASHTABLE提供了快速索引数据的能力。Redis中HASHTABLE是用数组存储元素，位置相同则用链表连接，为了对性能更友好，Redis使用渐进式扩缩容的机制，让扩缩容对整体的影响较低。

相比于ZIPLIST，HASHTABLE就是面试的超级热点，不管需要掌握它的大体思路，方方面面的细节都需要深入掌握，到这里，我们HASHTABLE相关的知识就讲完了，下一节会继续HASHTABLE的面试点面试题分析。