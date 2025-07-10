
## 1 引言

在前两章中，我们学习了GPIO的基本概念和编程接口。但在实际开发中，仅仅会写代码是不够的。当LED不亮、按键没反应时，你**需要有一套系统的调试方法来定位问题**。

调试就像医生看病，需要先做检查（查看GPIO状态），再对症下药（修改配置）。Linux为我们提供了丰富的调试工具，从简单的命令行操作到底层的寄存器访问，每种方法都有其适用场景。

本章将介绍GPIO调试的各种方法，并探讨一些进阶话题，如GPIO中断处理、直接寄存器访问、GPIO与Pinctrl的交互等。这些内容将帮助你成为GPIO调试的高手，在遇到问题时能够快速定位和解决。

## 2 通过sysfs调试GPIO

### 2.1 sysfs GPIO接口简介

sysfs是Linux内核提供的一个虚拟文件系统，挂载在`/sys`目录下。GPIO子系统通过sysfs暴露了一套简单易用的接口，让我们可以在用户空间直接控制GPIO。

这就像给GPIO提供了一个"控制面板"，你可以通过读写文件的方式来操作GPIO，而不需要编写内核驱动。这在调试阶段特别有用，可以快速验证硬件连接是否正确。

首先，让我们查看系统中的GPIO控制器：
```shell
cat@lubancat:/sys/class/gpio$ ls
export  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
```
- `export`：用于将指定编号的 GPIO 引脚导出，只写文件，不能读取
	- 注意：如果对应的GPIO已经导出或者内核在使用，边无法导出
- `unexport`：将导出的 GPIO 引脚删除，只写文件，不可读

每个`gpiochipX`代表一个GPIO控制器，X是该控制器的基地址。我们可以查看控制器的详细信息：
```shell
cat@lubancat:/sys/class/gpio/gpiochip0$ ls
base  device  label  ngpio  power  subsystem  uevent

cat@lubancat:/sys/class/gpio/gpiochip0$ cat base
0      # 基地址为0

cat@lubancat:/sys/class/gpio/gpiochip0$ cat ngpio  
32     # 该控制器管理32个GPIO

cat@lubancat:/sys/class/gpio/gpiochip0$ cat label
gpio0  # 控制器标签
```

> [!tip]+ 实用技巧 
> 在LubanCat开发板上，GPIO引脚索引的计算公式是：`pins = 32 * bank + 8 * group + x`。例如GPIO0_A6的索引是：`32 * 0 + 8 * 0 + 6 = 6`。

### 2.2 导出和控制GPIO

要通过sysfs控制GPIO，需要先将其"导出"。这个过程就像在控制面板上为某个GPIO创建一个控制界面。

**步骤1：导出GPIO**
```shell
# 导出GPIO6（对应GPIO0_A6）
cat@lubancat:/sys/class/gpio$ echo 6 | sudo tee export > /dev/null
# echo 6：标准输出6
# |：将标准输出连接到后面的标准输入
# sudo tee export：读取标准输入的数据，将数据写入到export文件，同时输出到tee的标准输出（屏幕显示）
# > /dev/null：将tee的标准输入重定向到空设备（取消屏幕显示）

# 查看结果，会创建一个gpio6目录
cat@lubancat:/sys/class/gpio$ ls
export  gpio6  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
```

**步骤2：查看GPIO属性**
```shell
cat@lubancat:/sys/class/gpio$ cd gpio6/ && ls
active_low  device  direction  edge  power  subsystem  uevent  value
```
这些文件的含义：
- `direction`：GPIO方向（in/out），默认状态是in
- `value`：GPIO电平值（0/1），默认状态是0
- `edge`：中断触发方式
	- none：非中断引脚
	- rising：上升沿触发
	- falling：下降沿触发
	- both：边沿触发
- `active_low`：是否低电平有效，默认状态是0
	- active_low = 0时，表示低电平的value是0
	- active_low = 1时，表示低电平的value是1

