---
文章标题: "[[6_jdbc.properties 文件问题]]" 
文章作者: Dakkk
文章概要: |
  AI未能生成概要。
tags:
  - "未分类"
相关文章: 暂无 🤷
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/6_jdbc.properties 文件问题.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---


问题情况:
```shell
java.io.FileNotFoundException: Could not open ServletContext resource [/jdbc.properties]
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/15d91f36be697131ac2a55ae46597928.png)
解决办法:
- 因为是使用注解开发 , 所以在使用@PropertySource注解的时候 , 需要加上 `classpath:`

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7149685d4660e96bb03dca14772401ef.png)
