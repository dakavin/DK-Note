
### 查询的结构

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

### SELECT 执行顺序

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

### 4.3 SQL的执行原理

- 注意上述执行过程中，每个细分的过程<span style="background:#affad1">都会生成一个虚拟表</span>即可