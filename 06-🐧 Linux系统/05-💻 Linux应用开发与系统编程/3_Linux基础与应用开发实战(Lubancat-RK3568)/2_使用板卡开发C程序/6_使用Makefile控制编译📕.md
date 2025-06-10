---
文章标题: "[[6_使用Makefile控制编译📕]]"
文章作者: Dakkk
文章概要: |
  介绍Linux环境下Makefile编译控制工具的基础语法、变量使用、函数调用和分支控制，通过实例演示如何编写高效的编译脚本管理多文件项目。
tags:
  - Makefile
  - Linux编译
  - GCC
  - 自动化构建
  - 变量函数
  - 伪目标
  - 依赖管理
  - 交叉编译
相关文章:
  - "[[../../../01-❇️ 参考资料/0_个人实用技巧/6_Makefile文件示例]]"
  - "[[../../../../02-💻 开发环境/2_环境、启动和编译烧录相关(迅为)/3_编译与烧写（保留，直接看Lubancat板卡即可）]]"
  - "[[1_初识C语言]]"
  - "[[1_搭建多模块工程（Spring Initializr）]]"
  - "[[1_Maven入门和进阶]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/2_使用板卡开发C程序/6_使用Makefile控制编译📕.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 17:05:46
修改时间: 2025-05-28 00:19:51
---


