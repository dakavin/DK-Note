---
文章标题: "[[9_Linux设备树_LED灯实验]]"
文章作者: Dakkk
文章概要: 文章通过一个基于Linux设备树的LED灯驱动实验，详细讲解了设备树节点定义、平台驱动编写、`probe`函数中寄存器映射与LED初始化，以及字符设备接口实现。它展示了嵌入式Linux中设备树与平台驱动协同控制硬件的完整流程，是理解Linux驱动开发的关键实践。
tags:
  - Linux设备树
  - LED驱动
  - 平台驱动
  - 字符设备
  - GPIO控制
  - 内核编程
  - 嵌入式Linux
  - 硬件驱动
相关文章:
  - "[[7_平台设备驱动]]"
  - "[[1_快速开始]]"
  - "[[8_Linux设备树]]"
  - "[[14_数码管]]"
  - "[[7-2_继电器]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/9_Linux设备树_LED灯实验.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 16:55:39
修改时间: 2025-06-10 15:19:51
---

## 1 前言

通过前面章节的学习，我们已经掌握了编写简单设备树节点的方法，也学会了使用of函数从设备树中获取节点资源。这一节我们将通过一个实际的LED灯驱动程序来加深对设备树的理解和应用

**实验说明**
本实验使用Lubancat_RK开发板上的LED灯进行演示。我们用的是板子上的系统灯，当然你也可以自己接一个LED到GPIO引脚上。以lubancat2为例，具体的硬件操作可以参考[5_字符设备驱动—点亮LED灯实验](5_字符设备驱动—点亮LED灯实验.md)章节

## 2 编程思路

整体实验分为一下几个步骤：
1. **添加LED灯的设备树节点**：告诉系统我们有个LED设备
2. **编写平台设备驱动框架**：搭建驱动的基本
3. **实现probe函数**：驱动和设备树匹配时自动执行的初始化函数
4. **实现字符设备操作函数**：主要是write操作，用来控制LED
5. **编写测试应用程序**：用户空间的程序，用来测试驱动

## 3 代码详解

### 3.1 添加设备树节点

修改文件夹的位置：`arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dtsi`

**Lubancat2板卡LED灯（CPIO0_C7）的设备树**
```dts
	/* 添加led_test节点 */
	led_test {
		#address-cells = <1>;
		#size-cells = <1>;
		compatible = "dk,led_test";

		//例程是控制LED GPIO0_C7
		//数据寄存器（高16位的地址）
		led@0xfdd60004{
			//数据寄存器和数据方向寄存器的地址和大小
			reg = <0xfdd60004 0x00000004>,    // 数据寄存器
				<0xfdd6000C 0x00000004>;    // 方向寄存器
			status = "okay";
		};
	};
```
- `compatible = "fire,led_test"`：这是驱动和设备树匹配的关键，驱动会寻找这个标识
- `reg`属性：包含了控制LED需要的寄存器地址
- `status = "okay"`：表示这个设备节点是可用的

> [!warning]+ 如果找不到DR和DDR
> - 解决：在父节点添加ranges属性
> - 原因：父节点定义了address和size时，必须提供ranges，用来告诉内核“子节点的地址就是物理地址，不需要转换”


### 3.2 编写驱动程序

基于设备树的驱动和平台总线驱动很相似，主要区别是我们不需要单独的编写平台设备，设备树节点就充当了平台设备的角色

#### 3.2.1 驱动入口函数

```c
// 驱动初始化函数
static init __init led_platform_driver_init(void){
	int DriverState;
	DriverState = platform_driver_register(&led_platform_driver);
	printk(KERN_EMERG "\tDriverState is %d\n",DriverState);
	return 0;
}
```
- 入口函数很简单，就是注册一个平台驱动

#### 3.2.2 定义平台驱动结构体

```c
static const struct of_device_id led_ids[] = {
	{.compatible = "dk,led_test"},
	{}
};

//定义平台驱动结构体
struct platform_driver led_platform_driver = {
	.probe = led_probe,
	.driver = {
		.name = "leds-platform",
		.owner = THIS_MODULE,
		.of_match_table = led_ids,
	},
};
```
- `led_ids`是匹配表，告诉系统这个驱动要匹配哪个设备树节点
- `probe = led_probe`：指定匹配承购后执行的函数
- `of_match_table = led_ids`：指定使用那个匹配表

