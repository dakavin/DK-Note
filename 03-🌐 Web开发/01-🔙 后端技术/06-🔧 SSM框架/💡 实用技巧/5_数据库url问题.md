---
文章标题: "[[5_数据库url问题]]" 
文章作者: Dakkk
文章概要: |
  AI未能生成概要。
tags:
  - "未分类"
相关文章: 暂无 🤷
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/5_数据库url问题.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---


问题情况 : javax.net.ssl.SSLException: closing inbound before receiving peer‘s close_notify异常


解决办法 : 在数据库连接配置文件中，配置连接数据库的url时，加上useSSL=false。