关于Makefile的详细使用可参考《跟我一起写Makefile》一书或GNU官方的make说 明文档：[https://www.gnu.org/software/make/manual](https://www.gnu.org/software/make/manual)，本章仅以示例对Makefile的基础语法进行讲解。

LubanCat-RK系列板卡出厂并没有自带Makefile工具，需要我们自行下载安装
```shell
sudo apt install make
```

本章的示例代码目录为：`base_linux/makefile/`
## 1 Makefile小实验

**makefiel的基本格式**
```makefile
目标:依赖的文件或其他目标
#注意:命令前用tab
	命令1
	命令2
#第一个目标，是最终目标及make的默认目标
```

首 先使用编辑器创建一个名为“Makefile”的文件，输入如下代码并保存，其中使用“#”开头的 行是注释，自己做实验时可以不输入，另外要注意在“ls -lh”、”touch test.txt”等命令前要使用Tab键，不能使用空格代替。
```makefile
#目标a，依赖与目标targetc和targetb
#目标要执行shell命令 ls -lh
targeta:targetc targetb
	ls -lh

#目标b，无依赖
#目标要执行shell命令 touch创建文件
targetb:
	touch test.txt

#目标c，无依赖
#目标要执行shell命令，pwd
targetc:
	pwd

#目标d，无依赖
#由于abc目标不依赖于目标d，所以直接make时，目标d不会被执行
#不过可以使用make targetd命令执行
targetd:
	rm -f test.txt
```

这个Makefile文件主要是定义了四个目标 操作，先大致了解它们的关系：
- `targeta`：这是Makefile中的第一个目标代号，在符号“:”后 面的内容表示它依赖于targetc和targetb目标，它自身的命令为“ls -lh”，列出当前目录下的内容。
- `targetb`：这个目标没有依赖其它内容，它要执行的命令为“touch test.txt”，即创建一个test.txt文件。
- `targetc`：这个目标同样也没有依赖其它内容，它要执行的命令为“pwd”，就是简单地显示当前的路径。
- `targetd`：这个目标无依赖其它内容，它要执行的命令为“rm -f test.txt”，删除 目录下的test.txt文件。与targetb、c不同的是，没有任何其它目标依赖于targetd，而且 它不是默认目标。

下面使用这个Makefile执行各种make命令，对比不同make命令的输出，可以清楚地了解Makefile的机制。 在主机Makefile所在的目录执行如下命令：
```c
#在主机上Makefile所在的目录执行如下命令
#查看当前目录的内容
ls

#执行make命令，make会在当前目录下搜索“Makefile”或“makefile”，并执行
make

#可看到make命令后的输出，它执行了Makefile中编写的命令

#查看执行make命令后的目录内容，多了test.txt文件
ls

#执行Makefile的targetd目标，并查看，少了test.txt文件
make targetd
ls

#执行Makefile的targetb目标，并查看，又生成了test.txt文件
make targetb
ls

#执行Makefile的targetc目标
make targetc
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/197bcefe3dceec9ff25a0348605d4f77.png)

**make命令：**
- 在终端上执行make命令时，make会在当前目录下搜索名为“Makefile”或“makefile”的文件，然后 根据该文件的规则解析执行。如果要指定其它文件作为输入规则，可以通过“-f”参数指定输 入文件，如“make -f 文件名”。
- 此处make命令读取我们的Makefile文件后，发现targeta是Makefile的第一个目标，它会被当成默认目标执行。
- 又由于targeta依赖于targetc和targetb目标，所以在执行targeta自身的命令之前，会先去完成targetc和targetb。
- targetc的命令为pwd，显示了当前的路径。
- targetb的命令为touch test.txt ，创建了test.txt文件。
- 最后执行targeta自身的命令ls -lh ，列出当前目录的内容，可看到多了一个test.txt文件

**make targetd 、make targetb、make targetc命令：**
- 由于targetd不是默认目标，且不被其它任何目标依赖，所以直接make的时 候targetd并没有被执行，想要单独执行Makefile中的某个目标，可以使用”make 目标 名“的语法，例如上图中分别执行了”make targetd“ 、”make targetb“ 和”make targetc“指令，在执行”make targetd”目标时，可看到它的命令rm -f test.txt被执行，test.txt文件被删除。

从这个过程，可了解到make程序会根据Makefile中描述的目标与依赖关系，执行达成目标需要的shell命令。`简单来说，Makefile就是用来指导make程序如何干某些事情的清单。`
## 2 使用Makefile编译程序

### 2.1 使用GCC编译多个文件

接着我们使用Makefile来控制程序的编译，为方便说明，先把前面章节 的hello.c程序分开成三个文件来写，分别为hello_main.c主文件，hello_func.c函数文 件，hello_func.h头文件，其内容如下代码所示
```c
// base_linux/makefile/test2/hello_main.c
#include "hello_func.h"

int main()
{
    hello_func();
    return 0;
}

// base_linux/makefile/test2/hello_func.c
#include <stdio.h>
#include "hello_func.h"

void hello_func(void)
{
    printf("hello, world! This is a C program.\n");
    for (int i=0; i<10; i++ ) {
        printf("output i=%d\n",i);
    }
}

// base_linux/makefile/test2/hello_func.h
void hello_func(void);
```

也就是说hello_main.c的main主函数调用了hello_func.c文件的打 印函数，而打印函数在hello_func.h文件中声明，在复杂的工程中这是常见的程序结构。

如果我们直接使用GCC进行编译，需要使用如下命令：
```shell
# -I .  用于告诉编译器头文件的路径是在当前目录（.）
gcc -I . hello_main.c hello_fun.c -o hello_main

#运行生成的hello_main程序
./hello_main
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/289dc1eeb975f18c61e2512aa5a85cf5.png)

**新增的“-I .”是告诉编 译器头文件路径，让它在编译时可以在“.”（当前目录）寻找头文件，其实不加”-I .”选项也是能正常编译通过的，此处只是为了后面演示Makefile的相关变量。**
### 2.2 使用Makefile编译

可以想像到，只要把gcc的编译命令按格式写入到Makefile，就能直接 使用make编译，而不需要每次手动直接敲gcc编译命令。

操作如下使用编辑器在hello_main.c所在的目录新建一个名为“Makefile”的文件，并 输入如下内容并保存。
```makefile
# 默认目标
# hello_main依赖于hello_main.c和hello_func.c文件
hello_main: hello_main.c hello_func.c
	gcc -o hello_main hello_main.c hello_func.c -I .

#clean目标，用来删除编译生成的文件
clean:
	rm -f *.o hello_main
```

该文件定义了**默认目标hello_main用于编译程序**，**clean目标用于删除 编译生成的文件**。

特别地，其中hello_main目标名与gcc编译生成的文件名”gcc -o hello_main”设置成一致了，也就是说，此处的目标hello_main在Makefile看来，已经是 一个目标文件hello_main。

