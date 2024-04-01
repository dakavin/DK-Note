## 目录

- [1 前言](#1%20%E5%89%8D%E8%A8%80)
- [2 List](#2%20List)
	- [2.1 Vectory](#2.1%20Vectory)
	- [2.2 ArrayList](#2.2%20ArrayList)
	- [2.3 LinkedList](#2.3%20LinkedList)
- [3 Set](#3%20Set)
	- [3.1 HashSet](#3.1%20HashSet)
- [4 Map](#4%20Map)
	- [4.1 hashMap](#4.1%20hashMap)
	- [4.2 HashTable](#4.2%20HashTable)
	- [4.3 ConcurrentHashMap](#4.3%20ConcurrentHashMap)

## 1 前言

Java集合框架Collection是由一套设计优良的接口和类组成的，使我们操作成批的数据或对象元素极为方便，这些接口和类有很多对抽象数据类型操作的API。并且Java用面向对象的设计对这些数据结构和算法进行了封装，这就极大的减化了我们在编程上的工作量。

下面的图是一张Java Collection的UML （来源：https://blog.csdn.net/u010887744/article/details/50575735）

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d63c2d645acfc43e9edd01d6dcda60ea.png)

我们常用的类有如下几个：Vector, ArrayList, LinkedList, HashSet,HashTable, HashMap

Vector, ArrayList, LinkedList都实现的List接口，继承的AbstractList抽象类

HashSet实现的是set接口，继承的AbstractSet抽象类

HashMap实现的是map接口，继承的AbstractMap抽象类，

HashTable实现的是map接口，继承的Dictionary类

## 2 List

List存储一组不唯一，有序（按插入顺序存储）的对象，类似于Java数组，能够通过索引（元素在List中位置，类似于数组的下标）来访问List中的元素

### 2.1 Vectory

内部维护了一个Object数组，在源码中Vectory定义了一个Object[]的成员变量

protectedObject[] elementData;

可以插入重复的元素，在源码中可以看到，只要数组有空间，就直接加到数组的下一个单元
```java
public synchronized void add(E obj){
	modCount++;

	ensureCapacityHelper(elementCount + 1);

	elementData[elementCount++] = obj;
}
```

存储的顺序和插入的顺序一致，因为，Vectory的内部实现就是一个对象数组，如果我们不指定插入的位置，`默认就是将新元素插入到尾部`

查询速度快，可以通过索引快速随机读取

添加、删除慢，需要移动数据

线程安全，Vectory里面的所有函数都添加了synchronized关键字，`Vectory是一个线程安全的Collection`

**Vectory的遍历方式**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c748b543d6ba0bf0e5424a077623a3b2.png)


### 2.2 ArrayList

和Vectory一样，内部实现也是一个Object数组，用来存储一组不唯一，有序的对象，不同的是，ArrayList是非同步的，所以在单线程环境中一般使用ArrayList，多线程环境中使用Vectory

内部维护了一个Object数组，在源码中ArrayList定义了一个Object[]的成员变量

private transientObject[] elementData;

可以插入重复的元素，在源码中可以看到，只要数组有空间，就直接加到数组的下一个单元
```java
public boolean add(E e){
	ensureCapacity(size + 1);

	elementData[size++] = e;

	return true;
}
```

存储的顺序和插入的顺序一致

询速度快，可以通过索引快速随机读取

添加、删除慢，需要移动数据

线程不安全，ArrayList里面的所有函数都没有同步

**ArrayList的遍历方法**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b2f264d279cd75ed14ac5435e32337e.png)

### 2.3 LinkedList

LinkedList和ArrayList,Vectory都不一样，它是一个双向链表。用来存储存储一组不唯一，有序（按插入顺序存储）的对象

在LinkedList内部维护了2个Node结点，一个Prev,一个next.

```java
transient Node<E> first; //链表的头结点

transient Node<E> last; //链表的尾节点
```

可以插入重复的元素，而且存储的顺序和插入的顺序一至，在源码中每次添加新元素都是添加到末尾

```java
public boolean add(E e) {

    final Node<E> l = last;

    final Node<E> newNode = new Node<>(l, e, null);

    last = newNode;

    if (l == null)

        first = newNode;_//原来链表为空，这是插入的第一个元素

    else

        l.next = newNode;

    size++;

    return true;

}
```

查询慢,需要移动指针来遍历

添加,删除较快

线程不安全，和LinkedList所有函数都没有同步

**LinkedList的遍历方法：**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b142b1b57178d7fbf58cf471d1aa13b.png)

## 3 Set

是一种`无序，不包含重复元素的Collection`, Set和 List虽然都是存储的对象但是实现方式却完全不一样，List是以数组为基础实现的而set是以hashMap为基础实现的
### 3.1 HashSet

内部`通过HashMap实现`，从源码中可以看到，内部维护了一个HashMap的对象

private transient HashMap map;

不允许重复元素插入(当插入重复元素后，会将原有的元素的值都覆盖掉)，从源码中可以看到当有新元素添加到map中时，会加到map的key中,而value用object代替

```java
private static final Object PRESENT = new Object(); //hashMap中的value

public boolean add(E e) {

	return map.put(e, PRESENT)==null;

}
```

元素插入的顺序和存储的顺序不一至

允许null元素

没有做线程安全，因为所有函数都没有同步

**HashSet的遍历方法：**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d0235feac4b7fdc4c303aba8aa7730d.png)


## 4 Map


Map是一种`储存键值对 (key-value)对象的collection`,在key-value的结果储存中，key不能重复，value可以重复。

### 4.1 hashMap

是一个散列表，通过链地址法来解决冲突。

内部实现是个Entry(Entry是一个可以构成单向链表的类)的数组，在源码中内部定义了一个Entry成员变量：

	private transient Entry[] table;

使用链地址法来处理冲突，在源码中可以发现HashMap是先通过hash(key)生成hash值，然后通过hash值计算存储的位置

	int hash = hash(key);

	int index = hash &(table.length-1);

HashMap在添加新元素时，找到了储存的数组位置 index后会遍历链表tab[index]，如果在链表tab[index]中找到相同的key,替换该key对应的value,找不到新建一个Entry,并插入到tab[index]链表

```java
for (Entry e =table[index]; e != null; e = e.next) {

	Object k；

	if (e.hash == hash && ((k =e.key) == key || key.equals(k))) {

		V oldValue =e.value;

		e.value = value;

		e.recordAccess(this);

		return oldValue;

	}

}

modCount++;

addEntry(hash, key, value, index);
```

元素插入的顺序和存储的顺序不一致

key允许为null(但只能有一条并且放在tbl[0]，如果再写入会将第一次的value覆盖掉)

非线程安全，因为所有函数都没有同步

**HashMap的遍历方法：**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/579fa667d6bd2dde8126a30bf3b993a2.png)



### 4.2 HashTable

是一个散列表，用来储存键值对(key-value)对像的collection。它是继承自Dictionary,实现的Map接口。

内部实现是个Entry的数组和HashMap一样，在内部维护了一个Entry数组的成员变量：

	private transient Entry[] table;

使用链地址法来处理冲突， 和HashMap一样也是先通过hash(key)生成hash值，然后通过hash值计算存储的位置。但是不一样的是他们的index计算方式，`HashMap是用Hash值 “与”运算数组的长度-1，而HashTable是Hash值除数组的长度取余数`。

	int hash = hash(key);
	
	int index = (hash &0x7FFFFFFF) % tab.length;

添加元素和HashMap一样，找到了储存的数组位置 index后会遍历链表tab[index]，如果在链表tab[index]中找到相同的key,替换该key对应的value,找不到新建一个Entry,并插入到tab[index]链表中

	Entry e = tab[index];
	
	tab[index] = newEntry<>(hash, key, value, e);

元素插入的顺序和存储的顺序不一致

key和value都不能插入null

`线程安全，因为HashTable的函数都是加了synchronized`

public synchronized V put(K key, V value){

。。。。}

**HashTable的遍历方法：**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0f0fe324f0fe1f303f71b7215105a99d.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c73abec9568dc2671a2eb02c787ff1ce.png)


从上面的遍历方式我们可以看出来，HashTable比HashMap多了Elements的遍历，所以HashTable和HashMap继承类是不一样的。

### 4.3 ConcurrentHashMap

和HashMap,HashTable一样都是储存键值对(key-value)对像的collection，HashMap是非线程安全，而HashTable是线程安全，所以在多线程中，我们一般使用HashTable,但HashTable是对整个hash表结构做锁定操作的，这样在锁表的期间，别的线程就需要等待了，性能不是很好。

`ConcurrentHashMap就解决HashTable的缺陷，因为在ConCurrentHashMap中，加锁是采用的分段加锁技术`。

ConcurrentHashMap内部是由一个Segment数组和一个HashEntry链表构成的，在每一个Segment段都设置了一把锁，所以对于不同Segment的数据进行操作是不用考虑锁竞争的

	final Segment[] segments;

segment类似于hashmap,内部实现是一个HashEntry数组

`transient volatileHashEntry[] table;(HashEntry是一个可以构成单向链表的类)

ConcurrentHashMap的冲突解决方式和HashTable, HashMap都一样，先计算key的hash值，然后将该hash值右移segmentShift位（segmentShift  = 32 - segments.length 2次开方的值）然后 “与”运算segmentMask（segmentMask  = segments.length -1）

`int hash = hash(key);

`int j = (hash>>> segmentShift) & segmentMask;

在ConcurrentHashMap中执行put操作时，ConcurrentHashMap就会调用Segment的put操作，但这个时候并不加锁，只是确定Segment的index位置。

```java
public V put(K key, V value) {

Segment s;

。。。。。。

return s.put(key, hash,value, false);

}
```

在Segment中执行put操作时，1:加锁，2:添加到HashEntry的链表中，3:释放锁

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {

HashEntrynode = tryLock() ? null : scanAndLockForPut(key, hash, value);

Try{

	。。。。       

}finally {

	unlock();

}

}
```

key和value都不能插入null

不允许重复的key插入(如果重复key对应的value会被覆盖)，value可重复

**ConcurrentHashMap的遍历方法：**
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/98d4909ab354ae0e6eac2011018b100e.png)




  
  
