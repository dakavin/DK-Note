## 1 怎么开启AOF

打开Redis配置文件

在其中可以看到如下配置：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8cd00b1f922ed5d848f55f608a71d2d.png)

appendonly设置为yes，即可打开AOF，从这个现象来看，可以看到Redis官方任务RDB绝大多数时候都还是需要的，而AOF更依赖实际的业务场景。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/69d3d6cab74351e15745cbd10a0653e3.png)

打开之后， Redis每条更改数据的操作都会记录到AOF文件中，当你重启，AOF会助你重建状态，相当于就是请求全部重放一次，所以AOF恢复起来会比较慢。

## 2 AOF写入流程

从上面的描述，我们可以看出，执行请求时，每条日志都会写入到AOF。

**注意：只会记录写操作命令，读操作命令是不会被记录的**，因为没意义。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/beb09e41a3ef237e6a93b7978b43de05.png)

这不免会让人担心，是否会影响Redis的执行性能，答案是肯定的，多了一步操作，或多或少都会带来损耗，但是Redis实际是提供了不同的策略来选择不同程序的损耗。

我们先从比较宏观的视角，介绍Redis提供的3种刷盘策略，以便根据需要进行不同的选择。
1. `appendfsync always`：每次请求都刷入AOF，用官方的话说，非常慢，非常安全。
2. `appendfsync everysec`：每秒刷一次盘，用官方的话说就是足够快了，但是在崩溃场景下你可能会丢失1秒的数据。
3. `appendfsync no`：不主动刷盘，让操作系统自己刷，一般情况Linux会每30s刷一次盘，这种策略下，可以说性能的影响最小，但是如果发生崩溃，可能会丢失相对比较多的数据。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1a5e17d023a07959b8efeedf6eb88cc2.png)

Redis建议是方案二，也就是每秒刷一次盘，这种方式下速度也足够快了，同时崩溃时损失的数据只有1s，这在大多数场景都是可以接受的。

当然了，我们要根据实际业务来选择，比如就是做简单的缓存，并且不存在什么超级热点缓存，那么丢失30s也不是什么大事，这时候如果追求性能的机制，可以选择方案3

方案1说实话倒是很少有场景会使用，因为Redis本身是无法做到完全不丢失数据，Redis的定位就不是完全可靠，通常也没必要损耗大量性能去追求立即刷盘

下面，我们来讲讲写入AOF文件的细节，来搞清楚刷盘策略到底如何应用的。

## 3 写入AOF细节

Redis 写入 AOF 日志的过程，如下图：![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/11c0d449c3f9b269d88735e6b1a54a99.png)
`理解1：`
- 执行数据操作命令
- 命令写入到aof_buf缓冲区
- 调用write()方法，与系统进行IO操作，将aof_buf数据写入内核缓冲区
- 调用fsync()方法，与系统进行IO操作，让内核发起写入操作，将内核缓冲区数据写入硬盘
	1. 三种3种刷盘策略，决定了fsync()方法调用情况
	2. always：在Redis进程中，主线程执行fsync()方法
	3. Everysec：在Redis进程中，创建一个子线程异步处理fsync()方法
	4. no：不执行fsync()方法

写入AOF，其实是分了好几步来的。

- `第一步`：其实将数据写入AOF缓存中，这个缓存名字是aof_buf，其实就是一个sds数据![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0972ab142145ac0d3ce3ce508dafeac6.png)
- `第二步`：aof_buf对应数据刷入磁盘缓冲区，什么时候做这个事情呢？
	- 事实上，Redis源码中一共有4个时机，会调用一个叫flushAppendOnlyFile的函数，这个函数会使用write函数来将数据写入操作系统缓冲区：
		1. 处理完事件处理后，等待下一次事件到来之前，也就是beforeSleep中。
		2. 周期函数serverCron中（每隔一段时间就会触发执行），这也是我们打过很多次交道的老朋友了
		3. 服务器退出之前的准备工作时
		4. 通过配置指令关闭AOF功能时

- `第三步`：刷盘，即调用系统的flush函数，刷盘其实还是在flushAppendOnlyFile函数中，是在write之后，但是不一定调用了flushAppendOnlyFile，flush就一定会被调用，这里其实是支持一个刷盘时机的配置，这一步受刷盘策略影响是最深的，如下面代码所示，如果是`appendfsync always策略`，那么就`立即调用redis_fsync刷盘`，如果是`AOF_FSYNC_EVERYSEC策略`，满足条件后会用aof_backgroud_fsync`使用后台线程异步刷盘`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8b386ab37d591015d61c474de048345.png)

讲到这里，我们应该可以很清晰画出一个相对宏观的完整写入示意图：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0b8f7f9d1f1259863fb526e2a94d7327.png)

下面，我们来看一个AOF写入例子，以进一步掌握AOF
## 4 AOF例子

执行如下命令：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d78abe28ad59e939627d397d66ab79f2.png)

