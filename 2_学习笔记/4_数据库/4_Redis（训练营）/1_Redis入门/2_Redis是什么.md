
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