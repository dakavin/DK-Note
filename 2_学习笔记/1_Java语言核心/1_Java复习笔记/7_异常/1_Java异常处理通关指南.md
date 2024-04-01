
## 1 前言

在理想世界中，程序永远不会出现问题，用户输入的数据永远是正确的，逻辑没有任何问题 ，选择打开的文件也一定是存在的，内存永远是够用的……！但是现实世界里一旦出现这些问题，如果处理不好，程序就不能正常运行了，导致影响用户体验，用户就有可能再也不使用这个程序了。 

出现异常时，对外要给出明确友好的提示消息。对内，程序自己尽量做好补救措施，实在不行了要及时释放占有的资源，以免影响其他线程的任务造成整个程序的崩溃。所以程序的异常处理非常重要。  

在一开始入门学习编程的时候，为了快速学习语法点我们写的都是一些符合预期能执行下去的程序，不过在实际开发项目的时候就不能太单纯了，一定要处处小心，多质疑自己写的还有他人的程序，即使是其他三方库的程序也要质疑一下--出错了该怎么办？直接忽视会不会让我在公司就无了？  

在程序出错的时候，Java 使用的是异常机制，支持将错误信息封装起来，并让程序跳出正常的处理流程，交给异常处理部分去处理。 

说到异常处理，很多人都会想起 Try- Catch语法，下面我们先通过一个简单的代码示例，通过运行和解读这个示例来快速了解一下什么是异常，以及异常处理具有行为习惯。

## 2 Try Catch语句

在Java程序中，使用Try Catch语句来进行程序的异常处理，先看下面这个简单的例子，理解一样Try Catch语句的执行流程。

```csharp
package com.example.learnexception;

public class ExceptionFirstExpression {
    public static void main(String[] args) {
        try {
            int[] arr = new int[1];
            arr[1] = 9;
        } catch (Exception ex) {
            int abc = 999;
            ex.printStackTrace();
        }

        try {
            String str = "";
            str.substring(9, 10);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        System.out.println("程序执行结束");
    }
}
```

在解读上面的程序执行结果前，先了解一下依靠 try catch 语句做异常处理的执行流程。

在 try 语句中如果发生了不符合正确预期的异常情况（Exception），那么程序的执行流程会跳转到 catch 语句中。  

Java会将异常相关信息封装在一个异常类的实例中，上面 catch 代码块中 `ex 变量就是指向这个异常实例的引用`。

处理异常最简单的方法，就是调用异常实例的printStackTrace方法将异常信息打印出来输出到控制台或者日志。  
当 catch 块内的语句执行完毕后，程序会继续向下顺序执行。

上面的程序有两个tray catch 块，可以看到两个try块中的语句都有问题，第一个是数组索引越界（数组长度为1索引只有0），第二个是字符串索引的越界。他们都会导致程序抛出异常，我们执行程序看一下结果：

```css
java.lang.ArrayIndexOutOfBoundsException: 1
    at com.qsc.ebao.insureplan.controller.front.ExceptionFirstExpression.main(ExceptionFirstExpression.java:7)
java.lang.StringIndexOutOfBoundsException: String index out of range: 10
    at java.lang.String.substring(String.java:1963)
    at com.qsc.ebao.insureplan.controller.front.ExceptionFirstExpression.main(ExceptionFirstExpression.java:16)
程序执行结束
```

可以看到程序仍然执行到了最后，不过在两个catch 语句分别捕获了java.lang.ArrayIndexOutOfBoundsException 和 java.lang.StringIndexOutOfBoundsException 两个异常，并把他们的实例赋值给了 ex 。在例程的catch块的异常处理中用，则是用 ex.printStackTrace 打印出了异常明细和导致异常的调用栈。

因为正常捕获到异常并进行了处理，所以上面的例程仍然能平稳执行到最后。

了解了 try catch 语句的执行流程后，我们再来深入地看一下关于异常实例所代表的异常（Exception）在 Java 中的所拥有的体系。

## 3 异常的分类

Java中的异常由 Throwable 类及其子类来描述。Throwable类是Java异常类型的顶层父类，一个对象只有是 Throwable 类或者其子类的实例，它才是一个异常对象，才能被Java的异常处理机制识别。

Java 提供的系统异常有很多，我们还可以自定义异常，不过 Java 的异常分类中我们只需要记住几个关键的分类即可，不必所有异常类的行为全都得背一遍。

下图是一个异常类族的层级图。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aaea9d72ff704da05173e5fb6fdf7071.png)

Throwable有两个直接子类，Error类和Exception类

