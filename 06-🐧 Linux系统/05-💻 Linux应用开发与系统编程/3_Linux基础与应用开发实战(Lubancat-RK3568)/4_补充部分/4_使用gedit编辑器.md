---
文章标题: "[[4_使用gedit编辑器]]"
文章作者: Dakkk
文章概要: |
  文章介绍Ubuntu中`gedit`编辑器终端使用，强调通过`sudo gedit`获取管理员权限编辑系统配置文件。它解释了普通用户权限限制，并建议Linux新手在图形界面下优先使用`gedit`替代`Vim`，以降低学习门槛，同时指出`sudo`对修改系统文件的重要性。
tags:
  - gedit
  - Ubuntu
  - 文本编辑器
  - sudo
  - 配置文件
  - Linux命令行
  - 新手教程
相关文章:
  - "[[../../../../01-📚 知识管理/06-🛠️ 技巧/4_Ubuntu系统/1_参考文档]]"
  - "[[1_properties 和 yml]]"
  - "[[../../../../01-📚 知识管理/06-🛠️ 技巧/5_nvim/1_win环境搭建]]"
  - "[[2_获取数据库连接]]"
  - "[[../../../../01-📚 知识管理/06-🛠️ 技巧/4_Ubuntu系统/2_ubuntu系统移植（dd）]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/4_补充部分/4_使用gedit编辑器.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-04 18:14:04
修改时间: 2025-06-04 18:17:32
---


gedit是在Ubuntu系统下的地位就如同Windows系统下的记事本软件，它的功能不算强大，但因为是系统自带的， 说不上什么时候我们就会图方便使用它用来轻度编辑和记录一些内容

在图形界面下使用gedit编辑器在前面已经介绍过，此处我们演示下在终端打开该编辑器。 在前面的 [《使用Linux命令行/命令究竟是什么》](https://doc.embedfire.com/linux/stm32mp1/linux_base/zh/latest/linux_basis/command_line/command_line.html#id9) 中提到， 命令的本质就是一些应用程序，Ubuntu系统自带的gedit编辑器，我们也可以通过终端来打开：

```shell
# 在终端中执行行下列命令：
gedit 要打开或新建的文件名
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/697a83f1d356cfc79ae098a5e60a7145.png)

为什么要用终端打开gedit？因为在修改系统配置文件时，通常需要管理员权限，而在图形界面下通过gedit打开文件， 只能使用当前用户的身份，这种情况下是无法对这些文件进行修改的。

但是，如果我们在终端下，使用sudo命令为gedit增加权限来打开这些文件， 则可以对它进行修改，在 [3_apt及yum包管理工具](3_apt及yum包管理工具.md)小节就介绍了这样的方式对/etc/apt/sources.list 文件进行了修改。

执行以下命令可进行对比，使用普通用户权限与sudo权限打开配置文件的差异如下图所示。
```shell
#在后面的说明中，#号表示注释，它后面的内容不要输入到终端中

gedit /etc/apt/sources.list         #以普通用户身份打开配置文件
sudo gedit /etc/apt/sources.list    #使用sudo增加权限打开配置文件
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8f8628f6d132888f36abedaf1e6f7197.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d6c704d2ee6e82ac2f80d63f5b6e78b4.png)

总而言之，当我们使用有图形界面的Ubuntu桌面版进行开发时，使用sudo gedit的方式修改配置文件是一个非常好的选择。

Linux新手在配置开发环境时，对Linux本来就不熟悉，刚学习配置开发环境时一般又一头雾水， 再加上网上的教程一般是使用Vim来修改配置文件，新手照着敲命令使用Vim编辑器来改内容， 经过Vim的一番操作后，直接就劝退了，我们从此损失了一位未来的技术大牛。

在网上以及我们后面的各种修改配置文件的命令中，默认都是使用Vi/Vim来修改，因为系统一般自带Vi， 而且不需要区分系统是否需要使用图形界面。但是对于刚学习Linux的用户，能不用Vim就不去用，在使用带图形界面的Ubuntu时， 看到命令中的vim时心里就默认把命令中的“vim”替换成“gedit”即可。

例如：
```shell
#使用sudo权限通过vim打开文件
sudo vim /etc/apt/sources.list
#使用sudo权限通过gedit打开文件
sudo gedit /etc/apt/sources.list
```

以上命令区别仅为“vim”和“gedit”，即使是Vim编辑器同样也是需要sudo权限才能对文件进行编辑的， 如果不使用sudo，以普通用户的身份使用Vim编辑器打开配置文件，同样也是只有“只读”权限。所以，千万不要把Vi/Vim编辑器神圣化了。