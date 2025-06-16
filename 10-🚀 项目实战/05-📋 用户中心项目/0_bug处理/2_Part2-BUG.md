---
文章标题: "[[2_Part2-BUG]]"
文章作者: Dakkk
文章概要: |
  文章列举并解决了四个常见的Java开发问题：Mybatis-plus版本兼容性、数据库连接配置错误、IDEA插件失效及序列化ID生成问题，提供了直接的解决方案，帮助开发者快速排查并解决日常开发中的常见故障。
文章标签:
  - Java开发
  - SpringBoot
  - Mybatis-plus
  - IDEA
  - 常见问题
  - 故障排除
相关文章:
  - "[[../../../08-🛠️ 开发工具/02-🔧 版本控制/04-💡 实用技巧/0_Git问题解决]]"
  - "[[8_Mybatisplus]]"
  - "[[_使用工具的软件驱动]]"
  - "[[0_导读]]"
  - "[[0_导读]]"
文章分类: 🚀 项目实战
文章路径: 10-🚀 项目实战/05-📋 用户中心项目/0_bug处理/2_Part2-BUG.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-17 13:55:23
---

## 问题1

问题：Java8 + SpringBoot2 + Mybatis-plus3.5.3 ，mapper类加了@Mapper注解，spring配置类加了@MapperScan注解，整合Mybatis-plus出现如下报错
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92db0e4aedc0b3751154224780996b66.png)

原因：Mybatis-plus版本太新

解决：切换至使用的人 较多的版本 3.5.3.1即可解决

## 问题2

问题：出现下图报错 Access denied for user 'mikey'@'localhost' (using password: YES)
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/316c8a86ee87f248326963893698c9b7.png)

原因：yml文件，数据库用户名，写成了user

解决：应该写为username


## 问题3

问题：IDEA插件Auto filliing Java call arguments 无法正常使用

原因：不支持IDEA2023.3之后的版本

解决：
- 在IDEA 2023.3.3 的版本后失效，手动更换可以生效的插件
- 地址：https://github.com/CymricNPG/AutoFillingCallArguments/releases/tag/1.2.1
- 具体的操作流程：https://www.jianshu.com/p/6a8dba3e7dff

## 问题4

问题：IDEA2023版本，使用alt+enter无法生成序列化ID

原因：版本没有默认自动生成序列化ID

解决：[【IDEA2023自动生成序列化ID】_idea serializable 生成id 快捷键-CSDN博客](https://blog.csdn.net/qq_43495421/article/details/130662017)