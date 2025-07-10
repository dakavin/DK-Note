---
文章标题: "[[13_IIC、传感器温湿度]]" 
文章作者: Dakkk
文章概要: |
  详细介绍I2C协议基础和51单片机软件模拟I2C时序，通过温湿度传感器GXHT3L实现数据读取和CRC校验
tags:
- "I2C协议"
- "51单片机"
- "温湿度传感器"
- "软件I2C"
- "串行通信"
- "嵌入式开发"
- "GXHT3L"
- "传感器驱动"
相关文章:
- "[[_常用函数]]"
- "[[0_课程完整内容]]"
- "[[10_蜂鸣器]]"
- "[[17_练习3：串口调光]]"
- "[[3_创建工程]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/01-🎯 51单片机/02-🔌 外设编程/13_IIC、传感器温湿度.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-01-01 23:30:23
修改时间: 2025-05-27 23:31:07
---

## 1 I2C简介

IIC协议，又称I2C协议，是由PHILP公司在80年代开发的两线式串行**总线**，用于**连接微控制器及其外围设备**，I2C属于**半双工同步通信方式**。

1、`I2C`是一种`同步`的串行通信 **总线协议，它可以在多个设备之间传输数据** 。 I2C总线由两根线组成：数据线（SDA）和时钟线（SCL）。它使用主从模式，其中一个设备作为主设备控制总线并向其他设备发出命令。I2C协议可以支持高速数据传输和多设备通信，但它的距离限制较短。

2、`UART`是一种`异步`的串行通信协议，它用于在两个设备之间传输数据。UART协议使用两根线：TX（发送）和RX（接收）。UART没有时钟线，数据传输的时序是通过发送和接受设备之间的协议约定实现的。UART协议通常用于短距离通信，例如在计算机和串口设备之间进行通信。

3、因此，I2C和UART协议在通信的方式、数据传输速度和距离限制等方面存在差异，根据具体的应用场景和需求选择合适的协议更为重要

### 1.1 多主控（multimastering）

其中任何能够进行发送和接收的设备都可以称为主总线，一个主控能够控制信号的传输和时钟频率。当然，在任何时间点上只能有一个主控

**串口是通过协商波特率，来确定时钟频率的**
### 1.2 特征：简单和有效

两根线
- 在标准模式下，I2C总线的最大长度为5M，最大速率为 100Kbit/s
- 在快速模式下，I2C总线的最大长度为1M，最大速率为 400Kbit/s
- 在高速模式下，I2C总线的最大长度为0.4M，最大速率为 3.4Mbit/s

**需要注意的是，总线长度的实际限制还取决于总线上的电容负载和电缆质量等因素**

## 2 组成

I2C串行总线一般有两根信号线，一根是双向的数据线SDA，另一根是时钟线SCL，其时钟信号是由主控器件产生，数据线是用来传输数据的，时钟线是用来使双方通信的时钟同步。

所有接到I2C总线设备上的串行数据SDA都截到总线的SDA上，各设备的时钟线SCL接到总线的SCL上，对于并联在一条总线上的每个IC都有唯一的地址

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/a0a36f3183879f2bffe872fbb0481ff2.png)
### 2.1 主机与从机

主机（master），就是老板，上下班时间，都是他来决定的，指令都是他来发出的，他要找谁干啥，谁就干啥，一般主机就是我们的单片机

从机（slave），就是员工，需要做什么听老板的，一般从机就是外围设备，例如温湿度传感器、EEPROM芯片、测距仪等I2C子设备

### 2.2 硬件I2C与软件I2C

`硬件I2C`：芯片里面把I2C的通讯协议通过电路实现了，由专用的I2C引脚，只需要配置下寄存器就能实现I2C通讯（将数据放入对应的寄存器即可）
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/993e30f4d4bf26d9d4b79b81999e0941.png)


`软件I2C`：根据I2C通讯的时序协议，自己找两个引脚，按照I2C协议的时序实现

