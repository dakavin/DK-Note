---
文章标题: "[[8_调试工具(gdb和strace)]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux下两个重要调试工具gdb和strace的使用方法，gdb用于程序断点调试，strace用于跟踪系统调用，帮助开发者快速定位和解决程序bug
tags:
- "gdb"
- "strace"
- "Linux调试"
- "系统调用"
- "断点调试"
- "程序调试工具"
- "命令行工具"
- "开发工具"
相关文章:
- "[[1_ag代码搜索工具]]"
- "[[0_vscode的安装与使用]]"
- "[[1_Git快速入门]]"
- "[[2_环境搭建]]"
- "[[2_yarn的安装和使用]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/2_使用板卡开发C程序/8_调试工具(gdb和strace).md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 17:05:46
修改时间: 2025-05-28 00:19:51
---


我们写程序的时候，难免会遇到程序出现bug的时候， 好的调试工具可以让我们更快的找到bug所在的地方， 轻松解决bug，推荐两个好用的调试工具工具
- gdb
- strace

**💡提示：** 除了以下推荐的两个调试工具，linuix里有各种各样强大的调试工具 读者可百度搜索，并根据自己的需求获取自己需要的调试工具

## 1 gdb

经典的调试工具，功能很强大，

注意此时**编译的时候需要增加 -g 选项， 并用-Og进行优化**。

**多线程下可以attach到进程来调试**。

在命令行中输入 **gdb** 进入gdb命令行模式

gdb具有大量命令调试，这里不能将其一一列举， 这里给大家介绍一些常用的基础命令

**表1-1**

| 命令        | 缩写   | 用法        | 作用                                             |
| --------- | ---- | --------- | ---------------------------------------------- |
| start     |      | start     | 开始执行程序,在main函数的第一条语句前面停下来                      |
| file      |      | file xxx  | 打开调试的文件                                        |
| run       | r    | ru        | 运行程序（第一条语句开始运行）                                |
| break     | b    | 见下表1-2    | 设置断点                                           |
| continue  | c    | c         | 继续程序的运行,直到遇到下一个断点                              |
| list      | l    | 见下表1-3    | 显示多行源代码                                        |
| step      | s    | s         | 执行下一条语句,如果该语句为函数调用,则进入函数执行其中的第一条语句             |
| next      | n    | n         | 执行下一条语句,如果该语句为函数调用,不会进入函数内部执行(即不会一步步地调试函数内部语句) |
| display   | disp | disp n    | 跟踪查看某个变量,每次停下来都显示它的值                           |
| print     | p    | p 或 p n   | 打印内部变量值                                        |
| watch     |      | watch     | 监视变量值的变化                                       |
| backtrace | bt   | bt        | 产看函数调用信息(堆栈)                                   |
| frame     | f    | f         | 查看栈帧                                           |
| quit      | q    | q         | 退出GDB环境                                        |
| kill      | k    | k         | 终止正在调试的程序                                      |
| 设置变量      |      | set var=x | 设置变量的值                                         |
| return    |      | return x  | 结束当前调用函数并返回指定值，到上一层函数调用处停止程序执行。                |
| finish    | fi   | fi        | 结束当前正在执行的函数，并在跳出函数后暂停程序的执行                     |
| jump      | j    | j x       | 结束当前调用函数井返回指定值，到上一层函数调用处停止程序执行                 |
**表1-2**

|命令|功能|
|---|---|
|break n|设置第几行作为断点（搭配list使用）|
|info breakpoints|列出所加的断点|
|delete breakpoints 1|删除第n个断点（n为info breakpoints获得的断点号）|
|disable/enable n|表示使得编号为n的断点暂时失效或有效|

**表1-3**

|命令|功能|
|---|---|
|list|默认情况下,一次显示10行,第一次使用时,从代码其实位置显示|
|list n|显示已第n行未中心的10行代码|
|list functionname|显示以functionname的函数为中心的10行代码|
|list -|显示刚才打印过的源代码之前的代码|
## 2 strace

strace是一个用来**跟踪系统调用**的简易工具。
- 最简单的用途就是**跟踪一个程序整个生命周期里所有的系统调用**， 并把调用参数和返回值以文本的方式输出。
- Strace还可以**跟踪发给进程的信号**
- 支持attach正在运行的进程 `strace -p <pid>`
- 当多线程环境下， 需要跟踪某个线程的系统调用， 可以先`ps -efL | grep <Process Name>` 查找出该进程下的线程， 然后调用`strace –p <pid>`进行分析

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1473276a1c04ba12b3dca1c0c92f60fd.png)

