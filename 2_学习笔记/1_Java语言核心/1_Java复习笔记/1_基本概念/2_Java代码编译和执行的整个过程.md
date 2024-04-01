
## 1 整个过程

Java代码的编译和执行的整个过程大概是：

- 开发人员编写Java代码（.java文件），然后将之编译成字节码（.class文件），再然后字节码被装入内存，一旦字节码进入虚拟机，它就会被解释器解释执行，或者是被即时代码发生器有选择的转换成机器码执行。

1. Java代码编译是由Java源码编译器来完成，也就是Java代码到JVM字节码（.class文件）的过程。流程图如下所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58247ecec1fe3c309db9706251cf4804.png)
2. Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/382bdd79a84cbf4c912db04b7ec5d185.png)

Java代码编译和执行的整个过程包含了一下三个重要的机制：
1. Java源码的编译机制
2. 类加载机制
3. 类执行机制

## 2 Java源码编译机制

1. 分析和输入到符号表
2. 注解处理
3. 语义分析和生成class文件

流程图如下所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/037ca0971dc458b41869ec793c33f582.png)
最后生成的class文件由以下部分组成：
1. 结构信息：包括class文件格式版本号及各部分的数量与大小信息
2. 元数据：对应于Java源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域和方法声明信息和常量池
3. 方法信息：对应Java源码中语句和表达式对应的信息。包含字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试符号信息

## 3 类加载机制

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/303cae3d52364422a7cd22df89048ec6.png)
1. Bootstrap ClassLoader
	- 负责加载`$JAVA_HOME$`中`jre/lib/rt.jar`里所有的class，由C++实现，不是ClassLoader子类
2. Extension ClassLoader
	- 负责加载java平台中扩展功能的一些jar包，包括`$JAVA_HOME$`中`jre/lib/*.jar`或`-Djava.ext.dirs`指定目录下的jar包
3. App ClassLoader
	- 负责记载classpath中指定的jar包及目录中class
4. Custom ClassLoader
	- 属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会工具j2ee规范自行实现ClassLoader

加载过程中会检查类是否已被加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检查，只要某个ClassLoader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。

而加载的顺序是自顶而下，也就是由上层来逐层尝试加载此类

## 4 类执行机制

JVM是基于堆栈的虚拟机。

JVM为每个新建的线程都分配一个堆栈，也就是说，对于一个Java程序来说，它的运行就是通过对堆栈的操作来完成的。

堆栈已帧为单位保存线程的状态。JVM对堆栈只进行两种操作：以帧为单位的压栈和出栈操作

JVM执行class字节码，线程创建猴，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个帧，每个栈帧对应着每个方法的每次调用，而栈帧又是由局部变量区和操作数栈两部分组成，局部变量去用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。

栈的结构如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/50f8435f78d6bbbe35ef7604211bd32c.png)