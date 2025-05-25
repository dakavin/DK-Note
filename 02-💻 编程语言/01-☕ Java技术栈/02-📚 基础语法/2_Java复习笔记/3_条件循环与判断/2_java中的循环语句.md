---
文章标题: "[[2_java中的循环语句]]" 
文章作者: Dakkk
文章概要: |
  本文系统介绍了Java中的while、do-while和for三种核心循环语句的语法、执行流程和基本应用。文章通过示例代码阐述了单层及双重循环的使用，并详细讲解了`break`（包括带标签）和`continue`语句如何控制循环的执行流程，帮助读者掌握循环结构基础。
文章标签:
- "Java"
- "循环"
- "while循环"
- "do-while循环"
- "for循环"
- "嵌套循环"
- "break"
- "continue"
相关文章:
- "[[3_流程控制语句]]"
- "[[1_Java基本数据类型]]"
- "[[1_Java面向对象]]"
- "[[1_Java语言概述]]"
- "[[11_集合框架]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/3_条件循环与判断/2_java中的循环语句.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-26 07:17:21
---

## 1 前言

有些程序段在某些条件下重复执行多次，称为循环结构程序。Java提供了3种循环语句实现循环结构，包括while语句、do-while语句、for语句。它们的共同点是根据给定条件来判断是否继续执行指定的程序段（循环体）。如果满足执行条件，就继续执行循环体，否则就不再执行循环体，结束循环语句。
## 2 while循环

while循环的语法如下：

​ while(布尔表达式){

​ 循环体;

}

说明：

1. 布尔表达式表示循环体执行的条件，当条件为true时执行循环体。

2. 循环体既可以是一条简单的语句，也可以是复合语句。

3. while语句的执行过程是：计算布尔表达式的值，如果其值是true，执行循环体；再计算布尔表达式的值，如果其值是true，再执行循环体，形成循环，直到布尔表达式的值变为false，结束循环。

4. 使用while循环，计算1+2+3+....+100的和，并显示结果 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fbc1ccbd516ca9c05d105cf29c0266a.png)

## 3 do while循环

do-while循环的语法如下：

​ do{

​ 循环体;

}while(布尔表达式);

说明：

1. 布尔表达式表示循环执行的条件。

2. 循环体既可以是一条语句，也可以是语句序列。

3. do-while语句执行的过程是：执行循环体，计算布尔表达式的值，如果其值为true，再执行循环体，形成循环，直到布尔表达式的值变为false，结4束循环，执行do-while语句后的语句。

4. 使用do-while循环，计算1+2+3+...+100的和，并显示结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3c6f1dd2181b18c163c59184cc9cc2c.png)


## 4 for循环

for循环的语法结构：

​ for(表达式1; 表达式2; 表达式3){

​ 循环体;

}

说明：

1. 表达式1的作用是给循环变量初始化。

2. 表达式2的作用是给出循环条件。

3. 表达式3的作用是改变循环变量的值。

4. 循环体可以是一条或多条语句。

5. for循环的执行过程是：执行表达式1，计算表达式2，如果表达式2的值为true，执行循环体，执行表达式3，改变循环变量的值，再计算表达式2的值，如果是true，再进入循环体，形成循环，直到表达式2的值为false，结束循环，执行for后面的语句。

6. 使用for循环，计算1+2+3+...+100的和，并显示结果![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f76a851c868e674e0f366e9902d7d563.png)


## 5 双重for循环

打印乘法口诀表
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e0884b575174a8d7c2f7d0b57ce7c33c.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/91eaa850f2d93139c199d49a91b64e04.png)



## 6 break语句的使用 
break是结束当前最近的一个循环

使用while循环计算1+2+3+...,当和超过100时，结束循环，输出一共相加了多少个数![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20643e55afa21f1de35e6bf49ee79fc8.png)

也可以使用标签 break lable; break 直接结束了这个for循环![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/710e9c54572d0121d80bdd1b8a67173d.png)


输出为：123

## 7 continue的使用
 
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/be706c66467058391d84414809e3a16a.png)


输出为： 123123123123

continue只是结束当前的一次循环，后面还是会继续执行
