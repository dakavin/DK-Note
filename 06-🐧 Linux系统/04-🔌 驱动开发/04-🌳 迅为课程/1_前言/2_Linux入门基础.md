## 1 安装VMware和Ubuntu

**网上找最新的资料，或者根据板卡给的软件进行安装即可**
## 2 Ubuntu系统

### 2.1 介绍

**Ubuntu 和 Linux的关系**
Ubuntu 是 linux 发行版之一，它和 linux 的关系是包含与被包含的关系，也就是说我们的 ubuntu 是包含在linux里面的。我们常见的操作系统有 windows、linux、iOS 还有 Android

**Linux的发行版**
我们 linux 的发行版是不是也有很多呢？除了ubuntu，我们 linux 的发行版还有 radhat、centos、debian、openwrt 。

那么我们为什么在学习和开发的时候选用ubuntu 作为我们的系统呢，而不是选择别的呢，因为ubuntu 有着**良好的图形界面**和**非常强大的 ape-get 功能**，所以我们一般都是用 ubuntu 来进行学习和开发的。

**常见的Ubuntu**
Ubuntu 它也是分为好多个种类，从外观上分，它分为有界面的 ubuntu 和没有界面的 ubuntu
- 没有界面的ubuntu 我们把它叫做 ubuntu-core,也就是ubunut 的文件系统
- 有界面的 ubuntu 我们把它叫做ubuntu-desktop

有界面的 ubuntu 和没有界面的ubuntu 它主要区别在有没有显示界面上，没有界面的ubuntu 它除了不能显示外，其他的和我们有界面的都是一样的。有界面的 ubuntu 他主要是在 X86 上运行的，但是并不是说我们的 ARM芯片不能跑有界面的ubuntu,比如说我们的 4412、4418、6818、iMX6Q、3399，这些性能强大的 arm 芯片,都是可以跑有界面的ubuntu 的，但是我们的i.MX6ull，因为它的性能比较弱，所以它只能跑没有界面的ubuntu

**那么有界面的 ubuntu 他是怎么组成的呢？**
有界面的 ubuntu 它是由 ubuntu-core 也就是ubuntu 的文件系统加上第三方桌面组成的。我们第三方桌面种类不同，我们组成的 ubuntu 就是不一样的。比如说我们用到的ubuntu，它是由 ubuntu-core 加上 gnome 这个第三方桌面组成的。除了 gnome 这个第三方桌面我们还有 kde 这个比较常见的第三方桌面，这个第三方桌面加我们 ubuntu 他就组成了kubuntu，中文名就叫库 ubuntu。除了
kde 我们还有一个常见的第三方桌面 Ixde，这个第三方桌面加我们的文件系统就组成了 lubuntu，lubuntu 他是一个轻量级的 ubuntu，它可以在配置不是很好的地方运行。
### 2.2 root用户

#### 2.2.1 命令行组成

在启用 root 用户之前，我们先来了解一下，ubuntu 命令的组成。

我们打开ubuntu的终端，我们现在的命令行是由 `topeet@ubuntu:~$`这几个字母组成，那么这几个字母都代表什么意思呢?
```shell
topeet：当前操作用户
Ubuntu:代表主机名
~:代表当前目录名
$:代表普通用户操作权限
：代表root 用户权限
```

首先 topeet 代表当前操作用户，也就是说我们当前操作的用户为topeet，@ 是固定格式，ubuntu 代表的是主机名，也就是我们这台虚拟机 ubuntu 它的主机名叫做 ubuntu，这是我们安装ubuntu 的时候自己命名的。冒号同样是固定格式，～代表的是当前目录名，$代表的是普通用户操作权限，也就是非root 用户显示。
#### 2.2.2 root用户的作用

了解了命令行的组成之后，我们再来启用root用户。我们是嵌入式开发人员，我们使用ubuntu系统主要是来做嵌入式开发的，不是linux 运维，所以我们没有必要像linux 运维那样对root 权限非常的敏感。作为一个嵌入式开发人员，系统的权限都要为我们打开。我们在安装系统的时候，root用户是被禁用的，提示创建的用户是被分到admin组的，使用admin组的用户，可以启用并设置root用户。
#### 2.2.3 启用和退出root用户