- Error类表示系统的内部错误和资源耗尽错误，这些错误发生在虚拟机自身、或者发生在虚拟机试图执行应用时，这些异常在应用程序的控制和处理能力之外，一旦发生，Java虚拟机一般会选择线程终止。
- Exception类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。其中RuntimeException和其子类为运行时异常，即非检查异常；而Exception的其他子类为非运行时异常，即受检查异常。

上面异常的类族分级，都不用可以去记，搞不清楚的时候回来复习一下。

`对我们写程序真正有影响，需要我们牢记的是上面描述里提到的非检查异常和受检查异常，这是根据Java对异常的处理要求进行的分类`

- 非检查异常（Unchecked Exception）：Error 和 RuntimeException 以及它们的子类都属于非检查异常。Java不强制一定要在程序里用try catch 或者 throws处理这类异常。这样的异常发生的原因多半是代码写的问题。比如 除数为0错误 ArithmeticException，强制类型转换异常 ClassCastException，数组索引越界 ArrayIndexOutOfBoundException，使用空对象NullPointerException等等。

- 受检查异常（Checked Exception）：除了上面描述的非检查异常意外，其他异常类都属于受检查异常。Java强制要求在程序的方法中通过try catch语句捕获处理它，或者是用throws语句抛出给它的外层，否则编译不会通过。`这类异常一般是由程序的运行环境导致的。程序可能被运行在各种未知的环境下，且无法干预用户如何使用我们编写的程序，于是程序就应该为这样的异常做好处理准备。如：SQLException、IOException、ClassNotFoundException等。`

下面的例程执行try块中的程序时会抛出 ClassNotFoundException， 它是受检查异常，如果不用try catch 处理或者声明要抛出这个异常，是不能通过编译的。

```java
package com.example.learnexception;

public class MustChecked {
    // 这里的throws和try catch 选其一即可， try catch 后 throws 也就没机会执行了。
    public static void main(String[] args) throws ClassNotFoundException  {
        try {
            Class clazz = Class.forName("com.example.learnexception.MustCheckedAbc");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

在这个示例程序里，其实我们已经了解了Java程序的方法里遇到异常情况跑出异常的语法，下面我们把抛出异常的使用场景再好好分析一下。

## 4 抛出异常

抛出异常，分为抛出方法内使用的其他包产生的异常（re-throw） --即不处理，再往外抛，交给上层处理。以及抛出自己代码的异常。

可以使用throws关键字在方法上声明方法可能会抛出得异常，抛出得异常类型可以是实际异常的父类或本身。throws关键字可以声明抛出多个类型的异常，用逗号分开就好。

```java
public class ThrowIt {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class clazz = Class.forName("abc");
        clazz.getField("");
    }

}
```

上面是声明了方法可能会抛出 ClassNotFoundException 和 NoSuchFieldException 异常。  
我们也可以自己创建一个异常然后抛出去，在代码里主动抛出异常，要使用 throw 语句（注意是单数形式，不要和方法声明上的throws搞混了）

```java
package com.example.learnexception;

public class NewAndThrowIt {

    public static void causeException() throws Exception {
        int a = 1;
        if (a < 10) {
            throw new Exception("Custome Exception");
        }
    }

    public static void causeRuntimeException() throws RuntimeException {
        // 可以创建一个unchecked exception，然后用throw语句抛出去
        // 这时候，方法定义里可以有，也可以没有throws语句
        throw new RuntimeException("");
    }

}
```

方法 causeException 创建了一个 checked exception 实例，然后用 throw 关键字扔出去，这时候就需要方法声明里用 throws 明确声明方法会抛出的异常类型。  
方法 causeRuntimeException 创建一个unchecked exception，然后用throw关键字扔出去，这时候，方法声明里可以有，也可以没有throws语句。

在定义接口时，也可以在接口方法声明上加上 throws 语句，限制实现类如果抛出异常的话，必须抛出 throws 声明的类或者其子类。

```java
public interface IntfaceWithEx {

    void methodWithEx() throws Exception;

}
```

## 5 异常的传递和异常链

现实世界里，用Java开发的程序是由一个个类组成的，类的功能由一个个方法来实现，所以一个功能是靠着方法的层层调用实现的，在异常处理时最基本的 ex.printStackTrace() 就是输出产生异常时方法的调用栈--即可以理解为事发现场。  
Java 的异常也是在方法调用栈上传递的，要么沿着方法调用栈一路往上层方法上抛，最终无人处理造成当前线程异常退出，要么被调用栈中的某个方法 Catch 住。  
在异常处理中比较常见的场景是，比如底层 IO 出现异常，我们的程序有可能会 Catch 住这个异常再根据自己的需要抛出新异常，如果这时只是简单地 throw 出创建的新异常，势必会导致原有的异常信息丢失，那么如何保持整个异常的链路呢？在 Throwable 及其子类的重载构造方法中都可以接受一个 Throwable 类型的 cause 参数

```java
public class Exception extends Throwable {
    
