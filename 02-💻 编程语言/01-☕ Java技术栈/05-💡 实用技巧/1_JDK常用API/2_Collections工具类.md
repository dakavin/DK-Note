---
文章标题: "[[2_Collections工具类]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了Java `Collections` 工具类，详述其在集合操作中的常用方法，包括排序、查找、复制、添加及线程同步。此外，它明确区分了 `Collection` 接口与 `Collections` 工具类，是理解和应用Java集合框架的基础。
tags:
- "Java"
- "Collections"
- "集合工具类"
- "排序"
- "查找"
- "线程同步"
- "Collection接口"
相关文章:
- "[[6_Collections工具类（不重要）]]"
- "[[3_977.有序数组的平方]]"
- "[[3_Set]]"
- "[[7_Object]]"
- "[[1_Arrays工具类]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/1_JDK常用API/2_Collections工具类.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:53:49
---

## 1 Collections概述  
- Collections是一个操作Set、List、Map等集合的工具类  
  
## 2 常用方法  
### 2.1 排序操作：  
- `reverse(List)`：反转 List 中元素的顺序  
- `shuffle(List)`：对 List 集合元素进行随机排序  
- `sort(List)`：根据元素的自然顺序对指定 List 集合元素按升序排序  
- `sort(List，Comparator)`：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序  
- `swap(List，int， int)`：将指定 list 集合中的 i 处元素和 j 处元素进行交换  
  
### 2.2 查找  
- `Object max(Collection)`：根据元素的自然顺序，返回给定集合中的最大元素  
- `Object max(Collection，Comparator)`：根据 Comparator 指定的顺序，返回给定集合中的最大元素  
- `Object min(Collection)`：根据元素的自然顺序，返回给定集合中的最小元素  
- `Object min(Collection，Comparator)`：根据 Comparator 指定的顺序，返回给定集合中的最小元素  
- `int binarySearch(List list,T key)`: 在List集合中查找某个元素的下标，但是List的元素必须是T或T的子类对象，而且必须是可比较大小的，即支持自然排序的。而且集合也事先必须是有序的，否则结果不确定。  
- `int binarySearch(List list,T key,Comparator c)`: 在List集合中查找某个元素的下标，但是List的元素必须是T或T的子类对象，而且集合也事先必须是按照c比较器规则进行排序过的，否则结果不确定。  
- `int frequency(Collection c，Object o)`：返回指定集合中指定元素的出现次数  
  
### 2.3 复制、替换  
- `void copy(List dest,List src)`：将src中的内容复制到dest中  
- `boolean replaceAll(List list，Object oldVal，Object newVal)`：使用新值替换 List 对象的所有旧值  
- 提供了多个unmodifiableXxx()方法，该方法返回指定 Xxx的不可修改的视图。  
  
### 2.4 添加  
- `boolean addAll(Collection  c,T... elements)`将所有指定元素添加到指定 collection 中。  
  
### 2.5 同步  
- Collections 类中提供了多个 `synchronizedXxx()` 方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题  
  
  
  
## 3 面试题：

- 区分Collection 和 Collections（需要知道，其他了解）

- Collection：集合框架中的用于存储一个一个元素的接口，又分为List和Set等子接口。  

- Collections：用于操作集合框架的一个工具类。此时的集合框架包括：Set、List、Map