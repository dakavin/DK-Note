---
文章标题: "[[5_挂载NFS网络文件系统]]" 
文章作者: Dakkk
文章概要: |
  本文详细讲解了如何在Linux开发主机与嵌入式开发板间搭建NFS网络文件系统。内容涵盖NFS原理、环境配置、服务端安装与共享目录设置，以及客户端挂载与测试，旨在为嵌入式应用传输和调试提供便利。
tags:
- "NFS"
- "网络文件系统"
- "嵌入式开发"
- "Linux"
- "文件共享"
- "挂载"
- "开发板"
相关文章:
- "[[0_介绍]]"
- "[[1_本章代码及文章获取]]"
- "[[1_为什么学习Linux开发📕]]"
- "[[1_Mobaxterm软件的使用]]"
- "[[2_scp命令，拷贝文件]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/05-💻 Linux应用开发与系统编程/3_Linux基础与应用开发实战(Lubancat-RK3568)/4_补充部分/5_挂载NFS网络文件系统.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-04 18:11:49
修改时间: 2025-06-10 15:05:44
---

## 1 前言

本文介绍如何挂载NFS网络文件系统，为后面的主机编译生成的ARM Linux应用传输到开发板做准备

## 2 网络文件系统简介

网络文件系统，常被称为NFS（Network File System），它是一种非常便 捷的在服务器与客户端通过网络共享文件的方式，具体见下图。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/887b90bf7b2e4717494a86309c16c5c0.png)

开启了NFS服务后，客户端访问服务器共享的文件时如同访问本 地存储器（磁盘/SD卡/NAND FLASH等）上的文件一样，对于上层应用来说没有 任何差别，在嵌入式开发时，我们常常利用这个特性在主机上共享文件，主要应用场景如下：
- 在NFS服务器上编译应用软件，客户端（开发板）通过NFS访问并运行应用程序进行测试。
- 把NFS作为根文件系统来启动

## 3 使用NFS的实验环境架构

在后面章节的实验中，我们常常通过NFS给开发板共享开发主机编写的应 用程序，在本节内容将介绍如何在开发板和开发主机之间共享目录。我们要构 建的使用NFS文件系统的实验环境架构如下图所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5c95e67c5568a351a04bd6d5179e434a.png)

在这样的环境中，开发板与开发主机接入到同一个局域网中，然后开发主机 提供NFS服务，开发板通过NFS与开发主机连接共享文件。

开发主机生成的目标板应 用程序放在NFS的共享文件夹内，开发板访问该文件夹执行应用程序进行测试。

在另一方面，开发主机与开发板通过串口连接，使用串口终端控制开发板。

## 4 搭建NFS环境

接下来介绍如何利用NFS搭建上述实验环境。主要包含连接网络、主机开启NFS服务 以及开发板挂载文件系统三个步骤。

我们的目标是使开发主机与开发板的以下目录映射起来：

开发主机共享目录：/home/dakkk

开发板的挂载目录：/mnt

### 4.1 连接到局域网

在本应用场景中开发主机和开发板需要通过网络互相访问，另外由于NFS文件系统 暴露到公网需要处理很多安全问题，为简化操作，我们把开发主机和开发板连接到 局域网络，即都使用网线把它们连接到同一个交换机（路由器）上，如上图所示。

如果开发主机是安装在虚拟机上，注意要在VirtualBox把虚拟机的网络配置 改成“桥接网卡”模式，见 下图，若修改了该配置，需要重启虚拟机才生效。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f58b848b4623dcb8c28171cffabca8c5.png)

连接好网络后在开发主机和开发板之间进行互ping测试，以保证网络互通。ping命令格式为：
```shell
ping [目标主机名或目标ip地址]
```

这里以ping主机名为例，开发主机ping开发板，见下图:
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/013176b3886c4d35e74126d3a7bdd71f.png)

若测试成功，会返回ttl域和时间，使用Ctrl+c可退出命令。

在ping命令的打印信息中，我们还可以了解到本示例中开发主机的IP为192.168.31.147，开发 板的IP为192.168.31.69，我们也可以使用ifconfig命令来查看它们的ip地址和子网掩码，如下图:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9f675f19d9022a95c282762fb39682c6.png)

