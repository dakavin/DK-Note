
在前面的章节中，我们学习了kobject的基本概念和结构。现在让我们通过实际的编程实践来深入理解kobject的创建、使用和释放机制。

简单来说，这一章将带你**动手实践**kobject的完整生命周期：从创建到释放，从简单使用到引用计数管理。

> [!note]+ 实践目标 
> 通过本章的学习，你将掌握kobject的创建方法、释放机制、引用计数原理，并能够在实际项目中正确使用kobject。

## 1 kobject创建实践

### 1.1 两种创建方法对比

在Linux内核中，创建kobject有两种主要方法，每种方法都有其适用场景：

**方法一：便捷函数法**
- 使用`kobject_create_and_add()`函数
- 内核自动处理初始化和类型设置
- 适合简单的kobject创建

**方法二：手动控制法**
- 使用`kzalloc()`分配内存 + `kobject_init_and_add()`
- 需要自定义kobj_type结构体
- 适合需要自定义行为的kobject

### 1.2 便捷函数分析

kobject_create_and_add()函数代码路径：
- c文件：`内核源码/lib/kobject.c +760`
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5b3755c7afac1f6b4559ca770dc1e5f0.png)
- h文件：`内核源码/include/linux/kobject.h +112`

kobject_create()函数代码路径
- c文件：同上
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/458b2144694081c30cd670bdbfb65a03.png)
- h文件：同上

**流程分析**：
1. 调用`kobject_create()`分配内存
2. 调用`kobject_init()`进行初始化
3. **设置默认的ktype**
4. 调用 kobject_add()函数将 kobject添加到系统中

### 1.3 手动控制分析

**手动创建的流程**：
1. 使用`kzalloc()`手动分配内存
2. 实现自定义的`kobj_type`结构体
3. 调用`kobject_init_and_add()`初始化并添加
4. 调用 kobject_add()函数将 kobject添加到系统中

> [!tip]+ 提示
> 无论是哪种方法， 最终都会调用 kobject_add()函数将 kobject 添加到系统中， 以使其可见。了解了 kobject 的创建过程后， 我们可以深入学习 kobject 的释放过程。

> [!tip]+ 选择建议 
> 如果只是创建简单的目录结构，使用方法一；
> 如果需要自定义属性和行为，使用方法二。

### 1.4 实践代码：创建kobject

让我们通过一个完整的内核模块来演示两种创建方法：

```c fold
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/slab.h>

//定义三个kobj指针变量
struct kobject *dk_kobj1, *dk_kobj2, *dk_kobj3;

//定义kobj_type结构体，用于方法二
struct kobj_type dk_type;

//模块初始化函数
static int __init my_kobj_init(void){
    int ret;

    //方法1：使用便捷函数
    // 父对象为NULL，表示在校生/sys目录下
    dk_kobj1 = kobject_create_and_add("dk_kobj1", NULL);

    //创建第二个，表示层次关系
    dk_kobj2 = kobject_create_and_add("dk_kobj2", dk_kobj1);

    //方法二：手动分配内存并初始化
    //1.分配内存
    dk_kobj3 = kmalloc(sizeof(struct kobject), GFP_KERNEL);
    if (dk_kobj3 == NULL){
        printk("kmalloc error");
        return -ENOMEM;
    }
    //2.初始化并添加到系统
    ret = kobject_init_and_add(dk_kobj3, &dk_type, NULL, "%s", "dk_kobj3");
    if(ret){
        printk("kobject_init_and_add error");
        kfree(dk_kobj3);
        return ret;
    }

    printk(KERN_INFO "kobject创建成功\n");
    return 0;
}

static void __exit my_kobj_exit(void){
    //注意：kobj1和kobj2释放时自动处理引用计数
    //kobj1和kobj2释放kobj2即可,暂时注释，稍后解释原因
    kobject_put(dk_kobj2);
    kobject_put(dk_kobj3);

    printk(KERN_INFO "kobject释放完成\n");
}

module_init(my_kobj_init);
module_exit(my_kobj_exit);

MODULE_LICENSE("GPL");
```

- Makefile参考之前内核模块章节写的
- 编译命令一样可以参考内核模块章节写的

编译并加载模块后，可以在/sys目录下看到创建的kobject：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b1f3c0b7270b3dc28792ed3d23a65011.png)