**步骤3：配置和控制GPIO**
```shell
# 设置为输出模式
cat@lubancat:/sys/class/gpio/gpio6$ echo out | sudo tee direction > /dev/null
cat@lubancat:/sys/class/gpio/gpio6$ cat direction
out

# 输出高电平
cat@lubancat:/sys/class/gpio/gpio6$ echo 1 | sudo tee value > /dev/null

# 输出低电平
cat@lubancat:/sys/class/gpio/gpio6$ echo 0 | sudo tee value > /dev/null

# 设置为输入模式
cat@lubancat:/sys/class/gpio/gpio6$ echo in | sudo tee direction > /dev/null

# 读取输入值
cat@lubancat:/sys/class/gpio/gpio6$ cat value
0
```

**步骤4：取消导出**
```shell
# 使用完后，取消导出释放GPIO
cat@lubancat:/sys/class/gpio$ echo 6 | sudo tee unexport > /dev/null
cat@lubancat:/sys/class/gpio$ ls
export  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
```

### 2.3 内核配置要求

要使用sysfs GPIO接口，需要在内核配置中启用相应选项：

```txt
Device Drivers --->
    -*- GPIO Support --->
        [*] /sys/class/gpio/... (sysfs interface)
```

## 3 Linux应用程序控制GPIO

在了解了sysfs接口的原理后，我们可以**编写应用程序来自动化GPIO控制**。这在需要精确时序控制或复杂逻辑时特别有用。

### 3.1 GPIO输出控制程序

下面是一个完整的GPIO控制程序，可以**通过命令行参数控制GPIO**：
```c fold
/* gpioctrl.c - GPIO控制程序 */
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

static int fd;                   // 文件描述符
static int ret;                  // 返回值
static char gpio_path[100];      // GPIO路径
static int len;                  // 字符串长度

/**
 * gpio_export - 导出GPIO引脚
 * @argv: GPIO编号字符串
 * 
 * 向/sys/class/gpio/export写入GPIO编号，创建GPIO控制接口
 */
int gpio_export(char *argv)
{
    fd = open("/sys/class/gpio/export", O_WRONLY);
    if (fd < 0) {
        printf("open /sys/class/gpio/export error\n");
        return -1;
    }
    
    len = strlen(argv);
    ret = write(fd, argv, len);  // 相当于命令：echo argv > export
    if (ret < 0) {
        printf("write /sys/class/gpio/export error\n");
        close(fd);
        return -2;
    }
    
    close(fd);
    return 0;
}

/**
 * gpio_unexport - 取消导出GPIO引脚
 * @argv: GPIO编号字符串
 */
int gpio_unexport(char *argv)
{
    fd = open("/sys/class/gpio/unexport", O_WRONLY);
    if (fd < 0) {
        printf("open /sys/class/gpio/unexport error\n");
        return -1;
    }
    
    len = strlen(argv);
    ret = write(fd, argv, len);
    if (ret < 0) {
        printf("write /sys/class/gpio/unexport error\n");
        close(fd);
        return -2;
    }
    
    close(fd);
    return 0;
}

/**
 * gpio_ctrl - 控制GPIO属性
 * @arg: 属性名（如"direction"、"value"）
 * @val: 属性值（如"out"、"1"）
 */
int gpio_ctrl(char *arg, char *val)
{
    char file_path[100];
    
    /* 构建属性文件路径 */
    sprintf(file_path, "%s/%s", gpio_path, arg);
    
    fd = open(file_path, O_WRONLY);
    if (fd < 0) {
        printf("open %s error\n", file_path);
        return -1;
    }
    
    len = strlen(val);
    ret = write(fd, val, len);
    if (ret < 0) {
        printf("write %s error\n", file_path);
        close(fd);
        return -2;
    }
    
    close(fd);
    return 0;
}

int main(int argc, char *argv[])
{
    if (argc != 3) {
        printf("Usage: %s <gpio_num> <value>\n", argv[0]);
        printf("Example: %s 15 1  # Set GPIO15 to high\n", argv[0]);
        return -1;
    }
    
    /* 构建GPIO路径 */
    sprintf(gpio_path, "/sys/class/gpio/gpio%s", argv[1]);
    
    /* 检查GPIO是否已导出 */
    if (access(gpio_path, F_OK)) {
        /* 未导出则导出GPIO */
        gpio_export(argv[1]);
    }
    
    /* 配置为输出模式 */
    gpio_ctrl("direction", "out");
    
    /* 设置输出电平 */
    gpio_ctrl("value", argv[2]);
    
    /* 取消导出 */
    gpio_unexport(argv[1]);
    
    return 0;
}
```

