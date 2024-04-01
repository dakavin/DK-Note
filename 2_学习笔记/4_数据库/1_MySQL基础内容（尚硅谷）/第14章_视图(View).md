
## 1. 常见的数据库对象

|对象|描述|
|--|--|
|表(TABLE)| 表是存储数据的逻辑单元，以行和列的形式存在，列就是字段，行就是记录
|数据字典|就是系统表，存放数据库相关信息的表。系统表的数据通常由数据库系统维护，`程序员通常不应该修改，只可查看`|
|约束(CONSTRAINT)|执行数据校验的规则，用于保证数据完整性的规则|
|视图(VIEW) |一个或者多个数据表里的数据的逻辑显示，`视图并不存储数据`|
|索引(INDEX) |用于`提高查询性能`，相当于书的目录|
|存储过程(PROCEDURE)|用于完成一次完整的业务处理，没有返回值，但可通过传出参数将多个值传给调用环境|
|存储函数(FUNCTION)|用于完成一次特定的计算，具有一个返回值|
|触发器(TRIGGER)|相当于一个`事件监听器`，当数据库发生特定事件后，触发器被触发，完成相应的处理|

## 2. 视图概述

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7d401bb25c9a47f14c03cedc22ec15f.jpeg)



### 2.1 为什么使用视图？

- 视图一方面可以帮我们使用表的一部分而不是所有的表，另一方面也可以针对不同的用户制定不同的查询视图。

- 比如，针对一个公司的销售人员，我们只想给他看部分数据，而某些特殊的数据，比如采购的价格，则不会提供给他。再比如，人员薪酬是个敏感的字段，那么只给某个级别以上的人员开放，其他人的查询视图中则不提供这个字段。

- 刚才讲的只是视图的一个使用场景，实际上视图还有很多作用。最后，我们总结视图的优点。

### 2.2 视图的理解

