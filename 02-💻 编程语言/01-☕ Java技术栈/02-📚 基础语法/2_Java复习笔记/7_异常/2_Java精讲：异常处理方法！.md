---
文章标题: "[[2_Java精讲：异常处理方法！]]"
文章作者: Dakkk
文章概要: |
  文章详细阐述Java异常处理，涵盖异常分类、声明与捕获（`try-catch-finally`, `try-with-resource`）、`throw`/`throws`。同时介绍了断言（assert）用于调试，并深入讲解了日志（`java.util.logging`等）框架的使用方法和最佳实践，提升代码健壮性。
tags:
  - Java
  - 异常处理
  - try-catch-finally
  - try-with-resource
  - 断言
  - 日志
  - Log4j
相关文章:
  - "[[9_异常]]"
  - "[[6_jdbc.properties 文件问题]]"
  - "[[0_基础加强]]"
  - "[[0_链表理论基础]]"
  - "[[1_203.移除链表元素]]"
文章分类: 💻 编程语言
文章路径: 02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/7_异常/2_Java精讲：异常处理方法！.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:36:04
---

## 1 异常分类

异常对象均派生于Throwable类，而Throwable又分解为Error和Exception两个分支，其中Error应该在代码层面避免，而不是作异常处理。 设计程序时一般只关注Exception类下的异常，这下面包含两个异常子类，一个是RuntimeException，另一个是IOException。
### 1.1 RuntimeException

一般来说，RuntimeException属于编程错误造成的异常，包括数组越界、空指针等等，这类异常与Error类下的异常统称为非检查型异常。

### 1.2 IOException

IOException属于非编程错误造成的异常，例如试图超越文件尾部读取数据、打开一个不存在的文件等等。这类异常称为检查型异常，程序员要为检查型异常提供异常处理器。

### 1.3 自定义异常类

有时需要抛出不包括在标准异常类中的异常，就需要定义一个派生于Exception（或其子类）的类，用来满足需求。

自定义异常类通常需要有两个构造器，一个无参，一个带详细信息参数。

```java
// 自定义异常类
public class ExceptionSelf extends Exception{
    // 无参构造器
    public ExceptionSelf(){}
    // 带详细描述信息参数的构造器
    public ExceptionSelf(String msg){
        super(msg);
    }
}
```

## 2 异常处理

### 2.1 声明异常与抛出异常

1）**在方法首部声明这个方法可能抛出的异常，用的是throws关键字，这将把异常抛给上一级处理**。如果一个方法可能出现的异常不止一种，那么就需要用逗号分隔不同的异常全部列举出来。同时，也不允许声明非检查型的异常，即从Error类或者RuntimeException类继承的异常。

这里需要注意的是，如果超类方法没有声明任何检查型异常，那么子类方法也不能声明。

2）也可以在可能出错的地方**主动抛出一个异常对象，用的是throw关键字**，如果这里没有对异常进行捕获处理，那么同样需要在方法头部声明该异常。

```java
import java.io.EOFException;
class ExceptionDemo {
    public void getName() throws EOFException{
        throw new EOFException();
    }
}
```

3）**throw throws的区别**

throw是抛出一个异常类对象，在代码中去抛出，一般格式为throw new xx()

而throws是抛出异常给上一级去作处理，在方法首部去声明，这里只是抛出异常类名，一般格式为：throws xxx

### 2.2 捕获异常

**1）try...catch...finally**

把可能出现异常的地方放在try代码块内，在后面接上catch处理对应的异常，一个try可以有多个catch子句（不能存在子类关系）用于捕获不同的异常。如果try中的语句出现异常，那么将跳过剩下的try代码，直接执行catch语句内容，如果带有finally子句，将在catch代码执行完毕后执行，一般用于IO设备的关闭等操作，注意不要把控制流的语句放在里面。

```csharp
public static void main(String[] args){
    try{
        // 这是可能出现异常的代码块
        int sum = 0;
    }
    catch(Exception err){
        // 对对应异常进行处理
        System.out.println(err.getMessage());
    }
    finally {
        // 一般执行关闭流的操作
        System.out.println("do the close operate");
    }
}
```

在工具类中一般不适用这种方法，而是将异常throws给调用者去做对应的处理。更加合适的操作是：在catch子句中对异常类型进行转换，将原始异常设置为新异常的原因，然后再throw抛给调用者。

**2）try-with-resource**

这就出现一个问题，如果在finally语句的关闭操作也出现异常，那还需要进行再一次的异常处理，于是有了try-with-resource这种捕获异常的方法，与python的with xxx as xxx 操作同理。在try代码块运行结束之后，资源会被自动关闭，而对于这种操作也可以有catch和finally子句，这些子句会在资源关闭后执行。

