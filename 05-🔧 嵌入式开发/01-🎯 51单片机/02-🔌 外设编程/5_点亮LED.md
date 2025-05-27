---
文章标题: "[[5_点亮LED]]" 
文章作者: Dakkk
文章概要: |
  讲解51单片机LED灯控制原理和编程实现，涵盖GPIO操作、电流限制计算、延时函数设计和流水灯效果实现
tags:
- "51单片机"
- "LED控制"
- "GPIO编程"
- "寄存器操作"
- "延时函数"
- "流水灯"
- "嵌入式开发"
- "C51编程"
相关文章:
- "[[15_练习一：动态流水灯控制]]"
- "[[17_练习3：串口调光]]"
- "[[0_课程完整内容]]"
- "[[19_触控检测芯片]]"
- "[[7_GPIO输入按键]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/01-🎯 51单片机/02-🔌 外设编程/5_点亮LED.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-01-01 23:30:23
修改时间: 2025-05-27 23:05:39
---

## 1 应用场景

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/e184575813fa78c887f3e43abf7f019e.png)

## 2 点灯原理

插件led灯珠长引脚为正极,短引脚为负极
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/0149a56f85e97ca5b16266957e745234.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/70f1dbdb8abd1adc1845d78116f138af.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/7d02729a038990cc2428c1d0dc0eacd8.png)


LED（发光二极管）**两端存在电压差**，有一定的电流流过时会亮起。电流可以理解为水流，电压差可以理解为水位差，当两个点水位高度不一样时，水流会从高水位流向低水位。

如下图：在+给个电压，-的地方接地，就可以亮起来了
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/89a7ff124bec10c7bab45f0f6a000486.png)

**注意：** 流过LED的电流需要在一定范围内，否则会烧坏LED，**一般小于20ma**，所以我们就需要串联电阻分压，那串联的电阻需要多大阻值呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/220e0a22bc417d7dc3e27da930c877f5.png)


**LED的限流电阻计算：**
- 一般插件LED电流是20ma左右，压降: 红/黄色1.8V, 蓝/白色 3V， 实际电压要看 LED 规格书
- 一般贴片LED电流是5ma左右，  压降（电压分了一部分给电阻）:红/绿/橙色1.8V（灯实际获得的电压），蓝/白色 3V

- 例如：供电电压是3.3V，黄色贴片LED
	- 根据V = I * R ，则R= (3.3 - 1.8) /0.005 （5ma = 0.005）, 所以 R = 300 欧姆
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/8147fc3501ddc317a8b40bf3cbd2369b.png)


**很多时候你看到别人设计的电路中，LED串联的电阻去到几百欧或几千欧都有，是设计错了吗？**
- 实际上这是非常合理的，因为大多数电路中，LED只是一个提示灯，对亮度没有要求，反而希望把功耗降低，所以需要增大限流电阻来实现超低电流，像产品中的贴片LED去到0.5ma也是能看清楚灯光的
## 3 点亮LED

### 3.1 单片机如何点亮LED

首先是接线部分，我们可以`通过单片机的引脚，又叫做GPIO`，这些IO都是具备**输出和输入能力**的

什么是 GPIO 呢？GPIO 的 英文全程是 General-purpose input/output，翻译过来就是：通用输入输出，也就是我们可以
- 作为输出（I）：就可以设置某个引脚输出高低电平
- 作为输入（O）：就读取某个引脚输入的电平

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/7e3b99f375815e3aeb2f5112faf74e74.png)

对于这种有44个引脚的封装方式的51单片机，它有以下IO口：
- P0段：P0.0 P0.1 P0.2 P0.3 P0.4 P0.5 P0.6 P0.7
- P1段：P1.0 P1.1 P1.2 P1.3 P1.4 P1.5 P1.6 P1.7
- P2段：P2.0 P2.1 P2.2 P2.3 P2.4 P2.5 P2.6 P2.7
- P3段：P3.0 P3.1 P3.2 P3.3 P3.4 P3.5 P3.6 P3.7
- P4段：P4.0 P4.1 P4.2 P4.3 P4.4 P4.5 P4.6
其中，P0段的GPIO口叫做P0 IO口，简称P0口

如果要P0作为IO口输出的话，要加一个上拉电阻，因为它是开漏输出（电路内部的驱动部分只负责将输出端拉低至低电平）的，不能输出高电平的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/255fec937775bd5b60fb8c61dc6f8f3a.png)

