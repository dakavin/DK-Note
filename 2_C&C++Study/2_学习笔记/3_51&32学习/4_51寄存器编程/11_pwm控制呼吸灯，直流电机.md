
## 1 什么是PWM

PWM是脉冲宽度调制的缩写，它是一种通过**调整脉冲信号的高电平和低电平时间比例**来控制电路输出的技术。

简单来说，PWM是一种控制电子设备输出电压或电流的方法，它可以通过**调整脉冲信号的宽度来控制输出信号的平均电压或电流**，常应用于电机控速，开关电源等领域；
## 2 PWM的频率与占空比

在PWM波形中有两个主要的参数，分别是：`频率`、`占空比`；

**PWM的频率**
- 指1s内信号从高电平到低电平再回高电平的次数（一个周期）
- 如果频率为1000Hz，即1s内来回了1000次，那么每次的时间就是1ms，此信号一个周期就是1ms
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/48a1463231734310c8c6b60fb1c27605.png)
**占空比**
- 一个脉冲周期内，高电平的时间与整个周期时间的比例
- 高电平时间 / 整个周期时间

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/c0102f4d6984615ca2729df5a24e56a7.png)

- 调节占空比最终会反映到输出的电流、电压上，或者可以理解为输出的总能量变化
	- 100%占空比时，输出100%能量
	- 50%占空比时，只会输出一半的能量
	- 例如50%占空比控制LED会比较暗，控制电机力气会比较小

## 3 呼吸灯控制

下面使用IO模拟的方式，实时修改占空比，控制LED的呼吸灯效果：
- 延时函数通过软件获取，然后修改：添加一个t的参数，然后在while循环中t--
	- `t==1` 表示该函数延时10us
	- `t==10` 表示该函数延时100us
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/1ce9276ed11f06a35bcc9a59e5487505.png)

```c
#include <reg52.h>

sbit led = P2^7;

void Delayus(int t){
	while(t--){
		unsigned char i;
		i = 2;
		while(--i);
	}
}

void main(){
	int time = 0;
	led = 0;
	while(1){
		for(time = 0;time < 100;time++){  // 从暗逐渐变亮的过程
			led = 0;
			Delayus(time); // 决定低电平的时间占比
			led = 1; 
			Delayus(100 - time); // 决定高电平的时间占比
		}
		for(time = 0;time < 100;time++){  // 从亮逐渐变暗的过程
			led = 1;
			Delayus(time);
			led = 0;
			Delayus(100 - time);
		}
	}
}
```

除了使用IO模拟的方式，我们也可以**通过定时器功能，产生更加精确的PWM波形**
- 设置定时器的溢出时间可以改变PWM的周期频率
- 设置比较值大小可以调节PWM的占空比

## 4 直流电机控制

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/2dc9d739c65df59a2df587e4cbb983ea.png)

直流电机是指能将直流电能转换成机械能转动的设备
- 直流电机的结构是由定子和转子组成。直流电机运动时，不动的部分称为定子，定子的主要作用是产生磁场。运行时转动的部分称为转子，其主要作用是产生电磁转矩和感应电动势
- 直流电机没有正负之分，在两端加上直流电就能工作。交换电极可以改变电机旋转的方向

51单片机作为一个开发板，启动的主要作用是控制，而直流电机作为一个较大功率的器件，当然不能通过51单片机的IO引脚进行驱动，需要使用一些驱动电路或者芯片进行驱动，板卡上用的是`TC118S`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/98f0e42679ceca9fd76587ec3087300d.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/3ca92192f1500b8689a8fdf499546583.png)

- 可以看到，电机的IO经过了TC118S电机驱动芯片，作用是增加IO的驱动电流
- 如果要电机跑起来的话，就需要给电机两端一个电压差，即AN1输出高电平，AN2输出低电平，那么对应芯片的OUTA（高电平）、OUTB（低电平）

```c
#include <reg52.h>

sbit dc_an1 = P2^4;
sbit dc_an2 = P2^3;

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
	while(1){
		dc_an1 = 1;
		dc_an2 = 0;
		delay_ms(2000);
		dc_an1 = 0;
		dc_an2 = 0;
		delay_ms(2000);
	}
}
```


使用PWM来控制电机的输出频率和占空比，我们可以使用定时器来设置频率和占空比
- **选择定时器0，模式1：** 所以TMOD = 0x1；
- 单片机系统时钟是11.0592MHz时，在12T模式，定时器频率为11.0592MHz除12为921600Hz
	- 一秒计数921600次，`10ms 9216次` `20ms 9216*2次` `1ms 约922次`
	- 所以TIMS = 65536 - 922，TH0 = TIMS >> 8  , TL0 = TIMS
- 开启定时器0的中断，ET0 = 1 ， EA = 1 
- 定时器0开始计数 TR0 = 1；

- 设置参数 count 和 compare 来调整占空比
- 一个完整的 PWM 周期为 100ms（因为 `count` 在 0~99 循环）
- 通过调整 `compare`，控制高电平持续时间比例，从而改变电机转速。

```C
#include <reg52.h>
//约1ms溢出
#define TIMS (65536 - 922) //1ms，可以通过调整这个数，来调整PWM的频率
sbit dc_an2= P2^3;
sbit dc_an1= P2^4;
unsigned char count,compare;

//延时ms函数，参数用来改变延时的具体时间
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
        TMOD = 0x01;      //配置定时器0为16位定时器，TH0、TL0全用
        TH0 = TIMS >> 8;  //设置定时初值高8位
        TL0 = TIMS;       //设置定时初值低8位
        ET0 = 1;  //开启定时器0中断                                          
        EA  = 1;  //开启全局中断                                                      
        TR0 = 1;  //定时器0开始计数  
        //初始化电机，停止运行
        dc_an1 = 0;
        dc_an2 = 0;  
        while(1){
            compare=10;
            delay_ms(5000);
            compare=30;
            delay_ms(5000);
            compare=60;
            delay_ms(5000);
            compare=90;
            delay_ms(5000);
        }
}

//1ms触发执行一次
void Timer0() interrupt 1
{
        //每次产生中断后重新设置下次定时器初值 - 1毫秒产生1次中断
        TH0 = TIMS >> 8;
        TL0 = TIMS;
        //当compare为10时，输出10ms高电平，90ms低电平
        //当compare为60时，输出60ms高电平，40ms低电平
        //当compare为90时，输出90ms高电平，10ms低电平
        count++;//定时计数器的计数值，1ms加1
        count%=100;//限制count在0-99内，count回到0的时候，表示一个PWM周期（100ms）
        if(count<compare)//计数值与比较值进行比较
        {
            dc_an2=1;
        }
        else
        {       
            dc_an2=0;
        }
}

```