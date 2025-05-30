---
文章标题: "[[0_PPT内容]]" 
文章作者: Dakkk
文章概要: |
  介绍单片机在智能家居、汽车电子等产品中的应用，详细阐述了从方案设计到产品开发的完整企业级硬件开发流程。
tags:
- "单片机"
- "嵌入式开发"
- "硬件方案设计"
- "产品开发"
- "芯片选型"
- "智能硬件"
- "消费电子"
相关文章:
- "[[_常用函数]]"
- "[[0_学习资料汇总]]"
- "[[1_STM32介绍]]"
- "[[1_本章代码及文章获取]]"
- "[[2_嵌入式软件学习路线]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/01-🎯 51单片机/01-📚 基础入门/0_PPT内容.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-01-01 23:32:25
修改时间: 2025-05-27 23:05:39
---

## 1 单片机开发的产品

- 智能家居设备：如智能插座、智能灯泡、智能门锁等
- 智能穿戴设备：如智能手表、智能健康监测器等
- 电子玩具：如迷你飞行器、智能机器人等
- 汽车电子设备：如发动机控制单元、车载娱乐系统等
- 工业自动化：数据采集、测控技术
- 仪器仪表：数字示波器、数字信号源、数字万用表、感应电流表等
- 医疗设备：如血压计、血糖仪等
- 安防设备：如监控摄像头、门禁系统等

凡是与控制或简单计算有关的电子设备都可以用单片机来实现。

## 2 单片机就职方向

- 芯片原厂：ST、TI、RK、全志、NXP、MTK、展锐、海思等等
- 模组厂商：移远、广和通、中移动、中兴物联等等
- 消费电子产品：小米、华为、DJI、oppo等等
- 商业&工业产品：海康、大华、大族、优必选、讯飞等等
- 汽车电子领域：
	- 主机厂：比亚迪、广汽、上汽、吉利等
	- 供应商：马瑞利、采埃孚、博世、宁德时代、
- 家电领域：美的、海尔、格力
- 互联网厂家：阿里、腾讯、京东、头条、百度、美团都有硬件相关的事业部

## 3 设计产品硬件方案（企业）

方案设计流程：[‌⁠‬​​‌​⁠‍﻿​‬⁠​​﻿​​﻿​‌﻿﻿﻿‬⁠​‍​‍‬​​​⁠​​⁠​​​‌​​‍‌​​⁠​各类硬件方案设计经验 - 飞书云文档](https://x509p6c8to.feishu.cn/docs/doccnemvJHCIN7bKA5ifqDNnwqb)
开发流程：[智能硬件产品项目研发-ProcessOn](https://www.processon.com/diagraming/5fd4c38363768906e6d881ee)


- 确定设计标准：消费级、工业级、汽车级、军工级
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/431322884c7000619d5990c88ae2bb3d.png)

- 确定功能需求：外设、接口、频率、需求复杂度、使用场景等等
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/816a131aecb20a1012ca8a6e619f9028.png)

- 进行方案选型：8位、16位、32位、64位
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/ed0650a4e66fdbb0820f66ee90db21f4.png)

- 进行外设选型传感器、执行器、交互模块、通信模块等
- 开发调试：硬件设计、驱动开发、软件开发、联调
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/01/fa8c083cb51dbf1121a8f8efd52d6ef5.png)

- 测试认证：不同地区销售，认证需求不同，3C、CE、FCC
- 试产
- 小批
- 上市

## 4 选择芯片

[‌⁠⁠‬​‬‍​​‍﻿⁠﻿​‬​​‌​​‬​​‬‬​‬​‍‬⁠​‌​​‬​​​​​​​﻿‍‬​⁠⁠​嵌入式固件方案设计规范 - 飞书云文档](https://x509p6c8to.feishu.cn/docs/doccns6Ap2bpmPNlI8wRwcxu4rI)

- 资源获取：芯片手册、使用文档、示例源码、FAE/论坛/原厂支持群
- 环境搭建：开发板、软件环境搭建、验证程序烧录、运行
- 基础功能验证：芯片基础功能、外设功能验证，了解芯片有哪些功能，哪些资源可以使用
- 产品开发：产品系统确定、协议确定、框架搭建、功能开发
- 集成
- 联调
- 测试
- 发布版本

## 5 单片机产品模块构成

单片机产品
- 电路板
	- 单片机最小系统
		- 芯片：STC89C52
		- 电源电路：3.3V/5V
		- 晶振电路
		- 复位电路
	- 外围驱动电路
- 固件 <---- 代码编译
- 结构、外设


