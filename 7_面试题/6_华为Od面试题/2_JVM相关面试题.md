
## 1 JVM组成

### 1.1 JVM由那些部分组成，运行流程是什么？

**JVM是什么**
- Java Virtual Machine Java程序的运行环境（java二进制字节码的运行环境）
- 好处：
	- 一次编写，到处运行
	- 自动内存管理，垃圾回收机制
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/afbffe0050dee13ca09b42286bdf9c90.png)

---

**JVM的组成部分**
![image.png|800](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d5963a352ddb4862fef0209184359fa.png)

![image.png|800](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3bbab7fc91d4ca0fe4883d1018c3142f.png)

从图中可以看出 JVM 的主要组成部分 
- `ClassLoader`（类加载器）
- `Runtime Data Area`（运行时数据区，内存分区）
- `Execution Engine`（执行引擎）
- `Native Method Library`（本地库接口）

---

**JVM运行流程**
- 类加载器（ClassLoader）把==Java代码转换为字节码==
- 运行时数据区（Runtime Data Area）把==字节码加载到内存==中，而字节码文 件只是JVM的一套指令集规范，并不能直接交给底层系统去执行，而是==由执行引擎运行==
- 执行引擎（Execution Engine）将==字节码翻译为底层系统指令==，再==交由CPU 执行去执行==，此时==需要调用其他语言的本地库接口（Native Method Library）==来 实现整个程序的功能。

--- 

**运行时数据区的主要组成部分：堆、方法区、栈、本地方法栈、程序计数器**
- ==堆==解决的是对象实例存储的问题，垃圾回收器管理的主要区域
- ==方法区==可以认为是堆的一部分，用于存储已被虚拟机加载的信息，常量、静态变量、即时编译器编译后的代码
- ==栈==解决的是程序运行的问题，栈里面存的是栈帧，栈帧里面存的是局部变量 表、操作数栈、动态链接、方法出口等信息
- 本地方法栈与栈功能相同，本地方法栈执行的是本地方法，是Java调用非 Java代码的接口
- 程序计数器（PC寄存器）程序计数器中存放的是当前线程所执行的字节码的 行数。JVM工作时就是通过改变这个计数器的值来选取下一个需要执行的字节码 指令

### 1.2 什么是程序计数器？

程序计数器：**线程私有的，内部保存的字节码的行号。用于记录正在执行的字节码指令的地址**

> javap -verbose xx.class 打印堆栈大小，局部变量的数量和方法的参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e15b6e67d47ee1347f32de4e99a019f8.png)

 java虚拟机对于**多线程是通过线程轮流切换并且分配线程执行时间**。在任何的 一个时间点上，一个处理器只会处理执行一个线程，如果当前被执行的这个线程 它所分配的执行时间用完了【挂起】。处理器会切换到另外的一个线程上来进行 执行。并且这个线程的执行时间用完了，接着处理器就会又来执行被挂起的这个 线程。 
 
 那么现在有一个问题就是，当前处理器如何能够知道，对于这个被挂起的线 程，它上一次执行到了哪里？那么这时就需要从程序计数器中来回去到当前的这 个线程他上一次执行的行号，然后接着继续向下执行。 
 
 **程序计数器是JVM规范中唯一一个没有规定出现OOM的区域，所以这个空间也 不会进行GC**

### 1.3 能详细介绍Java堆吗？

==线程共享的区域==：主要用来==保存对象实例，数组==等，当堆中没有内存空间可分配 给实例，也无法再扩展时，则抛出OutOfMemoryError异常
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cfcdfa17fed97487d239989692b4ea53.png)

- 年轻代被划分为三部分，Eden区和两个大小严格相同的Survivor区，根据JVM 的策略，在经过几次垃圾收集后，任然存活于Survivor的对象将被移动到老年 代区间
- ==老年代主要保存生命周期长的对象==，一般是一些老的对象
- ==元空间保存的类信息、静态变量、常量、即时编译后的代码==

> Java8，HotSpots取消了永久代，取而代之的是元空间（Metaspace），换言之，就是方法区是在的，只是实现方式由原来的永久代变成了现在的元空间了
> 
> **如上图：物理内存不再与堆连续，而是直接存在于本地内存中，理论上，机器内存有多大，元空间就有多大，这样就避免了堆上出现OOM的情况**

