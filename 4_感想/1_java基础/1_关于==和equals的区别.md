## 1. 了解==
 == 可以比较任意的数据类型，基本数据类型就比较数据值，引用数据类型就比较地址值；
## 2. 了解equals（）
属于Object中的一个方法

```java
public boolean equals(Object obj) {  
    return (this == obj);  
}
```

所以比较的是两个具体的对象是否指向同一个地址值

## 3.  特殊的
1. String类
	- 与指定的StringBuffer进行比较，当且仅当此String表示与指定的StringBuffer相同的字符序列时，结果为真。
	- 即比较的是每一个字符
	- 所以String是否相等建议使用它重写后的equals()方法

源码：
```java
public boolean equals(Object anObject) {  
    if (this == anObject) {  
        return true;  
    }  
    return (anObject instanceof String aString)  
            && (!COMPACT_STRINGS || this.coder == aString.coder)  
            && StringLatin1.equals(value, aString.value);  
}
```

为什么不用 == 呢，因为比较的是地址值
```java
String str1 = "hello";
String str2 = "hello";

System.out.println(str1 == Str2); //true

String str3 = new String("hello");
String str4 = new String("hello");

System.out.println(str3 == str4); //false
System.out.println(Str3 == str1 || str3 == str2);
//false
```

解析：
	1. str1 和 str2 都指向字符常量池中同一个value的地址值
	2. str3 和 str4 指向堆空间中，不同地址值的String对象