现在我们通过一个完整的实验来学习：**如何在实际项目中使用tasklet处理中断下半部？**

简单来说，这个实验就像搭建一个"智能门铃系统"。当有人按门铃（GPIO按键中断），门铃控制器立即响应记录按铃事件（上半部），然后慢慢执行复杂的处理工作，比如记录日志、发送通知等（下半部tasklet），而不会让按门铃的人等太久。

> [!note]+ 重要笔记
> 
> 本实验将综合运用设备树配置、GPIO中断处理、tasklet下半部机制和字符设备驱动框架，展示完整的Linux驱动开发流程。

## 1 硬件环境分析

### 1.1 硬件连接说明

让我们先了解实验的硬件环境：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a8576e0d71874495cc8d28d4f7d66b72.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8e3964fa8b4c25a8f2819cd1d8ed8ea6.png)

从硬件连接图可以看出，我们将使用**第7个管脚**来触发中断。换句话说，这个引脚就像门铃的按钮，当被按下时会产生中断信号。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b6d5ed8336913a268fd5915d53f5f2d.png)

> [!tip]+ 重要提示
> 
> 在实际硬件调试中，确认引脚连接是第一步，错误的引脚配置会导致中断无法正常触发。

### 1.2 GPIO引脚映射

根据硬件原理图，按键连接到**GPIO0_RK_PB0**引脚，这个引脚将被配置为中断输入模式。简单来说，就是把这个引脚当作"感应器"，检测按键的按下动作。

## 2 设备树配置

### 2.1 设备树节点定义

设备树就像硬件的"说明书"，告诉内核如何识别和配置我们的按键设备。

**文件位置**：`kernel/arch/arm64/boot/dts/rockchip/rk356x-lubancat-rk_series.dts`

```dts
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/interrupt-controller/irq.h>

&{/} {  // 加入到根节点
    button_interrupt: button_interrupt {
        status = "okay";
        compatible = "button_interrupt";
        
        // 引脚控制配置
        pinctrl-names = "default";
        pinctrl-0 = <&button_interrupt_pin>;
        
        // 所属的中断控制器
        interrupt-parent = <&gpio0>;
        // 中断信号源的描述
        interrupts = <RK_PB0 IRQ_TYPE_LEVEL_LOW>;
        
        // GPIO的描述
        buttons-gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_HIGH>;
    };
};

&{/pinctrl} {  // 加入到引脚控制节点
    button_interrupt {
        button_interrupt_pin: button_interrupt_pin {
            rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
};
```

**配置详解**：

> [!example]+ 示例
> 
> **设备树配置就像填写设备"身份证"**：
> 
> - **compatible**：设备类型标识，驱动程序通过它找到对应设备
> - **interrupt-parent**：指定"上级领导"（中断控制器）
> - **interrupts**：描述中断的具体特性（引脚和触发方式）
> - **buttons-gpios**：GPIO引脚的详细描述

**关键配置说明**：

- **IRQ_TYPE_LEVEL_LOW**：低电平触发中断，按键按下时引脚电平拉低
- **GPIO_ACTIVE_HIGH**：GPIO高电平有效
- **&pcfg_pull_none**：不配置内部上拉或下拉电阻

> [!warning]+ 警告或注意
> 
> 中断触发类型的配置必须与硬件电路匹配，错误的配置会导致中断无法正常工作或产生误触发。

### 2.2 设备树编译

配置完成后，需要编译设备树使配置生效。换句话说，就是让系统"读懂"我们编写的硬件说明书。

```bash
# 单独编译设备树
make dtbs
```

## 3 驱动程序实现

### 3.1 驱动程序框架

我们的驱动程序需要整合多个Linux内核机制：

> [!note]+ 重要笔记
> 
> **本驱动综合运用的技术**：
> 
> - 设备树解析：获取硬件配置信息
> - 字符设备驱动框架：提供用户空间接口
> - GPIO管理：控制引脚配置
> - 中断处理：响应按键事件
> - tasklet机制：处理中断下半部

**完整驱动代码**：

