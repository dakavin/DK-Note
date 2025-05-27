---
文章标题: "[[1_Java反射]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了Java反射机制的定义、功能与应用，包括Class、Field、Method、Constructor等核心API的用法。深入阐述了代理模式，特别是Java动态代理的实现原理及与反射的结合，并简述了反射与泛型的应用。
tags:
- "Java"
- "反射"
- "动态代理"
- "Class类"
- "Field"
- "Method"
- "Constructor"
- "泛型"
相关文章:
- "[[11_反射]]"
- "[[7_DAO及相关实现类]]"
- "[[0_基础加强]]"
- "[[10_泛型]]"
- "[[12_注解]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/02-📚 基础语法/2_Java复习笔记/9_反射/1_Java反射.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:36:04
---

## 1 概述

### 1.1 反射机制定义

Java反射机制是在运行状态中，对任意一个类，都能知道这个类中的所有属性和方法；对任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制

- Java反射机制的功能
	1. 在运行时判断任意一个对象所属的类
	2. 在运行时构造任意一个类的对象
	3. 在运行时判断任意一个类所具有的属性和方法
	4. 在运行时调用任意一个对象的方法
	5. 生成动态代理

- Java反射机制的应用场景
	1. 逆向代码，例如反编译
	2. 与注解相结合的框架；例如Rettrofit
	3. 单纯的反射机制应用框架：例如EventBus
	4. 动态生成类框架；例如json；

## 2 通过Java反射查看类信息

### 2.1 反射机制相关类

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ded99faf489a0a01a75c0c6d506f90d.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5203aa908f85799a1e32bb94870f77f4.png)

### 2.2 Class类

[Class](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fjava%2Flang%2FClass)代表类的实体，在运行的Java应用程序中表示类和接口。在这个类中提供了很多有用的方法，这里对他们简单的分类介绍。

常用的获取Class对象的3种方式：

```csharp
//1、使用Class类的静态方法
class = Class.forName("java.lang.String");
//2、使用类的.class语法。
class = String.class;
//3、使用对象的getClass()方法。如：
String str = "aa";
Class<?> classType1 = str.getClass();
```

- 获得类相关的方法：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/66044bbbac8277f1229a06f7492b63f7.png)

- 获得类中属性相关的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5017af4dc420dbf85e1efef1447fdf4a.png)

- 获得类中注解相关的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9bce1f05e3f663fe6ac35faf1e96727b.png)

- 获得类中构造器相关的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a54ce7013e4707e99ab2287c939e66f6.png)

- 获得类中方法相关的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42dc97809da0d96a320acbb51f13628c.png)

- 类中其他重要的方法
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab832837da6d5eb53e4b63fd1730bd6f.png)

### 2.3 Field类

[Field](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fjava%2Flang%2Freflect%2FField)代表类的成员变量（成员变量也称为类的属性）。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6642e37d3b888ddf81f603f529bd9507.png)

### 2.4 Method类

[Method](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fjava%2Flang%2Freflect%2FMethod)代表类的方法。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6b699f8e1faf5c55256c072122d86985.png)

```java
try {
	//反射得到类 包名+类名，我这边没有包名就直接是 "ReflectionTest"
	Class clazz = Class.forName("com.alan.test.Book");
	//直接java反射得到方法
	Method method = clazz.getMethod("setName", String.class);
	method.setAccessible(true);
	method.invoke(clazz.newInstance(), "xiaojun");
} catch (NoSuchMethodException | IllegalAccessException | InstantiationException | InvocationTargetException | ClassNotFoundException e) {
	e.printStackTrace();
}
```

当通过Method的invoke()方法来调用对应的方法时，Java会要求程序必须有调用该方法的权限。如果程序确实需要调用某个对象的private方法，则可以先调用Method对象的如下方法。  
setAccessible(boolean flag)：将Method对象的acessible设置为指定的布尔值。  
值为true，指示该Method在使用时应该取消Java语言的访问权限检查；值为  
false，则知识该Method在使用时要实施Java语言的访问权限检查。

