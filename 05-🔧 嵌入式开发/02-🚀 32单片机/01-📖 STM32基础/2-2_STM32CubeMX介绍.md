---
文章标题: "[[2-2_STM32CubeMX介绍]]" 
文章作者: Dakkk
文章概要: |
  STM32CubeMX图形化配置工具的界面介绍和使用指南，包含工程创建、功能配置、时钟设置、工程管理等核心功能模块的操作说明。
tags:
- "STM32CubeMX"
- "STM32"
- "嵌入式开发"
- "图形化配置"
- "工程管理"
- "时钟配置"
- "低功耗"
- "芯片配置"
相关文章:
- "[[0_课程完整内容]]"
- "[[3_不同编程方式-寄存器、标准库、HAL库、LL库]]"
- "[[4_STM32CubeMX工程创建]]"
- "[[5_设置烧录参数，烧录固件]]"
- "[[7_GPIO输入按键]]"
文章分类: "🔧 嵌入式开发"
文章路径: "05-🔧 嵌入式开发/02-🚀 32单片机/01-📖 STM32基础/2-2_STM32CubeMX介绍.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-02-10 02:26:34
修改时间: 2025-05-27 23:05:39
---

## 1 官方使用手册

英文手册
https://www.stmcu.com.cn/Designresource/detail/user_manual/711316

中文手册
https://www.stmcu.com.cn/Designresource/detail/localization_document/710583

## 2 界面说明

**首页**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/9b3d75507577dac65108e3dd9ddcdea9.png)

**新建工程页-根据芯片型号创建工程（常用）**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/cbf735caed10abaa497efa8eafd74fe8.png)

**新建工程页-根据竞品厂家芯片创建工程**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/20ad9e675c10c95613f3dfccd1c81e8c.png)

**功能配置页面**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/e7609cacff45d70040b20ee458f10795.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/660349ba15ea5c132aac81837e52c9e8.png)

**时钟配置页**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/43ba86badbcef27f8a04037d32bb1ed9.png)

**工程设置**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/a8c0d8c0317116027fc5ea471079dc45.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/c721c1f4ea0d7f8200eff63756737fa5.png)

## 3 工具

低功耗产品待机工作时长评估，我们可以根据自己应用的运行情况，设置运行和休眠的功耗、时间，设置产品的电池电量等参数，就可以计算出理论的待机时长、平均功耗等数据

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/02/2169a699b2709e99c5c01d3631865b0b.png)
