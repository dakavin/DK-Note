
## 1、面试考点分析

Hash是字典对象，它和Set一样底层编码模式有HASHTABLE。

面试的考察点集中在Hash常规操作和底层实现，常规操作必须熟练掌握，底层实现要了解其编码方式，其中HASHTABLE是重中之重，面试大热门考点，下一节会单独讲解HASHTABLE。

## 2、面试题

### 2.1 Hash是有序的吗？

分析：
- Hash的底层实现是ZIPLIST和HASHTABLE，他们都是无序的

回答：
- Hash的底层实现是ZIPLIST或HASHTABLE，他们都是无序的。所以Hash是无序的。

### 2.2 如何查看Hash所有成员？如何查找某个成员

分析：
- 基本操作考察

回答：
- HGETALL可以查看Hash所有成员，HGET可以以成员为关键字，查找对应的数据
### 2.3 如何查看Hash中成员个数？

分析：
- 基本操作考察

回答：
- HLEN可以查看Hash中成员总数，参数就是Hash对象，即HLEN key
### 2.4 HASH的编码方式是什么？

分析：
- Hset底层有两种编码结构，ZIPLIST和HASHTABLE
- ZIPLIST使用于元素较少且单个元素长度较少的情况，这里的阈值分别为元素个数少于512个，值和键长度都小于64字节。

回答：
- 一个是ZIPLIST、一个是HASHTABLE。ZIPLIST适用于元素较少且单个元素长度较小的情况，其他情况使用HASHTABLE。
### 2.5 Hash查找某个key的平均时间复杂度是多少？

分析：
- 这种问对象复杂度的，要考虑多种底层编码。ZIPLIST需要遍历，平均复杂度为O(N)，HASHTABLE是字典，可以O(1)找到对应的key

回答：
- HSet有两种底层结构，ZIPLIST是O(N)，HASHTABLE则是O(1)
### 2.6 Hash为什么要用两种编码方式

分析：
- 对象使用多种编码方式是Redis的一个特点，我们前面学是String、List、Set都是如此，核心就是因地制宜，在不同场景下使用最合适的方式。

回答：
- 采用两种编码方式的原因是ZIPLIST更节约内存，所在小数据量时使用。而数据多起来了，需要HASHTABLE提供更高的查找性能、更新性能。