    public Exception(String message) {
        super(message);
    }
    
    public Exception(String message, Throwable cause) {
        super(message, cause);
    }
    
    ......
}

public class Throwable implements Serializable {
    
    ......
    public synchronized Throwable getCause() {
        return (cause==this ? null : cause);
    }   
}
```

该参数代表原有的异常，通过 getCause ()就可以获取该原始异常信息。通过这种方法可以保持整个异常链的连续性。

## 6 自定义异常

在写应用程序时经常会需要定义一些自己需要的异常。异常最重要的信息是三块：类型、错误信息和调用栈。

异常类型，在catch、throws 、throw 这些关键字出现的地方都会进行异常类型的判断  
错误信息，描述了异常的详细信息。  
调用栈，出现异常时的调用栈，调用栈是由异常的基类 Throwable 帮助封装的，不需要程序员开发人员人为进行干预。

下面是一个自定义的异常类 UserException，仿照Excption类的重载构造方法，我们也给它定义了三个重载构造方法。

```java
package com.example.learnexception.exceptions;

public class UserException extends Exception {

    public UserException() {
    }

    public UserException(String message) {
        super(message);
    }

    public UserException(String message, Throwable cause) {
        super(message, cause);
    }

    public UserException(Throwable cause) {
        super(cause);
    }
}
```

构造方法里的 Throwable 类型的参数 cause 在上节异常链里已经提过，表示导致当前这个异常的异常。 比如在创建User时，由于数据库网络问题导致了创建失败，那么在抛出 UserException 异常的时候就可以把数据库 IO 异常作为 cause 传给UserException的构造方法创建 UserException 异常实例。  
下面是一个使用我们自定义异常 UserException 的示例程序

```java
package com.example.learnexception;

import com.example.learnexception.exceptions.UserException;

public class ExCaller {

    public void callThrowException() throws UserException {
        // 可以在这里catch异常，然后封装成自己的异常，并增加相应的异常描述
        System.out.println("ExCaller.callThrowException开始");
        try {
            Class.forName("com.example.TestForException");
        } catch (ClassNotFoundException ex) {
            throw new UserException("there's something wrong", ex);
        }
        System.out.println("ExCaller.callThrowException 方法调用结束");
    }

}
```

上面的例程我们在方法里创建一个 TestForException 类的实例，因为不存在这个类，所以try 块里会抛出 ClassNotFoundException 异常，在catch里我们捕获了这个异常，封装成自己的异常，并增加了相应的异常描述。

```java
package com.example.learnexception;

import com.example.learnexception.exceptions.UserException;

public class CallExceptionAppMain {
    public static void main(String[] args) {
        ExCaller caller = new ExCaller();
        try {
            caller.callThrowException();
        } catch (UserException ex) {
            ex.printStackTrace();
        }
    }
}
```

catch 语句是根据异常类型匹配来捕捉相应类型的异常的。如果类型不匹配，catch语句是不会执行的，异常会继续抛出。  
也就是说，如果使用 catch (Throwable) 会捕捉到所有的异常，包括Error，但是建议 catch 的范围尽量精准， 不要用范围太大的异常类父类，比如 Exception 在 catch 时就尽量少用，实在不行也最多只捕捉 Exception 类型的异常就行了，不要使用 Throwable 。

注意：

如果 catch 一个其实并没有可能被抛出的受检查异常，Java程序会报错，因为 Java明确的知道这个类型的异常不会发生。  
如果 catch 一个非检查异常，Java程序不会报错。  
如果throws一个其实并没有被抛出的checked exception或者unchecked exception，Java程序不会报错。

掌握了自定义异常后，让我们在回到Try-Catch语法上，上面我们一直用的其实不是这个语法的完全态，让我们再下一节仔细说说。

## 7 Try Catch Finally
  
try catch 语句的形式可以扩展成try catch finally 的形式。下面通过一个示例程序，了解一下，加了 finally 之后的作用。

```csharp
package com.example.learnexception;

public class TryCatchFinallyAppMain {

    private static int VAL = 0;

    public static void main(String[] args) {
        System.out.println(withFinally());
        System.out.println(VAL);
    }

