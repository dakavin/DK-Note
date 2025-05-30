---
文章标题: "[[1_Arrays工具类]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了Java的`java.util.Arrays`工具类，详细阐述了其提供的核心数组操作方法，包括字符串表示、排序、二分查找、复制、比较和填充。它是Java数组编程的实用指南。
tags:
- "Java"
- "Arrays"
- "数组操作"
- "Arrays工具类"
- "排序"
- "二分查找"
- "数组复制"
相关文章:
- "[[4_数组]]"
- "[[1_ArrayList源码分析]]"
- "[[2_541.反转字符串II]]"
- "[[3_977.有序数组的平方]]"
- "[[3_Set]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/1_JDK常用API/1_Arrays工具类.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:53:49
---

java.util.Arrays类即为操作数组的工具类，包含了用来操作数组（比如排序和搜索）的各种方法。 比如：

* `数组元素拼接`
  * static String toString(int\[] a) ：字符串表示形式由数组的元素列表组成，括在方括号（"\[]"）中。相邻元素用字符 ", "（逗号加空格）分隔。形式为：\[元素1，元素2，元素3。。。]

  * static String toString(Object\[] a) ：字符串表示形式由数组的元素列表组成，括在方括号（"\[]"）中。相邻元素用字符 ", "（逗号加空格）分隔。元素将自动调用自己从Object继承的toString方法将对象转为字符串进行拼接，如果没有重写，则返回类型@hash值，如果重写则按重写返回的字符串进行拼接。

* `数组排序`
  * static void sort(int\[] a) ：将a数组按照从小到大进行排序

  * static void sort(int\[] a, int fromIndex, int toIndex) ：将a数组的\[fromIndex, toIndex)部分按照升序排列

  * static void sort(Object\[] a) ：根据元素的自然顺序对指定对象数组按升序进行排序。

  * static \<T> void sort(T\[] a, Comparator\<? super T> c) ：根据指定比较器产生的顺序对指定对象数组进行排序。

* `数组元素的二分查找`
  * static int binarySearch(int\[] a, int key)  、static int binarySearch(Object\[] a, Object key) ：要求数组有序，在数组中查找key是否存在，如果存在返回第一次找到的下标，不存在返回负数。

* `数组的复制`
  * static int\[] copyOf(int\[] original, int newLength)  ：根据original原数组复制一个长度为newLength的新数组，并返回新数组

  * static \<T> T\[] copyOf(T\[] original,int newLength)：根据original原数组复制一个长度为newLength的新数组，并返回新数组

  * static int\[] copyOfRange(int\[] original, int from, int to) ：复制original原数组的`[from,to)`构成新数组，并返回新数组

  * static\<T> T\[] copyOfRange(T\[] original,int from,int to)：复制original原数组的\[from,to)构成新数组，并返回新数组

* `比较两个数组是否相等`
  * static boolean equals(int\[] a, int\[] a2) ：比较两个数组的长度、元素是否完全相同

  * static boolean equals(Object\[] a,Object\[] a2)：比较两个数组的长度、元素是否完全相同

* `填充数组`
  * static void fill(int\[] a, int val) ：用val值填充整个a数组

  * static void fill(Object\[] a,Object val)：用val对象填充整个a数组

  * static void fill(int\[] a, int fromIndex, int toIndex, int val)：将a数组\[fromIndex,toIndex)部分填充为val值

  * static void fill(Object\[] a, int fromIndex, int toIndex, Object val) ：将a数组\[fromIndex,toIndex)部分填充为val对象

- `补充：`
	- 关于数组复制的底层调用：[3.3 `System.arraycopy()` 和 `Arrays.copyOf()`方法](../../../../11-🎉%20面试/1_JavaGuide/1_Java面试题/2_集合源码分析‼️‼️/1_ArrayList源码分析.md#3.3%20`System.arraycopy()`%20和%20`Arrays.copyOf()`方法)