**51是不支持硬件I2C的，所以我们要驱动I2C设备，只能通过代码实现软件I2C，简单来说就是通过IO口模拟I2C的时序**

通过I2C传输1各Byte数据的时序如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/43f641fd03d1c548e7bdf8ff49a9c7ab.png)

I2C完成的通讯过程如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/94c53f3ff0fb401fa0cdc09253a74412.png)

所以，**I2C完整的通讯过程**
1. 总线为空闲状态时，SCL=1，SDA=1；
2. 开始传输数据时，SCL=1（高电平），主机将SDA从1变成0
3. 发出需要通讯的从机地址
	- 地址一般是8bit（也有16bit），真实的地址是7bit，最后1bit用来表示读或写（1-读/0-写）
	- 该过程就是主机向SDA发送了8bit的数据，即从机地址也是数据
4. 从机给主机应答信号（ACK）
5. ACK后，开始传输数据，根据地址最后1bit确定是读或写
	- 写，主机来控制SDA的电平变化
	- 读，从机来控制SDA的电平变化
6. 每次8bit的数据传输完成，都有一个应答信号（ACK，谁接收，谁应答）
7. 最后，在SCL为高电平时，主机把SDA从低电平拉高，表示结束

#### 2.2.1 10us延时函数

延时函数影响I2C速率
- 选择速率为50Kbit/s
	- 50Kbit/s = 1s 50k周期 ---> 1周期 = 1/50ks = 20us
	- 高电平和低电平的时间为各10us
- 10us发送1bit，1ms发送100bit，1s发送100kbit

```c
void delay10us()                //@11.0592MHz
{
	unsigned char i;
	i = 2;
	while (--i);
}
```

#### 2.2.2 起始信号

发送数据前要先发送起始信号，告知总线和从机开始通讯
- SCL为高时，SDA从高变低

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/43f641fd03d1c548e7bdf8ff49a9c7ab.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/700d32722c7c8d0e76ca6d4a1932a3b1.png)

```c
#include <reg52.h>

sbit sda = P0^1;
sbit scl = P0^2;

void i2c_start()
{
	scl = 1;
	sda = 1;
	delay10us(); //起始信号建立时间，高电平时间持续大于4.7us
	sda = 0;     //SDA拉低，下降沿
	delay10us(); //起始信号保持时间
}
```

#### 2.2.3 结束信号

发送完成需要发结束信号，告知总线和从机通讯完成
- SCL为高时，SDA从低变高

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/cbb18e1efd04e50e3ee4743eddbeeb33.png)

```c
void i2c_stop()
{
	scl = 1;
	sda = 0;
	delay10us(); //停止信号建立时间，SDA高电平时间持续大于4.7us
	sda = 1;     //SDA拉高，上升沿
	delay10us(); //总线空闲时间保持
}
```
#### 2.2.4 数据信号

逐位发送数据或读取数据
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/43f641fd03d1c548e7bdf8ff49a9c7ab.png)

对于数据 0x20 = 0b0010 0000
- 从高往低发，即左往右

**发送1byte**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/0b0d31df07d7c879df4efbe7a1aed39d.png)

优化后
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/f9bd80d1c4c9e1e9ff5cb96e9c73d8f5.png)

- 1byte = 8bit，发送1bit的流程
	- SCL拉低
	- SDA发送数据，维持一段时间
	- SCL拉高，维持一定时间
	- SCL拉低
