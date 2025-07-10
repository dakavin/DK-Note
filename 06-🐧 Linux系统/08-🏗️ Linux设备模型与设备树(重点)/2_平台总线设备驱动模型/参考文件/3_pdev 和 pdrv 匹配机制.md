

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8963f4290b7ca6398afedfa95f57ed14.png)


  

从设备树转换得来的 platform_device 会被注册进内核里，以后当我们每 注册一个 platform_driver 时，它们就会两两确定能否配对，如果能配对成功 就调用 platform_driver 的 probe 函数。

主流Linux4.19 主流都是用的2 为主。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f4de9cea6bfb246299f77d92d2eb2361.png)


最先比较

⚫ **platform_device.driver_override** 和 **platform_driver.driver.name**

可以设置 platform_device 的 driver_override，**强制**选择某个 platform_driver。

然后比较

  

- Of
    

```C
static int __of_device_is_compatible(const struct device_node *device,
                                     const char *compat, const char *type, const char *name)
{
        struct property *prop;
        const char *cp;
        int index = 0, score = 0;

        /* Compatible match has highest priority */
        if (compat && compat[0]) {
                prop = __of_find_property(device, "compatible", NULL);
                for (cp = of_prop_next_string(prop, NULL); cp;
                     cp = of_prop_next_string(prop, cp), index++) {
                        if (of_compat_cmp(cp, compat, strlen(compat)) == 0) {
                                score = INT_MAX/2 - (index << 2);
                                break;
                        }
                }
                if (!score)
                        return 0;
        }
```

  

  

⚫ ② platform_device. name 和 platform_driver.id_table[i].name 设备树 ：id_table ----- 设备树

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1f8effda935f54a58d27150a1c382941.png)


![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/abc666c4617f6b3de8c8b59a58f73850.png)


Platform_driver.id_table 是“platform_device_id”指针，表示该 drv 支持若干 个 device，它里面列出了各个 device 的{.name, .driver_data}，其中的“name”表示该 drv 支持的设备的名字，driver_data 是些提供给该 device 的私有数据。

一对多的关系（一 platform_driver ; 多 platform_device ）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9bf0d2ced81898446473d01ac11d926e.png)


- ③
    

  

最后比较

**⚫ ④ platform_device.name 和 platform_driver.driver.name**

platform_driver.id_table 可能为空， 这时可以根据 platform_driver.driver.name 来寻找同名的 platform_device。

  

函数调用关系：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/47c31cfa7eeecb2e2a75ae486568f3da.png)
