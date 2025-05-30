---
文章标题: "[[1_Java基本数据类型]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了Java的8种基本数据类型，包括内存占用、常量表示和变量声明规则。同时，清晰定义了Java标识符的命名规范，并列举了所有关键字。为Java编程初学者提供了核心基础知识。
tags:
- "Java"
- "基本数据类型"
- "标识符"
- "关键字"
- "数据类型"
- "变量"
- "常量"
- "内存"
相关文章:
- "[[4_变量]]"
- "[[2_变量与进制]]"
- "[[2_包装类型]]"
- "[[4_Java关键字50个深度解析]]"
- "[[2_基本语法]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/2_数据类型、关键字/1_Java基本数据类型.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:36:04
---

## 1 基本数据类型

java语言中有8中基本数据类型，分为四大类型：

- 逻辑类型：boolean
- 整数类型：byte、short、int、long
- 浮点类型：float、double
- 字符类型：char

### 1.1 逻辑类型

- 常量：true、false
- 变量：使用关键字声明逻辑变量，若不赋初值，默认为false boolean ni = true

### 1.2 整数类型

`byte型`：`分配1字节`内存，占8位.

- 变量：使用关键字byte来声明byte型变量。 例如：byte i=21;

`short型`：`分配2字节`内存，占16位.

- 变量：使用short关键字来声明short型变量。 例如：short sum=12;

(注：java中不存在byte 和short的常量表示法，但可以把不超出byte和short范围的int型常量赋给byte或short变量）

`int型`：`分配4字节`内存，占32位

- 变量：使用int关键字来声明int型变量 int t=9;

- 常量：123、3000（十进制）、045（八进制，0为前缀）、0x3ADB(十六进制， 0x为前缀）、二进制 0b110(用0b做前缀）

`long型`：`分配8字节`内存，占64位

- 常量：`long型常量用后缀L`来表示，例如782L
	- "L"理论上不分大小写，但是若写成"l"容易与数字"1"混淆，不容易分辩。所以`最好大写`。

- 变量：使用long关键字来声明long型变量 long i=897L;

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fe91828541d35e0f19e038f4ed0fee6e.png)

### 1.3 浮点类型

`float型`：`分配4字节`，占32位，实际储存8位有效数字  
- 常量：234.4f、32213.3F(`常量后缀f、F必须存在，不可省`）

- 变量：使用float关键字来声明float型变量 float ch=32.3f;

`double型`:`分配8字节内存`，占64位，实际储存16位有效数字

- 常量：324.9d、0.05

- 变量：使用double关键字来声明double型变量 double th=8.9;

  ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0021d70885005995eaebe021853fd57.png)


### 1.4 字符类型

char型：分配2字节内存，占16位，变量范围：0~65535

- 常量：'a'、'A'、‘好'、'`\n`' 用单引号括上的Unicode中的一个字符

- 变量：使用char关键字来声明char型变量 char ch='你';

## 2 标识符与关键字

### 2.1 标识符

用来表示类名、变量名、数组名等有效字符序列称为标识符。

要求: 
(1)、标识符的第一个字符不能是数字字符

(2)、标识符由数字、下划线、字母、美元符号组成

(3)、标识符不能是关键字

(4)、标识符不能是true、false、null

例子：dns_21、`_we23`、`$987ewa` 都是标识符

### 2.2 关键字

abstract 、assert 、boolean、break、byte、case、catch、char、class、const、continue、default 、do、double、else、enum、extends、final、finally、float、for、goto、if、implements、import、instanceof、int、interface、long、native、new、package、private、protected、public、return、short 、static、strictfp、super、switch、synchronized、this 、throw、throws、transient 、try 、void、volatile、while

  

  