```csharp
public static void main(String[] args){
    // 把需要打开的流资源写在try后的括号中
    try(var in = new Scanner(new FileInputStream("I:/javastudy/demo.txt"), StandardCharsets.UTF_8)){
        while(in.hasNext()){
            System.out.println(in.next());
        }
    }
    // 作异常处理 此时流资源已关闭
    catch (Exception err){
        System.out.println(err.getMessage());
    }
    // 无需使用finally子句进行资源关闭
}
```
## 3 断言

1、断言是jdk1.4后引入的内容，用关键字assert来表示。如果在程序中需要检测一个参数是否合法，一般是使用if语句来操作，但是测试完毕这部分代码依旧会存于程序中，这时候就需要引入assert断言，断言不是程序的一部分，在测试完毕之后移除该代码，程序仍不会收到影响。（注：idea默认断言是关闭的，需要加上 -ea 运行参数启动）

2、断言语法格式为：**assert condition : expression(可省略)**

如果condition不成立，程序将运行expression然后终止运行并抛出一个AssertionError。如果condition成立，程序正常运行。

```csharp
public static void main(String[] args){
    int sum = 6;
    assert sum==5 : "sum不等于5";
    System.out.println("---如果断言正常---");

}
```

3、断言失败是致命的，不可恢复的，不应使用断言与用户进行沟通，应该只用于内部调试测试过程。
## 4 日志

### 4.1 日志框架

1）**Log4j 或 Log4j 2** —— Apache的开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件、甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；用户也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，用户能够更加细致地控制日志的生成过程。这些可以通过一个配置文件（XML或Properties文件）来灵活地进行配置，而不需要修改程序代码。Log4j 2则是前任的一个升级，参考了Logback的许多特性；

2）**Logback**—— Logback是由log4j创始人设计的又一个开源日记组件。logback当前分成三个模块：logback-core,logback- classic和logback-access。logback-core是其它两个模块的基础模块。logback-classic是log4j的一个改良版本。此外logback-classic完整实现SLF4J API使你可以很方便地更换成其它日记系统如log4j或JDK14 Logging；

3）**java.util.logging** —— JDK内置的日志接口和实现，功能比较简；

4）**Slf4j** —— SLF4J是为各种Logging API提供一个简单统一的接口），从而使用户能够在部署的时候配置自己希望的Logging API实现；

5）**Apache Commons Logging** —— Apache Commons Logging （JCL）希望解决的问题和Slf4j类似。

为了避免引入依赖的繁琐性，这里用java内部的日志框架来记录各项操作。

### 4.2 日志使用

1）基本日志使用：要生成简单的日志可使用全局日志记录器并调用info方法。使用这种方法日志将会直接打印在控制台。

```cpp
import java.util.logging.*;
public class ExceptionSelf{
    public static void main(String[] args){
        // 关闭所有级别日志输出
        Logger.getGlobal().setLevel(Level.OFF);
        // 开启所有级别日志输出
        Logger.getGlobal().setLevel(Level.ALL);
        // 输出一条info级别的日志信息在控制台
        Logger.getGlobal().info("这是一条测试日志信息");
    }
}
```

2）高级日志使用：可以自定义日志记录器，而不是全部记录在全局记录器中，而且用静态变量存储日志存记录器的引用可以避免垃圾误回收。

```java
// 这里的日志记录器名最好定义为当前类的类名 xx.class.getName()
// 并且日志记录器一般定义之后不会再改变，所以存储它的引用的变量应为final类型
private static final Logger Log = Logger.getLogger(ExceptionSelf.class.getName());
```

日志级别如下，一般默认情况下只会记录INFO以上的日志，也可以手动调节显示级别。

```cpp
SEVERE
WARNING
INFO
CONFIG
FINE
FINER
FINEST
// 可以进行级别调整
Log.setLevel(Level.WARNING);
```

3）注意点

a、在一个对象中通常只使用一个Logger对象，Logger应该是static final的，只有在少数需要在构造函数中传递logger的情况下才使用private final。

b、输出Exceptions的全部Throwable信息，因为logger.error(msg)和logger.error(msg,e.getMessage())这样的日志输出方法会丢失掉最重要的StackTrace信息。

c、不允许记录日志后又抛出异常，因为这样会多次记录日志，只允许记录一次日志。

d、不允许出现System print(包括System.out.println和System.error.println)语句。

e、不允许出现printStackTrace。

f、日志性能的考虑，如果代码为核心代码，执行频率非常高，则输出日志建议增加判断，尤其是低级别的输出<debug、info、warn>。