- 总结：
	- 视图可以看做是一个`虚拟表`，本身是`不存储数据`的。所以视图的本质，就是一个`存储起来的SELECT语句`
	- 视图中SELECT语句中涉及到的表，称为`基表`
	- 针对视图进行[[第03章_基本的SELECT语句#1.2 SQL的父类|DML操作]]，会影响到对应的基表中的数据。反之亦然
	- 视图本身的删除，不会导致基表中数据的删除
	- 视图的应用场景：
		- <font color="#d83931">小型项目的数据库可以不使用视图</font>
		- <font color="#d83931">在大型项目中</font>，以及数据表比较复杂的情况下，<font color="#d83931">视图的价值就凸显出来了</font>，它可以帮助我们把经常查询的结果集放到虚拟表中，提升使用效率。理解和使用起来都非常方便。
	- 视图的优点：简化查询、控制数据的访问

## 3. 创建视图

- `在 CREATE VIEW 语句中嵌入子查询`
```mysql
CREATE [OR REPLACE]
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION]


CREATE VIEW 视图名称
AS 查询语句
```

### 3.1 创建单视图

```mysql

CREATE VIEW vu_emp1
AS
SELECT employee_id , last_name , salary
FROM emps

SELECT * FROM vu_emp1

CREATE VIEW vu_emp2
AS
#查询语句中字段的别名会作为视图的字段名
SELECT employee_id emp_id , last_name lname, salary
FROM emps
WHERE salary > 8000

SELECT * FROM vu_emp2

#确定视图中字段名的方式2
CREATE VIEW vu_emp3(emp_id , NAME ,monthly_sal)
AS
SELECT employee_id emp_id , last_name lname, salary
FROM emps
WHERE salary > 8000

SELECT * FROM vu_emp3

#视图中存在表中没有的数据
CREATE VIEW vu_emp_sal
AS
SELECT department_id , AVG(salary) avg_sal
FROM emps
WHERE department_id IS NOT NULL
GROUP BY department_id

SELECT * FROM vu_emp_sal
```

### 3.2 创建多表联合视图

```mysql
CREATE VIEW vu_emp_dept
AS
SELECT e.employee_id ,e.department_id , d.department_name
FROM emps e JOIN depts d
ON e.department_id = d.department_id

SELECT * FROM vu_emp_dept
```

- `利用视图对数据进行格式化`
	- 我们经常需要输出某个格式的内容，比如我们想输出员工姓名和对应的部门名，对应格式为emp_name(department_name)，就可以使用视图来完成数据格式化的操作：
```mysql
CREATE VIEW vu_emp_dept1
AS 
SELECT CONCAT(e.employee_id ,' (', d.department_name,')') emp_info
FROM emps e JOIN depts d
ON e.department_id = d.department_id

SELECT * FROM vu_emp_dept1
```

### 3.3 基于视图创建视图

```mysql
CREATE VIEW vu_emp4
AS
SELECT employee_id ,last_name
FROM vu_emp1

SELECT * FROM vu_emp4
```

## 4. 查看视图

- 语法1：查看数据库的表对象、视图对象
```mysql
SHOW TABLES;
```

- 语法2：查看视图的结构
```mysql
DESC / DESCRIBE 视图名称;
```

- 语法3：查看视图的属性信息
```mysql
# 查看视图信息（显示数据表的存储引擎、版本、数据行数和数据大小等）
SHOW TABLE STATUS LIKE '视图名称'\G

#执行结果显示，注释Comment为VIEW，说明该表为视图，其他的信息为NULL，说明这是一个虚表。
```


- 语法4：查看视图的详细定义信息
```mysql
SHOW CREATE VIEW 视图名称;
```

## 5. 更新视图的数据

### 5.1 一般情况

- MySQL支持使用INSERT、UPDATE和DELETE语句对视图中的数据进行插入、更新和删除操作。当视图中的数据发生变化时，数据表中的数据也会发生变化，反之亦然。

```mysql
# 更新视图上的数据会影响基表的数据
UPDATE vu_emp1
SET salary = 20000
WHERE employee_id = 101

SELECT * FROM vu_emp1

SELECT employee_id , last_name , salary
 FROM emps
 
 #更新基表的数据会影响视图上的数据
 UPDATE emps
 SET salary = 10000
 WHERE employee_id = 101
 
 #DELETE、INSERT 操作同理
```

### 5.2 不可更新的视图

- 要使视图可更新，视图中的行和底层基本表中的行之间`必须存在一对一的关系`。另外当视图定义出现如下情况时，视图不支持更新操作：
	- 定义视图的SELECT语句中使用了 JOIN联合查询 ，数学表达式 ， 子查询 ， DISTINCT ， 聚合函数 ，GROUP BY ， HAVING ， UNION 等操作

>虽然可以更新视图数据，但总的来说，视图作为虚拟表，主要用于方便查询，`不建议更新视图的数据`。对视图数据的更改，都是通过对实际数据表里数据的操作来完成的。

## 6. 修改、删除视图

### 6.1 修改视图

- 方式1：使用CREATE `OR REPLACE` VIEW 子句`修改视图`
```mysql
CREATE OR REPLACE VIEW vu_emp1
AS
SELECT employee_id , last_name , salary , email
FROM emps
WHERE salary > 7000
```

>说明：CREATE VIEW 子句中各列的别名应和子查询中各列相对应。

- 方式2：ALTER VIEW
```mysql
ALTER VIEW 视图名称
AS
查询语句

ALTER VIEW vu_emp1
AS 
SELECT employee_id , last_name , salary , email , hire_date
FROM emps
WHERE salary > 7000
```

### 6.2 删除视图

- 删除视图只是删除视图的定义，并不会删除基表的数据。
```mysql
DROP VIEW IF EXISTS 视图名称;

DROP VIEW IF EXISTS vu_emp4 , vu_emp3

SHOW TABLES
```

- 说明：基于视图a、b创建了新的视图c，如果将视图a或者视图b删除，会导致视图c的查询失败。这样的视图c需要手动删除或修改，否则影响使用。

## 7. 总结

### 7.1 视图优点

`1. 操作简单(简化查询)`

- 将经常使用的查询操作定义为视图，可以使开发人员不需要关心视图对应的数据表的结构、表与表之间的关联关系，也不需要关心数据表之间的业务逻辑和查询条件，而只需要简单地操作视图即可，极大<font color="#d83931">简化了开发人员对数据库的操作</font>。

`2. 减少数据冗余`

- 视图跟实际数据表不一样，它存储的是查询语句。所以，在使用的时候，我们要通过定义视图的查询语句来获取结果集。而<font color="#d83931">视图本身不存储数据，不占用数据存储的资源，减少了数据冗余</font>。

`3. 数据安全`

- MySQL将用户对数据的访问限制在某些数据的结果集上，而这些数据的结果集可以使用视图来实现。用户不必直接查询或操作数据表。这也可以理解为<font color="#d83931">视图具有隔离性</font>。视图相当于在用户和实际的数据表之间加了一层虚拟表。同时，MySQL可以根据权限将用户对数据的访问限制在某些视图上，<font color="#d83931">用户不需要查询数据表，可以直接通过视图获取数据表中的信息</font>。这在一定程度上保障了数据表中数据的安全性。

`4. 适应灵活多变的需求` 

- 当业务系统的需求发生变化后，如果需要改动数据表的结构，则工作量相对较大，可以<font color="#d83931">使用视图来减少改动的工作量</font>。这种方式在实际工作中使用得比较多。

`5. 能够分解复杂的查询逻辑` 

- 数据库中如果存在复杂的查询逻辑，则可以将<font color="#d83931">问题进行分解，创建多个视图</font>获取数据，再将创建的多个视图结合起来，完成复杂的查询逻辑。

### 7.2 视图不足

- 如果我们在实际数据表的基础上创建了视图，那么，`如果实际数据表的结构变更了，我们就需要及时对相关的视图进行相应的维护`。特别是嵌套的视图（就是在视图的基础上创建视图），维护会变得比较复杂， 可读性不好，容易变成系统的潜在隐患。因为创建视图的 SQL 查询可能会对字段重命名，也可能包含复杂的逻辑，这些都会增加维护的成本。

- 实际项目中，如果视图过多，会导致数据库维护成本的问题。

- 所以，在创建视图的时候，你要结合实际项目需求，综合考虑视图的优点和不足，这样才能正确使用视图，使系统整体达到最优。