在上图中开发板使用ifconfig命令可查看到开发板有两个网卡，分别是其两 个网络接口的信息，其中eth1没有接网线，所以不能正常使用，而eth2可看到其IP地 址为192.168.31.69，在开发主机的终端上可使用该IP进行ping测试。

> [!warning]+ 注意
> 若在开发主机上使用ifconfig提示找不到命令，请使用如下命令进行安装：
> ```shell
> #以下命令在主机上运行
   sudo apt install net-tools
> ```

根据开发主机的ip和子网掩码，我们可以判断开发主板处于`192.168.31.*`的网段中， 我们只要保证开发主机网络也在同一个局域网网段中，就能让它们通信

若使用ifconfig命令查看不到IP地址，或IP地址与开发主机的IP不在同一个网段，请检查网络连接。

如果使用主机名无法ping通，请直接用IP地址进行ping测试

### 4.2 在开发主机开启NFS服务

确认网络并了解局域网IP地址信息后，可配置开发主机的NFS服务，以下步骤说明均在开发主机上进行操作。

**1、安装NFS服务**
Ubuntu系统默认没有安装NFS服务，需要使用如下命令安装NFS服务端软件：
```shell
#以下命令在主机上运行
sudo apt install nfs-kernel-server
```

**2、查看用户id**
在配置NFS时需要使用到用户uid和组gid，可使用id命令查看，在开发主机上的终端输入如下命令：
```shell
#以下命令在主机上运行
id
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/13f2aaba346a9cd1f8418241a032ba49.png)
上图使用id命令用户id和组id

从上图可知本开发主机的用户uid和组gid均为1000，请使用该命令查看自己的开发主机相 关id，在下面的配置文件中将会使用到

**3、配置NFS**
安装NFS服务后，会新增一个`/etc/exports`文件（即/etc目录下名字为exports的文件），NFS服务根据它的配置来运行，其默认内容可通过命令`cat /etc/exports`查看，它默认包含了一些配置 的范例，内容如所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dc1f2dff6d4655a32a490a258ac96b0e.png)

详细的帮助说明可以使用 命令man nfs查看，此处直接以我们配置的范例进行讲解。

修改配置文件常常需要系统用户权限，所以通常使用sudo vim或sudo gedit来打开修改，要使用gedit编辑器的话，把后面的vim命令换成gedit即可。

使用vim打开/etc/exports文件命令如下：
```shell
#以下命令在主机上运行，可用gedit替换vim
sudo vim /etc/exports
```

在/etc/exports文件末尾添加如下语句并保存，**注意如下语句写到/etc/exports文件是在同一行**。
```shell
#把以下内容添加至/etc/exports文件末尾，注意以下内容处于同一行
#以下内容的IP地址和uid，gid需要根据自己的环境进行修改
/home/dakkk 192.168.31.0/24(rw,sync,all_squash,anonuid=1000,anongid=1000,no_subtree_check)
```

> [!warning]+ 注意
> 注意具体的配置需要根据自己的实验环境进行配置，请理解如下说明根据自己的实验环境进行修改：

- /home/embedfire/workdir：要共享的开发主机目录，注意使用空格与后面的配置隔开。
- 192.168.0.0/24：配置谁可以访问，其中的/24是掩码，此处表示24个1，即11111111.11111111.11111111.00000000， 即掩码是255.255.255.0。结合前面192.168.0.0表示此处配置IP为 `192.168.0.*`的主机均可以访问该目录，即局域网上的所有主机。
	- 若局域网是其它网段，请参考此处的配置，不能直接用星号表示，如欲配 置192.168.31.* 的局域网下所有机器都可以访问，则配置为 192.168.31.0/24。
	- 这个配置域也可以直接写可访问的主机名，如把“192.168.0.0/24”替换为开 发板主机名“npi”，则仅该开发板能访问共享的目录。

- rw: 表示客户机的权限，rw表示可读写，具体的权限还受文件系统的rwx及用户身份影响。
- sync：资料同步写入到内存与硬盘中。
- anonuid=1000：将客户机上的用户映射成指定的本地用户ID的用户，此处998是开 发主机embedfire用户的uid，此处请根据具体的主机用户uid进行配置。
- anongid=1000： 将客户机上的用户映射成属于指定的本地用户 组ID，此处998是开发主机embedfire用户组gid，此处请根据具体的主机用户组gid进行配置。
- no_subtree_check：不检查子目录权限，默认配置。

本配置中的anonuid和anongid把客户机的用户映射成本地uid/gid为1000的用户，即开发 主机的dakkk，那么当在开发板上使用与开发主机不同的用户访问NFS共享目录时，都会 具有dakkk的权限，方便互相访问。如开发板上的root用户创建文件，在开发主机上会被认为是dakkk创建的；在开发主机上仅dakkk能读写的文件，在开发板上的root或其它用户也 能对该文件进行读写（被当成了dakkk）。当然这并不是一种安全的访问设置方式，不过这用在我们的开发中是非常方便的配置。

**4、创建共享目录**

为了确保共享的配置有效，我们还需要创建共享的目录，在本例子中共享的目录为/home/dakkk

**5、更新exports配置**

修改完/etc/exports文件并保存后，可使用exportfs命令更新配置：
```shell
#以下命令在主机上运行
sudo exportfs -arv
```
- -a：全部mount或umount文件/etc/exports中的内容。
- -r：重新mount文件/etc/exports中的共享内容。
- -u：umount目录。
- -v：在exportfs的时候，将详细的信息输出到屏幕上。

若配置正常，该命令执行后会列出共享的目录项，本示例的执行结果见下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d362fd1e199c5ddc625829b3fe4130c7.png)

**6、查看NFS共享情况**

使用showmount –e 可查看当前NFS服务器的加载情况，具体见下图。
```shell
#以下命令在主机上运行
showmount -e
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/774abf9dec118a87a535f64ae0df5ec5.png)

