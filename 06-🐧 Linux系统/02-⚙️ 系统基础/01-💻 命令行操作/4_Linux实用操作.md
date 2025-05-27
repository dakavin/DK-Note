---
文章标题: "[[4_Linux实用操作]]" 
文章作者: Dakkk
文章概要: |
  AI未能生成概要。
tags:
  - "未分类"
相关文章: 暂无 🤷
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/02-⚙️ 系统基础/01-💻 命令行操作/4_Linux实用操作.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:19:51
---

## 1 各类小技巧

- `ctrl + c 强制停止`
	- Linux某些程序的运行，如果想要强制停止它，可以使用快捷键ctrl + c![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e0920442f62fe94f3f2a2253c61b36f.png)


	- 命令输入错误，也可以通过快捷键ctrl + c，退出当前输入，重新输入![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1de9a8a5a54e40fac14cad7eaec734e5.png)


- `ctrl + d 退出或登出`
	- 可以通过快捷键：ctrl + d，退出账户的登录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/391b31fede24e13693e92c315364d743.png)


	- 或者退出某些特定程序的专属页面![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d94a8e00dd4b984c79a34a96c4e83730.png)


	- `不能用于退出vi/vim`

- `历史命令搜索`
	- 可以通过history命令，查看历史输入过的命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88b11c850d1be2f16e9146a6fafbe478.png)


	- 可以通过：!命令前缀，自动执行上一次匹配前缀的命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/da021b39a2aae0558d6def18d9250dc7.png)


	- 可以通过快捷键：ctrl + r，输入内容去匹配历史命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2fe67f75f411c86df058beb1f7de3a19.png)


		- 如果搜索到的内容是你需要的，那么：
			- 回车键可以直接执行
			- 键盘左右键，可以得到此命令（不执行）

- `光标移动快捷键`
	- ctrl + a，跳到命令开头
	- ctrl + e，跳到命令结尾
	- ctrl + 键盘左键，向左跳一个单词
	- ctrl + 键盘右键，向右跳一个单词

- `清屏`
	- 通过快捷键ctrl + l，可以清空终端内容
	- 或通过命令clear得到同样效果
## 2 软件安装

- `Linux系统的应用商店`
- 操作系统安装软件有许多种方式，一般分为：
	- 下载安装包自行安装
		- 如win系统使用exe文件、msi文件等
		- 如mac系统使用dmg文件、pkg文件等
	- 系统的应用商店内安装
		- 如win系统有Microsoft Store商店
		- 如mac系统有AppStore商店
- Linux系统同样支持这两种方式，我们首先，先来学习使用：`Linux命令行内的”应用商店”，yum命令安装软件`

- `yum命令`
- yum：RPM包软件管理器，用于自动化安装配置Linux软件，并可以自动解决依赖问题
- 语法：`yum [-y] [install | remove | search] 软件名称`
	- 选项：-y，自动确认，无需手动确认安装或卸载过程
	- install：安装
	- remove：卸载
	- search：搜索
- yum命令需要root权限哦，可以su切换到root，或使用sudo提权
- yum命令需要联网

- `yum [-y] install wget`， 通过yum命令安装wget程序![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7041b2f332d148851113fd2d8c7b2fc.png)


- yum [-y] remove wget，通过yum命令卸载wget命令
- yum search wget，通过yum命令，搜索是否有wget安装包

- `拓展：Ubuntu的软件安装`
- CentOS系统和Ubuntu是使用不同的包管理器
- CentOS使用yum管理器，`Ubuntu使用apt管理器`
- 通过前面学习的WSL环境，我们可以得到Ubuntu运行环境
- 语法：`apt [-y] [install remove search] 软件名称`
- 用法和yum一致，同样需要root权限
## 3 systemctl

- Linux系统很多软件（内置或第三方）均支持使用systemctl命令控制：启动、停止、开机自启
- 能够被systemctl管理的软件，一般也称之为：服务
- 语法：`systemctl start | stop | status | enable | disable 服务名`
	- enable和disable涉及开关机启动