**目录结构**：
```c
/sys/
├── mykobject01/
│   └── mykobject02/
└── mykobject03/
```

> [!success]+ 创建成功 
> 可以看到kobject在sysfs中创建了对应的目录结构，体现了父子层次关系。

## 2 kobject释放机制深度解析

### 2.1 引用计数的工作原理

在深入了解释放机制之前，我们需要理解 **引用计数** 这个关键概念。

**引用计数**是一种内存管理技术，用于跟踪对象或资源的引用数量。它通过以下机制工作：
1. 对象创建时，引用计数初始化为1
2. 当有新引用指向对象时，引用计数增加
3. 当引用被删除时，引用计数减少
4. 当引用计数为0时，自动释放对象

**引用示例**：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dda119fd89465bef2a50282308f1e4f2.png)

### 2.2 kref结构详解

**kref嵌入在kobject中**：
```c
struct kobject {
    // ... 其他字段
    struct kref kref;  // 引用计数
    // ... 其他字段
};
```

Linux内核中的引用计数通过`kref`结构实现：
```c
struct kref{
	refcount_t refcount;
}

typedef struct{
	atmoic_t refs;
} refcount_t;

typedef struct {
    int counter;
} atomic_t;
```

### 2.3 引用计数API

```c
// 初始化引用计数为1
static inline void kref_init(struct kref *kref)
{
    refcount_set(&kref->refcount, 1);
}

// 增加引用计数
static inline void kref_get(struct kref *kref)
{
    refcount_inc(&kref->refcount);
}

// 减少引用计数，当计数为0时调用release函数
static inline int kref_put(struct kref *kref, void (*release)(struct kref *kref))
{
    if (refcount_dec_and_test(&kref->refcount)) {
        release(kref);
        return 1;
    }
    return 0;
}

// 设置引用计数值
static inline void refcount_set(refcount_t *r, int n)
{
    atomic_set(&r->refs, n);
}
```

### 2.4 kobject_put释放流程分析

```c
// 内核源码/lib/koject.c
void kobject_put(struct kobject *kobj)
{
	if (kobj) {
		//1. 内核对象没有初始化，直接警告
		if (!kobj->state_initialized)
			WARN(1, KERN_WARNING
				"kobject: '%s' (%p): is not initialized, yet kobject_put() is being called.\n",
			     kobject_name(kobj), kobj);
		//2. 通过kref_put来实现引用计数的减少
		kref_put(&kobj->kref, kobject_release);
	}
}
EXPORT_SYMBOL(kobject_put);
```

通过上面我们知道kref_put函数，在引用计数为0的时候，会调用release函数（`默认是kobject_release`）
```c
static void kobject_release(struct kref *kref)
{
	struct kobject *kobj = container_of(kref, struct kobject, kref);
#ifdef CONFIG_DEBUG_KOBJECT_RELEASE
	unsigned long delay = HZ + HZ * (get_random_int() & 0x3);
	pr_info("kobject: '%s' (%p): %s, parent %p (delayed %ld)\n",
		 kobject_name(kobj), kobj, __func__, kobj->parent, delay);
	INIT_DELAYED_WORK(&kobj->release, kobject_delayed_cleanup);

	schedule_delayed_work(&kobj->release, delay);
#else
	kobject_cleanup(kobj);
#endif
}
```
- 可以看到，最后会调用kobject_cleanup函数