这样的好处是
- make每次执行的时候，会`检查`hello_main文件和依赖 文件hello_main.c、hello_func.c的修改日期
- 如果依赖文件的`修改日期`比hello_main文件的 日期`新`，那么make会执行目标其下的Shell命令更新hello_main文件
- 否则不会执行。

请运行如下命令进行实验：
```shell
#在主机上Makefile所在的目录执行如下命令
#若之前有编译生成hello_main程序，先删除

rm hello_main
ls

#使用make根据Makefile编译程序
make
ls

#执行生成的hello_main程序
./hello_main

#再次make，会提示hello_main文件已是最新
make

#使用touch命令更新一下hello_func.c的时间
touch hello_func.c

#再次make，由于hello_func.c比hello_main新，所以会再编译
make
ls
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/11abfe1989bbf07116599fcf608d81d2.png)

如上图所示，有了Makefile后，我们实际上只需要执行一下make命令就可以完成 整个编译流程。

图中还演示了make会对目标文件和依赖进行更新检查，当依赖文件 有改动时，才会再 次执行命令更新目标文件。
## 3 目标、依赖和命令

下面我们再总结一下Makefile中跟目标相关的语法：
```makefile
[目标1]：[依赖]
	[命令1]
	[命令2]

[目标2]：[依赖]
	[命令1]
	[命令2]
```

**目标：**
- make要做的事情，可以是简单的代码，也可以是目标文件
- 需要顶格书写，不能有空格或Tab
- 一个Makefile可以有多个目标，写在最前面的目标会被Make程序作为 `默认目标`

**依赖：**
- 达成目标依赖的某些文件或其他目标

**命令：**
- make达成目标所需要的命令
- 只有目标不存在 或 依赖文件的修改时间比目标文件新 才会执行命令
- 特别注意命令的开头要用tab键
## 4 伪目标📕

**问题：** 在test1的时候，假如当前目录下存在targeta、targetb、targetc文件，并且都是最新的，那么make targeta就不会正常执行了

**解决：** Makefile使用`.PHONY`前缀来区分目标代号（`伪目标`）和目标文件，phony单词翻译本身就是假的意思
- 即不期待生成目标文件，就应该将其定义为 **伪目标** 

**修改test1的Makefile**
```makefile
# 使用.PHONY表示targeta是个伪目标
.PHONY:targeta
targeta: targetc targetb
	ls -lh

#使用.PHONY表示targetb是个伪目标
.PHONY:targetb
targetb:
	touch test.txt

#使用.PHONY表示targetc是个伪目标
.PHONY:targetc
targetc:
	pwd

#使用.PHONY表示targetd是个伪目标
.PHONY:targetd
targetd:
	rm -f test.txt
```

**修改test2的Makefile**
```makefile
# 默认目标
# hello_main依赖于hello_main.c和hello_func.c文件
hello_main: hello_main.c hello_func.c
	gcc -o hello_main hello_main.c hello_func.c -I .

#clean伪目标，用来删除编译生成的文件
.PHONY:clean
clean:
	rm -f *.o hello_main
```

GNU组织发布的软件工程代码的Makefile，常常会有类似以上代码中定义的clean伪目标，用于清 除编译的输出文件。常见 的还有“all”、“install”、“print”、“tar”等分别用于编译所有内容、安装已 编译好的程序、列出被修改的文件及打包成tar文件。虽然并没有固定的要求伪目标必须用这些 名字，但可以参考这些习惯来编写自己的Makefile。

如果以上代码中不写“.PHONY:clean”语句，并且在目录下创建一个名为clean的文件，那么当 执行“make clean”时，clean的命令并不会被执行，感兴趣的可以亲自尝试一下。
## 5 默认规则📕

### 5.1 简介

在[2_程序编译GCC📕](2_程序编译GCC📕.md) 章节中提到整个编译过程包含下图的步骤，make在执行时也是同样的流程，**不过在Makefile的实际应用中，通常会把 编译 和 链接 过程分开**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/bf82b8caca504bad282b0f6c9aa3b813.png)
- 即hello_main目标文件本质上是依赖 `hello_main.o` 和 `hello_fun.o` 这两个文件，它两链接起来就能得到最终的hello_main目标文件
- 另外，**make有一个默认规则，当找不到 xxx.o 文件时，会查找目录下同名的 xxx.c 文件进行编译**

根据这个默认规则，我们可以吧Makefile修改如下：
```makefile
hello_main:hello_main.o hello_fun.o
	gcc hello_main.o hello_fun.o -o hello_main

