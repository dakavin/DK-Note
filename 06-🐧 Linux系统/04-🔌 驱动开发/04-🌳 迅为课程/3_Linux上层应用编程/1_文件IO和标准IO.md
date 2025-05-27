---
文章标题: "[[1_文件IO和标准IO]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux文件IO和标准IO的区别，文件IO直接调用系统调用无缓存，标准IO通过C库函数有缓存机制，详细讲解open、close、read、write、lseek等API使用
tags:
- "Linux文件IO"
- "标准IO"
- "系统调用"
- "文件描述符"
- "缓存机制"
- "C语言"
- "open函数"
- "read函数"
- "write函数"
- "lseek函数"
相关文章:
- "[[_常用函数]]"
- "[[0_笔记（重点备忘）]]"
- "[[0_课程完整内容]]"
- "[[0_C语言提纲]]"
- "[[1_704.二分查找]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/04-🌳 迅为课程/3_Linux上层应用编程/1_文件IO和标准IO.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-14 14:45:22
修改时间: 2025-05-28 00:19:51
---

## 1 Linux提供的文件接口介绍

文件 IO 是 Linux 系统提供的接口，针对文件和磁盘进行操作，**不带缓存机制**；
标准 IO 是 C 语言函数库里的标准 I/O 模型，在 stdio.h 中定义，通过缓冲区操作文件，**带缓存机制**。

Linux 系统中一切皆文件，包括普通文件，目录，设备文件（不包含网络设备），管道，fifio 队列，socket 套接字等，在终端输入“ls -l”可查看文件类型和权限。

标准 IO 和文件 IO 常用 API 如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d2d62a7b731d438d353a34122833c505.png)

标准 IO 和文件 IO 的区别如下图所示：
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/97aecbfb490afed85440d16fce45ac25.png)
- 文件 IO 是直接调用内核提供的系统调用函数，头文件是 `unistd.h`
- 标准 IO 是间接调用系统调用函数，头文件是 `stdio.h`，
- 文件 IO 是依赖于 Linux 操作系统的，标准 IO 是不依赖操作系统的，所以在任何的操作系统下，使用标准 IO，也就是 C 库函数操作文件的方法都是相同的

对于文件 IO 来说，一切都是围绕文件操作符来进行的。在 Linux 系统中，所有打开的文件都有一个对应的文件描述符。文件描述符的本质是一个非负整数，当我们打开一个文件时，系统会给我们分配一个文件描述符。当我们对一个文件做读写操作的时候，我们使用 open 函数返回的这个文件描述符会标识该文件，并将其作为参数传递给 read 或者 write 函数。在 posix.1 应用程序里面，文件描述符 0,1,2 分别对应着标准输入，标准输出，标准错误。
## 2 open

**函数定义：**
- `open()`：通过系统调用，可以打开文件，并返回文件描述符。

**参数含义：**
- `pathname`：路径和文件名
- `flags`：文件打开方式，可用多个标志位按位或设置，常用的标志位参数如下：

```Bash
O_CREAT    要打开的文件名不存在时自动创建改文件。 
O_EXCL     要和 O_CREAT 一起使用才能生效，如果文件存在则 open()调用失败。 
O_RDONLY   只读模式打开文件。 
O_WRONLY   只写模式打开文件。 
O_RDWR     可读可写模式打开文件。 
O_APPEND   以追加模式打开文件。 
O_NONBLOCK 以非阻塞模式打开。
```

- `mode`：权限掩码，对不同用户和组设置可执行，读，写权限，使用八进制数表示，此参数可不写。

**返回值**：
- open()执行成功会返回 int 型文件描述符，出错时返回-1。

**实验代码**
```c
#include<stdio.h> 
#include<stdlib.h> 
#include<sys/types.h> 
#include<sys/stat.h> 
#include<fcntl.h> 

int main(int argc,char *argv[])
{ 
    int fd; 
    fd=open("a.c",O_CREAT|O_RDWR,0666); 
    if(fd<0) 
    { 
        printf("open is error\n"); 
    } 
    printf("fd is %d\n",fd); 
    return 0; 
}
```
## 3 close

**函数定义：**

- `close()`：可以关闭文件，通过系统调用取消文件描述符到文件的映射。

**参数含义：**
- fd:文件描述符

**返回值：**
- 成功返回 0；错误返回-1.

