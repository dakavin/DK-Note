## 1 Linux基础命令
### 1.1 inux的目录结构

#### 1.1.1 Linux和Windows的目录结构
- Linux的目录结构是一个树型结构
- Windows 系统可以拥有多个盘符, 如 C盘、D盘、E盘![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b152dfe7942e8f1b8073545885eeba70.png)
- Linux没有盘符这个概念, 只有一个根目录 /, 所有文件都在它下面![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a68e8049a776e878b77a6caa7eae3df2.png)
#### 1.1.2 Linux路径的描述方式

- 在Linux系统中，路径之间的层级关系，使用：`/` 来表示
- •在Windows系统中，路径之间的层级关系，使用： `\` 来表示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8735cf1c15f40b864eda5a8996ffd193.png)
### 1.2 Linux命令入门

- 学习Linux，本质上是学习在命令行下熟练使用Linux各类命令
	- 命令行：即Linux终端（Terminal），是一种命令提示符页面。以纯字符的形式操作系统，可以使用各种字符化命令对系统发出操作指令
	- 命令：即Linux程序。一个命令就是一个Linux程序，命令没有图形化界面，可以在命令行（终端）上提供字符化的反馈![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bf98ce6132b0fbfd63041d38e1360c45.png)
#### 1.2.1 Linux命令基础

- 无论是什么命令，用于什么用途，在Linux中，命令有其通用的格式
- 格式：`command [-options] [parameter]`
	- command： 命令本身
	- -options：[可选，非必填]命令的一些选项，可以通过选项控制命令的行为细节
	- parameter：[可选，非必填]命令的参数，多数用于命令的指向目标

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/919b6eb7febc1ee8ec31c562e7e8cfe3.png)
#### 1.2.2 ls命令入门

- ls命令的作用是列出目录下的内容，语法细节如下：
	- `ls [-a -l -h] [linux路径]`
	- -a -l -h 是可选的选项
	- Linux路径是此命令可选的参数
- 当不使用选型和参数，直接使用ls命令本体，表示：以平铺的方式列出当前工作目录下的内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c8a5c965dc6b9c9ae51ea320772e094.png)
- 直接输入ls命令，表示列出当前工作目录下的内容，***当前工作目录是？***
- Linux系统的命令行终端，在启动的时候，默认会加载:
	- 当前登录用户的HOME目录作为当前工作目录，所以ls命令列出的是HOME目录的内容
	- HOME目录：`每个Linux操作用户在Linux系统的个人账户目录`，路径在：`/home/用户名`
		- 如图中的Linux用户是itheima，其HOME目录是：/home/itheima![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/41358838ddefd97e5ae6106a7e504ddd.png)
		- Windows系统和Linux系统，均设有用户的HOME目录，如图：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/73fbb690ad830ee757919b2a3074986c.png)
#### 1.2.3 ls命令的参数和选项 

- 那么ls的选项和参数具体有什么作用呢？
- 首先我们先来看`参数`。
	- 当ls不使用参数，表示列出：当前工作目录的内容，即用户的HOME目录
	- 当使用参数，ls命令的参数表示：指定一个Linux路径，列出指定路径的内容
	- 通过`ls / ` 列出了根目录的内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b3bbda9f02bbcc46ae5be6790964e455.png)
- 再看`选项`
	- `-a选项`，表示：all的意思，即列出全部文件（`包含隐藏的文件/文件夹`）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eca8d4ede91e65c2c7adfbb5f86a2161.png)
	- 在Linux系统中，以”.”开头的文件或文件夹会自动隐藏，只有通过-a选项才可以展示出来

	- `-l选项`，表示：以列表（竖向排列）的形式展示内容，并展示更多信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a37720baf3024cb81e6b64e03f18a9ff.png)
	- 语法中的选项是可以组合使用的，比如学习的-a和-l可以组合应用
		- `ls -l -a  /  ls -la  /  ls -al`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/200f6f3839e47d6ca1e2e75c6b993172.png)
	- 除了选项本身可以组合以外，`选项和参数也可以一起使用`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d0b181c9bb1551420f167f2d031f321.png)
	- `-h 选项` 表示以易于阅读的形式，列出文件大小，如K、M、G
	- -h选项必须要搭配 -l 一起使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ec4b58ea37a1a8daf7542a0d14661d3.png)
### 1.3 目录切换相关命令(cd/pwd)

#### 1.3.1 cd切换工作目录

- 当Linux终端（命令行）打开的时候，会默认以用户的HOME目录作为当前的工作目录
- 我们可以通过cd命令，更改当前所在的工作目录
- cd命令来自英文：Change Directory
- 语法：`cd [LInux路径]`
	- cd命令无需选项，`只有参数，表示要切换到哪个目录下`
	- cd命令直接执行，`不写参数，表示回到用户的HOME目录`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb7b900980d45639b5d899a840b5ae13.png)
cd app  切换到app目录
cd ..      切换到上一层目录
cd /       切换到系统根目录
cd ~      切换到用户主目录
cd -       切换到上一个所在目录

使用tab键来补全文件路径
#### 1.3.2 pwd展示当前工作目录

- 通过ls来验证当前的工作目录，其实是不恰当的。
- 我们可以通过pwd命令，来查看当前所在的工作目录。
- pwd命令来自：Print Work Directory
- 语法：`pwd
- pwd命令，无选项，无参数，直接输入pwd即可![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b5ada68532e382606ac06e5fe6da3b29.png)
### 1.4 相对、决定和特殊路径符

