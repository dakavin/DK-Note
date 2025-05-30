---
文章标题: "[[4_Java关键字50个深度解析]]" 
文章作者: Dakkk
文章概要: |
  文章深度解析了Java 50个核心关键字，涵盖基本类型、控制流、异常处理、访问修饰符及字段修饰符。它结合代码示例、内存模型（如对象池、常量池）和JVM底层原理（如synchronized锁机制、vtable、instanceof算法）详细阐述了各关键字的功能与深层行为，帮助开发者全面理解Java语言核心。
tags:
- "Java"
- "关键字"
- "JVM原理"
- "并发编程"
- "异常处理"
- "数据类型"
- "访问控制"
- "内存模型"
相关文章:
- "[[5_特殊关键字的理解]]"
- "[[4_变量]]"
- "[[4_技术一面（华为od）]]"
- "[[4_Queue]]"
- "[[6_BigDecimal详解]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/2_数据类型、关键字/4_Java关键字50个深度解析.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:36:04
---

Java关键字列表如下，包含50个关键，所有字符都是小写

[https://docs.oracle.com/javase/tutorial/java/nutsandbolts/_keywords.html](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/_keywords.html)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58b05d090bf56c3b579239c66cc81129.png)

## 1 基本类型

先介绍基本类型关键字如下![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0e49454878cf00003c4c1bb0f3f00bda.png)

参考解析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/90d153782e056914d3d2caaeaa852a75.png)

`注意`：Java浮点数默认是double， 整形默认是int，另外有一个常量池的缓存也需要注意，还有Integer.valueof()方法是Integer a = 10语句的默认赋值方法。

```kotlin
/**
 * 8种基本类型的包装类和对象池
 *      包装类：java提供的为原始数据类型的封装类，如：int(基本数据类型)，Integer封装类。
 *      对象池：为了一定程度上减少频繁创建对象，将一些对象保存到一个"容器"中。
 * 
 *  Byte,Short,Integer,Long,Character。这5种整型的包装类的对象池范围在-128~127之间，也就是说，
 *  超出这个范围的对象都会开辟自己的堆内存。
 * 
 *  Boolean也实现了对象池技术。Double,Float两种浮点数类型的包装类则没有实现。
 *  String也实现了常量池技术。
 * 
 * 自动装箱拆箱技术
 *  JDK5.0及之后允许直接将基本数据类型的数据直接赋值给其对应地包装类。
 *  如：Integer i = 3;（这就是自动装箱）
 *  实际编译代码是：Integer i=Integer.valueOf(3); 编译器自动转换
 *  自动拆箱则与装箱相反：int i = (Integer)5;
 */
public class Test {
    public static void main(String[] args) {
        
        //基本数据类型常量池范围-128~127
        Integer n1 = -129;
        Integer n2 = -129;
        Long n3 = 100L;
        Long n4 = 100L;
        Double n5 =  12.0;
        Double n6 = 12.0;
        //false
        System.out.println(n1 == n2);
        //true
        System.out.println(n3 == n4);
        //false
        System.out.println(n5 == n6);
        
        //String常量池技术,注意：这里String不是用new创建的对象
        String str1 = "abcd";
        String str2 = "abcd";
        //true
        System.out.println(str1 == str2);
                

    }
}

   //5种整形的包装类Byte,Short,Integer,Long,Character的对象，

   //在值小于127时可以使用常量池

   Integer i1=127;

   Integer i2=127;

   System.out.println(i1==i2)//输出true

   //值大于127时，不会从常量池中取对象

   Integer i3=128;

   Integer i4=128;

   System.out.println(i3==i4)//输出false

   //Boolean类也实现了常量池技术

   Boolean bool1=true;

   Boolean bool2=true;

   System.out.println(bool1==bool2);//输出true

   //浮点类型的包装类没有实现常量池技术

   Double d1=1.0;

   Double d2=1.0;

   System.out.println(d1==d2)//输出false
```

- `char 字符  `
    Java语言的一个关键字，用来定义一个字符类型 。
    
- `short 短的  `
    Java语言的关键字，用来定义一个short类型的变量。
    
- `int 整数类型  `
    Java(TM)的一个关键字，用来定义一个整形变量  
    Java(TM)的一个关键字，用来定义一系列的方法和常量。它可以被类实现，通过implements关键字。
    
- `long 长整型  `
    Java语言的一个关键字，用来定义一个long类型的变量。
    
- `float 浮点数`  
    一个Java语言的关键字，用来定义一个浮点数变量 。
    
- `double 双精度型  `
    一个Java语言的关键字，用来定义一个double类型的变量 。
    


## 2 条件分支和循环语句关键字

- `for 为了（循环语句）  `
    一个Java语言的关键字，用来声明一个循环。程序员可以指定要循环的语句，推出条件和初始化变量。
    
