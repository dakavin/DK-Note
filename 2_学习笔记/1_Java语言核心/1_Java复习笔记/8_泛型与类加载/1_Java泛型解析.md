
## 1 为什么要使用泛型

### 1.1 类型参数的好处

1. 类型安全：泛型的主要目标是提高Java程序的类型安全。通过指导使用泛型定义的变量的类型限制，编译器可以在一个高得多的程度上验证类型假设。没有泛型，这些假设就只存于程序员的头脑中（或如果幸运的话，还存于代码注释中）。

2. 消除强制类型转换：泛型的一个附带好处是，消除源码中的许多强制类型转换。这使得代码更加可读，并且减少了出错机会。

Java语言引入泛型的好处是安全简单。泛型的好处是在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高代码的重用率

## 2 定义简单的泛型类

泛型类的定义比较简单，如下便可以定义一个泛型类，在实例化泛型类的时候必须指定泛型的具体类型

```java
public class Pair<T>{
    private T first;
    private T second;
    
    public Pair(){
        first = null;second = null;
    }
    
    public T getFirst(){
        return first;
    }
    
    public T getSecond(){
        return second;
    }
    
    public void setFirst(T newValue){
        first = newValue;
    }
    public void setSecond(T newValue){
        second = newValue;
    }
}
```

泛型在使用中还有一些规则和限制：

- 泛型的类型参数只能是类（包括自定义类），不能是简单类型。
- 同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的
- 泛型的类型参数可以有多个

泛型类可以定义多个类型变量，例如
```java
public class Pari<T,U>{
    ...
}
```

## 3 泛型方法

Java中的泛型方法相对复杂一点，在调用的时候需要指明泛型类型

`定义泛型的语法`：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5d6ee18b5a3ec549faec825c1b1cb609.png)
`调用泛型的语法：`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/911823a421ecc31f41e529daec52fe19.png)

`定义泛型方法时，必须在返回值前面追加一个<T>`，来什么这是一个泛型方法，持有一个泛型T，然后才可以用泛型T作为方法的返回值。

`注意：`类型变量放在修饰符的后面，返回类型的前面

既然是泛型方法，就代表这我们不知道具体的类型是什么，也不知道构造方法如何，因此没有办法去new一个对象，但可以`利用变量c的netInstance方法去常见对象，也就是利用反射创建对象`。

泛型方法要求的参数是`Class<T>`类型，而Class.forName()方法的返回值也是`Class<T>`，因此可以用Class.forName()作为参数。其中，forName()方法中的参数是何种类型，返回的`Class<T>`就是何种类型。

在本例中，forName()方法中传入的是User类的完整路径，因此返回的是`Class<User>`类型的对象，因此调用泛型方法时，变量c的类型就是User，因此泛型方法中的泛型T就被指明为User，因此变量obj的类型为User。

## 4 类型变量的限定

我们都知道在方法前指定了`<T>`，那么就说这个泛型类型和类定义是的泛型类型无关，所以可以在普通类中定义泛型方法，泛型可以限定泛型类型变量必须是实现某几种接口或继承某个类，多个限定类型通过`&`分隔，如：
```java
public static <T extends Comparable> T min(T[] a)...
```

对泛型进行限制，使其只有集成或实现Comparable的类才能使用该方法

(1) `? extends X:表示类型的上界`

特点：

- 限定 ? 为 X 的子类型，但不知道是哪个子类型
- 可以安全的访问数据，访问X及其子类型

```dart
<T extends BoundingType>
```

T表示绑定类型的子类型，T和绑定类型可以是类或者接口。

一个变量或者通配符可以绑定多个限定，用“&”分开

```dart
T extends Comparable & Serializable
```

`若T的限定类型是类，则有且最多只有一个，且放于接口前面`

## 5 泛型代码和虚拟机

Java虚拟即是不存在泛型类型对象的，所有对象都属于普通类，甚至在泛型实现的早期版本中，可以将使用泛型的程序编译为在1.0虚拟机上能够运行的.class文件，这个向后兼容性后期被抛弃了，所以后来如果用Sun公司的编译器编译的泛型代码，是不能运行在Java5.0之前的虚拟机的，这样就导致了一些实际生产的问题，如一些遗留代码如何跟新的系统进行衔接，要弄明白这个问题，需要先了解一下虚拟机是怎么执行泛型代码的。

### 5.1 类型擦除

类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。编译器职位泛型类型生成一份字节码，并将其实例关联到这份字节码上。

类型擦除的关键在于从泛型类型中取出类型参数的相关信息，并且在必要的时候添加类型检查和类型转换的方法。

`虚拟机的一种机制：擦除类型参数，并将其替换成限定类型，没有限定类型用Object代替`