```shell
# 进入root
su root
# 退出root
exit
```
### 2.3 apt-get

什么是apt-get 呢，在 windows 上安装软件，我们一定都非常的熟悉了，我们直接下载安装就可以了，我们在 ubuntu 上有的软件也可以这样做，比如说这个软件支持 linux 系统，那么我们就可以通过浏览器，然后下载这个软件来安装，但是我们通常在开发的时候我们用不到它，基本都是使用 apt-get 来下载的，这个命令它可以实现软件的自动下载，安装，配置。
#### 2.3.1 使用前检查

apt-get 命令它采用的是 cs 模式，也就是客户端和服务器的模式，我们的 ubuntu 系统是作为客户端，当我们需要下载软件的时候，我们就向服务器发出请求，所以说，我们要使用 apt-get 之前，我们的 **ubuntu系统必须是可以联网的**。

一般没有网络都是VM虚拟机的网络桥接有问题，设置一下就好了
#### 2.3.2 设置下载源

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

**先备份一下自己的系统源清单**
```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.bak1
```

**然后用vi编辑list清单，cv即可**

#### 2.3.3 其他指令

```shell
# 更新下载源
apt-get update
# 安装软件
apt-get install vim
# 软件更新
apt-get upgrade vim
# 软件卸载
apt-get remove vim
```
## 3 Vim

```shell
# 安装
sudo apt install vim
# 打开
vi       #若系统安装了vim，该命令会自动打开vim软件
vim      #打开vim软件
vi 文件名   #若文件存在则打开，文件不存在则创建
vim 文件名
```

Vim编辑器的三种工作模式：
- 一般模式（normal mode）：一般模式用来浏览文本，查找内容，但是不可以编辑，在该模式下的键盘输入会被当成快捷键， 如复制粘贴等。打开Vim时，默认是工作在一般模式。
- 插入模式（insert mode）：插入模式下具有普通编辑器的功能，该模式下的键盘输入会被当成文本内容。
- 命令行模式（command-line mode）：命令行模式支持保存、退出、替换等命令，以及Vim的高级功能。

我们在使用Vim时，常常会在这三种模式之间进行切换，切换方式可以参考下图。
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d8cb7e73f853a7d16d0852024e77d753.png)

### 3.1 插入模式

Vim提供多个快捷键来从一般模式进入插入模式，见下表。进入插入模式之后，就可以正常地编辑文本了， 使用方向键来移动光标，回车键进行换行，操作方式与Windows记事本没有什么区别。

|快捷键|功能描述|
|---|---|
|i|在当前光标所在位置插入文本|
|a|在当前光标所在位置的下一个字符插入文本|
|o|在光标所在位置后插入新行|
|r|替换当前光标所在位置的字符|
|R|可以替换当前光标所在位置之后的字符，按下“Esc”则退出|
|ESC|退出插入模式|

### 3.2 一般模式

在任意模式下按按键“Esc”可进入到一般模式。下表列出了一般模式下常用的快捷键。 在一般模式下，可以进行复制，粘贴，删除，查找替换某个关键字等。

|快捷键|功能描述|
|---|---|---|
|光标移动|k / ↑|光标向上移动|
||j / ↓|光标向下移动|
||h / ←|光标向左移动|
||l / →|光标向右移动|
||PageUp|向上翻页|
||PageDown|向下翻页|
||nG|跳转到第n行|
|文本查找与替换|/word|在文件中搜索关键字word|
||n|查找下一个关键字|
||N|查找上一个关键字|
||:1,$s/word1/word2/gc|将文本中的所有关键字word1用word2进行替换，需要用户进行确认。（使用:1,$s/word1/word2/g则直接全部替换）。这实际是运行在命令模式。|
|撤销重做|u|撤销上一步的操作，等价于Windows的Ctrl+Z|
||Ctrl+r|重做上一步的操作。|
|删除、剪切、复制、粘贴|d|删除光标所选的内容|
||dd|删除当前行|
||ndd|删除光标后n行|
||x|剪切光标选中的字符|
||y|复制光标所选的内容|
||yy|复制当前行|
||nyy|复制当前行后n行|
||p|将复制的数据粘贴在当前行的下一行|
||P|将复制的数据粘贴在当前行的上一行|
|区块操作|v|选择多个字符|
||V|可以选择多行|
||ctrl+v|可以选择多列|

