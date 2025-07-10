理解 ConfigFS 的核心数据结构对于深入使用和定制 ConfigFS 非常重要， 可以帮助开发者更好地进行设备的配置和管理， 提高系统的灵活性和可扩展性。

## 1 数据结构

configFs 文件系统相关的几个核心数据结构。ConfigFS 的核心数据结构主要包括以下几个部分：

- configfs_subsystem： configfs_subsystem 是一个顶层的数据结构， 用于表示整个 ConfigFS 子系统。 它包含了根配置项组的指针， 以及 ConfigFS 的其他属性和状态信息。
    
- config_group： config_group 是一种特殊类型的配置项， 表示一个配置项组。 它可以包含一组相关的配置项， 形成一个层次结构。 config_group 结构包含了父配置项的指针， 以及指向子配置项的链表。
    
- config_item： 这是 ConfigFS 中最基本的数据结构， 用于表示一个配置项。 每个配置项都是一个内核对象， 可以是设备、 驱动程序、 子系统等。 config_item 结构包含了配置项的类型、名称、 属性、 状态等信息， 以及指向父配置项和子配置项的指针。
    

这些数据结构之间的关系可以形成一个树形结构， 其中 configfs_subsystem 是根节点， config_group 表示配置项组， config_item 表示单个配置项。 子配置项通过链表连接在一起，

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7644489bef68df3f1c440d1f41eaa3ec.png)


  

## 2 结构体关系

```C
struct configfs_subsystem {
    struct config_group su_group;
    struct mutex su_mutex;
};
```

configfs_subsystem 结构体中包含 config_group 结构体， config_group 结构体如下所示：

```C
struct config_group {
    struct config_item cg_item;
    struct list_head cg_children;
    struct configfs_subsystem *cg_subsys;
    struct list_head struct list_head default_groups;group_entry;
};
```

config_group 结构体中包含 config_item 结构体， config_item 结构体如下所示：

```C
struct config_item {
    char *ci_name;
    char ci_namebuf[CONFIGFS_ITEM_NAME_LEN]; //目录的名字
    struct kref ci_kref;
    struct list_head ci_entry;
    struct config_item *ci_parent;
    struct config_group *ci_group;
    const struct config_item_type *ci_type; //目录下属性文件和属性操作
    struct dentry *ci_dentry;
};
```

## 3 案例

### 3.1 configfs_subsystem 实例

这段代码定义了一个名为 dtbocfg_root_subsys 的 configfs_subsystem 结构体实例， 表示ConfigFS 中的一个子系统。

```C
static struct configfs_subsystem dtbocfg_root_subsys = {
    .su_group = {
        .cg_item = {
            .ci_namebuf = "device-tree",
            .ci_type    = &dtbocfg_root_type,
        },
    },
  .su_mutex = __MUTEX_INITIALIZER(dtbocfg_root_subsys.su_mutex),
};
```

首先， dtbocfg_root_subsys.su_group 是一个 config_group 结构体， 它表示子系统的根配置项组。 在这里， 该结构体的.cg_item 字段表示根配置项组的基本配置项。

- .ci_namebuf = "device-tree"： 配置项的名称设置为"device-tree"， 表示该配置项的名称为
    

"device-tree"。

- .ci_type = &dtbocfg_root_type： 配置项的类型设置为 dtbocfg_root_type， 这是一个自定义的配置项类型。
    

接下来， .su_mutex 字段是一个互斥锁， 用于保护子系统的操作。 在这里， 使用了__MUTEX_INITIALIZER 宏来初始化互斥锁。

  

通过这段代码， 创建了一个名为"device-tree"的子系统， 它的根配置项组为空。 可以在该子系统下添加更多的配置项和配置项组， 用于动态配置和管理设备树相关的内核对象。

  

### 3.2 config_group 实例化

这段代码的作用是初始化和注册一个名为"device-tree"的 ConfigFS 子系统， 并在其下创建一个名为"overlays"的配置项组。 Linux 系统下， 在 device-tree 子系统下创建了 overlays 容器。

```C
static struct config_group dtbocfg_overlay_group;

static struct configfs_group_operations dtbocfg_overlays_ops = {
    .make_item      = dtbocfg_overlay_group_make_item,
    .drop_item      = dtbocfg_overlay_group_drop_item,
};

static struct config_item_type dtbocfg_overlays_type = {
    .ct_group_ops   = &dtbocfg_overlays_ops,
    .ct_owner       = THIS_MODULE,
};

static int __init dtbocfg_module_init(void)
{
    int retval = 0;

    pr_info("%s\n", __func__);

    config_group_init(&dtbocfg_root_subsys.su_group);
    config_group_init_type_name(&dtbocfg_overlay_group, "overlays", &dtbocfg_overlays_type);

    retval = configfs_register_subsystem(&dtbocfg_root_subsys);
    if (retval != 0) {
        pr_err( "%s: couldn't register subsys\n", __func__);
        goto register_subsystem_failed;
    }

    retval = configfs_register_group(&dtbocfg_root_subsys.su_group, &dtbocfg_overlay_group);
    if (retval != 0) {
        pr_err( "%s: couldn't register group\n", __func__);
        goto register_group_failed;
    }

    pr_info("%s: OK\n", __func__);
    return 0;

  register_group_failed:
    configfs_unregister_subsystem(&dtbocfg_root_subsys);
  register_subsystem_failed:
    return retval;
}
```

