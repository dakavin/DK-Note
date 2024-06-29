
## 1 String、StringBuffer、StringBuilder的区别？

**可变性**

`String` 是不可变的（后面会详细分析原因）。

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final` 和 `private` 关键字修饰，最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。

```
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    //...
}
```

**线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，线程安全。`AbstractStringBuilder` 是 `StringBuilder` 与 `StringBuffer` 的公共父类，定义了一些字符串的基本操作，如 `expandCapacity`、`append`、`insert`、`indexOf` 等公共方法。`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

**性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

`回答思路：`
	可安性
	1. 可变性：String不可变；Buffer和Builder可变，它两都是继承AbstractStringBuilder，有很多修改字符串的方法；
	2. 线程安全：String(常量)和Buffer(同步锁)线程安全，Builder线程不安全；
	3. 性能：String每次都会生成新对象；Builder和Buffer都是对象本身修改，Builder性能较高

## 2 String为什么是不可变的？

`String` 类中使用 `final` 关键字和`private`关键字修饰字符数组来保存字符串的

> 我们直到final修饰的类不能被继承，修饰的方法不能被重写，修饰的变量是基本数据类型就不能改变，修饰的变量是引用类型就不能指向其他对象；
> final修饰的字符数组，只是表示指向不可变，但是该字符数组还是可以变化的

`String` 真正不可变有下面几点原因：

1. 保存字符串的数组被 `final` 修饰`且为私有的`，并且`String` 类没有提供/暴露修改这个字符串的方法。
2. `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变。

**Java 9 为何要将 `String` 的底层实现由 `char[]` 改成了 `byte[]` ?**

新版的 String 其实支持两个编码方案：Latin-1 和 UTF-16。如果字符串中包含的汉字没有超过 Latin-1 可表示范围内的字符，那就会使用 Latin-1 作为编码方案。Latin-1 编码方案下，`byte` 占一个字节(8 位)，`char` 占用 2 个字节（16），`byte` 相较 `char` 节省一半的内存空间。

JDK 官方就说了绝大部分字符串对象只包含 Latin-1 可表示的字符。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2ae49b46e3e50c5214963a3377127dcf.png)

如果字符串中包含的汉字超过 Latin-1 可表示范围内的字符，`byte` 和 `char` 所占用的空间是一样的。

`回答思路：`
	1. 内部字符数组是final修饰（引用指向不变），并且是private修饰的（子类无法获取）；
	2. String类是final修饰的（不可被继承，加了一层保险）
	3. String在java9后改为byte数组，可以节省空间

## 3 字符串拼接用 + 还是 StringBuilder？

Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

```java
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8b5c0e0786474e680f6d9e0170f52c5d.png)

可以看出，字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。

```java
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);
```

`StringBuilder` 对象是在循环内部被创建的，这意味着每循环一次就会创建一个 `StringBuilder` 对象。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35106e09bbcd98058732cf698afd0b0e.png)

如果直接使用 `StringBuilder` 对象进行字符串拼接的话，就不会存在这个问题了。

```java
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
System.out.println(s);
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eaf4464cffc6a3d8e996527663e285fe.png)

如果你使用 IDEA 的话，IDEA 自带的代码检查机制也会提示你修改代码。

不过，使用 “+” 进行字符串拼接会产生大量的临时对象的问题在 JDK9 中得到了解决。在 JDK9 当中，字符串相加 “+” 改为了用动态方法 `makeConcatWithConstants()` 来实现，而不是大量的 `StringBuilder` 了。这个改进是 JDK9 的 [JEP 280open in new window](https://openjdk.org/jeps/280) 提出的，这也意味着 JDK 9 之后，你可以放心使用“+” 进行字符串拼接了。关于这部分改进的详细介绍，推荐阅读这篇文章：还在无脑用 [StringBuilder？来重温一下字符串拼接吧open in new window](https://juejin.cn/post/7182872058743750715) 。

`回答思路：`
	1. 字符串使用+拼接，底层使用的还是StringBuilder的append方法
	2. 不过使用+拼接，多导致创建多个StringBuilder对象
	3. 这个问题在JDK9之后优化了，采用动态方法
## 4 String#equals() 和 Object#equals() 有何区别？

`回答思路：`
	1. String的equals()方法是重写过的，比较的是String字符串值是否相等；
	2. Object的equals()方法是比较对象的内存地址值；

## 5 字符串常量池了解吗？

`回答思路：`
	1. **字符串常量池** 是 JVM 为了`提升性能和减少内存消耗`针对字符串（String 类）专门开辟的一块区域，`主要目的是为了避免字符串的重复创建`。
	2. JDK6中，字符串常量池在方法区。JDK7开始，就移到堆空间
	3. new出来的对象，会在堆创建一个字符数组，字符数组再指向字符串常量池的数据（第6题答案）

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String s1 = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String s2 = "ab";
// new出来的对象，会在堆创建一个字符数组，字符数组再指向字符串常量池的数据
String s3 = new String("ab");

String s4 =  new String("ab");
System.out.println(s1==s2);// true
System.out.println(s2==s3);// false
System.out.println(s3==s4);// false
```