### 3.3 命令行模式

在一般命令模式下，按下键盘的冒号键“:”，就可以进入命令行模式，继续输入要执行的命令按回车即可执行。

|快捷键|功能描述|
|---|---|
|w|保存文档|
|w <filename>|另存为以<filename>为文件名的文档|
|r <filename>|读取文件名为filename的文档|
|q|直接退出软件，前提是文档未做任何修改|
|q!|不保存修改，直接退出软件|
|wq|保存文档，并退出软件。|
|set nu|在行首加入行号|
|set nonu|不显示行号|
|set hlsearch|搜索结果高亮显示|
|! command|回到终端窗口，执行command命令，按回车键可切回vim。|

## 4 Linux

### 4.1 基础部分

**相关操作可以查看下面的文档链接**
- [1_初识Linux](../../../02-⚙️%20系统基础/01-💻%20命令行操作/1_初识Linux.md)
- [2_Linux基础命令](../../../02-⚙️%20系统基础/01-💻%20命令行操作/2_Linux基础命令.md)
- [3_用户和权限](../../../02-⚙️%20系统基础/01-💻%20命令行操作/3_用户和权限.md)
- [4_Linux实用操作](../../../02-⚙️%20系统基础/01-💻%20命令行操作/4_Linux实用操作.md)
- [5_Linux常见指令](../../../02-⚙️%20系统基础/01-💻%20命令行操作/5_Linux常见指令.md)
- [6_Linux常用操作](../../../02-⚙️%20系统基础/01-💻%20命令行操作/6_Linux常用操作.md)
### 4.2 linux帮助手册

- 输入 man man 命令来查看每个页的具体含义
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/778c6378a5f99716075f9a55c629ccb8.png)
- 使用手册查看命令
	- man -f pwd：查看命令在第几页
	  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f896211a54a0cb593dfad799472a4028.png)
	- 查看pwd介绍/使用，man 1 pwd

### 4.3 Linux文件系统

**linux 为什么需要文件系统？**
- Linux 系统必须要挂载一个文件系统，如果系统不能从指定的设备挂载，系统就会出错

**linux 常见文件系统的类型都有哪些？**
- ext3 ， ext4 ， proc 文件系统 ， sysfs 文件系统

- ext3 文件系统是从 ext2 发展过来的，而且完全兼容 ext2 文件系统，并且比 ext2 要小，要可靠。
- ext4 文件系统是在 ext3 的基础上改进的，并且 ext4 文件系统在性能和可靠性上都要比 3 的表现更好，而且功能也非常的丰富，并且 ext4 完全兼容 ext3 ，ext3 只支持 32000 个子目录，但是 ext4 支持无限数量的子目录，所以比 3 更优秀。
- `Proc 文件系统`，这个文件系统是 linux 系统中特殊的文件系统，实际上它是只存在内存中的，他是一个伪文件系统。这个文件系统是内核和内核模块用来向进程发送消息的机制。

**ubuntu 的文件系统类型是什么呢？**
- 使用命令df来查看，其中df -Th 更容易读的方式显示
- 如果不想看文件系统的内容，就可以不加 T 参数，直接输入 df -h 参数，这样就能看到一个磁盘的使用状况
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/50798c0e0f156c895b47ee266520e60f.png)
- /dev/sda5 是 ubuntu 的主分区，Type 是文件系统的类型。所以我们 ubuntu 的主分区的文件类型就是 ext4。ext4 上边的 tmpfs 是虚拟内存文件系统
- Use% 是磁盘使用率，这里要注意下，如果 /dev/sda1 使用率在 90%以上都要用满了，就要注意了，可能会造成我们系统出问题
- 最后一个 Mounted on 是磁盘挂载的目录，就是说磁盘挂载到哪个目录下，这里 /dev/sda5 就挂载到了 / 目录上面

