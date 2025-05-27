---
文章标题: "[[2_程序编译GCC📕]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux下使用GCC编译工具链进行C语言程序开发的完整流程，包括预处理、编译、汇编、链接四个阶段，以及相关工具的使用方法
tags:
- "GCC编译器"
- "Linux编程"
- "C语言"
- "编译过程"
- "Binutils工具"
- "glibc库"
- "readelf"
- "动态链接"
- "静态链接"
相关文章:
- "[[_常用函数]]"
- "[[0_笔记（重点备忘）]]"
- "[[0_课程完整内容]]"
- "[[0_C语言提纲]]"
- "[[1_704.二分查找]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/2_使用板卡开发C程序/2_程序编译GCC📕.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 17:05:46
修改时间: 2025-05-28 00:19:51
---


本章以Linux下使用GCC编译Hello World程序来讲解Linux C编程的相关流程和概念。 我们可以在电脑上的linux系统以及LubanCat-RK板卡上编译C语言程序

在Windows下开发C程序代码可以用Visual Studio，开发MCU的程序可以使用Keil、IAR等IDE集成开发环境；而在Linux下也有类似的IDE，如eclipse、Clion等。在这些环境下开发通常我们按照它们预定的步骤建立工程模板，再编写具体的代码，直接点击对应的编译、运行按钮即可完成操作。

在开发大型应用程序特别是调试的时候，使用IDE是非常好的选择， IDE的一个特点是它把各种常用操作封装成图形界面供用户使用，但如同学习Shell命令行的原因一样，在图形界面之下还潜藏着海量的功能，**在Linux下的日常开发中常常直接使用命令行来操作，编译时配合其它命令行工具的时候简单快捷，而且非常直观，有利于了解编译的原理**。

本章通过解构hello world程序在Linux下的编译运行过程， **掌握GCC、readelf、ldd工具的基本使用**，便于理解开发流程以及后期建立编译工具链是要做什么事情。

了解各编译步骤及其生成的文件，这对后期编写Makefile及使用其它工具大有好处。 了解程序的链接过程有利于明白为什么某些程序需要 依赖特定的文件，从而方便专门定制Linux文件系统。

本章的示例代码目录为：`base_linux/hello`
## 1 GCC编译工具链

GCC编译工具链（toolchain）是指以GCC编译器为核心的一整套工具，用于把源代码转化成可执行应用程序。它主要包含以下三部分内容：
- `gcc-core`：即GCC编译器，用于完成**预处理**和**编译过程**，例如把C代码转换成汇编代码。
- `Binutils` ：除GCC编译器外的一系列小工具包括了**链接器ld**，**汇编器as**、目标文件格式查看器readelf等。
- `glibc`：包含了主要的 **C语言标准函数库**，C语言中常常使用的打印函数printf、malloc函数就在glibc 库中

在很多场合下会直接用GCC编译器来指代整套GCC编译工具链。
### 1.1 GCC编译器

GCC（GNU Compiler Collection）是由 GNU 开发的编程语言编译器。 GCC最初代表“GNU C Compiler”，当时只支持C语言。 后来又扩展能够支持更多编程语言，包括 C++、Fortran 和 Java 等。 因此，GCC也被重新定义为“GNU Compiler Collection”，成为**历史上最优秀的编译器， 其执行效率与一般的编译器相比平均效率要高 20%~30%**。

