
## 1 前言

### 1.1 为什么学习各类软件在Linux上的部署

在前面，我们学习了许多的Linux命令和高级技巧，这些知识点比较零散，同学们跟随着课程的内容进行练习虽然可以基础掌握这些命令和技巧的使用，但是并没有一些具体的实操能够串联起来这些知识点。

所以，现在我们设计了各类软件在Linux上部署安装的实战章节，可以让同学们：

- 对前面学习的各类操作命令进行复习和练习，从而深度掌握它们
- 本章节中演示部署的软件，包含了IT行业各类岗位中所必须使用的，如：Java后台、大数据开发、运维开发、测试、AI等。无论学习Linux后从事什么岗位，这些内容都会给你带来帮助

> 对于零基础学员，实战课程中所讲解的软件大概率多数大家并不了解。
>
> 所以，课程仅涉及到安装部署，不对软件的使用做详细说明。
>
> 同学们在这个过程中，可能会遇到各种各样的错误，`不要怕`，解决它，将会给你带来极大的提升。


**学习目标**

对于本部分的内容学习，我们设计两个目标：
- 对于零基础或未从业的学员，不要求深入理解所安装部署的软件是什么，仅仅能够跟随课程成功的将其部署安装并运行成功即可
- 在这个过程中，主要锻炼大家对Linux操作系统的熟练度，此乃零基础未从业学员的第一学习目标
- 对于有基础或已从业的学员，本章节讲解的软件涵盖了大多数IT从业者所能接触到的，特别是大数据开发、后端开发两个主流方向，可以作为参考资料，以便在工作中有所帮助。

本章节内的各类软件安装，==不强制要求全部学习==
1. 零基础学员，建议全部学习，作为前面学习内容的总结和实战
2. IT从业者、有经验学员，可以按需选择，选择工作中需要用到的进行学习

> 章节内包含的软件并非100%涵盖了IT开发领域中所需要的内容。
>
> 如果您对某些软件的安装有强烈需求，且课程中没有提供教程，可以私信B站："黑马大数据曹老师"，老师会酌情根据时间安排补充上去哦。

### 1.2 前置要求

1. 实战章节要求同学们==务必全部学习前面的知识点==，即：初识Linux、Linux基础命令、Linux权限管理、Linux高阶技巧这4个章节，请勿跳过前面的章节学习实战章节。
2. 实战章节中会开启多台虚拟机，请尽量确保电脑的内存在：8GB（包含8GB）以上。如内存不足可以扩充内存条或购买阿里云、UCloud等云服务器临时使用（1个月多台低配服务器几十块左右）

> 对于云平台上购买服务器，可以参阅最后的章节（云服务）

### 1.3 注意

下面全部的软件安装的相关流程，90%都是取自软件自身的官方网站。

一个合格的程序员要有良好的信息收集能力哦

## 2 MySQL

### 2.1 简介

MySQL数据库管理系统（后续简称MySQL），是一款知名的数据库系统，其特点是：轻量、简单、功能丰富。

MySQL数据库可谓是软件行业的明星产品，无论是后端开发、大数据、AI、运维、测试等各类岗位，基本上都会和MySQL打交道。

让我们从MySQL开始，进行实战的Linux软件安装部署。

本次课程分为2个版本进行安装：

- MySQL 5.7版本安装
- MySQL 8.x版本安装

> 由于MySQL5.x和8.x各自有许多使用者，所以这两个版本我们都演示安装一遍

### 2.2 注意

MySQL的安装过程中，除了会使用Linux命令外，还会使用到少量的数据库专用的：SQL语句

对于SQL语句我们并未涉及，所以可以跟随教程的内容，复制粘贴即可


> 如有时间，建议可以在学习完Linux系统之后，学习一下MySQL数据库
>
> 无论从事什么方面的开发，Java后端、大数据、AI、前端、Linux运维等，都会要求掌握MySQL数据库的
>
> 可以说，MySQL是IT开发从业者必备的技能了。

### 2.3 卸载

#### 2.3.1 查看是否安装过MySQL

- 如果你是用rpm安装, 检查一下RPM PACKAGE：
```shell
rpm -qa | grep -i mysql # -i 忽略大小写
```

