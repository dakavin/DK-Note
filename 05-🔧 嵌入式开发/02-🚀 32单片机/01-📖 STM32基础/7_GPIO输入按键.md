---
文章标题: "[[7_GPIO输入按键]]"
文章作者: Dakkk
文章概要: |
  介绍STM32 GPIO输入模式配置，通过CubeMX设置按键输入引脚，分析上拉下拉模式选择，实现按键控制LED功能的完整开发流程。
tags:
  - STM32
  - GPIO
  - 按键输入
  - CubeMX
  - 上拉下拉
  - LED控制
  - 嵌入式开发
  - HAL库
相关文章:
  - "[[0_课程完整内容]]"
  - "[[8_外部中断（重点）]]"
  - "[[9_定时器]]"
  - "[[../../../06-🐧 Linux系统/07-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/06_📕设备树中断的属性描述 (最常用)]]"
  - "[[../../../06-🐧 Linux系统/07-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/07_设备树中 GPIO 相关属性 (定义)]]"
文章分类: 🔧 嵌入式开发
文章路径: 05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/7_GPIO输入按键.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 查看原理图，并设置GPIO

根据原理图，找到KEY1对应的PC3

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/80a242ba25c4e372afc1a33bdf161cc9.png)

在51的时候，我们已经了解了按键
- 输入
- 按下的时候，输入0，没有按的时候，输入1
**所以，我们可以读取按键，对应的GPIO状态，来实现按键的功能**

找到CuBeMX中的PC3，设置为GPIO_Input，修改引脚名称为KEY1
- 也可以在GPIO配置属性中修改
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/df8fb16276a03925bc3db80385bd73fb.png)

- 引脚模式，默认为输入模式，不可更改
- 开启引脚外部浮空/上拉/下拉

**模式的作用：（外部没有接其他外设的时候）**
- 浮空模式：IO会变成高阻态，IO电平由外部电平决定
- 上拉模式：在无外部信号时，IO电平是高电平
- 下拉模式：在无外部信号时，IO电平是低电平

**如何选择？**
- 根据原理图设计选择
- 如果外部有上下拉，可以选择浮空
- 如果外部没有上下拉，内部就选择上下拉（不能选浮空，不然IO不稳定），钳住IO电平，让IO信号更稳定
	- 外部有效信号是高，选择下拉
	- 外部有效信号是低，选择上拉（假设外部是按键，在按下的时候，信号是低电平有效）
- **因为我们KEY的外部是由上拉电阻，所以选择浮空即可**

**TTL肖特基触发器：信号经过触发器时，模拟信号转化为0和1的数字信号**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/bded0c888a85dcacd1bd1bfd2ccee0dd.png)

**最后，PC3这个引脚的设置如下图：**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/997c93549ac551b2f8900a2d1b5f817f.png)

## 2 代码编写

**生成完项目代码后，可以看看MX_GPIO_init函数的内容，是否有生成的代码**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/534ed0dc6644e74a7dba61c0e988a369.png)

**此时，如果我们需要读取IO的值，如何读取呢**
- 还是找到gpio对应的h文件，然后找到相关函数的定义
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/e72ded4eafc60c7298522aa6e7898684.png)
- 所以我们可以实现读取按键是否按下，设置LED亮灭功能

```c
while (1)
{
	/* USER CODE END WHILE */
	//判断按键KEY是否被按下
	if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin) == 0){
		//延时10ms消除按键抖动
		HAL_Delay(10);
		//再次判断按键KEY是否依然被按下
		if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin) == 0){
			//对LED引脚进行取反操作
			HAL_GPIO_TogglePin(LED1_GPIO_Port,LED1_Pin);
			//等待按键抬起
			while(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin) == 0);
		}
		
	}
	/* USER CODE BEGIN 3 */
}
```

## 3 补充：模拟输入

在我们需要读取模拟量时，例如ADC或DAC的功能，就需要把IO设置为模拟数据，后续ADC章节讲解
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/96d75985583fa6c708e4d6824d5990a7.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/cea53d44a361cefa0f42568fdf3c4d4c.png)

简单总结下这两节课学习的输出和输入功能，我们就可以知道什么时候用哪种模式：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/03/1f2898bb8d77f608a1890816eed0e35c.png)

疑惑：为什么没有复用输入功能的配置？

因为复用输入是直接连接的，没有可配置的地方，也不需要配置。