如何使用的板卡没有预装gcc编译器，使用下列方法联网安装
```shell
sudo apt install gcc

# 查看gcc的版本以及所在的位置
#在板卡上执行如下命令
gcc -v          #查看gcc编译器版本
which gcc       #查看gcc的安装路径
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/24430eb57c67978b9f7975cde2dcc0e5.png)

图中两处信息表示：
- `Target: aarch64-linux-gnu` 表示该GCC的目标平台为ARM64位架构， 表示它编译生成的**应用程序只适用于ARM板卡平台，不适用于x86架构**。
- `gcc version 8.3.0` 表明该GCC的版本为8.3.0
	- 部分程序可能会对编译器版本有要求，例如编译指定版本的uboot、linux内核可能会对gcc有版本要求
### 1.2 Binutils工具集

Binutils（bin utility），是GNU二进制工具集，通常跟GCC编译器一起打包安装到系统，它的官方说明网站地址为： [https://www.gnu.org/software/binutils/](https://www.gnu.org/software/binutils/) 

在进行程序开发的时候通常不会直接调用这些工具，而是在使用GCC编译指令的时候**由GCC编译器间接调用**。下面是其中一些常用的工具：
- `as`：`汇编器`，把汇编语言代码转换为机器码（目标文件）。
- `ld`：`链接器`，把编译生成的多个目标文件组织成最终的可执行程序文件。
- `readelf`：可用于**查看目标文件或可执行程序文件的信息**。
- `nm` ： 可用于**查看目标文件中出现的符号**。
- `objcopy`： 可**用于目标文件格式转换**，如.bin 转换成 .elf 、.elf 转换成 .bin等。
- `objdump`：可用于**查看目标文件的信息**，最主要的作用是`反汇编`。
- `size`：可用于查看目标文件不同部分的尺寸和总尺寸，例如代码段大小、数据段大小、使用的静态内存、总大小等

系统默认的Binutils工具集位于/usr/bin目录下，可使用如下命令查看系统中存在的Binutils工具集：
```shell
#在Ubantu上执行如下命令
ls /usr/bin/ | grep linux-gnu-
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d9a6bdc02cd83e2f2eace332f159e0e8.png)

图中列出的是Binutils工具的完整名字，在终端中使用时通常直接使用它们的别名即可， 在后面的讲解我们会使用到readelf工具。
### 1.3 glibc库

glibc库是GNU组织为**GNU系统**以及**Linux系统**编写的`C语言标准库`，因为绝大部分C程序都依赖该函数库，该文件甚至会直接影响到系统的正常运行，例如常用的文件操作函数read、write、open，打印函数printf、动态内存申请函数malloc等

**在板卡的Debian系统下，libc.so.6是glibc的库文件**，可直接执行该库文件查看版本，在主机上执行如下命令
```shell
#以下是debain 64位机的glibc库文件路径，可直接执行
/lib/aarch64-linux-gnu/libc.so.6
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5c0319ee79e14203a3b653401991a2f8.png)

图中表示本系统中使用的glibc是2.28版本，是由GCC 8.3.0版本的编译器编译出来的。

学习C语言的时候，可能有同学特别好奇printf、malloc之类的函数是如何实现的， 但是在Windows下的C库是不开源的，无法查看，而在Linux下， 则可以直接研究glibc的源代码，甚至加入开发社区贡献自己的代码，glibc的官网地址为： [https://www.gnu.org/software/libc/](https://www.gnu.org/software/libc/) ，可在该网站中下载源代码来学习。

为了更直观地感受GCC编译工具，请跟着以下的步骤来打开新世界的大门吧。
## 2 HelloWorld

### 2.1 创建工作目录

先在当前用户下创建一个本章节使用的工作目录test。
```shell
#在Debian上执行如下命令
mkdir test #创建test目录
cd test       #切换目录
```
### 2.2 编写代码文件

使用编辑器新建一个名为hello.c的文件，输入如下面的示例代码并保存至hello_c目录下。
- 参考代码路径：base_linux/hello/hello/hello.c

```c
#include <stdio.h>

int main()
{
    printf("Hello, World!\n");
    return 0;
}
```
### 2.3 编译并执行

```shell
#使用gcc把hello.c编译成hello程序
gcc hello.c -o hello

ls        #查看目录下的文件
./hello   #执行生成的hello程序

#若提示权限不够或不是可执行文件，执行如下命令再运行hello程序
chmod u+x hello #给hello文件添加可执行权限
```

- 权限相关操作可以参考：[4 修改权限控制 - chmod](../../../../02-⚙️%20系统基础/01-💻%20命令行操作/3_用户和权限.md#4%20修改权限控制%20-%20chmod)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f4c81d230b63d1e3e73fa0b24b6b75d0.png)

**这就是在Linux下使用GCC开发简单C应用程序并运行的基本流程，下面我们针对GCC编译过程进行讲解。**
## 3 GCC编译过程
### 3.1 基本语法

GCC使用的命令语法如下：
```shell
gcc [选项] 输入的文件名
```

常用选项：
- `-o`：小写字母“o”，**指定生成的可执行文件的名字**，不指定的话生成的可执行文件名为a.out。
- `-E`：**只进行预处理**，既不编译，也不汇编。
- `-S`：**只编译**，不汇编。
- `-c`：**编译并汇编**，但不进行链接。
- `-g`：生成的可执行文件带调试信息，方便使用gdb进行调试。
- `-Ox`：大写字母“O”加数字，设置程序的优化等级，如“-O0”“-O1” “-O2” “-O3”， 数字越大代码的优化等级越高，编译出来的程序一般会越小，但有可能会导致程序不正常运行

举例：
- 只有一个test.c文件
```shell
gcc -E test.c -o test.i    # 预处理
gcc -S test.c -o test.s    # 编译
gcc -c test.c -o test.o    # 汇编

