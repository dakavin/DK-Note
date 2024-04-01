## 目录

- [1 Java集合类简介](#1%20Java%E9%9B%86%E5%90%88%E7%B1%BB%E7%AE%80%E4%BB%8B)
- [2 Java集合和数组的区别](#2%20Java%E9%9B%86%E5%90%88%E5%92%8C%E6%95%B0%E7%BB%84%E7%9A%84%E5%8C%BA%E5%88%AB)
- [3 Java集合类之间的继承关系](#3%20Java%E9%9B%86%E5%90%88%E7%B1%BB%E4%B9%8B%E9%97%B4%E7%9A%84%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB)
- [4 Collection接口](#4%20Collection%E6%8E%A5%E5%8F%A3)
	- [4.1 简介](#4.1%20%E7%AE%80%E4%BB%8B)
	- [4.2 接口中定义的方法](#4.2%20%E6%8E%A5%E5%8F%A3%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%B9%E6%B3%95)
	- [4.3 使用Iterator遍历集合元素](#4.3%20%E4%BD%BF%E7%94%A8Iterator%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E5%85%83%E7%B4%A0)
- [5 Set集合](#5%20Set%E9%9B%86%E5%90%88)
	- [5.1 简介](#5.1%20%E7%AE%80%E4%BB%8B)
- [6 List集合](#6%20List%E9%9B%86%E5%90%88)
	- [6.1 简介](#6.1%20%E7%AE%80%E4%BB%8B)
	- [6.2 接口中定义的方法](#6.2%20%E6%8E%A5%E5%8F%A3%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%B9%E6%B3%95)
- [7 Queue集合](#7%20Queue%E9%9B%86%E5%90%88)
	- [7.1 简介](#7.1%20%E7%AE%80%E4%BB%8B)
	- [7.2 接口中定义的方法](#7.2%20%E6%8E%A5%E5%8F%A3%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%B9%E6%B3%95)
- [8 Map集合](#8%20Map%E9%9B%86%E5%90%88)
	- [8.1 简介](#8.1%20%E7%AE%80%E4%BB%8B)
	- [8.2 与Set集合、List集合的关系](#8.2%20%E4%B8%8ESet%E9%9B%86%E5%90%88%E3%80%81List%E9%9B%86%E5%90%88%E7%9A%84%E5%85%B3%E7%B3%BB)
	- [8.3 接口中定义的方法](#8.3%20%E6%8E%A5%E5%8F%A3%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%B9%E6%B3%95)

## 1 Java集合类简介

Java集合大致可以风分为Set、List、Queue和Map四种体系，其中Set代表无序、不可重复的集合；List代表有序、重复的集合；而Map则代表具有映射关系的集合，Java5又增加了Queue体系集合，代表一种队列集合实现。

Java集合就像一种容器，可以把多个对象（实际是对象的引用，当习惯上都称为对象）“丢进”该容器中。从Java5增加了泛型以后，Java集合可以记住容器中对象的数据类型，使得代码更加简洁、健壮。

## 2 Java集合和数组的区别

1. 数组长度在初始化时固定，意味着只能保存定长的数据。而集合可以保存数量不确定的数据。同时可以保存具有映射关系的数据（即关联数组，键值对Key-Value）

2. 数组元素即可以是基本类型的值，也可以是对象。集合里只能保存对象（实际上只是保存对象的引用变量），基本数据类型的变量要转换成对应的包装类才能放入集合类中。

## 3 Java集合类之间的继承关系

Java的集合类主要有两个接口派生而出：Collection 和 Map，Collection和Map是Java集合框架的根接口。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/731d35063988ecb686280afa8cbd0cee.png)

图中，`ArrayList、HashSet、LinkedList、TreeSet`是我们经常会有用到的已实现的集合类。

Map实现类用于保存具有映射关系的数据。Map保存的每项数据都是key-value对，也就是有key和value两个值组成。Map里的key是不可重复的，key用户标识集合里的每项数据。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa34bff4163aad8b143bb4541301b744.png)

图中，`HashMap、TreeMap`是我们经常会用到的集合类。

## 4 Collection接口

### 4.1 简介

Collection接口是Set、Queue、List的父接口。

Collection接口中定义了多种方法可供子类进行实现，以实现数据操作。由于方法比较多，直接把JDK问到上的内容搬过来

### 4.2 接口中定义的方法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7b218ff5210c2d422680eacb62bfdb29.png)

可以看出Collection用法有：添加元素、删除元素、返回Collection集合的个数以及清空集合等。

其中重点介绍iterator()方法，该方法的方绘制是`Iterator<E>`

### 4.3 使用Iterator遍历集合元素

Iterator接口经常被称作迭代器，它是Collection接口的父接口。但Iterator主要用于遍历集合中的元素。

Iterator接口中主要定义了2个方法：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1103b2153c9ca88412b58ce360e2baf0.png)

下面程序简单示范了通过Iterator对象逐个获取元素的逻辑
```java
public class IteratorExample {  
    public static void main(String[] args) {  
        // 常见集合，添加元素  
        Collection<Integer> integers = new ArrayList<>();  
        for (int i = 0; i < 10; i++) {  
            integers.add(i);  
        }  
  
        // 获取integers集合的迭代器  
        Iterator<Integer> iterator = integers.iterator();  
        while (iterator.hasNext()){ //判断是否有下一个元素  
            //逐个遍历，取得元素后进行后续操作  
            System.out.println(iterator.next()); //取出该元素  
        }  
    }  
}
```

*注意*：当使用Iterator对集合元素进行迭代时，Iterator并不是把集合元素本身传递给了迭代变量，而是把集合元素的值传递给了迭代变量（就如同参数传递是值传递，基本数据类型传递的是值，引用类型传递的仅仅是对象的引用变量），所以`修改迭代变量的值对集合元素本身没有任何影响`。

下面的程序演示了这一点：
```java
public class IteratorExample1 {  
    public static void main(String[] args) {  
        List<String> list = Arrays.asList("Java语音","C语音","C++语音");  
  
        Iterator<String> iterator = list.iterator();  
        while (iterator.hasNext()){  
            String next = iterator.next();  
            //集合元素的值传给了迭代变量，仅仅传递了对象引用。保存的仅仅是指向对象内存空间的地址  
            next = "修改后的";  
            System.out.println(next);  
        }  
        System.out.println(list);  
    }  
}
```

输出结果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7505801827ff3149e869405efa0d244f.png)
## 5 Set集合

### 5.1 简介

Set集合与Collection集合基本相同，没有提供任何额外的方法。实际上Set就是Collection，只是行为略有不同（Set不允许包含重复元素）。

Set集合不允许包含相同的元素，如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()方法返回false，且新元素不会被加入
## 6 List集合

### 6.1 简介

List集合代表一个元素有序、可重复的集合，集合中每个元素都有其对应的顺序索引。

List集合允许使用重复元素，可以通过索引来访问指定位置的集合元素，List集合默认按集合的添加顺序位置设置索引。

List作为Collection接口的子接口，可以使用Collection接口里的全部方法。而且由于List是有序集合，因此List集合里增加了一些根据索引来操作集合元素的方法。

### 6.2 接口中定义的方法

> `void add(int index, Object element):` 在列表的指定位置插入指定元素（可选操作）。  
> `boolean addAll(int index, Collection<? extends E> c) `:  将集合c 中的所有元素都插入到列表中的指定位置index处。  
> `Object get(index)`: 返回列表中指定位置的元素。  
> `int indexOf(Object o)`: 返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。  
> `int lastIndexOf(Object o)`:返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。  
> `Object remove(int index)`:  移除列表中指定位置的元素。  
> `Object set(int index, Object element)`:用指定元素替换列表中指定位置的元素。  
> `List subList(int fromIndex, int toIndex)`: 返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的所有集合元素组成的子集。  
> `Object[] toArray()`: 返回按适当顺序包含列表中的所有元素的数组（从第一个元素到最后一个元素）。

除此之外，Java 8还为List接口添加了如下两个默认方法。

> `void replaceAll(UnaryOperator operator)`:根据operator指定的计算规则重新设置List集合的所有元素。  
> `void sort(Comparator c)`:根据Comparator参数对List集合的元素排序。

## 7 Queue集合

### 7.1 简介

Queue用户模拟队列这种数据结构，队列通常是指“先进先出FIFO”的容器，队列的头部是在队列中存放时间最长的元素，队列的尾部是保存在队列中存放时间最短的元素。

新元素插入到队列的尾部，访问元素操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。

### 7.2 接口中定义的方法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53d7c6acd3d736b3646fbaebb1032dfa.png)

## 8 Map集合

### 8.1 简介

Map用户保存具有映射关系的数据，因此Map集合里保存着两组数，一组值用户保存Map中的Key，另一组值用户保存Map里的Value，key和value都是可以任何引用类型的数据。Map的key不允许重复，即同一个Map对象的任何两个key通过equals方法比较总是返回false。

如下图所描述，key和value之间存在单向一对一关系，即通过指定的key，总能找到唯一的、确定的value。从Map中取出数据时，只要给出指定的key，就可以取出对应的value。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1321ecd4d7db854877ee094deab14830.png)


### 8.2 与Set集合、List集合的关系

1. 与Set集合的关系
	如果 把Map集合里所有的key 放在一起看，它们就组成了一个Set集合（所有的key没有顺序，key之间不能重复），实际上Map确定包含了一个keySet()方法,用户返回Map里所有key组成的Set集合

2. 与List集合的关系
	如果把Map里所有value放在一起看，它们有非常类似一个list：元素之间可以重复，每个元素可以根据索引来查找，只是Map中索引不再使用整数值，而是以另外一个对象作为索引

### 8.3 接口中定义的方法

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0829c8a212b4aeaa4bf28747ceb75860.png)

Map中还包括了一个内部类Entry，该类封装了一个key-value对。Entrt包含如下三个方法：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b7b0105ecf180eb7602ff38fdc32c21.png)

Map集合最经典的用法就是成对地添加、删除键值对，然后就是判断该Map中是否包含指定的key，和指定的value，也可以通过Map提供的ketSet()方法获取所有key组成集合，然后使用foreach循环来遍历Map的所有key，根据key即可遍历所有的value。

下面代码示范Map的一些基本功能：
```java
public class MapTest {  
    public static void main(String[] args) {  
        Integer i1 = 1;  
        Integer i2 = 2;  
        Map<String,Integer> map = new HashMap<>();  
        //成对放入key-value对  
        map.put("第一个",i1);  
        map.put("第二个",i2);  
        //判断是否包含指定的key  
        System.out.println(map.containsKey("第一个"));  
        //判断是否包含指定的value  
        System.out.println(map.containsValue(1));  
  
        //循环遍历  
        //1.获得Map中所有key组成的set集合  
        Set<String> strings = map.keySet();  
        //2.使用foreach进行遍历  
        for (String s : strings) {  
            //根据key获得指定的value  
            System.out.println(map.get(s));  
        }  
        //根据key来移除key-value对  
        map.remove("第一个");  
  
        System.out.println(map);  
    }  
}
```

输出结果：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/74383e6879e910e48896db5959ab45d7.png)