**实验代码**：
```c
#include<stdio.h> 
#include<stdlib.h> 
#include<sys/types.h> 
#include<sys/stat.h> 
#include<fcntl.h> 
#include<unistd.h>
int main(int argc,char *argv[])
{
    int fd; 
    fd=open("a.c",O_CREAT|O_RDWR,0666); 
    if(fd<0)
    { 
        printf("open is error\n"); 
    } 
    printf("fd is %d\n",fd); 
    close(fd); 
    return 0;
}
```
## 4 read

**函数定义：**
- read()：读取文件最常用的函数，每次从 fd 读取 count 个字节，保存到 buf 中。

**参数定义：**
  - fd： 要读的文件描述符
  - buf： 缓冲区，存放读到的内容。
  - count： 每次读取的字节数

**返回值：**
  - 返回值大于 0，表示读取到的字节数；
  - 等于 0 在阻塞模式下表示到达文件末尾或没有数据可读（EOF），并调用阻塞；
  - 等于-1 表示出错，在非阻塞模式下表示没有数据可读。

**实验代码**
```c
#include<stdio.h> 
#include<stdlib.h> 
#include<sys/types.h> 
#include<sys/stat.h> 
#include<fcntl.h> 
#include<unistd.h> 
int main(int argc,char *argv[]) 
{ 
    int fd; 
    char buf[32]={0};
    ssize_t ret; 
    fd=open("a.c",O_CREAT|O_RDWR,0666); 
    if(fd<0) 
    { 
        printf("open is error\n"); 
        return -1; 
    } 
    printf("fd is %d\n",fd); 
    ret=read(fd,buf,32); 
    if(ret<0) 
    { 
        printf("read is error\n"); 
        return -2; 
    } 
    printf("buf is %s\n",buf); 
    printf("ret is %ld\n",ret); 
    close(fd); 
    return 0; 
}
```
## 5 write

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/398cf515fe0cd5cc8d9fa790097a03d3.png)

**函数定义**
- write()：常用来写文件，定义如下

**参数含义：**
  - fd: 文件描述符；
  - buf: 缓存区，存放将要写入的数据；
  - count： 每次写入的个数。

**函数功能：**
- 每次从 buf 缓存区拿 count 个字节写入 fd 文件。

**返回值：**
  - 大于或等于 0 表示执行成功，返回写入的字节数；
  - 返回-1 代表出错。

**实验代码**

```c
#include<stdio.h> 
#include<stdlib.h> 
#include<sys/types.h> 
#include<sys/stat.h> 
#include<fcntl.h> 
#include<unistd.h> 
int main(int argc,char *argv[])
{ 
    int fd; 
    char buf[32]={0}; 
    ssize_t ret; 
    fd=open("a.c",O_CREAT|O_RDWR,0666); 
    if(fd<0) 
    { 
        printf("open is error\n"); 
        return -1; 
    } 
    write(1,"hello\n",6); 
    close(fd); 
    return 0; 
}
```
## 6 lseek

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cdb02b1d6d190bb6f52d54c17686efc4.png)

**函数定义**
- lseek()：移动文件的读写位置，定义如下：

**参数含义：**
  - fd: 文件描述符；
  - off_t offset: 偏移量，单位是字节的数量，可以正负，如果是负值表示向前移动；如果是正值，表示向后移动。
  - whence： 当前位置的基点，可以使用以下三组值。
		SEEK_SET：相对于文件开头
		SEEK_CUR:相对于当前的文件读写指针位置
		SEEK_END:相对于文件末尾

**函数功能：**
- 相对于上面的基点，移动文件描述符所在的位置。

**返回值：**
- 成功返回当前位移，失败返回-1

**实验代码**
```c
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/types.h> 
#include <sys/stat.h> 
#include <fcntl.h> 
#include <unistd.h> 
int main(int argc, char *argv[]) 
{ 
    int fd; 
    char buf[32] = {0}; 
    ssize_t ret; 
    fd = open("a.c", O_CREAT | O_RDWR, 0666); 
    if (fd < 0) 
    { 
        printf("open is error\n"); 
        return -1; 
    } 
    // printf("fd is %d\n", fd); 
    ret = read(fd, buf, 32); 
    if (ret < 0) 
    { 
        printf("read is error\n"); 
        return -2; 
    } 
    printf("buf is %s\n", buf); 
    printf("ret is %ld\n", ret); 
    lseek(fd,0,SEEK_SET); 
    ret = read(fd, buf, 32); 
    printf("ret is %ld\n", ret);
    close(fd); 
    return 0; 
}
```