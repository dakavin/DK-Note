---
文章标题: "[[3_gcc常用命令及参考]]" 
文章作者: Dakkk
文章概要: |
  介绍GCC编译器的基础命令使用，包括预处理、编译、汇编、链接四个阶段的分别操作，以及readelf和ldd工具用于查看ELF文件内容和动态库依赖关系。
tags:
- "GCC"
- "编译器"
- "预处理"
- "汇编"
- "链接"
- "readelf"
- "ldd"
- "ELF格式"
相关文章:
- "[[14_(进阶)程序编译+链接]]"
- "[[1_初识C语言]]"
- "[[1_MinGW编译器的安装和配置]]"
- "[[10_程序编译和调试]]"
- "[[1_VsCode & Mingw]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/00-❇️实用技巧/3_gcc常用命令及参考.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-12 12:18:18
修改时间: 2025-05-28 00:19:51
---

参考文章：[2_程序编译GCC📕](../02-💾%20Lubancat-RK3568/3_Linux基础与应用开发实战/2_使用板卡开发C程序/2_程序编译GCC📕.md)

## 1 ESc(i s o) 预处理、编译和汇编

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

## 2 readelf

Linux下生成的`*.o`目标文件、`*.so`动态库文件以及下一小节链接阶段生成最终的**可执行文件都是elf格式的， 可以使用“readelf”工具来查看它们的内容**。

```shell
#在hello.o所在的目录执行如下命令
readelf -a hello.o
```

## 3 ldd

在Ubuntu下，可以使用ldd工具查看动态文件的库依赖，尝试执行如下命令：
```shell
#在hello所在的目录执行如下命令
ldd hello
ldd hello_static
```

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a3f8f375c393ea3c5b0f04afb4fdf1a1.png)



