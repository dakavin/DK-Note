---
文章标题: "[[1_Java值传递详解]]" 
文章作者: Dakkk
文章概要: |
  文章深入解析Java的参数传递机制，强调其只支持值传递。通过基本类型和引用类型的多个案例，清晰证明无论是基本类型还是引用类型，传递的都是实参值或其地址的副本。同时对比C++的引用传递，并探讨Java不引入引用传递的设计考量。
文章标签:
- "Java"
- "值传递"
- "引用传递"
- "参数传递"
- "Java基础"
- "内存机制"
- "对象引用"
相关文章:
- "[[2_包装类型]]"
- "[[3_关键字]]"
- "[[5_面向对象编程（基础）]]"
- "[[6_函数]]"
- "[[0_基础加强]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/1_JavaGuide/1_Java面试题/1_Java基础重要知识点‼️/1_Java值传递详解.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 形参&实参

方法的定义可能会用到 **参数**（有参的方法），参数在程序语言中分为：

- **实参（实际参数，Arguments）**：用于传递给函数/方法的参数，必须有确定的值。
- **形参（形式参数，Parameters）**：用于定义函数/方法，接收实参，不需要有确定的值。

```java
String hello = "Hello!";
// hello 为实参
sayHello(hello);
// str 为形参
void sayHello(String str) {
    System.out.println(str);
}
```

## 2 值传递&引用传递

程序设计语言将实参传递给方法（或函数）的方式分为两种：

- **值传递**：方法接收的是实参值的拷贝，会创建副本。
- **引用传递**：方法接收的直接是`实参所引用的对象在堆中的地址`，不会创建副本，对形参的修改将影响到实参。

很多程序设计语言（比如 C++、 Pascal )提供了两种参数传递的方式，不过，在 Java 中只有值传递。

## 3 为什么Java只有值传递

**为什么说 Java 只有值传递呢？** 不需要太多废话，我通过 3 个例子来给大家证明。

### 3.1 案例1：传递基本类型参数

代码：

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;
    swap(num1, num2);
    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

输出：

```java
a = 20
b = 10
num1 = 10
num2 = 20
```

解析：

在 `swap()` 方法中，`a`、`b` 的值进行交换，并不会影响到 `num1`、`num2`。因为，`a`、`b` 的值，只是从 `num1`、`num2` 的复制过来的。也就是说，a、b 相当于 `num1`、`num2` 的副本，副本的内容无论怎么修改，都不会影响到原件本身。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e90a0630245fb1c019725b306fe1240a.png)

通过上面例子，我们已经知道了`一个方法不能修改一个基本数据类型的参数`，而对象引用作为参数就不一样，请看案例 2。
### 3.2 案例2：传递引用类型参数1

代码：

```java
  public static void main(String[] args) {
      int[] arr = { 1, 2, 3, 4, 5 };
      System.out.println(arr[0]);
      change(arr);
      System.out.println(arr[0]);
  }

  public static void change(int[] array) {
      // 将数组的第一个元素变为0
      array[0] = 0;
  }
```

输出：

```
1
0
```

解析：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7425b530d367db9abe2f0090bbbf5368.png)

看了这个案例很多人肯定觉得 Java 对引用类型的参数采用的是引用传递。

`实际上，并不是的，这里传递的还是值，不过，这个值是实参的地址罢了！`

也就是说 `change` 方法的参数拷贝的是 `arr` （实参）的地址，因此，它和 `arr` 指向的是同一个数组对象。这也就说明了为什么方法内部对形参的修改会影响到实参。

为了更强有力地反驳 Java 对引用类型的参数采用的不是引用传递，我们再来看下面这个案例！

### 3.3 案例3：传递引用类型参数2

```java
public class Person {
    private String name;
   // 省略构造函数、Getter&Setter方法
}

public static void main(String[] args) {
    Person xiaoZhang = new Person("小张");
    Person xiaoLi = new Person("小李");
    swap(xiaoZhang, xiaoLi);
    System.out.println("xiaoZhang:" + xiaoZhang.getName());
    System.out.println("xiaoLi:" + xiaoLi.getName());
}

public static void swap(Person person1, Person person2) {
    Person temp = person1;
    person1 = person2;
    person2 = temp;
    System.out.println("person1:" + person1.getName());
    System.out.println("person2:" + person2.getName());
}
```

输出:

```
person1:小李
person2:小张
xiaoZhang:小张
xiaoLi:小李
```

解析：

怎么回事？？？两个引用类型的形参互换并没有影响实参啊！

`swap` 方法的参数 `person1` 和 `person2` 只是拷贝的实参 `xiaoZhang` 和 `xiaoLi` 的地址。因此， `person1` 和 `person2` 的互换只是拷贝的两个地址的互换罢了，并不会影响到实参 `xiaoZhang` 和 `xiaoLi`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7291e0bbf9bde8586d647bb5e59bf90e.png)
## 4 引用传递是怎么样的？

看到这里，相信你已经知道了 Java 中只有值传递，是没有引用传递的。  
但是，引用传递到底长什么样呢？下面以 `C++` 的代码为例，让你看一下引用传递的庐山真面目。

```C++
#include <iostream>

void incr(int& num)
{
    std::cout << "incr before: " << num << "\n";
    num++;
    std::cout << "incr after: " << num << "\n";
}

int main()
{
    int age = 10;
    std::cout << "invoke before: " << age << "\n";
    incr(age);
    std::cout << "invoke after: " << age << "\n";
}
```

输出结果：

```
invoke before: 10
incr before: 10
incr after: 11
invoke after: 11
```

分析：可以看到，在 `incr` 函数中对形参的修改，可以影响到实参的值。要注意：这里的 `incr` 形参的数据类型用的是 `int&` 才为引用传递，如果是用 `int` 的话还是值传递哦！
## 5 为什么Java不引用引用传递呢？

引用传递看似很好，能在方法内就直接把实参的值修改了，但是，为什么 Java 不引入引用传递呢？

**注意：以下为个人观点看法，并非来自于 Java 官方：**

1. 出于安全考虑，方法内部对值进行的操作，对于调用者都是未知的（把方法定义为接口，调用方不关心具体实现）。你也想象一下，如果拿着银行卡去取钱，取的是 100，扣的是 200，是不是很可怕。
2. Java 之父 James Gosling 在设计之初就看到了 C、C++ 的许多弊端，所以才想着去设计一门新的语言 Java。在他设计 Java 的时候就遵循了简单易用的原则，摒弃了许多开发者一不留意就会造成问题的“特性”，语言本身的东西少了，开发者要学习的东西也少了。

## 6 总结

Java 中将实参传递给方法（或函数）的方式是 **值传递**：

- 如果`参数是基本类型`的话，很简单，`传递的就是基本类型的字面量值的拷贝`，会创建副本。
- 如果`参数是引用类型`，`传递的就是实参所引用的对象在堆中地址值的拷贝`，同样也会创建副本。
