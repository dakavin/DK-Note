
📢`SPI`是一种常见的设备通用通信协议。它有一个独特优势就是可以无中断传输数据，可以连续地发送或接收任意数量的位。而在`I2C`和`UART`中，数据以数据包的形式发送，有着限定位数。

下图是一个SPI一主多从通信系统，主机通过共享的SCLK、MOSI、MISO总线连接多个从机，并通过独立的片选信号SS1、SS2、SS3分别控制每个从机的通信
 ![SPI.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4cc94e34a791aa94a31c14ec0ab691d8.png)
手册：

SPI Block Guide V04.01.pdf

## 1 SPI优缺点

- 优点 `SPI`通讯无起始位和停止位，因此数据可以连续流传输而不会中断；没有像`I2C`这样的复杂的从站寻址系统，数据传输速率比`I2C`更高（几乎快两倍）。独立的`MISO`和`MOSI`线路，可以同时发送和接收数据。
    
- 缺点 `SPI`使用四根线（`I2C`和`UART`使用两根线），没有信号接收成功的确认（`I2C`拥有此功能），没有任何形式的错误检查（如`UART`中的奇偶校验位等）。
    

## 2 SPI通信工作原理


在`SPI`设备中，设备分为**主机**与**从机**系统。

- 主机是控制设备（通常是控制器）
    
- 从机（通常是传感器或存储芯片）从主机那获取指令。
    

一套SPI通讯共包含四种信号线：

- `MOSI (Master Output/Slave Input)` – 信号线，主机输出，从机输入。
    
- `MISO (Master Input/Slave Output)` – 信号线，主机输入，从机输出。
    
- `SCLK (Clock)` – 时钟信号。
    
- `SS/CS (Slave Select/Chip Select)` – 片选信号。
    

### 2.1 时钟信号

每个时钟周期传输一位数据，因此数据传输的速度取决于时钟信号的频率。 时钟信号由于是主机配置生成的，因此`SPI`通信始终由主机启动。

**设备共享时钟信号的任何通信协议都称为同步**。`SPI`是一种同步通信协议，还有一些异步通信不使用时钟信号。 例如在`UART`通信中，双方都设置为预先配置的波特率，该波特率决定了数据传输的速度和时序。

下图是一个标准的SPI一主一从通信连接图，主机和从机通过MOSI、MISO、SCLK和SS/CS四条信号线进行连接
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/07234052a6142ddf81f768e1f8b2aaeb.png)


### 2.2 片选信号

主机通过拉低从机的`CS/SS`来使能通信。 在空闲/非传输状态下，片选线保持高电平。在主机上可以存在多个`CS/SS`引脚，允许主机与多个不同的从机进行通讯。

下图是一个SPI一主多从系统连接图，一个主机通过共享的MOSI、MISO、SCLK总线和独立的片选信号SS1、SS2、SS3分别连接三个从机设备
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/482b81aa18632771fd9bcea32483389e.png)


### 2.3 MOSI和MISO

主机通过`MOSI`以串行方式将数据发送给从机，从机也可以通过`MISO`将数据发送给主机，两者可以同时进行。所以理论上，`SPI`是一种全双工的通讯协议。

传输步骤: 以传输模式：`CPOL = 0`、`CPHA = 0` 为例：

1. 主机输出时钟信号
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/07234052a6142ddf81f768e1f8b2aaeb.png)


1. 主机拉低`SS / CS`引脚，激活从机
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4fc475cbbbb11f391d0cfe25e89d94ff.png)


1. 主机通过`MOSI`将数据发送给从机
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5caecc49ffc06763540696caa4b83a3b.png)


1. 如果需要响应，则从机通过`MISO`将数据返回给主机
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5a35063e1507181109f75903e4bd7dc0.png)


## 3 数据传输

为了开始通信，总线上的主设备需要使用从设备支持的频率来配置时钟，这个频率最高为几兆赫兹左右。然后主设备将某个从设备的SS线置为低电平，来选中这个从设备。如果等待时间是必要的话（例如进行模数转换），主设备必须在这段时间结束后，才可以发出时钟周期信号。