- 系统内置的服务比较多，比如：
	- NetworkManager，主网络服务
	- network，副网络服务
	- firewalld，防火墙服务
	- sshd，ssh服务（FinalShell远程登录Linux使用的就是这个服务）

- 除了内置的服务以外，部分第三方软件安装后也可以以systemctl进行控制
	- yum install -y ntp，安装ntp软件
	- 可以通过`ntpd服务名`，配合systemctl进行控制

	- yum install -y httpd，安装apache服务器软件
	- 可以通过httpd服务名，配合systemctl进行控制

- `部分软件安装后没有自动集成到systemctl中，我们可以手动添加`
## 4 软连接

- `ln命令创建软连接`
- 在系统中创建软链接，可以将文件、文件夹链接到其它位置
- 类似Windows系统中的`《快捷方式》`
- 语法：`ln -s 参数1 参数2`
	- -s选项，创建软连接
	- 参数1：被链接的文件或文件夹
	- 参数2：要链接去的目的地
- 实例：
	- ln -s /etc/yum.conf ~/yum.conf
	- ln -s /etc/yum ~/yum![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/417b71e682fa21bba98713838bfd082c.png)


## 5 日期、时区

- `date命令`
- 通过date命令可以在命令行中查看系统的时间
- 语法：`date [-d] [+格式化字符串]`
	- -d 按照给定的字符串显示日期，一般用于日期计算
	- 格式化字符串：通过特定的字符串标记，来控制显示的日期格式
		- %Y   年
		- %y   年份后两位数字 (00..99)
		- %m   月份 (01..12)
		- %d   日 (01..31)
		- %H   小时 (00..23)
		- %M   分钟 (00..59)
		- %S   秒 (00..60)
		- %s   自 1970-01-01 00:00:00 UTC 到现在的秒数

- 使用date命令本体，无选项，直接查看时间![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c9cf4d1402a45d3342bc5bddd36bffcd.png)


- 可以看到这个格式非常的不习惯。我们可以通过格式化字符串自定义显示格式

- 按照2022-01-01的格式显示日期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3634ddc7ace4683769093a75d99fddae.png)


- 按照2022-01-01 10:00:00的格式显示日期![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/63049fd071c15351e3aaed4af1c7d914.png)


- 如上，`由于中间带有空格，所以使用双引号包围格式化字符串，作为整体`

- `date命令进行日期加减`
- -d选项，可以按照给定的字符串显示日期，一般用于日期计算![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1fb889b9c760fedff1d12f320c3053e7.png)


- 其中支持的时间标记为：
	- year年
	- month月
	- day天
	- hour小时
	- minute分钟
	- second秒
- -d选项可以和 格式化字符串配合一起使用哦

- `修改Linux时区`
- 通过date查看的日期时间是不准确的，这是因为：系统默认时区非中国的东八区
- 使用root权限，执行如下命令，修改时区为东八区时区![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7bca65075a50d77c3688f7258605722.png)


- 将系统自带的localtime文件删除，并将/usr/share/zoneinfo/Asia/Shanghai文件链接为localtime文件即可

- `ntp程序`
- 我们可以通过ntp程序自动校准系统时间
- 安装ntp：yum -y install ntp
- 启动并设置开机自启：
	- systemctl start ntpd
	- systemctl enable ntpd
- 当ntpd启动后会定期的帮助我们联网校准系统的时间
- 也可以手动校准（需root权限）：`ntpdate -u ntp.aliyun.com`
- 通过阿里云提供的服务网址配合ntpdate（安装ntp后会附带这个命令）命令自动校准![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e14d340c0317bbbfd525a48404a16552.png)


## 6 IP地址、主机名

### 6.1 IP地址

- `IP地址`
- 每一台联网的电脑都会有一个地址，用于和其它计算机进行通讯
- IP地址主要有2个版本，V4版本和V6版本（V6很少用，课程暂不涉及）
- IPv4版本的地址格式是：a.b.c.d，其中abcd表示0~255的数字，如192.168.88.101就是一个标准的IP地址