> Java7及以前的版本中，方法区都是永久代实现的
> ![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e51dc4277647fc10913b43a0ab92392a.png)
> 永久代的方法区和堆使用的物理内存是连续的
> 
> 对于永久代，如果动态生成很多class的时候，很有可能出现java.lang.OutOfMemoryError：PermGen space错误，这是因为永久代空间配置的大小有限

--- 

**Java 7 和 Java 8 中的内存结构对比图**
![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3c4a26af7e006b3ab352ef9b2f8cbd55.png)

---

**元空间介绍**
- 首先我们需要知道，方法区是逻辑上的概念（JVM规范的方法区），永久代/元空间是具体的内存空间

- ==永久代==
	- 在 HotSpot JVM 中，永久代（ ≈ 方法区）中用于==存放类和方法的元数据以及常量池==，比如Class 和 Method。每当一个类初次被加载的时候，它的元数据都会放到永久代中
	- ==永久代是有大小限制的==，因此如果加载的类太多，很有可能导致永久代内存溢 出，即OutOfMemoryError，为此不得不对虚拟机做调优
	- 那么，Java 8 中 PermGen 为什么被移出 HotSpot JVM 了？
	- 官网给出了解释：http://openjdk.java.net/jeps/122
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bedf2731d3609be74adb2baa99bead66.png)
		- 由于 PermGen 内存经常会溢出，引发OutOfMemoryError，因此 JVM 的开发 者希望这一块内存可以更灵活地被管理，不要再经常出现这样的 OOM
		- 移除 PermGen 可以促进 HotSpot JVM 与 JRockit VM 的融合，因为 JRockit 没 有永久代

- 永久代 ➡ 元空间
	- Perm 区中的==字符串常量池被移到了堆内存==中是在 Java7 之后
	- Java 8 时，PermGen 被元空间代替，其他内容比如类元信息、字段、静态属性、方法、常量等都移动到元空间区。比如 java/lang/Object 类元信息、静态属性、 System.out、整型常量等。

- 元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现。不过元空间与 永久代之间最大的区别在于：==元空间并不在虚拟机中，而是使用本地内存==。因此，默认情况下，元空间的大小仅受本地内存限制

### 1.4 什么是虚拟机栈？

Java Virtual machine Stacks (java 虚拟机栈)
- 每个线程运行时所需要的内存，称为虚拟机栈，先进后出 
- 每个栈由多个栈帧（frame）组成，对应着每次方法调用时所占用的内存 
- ==每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法==
  ![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/641d44cf484d2978bc464b97e43efdeb.png)
---

**垃圾回收是否涉及栈内存？** 
- 垃圾回收主要指就是堆内存，当栈帧弹栈以后，内存就会释放 


**栈内存分配越大越好吗？** 
- 未必，默认的栈内存通常为1024k 
- 栈帧过大会导致线程数变少，例如，机器总内存为512m，目前能活动的线程 数则为512个，如果把栈内存改为2048k，那么能活动的栈帧就会减半 

**方法内的局部变量是否线程安全？**
- 如果方法内局部变量没有逃离方法的作用范围，它是线程安全的 
- 如果是==局部变量引用了对象，并逃离方法的作用范围==，需要考虑线程安全 
- 比如以下代码：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e80f12f84c673126fd1a618df971984.png)
---

**栈内存溢出情况**
- 栈帧过多导致栈内存溢出，典型问题：==递归调用==
- 栈帧过大导致栈内存溢出
### 1.5 能不能解释一下方法区？

**方法区概述**
- 方法区(Method Area)是各个线程共享的内存区域
- 主要存储类的信息、运行时常量池、静态变量和构造函数等
- 虚拟机启动的时候创建，关闭虚拟机时释放
- 如果方法区域中的内存无法满足分配请求，则会抛出`OutOfMemoryError: Metaspace`
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a772224c5afb93f40d65f11de0b75bfa.png)
---

**常量池**
- 可以看作是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息
- 查看字节码结构（类的基本信息、常量池、方法定义） `javap -v xx.class`
- 比如下面是一个Application类的main方法执行，源码如下
```java
public class Application { 
	public static void main(String[] args) {
		System.out.println("hello world"); 
	}
}
```
- 找到类对应的class文件存放目录，执行命令：` javap -v Application.class` 查看字节码结构
  ![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7725bc6c3ac5c7438617472c8aefe56.png)
  - 下图，左侧是main方法的指令信息，右侧constant pool 是常量池
  - main方法按照指令执行的时候，需要到常量池中查表翻译找到具体的类和方法地 址去执行
    ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c87289b2105d0fffef113f2d85ab647d.png)
