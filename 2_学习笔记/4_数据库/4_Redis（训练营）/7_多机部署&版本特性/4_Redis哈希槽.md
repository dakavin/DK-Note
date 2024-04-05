
## 1 哈希槽是什么

在上节讲集群的时候，我们有提到，Redis是用哈希槽来管理数据分片。

具体而言，Redis中哈希槽是这样的：
- 一个Redis Cluster包含16384（2^14）个哈希槽，存储在Redis Clutser的`所有Key都可以通过Hash算法关联都某个槽`，而每个节点负责一部分槽，Key算出来再那个槽，就应该去负责这个槽的节点进行交互
## 2 Hash槽在Redis中的结构

代码定义如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a5629ec8c13ae4851b3bcf10ec30cc2a.png)
slots数据表示某个节点的槽关系。可能我们在代码前，会想像节点是记录所负责的槽，新增就加一个元素，减少就减一个元素。

但在Redis这里的实现，实际是用slots表示了整个hash槽空间，也就是表示了16384个槽，这个数据会记录每个槽的状态，如果是1，则表示自己负责的，可能有点抽象，直接上个图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7b2bb793d247bb4b2b4b6a20c66d943c.png)
通过这种位图的形式，就可以在O(1)的时间复杂度下，查询某个槽是否由该节点负责

## 3 储存槽的位置

我们在上一节有提到过，如果客户端请求错了节点，节点还会友情返回你这个数据应该从那个节点获得。

从这里我们能看出`除了记录了自己负责了那些槽，还需要知道某个槽所在位置`

这是怎么实现的呢？

首先，在分配槽之后，节点会向集群中其他节点放送消息，告知它负责了那些槽![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1efefc331ca7601cf4ee671d640583a1.png)

然后节点会维护一个clusterState结构，里面就记录了每个槽，是由那个节点负责的![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/834e82d981ce18f5ba29fe38b7ff8fb6.png)
比如slots[0]就表示0号槽指向了那个节点，也就是0号槽在那个节点上，如果没有挂靠在某个节点，就是NULL。

所以这里slots[ClUSTER_SSLOTS]整体的结构就如下图
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac113f81349342a8983bd86fb395dfeb.png)