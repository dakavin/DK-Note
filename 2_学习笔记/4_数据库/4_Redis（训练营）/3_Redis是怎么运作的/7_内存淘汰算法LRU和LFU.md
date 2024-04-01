
# 一、LRU

LRU一直以来都是一个非常流行的资源淘汰算法，为了减少内存消耗，Redis使用了近似LRU，下面我们来介绍一个LRU算法是什么，为什么Redis不用标准的LRU，Redis的近似LRU是怎么样的？
## 1、LRU算法是什么

最近很少使用，即记录每个Key的最近访问时间，`维护一个访问时间的数据`

## 2、Redis用LRU算法会有什么问题

如果为所有数据维护一个顺序列表，实际就是做一个双向链表，但是如果Redis数据稍微多些，这个链表就是巨大的成本，对于Redis而言，内存是最宝贵的，所以Redis选择了近似LRU算法。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/76b4c148163a6057bd78005d910cedf3.png)

## 3、Redis近似LRU算法

维护一个`全局链表`，对Redis来说是巨大的成本，所以`Redis选择采样的方式`来做，也就是近似LRU算法。
### 3.1 算法概述

在LRU模式，redisObject对象中lru字段存储的是key被访问时Redis的时钟server.lrulock，`当key被访问的时候，Redis会更新这个key的redisObject的lru字段`

注意，`Redis为了保证核心单线程服务性能，缓存了Unix操作系统时钟`，默认每100毫秒更新一次，缓存的值是Unix时间戳取模2^24

近似LRU算法在现有数据结构的基础上`采用随机采样的方式来淘汰元素`，当内存不足时，就执行一次近似LRU算法。

具体步骤就是随机采用n个key，这个采样个数默认是5，然后根据时间戳淘汰最旧的那个key，如果淘汰后内存还是不足，就继续随机采样来淘汰

### 3.2 采样范围

Redis可以选择范围策略，有两种：
- allkeys，所有key中随机采样；---`allkeys-lru`
- volatile从有过期时间的key随机采样；---`volatile-lru`

### 3.3 淘汰池优化

近似LRU，优点在于节约了内存，但是计算机世界没有银弹，它的缺点就是不是一个完整的LRU，随机采用得到的结果，其实不是全局正在的最久未访问，在数据量大的情况，尤其是这样。

Redis3.0对近似LRU算法进行了一些优化。

新算法会维护一个大小为16的候选池，池中的数据根据访问时间进行排序。第一次随机选取的key都会放入池中，然后淘汰掉最久未访问的，比如`第一次选了5个，淘汰了1个，剩下4个继续留在池子里`。

随后`每次随机选取的key只有活性比池子里活性最小的key还小时才会放入池中`，当`池子满`了，如果`有新的key需要分区，则将池中活性最大的key移除`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bd5db29b067adea3d814bbea2aa5173c.png)

通过池子存储，在池子里的数据会越来越接近真实的活性最低，所以其表现也非常接近真正的lRU，下面我们来看看LRU算法对比，从其中也能看出相同结论。

## 4、LRU算法对比

这是Redis官方做测试之后的效果图：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e1dcc8d3f1d5f8471aa426f222e60ed5.png)

可以看到途中有三种不同颜色的点：
1. 浅灰色是被淘汰的数据
2. 灰色是没有被淘汰的老数据
3. 绿色是新加入的数据

我们能看到Redis3.0采样数是10，生成的图最接近于严格的LRU。
而同样使用5个采样数，Redis3.0也要优于Redis2.8
# 二、LFU

## 1、LFU是什么？

LFU淘汰算法，即Least Frequently Used，最不频繁淘汰算法，顾名思义，有限淘汰活跃度低，使用频率最低的。

## 2、为什么4.0引入LFU

LRU本身已经能解决大部分问题，但是脱离频率，只谈最近访问，在部分场景是得不到我们希望的结果，比如：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d99a1112947ff60d14bcbab133d5735b.png)

如上所示，key niuniu频率很高，key mart虽然是最近访问的，但是实际频率低（我们假设没有其他key的干扰），如果内存满了，会淘汰key niuniu

但如果用LFU的话，会淘汰key mart，可以保留频率较高的key niuniu

希望按访问频率来进行淘汰，这其实是一个很正常的需求，LFU就是专门做这个事的。
## 3、LFU

我们复习下redisObject，其结构如下：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d9447482ba197c96edb21c2e5f099d6f.png)

如果使用LRU，那么redisObject中lru字段，就是用来存储最近访问时间的，这个字段长度是LRU_BITS，这个值一致都是24位。

如果是LFU、因为LRU、LFU是不会同时开启，所以两者可以说是互斥，基于这种情况，加上节约内存的考虑，`Redis在LFU策略下复用lru字段，还是用它来表示LFU的信息，不过将24拆解，高16bit存储ldt（Last Decrement Time），低8bit存储logc（Logistic Counter）`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8521ecaa6338cc0cf909ecc99e73eb24.png)

如图所示，高的16位保存了上次访问时间戳，因为少了8位，所以LFU时间精度是1分钟，不然用秒的话2^16次方只能表示65536秒，后8位存储的是一个访问计数。

一个Key是否活跃，就是这两个字段综合决定的。

如果上一次访问时间很久，那么访问计数就会衰减，这里有人问为什么要衰减，因为只是简单的增加访问计数的方法并不完美，访问的热度一直在变，比如一个key，原来是255，夸张一点，一年没访问了，不该归零吗？

而本身访问，会增加访问计数。

当然，最后起作用的就是访问计数。

### 3.1 源码分析

为了进一步理解，我们来看看源码是怎么写的。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/82063e25e76579845c57ef12e70be7a5.png)

LFU_INIT_VAL初始值为5
/#define LFU_INIT_VAL 5

前8行是对redisObject的初始化。
后面是对redisObjecy中的lru字段进行初始化，如果达到了maxmemory且策略是LFU，那么前16位设置时间戳，后8位设置访问计数，如果是LRU策略，lru字段整个24位都是当前时间戳

### 3.2 LFU数据更新（发生在某个Key被访问到）

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/11ce5a4c9d1bba6777e7b48f9fbc2d7d.png)

#### 3.2.1 第一步，计算次数衰减

- `先按照上次访问距离当前的时长，来对 logc 进行衰减；`

因为无论访问是多频繁，相对于上次访问，一定有时间间隔，根据间隔，来计算你应该减少的次数。使用的函数就是LFUDecrAndRetuen。

为什么要衰减，因为只是简单的增加访问计数的方法并不完美，访问的热度一直在变。

#### 3.2.2 第二步，一定概率增加访问计数

这里可能会疑问，为什么是一定概率增加访问计数，次数不足5次，那一定会增加，如果大于5次，小于255次，会一定概率加1，原来的次数越大，越困难。

处理原来的次数影响外，还有一个lfu-log-factor参数可以被设置的。也就是说，可以通过这个参数来调节难度，这个越大，难度也越大，如果为0，那么每次必然+1，很快就能255,255[10] -> 1111 1111[2]，255就是最大值

默认是10，需要1M流量才能达到最大值。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6013c8172524cb12ece0bca5ac3d667d.png)

#### 3.2.3 第三步，更新

当前时间更新到高16位，次数更新到低8位。