---
文章标题: "[[6_STM32CubeMX实现HAL点灯]]" 
文章作者: Dakkk
文章概要: |
  介绍使用STM32CubeMX配置GPIO控制LED，从原理图分析、GPIO工作原理到CubeMX配置和HAL库函数使用的完整入门教程。
tags:
- "STM32"
- "STM32CubeMX"
- "GPIO"
- "HAL库"
- "LED控制"
- "嵌入式开发"
- "推挽输出"
- "开漏输出"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[2-1_传统手动创建工程方法]]"
- "[[2-2_STM32CubeMX介绍]]"
- "[[3_不同编程方式-寄存器、标准库、HAL库、LL库]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:32:59
---
### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)





### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)




### 分析说明

这是一篇STM32入门级教程"
- "主要介绍如何使用STM32CubeMX图形化工具配置GPIO来控制LED。文章从原理图分析开始"
- "详细讲解了GPIO的工作原理（包括保护二极管、上下拉电阻、推挽/开漏输出模式）"
- "然后演示了CubeMX的配置过程和HAL库函数的使用方法。内容结构清晰"
- "适合STM32初学者学习GPIO基础操作和HAL库编程。"
相关文章:
- "[[7_GPIO输入按键]]"
- "[[0_课程完整内容]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/06_📕设备树中断的属性描述 (最常用)]]"
- "[[../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/07_设备树中 GPIO 相关属性 (定义)]]"
- "[[1_本章代码及文章获取]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/6_STM32CubeMX实现HAL点灯.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 原理图说明

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ed3ba3500601be85d07b79893deca71f.png)

LED外接电源，管脚控制拉低（管脚是输出的），形成电压差，那么LED就可以亮
反之，管脚拉高，输出高电平，LED就是灭的
- PA4 ---> LED3
- PA5 ---> LED2
- PA6 ---> LED1
## 2 新建工程

打开我们之前创建的ProjectTemplates，在右侧的PinOut View中找到PA6
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e5b1ca93cffa06ba757d3fc1bc1fb6d0.png)

设置PA6为输出模式，这里我们可以看到PA6这个IO是支持多种功能的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/7c070e0ef5858595e5d2ddede91141d2.png)

例如ADC、SPI、TIM、GPIO这些功能，与芯片手册是一一对应的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5e70177eea86efab7ae910d1e7696fd2.png)

## 3 GPIO输出参数说明

### 3.1 GPIO原理讲解

先了解原理部分：在芯片手册中，可以看到GPIO部分的结构图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0873e64fa3ec1cc63815eb404b238a87.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/511014e4925846c7f660841ff037d393.png)

**保护二极管：**
- 图中的保护方式是`钳位二极管保护`
	- 当输入电压低于VSS的时候，下方二极管导通，将过低的电压泄放到地
	- 当输入电压高于VDD_FT（一般是系统电压）的时候，上方的二极管导通，将过高的电压泄放到电源轨
- IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入，防止不正常电压引入芯片烧毁
- 当引脚电压高于VDD_FT时，上方的二极管导通
- 当引脚电压低于VSS时，下方的二极管导通

**上拉、下拉电阻：**
- 控制引脚默认状态的电压，有三种状态（上拉、下拉、浮空-模拟输入）
- 开启上拉的时候引脚默认电压为高电平
- 开启下拉的时候引脚默认电压为低电平
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/74154a2b944b8dd12ab83c20cfe93375.png)


**输出模式：**
- 推挽输出模式下，P-MOS和N-MOS都正常工作
- 开漏输出模式下，只有下面的N-MOS工作，即GPIO无法输出高电平，需要外部上拉电阻
- P-MOS和N-MOS是两个二极管

**输出控制类似为一个非门**

**推挽输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，P-MOS关闭，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，P-MOS打开，IO输出VDD

**开漏输出模式：**
- 输出寄存器为0时，经过输出控制模块，转为1，此时N-MOS打开，IO输出低
- 输出寄存器为1时，经过输出控制模块，转为0，此时N-MOS关闭，IO为高阻态

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2790b83e2cfed19b250023af6d5e3d3b.png)

### 3.2 GPIO使用讲解

`GPIO output level:`
- 上电后的默认输出电平，有low和high两个选项
- 因为LED阴极连接单片机引脚，设置高电平时，上电后LED是熄灭状态的

`GPIO mode:`
- 推挽输出和开漏输出两种模式选择
- 区别主要在于：
	- 推挽输出1代表VCC，0代表GND
	- 开漏输出1代表高阻态，0代表GND
	- 简单总结--->如果不需要VCC的输出电压，或者是线与功能（I2C、SMBUS类总线占用原理），则选择开漏输出，否则选择推挽输出
- 分析一波：
	- 当开漏输出1，IO为高阻态，IO电压由外部上拉电阻决定，我们可以在外部接上拉电阻，上拉电源为5V，实现IO控制5V输出的效果
	- 若有很多个开漏模式引脚连接到一起时，只有所有引脚都输出高阻态，才由上拉电阻提供高电平
	- 若其中一个引脚为低电平，那线路相当于接地，使得整个线路都为低电平，这也是I2C、SMBus等总线判断总线占用状态的原理（某个从机要发信号时，先检测IO状态，高电平可以发，低电平说明有设备在占用总线，就不能发）
- 所以，对于LED来说，我们选择推免输出模式即可

