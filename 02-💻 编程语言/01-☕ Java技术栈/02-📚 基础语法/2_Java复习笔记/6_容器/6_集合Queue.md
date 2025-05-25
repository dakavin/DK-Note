---
文章标题: "[[6_集合Queue]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了Java集合框架中的Queue接口，重点阐述了PriorityQueue（优先队列）的排序特性、底层数组实现和时间复杂度。接着，深入分析了Deque接口及其ArrayDeque（双端队列）实现类，揭示了其作为队列和栈的功能、循环数组的内部工作原理及扩容机制。
文章标签:
- "Java"
- "Queue"
- "PriorityQueue"
- "Deque"
- "ArrayDeque"
- "数据结构"
- "集合框架"
- "循环数组"
相关文章:
- "[[3_集合框架Collection、Map]]"
- "[[11_集合框架]]"
- "[[12_泛型]]"
- "[[2_鱼骨头聊算法]]"
- "[[1_面向对象]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/6_容器/6_集合Queue.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 前言

Queue用于模拟队列这种数据结构，队列通常是指“先进先出”的容器，新元素插入到队列尾部，访问元素操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34b78987b34f464b91d780b7b472547e.png)

这种结构就如同我们生活中的排队一样

下面我们就来介绍Queue中的一个重要的实现类PriorityQueue

## 2 PriorityQueue

PriorityQueue保存队列元素的顺序不是按加入队列的顺序，而是按照队列元素的大小进行重新排序。因此当调用peek()或pool()方法取出队列中头部的元素时，并不是取出最先进入队列的元素，而是`取出队列中最小的元素`。

### 2.1 PriorityQueue的排序方式

PriorityQueue中的元素可以默认自然排序（也就是数字默认是小的在队列头，字符串则按字典序排列）或者通过提供的Comparator（比较器）在队列实例化时指定的排序方式。

`注意：`队列的头是按指定排序方式的最小元素。如果多个元素都是最小值，则头是其中一个元素——选择方法是任意的。

`注意：`当PriorityQueue中没有指定Comparator时，加入PriorityQueue的元素必须实现了Comparable接口，否则会导致ClassCastException

下面具体写个例子来展示PriorityQueue中的排序方式
```java
public class PriorityQueueTest {  
    public static void main(String[] args) {  
        PriorityQueue<Integer> pq = new PriorityQueue<>();  
        pq.add(5);  
        pq.add(2);  
        pq.add(1);  
        pq.add(10);  
        pq.add(3);  
  
        while (!pq.isEmpty()){  
            System.out.printf(pq.poll() + ",");  
        }  
  
        System.out.println();  
  
        //采用降序排列的方式，小的在队尾  
        Comparator<Integer> objectComparator = new Comparator<>() {  
            @Override  
            public int compare(Integer i1, Integer i2) {  
                return i2 - i1;  
            }  
        };  
  
        PriorityQueue<Integer> pq1 = new PriorityQueue<>(objectComparator);  
        pq1.add(2);  
        pq1.add(8);  
        pq1.add(9);  
        pq1.add(3);  
        pq1.add(1);  
        while (!pq1.isEmpty()){  
            System.out.printf(pq1.poll() + ",");  
        }  
    }  
}
```

输出结果：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1066cf12b497551ba4e73d81f273b15a.png)

由此可以看出，默认情况下PriorityQueue采用自然排序。指定Comparator的情况下，PriorityQueue采用指定的排序方式。

### 2.2 PriorityQueue的方法

PriorityQueue实现了Queue接口，下面列举PriorityQueue的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b79041c5c00c101ac9d2fea7af84b5a5.png)


### 2.3 PriorityQueue的本质

PriorityQueue本质也是一个动态数组，在这一方面与ArrayList是一致的。

PriorityQueue调用默认的构造方法时，使用默认的初始容量（`DEFAULT_INITIAL_CAPACITY = 11`）创建一个PriorityQueue，并根据其自然顺序来排序其元素（使用加入其中的集合元素实现的Comparable）
```java
 public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }
```

当使用指定容量的构造方法时，使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序来排序其元素（使用加入其中的集合元素实现的Comparable）。

```java
 public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
    }
```