#### 1.4.1 相对和绝对路径的写法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dd002967e3e74921a361eefe4f0a2e66.png)

- 如图，通过pwd得知当前所在是HOME目录：/home/itheima
- 现在想要通过cd命令，切换工作目录到Desktop文件夹中去。
- 那么，cd命令的参数（Linux路径）如何写呢？
	- `绝对路径`：cd /home/dakkk/Desktop
	- `相对路径`：cd Desktop
- 上述两种写法，都可以正确的切换目录到指定的Desktop中

- 绝对路径：`以根目录为起点`，描述路径的一种写法，路径描述`以/开头`

- 相对路径：`以当前目录为起点`，描述路径的一种写法，路径描述`无需以/开头`

#### 1.4.2 几种特殊的路径表示符

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cd25a62a3f69e36a7ce677aff4e60d89.png)

- 如图，当前工作目录处于：/home/itheima/Desktop
- 现在想要，向上回退一级，切换目录到/home/itheima中，如何做？
	- 可以直接通过cd，即可回到HOME目录
	- 也可以通过特殊路径符来完成。

- `特殊路径符`：
	- `.  表示当前目录`，比如 cd ./Desktop 表示切换到当前目录下的Desktop目录内，和cd Desktop效果一致
	
	- `..  表示上一级目录`，比如：cd ..   即可切换到上一级目录，cd ../..  切换到上二级的目录
	
	- `~  表示HOME目录`，比如：cd ~    即可切换到HOME目录或cd ~/Desktop，`切换到HOME内的Desktop目录`
### 1.5 创建目录命令

- 通过mkdir命令可以创建新的目录（文件夹）
- mkdir来自英文：Make Directory
- 语法：`mkdir [-p] Linux路径`
	- `参数必填，表示Linux路径`，即要创建的文件夹的路径，相对路径或绝对路径均可
	- -p选项可选，表示`自动创建不存在的父目录`，适用于创建连续多层级的目录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/458ac21dc01c55ef6992e3f44ff1e8cd.png)

- `mkdir -p 选项`
- 如果想要一次性创建多个层级的目录，如下图![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9827c353fadc7def1ad9bdf0c9390cfb.png)
- 会报错，因为上级目录itcast和good并不存在，所以无法创建666目录
- 可以通过-p选项，将一整个链条都创建完成。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c3c8c27053e7d74f4b5faf42c735172b.png)
- 注意：`创建文件夹需要修改权限`，请确保操作均在HOME目录内，不要在HOME外操作涉及到权限问题，HOME外无法成功
### 1.6 文件操作命令

