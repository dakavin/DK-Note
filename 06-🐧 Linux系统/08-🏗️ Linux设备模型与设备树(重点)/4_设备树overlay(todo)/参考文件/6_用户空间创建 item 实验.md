我们已经成功在/sys/kernel/config/目录下创建了 myconfigfs 子系统，并在这个子系统下创建了 mygroup 容器， 但是 mygroup 容器下不能使用 mkdir 创建 item。

## 1 代码

```C
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>

// 定义一个名为"mygroup"的config_group结构体
static struct config_group mygroup;

// 自定义的配置项结构体
struct myitem
{
    struct config_item item;
};

// 配置项释放函数
void myitem_release(struct config_item *item)
{
    struct myitem *myitem = container_of(item, struct myitem, item);
    kfree(myitem);
    printk("%s\n", __func__);
}

// 配置项操作结构体
struct configfs_item_operations myitem_ops = {
    .release = myitem_release,
};

// 配置项类型结构体
static struct config_item_type mygroup_item_type = {
    .ct_owner = THIS_MODULE,
    .ct_item_ops = &myitem_ops,
};

// 创建配置项函数
struct config_item *mygroup_make_item(struct config_group *group, const char *name)
{
    struct myitem *myconfig_item;
    printk("%s\n", __func__);
    myconfig_item = kzalloc(sizeof(*myconfig_item), GFP_KERNEL);
    config_item_init_type_name(&myconfig_item->item, name, &mygroup_item_type);
    return &myconfig_item->item;
}

// 配置组操作结构体
struct configfs_group_operations mygroup_ops = {
    .make_item = mygroup_make_item,
};

// 定义名为"mygroup_config_item_type"的config_item_type结构体，用于描述配置项类型。
static const struct config_item_type mygroup_config_item_type = {
    .ct_owner = THIS_MODULE,
    .ct_group_ops = &mygroup_ops,
};

// 定义名为"myconfig_item_type"的配置项类型结构体
static const struct config_item_type myconfig_item_type = {
    .ct_owner = THIS_MODULE,
    .ct_group_ops = NULL,
};

// 定义一个configfs_subsystem结构体实例"myconfigfs_subsystem"
static struct configfs_subsystem myconfigfs_subsystem = {
    .su_group = {
        .cg_item = {
            .ci_namebuf = "myconfigfs",
            .ci_type = &myconfig_item_type,
        },
    },
};

// 模块的初始化函数
static int myconfig_group_init(void)
{
    // 初始化配置组
    config_group_init(&myconfigfs_subsystem.su_group);
    // 注册子系统
    configfs_register_subsystem(&myconfigfs_subsystem);

    // 初始化配置组"mygroup"
    config_group_init_type_name(&mygroup, "mygroup", &mygroup_config_item_type);
    // 在子系统中注册配置组"mygroup"
    configfs_register_group(&myconfigfs_subsystem.su_group, &mygroup);
    return 0;
}

// 模块退出函数
static void myconfig_group_exit(void)
{
    // 注销子系统
    configfs_unregister_subsystem(&myconfigfs_subsystem);
}

module_init(myconfig_group_init); // 指定模块的初始化函数
module_exit(myconfig_group_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
```

## 2 实验

输入“ mkdir test” 命令创建 config_item