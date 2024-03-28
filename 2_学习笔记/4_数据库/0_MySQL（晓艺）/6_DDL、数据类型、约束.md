
## 一、数据类型

### 1.1 常见的数据类型

- 数值型
- 整型
	- 小数
	- 定点数
	- 浮点数

- 字符型

- 较短的文本：char、varchar

- 较长的文本：text、blob（较长的二进制数据）

- 日期型

### 1.2 整型

- `分类`：


- `特点`：

- 如果不设置无符号还是有符号，默认是有符号，如果想设置无符号，需要添加unsigned关键字

- 有符号int 的范围 从 -2^31 (-2,147,483,648) 到 2^31 - 1 (2,147,483,647) 的整型数据（所有数字）

- 无符号int 的范围 从 0 到 2^32 - 1 (4 294 967 295) 的整型数据（所有数字）

- 如果插入的数值超出了整型的范围,会报out of range异常,插入失败

-   如果不设置长度，会有默认的长度,长度代表了显示的最大宽度，如果不够会用0在左边填充，但必须搭配zerofill使用！示例如下：

```mysql
create table t (t int(4) zerofill);
insert into t set t = 10;
select * from t
```


- `设置无符号和有符号`
```mysql
DROP TABLE IF EXISTS tab_int;
CREATE TABLE tab_int(
     t1 INT(7) ,
     t2 INT(7) UNSIGNED

);

DESC tab_int;
INSERT INTO tab_int VALUES(-123456);
INSERT INTO tab_int VALUES(-123456,-123456);
INSERT INTO tab_int VALUES(2147483648,4294967296);
INSERT INTO tab_int VALUES(123,123);
SELECT * FROM tab_int;
```

### 1.3 小数

#### 1.3.1 浮点型

`float(M,D)`

- 占4个字节，有符号范围：-3.402 823 466 E+38，-1.175 494 351 E-38，无符号范围：0，(1.175 494 351 E-38，3.402 823 466 351 E+38)

`double(M,D)`

- 占8个字节，有符号范围：-1.797 693 134 862 315 7 E+308，-2.225073858507 2014E-308，无符号范围：0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)

#### 1.3.2 定点型

`dec(M，D)`

`decimal(M,D)`

- `特点`：
	- M：整数部位+小数部位
	- D：小数部位
	- 如果超过范围，会报out of range异常,插入失败
	- M和D都可以省略

- `decimal和float、double的区别`
	- 如果是decimal，则M默认为10，D默认为0
	- 如果是float和double，则会根据插入的数值的精度（由计算机硬件和操作系统决定）来决定精度
	- 定点型的精确度较高，如果要求插入数值的精度较高如货币运算等则考虑使用
	- 原因：`float和double在进行运算的时候容易造成精度丢失`（根本原因还是二进制无法准备表示某些小数）

- 测试M和D
```mysql
DROP TABLE tab_float;

CREATE TABLE tab_float(

     f1 FLOAT(5,2),

     f2 DOUBLE(5,2),

     f3 DECIMAL

);

SELECT * FROM tab_float;

DESC tab_float;

INSERT INTO tab_float VALUES(123.4523,123.4523,123.4523);

INSERT INTO tab_float VALUES(123.456,123.456,123.456);

INSERT INTO tab_float VALUES(123.4,123.4,123.4);

INSERT INTO tab_float VALUES(1523.4,1523.4,1523.4);
```


### 1.4 字符型

- `分类`
	- `较短的文本`：
		- char
		- varchar
	
	- `其他`：
		- binary和varbinary用于保存较短的二进制
		- enum用于保存枚举
		- set用于保存集合
	
	- `较长的文本`：
		- text
		- blob(较大的二进制)

- `特点`：

|写法|  M的意思       |  特点        |      空间的耗费    |    效率|
|---|---|---|---|---|
|char|char(M)| 最大的字符数，可以省略，默认为1|固定长度的字符|比较耗费|          |     高|
|varchar| varchar(M)| 最大的字符数，不可以省略|可变长度的字符|比较节省|        低|

- `例子`：char(20)  varchar(50)

- char保存时右侧填充空格以达到指定的长度

- varchar   长度可变

### 1.5 日期型

- `分类`：
	- date只保存日期
	- time 只保存时间
	- year只保存年
	- datetime保存日期+时间
	- timestamp保存日期+时间

- `特点`：

|字节|范围|时区等的影响|
|---|---|---|
|datetime|8|1000——9999|不受|
|timestamp|4|1970-2038|受|

- TIMESTAMP存储时先转换为UTC(世界标准时间)格式进行保存，查询时，再转回当前时区

```mysql
CREATE TABLE tab_date(
     t1 DATETIME,
     t2 TIMESTAMP

);

INSERT INTO tab_date VALUES(NOW(),NOW());
SELECT * FROM tab_date;
SHOW VARIABLES LIKE 'time_zone';
SET time_zone='+9:00';
```

## 二、DDL语言

`数据库定义语言`

### 2.1 库的管理

- 创建、修改、删除

#### 2.1.1 库的创建

/*

语法：

create database  [if not exists]库名;

*/

案例：创建库Books

CREATE DATABASE IF NOT EXISTS books ;

#### 2.1.2 库的修改

更改库的字符集

ALTER DATABASE books CHARACTER SET gbk;

#### 2.1.3 库的删除

DROP DATABASE IF EXISTS books;

### 2.2 表的管理
创建、修改、删除

#### 2.2.1 表的创建 ★

语法：

