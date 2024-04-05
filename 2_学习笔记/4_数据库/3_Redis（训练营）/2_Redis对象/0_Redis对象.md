
## 1 Redis Object是什么？

Redis 是key-value存储，`key和value在Redis中都被抽象为对象`

`key只能是String对象`，而`Value支持丰富的对象种类`，包括String、List、Set、Hash、Sorted Set、Stream等

## 2 Object在内存中是什么样子

redisObject定义如下：
```C
// from Redis 5.0.5
#define LRU_BITS 24

typedef struct redisObject{
	unsigned type:4;
	unsigned encoding:4;
	unsigned lru:LRU_BITS; /* LRU time or
							* LFU data */
	int refcount;
	void *ptr;
}robj;
```

`type`：是哪种Redis对象

`encoding`：表示用哪种底层编码，用`OBJECT ENCODING[key]`可以看到对应的编码方式

`lru`：记录对象访问信息，用于内存淘汰，这个可以先忽略，后续章节会详细介绍

`refcount`：引用计数，用来描述有多少个指针，指向该对象

`ptr`：内容指针，指向实际内容

## 3 对象与数据结构

实际操作的主要有`6个Redis对象`，他们底层依赖一些数据结构，包括字符串、跳表、哈希表、压缩列表、双端链表等，不同对象可能有依赖相同的数据结构，有些教程会先讲这些底层的结构，但是本课程会以对象为驱动，讲到某个对象的时候，再介绍关联的数据结构。

这也是本课程的一个准则，实践优于原理，对使用者而言，不会直接使用底层数据结构，直接打交道的是对象，先熟悉对象操作，再自然探究对象底层结构，这种关联性学习的坡度也会平缓些。


## 4 Redis通用命令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

- `KEYS`：查看符合模板的所有key
	- 因为redis中每个查询是单线程的，所以不建议在生产环境设备上使用keys
- `DEL`：删除一个指定的key
- `EXISTS`：判断key是否存在
- `EXPIRE`：给一个key设置有效期，有效期到期时该key会被自动删除
- `TTL`：查看一个KEY的剩余有效期

通过help [command] 可以查看一个命令的具体用法，例如：

```sh
# 查看keys命令的帮助信息：
127.0.0.1:6379> help keys

KEYS pattern
summary: Find all keys matching the given pattern
since: 1.0.0
group: generic
```

