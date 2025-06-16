---
文章标题: "[[7_搭建本地ai知识库]]"
文章作者: Dakkk
文章概要: |
  文章详细指导如何在本地搭建AI知识库，核心是利用Ollama下载并运行Deepseek等大模型及bge-m3嵌入模型。随后，配置环境变量，并通过Obsidian的Copilot和Smart Composer插件实现与本地AI的联动，提升个人知识管理能力。
文章标签:
  - 本地AI
  - Ollama
  - Obsidian
  - 大语言模型
  - 知识库
  - Deepseek
  - 嵌入模型
  - AI助手集成
相关文章:
  - "[[../../../01-📚 知识管理/05-📹 视频脚本/1_Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示)]]"
  - "[[2_图床搭建及使用]]"
  - "[[0_obsidian使用的插件说明]]"
  - "[[1_使用editing-toolbar插件卡顿]]"
  - "[[2_Obsidian目录相关]]"
文章分类: 🔧 系统配置
文章路径: 13-🔧 系统配置/05-🛠️ Obsidian技巧/7_搭建本地ai知识库.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-24 22:00:57
修改时间: 2025-05-24 22:01:49
---

参考文章：[‌⁠​​﻿​‬​​⁠﻿‌​​‬​​﻿​⁠‌﻿⁠​​⁠​​​​⁠‬‌​​​​﻿​​​‌‍⁠​​​⁠​‬本地部署Deepseek+Obsidian 教程文档-Xuan酱-0414 - 飞书云文档](https://kow0q7t873.feishu.cn/docx/YZKkdTxDGob5t2xSxJ1cBjqBnqf)
## 1 本地ai部署

### 1.1 下载安装 Ollama

- 官网：https://ollama.com/download
- 下载 Ollama 后，解压缩然后安装程序

### 1.2 下载模型（以 deepseek-r1:14b 为例，建议根据自己点电脑配置选择）

1. Windows 用按下Win +R快捷键，输入CMD打开命令提示符，Mac 用户打开终端。
2. 在命令窗口中输入代码 “ollama pull deepseek-r1:14b” 然后回车。
3. 这个时候就会自动下载这个模型，会看到类似这样的进度，等进度条走到 100%，下载时间取决于你的网速
	```shell
	pulling manifest
	pulling aabd4debf0c8... 100% ▕████████████████▏ 9.8GB
	pulling 369ca498f347... 100% ▕████████████████▏ 387 B
	verifying sha256 digest
	writing manifest
	success
	```
4. 进图条走完后，可以直接与这个大模型对话。
5. 下载bge-m3同理，命令窗口输入“ollama pull bge-m3”，然后回车
6. 输入“/bye”退出模型。
7. 安装嵌入模型：在命令窗口输入命令 “ollama pull bge-m3” 回车，然后等待安装完成
8. 下载完成后，可以输入“ollama list”再次检查已安装的模型：
9. 你应该会看到：
	```Plain
	NAME            ID              SIZE    MODIFIED
	deepseek-r1:14b  xxxxxxxxxxxx    9.8 GB  Just now
	bge-m3          xxxxxxxxxxxx    1.5 GB  1 day ago
	```
	- 确认 deepseek-r1:14b、bge-m3 已经出现在列表中。
10. 再输入 “/bye” 命令，退出安装过程，关闭命令提示符

### 1.3 配置环境变量

再次打开命令提示符窗口，使用命令 OLLAMA_ORIGINS=app://obsidian.md* ollama serve 
重新启动 Ollama（记得重启后再去配置Copilot）
或者手动：
1. 在电脑设置中搜索并打开编辑系统环境变量。
2. 点击右下角环境变量-系统变量-新建，变量名填 OLLAMA_ORIGINS，变量值填 app://obsidian. md* ollama serve，填完之后点确定保存即可

## 2 远程API获取

SiliconFlow API：https://cloud.siliconflow.cn/
Deepseek API：https://platform.deepseek.com/api_keys
## 3 Obsidian联动

### 3.1 安装插件

安装如下插件，需要魔法
- Copilot
- Smart Composer
### 3.2 本地ai设置

1. 进入Obsidian，在插件市场安装Copilot、Smart Composer插件并启用 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8123209f2079e24ff1fcc949a6a347d0.png)

2. 打开Copilot插件设置页面，在模型选项卡按照以下内容填写。![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c7902193debbd3b63ae18471799cddc0.png)

3. 在对话窗口切换对话模型，就可以使用了。![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/443e6cca55d8cea1a3f7d49863bdf90d.png)

4. 使用同样的方法，安装嵌入模型，按照以下步骤安装，模型名称为“bge-m3”![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/334eb896557e0c5e7f0bbd80c9c4b72e.png)
5. Smart Composer同样的操作。

### 3.3 远程API设置

和本地设置一样，不做详细的说明
## 