```mysql
create table 表名(
     列名 列的类型【(长度) 约束】,
     列名 列的类型【(长度) 约束】,
     列名 列的类型【(长度) 约束】,
     ...
     列名 列的类型【(长度) 约束】
)

#案例：创建表Book
CREATE TABLE book(
     id INT,编号
     bName VARCHAR(20),图书名
     price DOUBLE,价格
     authorId  INT,作者编号
     publishDate DATETIME出版日期
);
DESC book;

#案例：创建表author
CREATE TABLE IF NOT EXISTS author(
     id INT,
     au_name VARCHAR(20),
     nation VARCHAR(10)
)
DESC author;
```



#### 2.2.2 表的修改


```mysql
alter table 表名 add|drop|modify|change column 列名 【列类型 约束】;
```

- `修改列名`

ALTER TABLE book CHANGE COLUMN publishdate pubDate TIMESTAMP;

- `修改列的类型或约束`

ALTER TABLE book MODIFY COLUMN pubdate DATETIME;

- `添加新列`

ALTER TABLE author ADD COLUMN annual DOUBLE;

- `删除列`

ALTER TABLE book_author DROP COLUMN  annual;

- `修改表名`

ALTER TABLE author RENAME TO book_author;

DESC book;

#### 2.2.3 表的删除

```mysql
DROP TABLE IF EXISTS book_author;
SHOW TABLES;

通用的写法：
DROP DATABASE IF EXISTS 旧库名;
CREATE DATABASE 新库名;
DROP TABLE IF EXISTS 旧表名;
CREATE TABLE  表名();
```

#### 2.2.4 表的复制

```mysql
INSERT INTO author VALUES
(1,'村上春树','日本'),
(2,'莫言','中国'),
(3,'冯唐','中国'),
(4,'金庸','中国');

SELECT * FROM Author;
SELECT * FROM copy2;

#1.仅仅复制表的结构

CREATE TABLE copy LIKE author;

#2.复制表的结构+数据

CREATE TABLE copy2
SELECT * FROM author;

#只复制部分数据

CREATE TABLE copy3
SELECT id,au_name
FROM author
WHERE nation='中国';

#仅仅复制某些字段

CREATE TABLE copy4
SELECT id,au_name
FROM author
WHERE 0;
```


## 三、约束

### 3.1 主键约束

- `要求主键列的数据唯一，并且不允许为空，一张表的主键约束只能有1个`。

- `单一主键`

```mysql
create table t_user(
        id bigint primary key,
        name varchar(255)

);

insert into t_user(id,name) values (1,'小明');#主键不受影响，任然为0
insert into t_user(id,name) values (1,'小龙');
insert into t_user(name) values ('小龙');
```

- `联合主键`

```mysql
create table t_user(
          id bigint,
        name varchar(255),
        primary key(id,name)

);

#不是复合主键中的所有值都重复，就不算重复

insert into t_user(no,name) values (1,'小明');
insert into t_user(no,name) values (1,'小龙');
select * from t_user;
```


### 3.2 外键约束

- 为两个表的数据建立连接，约束两个表中数据的一致性和完整性。

```mysql
create table t_area(
        area_number int(1) primary key,
        area char(4)
);

create table t_user(
        id bigint,
        name varchar(255),
        a_number int(1),
        foreign key(a_number) references t_area(area_number)

);

insert into t_area values (1,'中国地区'),(0,'海外地区');
insert into t_user(name,a_number) values ('小明',1), ('小龙',1),('Ana',0),('Jan',0),('Fak',0),('小法',1);

select * from t_area;
select * from t_user;
```

操作顺序

删除数据时后删除被外键依赖的表

添加数据时先添加被外键依赖的表

### 3.3 非空约束

`对应的列值不可以为null`

```mysql
create table t_user (
         id bigint,
        name varchar(255) not null
);
insert into t_user(id) values(1);
```

### 3.4 唯一性约束

- unique约束的字段**不能重复**

- 对单个列进行约束

```mysql
create table t_user (
        id bigint unique,
        name varchar(255)
);

insert into t_user(id,name) values (1,'小明');
insert into t_user(id,name) values (1,'小龙');
```

 - 对多个列进行约束

```mysql
create table t_user(
        id bigint,
        name varchar(255),
        password varchar(15),
        unique(id,name)
);

insert into t_user(id,name) values (1,'小明');
insert into t_user(id,name) values (1,'小龙');
select * from t_user;
```

### 3.5 默认值约束

- 字段设置默认值

- 插入数据行时若该字段为空，则该列值使用默认值填充

```mysql
create table t_user(
       id bigint,
       name varchar(255) DEFAULT 'lili'
);
insert into t_user(id) values (1);
```

6、表的字段值自增

- Mysql一般都会设置主键自增

- 插入数据时该字段从1自增

```mysql
create table t_user(
        id bigint primary key auto_increment,
        name varchar(255)
);

insert into t_user(name) values ('a');
insert into t_user(name) values ('b');
insert into t_user(name) values ('c');
insert into t_user(name) values ('d');
insert into t_user(name) values ('e');
select * from t_user;
```

### 3.6 补充：
除了建表时的添加约束，`也可以在建表后进行添加或删除约束`，语法如下，供大家以后用到时查阅。

1.主键约束

添加:alter table  table_name `add` primary key (字段…)
删除:alter table table_name `drop` primary key

2.非空约束

添加:alter  table table_name `modify` 列名 数据类型  not null
删除:alter table table_name `modify` 列名 数据类型 null

3.唯一约束

添加:alter table table_name `add` unique 约束名（字段…）
删除:alter table table_name `drop` key 约束名

4.自动增长

添加:alter table table_name  `modify` 列名 int  auto_increment
删除:alter table table_name `modify` 列名 int 

5.外键约束

添加:alter table table_name `add` constraint 约束名 foreign key(外键列)
references 主键表（主键列）
删除:alter table table_name `drop` foreign key 约束名

6.默认值

添加:alter table table_name alter 列名  `set` default '值'
删除:alter table table_name alter 列名  `drop` default