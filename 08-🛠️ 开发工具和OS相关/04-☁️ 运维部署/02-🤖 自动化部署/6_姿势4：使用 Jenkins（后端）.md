---
文章标题: "[[6_姿势4：使用 Jenkins（后端）]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了使用Jenkins进行后端开发的前置条件，主要包括本地安装Docker，并指引读者关闭`home`、`usercenter-frontend`和`usercenter-backend`三个仓库的Actions工作流。
tags:
- "Jenkins"
- "Docker"
- "CI/CD"
- "后端开发"
- "GitHub Actions"
- "持续集成"
相关文章:
- "[[8_补充：本地使用Docker安装Jenkins]]"
- "[[0_导论]]"
- "[[0_参考文章索引]]"
- "[[1_📕标签模块接口分析]]"
- "[[1_📕分类管理模块接口分析]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/04-☁️ 运维部署/02-🤖 自动化部署/6_姿势4：使用 Jenkins（后端）.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 00:49:23
---

## 1 前置条件

首先，**本地安装Docker，请参考《9_补充：安装Docker（本地和Linux）》**

然后，我们需要关闭之前 home 、 usercenter-frontend 、 usercenter-backend 这三个仓库的Actions工作流

按照下图，即可关闭仓库的Actions工作流

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/94694deed5977484b2898746011f2cc0.png)