- 检查mysql service：
```shell
systemctl status mysqld.service
```

#### 2.3.2 MySQL的卸载

1. 关闭 mysql 服务
```shell
systemctl stop mysqld.service
```

2. 查看当前 mysql 安装状况
```shell
rpm -qa | grep -i mysql
# 或
yum list installed | grep mysql
```

3. 卸载上述命令查询出的已安装程序
```shell
yum remove mysql-xxx mysql-xxx mysql-xxx mysqk-xxxx
```
务必卸载干净，反复执行`rpm -qa | grep -i mysql`确认是否有卸载残留

4. 删除 mysql 相关文件
- 查找相关文件
```shell
find / -name mysql
```
- 删除上述命令查找出的相关文件

```shell
rm -rf xxx
```

5. 删除 my.cnf
```shell
rm -rf /etc/my.cnf
```

### 2.4 安装

#### 2.4.1 安装

首先，尝试一下直接使用 yum 安装 MySQL
```text
yum install mysql-community-server
```

如果成功，表示不需要配置MySQL rpm 源信息，直接就安装完成了

但是，如果出现以下错误：
```text
Loading mirror speeds from cached hostfile
没有可用软件包 mysql-community-server。
错误：无须任何处理
```

表示我们没有添加安装包的源信息，需要安装 MySQL rpm 源信息

#### 2.4.2 安装 MySQL rpm 源信息

