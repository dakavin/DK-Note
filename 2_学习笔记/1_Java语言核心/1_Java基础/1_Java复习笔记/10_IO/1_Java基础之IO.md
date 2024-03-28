
# 1、Java IO的体系结构

Java I/O主要包括如下几个层次，包含三个部分：

  1. **流式部分**――IO的主体部分，字节流和字符流

  2. **非流式部分**――主要包含一些辅助流式部分的类，如：File类、RandomAccessFile类和FileDescriptor等类；

  3. **其他类**--文件读取部分的与安全相关的类，如：SerializablePermission类，以及与本地操作系统相关的文件系统的类，如：FileSystem类和Win32FileSystem类和WinNTFileSystem类

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240228003634305.png)

# 2、IO流

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240228003641107.png)

## 2.1 流的概念和作用

流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即**数据在设备间传输称之为流**。

流的**本质**是**数据传输**，根据数据传输的特性将流区分为各种类，方便更直观的进行数据操作。
## 2.2 流的分类

**（1）根据处理数据类型的不同分为：字符流和字节流**
        **1)**  **字节流：** 数据流中最小的数据单元是字节
        **2)**  **字符流：** 数据流中最小的数据单元是字符， Java中的字符是Unicode编码，一个字符占用两个字节。
             **字符流的由来：** Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。为此，JAVA中引入了处理字符的流。因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。

**（2） 根据数据流向不同分为：输入流和输出流**
    **1) 输入流**
	     程序从输入流读取数据源。数据源包括外界(键盘、文件、网络…)，即是将数据源读入到程序的通信通道
     **2) 输出流**
	   程序向输出流写入数据。将程序中的数据输出到外界（显示器、打印机、文件、网络…）的通信通道。
     **3）特性**
	  相对于程序来说，输出流是往存储介质或数据通道写入数据，而输入流是从存储介质或数据通道中读取数据，一般来说关于流的特性有下面几点：

1、先进先出，最先写入输出流的数据最先被输入流读取到。

2、顺序存取，可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（RandomAccessFile**可以从文件的任意位置进行存取（输入输出）操作**）

3、只读或只写，每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

**（3） 按数据来源（去向）分类**：
        1、File（文件）： FileInputStream, FileOutputStream, FileReader, FileWriter
        2、byte[]：ByteArrayInputStream, ByteArrayOutputStream
        3、Char[]: CharArrayReader,CharArrayWriter
        4、String:StringBufferInputStream, StringReader, StringWriter
        5、网络数据流：InputStream,OutputStream, Reader, Writer
## 2.3 字节流

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240228003856307.png)

**（1）输入字节流 InputStream**
        从IO中输入字节流的继承图中可以看出。
        1）InputStream是所有数据字节流的父类，它是一个抽象类。
        2）ByteArrayInputStream、StringBufferInputStream、FileInputStream是三种基本的介质流，它们分别从Byte数组、StringBuffer、和本地文件中读取数据，PipedInputStream是从与其他线程共用的管道中读取数据。
        3）ObjectInputStream和所有FileInputStream 的子类都是装饰流（装饰器模式的主角）。

**InputStream中的三个基本的读方法**
        abstract int read() ：读取一个字节数据，并返回读到的数据，如果返回-1，表示读到了输入流的末尾。
        intread(byte[] b) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾。
        intread(byte[]?b, int off, int len) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾。off指定在数组b中存放数据的起始偏移位置；len指定读取的最大字节数。
        **流结束**的判断：方法read()的返回值为-1时；readLine()的返回值为null时。

**其它方法**
      long skip(long n)：在输入流中跳过n个字节，并返回实际跳过的字节数。
      int available() ：返回在不发生阻塞的情况下，可读取的字节数。
      void close() ：关闭输入流，释放和这个流相关的系统资源。
      voidmark(int?readlimit) ：在输入流的当前位置放置一个标记，如果读取的字节数多于readlimit设置的值，则流忽略这个标记。
      void reset() ：返回到上一个标记。
      booleanmarkSupported() ：测试当前流是否支持mark和reset方法。如果支持，返回true，否则返回false。

