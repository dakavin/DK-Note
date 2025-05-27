---
文章标题: "[[2_Redis是什么]]" 
文章作者: Dakkk
文章概要: |
  Redis是开源内存数据结构存储系统，支持多种数据类型，可用作数据库、缓存、消息代理，具备复制、持久化、集群等企业级特性
tags:
- "Redis"
- "内存数据库"
- "缓存"
- "NoSQL"
- "数据结构"
- "键值存储"
- "分布式"
- "持久化"
相关文章:
- "[[0_Redis对象]]"
- "[[2_Redis单线程面试与分析]]"
- "[[1_持久化介绍]]"
- "[[1_String]]"
- "[[5_Hash]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/02-🔴 Redis/03-🎓 训练营笔记/1_Redis入门/2_Redis是什么.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

我们来看官方定义：
> Redis is an open source(BSD licensed), in-memory data structure store used as a database, cache, ==message broker(消息代理)==, and ==streaming engine（流式计算引擎）==. 
> Redis provides data structures such as strings, hashes, sets, ==sorted sets with range queries（带范围的查询的排序集合）==, ==bitmaps（位图）==, ==hyperloglogs（超日志）==, ==geospatial indexes（地理空间索引）==, and streams.
> Redis has ==built-in（内置的）== ==replication（复制）==, Lua scripting, LRU eviction, tansactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster


- `核心`
	- Redis是一个内存存储
	- Redis是一个Key—Value存储

- `解决什么问题`
	- 常用于缓存、消息中转、数据流引擎(?)

- `支持什么数据结构`
	- strings
	- hashes
	- lists
	- sets
	- sorted sets with range queries
	- bitmaps
	- hyperloglogs
	- geospatial indexes
	- stream
	- 等一系列数据结构

- `有什么特性`
	- 内置复制
	- Lua脚本
	- LRU驱逐
	- 事务
	- 持久化
	- 哨兵和集群