- `if 如果  `
    Java编程语言的一个关键字，用来生成一个条件测试，如果条件为真，就执行if下的语句。
    
- `else 否则  `
    一个Java语言的关键字，如果if语句的条件不满足就会执行该语句。
    
- `break 跳出  `
    一个Java的关键字，用来改变程序执行流程，立刻从当前语句的下一句开始执行从。如果后面跟有一个标签，则从标签对应的地方开始执行 。
    
- `case 实例  `
    Java语言的关键字，用来定义一组分支选择，如果某个值和switch中给出的值一样，就会从该分支开始执行。
    
- `continue 继续  `
    一个Java的关键字，用来打断当前循环过程，从当前循环的最后重新开始执行，如果后面跟有一个标签，则从标签对应的地方开始执行。
    
- `do ` 
    一个Java语言的关键字，用来声明一个循环，这个循环的结束条件可以通过while关键字设置
    
- `while 一会儿（循环语句） ` 
    Java语言的一个关键字，用来定义一段反复执行的循环语句。循环的退出条件是while语句的一部分。
    
- `关于break和continue。  `
    continue语句与break语句相关，但较少用到。continue语句用于使其所在的for、while或do-while语句开始下一次循环。在while与do-while语句中，continue语句的执行意味着立即执行测试部分；在for循环语句中，continue语句的执行则意味着使控制传递到增量部分。
    

## 3 异常处理机制

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d164523c7d2b06a349433604ae1e50d2.png)

[0_try catch finally原理](../8_异常/0_try%20catch%20finally原理.md)

try catch jvm原理 [https://blog.csdn.net/TellH/article/details/70940757#t0](https://blog.csdn.net/TellH/article/details/70940757#t0)

- `throw 投、抛  `
    Java语言的关键字，允许用户抛出一个exception对象或者任何实现
    
- `throwable的对象  `
    throws 声明抛弃异常  
    Java语言的关键字，用在方法的声明中来说明哪些异常这个方法是不处理的，而是提交到程序的更高一层。
    
- `try 尝试、审判  `
    Java语言的关键字，用来定义一个可能抛出异常语句块。如果一个异常被抛出，一个可选的catch语句块会处理try语句块中抛出的异常。同时，一个finally语句块会被执行，无论一个异常是否被抛出。
    
- `catch 捕捉  `
    Java的一个关键字，用来声明当try语句块中发生运行时错误或非运行时异常时运行的一个块。
    
- `finally 最后  `
    一个Java语言的关键字，用来执行一段代码不管在前面定义的try语句中是否有异常或运行时错误发生。
    

```csharp
package test;  
//jdk 1.8  
public class TestException1 {  
  
      
    /** 
     * catch中的return和throw是不能共存的(无论谁先谁后都编译不通过) 
     * 如果只throw e，则必须try-catch捕捉异常e或用throws抛出异常e 
     * 如果只return ，则在catch段正常返回值 
     */  
    int testEx0(){  
        boolean ret = true;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return 0;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = false;  
            return -1;  
            throw e;  
        }  
    }  
    /** 
     * 在finally中的return和throw是不能共存的(无论谁先谁后都编译不通过) 
     * 如果只throw e，则必须try-catch捕捉异常e或用throws抛出异常e 
     * 如果只return ，则在catch段正常返回值 
     */  
    int testEx00(){  
        int  ret = 0;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return ret;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = -1;  
        }finally{  
            ret = 1;  
            System.out.println("testEx, finally; return value=" + ret);  
            throw e;  
            return ret;  
        }  
    }  

	// 上述两个方法，编译不能通过，只能return或throw选一个

    boolean testEx01(){  
        boolean ret = true;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return true;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = false;  
            return ret;  
        }finally{  
            System.out.println("testEx, finally; return value=" + ret);  
              
        }  
    }  

	/** 
     * 结果： 
        testEx, catch exception 
        testEx, finally; return value=false 
        false 
        结论：在finally里没有return的时候：先执行finally的语句，再执行catch的return 
     */  
 
    int testEx02(){  
        int ret = 0;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return ret;  
        } catch (Exception e) {  
            ret = -1;  
            System.out.println("testEx, catch exception");  
            return ret;  
        }finally{  
            ret = 1;  
            System.out.println("testEx, finally; return value=" + ret);  
            return ret;  
        }  
    }  

	/** 
     *  结果： 
     *  testEx, catch exception 
        testEx, finally; return value=1 
        1 
        结论：在finally里有return的时候：先执行finally的语句和return，忽略catch的return 
     */ 
 
    boolean testEx03(){  
        boolean ret = true;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return true;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = false;  
            throw e;  
        }finally{  
            System.out.println("testEx, finally; return value=" + ret);  
              
        }  
    }  

    /** 
     * 编译能通过， 
     * 但运行时抛异常（当然也没有返回值） 
     * @return 
     */ 
 
    boolean testEx031(){  
        boolean ret = true;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return true;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = false;  
            throw new Exception(e);  
        }finally{  
            System.out.println("testEx, finally; return value=" + ret);  
              
        }  
    }  
	/** 
     * 编译不能通过（必须加throws主动抛异常，或try-catch捕捉，） 
     * 但运行时抛异常（当然也没有返回值） 
     * @return 
     * @throws Exception  
     */ 
 
    int testEx04(){  
        int ret = 0;  
        try {  
            int c = 12 / 0;  
            System.out.println("testEx,successfully");  
            return ret;  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = -1;  
            throw e;  
        }finally{  
            ret = 1;  
            System.out.println("testEx, finally; return value=" + ret);  
            return ret;  
        }  
    }  
	/** 
     * 结果： 
     *  testEx, catch exception 
        testEx, finally; return value=1 
        1  
        结论： 
        函数在finally里正常返回return的值，无异常，显然catch中的throw被忽略 
     */ 
      
    public static void main(String[] args) {  
        try {  
            System.out.println(new TestException1().testEx0());  
            //System.out.println(new TestException1().testEx00());  
            //System.out.println(new TestException1().testEx01());  
            //System.out.println(new TestException1().testEx02());  
            //System.out.println(new TestException1().testEx03());  
            //System.out.println(new TestException1().testEx031());  
            //System.out.println(new TestException1().testEx04());  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}  
```
  
  
## 4 类或变量的权限控制

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48dd768e0bc0ae2ac355b2b5a378bbb4.png)