打开 [http://dev.mysql.com/downloads/repo/yum/](https://link.zhihu.com/?target=http%3A//dev.mysql.com/downloads/repo/yum/)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/16c4bc328f124cf2085f94f7e86af223.webp)

根据你的系统版本，选择对应的安装包，例如我的是CentOS 7.5，这个系统的Linux内核是 Linux 7，所以我选择了红框内的地址，大家依次类推。

拼接下载地址头：[http://dev.mysql.com/get/](https://link.zhihu.com/?target=http%3A//dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm)，得到以下地址

```text
 http://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
```

使用 wget + 刚才拼接的地址，下载安装包源信息
```text
wget  http://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
```

rpm 安装源信息
```text
rpm -ivh mysql80-community-release-el7-7.noarch.rpm
```

#### 2.4.3 再次安装

再尝试使用 yum 安装MySQL
```text
yum install mysql-community-server
```

安装过程中，会提示让我们确认，一律输入 `y` 按回车即可

安装完成后，yum会自动覆盖自带的mariaDB，所以不需要我们手动卸载它

#### 2.4.4 检查安装是否成功

检查一下刚才的安装是否成功
```text
rpm -qa | grep mysql
```

输出：
```text
mysql-community-libs-compat-8.0.33-1.el7.x86_64
mysql-community-icu-data-files-8.0.33-1.el7.x86_64
mysql80-community-release-el7-7.noarch
mysql-community-common-8.0.33-1.el7.x86_64
mysql-community-libs-8.0.33-1.el7.x86_64
mysql-community-server-8.0.33-1.el7.x86_64
mysql-community-client-8.0.33-1.el7.x86_64
mysql-community-client-plugins-8.0.33-1.el7.x86_64
```
输出类似以上内容，表示安装完成

检查mariaDB是否被覆盖
```text
rpm -qa | grep mariadb
```
输出空，表示 mariaDB 已经被成功覆盖。

#### 2.4.5 3.5 MySQL 常用命令

```text
# 启动
systemctl start mysqld

# 第一次启动后，可以查看mysql初始化密码
grep 'temporary password' /var/log/mysqld.log

# 重启
systemctl restart mysqld

# 停止
systemctl stop mysqld

#查看状态
systemctl status mysqld

#开机启动
systemctl enable mysqld
systemctl daemon-reload

# 查看进程、版本信息
ps -ef | grep mysql
或
netstat -atp

# 登录
mysql -u root -p'密码内容'

# 查看所有表
show databases;

# 进入数据库
use 表名

# 查看所有表
show tables

# 查看某张表信息
desc 表名

# 查
select * from 表名
# 删
delete from 表名 where field=xx
# 改
update 表名 set field='xxx' where field='xxx';
```

#### 2.4.6 登录和修改密码

我们安装的时候，并没有设置初始密码
所以 mysql 在第一次启动的时候，会自动初始化一个密码
通过以下这行代码，我们可以查看 mysql 自动初始化的密码：
```text
# 第一次启动后，可以查看mysql初始化密码
grep 'temporary password' /var/log/mysqld.log

输出（root@localhost: 后面的是密码）：
2023-04-21T06:03:27.071550Z 6 [Note] [MY-010454] [Server] A temporary password
is generated for root@localhost: r2to%yZ%a)%s
```

**登录**
```text
# 登录mysql，一定要注意：-p和'密码'之间是没有空格的
mysql -u root -p'r2to%yZ%a)%s'
```

**修改 root 密码**
注意了，默认的密码策略，需要：大写英文 + 特殊字符 + 数字
```text
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root_123';
```


有些同学可能会觉得，老子密码想设置啥就设置啥，轮得到你这 mysql 在这里瞎BB？
那也是可以修改密码校验策略的
首先，安装密码验证插件
> 但是有个前提，你还是需要先按它的要求修改第一次密码，才能安装密码验证策略插件，哈哈
```text
1、先按照mysql的要求，修改一次密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root_123';

2、退出mysql
exit

3、重新登录mysql
mysql -u root -p'Root_123'

4、安装密码验证插件
install plugin validate_password soname 'validate_password.so';

5、查看是否启用了插件
select plugin_name, plugin_status from information_schema.plugins where plugin_name like 'validate%';
+-------------------+---------------+
| plugin_name       | plugin_status |
+-------------------+---------------+
| validate_password | ACTIVE        |
+-------------------+---------------+
输出这样的内容，表示成功启用
```

查看验证策略的键、值信息
```text
SHOW VARIABLES LIKE 'validate_password%';

对于高版本的mysql，例如mysql 8，验证策略的key，是 validate_password.xxx
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | MEDIUM|
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+


对于低版本的mysql，例如mysql 5.7，验证策略的key，是 validate_password_xxx
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | MEDIUM|
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
```

我们修改密码策略和密码长度
我的策略信息的 key ，是 `validate_password.xxx`这个格式的，所以按照如下进行设置
```text
设置密码校验策略为：0（只验证密码长度）
set global validate.password_policy=0;

设置密码最低长度=N，例如设置密码最低长度=6，也就是密码最少要设置6个字符及以上
set global validate.password_length=6;
```

好了，现在密码就可以按照你刚才配置的策略，来进行设置密码了
```text
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

#### 2.4.7 开放 root 账户远程登录

```mysql
# 可以直接查看user表
use mysql;
select host.user from user;
```

```text
# 登录
mysql -u root -p'密码'

# 如果你的数据库是 mysql 8 及以上
# 1、进入数据库
use mysql
# 2、修改user表
update user set host='%' where user='root';

# mysql 5.7 及之前，执行这行代码即可
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;

# 重载授权表(一定要执行)
FLUSH PRIVILEGES;

# 退出
exit

# 重启
systemctl restart mysqld
```

`实际开发场景下,host的值建议严格一点,%表示通配符`

#### 2.4.8 端口开放

经过实测，使用阿里云或者腾讯云，在服务器上，无需配置 `iptables` 端口信息

但必须在阿里云或者腾讯云控制台 - 服务器 - 安全组，开放 3306端口。

至此，MySQL就安装完成并可用了，请妥善保存好MySQL的root密码。
#### 2.4.9 字符集相关设置

在MysQL8.0版本之前，默认字符集为1atin1，utf8字符集指向的是utf8mb3。网站开发人员在数据库设计的时候往往会将编码修改为utf8字符集。如果遗忘修改默认的编码，就会出现乱码的问题。

从MySQL8.0开始，数据库的默认编码将改为utf8mb4，从而避免上述乱码的问题。

```mysql
show variables like 'character%';
```

MySQL8.0的字符集
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6dc3eba5f4f812bb258a9eb091dea9cc.png)

MySQL5.7的字符集(**server和database默认的字符集是latin1**)
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c06fae55be38def7d23663503f8d438.png)


- `character_set_server`：服务器级别的字符集
- `character_set_database`：当前数据库的字符集
- character_set_client：服务器解码请求时使用的字符集
- character_set_connection：服务器处理请求时会把请求字符串从character_set_client转为character_set_connection 
- character_set_results：服务器向客户端返回数据时使用的字符集

小结
- 如果`创建或修改列`时没有显式的指定字符集和比较规则，则该列`默认用表的`字符集和比较规则
- 如果`创建表时`没有显式的指定字符集和比较规则，则该表`默认用数据库的`字符集和比较规则
- 如果`创建数据库时`没有显式的指定字符集和比较规则，则该数据库`默认用服务器的`字符集和比较规则

**请求到响应过程中字符集的变化**

```mermaid
graph TB
A(客户端) --> |"使用操作系统的字符集编码请求字符串"| B(从character_set_client转换为character_set_connection)
B --> C(从character_set_connection转换为具体的列使用的字符集)
C --> D(将查询结果从具体的列上使用的字符集转换为character_set_results)
D --> |"使用操作系统的字符集解码响应的字符串"| A

```

#### 2.4.10 修改mysql5.7的字符集
- 在window下, 我们修改的是my.ini文件;
- 在linux下,我们修改的是my.cf文件
```shell
vim /etc/my.cf
```
- 在该文件的最后加上中文字符集配置
```cf
character_set_server=utf8
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f4a0915ddf4520a3fc9b1bcce6c01e45.png)

- 最后重启一下mysql服务
```shell
systemctl restart mysqld.service
```

## 3 Tomcat

### 3.1 简介

Tomcat 是由 Apache 开发的一个 Servlet 容器，实现了对 Servlet 和 JSP 的支持，并提供了作为Web服务器的一些特有功能，如Tomcat管理和控制平台、安全域管理和Tomcat阀等。


简单来说，Tomcat是一个WEB应用程序的托管平台，可以让用户编写的WEB应用程序，被Tomcat所托管，并提供网站服务。

> 即让用户开发的WEB应用程序，变成可以被访问的网页。
### 3.2 安装

Tomcat的安装非常简单，主要分为2部分：

1. 安装JDK环境
2. 解压并安装Tomcat


> 本次安装使用Tomcat版本是：10.0.27版本，需要Java（JDK）版本最低为JDK8或更高版本
>
> 课程中使用的JDK版本是：JDK8u351版本

#### 3.2.1 安装JDK环境

1. 下载JDK软件

   https://www.oracle.com/java/technologies/downloads

   在页面下方找到：

   <img src="https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/17/20221017163411.png" alt="image-20221017163411651" style="zoom: 67%;" />

   下载`jdk-8u351-linux-x64.tar.gz`

   ![image-20221017163440491|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f88ebc3937a69377c7ac59079730138e.png)

   ==在弹出的页面中输入Oracle的账户密码即可下载（如无账户，请自行注册，注册是免费的）==

2. 登陆Linux系统，切换到root用户

   ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/471e483abf0f0a8de8656d6e3558fde6.png)

