---
文章标题: "[[6_Linux常用操作]]"
文章作者: Dakkk
文章概要: |
  介绍Linux系统常用操作命令，涵盖软件管理、系统服务、文件操作、网络工具、系统监控、用户权限管理等基础运维技能
tags:
  - Linux
  - 系统管理
  - 命令行
  - 运维
  - 网络工具
  - 用户权限
  - 文件操作
  - 系统监控
相关文章:
  - "[[../01-❇️ 参考资料/0_个人实用技巧/5_Linux相关命令]]"
  - "[[../05-💻 Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/1_Linux系统/7_Linux命令行]]"
  - "[[3_CentOS 安装 Docker]]"
  - "[[4_Ubuntu 安装 Docker]]"
  - "[[../01-❇️ 参考资料/0_个人实用技巧/4_apt命令及问题]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/02-⚙️ 系统基础/01-💻 命令行操作/6_Linux常用操作.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:19:51
---

## 1 软件安装

- CentOS系统使用：
  - yum [install remove search] [-y] 软件名称
    - install 安装
    - remove 卸载
    - search 搜索
    - -y，自动确认
- Ubuntu系统使用
  - apt [install remove search] [-y] 软件名称
    - install 安装
    - remove 卸载
    - search 搜索
    - -y，自动确认

> yum 和 apt 均需要root权限

## 2 systemctl

功能：控制系统服务的启动关闭等

语法：`systemctl start | stop | restart | disable | enable | status 服务名`

- start，启动
- stop，停止
- status，查看状态
- disable，关闭开机自启
- enable，开启开机自启
- restart，重启



## 3 软链接

功能：创建文件、文件夹软链接（快捷方式）

语法：`ln -s 参数1 参数2`

- 参数1：被链接的
- 参数2：要链接去的地方（快捷方式的名称和存放位置）



## 4 日期

语法：`date [-d] [+格式化字符串]`

- -d 按照给定的字符串显示日期，一般用于日期计算

- 格式化字符串：通过特定的字符串标记，来控制显示的日期格式
  - %Y   年%y   年份后两位数字 (00..99)
  - %m   月份 (01..12)
  - %d   日 (01..31)
  - %H   小时 (00..23)
  - %M   分钟 (00..59)
  - %S   秒 (00..60)
  - %s   自 1970-01-01 00:00:00 UTC 到现在的秒数



示例：

- 按照2022-01-01的格式显示日期

  ![image-20221027220514640|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e933b356f1c1f16e9dfac7ac97dd2f82.png)

- 按照2022-01-01 10:00:00的格式显示日期

  ![image-20221027220525625|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ff24da055e4a034c427b616b0447863.png)

- -d选项日期计算

  ![image-20221027220429831|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cb44efbfa29f8031b86af96d14e785a6.png)

  - 支持的时间标记为：

    ![image-20221027220449312|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa9126262fd95aea49295d4dc7b83f32.png)





## 5 时区

修改时区为中国时区

![image-20221027220554654|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7747f733ee4f09d2d86546f645a5f2e.png)



## 6 ntp

功能：同步时间

安装：`yum install -y ntp`

启动管理：`systemctl start | stop | restart | status | disable | enable ntpd`



手动校准时间：`ntpdate -u ntp.aliyun.com`



## 7 ip地址

格式：a.b.c.d

- abcd为0~255的数字



特殊IP：

- 127.0.0.1，表示本机
- 0.0.0.0
  - 可以表示本机
  - 也可以表示任意IP（看使用场景）



查看ip：`ifconfig`



## 8 主机名

功能：Linux系统的名称

查看：`hostname`

设置：`hostnamectl set-hostname 主机名`



## 9 配置VMware固定IP

1. 修改VMware网络，参阅PPT，图太多