### 2.5 Constructor类

[Constructor](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fjava%2Flang%2Freflect%2FConstructor)代表类的构造方法。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/641c5dd5598ac5d0dde1350ebea9db6d.png)

## 3 代理模式

### 3.1 概述

定义：给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象问接地操控原对象。

#### 3.1.1 代理模式的理解

代理模式使用代理对象完成用户请求，屏蔽用户对真实对象的访问。现实世界的代理人被授权执行当事人的一些事宜，无需当事人出面，从第三方的角度看，似乎当事人并不存在，因为他只和代理人通信。而事实上代理人是要有当事人的授权，并且在核心问题上还需要请示当事人。在软件设计中，使用代理模式的意图也很多，比如因为安全原因需要屏蔽客户端直接访问真实对象，或者在远程调用中需要使用代理类处理远程方法调用的技术细节，也可能为了提升系统性能，对真实对象进行封装，从而达到延迟加载的目的。
#### 3.1.2 代理模式的参与者

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f84c166dd12e69142d292091b9fcc7cc.png)

代理模式分为四个角色:

- 主题接口：`Subject 是委托对象和代理对象都共同实现的接口`，即代理类的所实现的行为接口。Request()是委托对象和代理对象共同拥有的方法。
- 目标对象：RealSubject 是原对象，也就是被代理的对象。
- 代理对象：Proxy是代理对象，用来封装真是主题类的代理类。
- 客户端：使用代理类和主题接口完成一些工作。
#### 3.1.3 代理模式的分类

代理的实现分为：

- 静态代理：代理类是在`编译时就实现好的`。也就是说Java 编译完成后代理类是一个实际的 class 文件；
- 动态代理：`代理类是在运行时生成的`。也就是说 Java 编译完之后并没有实际的class 文件，而是在运行时动志生成的类字节码，并加载到JVM中。
#### 3.1.4 代理模式的实现思路

- 代理对象和目标对象均实现同一个行为接口
- 代理类和目标类分别具体实现接口逻辑。
- 在代理类的构造函数中实例化一个目标对象。
- 在代理类中调用目标对象的行为接口。
- 客户端想要调用目标对象的行为接口，只能通过代理类来操作。
#### 3.1.5 静态代理模式的简单实现

假如有这样的需求，要在某些模块方法调用前后加上一些统一的前后处理操作，比如在添加购物车、修改订单等操作前后统一加上登陆验证与日志记录处理，该怎样实现？首先想到最简单的就是直接修改源码，在对应模块的对应方法前后添加操作。如果模块很多，你会发现，修改源码不仅非常麻烦、难以维护，而且会使代码显得十分臃肿。这时侯就轮到代理模式上场了，它可以在被调用方法前后加上自己的操作，而不需要更改被调用类的源码，大大地降低了模块之问的耦合性，体现了极大的优势。

### 3.2 Java斥射机制与动态代理

#### 3.2.1 动态代理介绍

动态代理是指在运行时动态生成代理类。即代理类的字节码将在运行时生成并载入当前代理的 ClassLoader。与静态处理类相比，动态类有诸多好处。