**（2）输出字节流 OutputStream**
        从IO中输入字节流的继承图中可以看出。
        1）OutputStream是所有输出字节流的父类，它是一个抽象类。
        2）ByteArrayOutputStream、FIleOutputStream是两种基本的介质，它们分别向Byte 数组，和本地文件中写入数据。PipedOutputStream是从与其他线程共用的管道中写入数据。
        3）ObjectOutputStream和所有FileOutputStream的子类都是装饰流。

**outputStream中的三个基本的写方法**
        abstract void write(int b)：往输出流中写入一个字节。
        void write(byte[] b) ：往输出流中写入数组b中的所有字节。
        void write(byte[] b, int off, int len) ：往输出流中写入数组b中从偏移量off开始的len个字节的数据。

**其它方法**
        void flush() ：刷新输出流，强制缓冲区中的输出字节被写出。
        void close() ：关闭输出流，释放和这个流相关的系统资源。
## 2.4 字符流

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20240228004106854.png)

**（1）字符输入流Reader**
        在上面的继承关系图中可以看出：
        1、Reader是所有的输入字符流的父类，它是一个抽象类。
        2、CharReader、StringReader 是两种基本的介质流，它们分别将Char数组、String中读取数据。PipedInputReader 是从与其他线程共用的管道中读取数据。
        3、BuffereReader 很明显的是一个装饰器，它和其子类复制装饰其他Reader对象。
        4、FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，回增加一个行号。
        5、InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader 可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream转变为Reader 的方法。我们可以从这个类中得到一定的技巧。Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致。

**主要方法：**
         (1) public int read() throws IOException; //读取一个字符，返回值为读取的字符 
        (2) public int read(char cbuf[]) throws IOException; //读取一系列字符到数组cbuf[]中，返回值为实际读取的字符的数量
        (3) public abstract int read(char cbuf[],int off,int len) throws IOException;//读取len个字符，从数组cbuf[]的下标off处开始存放，返回值为实际读取的字符数量，该方法必须由子类实现*/

**（2）字符输出流Writer**
        在上面的关系图中可以看出：
        1、Writer 是所有输出字符流的父类，它是一个抽象类。
        2、CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。PipedInputWriter 是从与其他线程共用的管道中读取数据。
        3、BuffereWriter 很明显是一个装饰器，他和其子类复制装饰其他Reader对象。
        4、FilterWriter 和PrintStream 及其类似，功能和使用也非常相似。
        5、OutputStreamWriter 是OutputStream 到Writer 转换到桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一下SourceCode）。功能和使用OutputStream极其类似。

**主要方法：**
        (1)  public void write(int c) throws IOException； //将整型值c的低16位写入输出流
        (2)  public void write(char cbuf[]) throws IOException； //将字符数组cbuf[]写入输出流
        (3)  public abstract void write(char cbuf[],int off,int len) throws IOException； //将字符数组cbuf[]中的从索引为off的位置处开始的len个字符写入输出流
        (4)  public void write(String str) throws IOException； //将字符串str中的字符写入输出流
        (5)  public void write(String str,int off,int len) throws IOException； //将字符串str 中从索引off开始处的len个字符写入输出流

# 3、非流式

## 3.1 File类

File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。File类保存文件或目录的各种数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名、判断文件是否存在、获取当前目录中的文件列表、创建、删除文件和目录等方法。

## 3.2 RandomAccessFile类

该对象不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。该对象特点：

1、该对象只能操作文件，所以构造函数接受两种数据类型的参数：

· 字符串文件路径

· File对象

2、该对象既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式（r , rw）。

注意：**该对象在实例化时，如果要操作的文件不存在，会自动创建；如果文件存在，写数据未指定位置，会从头开始写，即覆盖原有的内容。**可以用于多线程下载或多个线程同时写数据到文件。