在每个SPI时钟周期内，都会发生全双工数据传输。主设备在MOSI线上发送一个位，从设备读取它，同时从机在MISO线上发送一位数据，主机读取它。即使只有单向数据传输的目的，主从机之间的通信工作方式仍然是双工的。

传输通常会使用给定字长的两个移位寄存器，一个在主设备中，一个在从设备中，这两个寄存器连接成一个虚拟的环形缓冲器。数据通常先从最高位移出。在时钟信号边沿，主机和从机均移出一位，然后在传输线上输出给对方。在下一个时钟沿，每个接收器都从传输线接受对方发出的数据位，并且从移位寄存器的最低位推入。每完成这样一个移出——推入的周期后，主机和从机就交换寄存器中的一位数据。当所有数据位都经过了这样的移出——推入过程后，主机和从机就完成了寄存器上的数据交换。如果需要交换的数据比寄存器的位数还要长的话，则需要重新加载移位寄存器并重复该过程。传输可能会持续任意数量的时钟周期。完成后，主设备会停止发送时钟信号，并通常会取消选择从设备。

下图是一个SPI主从设备的内部移位寄存器结构图，显示了主机和从机通过SCLK时钟同步，在MOSI和MISO线上进行8位数据的串行移位传输
![数据传输.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8783b976f742849ee93b6d1e7cca67db.png)

传输寄存器通常包含`8`位。但是其他字长也很常见，例如触摸屏控制器或音频编解码器通常采用`16`位字长（如德州仪器的`TSC2101`），许多数模转换或者模数转换的设备则会采用`12`位字长。

所有在总线上的没有被片选线激活的从设备必须忽略输入时钟和`MOSI`信号，并且不得从`MISO`发送数据。

## 4 模式

分别是 CPOL （Clock POlarity）和 CPHA （Clock PHAse）。

- CPOL配置SPI总线的极性
    
- CPHA配置SPI总线的相位
    

### 4.1 SPI总线的极性：CPOL

极性，会直接影响SPI总线空闲时的时钟信号是**高电平**还是**低电平**。

- CPOL = 1：表示空闲时是高电平
    
- CPOL = 0：表示空闲时是低电平

如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c990ff33703aa1e3896c6999ba501271.png)


### 4.2 SPI总线的相位

一个时钟周期会有2个跳变沿。而相位，直接决定SPI总线从那个跳变沿开始采样数据。

- CPHA = 0：表示从第一个跳变沿开始采样
    
- CPHA = 1：表示从第二个跳变沿开始采样

如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8b6fbd31139ad6207ddbb18627272839.png)


### 4.3 4种模式

CPOL 和 CPHA 的不同组合，形成了SPI总线的不同模式。

|   |   |   |
|---|---|---|
|mode|CPOL|CPHA|
|mode 0|0|0|
|mode 1|0|1|
|mode 2|1|0|
|mode 3|1|1|

### 4.4 模式0 (CPOL=0; CPHA=0)

特性：

```TOML
CPOL = 0：空闲时是低电平，第1个跳变沿是上升沿，第2个跳变沿是下降沿
CPHA = 0：数据在第1个跳变沿（上升沿）采样
```

如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c79e585de4a665759209f7b937013045.png)


### 4.5 模式1 (CPOL=0; CPHA=1)

特性：

```TOML
Copy
CPOL = 0：空闲时是低电平，第1个跳变沿是上升沿，第2个跳变沿是下降沿
CPHA = 1：数据在第2个跳变沿（下降沿）采样
```

效果图：如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/53772e02305f719b7f799ca7e42c9cf4.png)


### 4.6 模式2 (CPOL=1; CPHA=0)

特性：

```TOML
Copy
CPOL = 1：空闲时是高电平，第1个跳变沿是下降沿，第2个跳变沿是上升沿
CPHA = 0：数据在第1个跳变沿（下降沿）采样
```

效果图：如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/da72d6c6906f24fbb4a45c0fdfea4ec4.png)


### 4.7 模式3 (CPOL=1; CPHA=1sa)
如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b3422cdf06fd21012c1b715a6d62218e.png)
