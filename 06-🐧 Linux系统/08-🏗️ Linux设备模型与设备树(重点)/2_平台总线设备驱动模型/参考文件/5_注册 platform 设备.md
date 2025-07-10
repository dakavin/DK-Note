  
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f90eccf2c2efe53e36f89f16906bd744.png)


# 一、platform_device_register 注册函数

platform_device_register 函数用于将 platform_device 结构体描述的平台设备注册到内核中。 下面是对 platform_device_register 函数的详细介绍：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e2338d7854b6bb8a6cb6490e0ce21b15.png)


pdev 参数是一个指向 platform_device 结构体的指针， 其中包含了描述平台设备的各种属性和信息。 platform_device 结构体包含了设备名称、 设备资源、 设备 ID 等信息， 用于描述和标识平台设备， 会在接下来的小节对该结构体进行详细的介绍。

该函数在内核源码目录下的“/include/linux/platform_device.h” 文件中， 具体内容如下所示：
```c
extern int platform_device_register(struct platform_device *);
```

函数声明中的 extern 关键字表示该函数在其他地方定义， 而不是在当前文件中实现。 这样的声明通常出现在头文件中， 用于告诉编译器该函数的定义存在于其他源文件中， 以便在编译时能够正确引用该函数。

而 `platform_device_register` 实际定义在“`/drivers/base/platform.c`” 文件中， 相关定义如下所示：

```C
int platform_device_register(struct platform_device *pdev)
{ 
    device_initialize(&pdev->dev);
    arch_setup_pdev_archdata(pdev);
    return platform_device_add(pdev);
}
```

函数内部有三个主要的操作。

- 第 3 行： 调用了 device_initialize 函数， 用于对 pdev->dev 进行初始化。 pdev->dev 是 struct platform_device 结构体中的一个成员， 它表示平台设备对应的 struct device 结构体。 通过调用device_initialize 函数， 对 pdev->dev 进行一些基本的初始化工作， 例如设置设备的引用计数、设备的类型等。
    
- 第 4 行： 调用了 arch_setup_pdev_archdata 函数， 用于根据平台设备的架构数据来设置 pdev的架构相关数据。 这个函数的具体实现可能与具体的架构相关， 它主要用于在不同的架构下对平台设备进行特定的设置。
    
- 第 5 行 ： 调用了 platform_device_add 函 数 ， 将平台设备pdev 添加到内核中 。platform_device_add 函数会完成平台设备的添加操作， 包括将设备添加到设备层级结构中、 添加设备的资源等。 它会返回一个 int 类型的结果， 表示设备添加的结果。
    

```C

/**
 * platform_device_add - add a platform device to device hierarchy
 * @pdev: platform device we're adding
 *
 * This is part 2 of platform_device_register(), though may be called
 * separately _iff_ pdev was allocated by platform_device_alloc().
 */
int platform_device_add(struct platform_device *pdev)
{
        u32 i;
        int ret;
        // 检查输入的平台设备指针是否为空
        if (!pdev)
                return -EINVAL;
        // 如果平台设备的父设备为空， 将父设备设置为 platform_bus
        if (!pdev->dev.parent)
                pdev->dev.parent = &platform_bus;
        // 将平台设备的总线设置为 platform_bus_type
        pdev->dev.bus = &platform_bus_type; //挂在platform bus下
        // 根据平台设备的 id 进行不同的处理
        switch (pdev->id) {
        default:
                // 根据设备名和 id 设置设备的名字
                dev_set_name(&pdev->dev, "%s.%d", pdev->name,  pdev->id);
                break;
        case PLATFORM_DEVID_NONE:
                // 如果 id 为 PLATFORM_DEVID_NONE， 则只使用设备名作为设备的名字
                dev_set_name(&pdev->dev, "%s", pdev->name);
                break;
        case PLATFORM_DEVID_AUTO:
                /*
                 * Automatically allocated device ID. We mark it as such so
                 * that we remember it must be freed, and we append a suffix
                 * to avoid namespace collision with explicit IDs.
                 */
                ret = ida_simple_get(&platform_devid_ida, 0, 0, GFP_KERNEL);
                if (ret < 0)
                        goto err_out;
                pdev->id = ret;
                pdev->id_auto = true;
                dev_set_name(&pdev->dev, "%s.%d.auto", pdev->name, pdev->id);
                break;
        }
        // 遍历平台设备的资源列表， 处理每个资源
        for (i = 0; i < pdev->num_resources; i++) {
                struct resource *p, *r = &pdev->resource[i];
                // 如果资源的名称为空， 则将资源的名称设置为设备的名字
                if (r->name == NULL)
                        r->name = dev_name(&pdev->dev);

                p = r->parent;
                if (!p) {
                        // 如果资源没有指定父资源， 则根据资源类型设置默认的父资源
                        if (resource_type(r) == IORESOURCE_MEM)
                                p = &iomem_resource;
                        else if (resource_type(r) == IORESOURCE_IO)
                                p = &ioport_resource;
                }
                // 如果父资源存在， 并且将资源插入到父资源中失败， 则返回错误
                if (p && insert_resource(p, r)) {
                        dev_err(&pdev->dev, "failed to claim resource %d: %pR\n", i, r);
                        ret = -EBUSY;
                        goto failed;
                }
        }
        // 打印调试信息， 注册平台设备
        pr_debug("Registering platform device '%s'. Parent at %s\n",
                 dev_name(&pdev->dev), dev_name(pdev->dev.parent));
                 
         //Registering platform device 'led_pdev.0'. Parent at platform
         
        // 添加设备到设备层级中， 注册设备
        ret = device_add(&pdev->dev);
        if (ret == 0)
                return ret;

 failed:
         // 如果设备 ID 是自动分配的， 需要移除已分配的 ID
        if (pdev->id_auto) {
                ida_simple_remove(&platform_devid_ida, pdev->id);
                pdev->id = PLATFORM_DEVID_AUTO;
        }
        // 在失败的情况下， 释放已插入的资源
        while (i--) {
                struct resource *r = &pdev->resource[i];
                if (r->parent)
                        release_resource(r);
        }

 err_out:
        return ret;
}
EXPORT_SYMBOL_GPL(platform_device_add);
```

