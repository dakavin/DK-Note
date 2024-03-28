
## 1、Comparable 和 Comparator 的区别？

`Comparable` 接口和 `Comparator` 接口都是 Java 中用于排序的接口，它们在实现类对象之间比较大小、排序等方面发挥了重要作用：

- `Comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
- `Comparator`接口实际上是出自 `java.util` 包它有一个`compare(Object obj1, Object obj2)`方法用来排序

一般我们需要对一个集合使用自定义排序时，我们就要重写`compareTo()`方法或`compare()`方法，当我们需要对某一个集合实现两种排序方式，比如一个 `song` 对象中的歌名和歌手名分别采用一种排序方法的话，我们可以重写`compareTo()`方法和使用自制的`Comparator`方法或者以两个 `Comparator` 来实现歌名排序和歌星名排序，第二种代表我们只能使用两个参数版的 `Collections.sort()`.

#### Comparator 定制排序
```java
ArrayList<Integer> arrayList = new ArrayList<Integer>();
arrayList.add(-1);
arrayList.add(3);
arrayList.add(3);
arrayList.add(-5);
arrayList.add(7);
arrayList.add(4);
arrayList.add(-9);
arrayList.add(-7);
System.out.println("原始数组:");
System.out.println(arrayList);
// void reverse(List list)：反转
Collections.reverse(arrayList);
System.out.println("Collections.reverse(arrayList):");
System.out.println(arrayList);

// void sort(List list),按自然排序的升序排序
Collections.sort(arrayList);
System.out.println("Collections.sort(arrayList):");
System.out.println(arrayList);
// 定制排序的用法
Collections.sort(arrayList, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
System.out.println("定制排序后：");
System.out.println(arrayList);
```

Output:

```
原始数组:
[-1, 3, 3, -5, 7, 4, -9, -7]
Collections.reverse(arrayList):
[-7, -9, 4, 7, -5, 3, 3, -1]
Collections.sort(arrayList):
[-9, -7, -5, -1, 3, 3, 4, 7]
定制排序后：
[7, 4, 3, 3, -1, -5, -7, -9]
```

#### 重写 compareTo 方法实现按年龄来排序

```java
// person对象没有实现Comparable接口，所以必须实现，这样才不会出错，才可以使treemap中的数据按顺序排列
// 前面一个例子的String类已经默认实现了Comparable接口，详细可以查看String类的API文档，另外其他
// 像Integer类等都已经实现了Comparable接口，所以不需要另外实现了
public  class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * T重写compareTo方法实现按年龄来排序
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.getAge()) {
            return 1;
        }
        if (this.age < o.getAge()) {
            return -1;
        }
        return 0;
    }
}
```

```java
public static void main(String[] args) {
	TreeMap<Person, String> pdata = new TreeMap<Person, String>();
	pdata.put(new Person("张三", 30), "zhangsan");
	pdata.put(new Person("李四", 20), "lisi");
	pdata.put(new Person("王五", 10), "wangwu");
	pdata.put(new Person("小红", 5), "xiaohong");
	// 得到key的值的同时得到key所对应的值
	Set<Person> keys = pdata.keySet();
	for (Person key : keys) {
		System.out.println(key.getAge() + "-" + key.getName());

	}
}
```

Output：

```
5-小红
10-王五
20-李四
30-张三
```

`回答思路：`
	1. 两个都是用于比较对象的接口
	2. `Comparable`来源于`java.lang`包，有一个 `compareTo(Object obj)`方法用来排序
	3. `Comparator`来源于`java.util` 包，有一个`compare(Object obj1, Object obj2)`方法用来排序
	4. 一般TreeSet、TreeMap集合中的数据，需要实现上述的接口
	5. 在Collections工具类中，sort()方法也需要一个Comparator的实现类
## 2、无序性和不可重复性的含义是什么？

`回答思路：`
	1. 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是`根据数据的哈希值决定`的。
	2. 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法
## 3、比较HashSet、LinkedHashSet和TreeSet

`回答思路：`
	1. 都是Set接口的实现类，元素唯一，线程都不安全
	2. `底层数据结构`：
		- HashSet是HashMap（数组+单向链表+红黑树）
		- LinkedHashSet是HashSet的子类（数组+`双向`链表+红黑树）
		- TreeSet是红黑树（元素有序，按照自然排序和定制排序）
	3. `引用场景`：
		- HashSet用于不需要保证元素的插入和取出顺序
		- LinkedHashSet用于需要保证元素的插入和取出 FIFO
		- TreeSet用于支持对元素排序


## 4、HashSet的一个代码题

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240317160039.png)


1. 第一个输出：正常输出set中的元素
2. 第二个输出：修改了u2的name，导致set在删除的时候，计算的hash与之前的不一样，无法找到u2，所以没有删除元素
3. 第三个输出：修改的u2的索引还在之前的索引，所以添加1001-CC的时候可以正常添加进去
4. 第三个输出：添加1001-AA的时候，索引就在u2所在的地方，但是调用u2已经发生改变了，与1001-AA不equals，所以可以正常添加进去


