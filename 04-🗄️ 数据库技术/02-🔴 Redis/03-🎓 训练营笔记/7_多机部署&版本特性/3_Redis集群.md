---
文章标题: "[[3_Redis集群]]" 
文章作者: Dakkk
文章概要: |
  介绍Redis集群概念、哈希槽分片机制，详细演示如何部署3节点集群、连接节点、分配槽位、操作集群及添加新节点的完整实操流程。
tags:
- "Redis集群"
- "哈希槽"
- "分片"
- "集群部署"
- "Redis配置"
- "高可用"
- "分布式存储"
- "集群扩容"
相关文章:
- "[[4_Redis哈希槽]]"
- "[[3_Redis多线程面试与分析]]"
- "[[秒杀系统架构设计]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/02-🔴 Redis/03-🎓 训练营笔记/7_多机部署&版本特性/3_Redis集群.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:57
---

## 1 集群是什么

Redis性能很强大，单机能做到写几万，读十万的tps，非常的高，可以说大多数创建都完全能cover。

但是随着互联网的发展，还是有一些场景，需要更高的tps。

比如秒杀，双十一的时候，特别是2018年附近，一到0点，瞬间请求应该是千万级的。这时候，一个Redis就有点孤掌难鸣，怎么办呢，多个Reids合作解决。

`集群就是多个Redis一起对外提供服务，对外看起来就像一个单机服务的解决方案。`
## 2 Redis集群怎么分片

`Redis使用的哈希槽（Hash Slot）的方式来分片`，可以理解为主动配置，划分不同的范围给每个节点。

为了方便理解，我们先看看怎么部署一个集群。
## 3 部署集群

### 3.1 启动3个Redis服务

首先我们建一个目录redisCluster，里面创建3个子目录7000,7001,7002，这些目录里面都copy一份redis.conf配置，在其中修改下面3个字段：
1. cluster-enabled，设置为yes，这样才能成为集群的一份子
2. port，修改为对应数字，比如7000目录port就改为7000
3. dir改为./  表示数据放在当前目录下

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b240ef28bdd4598b4f581987721ce21e.png)

然后7000,7001,7002目录下，分别使用如下命令，启动服务![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d2eec08b5fb007fe1726ec46f589f1fb.png)
执行之后，目录结构如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ec339e9a23925822d7f0df0071c7fd35.png)
这样3个redis服务器就起来了，但是他们还是孤立的节点。我们通过reids-cli -p 7000连接上7000节点，再使用cluster nodes就可以看到只有他孤零零一个节点在自己的集群中![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1bc25965eed30252c53e4d8ace522411.png)
下面，我们需要将他们连接起来，组成一个整体
### 3.2 连接Redis服务，组成集群

使用cluster meet命令，可以连接上其他节点，现在我们在7000上连接7001 和 7002![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1c3cb9784a6f7d5d38ac484716a6baa8.png)
连接完成后，cluster nodes可以看到他们已经联合起来了![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f051db11db629d8ec1a1c6eab243a6da.png)

我们这里还需要做一步，为cluster中每个节点，划分职责，也就是`给他们分配负责的数据区间`，这里Redis视同的是一个叫Hash槽的概念，即将数据分为了多个槽，每个节点负责一些槽，这个Hash槽，我们下一节会重点介绍，这里初步理解即可。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/974439ea94854bdc063ee4b589eaaadc.png)
这三条命令的过程用这张图比较形象：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/833aa9188a8349af14b2e8ba27f72c91.png)
这时候再查看，就能看到每个节点负责那些槽了：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a487cbb2392055f49469e20e158134ec.png)
## 4 操作集群

操作key，若key不在对应的槽(slot)，则让你MOVED![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0cc9341e60fdc6e0f01215fcb8f16a9c.png)
到这一步，我们的集群就算初步可以使用了。
## 5 添加节点

第一步，还是起一个Redis服务，这里还在之前的机器上起，prot 7003

第二步，用meet命令将新节点加入集群![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/116f78c6d6a46cdb60af16c3a4929a4b.png)
此时，新加入的节点还未负责任何槽![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9bfa329611797da82c89ba02c6976dc0.png)
第三步，分配一些槽给这个新节点，命令为`redis-cli --cluster reshard 127.0.0.1:7000`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f2301ab2caa5cbe3f363d22d5021b9a5.png)
于是从7000和7001，分别拿了50个slots给7003，这里分配的就是节点各自按编号最小的50个左右slot，比如7000，就是0-50![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/17d137dd10d8ab7837741352cda1a179.png)
完成之后![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/17e354af7a49903120f3b708988ba05a.png)
## 6 总结

本节我们聚焦于集群是什么，解决什么问题，以及如何搭建一个集群，通过本节，希望大家对集群有个直观的理解，下一节，我们将介绍集群的分片模式
## 7 面试题

1. 集群是什么？
2. 怎么访问Redis集群
3. Redis集群用的是一致性Hash算法吗？