#### 1.6.1 使用touch创建文件

- 可以通过touch命令创建文件
- 语法：`touch Linux路径`
	- `touch命令无选项，参数必填`，表示要创建的文件路径，相对、绝对、特殊路径符均可以使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa06be76d06ee8e465397c07fefb9d7c.png)
#### 1.6.2 使用cat、more查看文件内容

- 有了文件后，我们可以`通过cat命令查看文件的内容`
- 不过，现在我们还未学习vi编辑器，无法向文件内编辑内容，所以，暂时，我们先通过图形化
- 在图形化中，手动向文件内添加内容，以测试cat命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28e595d61cc5669bc7370e3c11c97e0e.png)
- 准备好文件内容后，可以通过cat查看内容。
- 语法：`cat Linux路径`
	- cat同样没有选项，只有必填参数，参数表示：被查看的文件路径，相对、绝对、特殊路径符都可以使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/96fdadde5d8c0214e122088ecbe1f649.png)
- more命令同样可以查看文件内容，同cat不同的是：
	- `cat是直接将内容全部显示出来`
	- `more支持翻页`，如果文件内容过多，可以一页页的展示

- 语法：`more Linux路径`
	- 同样没有选项，只有必填参数，参数表示：被查看的文件路径，相对、绝对、特殊路径符都可以使用

- Linux系统内置有一个文件，路径为：/etc/services，可以使用more命令查看
- more /etc/services
	- 在查看的过程中，`通过空格翻页`
	- `通过q退出查看`


- less用法和more类似，不同的是less可以通过PgUp、PgDn键来控制
	- less yum.conf
	- PgUp 和 PgDn 进行上下翻页


- `tail命令是在实际使用过程中使用非常多的一个命令`，它的功能是：`用于显示文件后几行的内容`。
	- tail -10 /etc/passwd    查看后10行数据
	- tail -f catalina.log    `动态查看日志(*****)`
	- `ctrl+c 结束查看`
#### 1.6.3 使用cp复制文件/文件夹

- cp命令可以用于复制文件或文件夹，cp命令来自英文单词：copy
- 语法：`cp [-r] 参数1 参数2`
	- -r选项，可选，用于复制文件夹使用，表示递归
	- 参数1，Linux路径，表示`被复制的文件或文件夹`
	- 参数2，Linux路径，表示`要复制去的地方`
- 复制文件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5b0126f505c44dd03eb1b214fa711f2.png)
- 复制文件夹![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb188fb6bfc9d2a20e51eaecf4598ee2.png)
- `复制文件夹，必须使用-r选项，否则不会生效`
#### 1.6.4 使用mv移动文件/文件夹

- mv命令可以用于移动文件或文件夹，mv命令来自英文单词：move
- 语法：`mv 参数1 参数2`
	- 参数1，Linux路径，表示`被移动的文件或文件夹`
	- 参数2，Linux路径，表示`要移动去的地方，如果目标不存在，则进行改名，确保目标存在`
- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b95c1670268368210233c3046515360.png)
- 如下图，目标不存在，则有改名的效果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b5a50ff61a5a459614f297661b4f685.png)
#### 1.6.5 使用rm删除文件/文件夹

- rm命令可用于删除文件、文件夹
- rm命令来自英文单词：remove
- 语法：`rm [-r -f] 参数1 参数2 ... 参数N`
	- 同cp命令一样，-r选项用于删除文件夹
	- `-f表示force`，强制删除（不会弹出提示确认信息）
		- 普通用户删除内容不会弹出提示，只有root管理员用户删除内容会有提示所以一般普通用户用不到-f选项
	- 参数1、参数2、......、参数N 表示要删除的文件或文件夹路径，`按照空格隔开`