# 以下是make的默认规则，可以不写
hello_main.o:hello_main.c
	gcc hello_main.c -o hello_main.o

hello_fun.o:hello_fun.c
	gcc hello_fun.c -o hello_fun.o
```

- 将依赖文件由c文件改为o文件，gcc编译命令也做了相应修改
- 由于c编译成同名的o文件是make的默认规则，所以上面4-9行内容可以不写

使用修改后的Makefile编译结果如下所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/606bd0a156ae43c8b42c4996c192cefd.png)

- 可以看到，执行了两条额外的 `cc` 编译命令
- 这是由make默认规则执 行的，它们把C代码编译生成了同名的.o文件
- 然后make根据Makefile的命令链接这两 个文件得到最终目标文件hello_main

### 5.2 缺陷📕

`缺陷1：`使用C自动编译成*.o的默认规则有个缺陷，由于没有显式地表示*.o依赖于.h头文 件，假如我们**修改了头文件的内容，那么*.o并不会更新**，这是不可接受的

`缺陷2：`默认规则使用固定的“cc”进行编译，假如我们想使用ARM-GCC进行交叉编译，那么系统默 认的“cc”会导致编译错误。

`解决：`要解决这些问题并且让Makefile变得更加通用，需要引入变量和分支进行处理。

## 6 使用变量
### 6.1 基本语法

在Makefile中的变量，有点像 C语言的宏定义，在引用变量的地方使用变量 值进行替换。变量的命名可以包含字符、数字、下划线，区分大小写，定义变量的方式有以下四种：
- `=`：延时赋值，只有在调用的时候，才会被赋值
- `:=`：直接复制，变量的值在定义时已经确定了，和延时赋值相反
- `?=`：若变量的值为空，则赋值，通常用于设置默认值
- `+=`：追加赋值，往变量后面增加新的内容

**使用变量的语法如下**
```makefile
$(变量名)
```

**举例：** 讲解这四种定义方式，对于后两种赋值方式 比较简单，主要思考延时赋值和直接赋值的差异，实验代码如下所示。

```makefile
VAR_A = FILEA
VAR_B = $(VAR_A)
VAR_C := $(VAR_A)
VAR_A += FILEB
VAR_D ?= FILED
.PHONY:check
check:
	@echo VAR_A=$(VAR_A)  
	@echo VAR_B=$(VAR_B)  
	@echo VAR_C=$(VAR_C)  
	@echo VAR_D=$(VAR_D)  
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/957bfbbb12bbdae380b1842482714d55.png)

- VAR_A：追加赋值，所以为 FILEA FILEB
- VAR_B：延时赋值，只用在调用的时候才赋值，所以是A变量修改后的值，FILEA FILEB
- VAR_C：立即赋值，所以是A变量修改前的值，FILEA
- VAR_D：变量为空，赋值为 FILED
### 6.2 改造默认规则

接下来使用变量对前面的hello_main的Makefile进行改造，如下
```makefile
#定义变量
CC=gcc
CFLAGS=-I .
DEPS=hello_fun.h

#目标文件
hello_main:hello_main.o hello_fun.o
	$(CC) -o hello_main hello_main.o hello_fun.o 

# *.o文件生成规则
%.o:%.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean
clean:
	rm -f *.o hello_main
```

- 第1~4行：定义变量
- 第8行：使用$(CC)替换gcc，便于后续更换不同的编译器；若交叉编译，只要把CC修改即可
- 第11行：
	- `%`是通配符，类似于`*` 
	- `%.o:%.c`表示，o文件依赖与c文件的默认规则，还依赖变量DEPS表示的头文件
