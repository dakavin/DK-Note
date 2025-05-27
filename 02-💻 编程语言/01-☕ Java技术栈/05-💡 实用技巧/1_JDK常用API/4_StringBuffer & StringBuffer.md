---
文章标题: "[[4_StringBuffer & StringBuffer]]" 
文章作者: Dakkk
文章概要: |
  文章详细列举并示例了Java中`StringBuilder`和`StringBuffer`类常用的字符串操作API，包括追加、删除、替换、插入、查找、截取、反转等，并指出它们API一致且支持方法链操作。
tags:
- "Java"
- "StringBuilder"
- "StringBuffer"
- "字符串操作"
- "API"
- "方法链"
- "字符串拼接"
- "容器类"
相关文章:
- "[[3_字符串替换]]"
- "[[1_JDBC概述]]"
- "[[4📕_151.翻转字符串里的单词]]"
- "[[5_左或右旋转字符串]]"
- "[[6_383.赎金信]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/1_JDK常用API/4_StringBuffer & StringBuffer.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:53:49
---

StringBuilder、StringBuffer的API是完全一致的，并且很多方法与String相同。

## 1 常用API

（1）StringBuffer append(xx)：提供了很多的append()方法，用于进行字符串追加的方式拼接
（2）StringBuffer delete(int start, int end)：删除`[start,end)`之间字符
（3）StringBuffer deleteCharAt(int index)：删除`[index]`位置字符
（4）StringBuffer replace(int start, int end, String str)：替换`[start,end)`范围的字符序列为str
（5）void setCharAt(int index, char c)：替换[index]位置字符
（6）char charAt(int index)：查找指定index位置上的字符
（7）StringBuffer insert(int index, xx)：在[index]位置插入xx
（8）int length()：返回存储的字符数据的长度
（9）StringBuffer reverse()：反转

> - 当append和insert时，如果原来value数组长度不够，可扩容。
>
> - 如上(1)(2)(3)(4)(9)这些方法支持`方法链操作`。原理：
>	![image-20230922020103193.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7049ba6e27ba5b8a5fea1b300cd38e25.png)


## 2 其它API

（1）int indexOf(String str)：在当前字符序列中查询str的第一次出现下标
（2）int indexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str的第一次出现下标
（3）int lastIndexOf(String str)：在当前字符序列中查询str的最后一次出现下标
（4）int lastIndexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str的最后一次出现下标
（5）String substring(int start)：截取当前字符序列[start,最后]
（6）String substring(int start, int end)：截取当前字符序列`[start,end)`
（7）String toString()：返回此序列中数据的字符串表示形式
（8）void setLength(int newLength) ：设置当前字符序列长度为newLength

```java
@Test
public void test1(){
    StringBuilder s = new StringBuilder();
    s.append("hello").append(true).append('a').append(12).append("atguigu");
    System.out.println(s);
    System.out.println(s.length());
}

@Test
public void test2(){
    StringBuilder s = new StringBuilder("helloworld");
    s.insert(5, "java");
    s.insert(5, "chailinyan");
    System.out.println(s);
}

@Test
public void test3(){
    StringBuilder s = new StringBuilder("helloworld");
    s.delete(1, 3);
    s.deleteCharAt(4);
    System.out.println(s);
}
@Test
public void test4(){
    StringBuilder s = new StringBuilder("helloworld");
    s.reverse();
    System.out.println(s);
}

@Test
public void test5(){
    StringBuilder s = new StringBuilder("helloworld");
    s.setCharAt(2, 'a');
    System.out.println(s);
}

@Test
public void test6(){
    StringBuilder s = new StringBuilder("helloworld");
    s.setLength(30);
    System.out.println(s);
}