### 4.3 安装NFS客户端

开发主机开启NFS服务后，我们还需要在开发板安装NFS客户端，来让开发板使用NFS服务

执行安装NFS客户端命令:
```shell
sudo apt install nfs-common -y
```

查看NFS服务器共享目录

在开发板上执行“showmount -e +“NFS服务器IP””命令。**注意在不同网络环境下，NFS服务器IP可能不一样，以实际情况为准。**

```shell
showmount -e 192.168.31.147
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7721575020278a763df3cdef24883ef5.png)

**临时挂载NFS文件目录**

使用mount命令挂载NFS服务器的共享目录到开发板/mnt目录下：

注意:需要把下面的192.168.31.147设置为用户实际网络环境下的NFS服务器IP
```shell
#以下命令在开发板上运行
sudo mount -t nfs 192.168.31.147:/home/dakkk /mnt/dakkk_server
```
- -t nfs：指定挂载的文件系统格式为nfs。
- 192.168.31.147：指定NFS服务器的IP地址。
- /home/dakkk：指定NFS服务器的共享目录。
- /mnt：本地挂载目录，即要把NFS服务器的共享目录映射到开发板的/mnt目录下。

若挂载成功，终端不会有输出，Linux的哲学思想是“没有消息便是好消息”
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b34b5feee2573e000ff7f711b02229e.png)


使用这种方式挂载目录只是临时的，开发板在重启后需要重新挂载该NFS目录才能访问。

**测试NFS共享目录**

挂载成功后，在NFS服务器的共享目录下，输入“sudo touch hello.txt”命令， 则在共享目录下创建一个hello_world.txt文件，如下图:
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b39413b8a016fdadb9ceb83b6af8c081.png)

进入开发板的/mnt目录下，可以查看到NFS服务器的共享目录中的hello_world.txt文件。 如下图:
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3ea8a986437cff4e2d701eddbf2e0aea.png)

**取消挂载**

当客户机在网络上无法找到NFS共享的目录时，如开发主机关机时，在NFS的客户机 的终端常常会输出一些提示，或在使用ls命令查看共享目录会导致长时间等待，这时可以对目录使用umount命令取消挂载，示例如下：
```shell
#以下命令在开发板上运行
sudo umount /mnt
```

使用该命令时以要取消挂载的目录作为参数即可，没有输出表示执行正常。如果 在当前挂载的目录进行umount操作，会提示“device is busy”。建议取消挂 载时，先切换到家目录“~”，在进行umount操作。