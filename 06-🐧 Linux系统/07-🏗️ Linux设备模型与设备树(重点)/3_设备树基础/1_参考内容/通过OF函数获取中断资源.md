

## irq_of_parse_and_map 函数

该函数的主要功能是解析设备节点的"interrupts"属性， 并将对应的中断号映射到系统的中断号。 "interrupts"属性通常以一种特定的格式表示， 可以包含一个或多个中断号。 通过提供索引号， 可以获取对应的中断号。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/05829e373cb1fbdf815078ce3af95f78.png)

实现：

arch/sparc/kernel/of_device_common.c

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/19e1b083c4771a97fd26a071bc57728c.png)


  

填充：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d214db99ad8e7b37f08af07f00020a13.png)


`postcore_initcall` 是 Linux 内核中的一个函数，它是初始化系统核心代码的一部分。在 Linux 内核启动的早期阶段，会执行各种初始化函数来建立基本的系统环境。这些初始化函数按照一定的顺序执行，以确保系统的各个部分能够正确地初始化和交互。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b3a749a65614147093348a234af2aee4.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bd809a89a679ca8474b155b1c8289627.png)


## irq_get_trigger_type 函数

该函数的主要功能是从给定的中断数据结构中提取中断触发类型。 中断触发类型描述了中断信号的触发条件， 例如边沿触发（edge-triggered） 或电平触发（level-triggered） 等。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/24bd2ddfd4e7ad1d6cd72d332e8be66f.png)


## 代码案例

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/80143e9986e140fc863592e29a51f684.png)


### 案例1：

中断资源的获取

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/be0ca8065c71468eb92d55e13cb7c483.png)


中断使用

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6adb811b28bfc1479814f9988b265a11.png)