```c
//单独实现
void i2c_write_bit(unsigned char databit){
	scl = 0; // 开始发送前，拉低SCL
	if(databit == 0){ // 设置需要发送的数据
		sda = 0;
	}else {
		sda = 1;
	}
	delay5us();
	scl = 1;
	delay10us();
	scl = 0;     
	delay5us();
}

void i2c_write_bit(unsigned char databit){
	scl = 0; // 开始发送前，拉低SCL，也可以放在sda设置的后一行，因为同时设置的
	if(databit == 0){ // 设置需要发送的数据
		sda = 0;
	}else {
		sda = 1;
	}
	delay10us();
	scl = 1;
	delay10us(); // 这里是考虑到，scl
}

void i2c_write_byte(unsigned char datasend){
	int i;
	for(i = 0;i < 8;i++){
		if(datasend & 0x80){
			i2c_write_bit(1);
		}else {
			i2c_write_bit(0);
		}
		datasend = datasend << 1; //左移一位，for循环8次，直到数据发送完
	}
}

// 合二为一
void i2c_write_byte(unsigned char datasend){
	unsigned char i;
	for(int i = 0;i < 8;i++){
		scl = 0;
		if(datasend & 0x80){
			sda = 1;
		}else {
			sda = 0;
		}
		datasend = datasend << 1; //左移一位，for循环8次，直到数据发送完
		delay10us();
		scl = 1;
		delay10us();
		scl = 0;
	}
}
```

**读取1byte**
- 1byte=8bit 接收1bit的流程是
	- **注意**：sda需要先处于低电平，从机检测到该状态后，在SCL拉高的时候，才会发送数据给主机
	- SCL拉低，延时5us
	- 拉高SCL延时10us，等待数据稳定，读取SDA的值
	- SCL拉低，延时5us（可以和前面的5us组合）
```c
// 单独实现
unsigned char i2c_read_bit(){
    unsigned char databit = 0;
    scl=0;   //每bit接收前，先设置scl为0
    delay5us();//scl为低电平的时间
    scl=1; //拉高scl开始接收数据
    delay10us();//等待数据稳定的时间
    if(sda)
       databit  1;
    else
       databit = 0; 
    scl=0;
    delay5us();
    return databit;
}

unsigned char i2c_read_bit(){
    unsigned char databit = 0;
    scl=0;   //每bit接收前，先设置scl为0
    delay10us();//scl为低电平的时间
    scl=1; //拉高scl开始接收数据
    delay10us();//等待数据稳定的时间
    if(sda)
       databit = 1;
    else
       databit = 0;
	 // 因为后续，读取数据，还会拉低scl，所以就不写了，直接在scl拉低时延时10us
	return databit;
}

unsigned char i2c_read_byte()//接收数据
{
	unsigned char i = 0;
    unsigned char value = 0;
    sda = 1;    //sda置位空闲状态
    for(i=0;i<8;i++)
    {
		if(i2c_read_bit())
			value = value | 0x01;
		else
			value = value | 0x00;
		if(i < 7) // 第一次不需要移位
			value = value << 1;
    }
    scl=0; //接收完成后，设置scl为0
    delay10us();
    return value;
}

//合二为一
unsigned char i2c_read_byte()//接收数据
{
		int i,value;
		sda = 1;    //sda置位空闲状态
		for(i=0;i<8;i++)
		{
			scl=0; //每bit接收前，先设置scl为0
			delay10us();//scl为低电平的时间
			scl=1; //拉高scl开始接收数据
			delay10us();//等待数据稳定的时间
			value=value<<1;
			if(sda){
			//每次向左移动一位以后，如果sda=1的时候就把最后一位置1，sda=0的时候则不用置，因为向左移动就有一个0了
				value=value|0x01;
			}
		}
		scl=0; //接收完成后，设置scl为0
		delay10us();
		return value;
}
```
#### 2.2.5 应答信号

完成数据发送后，通过读取应答信号判断从机是否收到数据
- 应答信号（从机）：SCL从低到高再到低时，SDA都为低
- 等待应答信号（主机）：SDA和SCL拉高（总线空闲），SCL从低到高，检查SDA是否被拉低，如果被拉低，则获取到应答信号

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/c308d26d65bf4394bf09834a7feb23e8.png)