### 4.4 Linux第一个程序Hello World

在 linux 上也就是 ubuntu 上我们编译程序使用的是 gcc

**什么是gcc?**
- gcc 全称（gnu compiler collection）即编译套件，gcc 可以支持多种计算机体系结构，比如 X86、 ARM 、MIPI .我们使用的 ubuntu 默认自带的 gcc
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/38c5254c3fbba5fac0697b6a720ab327.png)
**gcc的基本用法**
- gcc 选项 文件名
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/18698084f3b0aa154e5a5f6b300b9ed7.png)
- 使用 gcc 编译器编译出来的可执行文件是 X86 的，不能在 arm 开发板上运行。可以使用 file 命令来查 看文件类型
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/69cff7e04f6eedeb49e9bfa29e7efd0d.png)
**gcc流程：预处理，编译，汇编，链接**
```shell
# 预处理
gcc -E hello.c -o hello.i
# 编译
gcc -S hello.i -o hello.s
# 汇编
gcc -c hello.s -o hello.o
# 动态链接
gcc hello.o -o hello
```

**第一阶段**：预处理阶段，编译器会对头文件或者宏定义进行展开，或者条件编译的选择我们可以使用 -E 参数得到预处理文件
- -E ：只对文件进行预处理，不编译和链接。
- 使用 gcc -E hello.c -o hello.i 得到预处理后的文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/da08725e9a74a6de5fb89d34a87d8c85.png)
**第二阶段**：编译，把文件编译成汇编代码
- -S 参数 将 hello.i 文件编译成 hello.s 文件
- gcc -S hello.i -o hello.s

**第三阶段：** 汇编，把汇编文件编译机器码
- -c 参数 可以把 hello.s 文件编译成 hello.o 文件
- gcc -c hello.s -o hello.o

**第四阶段**：链接
- 直接把目标文件编译成可执行文件
	- gcc hello.o -o hello
- 链接分为静态链接和动态链接，gcc 默认的是动态链接，特点是生成的程序下，但是需要依赖库
- 静态链接：使用 -static 参数就是静态链接，因为程序里面包含了需要的库，所以体积比较大

### 4.5 Linux环境变量

**环境变量的概念：**
- 环境变量是系统预设值的参数。Linux 是一个多用户的操作系统，所以每一个用户也都有自己的环境变量
- 比如我们之前学习的命令我们不管在哪个路径下输入，都是可以执行成功的，因为系统已经把命令的搜索路径提前设置好了

**常用的环境变量 PATH**
- 可以通过echo命令，显示PATH，`echo $PATH`
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ee731f9b183395be634ca3fdc667949c.png)
**修改环境变量的方式**
- 举例：把 /home/lwk_practice/test 路径加到 PATH 变量里面去

