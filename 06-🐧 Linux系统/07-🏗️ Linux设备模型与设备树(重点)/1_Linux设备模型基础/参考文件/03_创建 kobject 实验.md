创建 kobject 的实践， 通过使用 kobject 的 API 函数来创建系统根目录下的目录。

## 1 方法

本章介绍了两种创建 kobject 的方法:

- 一种是使用 kobject_create_and_add 函数
    
- 另一种是使用 kzalloc 和 kobject_init_and_add 函数， 还介绍了如何释放创建的 kobject。
    

  

1. 使用 kobject_create_and_add()函数创建 kobject：
    

kobject_create_and_add()函数

- 首先调用 kobject_create()函数， 该函数使用 kzalloc()为kobject 分配内存空间。
    
- 在 kobject_create()函数中， 调用 kobject_init()函数对分配的内存进行初始化，
    
- 并指定了默认的 ktype。
    
- 接下来， 调用 kobject_add()函数将 kobject 添加到系统中， 使其可见。
    
- kobject_add()函数内部调用了 kobject_add_internal()函数， 该函数负责将 kobject 添加到父对象的子对象列表中， 并创建相应的 sysfs 文件系统条目。
    

2. 使用 kobject_init_and_add()函数创建 kobject：
    

kobject_init_and_add()函数：

- 需要手动分配内存，
    
- 并通过 kobject_init()函数对分配的内存进行初始化。
    
- 此时需要自己实现 ktype 结构体。
    
- 初始化完成后， 调用 kobject_add()函数将 kobject添加到系统中。
    

  

无论是哪种方法， 最终都会调用 **kobject_add()** 函数将 **kobject** 添

  

## 2 实现

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4890a95cf8ea7c0079f5eb8ce80d18e4.png)


其中：kobject_creat

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a29193560c947e3b22238d8efcce8f88.png)


## 3 实验代码

我们编写驱动代码演示如何在 Linux 内核模块中创建和管理 kobject 对象， 其中包括俩种方法创建 kobject 对象， 分别使用 kobject_create_and_add 和 kobject_init_and_add 函数。 通过这些操作， 可以创建具有层次关系的 kobject 对象， 并在模块初始化和退出时进行相应的管理的释放。代码如下所示：

```C
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

// 定义了三个kobject指针变量：mykobject01、mykobject02、mykobject03
struct kobject *mykobject01;
struct kobject *mykobject02;
struct kobject *mykobject03;

// 定义了一个kobj_type结构体变量mytype，用于描述kobject的类型。
struct kobj_type mytype;
// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;
    // 创建kobject的第一种方法
    // 创建并添加了名为"mykobject01"的kobject对象，父kobject为NULL
    mykobject01 = kobject_create_and_add("mykobject01", NULL);
    // 创建并添加了名为"mykobject02"的kobject对象，父kobject为mykobject01。
    mykobject02 = kobject_create_and_add("mykobject02", mykobject01);

    // 创建kobject的第二种方法
    // 1 使用kzalloc函数分配了一个kobject对象的内存
    mykobject03 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 2 初始化并添加到内核中，名为"mykobject03"。
    ret = kobject_init_and_add(mykobject03, &mytype, NULL, "%s", "mykobject03");

    return 0;
}

// 模块退出函数
static void mykobj_exit(void)
{
    // 释放了之前创建的kobject对象
 //   kobject_put(mykobject01);
    kobject_put(mykobject02);
    kobject_put(mykobject03);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
```

```C++
export ARCH=arm64#设置平台架构
export CROSS_COMPILE=aarch64-linux-gnu-#交叉编译器前缀
obj-y := device.o
KDIR :=/home/lubancat/LubanCat_SDK/kernel    #这里是你的内核目录                                                                                                                            
PWD ?= $(shell pwd)
all:
    make -C $(KDIR) M=$(PWD) modules    #make操作
clean:
    make -C $(KDIR) M=$(PWD) clean    #make clean操作
```

## 结果

驱动加载之后， 我们进入/sys/目录下， 如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c9702ea0574d1a73b69a123ead027ce4.png)