```c
// 发送应答信号
void i2c_ack(){
	scl = 0;
	sda = 0; // SDA拉低，发出应答信号
	delay10us();
	scl = 1;
	delay10us();
	scl = 0;
}

// 等待应答信号
unsigned char i2c_wait_ack(){
	unsigned char ucErr = 0;
	sda = 1;  // 先设置SDA高电平，然后检测从机应答（应答会拉低）
	scl = 0;
	delay10us();
	scl = 1;
	delay10us();
	while(sda){ // SDA为高电平，就表示没有检查到ACK
		ucErr++;
		delay10us();
		if(ucErr > 250){ //等待一段时间，还没有ack，就停止总线
			return 1;
		}
	}
	scl = 0;
	return 0;
}

void i2c_wait_ack(){
	unsigned char ucErr = 0;
	sda = 1;  // 先设置SDA高电平，然后检测从机应答（应答会拉低）
	scl = 0;
	delay10us();
	scl = 1;
	delay10us();
	while(sda){ // SDA为高电平，就表示没有检查到ACK
		delay10us();
		ucErr++;
		if(ucErr > 100){ //等待一段时间，还没有ack，就停止总线
			break;
		}
	}
	scl = 0;
}

// 非应答信号
void i2c_nack(){
	scl = 0;
	sda = 1;   //SDA拉高，发出非应答信号
	delay10us();
	scl = 1;
	delay10us();
	scl = 0;
}
```

### 2.3 使用i2c（逻辑分析仪分析）

```c
void main(){
	i2c_start();
	i2c_write_byte(0x55);
	i2c_wait_ack();
	i2c_stop();
	while(1);
}
```

**使用逻辑分析仪**
- 杜邦线，用两根线接到逻辑分析的任意channel，然后另外一段接到板子上SCL和SDA的针脚
- **注意，一定要接地！**
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/0c8aba61d3fe21acfa1f2dc1055be467.png)
- 逻辑分析仪开始start，然后复位一下芯片，可以看到如下数据波形
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/f3626464329e8278a70b52cbd019767e.png)
  - 分析数据可以看到如下：
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/2079a0d1077bb9d7f47d6a6abd1b9859.png)
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/69e2d37323930faf9478089a7a76b199.png)

## 3 温湿度传感器：CJ-GXHT3L

### 3.1 芯片及管脚介绍

**芯片介绍：** GXHT3L-DIS 是中科银河芯开发的新一代单芯片集成温湿度一体传感器，它的特点如下
- I2C接口，通信速度高达 1MHZ
- 两个用户可选择的地址
- 3L系列典型精度为±4%RH和±0.5°C  30系列典型精度为±3%RH和±0.3°C 31系列典型精度为±2%RH和±0.3°C 
- 单芯片集成温湿度传感器
- 高可靠性和长期稳定性
- 测量 0~100%范围相对湿度，-45~130°C范围内湿度
- 芯片手册地址：https://item.szlcsc.com/3199174.html

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/8e6b08607db400688fcdc36918f2a239.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/5c5dc704130e762f8ce403bd027a4666.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/0f2f63edf42253b176f3069950d6083d.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/8466f37af531ee4d3c1300949e3174c5.png)

**原理图介绍**
- P0.1和P0.2这两个IO在51单片机上是没有上拉能力的，所以在外面加了上拉电阻，提高电平

**关于ADDR管脚说明**
- 通过改变ADDR的连接方式可以改变传感器的I2C地址
	- 当ADDR接`低电平`时，传感器芯片的地址为 `0x44`
	- 当ADDR接`高电平`时，传感器芯片的地址为 `0x45`
- 注意：通过过程中ADDR的电平不能发生改变，这种地址选择方式可以将两颗 GXHT3L-DIS连接在同一个I2C总线上
- 注意：I2C的地址是指I2C读写命令头的高7位。读写命令头的最低位是读写指示位，0为写，1为读
- ADDR管脚不能悬空

**注意的是，0x44指的是I2C地址的高7位，第8位为读写标志位**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/e4235c95aa0da045fb96b49b7935538f.png)

### 3.2 单次转换模式

作用：每次初始化这个模式，只能读取一次温湿度值
应用：低功耗场景（休眠和唤醒）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/9e2a0ab0507317c20568c61941f4806b.png)