**方式一：** 直接使用命令设置
- 命令格式：export 变量=新增的变量值：$变量
```shell
export PATH=/home/lwk_practice/test/:$PATH
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9fe2d85746e98596732f9dbf188f6633.png)
- 使用这个方法环境变量是立刻生效的，但是只是`临时改变，我们重新打开再关闭终端就没有了`，而且只对当前用户生效

**方式二：**  直接在配置文件（`.bashrc`）里边加上我们的环境变量
- 文件路径：`./home/lubancat/.bashrc`  或者是 `~/.bashrc`
- 命令格式：export 变量=新增的变量值：$变量
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f49c33902f2b6ab88cfe3da9c25feb99.png)
- 注意修改后，需要使用如下命令，更新一下
```shell
source ~/.bashrc
```
- `修改这个文件是永久的`，但是也是仅对当前用户有效
### 4.6 Linux编写自己的shell命令

我们之前用的ls就是shell命令

实验如下：
- 在当前文件创建一个commond.c文件，然后使用gcc进行编译
- 我们可以发现在当前目录下，使用 ./commond 是可以运行的，但是在其他目录就不能运行了

- 方式一：拷贝commond文件到 /bin/ 目录下，可以运行
```shell
cp commond /bin/
```

- 方式二：将commond的路径加入到PATH这个变量中
```shell
export PATH=/home/lubancat/lwk_practice/test/:$PATH
echo $PATH
```
- 这样就可以在任何目录下执行commond这个可执行文件了

**总结**
- 平常使用的命令就是一个可执行程序，而且在键盘上输入了我们的命令之后，这个命令发给了我们的 shell
- 也就是下图中的bash，是发送给它的，然后它会根据我们输入的这个字符串去环境变量里面去找，去看看有没有和我们的名字一样的程序
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a89efe3c4d48867eff73f302bbaad7fb.png)
## 5 Make工具和makefile文件

### 5.1 前言

前面我们在编写 linux 上第一个程序 hello world 的时候是直接使用 gcc 命令的。我们编译一个程序是非常的简单的，直接输入 gcc 然后跟上程序的名称再跟上指定生成程序的名称，就可以很轻松的编译出 hello这个可执行文件了。

但是如果我们以后工作的时候要编译一个工程，这个工程里面有很多的源文件，这时候我们全部使用这个命令来编译那就非常的麻烦了，而且如果我们修改了一个源文件，那么我们使用命令来编译就要再次执行一遍这个过程，就会非常的耗时间。

如果有小伙伴以前学习过单片机，大家可以类比下单片机开发软件 keil 里面的单独编译和全部编译。 单独编译是很省时间的，全部编译就会非常的耗时间，我们使用命令来编译就相当于我们单片机软件中的全部编译。

为了解决编译一个工程非常繁琐这个问题，前人就给我们发明了编译辅助工具 make 工具，它的编译思路是非常简单的，它会在编译之前先比较哪个文件的时间发生了改变，`如果说这个文件它修改的时间要晚于编译生成的文件，那么它就会按照要求重新构建这些文件`，而不是说再浪费时间重新构建其他的文件了。

假如在单片机上用 keil 写了一个 c 文件，这个工程里边别的文件没有改，那么我们就不用点全部编译，只要编译一下我们修改过的文件就可以了。make 也是这样的，只不过它比较聪明，它不用再人为的去判断了，在编译之前会自动帮我们判断。

### 5.2 什么是make工具

make 工具是编译辅助工具，用于解决使用命令编译工程非常繁琐的问题。

调用这个命令工具：我们在 win 上编程使用 ide ，我们有图形界面，有相应的按钮，比如说 build 或者run 来编译。其实 make 这个编译辅助工具使用也是非常简单的，我们在控制台上直接输入命令，它就会自动调用 make 工具。
### 5.3 如何使用make工具

直接在控制台输入 make 命令，就会调用 make 工具。

直接在当前目录下输入 make ，然后报错了，因为我没有告诉 make 这个工具它按照什么规则来编译我们的程序
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0b443ec66bff3559688299eb2b8ab2ef.png)
### 5.4 什么是makefile

**Makefile 就是描述了整个工程编译连接等规则的文件**，我们在终端输入完 make 命令之后，调用 make工具，make 就会在当前目录按照文件名就会找 makefile 文件，Makefile 的命名必须是 makefile 或Makefile ，m 大写小写都是可以的

刚才输入命令报的这个错就找到原因了，是因为在当前目录下是没有 makefile 这个文件的。

我们在当前目录新建一个 Makefile 文件，然后在当前目录下输入 make 命令，我输入完 make 命令，他就会调用 make 工具，make 工具就会在当前目录下找到 makefile 这个文件，这里又报错了，因为当前目录创建的 makefile 文件，他虽然找到了但是里面是空的，因为没有包含任何的规则。如下图所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f7096f3284baded449bc0ee7ccd82f95.png)

先给大家写一个简单的来试一下，打开 makefile 文件，敲的时候大家一定要按 Tab 首行缩进，不能用空格，然后我们输入内容，保存退出，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e1e4847351243314128861a4abfe1db2.png)

**补充：设置 vim 首行缩进**
-  vi /etc/vim/vimrc  （rc结尾的一般为配置文件）
- 输入 set tabstop=4， 保存后退出即可


然后输入 make ,在当前这个目录下，我就成功的生成 hello 这个可执行文件，执行一下，可以看到成功输出了，到这里我们的编译流程就已经讲完了，我们也成功的利用 makefile 文件和 make 工具，把 hello.c编译成了 hello 可执行文件。如下图所示。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2e57892ae90028d742bf3e85c0ba68f3.png)

我们弄清楚了什么是 make 工具，怎么来调用 make 工具，makefile 又是什么，弄清楚了他们的关系后，后面我们再学习 makefile 语法和裸机编写 makefile 时，就非常的容易了。
## 6 makefile基本语法

### 6.1 基本语法

语法格式：
```makefile
目标:依赖
	（tab缩进）命令