platform_device_unregister 函数的作用是取消注册已经注册的平台设备， 从内核中移除设备 。 它先调用 platform_device_del 函数将设备 从设备层级结构中移除 ， 然后调用platform_device_put 函数减少设备的引用计数， 确保设备在不再被使用时能够被释放。

  

ret = device_add(&pdev->dev); //最後

  

  

## platform_device_unregister 注销函数
```c
void platfrom_device_unregister(struct platform_device *pdev){
	platform_device_del(pdev);
	platform_device_put(pdev);
}
```

函数内部有两个主要的操作：

- 第 3 行： 调用了 platform_device_del 函数， 用于将设备从 platform 总线的设备列表中移除。它会将设备从设备层级结构中移除， 停止设备的资源分配和驱动的匹配。
    
- 第 4 行： 这一步调用了 platform_device_put 函数， 用于减少对设备的引用计数。 这个函数会检查设备的引用计数， 如果引用计数减为零， 则会释放设备结构体和相关资源。 通过减少引用计数， 可以确保设备在不再被使用时能够被释放。
    

platform_device_unregister 函数的作用是取消注册已经注册的平台设备， 从内核中移除设备 。 它先调用 platform_device_del 函数将设备从设备层级结构中移除 ， 然后调用platform_device_put 函数减少设备的引用计数， 确保设备在不再被使用时能够被释放。

## platform_device 结构体

platform_device 结构体是用于描述平台设备的数据结构。 它包含了平台设备的各种属性和信息， 用于在内核中表示和管理平台设备。 该结构体定义在内核的“/include/linux/platform_device.h” 文件中， 具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a9aa11a5d31366eb71409d2df13b8999.png)


下面对于几个重要的参数和结构体进行讲解

1. const char ***name**： 设备的名称， 用于唯一标识设备。 必须提供一个唯一的名称， 以便内核能够正确识别和管理该设备。
    
2. int id： 设备的 ID， 可以用于区分同一种设备的不同实例。 这个参数是可选的， 如果不需要使用 ID 进行区分， 可以将其设置为-1，
    
3. struct device dev： 表示平台设备对应的 struct device 结构体， 用于设备的基本管理和操作。必须为该参数提供一个有效的 struct device 对象， 该结构体的 release 方法必须要实现， 否则在编译的时候会报错。
    
4. u32 num_resources： 设备资源的数量。 如果设备具有资源（ 如内存区域、 中断等） ， 则需要提供资源的数量。
    
5. struct resource *resource： 指向设备资源的指针。 如果设备具有资源， 需要提供一个指向资源数组的指针， 会在下个小节对该结构体进行详细的讲解。
    

## resource 结构体

struct resource 结构体用于描述系统中的设备资源， 包括内存区域、 I/O 端口、 中断等，该结构体定义在内核的“ /include/linux/ioport.h” 文件中， 具体内容如下所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/e0f56fcd63c6c7f176682bc621d517a0.png)


其中最重要的是前四个参数， 每个参数的具体介绍如下所示：

1. **resource_size_t start**： 资源的起始地址。 它表示资源的起始位置或者起始寄存器的地址。
    
2. **resource_size_t end**： 资源的结束地址。 它表示资源的结束位置或者结束寄存器的地址。
    
3. const char ***name**： 资源的名称。 它是一个字符串， 用于标识和描述资源。
    
4. unsigned long **flags**： 资源的标志位。 它包含了一些特定的标志， 用于表示资源的属性或者特征。 例如， 可以用标志位来指示资源的可用性、 共享性、 缓存属性等。 flags 参数的具体取值和含义可以根据系统和驱动的需求进行定义和解释， 但通常情况下， 它用于表示资源的属性、 特征或配置选项。 下面是一些常见的标志位及其可能的含义：
    

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/74d30a4281f29713b36c646b46feadc3.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c3f2787d1ce94c2cf2343a93213a7f13.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/72363cd9c3f7dbab736ca67b09a01974.png)