---

**运行时常量池**
- 常量池是 `*.class` 文件中的
- 当该类被加载，它的==常量池信息就会放入运行时常量池==，并把里面的==符号地址变为真实地址==
### 1.6 听过直接内存吗？

**不受 JVM 内存回收管理，是虚拟机的系统内存**，常见于 NIO 操作时，用于数据缓冲区，分配回收成本较高，但读写性能高，不受 JVM 内存回收管理

**举例：**
- 需求，在本地电脑中的一个较大的文件（超过100m）从一个磁盘挪到另外一个 磁盘
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/034f61c0b7fd4486c88ec06fba9eecd9.png)
- 代码如下：
```java
public class MyTest {  
    static final String FROM = "C:\\Users\\mikey\\Downloads\\Video\\一看到.ts";  
    static final String TO = "G:\\ss\\1_Video\\一看到.ts";  
    static final int _1Mb = 1024 * 1024;  
  
    public static void main(String[] args) {  
        io();   //io 用时：4001.6542  
        directBUffer();  // directBuffer 用时：479.295165 
    }  
  
    private static void directBUffer(){  
        long start = System.nanoTime();  
        try (  
            FileChannel from = new FileInputStream(FROM).getChannel();  
            FileChannel to = new FileInputStream(TO).getChannel();  
            ){  
            ByteBuffer bb = ByteBuffer.allocateDirect(_1Mb);  
            while(true){  
                int len = from.read(bb);  
                if(len == -1) break;  
                bb.flip();  
                to.write(bb);  
                bb.clear();  
            }  
        }catch (IOException e){  
            e.printStackTrace();  
        }  
        long end = System.nanoTime();  
        System.out.println("directBuffer 用时：" + (end - start) / 1000_000.0);  
    }  
  
    private static void io(){  
        long start = System.nanoTime();  
        try (  
                FileInputStream from = new FileInputStream(FROM);  
                FileOutputStream to = new FileOutputStream(TO);  
        ){  
            byte[] buf = new byte[_1Mb];  
            while(true){  
                int len = from.read(buf);  
                if(len == -1) break;  
                to.write(buf,0,len);  
            }  
        }catch (IOException e){  
            e.printStackTrace();  
        }  
        long end = System.nanoTime();  
        System.out.println("io 用时：" + (end - start) / 1000_000.0);  
    }  
}
```
- 可以发现，使用传统的IO的时间要比NIO操作的时间长了很多了，也就说NIO的 读性能更好
- 这个是跟我们的JVM的直接内存是有一定关系，如下图，是传统阻塞IO的数据传 输流程
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/812ff06219e8841750aa634d1ee18153.png)

- 下图是NIO传输数据的流程，在这个里面主要使用到了一个直接内存，不需要在 堆中开辟空间进行数据的拷贝，jvm可以直接操作直接内存，从而使数据读写传 输更快
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ab196ef3f7383cdbcb5ba347144a537.png)

### 1.7 堆栈的区别是什么？

- 栈内存一般会用来存储局部变量和方法调用，但堆内存是用来存储Java对象 和数组的的。堆会GC垃圾回收，而栈不会
- 栈内存是线程私有的，而堆内存是线程共有的
- 两者异常错误不同，但如果栈内存或者堆内存不足都会抛出异常
	- 栈空间不足：java.lang.StackOverFlowErro
	- 堆空间不足：java.lang.OutOfMemoryError

## 2 类加载器

### 2.1 什么是类加载器，有哪些？

### 2.2 什么是双亲委派机制？

### 2.3 JVM为什么采用双亲委派机制？

### 2.4 说一下类装载的执行过程！

## 3 垃圾回收

### 3.1 简述Java垃圾回收机制？

### 3.2 对象什么时候可以被垃圾回收？

### 3.3 JVM 垃圾回收算法有哪些？

### 3.4 分代收集算法

### 3.5 JVM有哪些垃圾回收器？

### 3.6 详细聊一些G1垃圾回收器

### 3.7 强引用、软引用、弱应用和虚引用的区别？

## 4 JVM实践（调优）

### 4.1 JVM 调优的参数可以在哪里设置？

### 4.2 用的 JVM 调优参数都有哪些？

### 4.3 Java内存泄漏的排查思路？

### 4.4 CPU飙高排查方案与思路？

## 5 面试现场