    private static int withFinally() {
        int len = 0;
        try {
            String s = null;
            return s.length();
        } catch (Exception ex) {
            // 异常的处理：在有返回值的情况下，返回一个特殊的值，代表情况不对，有异常
            len = -1;
            System.out.println("执行catch里的return语句");
            return len;
        } finally {
            // 可以认为finally语句会在方法返回后，后面的方法开始前，会在return语句后
            // 无论是因为return结束还是因为异常结束，finally语句都会执行
            System.out.println("执行finally语句");
            // finally里最好不要有return语句
//            return -2;

            // finally里给前面 return 表达式里的变量值赋值没用
            len = -2;

            VAL = 999;
            System.out.println("finally语句执行完毕");
        }
    }


}
```

不管是否有异常产生，finally块中代码都会执行，`无论是因为 return 结束还是因为异常结束，finally语句都会在方法退出后执行`。 

finally是在 return 后面的表达式运算后执行的，所以`函数返回值是在 finally 执行前确定的`。无论finally 中的代码怎么样，返回的值都不会改变，仍然是之前return语句中保存的值，所以我们不要在 finally 的代码块里尝试修改要 return 给调用者的返回值。  

再有一点还要注意，finally中最好不要包含return，否则程序会提前退出，返回值不再是 try 或 catch 中保存的返回值。

其实 finally 因为其最后必定执行的特点，特别适合于关闭文件、网络IO等释放程序占用资源的操作，不过因为还是有可能会被人为忘记，所以 Java 又在 1.7 版本引入了新语法，避免遗忘释放资源。

## 8 可自动回收资源的Try语句

和网络、文件等资源相关的异常处理起来比较繁杂，尤其是在处理多个资源的时候，比如读取文件内容再通过网络发送出去，在文件资源操作和网络资源操作的时候都可能出错（文件找不到，权限不对，网络连接不上等等）这种情况下就要需要在 finally 块中把资源主动关闭。 但有可能会被遗漏， 所以后来 java 1.7 就有了 try-with-resources ---支持自动关闭资源的 try 语句。  
那怎么让 try 语句自动帮助我们关闭异常呢？首先这个资源要能支持自动关闭，Java 里只要实现了 AutoCloseable 接口的类的实例，就会被认为是一个可被自动关闭的资源。

```java
package com.example.learnexception;

import java.io.IOException;

public class AutoClosableTestResource implements AutoCloseable {

    private String resourceName;
    private int counter;

    public AutoClosableTestResource(String resName) {
        this.resourceName = resName;
    }

    // read 方法会随便的读取出数据，或者抛出IOException
    public String read() throws IOException {
        counter++;
        if (Math.random() > 0.1) {
            return "You got lucky to read from " + resourceName + " for " + counter + " times...";
        } else {
            throw new IOException("resource不存在哦");
        }
    }

    @Override
    public void close() throws Exception {
        System.out.println("资源释放了:" + resourceName);
    }
}
```

到时候 try 语句就会帮助我们自动调用 close 方法释放掉资源。接下来看怎么在 try-with-resources 语法里使用这个资源。

```csharp
package com.example.learnexception;

public class TryWithResource {
    public static void main(String[] args) {
        try (
                AutoClosableTestResource res1 = new AutoClosableTestResource("res1");
                AutoClosableTestResource res2 = new AutoClosableTestResource("res2")
        ) {
            while (true) {
                System.out.println(res1.read());
                System.out.println(res2.read());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

跟常规的try catch语句相比，try 后的小括号里创建可以自动关闭的资源，这里创建的资源能被 try 代码块里的代码使用，等出现异常或者正常退出的时候就会帮我们释放掉这里的资源。

我们执行以下上面的例程，可以看到以输出。

```css
You got lucky to read from res1 for 1 times...
You got lucky to read from res2 for 1 times...
You got lucky to read from res1 for 2 times...
You got lucky to read from res2 for 2 times...
资源释放了:res2
资源释放了:res1
java.io.IOException: resource不存在哦
    at com.example.learnexception.AutoClosableTestResource.read(AutoClosableTestResource.java:20)
    at com.example.learnexception.TryWithResource.main(TryWithResource.java:10)
```

try 后面括号里创建的资源，在程序退出前都被释放了。我们后面再写与资源操作有关的代码时，尽量都使用这种形式，日常咱们开发能接触到的资源也都已经实现了 AutoCloseable，支持被自动关闭。  
总结  
本篇文章，把 Java 的异常处理的知识点以及实际开发中的应用盘点了一遍，内容比较多，建议看完文章理解就行，不用非得全部记住，需要重点关注的知识点在文章里已经做重点说明，后面实际开发项目时，也可以多拿这篇内容做参考

