<h1>目录</h1>
- [1 前言](#1%20%E5%89%8D%E8%A8%80)
- [2 while循环](#2%20while%E5%BE%AA%E7%8E%AF)
- [3 do while循环](#3%20do%20while%E5%BE%AA%E7%8E%AF)
- [4 for循环](#4%20for%E5%BE%AA%E7%8E%AF)
- [5 双重for循环](#5%20%E5%8F%8C%E9%87%8Dfor%E5%BE%AA%E7%8E%AF)
- [6 break语句的使用](#6%20break%E8%AF%AD%E5%8F%A5%E7%9A%84%E4%BD%BF%E7%94%A8)
- [7 continue的使用](#7%20continue%E7%9A%84%E4%BD%BF%E7%94%A8)

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

4. 使用while循环，计算1+2+3+....+100的和，并显示结果 ![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202000417225.png)

## 3 do while循环

do-while循环的语法如下：

​ do{

​ 循环体;

}while(布尔表达式);

说明：

1. 布尔表达式表示循环执行的条件。

2. 循环体既可以是一条语句，也可以是语句序列。

3. do-while语句执行的过程是：执行循环体，计算布尔表达式的值，如果其值为true，再执行循环体，形成循环，直到布尔表达式的值变为false，结4束循环，执行do-while语句后的语句。

4. 使用do-while循环，计算1+2+3+...+100的和，并显示结果![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202000439499.png)


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

6. 使用for循环，计算1+2+3+...+100的和，并显示结果![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001540267.png)


## 5 双重for循环

打印乘法口诀表
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001555280.png)


![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001600241.png)



## 6 break语句的使用 
break是结束当前最近的一个循环

使用while循环计算1+2+3+...,当和超过100时，结束循环，输出一共相加了多少个数![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001626799.png)

也可以使用标签 break lable; break 直接结束了这个for循环![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001634105.png)


输出为：123

## 7 continue的使用
 
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240202001652063.png)


输出为： 123123123123

continue只是结束当前的一次循环，后面还是会继续执行
