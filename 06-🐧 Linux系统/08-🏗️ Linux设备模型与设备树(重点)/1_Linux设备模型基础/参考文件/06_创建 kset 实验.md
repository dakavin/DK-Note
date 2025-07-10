本章节是关于在 Linux 上创建 kset 的实验。 在实验中介绍了如何使用代码创建 kset， 并将多个 kobject 与 kset 关联起来。 通过演示实验现象， 讲解了 kset 是一组 kobject 的集合， 并解释了 kobject 在 sys 目录下生成的原因


## kset重点知识

- kset结构体
    
- kset的作用
    

![whiteboard_exported_image (1).png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e3543fc6e87d43f67d55189d66658bc3.png)


## kset编程接口

- 创建一个kset对象
    
- 指定kset的parent
    
- 创建kset下的属性
    

```C
void kset_init (struct kset *kset);
int kset_register (struct kset *kset);
void kset_unregister (struct kset *kset);
struct kset *kset_create_and_add(const char *name, const struct kset_uevent_ops *u,
struct kobject *parent_kobj)
```

  

## 实验

这段代码用于定义并初始化两个自定义内核对象 mykobject01 和mykobject02， 并将它们添加到一个自定义内核对象集合 mykset 中。 这些自定义内核对象可以用于在 Linux 内核中表示和管理特定的功能或资源。

```C
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

// 定义kobject结构体指针，用于表示第一个自定义内核对象
struct kobject *mykobject01;
// 定义kobject结构体指针，用于表示第二个自定义内核对象
struct kobject *mykobject02;
// 定义kset结构体指针，用于表示自定义内核对象的集合
struct kset *mykset;
// 定义kobj_type结构体，用于定义自定义内核对象的类型
struct kobj_type mytype;

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;

    // 创建并添加kset，名称为"mykset"，父kobject为NULL，属性为NULL
    mykset = kset_create_and_add("mykset", NULL, NULL);

    // 为mykobject01分配内存空间，大小为struct kobject的大小，标志为GFP_KERNEL
    mykobject01 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 将mykset设置为mykobject01的kset属性
    mykobject01->kset = mykset;
    // 初始化并添加mykobject01，类型为mytype，父kobject为NULL，格式化字符串为"mykobject01"
    ret = kobject_init_and_add(mykobject01, &mytype, NULL, "%s", "mykobject01");

    // 为mykobject02分配内存空间，大小为struct kobject的大小，标志为GFP_KERNEL
    mykobject02 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 将mykset设置为mykobject02的kset属性
    mykobject02->kset = mykset;
    // 初始化并添加mykobject02，类型为mytype，父kobject为NULL，格式化字符串为"mykobject02"
    ret = kobject_init_and_add(mykobject02, &mytype, NULL, "%s", "mykobject02");

    return 0;
}

// 模块退出函数
static void mykobj_exit(void)
{
    // 释放mykobject01的引用计数
    kobject_put(mykobject01);

    // 释放mykobject02的引用计数
    kobject_put(mykobject02);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
```

驱动加载之后， 我们进入/sys/目录下， 可以看到创建生成的 kset， 如下图所示， 我们进到mykset 目录下， 可以看到创建的 kobject。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/aafe115755a8fae2cb5ef167fe511462.png)


  

  

# 五、/sys/bus 的创建案例

kset_create_and_add("bus"

`drivers/base/bus.c`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d6231139f4ac4f82b28f715df2c2d9bc.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a95208c3912c9c9da01fdb9a0a5b0e09.png)
