---
文章标题: "[[8_ZSet面试与分析]]" 
文章作者: Dakkk
文章概要: |
  文章分析Redis ZSet面试考点，涵盖其基本操作、ZIPLIST与SKIPLIST+字典的底层编码。深入探讨ZSet结合跳表和字典的优势，并详细解释跳表在Redis中为何优于B+树和红黑树，强调实现简单性与效率。
文章标签:
- "Redis"
- "ZSet"
- "跳表"
- "数据结构"
- "面试"
- "B+树"
- "红黑树"
- "底层实现"
相关文章:
- "[[7_底层结构跳表]]"
- "[[2_List面试与分析]]"
- "[[7_跳表面试与分析]]"
- "[[8_ZSet]]"
- "[[1_String面试与分析]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/3_训练营/8_Redis面试题/1_Redis对象/8_ZSet面试与分析.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 面试考点分析

ZSet，也就是有序集合，是非常热门的考察内容。

考点集中在基本操作、编码方式、底层结构。

其中，跳表这个数据结构考察的频率非常高
## 2 面试题

### 2.1 ZSet如何添加元素，如何移除元素？

分析：
- 基础操作考察

回答：
- ZADD命令可以用于创建ZSet，也可以用于添加元素，ZREM命令可以移除元素
### 2.2 ZSet如何从大到小查找范围

分析：
- 基础操作考察

回答：
- ZRANGE命令可以按分数从小到大的顺序查找范围，ZREVRANGE则是逆序

### 2.3 ZSet底层有几种编码方式?

分析：
- 基础知识考察，ZSet的底层编码可以是ZIPLIST或者SKIPLIST+字典，需要分情况说出不同的编码类型。
- ZIPLIST的SKIPLIST+字典的选择阈值不一定需要记得清楚，记不清说大概128或256也可
- 当有ZSet对象可以同时满足一下两个条件的时候，对象使用ZIPLIST编码；
	1. ZSet保存的元素数量小于128个
	2. ZSet保存的所有元素的长度都小于64字节
- 不能满足以上两个条件的有序集合对象将使用SKIPLIST+字典 编码

回答：
- ZSet就是有序集合对象，ZSet对象的底层编码有两种：ZIPLIST 和 SKIPLIST+字典
- 如果一个ZSet对象中的所有元素同时满足：元素数量少于128个，且所有元素的长度都小于64字节，那么会使用ZIPLIST编码，否则使用SKIPLIST+字典 编码。

### 2.4 ZSet查询节点总数的平均时间复杂度是多少？

分析：
- 我们可以查看相关的API底层源码：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae79d0315296928255f9ecbd831a22bc.png)
- 查询有序集合成员总数的API是`zsetLength`，而当编码模式是`REDIS_ENCODING_SKIPLIST`，源码中会直接返回跳表表头结构的length字段，平均时间复杂度是O(1)

回答：
- 无论是ZIPLIST编码，还是SKIPLIST编码，查询节点总数的平均时间复杂度都是O(1)，因为对应结构的表头结构中都定义了一个保存节点数量的字段，查询节点总数时会直接返回这个字段。
### 2.5 ZSet为什么将跳表和HASHTABLE要配合使用？

分析：
- 使用了两种数据结构来实现ZSet，为了让有序集合拥有这两种数据结构的优势。可以结合跳表和字典相较于各自的优势来进行回答。

回答：
- 为了结合这两种数据结构各自的优势，当ZSet要`根据成员来查找分值的时候只用使用字典来查找，时间复杂度为O(1)`。
- 而当Zset要执行范围操作时，比如ZRANK、ZRANGE等命令是，将使用原本就有序的跳跃表来实现。

### 2.6 ZSet为什么用跳表而不是B+树？

分析：
- 这个问题可以结合跳表和B+树的结构，以及B+树是为了解决什么问题而被做我MySql的索引结构来回答。
- B+树属于多路平衡查找树，由于是多叉树所以层数低，可以减少磁盘访问；而且数据值存储在叶子节点中，非叶子节点值存储索引值，使得一次IO读取的数据量很大。这样的设计使得B+树非常适合作为磁盘数据库索引的结构
- 而Redis是内存型数据库，要求的是高效率以及低内存，显然Redis对B+树层数低的有点并不感冒，除此之外跳表占用的内存更少、实现更加简单。而且跳表的插入性能更高，虽然两者的插入平均时间复杂度相当，但是跳表插入数据后只需要修改前进和后退指针即可，而B+树还需要维护树的平衡，比如超过该叶子结点的最大值，此时这个节点就会分裂两个叶子节点，然后需要更新节点指针，来保持树的一个平衡；

回答：
- B+树的`数据都存在叶子节点中`，而且它是`多叉树高低`，这两个特点使得它`适合磁盘存储`。而Redis是一个内存数据库，B+树高低的优势荡然无存，所以选择了实现更加简单的跳表
- 而且跳表的插入性能更高，虽然两者的插入平均时间复杂度相当，但是`跳表插入数据后只需要修改前进和后退指针即可`，而`B+树还需要维护树的平衡，细节上有额外开销`。

### 2.7 ZSet为什么用跳表而不是红黑树？

分析：
- 这个问题可以分析一下Redis作者之前对为什么选择跳表的回复：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/070e70a758523a076e070a094f5040e5.png)
1. 这是和B树相比，主要是说跳表的层高概率可以调整，内存是可以比B树少的，这个应该是应对内存质疑不是重点；
2. ZRANGE这种范围查询查找，在ZSet中用的是最多的，跳表的缓存局部性不会至少比平衡树差
3. 足够简单，这是最核心的

回答：
- 首先，跳表的`实现简单，维护起来也简单`。这是最核心的一个原因，其他不见得有明显优势
- 其次，跳表的`内存使用、缓存局部性`都不比平衡树差
- 所以结合简单这个核心原因，作者就选择了跳表。