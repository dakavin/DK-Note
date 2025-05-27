---
文章标题: "[[6_JDBC学习的一些发现]]" 
文章作者: Dakkk
文章概要: |
  介绍了JDBC学习中的实践发现，包括事务自动回滚机制、JUnit批量测试配置、日期类型转换方法以及项目中导入jar包的详细步骤。
tags:
- "JDBC"
- "事务管理"
- "JUnit测试"
- "Java日期处理"
- "jar包管理"
- "IDE配置"
- "项目结构"
相关文章:
- "[[2_📕文章管理：文章发布接口开发（1）]]"
- "[[2_安装VsCode 开发工具]]"
- "[[2_VsCode使用]]"
- "[[4_📕文章管理：文章删除接口开发]]"
- "[[4_JavaEE的13项规范]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/💡 实用技巧/6_JDBC学习的一些发现.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 关于一个方法的结束

- 一个方法中，定义了事务不是默认提交的
- 并且在方法的最后，没有进行事务的提交或者回滚
- `那么方法结束时，该事务会自动回滚到最初状态！！！`

## 2 junit的批量测试操作

1. 在src或其他包下定义一个test source root类型的包
	- 右键该包，在mark directory as 中选择
	- 或者在project structure中按下图进行选择

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a24e492eac744b8013c61fe6bc1b7b5.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6f40c8eceac79fb3e83a12b155fdb25f.png)



2. 在测试所在类中alt + ins 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/45dfa5bd56d58cac2ebf1d9b5105220f.png)



## 3 修改Date的做法

```java
LocalDate localDate = LocalDate.of(1999,9,30);

//转化为sql下的date
java.sql.Date.valueOf(localDate);
```

## 4 导入jar包的操作

1. 在项目下，新建一个名为lib文件夹
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f66cf4204fe3db35e9245b09b2e94c5c.png)


1. 将jar拷贝到该文件夹中，右键jar，点击add as library

1. 在project structure中，给每个模块添加该jar文件

## 5 添加XML文件的方法

