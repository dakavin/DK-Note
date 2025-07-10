用户如果要对外设进行操作， 对应的设备驱动不仅要具备读写的能力， 还需要对硬件进行控制。 以点亮 LED 灯驱动实验为例， 应用程序通过向内核空间写入 1 和 0 从而控制 LED 灯的亮灭， 但是读写操作主要是数据流对数据进行操作， 而一些复杂的控制通常需要非数据操作，这时本章节要学习的 ioctl 函数就闪耀登场了。

## ioctl 基础

**ioctl** 是设备驱动程序中用来控制设备的接口函数， 一个字符设备驱动通常需要实现设备的打开、 关闭、 读取、 写入等功能， 而在一些需要细分的情况下， 就需要扩展新的功能， 通常以增设 **ioctl**()命令的方式来实现。

下面将从应用层和驱动函数两个方面来对 ioctl 函数进行学习。

### 应用层

**函数原型：**

int ioctl(int fd, unsigned int cmd, unsigned long args);头文件：

#include <sys/ioctl.h>

**函数作用：**

用于向设备发送控制和配置命令。

**参数含义：**

- fd ： 是用户程序打开设备时返回的文件描述符
    
- cmd ： 是用户程序对设备的控制命令，
    
- args： 应用程序向驱动程序下发的参数， 如果传递的参数为指针类型， 则可以接收驱动向用户空间传递的数据（在下面的实验中会进行使用）
    

上述三个参数中， 最重要的是**第二个** **cmd** **参数**， 为 unsigned int 类型， 为了高效的使用 cmd 参数传递更多的控制信息， 一个 unsigned int cmd 被拆分为了 4 段， 每一段都有各自的意义， unsigned int cmd 位域拆分如下：

- cmd[31:30]—数据（args） 的传输方向（读写）
    
- cmd[29:16]—数据（args） 的大小
    
- cmd[15:8]—>命令的类型， 可以理解成命令的密钥， 一般为 ASCII 码（0-255 的一个字符， 有部分字符已经被占用， 每个字符的序号段可能部分被占用）
    
- cmd[7:0] —>命令的序号， 是一个 8bits 的数字（序号， 0-255 之间）
    

cmd 参数由 ioctl 合成宏定义得到， 四个合成宏定义如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b195288cf4bded456343361ca3aca13.png)


宏定义参数说明如下所示：

- type： 命令的类型， 一般为一个 ASCII 码值， 一个驱动程序一般使用一个 type
    
- nr： 该命令下序号。 一个驱动有多个命令， 一般他们的 type， 序号不同
    
- size： args 的类型
    

例如可以使用以下代码定义不需要参数、 向驱动程序写参数、 向驱动程序读参数三个宏：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c6369386fdce0b15af0512b9e7783c92.png)


至此， 关于应用程序的 ioctl 相关知识就讲解完成了。

### 驱动函数

应用程序中 ioctl 函数会调用 file_operation 结构体中的 unlocked_ioctl 接口， 接口定义如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/12f78f04684995044b622084174501e8.png)


参数说明如下所示：

- file： 文件描述符。
    
- cmd： 与应用程序的 cmd 参数对应， 在驱动程序中对传递来的 cmd 参数进行判断从而做出不同的动作。
    
- arg： 与应用程序的 arg 参数对应， 从而实现内核空间和用户空间参数的传递。
    

至此， 关于驱动函数中的 ioctl 相关知识就讲解完成了。 在下一小节中将进行 ioctl 驱动传参实验。

## 实验程序


### 驱动

```C fold
#include <linux/module.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/uaccess.h>
#define CMD_TEST0 _IO('L',0)
#define CMD_TEST1 _IOW('L',1,int)
#define CMD_TEST2 _IOR('L',2,int)

struct device_test{

    dev_t dev_num;  //设备号
     int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
};
static struct device_test dev1;

static long cdev_test_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    int val;//定义int类型向应用空间传递的变量val
    switch(cmd){
        case CMD_TEST0:
            printk("this is CMD_TEST0\n");
            break;      
        case CMD_TEST1:
            printk("this is CMD_TEST1\n");
            printk("arg is %ld\n",arg);//打印应用空间传递来的arg参数
            break;
        case CMD_TEST2:
            val = 1;//将要传递的变量val赋值为1
            printk("this is CMD_TEST2\n");
            if(copy_to_user((int *)arg,&val,sizeof(val)) != 0){//通过copy_to_user向用户空间传递数据
                printk("copy_to_user error \n");    
            }
            break;          
    default:
            break;
    }
    return 0;
}
/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .unlocked_ioctl = cdev_test_ioctl,
};
static int __init timer_dev_init(void) //驱动入口函数
{
    /*注册字符设备驱动*/
    int ret;
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
    dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
    dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }

return 0;

err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}

static void __exit timer_dev_exit(void) //驱动出口函数
{
    /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
}
module_init(timer_dev_init);
module_exit(timer_dev_exit);
MODULE_LICENSE("GPL v2");
```

  

### 应用

```C
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>
#include <string.h>

#define CMD_TEST0 _IO('L',0)
#define CMD_TEST1 _IOW('L',1,int)
#define CMD_TEST2 _IOR('L',2,int)

int main(int argc,char *argv[]){

    int fd;//定义int类型的文件描述符fd
    int val;//定义int类型的传递参数val
    fd = open("/dev/test",O_RDWR);//打开test设备节点
    if(fd < 0){
        printf("file open fail\n");
    }
    if(!strcmp(argv[1], "write")){
        ioctl(fd,CMD_TEST1,1);//如果第二个参数为write，向内核空间写入1
    }
    else if(!strcmp(argv[1], "read")){
        ioctl(fd,CMD_TEST2,&val);//如果第二个参数为read，则读取内核空间传递向用户空间传递的值
        printf("val is %d\n",val);

    }
    close(fd);
}
```

## 实验结果

开发板启动之后， 使用以下命令进行驱动模块的加载
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/70942d325a123bf3e597e8a9c776317f.png)


然后使用以下命令通过 ioctl 向内核空间传递 arg 参数， 传递成功如下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b81a1eec48ed4b1d7cfb70cb210a82bd.png)


然后使用以下命令通过 ioctl 读取内核空间向用户空间传递的 val 值， 读取成功如下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/70e9dcaa2dc32fb1f2652f26b538c26b.png)