补充知识点：
- 当寄存器中的某个位被写入`0`时，其对应的输出通常表现为低电平（Low Level）。这是由数字逻辑电路的工作原理决定的。
- **推挽输出（Push-Pull）**：这种输出结构由两个晶体管组成，一个用于将输出拉高（高电平），另一个用于将输出拉低（低电平）。
    - 当寄存器写入`0`时，输出驱动电路将拉低电平（即下拉到`0V`）
    - 当寄存器写入`1`时，输出驱动电路将拉高电平（即接近电源电压）
- **开漏输出（Open-Drain/Collector）**：如果是开漏输出，当写入`0`时，输出引脚会被拉低到低电平；当写入`1`时，输出引脚进入高阻态，`由外部上拉电阻决定是否为高电平`

所以，所以我们的做法是：将LED和电阻串联，`一端接负极（接地）`，`另一端接单片机IO口`，然后 **控制单片机的IO口电平** 就可以决定LED是否点亮了
- 

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/90a1e281fa955697d9a10fbebc84953e.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/ab750693e3261acd869e307890dab89b.png)

注意：通常我们**不采用第一种情况**，因为单片机IO口虽然可以达到高电压，但是输出电流不会大，因此单片机IO口给LED供电时可能会出现**电流不足，导致LED不会全亮**。 

因此，**我们一般选择第二种（原理图也是这种），利用电源直接供电，电流灌入单片机IO口的方式来给LED灯供电**

### 3.2 编程控制IO输出高低电平

让IO空输出高低电平，其实就是`操作芯片对应IO的寄存器（变量）`即可，这部分我们可以在芯片手册上看到

在芯片手册3.3节（特殊功能寄存器SFRs）可以看到

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/3c1fcbb848d32fe2d4f5902d8fc02fb0.png)

51单片机的每组IO口的电平状态都被储存在一个单独的8位寄存器中，寄存器的位为0或1就分别对应其相应的GPIO电平为低或高。

**那如何点亮LED呢？板卡LED部分的原理图如下：**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/a10a6f9ac1a72d479b373101759d7d77.png)

我们可以看到LED1 LED2 LED3分别接在P2.7 P2.6 P2.5上，所以要点亮LED1，就需要`设置P2.7输出低电平`，如何用代码设置IO输出低电平呢？下面是操作方法：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/90e1c1c2cea0e727e0017611d0a2ec01.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/bbd91709890e78ced97b7966ecf0261f.png)

LED灯的顺序如下哦：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/af4f40e05260b253384660803791d924.png)


### 3.3 方式一：操作整个寄存器

直接在C语言代码中用赋值法即可，例如操作P2口的P2.0 P2.1 P2.4 P2.6口输出高电平的代码是
```c
P2 = 0x53;
// 0x53的是16进制的53，即53H，对应二进制01010011，即01010011B，所以它对应P2.0 P2.1 P2.4 P2.6口输出高电平
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/39760ec74d9d3e42346a252d4545efc5.png)

这里的0x53是十六进制的表示方法，为什么需要十六进制，什么是十进制，什么是二进制呢？
- 答案：更好的表示数值。
- **十进制:** 十进制是人类最自然的数字系统，因为我们有十个手指，所以我们使用十进制进行计数，使用 0-9 来表示数字，逢10进1。
- **二进制:** 二进制的出现是因为数字电路中，数字信号只有高/低或开/关两种状态，所以只需要使用两个数字 0 和 1 来表示数字，优点是非常直观，缺点是书写起来冗长，过长的二进制容易记录错误，不便阅读和书写。 
- **十六进制：** 为了可以更紧凑地表示二进制数据，就有了十六进制，使用十六个数字 0-9 和字母 A-F 来表示数字，在代码中，使用0x前缀来表示十六进制

```C
十进制
char i = 10;
十六进制
char i = 0x0a;
```

![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/c1ef9820d80f72927505a820942f7289.png)

所以，我们可以设置P2.0-P2.7输出都为低电平，代码这样写。

```Java
#include <reg52.h>
void main(){
  P2=0x00;  //0000 0000
  while(1)
  {
    
  }
}
```

那如果我们只想控制某个IO，例如P2.7，不想控制P2所有IO呢？是否有办法？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/d4d9cce82eb876f9628f1a357ffb20de.png)
```c
P2=0x7F 
```
### 3.4 方式二：操作寄存器某个位

在C语言中，先定义你要操作的位，使用`关键字sbit`（这是标准C语言没有的关键字，是C51编译器的拓展关键字，用于定义寄存器的某个位）

你就可以理解成sbit就是给寄存器的某个位起一个别名。之后用等号赋值即可更改此位的0或1
```c
sbit led1= Px^y;
```

符号`＾`在C51中也表示寄存器的第某位。如上方代码，`Px^y表示Px寄存器的第y位`。
```c
//定义变量led1表示P2寄存器的第7位 
sbit led1= P2^7;
```

注意:
- 在 Keil C 编译器中，`^` 被设计为一个“位选择符”
## 4 闪烁控制

前面，我们通过设置某个引脚输出低电平，点亮LED，也就是我们设置输出高电平时，会关闭LED，那我们需要实现LED闪烁的功能时，如何实现呢？

答案是：先点亮LED，延时一段时间，关闭LED，再延时一段时间，如此反复，就可以得到一个闪烁的LED功能。

```c
#include "reg52.h"