- 可以通过命令：ifconfig，查看本机的ip地址，如无法使用ifconfig命令，可以安装：yum -y install net-tools![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb48a06adce1be9cbf57bf8ae08a8e7f.png)



- `特殊IP地址`
- 除了标准的IP地址以外，还有几个特殊的IP地址需要我们了解：
	- 127.0.0.1，这个IP地址用于指代本机![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e86b03cd38580dd977c5e2a8df080f9e.png)


	- 0.0.0.0，特殊IP地址
		- 可以用于指代本机
		- 可以在端口绑定中用来确定绑定关系（后续讲解）
		- 在一些IP地址限制中，表示所有IP的意思，如放行规则设置为0.0.0.0，表示允许任意IP访问

- `主机名`
- 每一台电脑除了对外联络地址（IP地址）以外，也可以有一个名字，称之为主机名
- 无论是Windows或Linux系统，都可以给系统设置主机名
	- Windows系统主机名![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56e1b7b4727c9f63297dd6a3acfe7d72.png)


	- Linux系统主机名![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48f283b70862e6bf3066ff43df96efa7.png)



- `在Linux中修改主机名`
- 可以使用命令：hostname查看主机名![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48f283b70862e6bf3066ff43df96efa7.png)


- 可以使用命令：hostnamectl set-hostname 主机名，修改主机名（需root）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9e2eb4502d33ebda6f637b58f6da682.png)


- 重新登录FinalShell即可看到主机名已经正确显示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a34e6250ee96430011bcd6d4be8b216.png)



- `域名解析`
- IP地址实在是难以记忆，有没有什么办法可以通过主机名或替代的字符地址去代替数字化的IP地址呢
- 实际上，我们一直都是`通过字符化的地址去访问服务器，很少指定IP地址`
- 比如，我们在浏览器内打开：www.baidu.com，会打开百度的网址，其中，www.baidu.com，是百度的网址，我们称之为：域名

- 访问www.baidu.com的流程如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a0bf2366442b6f0578fefe63f84d4474.png)


- 先查看本机的记录（私人地址本）
	- Windows看：`C:\Windows\System32\drivers\etc\hosts`
	- Linux看：`/etc/hosts`
- 再联网去DNS服务器（如114.114.114.114，8.8.8.8等）询问

- `配置主机名映射`
- 比如，我们FinalShell是通过IP地址连接到的Linux服务器，那有没有可能通过域名（主机名）连接呢？
- 可以，我们只需要在Windows系统的：`C:\Windows\System32\drivers\etc\hosts`文件中配置记录即可![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/334900dd4af18a363903cfea8f95eddd.png)


- 之后使用finalshell的时候，就可以`不用写linux的ip地址，直接写主机名称即可`

### 6.2 虚拟机配置固定IP

- `为什么需要固定IP`
- 当前我们虚拟机的Linux操作系统，其IP地址是通过DHCP服务获取的
- DHCP：***动态获取IP地址，即每次重启设备后都会获取一次，可能导致IP地址频繁变更***
- 原因1：办公电脑IP地址变化无所谓，但是我们要远程连接到Linux系统，如果IP地址经常变化我们就要频繁修改适配很麻烦
- 原因2：在刚刚我们配置了虚拟机IP地址和主机名的映射，如果IP频繁更改，我们也需要频繁更新映射关系

- `在VMware Workstation中配置固定IP`
	- 配置固定IP需要2个大步骤：
		1. 在VMware Workstation（或Fusion）中配置IP地址网关和网段（IP地址的范围）
		2. 在Linux系统中手动修改配置文件，固定IP
	- 步骤如下：
	- 打开VM中的编辑，找到虚拟网络编辑器，然后![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13f22f7c56b7db5654a47b274fbab640.png)


	- 上述1、2、3完成后，点击Nat设置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0d790e73f648a79d90f167d8b8c700c.png)


	- 使用vim编辑/etc/sysconfig/network-scripts/ifcfg-ens33文件，填入如下内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a70ad2fbbd3aa0f8dd71cb2a7edd07e8.png)


	- 执行：systemctl restart network 重启网卡，执行ifconfig即可看到ip地址固定为192.168.88.130了