```

举例
```makefile
all:
	gcc hello.c -o hello
```
- 目标：all
- 依赖：空
- 命令：gcc hello.c -o hello

上面的例子也可写成：
```makefile
all:hello.o
	gcc hello.o -o hello
	
hello.o:hello.c
	gcc -c hello.c -o hello.o
```
- 目标：all 和 hello.o
- 依赖：hello.o 和 hello.c
- 命令： 略

因为 all 依赖 hello.o 文件，所以要先执行 gcc -c hello.c -o hello.o 得到 hello.o 文件，然后才可以执行 gcc hello.c -o hello 。所以输入 make 命令后执行顺序如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/32fbdf37bd0c4ab359b843b8d674e7b1.png)

编译的时候，我们可以使用 `make 目标`来编译，如果我们不指定目标的话，默认执行的是第一个目标所对应的规则。也就是说 make 和 make all 是一样的。如上面的例子

接下来，我们使用 make 目标的方法来编译。我们修改 makefile 代码如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4d96edd2921decd61a8fc559a9ac60dc.png)

然后我们输入命令 make clean 就可以直接执行 `rm -rf *.o commond `命令。如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/634f09b4c73973eb085f33ebc25b6d7b.png)

但是，我们在`当前目录下不能存在 makefile 目标名一样的文件`。比如我在当前目录下创建一个名为 clean的文件，然后执行 make clean 命令就会报错。如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/bc4a527dcb462f31a81093eb537ce800.png)

为了解决这个问题，makefile 引入了一个新的概念，叫做伪目标，我们使用伪目标来声明 clean 就可以避免与当前目录下的同名文件发生冲突

**伪目标格式**
`.PHONY:目标`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/836278031dc13316fa80e439c7ca4819.png)

然后我们在执行 make clean 命令。尽管当前目录下有 clean 同名文件， make clean 命令也可以执行 成功。如下图所示。
### 6.2 变量和变量赋值

变量可以对许多地方使用，比如目标，依赖，或者命令。

变量的赋值可以使用： = ？= := +=

变量的使用：makefile 通过`$()` 来完成变量的引用

**示例1：** 使用 `:=` 来赋值，立即赋值
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1654662a915ad3ffc91532335a714e48.png)
- 在执行 var：=aaa 的同时变量值已经被确定了，所以最后打印 为 aaabbb，而不是 cccbbb

**示例2：** 使用`=`来赋值，延迟赋值
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9a7274124558e89477aeb4beb60cbab8.png)
- 我们最后给变量 var1赋值为 ccc ,所以最后打印为 cccbbb ，而不是 aaabbb


**示例3：** 使用 `?=`来赋值，前面没有赋值就用 `?=` 后的值，前面有赋值，就使用前面的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a622fb787dcf5d108907f3984aa6bd5b.png)
- 如果 var1 变量前面没有被赋值，那么就给它赋值为 ccc ，如果前面已经赋值了，就适应前面的值，所以，打印为 aaabbb ,而不是 cccbbb


**示例4：** 使用`+=`来赋值，追加赋值
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d9b89ad44903e7bd3463d1ee9a58c258.png)
- 在我们前面定义的好的字符串里面在添加进去新的字符串，所以运行会打印 aaa bbbccc 。不过中间会有空格
### 6.3 自动化变量

**自动化变量就是不用定义且会随着上下程序的不同而发生变化的变量叫做自动化变量**

这里介绍三个最常用的自动化变量：
- `$@`: 表示所有目标
- `$<` :表示第一个依赖文件，如果依赖模式是%，那么他就表示一系列文件。%为通配符，类似 linux 上的 `*`）
- `$^` :表示所有依赖

在了解这三个自动化变量之前，我们先来写一个程序：
```c
// main.c
#include <stdio.h>
#include "hello.h"