**Clock stretching 关闭**
- 如果该功能关闭，那么发送温湿度转换命令后，如果温湿度转换还没有完成就开始读温湿度数据，这时候芯片会给出NACK
- 只有等待时间足够长，保证温湿度转换已完成，再读取数据才会得到芯片响应

**Clock stretching 开启**
- 如果该功能开启，不论温湿度测量是否完成，只要上位机发送读数据头，芯片都会给出ACK，然后将SCL拉低
- 一旦测量完成会立即释放SCL总线，然后芯片开始发送测量到的数据

例子如下图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/810b0bc6855d46916b2f2dd305843520.png)

### 3.3 连续转换模式

作用：初始化一次这个模式，芯片会一直检测温湿度，可以随时读取最新的温湿度
#### 3.3.1 设置转换模式

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/8a8cd5c53e9e5f9a5925950bcc02cb79.png)

通过I2C器件的地址，来设置芯片的高重复率和周期转换频率
- 重复率：高——精度高、转换耗时和功耗也高
- 周期转换频率mps：每秒转换频率
- 如：0x2130 ---> 21表示mps为1，即每s转换一次，30代表高重复率

设置读写地址
- 0x44 ---> 0b0100 0100
- 1000 1000 ---> 写地址
- 1000 1001 ---> 读地址

```c
// 注意：addr = 0x44时，实际发送到的总线数据应该是 0x44 + 读写标志位，0写 1读
#define GXHT3L_ADDR_WRITE 0x44<<1
#define GXHT3L_ADDR_READ (0x44<<1)+1

//MSB和LSB设置为0x2130 初始化进入连续转换模式，高重复率，每秒测量一次
void gxht30_init(){
	i2c_up();
	i2c_write_byte(GXHT3L_ADDR_WRITE);
	i2c_wait_ack();
	i2c_write_byte(0x21); // MSB设置msp
	i2c_wait_ack();
	i2c_write_byte(0x30); // LSB设置重复率
	i2c_wait_ack();
	i2c_down();
}
```

#### 3.3.2 进行数据读取

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/beebfbc4d8262d828568aa2c2ffd4522.png)

**开始连续转换读取的命令0xE000**
1. 发送写标志+设备地址，发送开始连续指令（0xE000 ---> MSB = 0xE0 LSB = 0x00）
2. 发送读标志+设备地址，等待SCL拉低，读取温湿度数据

```c
// 周期测量模式读取数据命令
void gxht30_read_mode(){
	i2c_up();
	i2c_write_byte(GXHT3L_ADDR_WRITE);
	i2c_wait_ack();
	i2c_write_byte(0xE0);
	i2c_wait_ack();
	i2c_write_byte(0x00);
	i2c_wait_ack();
	i2c_down();
}

void gxht30_read(){
	int index = 0;
	unsigned char buffer[6];
	i2c_up();
	i2c_write_byte(GXHT3L_ADDR_READ);
	i2c_wait_ack();
	/*
	buffer[0]=i2c_read_byte(1); // 温度高8位
	i2c_ack();
	buffer[1]=i2c_read_byte(1); // 温度低8位
	i2c_ack();
	buffer[2]=i2c_read_byte(1); // CRC
	i2c_ack();
	buffer[3]=i2c_read_byte(1); // 湿度高8位
	i2c_ack();
	buffer[4]=i2c_read_byte(1); // 温度低8位
	i2c_ack();
	buffer[5]=i2c_read_byte(0); // CRC
	i2c_nack();
	i2c_down();
	*/
	for(index = 0;index < 6;index++){
		buffer[index] = i2c_read_byte();
		if(index == 5){
			i2c_nack();
		}else{
			i2c_ack();
		}
	}
	i2c_down();
}
```
#### 3.3.3 温湿度转换

**公式如下：**
- 相对湿度转换公式（%RH）： $RH = 100 * \frac{S_{RH}}{2^{16} - 1}$
- 温度转换公式 (°C & °F):
	- $T[°C] = -45 + 175 * \frac{S_T}{2^{16} - 1}$
	- $T[°F] = -49 + 315 * \frac{S_T}{2^{16} - 1}$

