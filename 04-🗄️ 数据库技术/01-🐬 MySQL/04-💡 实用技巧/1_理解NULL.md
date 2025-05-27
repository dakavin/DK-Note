---
文章标题: "[[1_理解NULL]]" 
文章作者: Dakkk
文章概要: |
  MySQL中NULL值的比较操作详解，包括等号、不等号、安全等号和IS NULL/IS NOT NULL的使用区别和返回结果
tags:
- "MySQL"
- "NULL"
- "数据库"
- "比较操作符"
- "SQL"
- "三值逻辑"
- "安全等号"
- "IS NULL"
相关文章:
- "[[0_手撕SQL]]"
- "[[1_📕博客设置模块功能分析、表设计]]"
- "[[1_📕文章管理模块功能分析、表设计]]"
- "[[1_数据库基础知识总结]]"
- "[[1_JDBC概述]]"
文章分类: "🗄️ 数据库技术"
文章路径: "04-🗄️ 数据库技术/01-🐬 MySQL/04-💡 实用技巧/1_理解NULL.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:09:58
---

- 理解 = 、<>、 <=>、 IS NULL、 IS NOT NULL
```mysql
SELECT 10 = 10		# 1
SELECT 10 = null		# null
SELECT 10 <> null		# null
SELECT  10 <=> null 	# 0
SELECT null = null		# null
SELECT null IS NULL		# 1
SELECT 10 IS NOT NULL  # 1
SELECT 10 IS NULL		# 0
SELECT 10 != null		# null
```
- 总结对于有null参与的比较
	- <=>、 IS NULL、 IS NOT NULL ，才能返回正确的值（1/0）
	- = 、<> ，直接返回的是null