#假如有o文件
gcc test.o -o test   # 将 test.o 链接成可执行文件 test

#没有o文件，只有c文件，一步到位
gcc test.c -o test   # 预处理、编译、汇编和链接，生成可执行文件 test
```

- 如果test.c依赖sum.c文件
```shell
# 都编译为目标文件
gcc -c test.c -o test.o   # 编译 test.c，生成 test.o
gcc -c sum.c -o sum.o     # 编译 sum.c，生成 sum.o

# 链接目标文件
gcc test.o sum.o -o myprogram   # 链接 test.o 和 sum.o，生成可执行文件 myprogram
```

**编译过程讲解**
- GCC编译选项除了-g和-Ox选项，其它选项实际上都是编译的分步骤，即只进行某些编译过程。
```shell
#直接编译成可执行文件
gcc hello.c -o hello

#以上命令等价于执行以下全部操作
#预处理，可理解为把头文件的代码汇总成C代码，把*.c转换得到*.i文件
gcc -E hello.c -o hello.i

#编译，可理解为把C代码转换为汇编代码，把*.i转换得到*.s文件
gcc -S hello.i -o hello.s

#汇编，可理解为把汇编代码转换为机器码，把*.s转换得到*.o，即目标文件
gcc -c hello.s -o hello.o

#链接，把不同文件之间的调用关系链接起来，把一个或多个*.o转换成最终的可执行文件
gcc hello.o -o hello
```

GCC 编译工具链在编译一个C源文件时需要经过以下 4 步：
1. `预处理`，在预处理过程中，对源代码文件中的文件包含(include)、 预编译语句(如宏定义define等)进行展开，生成.i文件。 可理解为**把头文件的代码、宏之类的内容转换成更纯粹的C代码**，不过生成的文件以.i为后缀。
2. `编译`，把预处理后的.i文件**通过编译成为汇编语言**，生成.s文件，即把代码从C语言转换成汇编语言，这是GCC编译器完成的工作。
3. `汇编`，将汇编语言文件经过汇编，生成目标文件.o文件，每一个源文件都对应一个目标文件。即把**汇编语言的代码转换成机器码**，这是as汇编器完成的工作。
4. `链接`，最后**将每个源文件对应的.o文件链接起来**，就生成一个可执行程序文件，这是链接器ld完成的工作。

以上一节的hello.c为例，后面括号代表的是gcc的参数，分步骤编译过程如下图所示。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1aae3ec24b307e70d91de1df0ed4c404.png)

关于编译原理，大家可以找专门的书籍阅读加深理解，这对程序开发大有裨益，下面带领大家浏览一下各个阶段生成的文件。
### 3.2 预处理阶段

使用GCC的参数“-E”，可以让编译器生成.i文件，参数“-o”，可以指定输出文件的名字。

具体执行命令如下：
```shell
#预处理，可理解为把头文件的代码汇总成C代码，把*.c转换得到*.i文件
gcc -E hello.c -o hello.i
```

直接用编辑器打开生成的hello.i，可以看到如下的内容
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6d9fdcca178d5c4692563488923028fb.png)

文件中以“#”开头的是注释，可看到有非常多的类型定义、函数声明被加入到文件中， 这些就是预处理阶段完成的工作，相当于它把原C代码中包含的头文件中引用的内容汇总到一处。 如果原C代码有宏定义，还可以更直观地看到它把宏定义展开成具体的内容（如宏定义代表的数字）
### 3.3 编译阶段

GCC可以使用-S选项，让编译程序生成汇编语言的代码文件（.s后缀）。 在这个过程，GCC会检查各个源文件的语法，即使我们调用了一个没有定义的函数，也不会报错。

具体命令如下，生成的hello.s文件可直接使用编辑器打开。
```shell
#编译，可理解为把C代码转换为汇编代码，把*.i转换得到*.s文件
gcc -S hello.i -o hello.s