`GPIO Pull-up / Pull-down:`
- GPIO上下拉设置
- 当GPIO被配置为推挽输出时，上下拉可以控制输出电平的稳定性
- 当GPIO被配置为开漏输出时，上下拉可以控制输出电平的状态。在没有上下拉的情况下，开漏输出的GPIO会处于高阻态，输出电平由外部上下拉决定。通过配置上拉电阻可以使GPIO处于高电平状态，通过配置下拉电阻可以使GPIO处于低电平状态
- **因为我们选择的是推挽输出，所以设置上拉即可**

`Maxinum output speed:`
- 最大的输出速度，一般选择low就可以啦
- 配置高速：
	- 输出频率高，噪音大，功耗高
	- 如：配置SPI、CAN引脚，要求通信稳定，速度过低会造成输出失真，通信异常
- 配置低速：
	- 输出频率低，噪音小，功耗低，提高系统EMI（电磁干扰）效能
	- 低功耗产品会选择低，实际应用中，满足要求的情况下选低不选高
- **这里我们选择低速即可**

`User Label:`
- 用户标签，给引脚设置名称，如果遇到标签未生成，可以关闭所有软件重新生成
- **这里我们设置为LED1，可以方便后续编写代码**

然后设置硬件对应的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0f06a7fa321f1dbde72bc4d3361a2591.png)

## 4 代码生成和理解

然后点击生成代码，重新打开工程，打开main.c，这里就是整个工程的入口了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/be8df0e7bed4206e99f434960cff0432.png)

`HAL_Init();` ：用于初始化HAL库相关配置

`SystemClock_Config();` ：用于初始化芯片时钟配置

`MX_GPIO_Init();` ：用于初始化GPIO配置

跳转MX_GPIO_Init(),在这里，我们可以看到里面的配置是由CUBEMX中的配置生成的
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/ce00de0af335e04e9aee25c2f3064cf4.png)
## 5 讲解和编写代码

### 5.1 接口讲解

主要是讲解MX_GPIO_Init()函数中的调用

- HAL_GPIO_WritePin函数
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d4065733c4f8163c6247f6cee84ade81.png)
```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
/*
GPIOx = GPIOA / LED1_GPIO_Port
GPIO_PIN = GPIO_PIN_6 / LED1_Pin
PinState = GPIO_PIN_SET(对应高电平，RESET为低电平)
对应初始化时，设置GPIOA6输出高电平
*/
```

- 结构体GPIO_InitTypeDef
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/08fc67137d18c037e2632f56f4790c81.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/00e548b671824bf61a2d7e7b1303ca45.png)
- HAL_GPIO_Init函数
	-  最后初始化GPIOA GPIO_PIN_6为推挽输出，上拉，速度为低速


**如何控制这个IO口进行输出呢？**
- 选择HAL_GPIO_Init函数，找到其定义的c文件
- 然后在c文件中，右键跳转到h文件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8450dc28dddc2dfa04da72d9279f6c1d.png)
- 往下拉，就可以看到定义的一些函数（和GPIO操作相关的所有函数）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/39d176f6282798fdeb5821d199f8d14e.png)
**注意：**
- 在gpio.c文件中HAL_GPIO_WritePin中参数没有更换为对应的LED1宏定义（在main.h文件中有）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e3b02013be4e680de85416381aaf4e42.png)
### 5.2 代码编写

**控制GPIO输出**
- 使用函数HAL_GPIO_WritePin，来设置Pin输出高低电平
- `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_RESET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(LED1_GPIO_Port,LED1_Pin,GPIO_PIN_SET);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

- 当然，也可以使用HAL_GPIO_TogglePin函数控制输出电平翻转
- `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`
```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

最后编译、烧录和运行，就可以看到效果啦

## 6 多个LED控制

如果希望所有灯同时闪烁，则同时添加多个IO
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6a14207ef00eb8871fc1b0b214ca0617.png)

```c
int main(void)
{
  // ....
  while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
		HAL_GPIO_TogglePin(LED2_GPIO_Port,LED2_Pin);
		HAL_GPIO_TogglePin(LED3_GPIO_Port,LED3_Pin);
		HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```
## 7 补充：复用输出功能

在gpio的h文件中，我们可以看到输入（GPIO_MODE_INPUT）、两种输出功能（GPIO_MODE_OUTPUT_PP、GPIO_MODE_OUTPUT_OD），那么还有其他的功能是什么呢？
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/625a70ea851362e0b47c77776d46a10e.png)

- 其他的是复用功能，我们可以查看文档，复用功能的配置
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/595d65c660db206d9af493780c6f2c48.png)

复用功能输出：使用该功能的时候，**上面输出对应的连接点就会被断开**

来自片上外设：当我们想要这个引脚作为I2C或者SPI的接口，这时I2C或SPI就是片上外设，这个时候配置该IO功能的时候，就要**必须**用复用功能进行配置
- 可以理解为GPIO口被用作第二功能时的配置情况，即并非作为通用IO口使用

**两种模式进行复用**
- 复用开漏输出（I2C）
	- 复用开漏输出和开漏输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器
- 复用推挽输出（SPI、UART、CAN）
	- 复用推挽输出和推挽输出原理一样，区别在于输出控制源不同，前者是通过外设，后者是通过CPU写寄存器

**可以在芯片手册，看到复用功能时，对应的表格**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c9e37c5c9fb3492d0f1e0317b7ccef22.png)



