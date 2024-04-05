## 1 集群化环境前置准备

### 1.1 介绍

在前面，我们所学习安装的软件，都是以单机模式运行的。

后续，我们将要学习大数据相关的软件部署，所以后续我们所安装的软件服务，大多数都是以集群化（多台服务器共同工作）模式运行的。


所以，在当前小节，我们需要完成集群化环境的前置准备，包括创建多台虚拟机，配置主机名映射，SSH免密登录等等。
### 1.2 部署

#### 1.2.1 配置多台Linux虚拟机

安装集群化软件，首要条件就是要有多台Linux服务器可用。

我们可以使用VMware提供的克隆功能，将我们的虚拟机额外克隆出3台来使用。

1. 首先，关机当前CentOS系统虚拟机（可以使用root用户执行`init 0`来快速关机）

2. 新建文件夹![image-20221025104157628|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7137992fe7cca8d2f800184da94d28b.png)

   文件夹起名为：`虚拟机集群`

3. 克隆![image-20221025104131303|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61961c2ea65382951a892e1436a626a9.png)

   ![image-20221025104312091|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5340b621dfdd01cb82d1e274081c2f1b.png)

   ![image-20221025104329109|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c975da69610bf8b7fe445ba3b1d64ace.png)

   ![image-20221025104345484|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d546c21ea4bfef789afdc2778cc61d3.png)

   ![image-20221025104414576|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57ddbe0400f1301ec738e76880d71cb3.png)

   ![image-20221025104427160|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/77e8e91f20fc0526a2c8c8ca9999e4ed.png)

   ![image-20221025104432927|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a3e50d7e959f51b4e8a4f06ccfe13df1.png)

   ![image-20221025104446044|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/144fb6ff63fed2658d08fc5d68ca1a6d.png)

4. 同样的操作克隆出：node2和node3

   ![image-20221025104825204|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2ad0f322fccea6fe2618be034753852f.png)

5. 开启node1，修改主机名为node1，并修改固定ip为：192.168.88.131

   ```shell
   # 修改主机名
   hostnamectl set-hostname node1
   
   # 修改IP地址
   vim /etc/sysconfig/network-scripts/ifcfg-ens33
   IPADDR="192.168.88.131"
   
   # 重启网卡
   systemctl stop network
   systemctl start network
   # 或者直接
   systemctl restart network
   ```

6. 同样的操作启动node2和node3,

   修改node2主机名为node2，设置ip为192.168.88.132

   修改node2主机名为node3，设置ip为192.168.88.133

7. 配置FinalShell，配置连接到node1、node2、node3的连接

   > 为了简单起见，建议配置root用户登录



#### 1.2.2 准备主机名映射

1. 在Windows系统中修改hosts文件，填入如下内容：

   > 如果同学们使用MacOS系统，请：
   >
   > 1. sudo su -，切换到root
   > 2. 修改/etc/hosts文件

   ```shell
   192.168.88.131 node1
   192.168.88.132 node2
   192.168.88.133 node3
   ```

2. 在3台Linux的/etc/hosts文件中，填入如下内容（==3台都要添加==）

   ```shell
   192.168.88.131 node1
   192.168.88.132 node2
   192.168.88.133 node3
   ```



#### 1.2.3 配置SSH免密登录

##### 1.2.3.1 简介

SSH服务是一种用于远程登录的安全认证协议。

我们通过FinalShell远程连接到Linux，就是使用的SSH服务。

SSH服务支持：

1. 通过账户+密码的认证方式来做用户认证
2. 通过账户+秘钥文件的方式做用户认证



SSH可以让我们通过SSH命令，远程的登陆到其它的主机上，比如：

在node1执行：ssh root@node2，将以root用户登录node2服务器，输入密码即可成功登陆

或者ssh node2，将以当前用户直接登陆到node2服务器。



##### 1.2.3.2 SSH免密配置

后续安装的集群化软件，多数需要远程登录以及远程执行命令，我们可以简单起见，配置三台Linux服务器之间的免密码互相SSH登陆

1. 在每一台机器都执行：`ssh-keygen -t rsa -b 4096`，一路回车到底即可

2. 在每一台机器都执行：

   ```shell
   ssh-copy-id node1
   ssh-copy-id node2
   ssh-copy-id node3
   ```

3. 执行完毕后，node1、node2、node3之间将完成root用户之间的免密互通



#### 1.2.4 配置JDK环境

后续的大数据集群软件，多数是需要Java运行环境的，所以我们为==每一台==机器都配置JDK环境。



JDK配置参阅：`Tomcat`安装部署环节。



#### 1.2.5 关闭防火墙和SELinux

集群化软件之间需要通过端口互相通讯，为了避免出现网络不通的问题，我们可以简单的在集群内部关闭防火墙。

==在每一台机器都执行==

```shell
systemctl stop firewalld
systemctl disable firewalld
```



Linux有一个安全模块：SELinux，用以限制用户和程序的相关权限，来确保系统的安全稳定。

SELinux的配置同防火墙一样，非常复杂，课程中不多涉及，后续视情况可以出一章SELinux的配置课程。

在当前，我们只需要关闭SELinux功能，避免导致后面的软件运行出现问题即可，

==在每一台机器都执行==

```shell
vim /etc/sysconfig/selinux

# 将第七行，SELINUX=enforcing 改为
SELINUX=disabled
# 保存退出后，重启虚拟机即可，千万要注意disabled单词不要写错，不然无法启动系统
```

#### 1.2.6 添加快照

为了避免后续出现问题，在完成上述设置后，为==每一台虚拟机==都制作快照，留待使用。
### 1.3 补充命令 - scp

后续的安装部署操作，我们将会频繁的在多台服务器之间相互传输数据。

为了更加方面的互相传输，我们补充一个命令：scp

scp命令是cp命令的升级版，即：ssh cp，通过SSH协议完成文件的复制。

其主要的功能就是：在不同的Linux服务器之间，通过`SSH`协议互相传输文件。