3. 通过FinalShell，上传下载好的JDK安装包

   ![image-20221017163706026|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/000728915290105aa584e5dab2243211.png)

4. 创建文件夹，用来部署JDK，将JDK和Tomcat都安装部署到：/export/server 内

   ```shell
   mkdir -p /export/server
   ```

5. 解压缩JDK安装文件

   ```shell
   tar -zxvf jdk-8u351-linux-x64.tar.gz -C /export/server
   ```

6. 配置JDK的软链接

   ```shell
   ln -s /export/server/jdk1.8.0_351 /export/server/jdk
   ```

7. 配置JAVA_HOME环境变量，以及将$JAVA_HOME/bin文件夹加入PATH环境变量中

   ```shell
   # 编辑/etc/profile文件
   export JAVA_HOME=/export/server/jdk
   export PATH=$PATH:$JAVA_HOME/bin
   ```

8. 生效环境变量

   ```shell
   source /etc/profile
   ```

9. 配置java执行程序的软链接

   ```shell
   # 删除系统自带的java程序
   rm -f /usr/bin/java
   # 软链接我们自己安装的java程序
   ln -s /export/server/jdk/bin/java /usr/bin/java
   ```

10. 执行验证：

    ```shell
    java -version
    javac -version
    ```



#### 3.2.2 解压并部署Tomcat

