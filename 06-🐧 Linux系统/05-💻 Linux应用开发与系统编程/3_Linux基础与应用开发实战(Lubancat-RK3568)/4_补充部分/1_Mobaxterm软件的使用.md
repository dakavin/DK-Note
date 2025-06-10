---
文章标题: "[[1_Mobaxterm软件的使用]]"
文章作者: Dakkk
文章概要: |
  介绍了使用MobaXterm软件通过串口连接开发板的详细步骤，包括驱动安装、端口配置、软件设置和终端登录等操作流程。
tags:
  - MobaXterm
  - 串口连接
  - 开发板
  - 终端
  - 串口驱动
  - 嵌入式开发
  - 串口调试
  - 硬件调试
相关文章:
  - "[[../../../../05-🔧 51&32单片机开发/01-🎯 51单片机/01-📚 基础入门/_逻辑分析仪的安装]]"
  - "[[../../../../05-🔧 51&32单片机开发/01-🎯 51单片机/01-📚 基础入门/_使用工具的软件驱动]]"
  - "[[../../../../05-🔧 51&32单片机开发/02-🚀 32单片机/01-📖 STM32基础/7-1_调试]]"
  - "[[../../../../05-🔧 51&32单片机开发/01-🎯 51单片机/01-📚 基础入门/_常用函数]]"
  - "[[../../../../05-🔧 51&32单片机开发/0_课程完整内容]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/4_补充部分/1_Mobaxterm软件的使用.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 22:10:42
修改时间: 2025-05-28 00:19:51
---

## 1 串口连接开发板

也可以看[3.4 串口终端登录](../../../../02-💻%20开发环境/3_Lubancat-RK3568/1_快速使用手册/1_快速开始.md#3.4%20串口终端登录)

开发板串口终端使用的默认串口通讯参数为： **115200-N-8-1** 

下面我们在Windows系统的开发主机使用MobaXterm软件登录串口终端，使用其它系统或工具的方式类似：
1. 安装USB转串口驱动，Pro板载USB转串口和Mini板的 [USB转TTL串口线](https://detail.tmall.com/item.htm?spm=a1z10.5-b.w4011-22026361158.32.4b7c7e18f4CE7E&id=600554874281&rn=c55898db719eb30c307b1d2cc23d79b8&abbucket=18) 都使用CH340驱动， 下载地址：[http://www.wch.cn/products/CH340.html](http://www.wch.cn/products/CH340.html) 。
   FT232RL驱动下载地址：[FTDI - 驱动程式](https://www.ftdichip.cn/FTDrivers.htm)
    
2. Pro板使用USB线连接电脑与开发板的 `USB转串口`；Mini板使用 [USB转TTL串口线](https://detail.tmall.com/item.htm?spm=a1z10.5-b.w4011-22026361158.32.4b7c7e18f4CE7E&id=600554874281&rn=c55898db719eb30c307b1d2cc23d79b8&abbucket=18) 连接至开发板的 `UART TTL接口`。
    
3. 使用DC电源给开发板供电并开机。Pro板不能只使用USB供电，功率不够。
    
4. 查看端口号。开发板供电并开机后，在Windows电脑上 `右键我的电脑->属性->设备管理器的->端口` ， 设备下会新增一个 `USB-SERIAL CH340` 设备，点开查看自己电脑上该COM口的编号， 这在不同的电脑上编号是不同的，如下图所示的本例子为COM4，后面请根据自己的COM号连接
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d8378661908c5ef05a72ce2eda967b82.png)
5. 安装MobaXterm软件，在软件官网选择免费版安装即可：[https://mobaxterm.mobatek.net/download.html](https://mobaxterm.mobatek.net/download.html)
6. 打开MobaXterm软件
7. 点击菜单栏 「sessions」 –> 「new session」，即可弹出 「session setting」 对话框。 从会话对话框中可以看到，MobaXterm支持非常多的连接方式，此处我们使用串口连接方式，如下图所示:
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/821ed216e6dbb538d255ad1d0021ccad.png)
8. 把MobaXterm的串口的通讯速率配置成开发板串口终端使用的默认值，即 `115200` ， 本例子使用的 端口号为“COM4”， `注意该端口号要根据自己的实验环境进行选择` ， 即在步骤（4）中查看的端口号。如下图:
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4e26bbfd11d2f128910fe8984a36b0d7.png)
9. 选好串口号及波特率后，点击OK就完成连接了。左边标签栏会记录这次的session， 以后可以直接从标签栏打开会话窗口。如下图:
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0030cb96fbd08111bb4207b367f69257.png)
10. 如果是在开发板开机前就建立了串口终端连接，那么在开机时会看到开发板在启动时的信息输出，见下图
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/049c63eec238bbb43d8b778714bf38ac.png)
11. 如果是在开发板开机后才建立的连接，开发板之前的输出没有接收到，这时直接按几下回车即可，见下图：
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2588aaca9608880f6662149f390e912f.png)
12. 无论是以上哪种情况，开发板的启动流程执行完毕时，只要按回车后终端都会提示login， 此时终端在等待用户的输入，它需要知道我们希望以哪个用户名登录终端。 我们的开发板默认用户为：`cat`，密码：`temppwd`。所以在提示界面中输入用户密码并回车登录即可，见下图。
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d0baf9e2eee8b3122e2dcee66f057400.png)
13. 至此，我们就成功通过串口登录到开发板的终端了，接下来我们就可以使用各种命令来控制开发板。
14. 按前面的配置，在使用终端软件串口登录，当输入太长的命令时，会出现字符重叠、换行错误的情况， 非常影响使用感受，这是因为终端软件的换行长度与开发板串口终端长度不一致造成的， 可在终端软件进行如下配置强制使用相同的换行长度，开发板默认的串口终端行长度为80个字符， 也可以使用命令“stty size”进行查看。
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3b775d206c76f0f22ff5c37825c72f9b.png)
注意如果使用的是ssh连接，那么终端与开发板的行长度是自动适应的，如果强制换行长度还会多此一举。