#也可以直接以C文件作为输入进行编译，与上面的命令是等价的
gcc -S hello.c -o hello.s
```

编译生成的hello.s文件内容如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4aa4c8166bb4db9b6ac05d36bac21c7d.png)

### 3.4 汇编阶段

GCC的参数“c”表示只编译(compile)源文件但不链接，会将源程序编译成目标文件（.o后缀）。计算机只认识0或者1，不懂得C语言，也不懂得汇编语言，经过编译汇编之后，生成的目标文件包含着机器代码，这部分代码就可以直接被计算机执行。一般情况下，可以直接使用参数“c”，跳过上述的两个过程，具体命令 如下：
```shell
#汇编，可理解为把汇编代码转换为机器码，把*.s转换得到*.o，即目标文件
gcc –c hello.s –o hello.o

#也可以直接以C文件作为输入进行汇编，与上面的命令是等价的
gcc –c hello.c –o hello.o
```

`.o`是为了让计算机阅读的，所以不像前面生成的`.i`和`*.s`文件直接使用字符串来记录， 如果直接使用编辑器打开，只会看到乱码，如下图。
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/500dbd1964ddb18827863272906c5fcc.png)

Linux下生成的`*.o`目标文件、`*.so`动态库文件以及下一小节链接阶段生成最终的**可执行文件都是elf格式的， 可以使用“readelf”工具来查看它们的内容**。

```shell
#在hello.o所在的目录执行如下命令
readelf -a hello.o
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f74930f28540260e40afa4155193c564.png)

从readelf的工具输出的信息，可以了解到目标文件包含ELF头、程序头、节等内容
- 对于`*.o`目标文件或`*.so`库文件，编译器在链接阶段利用这些信息把多个文件组织起来
- 对于可执行文件，系统在运行时根据这些信息加载程序运行。
### 3.5 链接阶段

**链接过程，是将汇编过程生成的所有目标文件进行链接，生成可执行文件。**

例如一个工程里包含了A和B两个代码文件，编译后生成了各自的A.o和B.o目标文件， 如果在代码A中调用了B中的某个函数fun，那么在A的代码中只要包含了fun的函数声明， 编译就会通过，而不管B中是否真的定义了fun函数（当然，如果函数声明都没有，编译也会报错）。 也就是说A.o和B.o目标文件在编译阶段是独立的，而在链接阶段， 链接过程需要把A和B之间的函数调用关系理顺，也就是说要告诉A在哪里能够调用到fun函数， 建立映射关系，所以称之为链接。若链接过程中找不到fun函数的具体定义，则会链接报错。

虽然本示例只有一个hello.c文件，但它调用了C标准代码库的printf函数， 所以链接器会把它和printf函数链接起来，生成最终的可执行文件。

链接分为两种：
- `动态链接`，GCC编译时的默认选项。**动态是指在应用程序运行时才去加载外部的代码库**， 例如printf函数的C标准代码库`*.so`文件存储在Linux系统的某个位置， hello程序执行时调用库文件*.so中的内容，不同的程序可以共用代码库。 所以动态链接生成的程序比较小，占用较少的内存。

- `静态链接`，链接时使用选项“`–static`”，它在编译阶段就会把所有用到的库打包到自己的可执行程序中。 所以静态链接的优点是具有较好的兼容性，不依赖外部环境，但是生成的程序比较大。

请尝试执行如下命令体验静态链接与动态链接的区别：
```shell
#在hello.o所在的目录执行如下命令
#动态链接，生成名为hello的可执行文件

gcc hello.o –o hello

#也可以直接使用C文件一步生成，与上面的命令等价
gcc hello.c -o hello

#静态链接，使用--static参数，生成名为hello_static的可执行文件
gcc hello.o –o hello_static --static

#也可以直接使用C文件一步生成，与上面的命令等价
gcc hello.c -o hello_static --static
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3fb27b2884ab7dae02a3bd9665ffe0cf.png)

从图中可以看到，使用动态链接生成的hello程序才9.3KB， 而使用静态链接生成的hello_static程序则高达575KB。

在Ubuntu下，可以使用ldd工具查看动态文件的库依赖，尝试执行如下命令：
```shell
#在hello所在的目录执行如下命令
ldd hello
ldd hello_static
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a3f8f375c393ea3c5b0f04afb4fdf1a1.png)

**可以看到上图中**
- 动态链接生成的hello程序依赖于库文件linux-vdso.so.1、libc.so.6 以及ld-linux-aarch64.so.1，其中的libc.so.6就是我们常说的C标准代码库， 我们的程序中调用了它的printf库函数。
- 静态链接生成的hello_static没有依赖外部库文件。
## 4 HW进阶版1（main带参）

