---
文章标题: "[[5_Hash]]" 
文章作者: Dakkk
文章概要: |
  介绍Redis Hash数据结构的基本概念、适用场景、常用操作命令（HSET、HGET等）及底层编码实现（ZIPLIST和HASHTABLE）
tags:
- "Redis"
- "Hash"
- "数据结构"
- "键值存储"
- "HSET"
- "HGET"
- "ZIPLIST"
- "HASHTABLE"
相关文章:
- "[[5_Hash面试与分析]]"
- "[[2_List面试与分析]]"
- "[[3_压缩列表面试与分析]]"
- "[[6_HASHTABLE面试与分析]]"
- "[[6_底层结构HASHTABLE]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/02-🔴 Redis/03-🎓 训练营笔记/2_Redis对象/5_Hash.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:57
---

## 1 是什么？

Redis Hash是一个field、value都为string的hash表，存储在Redis内存中。

Redis中每个hash可以存储`2^32-1`键值对（40多亿）。

## 2 适用场景

适用于O(1)时间字典查找某个field对应数据的场景，比如任务信息的配置，就可以任务类型为field，任务配置参数为value。

## 3 常用操作

我们还是从创建、查询、更新、删除这几个基本操作来了解Hash。

创建：产生一个Hash对象，可以使用HSET、HSETNX创建。

查询：HGET查询单个元素；HGETALL查询所有数据；HLEN查询数据总数；HSCAN进行游标迭代查询

更新：HSET可以增加新元素，HDEL删除元素

删除：DEL可以删除一个Hash对象

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e7da5a79b123caa0e731008c77063546.png)

### 3.1 写操作

#### 3.1.1 HSET

语法：`HSET key field value`
功能：为集合对应field设置value数据

```shell
127.0.0.1:6379> HSET hsetniuniu f1 v1 f2 v2 f3 v3
(integer) 3
```
#### 3.1.2 HSETNX

语法：`HSETNX key field value`
功能：如果field不存在，则为集合对应field设置value数据

```shell
127.0.0.1:6379> HSETNX hsetniuniu f1 newv1
(integer) 0
127.0.0.1:6379> HSETNX hsetniuniu f4 v4
(integer) 1
127.0.0.1:6379> HSETNX hsetniuniu f5 v5
(integer) 1
```

由于f1已经存在，所以不会再设置f1，f1保持原来的value，即v1

f4、f5原来不存在，设置为指定value

#### 3.1.3 HDEL

语法：HDEL key field [field ...]
功能：删除指定field，可以一次删除多个

```shell
127.0.0.1:6379> HDEL hsetniuniu f4 f5
(integer) 2
```
#### 3.1.4 DEL

语法：DEL key [key ...]
功能：删除Hash对象

```shell
127.0.0.1:6379> del hsetniuniu
(integer) 1
```
#### 3.1.5 HMSET

语法：HMSET key field value  [field value ...]
功能：可以设置多个键值对。

在Redis使用过程中，会发现Redis Hash的两个智联HSET和HMSET非常类型，网上有一种说法是HMSET支持一次设置多个field到哈希表，而HSET则不能，这是错误的，实际HSET是可以设置多个field的。

我们阅读官方文档可以发现一下说明：
>As Per Redis 4.0.0, HMSET is considered deprecated. 
>Please use HSET in new code.

在Redis4.0.4，HMSET被视为已弃用。请在写代码中使用HSET

也就是说，在4.0.0之前，HSET只能设置单个键值对。同时设置多个时必须使用HMSET。而现在HSET也允许接受多个键值对做参数了。
### 3.2 读操作

#### 3.2.1 HGETALL

语法：HGETALL key
功能：查找全部数据

```shell
127.0.0.1:6379> HGETALL hsetniuniu
1) "f1"
2) "v1"
3) "f2"
4) "v2"
5) "f3"
6) "v3"
```
#### 3.2.2 HGET

语法：HGET key field
功能：查找某个key

```shell
127.0.0.1:6379> HGET hsetniuniu f2
"v2"
```
#### 3.2.3 HLEN

语法：HLEN key
功能：查找Hash中元素总数

```shell
127.0.0.1:6379> HLEN hsetniuniu
(integer) 3
```
#### 3.2.4 HSCAN

语法：`HSCAN key cursor [MATCH pattern] [COUNT count]`
功能：从指定位置查询一定数量的数据，这里要注意，如果是小数据量下，出于ZIPLIST时，COUNT不管填多少，都是返回全部，因为ZIPLIST本身就用于小集合，没必要说再切分成几段来返回

```shell
127.0.0.1:6379> HSCAN hsetniuniu 0 COUNT 10
1) "0"
2) 1) "f1"
   2) "v1"
   3) "f2"
   4) "v2"
   5) "f3"
   6) "v3"
```

使用MATCH匹配
```shell
127.0.0.1:6379> HSCAN hsetniuniu 0 MATCH *2
1) "0"
2) 1) "f2"
   2) "v2"
```
## 4 原理

### 4.1 编码格式

Hash底层有两种编码结构，一个是压缩列表，一个是HASHTABLE。同时满足一下两个条件，用压缩列表：
1. Hash对象保存的所有值和键的长度都小于64字节；
2. Hash对象元素个数少于512个。

两个条件任何一条都不满足，编码结构就用HASHTABLE
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f92ac6bda641f286920595beac0a93c7.png)

ZIPLIST之前有讲解过，其实就是在数据量较小时间数据紧凑排列，对应到Hash，就是将field-value当做entry放入ZIPLIST，结构如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3339a3f80e6d569e86cd0484f4221abc.png)
HASHTABLE在之前无序集合Set中也有应用，和Set的区别在于，在Set中value始终为NULL，但是在HSet中，是有对应的值的。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3efb18bee0101cc68e2c7e6ff8ef46f.png)
## 5 总结

Hash是字典，可以存储多个field-value的映射关系，比如学生分数、任务配置等。

Hash的底层编码有ZIPLIST和HASHTABLE两种，这两种数据结构之前分别在List和Set章节出现过，书序底层数据结构，需要理解。