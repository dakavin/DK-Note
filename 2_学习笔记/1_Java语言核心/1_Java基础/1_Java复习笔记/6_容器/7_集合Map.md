<h1>目录</h1>
- [1 HashMap](#1%20HashMap)
- [2 HashMap和Hashtable的区别](#2%20HashMap%E5%92%8CHashtable%E7%9A%84%E5%8C%BA%E5%88%AB)
- [3 HashMap判断key和value相等的标准](#3%20HashMap%E5%88%A4%E6%96%ADkey%E5%92%8Cvalue%E7%9B%B8%E7%AD%89%E7%9A%84%E6%A0%87%E5%87%86)
	- [3.1 key判断相等的标准](#3.1%20key%E5%88%A4%E6%96%AD%E7%9B%B8%E7%AD%89%E7%9A%84%E6%A0%87%E5%87%86)
	- [3.2 value判断相等的标准](#3.2%20value%E5%88%A4%E6%96%AD%E7%9B%B8%E7%AD%89%E7%9A%84%E6%A0%87%E5%87%86)
- [4 HashMap的本质](#4%20HashMap%E7%9A%84%E6%9C%AC%E8%B4%A8)
	- [4.1 HashMap的构造函数](#4.1%20HashMap%E7%9A%84%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
	- [4.2 Node类型](#4.2%20Node%E7%B1%BB%E5%9E%8B)
	- [4.3 HashMap遍历方式](#4.3%20HashMap%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
	- [4.4 LinkedHashMap](#4.4%20LinkedHashMap)
- [5 TreeMap](#5%20TreeMap)
	- [5.1 TreeMap的排序方式](#5.1%20TreeMap%E7%9A%84%E6%8E%92%E5%BA%8F%E6%96%B9%E5%BC%8F)
	- [5.2 TreeMap中判断两个元素key、value相等的标准](#5.2%20TreeMap%E4%B8%AD%E5%88%A4%E6%96%AD%E4%B8%A4%E4%B8%AA%E5%85%83%E7%B4%A0key%E3%80%81value%E7%9B%B8%E7%AD%89%E7%9A%84%E6%A0%87%E5%87%86)
	- [5.3 TreeMap的本质](#5.3%20TreeMap%E7%9A%84%E6%9C%AC%E8%B4%A8)
	- [5.4 TreeMap遍历方式](#5.4%20TreeMap%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
- [6 Map实现类的性能分析及适用场景](#6%20Map%E5%AE%9E%E7%8E%B0%E7%B1%BB%E7%9A%84%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E5%8F%8A%E9%80%82%E7%94%A8%E5%9C%BA%E6%99%AF)

## 1 HashMap

HashMap是一个散列表，它存储的内容是键值对（key-value）映射。

既然要介绍HashMap，那么就顺带介绍Hashtable，两者进行对比。

HashMap和Hashtable都是Map接口的经典实现类，它们之间的关系完全类似于之前介绍的ArrayList和Vector的关系。由于Hashtable是个古老的Map实现类（从HashTable的命名规范就可以看出，t没有大写），需要方法比较繁琐，不符合Map接口的规范。但是Hashtable也具有HashMap不具有的有点。下面我们进行两者之间的比对。

## 2 HashMap和Hashtable的区别

1. Hashtable是一个线程安全的Map实现，但HashMap是线程不安全的实现，所有HashMap比Hashtable的性能好一些；但如果有多个线程访问同一个Map对象，采用Hashtable实现类会更好
2. Hashtable不允许使用null作为key和value，否则会发生NullPointException异常；但是HashMap可以使用null作为key和value

## 3 HashMap判断key和value相等的标准

### 3.1 key判断相等的标准

类似于HashSet，HashMap与Hashtable判断两个key相等的标准：两个key通过equals()方法比较返回true，两个key的哈希值也相等，则认为两个key 是相等的

`注意：`用作key的对象必须实现了hashCode()方法和equals()方法。并且最好两者返回的结果一致，即如果equals()返回true，hashCode()相等。

### 3.2 value判断相等的标准

HashMap与Hashtable判断两个value相等的标准：两个value通过equals()方法比较返回true即可

`注意：`HashMap中key所组成的集合元素不能重复，value所组成的集合元素可以重复

下面程序示范了HashMap判断key与value相等的标准
```java
public class A {
    public int count;

    public A(int count) {
        this.count = count;
    }
    //根据count值来计算hashCode值
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + count;
        return result;
    }
    //根据count值来判断两个对象是否相等
    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        A other = (A) obj;
        if (count != other.count)
            return false;
        return true;
    }
    

}
```

```java
public class B {
    public int count;
    public B(int count) {
        this.count = count;
    }
     //根据count值来判断两个对象是否相等
    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        B other = (B) obj;
        if (count != other.count)
            return false;
        return true;
    }

}
```

```java
public class HashMapTest {
    public static void main(String[] args){
        HashMap map = new HashMap();
        map.put(new A(1000), "集合Set");
        map.put(new A(2000), "集合List");
        map.put(new A(3000), new B(1000));
       //仅仅equals()比较为true，但认为是相同的value
        boolean isContainValue = map.containsValue(new B(1000));
        System.out.println(isContainValue);
      //虽然是不同的对象，但是equals()和hashCode()返回结果都相等
        boolean isContainKey = map.containsKey(new A(1000));
        System.out.println(isContainKey);
      //equals()和hashCode()返回结果不满足key相等的条件
        System.out.println(map.containsKey(new A(4000)));
    }
    
}
```
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240227013030975.png)

`注意：` 如果是加入HashMap的key是个可变对象，在加入到集合后又修改key的成员变量的值，可能导致hashCode()值以及equal()的比较结果发生变化，无法访问到该key。一般情况下不要修改。

## 4 HashMap的本质

下面我们从源码角度来理解HashMap

### 4.1 HashMap的构造函数

```dart
// 默认构造函数。
HashMap()

// 指定“容量大小”的构造函数
HashMap(int capacity)

// 指定“容量大小”和“加载因子”的构造函数
HashMap(int capacity, float loadFactor)

// 包含“子Map”的构造函数
HashMap(Map<? extends K, ? extends V> map)
```

从构造函数中，理解到两个重要的元素：容量大小（capacity）以及加载因子（loadFactor）

容量大小（capacity）是哈希表的容量，初始容量是哈希表在创建时的容量（即`DEFAULT_INITIAL_CAPACITY = 1 << 4`）

加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超过了 加载因子与点前容量的乘积时，则要对该哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数

通常，默认加载因子是0.75（即`DEFAULT_LOAD_FACTOR = 0.75f`），这是在时间和空间成本上寻求的一种折中。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数HashMap类的操作中，包括get和put操作，都反映了这一点）。在设置容量时应该考虑到映射中所需的条目数以及加载因子，以便最大限度地减少resize操作。如果容量大于最大条目数除以加载因子，则不会发生rehash操作
### 4.2 Node类型

HashMap是通过“拉链法”实现的哈希表。它包括几个重要的成员变量：table，size，threshold，loadFactor。

table是一个Node[]数组类型，而Node实际上就是一个单向链表。哈希表中的键值对都是存储在Node数组中的

size是HashMap的大小，它是HashMap保存的键值对的数量

threshold是HashMap的阈值，用于判断是否需要调整HashMap的容量。threshold的值 = “容量 * 加载因子” ， 当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍

loadFactor就是加载因子

要想理解HashMap，首先就要理解基于Node实现的“拉链法”。

Java中数据存储方式最底层的两种结构，一种是数组，一种就是链表。

数组的特点：连续空间，寻址迅速，但在删除或添加元素的时候需要有较大幅度的移动，所以查询快，增删慢

而链表正好相反，由于空间不连续，寻址困难，增删元素只需要修改指针，所以查询速度慢，增删快。

有没有一种数据结构来综合一下数组和链表，以发挥它们各自的优势？

`哈希表！`

哈希表具有较快（常量级）的查询速度，以及相对较快的增删速度，所以很适合在海量数据的环境中使用，一般实现哈希表的方法采用“拉链法”，我们可以理解为“链表的数组”，如下图 
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240227014321592.png)

图中，我们可以发现哈希表是由数组+链表組成的，一个长度为16的数组中，每個元素存储的是一个链表的头结点。那么这些元素是按照什么样的规则存储到数组中呢？  
一般情況是通过hash(key)获得，也就是元素的key的哈希值。如果hash(key)值相等，则都存入该hash值所对应的链表中。它的內部其实是用一個Node数组來实现。

**所以每个数组元素代表一个链表，其中的共同点就是hash(key)相等。**

下面我们来了解下链表的基本元素Node。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        // 指向下一个节点
        Node<K,V> next;
        //构造函数。
      // 输入参数包括"哈希值(hash)", "键(key)", "值(value)", "下一节点(next)"
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
         // 判断两个Node是否相等
        // 若两个Node的“key”和“value”都相等，则返回true。
        // 否则，返回false
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

再此结构下，实现了集合的增删改查功能，由于本篇的篇幅有限，这里就不具体介绍其源码实现了。

### 4.3 HashMap遍历方式

1.遍历HashMap的键值对

第一步：根据entrySet()获取HashMap的“键值对”的Set集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

2.遍历HashMap的键

第一步：根据keySet()获取HashMap的“键”的Set集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

3.遍历HashMap的值

第一步：根据value()获取HashMap的“值”的集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

### 4.4 LinkedHashMap

HashSet有一个LinkedHashSet子类，HashMap也有一个LinkedHashMap子类；

LinkedHashMap使用双向链表来维护key-value对的次序。  

LinkedHashMap需要维护元素的插入顺序，因此性能略低于HashMap的性能；但是因为它以链表来维护内部顺序，所以在迭代访问**Map里的全部元素时有较好的性能**。迭代输出LinkedHashMap的元素时，将会按照添加key-value对的顺序输出。  

**本质上来讲，LinkedHashMap=散列表+循环双向链表**

## 5 TreeMap

TreeMap是SortedMap接口的实现类。TreeMap 是一个**有序的key-value集合**，它是通过红黑树实现的，每个key-value对即作为红黑树的一个节点。

### 5.1 TreeMap的排序方式

TreeMap有两种排序方式，和TreeSet一样。

`自然排序`：TreeMap的所有key必须实现Comparable接口，而且所有的key应该是同一个类的对象，否则会抛出ClassCastException异常。

`定制排序`：创建TreeMap时，传入一个Comparator对象，该对象负责对TreeMap中的所有key进行排序。

### 5.2 TreeMap中判断两个元素key、value相等的标准

类似于TreeSet中判断两个元素相等的标准，TreeMap中判断两个key相等的标准是：两个key通过compareTo()方法返回0，TreeMap即认为这两个key是相等的。

TreeMap中判断两个value相等的标准是：两个value通过equals()方法比较返回true。

**注意：**如果使用自定义类作为TreeMap的key，且想让TreeMap良好地工作，则重写该类的equals()方法和compareTo()方法时应保持一致的返回结果：两个key通过equals()方法比较返回true时，它们通过compareTo()方法比较应该返回0。如果两个方法的返回结果不一致，TreeMap与Map接口的规则就会冲突。

除此之外，与TreeSet类似，TreeMap根据排序特性，也添加了一部分新的方法，与TreeSet中的一致。可以参考前面的文章。
### 5.3 TreeMap的本质

`红黑树`

R-B Tree，全称是Red-Black Tree，又称为“红黑树”，它一种特殊的二叉查找树。红黑树的每个节点上都有存储位表示节点的颜色，可以是红(Red)或黑(Black)。

红黑树的特性:  
（1）每个节点或者是黑色，或者是红色。  
（2）根节点是黑色。  
（3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]  
（4）如果一个节点是红色的，则它的子节点必须是黑色的。  
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

注意：  
(01) 特性(3)中的叶子节点，是只为空(NIL或null)的节点。  
(02) 特性(5)，确保没有一条路径会比其他路径长出俩倍。因而，红黑树是相对是接近平衡的二叉树。

**红黑树的时间复杂度为: O(log n)**  
更多关于红黑树的增删改查操作，可以参考[这篇文章](https://link.jianshu.com/?t=http://www.cnblogs.com/skywang12345/p/3245399.html)。

**可以说TreeMap的增删改查等操作都是在一颗红黑树的基础上进行操作的。**
### 5.4 TreeMap遍历方式

遍历TreeMap的键值对

第一步：根据entrySet()获取TreeMap的“键值对”的Set集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

遍历TreeMap的键

第一步：根据keySet()获取TreeMap的“键”的Set集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

遍历TreeMap的值

第一步：根据value()获取TreeMap的“值”的集合。  
第二步：通过Iterator迭代器遍历“第一步”得到的集合。

## 6 Map实现类的性能分析及适用场景

HashMap与Hashtable实现机制几乎一样，但是HashMap比Hashtable性能更好些。  
LinkedHashMap比HashMap慢一点，因为它需要维护一个双向链表。  
TreeMap比HashMap与Hashtable慢（尤其在插入、删除key-value时更慢），因为TreeMap底层采用红黑树来管理键值对。  
**适用场景：**  
一般的应用场景，尽可能多考虑使用HashMap，因为其为快速查询设计的。  
如果需要特定的排序时，考虑使用TreeMap。  
如果仅仅需要插入的顺序时，考虑使用LinkedHashMap。