sbit led1 = P2^7;

void main()
{
	while(1){
		led1 = 0;
		led1 = 1;
	}
}
```

注意：上面这个代码，理论是可以实现LED的闪烁，但是我们的代码实际执行是很快的，所以我们肉眼无法察觉到LED在闪烁，即LED1是常亮的
- 所以，我们需要再亮和灭之间加一个延时

在单片机中如何实现延时功能呢？有以下几种生成延时函数的生成方法

**1、让芯片循环执行一些无意义的代码**
- 我们知道，芯片执行每一句代码都需要时间，假设执行一句代码的时间为1ms，那我们点亮LED，然后执行500句（500ms）无意义的代码，再关闭LED，就可以得到闪烁的效果啦。

> 51单片机的时钟、机器周期和指令周期
> 1、时钟周期
> 	主频：芯片主时钟的频率，通常以MHz或者GHz为单位，**单片机的主时钟频率是外部晶振**。时钟周期（振荡周期） = 晶振频率的倒数。例如12M的晶振，它的时间周期就是1/12 us)，是计算机中最基本的、最小的时间单位
> 2、机器周期
> 	在芯片中，为了便于管理，常把一条指令的执行过程划分为若干个阶段，每一阶段完成一项工作。例如，取指令、存储器读、存储器写等，这每一项工作称为一个基本操作。完成一个基本操作所需要的时间称为机器周期。一般情况下，**一个机器周期由若干个时钟周期组成**。
> 3、指令周期
> 	CPU从存储器中取出并执行一条指令所需的全部时间称之为指令周期。
> 	
> 	CPU执行一条指令的过程，计算机每执行一条指令的过程，可分解为如下步骤：
> 		Instruction Fetch（取指令）：指令放在存储器，通过PC寄存器和指令寄存器取出指令的过程，由控制器（Control Unit）操作。 从PC寄存器找到对应指令地址，据指令地址从内存把具体指令加载到指令寄存器，然后PC寄存器自增；
> 		
> 		Instruction Decode（译码）：据指令寄存器里面的指令，是哪一种类型的指令，解析成要进行什么操作，具体要操作哪些寄存器、数据或内存地址。该阶段也是由控制器执行；
> 		
> 		Execute（执行）：实际执行算术逻辑操作、数据传输或者直接的地址跳转操作。无论是算术操作、逻辑操作的指令，还是数据传输、条件分支的指令，都由算术逻辑单元（ALU）操作，即由运算器处理。如果是一个简单的无条件地址跳转，那可直接在控制器里完成，无需运算器
> 		
> 	重复1～3的过程，这个循环完成的时间即指令周期，指令不同，所需的机器周期数也不同，一般由1~4个机器周期组成。
> 	
> 	如：MOV A, Rn （数据传送指令） 就只需要一个机器周期， DIV AB （除法指令）就需要四个机器周期。
> 
> 注意：上面讲的指令周期是51单片机的汇编指令，但我们写的程序是用C语言，C语言最终是会编译成汇编指令，在keil上的debug中可以看得到，后面讲的内容中延时程序就是通过以上的指令周期来进行计算的

具体如何计算呢：使用编译器编译为汇编代码，结合芯片的指令周期计算时间，可以参考https://godbolt.org/
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
```

```c
#include <reg52.h>

sbit led = P2^7;

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
    while(1)
    {
        led = 0;
        delay_ms(500);
        led = 1;
        delay_ms(500);
    }
}
```

**2、使用STC-ISP工具生成**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/3b5085cf973db3620b3c347590e4bf88.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/8bdbabd74bff4f110922fa89664a9e8c.png)


可以看到，我们芯片晶振使用的延时是 11.0592 MHz的

可以选择延时为 1000ms，然后拷贝c代码
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/d832e31ddf660615e28a2eea817de1c9.png)

