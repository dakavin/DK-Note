---
文章标题: "[[1_Linux下的MySQL]]"
文章作者: Dakkk
文章概要: |
  介绍Linux环境下MySQL的安装、字符集配置、大小写规范设置及sql_mode配置，涵盖字符集问题解决、编码转换过程和严格模式设置
tags:
  - MySQL
  - Linux
  - 字符集
  - 配置
  - 数据库安装
  - sql_mode
  - 编码
  - CentOS
相关文章:
  - "[[1_Part1]]"
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/9_常见环境安装/2_安装MySQL]]"
  - "[[3_字符集详解]]"
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/3_CentOS 安装 Docker]]"
  - "[[3_Linux上的小技巧]]"
文章分类: 🗄️ 数据库技术
文章路径: 04-🗄️ 数据库技术/01-🐬 MySQL/02-🚀 高级特性/1_Linux下的MySQL.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

## 1 安装前说明
### 1.1 Linux系统及工具的准备

- `当前文件夹下，有克隆虚拟机的word教程`

- 安装并启动好两台虚拟机：`CentOS 7`

- 掌握克隆虚拟机的操作
	- 在VM程序中,右键虚拟机,克隆即可(注意选择完整克隆)
- mac地址修改
	- 操作如图所示  ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/16d3e9b45f2af1982d06a4500b15533b.png)

	- 然后先启动被克隆的虚拟机 , 以保证原来的虚拟机IP不会变, 因为克隆后的虚拟机IP是原来的虚拟机IP
	- 如果原来被克隆的虚拟机是固定IP的 , 就不需要考虑上述问题
- 主机名的修改
	- 进入新克隆的虚拟机 , 使用vim指令 , 打开 `/etc/hostname` 这个文件,修改主机名 
	- 然后重启 , 输入 reboot 指令
- ip地址固定 :  参考文章 [6.2 虚拟机配置固定IP](../../../06-🐧%20Linux系统/02-⚙️%20系统基础/01-💻%20命令行操作/4_Linux实用操作.md#6.2%20虚拟机配置固定IP)
- UUID修改 :  同样在固定ip地址的文件 , 修改其中的UUID值即可,随便修改

## 2 卸载与安装

具体卸载和安装过程, 请查看这篇文章: [2 MySQL](../../../06-🐧%20Linux系统/02-⚙️%20系统基础/01-💻%20命令行操作/7_Linux系统软件安装-1.md#2%20MySQL)
## 3 字符集相关问题

### 3.1 字符集相关设置

在MysQL8.0版本之前，默认字符集为1atin1，utf8字符集指向的是utf8mb3。网站开发人员在数据库设计的时候往往会将编码修改为utf8字符集。如果遗忘修改默认的编码，就会出现乱码的问题。

从MySQL8.0开始，数据库的默认编码将改为utf8mb4，从而避免上述乱码的问题。

```mysql
show variables like 'character%';
```

MySQL8.0的字符集
![image.png|380|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6dc3eba5f4f812bb258a9eb091dea9cc.png)

MySQL5.7的字符集(**server和database默认的字符集是latin1**)
![image.png|380|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c06fae55be38def7d23663503f8d438.png)

### 3.2 修改mysql5.7的字符集
- 在window下, 我们修改的是my.ini文件;
- 在linux下,我们修改的是my.cf文件
```shell
vim /etc/my.cf
```
- 在该文件的最后加上中文字符集配置
```cf
character_set_server=utf8
```

![image.png|380|380|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f4a0915ddf4520a3fc9b1bcce6c01e45.png)

- 最后重启一下mysql服务
```shell
systemctl restart mysqld.service
```

### 3.3 mysql5.7中已有的表字符集修改

修改已有数据库的字符集
```mysql
alter database dbtestl character set 'utf8mb4';
```

修改已有表的字符集
```mysql
alter table t_emp convert to character set 'utf8mb4';
```

> 注意：但是原有的数据如果是用非uft8编码的话，数据本身编码不会发生改变。已有数据需要导出或删除，然后重新插入。
### 3.4 各级别的字符集

MySQL有4个级别的字符集和比较规则，分别是：
- 服务器级别
- 数据库级别
- 表级别
- 列级别

执行如下SQL语句可以看到所有级别的字符集
```mysql
show variables like 'character%'
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb5412686bd39d56ab912c3ac1a4d652.png)


- `character_set_server!`：服务器级别的字符集
	- server的字符集 , 决定 database的字符集
- `character_set_database!`：当前数据库的字符集

小结
- 如果`创建或修改列`时没有显式的指定字符集和比较规则，则该列`默认用表的`字符集和比较规则
- 如果`创建表时`没有显式的指定字符集和比较规则，则该表`默认用数据库的`字符集和比较规则
- 如果`创建数据库时`没有显式的指定字符集和比较规则，则该数据库`默认用服务器的`字符集和比较规则
- **实际开发,使用MySQL8版本,就不需要去修改这些结构的字符集了!**

### 3.5 字符集与比较规则

**utf8与utf8mb4**

utf8字符集表示一个字符需要使用1~4个字节，但是我们常用的一些字符使用1~3个字节就可以表示了。而字符集表亦一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计MySQL的设计者偷偷的定义了两个概念：
- `utf8mb3` : 阉割过的utf8字符集，只使用1~3个字节表示字符。
- `utf8mb4` : 正宗的utf8字符集，使用1~4个字节表示字符。

在MySQL中utf8是utf8b3的别名，所以之后在MySQL中提到utf8就意味着使用1~3个字节来表示一个字符。

如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情，那请使用utf8mb4.

此外，通过如下指令可以查看MySQL支持的字符集：
```mysql
SHOW CHARSET;
#或者
SHOW CHARACTER SET;
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/941abe80fd4ba4cfe6e159058f0eef57.png)


**比较规则**

上表中，MySQL版本一共支持41种字符集，其中的 Default collation 列表示这种字符集中一种默认 的比较规则，里面包含着该比较规则主要作用于哪种语言，比如 utf8_polish_ci 表示以波兰语的规则 比较， utf8_spanish_ci 是以西班牙语的规则比较， utf8_general_ci 是一种通用的比较规则。 后缀表示该比较规则是否区分语言中的重音、大小写。具体如下：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10b6ed1441afdfbe9a0f12e31db05b9b.png)

