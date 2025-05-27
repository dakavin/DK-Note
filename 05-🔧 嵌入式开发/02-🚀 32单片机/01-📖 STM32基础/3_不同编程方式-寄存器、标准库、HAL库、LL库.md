---
文章标题: "[[3_不同编程方式-寄存器、标准库、HAL库、LL库]]" 
文章作者: Dakkk
文章概要: |
  介绍STM32的四种编程方式：寄存器、标准库、HAL库、LL库的特点和对比，重点推荐使用HAL库作为主流开发方式
tags:
- "STM32"
- "HAL库"
- "LL库"
- "标准库"
- "寄存器编程"
- "嵌入式开发"
- "CubeMX"
- "微控制器"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[5_设置烧录参数，烧录固件]]"
- "[[8_外部中断（重点）]]"
- "[[9_定时器]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/3_不同编程方式-寄存器、标准库、HAL库、LL库.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 库介绍

常见的库有标准库、HAL库、LL库

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8f1a678bfbebe31e6e394e8d28ccf5c8.png)

## 2 寄存器

寄存器编程是最接近底层的编程方式，也是运行效率最高的，但是缺点是编程效率低，维护难度高，排查问题效率低

51只有三十来个8位寄存器，而STM32不同版本的寄存器个数是51的数倍，甚至数十倍，还是32位的寄存器，所以官方把这部分寄存器操作封装成一个个函数，我们直接调用函数就可以完成功能的开发

## 3 寄存器编程和库函数编程

下面我们来实现两个功能进行对比

- 功能一：设置GPIOC PIN13为输出模式，控制LED的亮灭
```c
//基于寄存器编程
#include "stm32f10x.h"                  
int main(void)
{
	/*打开时钟 APB2 IOPC*/
	RCC->APB2ENR = 0x00000010;
	/*配置GPIOC13 设置推挽输出 速率50MHZ*/
	GPIOC->CRH = 0x00300000;
	/*设置GPIO输出*/
	GPIOC->ODR = 0x00002000;       
	while(1)
	{
		
	}
}

//基于库编程
#include "stm32f10x.h"
int main(void)
{
	/*打开时钟*/
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);                        
	/*配置GPIO*/
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode        = GPIO_Mode_Out_PP; 
	GPIO_InitStructure.GPIO_Speed       = GPIO_Speed_50MHz;
	GPIO_InitStructure.GPIO_Pin         = GPIO_Pin_13;
	GPIO_Init(GPIOC, &GPIO_InitStructure);
	/*设置GPIO输出*/
	GPIO_SetBits(GPIOC, GPIO_Pin_13);                   
	while(1)
	{
		
	}
}
```

- 功能二：实现设置串口USART1 波特率115200 8位数据位 1位停止位 无奇偶校验
```c
//寄存器编程
void uart_init(u32 pclk2,u32 baudrate)
{           
        float temp;
        u16 mantissa;                
        u16 fraction;           
        temp=(float)(pclk2*1000000)/(baudrate*16);//得到USARTDIV
        mantissa=temp;                         //得到整数部分
        fraction=(temp-mantissa)*16;           //得到小数部分         
        mantissa<<=4;
        mantissa+=fraction; 
        RCC->APB2ENR|=1<<2;   //使能PORTA口时钟  
        RCC->APB2ENR|=1<<14;  //使能串口时钟 
        GPIOA->CRH&=0XFFFFF00F;//IO状态设置
        GPIOA->CRH|=0X000008B0;//IO状态设置 
        RCC->APB2RSTR|=1<<14;   //复位串口1
        RCC->APB2RSTR&=~(1<<14);//停止复位                      
        USART1->BRR=mantissa; // 波特率设置         
        USART1->CR1|=0X200C;  //1位停止,无校验位.
}

//库函数编程
void MX_USART1_UART_Init(u32 baudrate)
{
        //IO初始化
        GPIO_InitStruct.Pin = GPIO_PIN_2;
        GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
        GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
        GPIO_InitStruct.Pin = GPIO_PIN_3;
        GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
        GPIO_InitStruct.Pull = GPIO_NOPULL;
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
        //串口初始化
        UART_HandleTypeDef huart1;
        huart1.Instance = USART1;
        huart1.Init.BaudRate = baudrate;
        huart1.Init.WordLength = UART_WORDLENGTH_8B;
        huart1.Init.StopBits = UART_STOPBITS_1;
        huart1.Init.Parity = UART_PARITY_NONE;
        huart1.Init.Mode = UART_MODE_TX_RX;
        HAL_UART_Init(&huart1);
}
```

可以看到从编程难度、维护难度、移植难度几个维度对比，库函数编程都是优于寄存器编程的。

## 4 标准库

标准库（Standard Peripheral Library）是STMicroelectronic提供最基本的库。

**标准库是最早的，现在已经停止维护**，它提供了对STM32微控制器的底层寄存器和外设的直接访问。

标准库的设计目的是提供高度灵活和低层次的硬件控制，以满足对性能和资源的严格要求。

使用标准库，开发人员可以直接操作寄存器来配置和控制微控制器的功能，都需要手动编写大量的底层代码。
## 5 HAL库

HAL库（Hardware Abstraction Layer）是STMicroelectronic为了提供更高级别的抽象和简化开发而引入的库。

HAL库将底层硬件操作抽象为高级函数调用。这样，开发人员可以使用更高级别的API函数来进行配置和控制微控制器的功能，而不需要直接操作底层寄存器。

**HAL库提供了一种更易用和可移植的编程模型，并减少了编写底层代码的工作量 。它还支持多种开发板和外设，提供了一致的接口，简化了代码移植和复用**

HAL库适用于大多数应用程序，尤其是中等复杂性的项目
## 6 LL库

LL库（Low-Level Library）是STMicroelectronic在HAL库基础上提供的更低级别的库。

LL库提供了对底层寄存器和外设的更直接的访问，并提供了一组更低级别的API函数。LL库保留了更多的硬件细节，为开发人员提供了更高级别的灵活性和控制

**使用LL库，开发人员可以直接编写更底层的代码，实现对微控制器和外设的精细控制。**

LL库适用于对性能和资源要求极高，以及对底层硬件控制有特殊需求的应用

## 7 HAL库的优点

- 最大可移植性
- 提供了一整套一致的中间件组件，如RTOS、USB、TCP/IP和图形等
- 通用的用户友好的API函数接口
- ST新出的芯片已经没有标准库
- HAL库已经支持STM32全线产品
## 8 总结

**目前ST公司已不对标准库进行维护了，** 按照目前的状况来说HAL为学习的主流。针对HAL库ST公司也推出了一款图形配置工具CubeMX，可通过配置自动生成初始化代码

**现在最好是使用HAL库为主，LL库在特殊情况下使用，因为这是官方主推的库，并且CubeMX确实是个很好用的工具，而里面只有HAL库和LL库**