- `删除文件`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7a1cfee505845d86af8e59a663b06b0a.png)
- `删除多个文件`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bfa6c32dd17ae5df7f3a7254a7761e80.png)
- `删除文件夹`，如下图，必须使用-r选项才可以![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a1cc012c1d13fb2d87761549dd3450c5.png)


- 演示`强制删除`，-f选项
- 可以通过 su - root，并输入密码（和普通用户默认一样）临时切换到root用户体验
- 通过输入exit命令，退回普通用户。（临时用root，用完记得退出，不要一直用，关于root我们后面会讲解）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b27d206bbda8acd48618cd57630c0dd.png)


- `rm命令支持通配符 *`，用来做模糊匹配
- 符号* 表示通配符，即匹配任意内容（包含空）
- 示例：
	- `test*`，表示匹配任何`以test开头`的内容
	- `*test`，表示匹配任何`以test结尾`的内容
	- `*test*`，表示匹配`任何包含test`的内容
- •删除所有以test开头的文件或文件夹![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1249ffa30d6d8403cbc233c1e2490209.png)

- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a15b8052d773c4add37bce746f37ae1d.png)

### 1.7 查找命令
#### 1.7.1 which命令

- 我们在前面学习的Linux命令，其实它们的本体就是一个个的二进制可执行程序。
- 和Windows系统中的.exe文件，是一个意思
- 我们可以通过which命令，`查看所使用的一系列命令的程序文件`存放在哪里
- 语法：`which 要查找的命令`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4dfda9644c41cd5447086566afff3d38.png)

#### 1.7.2 find命令 - 按文件名查找文件

- 在图形化中，我们可以方便的通过系统提供的搜索功能，搜索指定的文件。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ebcf76bdf99e3d76605c650f3a5ace7e.png)


- 同样，在Linux系统中，我们可以通过find命令去搜索指定的文件。

- `按照文件名查找文件`
- 语法：`find 起始路径 -name "被查找文件名"`
- 注意使用root权限
- 查找文件名叫做：test的文件，从根目录开始搜索 
- `find / -name “test”`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/81cf724093811c07c013943324105674.png)



- 被查找文件名，`支持使用通配符 * 来做模糊查询`。
- 符号* 表示通配符，即匹配任意内容（包含空），示例：
	- `test*`，表示匹配任何以test开头的内容
	- `*test`，表示匹配任何以test结尾的内容
	- `*test*`，表示匹配任何包含test的内容
- 基于通配符的含义，可以结合find命令做文件的模糊查询。

- `按文件大小查找文件`
- 语法：`find 起始路径 -size +|-n[kMG]`
	- +、- 表示大于和小于
	- n表示大小数字
	- kMG表示大小单位，k(小写字母)表示kb，M表示MB，G表示GB
- 示例：
	- 查找小于10KB的文件： find / -size -10k
	- 查找大于100MB的文件：find / -size +100M
	- 查找大于1GB的文件：find / -size +1G
### 1.8 grep、wc和管道符

#### 1.8.1 使用grep过滤文件内容

- 可以通过grep命令，从文件中通过关键字过滤文件行。
- 语法：`grep [-n] 关键字 文件路径`
	- 选项-n，可选，表示在结果中显示匹配的行的行号
	- 关键字，必填，表示过滤的关键字，`带有空格或其它特殊符号，建议使用””将关键字包围起来`
	- 文件路径，必填，表示要过滤内容的文件路径，可作为内容输入端口
- 现在，通过touch命令在HOME目录创建itheima.txt，并通过图形化页面编辑并保存如下内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e94cae82ed76b782ad7a49ede9978817.png)


	- 过滤itheima关键字![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b84b38967d15b73b4e5173e4d356d38a.png)


	- 过滤itcast关键字![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e35b1ac3e04db14be65292c0c46b50bd.png)


	- 过滤code关键字，并显示行号![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b5c260cb83bc2d284f7717c4ec0f5e1.png)


