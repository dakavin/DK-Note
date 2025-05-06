
## 1 创建工程

**第一步：打开STM32CubeMX，选择File -> New Project**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/85a492c21c48f878c06feec4a4c52b1e.png)
- 如果首次使用，可能会自动下载一些依赖包，可以等待下载完成
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/cb85da2910e70406c72bbef7198c31f6.png)
**第二步：选择对应芯片 MCU/MPU Selector -> 输入“STM32F103RC” -> 选择搜索到的芯片“STM32F103RCTx” -> Start Project** `选择芯片的方式来创建工程`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/257c7db0169869dff6bca09643e4fcf0.png)

- 上方有芯片对应的各项手册！

点击Start Project后，等待创建完成即可看到下方界面
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/0e1636ba887f37297bb775ff409a10c3.png)
## 2 设置时钟源

**芯片要运行起来，必须要有时钟源**，在STM32中，我们可以选择外部或内部时钟作为芯片时钟源

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/6cafcebf7fc109365c79960a978486f1.png)

这个图中，我先只关注LSI HSI 、LSE HSE和HCLK
- LSI：Low-Speed Internal clock，低速内部时钟
	- 通常是一个低功耗的RC振荡器，频率较低（如32 kHz），用于看门狗或RTC等低功耗场景
- HSI：High-Speed Internal clock，高速内部时钟
	- 通常是一个RC振荡器，频率较高（如8 MHz或16 MHz），用于系统主时钟或备用时钟源
- LSE：Low-Speed External clock，低速外部时钟
	- 通常是一个外部晶体振荡器，频率较低（如32.768 kHz），用于RTC等需要高精度的场景
- HSE：High-Speed External clock，高速外部时钟
	- 通常是一个外部晶体振荡器或时钟源，频率较高（如4 MHz到26 MHz），用于系统主时钟
- HCLK：**H**ost **C**lock 或 **H**igh-speed **C**lock
	- HCLK 通常由 SYSCLK 分频得到，分频系数可通过寄存器配置
	- 一般用于CPU的运行时钟、控制 Flash 和 SRAM 的访问速度、为 GPIO、定时器、USART 等外设提供时钟
### 2.1 内部时钟 LSI HSI

STM32 MCU内部自带RC振荡电路，其内部时钟就是RC振荡器产生的。

但是RC振荡器精度远低于晶振，且容易受到温度的影响
### 2.2 外部时钟 LSE HSE

外部时钟一般有两种接法
- 外部接`有源晶振`或`其他直接时钟`输入源：BYPASS Clock Source模式（旁路时钟源）
- 外部接`无源晶振`：Crystal/Ceramic Resonator模式（晶体/陶瓷晶振）
### 2.3 设置外部时钟

如果需要选择外部时钟，在**RCC界面配置HSE和LSE即可**
- HSE高速时钟设置为外部无源晶振
- LSE为低速时钟，可以不设置，因为我们板卡没有接低速晶振，当用到RTC，并且对精度有要求才加

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/302b496a9cb25ccf118dc91da7143299.png)

同时配置芯片运行时钟频率，这里我们设置HCLK为72Mhz，按回车后，会自动生成其它配置
- 外部时钟HSE 8MHz
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/5ac37651679f61dad558d538bdf05115.png)
- PLL倍频9倍（`8*9=72`）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/60a7398774439d0ba5e1ebb8585b04a7.png)
- 系统时钟来源选择为PLLCLK

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/80619218b889375835479d115f459cb5.png)


### 2.4 设置烧录调试方式

STM32作为控制芯片时，程序烧写非常关键的一步，而烧写接口的稳定性及必要时的简洁性就显得尤为重要。

目前常用的两种接口是JTAG和SWD，而我们板卡使用SWD接口作为调试接口，SWD（Serial Wire Debug 串行调试），接口仅需4个，分别是VCC、GND、SWIO（双向数据接口）、SWCLK（时钟）

