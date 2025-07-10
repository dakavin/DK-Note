
## 1 初始化流程

在系统初始化的过程中，我们需要将`DTB`转换成节点是`device_node`的树状结构，以便后续方便操作。具体的代码位于`setup_arch->unflatten_device_tree`中。

## 2 dtb转换为device_node

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a848268112934440d31b3852fe491d73.png)


### 2.1 device_nodej结构图

`Device Tree`中的每一个`node`节点经过`kernel`处理都会生成一个`struct device_node`的结构体, `struct device_node`最终一般会被挂接到具体的`struct device`结构体。`struct device_node`结构体描述如下:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d8dce51ca26ac5a4a7e205123a3463a3.png)


```C++
struct device_node {
    const char *name;             /* name属性的值, 没有为<NULL> */
    const char *type;             /* device_type属性的值，没有为<NULL> */
    phandle phandle;              /* 可用于其他节点引用的标记 */
    const char *full_name;        /* 节点名称 */
    struct fwnode_handle fwnode;
 
    struct  property *properties; /* 指向该设备节点下的第一个属性，其他属性与该属性链表相接 */
    struct  property *deadprops;  /* removed properties */
    struct  device_node *parent;  /* 设备的父节点 */
    struct  device_node *child;   /* 设备子节点 */
    struct  device_node *sibling; /* 设备兄弟节点 */
    struct  kobject kobj;
    unsigned long _flags;
    void    *data;
};
```

### 2.2 Property

`kernel`会根据`Device Tree`中所有的属性解析出数据填充`struct property`结构体。

`struct property`结构体描述如下:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5b12a0e2ed2b45a382d983ee4cd725cf.png)


`kernel`根据`Device Tree`的文件结构信息转换成`struct property`结构体, 并将同一个`node`节点下面的所有属性通过`property.next`指针进行链接, 形成一个单链表。

## 3 unflatten_device_tree调用上下文

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9969ddcfd34ab9c4eb6df9e158b2a3b9.png)


```C++
函数调用关系：
start_kernel // init/main.c
    --->setup_arch(&command_line);  // arch/arm/kernel/setup.c
        --->unflatten_device_tree();   // msm-4.14\drivers\of\fdt.c 
```

### 3.1 unflatten_device_tree

drivers/of/fdt.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/70173b88384db9e4a955da633a44cd9f.png)


参数`initial_boot_params`为`dtb`文件的起始地址, `of_root`用于保存`device_node`的根节点, 以后可以通过该变量找到所有的设备节点, `early_init_dt_alloc_memory_arch`该参数为回调函数用于分配内存给设备节点使用;

### 3.2 3.2` __unflatten_device_tree`

①: 用于计算`Device Tree`转换成`struct device_node`和`struct property`结构体需要分配的内存大小; ②: 调用回调函数根据`unflatten_dt_node()`计算出的`size`分配内存; ③: 填充每一个`struct device_node`和`struct property`结构体;

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/706dac8976bddb8460365ac1858a51ed.png)


这个函数中调用了两次unflatten_dt_node(), 但是两次调用的含义是不同的;

### 3.3 unflatten_dt_node

①: 调用`fdt_get_name()`获取该节点名称; ②: 特殊处理根节点将要分配的内存大小; ③: 从`mem`中为该设备节点分配内存, `size为sizeof(struct device_node) + allocl`, `allocl`为节点名字长度; ④: 成员`full_name`和变量 `fn`都指向该设备节点`data`成员, 接着将节点名字赋值给`fn`变量, 即如下图所示:

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/346b5474ac76136de22ccb9f61e929b2.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7db27552a4e4b790e0623d884ff270c6.png)


fdt_next_node

目录:scripts/dtc/libfdt/fdt.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4aba9b460654d776b0e0d0f2aaba6400.png)

populate_node