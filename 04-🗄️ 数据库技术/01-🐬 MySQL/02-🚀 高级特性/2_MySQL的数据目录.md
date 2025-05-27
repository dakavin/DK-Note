---
文章标题: "[[2_MySQL的数据目录]]" 
文章作者: Dakkk
文章概要: |
  介绍MySQL8数据目录结构，包括数据库文件存放路径、系统数据库功能、InnoDB和MyISAM存储引擎的文件组织方式及表空间概念。
tags:
- "MySQL"
- "数据目录"
- "存储引擎"
- "InnoDB"
- "MyISAM"
- "表空间"
- "文件系统"
- "数据库管理"
相关文章:
- "[[5_存储引擎]]"
- "[[2_从数据页的角度看 B+ 树]]"
- "[[2_MySQL 一行记录是怎么存储的？]]"
- "[[3_用户-权限管理-角色和配置文件]]"
- "[[4_逻辑架构]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/01-🐬 MySQL/02-🚀 高级特性/2_MySQL的数据目录.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

## 1 MySQL8的主要目录结构

使用指令，查看mysql的目录结构：
```shell
find / -name mysql
```

得到如下结果：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48760c102e17a6d6425162b281e13a4f.png)

### 1.1 数据库文件的存放路径

**MySQL数据库文件的存放路径：`/var/lib/mysql/`**
```mysql
mysql> show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.04 sec)
```

从结果中，可以看出，在Centos系统重，MySQL的数据目录就是 `/var/lib/mysql/`
### 1.2 相关命令目录

**相关命令目录：`/usr/bin`（mysqladmin、mysqlbinlog、mysqldump等命令）和/usr/sbin**
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ec3e6e6ecc71c1ca242c90a2f310bd5.png)

### 1.3 配置文件目录

**配置文件目录：`/usr/share/mysql-8.0`（命令及配置文件），`/etc/mysql`（如my.cnf）**
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a6bf07be74870eb9c32f9b49cc0df6e8.png)

## 2 数据库和文件系统的关系

像`InnoDB、MyISAM`这样的存储引擎都是把表存储在磁盘上的，操作系统用来管理磁盘的结构被称为**文件系统**

所以用专业一点的话来表述就是：
- 像InnoDB、MyISAM这样的存储引擎都是把表存储在文件系统上的。当我们想读取数据的时候，这些存储引擎会从文件系统中把数据读出来返回给我们，当我们想写入数据的时候，这些存储引擎会把这些数据又写回文件系统。

本章学习一下InnoDB和MyISAM这两个存储引擎的数据如何在文件系统中存储。
### 2.1 查看默认数据库

查看一下在我的计算机上当前有哪些数据库：
```mysql
SHOW DATABASES;
```

可以看到有4个数据库是属于MySQL自带的系统数据库
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0d19715ee7df05f6892cd0006267e40b.png)

- `mysql` 
	- MySQL 系统自带的核心数据库，它存储了MySQL的**用户账户和权限信息**，一些**存储过程、事件的定 义信息**，一些**运行过程中产生的日志信息**，一些帮助信息以及时区信息等。 
- `information_schema` 
	- MySQL 系统自带的数据库，这个数据库**保存着MySQL服务器 维护的所有其他数据库的信息** ，比如有 哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些 描述性信息，有时候也称之为 元数据 。在系统数据库 information_schema 中提供了一些以 innodb_sys 开头的表，用于表示内部系统表。
		```mysql
		mysql> USE information_schema; 
		Database changed 
		mysql> SHOW TABLES LIKE 'innodb_sys%'; 
		+--------------------------------------------+ 
		| Tables_in_information_schema (innodb_sys%) |
		+--------------------------------------------+ 
		| INNODB_SYS_DATAFILES | | INNODB_SYS_VIRTUAL| 
		| INNODB_SYS_INDEXES   | | INNODB_SYS_TABLES | 
		| INNODB_SYS_FIELDS    | | INNODB_SYS_TABLESPACES | 
		| INNODB_SYS_FOREIGN_COLS | | INNODB_SYS_COLUMNS | 
		| INNODB_SYS_FOREIGN   | | INNODB_SYS_TABLESTATS | 
		+--------------------------------------------+ 
		10 rows in set (0.00 sec)
		```
  

- `performance_schema` 
	- MySQL 系统自带的数据库，这个数据库里主要保存**MySQL服务器运行过程中的一些状态信息**，可以 用来 监控 MySQL 服务的各类性能指标 。包括统计最近执行了哪些语句，在执行过程的每个阶段都 花费了多长时间，内存的使用情况等信息。 
- `sys` 
	- MySQL 系统自带的数据库，这个数据库主要是**通过 视图 的形式把 information_schema 和 performance_schema 结合起来**，帮助系统管理员和开发人员监控 MySQL 的技术性能。

### 2.2 数据库在文件系统中的表示

查看一下Linux系统中，数据目录的内容
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/32165362653234a06b27c84e32154003.png)

这个数据目录下的文件和子目录比较多，除了` information_schema` 这个系统数据库外，其他的数据库 在 **数据目录** 下都有对应的子目录。

以我的 atguigudb 数据库为例，在MySQL8.0 中打开：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/05db925d896c8c6e3e5e7786c629b3f2.png)

### 2.3 表在文件系统中的表示

#### 2.3.1 InnoDB存储引擎模式

注意：**MySQL8.0中不再单独提供.frm，而是合并在.ibd文件中** ， 即将表结构、数据、索引，都放在了ibd文件中