```c
void gxht30_read(){
	int index = 0;
	unsigned char buffer[6];
	unsigned short tem=0,hum=0;
	float temperature = 0.0;
	float humidity = 0.0;
	
	i2c_up();
	i2c_write_byte(GXHT3L_ADDR_READ);
	i2c_wait_ack();
	for(index = 0;index < 6;index++){
		buffer[index] = i2c_read_byte();
		if(index == 5){
			i2c_nack();
		}else{
			i2c_ack();
		}
	}
	i2c_down();
	
	// 合并高字节
	tem = (buffer[0]<<8)|buffer[1];
	hum = (buffer[3]<<8)|buffer[4];
	// 进行温湿度转换
	temperature = (175*(float)tem/65535.0-45.0);
	humidity = 100*(float)hum/65536.0
}
```
#### 3.3.4 CRC8 校验

循环冗余校验，是一种根据网络数据包或电脑文件等数据产生简短固定位数校核码的快速算法，主要用来检测或校核数据传输或者保存可能出现的错误。

数据传输的CRC校验算法如下表所示，CRC校验对象是在它之前传输的2个字节数据

|   属性   |              值              |
| :----: | :-------------------------: |
|   名字   |            CRC-8            |
|   宽度   |            8 bit            |
|  校验对象  |          从取或写入的数据           |
| 生成多项式  | $x^8 + x^7 + x^5 + x^4 + 1$ |
|   初值   |            0xFF             |
|  反向输入  |            False            |
|  反向输出  |            false            |
| 初始 XOR |            0x00             |
|   示例   |     CRC (0xBEEF) = 0x92     |

```c
#define POLYNOMIAL 0x31 // P(x)= x^8+x^7+x^5+x^4+1 = 0011 0001

// CRC校验函数
unsigned char gxht30_crc8(unsigned char *crcdata,unsigned char nbrOfBytes){
	unsigned char Bit;         // bit mask
	unsigned char crc = 0xFF;  // calculated checksum
	unsigned char byteCtr;     // byte counter
	for(byteCtr = 0;byteCtr < nbrOfBytes;byteCtr++){
		crc ^= (crcdata[byteCtr])
		for(Bit = 8;Bit >0;--Bit){
			if(crc & 0x80){
				crc = (crc<<1)^POLYNOMIAL;
			}else{
				crc = (crc<<1);
			}
		}
	}
	return crc;
}

void gxht30_read(){
	int index = 0;
	unsigned char buffer[6];
	unsigned short tem=0,hum=0;
	float temperature = 0.0;
	float humidity = 0.0;
	
	i2c_up();
	i2c_write_byte(GXHT3L_ADDR_READ);
	i2c_wait_ack();
	for(index = 0;index < 6;index++){
		buffer[index] = i2c_read_byte();
		if(index == 5){
			i2c_nack();
		}else{
			i2c_ack();
		}
	}
	i2c_down();
	if(buffer[2]!=gxht30_crc8(buffer,2)) return；
	if(buffer[5]!=gxht30_crc8(&buffer[3],5)) return；
	
	// 合并高字节
	tem = (buffer[0]<<8)|buffer[1];
	hum = (buffer[3]<<8)|buffer[4];
	// 进行温湿度转换
	temperature = (175*(float)tem/65535.0-45.0);
	humidity = 100*(float)hum/65536.0
}
```
#### 3.3.5 串口打印温湿度

