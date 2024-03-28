## 目录

- [1 List集合](#1%20List%E9%9B%86%E5%90%88)
	- [1.1 List集合判断元素相等的标准](#1.1%20List%E9%9B%86%E5%90%88%E5%88%A4%E6%96%AD%E5%85%83%E7%B4%A0%E7%9B%B8%E7%AD%89%E7%9A%84%E6%A0%87%E5%87%86)
	- [1.2 ListIterator](#1.2%20ListIterator)
- [2 ArrayList](#2%20ArrayList)
	- [2.1 ArrayList简介](#2.1%20ArrayList%E7%AE%80%E4%BB%8B)
	- [2.2 ArrayList的本质](#2.2%20ArrayList%E7%9A%84%E6%9C%AC%E8%B4%A8)
	- [2.3 ArrayList 和 Vector的区别](#2.3%20ArrayList%20%E5%92%8C%20Vector%E7%9A%84%E5%8C%BA%E5%88%AB)
	- [2.4 Stack](#2.4%20Stack)
	- [2.5 ArrayList的遍历方式](#2.5%20ArrayList%E7%9A%84%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
- [3 LinkedList](#3%20LinkedList)
	- [3.1 LinkedList简介](#3.1%20LinkedList%E7%AE%80%E4%BB%8B)
	- [3.2 LinkedList方法](#3.2%20LinkedList%E6%96%B9%E6%B3%95)
	- [3.3 LinkedList本质](#3.3%20LinkedList%E6%9C%AC%E8%B4%A8)
	- [3.4 LinkedList遍历方式](#3.4%20LinkedList%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
- [4 ArrayList和LinkedList性能对比](#4%20ArrayList%E5%92%8CLinkedList%E6%80%A7%E8%83%BD%E5%AF%B9%E6%AF%94)


![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240226225105771.png)
## 1 List集合

### 1.1 List集合判断元素相等的标准

List判断两个对象相等只要通过equals()方法比较返回true即可

下面以代码具体展示
创建一个Book类，并重写equals()方法，如果两个Book对象的name属性相同则认为两个对象相等。
```java
public class Book {
    public String name;

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Book other = (Book) obj;
        if (this.name == other.name) {
            return true;
        } 
        return false;
    }
}
```

向List集合中加入book1对象，然后调用remove(Object o)方法，从集合中删除指定对象，这个时候指定对象是book2
```java
public static void main(String[] args){
        Book book1 = new Book();
        book1.name = "Effective Java";
        Book book2 = new Book();
        book2.name = "Effective Java";
        List<Book> list = new ArrayList<Book>();
        list.add(book1);
        list.remove(book2);
        System.out.println(list.size());
    }
```

输出结果为0

可见把book1对象从集合中删除了，这表明List集合判断两个对象相等只要通过equals()方法比较返回true即可。  
与Set不同，List还额外提供了一个listIterator()方法，该方法返回一个ListIterator对象。下面具体介绍下ListIterator。

### 1.2 ListIterator

ListIterator接口在Iterator接口基础上增加了如下方法：

> **boolean hasPrevious(): **  如果以逆向遍历列表。如果迭代器有上一个元素，则返回 true。  
>  Object previous():** 返回迭代器的前一个元素。  
> **void add(Object o):** 将指定的元素插入列表（可选操作）。

与Iterator相比，ListIterator增加了前向迭代的功能，还可以通过add()方法向List集合中添加元素。

## 2 ArrayList

既然要介绍ArrayList，那么就顺带一起介绍Vector。因为二者的用法功能非常相似，可以一起了解比对。

### 2.1 ArrayList简介

ArrayList和Vector作为List类的两个典型实现类，完全支持之前介绍的List接口的全部功能。

ArrayList和Vector类都是基于数组实现的List类，所以ArrayList和Vector类封装了一个动态的，允许再分配的Object[]数组。

ArrayList和Vector对象使用initalCapacity参数来设置改数组的长度，当向ArrayList和Vector添加元素超过了该数组的长度时，它们的initialCapacity会自动增加。

下面我们通过阅读JDK 1.8 ArrayList源码来了解这些内容。

### 2.2 ArrayList的本质

当以`List<Book> list = new ArratList<>(3);`方式创建ArrayList集合时
```java
//动态Object数组，用来保存加入到ArrayList的元素
Object[] elementData;

//ArrayList的构造函数，传入参数为数组大小
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
             //创建一个对应大小的数组对象
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //传入数字为0，将elementData 指定为一个静态类型的空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

当以`List<Book> list = new ArrayList<Book>();`方式创建ArrayList集合时，不指定集合的大小

```cpp
/**
     *Constructs an empty list with an initial capacity of ten。意思是：构造一个空数组，默认的容量为10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

在这里可以看出`private static final int DEFAULT_CAPACITY = 10;`默认容量确实为10。  
当向数组中添加元素`list.add(book1);`时：  
先调用add(E e)方法

```java
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 数组的大小增加1
        elementData[size++] = e;
        return true;
    }
```

在该方法中，先调用了一个ensureCapacityInternal()方法，顾名思义：该方法用来确保数组中是否还有足够容量。  
经过一系列方法（不必关心），最后有个判断：如果剩余容量足够存放这个数据，则进行下一步，如果不够，则需要执行一个重要的方法：

```cpp
 private void grow(int minCapacity) {
        //......省略部分内容  主要是为了生成大小合适的newCapacity
       //下面这行就是进行了数组扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

**由此，我们就清楚地明白了，ArrayList是一个动态扩展的数组，Vector也同样如此。**  
如果开始就知道ArrayList或Vector集合需要保存多少个元素，则可以在创建它们时就指定initalCapacity的大小，这样可以提高性能。  
此外，ArrayList还提供了两个额外的方法来调整其容量大小：

> **void ensureCapacity(int minCapacity):** 如有必要，增加此 ArrayList 实例的容量，以确保它至少能够容纳最小容量参数所指定的元素数。  
> **void trimToSize():** 将此 ArrayList 实例的容量调整为列表的当前大小。

### 2.3 ArrayList 和 Vector的区别

1. ArrayList是线程不安全的，Vector是线程安全的。  
2. Vector的性能比ArrayList差。

### 2.4 Stack

Stack是Vector的子类，用户模拟“栈”这种数据结构，“栈”通常是指FILO的容器。最后“push”进栈的元素，将被最先“pop”出栈。如图所示：

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240226223928088.png)

Stack类里提供了如下几个方法：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240226223946060.png)


Stack与Vector一样，是线程安全的，但是性能较差，尽量少用Stack类。如果要实现栈”这种数据结构，可以考虑使用LinkedList(下面就会介绍)。

### 2.5 ArrayList的遍历方式

ArrayList支持3种遍历方式  

**(01) 第一种，通过迭代器遍历**

```csharp
Integer value = null;
Iterator iter = list.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```

**(02) 第二种，随机访问，通过索引值去遍历**  
由于ArrayList实现了RandomAccess接口，它支持通过索引值去随机访问元素。

```csharp
Integer value = null;
int size = list.size();
for (int i=0; i<size; i++) {
    value = (Integer)list.get(i);        
}
```

**(03) 第三种，for循环遍历**

```csharp
Integer value = null;
for (Integer integ:list) {
    value = integ;
}
```

**遍历ArrayList时，使用随机访问(即，通过索引序号访问)效率最高，而使用迭代器的效率最低。** 具体可以测试下。

## 3 LinkedList

### 3.1 LinkedList简介

LinkedList类是List接口的实现类——这意味着它是一个List集合，可以根据索引来随机访问集合中的元素。除此之外，LinkedList还实现了Deque接口，可以被当做成双端队列来使用，因此既可以被当成“栈”来使用，也可以当成队列来使用。

LinkedList的实现机制与ArrayList完全不同。ArrayList内部是以数组的形式来保存集合中的元素的，因此随机访问集合元素时有比较好的性能；而LinkedList内部以链表的形式来保存集合中的元素，因此随机访问集合元素性能较差，但在插入、删除元素时性能比较出色。

由于LinkedList双端队列的特性，所以新增了一些方法

### 3.2 LinkedList方法

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240226224356629.png)

### 3.3 LinkedList本质

LinkedList调用默认构造函数，创建一个链表。由于维护了一个表头，表尾的Node对象的变量。可以进行后续的添加元素到链表中的操作，以及其他删除，插入等操作。也因此实现了双向队列的功能，即可向表头加入元素，也可以向表尾加入元素

```java
//成员变量：表头，表尾
transient Node<E> first;
transient Node<E> last;
//默认构造函数，表示创建一个空链表
public LinkedList() {
    }
```

下面来了解Node类的具体情况

```cpp
private static class Node<E> {
        //表示集合元素的值
        E item;
       //指向下个元素
        Node<E> next;
     //指向上个元素
        Node<E> prev;
...................................省略
    }
```

由此可以具体了解链表是如何串联起来并且每个节点包含了传入集合的元素。  
下面以增加操作，具体了解LinkedList的工作原理。

```java
public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

调用`linkLast(e);`方法，默认向表尾节点加入新的元素

```java
 void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

更新表尾节点，建立连接。其他操作类似，维护了整个链表。  
下面具体来看，**如何将“双向链表和索引值联系起来的”？**

```cpp
 public E get(int index) {
        checkElementIndex(index);//检查索引是否有效
        return node(index).item;
    }
```

调用了`node(index)`方法返回了一个Node对象，其中`node(index)`方法具体如下

```csharp
Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

首先会比较“index”和“双向链表长度的1/2”；若前者小，则从链表头开始往后查找，直到index位置；否则，从链表末尾开始先前查找，直到index位置。这就是“双线链表和索引值联系起来”的方法。  
到此我们便会明白，LinkedList在插入、删除元素时性能比较出色，随机访问集合元素时性能较差。

### 3.4 LinkedList遍历方式

LinkedList支持多种遍历方式。  
1.通过迭代器遍历LinkedList  
2通过快速随机访问遍历LinkedList  
3.通过for循环遍历LinkedList  
4.通过pollFirst()遍历LinkedList  
5.通过pollLast()遍历LinkedList  
6通过removeFirst()遍历LinkedList  
7.通过removeLast()遍历LinkedList  
实现都比较简单，就不贴代码了。  

**其中采用逐个遍历的方式，效率比较高。采用随机访问的方式去遍历LinkedList的方式效率最低。**

**LinkedList也是非线程安全的。**

## 4 ArrayList和LinkedList性能对比

ArrayList 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低。ArrayList应使用随机访问(即，通过索引序号访问)遍历集合元素。  
LinkedList 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率高。LinkedList应使用采用逐个遍历的方式遍历集合元素。  
如果涉及到“动态数组”、“栈”、“队列”、“链表”等结构，应该考虑用List，具体的选择哪个List，根据下面的标准来取舍。  
(01) 对于需要快速插入，删除元素，应该使用LinkedList。  
(02) 对于需要快速随机访问元素，应该使用ArrayList。  
(03) 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList)。对于“多线程环境，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector)。


