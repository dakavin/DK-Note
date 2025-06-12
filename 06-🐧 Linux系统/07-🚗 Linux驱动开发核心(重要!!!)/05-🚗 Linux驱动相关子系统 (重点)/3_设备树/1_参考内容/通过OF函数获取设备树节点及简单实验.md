

📢本篇将介绍通过OF函数获取设备树节点实验

## 获取获取设备树节点

---

在 `Linux` 内核源码中提供了一系列的 `of` 操作函数来帮助我们获取到设备树中编写的属性，在内核中以 `device_node` 结构体来对设备树进行描述， 所以 `of` 操作函数实际上就是获取`device_node` 结构体， 所以接下来我们学习的 `of` 操作函数的返回值都是 `device_node` 结构体。

**① of_find_node_by_name 函数**

of_find_node_by_name 是 Linux 内核中用于通过节点名称查找设备树节点的函数。 下面是对 of_find_node_by_name 函数的详细介绍：

**② of_find_node_by_path 函数**

of_find_node_by_path 是 Linux 内核中用于通过节点路径查找设备树节点的函数。 下面是对 of_find_node_by_path 函数的详细介绍：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/36dd1a2279fdf2ece06e08fa46b103fe.png)


of_find_node_by_path 函数通过节点路径在设备树中进行查找。 路径是设备树节点从根节点到目标节点的完整路径。 可以通过指定正确的路径来准确地访问设备树中的特定节点。

使用 of_find_node_by_path 函数时， 可以直接传递节点的完整路径作为 path 参数， 函数会在设备树中查找匹配的节点。 这对于已知节点路径的情况非常有用。

**③ of_get_parent 函数**

在 Linux 内核中， of_get_parent 函数用于获取设备树节点的父节点。下面是对 of_get_parent函数的详细介绍：

  

使用 of_get_parent 函数时， 可以将特定的设备树节点作为参数传递给函数， 然后它将返回该节点的父节点。 这对于在设备树中导航和访问节点之间的层次关系非常有用。

父节点在设备树中表示了节点之间的层次结构关系。 通过获取父节点， 你可以访问上一级节点的属性和配置信息， 从而更好地理解设备树中的节点之间的关系。

**④ of_get_next_child 函数**

在 Linux 内核中， of_get_next_child 函数用于获取设备树节点的下一个子节点。 下面是对of_get_next_child 函数的详细介绍：

使用 of_get_next_child 函数时， 可以传递当前节点以及上一个子节点作为参数。 函数将从上一个子节点的下一个节点开始， 查找并返回下一个子节点。 设备树中的子节点表示了节点之间的层次关系。 通过获取子节点， 你可以遍历和访问当前节点的所有子节点， 以便进一步处理它们的属性和配置信息。

**⑤ of_ find_ compatible_ node 函数**

用 of_find_compatible_node 函数。 该函数用于在设备树中查找与指定兼容性字符串匹配的节点。

使用 of_find_compatible_node 函数时， 可以指定起始节点和需要匹配的设备类型字符串以及兼容性字符串。 函数会从起始节点开始遍历设备树， 查找与指定兼容性字符串匹配的节点，并返回匹配节点的指针。

**⑥ of_ find matching node_ and_ match 函数**

在 Linux 内核中， of_ find matching node_ and_ match 函数用于根据给定的 of_device_id 匹配表在设备树中查找匹配的节点。

  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1f2811bad0f15d1f50a3ec4f5bbb2258.png)


of_find_matching_node_and_match 函数在设备树中遍历节点， 对每个节点使用__of_match_node 函数进行匹配。 如果找到匹配的节点， 将返回该节点的指针， 并将 match 指针更新为匹配到的 of_device_id 条目， 函数会自动增加匹配节点的引用计数。 以下是使用 of_find_matching_node_and_match 函数的示例代码：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0d2238f5d4cebd3d4a62536d16a4c060.png)


在上述示例中， 我们定义了一个 of_device_id 匹配表 my_match_table， 其中包含了一个兼容性字符串为"vendor,device"的匹配项。 然后， 我们使用 of_find_matching_node_and_match 函数从根节点开始查找匹配的节点。

## 驱动程序

驱动代码：

```C
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/of.h>

struct device_node *mydevice_node;      
const struct of_device_id *mynode_match;
struct of_device_id mynode_of_match[] = {
        {.compatible="my devicetree"},
        {},
};
const char *value_compatible;
const int int_value;
// 平台设备的初始化函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 通过节点名称查找设备树节点
    mydevice_node = of_find_node_by_name(NULL, "myLed");
    printk("mydevice node is %s\n", mydevice_node->name);
    
        // 通过节点路径查找设备树节点
    mydevice_node = of_find_node_by_path("/myLed");
    printk("mydevice node is %s\n", mydevice_node->name);
        
    // 获取父节点
    mydevice_node = of_get_parent(mydevice_node);
    printk("myled's parent node is %s\n", mydevice_node->name);/
            
    // 获取子节点
    mydevice_node = of_get_next_child(mydevice_node, NULL);
    printk("myled's sibling node is %s\n", mydevice_node->name);

    // 使用compatible值查找节点
    mydevice_node=of_find_compatible_node(NULL ,NULL, "my devicetree");
    printk("mydevice node is %s\n" , mydevice_node->name);
        
   //根据给定的of_device_id匹配表在设备树中查找匹配的节点
   mydevice_node=of_find_matching_node_and_match(NULL , mynode_of_match, &mynode_match);
        printk("mydevice node is %s\n" ,mydevice_node->name);
        return 0;
        
        
    of_property_read_string(mydevice_node, "compatible", &value_compatible);
    printk("compatible value is %s\n", value_compatible); //my devicetree
    
    of_property_read_u32(mydevice_node, "modetest", &int_value);
    
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}


const struct of_device_id of_match_table_id[]  = {
        {.compatible="my devicetree"},
};

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
                .of_match_table =  of_match_table_id,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    platform_driver_unregister(&my_platform_driver);
    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit); 
```

设备树代码：

```C++
   myLed{
            compatible = "my devicetree";
            reg = <0xFDD60000 0x00000004>;
            mode = "2";
            modetest = 2;
    };
```