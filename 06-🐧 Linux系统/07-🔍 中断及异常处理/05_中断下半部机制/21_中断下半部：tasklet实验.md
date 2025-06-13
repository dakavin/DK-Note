1. 设备树添加
    
2. 驱动解析
    

  

硬件：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a8576e0d71874495cc8d28d4f7d66b72.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8e3964fa8b4c25a8f2819cd1d8ed8ea6.png)


操作第七个管脚触发中断。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b6d5ed8336913a268fd5915d53f5f2d.png)


## 设备树

文件位置：kernel/arch/arm64/boot/dts/rockchip/rk356x-lubancat-rk_series.dts

单独编译内核

```C
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/interrupt-controller/irq.h>

&{/} {//加入到/ 节点
    button_interrupt: button_interrupt {
        status = "okay";
        compatible = "button_interrupt";
        //pinctrl
        pinctrl-names = "default";
        pinctrl-0 = <&button_interrupt_pin>;
        
        // 所属的中断控制器
        interrupt-parent = <&gpio0>;
        // 中断信号源的描述
        interrupts = <RK_PB0 IRQ_TYPE_LEVEL_LOW>;
        
        //GPIO的描述
        buttons-gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_HIGH>;
    };
};

&{/pinctrl} {//加入到/pinctrl 节点
    button_interrupt {
        button_interrupt_pin: button_interrupt_pin {
            rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
    };
};
```

  

## 驱动

cat /sys/kernel/debug/gpio

- 设备树的语法
    
- 中断在设备树中的解析
    
- 字符设备驱动框架
    
- GPIO的申请: gpio_request\gpio_direction_input
    
- 中断的request_irq API 注册
    
- 中断下半部tasklet
    