2. 设置Linux内部固定IP

   修改文件：`/etc/sysconfig/network-scripts/ifcfg-ens33`

   示例文件内容：

   ```shell
   TYPE="Ethernet"
   PROXY_METHOD="none"
   BROWSER_ONLY="no"
   BOOTPROTO="static"			# 改为static，固定IP
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   IPV6INIT="yes"
   IPV6_AUTOCONF="yes"
   IPV6_DEFROUTE="yes"
   IPV6_FAILURE_FATAL="no"
   IPV6_ADDR_GEN_MODE="stable-privacy"
   NAME="ens33"
   UUID="1b0011cb-0d2e-4eaa-8a11-af7d50ebc876"
   DEVICE="ens33"
   ONBOOT="yes"
   IPADDR="192.168.88.131"		# IP地址，自己设置，要匹配网络范围
   NETMASK="255.255.255.0"		# 子网掩码，固定写法255.255.255.0
   GATEWAY="192.168.88.2"		# 网关，要和VMware中配置的一致
   DNS1="192.168.88.2"			# DNS1服务器，和网关一致即可
   ```



## 10 ps命令

功能：查看进程信息

语法：`ps -ef`，查看全部进程信息，可以搭配grep做过滤：`ps -ef | grep xxx`

|**选项**|**描述**|
|---|---|
|`-A` 或 `--all`|显示所有进程（包括其他用户的进程）。等同于 `-e`。|
|`-e` 或 `--everyone`|显示所有进程，等同于 `-A`。|
|`-a`|显示所有终端上的进程，除了会话领导进程。|
|`-u`|显示与特定用户相关的进程（通过用户名或 UID）。|
|`-x`|显示没有控制终端的进程。|
|`-f`|显示进程的完整格式（包含父进程 ID、进程启动时间等）。|
|`-l`|显示进程的长格式，包括更多的详细信息（如进程状态、优先级等）。|
|`-o`|自定义输出显示的内容，指定要显示的列（如 `ps -eo pid,comm`）。|
|`-p`|显示指定 PID 的进程信息。|
|`-C`|显示特定命名进程的所有进程（如 `ps -C nginx`）。|
|`-H`|显示进程树（层级结构）。|
|`-eF`|显示所有进程及其所有完整信息。|
|`-T`|显示所有终端上的进程。|
|`--sort`|根据指定的字段对输出结果排序（如 `--sort=-pcpu`）。|
|`-v`|显示进程的详细信息（如内存使用情况、进程状态等）。|

|**字段**|**描述**|
|---|---|
|`PID`|进程 ID|
|`TTY`|终端类型（进程所在的终端）|
|`TIME`|进程使用的 CPU 时间|
|`CMD`|启动进程的命令|
|`USER`|进程所有者的用户名|
|`%CPU`|进程占用的 CPU 时间百分比|
|`%MEM`|进程占用的内存百分比|
|`STAT`|进程状态（如 S 表示休眠，R 表示运行）|
|`PPID`|父进程 ID|
|`UID`|用户 ID|
|`START`|进程启动时间|
## 11 kill命令

![image-20221027221303037|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d2d25e324c4dd86928b5ea5513d2ed0e.png)



## 12 nmap命令

![image-20221027221241123|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fef18b7c9c7558dd06e8f532d7d865ba.png)



## 13 netstat命令

功能：查看端口占用

用法：`netstat -anp | grep xxx`



## 14 ping命令

测试网络是否联通

语法：`ping [-c num] 参数`

![image-20221027221129782|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4e926b1b2410740ad1ef3436082055c1.png)



## 15 wget命令

![image-20221027221148964|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5705a860b7cb6b0c921d88bd5a836729.png)

## 16 curl命令

![image-20221027221201079|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce03a68d751236afc00aaea6d7375f3d.png)

![image-20221027221210518|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/efec8ed10a2571123942d2be9aabe0d9.png)



## 17 top命令

功能：查看主机运行状态

语法：`top`，查看基础信息



可用选项：