```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <linux/of_gpio.h>
#include <asm/io.h>
#include <linux/device.h>
#include <linux/irq.h>
#include <linux/of_irq.h>

/*------------------字符设备相关定义----------------------*/
#define DEV_NAME "button"
#define DEV_CNT (1)

static dev_t button_devno;              // 字符设备的设备号
static struct cdev button_chr_dev;      // 字符设备结构体
struct class *class_button;             // 保存创建的类
struct device *device_button;           // 保存创建的设备

/*------------------设备相关变量----------------------*/
struct device_node *button_device_node = NULL;  // 按键设备节点结构体
unsigned button_GPIO_number = 0;                 // button使用的GPIO引脚编号
u32 interrupt_number = 0;                        // button引脚中断编号
atomic_t button_status = ATOMIC_INIT(0);         // 原子变量保存按键状态，初始值为0

struct tasklet_struct button_tasklet;            // tasklet结构体

// tasklet处理函数（下半部）
void button_tasklet_hander(unsigned long data)
{
    int counter = 1;
    
    // 模拟耗时处理工作
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d\n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d\n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d\n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d\n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d\n", counter++);
}

// 中断处理函数（上半部）
static irqreturn_t button_irq_hander(int irq, void *dev_id)
{
    printk(KERN_ERR "button_irq_hander----------enter");
    
    // 按键状态计数加一
    atomic_inc(&button_status);
    
    // 调度tasklet执行下半部处理
    tasklet_schedule(&button_tasklet);
    
    printk(KERN_ERR "button_irq_hander----------exit");
    return IRQ_HANDLED;
}
```

### 3.2 字符设备操作函数

**设备打开函数**：
```c
// 设备打开函数
static int button_open(struct inode *inode, struct file *filp)
{
    int error = -1;

    // 获取按键设备树节点
    button_device_node = of_find_node_by_path("/button_interrupt");
    if(NULL == button_device_node) {
        printk("of_find_node_by_path error!");
        return -1;
    }

    // 获取按键使用的GPIO
    button_GPIO_number = of_get_named_gpio(button_device_node, "buttons-gpios", 0);
    if(0 == button_GPIO_number) {
        printk("of_get_named_gpio error");
        return -1;
    }

    // 申请GPIO资源
    error = gpio_request(button_GPIO_number, "button_gpio");
    if(error < 0) {
        printk("gpio_request error");
        gpio_free(button_GPIO_number);
        return -1;
    }

    // 设置引脚为输入模式
    error = gpio_direction_input(button_GPIO_number);

    // 获取中断号
    interrupt_number = irq_of_parse_and_map(button_device_node, 0);
    printk("irq_of_parse_and_map = %d\n", interrupt_number);

    // 申请中断
    error = request_irq(interrupt_number, button_irq_hander, 
                       IRQF_TRIGGER_RISING, "button_interrupt", device_button);
    if(error != 0) {
        printk("request_irq error");
        free_irq(interrupt_number, device_button);
        return -1;
    }

    // 初始化tasklet
    tasklet_init(&button_tasklet, button_tasklet_hander, 0);

    return 0;
}
```

**设备读取函数**：
```c
static ssize_t button_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
    int error = -1;
    int button_countervc = 0;

    // 读取按键状态值
    button_countervc = atomic_read(&button_status);

    // 结果拷贝到用户空间
    error = copy_to_user(buf, &button_countervc, sizeof(button_countervc));
    if(error < 0) {
        printk("copy_to_user error");
        return -1;
    }

    // 清零按键状态值
    atomic_set(&button_status, 0);
    return 0;
}
```

**设备关闭函数**：
```c
static int button_release(struct inode *inode, struct file *filp)
{
    // 释放申请的引脚和中断
    free_irq(interrupt_number, device_button);
    gpio_free(button_GPIO_number);
    return 0;
}
```

> [!tip]+ 重要提示
> 
> 这些函数就像设备的"生命周期管理"：
> 
> - **open**：设备"上电初始化"，配置所有必要资源
> - **read**：设备"数据交互"，向用户提供按键状态
> - **release**：设备"关机清理"，释放所有占用资源

### 3.3 驱动初始化和清理

**驱动初始化函数**：

```c
static int __init button_driver_init(void)
{
    int error = -1;

    // 动态分配设备编号
    error = alloc_chrdev_region(&button_devno, 0, DEV_CNT, DEV_NAME);
    if (error < 0) {
        printk("fail to alloc button_devno\n");
        goto alloc_err;
    }

    // 关联字符设备结构体与文件操作结构体
    button_chr_dev.owner = THIS_MODULE;
    cdev_init(&button_chr_dev, &button_chr_dev_fops);

    // 添加设备至系统
    error = cdev_add(&button_chr_dev, button_devno, DEV_CNT);
    if (error < 0) {
        printk("fail to add cdev\n");
        goto add_err;
    }

    // 创建设备类和设备节点
    class_button = class_create(THIS_MODULE, DEV_NAME);
    device_button = device_create(class_button, NULL, button_devno, NULL, DEV_NAME);

    return 0;

add_err:
    unregister_chrdev_region(button_devno, DEV_CNT);
    
alloc_err:
    return -1;
}
```

