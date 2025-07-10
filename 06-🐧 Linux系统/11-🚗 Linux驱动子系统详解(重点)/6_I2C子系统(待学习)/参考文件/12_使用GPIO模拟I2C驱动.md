之前我们使用的都是硬件 I2C，这意 味着无需自己编写相应的 I2C 时序代码。硬件 I2C 依赖于微控制器内部的专用硬件模块来处理 通信时序，从而简化了开发过程，提高了通信效率和可靠性。

## 编写起始和终止信号代码

添加起始信号和终止信号相关的代码，起始信号和终止信号通信时 序图如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4319d59f16d0ef4d89f44ec4b780118e.png)


起始信号为 SDA 线从高电平到低电平的跳变,同时 SCL 线保持高电平，终止信号为为 SDA 线从低电平到高电平的跳变,同时 SCL 线保持高电平。然后根据上述时序图完善起始信号 i2c_start 和终止信号 i2c_stop 相关的代码，编写完成的驱动程序如下图所示：

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/gpio/consumer.h>
#include <linux/gpio.h>
#include <linux/delay.h>
#include <linux/jiffies.h>
// 定义 I2C 总线的时钟线和数据线对应的 GPIO 引脚编号
#define I2C_SCL 11
#define I2C_SDA 12
// 声明两个 GPIO 描述符变量,用于保存 SCL 和 SDA 引脚的描述符
struct gpio_desc *i2c_scl_desc;
struct gpio_desc *i2c_sda_desc;
// I2C 起始条件函数
void i2c_start(void)
{
    // 将 SCL 和 SDA 引脚设置为输出模式,并初始化为高电平
    // 这是 I2C 总线的空闲状
    gpiod_direction_output(i2c_scl_desc, 1);
    gpiod_direction_output(i2c_sda_desc, 1);
    mdelay(1); // 延时 1 毫秒
    // 将 SDA 引脚设置为低电平,保持 SCL 为高电平
    // 这将产生 I2C 总线的起始条件
    gpiod_direction_output(i2c_sda_desc, 0);
    mdelay(1); // 延时 1 毫秒
    // 将 SCL 引脚设置为低电平
    // 起始条件建立完成
    gpiod_direction_output(i2c_scl_desc, 0);
    mdelay(1); // 延时 1 毫秒
}
// I2C 停止条件函数
void i2c_stop(void)
{
    // 将 SCL 和 SDA 引脚设置为低电平
    gpiod_direction_output(i2c_scl_desc, 0);
    gpiod_direction_output(i2c_sda_desc, 0);
    mdelay(1); // 延时 1 毫秒
    // 将 SCL 引脚设置为高电平
    gpiod_direction_output(i2c_scl_desc, 1);
    mdelay(1); // 延时 1 毫秒
    // 将 SDA 引脚设置为高电平
    // 这将产生 I2C 总线的停止条件
    gpiod_direction_output(i2c_sda_desc, 1);
    mdelay(1); // 延时 1 毫秒
}
// 驱动初始化函数
static int ft5x06_driver_init(void)
{
    // 将 GPIO 编号转换为 GPIO 描述符
    i2c_scl_desc = gpio_to_desc(I2C_SCL);
    if (i2c_scl_desc == NULL) {
        printk("gpio_to_desc error for SCL pin\n");
        return -1;
     }
    i2c_sda_desc = gpio_to_desc(I2C_SDA);
    if (i2c_sda_desc == NULL) {
        printk("gpio_to_desc error for SDA pin\n");
        return -1;
    }
    // 将 GPIO 引脚设置为输出模式,并初始化为高电平
    // 这是 I2C 总线的空闲状态
    gpiod_direction_output(i2c_scl_desc, 1);
    gpiod_direction_output(i2c_sda_desc, 1);
    i2c_start();
    i2c_stop();
    return 0;
}
// 驱动退出函数
static void ft5x06_driver_exit(void)
{
    // 释放 GPIO 描述符
    gpiod_put(i2c_scl_desc);
    gpiod_put(i2c_sda_desc);
}
// 注册驱动初始化和退出函数
module_init(ft5x06_driver_init);
module_exit(ft5x06_driver_exit);
MODULE_LICENSE("GPL");
```

## 编写发送和接收应答信号代码

添加发送和接收应答信号相关的代码，关于应答信号相关的具体时序图如下所示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/291c0b39633bba926922e159fc662830.png)


当发送设备在第 9 个时钟脉冲期间释放 SDA 线时,接收设备可以拉低 SDA 线并在此时钟高 电平期间保持稳定低电平,这就定义了应答信号。如果在第 9 个时钟脉冲期间 SDA 线保持高电 平,则定义为非应答信号。

编写完成的驱动程序如下图所示：

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/gpio/consumer.h>
#include <linux/gpio.h>
#include <linux/delay.h>
#include <linux/jiffies.h>

// 定义 I2C 总线的时钟线和数据线对应的 GPIO 引脚编号
#define I2C_SCL 11
#define I2C_SDA 12

// 声明两个 GPIO 描述符变量,用于保存 SCL 和 SDA 引脚的描述符
struct gpio_desc *i2c_scl_desc;
struct gpio_desc *i2c_sda_desc;

// I2C 起始条件函数
void i2c_start(void)
{
    // 将 SCL 和 SDA 引脚设置为输出模式,并初始化为高电平
    // 这是 I2C 总线的空闲状态
    gpiod_direction_output(i2c_scl_desc, 1);
    gpiod_direction_output(i2c_sda_desc, 1);
    mdelay(1); // 延时 1 毫秒
    
    // 将 SDA 引脚设置为低电平,保持 SCL 为高电平
    // 这将产生 I2C 总线的起始条件
    gpiod_direction_output(i2c_sda_desc, 0);
    mdelay(1); // 延时 1 毫秒
    
    // 将 SCL 引脚设置为低电平
    // 起始条件建立完成
    gpiod_direction_output(i2c_scl_desc, 0);
    mdelay(1); // 延时 1 毫秒
}

// I2C 停止条件函数
void i2c_stop(void)
{
    // 将 SCL 和 SDA 引脚设置为低电平
    gpiod_direction_output(i2c_scl_desc, 0);
    gpiod_direction_output(i2c_sda_desc, 0);
    mdelay(1); // 延时 1 毫秒
    
    // 将 SCL 引脚设置为高电平
    gpiod_direction_output(i2c_scl_desc, 1);
    mdelay(1); // 延时 1 毫秒
    
    // 将 SDA 引脚设置为高电平
    // 这将产生 I2C 总线的停止条件
    gpiod_direction_output(i2c_sda_desc, 1);
    mdelay(1); // 延时 1 毫秒
}

// 发送 ACK 信号
void i2c_send_ack(int ack) 
{
    // 设置 SDA 线为输出模式
    gpiod_direction_output(i2c_sda_desc, 0);
    
    if (ack) {
        // 发送 ACK 信号, SDA 线拉低
        gpiod_direction_output(i2c_sda_desc, 0);
    } else {
        // 发送 NACK 信号, SDA 线拉高
        gpiod_direction_output(i2c_sda_desc, 1);
    }
    
    // 拉高 SCL 线 1ms,然后拉低
    gpiod_direction_output(i2c_scl_desc, 1);
    mdelay(1);
    gpiod_direction_output(i2c_scl_desc, 0);
}

// 接收 ACK 信号
int i2c_recv_ack(void) 
{
    int value = 0;
    
    // 设置 SDA 线为输入模式
    gpiod_direction_input(i2c_sda_desc);
    
    // 拉高 SCL 线 1ms
    gpiod_direction_output(i2c_scl_desc, 1);
    mdelay(1);
    
    // 读取 SDA 线的电平状态
    if (gpiod_get_value(i2c_sda_desc)) {
        value = 1; // 接收到 NACK 信号
    } else {
        value = 0; // 接收到 ACK 信号
    }
    
    // 拉低 SCL 线
    gpiod_direction_output(i2c_scl_desc, 0);
    
    // 设置 SDA 线为输出模式并拉高
    gpiod_direction_output(i2c_sda_desc, 1);
    
    return value;
}

// 驱动初始化函数
static int ft5x06_driver_init(void)
{
    // 将 GPIO 编号转换为 GPIO 描述符
    i2c_scl_desc = gpio_to_desc(I2C_SCL);
    if (i2c_scl_desc == NULL) {
        printk("gpio_to_desc error for SCL pin\n");
        return -1;
    }
    
    i2c_sda_desc = gpio_to_desc(I2C_SDA);
    if (i2c_sda_desc == NULL) {
        printk("gpio_to_desc error for SDA pin\n");
        return -1;
    }
    
    // 将 GPIO 引脚设置为输出模式,并初始化为高电平
    // 这是 I2C 总线的空闲状态
    gpiod_direction_output(i2c_scl_desc, 1);
    gpiod_direction_output(i2c_sda_desc, 1);
    
    i2c_start();
    i2c_stop();
    
    return 0;
}

// 驱动退出函数
static void ft5x06_driver_exit(void)
{
    // 释放 GPIO 描述符
    gpiod_put(i2c_scl_desc);
    gpiod_put(i2c_sda_desc);
}

// 注册驱动初始化和退出函数
module_init(ft5x06_driver_init);
module_exit(ft5x06_driver_exit);
MODULE_LICENSE("GPL");
```

## 编写发送和接收数据函数