```java
public class Period<T extends Comparable<T> & Serializable> {  
      private T begin;  
      private T end;  
  
      public Period(T one, T two) {  
               if (one.compareTo(two) > 0) {begin = two;end = one;  
              } else {begin = one;end = two;}  
     }  
}  
```

  ```java
//擦除后
public class Period implements Serializable{  
      private Comparable begin;  
      private Comparable end;  
  
      public Period(Comparable one, Comparable two) {  
               if (one.compareTo(two) > 0) {begin = two; end = one;  
              } else {begin = one; end = two;}  
     }  
}  
```

Java泛型的处理几乎都在编译器中进行，编译器生成的字节码是不包含泛型信息的，泛型类型信息将在编译处理是被擦除，这个过程即类型擦除。通常情况下，Java是通过一下方式处理泛型：`Java编译器通过Code sharing方式为每个泛型类型常见唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类型实例映射到唯一的字节码表示是通过类型擦除(type erasue)实现的`。

>	Code sharing：对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。

`注意：`当存在情况：`class Interval<T extends Serializable & Comparable>`，原始类型用Serializable替换T，在有必要的时候向Comparable强制类型转换，为了提高效率，应该将没有方法的接口放在列表的后面。

类型擦除带来的灵异问题：

- 无法用同一泛型类型的实例区分方法签名

```csharp
    public class Erasure{  
  
            public void test(List<String> ls){  
                System.out.println("Sting");  
            }  
            public void test(List<Integer> li){  
                System.out.println("Integer");  
            }  
    }
```

- 不能同时catch同一泛型异常类的多个实例
- 泛型类的静态变量是可以共享的

```java
import java.util.*;  
  
public class StaticTest{  
    public static void main(String[] args){  
        GT<Integer> gti = new GT<Integer>();  
        gti.var=1;  
        GT<String> gts = new GT<String>();  
        gts.var=2;  
        System.out.println(gti.var);  
    }  
}  
class GT<T>{  
    public static int var=0;  
    public void nothing(T x){}  
}

//输出2
```

### 5.2 翻译泛型表达式

```java
Couple<Employee> couple = ...;  
Employee wife = couple.getWife();  
```

擦除后，getWife()返回的是Object类型，然后虚拟机会插入强制类型转换，将Object转换为Employee，所以虚拟机实际上执行了两条指令
- 调用Couple.getWife()方法
- 将Object转换成Employee类型
### 5.3 翻译泛型方法

```java
public static <T extends Comparable<T>> max(T[] arrays) {... }  
擦除后成了:  
public static Comoparable max(Comparable[] arrays) {... }  
```

```java
public class Period <T extends Comparable<T> & Serializable> {  
      private T begin;  
      private T end;  
  
      public Period(T one, T two) {  
               if (one.compareTo(two) > 0) {begin = two;end = one;  
              } else {begin = one;end = two;}  
     }  
     public void setBegin(T begin) {this. begin = begin;}  
     public void setEnd(T end) {this. end = end;}  
     public T getBegin() {return begin;}  
     public T getEnd() {return end;}  
}  
public class DateInterval extends Period<Date> {  
  
      public DateInterval(Date one, Date two) {  
               super(one, two);  
     }  
      public void setBegin(Date begin) {  
               super.setBegin(begin);  
     }  
}  
```

DateInterval类型擦除后，Period中的方法变成：
- public void setBegin(Object begin){...}

而DateInterval中的方法还是：
- public void setBegin(Date begin){...}

所以DateInterval从Period中继承了 public void setBegin(Object begin) {...}而自身又存在public void setBegin(Date begin) {...}方法，用户使用时问题发生了：
```java
Period<Date> period  = new DateInterval(...);  
period.setBegin(new Date());  
```

这里因为period引用指向了DateInterval实例，根据多态性，setBegin应该调用DateInterval对象的setBegin方法，可是这个擦除让Period中的public void setBegin(Object begin){...}被调用，导致了擦除与多台发生了冲突，怎么办呢？

虚拟机此时会在DateInterval类中生成一个桥方法（bridge method），调用过程发生了细微的变化：
```java
public void setBegin(Object begin) {  
     setBegin((Date)begin);  
 }  
```

有了这个合成的桥方法以后，code07中对setBegin的调用步骤如下：
```css
1.调用DateInterval.setBegin(Object)方法。
2.DateInterval.setBegin(Object)方法调用DateInterval.setBegin(Date)方法。
```

发现了吗，当我们在DateInterval中增加getBegin 方法之后会是什么样子呢？

是不是Period中有一个Object getBegin()的方法，而DateInterval中有一个Date getBegin()方法呢，这两个方法在Java是不能同时存在的，可是Java5以后增加了一个协变类型，使得这里是被允许的，看看DateInterval中getBegin方法就知道了：
```java
@Override  
public Date getBegin(){ return super.getBegin(); }  
```

