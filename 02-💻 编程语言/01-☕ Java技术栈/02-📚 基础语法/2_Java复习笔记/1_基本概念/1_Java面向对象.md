---
文章标题: "[[1_Java面向对象]]" 
文章作者: Dakkk
文章概要: |
  本文介绍了Java面向对象编程（OOP）的核心概念，包括对象的定义、特性以及面向对象的三大核心支柱：继承、封装和多态。文章通过实例阐明了这些特性如何提高代码的可重用性、可扩展性和可管理性。
tags:
- "Java"
- "面向对象"
- "OOP"
- "对象"
- "类"
- "继承"
- "封装"
- "多态"
相关文章:
- "[[6_面向对象基础]]"
- "[[1_面向对象]]"
- "[[5_Java面向对象]]"
- "[[3_定义你自己的类]]"
- "[[8_内部类笔记重写]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/1_基本概念/1_Java面向对象.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:36:04
---

面向对象简称 OO（`Object Oriented`），20 世纪 80 年代以后，有了`面向对象分析（OOA）`、 `面向对象设计（OOD）`、`面向对象程序设计（OOP）`等新的系统开发方式模型的研究。

对 [Java](http://c.biancheng.net/java/) 语言来说，一切皆是对象。把现实世界中的对象抽象地体现在编程世界中，一个对象代表了某个具体的操作。一个个对象最终组成了完整的程序设计，这些对象可以是独立存在的，也可以是从别的对象继承过来的。对象之间通过相互作用传递信息，实现程序开发。

## 1 对象的概念

Java 是面向对象的编程语言，对象就是面向对象程序设计的核心。`所谓对象就是真实世界中的实体，对象与实体是一一对应的，也就是说现实世界中每一个实体都是一个对象，它是一种具体的概念。`对象有以下特点：

- 对象具有`属性和行为`。
- 对象具有`变化的状态`。
- 对象具有`唯一性`。
- 对象都是`某个类别的实例`。
-  `一切皆为对象`，真实世界中的所有事物都可以视为对象。

例如，在真实世界的学校里，会有学生和老师等实体，学生有学号、姓名、所在班级等属性（数据），学生还有学习、提问、吃饭和走路等操作。学生只是抽象的描述，这个抽象的描述称为“类”。在学校里活动的是学生个体，即张同学、李同学等，这些具体的个体称为“对象”，“对象”也称为“实例”。

## 2 面向对象的三大核心特性

面向对象开发模式更有利于人们开拓思维，在具体的开发过程中便于程序的划分，方便程序员分工合作，提高开发效率。面向对象程序设计有以下优点。

1. 可重用性：代码重复使用，减少代码量，提高开发效率。下面介绍的面向对象的三大核心特性（继承、封装和多态）都围绕这个核心。
2. 可扩展性：指新的功能可以很容易地加入到系统中来，便于软件的修改。
3. 可管理性：能够将功能与数据结合，方便管理。

该开发模式之所以使程序设计更加完善和强大，主要是因为`面向对象具有继承、封装和多态 3 个核心特性`。

### 2.1 继承性

如同生活中的子女继承父母拥有的所有财产，`程序中的继承性是指子类拥有父类的全部特征和行为，这是类之间的一种关系。` `Java 只支持单继承。  `

例如定义一个语文老师类和数学老师类，如果不采用继承方式，那么两个类中需要定义的属性和方法如下图所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e28d314faa8aa0038594d9421ee8063c.png)
从图中能够看出，语文老师类和数学老师类中的许多属性和方法相同，这些相同的属性和方法可以提取出来放在一个父类中，这个父类用于被语文老师类和数学老师类继承。当然父类还可以继承别的类，如下图所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28f9ca3973b7dab496b338d63ce7bae6.png)
总结上图的继承关系，可以用概括的树形关系来表示，如下图所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a166c4bad3a531a7d9fe5c85609ffe3b.png)
从上图可以看出，学校主要人员是一个大的类别，老师和学生是学校主要人员的两个子类，而老师又可以分为语文老师和数学老师两个子类，学生也可以分为班长和组长两个子类。

使用这种层次的分类方式，是为了将多个类的通用属性和方法提取出来，放在它们的父类中，然后只需要在子类中各自定义自己独有的属性和方法，并以继承的形式在父类中获取它们通用属性和方法即可。

ps：[C++](https://c.biancheng.net/cplus/) 支持多继承，多继承就是一个子类可有多个父类。例如，客轮是轮船也是交通工具，客轮的父类是轮船和交通工具。多继承会引起很多冲突问题，因此现在很多面向对象的语言都不支持多继承。Java 语言是单继承的，即只能有一个父类，但 Java 可以实现多个接口（接口类似于类，但接口的成员没有执行体。详细了解可参考《[Java接口](https://c.biancheng.net/view/6540.html)》一节），可以防止多继承所引起的冲突问题。
### 2.2 封装性

封装是将代码及其处理的数据绑定在一起的一种编程机制，该机制保证了程序和数据不受外部干扰且不被误用。`封装的目的在于保护信息`，使用它的主要有点如下：
- 保护类中的信息，它可以阻止在外部定义的代码随意访问内部代码和数据
- 隐藏细节信息，一些不需要程序员修改和使用的信息，比如取款机中的键盘，用户只需要指导按那个键实现什么操作就可以，至于它内部是如何运行的，用户不需要指导。
- 有助于建立各个系统之间的松耦合关系，提高系统的独立性。当一个系统的实现方式发生变化，只要它的接口不变，就不会影响其他系统的使用。例如U盘，不管里面的存储方式怎么改变，只要U盘上的USB接口不变，就不会影响用户的正常操作
- 提高软件的复用率，降低成本。每个系统都是一个相对独立的整体，可以在不同的环境中得到使用。例如，一个U盘可以在多台电脑上使用。

`Java语音的基本封装单位是类`。由于类的用途是封装复杂性，所以类的内部由隐藏实现复杂性的机制。

`Java提供了私有和公有的访问模式，类的公有接口代表外部的用户应该知道或可以知道的每件东西，私有的方法数据只能通过该类的成员代码来访问，这就可以确保不会发生不希望的事情。`
### 2.3 多态性

面向对象的多态性，即“一个接口，多个方法”。多态性体现在父类定义的属性和方法被子类继承猴，可以具有不同的属性或表现方式。

`多态性允许一个接口被多个同类使用`，弥补了单继承的不足。多态概念可以用树形关系来表示，如图所示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/65d78cd91c81160019c68a4e4ed6c056.png)
可以看出，老师类中的许多属性和方法可以被语文老师类和数学老师类同时使用，这样也不易出错