- 首先， 通过 config_group_init()函数初始化了 dtbocfg_root_subsys.su_group， 即子系统的根配置项组。
    
- 接下来， 使用 config_group_init_type_name()函数初始化了dtbocfg_overlay_group， 表示名为"overlays"的配置项组， 并指定了配置项组的类型为dtbocfg_overlays_type， 这是一个自定义的配置项类型。
    
- 然后， 调用 configfs_register_subsystem()函数注册了 dtbocfg_root_subsys 子系统。
    
    - 如果注册失败， 将打印错误信息， 并跳转到register_subsystem_failed 标签处进行错误处理。
        
- 接着， 调用 configfs_register_group()函数注册了 dtbocfg_overlay_group 配置项组， 并将其添加到 dtbocfg_root_subsys.su_group 下。
    
    - 如果注册失败， 同样会打印错误信息， 并跳转到 register_group_failed 标签处进行错误处理。
        
- 最后， 如果所有的注册过程都成功， 将打印"OK"消息， 并返回 0， 表示初始化成功。 如果在注册配置项组失败时， 会先调用 configfs_unregister_subsystem()函数注销之前注册的子系统， 然后返回注册失败的错误码 retval。
    

## 4 属性和方法

我们要在容器下放目录或属性文件， 所以我们看一下 config_item 结构体， 如下所示：

```C
struct config_item {
    char *ci_name;
    char ci_namebuf[CONFIGFS_ITEM_NAME_LEN]; //目录的名字
    struct kref ci_kref;
    struct list_head ci_entry;
    struct config_item *ci_parent;
    struct config_group *ci_group;
    const struct config_item_type *ci_type; //目录下属性文件和属性操作
    struct dentry *ci_dentry;
};
```

config_item 结构体中包含了 config_item_type 结构体， config_item_type 结构体如下所示：

```C
struct config_item_type {
    struct module *ct_owner;
    struct configfs_item_operations *ct_item_ops; //item（目录） 的操作方法
    struct configfs_group_operations *ct_group_ops; //group（容器） 的操作方法
    struct configfs_attribute **ct_attrs; //属性文件的操作方法
    struct configfs_bin_attribute **ct_bin_attrs; //bin 属性文件的操作方法
};
```

config_item_type 结构体中包含了 struct configfs_item_operations 结构体， 如下所示：

```C
struct configfs_item_operations {
    //删除 item 方法， 在 group 下面使用 rmdir 命令会调用这个方法void (*release)(struct config_item *);
    int (*allow_link)(struct config_item *src, struct config_item *target);
    void (*drop_link)(struct config_item *src, struct config_item *target);
};  
```

config_item_type 结构体中包含了 struct configfs_group_operations 结构体， 如下所示：

```C
struct configfs_group_operations {
    //创建 item 的方法， 在 group 下面使用 mkdir 命令会调用这个方法
    struct config_item *(*make_item)(struct config_group *group, const char *name); //创建 group 的方法
    struct config_group *(*make_group)(struct config_group *group, const char *name);
    int (*commit_item)(struct config_item *item);
    void (*disconnect_notify)(struct config_group *group, struct config_item *item);
    void (*drop_item)(struct config_group *group, struct config_item *item); 
};  
```

config_item_type 结构体中包含了 struct configfs_attribute 结构体 ， 如下所示：

```C
struct configfs_attribute {
    const char *ca_name; 属性文件的名字
    struct module *ca_owner; 属性文件文件的所属模块
    umode_t ca_mode; 属性文件访问权限
    读写方法的函数指针， 具体功能需要自行实现。
    ssize_t (*show)(struct config_item *, char *);
    ssize_t (*store)(struct config_item *, const char *, size_t); 
};
```

### 4.1 config_item实例化

```C
static struct configfs_attribute *dtbocfg_overlay_attrs[] = {
    &dtbocfg_overlay_item_attr_status,
    NULL,
};

static struct configfs_bin_attribute *dtbocfg_overlay_bin_attrs[] = {
    &dtbocfg_overlay_item_attr_dtbo,
    NULL,
};

static struct configfs_item_operations dtbocfg_overlay_item_ops = {
    .release        = dtbocfg_overlay_release,
};

static struct config_item_type dtbocfg_overlay_item_type = {
    .ct_item_ops    = &dtbocfg_overlay_item_ops,
    .ct_attrs       = dtbocfg_overlay_attrs,
    .ct_bin_attrs   = dtbocfg_overlay_bin_attrs,
    .ct_owner       = THIS_MODULE,
};
```

```C
static struct configfs_group_operations dtbocfg_overlays_ops = {
    .make_item      = dtbocfg_overlay_group_make_item,
    .drop_item      = dtbocfg_overlay_group_drop_item,
};
```

```C
static struct config_item *dtbocfg_overlay_group_make_item(struct config_group *group, const char *name)
{
    struct dtbocfg_overlay_item *overlay;

    pr_debug("%s\n", __func__);

    overlay = kzalloc(sizeof(*overlay), GFP_KERNEL);

    if (!overlay)
        return ERR_PTR(-ENOMEM);
    overlay->id        = -1;
    overlay->dtbo      = NULL;
    overlay->dtbo_size = 0;

    config_item_init_type_name(&overlay->item, name, &dtbocfg_overlay_item_type);
    return &overlay->item;
}
```