- `private 私有的  `
    Java语言的一个关键字，用在方法或变量的声中。它表示这个方法或变量只能被这个类的其它元素所访问。
    
- `protected 保护类型  `
    Java语言的一个关键字，在方法和变量的声明中使用，它表示这个方法或变量只能被同一个类中的，子类中的或者同一个包中的类中的元素所访问。
    
- `public 公共的  `
    Java语言的一个关键字，在方法和变量的声明中使用，它表示这个方法或变量能够被其它类中的元素访问。

![](assets/image-20240201165829381.png)

  
  
## 5 修饰字段变量的关键字

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14fdfa0f78ed9cf34909ba715eb960b4.png)

- `final 最终、不可更改的`
    一个Java语言的关键字。你只能定义一个实体一次，以后不能改变它或继承它。更严格的讲：一个final修饰的类不能被子类化，一个final修饰的方法不能被重写，一个final修饰的变量不能改变其初始值。
    
- `volatile 挥发性、易挥发的  `
    Java语言的关键字，用在变量的声明中表示这个变量是被同时运行的几个线程异步修改的。  
    volatile是一个类型修饰符（type specifier）。它是被设计用来修饰被不同线程访问和修改的变量。如果没有volatile，基本上会导致这样的结果：要么无法编写多线程程序，要么编译器失去大量优化的机会。
    
- `static 静态的  `
    Java语言的关键字，用来定义一个变量为类变量。类只维护一个类变量的拷贝，不管该类当前有多少个实例。"static" 同样能够用来定义一个方法为类方法。类方法通过类名调用而不是特定的实例，并且只能操作类变量。
    
- `transient 短暂的、瞬时的  `
    Java语言的关键字，用来表示一个域不是该对象串行化的一部分。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，然而非transient型的变量是被包括进去的。
    

**synchronized同步控制**  

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b76cd0e66cc3d1a13a977628f19f9c68.png)


  
简单来说原理是

1. synchronized修饰对象或类  
    重量级锁也就是通常说synchronized的对象锁，锁标识位为10，其中指针指向的是monitor对象（在 Synchronized 代码块中的监视器 ）的起始地址。每个对象都存在着一个 monitor 与之关联，对象与其 monitor 之间的关系有存在多种实现方式，如 monitor 可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个 monitor 被某个线程持有后，它便处于锁定状态。 有monitorenter和monitorexit的操作
    
2. synchronized修饰方法  
    当方法调用时，调用指令将会 检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有 monitor ， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor
    
      ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10807784171437b8ab986dd8054a8f7c.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7949989d30d7f1c93845f980a617694a.png)