1. 不需要为(RealSubject )写一个形式上完全一样的封装类，假如主题接口  
    (Subiect）中的方法很多，为每一个接口写一个代理方法也很麻烦。如果接口有变动，则目标对象和代理类都要修改，不利于系统维护；
2. 使用一些动态代理的生成方法甚至可以在运行时制定代理类的执行逻辑，从而大大提升系统的灵活性。
#### 3.2.2 动态代理涉及的主要类

主要涉及两个类，这两个类都是java lang.reflect包下的类，内部主要通过反射来实现的。

1. java.lang.reflect.Proxy：  
    这是生成代理类的主类，通过 Proxy 类生成的代理类都继承了 Proxy 类。Proxy提供了用户创建动态代理类和代理对象的静态方法，它是所有动态代理类的父类。
    
2. java.lang.reflect.InvocationHandler:  
    这里称他为"调用处理器"，它是一个接口。当调用动态代理类中的方法时，将会直接转接到执行自定义的InvocationHandler中的invoke(方法。即我们动态生成的代理类需要完成的具体内容需要自己定义一个类，而这个类必须实现 InvocationHandler 接口，通过重写invoke()方法来执行具体内容。
    

Proxy提供了两个方法：

```java
//1、创建动态代理类
public static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces) throws IllegalArgumentException {
        Class<?> caller = System.getSecurityManager() == null ? null : Reflection.getCallerClass();
        return getProxyConstructor(caller, loader, interfaces).getDeclaringClass();
    }
//2、创建动态代理实例
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) {
        Objects.requireNonNull(h);
        Class<?> caller = System.getSecurityManager() == null ? null : Reflection.getCallerClass();
        Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);
        return newProxyInstance(caller, cons, h);
    }

参数解析：
ClassLoader loader：类加载器对象，哪个类加载器加载这个代理类到JVM方法区；
Class<?>[] interfaces：接口，表明代理类需要实现哪些接口；
InvocationHandler h：调用处理器类的实例，指定代理类干什么；
```
#### 3.2.3 动态代
理简单实现

```java
public class DynamicProxyDemo {
    public static void main(String[] args) {
        //1.创建目标对象
        RealSubject realSubject = new Realsubject () ;
        //2.创建调用处理器对象
        ProxyHandler handler = new ProxyHandler (realSubject);
        //3.动态生成代理对象
        Subject proxySubject = (Subject)Proxy.newProxyInstance(RealSubject.class.getClassLoader(),RealSubject.class.getInterfaces(), handler) ;
        //4.通过代理对象调用方法
        proxySubject.request);
    }
}

//主题接口
interface Subject{
  void request();
}

//目标对象类
class RealSubject implements Subject{
    public void request(){
        System.out.println("====RealSubject Request====");
    }
}

//代理类调用处理器
class ProxyHandler implements InvocationHandler {
        private Subject subject;

        public ProxyHandler(Subject subject) {
            this.subject = subject;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            //定义预处理的工作，当然你也可以根据method 的不同进行不同理工作
            System.out.println("====before===");
            //调用Realsubiect中的方法
            Object result = method.invoke(subject, args);
            System.out.println("====after===");
            return result;
        }
    }
```

## 4 泛型与Class类

### 4.1 概述

从JDK 1.5后，Java中引入泛型机制，Class 类也增加了泛型功能，从而允许使用泛型来限制Class类。例如：String.class的类型实际上是`Class<String>`。  
如果Class对应的类暂时未知，则使用`Class<?>`(?是通配符)。通过反射中使用泛型，可以避免使用反射生成的对象需要强制类型转换。泛型的好处众多，最主要的一点就是避免类型转换，防止出现ClassCastException，即类型转换异常。
### 4.2 使用反射来获取泛型信息

通过指定类对应的 Class 对象，可以获得该类里包含的所有Field，不管该Field 是使用 private 修饰，还是使用public 修饰。获得了Field 对象后，就可以很容易地获得该 Field 的数据类型，即使用如下代码即可获得指定Field 的类型。

```java
//获取Field对象的类型
Class<?> a = f.getType();
```

但这种方式只对普通类型的Field 有效。如果该Field 的类型是有泛型限制的类  
型，如Map<String, Integer-类型，则不能准确地得到该Field 的泛型参数。  
为了获得指定 Field 的泛型类型，应先使用如下方法来获取指定Field 的类型。

```java
//获取Field实例的泛型类型
Type type= f.getGenericType();
```

然后将Type 对象强制类型转换为 ParameterizedType 对象，ParameterizedType代表被参数化的类型，也就是增加了泛型限制的类型。  
Parameterized Type 类提供了如下两个方法：  
getRaw Type()：返回没有泛型信息的原始类型。  
getActualTypeArguments()：返回泛型参数的类型。

   




































