1. **表结构** 
	- 为了保存表结构， InnoDB 在 数据目录 下对应的数据库子目录下创建了一个专门用于 描述表结构的文 件 ，文件名是这样： `表名.frm`
	- 比方说我们**在 atguigu 数据库下创建一个名为 test 的表**
	- 那在数据库 `atguigu` 对应的子目录下就会创建一个名为` test.frm` 的用于描述表结构的文件。.frm文件 的格式在不同的平台上都是相同的。这个后缀名为.frm是以 二进制格式 存储的，我们直接打开是乱码 的。

2. **表中数据和索引**
	- **① 系统表空间（system tablespace）**
		- 默认情况下，InnoDB会**在数据目录**下创建一个名为 `ibdata1` 、大小为 `12M` 的文件，这个文件就是对应 的 **系统表空间** 在文件系统上的表示。怎么才12M？注意这个文件是 **自扩展文件** ，当不够用的时候它会自 己增加文件大小。
		- 当然，如果你想让系统表空间对应文件系统上多个实际文件，或者仅仅觉得原来的 `ibdata1` 这个文件名 难听，那可以在MySQL启动时配置对应的文件路径以及它们的大小，比如我们这样修改一下my.cnf 配置 文件：
			```cnf
			[server] 
			innodb_data_file_path=data1:512M;data2:512M:autoextend
			```

	- **② 独立表空间（file-per-table tablespace）**
		- 在MySQL5.6.6以及之后的版本中，InnoDB并不会默认的把各个表的数据存储到系统表空间中，而是为 **每 一个表建立一个独立表空间** ，也就是说我们创建了多少个表，就有多少个独立表空间。使用 **独立表空间** 来 存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表 名相同，只不过添加了一个 `.ibd` 的扩展名而已，所以完整的文件名称长这样：`表名.ibd`

	- **③ 系统表空间与独立表空间的设置**
		- 我们可以自己**指定使用 系统表空间 还是 独立表空间 来存储数据**，这个功能由启动参数 `innodb_file_per_table` 控制，比如说我们想刻意将表数据都存储到 系统表空间 时，可以在启动 MySQL服务器的时候这样配置：
			```cnf
			[server] 
			innodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间
			```
		- 默认情况下，是使用独立表空间的
	- **④ 其他类型的表空间** 
		- 随着MySQL的发展，除了上述两种老牌表空间之外，现在还新提出了一些不同类型的表空间，比如通用 表空间（general tablespace）、临时表空间（temporary tablespace）等。
#### 2.3.2 MyISAM存储引擎模式

1. **表结构** 
	- 在存储表结构方面， MyISAM 和 InnoDB 一样，也是在 数据目录 下对应的数据库子目录下创建了一个专 门用于描述表结构的文件： `表名.frm`

2. **表中数据和索引**
	- 在MyISAM中的索引全部都是 二级索引 ，**该存储引擎的 数据和索引是分开存放 的**。所以在文件系统中也是 使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。假如 test 表使用MyISAM存储引擎的话，那么在它所在数据库对应的 atguigu 目录下会为 test 表创建这三个文 件：
		```
		test.frm 存储表结构 
		test.MYD 存储数据 (MYData) 
		test.MYI 存储索引 (MYIndex)
		```
### 2.4 总结

举例： 数据库a ， 表b 。 

1、**如果表b采用 InnoDB** ，`data\a`中会产生1个或者2个文件： 
- b.frm ：描述表结构文件，字段长度等 
- 如果采用 系统表空间 模式的，数据信息和索引信息都存储在 ibdata1 中 
- 如果采用 独立表空间 存储模式，data\a中还会产生 b.ibd 文件（存储数据信息和索引信息） 

此外：
- ① MySQL5.7 中会在data/a的目录下生成 db.opt 文件用于保存数据库的相关配置。
- 比如：字符集、比较 规则。而MySQL8.0不再提供db.opt文件。 
- ② **MySQL8.0中不再单独提供b.frm，而是合并在b.ibd文件中**。 

2、**如果表b采用 MyISAM** ，`data\a`中会产生3个文件： 
- MySQL5.7 中： b.frm ：描述表结构文件，字段长度等。 
- MySQL8.0 中 b.xxx.sdi ：描述表结构文件，字段长度等 
- b.MYD (MYData)：数据信息文件，存储数据信息(如果采用独立表存储模式) 
- b.MYI (MYIndex)：存放索引信息文件

### 2.5 视图在文件系统中的表示

我们知道MySQL中的视图其实是虚拟的表，也就是某个查询语句的一个别名而已，所以在存储视图的时候是不需要存储真实的数据的，只需要把它的结构存储起来就行了。

和表一样，描述视图结构的文件也会被存储到所属数据库对应的子目录下边，只会存储一个视图名.frm的文件。如：emp_details_view.frm
### 2.6 其他的文件

除了我们上边说的这些用户自己存储的数据以外，数据目录下还包括为了更好运行程序的一些额外文件，主要包括这几种类型的文件：
- **服务器进程文件**
	我们知道每运行一个MySQL服务器程序，都意味着启动一个进程。MySQL服务器会把自己的进程ID写入到一个文件中。
- **服务器日志文件**
	在服务器运行过程中，会产生各种各样的日志，比如常规的查询日志、错误日志、二进制日志、rdo日志等。这些日志各有各的用途，后面讲解。
- **默认/自动生成的SSL和RSA证书和密钥文件**
	主要是为了客户端和服务器安全通信而创建的一些文件。