```C
#include <stdio.h> 
void uart_init(void)                //9600bps@11.0592MHz
{
        PCON &= 0x7F;                //波特率不倍速
        SCON = 0x50;                //8位数据,可变波特率
        TMOD &= 0x0F;                //清除定时器1模式位
        TMOD |= 0x20;                //设定定时器1为8位自动重装方式
        TL1 = 0xFD;                //设定定时初值
        TH1 = 0xFD;                //设定定时器重装值
        ET1 = 0;                //禁止定时器1中断
        TR1 = 1;                //启动定时器1
}

/*
**重写printf调用的putchar函数，重定向到串口输出
**需要引入头文件<stdio.h>
*****/
char putchar(char dat){
        //输出重定向到串口
        SBUF = dat;     //写入发送缓冲寄存器
        while(!TI);    //等待发送完成，TI发送溢出标志位 置1
        TI = 0;      //对溢出标志位清零
        return dat;  //返回给函数的调用者printf
}
```
#### 3.3.6 main函数实现

```c
//带参延时函数
void delay_ms(unsigned int xms)   //@12MHz
{
    unsigned int i, j;
    for(i=xms;i>0;i--)
    {
        for(j=124;j>0;j--)
        {}
    }
}

void main(){
	uart_init();
	gxht30_init();
	while(1){
		delay_ms(1000);
		gxht30_read_mode();
		gxht30_read();
	}
}
```

## 4 最终工程

