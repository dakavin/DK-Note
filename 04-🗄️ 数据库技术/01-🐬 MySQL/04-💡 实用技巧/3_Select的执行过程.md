---
文章标题: "[[3_Select的执行过程]]" 
文章作者: Dakkk
文章概要: |
  详细解析MySQL中SELECT语句的SQL99语法结构和执行顺序，从FROM/JOIN到LIMIT的完整执行流程及虚拟表生成原理
tags:
- "MySQL"
- "SELECT语句"
- "SQL执行顺序"
- "数据库查询"
- "SQL99语法"
- "虚拟表"
- "查询优化"
- "数据库原理"
相关文章:
- "[[1_📕博客设置模块功能分析、表设计]]"
- "[[1_Part1]]"
- "[[2_📕标签管理：接口开发]]"
- "[[2_安装MySQL]]"
- "[[3_缓存一致性面试与分析]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/01-🐬 MySQL/04-💡 实用技巧/3_Select的执行过程.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

## 1 查询的结构

- SQL99语法
```mysql
SELECT 字段1、字段2、... 、(存在聚合函数)
FROM 表1 (LEFT / RIGHT)JOIN 表2
ON 多表的连接条件
WHERE 过滤条件（不包含聚合函数）
GROUP BY 分组操作
HAVING 包含聚合函数的过滤条件
ORDER BY 按照ASC/DESC排序
LIMIT分页
```

## 2 SELECT 执行顺序

- <font color="#00b050">FROM ... JOIN</font> $\Longrightarrow$ <font color="#00b050">ON</font> $\Longrightarrow$ <font color="#00b050">WHERE</font> $\Longrightarrow$ <font color="#00b050">GROUP BY</font> $\Longrightarrow$ <font color="#00b050">HAVING</font> $\Longrightarrow$
- <font color="#00b050">SELECT</font> $\Longrightarrow$ <font color="#00b050">DISTINCT</font> $\Longrightarrow$ 
- <font color="#00b050">ORDER BY</font> $\Longrightarrow$ <font color="#00b050">LIMIT</font>

<span style="background:#d4b106">1. FROM开始，HAVING结束</span>
	- FROM ... JOIN ... ：合并多张表，笛卡尔积的形式的一个虚拟表
	- ON ：多表的连接条件，对上述虚拟表进行过滤（注意外连接）
	- WHERE ： 再次过滤数据
	- GROUP BY ：按相应字段进行分组
	- HAVING ：过滤存在聚合函数的条件
<span style="background:#d4b106">2. SELECT开始并结束</span> 
	- SELECT ：过滤不需要的字段（对列进行的过滤）
	- DISTINCT ： 过滤掉重复的行（即每列的值相同的话，就过滤）
<span style="background:#d4b106">3. ORDER BY开始，LIMIT结束</span>
	- ORDER BY : 按某个字段进行升序或降序操作
	- LIMIT ：对数据进行分页处理

## 3 SQL的执行原理

- 注意上述执行过程中，每个细分的过程<span style="background:#affad1">都会生成一个虚拟表</span>即可