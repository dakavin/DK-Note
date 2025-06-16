---
文章标题: "[[2_RocketMQ安装]]"
文章作者: Dakkk
文章概要: |
  本文详细介绍了Apache RocketMQ在Linux上的两种安装部署方法：传统手动安装（涉及下载、配置、脚本修改）和基于Docker的容器化部署。文章涵盖NameServer、Broker及RocketMQ控制台的启动与配置，并提供常见问题指导，帮助用户快速搭建RocketMQ环境。
tags:
  - RocketMQ
  - 消息队列
  - 安装部署
  - Linux
  - Docker
  - NameServer
  - Broker
  - 控制台
相关文章:
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/5_容器/3_进入容器]]"
  - "[[../../../08-🛠️ 开发工具/04-☁️ 运维部署/02-🤖 自动化部署/3_姿势2：Docker部署项目]]"
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/3_CentOS 安装 Docker]]"
  - "[[../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/3_环境安装/4_Ubuntu 安装 Docker]]"
  - "[[5_进程间通信]]"
文章分类: 🔬 中间件技术
文章路径: 09-🔬 中间件技术/01-📨 消息队列/01-🚀 RocketMQ/2_RocketMQ安装.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:16:27
---

## 1 RocketMQ安装

了解了mq的基本概念和角色以后，我们开始安装rocketmq，建议在linux上

### 1.1 下载RocketMQ

下载地址：[https://rocketmq.apache.org/dowloading/releases/](https://rocketmq.apache.org/dowloading/releases/)

注意选择版本，这里我们选择4.9.2的版本，后面使用alibaba时对应
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/09259352b350be5b7fb77d42762af0f5.png)


下载地址：

[https://archive.apache.org/dist/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip](https://archive.apache.org/dist/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip)


- `注意安装RocketMQ主要有JAVA_HOME的环境变量`
### 1.2 上传服务器

在root目录下创建文件夹
```shell
mkdir rocketmq
```

将下载后的压缩包上传到阿里云服务器或者虚拟机中去
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1493887fc6fbe9f107ba81a742805ecb.png)

### 1.3 解压

```shell
unzip rocketmq-all-4.9.2-bin-release.zip
```

如果你的服务器没有unzip命令，则下载安装一个
```shell
yum install unzip
```

目录分析
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f54c239f0600ccf1a537ff9e9bd25ac.png)


`Benchmark`：包含一些性能测试的脚本；
`Bin`：可执行文件目录；
`Conf`：配置文件目录；
`Lib`：第三方依赖；
`LICENSE`：授权信息;
`NOTICE`：版本公告；
### 1.4 配置环境变量

```shell
vim /etc/profile
```

在文件末尾添加
```shell
export NAMESRV_ADDR=阿里云公网IP:9876
```
- aliyun公网可以换成本机 192.168.88.130 / localhost

使文件修改生效
```shell
source /etc/profile
```
### 1.5 修改nameServer的运行脚本

进入bin目录下，修改runserver.sh文件,将71行和76行的Xms和Xmx等改小一点
因为初识需要是4g，大于虚拟机的硬盘和内存大小了

```shell
vim runserver.sh
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f6e2e9c7c21e6a27c8add8cbb4d16238.png)

保存退出

### 1.6 修改broker的运行脚本

进入bin目录下，修改runbroker.sh文件,修改67行

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7af64156c1670e303c9aca5648bf524d.png)


保存退出

### 1.7 修改broker的配置文件

进入conf目录下，修改broker.conf文件
```shell
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH

namesrvAddr=localhost:9876

autoCreateTopicEnable=true

brokerIP1=阿里云公网IP
```

添加参数解释

`namesrvAddr`：nameSrv地址 可以写localhost因为nameSrv和broker在一个服务器

`autoCreateTopicEnable`：自动创建主题，不然需要手动创建出来

`brokerIP1`：broker也需要一个公网ip，如果不指定，那么是阿里云的内网地址，我们再本地无法连接使用，`如果是本地不能写localhost，而是写本地的ip地址(公网地址)`

### 1.8 启动

首先在安装目录下创建一个logs文件夹，用于存放日志

```shell
mkdir logs
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/87c8e40e1188afb20eb01d56f88ca8ab.png)

一次运行两条命令

在bin目录下启动nameSrv
```shell
nohup sh bin/mqnamesrv > logs/namesrv.log &
```

在bin目录下启动broker 这里的-c是指定使用的配置文件
```shell
nohup sh bin/mqbroker -c conf/broker.conf > ./logs/broker.log &
```