**kobject_cleanup函数详解**：
```c
static void kobject_cleanup(struct kobject *kobj)
{
	//获取 kobj 的类型信息
    struct kobj_type *t = get_ktype(kobj);
    //保存 kobj 的名称
    const char *name = kobj->name;
	//打印调试信息， 显示 kobject 的名称、 地址、 函数名称和父对象的地址
    pr_debug("kobject: '%s' (%p): %s, parent %p\n",
             kobject_name(kobj), kobj, __func__, kobj->parent);
	//检查 kobj 的类型信息 t 是否存在， 并且检查 t->release 是否为 NULL
    if (t && !t->release)
        pr_debug("kobject: '%s' (%p): does not have a release() function, "
                 "it is broken and must be fixed.\n",
                 kobject_name(kobj), kobj);

    /* 处理uevent事件 */
    //检查 kobj 的状态变量
    //state_add_uevent_sent 为真而 state_remove_uevent_sent 为假， 表示调用者没有发送"remove" 事件
    if (kobj->state_add_uevent_sent && !kobj->state_remove_uevent_sent)
	    //自动发送 "remove" 事件
        kobject_uevent(kobj, KOBJ_REMOVE);

    /* 从sysfs中删除 */
    //真：调用者没有从 sysfs 中删除kobj
    if (kobj->state_in_sysfs)
	    //自动调用 kobject_del() 函数将其从 sysfs 中删除
        kobject_del(kobj);

    /* 调用自定义的release函数 */
    //再次检查 t 是否存在， 并且检查 t->release 是否存在
    if (t && t->release) {
	    //kobj 的类型定义了释放函数， 会调用该释放函数进行资源清理
        t->release(kobj);
    }

    /* 释放名称内存 */
    //如果存在， 表示 kobj 的名称是动态分配的， 会释放该名称的内存
    if (name) {
        pr_debug("kobject: '%s': free name\n", name);
        kfree(name);
    }
}
```

> [!warning]+ 重要提醒 
> kobject_cleanup() 函数的实现表明， 最终调用的释放函数是在 kobj_type 结构体中定义的。这解释了为什么在使用 kobject_init_and_add() 函数时， kobj_type 结构体不能为空的原因。 因为释放函数是在 kobj_type 结构体中定义的， 如果不实现释放函数， 就无法进行正确的资源释放。

### 2.5 默认释放函数

对于使用`kobject_create_and_add()`创建的kobject，内部调用了`kobject_create`用于创建内核对象。

在上一章我们知道，kobject_create创建的时候会给内核对象赋值上默认的ktype->`dynamic_kobj_ktype`，而这个`dynamic_kobj_ktype`中定义了默认的释放函数`dynamic_kobj_release`，详细可以看下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/789c5f6fbed8eece13fb7331305ed9e6.png)
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/adac2e8db5434749f1bc60ce3a2c4a6f.png)

## 3 引用计数实验

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/kernel.h>
#include <linux/kobject.h>


struct kobject *dk_kobj1;
struct kobject *dk_kobj2;
struct kobject *dk_kobj3;

struct kobj_type dk_ktype;

static int __init dk_kobj_init(void){
    int ret;
    //创建第一个kobject
    dk_kobj1 = kobject_create_and_add("dk_kobj1", NULL);
    printk("dk_kobj1 引用计数: %d\n",dk_kobj1->kref.refcount.refs.counter);
    //创建第二个kobject，为dk_kobj1添加子对象
    dk_kobj2 = kobject_create_and_add("dk_kobj2", dk_kobj1);
    printk("创建子对象后，dk_kobj1 引用计数: %d\n",dk_kobj1->kref.refcount.refs.counter);
    printk("dk_kobj2 引用计数: %d\n",dk_kobj2->kref.refcount.refs.counter);
    
    //手动创建第三个kobject
    dk_kobj3 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    ret = kobject_init_and_add(dk_kobj3, &dk_ktype, NULL, "%s", "dk_kobj3");
    printk("dk_kobj3 引用计数: %d\n",dk_kobj3->kref.refcount.refs.counter);

    return 0;
}

static void __exit dk_kobj_exit(void){
    printk("释放前的引用计数:\n");
    printk("dk_kobj1: %d\n", dk_kobj1->kref.refcount.refs.counter);
    printk("dk_kobj2: %d\n", dk_kobj2->kref.refcount.refs.counter);
    printk("dk_kobj3: %d\n", dk_kobj3->kref.refcount.refs.counter);
    
    // 释放子对象
    kobject_put(dk_kobj2);
    printk("释放dk_kobj2后，dk_kobj1引用计数: %d\n", 
           dk_kobj2->kref.refcount.refs.counter);
    
    // 释放其他对象
    kobject_put(dk_kobj1);   //没必要执行
    kobject_put(dk_kobj3);
}

module_init(dk_kobj_init);
module_exit(dk_kobj_exit);

