[为什么单线程的 Redis 如何做到每秒数万 QPS ？ (qq.com)](https://mp.weixin.qq.com/s/oeOfsgF-9IOoT5eQt5ieyw)
## 1 概述

上一节，我们整体介绍了Redis的处理过程，为了加深理解，本节我们通过源码的形式，进行深入分析，大家注意一下，面试时候一般不会直接讨论源码的，但是通过学习源码，可以帮助我们更好的理解，如果基础比较薄弱，也可以先跳过本节，不影响后续学习，等二刷的时候再看看
## 2 Redis的AE库

Redis实现了一个AE库，它是"A simple Event drived programming library"的简写，翻译过来就是`一个简单的事件驱动库`，它会根据系统的内核版本自动选择select、epoll、kqueue等不同多路复用的方式。

由于生产环境通常在linux上面，所以下面我们以常见的epoll为例，以下代码片段均来自redis 5.0.5
## 3 处理流程

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e97373bfcb2c2a45f6c4a10af148064c.png)

在整个Redis的工作过程中，只需要理解main函数中调用的 `initServer` 和 `aeMain` 这两个函数就足够了，事件处理循环aeMain函数，我们在下一节讲解

我们先看 initServer这个函数内，Redis做了什么事情：
- 创建一个 epoll 对象
- 对配置的监听端口进行 listen
- 把 listen socket 让 epoll 给管理起来

再来看aeMain函数做了哪些事情：
- 通过 epoll_wait 发现 listen socket 以及其它连接上的可读、可写事件
- 若发现 listen socket 上有新连接到达，则接收新连接，并追加到 epoll 中进行管理
- 若发现其它 socket 上有命令请求到达，则读取和处理命令，把命令结果写到缓存中，加入写任务队列
- 每一次进入 epoll_wait 前都调用 beforesleep 来将写任务队列中的数据实际进行发送
- 如若有首次未发送完毕的，当写事件发生时继续发送
### 3.1 创建epoll对象

- 调用aeCreateEventLoop方法，创建epoll对象

### 3.2 绑定监听服务 端口

- 调用 listenToPort函数，内部循环调用anetTcpServer函数
- 在anetTcpServer函数中，一直执行到bind和listen系统调用
### 3.3 监听端口，注册事件

- 如果在上述的listenToPort函数，监听到了服务端口，产生监听套接字
- 调用acCreateFileEvent函数，为该监听套接字绑定接收处理函数acceptTcpHandler
- 开始一个事件循环，通过epoll_wait的方式等待连接到达（连接套接字）
	- 这里解决了accept阻塞的问题

main -> initServer中，使用listenToPort监听端口，创建监听套接字
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5e00bc3d7daf5b439511d9f6f9e04a76.png)

接下来，还是在`initServer函数`中，会调用`aeCreateFileEvent函数`绑定对应的接收处理函数acceptTTcpHandler，绑定之后等到客户端连接请求发回来，就可以关联到这个处理函数。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/83cf639ca91c987bd87df251a5819b1b.png)

`main函数调用initServer之后，就开始在aeMain函数中开始一个EvenLoop事件循环`，通过`epoll_wait()的方式等待连接达到`，此时暂时还没有建立的连接，所以没有读写事件的发生，另外计时器等事件也是注册在事件框架中，但这里我们主要还是关注I/O事件。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/95b0459f2f033b26740e374b889d81dc.png)

### 3.4 连接到达处理

- `此时进入的是aeMain函数，处理用户请求`

- epoll_wait等待到连接请求的到来，当连接请求到来后
- 回调该监听套接字绑定的接收处理函数acceptTcpHandler
	1. 将监听套接字变为已连接(客户端)套接字，为已连接套接字绑定上readQueryFromClient处理函数
	2. 为已连接套接字注册一个可读事件，放入epoll_wait继续事件循环
		- 系统内核去通知我们该fd已经就绪了，这样就解决了recv阻塞等待的问题

当一个连接到达，事件循环就能收到这个信息，在监听套接字关联的`acceptTcpHandler函数`这时候就开始行动了，先生成一个客户端套接字，但这里也不是马上开始读取客户端发来的数据，因为也没办法确定此时一定能无阻塞地读取到完整的请求包，因此`为它注册一个可读事件，仍然放到如上所述的同一个epoll实例中`，让内核去通知我们该fd已经就绪了，这样就解决了recv阻塞等待的问题。其源码是从在回调函数中注册回调函数。

acceptTcpHandler -> accpetCommonHandler -> createClient 

createClient中关键片段：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2d3b4ece53a5ec338c261a9bc0d67df.png)

