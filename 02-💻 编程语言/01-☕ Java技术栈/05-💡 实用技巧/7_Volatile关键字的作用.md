---
文章标题: "[[7_Volatile关键字的作用]]" 
文章作者: Dakkk
文章概要: |
  本文深入解析Java `volatile`关键字，阐明其在多线程中的两大核心作用：确保线程可见性及禁止指令重排序。文章通过Java内存模型、内存屏障和双重检查锁定等实例，详细阐述其工作原理，并对比了`volatile`与`synchronized`的异同。
文章标签:
- "volatile"
- "Java多线程"
- "Java内存模型"
- "指令重排序"
- "内存屏障"
- "线程可见性"
- "双重检查锁定"
- "Synchronized"
相关文章:
- "[[5_特殊关键字的理解]]"
- "[[0_前置知识快速回顾]]"
- "[[2_Java并发]]"
- "[[1_Java多线程]]"
- "[[9_多线程]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/7_Volatile关键字的作用.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---


- Volatile关键字的作用主要有如下两个：  
	1. 线程的可见性：当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。  
	2. 顺序一致性：禁止指令重排序。

## 1 线程可见性

我们先通过一个例子来看看线程的可见性

```java
public class VolatileTest {
    boolean flag = true;

    public void updateFlag() {
        this.flag = false;
        System.out.println("修改flag值为：" + this.flag);
    }

    public static void main(String[] args) {
        VolatileTest test = new VolatileTest();
        new Thread(() -> {
            while (test.flag) {
            }
            System.out.println(Thread.currentThread().getName() + "结束");
        }, "Thread1").start();

        new Thread(() -> {
            try {
                Thread.sleep(2000);
                test.updateFlag();
            } catch (InterruptedException e) {
            }
        }, "Thread2").start();

    }
}
```

- 我们可以看到虽然线程Thread2已经把flag 修改为false了，但是线程Thread1没有读取到flag修改后的值，线程一直在运行

- 我们把flag 变量加上volatile ，重新运行程序，Thread1结束，说明Thread1读取到了flage修改后的值

- 说到可见性，我们需要先了解一下Java内存模型，Java内存模型如下所示：
 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/223ef379435c5d3a227c5d2d26770eea.png)
 - 线程之间的共享变量存储在主内存中（Main Memory）中，每个线程都一个都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。

- 所以当一个线程把主内存中的共享变量读取到自己的本地内存中，然后做了更新。在还没有把共享变量刷新的主内存的时候，另外一个线程是看不到的。

- 如何把修改后的值刷新到主内存中的？
- 现代的处理器使用<span style="background:#d4b106">写缓冲区</span><font color="#4f81bd">临时保存</font>向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，较少对内存总线的占用。
- 但是什么时候写入到内存是不知道的。
- 所以就引入了volatile，volatile是如何保证可见性的呢？

- 在X86处理器下通过工具获取JIT编译器生成的汇编指令来查看对volatile进行写操作时，会多出lock addl。Lock前缀的指令在多核处理器下会引发两件事情：
	1. 将当前处理器缓存行的数据写回到系统内存。
	2. 这个写回内存的操作会使其他cpu里缓存了该内存地址的数据无效。
		- 如果声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的还是旧的，在执行操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

## 2 顺序一致性

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分为如下三种：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d9f4341d4cf114d8daba63285580213a.png)


1属于编译器重排序，2和3属于处理器重排序。这些重排序可能会导致多线程程序出现内存可见性问题。

当变量声明为volatile时，Java编译器在生成指令序列时，会插入内存屏障指令。通过内存屏障指令来禁止重排序。

JMM内存屏障插入策略如下：
在每个volatile写操作的前面插入一个StoreStore屏障，后面插入一个StoreLoad屏障。
在每个volatile读操作后面插入一个LoadLoad，LoadStore屏障。

Volatile写插入内存屏障后生成指令序列示意图：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ded60f722f2f4cf26eb14309c4502e9.png)

Volatile读插入内存屏障后生成指令序列示意图：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1280ceac820d7926645e2b59c8e28055.png)


通过上面这些我们可以得出如下结论：编译器不会对volatile读与volatile读后面的任意内存操作重排序；编译器不会对volatile写与volatile写前面的任意内存操作重排序。

防止重排序使用案例：(单例模式下的获取)

```java
public class SafeDoubleCheckedLocking {
    private volatile static Instance instane;
    public  static Instance getInstane(){
        if(instane==null){
            synchronized (SafeDoubleCheckedLocking.class){
                if(instane==null){
                    instane=new Instance();
                }
            }
        }
        return instane;
    }
}
```

创建一个对象主要分为如下三步：
	1. 分配对象的内存空间。
	2. 初始化对象。
	3. 设置instance指向内存空间。

如果instane 不加volatile，上面的2，3可能会发生重排序。假设A，B两个线程同时获取，A线程获取到了锁，发生了指令重排序，先设置了instance指向内存空间。这个时候B线程也来获取，instance不为空，这样B拿到了没有初始化完成的单例对象（如下图
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bafc0bc01338dedf6967a33881fcceaa.png)

## 3 Volatile与Synchronized比较

1. Volatile是轻量级的synchronized，因为它不会引起上下文的切换和调度，所以Volatile性能更好。
2. Volatile只能修饰变量，synchronized可以修饰方法，静态方法，代码块。
3. Volatile对任意单个变量的读/写具有原子性，但是类似于i++这种复合操作不具有原子性。而锁的互斥执行的特性可以确保对整个临界区代码执行具有原子性。但是对于其他线程来说就没有保证其原子性了，因为，其他线程的修改对另外线程是可见的
4. 多线程访问volatile不会发生阻塞，而synchronized会发生阻塞。
5. volatile是变量在多线程之间的可见性，synchronize是多线程之间访问资源的同步性。

## 4 补充

- volatile关键字对任意单个volatile变量的读写操作可以保证原子性，但是类似于volatile++这样的复合操作就无法保证原子性，如果要对这样操作保证原子性，需要使用synchronized关键字
- synchronized可以保证原子性，有序性和可见性。

- 为了实现volatile的内存语义，编译器在生成字节码时会在指令序列中插入内存屏障来禁止特定类型的处理器重排序，以此来保证有序性。
