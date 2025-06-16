---
文章标题: "[[4_配置 Lombok]]"
文章作者: Dakkk
文章概要: |
  本文介绍了Lombok，一个Java库，通过注解消除冗余的样板代码，如Getter/Setter和构造函数。文章详细阐述了Lombok的多种用途，并指导读者如何添加Maven依赖以及在IntelliJ IDEA中配置Lombok插件，以提高开发效率。
文章标签:
  - Lombok
  - Java
  - IDEA
  - Maven
  - 样板代码
  - 注解
  - 开发效率
相关文章:
  - "[[0_基础加强]]"
  - "[[19_Mybatis入门]]"
  - "[[../../../../08-🛠️ 开发工具/04-☁️ 运维部署/02-🤖 自动化部署/5_姿势3：使用 GitHub Action（后端项目）]]"
  - "[[0_参考文章索引]]"
  - "[[0_导读]]"
文章分类: 🚀 项目实战
文章路径: 10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/4_配置 Lombok.md
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:34
修改时间: 2024-04-20 22:28:51
---

## 1 Lombok概念

Lombok 是一个超酷的 Java 库，**它能让你避免编写那些冗余的 Java 样板式代码**，如对象中的 `get`、`set`、`toString` 等方法，解放你的双手，堪称偷懒神器，在企业级项目开发中，是必会的一个库。本小节中，就带着大家在 IDEA 中配置好 Lombok。

![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/288725c50062ae36d350f38a51750b70.png)


## 2 Lombok用途

1. **简化 Getter 和 Setter 方法：** 在传统的 Java 开发中，你经常需要为每个类的属性手动编写 Getter 和 Setter 方法，但是有了 Lombok，你只需要在属性上加上 `@Getter` 和 `@Setter` 注解，Lombok 就会为你自动生成这些方法。

2. **自动生成构造函数：** 通过 `@NoArgsConstructor`、`@RequiredArgsConstructor` 或 `@AllArgsConstructor` 注解，你可以快速生成无参构造函数、带有必需参数的构造函数或者带有全部参数的构造函数。

3. **自动生成 equals 和 hashCode 方法：** 通过 `@EqualsAndHashCode` 注解，Lombok 会根据类的字段自动生成 `equals()` 和 `hashCode()` 方法，让你的类更易于比较和使用在集合中。

4. **日志记录更轻松：** 使用 `@Slf4j` 注解，你可以直接在类中使用 `log` 对象，而无需手动创建日志记录器。

5. **简化异常抛出：** 通过 `@SneakyThrows` 注解，你可以在方法中抛出受检异常，而无需显式地在方法上声明或捕获它们。

6. **数据类简化：** 使用 `@Data` 注解，Lombok 会为你自动生成所有常用方法，如 Getter、Setter、`toString()` 等，让你的数据类更加简洁。

7. **链式调用：** 使用 `@Builder` 注解，Lombok 可以帮你创建一个更优雅的构建器模式，让你的对象初始化更加流畅。
## 3 项目中添加Lombok依赖

想要在项目中使用 Lombok, 首先, 你需要在项目中添加 Lombok 依赖。前面小节中，其实已经将 Lombok 依赖添加好了，这里再提一下。

针对通过 Maven 构建的项目来说，只需要在项目的 `pom.xml` 文件中添加以下依赖：
```xml
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId> 
    <version>1.18.28</version>
	<scope>provided</scope>  
</dependency>
```
## 4 IDEA配置Lombok插件

TIP: IDEA 2020.3 及以上版本已经内置安装了 Lombok 插件，如果你的 IDEA 版本大于该版本，则不用管，否则需要按下面的步骤，来手动安装 Lombok 插件。

除了添加依赖外，还需要在 IDEA 中安装 Lombok 插件，才能正式的使用 Lombok。

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dffc45b2c1a42b402969110ff1eb80a4.png)
