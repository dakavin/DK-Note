
## 一．MySQL产品的介绍

MySQL是最受欢迎的开放源代码数据库，其他（Oracle、SQLserver等）相比，Mysql体积小，速度快，成本低。

## 二．数据库的好处

1.持久化数据到本地  2.便于查询和管理

## 三．MySQL服务的登录和退出

- 方式一：通过mysql自带的客户端 只限于root用户

- 方式二：通过cmd(需进入mysql安装目录或配置环境变量)
	- 登录：mysql 【-h主机名 -P端口号 】-u用户名 -p密码
	- 退出：exit

- 方式三：可视化图形工具（SQLyog或Navicat）


## 四．数据库存储数据的特点

1、将数据放到表中，表再放到库中

2、一个数据库中可以有多个表，每个表都有一个的名字，用来标识自己。`表名具有唯一性`

3、表具有一些特性，这些特性定义了数据在表中如何存储，`类似java中 “类”的设计`

4、表由列组成，我们也称为`字段`。所有表都是由一个或多个列组成的，`每一列类似java 中的“属性”`

5、表中的数据是按行存储的，`每一行类似于java中的“对象”`

## 五．MySQL的常见命令

1. 查看当前所有的数据库

```mysql
SHOW DATABASES;
```

2. 创建数据库

```mysql
CREATE DATABASE [IF NOT EXISTS] 库名 [CHARACTER SET charset_name]

# mysql8.0默认是utf8mb4
# most by 4bytes
```

3. 删除数据库

```mysql
DROP DATABASE [IF EXISTS] 库名；
```

4. 打开指定的库

```mysql
USE 库名；
```

5. 查看当前库的所有表

```mysql
SHOW TABLES；
```

6. 查看其它库的所有表

```mysql
SHOW TABLES FROM 库名；
```

7. 创建表

```mysql
#方式1：
CREATE TABLE 表名 VALUES(
	字段名 字段类型 约束，
	字段名 字段类型 约束，
	......
	）

#方式2：
CREATE TABLE 表名 AS
SELECT * FROM 其他表；
)
```

8. 查看表结构

```mysql
DESC 表名；
```

9. 查看所有数据;

```mysql
SELECT * FROM 表名；
```

10. 查看mysql的版本

```mysql
SELECT version()
```

11. 增加数据

```mysql
INSERT INTO 表名 VALUES（值列表1），（值列表2）......

INSERT INTO 表名 SET 字段名 = 值 ，字段名 = 值 ，......
```

12. 查看部分数据

```mysql
SELECT 字段列表 FROM 表名 WHERE ...
```

13. 删除数据

```mysql
DELETE * FROM 表名 WHERE ...

#若没有WHERE，则删除所有内容
```

14. 更新数据

```mysql
UPDATE 表名 SET 字段 = 某个值 WHERE

#若没有WHERE，则会更新全部数据
```

## 六．MySQL的语法规范

1.不区分大小写,但建议关键字大写，表名、列名小写

2.每条命令最好用分号结尾

3.注释 单行注释：#注释文字 单行注释：-- 注释文字 多行注释：/* 注释文字 */

## 七．数据类型

1.数值类型

Int 、double、float、decimal

2.字符串类型

char、varchar、blob、text、binary、varbinary

3.日期类型

datetime、time、timestamp

……