#### 1.8.2 使用wc统计内容数量

- 可以通过wc命令统计文件的行数、单词数量等
- 语法：`wc [-c -m -l -w] 文件路径`
	- -c，统计bytes数量
	- -m，统计字符数量
	- -l，统计行数
	- -w，统计单词数量
	- 文件路径，被统计的文件，可作为内容输入端口

	- 不带选项，统计文件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/256e4aeb882754b204818b2a92020330.png)


	- 统计字节数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/21e4e3499ad9266de4537edbc22cc0f2.png)


	- 统计字符数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb8e6cce50e7a695d1e88116de75c8ca.png)


	- 统计行数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c19ec7499acfad43e094412b15df4a2a.png)


	- 统计单词数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/475def581bf728a062d8ee3c5fa57a72.png)


#### 1.8.3 | 管道符的概念和应用

- 管道符的含义是：将管道符`左边命令的结果，作为右边命令的输入`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e6e2d87ef0e4fe066545259293c6a88a.png)

- 如上图：
	- cat itheima.txt的输出结果（文件内容）
	- 作为右边grep命令的输入（被过滤文件）

- 管道符的应用非常多
	- ls | grep Desktop，过滤ls的结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d209d26ad90661c782d96f06ef35cbeb.png)


	- find / -name “test” | grep “/usr/lib64”，过滤结果，只找路径带有/usr/lib64的结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb04804ac45fc68546d3c94bbfc302ec.png)


	- cat itheima.txt | grep itcast | grep itheima，`可以嵌套使用哦`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2fb9c37de9b70c19e151ca5994e12575.png)
		- cat itheima.txt的结果给 grep itcast 使用
		- cat itheima.txt | grep itcast 的结果给 grep itheima使用
### 1.9 echo和重定向符

#### 1.9.1 使用echo命令输出内容

- 可以使用echo命令在命令行内输出指定内容
- 语法：`echo 输出的内容`
	- 无需选项，只有一个参数，表示要输出的内容，复杂内容可以用””包围
	-  在终端上显示：Hello Linux![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d871d3f40f3df3ed2d65c01d675f21f.png)


	- `带有空格或\等特殊符号，建议使用双引号包围`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9906d52951c79ffebcfe6cafb88b7035.png)
		- 因为不包围的话，空格后很容易被识别为参数2，尽管echo不受影响，但是要养成习惯哦

#### 1.9.2 反引号\`的使用

- 看一下如下命令：echo pwd![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8f3b964fee2305e11b53be6f7cab9f83.png)


