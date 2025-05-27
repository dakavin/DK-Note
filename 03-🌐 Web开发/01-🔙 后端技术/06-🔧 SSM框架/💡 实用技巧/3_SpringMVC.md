---
文章标题: "[[3_SpringMVC]]" 
文章作者: Dakkk
文章概要: |
  AI未能生成概要。
tags:
  - "未分类"
相关文章: 暂无 🤷
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/3_SpringMVC.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 问题一：

**创建简单的MVC项目的时候，出现以下图片的问题**

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34da4c4402a5773be892a0ff4bb09a85.png)

解决：看自己的sdk版本，jdk版本是8，因此不能用spring6.x开头，只能用spring5以下，修改spring依赖版本后可以正常使用