编译和使用：
```shell
# 编译程序
gcc -o gpioctrl gpioctrl.c

# 使用示例
./gpioctrl 8 1    # 设置GPIO8输出高电平（LED亮）
./gpioctrl 15 0   # 设置GPIO15输出低电平（LED灭）
```

### 3.2 GPIO输入检测程序

下面的程序演示如何**读取GPIO输入状态**：
```c fold
/* gpio_input.c - GPIO输入检测程序 */
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

static int fd;
static int ret;
static char gpio_path[100];
static int len;
static char file_path[100];
static char buf[2];

/* gpio_export和gpio_unexport函数同上，省略 */

/**
 * gpio_read_value - 读取GPIO电平值
 * @arg: 属性名（通常是"value"）
 * 
 * 返回值：1-高电平，0-低电平，-1-错误
 */
int gpio_read_value(char *arg)
{
    sprintf(file_path, "%s/%s", gpio_path, arg);
    
    fd = open(file_path, O_RDONLY);
    if (fd < 0) {
        printf("open %s error\n", file_path);
        return -1;
    }
    
    ret = read(fd, buf, 1);
    close(fd);
    
    if (ret < 0) {
        printf("read %s error\n", file_path);
        return -1;
    }
    
    /* 判断读取到的值 */
    if (buf[0] == '1') {
        printf("GPIO value is HIGH\n");
        return 1;
    } else if (buf[0] == '0') {
        printf("GPIO value is LOW\n");
        return 0;
    }
    
    return -1;
}

int main(int argc, char *argv[])
{
    int value;
    
    if (argc != 2) {
        printf("Usage: %s <gpio_num>\n", argv[0]);
        return -1;
    }
    
    sprintf(gpio_path, "/sys/class/gpio/gpio%s", argv[1]);
    
    /* 导出GPIO */
    if (access(gpio_path, F_OK)) {
        gpio_export(argv[1]);
    }
    
    /* 配置为输入模式 */
    gpio_ctrl("direction", "in");
    
    /* 读取GPIO值 */
    value = gpio_read_value("value");
    printf("GPIO %s value is %d\n", argv[1], value);
    
    /* 取消导出 */
    gpio_unexport(argv[1]);
    
    return 0;
}
```

### 3.3 GPIO中断检测程序

