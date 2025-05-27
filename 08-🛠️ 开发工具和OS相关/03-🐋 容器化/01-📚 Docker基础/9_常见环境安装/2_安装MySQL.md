---
文章标题: "[[2_安装MySQL]]" 
文章作者: Dakkk
文章概要: |
  文章详细讲解了如何在Docker中安装和配置MySQL 5.7，涵盖目录准备、容器启动、数据卷挂载、自定义配置文件（字符集、时区）设置、用户与权限管理，以及Docker容器的基础操作，提供完整部署指南。
tags:
- "Docker"
- "MySQL"
- "安装"
- "配置"
- "数据持久化"
- "容器管理"
- "数据库部署"
相关文章:
- "[[1_数据库概述]]"
- "[[1_Linux下的MySQL]]"
- "[[1_MinGW编译器的安装和配置]]"
- "[[1_Part1]]"
- "[[2_📕Docker 本地安装 Minio 对象存储]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/03-🐋 容器化/01-📚 Docker基础/9_常见环境安装/2_安装MySQL.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

[Docker实操：安装MySQL5.7详解（保姆级教程）-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2356690)
## 1 准备

先创建3个目录，创建MySQL容器时会挂载容器的卷（Volume），用于Docker和[宿主机](https://cloud.tencent.com/product/cdh?from_column=20065&from=20065)（Centos）之间文件共享，包括配置文件、数据文件和日志文件。

**什么是卷（Volume）？命令 docker -v 中的“-v”就是这个卷，“-v”只是“--volume”的简写。**

**客官请留步，多少的看一下！！！**

**Docker官方文档解释卷的含义：**[**https://docs.docker.com/storage/volumes/**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fdocs.docker.com%2Fstorage%2Fvolumes%2F&source=article&objectId=2356690)

**由于本人的英文水平非常的一般，所有我看的中文文档：**[**https://www.dockerdocs.cn/storage/volumes/**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fwww.dockerdocs.cn%2Fstorage%2Fvolumes%2F&source=article&objectId=2356690)

![](https://developer.qcloudimg.com/http-save/yehe-admin/1e28f4f477606533b112357753feea5c.png)

**下面这个操作可以不用，因为下面会有整合完整的使用命令！！！**

使用 -p 创建多级目录，即 mydata 目录下创建 mysql 目录， mysql 目录下又创建 log 、data 、conf 三个目录：

```javascript
mkdir -p /docker-data/mysql/log
mkdir -p /docker-data/mysql/data
mkdir -p /docker-data/mysql/conf
```

先以最简单的方式启动容器，执行命令如下：

```cmd
docker run -d \
--name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7
```

参数说明：
- `-d`：以后台的方式运行；
- `--name mysql`：指定容器的名称为 `mysql`;
- `-p 3306:3306` 将宿主机的 3306 端口挂载到容器中的 3306 端口上；
- `-e MYSQL_ROOT_PASSWORD=123456`：指定 root 用户的密码为 123456；

命令执行后，可以通过 `docker ps`命令来查看正在运行的 `Docker` 容器，确认下 `MySQL` 容器是否运行成功

然后使用如下命令，复制docker容器中mysql的相关配置文件
```shell
# 将容器中的 mysql 配置文件复制到宿主机中指定路径下 
docker cp mysql:/etc/mysql/. /docker-data/mysql/conf

# 将容器中的 mysql 日志文件复制到宿主机中指定路径下 
docker cp mysql:/var/log/mysql/. /docker-data/mysql/log

# 将容器中的 mysql 数据文件复制到宿主机中指定路径下 
docker cp mysql:/var/lib/mysql/. /docker-data/mysql/data
```

末尾的`/.`表示将容器目录中的所有文件复制到宿主机目录中，而不是将整个目录复制到宿主机

## 2 安装

- 拉取MySQL指定版本的镜像

```javascript
docker pull mysql:5.7
```

- 运行容器

```javascript
docker run -p 3306:3306 --name mysql \
-v /docker-data/mysql/log:/var/log/mysql \
-v /docker-data/mysql/data:/var/lib/mysql \
-v /docker-data/mysql/conf:/etc/mysql \
--restart=always \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

```javascript
docker run -p 3306:3306 --name mysql \
-v /docker-data/mysql/log:/var/log/mysql \
-v /docker-data/mysql/data:/var/lib/mysql \
-v /docker-data/mysql/conf:/etc/mysql \
-v /etc/localtime:/etc/localtime \
--restart=always \
-e TZ='Asia/Shanghai' \
-d mysql:5.7
```

这个 Docker 命令是用于启动 MySQL 5.7 容器的，让我们解释其中的各个部分：

> `**docker run**`**：这是 Docker 启动容器的命令。**
> 
> `**-p 3306:3306**`**：这部分命令将主机的端口** `**3306**` **映射到容器内的** `**3306**` **端口。这样，您可以通过主机的** `**3306**` **端口来访问容器内运行的 MySQL 服务。**
> 
> `**--name mysql**`**：通过此选项，您为容器指定了一个名称，即** `**mysql**`**。这使得容器更容易识别和管理。**
> 
> `**-v /mydata/mysql/log:/var/log/mysql**`**：这是一个数据卷挂载操作，将主机上的** `**/mydata/mysql/log**` **目录挂载到容器内的** `**/var/log/mysql**` **目录。这样，MySQL 日志文件将在主机上存储，以供查看。**
> 
> `**-v /mydata/mysql/data:/var/lib/mysql**`**：同样，这是另一个数据卷挂载操作，将主机上的** `**/mydata/mysql/data**` **目录挂载到容器内的** `**/var/lib/mysql**` **目录。这用于将 MySQL 数据文件保存在主机上，以便数据持久化。**
> 
> `**-v /mydata/mysql/conf:/etc/mysql**`**：此挂载操作将主机上的** `**/mydata/mysql/conf**` **目录挂载到容器内的** `**/etc/mysql**` **目录。这样，您可以提供自定义的 MySQL 配置文件。**
> 
> `**--restart=always**`**：这个选项指示 Docker 在容器退出时自动重新启动容器。这对于确保 MySQL 服务一直可用非常有用。**
> 
> `**-e MYSQL_ROOT_PASSWORD=123456**`**：这个选项设置 MySQL 根用户的密码。在示例中，密码被设置为** `**123456**`
> 
> `**-d**`**：这个选项使容器在后台运行，以允许您继续在终端中执行其他命令。**
> 
> `**mysql:5.7**`**：这是要运行的 Docker 镜像的名称和标签。在此示例中，使用 MySQL 5.7 镜像。**

这个命令将启动一个 MySQL 5.7 容器，将 MySQL 数据、日志和配置文件挂载到主机上的目录中，设置 MySQL 根密码，并允许容器在后台运行，以及在容器退出时自动重新启动。这是一个典型的用例，用于在 Docker 中运行 [MySQL 数据库](https://cloud.tencent.com/product/cdb?from_column=20065&from=20065)容器。

## 3 宿主机新建配置文件

###### 3.1.1.1.1 在宿主机，宿主机，宿主机上新建！！！

自定义的 my.cnf 配置文件。 **注意！！！**在 `/mydata/mysql/conf/` 目录下创建自定义的 `my.cnf` 配置文件。文件名随意，`文件格式必须为 .cnf` 。

```javascript
vi /docker-data/mysql/conf/my.cnf
```

添加容器运行的配置参数。**使用的是`utf8mb4`编码而不是 utf8 编码**。

```javascript
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
init_connect="SET collation_connection = utf8mb4_unicode_ci"
init_connect="SET NAMES utf8mb4"
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

default-time-zone = '+08:00'
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
lower_case_table_names=1

# 启用错误日志
log_error = /var/log/mysql/error.log

# 启用查询日志
general_log = 1
general_log_file = /var/log/mysql/general.log

# 启用慢查询日志
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log

```

保存后，已经创建了一个名为 `/mydata/mysql/conf/my.cnf` 的 MySQL 配置文件。这个文件包含了一些 MySQL 的配置选项，用于配置 MySQL 服务器的字符集和排序规则等设置。让我解释一下这个配置文件的内容：

- `[client]` 部分包含了 MySQL 客户端的配置，确保客户端使用 UTF-8 字符集。
- `[mysql]` 部分也配置了 MySQL 客户端的默认字符集。
- `[mysqld]` 部分包含了 MySQL 服务器的配置选项，用于配置 MySQL 服务器的行为。

以下是这个配置文件的各个配置选项的解释：

> `**default-character-set=utf8mb4**` **和** `**default-character-set=utf8mb4**`**：这两个选项在** `**[client]**` **和** `**[mysql]**` **部分都设置了默认字符集为 UTF-8，确保客户端和服务器使用相同的字符集。**
> 
> `**init_connect='SET collation_connection = utf8mb4_unicode_ci'**` **和** `**init_connect='SET NAMES utf8mb4'**`**：这些选项在** `**[mysqld]**` **部分设置了初始化连接时执行的 SQL 语句。这些语句设置了连接的字符集和排序规则为 UTF-8 和** `**utf8mb4_unicode_ci**`**。**
> 
> `**character-set-server=utf8mb4**`**：这个选项设置了 MySQL 服务器的字符集为 UTF-8。**
> 
> `**collation-server=utf8mb4_unicode_ci**`**：这个选项设置了 MySQL 服务器的排序规则为** `**utf8mb4_unicode_ci**`**，通常用于支持国际化和多语言字符的正确排序。**
> 
> `**skip-character-set-client-handshake**`**：这个选项用于禁用客户端字符集握手，允许客户端和服务器之间的字符集设置更加灵活。**
> 
> `**skip-name-resolve**`**：这个选项禁用了主机名解析，以提高连接性能。**

它适用于确保 MySQL 以正确的字符集和排序规则处理数据。确保将这个配置文件用于启动 MySQL 服务器，可以通过 `-v` 选项将配置文件挂载到容器内。例如：

```javascript
docker run -d -p 3306:3306 --name mysql \
-v /docker-data/mysql/log:/var/log/mysql \
-v /docker-data/mysql/data:/var/lib/mysql \
-v /docker-data/mysql/conf:/etc/mysql \
--restart=always \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

将在容器内使用自定义配置文件 `/mydata/mysql/conf/my.cnf` 来启动 MySQL 服务器。

## 4 重启服务


```javascript
docker restart mysql
```

## 5 查看日志


```javascript
docker logs mysql
```

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/36e4d71cea1c0d041fe4c77507d133ad.png)

## 6 测试连接

![](https://developer.qcloudimg.com/http-save/yehe-admin/9fd58340e5390f5f38fe972289e8dcc8.png)

## 7 进入容器


```javascript
docker exec -it mysql bash
```

![](https://developer.qcloudimg.com/http-save/yehe-admin/e5cb1e81ffab8f3b96f3cb3ffab29fc9.png)

- 可以使用外部工具连接测试
```javascript
mysql -h 主机IP地址 -P 3306 -u root -p
```

### 7.1 退出MySQL服务

```javascript
\q
```

### 7.2 退出容器


```javascript
exit
```

## 8 **添加配置**

1. **修改容器中的MySQL时间不同步的问题**
2. **修改容器中的MySQL分组only_full_group_by问题**
3. **修改表名不区分大小写问题**

在MySQL配置文件（通常是my.cnf）中，确保已正确配置时区。您可以在配置文件中添加以下内容：
```javascript
[mysqld]
default-time-zone = '+08:00'
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
lower_case_table_names=1
```

这是一个 MySQL 配置文件（`my.cnf` 或 `my.ini`）中的一部分，用于设置[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)的默认时区、SQL 模式和其他选项。以下是这些选项的详细解释：

> `default-time-zone = '+08:00'`** ：设置数据库的默认时区为 UTC+8。这意味着在执行与日期和时间相关的操作时，数据库将根据这个时区进行转换。**
> 
> `sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`**：设置 SQL 模式，以便更严格地遵循 SQL 标准。这有助于避免潜在的数据问题和错误。具体来说，这些模式包括：**
> 
> `**STRICT_TRANS_TABLES**`**：禁止在严格模式下插入无效日期和数据。**
> 
> `**NO_ZERO_IN_DATE**`**：禁止使用零日期（如 '0000-00-00'）。**
> 
> `**NO_ZERO_DATE**`**：禁止使用零日期（如 '0000-00-00'）。**
> 
> `**ERROR_FOR_DIVISION_BY_ZERO**`**：将除以零的操作视为错误，而不是警告。**
> 
> `**NO_AUTO_CREATE_USER**`**：禁止自动创建用户。**
> 
> `**NO_ENGINE_SUBSTITUTION**`**：如果请求的存储引擎不可用，禁止自动使用替代存储引擎。**
> 
> `**lower_case_table_names = 1**`**：将所有表名存储为小写。这有助于避免因大小写不同而导致的表名混淆和错误。在某些操作系统（如** **Windows** **和 macOS）上，这个选项可能对大小写不敏感，而在其他操作系统（如** **Linux***）上可能对大小写敏感。设置为 1 表示启用该功能，0 表示禁用。**

### 8.1 重启服务/容器

```javascript
docker restart mysql
```

### 8.2 修改密码

#### 8.2.1 进入容器

```javascript
docker exec -it mysql bash
```

#### 8.2.2 登录MySQL

```javascript
mysql -u root -p123456
```

![](https://developer.qcloudimg.com/http-save/yehe-admin/20e5dab1f6abc7f7e3b746bb0bbfc9b0.png)

#### 8.2.3 修改root密码和创建用户

登录进mysql，要求更改密码（**密码需要一定的复杂度！！！**），并且给其他机器授权可以登录mysql
```mysql
ALTER USER USER() IDENTIFIED BY '密码';

# 立即刷新配置
flush privileges;

# 创建另外一个用户也拥有 root权限
create user '用户名'@'%' identified with mysql_native_password by '密码';

# 再次刷新配置
flush privileges;
```

此时就可以使用该用户来登录数据库了！

把所有数据库的所有表的所有权限赋值给位于所有IP地址的==新建用户==
```mysql
grant all privileges on *.* to '新建用户'@'%' with grant option;
```


**注意，注意，注意！！！到此为止，docker下的MySQL服务已经可用了，下面是一些细化操作。**

**注意，注意，注意！！！到此为止，docker下的MySQL服务已经可用了，下面是一些细化操作。**

**注意，注意，注意！！！到此为止，docker下的MySQL服务已经可用了，下面是一些细化操作。**

---

## 9 禁用 root 账户被外部工具连接

进入到容器里，连接mysql，删除mysql数据库user表中 user=“root”，host="%"的那条记录。因为这条数据会允许 root 账户被允许外部工具（如Navicat或SQLyog）连接，实际上，应该禁止这么做，正确做法是只允许 root 账户本地连接。如果想 root 账户继续被外部工具连接，那就把root密码设置得更复杂，过于简单不安全！

切换到 mysql 数据库，并查看 user 表。

```javascript
use mysql;
```

```javascript
select user,host from user;
```

![](https://developer.qcloudimg.com/http-save/yehe-admin/5a6d0712235fe684ad70c4784b1a929e.png)

删除mysql数据库user表中 user=“root”，host="%"的那条记录，并刷新权限。

**可以这么做，但是不建议这么干，改个复杂的密码，不暴露给其他使用者就好了！！！**

**可以这么做，但是不建议这么干，改个复杂的密码，不暴露给其他使用者就好了！！！**

**可以这么做，但是不建议这么干，改个复杂的密码，不暴露给其他使用者就好了！！！**

```javascript
delete user from mysql.user where user='root' and host='%';
```

```javascript
flush privileges;
```

---

## 10 创建新账户供外部工具连接

使用 CREATE 创建账户，例如对应mysql.user表中，字段user为 goboy，字段host为 % ，账号密码为 123456 ，“%”代表任何主机。使用 GRANT 授予账户特定权限。

创建用户和密码

```javascript
CREATE USER 'goboy'@'%' IDENTIFIED BY '123456';
```

授予账户特定权限。ALL 和 ALL PRIVILEGES 是一样的，可简写为 ALL 。

```javascript
GRANT ALL ON *.* TO 'goboy'@'%' WITH GRANT OPTION;
```

刷新账号权限。

```javascript
FLUSH PRIVILEGES;
```

![](https://developer.qcloudimg.com/http-save/yehe-admin/101f61ef0481a3459746e3a3c9f4e629.png)

## 11 新用户登录测试

![](https://developer.qcloudimg.com/http-save/yehe-admin/f2db4cc85a5e6254aa6aa868c2566e1c.png)

## 12 容器基础操作

### 12.1 启动容器

```javascript
docker start mysql
或
docker start 容器ID
```

### 12.2 停止容器

```javascript
docker stop mysql
或
docker stop 容器ID
```

### 12.3 删除容器

```javascript
docker rm mysql
或
docker rm 容器ID
```

### 12.4 重启容器


```shell
docker restart mysql
或
docker restart 容器ID
```

### 12.5 查看状态

#### 12.5.1 查看所有容器的运行状态，包括运行的和停止的


```javascript
docker ps -a
```

#### 12.5.2 查看所有运行中的容器的状态，不包括停止的

```javascript
docker ps
```