---
文章标题: "[[2_搭建多模块工程（通过 Maven Archetype）]]" 
文章作者: Dakkk
文章概要: |
  针对IDEA Spring Initializr不再支持Java 8创建Spring Boot项目的问题，本文介绍了如何利用Maven Archetype搭建Spring Boot多模块工程。此方法为在Java 8环境下构建复杂项目结构提供了一种实用替代方案。
文章标签:
- "Maven Archetype"
- "多模块工程"
- "Spring Boot"
- "项目搭建"
- "Java 8"
- "IDEA"
- "Spring Initializr"
相关文章:
- "[[1_idea创建不了spring2.X版本，无法使用JDK8]]"
- "[[1_搭建多模块工程（Spring Initializr）]]"
- "[[1_SpringBoot项目部署（Docker）]]"
- "[[0_参考文章索引]]"
- "[[0_导读]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/2_搭建多模块工程（通过 Maven Archetype）.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-17 00:48:34
修改时间: 2024-04-17 08:59:18
---

本小节为**补充内容**，原因是`IDEA` 新建项目时，如果通过 `Spring Initializr` 来创建 `Spring Boot` , 已经无法选择 `Java 8` 版本，通过上小节的教程，不知道该如何创建 `Spring Boot` 多模块工程。

本小节中，通过 `Maven Archetype` 来创建 `Spring Boot` 多模块工程。

参考链接：https://www.quanxiaoha.com/column/10161.html