GPIO的一个重要功能是中断检测。下面的程序演示如何**使用poll()函数等待GPIO中断**：
```c fold
/* gpio_interrupt.c - GPIO中断检测程序 */
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <poll.h>

static int fd;
static int ret;
static char gpio_path[100];
static int len;
static char file_path[100];
static char buf[2];
static struct pollfd fds[1];  // poll结构体数组

/* gpio_export、gpio_unexport、gpio_ctrl函数同上，省略 */

/**
 * gpio_interrupt - 监听GPIO中断事件
 * @arg: 属性名（通常是"value"）
 * 
 * 使用poll()等待GPIO状态变化
 */
int gpio_interrupt(char *arg)
{
    sprintf(file_path, "%s/%s", gpio_path, arg);
    
    fd = open(file_path, O_RDONLY);
    if (fd < 0) {
        printf("open %s error\n", file_path);
        return -1;
    }
    
    /* 配置poll结构体 */
    memset((void*)fds, 0, sizeof(fds));
    fds[0].fd = fd;
    fds[0].events = POLLPRI;  // 监听优先级事件（中断）
    
    /* 先读取一次，清除中断标志 */
    read(fd, buf, 2);
    
    printf("Waiting for GPIO interrupt...\n");
    
    /* 阻塞等待中断发生 */
    ret = poll(fds, 1, -1);  // -1表示永久等待
    if (ret <= 0) {
        printf("poll error\n");
        close(fd);
        return -1;
    }
    
    /* 检查是否是我们期待的事件 */
    if (fds[0].revents & POLLPRI) {
        /* 重新定位到文件开头 */
        lseek(fd, 0, SEEK_SET);
        
        /* 读取新的值 */
        read(fd, buf, 2);
        buf[1] = '\0';
        printf("GPIO interrupt detected! New value is %s\n", buf);
    }
    
    close(fd);
    return 0;
}

int main(int argc, char *argv[])
{
    if (argc != 2) {
        printf("Usage: %s <gpio_num>\n", argv[0]);
        printf("Example: %s 42  # Monitor GPIO42 interrupt\n", argv[0]);
        return -1;
    }
    
    sprintf(gpio_path, "/sys/class/gpio/gpio%s", argv[1]);
    
    /* 导出GPIO */
    if (access(gpio_path, F_OK)) {
        gpio_export(argv[1]);
    }
    
    /* 配置为输入模式 */
    gpio_ctrl("direction", "in");
    
    /* 配置中断触发方式 */
    gpio_ctrl("edge", "both");  // 双边沿触发
    
    /* 等待中断 */
    gpio_interrupt("value");
    
    /* 取消导出 */
    gpio_unexport(argv[1]);
    
    return 0;
}
```

使用示例：
```shell
# 编译程序
gcc -o gpio_interrupt gpio_interrupt.c

# 后台运行，监听GPIO42（假设连接了按键）
./gpio_interrupt 42 &

# 当按下/释放按键时，程序会打印中断信息
```

> [!note]+ 背景知识 中断触发方式（edge）有四种设置：
> 
> - `none`：不产生中断
> - `rising`：上升沿触发（低到高）
> - `falling`：下降沿触发（高到低）
> - `both`：双边沿触发（任何电平变化）

## 4 使用debugfs深入调试

### 4.1 debugfs简介

debugfs是Linux内核提供的调试文件系统，包含了比sysfs更详细的调试信息。它通常挂载在`/sys/kernel/debug`目录下。

如果该目录为空，需要先配置内核并挂载debugfs：
```shell
# 检查是否已挂载
mount | grep debugfs

# 如果未挂载，执行挂载
sudo mount -t debugfs none /sys/kernel/debug/
```

内核配置选项：
```txt
Kernel hacking --->
    [*] Debug Filesystem
```

### 4.2 查看GPIO使用情况

debugfs提供了全局的GPIO使用视图：
```shell
sudo cat /sys/kernel/debug/gpio
```

输出示例：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/51145a0426f0b06096726955f83fcba2.png)

这个输出显示了：
- 每个GPIO控制器的信息
- 已被占用的GPIO及其使用者
- GPIO的当前状态（输入/输出、电平）

当尝试使用已被占用的GPIO时：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ff7831e8e9ed207ec3df007547f2362c.png)

### 4.3 Pinctrl调试信息

参考：[5_调试验证与完整实践](../1_Pinctrl子系统/5_调试验证与完整实践.md)

### 4.4 实例：查看I2C引脚配置

让我们通过一个实例来理解如何使用这些调试信息。假设我们要查看I2C3的引脚配置：

首先查看设备树中的定义：
```d
/* arch/arm64/boot/dts/rockchip/rk3568.dtsi */
i2c3: i2c@fe5c0000 {
    compatible = "rockchip,rk3399-i2c";
    reg = <0x0 0xfe5c0000 0x0 0x1000>;
    clocks = <&cru CLK_I2C3>, <&cru PCLK_I2C3>;
    clock-names = "i2c", "pclk";
    interrupts = <GIC_SPI 49 IRQ_TYPE_LEVEL_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&i2c3m0_xfer>;
    #address-cells = <1>;
    #size-cells = <0>;
    status = "disabled";
};
```