**驱动清理函数**：
```c
static void __exit button_driver_exit(void)
{
    pr_info("button_driver_exit\n");
    
    // 删除设备和类
    device_destroy(class_button, button_devno);
    class_destroy(class_button);
    cdev_del(&button_chr_dev);
    unregister_chrdev_region(button_devno, DEV_CNT);
}

module_init(button_driver_init);
module_exit(button_driver_exit);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("embedfire lubancat, interrupt tasklet");
```

### 3.4 编译配置

**Makefile文件**：

```makefile
export ARCH=arm64                                  # 设置平台架构
export CROSS_COMPILE=aarch64-linux-gnu-           # 交叉编译器前缀
obj-m := button.o
KDIR :=/home/lubancat/LubanCat_SDK/kernel         # 内核目录路径

PWD ?= $(shell pwd)
all:
    make -C $(KDIR) M=$(PWD) modules               # 编译内核模块
    $(CROSS_COMPILE)gcc -o test_app test_app.c     # 编译测试应用
clean:
    make -C $(KDIR) M=$(PWD) clean                 # 清理编译文件
```

> [!warning]+ 警告或注意
> 
> 确保KDIR路径指向正确的内核源码目录，错误的路径会导致编译失败。

## 4 用户空间测试程序

### 4.1 测试应用程序

用户空间程序用于测试我们的驱动功能：

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int error = -20;
    int button_status = 0;

    // 打开设备文件
    int fd = open("/dev/button", O_RDWR);
    if (fd < 0) {
        printf("open file : /dev/button error!\n");
        return -1;
    }

    printf("wait button down... \n");
    do {
        // 读取按键状态
        error = read(fd, &button_status, sizeof(button_status));
        if (error < 0) {
            printf("read file error! \n");
        }
        usleep(1000 * 100); // 延时100毫秒
    } while (0 == button_status);
    
    printf("button Down !\n");

    // 关闭文件
    error = close(fd);
    if (error < 0) {
        printf("close file error! \n");
    }
    
    return 0;
}
```

**程序流程说明**：

1. **打开设备**：连接到我们的按键驱动
2. **轮询检测**：不断检查按键状态
3. **状态反馈**：检测到按键按下时输出提示
4. **资源清理**：关闭设备文件

> [!example]+ 示例
> 
> 这个测试程序就像一个"门铃监听器"，不断检查是否有人按门铃，一旦检测到按铃就立即提醒用户。

### 4.2 实验流程总结

**完整的实验执行流程**：

1. **编译驱动和应用**：
    
    ```bash
    make clean
    make
    ```
    
2. **加载驱动模块**：
    
    ```bash
    sudo insmod button.ko
    ```
    
3. **运行测试程序**：
    
    ```bash
    ./test_app
    ```
    
4. **按下按键触发中断**
    
5. **观察内核日志**：
    
    ```bash
    dmesg | tail
    ```
    
6. **卸载驱动**：
    
    ```bash
    sudo rmmod button
    ```
    

> [!note]+ 重要笔记
> 
> **预期实验现象**：
> 
> - 按键按下时，中断处理函数立即响应
> - tasklet在后台执行耗时处理，输出计数信息
> - 用户程序检测到按键事件并提示
> - 整个过程展示了上半部和下半部的协作

## 5 总结

通过这个完整的tasklet实验，我们深入掌握了Linux中断处理的实践应用：

> [!abstract]+ 摘要或总结
> 
> **实验收获总结**：
> 
> - 掌握了设备树配置中断和GPIO信息的方法
> - 学会了在驱动中解析设备树获取硬件配置
> - 理解了字符设备驱动的完整开发流程
> - 实践了中断上半部和tasklet下半部的协作机制
> - 体验了从硬件配置到用户应用的完整开发链条

这个实验展示了Linux驱动开发的完整流程，从硬件连接、设备树配置、驱动编写到用户测试，每个环节都至关重要。tasklet机制在其中发挥了关键作用，确保中断处理的实时性和系统的稳定性。

> [!question]+ 常见问题
> 
> **Q**: 如果按键按下后没有中断触发，应该如何排查？ 
> **A**: 首先检查硬件连接是否正确，然后确认设备树配置中的GPIO引脚和中断类型是否与硬件匹配，最后可以使用`cat /sys/kernel/debug/gpio`命令查看GPIO状态进行调试。