当使用指定的初始容量创建一个 PriorityQueue，并根据指定的比较器comparator来排序其元素。

```java
public PriorityQueue(int initialCapacity,
                         Comparator<? super E> comparator) {
        // Note: This restriction of at least one is not actually needed,
        // but continues for 1.5 compatibility
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }
```

从第三个构造方法可以看出，内部维护了一个动态数组。当添加元素到集合时，会先检查数组是否还有余量，有余量则把新元素加入集合，没余量则调用 `grow()`方法增加容量，然后调用`siftUp`将新加入的元素排序插入对应位置。

```java
 public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        size = i + 1;
        if (i == 0)
            queue[0] = e;
        else
            siftUp(i, e);
        return true;
    }
```

除此之外，还要注意：  

**①PriorityQueue不是线程安全的。** 如果多个线程中的任意线程从结构上修改了列表， 则这些线程不应同时访问 PriorityQueue 实例，这时请使用线程安全的PriorityBlockingQueue 类。

**②不允许插入 null 元素。**

**③PriorityQueue实现插入方法（offer、poll、remove() 和 add 方法） 的时间复杂度是O(log(n)) ；实现 remove(Object) 和 contains(Object) 方法的时间复杂度是O(n) ；实现检索方法（peek、element 和 size）的时间复杂度是O(1)。** 所以在遍历时，若不需要删除元素，则以peek的方式遍历每个元素。

**④方法iterator()中提供的迭代器并不保证以有序的方式遍历优PriorityQueue中的元素。**

## 3 Deque接口与ArrayDeque实现类

### 3.1 Deque接口

Deque接口是Queue接口的子接口，它代表一个双端队列。LinkedList也实现了Deque接口，所以也可以被当作双端队列使用。

因此Deque接口增加了一些关于双端队列操作的方法。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1f2a4e041237197be7cd8690696e0f7.png)

从上面方法中可以看出，Deque不仅可以当成双端队列使用，而且可以被当成栈来使用，因为该类里还包含了pop(出栈)、push(入栈)两个方法

### 3.2 Deque与Queue、Stack的关系

当Deque当作Queue队列使用时(FIFO)，添加元素是添加到队尾，删除时删除的是头部元素。从Queue接口继承的方法对应Deque的方法如图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbe29e94e887d71baa6894c53ddce343.png)
Deque 也能当Stack栈用（LIFO）。这时入栈、出栈元素都是在 双端队列的头部 进行。Deque 中和Stack对应的方法如图所示：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c266f27ea6824697887ac47462aa4110.png)

**注意：** Stack过于古老，并且实现地非常不好，因此现在基本已经不用了，可以直接用Deque来代替Stack进行栈操作。

### 3.3 ArrayDeque的本质

`循环数组`

ArrayDeque为了满足可以同时在数组两端插入或删除元素的需求，其内部的动态数组还必须是循环的，即循环数组（circular array），也就是说数组的任何一点都可能被看做起点或终点。

ArrayDeque维护了两个变量，表示ArrayDeque的头和尾
```java
//用数组存储元素
transient Object[] elements; // non-private to simplify nested class access
//头部元素的索引
transient int head;
//尾部下一个将要被加入的元素的索引
transient int tail;
//最小容量，必须为2的幂次方
private static final int MIN_INITIAL_CAPACITY = 8;
```

不过需要注意的是`tail`并不是尾部元素的索引，而是尾部元素的下一位，即下一个将要被加入的元素的索引。

当向头部插入元素时，head下标减一然后插入元素。而tail表示的索引为当前末尾元素表示的索引值加一。若当向尾部插入元素时，直接向tail表示的位置插入，然后tail再减一。

具体以下面图片为例解释![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5da3884a7c72167ec7efd6134cc20dcd.png)

在上图中：左边图表示从头部插入了4个元素，尾部插入了2个。在初始的时候，head=0，tail=0,。当从头部插入元素5，head-1，由于数组是循环数组，则移动到数组的最后位置插入5,。当从头部插入元素34，head-1然后再对应位置插入。

下面以此类推，最后在头部插入4个元素。

当在尾部插入12时，直接在0的位置插入，然后tail = tail + 1，当从尾部插入7时，直接在1的位置插入，然后 tail = tail + 1 = 2.最后队列中输出的顺序是 8,3,34,5,12,7