```C

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

/*------------------字符设备内容----------------------*/
#define DEV_NAME "button"
#define DEV_CNT (1)

static dev_t button_devno;       //定义字符设备的设备号
static struct cdev button_chr_dev; //定义字符设备结构体chr_dev
struct class *class_button;      //保存创建的类
struct device *device_button;        // 保存创建的设备

struct device_node  *button_device_node = NULL;  //定义按键设备节点结构体
unsigned  button_GPIO_number = 0;  //保存button使用的GPIO引脚编号
u32  interrupt_number = 0;         // button 引脚中断编号
atomic_t   button_status = ATOMIC_INIT(0);  //定义整型原子变量，保存按键状态 ，设置初始值为0

struct tasklet_struct button_tasklet;

void button_tasklet_hander(unsigned long data)
{
    int counter = 1;
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d  \n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d  \n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d  \n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d \n", counter++);
    mdelay(200);
    printk(KERN_ERR "button_tasklet_hander counter = %d \n", counter++);
}

static irqreturn_t button_irq_hander(int irq, void *dev_id)
{
    printk(KERN_ERR "button_irq_hander----------inter");
    /*按键状态加一*/
    atomic_inc(&button_status);

    tasklet_schedule(&button_tasklet);

    printk(KERN_ERR "button_irq_hander-----------exit");
    return IRQ_HANDLED;
}

//open
static int button_open(struct inode *inode, struct file *filp)
{
    int error = -1;

    /*获取按键 设备树节点*/
    button_device_node = of_find_node_by_path("/button_interrupt");
    if(NULL == button_device_node)
    {
        printk("of_find_node_by_path error!");
        return -1;
    }

    /*获取按键使用的GPIO*/
    button_GPIO_number = of_get_named_gpio(button_device_node ,"buttons-gpios", 0);
    if(0 == button_GPIO_number)
    {
        printk("of_get_named_gpio error");
        return -1;
    }

    /*申请GPIO  , 记得释放*/
    error = gpio_request(button_GPIO_number, "button_gpio");
    if(error < 0)
    {
        printk("gpio_request error");
        gpio_free(button_GPIO_number);
        return -1;
    }

    error = gpio_direction_input(button_GPIO_number);//设置引脚为输入模式

    /*获取中断号*/
    interrupt_number = irq_of_parse_and_map(button_device_node, 0);
    printk("irq_of_parse_and_map =  %d \n",interrupt_number);

    /*申请中断, 记得释放*/
    error = request_irq(interrupt_number,button_irq_hander,IRQF_TRIGGER_RISING,"button_interrupt",device_button);
    if(error != 0)
    {
        printk("request_irq error");
        free_irq(interrupt_number, device_button);
        return -1;
    }

    /*初始化button_tasklet*/
    tasklet_init(&button_tasklet,button_tasklet_hander,0);

    /*申请之后已经开启了，切记不要再次打开，否则运行时报错*/
    // // enable_irq(interrupt_number);

    return 0;

}

static ssize_t button_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
    int error = -1;
    int button_countervc = 0;

    /*读取按键状态值*/
    button_countervc = atomic_read(&button_status);

    /*结果拷贝到用户空间*/
    error = copy_to_user(buf, &button_countervc, sizeof(button_countervc));
    if(error < 0)
    {
        printk("copy_to_user error");
        return -1;
    }

    /*清零按键状态值*/
    atomic_set(&button_status,0);
    return 0;
}

/*字符设备操作函数集，.release函数实现*/
static int button_release(struct inode *inode, struct file *filp)
{
    /*释放申请的引脚,和中断*/
    free_irq(interrupt_number, device_button);
    gpio_free(button_GPIO_number);
    return 0;
}

/*字符设备操作函数集*/
static struct file_operations button_chr_dev_fops = {
    .owner = THIS_MODULE,
    .open = button_open,
    .read = button_read,
    .release = button_release
};

/*
*驱动初始化函数
*/
static int __init button_driver_init(void)
{
    int error = -1;

    /*采用动态分配的方式，获取设备编号，次设备号为0，*/
    error = alloc_chrdev_region(&button_devno, 0, DEV_CNT, DEV_NAME);
    if (error < 0)
    {
        printk("fail to alloc button_devno\n");
        goto alloc_err;
    }

    /*关联字符设备结构体cdev与文件操作结构体file_operations*/
    button_chr_dev.owner = THIS_MODULE;
    cdev_init(&button_chr_dev, &button_chr_dev_fops);

    /*添加设备至cdev_map散列表中*/ 
    error = cdev_add(&button_chr_dev, button_devno, DEV_CNT);
    if (error < 0) 
    {
        printk("fail to add cdev\n");
        goto add_err;
    }

    class_button = class_create(THIS_MODULE, DEV_NAME);                         //创建类
    device_button = device_create(class_button, NULL, button_devno, NULL, DEV_NAME);//创建设备 DEV_NAME 指定设备名，

    return 0;

add_err:
    unregister_chrdev_region(button_devno, DEV_CNT);    // 添加设备失败时，需要注销设备号
    printk("\n error! \n");
    
alloc_err:
    return -1;
}

/*
*驱动注销函数
*/
static void __exit button_driver_exit(void)
{
    pr_info("button_driver_exit\n");
    /*删除设备*/
    device_destroy(class_button, button_devno);        //清除设备
    class_destroy(class_button);                       //清除类
    cdev_del(&button_chr_dev);                         //清除设备号
    unregister_chrdev_region(button_devno, DEV_CNT);   //取消注册字符设备
}

module_init(button_driver_init);
module_exit(button_driver_exit);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("embedfire lubuncat , interrupt tasklet");
```

Makefile

```C
export ARCH=arm64#设置平台架构
export CROSS_COMPILE=aarch64-linux-gnu-#交叉编译器前缀
obj-m := button.o
KDIR :=/home/lubancat/LubanCat_SDK/kernel    #这里是你的内核目录                                                                                                                            
PWD ?= $(shell pwd)
all:
    make -C $(KDIR) M=$(PWD) modules    #make操作
    $(CROSS_COMPILE)gcc -o $(out) test_app.c
clean:
    make -C $(KDIR) M=$(PWD) clean    #make clean操作
```

  

## 应用

```C
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int error = -20;
    int button_status = 0;

    /*打开文件*/
    int fd = open("/dev/button", O_RDWR);
    if (fd < 0)
    {
        printf("open file : /dev/button error!\n");
        return -1;
    }

    printf("wait button down... \n");
    do
    {
        /*读取按键状态*/
        error = read(fd, &button_status, sizeof(button_status));
        if (error < 0)
        {
            printf("read file error! \n");
        }
        usleep(1000 * 100); //延时100毫秒
    } while (0 == button_status);
    printf("button Down !\n");

    /*关闭文件*/
    error = close(fd);
    if (error < 0)
    {
        printf("close file error! \n");
    }
    return 0;
}
```