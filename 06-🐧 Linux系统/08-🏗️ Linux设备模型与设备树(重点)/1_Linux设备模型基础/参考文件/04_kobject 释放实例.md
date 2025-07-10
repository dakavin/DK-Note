当引用计数器的值变为 0后， 会自动调用自定义的释放函数去执行释放的操作 。

## Kobj 是如何释放的？

要学习 kobject 是如何释放的， 那么我们要先明白 kobj 是如何创建的。 对于 kobject 的创建， 我们可以进一步分析这两种方法的实现细节。

1. 使用 kobject_create_and_add()函数创建 kobject：
    

kobject_create_and_add()函数 ------ 默认方法

- 首先调用 kobject_create()函数， 该函数使用 kzalloc()为kobject 分配内存空间。
    
- 在 kobject_create()函数中， 调用 kobject_init()函数对分配的内存进行初始化，
    
- 并指定了默认的 ktype。
    
- 接下来， 调用 kobject_add()函数将 kobject 添加到系统中， 使其可见。
    
- kobject_add()函数内部调用了 kobject_add_internal()函数， 该函数负责将 kobject 添加到父对象的子对象列表中， 并创建相应的 sysfs 文件系统条目。
    

2. 使用 kobject_init_and_add()函数创建 kobject：
    

kobject_init_and_add()函数： ---- 需要自己实现 ktype 结构体

- 需要手动分配内存，
    
- 并通过 kobject_init()函数对分配的内存进行初始化。
    
- 此时需要自己实现 ktype 结构体。
    
- 初始化完成后， 调用 kobject_add()函数将 kobject添加到系统中。
    

  

无论是哪种方法， 最终都会调用 **kobject_add()** 函数将 **kobject** 添加到系统中， 以使其可见。了解了 kobject 的创建过程后， 我们可以深入学习 kobject 的释放过程。

  

  

  

## kobject_put如何释放的

  

我们来追踪下 kobject 的释放函数——**kobject_put**()函数的实现。

  

在 Linux 内核中， kobject_put()函数用于减少 kobject 的引用计数， 并在引用计数达到 0 时释放 kobject 相关的资源。 如下图所示， 在函数里面， 当引用计数器的值变为 0 以后， 会调用release 函数执行释放的操作。

`lib/kobject.c`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c188785b2a0483d6af8b5904c6058711.png)


Linux 系统帮我们实现好了释放函数， 如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/74e890c2cc6b4a729fed5863c65df116.png)


在上图的 release 函数中， 该函数最终会去调用 kobject_cleanup 函数， kobject_cleanup 函数实现如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/66a624cbcea0de57bd519410f4ca3dff.png)


如上图所示：

- 函数定义了一个名叫 kobject_cleanup 的静态函数， 参数为一个指向 struct kobject 结构体的指针 kobj。
    
- 函数内部定义了一个指向 struct kobj_type 结构体的指针 t， 用于获取 kobj 的类型信息。
    
- 还定义了一个指向常量字符的指针 name， 用于保存 kobj 的名称。
    
- 接下来， 使用 pr_debug 打印调试信息， 显示 kobject 的名称、 地址、 函数名称和父对象的地址。
    
- 然后， 检查 kobj 的类型信息 t 是否存在， 并且检查 t->release 是否为 NULL。
    
- 如果 t 存在但 t->release 为 NULL， 表示 kobj 的类型没有定义释放函数， 会打印调试信息指示该情况。
    
- 接下来， 检查 kobj 的状态变量 state_add_uevent_sent 和 state_remove_uevent_sent。 如果 state_add_uevent_sent 为真而 state_remove_uevent_sent 为假， 表示调用者没有发送"remove" 事件， 会自动发送 "remove" 事件。
    
- 然后， 检查 kobj 的状态变量 state_in_sysfs。 如果为真， 表示调用者没有从 sysfs 中删除kobj， 会自动调用 kobject_del() 函数将其从 sysfs 中删除。
    
- 接下来， 再次检查 t 是否存在， 并且检查 t->release 是否存在。 如果存在， 表示 kobj 的类型定义了释放函数， 会调用该释放函数进行资源清理。
    
- 最后， 检查 name 是否存在。 如果存在， 表示 kobj 的名称是动态分配的， 会释放该名称的内存。 这就是 kobject_cleanup() 函数的实现。 它负责执行 kobject 的资源清理和释放操作，包括处理类型信息、 发送事件、 删除 sysfs 中的对象以及调用释放函数。
    

kobject_cleanup() 函数的实现表明， 最终调用的释放函数是在 kobj_type 结构体中定义的。这解释了为什么在使用 kobject_init_and_add() 函数时， kobj_type 结构体不能为空的原因。 因为释放函数是在 kobj_type 结构体中定义的， 如果不实现释放函数， 就无法进行正确的资源释放。

  

## dynamic_kobj_ktype

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d0b62df884e572582f76a59ca4a78ec5.png)


总结起来， kobj_type 结构体中的释放函数是为了确保在释放 kobject 时执行必要的资源清理和释放操作， 以确保系统的正确运行和资源管理。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cbfa3a78b6123b16a4cf5726760aec01.png)