```c
#C语言端
int main(int xxx , char **xxxx)

#命令端执行
./hello aaa bbb ccc
```

在main函数里，也是具有传参的，传的参数全部都会保存下来
- `int xxx`：保存参数的个数,上例中xxx = 4
- `char **xxx`：**保存传入的参数<-->指向字符串数组的指针**
	- argv[0] = "./hello"
	- argv[1] = "aaa"
	- argv[2] = "bbb"
	- argv[3] = "ccc"

板卡上使用编辑器新建一个名为hello_arg.c的文件，输入如下面的示例代码并保存至hello_arg.c目录下
```c
#include <stdio.h>

/**
 * 执行命令： ./hello lwk
 * argc = 2
 * argv[0] = ./hello
 * argv[1] = lwk
 */

int main(int argc,char **argv){
    int i;
    if(argc >= 2){
        printf("your enter : \"");
        // 打印每个参数
        for(int i = 0;i<argc;i++){
            printf("%s ",argv[i]);
        }
        printf("\"\n");
        // 打印参数的个数
        printf("you enter %d strings\n",argc);
    }else{
        printf("you must enter at least one string\n");
    }
    return 0;
}
```

**编译并执行**
```shell
#使用gcc把hello_arg.c编译成hello程序
gcc hello_arg.c -o hello

#我们可以尝试不同的值

./hello
./hello lubancat
./hello I am pc!
./hello I delete king of honor
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/203182357f4eded42720bcd41e4b98b8.png)

**❗️注意：** 输入的参数都是以字符串的形式保存， 如果输入数字的话，需要字符串转成数字类型才能使用
## 5 HW进阶版2

增加选项配置，像我们平常用的ls命令， 我们可以在后面加入不同的选项就可以实现不一样的功能

使用编辑器新建一个名为hello_opt.c的文件，输入如下面的示例代码并保存至hello_opt.c目录下。

**代码编写**
- 该程序主要依赖<getopt.h>的库，getopt函数

```c
#include <stdio.h>
#include <getopt.h>
#include <stdlib.h>
#include <string.h>

// 打印帮助信息
void usage(const char *agrv_0){
    printf("\nUsage %s:[-option] \n",agrv_0);
    printf("[-a] hello!\n");
    printf("[-b] i am your dad\n");
    printf("[-c<str>] str\n");
    printf("[-d<num>] printf num of '*' (num<100)\n");
    printf("[-h] get help\n");
    printf("\n");
    exit(1);
}

//打印n个*号
void d_option(char *num_str){
    int num;
    int ge,shi;
    shi = (char)num_str[0] - '0';
    ge = (char)num_str[1] - '0';
    num = shi * 10 + ge;
    for(int i = 0; i < num; i++){
        printf("*");
    }
    printf("*\n");
}

int main(int argc, char *argv[]){
    int i;
    int opt;
    //轮询获取选项
    //第三个参数是一个字符串，定义了程序接收的选项，:表示该选项需要一个参数
    //所以，c和d选项需要参数，abh不需要参数
    //其中，optarg 是 getopt() 提供的全局变量，用于获取当前处理的选项的参数
    while((opt = getopt(argc,argv,"c:d:abh"))!=-1){
        switch(opt){
            case 'a':
                printf("hello\n");
                break;
            case 'b':
                printf("i am your dad\n");
                break;
            case 'c':
                if(optarg){
                    if(optarg[0] == '-') usage(argv[0]);
                    else printf("%s\n",optarg);
                }else{
                    usage(argv[0]);
                }
                break;
            case 'd':
                if(optarg){
                    if(optarg[0] == '-'){
                        usage(argv[0]);
                    }else{
                        if(strlen(optarg)<3){
                            d_option(optarg);
                        }else{
                            usage(argv[0]);
                        }
                    }
                }else{
                    usage(argv[0]);
                }
                break;
            default:
                usage(argv[0]);
                break;
        }
    }
    return 0;
}
```

**编译并执行**
```shell
#使用gcc把hello_arg.c编译成hello程序
gcc hello_opt.c -o hello

#我们可以尝试不同的值

./hello
./hello -a
./hello -b
./hello -c xxxx
./hello -d num
./hello -abcd
……………………
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/299dc69b6d64e1d8b13a5586acd27960.png)
