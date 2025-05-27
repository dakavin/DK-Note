---
文章标题: "[[3_基本的SELECT语句]]" 
文章作者: Dakkk
文章概要: |
  介绍SQL语言的基础知识包括SQL概述、语法规则规范、SELECT语句的基本用法、列别名、去重、空值处理、表结构查看和数据过滤等核心操作
tags:
- "SQL"
- "SELECT语句"
- "数据库查询"
- "MySQL"
- "数据过滤"
- "WHERE子句"
- "列别名"
- "去重操作"
相关文章:
- "[[3_Select的执行过程]]"
- "[[1_理解NULL]]"
- "[[6_多表查询]]"
- "[[0_手撕SQL]]"
- "[[1_📕博客设置模块功能分析、表设计]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/01-🐬 MySQL/01-📚 基础语法/3_基本的SELECT语句.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

## 1 SQL的概述

### 1.1 SQL的背景知识

- 1974 年，IBM 研究员发布了一篇揭开数据库技术的论文《SEQUEL：一门结构化的英语查询语言》，直到今天这门结构化的查询语言并没有太大的变化，相比于其他语言，`SQL 的半衰期可以说是非常长`了

- SQL（Structured QUERY Language，结构化查询语言）是使用关系模型的数据库应用语言， `与数据直接打交道`，由`IBM`上世纪70年代开发出来。后由美国国家标准局（ANSI）开始着手制定SQL标准，先后有`SQL-86`，`SQL-89`，`SQL-92`，`SQL-99`等标准。

- `SQL 有两个重要的标准，分别是 SQL92 和 SQL99`，它们分别代表了 92 年和 99 年颁布的 SQL 标准，我们今天使用的 SQL 语言依然遵循这些标准。

### 1.2 SQL的父类

^17b4a1

- `DDL(Data Definition Languages):数据定义语言。` CREATE \ ALTER \ DROP \ RENAME \ TRUNCATE

- `DML(Data Manipulation Languages):数据操作语言。` INSERT \ DELETE \ UPDATE \ SELECT(重中之重)
	- ***SELECT是SQL语言的基础，最为重要。***

- `DCL(Data Control Languages):数据控制语言。` COMMIT \ ROLLBACK \ SAVEPOINT \ GRANT \ REVOKE

- 学习技巧：大局出发，落脚细节

>因为查询语句使用的非常的频繁，所以很多人把查询语句单拎出来一类：DQL（数据查询语言）。还有单独将COMMIT 、ROLLBACK 取出来称为TCL （Transaction Control Language，事务控制语言）。

## 2 SQL的规则与规范

### 2.1 SQL的规则 

- `必须要遵守的`
- SQL可以写在一行或者多行。为了提高可读性，各字句分行写，必要时使用缩进
- 每条命令以 ; 或 \g 或 \G 结束
- 关键字不能被缩写也不能分行
	- 必须保证所有的()、单引号、双引号是成对结束的
	- 必须使用英文状态下的半角输入方式
	- 字符串型和日期时间类型的数据可以使用单引号(' ')表示
	- 列的别名，尽量使用双引号(" ")，而且不建议省略as

### 2.2 SQL大小写规范

- MySQL在Windows环境下是大小写不敏感的
- MySQL在Linux环境下是大小写敏感的
	- 数据库名、表名、表的别名、变量名是严格区分大小写的
	- 关键字、函数名、列名（或字段名）是忽略大小写的
- 推荐采用统一的书写规范
	- 数据库名、表名、表别名、字段名、字段别名等都小写
	- SQL关键字、函数名、绑定变量等都大写


### 2.3 注释

- 可以使用如下格式的注释结构
```mysql
单行注释：#注释文字（MySQL特有的方式）
单行注释：-- 注释文字（--后面必须包含一个空格）
多行注释：/* 注释文字 *
```


### 2.4 命名规则（了解）


- 数据库、表名不得超过30个字符，变量名限制为29个
- 必须只能包含 A–Z, a–z, 0–9, _共63个字符
- 数据库名、表名、字段名等对象名中间不要包含空格
- 同一个MySQL软件中，数据库不能同名；同一个库中，表不能重名；同一个表中，字段不能重名
- 必须保证你的字段没有和保留字、数据库系统或常用方法冲突。如果坚持使用，请在SQL语句中使用`（着重号）引起来
- 保持字段名和类型的一致性，在命名字段并为其指定数据类型的时候一定要保证一致性。假如数据类型在一个表里是整数，那在另一个表里可就别变成字符型了


- 案例
```mysql
use dbtest1;

-- 这是一个查询语句
select * from employees;

insert into employees
values(1001,'Jerry');	#字符串、日期时间类型的变量需要使用一对''表示

# SELECT * FROM employees\G  #只能在命令行中使用