更多关于字符串常量池的介绍可以看一下 [Java 内存区域详解open in new window](https://javaguide.cn/java/jvm/memory-area.html) 这篇文章。

## 6 String s1 = new String("abc")；这句话创建了几个字符串对象？

会创建 1 或 2 个字符串对象。

1、如果字符串常量池中不存在字符串对象“abc”的引用，那么它会在堆上创建两个字符串对象，其中一个字符串对象的引用会被保存在字符串常量池中。

示例代码（JDK 1.8）：

```
String s1 = new String("abc");
```

对应的字节码：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f62171abb89d60d7b50ca802b499289.png)

`ldc` 命令用于判断字符串常量池中是否保存了对应的字符串对象的引用，如果保存了的话直接返回，如果没有保存的话，会在堆中创建对应的字符串对象并将该字符串对象的引用保存到字符串常量池中。

2、如果字符串常量池中已存在字符串对象“abc”的引用，则只会在堆中创建 1 个字符串对象“abc”。

示例代码（JDK 1.8）：

```
// 字符串常量池中已存在字符串对象“abc”的引用
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s2 = new String("abc");
```

对应的字节码：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28e929bf63faf406424288ddb28f3f3b.png)

这里就不对上面的字节码进行详细注释了，7 这个位置的 `ldc` 命令不会在堆中创建新的字符串对象“abc”，这是因为 0 这个位置已经执行了一次 `ldc` 命令，已经在堆中创建过一次字符串对象“abc”了。7 这个位置执行 `ldc` 命令会直接返回字符串常量池中字符串对象“abc”对应的引用。

`回答思路：`
	1. 会创建1个或2个字符串对象
	2. `字符串常量池没有该字符对象`，则先在堆中创建一个String的引用对象，然后再在字符串常量池中创建该字符对象；
	3. `字符串常量池有该字符对象`，直接在堆中创建一个String的引用对象

## 7 String#intern 方法有什么作用？

`String.intern()` 是一个 native（本地）方法，其作用是将指定的字符串对象的引用保存在字符串常量池中，可以简单分为两种情况：

- 如果字符串常量池中保存了对应的字符串对象的引用，就直接返回该引用。
- 如果字符串常量池中没有保存了对应的字符串对象的引用，那就在常量池中创建一个指向该字符串对象的引用并返回。
- `即将字符串引用指向字符串常量池中的数据`

示例代码（JDK 1.8） :

```java
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();
// 会在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();
// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true
// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false
// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true
```

`回答思路：`
	本质就是将字符串直接在字符串常量池中，创建一个字符对象（有就返回该引用，没有就直接创建并返回该引用）
	
## 8 String类型的变量和常量做 + 运算时发生了什么？

`回答思路：`
	常量  + 常量 ：结果在常量池
	有变量参与：结果在堆
	intern后：结果在常量池
	concat后：结果在堆
	final之后的String认为是常量
	字符串+的本质，就是StringBuilder调用append方法后返回一个新的String对象（第三个问题）

先来看字符串不加 `final` 关键字拼接的情况（JDK1.8）：

```java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

> **注意**：比较 String 字符串的值是否相等，可以使用 `equals()` 方法。 `String` 中的 `equals` 方法是被重写过的。 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是字符串的值是否相等。如果你使用 `==` 比较两个字符串是否相等的话，IDEA 还是提示你使用 `equals()` 方法替换。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9606aa3b92003740376d02af0bc44aad.png)

**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。**

在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠(Constant Folding)** 的代码优化。《深入理解 Java 虚拟机》中是也有介绍到：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/31d674197601a831455ad6c509196e5a.png)

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

```
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

示例代码：

```java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

被 `final` 关键字修饰之后的 `String` 会被编译器当做常量来处理，编译器在程序编译期就可以确定它的值，其效果就相当于访问常量。

如果 ，编译器在运行时才能知道其确切值的话，就无法对其优化。

示例代码（`str2` 在运行时才能确定其值）：

```java
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```
