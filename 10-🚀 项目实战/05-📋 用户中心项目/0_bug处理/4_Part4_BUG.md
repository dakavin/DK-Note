---
文章标题: "[[4_Part4_BUG]]"
文章作者: Dakkk
文章概要: |
  文章记录了前端登录按钮点击后无法跳转并报错“status undefined”的问题。原因是全局返回参数修改导致`setUserLoginState`无法读取信息。解决方案是删除登录页面上与该状态设置相关的代码。
文章标签:
  - 前端
  - Bug修复
  - 登录功能
  - JavaScript
  - 状态管理
  - API响应
  - 错误排查
  - 页面跳转
相关文章:
  - "[[0_JavaWeb概述]]"
  - "[[../../../01-📚 知识管理/05-📹 视频脚本/1_Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示)]]"
  - "[[10_※HTTP状态]]"
  - "[[10_前端工程化介绍 + ES6语法]]"
  - "[[11_Nodejs + npm]]"
文章分类: 🚀 项目实战
文章路径: 10-🚀 项目实战/05-📋 用户中心项目/0_bug处理/4_Part4_BUG.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐ 基础知识
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-17 13:50:51
---

## 问题1

问题：前端点击登录按钮后，无法自动跳转、报错 status undefined
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ad00b745599e5248f4692e29dc85657.png)

原因：修改了全局返回参数，这个setUserLoginState无法读取信息

解决：打开登录页面，直接删除这两个地方
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23e949578461dc95eb561592818f11eb.png)
