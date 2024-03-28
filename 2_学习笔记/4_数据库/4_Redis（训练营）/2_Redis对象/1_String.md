
## 1、是什么

String就是字符串，它是Redis中最基本的数据对象，最大为512MB，我们`可以通过修改redis.conf文件中的配置项proto-max-bulk-len来修改它`，一般来说是不用主动修改的。

## 2、适用场景

一般可以用来`存字节数据、文本数据、序列化后的对象数据`等。

也就是说只要是字符串，都可以往里面存。

具体一点的例子：
- 缓存场景，Value存Json字符串等信息
- 计数场景，因为Redis处理命令是单线程，所以执行命令的过程是原子的。因此String数据类型适合计数场景，比如计算访问次数，点赞，转发，库存数量等等。

## 3、常用操作

常用操作聚焦于创建、查询、更新和删除。

`创建`：即产生一个字符串对象数据，可以用SET、SETNX

`查询`：可以用GET，如果一次获取多个，可以用MGET

`更新`：也是用SET来更新的

`删除`：针对String对象本身的销毁，用DEL命令。

>拓展：一般推荐使用UNLINK来删除KEY，因为DEL是同步删除，删除大KEY的时候容易阻塞主线程，而UNLINK是异步删除，不存在阻塞主线程的问题

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231015151600.png)

### 3.1 写操作

#### 3.1.1 SET

语法：SET key value

功能：设置一个key的值为特点的value，成功则返回OK

String对象的创建或者更新都是该命令

```shell
SET strniuniu cat
OK
```

SET命令很重要，这里多讲`几个扩展参数`：
- EX second：设置键的过期时间为多少秒
- PX millisecond：设置键的过期时间为多少毫秒
- `NX：只有键不存在时，才对键进行设置操作`。SET key value NX 效果等同于SETNX key value，基本是替代了下面的SETNX操作
- XX：只有键已经存在时，才对键进行设置操作

```shell
SET strniuniu fish NX
null
SET strniuniu fish
OK
```

#### 3.1.2 SETNX

语法：SETNX key value

功能：用于在指定的key不存在时，为key设置指定的值，返回值0表示key存在不做操作，1表示设置成功。

如果对存在的Key，调用SETNX
```shell
SETNX strniuniu fish
（integer） 0
```

对不存在的key，调用SETNX
```shell
SETNX strmart fish
(integer) 1
```
>这里使用setnx创建一个新的key-value对时报错的话，可以试试输入这个命令试试。
>`config set stop-writes-on-bgsave-error no`

#### 3.1.3 DEL

语法：DEL key [key ...]

功能：删除对象，返回值为删除成功了几行。

```shell
DEL strmart
(integer) 1
```

### 3.2 读操作

#### 3.2.1 GET

语法：GET key

功能：查询某个key，存在就返回对应的value，如果不存在返回nil

```shell
GET strniuniu
"cat"
GET strmart
(nil)
```

#### 3.2.2 MGET

语法：MGET key [key ...]

功能：一次查询多个key，如果某个key不存在，对应位置返回nil

```shell
MGET strniuniu strmart
1) "cat"
2) (nil)
```

## 4、底层实现

### 4.1 三种编码方式

String看起来简单，但实际有三种编码方式，如下图所示：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231015152623.png)

INT编码：这个很好理解，就是存一个整型，可以用long表示的整数就以这种编码存储；（`能存储8个字节的整数`）

EMBSTR（Embedded String）：如果字符串小于等于阈值字节，使用该编码；

RAW编码：字符串大于阈值字节，则用该编码；
- 直接将字符串`以字节数组的形式存储`，适用于较长的字符串

这个阈值追溯源码的话，是用常量`OBJ_ENCODING_EMBSTR_SIZE_LIMIT`来表示，3.2版本之前是39字节，3.2版本之后是44字节
```c
//from Redis 7.0.8
/* 
Create a string object with EMBSTR encoding if it is smaller than OBJ_ENCODING_EMBSTR_SIZE_LIMIT, otherwise the RAW encoding is used.

The current limit of 44 is chosen so that thr biggest string object we allocate as EMBSTR will still fit into the 64 byte arena of jemalloc.
*/

#define OBJ_ENCODING_EMBSTR_SIZE_LIMIT 44
robj *createStringObject(const char *ptr, size_t len){
	if(len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
		return createEmbeddedStringObject(ptr,len);
	else
		return createRawStringObject(ptr,len);
}
```

