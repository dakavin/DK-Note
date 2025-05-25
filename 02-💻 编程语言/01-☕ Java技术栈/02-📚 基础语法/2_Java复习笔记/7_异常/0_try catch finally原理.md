---
文章标题: "[[0_try catch finally原理]]" 
文章作者: Dakkk
文章概要: |
  文章通过Java代码示例，深入解析`try-catch-finally`执行机制。重点分析`return`和`System.exit()`在`try`、`catch`、`finally`中的行为。揭示`finally`块的优先级，及其内部`return`或`System.exit()`对方法返回值和程序终止的关键影响。
文章标签:
- "Java"
- "异常处理"
- "try-catch-finally"
- "return语句"
- "System.exit"
- "控制流"
- "异常机制"
相关文章:
- "[[8_异常处理]]"
- "[[1_Java条件判断]]"
- "[[3_关键字]]"
- "[[1_面向对象]]"
- "[[1_Java函数]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/7_异常/0_try catch finally原理.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

前言：  
java 中的异常处理机制你真的理解了吗？掌握了吗？  

catch 体里遇到 return 是怎么处理？
finally 体遇到 return 怎么办？
finally 体里有 System.exit() 方法怎么处理？
当 catch 和 finally 体里同时遇上 return 怎么办？  
  
相信你在处理异常的时候不是每次都把它 throws 掉就完事了，很多时候异常是需要我们自己来 catch 并针对所抛出的 Exception 做一些后续的处理工作。  
  
直接上代码，先贴下面测试需要调用的方法：
```java
// catch 后续处理工作  
public static boolean catchMethod(){  
    System.out.println("call catchMethod and retuen --->> ");  
    return false;}  
  
// finally 后续处理工作  
public static void finallyMethod(){  
    System.out.println("call finallyMethod and do something --->> ");  
}
```

1. 抛出 Exception，没有 finally，当 catch 遇上 return
```java
public static boolean catchTest(){  
    try {  
        int i = 10 / 0; //抛出Exception，后续处理被拒绝  
        System.out.println(" i value is :" + i);  
        return true;    // Exception 意见抛出，没有获得被执行的机会  
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   // Exception抛出，获得了调用方法并放回方法值的机会  
    }  
}
```

后台输出结果
```java
--Exception--
call catchMethod and retuen --->> false
```

2. 抛出 Exception，当 catch 体里有 return，finally 体的代码块将在 catch 执行 return 之前被执行
```java
public static boolean catchFinallyTest1(){  
    try {  
        int i = 10 / 0; //抛出Exception，后续处理被拒绝  
        System.out.println(" i value is :" + i);  
        return true;    // Exception 意见抛出，没有获得被执行的机会  
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   // Exception抛出，获得了调用方法并放回方法值的机会，但方法值在finally执行完后才返回  
    }finally {  
        finallyMethod();    // Exception抛出，finally代码块将在catch执行return之前被执行  
    }  
}
```

后台输出结果
```java
--Exception--
call catchMethod and retuen --->> 
call finallyMethod and do something --->> false
```

3. 不抛出 Exception，当 finally 代码块里面遇上 return，finally 执行完后将结束整个方法
```java
public static boolean catchFinallyTest2(){  
    try {  
        int i = 10 / 2; //不抛出Exception
        System.out.println(" i value is :" + i);  
        return true;    // 获得被执行的机会，但执行需要在finally执行完成之后才能被执行 
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   
    }  finally {  
        finallyMethod();    
        return false；// finally中含有return语句，这个return将结束这个方法，不会在执行完之后再调回try或catch继续执行，方法到此结束，返回false
    }  
}
```

后台输出结果
```java
i value is : 5
call finallyMethod and do something --->> false
```

4. 不抛 Exception，当 finally 代码块里面遇上 System.exit() 方法 将结束和终止整个程序，而不只是方法
```java
public static boolean finallyExitTest(){  
    try {  
        int i = 10 / 2; //不抛出Exception
        System.out.println(" i value is :" + i);  
        return true;    // 获得被执行的机会，但执行需要在finally执行完成之后才能被执行 
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   
    }  finally {  
        finallyMethod();    
		System.exit(0)；// finally中含有System.exit()语句，将退出整个程序，程序将被终止
    }  
}
```

后台输出结果
```java
i value is : 5
call finallyMethod and do something --->> 
```

5. 抛出 Exception，当 catch 和 finally 同时遇上 return，catch 的 return 返回值将不会被返回，finally 的 return 语句将结束整个方法并返回
```java
public static boolean finallyExitTest(){  
    try {  
        int i = 10 / 0; //抛出Exception
        System.out.println(" i value is :" + i);  
        return true;    // 无法执行
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   // 获得执行的机会，但返回的结果将被finally截断
    }  finally {  
        finallyMethod();    
		return false；// return将结束整个方法，返回false
    }  
}
```

后台输出结果
```java
--Exception--

call finallyMethod and do something --->> false
```

6. 不抛出 Exception，当 finally 遇上 return，try 的 return 返回值将不会被返回，finally 的 return 语句将结束整个方法并返回
```java
public static boolean finallyExitTest(){  
    try {  
        int i = 10 / 2; //不抛出Exception
        System.out.println(" i value is :" + i);  
        return true;    // 获得被执行的机会，但返回被finally截断
    }catch (Exception e){  
        System.out.println("--Exception--");  
        return catchMethod();   
    }  finally {  
        finallyMethod();    
		return false；// return将结束整个方法，返回false
    }  
}
```

后台输出结果
```java
i value is : 5
call finallyMethod and do something --->>  false
```

`结语： ` （假设方法需要返回值）  java 的异常处理中， 

`在不抛出异常的情况下:`
程序执行完 try 里面的代码块之后，该方法并不会立即结束，
而是继续试图去寻找该方法有没有 finally 的代码块，  
如果没有 finally 代码块，整个方法在执行完 try 代码块后返回相应的值来结束整个方法；


如果有 finally 代码块，此时程序执行到 try 代码块里的 return 语句之时并不会立即执行 return，而是先去执行 finally 代码块里的代码，  


若 finally 代码块里没有 return 或没有能够终止程序的代码，程序将在执行完 finally 代码块代码之后再返回 try 代码块执行 return 语句来结束整个方法；  


若 finally 代码块里有 return 或含有能够终止程序的代码，方法将在执行完 finally 之后被结束，不再跳回 try 代码块执行 return。


在抛出异常的情况下，原理也是和上面的一样的，你把上面说到的 try 换成 catch 去理解就 OK 了 *_*