## 7 网络传输

### 7.1 下载和网络请求

- `ping命令`
- 可以通过ping命令，`检查指定的网络服务器是否是可联通状态`
- 语法：`ping [-c num] ip或主机名`
	- 选项：-c，检查的次数，不使用-c选项，将无限次数持续检查
	- 参数：ip或主机名，被检查的服务器的ip地址或主机名地址
- 示例：
	- 检查到baidu.com是否联通![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6cb2bac63fb6be2a685ab7ee94e05110.png)


	- 结果表示联通，延迟8ms左右
	- 检查到39.156.66.10是否联通，并检查3次![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aef91f08fd497c8949663a21967a2dfa.png)



- `wget命令`
- wget是非交互式的文件下载器，可以在命令行内下载网络文件
- 语法：`wget [-b] url`
	- 选项：-b，可选，后台下载，会将日志写入到当前工作目录的wget-log文件
	- 参数：url，下载链接
- 示例：
	- 下载apache-hadoop 3.3.0版本：`wget http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c6a31a587bd57d03e7ce66d0ff68c578.png)
	- 在后台下载：`wget -b http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz`
	- 通过tail命令可以监控后台下载进度：tail -f wget-log
- 注意：`无论下载是否完成，都会生成要下载的文件，如果下载未完成，请及时清理未完成的不可用文件`

- `curl命令`
- curl可以发送http网络请求，可用于：下载文件、获取信息等
- 语法：`curl [-O] url`
	- 选项：-O，用于下载文件，当url是下载链接时，可以使用此选项保存文件
	- 参数：url，要发起请求的网络地址
- 示例：
	- 向cip.cc发起网络请求：curl cip.cc![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/453a4bf58172ccb5496f18d42eabe28d.png)


	- 向python.itheima.com发起网络请求：curl python.itheima.com
	- 通过curl下载hadoop-3.3.0安装包：`curl -O http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz`

### 7.2 端口

- 端口，是`设备与外界通讯交流的出入口`。端口可以分为：物理端口和虚拟端口两类
- `物理端口`：又可称之为接口，是可见的端口，如USB接口，RJ45网口，HDMI端口等
- `虚拟端口`：是指计算机内部的端口，是不可见的，是用来操作系统和外部进行交互使用的

- 虚拟端口，有什么用？为什么需要它呢？
- 通过IP地址即可![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12294769a3d87c151489f486b52a5773.png)

- 计算机程序之间的通讯，通过IP只能锁定计算机，但是无法锁定具体的程序
- `通过端口可以锁定计算机上具体的程序，确保程序之间进行沟通`
- IP地址相当于小区地址，在小区内可以有许多住户（程序），而门牌号（端口）就是各个住户（程序）的联系地址

- Linux系统是一个超大号小区，可以支持65535个端口，这6万多个端口分为3类进行使用：
	- 公认端口：1~1023，通常用于一些系统内置或知名程序的预留使用，如SSH服务的22端口，HTTPS服务的443端口
		- 非特殊需要，不要占用这个范围的端口
	- 注册端口：1024~49151，通常可以随意使用，用于松散的绑定一些程序或服务

- `查看端口占用`
- 可以通过Linux命令去查看端口的占用情况
	- 使用nmap命令，安装nmap：yum -y install nmap
- 语法：`nmap 被查看的IP地址`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d01d73605b868d1733add4208f4ebb1f.png)


- 可以看到，本机（127.0.0.1）上有5个端口现在被程序占用了。
- 其中：
	- 22端口，一般是SSH服务使用，即FinalShell远程连接Linux所使用的端口

- `可以通过netstat命令，查看指定端口的占用情况`
- 语法：netstat -anp | grep 端口号
	- 安装netstat：yum -y install net-tools![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28931894fc4fa87cb5704d581a85d7aa.png)


- 如图，可以看到当前系统6000端口被程序（进程号7174）占用了
- 其中，0.0.0.0:6000，表示端口绑定在0.0.0.0这个IP地址上，表示允许外部访问![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80b895cbaede5cdb64dbdd61d9844269.png)

- 可以看到，当前系统12345端口，无人使用哦。

## 8 进程管理

- `进程`
- 程序运行在操作系统中，是被操作系统所管理的。
- 为管理运行的程序，每一个程序在运行的时候，便被操作系统注册为系统中的一个：进程
- 并会为每一个进程都分配一个独有的：进程ID（进程号）
	- Windows系统任务管理器![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/556e74c31ea5f8785f44213b4d1b0ca5.png)


	- Linux系统查看进程
	- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf736a8e039889d339a2d7749201342f.png)


- `查看进程`
- 可以通过ps命令查看Linux系统中的进程信息
- 语法：`ps [-e -f]`
	- 选项：-e，显示出全部的进程
	- 选项：-f，以完全格式化的形式展示信息（展示全部信息）
- 一般来说，固定用法就是： ps -ef 列出全部进程的全部信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c70ba7b8fc5e43539a5627d95273ff61.png)


- 从左到右分别是：
	- UID：进程所属的用户ID
	- PID：进程的进程号ID
	- PPID：进程的父ID（启动此进程的其它进程）
	- C：此进程的CPU占用率（百分比）
	- STIME：进程的启动时间
	- TTY：启动此进程的终端序号，如显示?，表示非终端启动
	- TIME：进程占用CPU的时间
	- CMD：进程对应的名称或启动路径或启动命令

- `查看指定进程`
- 在FinalShell中，执行命令：tail，可以看到，此命令一直阻塞在那里
- 在FinalShell中，复制一个标签页，执行：ps -ef 找出tail这个程序的进程信息
- 问题：是否会发现，列出的信息太多，无法准确的找到或很麻烦怎么办？
- 我们可以使用管道符配合grep来进行过滤，如：
- ps -ef | grep tail，即可准确的找到tail命令的信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3acc0bda25bf61f87e5945ccca6b5759.png)


- 过滤不仅仅过滤名称，进程号，用户ID等等，都可以被grep过滤哦
- •如：ps -ef | grep 30001，过滤带有30001关键字的进程信息（一般指代过滤30001进程号）

- `关闭进程`
- 在Windows系统中，可以通过任务管理器选择进程后，点击结束进程从而关闭它
- 同样，在Linux中，可以通过kill命令关闭进程。
- 语法：`kill [-9] 进程ID`
	- 选项：-9，表示强制关闭进程。不使用此选项会向进程发送信号要求其关闭，但是否关闭看进程自身的处理机制![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8232b111220a6680d4fa7839f1fb998.png)


## 9 主机状态

- `查看系统资源占用`
- 可以通过top命令查看CPU、内存使用情况，类似Windows的任务管理器
- 默认`每5秒刷新`一次，语法：`直接输入top即可，按q或ctrl + c退出`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9872a8b8485cece60365a6d41ad0c4b6.png)



- `top命令内容详解`
- `第一行`：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e10e6c9b351d8a6d6ab24ad11d57329d.png)
	- top：命令名称，14:39:58：当前系统时间，up 6 min：启动了6分钟，2 users：2个用户登录，load：1、5、15分钟负载
- `第二行`：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4112d905b59e9c04d3766c583e46d24f.png)
	- Tasks：175个进程，1 running：1个进程子在运行，174 sleeping：174个进程睡眠，0个停止进程，0个僵尸进程
- `第三行`:![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28be7afb663a794ad1a5a4d317bb82aa.png)
	- %Cpu(s)：CPU使用率，`us：用户CPU使用率，sy：系统CPU使用率`，ni：高优先级进程占用CPU时间百分比，id：空闲CPU率，wa：IO等待CPU占用率，hi：CPU硬件中断率，si：CPU软件中断率，st：强制等待占用CPU率
- `第四、五行`:![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/161f116575711c3b278663a93f654956.png)
	- Kib Mem：物理内存，total：总量，`free：空闲，used：使用`，buff/cache：buff和cache占用
	- KibSwap：虚拟内存（交换空间），total：总量，free：空闲，used：使用，buff/cache：buff和cache占用

- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a2403d82810699bd0a9d60e945156104.png)
	- PID：进程id
	- USER：进程所属用户
	- PR：进程优先级，越小越高
	- NI：负值表示高优先级，正表示低优先级
	- VIRT：进程使用虚拟内存，单位KB
	- RES：进程使用物理内存，单位KB
	- SHR：进程使用共享内存，单位KB
	- S：进程状态（S休眠，R运行，Z僵死状态，N负数优先级，I空闲状态）
	- %CPU：进程占用CPU率
	- %MEM：进程占用内存率
	- TIME+：进程使用CPU时间总计，单位10毫秒
	- COMMAND：进程的命令或名称或程序文件路径

- `top命令选项`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fbfe24c5aa544e183f4b269b8deb0336.png)
- `top交互式选项`
- 当top以交互式运行（非-b选项启动），可以用以下交互式命令进行控制![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/30d859ced94ff4f791a4f7039fa15289.png)
- `磁盘信息监控`
- 使用df命令，可以查看硬盘的使用情况
- 语法：`df [-h]`
	- 选项：-h，以更加人性化的单位显示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5300073a641f314d7a459bf0a4e496a0.png)
- 可以使用iostat查看CPU、磁盘的相关信息
- 语法：`iostat [-x] [num1] [num2]`
	- 选项：-x，显示更多信息
	- num1：数字，刷新间隔，num2：数字，刷新几次![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ff6d5f4207009cc557506974d9c70bf0.png)
- tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）
- "一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的

- `使用iostat的-x选项，可以显示更多信息`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/698a24dea953be88257c29f36c892c19.png)
	- rrqm/s：  每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge, 提高IO利用率, 避免重复调用）；
	- wrqm/s：  每秒这个设备相关的写入请求有多少被Merge了。
	- rsec/s：  每秒读取的扇区数；sectors
	- wsec/：  每秒写入的扇区数。
	- `rKB/s：  每秒发送到设备的读取请求数`
	- `wKB/s：  每秒发送到设备的写入请求数`
	- avgrq-sz   平均请求扇区的大小
	- avgqu-sz   平均请求队列的长度。毫无疑问，队列长度越短越好。   
	- await：    每一个IO请求的处理的平均时间（单位是微秒毫秒）。
	- svctm      表示平均每次设备I/O操作的服务时间（以毫秒为单位）
	- `%util：   磁盘利用率`

- `网络状态监控`
- 可以使用sar命令查看网络的相关统计（sar命令非常复杂，这里仅简单用于统计网络）
- 语法：`sar -n DEV num1 num2`
	- 选项：-n，查看网络，DEV表示查看网络接口
	- num1：刷新间隔（不填就查看一次结束），num2：查看次数（不填无限次数）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7bd94a1782574f5754f0f342f47ccfca.png)
		- IFACE 本地网卡接口的名称
		- rxpck/s 每秒钟接受的数据包
		- txpck/s 每秒钟发送的数据包
		- `rxKB/S 每秒钟接受的数据包大小，单位为KB`
		- `txKB/S 每秒钟发送的数据包大小，单位为KB`
		- rxcmp/s 每秒钟接受的压缩数据包
		- txcmp/s 每秒钟发送的压缩包
		- rxmcst/s 每秒钟接收的多播数据包
## 10 环境变量

- 在讲解which命令的时候，我们知道使用的一系列命令其实本质上就是一个个的可执行程序
- 比如，cd命令的本体就是：/usr/bin/cd 这个程序文件
- 我们是否会有疑问，为何无论当前工作目录在哪里，都能执行：/usr/bin/cd这个程序呢?
- 这就是环境变量的作用啦


- 环境变量是操作系统（Windows、Linux、Mac）在运行的时候，记录的一些关键性信息，用以辅助系统运行
- 在Linux系统中执行：`env命令即可查看当前系统中记录的环境变量`
- 环境变量是一种KeyValue型结构，即名称和值，如下图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fdc84f1d0b6b8d9b85ed4c748dae9a72.png)

- 在前面提出的问题中，我们说无论当前工作目录是什么，都能执行/usr/bin/cd这个程序，这个就是借助环境变量中：PATH这个项目的值来做到的。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57afe0346b39e6be9c2fb97d294b780a.png)
- PATH记录了系统执行任何命令的搜索路径，如上图记录了（路径之间以:隔开）：
	- /usr/local/bin
	- /usr/bin
	- /usr/local/sbin
	- /usr/sbin
	- /home/itheima/.local/bin
	- /home/itheima/bin
- 当执行任何命令，都会按照顺序，从上述路径中搜索要执行的程序的本体
- 比如执行cd命令，就从第二个目录/usr/bin中搜索到了cd命令，并执行

- `$符号`
- 在Linux系统中，`$符号被用于取”变量”的值`。
- 环境变量记录的信息，除了给操作系统自己使用外，如果我们想要取用，也可以使用。
- 取得环境变量的值就可以通过语法：`$环境变量名`  来取得
- 比如： `echo $PATH`
- 就可以取得PATH这个环境变量的值，并通过echo语句输出出来。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f8c105f05b377a3d4342ac600d10d2d7.png)
- 又或者：echo ${PATH}ABC![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/933f8895f8b31765caa27257c3d26c0d.png)
- 当和其它内容混合在一起的时候，可以通过{}来标注取的变量是谁

- `自行设置环境变量`
- Linux环境变量可以用户自行设置，其中分为：
	- 临时设置，语法：export 变量名=变量值![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4d3b67c3994c01639247faedc1c12562.png)
	- 永久生效
		- 针对当前用户生效，`配置在当前用户的：  ~/.bashrc文件中`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1efa201c50afa1e5e5e5ce0726ffbc2a.png)
		- 针对所有用户生效，`配置在系统的：  /etc/profile文件中`
		- 并通过语法：`source 配置文件`，进行立刻生效，或重新登录FinalShell生效![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6dab3f5e3105c42abb78708ccb16e153.png)
- `自定义环境变量PATH`
- 环境变量PATH这个项目里面记录了系统执行命令的搜索路径。
- 这些搜索路径我们也可以自行添加到PATH中去。
- 测试：
	- 在`当前HOME目录内创建文件夹，myenv`，`在文件夹内创建文件mkhaha`
	- 通过vim编辑器，在mkhaha文件内填入：`echo 哈哈哈哈哈`
	- 完成上述操作后，随意切换工作目录，执行mkhaha命令尝试一下，会发现无法执行
	- 在/etc/profile文件中修改`PATH的值`
	- 临时修改PATH：export PATH=$PATH:/home/itheima/myenv，再次执行mkhaha，无论在哪里都能执行了
	- 或将`export PATH=$PATH:/home/itheima/myenv`，填入用户环境变量文件或系统环境变量文件中去
## 11 上传、下载

- 我们可以通过FinalShell工具，方便的和虚拟机进行数据交换
- 在FinalShell软件的下方窗体中，提供了Linux的文件系统视图，可以方便的
	- 浏览文件系统，找到合适的文件，右键点击下载，即可传输到本地电脑
	- 浏览文件系统，找到合适的目录，将本地电脑的文件拓展进入，即可方便的上传数据到Linux中![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/72be225fc4e3f88e6f569cf36ebc64d6.png)
- `rz、sz命令`
- 当然，除了通过FinalShell的下方窗体进行文件的传输以外，也可以通过rz、sz命令进行文件传输
- rz、sz命令需要安装，可以通过：yum -y install lrzsz，即可安装
	- rz命令，进行上传，语法：直接输入rz即可![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84f0a303b58863c90827858349fbb007.png)
	- sz命令进行下载，语法：sz 要下载的文件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf300e7f5c5b8f7b3421089656b6d02e.png)
	- 文件会自动下载到桌面的：fsdownload文件夹中。
- `注意：rz、sz命令需要终端软件支持才可正常运行`，FinalShell、SecureCRT、XShell等常用终端软件均支持此操作

## 12 压缩、解压

- `压缩格式`
- 市面上有非常多的压缩格式
	- zip格式：Linux、Windows、MacOS，常用
	- 7zip：Windows系统常用
	- rar：Windows系统常用
	- tar：Linux、MacOS常用
	- gzip：Linux、MacOS常用
- 在Windows系统中常用的软件如：winrar、bandizip等软件，都支持各类常见的压缩格式，这里不多做讨论。
- 我们现在要学习，如何在Linux系统中操作：tar、gzip、zip这三种压缩格式完成文件的压缩、解压操作

- `tar命令`
- Linux和Mac系统常用有2种压缩格式，后缀名分别是
	- `.tar`，称之为tarball，归档文件，即简单的将文件组装到一个.tar的文件内，并没有太多文件体积的减少，仅仅是简单的封装
	- `.gz`，也常见为.tar.gz，gzip格式压缩文件，即使用gzip压缩算法将文件压缩到一个文件内，可以极大的减少压缩后的体积
- 针对这两种格式，使用`tar命令均可以进行压缩和解压缩的操作`
- 语法：`tar [-c -v -x -f -z -C] 参数1 参数2 ... 参数N`
	- -c，创建压缩文件，用于压缩模式
	- -v，显示压缩、解压过程，用于查看进度
	- -x，解压模式
	- -f，要创建的文件，或要解压的文件，-f选项必须在所有选项中位置处于最后一个
	- -z，gzip模式，不使用-z就是普通的tarball格式
	- -C，选择解压的目的地，用于解压模式

- `tar压缩`
	- `tar -cvf test.tar 1.txt 2.txt 3.txt`：将1.txt 2.txt 3.txt 压缩到test.tar文件内
	- `tar -zcvf test.tar.gz 1.txt 2.txt 3.txt`：将1.txt 2.txt 3.txt 压缩到test.tar.gz文件内，使用gzip模式
- ***注意：-z选项如果使用的话，一般处于选项位第一个；-f选项，必须在选项位最后一个***

- `tar 解压`
	- `tar -xvf test.tar`：解压test.tar，将文件解压至当前目录
	- `tar -xvf test.tar -C /home/itheima`：解压test.tar，将文件解压至指定目录（/home/itheima）
	- `tar -zxvf test.tar.gz -C /home/itheima`：以Gzip模式解压test.tar.gz，将文件解压至指定目录（/home/itheima）

- `zip 命令压缩文件`
- 可以使用zip命令，压缩文件为zip压缩包
- 语法：`zip [-r] 参数1 参数2 ... 参数N`
	- -r，被压缩的包含文件夹的时候，需要使用-r选项，和rm、cp等命令的-r效果一致
- 示例：
	- `zip test.zip a.txt b.txt c.txt`：将a.txt b.txt c.txt 压缩到test.zip文件内
	- `zip -r test.zip test itheima a.txt`：将test、itheima两个文件夹和a.txt文件，压缩到test.zip文件内

- `unzip 命令解压文件`
- 使用unzip命令，可以方便的解压zip压缩包
- 语法：`unzip [-d] 参数`
	- -d，指定要解压去的位置，同tar的-C选项
	- 参数，被解压的zip压缩包文件
- 示例：
	- `unzip test.zip`：将test.zip解压到当前目录
	- `unzip test.zip -d /home/itheima`：将test.zip解压到指定文件夹内（/home/itheima）
- ***注意：如果解压后与所在目录有同名的内容，会覆盖***