> Tomcat建议使用非Root用户安装并启动
>
> 可以创建一个用户：tomcat用以部署


1. 首先，放行tomcat需要使用的8080端口的外部访问权限

   > CentOS系统默认开启了防火墙，阻止外部网络流量访问系统内部
   >
   > 所以，如果想要Tomcat可以正常使用，需要对Tomcat默认使用的8080端口进行放行
   >
   > 放行有2种操作方式：
   >
   > 1. 关闭防火墙
   > 2. 配置防火墙规则，放行端口

   ```shell
   # 以下操作2选一即可
   # 方式1：关闭防火墙
   systemctl stop firewalld		# 关闭防火墙
   systemctl disable firewalld		# 停止防火墙开机自启
   
   # 方式2：放行8080端口的外部访问
   firewall-cmd --add-port=8080/tcp --permanent		# --add-port=8080/tcp表示放行8080端口的tcp访问，--permanent表示永久生效
   firewall-cmd --reload								# 重新载入防火墙规则使其生效
   ```

   > 方便起见，建议同学们选择方式1，直接关闭防火墙一劳永逸
   >
   > 防火墙的配置非常复杂，后面会视情况独立出一集防火墙配置规则的章节。

2. 以root用户操作，创建tomcat用户

   ```shell
   # 使用root用户操作
   useradd tomcat
   # 可选，为tomcat用户配置密码
   passwd tomcat
   ```

3. 下载Tomcat安装包

   ```shell
   # 使用root用户操作
   wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.27/bin/apache-tomcat-10.0.27.tar.gz
   # 如果出现https相关错误，可以使用--no-check-certificate选项
   wget --no-check-certificate https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.27/bin/apache-tomcat-10.0.27.tar.gz
	
	wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.93/bin/apache-tomcat-8.5.93.tar.gz
	wget --no-check-certificate https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.93/bin/apache-tomcat-8.5.93.tar.gz
   ```

   > 如果Linux内下载过慢，可以复制下载链接在Windows系统中使用迅雷等软件加速下载然后上传到Linux内即可
   >
   > 或者使用课程资料中提供的安装包

4. 解压Tomcat安装包

   ```shell
   # 使用root用户操作，否则无权限解压到/export/server内，除非修改此文件夹权限
   tar -zxvf apache-tomcat-10.0.27.tar.gz -C /export/server
   ```

5. 创建Tomcat软链接

   ```shell
   # 使用root用户操作
   ln -s /export/server/apache-tomcat-10.0.27 /export/server/tomcat
   ```

6. 修改tomcat安装目录权限

   ```shell
   # 使用root用户操作，同时对软链接和tomcat安装文件夹进行修改，使用通配符*进行匹配
   chown -R tomcat:tomcat /export/server/*tomcat*
   ```

7. 切换到tomcat用户

   ```shell
   su - tomcat
   ```

8. 启动tomcat

   ```shell
   /export/server/tomcat/bin/startup.sh
   ```

9. tomcat启动在8080端口，可以检查是否正常启动成功

   ```shell
   netstat -anp | grep 8080
   ```

   ![image-20221017223814737|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7affb2c0d60f83ce24fe6940e012077.png)

10. 打开浏览器，输入：

    http://centos:8080或http://192.168.88.130:8080

    使用主机名（需配置好本地的主机名映射）或IP地址访问Tomcat的WEB页面

    ![image-20221017223915498|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbc5ed0b7544a24e31ea429e7f4e0642.png)

至此，Tomcat安装配置完成。

## 4 四、Nginx

### 4.1 简介