![image-20221027221340729|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bf64dc7e4361c18f78bc537e74a6f8c5.png)



交互式模式中，可用快捷键：

![image-20221027221354137|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb87d4ae21ffbd260135fc2abf3f9fe2.png)



## 18 df命令

查看磁盘占用

![image-20221027221413787|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1df68be65dccbcd01344582efb28bea9.png)



## 19 iostat命令

查看CPU、磁盘的相关信息

![image-20221027221439990|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cb42e37ee86f5380e55cb1f74eec6046.png)

![image-20221027221514237|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/df72bdf7236ecd69769f21fa6de1bc0b.png)



## 20 sar命令

查看网络统计

![image-20221027221545822|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7ba5844f7046026cfdb350eb049aa84a.png)



## 21 环境变量

- 临时设置：export 变量名=变量值
- 永久设置：
  - 针对用户，设置用户HOME目录内：`.bashrc`文件
  - 针对全局，设置`/etc/profile`



### 21.1 PATH变量

记录了执行程序的搜索路径

可以将自定义路径加入PATH内，实现自定义命令在任意地方均可执行的效果



## 22 $符号

可以取出指定的环境变量的值

语法：`$变量名`

示例：

`echo $PATH`，输出PATH环境变量的值

`echo ${PATH}ABC`，输出PATH环境变量的值以及ABC

如果变量名和其它内容混淆在一起，可以使用${}





## 23 压缩解压

### 23.1 压缩

`tar -czvf 压缩包名.tar.gz 要压缩的目录/`
- **参数说明**：
    - `-c`：创建新的压缩包。
    - `-z`：使用 `gzip` 压缩（生成 `.tar.gz` 文件）。
    - `-v`：显示压缩过程（可选）。
    - `-f`：指定输出文件名。

`zip [-r] 参数1 参数2 参数N`

![image-20221027221906247|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8f55010100e3554e4aa8427f615bfdd.png)
### 23.2 解压

`tar -zxvf 被解压的文件 -C 要解压去的地方`

- -z表示使用gzip，可以省略
- -C，可以省略，指定要解压去的地方，不写解压到当前目录

`unzip [-d] 参数`

![image-20221027221939899|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2800b6d0cbf3df2e7fb8de67a3a81e97.png)
## 24 su命令

切换用户

语法：`su [-] [用户]`

![image-20221027222021619|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d7e62dd72f93e82f4a64db3860126359.png)



## 25 sudo命令

![image-20221027222035337|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1783597ae626cfd99b504bde958b5055.png)



比如：

```shell
itheima ALL=(ALL)       NOPASSWD: ALL
```

在visudo内配置如上内容，可以让itheima用户，无需密码直接使用`sudo`



## 26 chmod命令

修改文件、文件夹权限



语法：`chmod [-R] 权限 参数`

- 权限，要设置的权限，比如755，表示：`rwxr-xr-x`

  ![image-20221027222157276|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4574c1a29d8cee1add88e7ebc9c9e660.png)

- 参数，被修改的文件、文件夹

- 选项-R，设置文件夹和其内部全部内容一样生效



## 27 chown命令

修改文件、文件夹所属用户、组



语法：`chown [-R] [用户][:][用户组] 文件或文件夹`

![image-20221027222326192|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7817fb1d75437399f99757badbc0322.png)



## 28 用户组管理

![image-20221027222354498|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/86acb960b5b9ae9bfa649a3ffb685e05.png)



## 29 用户管理

![image-20221027222407618|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/95cc60e1a34831618c1cc05ace97ea88.png)



## 30 genenv命令

- `getenv group`，查看系统全部的用户组

  ![image-20221027222446514|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88209407fb897d07520fc9f7f507d926.png)

- `getenv passwd`，查看系统全部的用户

  ![image-20221027222512274|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/32ea3788f87c83dbc549bcf89e779727.png)



## 31 env命令

查看系统全部的环境变量

语法：`env`

