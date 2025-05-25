---
文章标题: "[[1_Java多线程]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了Java创建多线程的四种方式：继承Thread、实现Runnable/Callable接口及线程池。重点阐述了`ThreadPoolExecutor`核心参数，并强调不推荐使用`Executors`以规避OOM风险。文末概述了线程的六种状态。
文章标签:
- "Java多线程"
- "线程创建"
- "ThreadPoolExecutor"
- "线程池"
- "Callable"
- "Runnable"
- "线程状态"
- "OOM"
相关文章:
- "[[9_多线程]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/11_并发、多线程/1_Java多线程.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 创建多线程有几种方式

- 继承Thread类实现多线程
```java
public class ThreadTest extends Thread
{
    public static void main(String[] args)
    {
        ThreadTest t1 = new ThreadTest();
        t1.start();
    }

    @Override
    public void run()
    {
        System.out.println("我开始运行了!");
    }
}
```

- 实现Runnable接口
```java
public class ThreadTest implements Runnable
{
    public static void main(String[] args)
    {
        ThreadTest runable = new ThreadTest();
        Thread t1 = new Thread(runable);
        t1.start();
    }

    @Override
    public void run()
    {
        System.out.println("我开始运行了!");
    }
}
```

- 实现Callable接口（此线程有返回值）
```java
public class ThreadTest implements Callable<String>
{
    public static void main(String[] args) throws InterruptedException, ExecutionException
    {
        ThreadTest threadTest = new ThreadTest();
        FutureTask<String> task = new FutureTask<String>(threadTest);

        Thread t1 = new Thread(task);
        t1.start();

        //task.get会阻塞
        System.out.println("返回的结果是:" + task.get());
    }

    @Override
    public String call() throws Exception
    {
        return "我返回结果了!";
    }
}
```

`使用线程池ThreadPoolExecutor或者Executors创建线程  `
1：Executors.newFixedThreadPool：创建一个固定大小的线程池，可控制并发的线程数，超出的线程会在队列中等待；  

2：Executors.newCachedThreadPool：创建一个可缓存的线程池，若线程数超过处理所需，缓存一段时间后会回收，若线程数不够，则新建线程；

3：Executors.newSingleThreadExecutor：创建单个线程数的线程池，它可以保证先进先出的执行顺序；  

4：Executors.newScheduledThreadPool：创建一个可以执行延迟任务的线程池；

5：Executors.newSingleThreadScheduledExecutor：创建一个单线程的可以执行延迟任务的线程池；  

6：Executors.newWorkStealingPool：创建一个抢占式执行的线程池（任务执行顺序不确定）【JDK 1.8 添加】  

7：**ThreadPoolExecutor**最原始的创建线程池的方式，它包含了 7 个参数可供设置

```java
//此处使用的是ThreadPoolExecutor方式创建的线程
public class ThreadTest implements Runnable
{
    public static void main(String[] args)
    {
        ExecutorService pool = new ThreadPoolExecutor(2, 10, 100, TimeUnit.MINUTES, new LinkedBlockingQueue<>(10));
        for (int i = 0; i < 10; i++)
        {
            pool.execute(new ThreadTest());
        }
    }

    @Override
    public void run()
    {
        System.out.println(Thread.currentThread().getName());
    }
}
```

7 个参数代表的含义如下：

**参数 1：corePoolSize**  
线程数，线程池中始终存活的线程数。

**参数 2：maximumPoolSize**  
最大线程数，线程池中允许的最大线程数，当线程池的任务队列满了之后可以创建的最大线程数。

**参数 3：keepAliveTime**  
最大线程数可以存活的时间，当线程中没有任务执行时，最大线程就会销毁一部分，最终保持核心线程数量的线程。

**参数 4：unit:**  
单位是和参数 3 存活时间配合使用的，合在一起用于设定线程的存活时间 ，参数 keepAliveTime 的时间单位有以下 7 种可选：  
TimeUnit.DAYS：天  
TimeUnit.HOURS：小时  
TimeUnit.MINUTES：分  
TimeUnit.SECONDS：秒  
TimeUnit.MILLISECONDS：毫秒  
TimeUnit.MICROSECONDS：微妙  
TimeUnit.NANOSECONDS：纳秒

**参数 5：workQueue**  
一个阻塞队列，用来存储线程池等待执行的任务，均为线程安全，它包含以下 7 种类型：  
ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。  
LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。  
SynchronousQueue：一个不存储元素的阻塞队列，即直接提交给线程不保持它们。  
PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。  
DelayQueue：一个使用优先级队列实现的无界阻塞队列，只有在延迟期满时才能从中提取元素。  
LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。与SynchronousQueue类似，还含有非阻塞方法。  
LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。  
较常用的是 LinkedBlockingQueue 和 Synchronous，线程池的排队策略与 BlockingQueue 有关。

**参数 6：threadFactory**  
线程工厂，主要用来创建线程，默认为正常优先级、非守护线程。

**参数 7：handler**  
拒绝策略，拒绝处理任务时的策略，系统提供了 4 种可选：

AbortPolicy：拒绝并抛出异常。  
CallerRunsPolicy：使用当前调用的线程来执行此任务。  
DiscardOldestPolicy：抛弃队列头部（最旧）的一个任务，并执行当前任务。  
DiscardPolicy：忽略并抛弃当前任务。

   **注意：【强制要求】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。**

   **说明：Executors 返回的线程池对象的弊端如下：**  
   
1） FixedThreadPool 和 SingleThreadPool：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。

2）CachedThreadPool：允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM

## 2 线程的6大状态

1、新建（new）创建后尚未启动的线程状态  
2、运行（runnable）包含running和ready  
3、无限期等待（waiting）不会被分配cpu执行时间，需要显示被唤醒  
4、限期等待（timed waiting）在一定时间后会由系统自动唤醒  
5、阻塞（blocked）等待获取排它锁  
6、结束（terminated）已终止线程状态，线程已经结束执行