show create table employees\g
```

### 2.5 数据导入指令

- 方式1：在cmd命令行中登录MySQL账户，然后source 文件的全路径名
	- 举例：source d:\xxx.sql
	
- 方式2：基于具体的图形化界面的工具可以导入数据
	- 如：SQLyog中 选择 工具 -- 执行sql脚本 -- 选择xxx.sql即可


## 3 基本的SELECT语句

### 3.1 SELECT...
```mysql
SELECT 1;	#没有任何字句
SELECT 9/2;	#没有任何字句
```

### 3.2 SELECT...FROM...
- 语法
```mysql
SELECT	#标识选择那些列
FROM	#标识从那个表中选择
```

- 选择全部列
```mysql
SELECT *
FROM departments;
```

- 说明：
	- 一般情况下，除非需要使用表中所有的字段数据，最好不要使用 * 
	- 使用通配符虽然可以节省输入查询语句的时间，但是获取不需要的列数据通常会降低查询和所使用得应用程序的效率
	- 通配符的优势是，当不知道所需要的列名时，可以通过它直接获取
	- 所以在开发环境下，不建议直接使用`SELECT *`进行查询

- 选择特定的列
```mysql
SELECT department_id,location_id
FROM departments;
```

- 说明
	- MySQL中的SQL语句是不区分大小写的，因此SELECT和select的作用是相同的
	- **许多开发人员习惯将关键字大写、数据列和表名小写**
	- 建议按照这个编程习惯编写，便于代码阅读和维护

### 3.3 列的别名

- 重命名一个列
- 便于计算
- 紧跟列名，也可以在`列名和别名之间假如关键字AS，别名使用双引号`，以便在别名中包含空格或特殊的字符并区分大小写
- AS可以省略
- 建议别名简短，见名知意
- 举例
```mysql
SELECT last_name AS "name",commission_pct comm
FROM employees;

SELECT last_name "Name",salary*12 "Annual Salary"
FROM employees;
```

### 3.4 去除重复行

- 默认情况下，查询会返回全部行，包括重复行
```mysql
SELECT department_id
FROM employees;
```

- `在SELECT语句中使用关键字DISTINCT去除重复行`
```mysql
SELECT DISTINCT department_id
FROM employees;
```
- 注意：
	- DISTINCT使用后，会导致数据剩下几行，若此时在其前面加入其他没有此关键字修饰的列名，则会导致报错
	- 在其后假如其他的关键字，会将其后面列名中不重复的显示出来
	

### 3.5 空值参与运算

- 所有运算符或列值遇到null值，运算结果都为null
```mysql
SELECT employee_id,salary,commission_pct,
12 * salary * (1 + commission_pct) "annual_sal"
FROM employees;

SELECT employee_id,salary,commission_pct,
12 * salary * (1 + IFNULL(commission_pct,0)) "annual_sal"
FROM employees;
```

- 注意，在mysql里面，空值不等于空字符串。一个空字符串的长度是0，而一个空值长度是空。在MySQL中，空值是占用空间的


### 3.6 着重号

```mysql
#错误的案例
SELECT * FROM order;
#正确的案例
SELECT * FROM `ORDER`;
```
- 说明：我们需要保证表中的字段、表名等没有和保留字、数据库系统或常用方法冲突。如果真的相同，请在SQL语句中使用一对``（着重号）引起来


### 3.7 查询常数

- SELECT 查询还可以对常数进行查询。对的，就是在 SELECT 查询结果中增加一列固定的常数列。这列的取值是我们指定的，而不是从数据表中动态取出的。
- SQL 中的 SELECT 语法的确提供了这个功能，一般来说我们只从一个表中查询数据，通常不需要增加一个固定的常数列，但如果我们想整合不同的数据源，用常数列作为这个表的标记，就需要查询常数。
- 比如说，我们想对 employees 数据表中的员工姓名进行查询，同时增加一列字段corporation ，这个字段固定值为“尚硅谷”，可以这样写

```mysql
SELECT '你好呀' AS corporation,last_name FROM employees;
```

## 4 显示表结构

- 使用 DESCRIBE 或 DESC 命令，表示表结构
```mysql
DESCRIBE employees;
#或
DESC employees;
```

- 各个字段的含义分别解释如下
	- Field   : 表示字段名称  
	- Type    : 表示字段类型
	- Null    : 表示该列是否可以存储NULL值
	- Key     : 表示该列是否已编制索引。PRI表示该列是表主键的一部分；UNI表示该列是UNIQUE索引的一部分；MULbison该列中某个规定值允许出现多次
	- Default : 表示该列是否有默认值，如果有，那么值是多少
	- Extra   : 表示可以获取的与给定列有关的附加信息，例如AUTO_INCREMENT等

## 5 过滤数据

- 语法
```mysql
SELECT 字段1，字段2
FROM 表名
WHERE 过滤条件
```

- 使用WHERE字句，将不满足条件的行过滤掉
- WHERE字句紧随FROM字句

- 举例
```mysql
SELECT employee_id,last_name,job_id,department_id
FROM employees
WHERE department_id = 90;
```