#### 3.2.3 实现probe函数

**这个是最重要的函数，当驱动和设备树节点匹配成功后会自动执行**

```c
//定义 led 资源结构体，保存获取到的节点信息以及转换后的虚拟寄存器地址
struct led_resource{
	struct device_node *device_node;
	void __iomem *va_DR;
	void __iomem *va_DDR;
}

static int led_probe(struct platform_device *pdev){
	int ret = -1;
	unsigned int register_data = 0;

	printk(KERN_EMERG "\t match successed \n");

	//获取led_test的设备树节点
	led_test_device_node = of_find_node_by_path("/led_test");
	if (led_test_device_node == NULL){
        printk(KERN_ERR "\t  get led_test failed!  \n");
        return -1;
    }
	
	//获取led_test的子节点
	led_res.device_node = of_find_node_by_name(led_test_device_node,"led");
	if (led_res.device_node == NULL){
        printk(KERN_ERR "\n get led_device_node failed ! \n");
        return -1;
    }

	//获取reg属性，并转化为虚拟地址
	led_res.va_DR = of_iomap(led_res.device_node,0);
	if(led_res.va_DR == NULL){
        printk("of_iomap is error \n");
        return -1;
    }
    led_res.va_DDR = of_iomap(led_res.device_node,1);
    if(led_res.va_DDR == NULL){
        printk("of_iomap is error \n");
        return -1;
    }

	// 设置模式寄存器：输出模式
	//GPIO0_C7
	register_data = readl(led_res.va_DDR);  
	//低16位控制GPIO的输出模式
	register_data |= ((unsigned int)0x1 <<(7)); 
	//7+16 ，高16位控制低16位的写使能
	register_data |= ((unsigned int)0x1 <<(7+16)); 
	writel(register_data,led_res.va_DR);

	//设置置位寄存器：默认输出高电平
	register_data = readl(led_res.va_DR);
	//低16位控制GPIO的电平
	register_data |= ((unsigned int)0x1 <<(7)); 
	//7+16 ，高16位控制低16位的写使能
	register_data |= ((unsigned int)0x1 <<(7+16));
	writel(register_data,led_res.va_DDR);

	/* -----------注册字符设备----------- */
	// 1. 申请设备号
	ret = alloc_chrdev_region(&led_devno,0,DEV_CNT,DEV_NAME);
	if (ret < 0){
        printk("fail to alloc led_devno\n");
        goto alloc_err;
    }
    
    // 2.初始化字符设备
    led_chr_dev.owner = THIS_MODULE;
    cdev_init(&led_chr_dev,&led_chr_dev_fops);

	// 3.添加字符设备到散列表（系统）
	ret = cdev_add(&led_chr_dev,led_devno,DEV_CNT);
	if (ret < 0){
        printk("fail to add cdev\n");
        goto add_err;
    }

	// 4. 创建设备节点
	class_led = class_create(THIS_MODULE,DEV_NAME);
	device = device_create(class_led,NULL,led_devno,NULL,DEV_NAME);

	return 0

add_err：
	unregister_chrdev_region(led_devno,DEV_CNT);
	printk("\n error! \n");
alloc_err:
	return -1;
}
```

**函数工作流程**
1. **获取设备树节点**：使用of函数
2. **地址映射**：使用of_iomap函数
3. **初始化LED**：配置寄存器的初始状态（输出、高电平）
4. **注册字符设备**：创建字符设备，让用户程序可以访问

#### 3.2.4 实现字符设备操作函数

用户程序通过字符设备文件来控制LED，我们需要实现相应的操作函数。

