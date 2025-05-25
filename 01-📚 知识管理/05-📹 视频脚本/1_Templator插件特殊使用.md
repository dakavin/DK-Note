---
文章标题: "[[1_Templator插件特殊使用]]"
文章作者: Dakkk
文章概要: |
  文章演示如何在Obsidian中利用Templater插件，结合AI API自动生成笔记元数据（概要、标签、关键词），显著提升笔记效率。文中提供了核心JavaScript代码及实现演示。
文章标签:
  - Obsidian
  - Templater
  - AI
  - 自动化
  - 元数据
  - JavaScript
  - API调用
  - 笔记管理
相关文章:
  - "[[0_obsidian使用的插件说明]]"
文章分类: 📚 知识管理
文章路径: 01-📚 知识管理/05-📹 视频脚本/1_Templator插件特殊使用.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-26 01:41:54
修改时间: 2025-05-26 07:12:34
---

这里用了Liner插件帮我把修改时间更新了哈，哈哈哈哈哈
我说刚刚右边怎么会有提示

## 1 介绍

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

## 2 Templater插件

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

## 3 调用api

刚刚也说到了，本质就是JS+插件给的规则，那么怎么做的呢？

### 3.1 基础模板结构

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

### 3.2 api调用数据

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

### 3.3 完整的代码演示

这里演示一下我的代码吧，不过有api key，算了待会删掉换一个，俺可公益不了

展示完毕

## 4 后期想法

其实可以做很多拓展的

### 4.1 类似的插件（Text Generator）
我这个做法好像和 Text Generator插件有点像
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6322ceb806b4bfce5d7ddda36ae8412f.png)

- 害，待会还要注销掉这个api

### 4.2 后续拓展

**1、自定义提示词，修改其他内容啊**

**2、多个笔记批量处理，生成元数据**

**3、还可以检查文章质量啥的**

## 5 总结

第一次做视频有点乱，这个需要有一定基础的，容易劝退，先这样吧

不过就是好用！！！

再来一个

结束，谢谢大家观看哈！😘