然后查看pinctrl定义：
```dts
/* arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi */
i2c3 {
    i2c3m0_xfer: i2c3m0-xfer {
        rockchip,pins =
            /* i2c3_sclm0 */
            <1 RK_PA1 1 &pcfg_pull_none_smt>,
            /* i2c3_sdam0 */
            <1 RK_PA0 1 &pcfg_pull_none_smt>;
    };
    
    i2c3m1_xfer: i2c3m1-xfer {
        rockchip,pins =
            /* i2c3_sclm1 */
            <3 RK_PB5 4 &pcfg_pull_none_smt>,
            /* i2c3_sdam1 */
            <3 RK_PB6 4 &pcfg_pull_none_smt>;
    };
};
```

最后通过debugfs确认实际配置：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/26de8ba924cbba041ee09c80f29db229.png)

## 5 直接访问GPIO寄存器（了解）

在某些特殊场景下，我们可能需要直接访问GPIO寄存器。这种方法绕过了内核的GPIO子系统，应该谨慎使用。

### 5.1 使用IO命令操作寄存器

`io`命令是一个用于读写IO端口和内存映射寄存器的工具，在内核调试阶段特别有用。

#### 5.1.1 IO命令语法

```shell
io [选项] [地址] [操作] [数据]
```

选项说明：
- `-b`：以**字节为单位**进行I/O操作（默认为字）
- `-w`：以**字为单位**进行I/O操作
- `-l`：以**双字为单位**进行I/O操作

#### 5.1.2 LED引脚寄存器查找

要使用IO命令控制GPIO，**首先需要找到对应的寄存器地址**。以控制LED为例，假设LED连接在GPIO0_B7，我们需要：
1. 查找GPIO0控制器的基地址
2. 计算具体引脚在寄存器中的位置
3. 确定需要操作的寄存器（方向寄存器、数据寄存器等）

对于RK3568，GPIO0的基地址是0xFDD60000，主要寄存器偏移：
- GPIO_SWPORT_DR（数据寄存器）：0x0000
- GPIO_SWPORT_DDR（方向寄存器）：0x0004

#### 5.1.3 IO命令点灯测试

```shell
# 设置GPIO0_B7为输出（在方向寄存器中将第15位设为1）
# 先读取当前值
io -4 -r 0xfdd60004
# 假设读到0x00000000，设置第15位为1
io -4 -w 0xfdd60004 0x00008000

# 点亮LED（在数据寄存器中将第15位设为1）
# 先读取当前值
io -4 -r 0xfdd60000
# 设置第15位为1
io -4 -w 0xfdd60000 0x00008000

# 熄灭LED（在数据寄存器中将第15位设为0）
io -4 -w 0xfdd60000 0x00000000
```

> [!warning]+ 重要提醒 
> 使用IO命令直接操作寄存器时要特别小心，错误的操作可能导致系统崩溃。建议先读取寄存器的当前值，只修改需要改变的位。

### 5.2 通过/dev/mem设备控制GPIO

#### 5.2.1 Linux系统用户态访问内核态方式

在Linux系统中，用户空间程序通常不能直接访问物理内存地址，这是出于安全和稳定性的考虑。但Linux提供了几种机制让用户空间程序在特定条件下访问物理内存：

1. **/dev/mem设备**：提供对物理内存的原始访问
2. **/dev/kmem设备**：提供对内核虚拟内存的访问（新内核已移除）
3. **mmap系统调用**：将物理内存映射到用户空间

#### 5.2.2 /dev/mem设备介绍

`/dev/mem`是一个字符设备文件，主设备号为1，次设备号为1。它提供了对系统物理内存的访问能力。通过这个设备，我们可以：
- 读写物理内存中的任意地址
- 访问内存映射的硬件寄存器
- 调试硬件驱动程序

使用`/dev/mem`需要root权限，因为不当的使用可能破坏系统稳定性。

#### 5.2.3 /dev/mem设备的使用方法

