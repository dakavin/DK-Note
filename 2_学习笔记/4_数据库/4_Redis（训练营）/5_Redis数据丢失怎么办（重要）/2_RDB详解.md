## 1、怎么开启RDB持久化

我们先从实践入手，看一下怎么开启RDB持久化。

- 首先，我们打开redis配置文件，window是`G:\Redis\Redis-x64-5.0.14.1\redis.windows.conf`，在其中搜索可以发现如下配置：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0f9930279d06c89dd98f305e6b483da.png)
- 这里的配置语法是save interval num，表示没间隔interval秒，至少有num条写数据操作，写数据操作指的是增加、删除及更新，就会激活RDB持久化。
- 上面有3条save配置，他们的意思分别是：
	- 每900s，有1条写数据操作；
	- 每300s，有10条写数据操作；
	- 每60s，有10000写数据操作。
- 他们之间是并集关系，即只要满足其中一个条件，就达到RDB持久化的条件，有人会问，这里触发的save命令，还是bgsave命令？这里显然是bgsave，如果用save，时不时地阻塞一下怎么行。

这三条配置不是我们增加，是默认就存在的，这就是说redis默认已经开启了RDB持久化。

## 2、RDB文件存哪里

下面的参数决定了文件会存到哪里

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fcddedf211f2a40cdcf7529dbf81204e.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92b17dd52e14de748ac1aa68905f7de1.png)

RDB文件最终会长这个样子，是二进制文件，没有可读性，但是要主要前面有个REDIS字符串作为标记，这个后面讲混合持久化也会用到。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8f346aef2dfe69bd3f2fa96a96a8c688.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/45b06c3268744de7cac5d28dd131d231.png)

## 3、什么时候进行持久化

Redis持久化会在下面三种情况下进行：

1. `主动执行命令save`
```shell
127.0.0.1:6379> save
OK
```
Redis服务会有对应输出
- `ps：daemonize的值默认为no时，不会停留在Redis服务启动页面`
```shell
[2016] 24 Oct 22:50:51.673 * DB saved on disk
```
执行了save命令，就会在主线程生成RDB文件，由于和执行操作命令在同一个线程，所以如果写入RDB文件时间太长，会阻塞主线程，这个命令慎用。

2. `主动执行命令bgsave`
```shell
127.0.0.1:6379> bgsave
Background saving started
```
Redis服务会有对应输出
```shell
[2016] 24 Oct 22:52:11.621 * Background saving started by pid 12092
[2016] 24 Oct 22:52:11.678 # fork operation complete
[2016] 24 Oct 22:52:11.684 * Background saving terminated with success
```
和save不同，会创建一个子进程来生成RDB文件，这样可以避免主线程的阻塞。

3. `达到持久化配置阈值`
	- 上面有提到，Redis可以配置持久化策略，达到策略就会触发持久化，这里的持久化使用的方式是bgsave，从这可以看到Redis比较推荐的方式也是后台持久化，尽可能减少对主线程的影响
	- 达到阈值之后，由周期函数触发持久化。

4. `在程序正常关闭的时候执行`
	- 在关闭时，Redis会启动一次阻塞式持久化，以记录更全的数据![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1279ad998c3118f307765b183cea465.png)
	- 所以正常关闭丢失的数据不会丢失，崩溃才会（如果考虑主从同步的话，主节点没崩溃，从节点崩溃就算主节点正常关闭，也可能丢失数据）
		- 主节点挂了，从节点变成了主节点，就可能丢失数据
## 4、RDB具体做了什么

我们这里聚焦达到RDB持久化策略时，是如何进行持久化的，我们先看一下Redis输出：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/70e53dae698241441cccc1cac8ed745e.png)

从输出能看出，RDB确实是通过子进程来进行的，具体做了什么呢？官网写得很清楚
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/46239155d00814ec5ef7576f3400b0a2.png)

从整体上，是做了一下事项：
1. Fork出一个子进程来专门做RDB持久化
2. 子进程写数据到临时的RDB文件
3. 写完之后，替换旧的RDB文件

整体流程如下：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2e2461bacc2442e8195453b6066dadd.png)

下面还有一句：This method allows Redis to benefit from copy-onwrite semantics.

就是说这种方式`让Redis从写时复制技术受益`，Redis官方文档基本没废话，这句话看似无关轻重，实际上说明了：
- 执行RDB持久化过程中，Redis依然可以继续处理操作命令的，也就是数据是能被修改的，这就是通过写时复制技术实现的。

具体而言：fork创建子进程之后，通过写时复制技术，`子进程和父进程是共享同一片内存数据的`，因为创建子进程的时候，会复制父进程的页表，但是`页表指向的物理内存还是同一个`。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/047329619602fd453e4c1edecd678026.png)

只有在发送修改内存数据的情况时，物理内存才会被复制一份
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eec9c634de175d1dd2dc473f6b73336a.png)
`因为主进程要copy出一块物理内存，用于自己的写操作，旧的共享物理内存交给子进程`

就是这样，Redis使用bgsave对当前内存中的所有数据做快照，这个操作是由bgsave子进程在后台完成的，执行时不会阻塞父进程中的主线程，这就使得主线程同时可以修改数据。

这样的目的是为了减少创建子进程时的性能损耗，从而加快创建子进程的速度，毕竟创建子进程的过程中，是会阻塞主线程的。

可以看到，bgsave期间，`读数据互不影响`，如果由`写操作`发送，则`主进程复制一份内存，在这个复制的内存基础上，主进程再修改原来的数据，子进程持久化的依然是修改之前的数据`。

## 5、总结

RDB本质是Redis数据快照，这种方式是最常见、最稳定的数据持久化手段，Redis中RDB的触发方式有三种：
- 达到阈值周期函数触发
- 正常关闭Redis服务触发
- 主动执行BGSAVE命令触发

Redis是通过fork一个子进程的方式来进行RDB，配合写时复制技术，相当于是异步执行，和主进程互不干扰，将对执行流程的影响降到最低。