SWD的优点
- 高速模式更可靠；
- 接线少，占用的GPIO资源少；
- SWD搭配ST-Link仿真器使用，相比于JTAG的J-Link，更便宜

打开System Core选项卡，单击SYS选项
```C
SWD模式就选择serial Wire Debug
JTAG模式就选择JTAG，4pin和5pin的区别多了一个复位引脚
stlink调试就是SW模式，jlink调试就是JTAG模式
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/d00b778f4591d0742d82e9cfe2b1ec6e.png)


后续就可以使用Jlink烧录器进行烧录了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/8d4312bbac91671b7929a8f86d4e9cca.png)

### 2.5 设置工程

点击顶部工程管理,设置工程名称,设置工程保存路径，选择开发环境,如果使用keil开发,则选择MDK-ARM
- 注意:**不管工程名称还是路径都不要有中文，否则后面编译文件会出错**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/283bf2df1729ffbfab44b047bd9feac9.png)

- Project Name：工程名称
- Project Location：点击后面的"Browse"选择你想要将生成的工程保存到哪个目录里面。
- Application Structure：应用程序结构
	- Basic：是基础的结构，一般不包含中间件（RTOS、文件系统、USB设备等）
	- Advanced：相反就是包含中间件，一般针对相对复杂一点的工程，选择这个，后续方便扩展
- Toolchain/IDE：根据你用的编译软件进行选择 使用KEIL就选择keil的对应版本

### 2.6 源码输出设置

- 点击左侧Code Generator.选中仅复制需要的库,否则生成的工程会很大
- 选择将外设配置为单独的.c和.h文件

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/a56804e2296b1faedb5d10f5cb9ef409.png)

解释：
- copy all used libraries into the project folder：
	- `复制所有库文件`（不管工程需要用到还是没用到）到生成的工程目录中
- Copy only the necessary library files：
	- `只复制必要的库文件`。这个相比上一个减少了很多文件。比如你没有使用CAN、SPI…等外设，就不会拷贝相关库文件到你工程下。
- Add necessary library files as reference in the toolchain project configuration file ：
	- `在工具链项目配置文件中添加必要的库文件作为参考`。这里没有复制HAL库文件，只添加了必要文件（如main.c）。相比上面，没有Drivers相关文件。

- Generate peripheral initialization as a pair of’.c/.h’ files per peripheral：
	- `每个外设生成独立的.C .H文件，方便独立管理`。
	- 不勾：所有初始化代码都生成在main.c 
	- 勾选：初始化代码生成在对应的外设文件。 如UART初始化代码生成在uart.c中。
- Backup previously generated files when re-generating：
	- `在重新生成时备份以前生成的文件`。重新生成代码时，会在相关目录中生成一个Backup文件夹，将之前源文件拷贝到其中。
- keep user code when re-generating：
	- `重新生成代码时，保留用户代码(前提是代码写在规定的位置`。也就是生成工程文件中的BEGIN和END之间。否则同样会删除。后面会根据生成的工程进行说明)
- delete previously generated files when not re-generated：
	- `删除以前生成但现在没有选择生成的文件` 比如：之前生成了led.c，现在重新配置没有led.c，则会删除之前的led.c文件。(此功能根据自身要求进行取舍)

### 2.7 生成工程

点击右上角的GENERATE CODE，就可以生成工程
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/caf233351f45ba0907f5a9760a1edb98.png)

最后点击Open Project，就可以用你已经安装Keil MDK打开工程
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/f0299ecc326d27a40dc3293e086568a5.png)

点击Build，如果最终编译完成没有报错误，就完成STM32CubeMX的搭建啦。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/dc220f77757a5051895bb4ff7363908d.png)

### 2.8 bug处理

问题描述：使用STM32CubeMX创建工程，卡在了Copying libraries files
检查一下：工程所在路径有没有中文字符，或者一些特殊符号，去掉即可