- 本意是想，输出当前的工作路径，但是pwd被作为普通字符输出了。
- 我们可以通过将命令用反引号（通常也称之为飘号）\`将其包围
- 被\`包围的内容，会被作为命令执行，而非普通字符![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79820526556be4557464ed4b230bd8cb.png)


#### 1.9.3 tail命令跟踪文件更改

- 使用tail命令，可以查看文件尾部内容，跟踪文件的最新更改
- 语法：`tail [-f -num] Linux路径`
	- 参数，Linux路径，表示被跟踪的文件路径
	- 选项，-f，表示持续跟踪
	- 选项, -num，表示，查看尾部多少行，不填默认10行

- 查看/var/log/vmware-network.log文件的尾部10行：tail /var/log/vmware-network.log![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/841f383386bb2102e5b0e649900cccc4.png)


- 查看/var/log/vmware-network.log文件的尾部3行：tail -3 /var/log/vmware-network.log![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/51ebd47f6745387e957d95b78ac171d1.png)



#### 1.9.4 重定向符号的使用

- 重定向符：>和>>
	- >，将左侧命令的结果，`覆盖写入`到符号右侧指定的文件中
	- >>，将左侧命令的结果，`追加写入`到符号右侧指定的文件中

- echo “Hello Linux” > itheima.txt![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6be5f1628e1a1d0adcad3e56e9b75415.png)


- echo “Hello itheima” > itheima.txt，再次执行，覆盖新内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e40c902ef2a07de833927d08113747d.png)


- echo “Hello itcast” >> itheima.txt，再次执行，使用>>追加新内容![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/430be7445bf56d53010e10ac95105456.png)


### 1.10 系统管理命令

- ps 正在运行的某个进程的状态
	- ps –ef  查看所有进程
	- ps –ef | grep ssh 查找某一进程
	- kill 2868  杀掉2868编号的进程
	- kill -9 2868  强制杀死进程
## 2 Vi和Vim编辑器

### 2.1 概念

- vi或vim是visual interface的简称, 是Linux中最经典的文本编辑器
- 同图形化界面中的 文本编辑器一样，`vi是命令行下对文本文件进行编辑的绝佳选择`
- `vim 是 vi 的加强版本，兼容 vi 的所有指令，不仅能编辑文本，而且还具有 shell 程序编辑的功能`，可以不同颜色的字体来辨别语法的正确性，极大方便了程序的设计和编辑性

### 2.2 三种工作模式

- `三种工作模式`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2fd4d2dab42617f32e88447a1cc23d2a.png)


- `命令模式（Command mode）`
	- 命令模式下，所敲的按键编辑器都理解为命令，以命令驱动执行不同的功能
	- 此模型下，不能自由进行文本编辑
- `输入模式（Insert mode）`
	- 也就是所谓的编辑模式、插入模式
	- 此模式下，可以对文件内容进行自由编辑
- `底线命令模式（Last line mode）`
	- 以：开始，通常用于文件的保存、退出

- `命令模式`
	- 如果需要通过vi/vim编辑器编辑文件，请通过命令：`vi/vim 文件路径`
		- 如果文件路径表示的`文件不存在`，那么此命令会用于`编辑新文件`
		- 如果文件路径表示的`文件存在`，那么此命令用于`编辑已有文件`

### 2.3 vi编辑器的快速体验

- 执行顺序：进入文件（vim filename）--->命令模式---> 输入i/a/o--->输入模式--->esc键--->返回命令模式--->：--->进入底线命令模式--->回车键--->返回命令模式--->输入：wq--->退出编辑器

- 快速体验
	1. 使用：vim hello.txt，编辑一个新文件，执行后进入的是命令模式
	2. 在命令模式内，按键盘 i ，进入输入模式
	3. 在输入模式内输入：itheima and itcast.
	4. 输入完成后，按esc回退会命令模式
	5. 在命令模式内，按键盘 : ，进入底线命令模式
	6. 在底线命令内输入：wq，保存文件并退出vi编辑器

### 2.4 vim快捷键

- 命令模式下的一些常见快捷键

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bcd23a061f00e028fd9eef603db68f3c.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1102ed7bf3dbb07baaf765fd8b308c50.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e4d3a6ed2fc47d1820c53efe15054a96.png)


- 底线命令模式
	- 编辑模式没有什么特殊的，进入编辑模式后，任何快捷键都没有作用，就是正常输入文本而已
	- 唯一大家需要记住的，就是：通过esc，可以退回到命令模式中即可
	- 在命令模式内，输入: ，即可进入底线命令模式，支持如下命令![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7fbfccb2c7d77dd70fcc94e07d376b23.png)

### 2.5 补充：关于命令选项的说明

- 如果想要对命令的其它选项进行查阅，可以通过如下方式
	- 任何命令都支持：--help 选项， 可以通过这个选项，查看命令的帮助。
	- 如：ls --help， 会列出ls命令的帮助文档![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3eec9dc092000714827d8096d0f2f3ae.png)
- 如果想要查看命令的详细手册，可以通过man（manual， 手册）命令查看
	- man ls，就是查看ls命令的详细手册
	- man cd，就是查看cd命令的详细手册
- 大多数手册都是全英文的，如果阅读吃力，可以通过重定向符：man ls > ls-man.txt，输出手册到文件，然后通过翻译软件翻译内容查:看哦