只要知晓服务器的账户和密码（或密钥），即可通过SCP互传文件。


语法：
```shell
scp [-r] 参数1 参数2
- -r选项用于复制文件夹使用，如果复制文件夹，必须使用-r
- 参数1：本机路径 或 远程目标路径
- 参数2：远程目标路径 或 本机路径

如：
scp -r /export/server/jdk root@node2:/export/server/
将本机上的jdk文件夹， 以root的身份复制到node2的/export/server/内
同SSH登陆一样，账户名可以省略（使用本机当前的同名账户登陆）

如：
scp -r node2:/export/server/jdk /export/server/
将远程node2的jdk文件夹，复制到本机的/export/server/内


# scp命令的高级用法
cd /export/server
scp -r jdk node2:`pwd`/    # 将本机当前路径的jdk文件夹，复制到node2服务器的同名路径下
scp -r jdk node2:$PWD      # 将本机当前路径的jdk文件夹，复制到node2服务器的同名路径下
```

## 2 Zookeeper集群安装部署

### 2.1 简介

ZooKeeper是一个[分布式](https://baike.baidu.com/item/分布式/19276232?fromModule=lemma_inlink)的，开放源码的[分布式应用程序](https://baike.baidu.com/item/分布式应用程序/9854429?fromModule=lemma_inlink)协调服务，是Hadoop和[Hbase](https://baike.baidu.com/item/Hbase/7670213?fromModule=lemma_inlink)的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

除了为Hadoop和HBase提供协调服务外，Zookeeper也被其它许多软件采用作为其分布式状态一致性的依赖，比如Kafka，又或者一些软件项目中，也经常能见到Zookeeper作为一致性协调服务存在。

Zookeeper不论是大数据领域亦或是其它服务器开发领域，涉及到分布式状态一致性的场景，总有它的身影存在。

### 2.2 安装

Zookeeper是一款分布式的集群化软件，可以在多台服务器上部署，并协同组成分布式集群一起工作。



1. 首先，要确保已经完成了`集群化环境前置准备`环节的全部内容

2. 【node1上操作】下载Zookeeper安装包，并解压

   ```shell
   # 下载
   wget http://archive.apache.org/dist/zookeeper/zookeeper-3.5.9/apache-zookeeper-3.5.9-bin.tar.gz
   
   # 确保如下目录存在，不存在就创建
   mkdir -p /export/server
   
   # 解压
   tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz -C /export/server
   ```

3. 【node1上操作】创建软链接

   ```shell
   ln -s /export/server/apache-zookeeper-3.5.9 /export/server/zookeeper
   ```

4. 【node1上操作】修改配置文件

   ```shell
   vim /export/server/zookeeper/conf/zoo.cfg
   
   tickTime=2000
   # zookeeper数据存储目录
   dataDir=/export/server/zookeeper/data
   clientPort=2181
   initLimit=5
   syncLimit=2
   server.1=node1:2888:3888
   server.2=node2:2888:3888
   server.3=node3:2888:3888
   ```

5. 【node1上操作】配置`myid`

   ```shell
   # 1. 创建Zookeeper的数据目录
   mkdir /export/server/zookeeper/data
   
   # 2. 创建文件，并填入1
   vim /export/server/zookeeper/data/myid
   # 在文件内填入1即可
   ```

6. 【在node2和node3上操作】，创建文件夹

   ```shell
   mkdir -p /export/server
   ```

7. 【node1上操作】将Zookeeper 复制到node2和node3

   ```shell
   cd /export/server
   
   scp -r apache-zookeeper-3.5.9 node2:`pwd`/
   scp -r apache-zookeeper-3.5.9 node3:`pwd`/
   ```

8. 【在node2上操作】

   ```shell
   # 1. 创建软链接
   ln -s /export/server/apache-zookeeper-3.5.9 /export/server/zookeeper
   
   # 2. 修改myid文件
   vim /export/server/zookeeper/data/myid
   # 修改内容为2
   ```

9. 【在node3上操作】

   ```shell
   # 1. 创建软链接
   ln -s /export/server/apache-zookeeper-3.5.9 /export/server/zookeeper
   
   # 2. 修改myid文件
   vim /export/server/zookeeper/data/myid
   # 修改内容为3
   ```

10. 【在node1、node2、node3上分别执行】启动Zookeeper

    ```shell
    # 启动命令
    /export/server/zookeeper/bin/zkServer.sh start		# 启动Zookeeper
    ```

11. 【在node1、node2、node3上分别执行】检查Zookeeper进程是否启动

    ```shell
    jps
    
    # 结果中找到有：QuorumPeerMain 进程即可
    ```

12. 【node1上操作】验证Zookeeper

    ```shell
    /export/server/zookeeper/zkCli.sh
    
    # 进入到Zookeeper控制台中后，执行
    ls /
    
    # 如无报错即配置成功
    ```

至此Zookeeper安装完成

## 3 Kafka集群安装部署

### 3.1 简介

Kafka是一款`分布式的、去中心化的、高吞吐低延迟、订阅模式`的消息队列系统。

同RabbitMQ一样，Kafka也是消息队列。不过RabbitMQ多用于后端系统，因其更加专注于消息的延迟和容错。

Kafka多用于大数据体系，因其更加专注于数据的吞吐能力。

Kafka多数都是运行在分布式（集群化）模式下，所以课程将以3台服务器，来完成Kafka集群的安装部署。

### 3.2 安装

1. 确保已经跟随前面的视频，安装并部署了JDK和Zookeeper服务
   > Kafka的运行依赖JDK环境和Zookeeper请确保已经有了JDK环境和Zookeeper

2. 【在node1操作】下载并上传Kafka的安装包
   ```shell
   # 下载安装包
   wget http://archive.apache.org/dist/kafka/2.4.1/kafka_2.12-2.4.1.tgz
   ```

3. 【在node1操作】解压
   ```shell
   mkdir -p /export/server			# 此文件夹如果不存在需先创建
   
   # 解压
   tar -zxvf kafka_2.12-2.4.1.tgz -C /export/server/
   
   # 创建软链接
   ln -s /export/server/kafka_2.12-2.4.1 /export/server/kafka
   ```

4. 【在node1操作】修改Kafka目录内的config目录内的`server.properties`文件

   ````shell
   cd /export/server/kafka/config
   # 指定broker的id
   broker.id=1
   # 指定 kafka的绑定监听的地址
   listeners=PLAINTEXT://node1:9092
   # 指定Kafka数据的位置
   log.dirs=/export/server/kafka/data
   # 指定Zookeeper的三个节点
   zookeeper.connect=node1:2181,node2:2181,node3:2181
   ````

5. 【在node1操作】将node1的kafka复制到node2和node3

   ```shell
   cd /export/server
   
   # 复制到node2同名文件夹
   scp -r kafka_2.12-2.4.1 node2:`pwd`/
   # 复制到node3同名文件夹
   scp -r kafka_2.12-2.4.1 node3:$PWD
   ```

6. 【在node2操作】

   ```shell
   # 创建软链接
   ln -s /export/server/kafka_2.12-2.4.1 /export/server/kafka
   
   cd /export/server/kafka/config
   # 指定broker的id
   broker.id=2
   # 指定 kafka的绑定监听的地址
   listeners=PLAINTEXT://node2:9092
   # 指定Kafka数据的位置
   log.dirs=/export/server/kafka/data
   # 指定Zookeeper的三个节点
   zookeeper.connect=node1:2181,node2:2181,node3:2181
   ```

7. 【在node3操作】

   ```shell
   # 创建软链接
   ln -s /export/server/kafka_2.12-2.4.1 /export/server/kafka
   
   cd /export/server/kafka/config
   # 指定broker的id
   broker.id=3
   # 指定 kafka的绑定监听的地址
   listeners=PLAINTEXT://node3:9092
   # 指定Kafka数据的位置
   log.dirs=/export/server/kafka/data
   # 指定Zookeeper的三个节点
   zookeeper.connect=node1:2181,node2:2181,node3:2181
   ```

8. 启动kafka

   ```shell
   # 请先确保Zookeeper已经启动了
   
   # 方式1：【前台启动】分别在node1、2、3上执行如下语句
   /export/server/kafka/bin/kafka-server-start.sh /export/server/kafka/config/server.properties
   
   # 方式2：【后台启动】分别在node1、2、3上执行如下语句
   nohup /export/server/kafka/bin/kafka-server-start.sh /export/server/kafka/config/server.properties 2>&1 >> /export/server/kafka/kafka-server.log &
   ```

9. 验证Kafka启动

   ```shell
   # 在每一台服务器执行
   jps
   ```

   ![image-20221025174522487|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7331c3c3ca3fe71e62296736ae5a86e1.png)
### 3.3 测试Kafka能否正常使用

1. 创建测试主题
```shell
# 在node1执行，创建一个主题
/export/server/kafka_2.12-2.4.1/bin/kafka-topics.sh --create --zookeeper node1:2181 --replication-factor 1 --partitions 3 --topic test
```

2. 运行测试，请在FinalShell中打开2个node1的终端页面
```shell
# 打开一个终端页面，启动一个模拟的数据生产者
/export/server/kafka_2.12-2.4.1/bin/kafka-console-producer.sh --broker-list node1:9092 --topic test
# 再打开一个新的终端页面，在启动一个模拟的数据消费者
/export/server/kafka_2.12-2.4.1/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic test --from-beginning
```

## 4 大数据集群（Hadoop生态）安装部署

### 4.1 简介

1）Hadoop是一个由Apache基金会所开发的分布式系统基础架构。
2）主要解决，海量数据的存储和海量数据的分析计算问题。

Hadoop HDFS 提供分布式海量数据存储能力

Hadoop YARN 提供分布式集群资源管理能力

Hadoop MapReduce 提供分布式海量数据计算能力

#### 4.1.1 前置要求

- 请确保完成了集群化环境前置准备章节的内容
- 即：JDK、SSH免密、关闭防火墙、配置主机名映射等前置操作

#### 4.1.2 Hadoop集群角色

Hadoop生态体系中总共会出现如下进程角色：

1. Hadoop HDFS的管理角色：Namenode进程（`仅需1个即可（管理者一个就够）`）
2. Hadoop HDFS的工作角色：Datanode进程（`需要多个（工人，越多越好，一个机器启动一个）`）
3. Hadoop YARN的管理角色：ResourceManager进程（`仅需1个即可（管理者一个就够）`）
4. Hadoop YARN的工作角色：NodeManager进程（`需要多个（工人，越多越好，一个机器启动一个）`）
5. Hadoop 历史记录服务器角色：HistoryServer进程（`仅需1个即可（功能进程无需太多1个足够）`）
6. Hadoop 代理服务器角色：WebProxyServer进程（`仅需1个即可（功能进程无需太多1个足够）`）
7. Zookeeper的进程：QuorumPeerMain进程（`仅需1个即可（Zookeeper的工作者，越多越好）`）

#### 4.1.3 角色和节点分配

角色分配如下：
1. node1:Namenode、Datanode、ResourceManager、NodeManager、HistoryServer、WebProxyServer、QuorumPeerMain
2. node2:Datanode、NodeManager、QuorumPeerMain
3. node3:Datanode、NodeManager、QuorumPeerMain

![image-20221026202935745|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cd3874192a2a9d13af4f500655c825cf.png)

### 4.2 安装

#### 4.2.1 调整虚拟机内存

如上图，可以看出node1承载了太多的压力。同时node2和node3也同时运行了不少程序

为了确保集群的稳定，需要对虚拟机进行内存设置。

请在VMware中，对：

1. node1设置4GB或以上内存
2. node2和node3设置2GB或以上内存


> 大数据的软件本身就是集群化（一堆服务器）一起运行的。
>
> 现在我们在一台电脑中以多台虚拟机来模拟集群，确实会有很大的内存压力哦。

#### 4.2.2 Zookeeper集群部署

略

#### 4.2.3 Hadoop集群部署

1. 下载Hadoop安装包、解压、配置软链接
   ```shell
   # 1. 下载
   wget http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz
   
   # 2. 解压
   # 请确保目录/export/server存在
   tar -zxvf hadoop-3.3.0.tar.gz -C /export/server/
   
   # 3. 构建软链接
   ln -s /export/server/hadoop-3.3.0 /export/server/hadoop
   ```

2. 修改配置文件：`hadoop-env.sh`

   > Hadoop的配置文件要修改的地方很多，请细心

   cd 进入到/export/server/hadoop/etc/hadoop，文件夹中，配置文件都在这里

   修改hadoop-env.sh文件

   > 此文件是配置一些Hadoop用到的环境变量
   >
   > 这些是临时变量，在Hadoop运行时有用
   >
   > 如果要永久生效，需要写到/etc/profile中
   ```shell
   # 在文件开头加入：
   # 配置Java安装路径
   export JAVA_HOME=/export/server/jdk
   # 配置Hadoop安装路径
   export HADOOP_HOME=/export/server/hadoop
   # Hadoop hdfs配置文件路径
   export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
   # Hadoop YARN配置文件路径
   export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
   # Hadoop YARN 日志文件夹
   export YARN_LOG_DIR=$HADOOP_HOME/logs/yarn
   # Hadoop hdfs 日志文件夹
   export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hdfs
   
   # Hadoop的使用启动用户配置
   export HDFS_NAMENODE_USER=root
   export HDFS_DATANODE_USER=root
   export HDFS_SECONDARYNAMENODE_USER=root
   export YARN_RESOURCEMANAGER_USER=root
   export YARN_NODEMANAGER_USER=root
   export YARN_PROXYSERVER_USER=root
   ```

3. 修改配置文件：`core-site.xml`

   如下，清空文件，填入如下内容
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <!--
     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
   
       http://www.apache.org/licenses/LICENSE-2.0
   
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License. See accompanying LICENSE file.
   -->
   
   <!-- Put site-specific property overrides in this file. -->
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://node1:8020</value>
       <description></description>
     </property>
   
     <property>
       <name>io.file.buffer.size</name>
       <value>131072</value>
       <description></description>
     </property>
   </configuration>
   ```

4. 配置：`hdfs-site.xml`文件
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <!--
     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
   
       http://www.apache.org/licenses/LICENSE-2.0
   
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License. See accompanying LICENSE file.
   -->
   
   <!-- Put site-specific property overrides in this file. -->
   
   <configuration>
       <property>
           <name>dfs.datanode.data.dir.perm</name>
           <value>700</value>
       </property>
   
     <property>
       <name>dfs.namenode.name.dir</name>
       <value>/data/nn</value>
       <description>Path on the local filesystem where the NameNode stores the namespace and transactions logs persistently.</description>
     </property>
   
     <property>
       <name>dfs.namenode.hosts</name>
       <value>node1,node2,node3</value>
       <description>List of permitted DataNodes.</description>
     </property>
   
     <property>
       <name>dfs.blocksize</name>
       <value>268435456</value>
       <description></description>
     </property>
   
   
     <property>
       <name>dfs.namenode.handler.count</name>
       <value>100</value>
       <description></description>
     </property>
   
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>/data/dn</value>
     </property>
   </configuration>
   ```

5. 配置：`mapred-env.sh`文件
   ```shell
   # 在文件的开头加入如下环境变量设置
   export JAVA_HOME=/export/server/jdk
   export HADOOP_JOB_HISTORYSERVER_HEAPSIZE=1000
   export HADOOP_MAPRED_ROOT_LOGGER=INFO,RFA
   ```

6. 配置：`mapred-site.xml`文件
   ```xml
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <!--
     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
   
       http://www.apache.org/licenses/LICENSE-2.0
   
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License. See accompanying LICENSE file.
   -->
   
   <!-- Put site-specific property overrides in this file. -->
   
   <configuration>
     <property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
       <description></description>
     </property>
   
     <property>
       <name>mapreduce.jobhistory.address</name>
       <value>node1:10020</value>
       <description></description>
     </property>
   
   
     <property>
       <name>mapreduce.jobhistory.webapp.address</name>
       <value>node1:19888</value>
       <description></description>
     </property>
   
   
     <property>
       <name>mapreduce.jobhistory.intermediate-done-dir</name>
       <value>/data/mr-history/tmp</value>
       <description></description>
     </property>
   
   
     <property>
       <name>mapreduce.jobhistory.done-dir</name>
       <value>/data/mr-history/done</value>
       <description></description>
     </property>
   <property>
     <name>yarn.app.mapreduce.am.env</name>
     <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
   </property>
   <property>
     <name>mapreduce.map.env</name>
     <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
   </property>
   <property>
     <name>mapreduce.reduce.env</name>
     <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
   </property>
   </configuration>
   ```

7. 配置：`yarn-env.sh`文件

   ```shell
   # 在文件的开头加入如下环境变量设置
   export JAVA_HOME=/export/server/jdk
   export HADOOP_HOME=/export/server/hadoop
   export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
   export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
   export YARN_LOG_DIR=$HADOOP_HOME/logs/yarn
   export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hdfs
   ```

8. 配置：`yarn-site.xml`文件
   ```xml
   <?xml version="1.0"?>
   <!--
     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
   
       http://www.apache.org/licenses/LICENSE-2.0
   
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License. See accompanying LICENSE file.
   -->
   <configuration>
   
   <!-- Site specific YARN configuration properties -->
   <property>
       <name>yarn.log.server.url</name>
       <value>http://node1:19888/jobhistory/logs</value>
       <description></description>
   </property>
   
     <property>
       <name>yarn.web-proxy.address</name>
       <value>node1:8089</value>
       <description>proxy server hostname and port</description>
     </property>
   
   
     <property>
       <name>yarn.log-aggregation-enable</name>
       <value>true</value>
       <description>Configuration to enable or disable log aggregation</description>
     </property>
   
     <property>
       <name>yarn.nodemanager.remote-app-log-dir</name>
       <value>/tmp/logs</value>
       <description>Configuration to enable or disable log aggregation</description>
     </property>
   
   
   <!-- Site specific YARN configuration properties -->
     <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>node1</value>
       <description></description>
     </property>
   
     <property>
       <name>yarn.resourcemanager.scheduler.class</name>
       <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
       <description></description>
     </property>
   
     <property>
       <name>yarn.nodemanager.local-dirs</name>
       <value>/data/nm-local</value>
       <description>Comma-separated list of paths on the local filesystem where intermediate data is written.</description>
     </property>
   
   
     <property>
       <name>yarn.nodemanager.log-dirs</name>
       <value>/data/nm-log</value>
       <description>Comma-separated list of paths on the local filesystem where logs are written.</description>
     </property>
   
   
     <property>
       <name>yarn.nodemanager.log.retain-seconds</name>
       <value>10800</value>
       <description>Default time (in seconds) to retain log files on the NodeManager Only applicable if log-aggregation is disabled.</description>
     </property>
   
   
   
     <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
       <description>Shuffle service that needs to be set for Map Reduce applications.</description>
     </property>
   </configuration>
   ```

9. 修改workers文件
   ```shell
   # 全部内容如下
   node1
   node2
   node3
   ```

10. 分发hadoop到其它机器
   ```shell
   # 在node1执行
   cd /export/server
   
   scp -r hadoop-3.3.0 node2:`pwd`/
   scp -r hadoop-3.3.0 node2:`pwd`/
   ```

11. 在node2、node3执行
    ```shell
    # 创建软链接
    ln -s /export/server/hadoop-3.3.0 /export/server/hadoop
    ```

12. 创建所需目录

    - 在node1执行：
      ```shell
      mkdir -p /data/nn
      mkdir -p /data/dn
      mkdir -p /data/nm-log
      mkdir -p /data/nm-local
      ```

    - 在node2执行：
      ```shell
      mkdir -p /data/dn
      mkdir -p /data/nm-log
      mkdir -p /data/nm-local
      ```

    - 在node3执行：
      ```shell
      mkdir -p /data/dn
      mkdir -p /data/nm-log
      mkdir -p /data/nm-local
      ```

13. 配置环境变量

    在node1、node2、node3修改/etc/profile

    ```shell
    export HADOOP_HOME=/export/server/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    ```

    执行`source /etc/profile`生效

14. 格式化NameNode，在node1执行

    ```shell
    hadoop namenode -format
    ```

    > hadoop这个命令来自于：$HADOOP_HOME/bin中的程序
    >
    > 由于配置了环境变量PATH，所以可以在任意位置执行hadoop命令哦

15. 启动hadoop的hdfs集群，在node1执行即可

    ```shell
    start-dfs.sh
    
    # 如需停止可以执行
    stop-dfs.sh
    ```

    > start-dfs.sh这个命令来自于：$HADOOP_HOME/sbin中的程序
    >
    > 由于配置了环境变量PATH，所以可以在任意位置执行start-dfs.sh命令哦

16. 启动hadoop的yarn集群，在node1执行即可

    ```shell
    start-yarn.sh
    
    # 如需停止可以执行
    stop-yarn.sh
    ```

17. 启动历史服务器

    ```shell
    mapred --daemon start historyserver
    
    # 如需停止将start更换为stop
    ```

18. 启动web代理服务器

    ```shell
    yarn-daemon.sh start proxyserver
    
    # 如需停止将start更换为stop
    ```



##### 4.2.3.1 验证Hadoop集群运行情况

1. 在node1、node2、node3上通过jps验证进程是否都启动成功

2. 验证HDFS，浏览器打开：http://node1:9870

   创建文件test.txt，随意填入内容，并执行：

   ```shell
   hadoop fs -put test.txt /test.txt
   
   hadoop fs -cat /test.txt
   ```

3. 验证YARN，浏览器打开：http://node1:8088

   执行：

   ```shell
   # 创建文件words.txt，填入如下内容
   itheima itcast hadoop
   itheima hadoop hadoop
   itheima itcast
   
   # 将文件上传到HDFS中
   hadoop fs -put words.txt /words.txt
   
   # 执行如下命令验证YARN是否正常
   hadoop jar /export/server/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.0.jar wordcount -Dmapred.job.queue.name=root.root /words.txt /output
   ```



## 5 大数据NoSQL数据库HBase集群部署

### 5.1 简介

HBase 是一种[分布式](https://so.csdn.net/so/search?q=分布式&spm=1001.2101.3001.7020)、可扩展、支持海量数据存储的 NoSQL 数据库。



和Redis一样，HBase是一款KeyValue型存储的数据库。

不过和Redis设计方向不同

- Redis设计为少量数据，超快检索
- HBase设计为海量数据，快速检索

HBase在大数据领域应用十分广泛，现在我们来在node1、node2、node3上部署HBase集群。

### 5.2 安装

1. HBase依赖Zookeeper、JDK、Hadoop（HDFS），请确保已经完成前面

   - 集群化软件前置准备（JDK）
   - Zookeeper
   - Hadoop
   - 这些环节的软件安装

2. 【node1执行】下载HBase安装包

   ```shell
   # 下载
   wget http://archive.apache.org/dist/hbase/2.1.0/hbase-2.1.0-bin.tar.gz
   
   # 解压
   tar -zxvf hbase-2.1.0-bin.tar.gz -C /export/server
   
   # 配置软链接
   ln -s /export/server/hbase-2.1.0 /export/server/hbase
   ```

3. 【node1执行】，修改配置文件，修改`conf/hbase-env.sh`文件

   ```shell
   # 在28行配置JAVA_HOME
   export JAVA_HOME=/export/server/jdk
   # 在126行配置：
   # 意思表示，不使用HBase自带的Zookeeper，而是用独立Zookeeper
   export HBASE_MANAGES_ZK=false
   # 在任意行，比如26行，添加如下内容：
   export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"
   ```

4. 【node1执行】，修改配置文件，修改`conf/hbase-site.xml`文件

   ```shell
   # 将文件的全部内容替换成如下内容：
   <configuration>
           <!-- HBase数据在HDFS中的存放的路径 -->
           <property>
               <name>hbase.rootdir</name>
               <value>hdfs://node1:8020/hbase</value>
           </property>
           <!-- Hbase的运行模式。false是单机模式，true是分布式模式。若为false,Hbase和Zookeeper会运行在同一个JVM里面 -->
           <property>
               <name>hbase.cluster.distributed</name>
               <value>true</value>
           </property>
           <!-- ZooKeeper的地址 -->
           <property>
               <name>hbase.zookeeper.quorum</name>
               <value>node1,node2,node3</value>
           </property>
           <!-- ZooKeeper快照的存储位置 -->
           <property>
               <name>hbase.zookeeper.property.dataDir</name>
               <value>/export/server/apache-zookeeper-3.6.0-bin/data</value>
           </property>
           <!--  V2.1版本，在分布式情况下, 设置为false -->
           <property>
               <name>hbase.unsafe.stream.capability.enforce</name>
               <value>false</value>
           </property>
   </configuration>
   ```

5. 【node1执行】，修改配置文件，修改`conf/regionservers`文件

   ```shell
   # 填入如下内容
   node1
   node2
   node3
   ```

6. 【node1执行】，分发hbase到其它机器

   ```shell
   scp -r /export/server/hbase-2.1.0 node2:/export/server/
   scp -r /export/server/hbase-2.1.0 node3:/export/server/
   ```

7. 【node2、node3执行】，配置软链接

   ```shell
   ln -s /export/server/hbase-2.1.0 /export/server/hbase
   ```

8. 【node1、node2、node3执行】，配置环境变量

   ```shell
   # 配置在/etc/profile内，追加如下两行
   export HBASE_HOME=/export/server/hbase
   export PATH=$HBASE_HOME/bin:$PATH
   
   source /etc/profile
   ```

9. 【node1执行】启动HBase

   > 请确保：Hadoop HDFS、Zookeeper是已经启动了的

   ```shell
   start-hbase.sh
   
   # 如需停止可使用
   stop-hbase.sh
   ```

   > 由于我们配置了环境变量export PATH=$PATH:$HBASE_HOME/bin
   >
   > start-hbase.sh即在$HBASE_HOME/bin内，所以可以无论当前目录在哪，均可直接执行

10. 验证HBase

    浏览器打开：http://node1:16010，即可看到HBase的WEB UI页面

11. 简单测试使用HBase

    【node1执行】

    ```shell
    hbase shell
    
    # 创建表
    create 'test', 'cf'
    
    # 插入数据
    put 'test', 'rk001', 'cf:info', 'itheima'
    
    # 查询数据
    get 'test', 'rk001'
    
    # 扫描表数据
    scan 'test'
    ```


## 6 分布式内存计算Spark环境部署

### 6.1 注意

本小节的操作，基于：`大数据集群（Hadoop生态）安装部署`环节中所构建的Hadoop集群

如果没有Hadoop集群，请参阅前置内容，部署好环境。

### 6.2 简介

Spark是一款分布式内存计算引擎，可以支撑海量数据的分布式计算。

Spark在大数据体系是明星产品，作为最新一代的综合计算引擎，支持离线计算和实时计算。

在大数据领域广泛应用，是目前世界上使用最多的大数据分布式计算引擎。

我们将基于前面构建的Hadoop集群，部署Spark Standalone集群。

### 6.3 安装

1. 【node1执行】下载并解压

   ```shell
   wget https://archive.apache.org/dist/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz
   
   # 解压
   tar -zxvf spark-2.4.5-bin-hadoop2.7.tgz -C /export/server/
   
   # 软链接
   ln -s /export/server/spark-2.4.5-bin-hadoop2.7 /export/server/spark
   ```

2. 【node1执行】修改配置文件名称
   ```shell
   # 改名
   cd /export/server/spark/conf
   mv spark-env.sh.template spark-env.sh
   mv slaves.template slaves
   ```

3. 【node1执行】修改配置文件，`spark-env.sh`
   ```shell
   ## 设置JAVA安装目录
   JAVA_HOME=/export/server/jdk
   
   ## HADOOP软件配置文件目录，读取HDFS上文件和运行YARN集群
   HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
   YARN_CONF_DIR=/export/server/hadoop/etc/hadoop
   
   ## 指定spark老大Master的IP和提交任务的通信端口
   export SPARK_MASTER_HOST=node1
   export SPARK_MASTER_PORT=7077
   
   SPARK_MASTER_WEBUI_PORT=8080
   SPARK_WORKER_CORES=1
   SPARK_WORKER_MEMORY=1g
   ```

4. 【node1执行】修改配置文件，`slaves`
   ```shell
   node1
   node2
   node3
   ```

5. 【node1执行】分发
   ```shell
   scp -r spark-2.4.5-bin-hadoop2.7 node2:$PWD
   scp -r spark-2.4.5-bin-hadoop2.7 node3:$PWD
   ```

6. 【node2、node3执行】设置软链接
   ```shell
   ln -s /export/server/spark-2.4.5-bin-hadoop2.7 /export/server/spark
   ```

7. 【node1执行】启动Spark集群
   ```shell
   /export/server/spark/sbin/start-all.sh
   
   # 如需停止，可以
   /export/server/spark/sbin/stop-all.sh
   ```

8. 打开Spark监控页面，浏览器打开：http://node1:8081

9. 【node1执行】提交测试任务
   ```shell
   /export/server/spark/bin/spark-submit --master spark://node1:7077 --class org.apache.spark.examples.SparkPi /export/server/spark/examples/jars/spark-examples_2.11-2.4.5.jar
   ```

## 7 分布式内存计算Flink环境部署

### 7.1 注意

本小节的操作，基于：`大数据集群（Hadoop生态）安装部署`环节中所构建的Hadoop集群

如果没有Hadoop集群，请参阅前置内容，部署好环境。
### 7.2 简介

Flink同Spark一样，是一款分布式内存计算引擎，可以支撑海量数据的分布式计算。

Flink在大数据体系同样是明星产品，作为最新一代的综合计算引擎，支持离线计算和实时计算。

在大数据领域广泛应用，是目前世界上除去Spark以外，应用最为广泛的分布式计算引擎。

我们将基于前面构建的Hadoop集群，部署Flink Standalone集群

Spark更加偏向于离线计算而Flink更加偏向于实时计算。

### 7.3 安装

1. 【node1操作】下载安装包
   ```shell
   wget https://archive.apache.org/dist/flink/flink-1.10.0/flink-1.10.0-bin-scala_2.11.tgz
   
   # 解压
   tar -zxvf flink-1.10.0-bin-scala_2.11.tgz -C /export/server/
   
   # 软链接
   ln -s /export/server/flink-1.10.0 /export/server/flink
   ```

2. 【node1操作】修改配置文件，`conf/flink-conf.yaml`
   ```yaml
   # jobManager 的IP地址
   jobmanager.rpc.address: node1
   # JobManager 的端口号
   jobmanager.rpc.port: 6123
   # JobManager JVM heap 内存大小
   jobmanager.heap.size: 1024m
   # TaskManager JVM heap 内存大小
   taskmanager.heap.size: 1024m
   # 每个 TaskManager 提供的任务 slots 数量大小
   taskmanager.numberOfTaskSlots: 2
   #是否进行预分配内存，默认不进行预分配，这样在我们不使用flink集群时候不会占用集群资源
   taskmanager.memory.preallocate: false
   # 程序默认并行计算的个数
   parallelism.default: 1
   #JobManager的Web界面的端口（默认：8081）
   jobmanager.web.port: 8081
   ```

3. 【node1操作】，修改配置文件，`conf/slaves`
   ```shell
   node1
   node2
   node3
   ```

4. 【node1操作】分发Flink安装包到其它机器
   ```shell
   cd /export/server
   scp -r flink-1.10.0 node2:`pwd`/
   scp -r flink-1.10.0 node3:`pwd`/
   ```

5. 【node2、node3操作】
   ```shell
   # 配置软链接
   ln -s /export/server/flink-1.10.0 /export/server/flink
   ```

6. 【node1操作】，启动Flink
   ```shell
   /export/server/flink/bin/start-cluster.sh
   ```

7. 验证Flink启动
   ```shell
   # 浏览器打开
   http://node1:8081
   ```

8. 提交测试任务
   【node1执行】
   ```shell
   /export/server/flink/bin/flink run /export/server/flink-1.10.0/examples/batch/WordCount.jar
   ```

## 8 运维监控Zabbix部署

### 8.1 简介

Zabbix 由 Alexei Vladishev 创建，目前由其成立的公司—— Zabbix SIA 积极的持续开发更新维护， 并为用户提供技术支持服务。

Zabbix 是一个==企业级分布式开源监控解决方案==。

Zabbix 软件能够==监控==众多网络参数和服务器的==健康度、完整性==。Zabbix 使用灵活的告警机制，允许用户为几乎任何事件配置基于邮件的告警。这样用户可以快速响应服务器问题。Zabbix 基于存储的数据提供出色的报表和数据可视化功能。这些功能使得 Zabbix 成为容量规划的理想选择。

### 8.2 安装

>  安装整体步骤:

1. 准备Linux 服务器(虚拟机)
2. 安装Mysql
3. 安装zabbix( 包含 server  agent  web)
4. 配置 mysql, 为zabbix创建表结构
5. 配置zabbix server
6. 启动并开启开机自启动

![[Pasted image 20230909174137.png]]

#### 8.2.1 安装前准备 - MySql

安装ZabbixServer需要先安装好`Mysql`数据库

课程使用`Mysql 5.7`

安装步骤：
```shell
# 安装Mysql yum库
rpm -Uvh http://repo.mysql.com//mysql57-community-release-el7-7.noarch.rpm

# yum安装Mysql
yum -y install mysql-community-server

# 启动Mysql设置开机启动
systemctl start mysqld
systemctl enable mysqld

# 检查Mysql服务状态
systemctl status mysqld

# 第一次启动mysql，会在日志文件中生成root用户的一个随机密码，使用下面命令查看该密码
grep 'temporary password' /var/log/mysqld.log

# 修改root用户密码
mysql -u root -p -h localhost
Enter password:
 
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root!@#$';

# 如果你想设置简单密码，需要降低Mysql的密码安全级别
set global validate_password_policy=LOW; # 密码安全级别低
set global validate_password_length=4;	 # 密码长度最低4位即可

# 然后就可以用简单密码了（课程中使用简单密码，为了方便，生产中不要这样）
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
mysql> grant all privileges on *.* to root@'%' identified by 'root';
```

#### 8.2.2 安装Zabbix Server 和 Zabbix Agent

> 初始安装，我们先安装ZabbixServer以及在Server本机安装Agent。

打开官网下载页面：[https://www.zabbix.com/download?zabbix=4.0&os_distribution=centos&os_version=7&db=mysql](https://www.zabbix.com/download?zabbix=4.0&os_distribution=centos&os_version=7&db=mysql)

![[Pasted image 20230909174244.png]]

选择对应的版本，然后再下面官网给出了具体的安装命令，使用`rpm`和`yum`来进行安装。

需要有网络。

##### 8.2.2.1 安装Zabbix yum库

[documentation](https://www.zabbix.com/documentation/4.0/manual/installation/install_from_packages)

```shell
rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
yum clean all
```

##### 8.2.2.2 安装Zabbix Server、前端、Agent

```shell
yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent  
# 如果只需要安装Agent的话  
yum -y install zabbix-agent
```

##### 8.2.2.3 初始化Mysql数据库

[documentation](https://www.zabbix.com/documentation/4.0/manual/appendix/install/db_scripts)

> 在Mysql中操作

```shell
# 登录Mysql 数据库  
mysql -uroot -pYourPassword  
mysql> create database zabbix character set utf8 collate utf8_bin;  
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';  
# 或者: grant all privileges on zabbix.* to zabbix@'%' identified by 'zabbix';  
mysql> quit;
```

测试在Zabbix Server服务器上能否远程登录Mysql，如果可以登录继续向下走。

Import initial schema and data. You will be prompted to enter your newly created password.

```shell
# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

##### 8.2.2.4 d. 为Zabbix Server配置数据库

Edit file /etc/zabbix/zabbix_server.conf

```shell
DBPassword=password  
DBHost=mysql-host-ip-or-hostname
```

##### 8.2.2.5 e. 配置Zabbix的PHP前端

Edit file `/etc/httpd/conf.d/zabbix.conf`, uncomment and set the right timezone for you.`# php_value date.timezone Asia/Shanghai`

Start Zabbix server and agent processes and make it start at system boot:

```shell
systemctl restart zabbix-server zabbix-agent httpd # 启动、重启  
systemctl enable zabbix-server zabbix-agent httpd  # 开机自启
```

Now your Zabbix server is up and running!

#### 8.2.3 配置zabbix 前端（WEB UI）

**打开:`http://192.168.88.131/zabbix`**

即可进入Zabbix页面，在首次打开的时候，会进入设置页面，如图：![[Pasted image 20230909174603.png]]
**点击下一步，会检查相应的设置是否都正常**![[Pasted image 20230909174614.png]]
如果一切正常，点击下一步。

**配置DB连接**![[Pasted image 20230909174626.png]]
按具体情况填写即可

**配置Server细节**![[Pasted image 20230909174638.png]]
具体配置即可，Name表示这个Zabbix服务的名字，这里起名叫`ITHEIMA-TEST`

**安装前总结预览**

检查确认没有问题就下一步![[Pasted image 20230909174650.png]]
**配置完成**![[Pasted image 20230909174702.png]]
**初始管理员账户Admin密码zabbix**

输入账户密码后，就能进入zabbix页面了。

如下图：![[Pasted image 20230909174716.png]]
现在是一个崭新的zabbix等待我们去探索。

## 9 运维监控Grafana部署

### 9.1 简介

### 9.2 安装

#### 9.2.1 部署形式

`Grafana`支持两种部署形式

1. 自行部署, 可以部署在操作系统之上. 自行提供服务器, 域名等.
   
2. `Grafana`官方托管. 无需安装, 在线注册即可得到一个专属于自己的`Grafana`, 但是要花钱的. 是一种`SaaS`服务
   

我们课程选择方式1

#### 9.2.2 安装

`Grafana`支持常见的绝大多数操作系统, 如`windows` `mac` `linux` 同时也支持部署在`docker`中.

大多数情况下, `Grafana`都是部署在`linux`服务器之上. 所以本课程也是基于`Linux`系统来讲解.

对`windows` `mac`系统 或 `docker`部署有兴趣的同学, 请参考: [https://grafana.com/grafana/download](https://grafana.com/grafana/download)

我们部署`Grafana`可以使用`YUM`来进行部署.

```shell
# 创建一个文件  
vim /etc/yum.repos.d/grafana.repo  
​  
# 将下面的内容复制进去  
[grafana]  
name=grafana  
baseurl=https://packages.grafana.com/oss/rpm  
repo_gpgcheck=1  
enabled=1  
gpgcheck=1  
gpgkey=https://packages.grafana.com/gpg.key  
sslverify=1  
sslcacert=/etc/pki/tls/certs/ca-bundle.crt  
​  
# 最后安装  
yum install grafana
```

#### 9.2.3 配置说明

`grafana-server`具有许多配置选项，这些选项可以在`.ini`配置文件中指定，也可以使用环境变量指定。

> **Note.** `Grafana` needs to be restarted for any configuration changes to take effect.

##### 9.2.3.1 配置文件注释

`;`符号在`.ini`文件中全局表示注释 ()

##### 9.2.3.2 配置文件路径

如果是自己解压安装, 或者自行编译的方式安装, 配置文件在:

- 默认: `$WORKING_DIR/conf/defaults.ini`
  
- 自定义:`$WORKING_DIR/conf/custom.ini`
  
- 自定义配置文件路径可以被参数`--config`覆盖
  

> 对于`YUM` `RPM` 安装的方式, 配置文件在: `/etc/grafana/grafana.ini`

##### 9.2.3.3 使用环境变量

可以使用以下语法使用环境变量来覆盖配置文件中的所有选项：

```bash
GF_<SectionName>_<KeyName>
```

其中`SectionName`是方括号内的文本。一切都应为大写，`.`应替换为`_` 例如，给定以下配置设置：

```shell
# default section
instance_name = ${HOSTNAME}

[security]
admin_user = admin

[auth.google]
client_secret = 0ldS3cretKey
```

Then you can override them using:

```shell
export GF_DEFAULT_INSTANCE_NAME=my-instance
export GF_SECURITY_ADMIN_USER=true	# GF_ 固定 SECURITY 是SectionName ADMIN_USER 是配置的key 转大写 . 转 _
export GF_AUTH_GOOGLE_CLIENT_SECRET=newS3cretKey
```

#### 9.2.4 开始配置

`Grafana`支持使用`Sqlite3` `Postgresql` `Mysql`这三种数据库作为其`元数据`的存储.

我们课程使用`Mysql`. 和`zabbix`的元数据mysql共用一个实例

只需要配置如下内容即可:![[Pasted image 20230909175045.png]]
并登陆mysql, 执行:

`create database grafana CHARACTER SET utf8 COLLATE utf8_general_ci;`

创建`Grafana`使用的数据库作为元数据存储.

#### 9.2.5 启动

```shell
systemctl daemon-reload
systemctl start grafana-server
systemctl enable grafana-server
```

浏览器打开：http://node1:3000

默认账户密码：admin/admin