> 这里用了@Override，说明是覆盖了父类的Object getBegin()方法，而返回值可以指定为父类中的返回值类型的子类，这就是协变类型，这是Java5以后才可以允许的，允许子类覆盖了方法后指定一个更严格的类型(子类型)。

  总结：

- 1.记住一点，虚拟机中没有泛型，只有普通的类。
- 2.所有泛型的类型参数都用它们限定的类型代替，没有限定则用Object。
- 3.为了保持类型安全性，虚拟机在有必要时插入强制类型转换。
- 4.桥方法的合成用来保持多态性。
- 5.协变类型允许子类覆盖方法后返回一个更严格的类型。

## 6 约束与局限性

### 6.1 不能使用基本类型实例化泛型

不能使用基本类型作为类型参数，因为擦除之后，可能会是Object类型，Object类型是无法存储基本类型的。

### 6.2 运行时类型检查值适用于原始类型

- 使用getClass会返回一个原始类型，比如Object；
- 使用insatanceof和强制转换都会出现错误和警告；

```jsx
if(a instanceof Pari<String>) //Error
Pair<String> p = (Pair<String>) a;//Error

Pair<Employee> employee = ...
Pair<String> stringPari = ...
if(employee.getClasss()==stringPari.getClass())//返回true
```

### 6.3 不能创建参数化类型数组

不能实例化参数化类型数组，可以声明变量：`Pari<String>[] table`，只是不能new，这样做是为了保证数组安全，因为在类型擦除的时候会变为Object，复制数组可以add任何元素进去。

```java
Pair<String>[] table = new Pair<String>[10];//Error
```

### 6.4 不能实例化类型变量

类型擦除会将T修改为Object，而new Object()是不被允许的，可以通过反射来实例化一个泛型对象。

```cpp
public Pair(){
    first = new T();
    second = new T();
}
```
### 6.5 不能构造泛型数组

因为类型擦除，不允许实例化一个泛型数组，防止add的时候出现ArrayStoreException。

```csharp
public static <T extends Comparable >T[] minmax(T[] a){ //Error
    T[] mm = new T[2];
    ...
}

```

如果想实例化泛型数组，可以通过以下方法来解决：

- 通过反射来解决

```tsx
public static <T extends Comparable> T[] minmax(T ... t){
    T[] mm = (T[]) Array.newInstance(a.getClass().getComponentType(),2);
}
```

## 7 通配符类型

通配符类型中，允许参数类型变化，前面的 ？ extends X，可以让编译器知道只需要某个X的子类型，拒绝传递其他特定类型。

### 7.1 通配符超类型限定

表示类型的下界，格式是：？ super X。

特点：

1、限定为X和X的超类型，直至Object类，因为不知道具体是哪个超类型，因此方法返回的类型只能赋给Object。

2、因为可以向上转型，所以作为方法的参数时，可以传递X以及X的超类型。

3、作为方法的参数时，可以传递null。

作用：主要用来安全地写入数据，可以写入X及其超类型。

```dart
/** 
 * ICE 
 * 2016/10/17 0017 14:12 
 */  
public class Demo {  
    public static void main(String[] args) {  
        A a = new A();  
        B b = new B();  
        C c = new C();  
  
        D<? super A> d = new D<>();  
        Object o = d.get();  
  
        d.set(a);  
        d.set(b);  
        d.set(c);  
        d.set(null);  
    }  
}  
  
class A {  
    @Override  
    public String toString() {  
        return "A{}";  
    }  
}  
  
class B extends A {  
    @Override  
    public String toString() {  
        return "B{}";  
    }  
}  
  
class C extends A {  
    @Override  
    public String toString() {  
        return "C{}";  
    }  
}  
  
class D<T> {  
    public void set(T t) {  
    }  
  
    public T get() {  
        return null;  
    }  
} 
```

### 7.2 无限制

无限定不等于可以传任何值，相反，作为方法的参数时，只能传递null，作为方法的返回时，只能赋给Object。

```java
public class Demo {  
    public static void main(String[] args) {  
        D<?> d = new D<>();  
        Object o = d.get();  
        d.set(null);  
    }  
}  
  
class A {  
    @Override  
    public String toString() {  
        return "A{}";  
    }  
}  
  
class B extends A {  
    @Override  
    public String toString() {  
        return "B{}";  
    }  
}  
  
class C extends A {  
    @Override  
    public String toString() {  
        return "C{}";  
    }  
}  
  
class D<T> {  
    public void set(T t) {  
    }  
  
    public T get() {  
        return null;  
    }  
}  
```

有什么作用呢？对于一些简单的操作比如不需要实际类型的方法，就显得比泛型方法简洁，可以这样说：如果是“读”操作 则需要限定 上边界，如果是写操作则需要限定下边界；而无限定通配符表示只读，不能进行增加、修改。