- 第12行：
	- 特殊的变量`$@，$<`，可理解为Makefile文件保 留的关键字，是系统保留的自动化变量
	- `$@`代表了目标文件，即 `%.o`
	- `$<`代表了第一个依赖文件，即`%.c`
- 假如第11行%匹配的字符是hello_fun时，第12行的代码如下
```makefile
#当"%"匹配的字符为"hello_func"的话：
$(CC) -c -o $@ $< $(CFLAGS)
# 等价与
gcc -c -o hello_fun.o hello_fun.c -I .
```

也就是说makefile可以利用变量及自动化变量，来重写.o文件的默认生成 规则，以及增加头文件的依赖
### 6.3 改造链接规则

与*.o文件的默认规则类似，我们也可以使用变量来修改生成最终目标 文件的链接规则，具体参考如下代码。

```c
#定义变量
TARGET=hello_main
CC=gcc
CFLAGS=-I .
DEPS=hello_fun.h
OBJS=hello_main.o hello_fun.o

#目标文件
$(TARGET):$(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

# *.o文件生成规则
%.o:%.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean
clean:
	rm -f *.o hello_main
```

- 第2行：定义TARGET变量，表示目标文件名
- 第6行：定义OBJS变量，表示依赖的各个o文件
- 第9行：使用变量替换目标文件和依赖文件
- 第10行：使用自动化变量`$@`表示目标文件`$(TARGET)`，使用自动化变量`$^`表示所有的依赖文件即`$(OBJS)`

也就是说以上代码中的Makefile把编译及链接的过程都通过变量表示出来了，非常通用。 **使用这样的Makefile可以针对不同的工程直接修改变量的内容就可以使用。**
### 6.4 其他自动化变量（查阅）

Makefile中还有其它自动化变量，此处仅列出方便以后使用到的时候进行查阅，见下表。

|符号|意义|
|---|---|
|$@|匹配目标文件|
|$%|与$@类似，但$%仅匹配“库”类型的目标文件|
|$<|依赖中的第一个目标文件|
|$^|所有的依赖目标，如果依赖中有重复的，只保留一份|
|$+|所有的依赖目标，即使依赖中有重复的也原样保留|
|$?|所有比目标要新的依赖目标|
## 7 使用函数

在更复杂的工程中，头文件、源文件可能会放在二级目录，编译生成的*.o或 可执行文件也放到专门的编译输出目录方便整理，如下图所示。示例中*.h头文件 放在includes目录下，`*.c`文件放在sources目录下，编译输出存 放在build中。

实现这些复杂的操作通常需要使用Makefile的函数。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a04f8c1dfe8f5b0d220b5508b0f6725e.png)

### 7.1 函数格式及示例

在Makefile中调用函数的方法跟变量的使用 类似，以“$()”或“${}”符号包含函数名和参数，具体语法如下：
```makefile
$(函数名 参数)
#或者使用花括号
${函数名 参数}
```

下面以常用的notdir、patsubst、wildcard函数为例 进行讲解，并且示例中都是我们后面Makefile中使用到的内容。
#### 7.1.1 notdir函数