3. 轻量级锁和重量级锁  
    **轻量级锁。** 他的基本思想是，当线程要获取锁时把锁对象的 Mark Word 复制一份到当前线程的栈顶，然后执行一个 CAS 操作把锁对象的 Mark Word 更新为指向栈顶的副本的指针，如果成功则当前线程拥有了锁。  
    **偏向锁。** 获取的过程如下，当锁对象第一次被线程获取的时候，虚拟机把对象头中的标志位设为“01”，即偏向模式。同时使用CAS操作把获取到这个锁的线程的ID记录在对象的Mark Word之中的偏向线程ID，并将是否偏向锁的状态位置置为1。如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，直接检查ThreadId是否和自身线程Id一致，如果一致，则认为当前线程已经获取了锁，虚拟机就可以不再进行任何同步操作（例如Locking、Unlocking及对Mark Word的Update等）。  
    **详见原理：** [https://www.jianshu.com/p/ce4f5e43e0a8?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=weixin](https://www.jianshu.com/p/ce4f5e43e0a8?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=weixin)

## 6 其他关键字

- `this 这个`  
    Java语言的关键字，用来代表它出现的类的一个实例。this可以用来访问类变量和类方法。
    
- `void 空 ` 
    Java语言的关键字，用在Java语言的方法声明中说明这个方法没有任何返回值。"void"也可以用来表示一句没有任何功能的语句。
    

- `return 返回`  
    Java语言的一个关键字，用来结束一个方法的执行。它后面可以跟一个方法声明中要求的值。
    
- `import 导入 ` 
    Java(TM)编程语言的一个关键字，在源文件的开始部分指明后面将要引用的一个类或整个包，这样就不必在使用的时候加上包的名字。
    
- `implements 实现  `
    Java(TM)编程语言的一个关键字，在类的声明中是可选的，用来指明当前类实现的接口。 tm=tradeMark（Java商标的意思）
    
- `Abstract 抽象的  `
    一个Java语言中的关键字，用在类的声明中来指明一个类是不能被实例化的，但是可以被其它类继承。一个抽象类可以使用抽象方法，抽象方法不需要实现，但是需要在子类中被实现 。  
    **多态实现原理参考：**  
    从虚拟机角度看Java多态->（重写override）的实现原理  
    [https://www.cnblogs.com/2014asm/p/8633660.html](https://www.cnblogs.com/2014asm/p/8633660.html)
    

前面对jvm 的vtable 进行了研究和验证，再总结下特点：

1. vtable 分配在 instanceKlass对象实例的内存末尾 。
2. 其实vtable可以看作是一个数组，数组中的每一项成员元素都是一个指针，指针指向 Java 方法在 JVM 内部所对应的 method 实例对象的内存首地址 。
3. vtable是 Java 实现面向对象的多态性的机制，如果一个 Java 方法可以被继承和重写， 则最终通过 put_method_at函数将方法地址替换,完成 Java 方法的动态绑定。  
    4.Java 子类会继承父类的 vtable，Java 中所有类都继承自 java.lang.Object, java .lang.Object 中有 5 个虚方法（可被继承和重写）：  
    void finalize()  
    boolean equals(Object)  
    String toString()  
    int hashCode()  
    Object clone()  
    因此，如果一个 Java 类中不声明任何方法，则其 vtalbe 的长度默认为 5 。  
    5.Java 类中不是每一个 Java 方法的内存地址都会保存到 vtable 表中，只有当 Java子类中声明的 Java 方法是 public 或者 protected 的，且没有 final 、 static 修饰，并且 Java 子类中的方法并非对父类方法的重写时， JVM 才会在 vtable 表中为该方法增加一个引用 。  
    6.如果 Java 子类某个方法重写了父类方法，则子类的vtable 中原本对父类方法的指针会被替换成子类对应的方法指针，调用put_method_at函数替换vtable中对应的方法指针。

- `instanceof 判断对象类型 ` 
    一个二操作数的Java(TM)语言关键字，用来测试第一个参数的运行时类型是否和第二个参数兼容。  
    instance of 原理： [https://cloud.tencent.com/developer/article/1079376](https://cloud.tencent.com/developer/article/1079376)  
    　JavaSE 8 instanceof 的实现算法：[https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84673681526b663d030af56f3c78f023.png)



1、obj如果为null，则返回false；否则设S为obj的类型对象，剩下的问题就是检查S是否为T的子类型；

2、如果S == T，则返回true；

3、接下来分为3种情况，之所以要分情况是因为instanceof要做的是“子类型检查”，而Java语言的类型系统里数组类型、接口类型与普通类类型三者的子类型规定都不一样，必须分开来讨论。

①、S是数组类型：如果 T 是一个类类型，那么T必须是Object；如果 T 是接口类型，那么 T 必须是由数组实现的接口之一；

②、接口类型：对接口类型的 instanceof 就直接遍历S里记录的它所实现的接口，看有没有跟T一致的；

③、类类型：对类类型的 instanceof 则是遍历S的super链（继承链）一直到Object，看有没有跟T一致的。遍历类的super链意味着这个算法的性能会受类的继承深度的影响。

  