可以看到，`这里关键信息是创建了一个客户端对象`，然后将客户端套接字设置为非阻塞，然后加入到事件循环里，这里的关联函数是readQueryFromClient，这里大家应该已经能理解，等这个客户端套接字的读请求过来，就会关联到readQueryFromClient来处理

### 3.5 客户端数据处理

- epoll_wait发现已连接(客户端)套接字中传来数据
- 回调该已连接套接字绑定的readQueryFromClient处理函数
	- 解析并查找命令
	- 调用命令处理
	- 添加写任务到队列
		- prepareClientToWrite 判断是否需要返回数据，并且将当前 client 添加到等待写返回数据队列中
	- 将输出写到缓存等待发送
		- 调用 `_addReplyToBuffer` 和 `_addReplyObjectToList` 方法将返回值写入到输出缓冲区中，等待写入 socekt

收到一个客户端发来的数据后，readQueryFromClient就开始行动了，这里我们聚焦关注命令处理，忽略其他细节。

readQueryFromClient -> processInputBuffer -> processCommand，`processInputBuffer会解析命令`，`实际执行命令是在processCommand中`：

第一步：使用lookupCommand，找到对应命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57b7f7bbef128aef37fa9e662a9eff96.png)
第二步：做各项检查，比如检查，确认Redis可以执行本条命令，这里不做展开；

第三步：正在执行命令，更改内存数据，下面是说如果客户端启动了事务，则将命令放入队里中，这里我们不过多关注。如果没有开启事务，则调用call方法更改内存。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14e1cb2d3428081fd78a4c795c0dd359.png)
`call里面的事项关键是调用了proc方法，这个方法对应的就是命令执行函数`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88e268f444499ba10e8fa8c82acaa56c.png)

如果看完了这一系列操作，可能会比较奇怪，回包是在哪里进行的呢？实际上，执行完成之后，Redis会将结果放入client结构的输出缓冲区中，这个逻辑是在命令执行函数中，比如GET命令，它的proc其实就是
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/50fd41232bdf1261de1b2d03e50a188a.png)

getGenericCommand里面使用addReply将结果放入client的输出缓冲区。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35cc9c35ba9581096127288264cb1b1f.png)

注意这里也没有直接send()到客户端，那么什么时候触发回包到客户端呢?

### 3.6 给客户端回包

我们`回到上面aeMain主循环`的地方
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e3561326aa3272480118a81c61135c1.png)

`在进入aeProcessEvents前会调用 eventLoop -> beforesleep(eventLoop)，beforeSleep中有handleClientsPendingWrites，就是专门做回包的。`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c3cc598ab17f3077beeebc97591f4809.png)

其具体工作就是将client结构输出缓冲区的内容，发生给对应的客户端socket。

## 4 总结

本次我们分析了Redis AE库，走读了Redis完整的处理逻辑，相信通过本节课，大家对于Redis的处理流程有了一个更深入的理解。

处理模块算是告一段落，下面我们会关注另一个问题，Redis作为一个基于内存处理的组件，如果内存满了怎么办？

## 5 补充：

https://xiaolincoding.com/os/8_network_system/selete_poll_epoll.html#%E6%9C%80%E5%9F%BA%E6%9C%AC%E7%9A%84-socket-%E6%A8%A1%E5%9E%8B

### 5.1 Linux内核下的epoll机制

在 epoll 的系列函数里， epoll_create 用于创建一个 epoll 对象，epoll_ctl 用来给 epoll 对象添加或者删除一个 socket。epoll_wait 就是查看它当前管理的这些 socket 上有没有可读可写事件发生。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d0f44b0ee254685513c3dbe933a0089.png)

当网卡上收到数据包后，Linux 内核进行一系列的处理后把数据放到 socket 的接收队列。然后会检查是否有 epoll 在管理它，如果是则在 epoll 的就绪队列中插入一个元素。epoll_wait 的操作就非常的简单了，就是到 epoll 的就绪队列上来查询有没有事件发生就行了。关于 epoll 这只“牧羊犬”的工作原理参见[深入揭秘 epoll 是如何实现 IO 多路复用的](https://mp.weixin.qq.com/s?__biz=MjM5Njg5NDgwNA==&mid=2247484905&idx=1&sn=a74ed5d7551c4fb80a8abe057405ea5e&scene=21#wechat_redirect) (Javaer 习惯把基于 epoll 的网络开发模型叫做 NIO)  

在基于 epoll 的编程中，和传统的函数调用思路不同的是，我们并不能主动调用某个 API 来处理。因为无法知道我们想要处理的事件啥时候发生。所以只好提前把想要处理的事件的处理函数注册到一个**事件分发器**上去。当事件发生的时候，由这个事件分发器调用回调函数进行处理。这类基于实现注册事件分发器的开发模式也叫 Reactor 模型。