对应会生成了如下的AOF文件
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b04a28b4ec9f8da3d32e56f65841f1e6.png)

打开aof文件可以看到：
`commands are logged using the same format as the Redis protocol itself`

`aof记录的方式，和Redis协议本身是一样的`，可以看到这个协议可读性还是比较强的，具体的协议规则，我们也学过了

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8a6daa2d204b5d4a4daf24a41581c41.png)

## 5 AOF重写

AOF是不断写入的，这很容易带来一个疑问，如此下去AOF不是会不断膨胀吗？

针对这个问题，Redis采用了重写的方式来解决：
- Redis可以在AOF文件体积变得过大时，自动地在后台`Fork一个子进程(bgrewriteaof)`，专门对AOF进行重写（_扫描数据库中所有数据，逐一把内存数据的键值对转换成一条命令，再将命令记录到重写日志_）。说白了，就是针对相同Key的操作，进行合并，比如同一个Key的set操作，那就是后面覆盖前面的。
- 在重写的过程中，Redis不但将新的操作记录在`原有的AOF缓冲区`，而且还会记录在`AOF重写缓冲区`。
- 一旦新的AOF文件创建完毕，Redis就会将重写缓冲区内存，`追加`到新的AOF文件，再用新的AOF文件替换原来的AOF文件。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9836f77a8b3e29b64910d89e241eb7d.png)

这里可能会问，AOF达到多大会重写，实际上，这也是配置决定的，默认如下，同时满足这两个条件则重写。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/349e65c78558a45ff001f7666ce6f4cc.png)

也就是说，超过64M的情况下，或相比上次重写时的数据大了1倍，则触发重写，很明显，最后实际上还是在周期函数来检查和触发的。


AOF重写涉及`写时复制`问题：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8aafe39e74f33cd7e90473c73d3eed3a.png)




## 6 补充

- 为什么重写AOF的时候，不直接复用现有的AOF文件，而是先写入到新的AOF文件再覆盖现有的AOF文件呢？
	- 因为**如果 AOF 重写过程中失败了，现有的 AOF 文件就会造成污染**，可能无法用于恢复使用。
	- 所以 AOF 重写过程，先重写到新的 AOF 文件，重写失败的话，就直接删除这个文件就好，不会对现有的 AOF 文件造成影响

- 为什么要Fork一个子进程来处理AOF重写呢？
	- 子进程进行 AOF 重写期间，主进程可以继续处理命令请求，从而避免阻塞主进程；
	- 如果是使用线程，多线程之间会共享内存，那么在修改共享内存数据的时候，需要通过加锁来保证数据的安全，而这样就会降低性能
	- 用子进程，创建子进程时，`父子进程是共享内存数据的(页表)，不过这个共享的内存只能以只读的方式`，而当父子进程任意一方修改了该共享内存，就会发生「写时复制」，于是父子进程就有了独立的数据副本，就不用加锁来保证数据安全

- 子进程是怎么拥有主进程一样的数据副本的呢？
	- 主进程在通过 `fork` 系统调用生成 bgrewriteaof 子进程时，操作系统会把主进程的「**页表**」复制一份给子进程，这个页表记录着虚拟地址和物理地址映射关系，而不会复制物理内存，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42eee915696d7cac782acce9eb413883.png)
	- 这样一来，子进程就共享了父进程的物理内存数据了，这样能够**节约物理内存资源**，页表对应的页表项的属性会标记该物理内存的权限为**只读**

- 如果父进程需要修改数据（写）怎么办呢？
	- 当父进程或者子进程在向这个内存发起写操作时，CPU 就会触发**写保护中断**，这个写保护中断是由于违反权限导致的，然后操作系统会在「写保护中断处理函数」里进行**物理内存的复制**，并重新设置其内存映射关系，将父子进程的内存读写权限设置为**可读写**，最后才会对内存进行写操作，这个过程被称为「**写时复制(_Copy On Write_)**」![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/97f34814c19d228671dd0e7a7444184f.png)
	- 写时复制顾名思义，**在发生写操作的时候，操作系统才会去复制物理内存**，这样是为了防止 fork 创建子进程时，由于物理内存数据的复制时间过长而导致父进程长时间阻塞的问题。

- 期间会在哪里阻塞父进程
	- 创建子进程的途中，由于要复制父进程的页表等数据结构，阻塞的时间跟`页表的大小有关，页表越大，阻塞的时间也越长`
	- 创建完子进程后，如果子进程或者父进程修改了共享数据，就会`发生写时复制，这期间会拷贝物理内存，如果内存越大，自然阻塞的时间也越长`；

## 7 总结

AOF是通过记录日志来进行持久化，Redis提供了三种刷盘策略以应对不同的性能要求，分别是每次刷、每秒刷、不主动刷，其中Redis推荐每秒刷的模式。

针对AOF文件不断膨胀的问题，Redis还提供了重写机制，针对相同的Key的操作进行合并，来减少AOF的空间，下一届，我们也会继续讨论重写场景的一些优化。