MODULE_LICENSE("GPL");
```

**实验结果分析**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d3745399e18eb9c79b1321430a39b20e.png)
1. **创建对象**：dk_kobj1引用计数为1
2. **创建子对象**：dk_kobj1引用计数为2（子对象持有父对象引用）
3. **创建独立对象**：dk_kobj3引用计数为1

![image.png](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/87b379046868ff465ed46203aa5ab27c.png)

**更复杂的情况**：
- 如果objectA下创建两个子对象，objectA的引用计数为3
- 如果在objectB下创建子对象，objectB的引用计数为2

> [!tip]+ 引用计数规律 
> 父对象的引用计数 = 1（自身） + 子对象数量。这确保了只有当所有子对象都被释放后，父对象才会被真正释放。

**释放时的引用计数变化**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/17939a5f03584f5236d19b6fcaaba353.png)
- 当引用计数降为0时，对象被自动释放并进行相关的清理操作

## 4 自定义释放函数

当使用手动创建方法时，需要实现自定义的释放函数
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

//自定义的kobject结构体
struct dk_kobj{
    //这个是结构体成员，不需要分配内存
    struct kobject kobj;
    int value;
    char *data;
};

static struct dk_kobj *dk_kobj;

//自定义释放函数
static void dk_kobj_release(struct kobject *kobj){
    struct dk_kobj *dk_kobj = container_of(kobj, struct dk_kobj, kobj);

    printk(KERN_ALERT "释放自定义的dk_kobj: %p\n",dk_kobj);

    //释放动态分配的资源
    if(dk_kobj->data){
        kfree(dk_kobj->data);
    }

    //释放对象本身
    kfree(dk_kobj);
}

//定义dk_type
static struct kobj_type dk_type = {
    .release = dk_kobj_release,
    //还可以添加其他字段如sys_ops、default_attrs等
};

//使用示例
static int __init self_kobj_init(void){ 
    int ret;

    //分配内存
    dk_kobj = kzalloc(sizeof(struct dk_kobj), GFP_KERNEL);
    if(!dk_kobj){
        return -ENOMEM;
    }
    //初始化自定义字段
    dk_kobj->value = 27;
    dk_kobj->data = kzalloc(100, GFP_KERNEL);
    dk_kobj->data = "dk_kobj test self make";
    //初始化并添加kobj
    ret = kobject_init_and_add(&dk_kobj->kobj, &dk_type,NULL,"dk_kobj");
    if(ret){
        kfree(dk_kobj);
        return ret;
    }
    printk(KERN_ALERT "创建dk_kobj成功，指针：%p, 参数value：%d, 参数data：%s\n",dk_kobj,dk_kobj->value,dk_kobj->data);
    return 0;
}

static void __exit self_kobj_exit(void){
    printk(KERN_ALERT "开始删除dk_kobj\n");
    kobject_put(&dk_kobj->kobj);
};

module_init(self_kobj_init);
module_exit(self_kobj_exit);

MODULE_LICENSE("GPL");
```

**实验结果**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0282beb7d5ab6e9851cc2410bf2ee2ea.png)

> [!warning]+ 常见错误
> 
> 1. 忘记实现release函数
> 2. 在release函数中没有释放所有动态分配的资源
> 3. 不正确的引用计数管理导致内存泄漏或过早释放

## 5 实践建议

### 5.1 选择合适的常见方法

**使用便捷函数的场景**：
- 只需要简单的目录结构
- 不需要自定义属性和行为
- 快速原型开发

**使用手动创建的场景**：
- 需要嵌入到自定义结构体中
- 需要自定义属性文件
- 需要特殊的释放逻辑

### 5.2 引用计数管理原则

1. **配对原则**：每个`kobject_get()`都要有对应的`kobject_put()`
2. **层次释放**：先释放子对象，再释放父对象
3. **异常处理**：在错误路径中正确释放已分配的资源

### 5.3 调试技巧

```c
// 开启kobject调试信息
echo 1 > /sys/kernel/debug/kobject/debug

// 查看当前的kobject状态
cat /proc/slabinfo | grep kobject

// 检查sysfs目录结构
find /sys -name "your_object_name" -type d
```

> [!abstract]+ 实践总结 
> 通过本章的学习，我们掌握了：
> 
> - **两种kobject创建方法**及其适用场景
> - **引用计数机制**的工作原理和重要性
> - **kobject释放流程**的完整过程
> - **自定义释放函数**的实现方法
> - **最佳实践**和常见错误的避免

这些知识为我们使用更高级的设备模型组件（如总线、设备、驱动）奠定了坚实的基础。下一章我们将学习kset的实际使用方法！