把数组看成一个相接的圆形数组更好理解循环数组的含义

下面具体看看ArrayDeque怎么把循环数组实际应用的？

`addFirst(E e)`为例来研究

```csharp
public void addFirst(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[head = (head - 1) & (elements.length - 1)] = e;
        if (head == tail)
            doubleCapacity();
    }
```

当加入元素时，先看是否为空（**ArrayDeque不可以存取null元素，因为系统根据某个位置是否为null来判断元素的存在**）。然后head-1插入元素。`head = (head - 1) & (elements.length - 1)`很好的解决了下标越界的问题。这段代码相当于取模，同时解决了head为负值的情况。因为elements.length必需是2的指数倍（代码中有具体操作），elements - 1就是二进制低位全1，跟head - 1相与之后就起到了取模的作用。如果head - 1为负数，其实只可能是-1，当为-1时，和elements.length - 1进行与操作，这时结果为elements.length - 1。其他情况则不变，等于它本身。

当插入元素后，在进行判断是否还有余量。因为tail总是指向下一个可插入的空位，也就意味着elements数组至少有一个空位，所以插入元素的时候不用考虑空间问题。

下面再说说扩容函数doubleCapacity()，其逻辑是申请一个更大的数组（原数组的两倍），然后将原数组复制过去。过程如下图所示：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/51443e4a18336abd425217a888bf7954.png)


图中我们看到，复制分两次进行，第一次复制head右边的元素，第二次复制head左边的元素。

```dart
//doubleCapacity()
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // head右边元素的个数
    int newCapacity = n << 1;//原空间的2倍
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);//复制右半部分，对应上图中绿色部分
    System.arraycopy(elements, 0, a, r, p);//复制左半部分，对应上图中灰色部分
    elements = (E[])a;
    head = 0;
    tail = n;
}
```

由此，我们便理解了ArrayDeque循环数组添加以及扩容的过程，其他操作类似。  
**注意：** ArrayDeque不是线程安全的。 当作为栈使用时，性能比Stack好；当作为队列使用时，性能比LinkedList好。

### 3.4 ArrayDeque常用方法

```java
 1.添加元素
        addFirst(E e)在数组前面添加元素
        addLast(E e)在数组后面添加元素
        offerFirst(E e) 在数组前面添加元素，并返回是否添加成功
        offerLast(E e) 在数组后天添加元素，并返回是否添加成功
 
  2.删除元素
        removeFirst()删除第一个元素，并返回删除元素的值,如果元素为null，将抛出异常
        pollFirst()删除第一个元素，并返回删除元素的值，如果元素为null，将返回null
        removeLast()删除最后一个元素，并返回删除元素的值，如果为null，将抛出异常
        pollLast()删除最后一个元素，并返回删除元素的值，如果为null，将返回null
        removeFirstOccurrence(Object o) 删除第一次出现的指定元素
        removeLastOccurrence(Object o) 删除最后一次出现的指定元素
   
 
   3.获取元素
        getFirst() 获取第一个元素,如果没有将抛出异常
        getLast() 获取最后一个元素，如果没有将抛出异常
   
 
    4.队列操作
        add(E e) 在队列尾部添加一个元素
        offer(E e) 在队列尾部添加一个元素，并返回是否成功
        remove() 删除队列中第一个元素，并返回该元素的值，如果元素为null，将抛出异常(其实底层调用的是removeFirst())
        poll()  删除队列中第一个元素，并返回该元素的值,如果元素为null，将返回null(其实调用的是pollFirst())
        element() 获取第一个元素，如果没有将抛出异常
        peek() 获取第一个元素，如果返回null
      
 
    5.栈操作
        push(E e) 栈顶添加一个元素
        pop(E e) 移除栈顶元素,如果栈顶没有元素将抛出异常
        
 
    6.其他
        size() 获取队列中元素个数
        isEmpty() 判断队列是否为空
        iterator() 迭代器，从前向后迭代
        descendingIterator() 迭代器，从后向前迭代
        contain(Object o) 判断队列中是否存在该元素
        toArray() 转成数组
        clear() 清空队列
        clone() 克隆(复制)一个新的队列
```

以上就是关于集合Queue的介绍。


