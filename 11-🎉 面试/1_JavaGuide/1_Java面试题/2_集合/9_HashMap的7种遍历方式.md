---
文章标题: "[[9_HashMap的7种遍历方式]]" 
文章作者: Dakkk
文章概要: |
  本文详细介绍了Java HashMap的7种遍历方式，包括迭代器、ForEach、Lambda和Streams API。文章强调`entrySet`方法因单次遍历而比`keySet`方法性能更优，后者涉及二次查找，因此建议优先使用`entrySet`进行HashMap遍历。
文章标签:
- "HashMap"
- "Java"
- "遍历"
- "性能优化"
- "Streams API"
- "Lambda"
- "EntrySet"
- "KeySet"
相关文章:
- "[[5_Map]]"
- "[[2_📕📕前台首页：文章分页接口开发]]"
- "[[5_集合List]]"
- "[[5_批量插入]]"
- "[[8_数据库连接池]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/1_JavaGuide/1_Java面试题/2_集合/9_HashMap的7种遍历方式.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 01:07:00
---

随着 JDK 1.8 Streams API 的发布，使得 HashMap 拥有了更多的遍历的方式，但应该选择那种遍历方式？反而成了一个问题。

本文**先从 HashMap 的遍历方法讲起，然后再从性能、原理以及安全性等方面，来分析 HashMap 各种遍历方式的优势与不足**，本文主要内容如下图所示：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/385995b3a99713e50247bccbe36f50e3.png)

HashMap **遍历从大的方向来说，可分为以下 4 类**：

1. 迭代器（Iterator）方式遍历；
    
2. For Each 方式遍历；
    
3. Lambda 表达式遍历（JDK 1.8+）;
    
4. Streams API 遍历（JDK 1.8+）。
    

但每种类型下又有不同的实现方式，因此具体的遍历方式又可以分为以下 7 种：

1. 使用迭代器（Iterator）EntrySet 的方式进行遍历；
    
2. 使用迭代器（Iterator）KeySet 的方式进行遍历；
    
3. 使用 For Each EntrySet 的方式进行遍历；
    
4. 使用 For Each KeySet 的方式进行遍历；
    
5. 使用 Lambda 表达式的方式进行遍历；
    
6. 使用 Streams API 单线程的方式进行遍历；
    
7. 使用 Streams API 多线程的方式进行遍历。
    

接下来我们来看每种遍历方式的具体实现代码。

## 1 迭代器 EntrySet

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();  
        while (iterator.hasNext()) {  
            Map.Entry<Integer, String> entry = iterator.next();  
            System.out.println(entry.getKey());  
            System.out.println(entry.getValue());  
        }  
    }  
}
```

## 2 迭代器 KeySet

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        Iterator<Integer> iterator = map.keySet().iterator();  
        while (iterator.hasNext()) {  
            Integer key = iterator.next();  
            System.out.println(key);  
            System.out.println(map.get(key));  
        }  
    }  
}
```

## 3 ForEach EntrySet

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        for (Map.Entry<Integer, String> entry : map.entrySet()) {  
            System.out.println(entry.getKey());  
            System.out.println(entry.getValue());  
        }  
    }  
}
```

## 4 ForEach KeySet

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        for (Integer key : map.keySet()) {  
            System.out.println(key);  
            System.out.println(map.get(key));  
        }  
    }  
}
```

## 5 Lambda

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        map.forEach((key, value) -> {  
            System.out.println(key);  
            System.out.println(value);  
        });  
    }  
}
```

## 6 Streams API 单线程

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        map.entrySet().stream().forEach((entry) -> {  
            System.out.println(entry.getKey());  
            System.out.println(entry.getValue());  
        });  
    }  
}
```
## 7 Streams API 多线程

```java
public class HashMapTest {  
    public static void main(String[] args) {  
        // 创建并赋值 HashMap  
        Map<Integer, String> map = new HashMap();  
        map.put(1, "Java");  
        map.put(2, "JDK");  
        map.put(3, "Spring Framework");  
        map.put(4, "MyBatis framework");  
        map.put(5, "Java中文社群");  
        // 遍历  
        map.entrySet().parallelStream().forEach((entry) -> {  
            System.out.println(entry.getKey());  
            System.out.println(entry.getValue());  
        });  
    }  
}
```

## 8 总结

 `entrySet` 的性能比 `keySet` 的性能高出了一倍之多，因此我们应该尽量使用 `entrySet`  来实现 Map 集合的遍历。

`原因：`

`EntrySet` 之所以比 `KeySet` 的性能高是因为，`KeySet` 在循环时使用了 `map.get(key)`，而 `map.get(key)` 相当于又遍历了一遍 Map 集合去查询 `key` 所对应的值。为什么要用“又”这个词？那是因为**在使用迭代器或者 for 循环时，其实已经遍历了一遍 Map 集合了，因此再使用 `map.get(key)` 查询时，相当于遍历了两遍**。

而 `EntrySet` 只遍历了一遍 Map 集合，之后通过代码“Entry<Integer, String> entry = iterator.next()”把对象的 `key` 和 `value` 值都放入到了 `Entry` 对象中，因此再获取 `key` 和 `value` 值时就无需再遍历 Map 集合，只需要从 `Entry` 对象中取值就可以了。

所以，**`EntrySet` 的性能比 `KeySet` 的性能高出了一倍，因为 `KeySet` 相当于循环了两遍 Map 集合，而 `EntrySet` 只循环了一遍**。