代码如下：
```c
#include "reg52.h"
#include "intrins.h"

sbit led1 = P2^7;
sbit led2 = P2^6;
sbit led3 = P2^5;

void Delay500ms()		//@11.0592MHz
{
	unsigned char i, j, k;

	_nop_();  // 需要intrins.h头文件
	i = 4;
	j = 129;
	k = 119;
	do
	{
		do
		{
			while (--k);
		} while (--j);
	} while (--i);
}


void main()
{
	while(1){
		led1 = 0;
		Delay500ms();
		led1 = 1;
		Delay500ms();
	}
}
```
## 5 流水灯

如果需要实现多个灯的显示和熄灭呢？

其实就是流水灯的效果

```C
#include "reg52.h"
#include "intrins.h"

sbit LED1 = P2^7;
sbit LED2 = P2^6;
sbit LED3 = P2^5;

void Delay1000ms()                //@11.0592MHz
{
        unsigned char i, j, k;

        _nop_();
        i = 8;
        j = 1;
        k = 243;
        do
        {
                do
                {
                        while (--k);
                } while (--j);
        } while (--i);
}

void main(){
        while(1){
                LED1 = 0;
                LED2 = 1;
                LED3 = 1;
                Delay1000ms();
                LED2 = 0;
                LED1 = 1;
                LED3 = 1;
                Delay1000ms();
                LED3 = 0;
                LED1 = 1;
                LED2 = 1;
                Delay1000ms();
        }
}
```

### 代码优化1

使用P2直接控制

```c
#include "reg52.h"
#include "intrins.h"

void Delay500ms()		//@11.0592MHz
{
	unsigned char i, j, k;

	_nop_();
	i = 4;
	j = 129;
	k = 119;
	do
	{
		do
		{
			while (--k);
		} while (--j);
	} while (--i);
}


void main()
{
	while(1){
		P2 = 0x7F;
		Delay500ms();
		P2 = 0xBF;
		Delay500ms();
		P2 = 0xDF;
		Delay500ms();
	}
}
```

### 代码优化2

使用函数进行led的控制，本质上是二进制的运算

```c
#include "reg52.h"

void delay_ms(unsigned int xms) 
{
    unsigned int i, j;
    for(i=xms;i>0;i--)
    {
        for(j=124;j>0;j--)
        {}
    }
}


void open_led(unsigned int num){
	P2 = 0xFF & ~(1<<num)
}

void main()
{
	while(1){
		open_led(7);
		delay_ms(500);
		open_led(6);
		delay_ms(500);
		open_led(5);
		delay_ms(500);
	}
}
```


## 6 扩展（bit、sbit、sfr）

bit、sbit、sfr的区别
- 首先，bit和sbit都是C51扩展的变量类型
- bit是一种数据类型，他就跟int、char、double这些数据类型一样，只不过char=8位, bit=1位而已，这种变量只有两种值存在0或是1，和bool类似

- sbit更像是define或者typedef一样（define和typedef也有区别，这又是另外一个坑，这里就不说了），ta是为已经分配了内存空间的变量重新取一个别名，一般用来定义特殊功能寄存器的**位变量**，以方便对寄存器的某位进行操作的，例如
```shell
bit i = 0；意思就是在内存中划一块空间给i，让他存储0这个数据位。
sbit Flag = P0^1；意思就是给P0^1取一个别名，它叫做Flag，他是一个位操作变量，它代表访问的地址是P0^1，0x80的第二位数据位
```

- sbit的用法有三种：
	- 第一种方法：sbit 位变量名＝地址值
	- 第二种方法：sbit 位变量名＝SFR名称^变量位地址值
	- 第三种方法：sbit 位变量名＝SFR地址值^变量位地址值

```Java
如定义PSW中的OV可以用以下三种方法：
sbit OV=0xd2 （1）说明：0xd2是OV的位地址值
sbit OV=PSW^2 （2）说明：其中PSW必须先用sfr定义好
sbit OV=0xD0^2 （3）说明：0xD0就是PSW的地址值
因此这里用sfr P1_0=P1^0;就是定义用符号P1_0来表示P1.0引脚，如果你愿意也可以起P10一类的名字，只要下面程序中也随之更改就行了
```

- sfr 不是标准C 语言的关键字，而是Keil 为能直接访问80C51 中的SFR 而提供了一个新的关键词，其用法是：
	- `sfrt 变量名=地址值。`
	- 例：`sfr P1 = 0x90;`
	- 这样的一行即定义P1 与地址0x90 对应，P1 口的地址就是0x90.
	- SFR的定义在头文件reg51.h或reg52.h中。