*Nginx* (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

同Tomcat一样，Nginx可以托管用户编写的WEB应用程序成为可访问的网页服务，同时也可以作为流量代理服务器，控制流量的中转。

Nginx在WEB开发领域，基本上也是必备组件之一了。

### 4.2 安装

Nginx同样需要配置额外的yum仓库，才可以使用yum安装

> 安装Nginx的操作需要root身份

1. 安装yum依赖程序

   ```shell
   # root执行
   yum install -y yum-utils
   ```

2. 手动添加，nginx的yum仓库

   yum程序使用的仓库配置文件，存放在：`/etc/yum.repo.d`内。

   ```shell
   # root执行
   # 创建文件使用vim编辑
   vim /etc/yum.repos.d/nginx.repo
   # 填入如下内容并保存退出
   [nginx-stable]
   name=nginx stable repo
   baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=1
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true
   
   [nginx-mainline]
   name=nginx mainline repo
   baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true
   ```

   > 通过如上操作，我们手动添加了nginx的yum仓库

3. 通过yum安装最新稳定版的nginx

   ```shell
   # root执行
   yum install -y nginx
   ```

4. 启动

   ```shell
   # nginx自动注册了systemctl系统服务
   systemctl start nginx		# 启动
   systemctl stop nginx		# 停止
   systemctl status nginx		# 运行状态
   systemctl enable nginx		# 开机自启
   systemctl disable nginx		# 关闭开机自启
   ```

5. 配置防火墙放行

   nginx默认绑定80端口，需要关闭防火墙或放行80端口

   ```shell
   # 方式1（推荐），关闭防火墙
   systemctl stop firewalld		# 关闭
   systemctl disable firewalld		# 关闭开机自启
   
   # 方式2，放行80端口
   firewall-cmd --add-port=80/tcp --permanent		# 放行tcp规则下的80端口，永久生效
   firewall-cmd --reload							# 重新加载防火墙规则
   ```

6. 启动后浏览器输入Linux服务器的IP地址或主机名即可访问

   http://192.168.88.130 或 http://centos

   > ps：80端口是访问网站的默认端口，所以后面无需跟随端口号
   >
   > 显示的指定端口也是可以的比如：
   >
   > - http://192.168.88.130:80
   > - http://centos:80



至此，Nginx安装配置完成。

![image-20221018143113053|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7369969cc160c31724b7a2806afe4676.png)


## 5 RabbitMQ

### 5.1 简介

RabbitMQ一款知名的开源消息队列系统，为企业提供消息的发布、订阅、点对点传输等消息服务。

RabbitMQ在企业开发中十分常见，课程为大家演示快速搭建RabbitMQ环境。

### 5.2 安装

> rabbitmq在yum仓库中的版本比较老，所以我们需要手动构建yum仓库



1. 准备yum仓库

   ```shell
   # root执行
   # 1. 准备gpgkey密钥
   rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
   rpm --import https://packagecloud.io/rabbitmq/erlang/gpgkey
   rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
   
   # 2. 准备仓库文件
   vim /etc/yum.repos.d/rabbitmq.repo
   # 填入如下内容
   ##
   ## Zero dependency Erlang
   ##
   
   [rabbitmq_erlang]
   name=rabbitmq_erlang
   baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
   repo_gpgcheck=1
   gpgcheck=1
   enabled=1
   # PackageCloud's repository key and RabbitMQ package signing key
   gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
          https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   
   [rabbitmq_erlang-source]
   name=rabbitmq_erlang-source
   baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
   repo_gpgcheck=1
   gpgcheck=0
   enabled=1
   # PackageCloud's repository key and RabbitMQ package signing key
   gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
          https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   
   ##
   ## RabbitMQ server
   ##
   
   [rabbitmq_server]
   name=rabbitmq_server
   baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/$basearch
   repo_gpgcheck=1
   gpgcheck=0
   enabled=1
   # PackageCloud's repository key and RabbitMQ package signing key
   gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
          https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   
   [rabbitmq_server-source]
   name=rabbitmq_server-source
   baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/SRPMS
   repo_gpgcheck=1
   gpgcheck=0
   enabled=1
   gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   metadata_expire=300
   ```

3. 安装RabbitMQ

   ```shell
   # root执行
   yum install erlang rabbitmq-server -y
   ```
   
   ```shell
   Installed:
     erlang.x86_64 0:23.3.4.11-1.el7           rabbitmq-server.noarch 0:3.10.0-1.el7
   ```
   
4. 启动

   ```shell
   # root执行
   # 使用systemctl管控，服务名：rabbitmq-server
   systemctl enable rabbitmq-server		# 开机自启
   systemctl disable rabbitmq-server		# 关闭开机自启
   systemctl start rabbitmq-server			# 启动
   systemctl stop rabbitmq-server			# 关闭
   systemctl status rabbitmq-server		# 查看状态
   ```

5. 放行防火墙，RabbitMQ使用5672、15672、25672 3个端口

   ```shell
   # 方式1（推荐），关闭防火墙
   systemctl stop firewalld		# 关闭
   systemctl disable firewalld		# 关闭开机自启
   
   # 方式2，放行5672 25672端口
   firewall-cmd --add-port=5672/tcp --permanent		# 放行tcp规则下的5672端口，永久生效
   firewall-cmd --add-port=15672/tcp --permanent		# 放行tcp规则下的15672端口，永久生效
   firewall-cmd --add-port=25672/tcp --permanent		# 放行tcp规则下的25672端口，永久生效
   firewall-cmd --reload								# 重新加载防火墙规则
   ```

6. 启动RabbitMQ的WEB管理控制台

   ```shell
   rabbitmq-plugins enable rabbitmq_management
   ```

7. 添加admin用户，并赋予权限

   ```shell
   rabbitmqctl add_user admin 'Itheima66^'
   rabbitmqctl set_permissions -p "/" "admin" ".*" ".*" ".*"
   rabbitmqctl set_user_tags admin administrator
   ```

   

8. 浏览器打开管理控制台

   http://192.168.88.130:15672

   ![image-20221018154823983|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4703ea9428ca04aa20a1449824ba35a2.png)

至此，RabbitMQ已经安装完成了。

## 6 Redis安装部署【简单】

### 6.1 简介

redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。

redis的特点就是：`快`，可以基于内存存储数据并提供超低延迟、超快的检索速度

一般用于在系统中提供快速缓存的能力。

### 6.2 安装

参考文章：[3_环境搭建](../../../8_Java训练营（复习）/2_Redis复习笔记/1_Redis入门/3_环境搭建.md)

## 7 ElasticSearch安装部署

### 7.1 简介

[全文搜索](https://baike.baidu.com/item/全文搜索引擎)属于最常见的需求，开源的 [Elasticsearch](https://www.elastic.co/) （以下简称 es）是目前全文搜索引擎的首选。

它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 都采用它。

Elasticsearch简称es，在企业内同样是一款应用非常广泛的搜索引擎服务。

很多服务中的搜索功能，都是基于es来实现的。

### 7.2 安装

1. 添加yum仓库

   ```shell
   # root执行
   # 导入仓库密钥
   rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
   
   # 添加yum源
   # 编辑文件 
   vim /etc/yum.repos.d/elasticsearch.repo
   
   [elasticsearch-7.x]
   name=Elasticsearch repository for 7.x packages
   baseurl=https://artifacts.elastic.co/packages/7.x/yum
   gpgcheck=1
   gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
   enabled=1
   autorefresh=1
   type=rpm-md
   
   
   # 更新yum缓存
   yum makecache
   ```

2. 安装es

   ```shell
   yum install -y elasticsearch
   ```

3. 配置es

   ```shell
   vim /etc/elasticsearch/elasticsearch.yml
   
   # 17行，设置集群名称
   cluster.name: my-cluster
   
   # 23行，设置节点名称
   node.name: node-1
   
   # 56行，允许外网访问
   network.host: 0.0.0.0
   
   # 74行，配置集群master节点
   cluster.initial_master_nodes: ["node-1"]
   ```

4. 启动es

   ```shell
   systemctl start | stop | status | enable | disable elasticsearch
   ```

5. 关闭防火墙

   ```shell
   systemctl stop firewalld
   systemctl disable firewalld
   ```

6. 测试

   浏览器打开：http://ip:9200/?pretty

   ![image-20221025085432335|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/de38741b36888b2bfa7dce4735456b13.png)