```c
// open函数
static int led_chr_dev_open(struct inode *inode,struct file *filp){
	printk("\n led_chr_dev_open \n"); 
	return 0;
}

// write函数（操作寄存器的关键）
static ssize_t led_chr_dev_write(struct file *filp,const char __user *buf,size_t cnt,loff_t *offt){
	unsigned int register_data = 0;
	unsigned char write_data;

	int error = copy_from_user(&write_data,buf,cnt);
	if(error < 0)
		return -1;

	//设置led引脚输出电平
	if(wirte_data){
		//低16位置1
		register_data |= ((unsigned int)0x1 << (7));  
		//7+16，开启低16位的写使能 
		register_data |= ((unsigned int)0x1 << (23)); 
		// lubancat2的GPIO0_C7引脚输出高电平，绿灯灭
		writel(register_data, led_res.va_DR); 
	}else{
		//低16位置0
		register_data &= ~((unsigned int)0x1 << (7));  
		//7+16，开启低16位的写使能 
		register_data |= ((unsigned int)0x1 << (23)); 
		// lubancat2的GPIO0_C7引脚输出低电平，绿灯亮
		writel(register_data, led_res.va_DR);
	}

	return 0;
}

//字符设备操作函数
static struct file_operations led_chr_dev_fops = {
	.owner = THIS_MODULE,
    .open = led_chr_dev_open,
    .write = led_chr_dev_write,
}
```
- 用户程序写入数据时，会调用`led_chr_dev_write`函数
- 函数接收用户数据，根据数据值控制LED的亮灭
- 写入1时LED灭，写入0时LED亮

### 3.3 编写测试应用程序

用户空间的测试程序很简单，主要是打开设备文件，写入控制命令
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

int mian(int argc,char *argv[]){
	printf("led test_app\n");

	//判断输入的命令是否合法
	if(argc != 2){
		printf(" command error ! \n");
        printf(" usage : sudo test_app num [num can be 0 or 1]\n");
        return -1;
	}

	//打开设备文件
	int fd = open("/dev/led_test",O_RDWR);
	if(fd < 0){
		printf("open file : %s failed !\n", argv[0]);
        return -1
	}
	//将收到的命令值转化为数字
	unsigned char command = atoi(argv[1]);
	//写入命令
	int error = write(fd,&command,sizeof(command));
	if(error<0){
		printf("write file error! \n");
		close(fd);
	}

	//关闭文件
	error = close(fd);
	if(error < 0)
		printf("close file error!\n");

	return 0;
}
```

**使用方法**
```shell
sudo ./test_app 0  # LED亮
sudo ./test_app 1  # LED灭
```

## 4 编译步骤

#### 4.1.1 编译设备树

在内核目录下编译设备树（推荐，比较快）
```shell
# 先指定配置文件
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

# 然后编译设备树
sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8 dtbs
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f7ffa27b132fa3335fcd92d3519e1acf.png)

#### 4.1.2 编译驱动和应用程序

```shell
#使用了低版本的交叉编译器
#bear用于生成clangd索引时需要的compile_command.json文件
bear -- make CC=aarch64-linux-gnu-gcc-9 -j8
#如果本身就是低版本的交叉编译器
bear --make
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fc0a7c8938ba99d133265752ec1eb833.png)

## 5 程序运行结果

在鲁班猫系列板卡，系统设备树中均默认使能了`LED`的设备功能，需要关闭设备树的leds节点，可以修改leds节点的 `status = "okay";` 为 `status = "disabled";`，然后编译设备树镜像替换，也可以在板卡中直接使用一下命令关闭系统leds驱动对LED的控制
```shell
# 将led的亮度调为0，同时led的触发条件自动变为none
sudo sh -c 'echo 0 > /sys/class/leds/sys_led/brightness'
```

将设备树、驱动程序和应用程序通过NFS或SCP等方式拷贝到开发板中
```shell
scp arch/arm64/boot/dts/rockchip/rk3568-lubancat-2-v2.dtb cat@192.168.31.69:/home/cat/

scp led_test.ko test_app cat@192.168.31.69:/home/cat/
```

替换原来的设备树/boot/dtb/rk3568-lubancat.dtb，并重启开发板
```shell
sudo mv rk3568-lubancat-2-v2.dtb /boot/dtb/rk3568-lubancat-2-v2.dtb
```

重启后在目录/proc/device-tree/下，可以找到led_test，图中控制的是GPIO0_C7引脚，如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ab4299fcd6942414367edcab3a3287eb.png)

加载驱动
```shell
sudo insmod led_test.ko
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9908aa9650cb12f559c54605c5e3fa49.png)

驱动加载成功后直接运行应用程序如下所示
```shell
sudo ./test_app 0
sudo ./test_app 1
```