使用`/dev/mem`的基本步骤：
1. 打开`/dev/mem`设备文件
2. 使用`mmap()`将需要访问的物理地址映射到用户空间
3. 通过映射后的虚拟地址访问硬件寄存器
4. 使用完毕后`munmap()`解除映射
5. 关闭设备文件

#### 5.2.4 mmap函数详解

`mmap()`函数用于建立内存映射：
```c
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
```

参数说明：
- `addr`：建议的映射地址，通常传NULL让系统自动分配
- `length`：映射的长度
- `prot`：保护标志（PROT_READ | PROT_WRITE）
- `flags`：映射类型（MAP_SHARED）
- `fd`：文件描述符（/dev/mem的fd）
- `offset`：物理地址（必须是页对齐的）

#### 5.2.5 LED灯实验

下面是一个完整的通过`/dev/mem`控制LED的实验程序：
```c fold
/* led_mem.c - 通过/dev/mem直接控制LED */
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <string.h>

/* RK3568 GPIO0寄存器定义 */
#define GPIO0_BASE_ADDR     0xfdd60000
#define GPIO_SWPORT_DR      0x0000  /* 数据寄存器 */
#define GPIO_SWPORT_DDR     0x0004  /* 方向寄存器 */

#define GPIO_PIN_B7         15      /* GPIO0_B7 = 1*8 + 7 = 15 */
#define MAP_SIZE            0x1000  /* 映射大小4KB */

int main(int argc, char *argv[])
{
    int fd;
    void *gpio_base;
    volatile unsigned int *gpio_dr;   /* 数据寄存器指针 */
    volatile unsigned int *gpio_ddr;  /* 方向寄存器指针 */
    int operation;
    
    if (argc != 2) {
        printf("Usage: %s <on|off|blink>\n", argv[0]);
        return -1;
    }
    
    /* 解析命令参数 */
    if (strcmp(argv[1], "on") == 0) {
        operation = 1;
    } else if (strcmp(argv[1], "off") == 0) {
        operation = 0;
    } else if (strcmp(argv[1], "blink") == 0) {
        operation = 2;
    } else {
        printf("Invalid operation\n");
        return -1;
    }
    
    /* 打开/dev/mem设备 */
    fd = open("/dev/mem", O_RDWR | O_SYNC);
    if (fd < 0) {
        perror("Failed to open /dev/mem");
        return -1;
    }
    
    /* 映射GPIO0寄存器空间到用户空间 */
    gpio_base = mmap(NULL,                    /* 由系统分配映射地址 */
                     MAP_SIZE,                /* 映射大小 */
                     PROT_READ | PROT_WRITE,  /* 可读可写 */
                     MAP_SHARED,              /* 共享映射 */
                     fd,                      /* 文件描述符 */
                     GPIO0_BASE_ADDR);        /* 物理地址 */
    
    if (gpio_base == MAP_FAILED) {
        perror("mmap failed");
        close(fd);
        return -1;
    }
    
    /* 计算寄存器虚拟地址 */
    gpio_dr = (volatile unsigned int *)(gpio_base + GPIO_SWPORT_DR);
    gpio_ddr = (volatile unsigned int *)(gpio_base + GPIO_SWPORT_DDR);
    
    /* 读取并显示当前寄存器值 */
    printf("Before operation:\n");
    printf("  DDR = 0x%08x\n", *gpio_ddr);
    printf("  DR  = 0x%08x\n", *gpio_dr);
    
    /* 设置GPIO0_B7为输出模式 */
    *gpio_ddr |= (1 << GPIO_PIN_B7);
    
    /* 执行操作 */
    switch (operation) {
    case 0:  /* 关闭LED */
        *gpio_dr &= ~(1 << GPIO_PIN_B7);
        printf("LED turned OFF\n");
        break;
        
    case 1:  /* 打开LED */
        *gpio_dr |= (1 << GPIO_PIN_B7);
        printf("LED turned ON\n");
        break;
        
    case 2:  /* LED闪烁 */
        printf("LED blinking... Press Ctrl+C to stop\n");
        while (1) {
            *gpio_dr |= (1 << GPIO_PIN_B7);   /* 点亮 */
            usleep(500000);                    /* 延时500ms */
            *gpio_dr &= ~(1 << GPIO_PIN_B7);  /* 熄灭 */
            usleep(500000);                    /* 延时500ms */
        }
        break;
    }
    
    /* 显示操作后的寄存器值 */
    printf("After operation:\n");
    printf("  DDR = 0x%08x\n", *gpio_ddr);
    printf("  DR  = 0x%08x\n", *gpio_dr);
    
    /* 清理资源 */
    munmap(gpio_base, MAP_SIZE);
    close(fd);
    
    return 0;
}
```

