---
文章标题: "[[1_Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示)]]"
文章作者: Dakkk
文章概要: |
  本文展示如何结合Obsidian的Templater插件与AI模型，自动化生成笔记元数据。通过Templater调用JavaScript脚本与AI API交互，分析笔记内容并返回结构化数据，显著提升Obsidian笔记创建和管理效率，减少手动填写耗时。
tags:
  - Obsidian
  - Templater
  - AI自动化
  - 笔记元数据
  - 知识管理
  - 效率工具
  - JavaScript
  - API调用
相关文章:
  - "[[2_【续集高能】上次的Obsidian AI脚本弱爆了？看这个全功能进化版！(Templater)]]"
  - "[[3_【AI脚本第三弹】Obsidian笔记太多？用DataviewJS一键搞定所有元数据！]]"
  - "[[../06-🛠️ 技巧/1_obsidian/0_obsidian使用的插件说明]]"
  - "[[../../06-🐧 Linux系统/04-🔌 驱动开发/01-📝 训练营笔记/2_嵌入式基础能力/6_Linux应用开发/1_学习思维导图]]"
  - "[[10_文章分类：删除功能开发]]"
文章分类: 📚 知识管理
文章路径: 01-📚 知识管理/05-📹 视频脚本/1_Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示).md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-26 01:41:54
修改时间: 2025-05-27 20:52:58
---

## 1 视频建议

**一、视频标题 (可选，选择一个你认为最合适的):**

**Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示)** (吸引初学者)

**二、视频标签 (Tags):**

- Obsidian
- Templater
- AI
- 笔记元数据
- 自动化
- 效率工具
- 知识管理
- JavaScript (可以加上，因为Templater基于JS)
- 生产力
- Obsidian教程
- Gemini

**三、视频简介 (Description):**

```
第一次做视频，有点乱，请勿介意

【第一期：Obsidian自动化初探】

还在手动填写Obsidian笔记的标题、概要、标签吗？感觉有点浪费时间？

本期视频，我将带你初步体验如何利用Obsidian强大的Templater插件，结合AI（比如我这里用的Gemini模型），实现笔记元数据的自动化初步生成！

你会看到：
✅ 如何通过Templater调用简单脚本
✅ AI如何辅助生成文章概要和标签
✅ 一个提升笔记效率的小思路演示

⚠️ 注意：本期为初步功能演示，更强大的进阶版脚本和功能将在下一期视频中揭晓！敬请期待！

🔗 相关链接/工具：
*  Obsidian
*  Templater插件

感兴趣的小伙伴，欢迎【一键三连】，私信即可获取相关脚本代码！
喜欢这个视频吗？别忘了【关注】，第一时间获取后续更精彩的内容！
如果你有任何问题或想法，欢迎在评论区留言交流！
```

**四、AI封面制作提示语**

## 2 视频脚本

### 2.1 介绍

Hello大家好，这个是我第一条视频，主要是介绍一下Obsidian中Templator插件的特殊使用
- 就是右边这里，元数据，！！ 的**ai生成**
- 我这里文章概要、标签、难度、重要性都是ai自动生成的
- 使用这个功能可以提升咱们记录笔记的效率

这里我先**演示**一下吧
- 使用ctrl+p键进入源码模式，把右边笔记的元数据先删除了
- 然后我设置的快捷键是Alt+E，可以快速插入模板（这里我使用的是阿里云的oss+picgo图床哈，这样就可以减少本地图片的内容了，先不说了，继续演示）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/419703a6c5e5e9e17e4d27f04abfa7dd.png)
- 回复一下原来的模式，一样也是ctrl+p
- 可以看到都是ai生成的，我这里使用的模型是gemini-flash-2.5的，如果使用对文本友好的模型，如claude可能会效果更好

### 2.2 Templater插件

那么介绍一下这个插件吧

**下载（需要魔法）**
- 我这里就不提供包了，很多地方都有介绍
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9e4a939b2a8dacd7c928f052b9a0cad3.png)
- 额，英文打错了，不要介意

**设置介绍**
- 我在想弄不弄，算了，直接看人家的吧
- 链接：[看这个，这个博主介绍的挺好的](https://www.bilibili.com/video/BV1c64y1W7c2/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=268c1f3b89c763db9597d10733d3c3a3)

有点学习难度，涉及到这个插件自定义的语法，想用的慎重！

**这里就介绍一下基本语法好了**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d37d3eaa8ecb262a068e585c942e44a5.png)

### 2.3 调用api

刚刚也说到了，本质就是JS+插件给的规则，那么怎么做的呢？

#### 2.3.1 基础模板结构

我直接复制视频脚本的内容了

```javascript
<%*
// AI元数据生成模板
const content = tp.file.content;
const title = tp.file.title;
// 这里将调用AI API
const metadata = await generateMetadata(content, title);
// 输出生成的元数据
%>
```
- 这里是调用的脚本
- 使用刚刚说的符号 `<%*%>`  套住我们写的js代码

```txt
//---
title: <% tp.file.title %>
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags: <% metadata.tags %>
summary: <% metadata.summary %>
keywords: <% metadata.keywords %>
//---
```
- 然后这里就会生成对应的元数据

#### 2.3.2 api调用数据

```javascript
// 这里就是调用ai的api方法了
async function generateMetadata(content, title) {
	// 提示词
    const prompt = `
    请分析以下笔记内容，生成结构化元数据：
    
    标题：${title}
    内容：${content}
    
    请以JSON格式返回：
    {
        "tags": ["标签1", "标签2", "标签3"],
        "summary": "简洁的内容摘要",
        "keywords": ["关键词1", "关键词2", "关键词3"],
        "category": "主要分类",
        "difficulty": "难度等级",
        "priority": "优先级"
    }
    `;
    
    // API调用逻辑
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            // 这里写上apikey好了
            'Authorization': `Bearer ${API_KEY}`
        },
        body: JSON.stringify({
	        // 模型类型
            model: 'gpt-3.5-turbo',
            messages: [{ role: 'user', content: prompt }],
            temperature: 0.3
        })
    });
    
    const data = await response.json();
    return JSON.parse(data.choices[0].message.content);
}
```

#### 2.3.3 完整的代码演示

这里演示一下我的代码吧，不过有api key，算了待会删掉换一个，俺可公益不了

展示完毕

### 2.4 后期想法

其实可以做很多拓展的

#### 2.4.1 类似的插件（Text Generator）
我这个做法好像和 Text Generator插件有点像
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6322ceb806b4bfce5d7ddda36ae8412f.png)

- 害，待会还要注销掉这个api

#### 2.4.2 后续拓展

**1、自定义提示词，修改其他内容啊**

**2、多个笔记批量处理，生成元数据**

**3、还可以检查文章质量啥的**

### 2.5 总结

第一次做视频有点乱，这个需要有一定基础的，容易劝退，先这样吧

不过就是好用！！！

再来一个

结束，谢谢大家观看哈！😘

