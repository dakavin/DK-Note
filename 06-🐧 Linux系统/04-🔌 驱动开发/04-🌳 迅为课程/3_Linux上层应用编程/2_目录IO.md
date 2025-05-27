---
文章标题: "[[2_目录IO]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux系统编程中目录IO操作，包括mkdir创建目录、opendir/closedir打开关闭目录流、readdir读取目录内容等基本函数的使用方法和示例代码。
tags:
- "Linux系统编程"
- "目录IO"
- "mkdir"
- "opendir"
- "readdir"
- "C语言"
- "文件系统操作"
- "POSIX"
相关文章:
- "[[_常用函数]]"
- "[[0_笔记（重点备忘）]]"
- "[[0_课程完整内容]]"
- "[[0_C语言提纲]]"
- "[[1_704.二分查找]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/04-🌳 迅为课程/3_Linux上层应用编程/2_目录IO.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-14 14:45:36
修改时间: 2025-05-28 00:19:51
---

## 1 简介

文件 IO 和目录 IO 的对比：
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/caea19c7eec2c672b14e3bda363dc20c.png)

之前我们学习的文件 IO 和提到过的标准 IO 都是对文件操作，接下来学习的目录 IO 都是**对目录操作**
## 2 mkdir

**函数定义：**
- mkdir()：创建一个目录

**参数含义：**
- pathname:路径和文件名
- mode: 权限掩码，对不同用户和组设置可执行，读，写权限，使用八进制数表示，此参数可不写。

**返回值：**
- mkdir()执行成功会返回 0，出错时返回-1。

**实验代码**
```c
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/stat.h> 
#include <sys/types.h> 
int main(int argc, char *argv[]) 
{ 
    int ret; 
    if (argc != 2) 
    { 
        printf("Usage:%s <name file>\n", argv[0]); 
        return -1; 
    } 
    ret=mkdir(argv[1],0666); 
    if(ret<0) 
    { 
        printf("mkdir is error\n"); 
    } 
    printf("mkdir is ok\n"); 
    return 0; 
}
```
## 3 opendir/closedir

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5e8b677a708463bbfe8be892ee00566c.png)

**函数定义**opendir()：打开指定的目录，并返回 `DIR*`形态的目录流，
**参数含义：** name路径名字
**返回值：** 成功返回打开的目录流，失败返回 NULL

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/83b494b7603a7fc75b81a170b319572b.png)

**函数定义：** closedir()：关闭目录流。
**参数含义**： dirp：要关闭的目录流指针。

**实验代码**
```c
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/stat.h> 
#include <sys/types.h> 
#include <dirent.h> 
int main(int argc, char *argv[]) 
{ 
    int ret; 
    DIR *dp; 
    if (argc != 2) 
    { 
        printf("Usage:%s <name file>\n", argv[0]); 
        return -1; 
    } 
    dp = opendir(argv[1]); 
    if (dp != NULL) 
    {
        printf("opendir is ok\n"); 
        return -1; 
    } 
    closedir(dp); 
    return 0; 
}
```
## 4 readdir

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/96e5a89c8e608bd9c8f045b7d9c4fdb9.png)

**函数定义** readdir()：用来读一个目录

**参数含义：** `DIR *dirp`：要读取的目录流指针。

**返回值：** 成功返回读取到的目录流指针，失败返回 NULL。

**实验代码** 在程序中读取目录
```Bash
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/stat.h> 
#include <sys/types.h> 
#include <dirent.h>
int main(int argc, char *argv[]) 
{ 
    int ret; 
    DIR *dp; 
    struct dirent *dir; 
    if (argc != 2) 
    { 
        printf("Usage:%s <name file>\n", argv[0]); 
        return -1; 
    } 
    dp = opendir(argv[1]); 
    if (dp == NULL) 
    { 
        printf("opendir is error\n"); 
        return -2; 
    } 
    printf("opendir is ok\n"); 
    while (1) 
    { 
        dir = readdir(dp); 
        if (dir != NULL) 
        { 
            printf("file name is %s\n", dir->d_name); 
        } 
        else 
        break; 
    } 
    closedir(dp); 
    return 0; 
}
```