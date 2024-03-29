## 目录

- [目录](#%E7%9B%AE%E5%BD%95)
- [1 总览](#1%20%E6%80%BB%E8%A7%88)
- [2 if-else 语句](#2%20if-else%20%E8%AF%AD%E5%8F%A5)
- [3 else-if 语句](#3%20else-if%20%E8%AF%AD%E5%8F%A5)
- [4 switch选择语句](#4%20switch%E9%80%89%E6%8B%A9%E8%AF%AD%E5%8F%A5)

## 1 总览

java语言和C 语言类似，主要包括三种基本的控制流结构：

1. 程序顺序执行

2. 程序的判断结构

3. 程序的循环结构

如果把前面所学习的关键字理解成java语言的词汇量的话，那么结构化程序设计就是java语言所谓的”语法”了，这种跟计算机沟通的语言很简单。

前面章节中我们提过，java是以分号(;)作为一个语句的结束的，与换行符没有关系，任何一句表达式后面都必须带有一个分号(;)，这样才算正常结束，否则会报语法错误，例如：

```java
int a=100;
System.out.println(a);
```

以上都成为单条语句，而把多个语句放到一个形如 “{...代码...}”这样代码块中，成为语句块，就是以左大括号“｛”开始，以右大括号结束“｝”的代码我们成为语句块，左右大括号必须成对出现，语句块可以互相嵌套。语句块可以作为一个整体，类似把多个语句块组合成为一个语句块。我们在代码中可以任意使用包含多个语句成为一段语句块,有时也称为程序块。但是在一些情况下，大括号是不可缺少的，比如定义一个类的时候，或者定义一个普通方法时，后面就必须要使用大括号，包住一段语句块。例如以下的：

```java
public class Abc {//这个是必须的
   public static void main(String[] args) {//这个是必须的
      float a = 346.756565f;
      //这个大括号可以删除
      {
         int b = (int) a + 10; // 将 a 转换为整型
         System.out.println(b);
      }
   }
}
```


下面我们讲到的条件判断和循环判断的语法是会大量使用到程序块。

判断逻辑是我们生活中最常见的逻辑判断，计算机来执行跟人类思维也是极为类似的：如果[条件成立]就怎样做，否则就那样做。可以这么说，计算机最擅长的就是判断true/false了。

## 2 if-else 语句

if语句是最常见的判断语句，通过对条件(conditional)的判断觉得程序的走向。其基本格式如下：

**if(条件表达式)**

**语句1**

**else**

**语句2**

在执行该判断语句前，都是先执行了条件表达式的语句，条件表达式的返回结果必须是布尔值(boolean)，根据条件表达式的返回，如果是true，那么就执行语句1的内容，如果是false就执行else后面的语句2。如下面这个例子：
```java
int  i=1;
if(i>10)
  System.out.println(i+"大于10");
else
  System.out.println(i+"不大于10");
```

先定义了整型i并赋值1，然后在执行if里面的条件判断式i是否大于10，返回的结果是false，所以执行的是else后面的语句。

在上面的格式中， `else是可选部分`，所以最简单的条件判断式如下：

**if(条件表达式)  
**
**语句1**

如下面这个例子：

```java
int  i=1;
if(i>10)
   System.out.println(i+"大于10");
```

由于条件判断是false，所以这个程序不会输出任何内容。也正是因为else是可选部分，在嵌套使用的时候就会出现理解上的问题，有时候我们面对下面这个例子的时候就会感觉不清晰了

```java
int i = 1;
if (i > 10)
   if (i < 5)
       System.out.println(i + "小于5");
   else
       System.out.println(i + "不大于10");
```


else是对应那一个if呢？java是与最近一个if配对的。程序员在写代码时，适当的缩进代码也可以提高代码的可读性，当然我们有更好的解决办法。

if和else后面可以跟着语句，当然也可以跟着语句块，其格式如下：

**if(条件表达式){**

**语句块1**

**}else{**

**语句块2**

**}**

建议大家在写if语句时，就算后面只有一个语句，也可以使用大括号包住，形成语句块，这样可以提高程序的可读性，如上面的例子，可以修改成为这样：

```java
if (i > 10) {
    if (i < 5) {
        System.out.println(i + "小于5");
    } else {
        System.out.println(i + "不大于10");
   }
}
```


## 3 else-if 语句

有时候条件判断不止两个，可能就需要使用else-if语句了，其语法格式如下：

**if(条件表达式)**

**语句1**

**else if(条件表达式)**

**语句2**

**else if(条件表达式)**

**语句3**

**else if(条件表达式)**

**语句4**

**...**

**else**

**语句**

这样的语句在我们以后的编程中会经常用到，判断的过程是从上往下的判断条件表达式，如果第一个返回的是false就会判断第二个，依次类推，但是如果其中一个返回了true，那么就会执行后面的语句，然后整个else-if语句就会退出，后面如果还有else-if语句也不会在去判断执行的了。我们常常也会在最后面添加一个else语句，当然这也是可选的，这样的效果就是如果上面的所有的if判断都是false，那么就会执行else后面的语句。像上面的if-else一样，后面也是可以跟着语句块的，为了增强程序的可读性，我们后面也常常会使用语句块。格式如下：

**if(条件1){**

**条件1 == true时执行的逻辑**

**}else if(条件2){**

**条件2 == true时执行的逻辑**

**}else if(条件n){**

**条件n == true时执行的逻辑**

**}else{**

**以上条件均不满足而执行的默认的逻辑**

**}**

下面我们具几个实现，比如我们要判断用户年龄小于16岁时不允许登陆游戏网站，那么我们可以使用以下代码：
```java
int uage = 17;
if (uage < 18) {
   System.out.println("Sorry,请关注学业!");
} else {
   System.out.println("欢迎登陆！");
}
```


```java
int result=85;//成绩  
if(result>90){
  System.out.println("优秀");
}else if(result>80){
  System.out.println("良好");
}else if(result>60){
  System.out.println("合格");
}else{
  System.out.println("不合格");
}
```


if判断也可以嵌套使用，也就是在语句块里也可以包含一个if判断表达式，如下面这个例子。其中Scanner是获得用户输入对象，请看下面这个例子：


```java
import java.util.Scanner;
 public class Tt {
   public static void main(String[] args) {
      int num0;//第一个数
      int num1;//第二个数
      int type;//计算类型
 
      System.out.print("*"请输入num0: "*");
      Scanner scr = new Scanner(System.in);
      num0 = scr.nextInt();//程序会在此等待用户的输入
 
      System.out.print("*"请输入num1: "*");
      scr = new Scanner(System.in);
      num1 = scr.nextInt();
System.out.print("*"请输入计算类型(0表示加 ; 1表示减 ; 2表示乘  ; 3表示除): "*");
      scr = new Scanner(System.in);
      type = scr.nextInt();
 
if (type == 0) {
      System.out.println(num0 + "*"+"*" + num1 + "*" ="*" + (num0 + num1));
   } else if (type == 1) {
      System.out.println(num0 + "*"-"*" + num1 + "*" ="*" + (num0 - num1));
   } else if (type == 2) {
      System.out.println(num0 + "*"*"*" + num1 + "*" ="*" + (num0 * num1));
   } else if (type == 3) {
         // 除法，使用嵌套的if语句判定除数不能为0
       if (num1 == 0) {
          System.out.println("*"除数不能为0"*");
      } else {
          System.out.println(num0+"*"/"*"num1+ "*" ="*" + (num0 / num1));
      }
 
   } else {
   // 非法输入
     System.out.println("*"您的输入有误!计算类型只能是[0,1,2,3]"*");
   }
   }
}
```

在这个程序中用了Scnner获得用户的输入，程序运行到scr.nextInt()的时候会停下来，等待用户的输入，用户输入后按回车程序才会继续往下运行，在程序中我们对除法的判断又嵌套了一个除数不能为0的判断。

## 4 switch选择语句

switch语句是另一种判断语句的写法，这种语句在选择时是对case子句的值进行相等测试，其功能性其实和if判断语句一样，仅仅只是书写的方式不同，两者之间可以互通，语法上面没有if语句简介。其具体的语法格式如下：

**switch(被判断的变量)**

**{**

**case 条件1:**

**执行条件1的逻辑**

**break;**

**case 条件2:**

**执行条件1的逻辑**

**break;**

**case n:**

**执行条件n的逻辑**

**break;**

**default:**

**以上条件均不满足而执行的默认的逻辑**

**}**

switch后面的只是被判断的变量，当与case后面的条件相等是，那么case后面的语句就会执行，`最后面的default是可选项，可根据你的业务逻辑需要决定是否添加`，功能类似else语句，就是上面所有的case条件都不满足时就会执行default后面的语句。

值得注意的是，在`JDK 7以前参加被判断的变量的类型只可以是int, byte, char, short等数据类型，但是在JDK 7以后，被判断的变量的类型被增强支持对字符串String的判断`。如果你还是使用JDK 6就要特别注意这一点了。

一般来说，switch与case成功匹配，还会继续顺序执行以后所有的程序代码，因此一般都要在判断成功后面添加break语句跳出判断语句块。有关break关键字的详细说明我们会在后面的章节中说明。

看看这个例子：已知变量int month=1，使用switch判断语句，如果month等于1就输出"一月"，等于2就输出"二月"，如此类推。实现代码如下：

```java
int month=4; 
switch(month){
      case 1:
         System.out.println("一月");
         break;
      case 2:
         System.out.println("二月");
         break;
      case 3:
         System.out.println("三月");
         break;
      case 4:
         System.out.println("四月");
         break;
           //...中间的5~12用户自己补充。
      default:
         System.out.println("month只能是1~12");
}
```