```c
#include <reg52.h>
#include <stdio.h> 

sbit sda = P0^1;
sbit scl = P0^2;

#define GXHT3L_ADDR_WRITE 0x44<<1    //0b0100 0100 -> 0b1000 1000
#define GXHT3L_ADDR_READ  (0x44<<1)+1//0b0100 0100 -> 0b1000 1001

void Delay10us()                //@11.0592MHz
{
	unsigned char i;
	i = 2;
	while (--i);
}

void i2c_start()
{
	scl = 1;
	sda = 1;
	Delay10us();
	sda = 0;
}

void i2c_stop()
{
	scl = 1;
	sda = 0;
	Delay10us();
	sda = 1;
}

void i2c_write_bit(unsigned char databit)
{
	scl = 0;
	if(databit == 1)
		sda = 1;
	else
		sda = 0;
	Delay10us();
	scl = 1;
	Delay10us();
}

void i2c_write_byte(unsigned char datasend)
{
	int i = 0;
	for(i = 0;i< 8;i++){
		if(datasend & 0x80){
			i2c_write_bit(1);
		}else{
			i2c_write_bit(0);
		}
		datasend = datasend << 1;
	}
}

unsigned char i2c_read_bit()
{
	unsigned char databit = 0;
	scl = 0;
	Delay10us();
	scl = 1;
	Delay10us();
	if(sda == 1)
		databit =  1;
	else
		databit = 0;
	return databit;
}

unsigned char i2c_read_byte()
{
	unsigned char value = 0;
	unsigned char i = 0;
	sda = 1; //让总线处于空闲状态
	for(i = 0;i < 8;i++){
		if(i2c_read_bit() == 1){
			value = value | 0x01;
		}else{
			value = value | 0x00;
		}
		if(i<7)
			value = value << 1;
	}
	scl = 0;
	Delay10us();
	return value;
}

void i2c_ack()
{
	scl = 0;
	sda = 0;
	Delay10us();
	scl = 1;
	Delay10us();
	scl = 0;
}

void i2c_nack()
{
  scl = 0;
  sda = 1;     //SDA拉高，发出非应答信号
  Delay10us();
  scl = 1;
  Delay10us(); 
  scl = 0;
}

void i2c_wait_ack()
{
	unsigned char time = 0;
	scl = 0;
	sda = 1;
	Delay10us();
	scl = 1;
	Delay10us();
	while(sda){
		Delay10us();
		time ++;
		if(time > 100)
			break;
	}
	scl = 0;
	Delay10us();
}

void gxht30_init()
{
	i2c_start();
	i2c_write_byte(GXHT3L_ADDR_WRITE);
	i2c_wait_ack();
	i2c_write_byte(0x22);
	i2c_wait_ack();
	i2c_write_byte(0x20);
	i2c_wait_ack();
	i2c_stop();
}

void gxht30_read_mode()
{
	i2c_start();
	i2c_write_byte(GXHT3L_ADDR_WRITE);
	i2c_wait_ack();
	i2c_write_byte(0xE0);
	i2c_wait_ack();
	i2c_write_byte(0x00);
	i2c_wait_ack();
	i2c_stop();
	Delay10us();
}

#define POLYNOMIAL  0x31   // P(x) = x^8 + x^5 + x^4 + 1 = 00110001
 
//CRC校验函数
unsigned char gxht30_crc8(unsigned char *crcdata, unsigned char nbrOfBytes)
{
	unsigned char Bit;        // bit mask
	unsigned char crc = 0xFF; // calculated checksum
	unsigned char byteCtr;    // byte counter
	for(byteCtr = 0; byteCtr < nbrOfBytes; byteCtr++)
	{
		crc ^= (crcdata[byteCtr]);
		for(Bit = 8; Bit > 0; --Bit)
		{
			if(crc & 0x80) crc = (crc << 1) ^ POLYNOMIAL;
			else           crc = (crc << 1);
		}
	}
	return crc;
}

void gxht30_read_data()
{
	unsigned short tem,hum;
	int index = 0;
	float temperature,humidity;
	unsigned char buffer[6];
	i2c_start();
	i2c_write_byte(GXHT3L_ADDR_READ);
	i2c_wait_ack();
	for(index = 0; index < 6;index ++){
		buffer[index] = i2c_read_byte();
		if(index == 5)
			i2c_nack();
		else
			i2c_ack();
	}
	i2c_stop();
	if(gxht30_crc8(buffer,2) != buffer[2]){
		printf("crc error\n");
		return;
	}
	if(gxht30_crc8(&buffer[3],2) != buffer[5]){
		printf("crc error\n");
		return;
	}
	
	//合并两个8bit的数据为一个16bit的数据
	tem = (buffer[0] << 8) | buffer[1];
	hum = (buffer[3] << 8) | buffer[4];
	
	//进行温湿度转换
	temperature = (175.0*(float)tem/65535.0-45.0) ;
	// T = -45 + 175 * tem / (2^16-1)
	humidity = (100.0*(float)hum/65535.0);
	// RH = hum*100 / (2^16-1) 
	printf("temperature=%f humidity=%f\n",temperature,humidity);
}


void uart_init(void)                //9600bps@11.0592MHz
{
	PCON &= 0x7F;                //波特率不倍速
	SCON = 0x50;                //8位数据,可变波特率
	TMOD &= 0x0F;                //清除定时器1模式位
	TMOD |= 0x20;                //设定定时器1为8位自动重装方式
	TL1 = 0xFD;                //设定定时初值
	TH1 = 0xFD;                //设定定时器重装值
	ET1 = 0;                //禁止定时器1中断
	TR1 = 1;                //启动定时器1
}

/*
**重写printf调用的putchar函数，重定向到串口输出
**需要引入头文件<stdio.h>
*****/
char putchar(char dat){
	//输出重定向到串口
	SBUF = dat;     //写入发送缓冲寄存器
	while(!TI);    //等待发送完成，TI发送溢出标志位 置1
	TI = 0;      //对溢出标志位清零
	return dat;  //返回给函数的调用者printf
}

//带参延时函数
void delay_ms(unsigned int xms)   //@12MHz
{
    unsigned int i, j;
    for(i=xms;i>0;i--)
    {
        for(j=124;j>0;j--)
        {}
    }
}

void main()
{
	uart_init();
	gxht30_init();
	while(1)
	{
		delay_ms(1000);
		gxht30_read_mode();
		gxht30_read_data();
	}
}
```

## 5 彩蛋

有的细心的小伙伴，在做这个实验的时候发现，从传感器读取的数据会比实际环境温度高1~2度，这是因为这块彩色板卡设计时，为了美观做的牺牲

因为我们板卡工作时，电源模块会有温升，所以温湿度传感设计时得在芯片周边挖空开槽，或者使用插件的方式，但我们实际上打样回来后，发现实在太丑了

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/289fa1fb4d7a9d68f0f24a9243968875.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/f7e797ae43a85cb3a64799fb83e80f45.png)
