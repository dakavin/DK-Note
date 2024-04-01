
## 1、背景

上一节我们说到，Redis AOF文件随着不断膨胀，会通过重写机制来减少AOF文件大小。

功能上，倒是没啥问题，但是从性能和方案上来说，AOF重写功能都显得比较粗糙，这也是正常的，持久化作为一个相对分支的模块，在前期没有花费太多精力去优化完全可以理解

随着Redis不断完善，AOF重写功能在7.0得到了一个很大程度的优化。

我们下面先说说，原有的AOF重写，存在什么问题。
## 2、原有AOF重写方案弊端

我们详细分析下原有的方案：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f76cd4879d366c70280b85556e427dd1.png)

我们来分析一下，这个方案的优势和劣势。

优势：
1. 方案容易想到，前期快速实现

劣势：
1. 占用主进程CPU时间，在重写期间额外像==aof_rewrite_buf写数据==；==通过管道向子进程发送aof_rewrite_buf中的数据==；以及==子进程结束后将剩余aof_rewrite_buf写入临时文件==，这些都需要`消耗CPU时间的`，而Redis核心执行是单线程的，CPU资源的损耗会带来响应速度的降低。
	- 补充：通过管道传递信息给子进程，让子进程写磁盘，这已经是3.2引入的优化了，更早版本是主进程字节写，这样更容易影响正常请求
2. 额外的内存开销，在AOF重写期间，`主进程会将fork之后的数据变化同时写进aof_buf和aof_rewrite_buf中，这部分内容是重复的`。另外在子进程读取时候，会开辟一个读取内存空间，下面代码展示 了读取时候会开辟内存缓冲区。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fcdaabc1686a5f6f0e1da2afa9500c6d.png)
3. `额外的磁盘开销`，主进程处理会将执行过的写命令写到aof_buf之外，还会写一份 到aof_rewrite_buf中。aof_buf最终写入老文件，aof_rewrite_buf最新写入新的AOF文件，这相当于是两次磁盘消耗，而数据明明是一样的。

为了解决上述问题，我们需要先认清为什么问题的本质是有两个不太合理的设计：
1. 同时向aof_buf和aof_rewrite_buf写入
2. 父子进程传输文件数据
这两个是问题的本质，理清楚了本质，优化起来并不难，Redis7.0引入了MP-AOF方案

## 3、MP-AOF方案概述

MP-AOF，全称 Multi Part AOF，也就是多部件AOF，通俗点来说就是原来是`一个AOF文件，现在变成了多个AOF文件的配合`。

MP-AOF文件核心有两个Part组成：
1. `BASE AOF文件，基础AOF文件，记录了基本的命令`
2. `INCR AOF文件，记录了在重写过程中新增的操作命令`
这两个部件，就可以一起组成完整的AOF操作记录

原来是一个AOF文件，里面包含了操作的命令记录，MP-AOF则是提出来BASE AOF 和 INCR AOF两种文件的组合，一个BASE AOF 结合 INCR AOF，一起来表示所有的操作命令。

我们来看看这种新模型下的流程：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e4698c4b26cb9b442c2b216243dfc163.png)

可以看到，现在重写阶段，在主进程中写AOF缓冲区即可，AOF缓冲区的数据最终落入新打开的2.INCR AOF文件，至于子进程，就根据fork时数据库的数据，重写并生成新的一个2.BASE AOF文件。

新生成的2.BASE AOF和新打开的2.INCR AOF就代表了当前时刻Redis的全部数据。

写完之后，也不是覆盖原有的1.BASE AOF 和 1.INCR AOF，而只需要更新manifest文件，manifest是文件清单，描述了当前有效的BASE AOF、INCR AOF是哪个，这里可以简单理解manifest就是一个配置

那之前旧的1.BASE AOF、1.INCR AOF文件怎么办呢？Redis是将他们标记为HSTORY AOF，`这些HISTORY AOF会被Redis异步删除掉`。

这里为了给大家加深印象，也额外的说下，上面的1.BASE AOF、1.INCR AOF知识为了方便描述和记忆，他的名字其实不是这个，`实际的名字长这个样子`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7886e16a6917ecd9ccf977092b61c441.png)
至于manifest里面的内容，实际是长这个样子：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7dc2a8b4dc7af068a5aaefc655be258e.png)
>ps:appendonly.aof.1.base.rdb 是因为7.0默认开启了混合持久化，所以base后缀是rdb，这里不冲突，不开启他就还是aof


通过上述讲解，我们可以看到：
1. 我们在AOFRW期间不再需要aof_rewrite_buf
2. 我们也不需要父子进程通过管道传输操作数据
如此一来，CPU、内存、磁盘的性能损耗会降低很多，甚至代码复杂度也会降低，整体更加清晰，这也符合Redis简洁的理念。

## 4、总结

原有AOF重写因为额外引入了aof_rewrite_buf，以及相关联的会引入父子进程操作数据通信，会存在性能损耗等问题，为了解决这个问题Reids 7.0引入了MP-AOF方案，非常对症下药地解决了上述问题。

MP-AOF算不上是一个很热的面试考点，但是它的优化思路是非常值得我们学习的，如果面试中遇到AOF重写的问题，我们甚至可以主动讲述MP-AOF，这也可能成为一个加分点。