查看启动结果
`jps是查看java写的服务端的进程`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/912685530a1346b8e19ab5854acb5848.png)

### 1.9 关闭

关闭namesrv服务器
```shell
sh bin/mqshutdown namesrv
```

关闭broker服务
```shell
sh bin/mqshutdown broker
```

### 1.10 RocketMQ控制台的安装

Rocketmq 控制台可以可视化MQ的消息发送！

旧版本源码是在rocketmq-external里的rocketmq-console，新版本已经单独拆分成dashboard

网址： [https://github.com/apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)

下载地址：

[https://github.com/apache/rocketmq-dashboard/archive/refs/tags/rocketmq-dashboard-1.0.0.zip](https://github.com/apache/rocketmq-dashboard/archive/refs/tags/rocketmq-dashboard-1.0.0.zip)

下载后解压出来，在根目录下执行
```shell
mvn clean package -Dmaven.test.skip=true
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75871340ce31539c78057fe0725febeb.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4c4501110dfd17e795c6e1d3dd58f10.png)



将jar包上传到服务器上去
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9b4f8b446fddb8d866f2016f7efc84b.png)


然后运行

```shell
nohup java -jar ./dashboard --server.port=8001 rocketmq.config.namesrvAddr=127.0.0.1:9876 > ./rocketmq/logs/dashboard.log & 
```

`命令拓展:--server.port指定运行的端口`

--rocketmq.config.namesrvAddr=127.0.0.1:9876 指定namesrv地址

查看java服务进程
```shell
jps -l
```


访问： 192.168.88.130:8001

运行访问端口是8001，如果从官网拉下来打包的话，默认端口是8080

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7461cda316f590f6f3305249aa712d5d.png)


- `注意：`
	- 需要放行本机的额10100-11000之间的端口号
	- 需要检查内存是否足够，使用free -m 查看剩余内容够不够20%

## 2  RocketMQ安装之docker

### 2.1 下载RockerMQ需要的镜像

```shell
docker pull rocketmqinc/rocketmq

docker pull styletang/rocketmq-console-ng
```

### 2.2 启动NameServer服务

#### 2.2.1 创建NameServer数据存储路径

```java
mkdir -p /home/rocketmq/data/namesrv/logs /home/rocketmq/data/namesrv/store
```

#### 2.2.2 启动NameServer容器

```shell
docker run -d --name rmqnamesrv -p 9876:9876 -v /home/rocketmq/data/namesrv/logs:/root/logs -v /home/rocketmq/data/namesrv/store:/root/store -e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq sh mqnamesrv
```

### 2.3 启动Broker服务

#### 2.3.1 创建Broker数据存储路径

```shell
mkdir -p /home/rocketmq/data/broker/logs /home/rocketmq/data/broker/store
```

#### 2.3.2 创建conf配置文件目录

```shell
mkdir /home/rocketmq/conf
```

#### 2.3.3 在配置文件目录下创建broker.conf配置文件

```shell
# 所属集群名称，如果节点较多可以配置多个

brokerClusterName = DefaultCluster

#broker名称，master和slave使用相同的名称，表明他们的主从关系

brokerName = broker-a

#0表示Master，大于0表示不同的slave

brokerId = 0

#表示几点做消息删除动作，默认是凌晨4点

deleteWhen = 04

#在磁盘上保留消息的时长，单位是小时

fileReservedTime = 48

#有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；

brokerRole = ASYNC_MASTER

#刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；

flushDiskType = ASYNC_FLUSH

# 设置broker节点所在服务器的ip地址

brokerIP1 = 你服务器外网ip
```

#### 2.3.4 启动Broker容器

```shell
docker run -d  --name rmqbroker --link rmqnamesrv:namesrv -p 10911:10911 -p 10909:10909 -v  /home/rocketmq/data/broker/logs:/root/logs -v /home/rocketmq/data/broker/store:/root/store -v /home/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf --privileged=true -e "NAMESRV_ADDR=namesrv:9876" -e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf
```

### 2.4 启动控制台

```shell
docker run -d --name rmqadmin -e "JAVA_OPTS=-Drocketmq.namesrv.addr=**你的外网地址**:9876 \

-Dcom.rocketmq.sendMessageWithVIPChannel=false \

-Duser.timezone='Asia/Shanghai'" -v /etc/localtime:/etc/localtime -p 9999:8080 styletang/rocketmq-console-ng
```

### 2.5 正常启动后的docker ps

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6eb384657be265c74501f11d2f8370d2.png)

### 2.6 访问控制台

http://你的服务器外网ip:9999/

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0fcc59b75f3084a8170ada8fc6a174b3.png)