EMBSTR和RAW都是由redisObject和SDS两个结构组成，他们的差异在于，`EMBSTR下redisObject和SDS是连续的内存`，`RAW编码下redisObject和SDS的内存是分开的。`

EMBSTR优点是redisObject和SDS两个结构可以一次性分配空间，缺点在于如果重新分配空间，整体都需要再分配，所以`EMBSTR设计为只读`，`任何写操作之后EMBSTR都会变成RAW`，理念是发生修改的字符串通常会认为是易变的。

EMBSTR内存：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231015153940.png)

RAW内存：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231015153952.png)

随着我们的操作，编码可能会转换：

INT -> RAW : 当存的内容不再是整数，或者大小超过long的时候；

EMBSTR -> RAW : 任何写操作之后EMBSTR都会变成RAW，原因前面有解释；

我们注意到，字符串编码EMBSTR和RAW都包含一个结构叫SDS，`它是Simple Dynamic String的缩写，即简单的动态字符串，这是Redis内部作为基石的字符串封装，十分重要`，下面我们将详细介绍一下SDS；

### 4.2 为什么要SDS？

在C语言中，字符串用一个`'\0'`结尾的char数组表示。
比如“`hello niuniu`” 即 “`hello niuniu\0`”

C语言作为一个比较亲和底层的语言，很多基建就是这么简单，但是，对于某个应用来说不一定好用：
1. 每次计算字符长度的复杂度为O(N)
2. 对字符串进行追加，需要重新分配内存
3. [非二进制安全](https://www.cnblogs.com/jing99/p/11687308.html)

在Redis内部，字符串的追加和长度计算很常见，这两个简单的操作不应该成为性能的瓶颈，于是`Redis封装了一个叫SDS的字符串结构`，用来解决上述问题。下面我们来看看它是怎么样的结构

首先，Redis中SDS范围sdshdr8，sdshdr16，sdshdr32，sdshdr64，它们的字段属性都是一样，区别在于应对不同大小的字符串，我们以sdshdr8为例：
```c
//from Redis 7.0.8
struct _attribute_ ((_packed_)) sdshdr8{
	unit8_t len; /* used */
	unit8_t alloc; /* excluding the header and null terminator */
	unsigned char flags; /* 3 lsb of type, 5 unused bits*/
	char buf[];
};
```

其中有两个关键字段，`一个是len，表示使用了多少`；`一个是alloc，表示一共分配了多少内存`。这`两个字段的差值（alloc-len）就是预留空间的大小`。flags则是标记是哪个分类，比如sdshdr8的分类就是
```c
#define SDS_TYPE_8 1
```

从SDS的结构，我们很容易看出SDS是如何对症下药解决问题的：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020231015155843.png)

1. 增加长度字段len，快速返回长度；
2. 增加空余空间（alloc-len），为后续追加数据留余地；
3. 不再以`'\0'`作为判断标准，二进制安全。
	- 读的时候从buf[]里`读取len位`，中间的`'\0'`就不会作为截断标准

这里可能会想，SDS可以预留空间，那么预留空间有多大呢，规则如下：
- len小于1M的情况下，`alloc=2*len`，即预留len大小的空间；
- len大于1M的情况下，`alloc=2*1M`，即预留1M大小的空间；
- 简单来说，预留空间为min(len , 1M)

## 5、总结

本届我们学习了String这个Redis最基础的对象，String可以存储字符数据，文本数据，二进制数据，虽然String看起来比较简单，当实际上Redis采用了几种编码方式来应对不同的场景，以达到性能的极致，在后续的学习中，这种极致的设计也会出现很多次，这也是Redis高性能的原因之一；

下一节，我们将一起分析String的面试点，以及剖析几道容易出现的面试题