编译和使用：

```shell
# 编译程序
gcc -o led_mem led_mem.c

# 使用示例（需要root权限）
sudo ./led_mem on     # 点亮LED
sudo ./led_mem off    # 熄灭LED
sudo ./led_mem blink  # LED闪烁
```

Makefile示例：

```makefile
# Makefile for LED memory-mapped control
CC = gcc
CFLAGS = -Wall -O2
TARGET = led_mem

all: $(TARGET)

$(TARGET): led_mem.c
	$(CC) $(CFLAGS) -o $(TARGET) led_mem.c

clean:
	rm -f $(TARGET)

install:
	cp $(TARGET) /usr/local/bin/

.PHONY: all clean install
```

> [!note]+ 背景知识 
> 使用`/dev/mem`直接访问硬件寄存器是一种底层的调试方法。在实际产品中，应该通过正规的内核驱动来控制GPIO。这种方法主要用于：
> 
> 1. 驱动开发的前期验证
> 2. 硬件问题的快速定位
> 3. 特殊场景下的性能优化

`/dev/mem`是物理内存的设备文件，通过它可以直接访问物理地址。下面是一个完整的示例程序：
```c fold
/* gpio_mem.c - 通过/dev/mem直接控制GPIO */
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>

/* RK3568 GPIO0寄存器定义 */
#define GPIO0_BASE_ADDR     0xfdd60000
#define GPIO_SWPORT_DR      0x0000  /* 数据寄存器 */
#define GPIO_SWPORT_DDR     0x0004  /* 方向寄存器 */

#define MAP_SIZE            0x1000  /* 映射大小 */

int main(int argc, char *argv[])
{
    int fd;
    void *gpio_base;
    volatile unsigned int *gpio_dr;   /* 数据寄存器指针 */
    volatile unsigned int *gpio_ddr;  /* 方向寄存器指针 */
    unsigned int pin;
    unsigned int value;
    
    if (argc != 3) {
        printf("Usage: %s <pin> <value>\n", argv[0]);
        printf("Example: %s 7 1  # Set GPIO0_A7 to high\n", argv[0]);
        return -1;
    }
    
    pin = atoi(argv[1]);
    value = atoi(argv[2]);
    
    if (pin > 31) {
        printf("Pin number must be 0-31 for GPIO0\n");
        return -1;
    }
    
    /* 打开/dev/mem */
    fd = open("/dev/mem", O_RDWR | O_SYNC);
    if (fd < 0) {
        perror("open /dev/mem failed");
        return -1;
    }
    
    /* 映射GPIO寄存器 */
    gpio_base = mmap(NULL, MAP_SIZE, PROT_READ | PROT_WRITE, 
                     MAP_SHARED, fd, GPIO0_BASE_ADDR);
    if (gpio_base == MAP_FAILED) {
        perror("mmap failed");
        close(fd);
        return -1;
    }
    
    /* 计算寄存器地址 */
    gpio_dr = (volatile unsigned int *)(gpio_base + GPIO_SWPORT_DR);
    gpio_ddr = (volatile unsigned int *)(gpio_base + GPIO_SWPORT_DDR);
    
    /* 读取当前值 */
    printf("Before: DR=0x%08x, DDR=0x%08x\n", *gpio_dr, *gpio_ddr);
    
    /* 设置为输出 */
    *gpio_ddr |= (1 << pin);
    
    /* 设置输出值 */
    if (value) {
        *gpio_dr |= (1 << pin);   /* 置1 */
    } else {
        *gpio_dr &= ~(1 << pin);  /* 清0 */
    }
    
    /* 读取修改后的值 */
    printf("After: DR=0x%08x, DDR=0x%08x\n", *gpio_dr, *gpio_ddr);
    
    /* 清理资源 */
    munmap(gpio_base, MAP_SIZE);
    close(fd);
    
    return 0;
}
```