其他参考：[6.5 notdir 函数（去掉路径显示）](../../../../04-⚡%20嵌入式基础能力/1_Linux入门基础(迅为).md#6.5%20notdir%20函数（去掉路径显示）)

notdir函数用于**去除文件路径中的目录部分**。它的格式如下：
```makefile
$(notdir 文件名)
```

例如输入参数“./sources/hello_func.c”，函数执行后 的输出为“hell_func.c”，也就是说它会把输入中的“./sources/”路径部分去掉，保留 文件名。使用范例如下：
```makefile
#以下是范例
$(notdir ./sources/hello_func.c)

#上面的函数执行后会把路径中的“./sources/”部分去掉，输出为： hello_func.c
```
#### 7.1.2 wildcard函数

其他参考：[6.4 wildcard 函数（展开指定目录）](../../../../04-⚡%20嵌入式基础能力/1_Linux入门基础(迅为).md#6.4%20wildcard%20函数（展开指定目录）)

wildcard函数用于`获取文件列表，并使用空格分隔开`。它的格式如下：
```makefile
$(wildcard 匹配规则)
```

例如函数调用 `$(wildcard *.c)` ，函数执行后会把当前目录的所 有c文件列出。假设我们在上图中的Makefile目录下执行该函数，使用范例如下：
```makefile
#在sources目录下有hello_func.c、hello_main.c、test.c文件
#执行如下函数
$(wildcard sources/*.c)
#函数的输出为：
sources/hello_func.c sources/hello_main.c sources/test.c
```
#### 7.1.3 patsubst函数

其他参考：[6.7 patsubst 函数（替换文件后缀）](../../../../04-⚡%20嵌入式基础能力/1_Linux入门基础(迅为).md#6.7%20patsubst%20函数（替换文件后缀）)

patsubst函数功能为`模式字符串替换`。它的格式如下：
```makefile
$(patsubst 匹配规则, 替换规则, 输入的字符串)
```

当输入的字符串符合匹配规则，那么使用替换规则来替换字符串，当匹配规则中有“%”号时，替换规 则也可以例程“%”号来提取“%”匹配的内容加入到最后替换的字符串中。有点抽象，请直接阅读以下示例：
```makefile
$(patsubst %.c, build_dir/%.o, hello_main.c )
#函数的输出为：
build_dir/hello_main.o

#执行如下函数
$(patsubst %.c, build_dir/%.o, hello_main.xxx )
#由于hello_main.xxx不符合匹配规则"%.c"，所以函数没有输出
```

- 第一个函数调用中，由于“hello_main.c”符合“%.c”的匹配规则（%在Makefile中的类似于`*`通配符），而且“%”从“hello_main.c”中提取出了“hello_main”字符，把这部分内容放到替换规则“build_dir/%.o”的“%”号中，所以最终的输出为”build_di r/hello_main.o”

- 第二个函数调用中，由于由于“hello_main.xxx”不符合“%.c”的匹配规则，“.xxx”与“.c”对不上，所以不会进行替换，函数直接返回空的内容
#### 7.1.4 其他函数

- [6.6 dir 函数（取出目录）](../../../../04-⚡%20嵌入式基础能力/1_Linux入门基础(迅为).md#6.6%20dir%20函数（取出目录）)
- [6.8 foreach 函数](../../../../04-⚡%20嵌入式基础能力/1_Linux入门基础(迅为).md#6.8%20foreach%20函数)
### 7.2 多级结构工程的Makefile

接下来我们使用上面三个函数修改我们的Makefile，以适应包含多级目录的工程，修改后的内容如下所示
```makefile
#定义变量
TARGET=hello_main
#存放中间文件的路径
BUILD_DIR=build
#存放源文件的文件夹
SRC_DIR=src
#存放头文件的文件夹
INC_DIR=includes .
#源文件
SRCS=$(wildcard $(SRC_DIR)/*.c)
#目标文件
OBJS=$(patsubst %.c,$(BUILD_DIR)/%.o,$(notdir $(SRCS)))
#头文件
DEPS = $(foreach dir,$(INC_DIR),$(wildcard $(dir)/*.h))
#指定头文件的路径
CFLAGS=$(patsubst %,-I%,$(INC_DIR))

#目标文件
$(BUILD_DIR)/$(TARGET):$(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

# *.o文件生成规则
$(BUILD_DIR)/%.o:$(SRC_DIR)/%.c $(DEPS)
#创建一个编译目录，用于存放过程文件
#命令前带@，表示不在终端上输出
	@mkdir -p $(BUILD_DIR)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean cleanall
#删除输出文件夹
clean:
	rm -rf $(BUILD_DIR)
#全部删除
cleanall:
	rm -rf $(BUILD_DIR)
```

分析如下：
- 第4-8行：
	- BUILD_DIR赋值为工程编译输出路径build
	- SRC_DIR赋值为源文件路径sources
	- INC_DIR赋值为头文件路径includes和当前目录 `.`

- 第10行：SRCS存储所有需要编译的c文件，赋值为wildcard函数的输出，即`sources/hello_fun.c sources/hello_main.c`

- 第12行：OBJS存储所有要生成的o文件，赋值为patsubst函数的输出，即`build/hello_fun.o build /hello_main.o`

- 第14行：DEPS存储所有依赖的h文件，值为foreach函数的输出，`即将includes目录和`.`目录下所有的h文件拼接在一起`

- 第16行：CFLAGS存储包含的h文件路径，值为passubst函数的输出，即`-Iincludes -I.`

- 第19行：相比于之前，在`$(TARGET)`前增加了`$(BUILD_DIR)`路径，使得最终的可执行程序放在build目录下
- 第23行：与上面类似，给`.o`目标文件添加$(BUILD_DIR)路径
- 第27行：在执行编译前先创建build目录，以存放后面的.o文件，命令前的“@”表示执行该命令时不在终端上输出
- 第34行：rm删除命令也被修改成直接删除编译目录$(BUILD_DIR)
- 第38-39行：增加了删除所有架构编译目录的伪目标cleanall

使用该Makefile时，直接在Makefile的目录执行make即可：
```shell
#使用tree命令查看目录结构
#若提示找不到命令，使用 sudo apt install tree安装
tree

#编译
make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d0dfb7feed8fa00d4c3740de88e6980f.png)

## 8 使用分支

如果没有PC上的linux,可以跳过本节内容或者学习一下分支的语法。

为方便直接切换GCC编译器，我们还可以使用条件分支增加切换编译器 的功能。在Makefile中的条件分支语法如下：
```makefile
ifea(arg1,arg2)
分支1
else
分支2
endif
```

分支会比较括号内的参数“arg1”和“arg2”的值是否相 同，如果相同，则为真，执行分支1的内容，否则的话，执行分支2 的内容，参 数arg1和arg2可以是变量或者是常量。

使用分支切换GCC编译器的Makefile如下所示。(在pc上使用)
```makefile
#定义变量
#ARCH默认为x86，使用gcc编译器
ARCH ?= x86
TARGET = hello_main

#存放中间文件的路径
BUILD_DIR = build_$(ARCH)
#存放源文件的路径
SRC_DIR = sources
#存放头文件的路径
INC_DIR = includes .

#源文件
SRCS = $(wildcard $(SRC_DIR)/*.c)
#目标文件
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o,$(notdir $(SRCS)))
#头文件
DEPS = $(foreach dir,$(INC_DIR),$(wildcard $(dir)/*.h))

#指定头文件的路径
CFLAGS= $(patsubst %,-I%,$(INC_DIR))

#根据输入的ARCH变量，选择不同的编译器
#ARCH = x86,使用gcc
#ARCH = arm,使用arm-gcc
ifeq ($(ARCH),x86)
CC = gcc
else
CC = aarch64-linux-gnu-gcc
endif

#目标文件
$(BUILD_DIR)/$(TARGET):$(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

#o文件的生成规则
$(BUILD_DIR)/%.o:$(SRC_DIR)/%.c $(DEPS)
	@mkdir -p $(BUILD_DIR)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean cleanall
#按架构删除
clean:
	rm -rf $(BUILD_DIR)

#全部删除
cleanall:
	rm -rf build_x86 build_arm
```

修改后的Makefile文件分析如下：
- 第1~4行：增加ARCH的复制，若为空，则复制为x86
- 第9行：增加不同编译方式输出结果的文件夹
- 第25~32行：增加gcc编译器的选择
- 第48行：选择删除不同编译器的输出
- 第48行：把所有输出结果都删除

使用该Makefile时，直接在Makefile的目录执行make即可：
```shell
#使用tree命令查看目录结构
#若提示找不到命令，使用 sudo apt install tree安装
tree

#清除编译输出，确保不受之前的编译输出影响
make clean
#使用ARM平台
make ARCH=ARM64
#清除编译输出
make clean
#默认是x86平台
make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a454c42d69952119fec8f17d44fdc207.png)

本示例中的Makefile目前只支持使用一个源文件目录，如果有多个源文 件目录还需要改进，关于这些，我们在以后的学习中继续积累。