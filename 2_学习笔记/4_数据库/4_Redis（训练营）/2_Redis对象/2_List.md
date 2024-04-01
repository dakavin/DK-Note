
## 1、是什么？

Redis List是一组连接起来的字符串集合

### 1.1 有什么限制

List最大元素个数是2^32-1 (4294967295)，`新版本已经是2^64-1了`

## 2、适用场景

List作为一个列表存储，属于比较底层的数据结构，可以使用的场景非常多，比如存储一批任务数据，存储一批消息等。

## 3、常用操作

常用操作还是聚焦于创建、查询、更新和删除

创建：产生一个List对象，一般用LPUSH、RPUSH，分别对应从左侧进队列和右侧进队列。

查询：可以用LRANGE来进行范围查询，可以用LLEN查询元素个数。

更新：对列表对象而言就是追加元素、删除元素等操作，涉及LPUSH、LPOP、RPUSH、RPOP、LREM等操作。

删除：针对List对象本身的生成和销毁，上一章将String的时候我们有介绍，可以用DEL命令，下面再介绍一种方式：通过4.0引入的UNLINK命令进行对象删除。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8f92589bcb571c52ebb263c8bdb2a196.png)

下面是一些常用命令的例子

### 3.1 写操作

#### 3.1.1 LPUSH

语法：LPUSH key value [value ...]
功能：从头部增加元素，返回值为List中元素的总数。

```shell
LPUSH list s1 s2 s3
(integer) 3

LPUSH list s4
(integer) 4
```

s4是第二个命令插入的，插入之后它就在List头部，此时List结构如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d281a5db4ef44a56a4dcaa668eb4654d.png)
#### 3.1.2 RPUSH

语法：RPUSH key value [value ...]
功能：从尾部增加元素，返回值为List中元素的总数。

```shell
RPUSH list s5
(integer) 5
```

s5是RPUSH命令插入的，插入之后它就在List尾部，此时List结构如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4504727a0943334742efe26d37944019.png)
#### 3.1.3 LPOP

语法：LPOP key
功能：移除并获取列表的第一个元素

```shell
LPOP list
"s4"
```

s4从List中移除，此时List结构如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7284e68f03987e104a6646d0aa12a3e6.png)
#### 3.1.4 RPOP

语法：RPOP key
功能：移除并获取列表的最后一个元素

```shell
LPOP list
"s5"
```

s5从List中移除，此时List结构如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80512ab173eaf46ccec8d8bfb09cb672.png)
#### 3.1.5 LREM

语法：LREM key count value
功能：移除值等于value的元素。当count=0，则移除所有等于value的元素。当count>0，则从左到右开始移除count个。`当count<0则从右到左移除count个`。返回值为被移除元素的数量。

```shell
LREM list 0 s2
(integer) 1
```

#### 3.1.6 DEL

语法：DEL key [key ...]
功能：删除对象，返回值为删除成功了几个键

```shell
DEL list
(integer) 1
```

#### 3.1.7 UNLINK

语法：UNLINK key [key ...]
功能：删除对象，返回值为删除成功了几个键。和DEL主要有如下不同：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d48d1e7917c3b04e1cb620be9cf20a03.png)
简单来说：
- del命令同步删除命令，会阻塞客户端，指导删除完成
- unlink命令是异步删除命令，只是取消key在键空间的关联，让其不能再查到，删除是异步进行，所以不会阻塞客户端

```shell
LPUSH list s1 s2 s3
(integer) 3

UNLINK list
(integer) 1
```

### 3.2 读操作

我们使用如下命令，构造列表用于读操作的学习：
```shell
DEL list
(integer) 1

LPUSH list s1 s2 s3
(integer) 3
```

执行完之后，数据如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5037d56c6f3820cf5eb072e77d1c0cae.png)
#### 3.2.1 LLEN

语法：LLEN key
功能：查看List的长度，即List中元素的总数

```shell
LLEN list
(integer) 3
```

#### 3.2.2 LRANGE

语法：LRANGE key start stop
功能：查看start到stop为角标的元素，`[0,-1]表示查看所有元素`

```shell
LRANGE list 0 1
1) "s3"
2) "s2"
```

start、stop如果为负数，表示倒数第几个元素：
```shell
LRANGE list -2 -1
1) "s2"
2) "s1"
```

## 4、底层实现

### 4.1 编码方式

3.2版本之前，List对象有两种编码方式，一种是ZIPLIST，另一种是LINKEDLIST![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e402e746e2c934151d1913bc28def3ef.png)
当满足如下条件时，使用ZIPLIST编码：
1. 列表对象保存的所有字符串对象`长度都小于64字节`；
2. 列表对象元素`个数少于512个`，注意，这是LIST的限制，而不是ZIPLIST的限制；

`ZIPLIST底层用压缩列表`实现，这里我们假设列表中包含“hello”、“niuniu”、“mart”三个元素，ZIPLIST编码示意如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bf7ac316ae127c2e610559b683c6c044.png)
可以看到，“hello”、“niuniu”、“mart”都挨在一起，正如其名字一样，`ZIPLIST内存排列得很紧凑，可以有效节约内存空间`

其他几个数据字段我们这里可以先不关心，下一节会详细讲解ZIPLIST结构，目前我们只需要能宏观理解压缩列表的数据是紧凑相连即可。

如果不满足ZIP编码的条件，则使用LINKEDLIST编码。为了方便描述，我们还是假设列表中包含“hello”、“niuniu”、“mart”三个元素，如果使用LINKEDLIST编码，则是几个String对象的链接结构，结构示意图如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f01058ef89fda8c32b288b0ba732807.png)
可以看到，“hello”、“niuniu”、“mart”这个几个数据是以链表的形式连接在一起，实际上删除更为灵活，但是内存不如ZIPLIST紧凑，所以只有在列表个数或节点数据长度比较大的时候，才会使用LINKEDLIST编码，以`加快处理性能，一定程度上牺牲了内存`。

### 4.2 QUICKLIST横空出世

上面分析有说到，ZIPLIST是为了在数据较少时节约内存，LINKEDLIST是为了数据多时提高更新效率，ZIPLIST数据稍多时插入数据会导致很多内存复制。

但如果节点非常多的情况，LINKEDLIST链表的节点就很多，会占用不少内存。这种情况有没有办法优化呢？

3.2版本就引入了QUICKLIST。QUICKLIST其实就是ZIPLIST和LINKEDLIST的结合体。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61b759acc73db2a27a2b8e1eac9ea2f5.png)
LINKEDLIST原来是单个节点，只能存一个数据，现在单个节点存的是ZIPLIST，即多个数据。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c20c44f9c7194c1443472d065b3a3fa.png)

这个方案其实是用ZIPLIST、LINKEDLIST综合的结构，取代二者本身。
当数据较少的时候，QUICKLIST的节点就只有一个，此时其实相当于就是一个ZIPLIST。
当数据很多的时候，则同时利用了ZIPLIST和LINKEDLIST的优势

### 4.3 ZIPLIST优化

ZIPLIST本身存在一个连锁更新的问题，所以在Redis 7.0之后，使用了LISTPACK的编码模式取代了ZIPLIST，而他们其实本质都是一种压缩的列表，所以其实可以统一叫做压缩列表。

ZIPLIST的细节，以及LISTPACK是怎么优化的，我们将在下一节介绍

## 5、总结

List是Redis中的列表实现，它是一个可双端操作的对象，编码的底层数据结构设计ZIPLIST和LINKEDLIST。下一节，我们将详细分析LIST的底层数据结构压缩列表。