void main(){
    hello();
}

// hello.c
#include <stdio.h>
#include "hello.h"

void hello(void){
    printf("Hello, World!\n");
}

// hello.h
#ifdef _HELLO_
#define _HELLO_

void hello(void);

#endif

// makefile
hello:hello.o main.o
	gcc hello.o main.o -o hello
hello.o:hello.c
	gcc -c hello.c -o hello.o
main.o:main.c
	gcc -c main.c -o main.o
clean:
	rm -rf *.o hello
```

使用这个 makefile 虽然也可以成功编译，但是，一旦编译的文件多了，如果我们还这样来编写 makefile就会变得非常复杂。所以，自动化变量就派上用场啦。

接下来我们一步一步的来简化我这个 makefile 。
- 简化一：用变量表示依赖文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/009e53a25452301fd1bc5e0e1831c632.png)
	- 后续如果要增加依赖文件，直接通过 += 在var变量后面增加即可

- 简化二：使用通配符 `%` ，和自动化变量 `$<` 、`$@` 代替依赖和目标，
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cf7a6d69fa93c7f14632db70859f9e5b.png)

- 简化三：使用自动化变量 `$^` 表示所有文件依赖的列表
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2a764b1bfe433be9739f830efa860091.png)

### 6.4 wildcard 函数（展开指定目录）

格式：`$ (wildcard PATTENR)`
功能： 展开指定的目录
举例：在 ~/lwk_practice/test 目录有一个 a.c 的 c 文件和一个 test 的文件夹，在~/lwk_practice/test/test 文件夹下有一个 b.c 的文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/679bc886943e5c018ad5968afb3f009b.png)

- 我们在当前目录下创建的 makefile 里面写下如下代码，echo 前面加了@ 符号，echo 这个命令就不显示：
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4252f28d3b3c7a9ff290a1c791d44968.png)
- 执行结果
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/037c50f1682aee1364df3f74271d2f6e.png)
- 我们得到了 ./a.c 和 ./test/b.c ，所以 wildcard 函数会把我们指定的 ./ 和 ./test/ 目录下的 c 文件展开
### 6.5 notdir 函数（去掉路径显示）

格式：`$(notdir$(var))`
功能：去掉路径。
举例：我们在上面的 makefile 中加上以下代码，因为上面的例子我们得到的结果是 ./a.c 和 ./test/b.c 是有路径的，我们可以直接使用这个变量
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/bd27acd5f07b6091d63edacc1917b257.png)
- 因为 notdir 函数可以去掉路径，所以 /a.c 和 ./test/b.c 去掉路径就得到了 a.c 和 b.c
### 6.6 dir 函数（取出目录）

格式： `$(dir <names...>)`
功能：取出目录，这里的目录指的是最后一个反斜杠/ 之前的部分，如果没有反斜杠/就返回当前。
举例：我们在上面的例子中加入以下代码，如下图所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d9be09e8ac540701e8af38120d165d06.png)
- 因为 var 的值为 ./a.c 和 ./test/b.c ，所以取出目录就是 ./ 和 ./test 
### 6.7 patsubst 函数（替换文件后缀）

格式： `$(patsubst 原文件，目标文件，文件列表）`
功能：替换文件后缀
举例：我们在上面的例子中加入以下代码，如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c31108ede4b8f94b4b3f10af93b64c22.png)
- 这个函数会把 var1 变量的 a.c 和 b.c 的 .c 后缀替换为 .o

也可以写成 `var4=$(var1:%.c=%.o)`
### 6.8 foreach 函数

格式：`$（foreach <var>,<list>,<text>）`
功能：把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`<text>` 所包含的表达式，每一次 `<text>` 会返回一个字符串

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/12dcc1781c94187163db676744c0efd3.png)

- 因为 var2 变量的值为 ./ 和 ./test ，所以先把 ./ 取出来放在 n 变量，然后再执行 wildcard 函数取出 ./ 和 ./test 下面的 c 文件的路径