```shell


cat@lubancat:~/my_code/base_linux/makefile/test2$ strace -f ./build/hello_main
execve("./build/hello_main", ["./build/hello_main"], 0x7fcfbaf0f8 /* 25 vars */) = 0
brk(NULL)                               = 0x559fc92000
faccessat(AT_FDCWD, "/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=146734, ...}) = 0
mmap(NULL, 146734, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f94cf2000
close(3)                                = 0
openat(AT_FDCWD, "/lib/aarch64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0\360\16\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1435448, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f94d41000
mmap(NULL, 1507424, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f94b81000
mprotect(0x7f94cd9000, 61440, PROT_NONE) = 0
mmap(0x7f94ce8000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x157000) = 0x7f94ce8000
mmap(0x7f94cee000, 12384, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f94cee000
close(3)                                = 0
mprotect(0x7f94ce8000, 16384, PROT_READ) = 0
mprotect(0x5593600000, 4096, PROT_READ) = 0
mprotect(0x7f94d45000, 4096, PROT_READ) = 0
munmap(0x7f94cf2000, 146734)            = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}) = 0
brk(NULL)                               = 0x559fc92000
brk(0x559fcb3000)                       = 0x559fcb3000
write(1, "hello,world! This is a C program"..., 34hello,world! This is a C program.
) = 34
write(1, "output i = 0\n", 13output i = 0
)          = 13
write(1, "output i = 1\n", 13output i = 1
)          = 13
write(1, "output i = 2\n", 13output i = 2
)          = 13
write(1, "output i = 3\n", 13output i = 3
)          = 13
write(1, "output i = 4\n", 13output i = 4
)          = 13
write(1, "output i = 5\n", 13output i = 5
)          = 13
write(1, "output i = 6\n", 13output i = 6
)          = 13
write(1, "output i = 7\n", 13output i = 7
)          = 13
write(1, "output i = 8\n", 13output i = 8
)          = 13
write(1, "output i = 9\n", 13output i = 9
)          = 13
exit_group(0)                           = ?
+++ exited with 0 +++

```

- 第3-26行：
	- 配置程序的运行环境，调用相应的库，第11行的 `/lib/aarch64-linux-gnu/libc.so.6` 是glibc库文件，这些操作可以保护系统不受程序的干扰
	- 比如，有些软件想改写或破坏系统文件，在上面的保护环境下，程序运行不会运行在系统规定之外，即使程序无意改写了系统文件，系统都会阻止它
	- 如果程序进入死循环或崩溃了，系统可以依赖这些保护措施不受干扰，并且终止程序的运行
- 第27-49行：程序主体部分，输出格式有误，可能是多线程导致格式有点问题，正确的格式如下
	```shell
	write(1, "hello, world! This is a C program.\n", 34) = 34
	hello, world! This is a C program.
	
	write(1, "output i=0\n", 13) = 13
	output i=0
	```
	- write：写入东西
	- 1：文件描述符，这里表述标准输出，即向终端输出东西
	- output i=0：要输出的内容
	- 13：输出内容的字节大小，加上空格和换行符刚好13个
	- =13：write函数的返回值，表示写入了多少个字节


**为什么 第27-28行，就可以在屏幕输出文字呢？**
- 因为linux系统把`输出到命令行抽象成文件描述符`， 操作文件描述符就可以操作该设备
- 查看系统的描述符
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/daa11f0da759ef2f6a33d012598edac9.png)
	- stdin标准输入–fd=0–从终端设备输入内容
	- stdout标准输出–fd=1–从终端设备输出内容
	- stderr标准错误–fd=2–从终端设备输出命令的错误消息


**我们也可以用write代替printf试一试**
```c
cat@lubancat:~/test$ cat hello_sys.c

#include <unistd.h>
int main()
{
    write(1, "hello, world! This is a C program.\n", 35);
    write(1, "output i=0\n", 11);
    return 0;
}

cat@lubancat:~/test$ gcc hello_sys.c -o hello_sys

cat@lubancat:~/test$ ./hello_sys
hello, world! This is a C program.
output i=0
```

结果和printf一样。