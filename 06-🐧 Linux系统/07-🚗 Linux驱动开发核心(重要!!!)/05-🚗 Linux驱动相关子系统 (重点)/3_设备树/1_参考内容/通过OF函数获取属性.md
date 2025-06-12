
📢本篇将介绍通过OF函数获取属性。

linux/of.h

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b6a875c1d178adcc2e7ff064e4a0a5bb.png)


## 获取获取属性

---

**① of_find_property 函数**

of_find_property 函数用于在设备树中查找节点 下具有指定名称的属性。 如果找到了该属性， 可以通过返回的属性结构体指针进行进一步的操作， 比如获取属性值、 属性长度等。

  

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/572332375a2bff6b26ec232832d17862.png)


**② of_property_count_elems_of_size** **函数**

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性中元素的数量。 调用该函数可以用于获取设备树属性中某个属性的元素数量， 比如一个字符串列表的元素数量或**一个整数数组的元素数量**等。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/166376c368ee6facbec8fee56dea636a.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/31e09358fb95a95e57a888090d088520.png)


**③** **of_property_read_u32_index** **函数**

Number 二 1；

该函数在设备树中的设备节点下查找**指定名称的属性**， 并获取该属性在给定索引位置处的u32 类型的数据值。 这个函数通常用于从设备树属性中读取特定索引位置的整数值。 通过指定属性名和索引，可以获取属性中指定位置的具体数值。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ff1a30ae943024fc046aa0d859e2a2e7.png)


**④ of_property_read_u64_index 函数**

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性在给定索引位置处的u64 类型的数据值。 这个函数通常用于从设备树属性中读取特定索引位置的 64 位整数值。 通过指定属性名和索引， 可以获取属性中指定位置的具体数值。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/afa1f32d88abd2f420fe998f2ddd93db.png)


**⑤ of_property_read_variable_u32_array 函数** 该函数用于从设备树中读取指定属性名的变长数组。 通过提供设备节点、 属性名和输出数组的指针， 可以将设备树中的数组数据读取到指定的内存区域中。 同时， 还需要**指定数组的最小大小和最大大小**， 以确保读取到的数组符合预期的大小范围。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e939c86cadb18d4090a74735ac1460a6.png)


上面介绍的函数用于从指定属性中读取变长的 u32 数组， 下面是另外三个读取其他数组大小的函数： 这里给出了四个函数， 用于从设备树中读取数组类型的属性值：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/df0c2697a684ded7604424f9f3b7038b.png)


**⑥ of_property_read_string 函数**

Name "teriiee" 该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性的字符串值， 最后返回读取到的字符串的指针， 通常用于从设备树属性中读取字符串值。 通过指定属性名， 可以获取属性中的字符串数据。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ad80f5f02a6d705189a77861e36ab281.png)


## 案例

## 实验

```C++
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/of.h>

struct device_node *mydevice_node;      
int num;
u32 value_u32;
u64 value_u64;
u32 out_value[2];
const char *value_compatible;
struct property *my_property;

// 平台设备的探测函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 通过节点名称查找设备树节点
    mydevice_node = of_find_node_by_name(NULL, "myLed");
    printk("mydevice node is %s\n", mydevice_node->name);

    // 查找compatible属性
    my_property = of_find_property(mydevice_node, "compatible", NULL);
    printk("my_property name is %s\n", my_property->name);

    // 获取reg属性的元素数量
    num = of_property_count_elems_of_size(mydevice_node, "regtest", 4);
    printk("reg num is %d\n", num);

    // 读取reg属性的值
    of_property_read_u32_index(mydevice_node, "regtest", 0, &value_u32);
    of_property_read_u64_index(mydevice_node, "regtest", 0, &value_u64);
    printk("value u32 is 0x%X\n", value_u32);
    printk("value u64 is 0x%llx\n", value_u64);

    // 读取reg属性的变长数组
    of_property_read_variable_u32_array(mydevice_node, "regtest", out_value, 1, 2);
    printk("out_value[0] is 0x%X\n", out_value[0]);
    printk("out_value[1] is 0x%X\n", out_value[1]);

    // 读取compatible属性的字符串值
    of_property_read_string(mydevice_node, "compatible", &value_compatible);
    printk("compatible value is %s\n", value_compatible);

    return 0;
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
    {.compatible="rockchip,test_devicetree"},
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

MODULE_LICENSE("GPL");
```

设备树：

myLed{

compatible = "rockchip,test_devicetree";

regtest = <0xFDD60000 0x00000004>;

mode = "2";

status = "okay";

};