最后一列` Maxlen` ，它代表该种字符集表示一个字符最多需要几个字节。

常用操作:
```mysql
#查看GBK字符集的比较规则 
SHOW COLLATION LIKE 'gbk%'; 
#查看UTF-8字符集的比较规则 
SHOW COLLATION LIKE 'utf8%';

#查看服务器的字符集和比较规则 
SHOW VARIABLES LIKE '%_server'; 
#查看数据库的字符集和比较规则 
SHOW VARIABLES LIKE '%_database'; 
#查看具体数据库的字符集 
SHOW CREATE DATABASE dbtest1; 
#修改具体数据库的字符集 
ALTER DATABASE dbtest1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';

#查看表的字符集 
show create table employees; 
#查看表的比较规则 
show table status from atguigudb like 'employees'; 
#修改表的字符集和比较规则 
ALTER TABLE emp1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

### 3.6 请求到响应过程中字符集的变化

- character_set_client：服务器解码请求时使用的字符集
- character_set_connection：服务器处理请求时会把请求字符串从character_set_client转为character_set_connection 
- character_set_results：服务器向客户端返回数据时使用的字符集

```mysql
select * from emp1 where name = '我';
```
客户端向服务端发送如上数据
- 客户端的字符集会决定 '我' 是以什么形式编码的
	- 注意 : 一些可视化工具会使用自定义的字符集来编码发送的数据 , 不采用操作系统默认的字符集
- 服务端 , 先用client对数据进行解码
- 然后将解码的数据按照 connection 进行编码(转化)
- 如果表emp1 对应的列 name 采用的编码和connection一致 , 就会直接在列中找到对应的记录 , 并使用该列的字符集进行解码
	- 如果不一致 , 还需要进行一次字符集转换
- 找到对应的记录之后 , 会使用results 对结果进行编码 , 然后发送给客户端

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/139d6254978b6362dc7001a3fb0055a6.png)

## 4 规范

### 4.1 Windows 和 Linux 平台的区别

在 SQL 中，关键字和函数名是不用区分字母大小写的，比如 SELECT、WHERE、ORDER、GROUP BY 等关 键字，以及 ABS、MOD、ROUND、MAX 等函数名。

不过在 SQL 中，你还是要确定大小写的规范，因为在 Linux 和 Windows 环境下，你可能会遇到不同的大 小写问题。

**windows系统默认大小写不敏感 ，但是 linux系统是大小写敏感的 。**

```mysql
SHOW VARIABLES LIKE '%lower_case_table_names%'
```

- Windows系统下 : ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/46e9f3a78e5bfc69345d8c219fb0e734.png)

- Linux系统下 : ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d83d2826612326d9b537b94c976b4fb7.png)
- lower_case_table_names参数值的设置：
	- 默认为0 , 大小写敏感!
	- 设置1 , 大小写不敏感 ; 创建的数据都是以小写形式存放在磁盘上 , 对于Sql语句都是转换为小写进行查找
	- 设置2 , 创建的数据依据语句上格式存放 , 查找都是转换为小写进行

- 总结

MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：
1、数据库名、表名、表的别名、变量名是严格区分大小写的； 
2、关键字、函数名称在 SQL 中不区分大小写； 
3、列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写的；

### 4.2 Linux下大小写规则设置

当想设置为大小写不敏感时，要在 my.cnf 这个配置文件` [mysqld]` 中加入 `lower_case_table_names=1` ，然后重启服务器。

- 但是要在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将找不到数据库名。
- 此参数适用于MySQL5.7
- 在MySQL 8下禁止在重新启动 MySQL 服务时将 `lower_case_table_names` 设置成不同于初始化 MySQL 服务时设置的 `lower_case_table_names` 值。

如果非要将MySQL8设置为大小写不敏感，具体步骤为：
1、停止MySQL服务 
2、删除数据目录，即删除 `/var/lib/mysql `目录 
3、在MySQL配置文件` /etc/my.cnf `中添加` lower_case_table_names=1`
4、启动MySQL服务


### 4.3 SQL编写建议

如果你的变量名命名规范没有统一，就可能产生错误。这里有一个有关命名规范的建议：
> 1. 关键字和函数名称全部大写； 
> 2. **数据库名、表名、表别名、字段名、字段别名等全部小写**； 
> 3. SQL 语句必须以分号结尾。

## sql_mode的合理设置

### 宽松模式 vs 严格模式

**宽松模式**

如果设置的是宽松模式，那么我们在插入数据的时候，即便是给了一个错误的数据，也可能会被接受， 并且不报错。
- **举例** : 我在创建一个表时，该表中有一个字段为name，给name设置的字段类型时 char(10) ，如果我 在插入数据的时候，其中name这个字段对应的有一条数据的 长度超过了10 ，例如'1234567890abc'，超 过了设定的字段长度10，那么不会报错，并且取前10个字符存上，也就是说你这个数据被存为 了'1234567890'，而'abc'就没有了。但是，我们给的这条数据是错误的，因为超过了字段长度，但是并没 有报错，并且mysql自行处理并接受了，这就是宽松模式的效果。
- 应用场景 : 通过设置sql mode为宽松模式，来保证大多数sql符合标准的sql语法，这样应用在不同数据 库之间进行 迁移 时，则不需要对业务sql 进行较大的修改。

**严格模式**

出现上面宽松模式的错误，应该报错才对，所以MySQL5.7版本就将sql_mode默认值改为了严格模式。所 以在 生产等环境 中，我们必须采用的是严格模式，进而 开发、测试环境 的数据库也必须要设置，这样在 开发测试阶段就可以发现问题。**并且我们即便是用的MySQL5.6，也应该自行将其改为严格模式**。

**开发经验** ：MySQL等数据库总想把关于数据的所有操作都自己包揽下来，包括数据的校验，其实开发 中，我们**应该在自己 开发的项目程序级别将这些校验给做了** ，虽然写项目的时候麻烦了一些步骤，但是这 样做之后，我们在进行数据库迁移或者在项目的迁移时，就会方便很多。

若设置模式中包含了 NO_ZERO_DATE ，那么MySQL数据库不允许插入零日期，插入零日期会抛出错误而 不是警告。例如，表中含字段TIMESTAMP列（如果未声明为NULL或显示DEFAULT子句）将自动分配 DEFAULT '0000-00-00 00:00:00'（零时间戳），这显然是不满足sql_mode中的NO_ZERO_DATE而报错。

### 模式查看和设置

```mysql
select @@session.sql_mode 
select @@global.sql_mode 
#或者 
show variables like 'sql_mode';
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10b0d3472c47c11a2cf7f77d7ad4e7d5.png)

**临时设置方式：设置当前窗口中设置sql_mode**
```mysql
SET GLOBAL sql_mode = 'modes...'; #全局 
SET SESSION sql_mode = 'modes...'; #当前会话
```

举例：
```mysql
#改为严格模式。此方法只在当前会话中生效，关闭当前会话就不生效了。
set SESSION sql_mode='STRICT_TRANS_TABLES';

#改为严格模式。此方法在当前服务中生效，重启MySQL服务后失效。 
set GLOBAL sql_mode='STRICT_TRANS_TABLES';
```

**永久设置方式：在/etc/my.cnf中配置sql_mode**
- 在my.cnf文件(windows系统是my.ini文件)，新增：
```ini
[mysqld] sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

然后 重启MySQL 。 

当然**生产环境上是禁止重启MySQL服务的**，所以采用 **临时设置方式** + **永久设置方式** 来解决线上的问题， 那么即便是有一天真的重启了MySQL服务，也会永久生效了。