编译和运行（需要root权限）：

```shell
# 编译
gcc -o gpio_mem gpio_mem.c

# 运行（控制GPIO0_A7）
sudo ./gpio_mem 7 1   # 设置GPIO0_A7输出高电平
sudo ./gpio_mem 7 0   # 设置GPIO0_A7输出低电平
```

> [!warning]+ 重要提醒 
> 直接访问寄存器绕过了内核的资源管理机制，可能导致：
> 
> 1. 与内核驱动冲突
> 2. 破坏系统稳定性
> 3. 代码不可移植
> 
> 仅在特殊调试场景或对性能有极高要求时使用。

## 6 常见问题与解决方案

### 6.1 GPIO申请失败

**问题现象**：
```txt
gpio-15 (sysfs): gpiod_request: status -16
export_store: status -16
```

**原因分析**：
- GPIO已被其他驱动占用
- 设备树中有冲突的配置

**解决方法**：
1. 使用`cat /sys/kernel/debug/gpio`查看GPIO占用情况
2. 检查设备树中是否有多个节点使用同一个GPIO
3. 确认是否有内核驱动已经使用了该GPIO

### 6.2 GPIO电平设置无效

**可能原因**：
1. 引脚复用未正确配置为GPIO功能
2. GPIO方向设置错误
3. 硬件连接问题

**调试步骤**：
```shell
# 1. 检查引脚复用
sudo cat /sys/kernel/debug/pinctrl/*/pinmux-pins | grep <pin_number>

# 2. 检查GPIO状态
sudo cat /sys/kernel/debug/gpio | grep <gpio_number>

# 3. 用万用表测量实际电平
```

### 6.3 中断无法触发

**检查要点**：
1. GPIO是否配置为输入模式
2. 中断触发方式是否正确（edge配置）
3. 是否有上下拉电阻确保稳定的默认电平

**调试代码**：
```c
/* 打印中断相关信息 */
struct gpio_desc *gpio;
int irq;

gpio = gpiod_get(dev, "button", GPIOD_IN);
irq = gpiod_to_irq(gpio);

dev_info(dev, "GPIO chip: %s\n", gpio->gdev->chip->label);
dev_info(dev, "GPIO hwnum: %d\n", gpio->hwnum);
dev_info(dev, "IRQ number: %d\n", irq);
dev_info(dev, "IRQ trigger: 0x%x\n", irq_get_trigger_type(irq));
```

> [!tip]+ 实用技巧 调试GPIO问题时的黄金法则：
> 
> 1. 先用sysfs手动测试，确认硬件正常
> 2. 查看debugfs确认配置正确
> 3. 检查设备树配置是否有冲突
> 4. 使用示波器或逻辑分析仪查看实际波形

## 7 本章小结

通过本章的学习，我们掌握了GPIO调试的各种方法和技巧：

1. **sysfs接口**：最简单的调试方法，适合快速验证
2. **应用层编程**：通过C程序自动化GPIO控制
3. **debugfs调试**：深入了解GPIO和Pinctrl的配置状态
4. **直接寄存器访问**：特殊场景下的终极手段
5. **问题诊断**：系统化的问题定位和解决方法

这些调试技能在实际项目中非常重要。当你遇到"LED不亮"、"按键没反应"这类问题时，不要慌张，按照本章介绍的方法一步步排查，问题总能得到解决。

记住，调试是一门实践的艺术。多动手、多思考，你会发现GPIO虽然简单，但其中蕴含的Linux内核设计思想值得深入学习。这些知识不仅适用于GPIO，对理解其他Linux子系统也大有裨益。