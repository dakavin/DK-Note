
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/68b3d236dad5f450f957520975bb1edd.png)

在Linux驱动编程当中，会进程和`container_of()`函数打交道，所以特意拿出来和大家分享一下。

这个函数的功能不多，不过单靠自己去阅读内核源码分析，那可能非常难以理解，，编写内核源代码的大牛随便两行代码都会让我们看的云深不知处， 分析内核源代码需要我们有很好的知识积累以及技术沉淀。

其宏定义实现如下
```c
#define container_of(ptr, type, member) ({                      \
        const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
        (type *)( (char *)__mptr - offsetof(type,member) );})
```

**参数：**
- ptr：结构体变量中某个成员的地址
- type：结构体类型
- member：该结构体变量的具体名字

**返回值**：结构体type的首地址

**原理：**
- 通过已知类型type的成员member的地址ptr，计算出结构体type的首地址
- type的首地址 = ptr - size
- 需要注意：它们的大小都是以字节为单位计算的

container_of()函数过程如下：
- 判断ptr和member是否同一类型
- 计算size的大小，结构体的起始地址 = `(type *)((char *)ptr - size)`