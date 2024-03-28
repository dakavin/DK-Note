
## 1、介绍

Redis是内存存储，前面有提到，我们放在Redis的数据，都是以键值对形式，存在内存，本次我们将深入学习Redis的底层结构到底是什么样子

## 2、数据库结构

redisDb代表Redis数据库结构，上一章，我们讲到的`各种操作对象，就是存储在dict数据结构里`，本节后面的讲述中，我们都会以上一章节的操作对象为例来进行讲解

```C
// from Redis 5.0.5

/*
Redis database representation. There are multiple databases identified by integers from 0 (the default database) up to the max configured database. The database number is the 'id' field in the structure.
*/
typedef struct redisDb{
	/* The keyspace for this DB */
	dict *dict;
	/* Timeout of keys with a timeout set */
	dict *expires;
	/* Keys with client waiting for data（BLPOP） */
	dict *blocking_keys;
	/* Blocked keys that received a PUSH */
	dict *ready_keys;
	/* WATCHED keys for MULTI/EXEC CAS */
	dict *watched_keys;
	/* Database ID */
	int id;
	/* Average TTL, just for stats */
	long long avg_ttl;
	/* List of key names to attempt to defrag one by one ,gradual */
	list *defrag_later;
}redisDb;
```

这里我们`重点关注dict结构，其他字段可以先忽略`，它代表了我们存储的key-value存储，我们平常添加数据，就是往dict里添加。可以看到，dict其实就是我们前面介绍的Hash对象结构。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021190246.png)

下图可以更形象的说明：`redisDb即数据库对象，指向了数据字典，字典里包含了我们平常存储的k-v数据，k 是字符串对象，value支持任意Redis对象`，这些对象前面我们有深入介绍过。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021190345.png)

下面我们来看看增加、查询、更新、删除的操作，分别在内存存储是作用的体现。
## 3、添加数据

即将键值对，添加到dict结构字典中去，Key必须是String对象，Value为任何类型的对象都可以。

比如，我们使用命令：`SET hellomsg "hello mart"`，键空间会变成如下结构
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021190558.png)
## 4、查询数据

数据查询很清晰，在dict找到对应的key，即完成了查询。

## 5、更新数据

对已有的Key对象的任何变更操作，都是更新，比如针对String对象赋予新值，比如给List对象增减元素；

举个例子，使用命令 `RPUSH animals bird`，成功后得到的结构如下：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021190736.png)
## 6、 删除数据

删除即把Key和Value都从dict结构里删除。比如我们在原有基础上，使用命令：`DEL scoredic`，成功之后的结构会如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021190830.png)
## 7、过期键

过期键我们有介绍过，Redis数据都可以设置过期键，这样到了一定时间，这些对象就会自动过期并回收。那么过期键，又是存储在哪里呢？

我们再看一下redisDb结构，是的，过期键是存在expires字典上。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021191301.png)

假设上面例子的Key，都设置了过期时间，那么其结构如下：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231021191406.png)

注意，这里的dict中和expires中的Key对象，`实际都是存储的String对象的指针`，所以并不是会重复占用内容，Redis对内存的使用都是很珍惜的。
## 8、总结

Redis核心是字典存储，里面包含了各种key-value的映射。

我们最常用的对象操作，实际也都是在字典查找、字典更新