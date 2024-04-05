
## 1 问题描述

- 代码如下：
```java
InputStream is1 = ClassLoader.getSystemResourceAsStream("JDBC.properties");

InputStream is2 = JDBCUtils.class.getClassLoader().getResourceAsStream("JDBC.properties");
```
- 在java环境下运行，is1 和 is2 都不为空，可以加载配置文件
- 在Tomcat的环境下运行，is1 为空，加载不了配置文件

## 2 原因分析

- getResourceAsStream会先使用本类的类加载器去加载，本类没有类加载器，才会使用系统类加载器。也就是说getResourceAsStream功能覆盖了getSystemResourceAsStream，所以推荐直接使用getResourceAsStream就完事了。

## 3 Java中的类加载器

- 1.**启动类加载器**（Bootstrap ClassLoader）：顶层的类加载器，没有父类加载器。负责加载 /lib 目录下的，或被 -Xbootclasspath 参数所指定路径中的，并被 JVM 识别的（仅按文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录也不会被加载）类库加载到虚拟机内存中。所有被 Bootstrap classloader 加载的类，它的 Class.getClassLoader 方法返回的都是 null，所以也称作 NULL ClassLoader。


- 2.**扩展类加载器**（Extension CLassLoader）：由 sun.misc.Launcher$ExtClassLoader 实现，负责加载 <JAVA_HOME>/lib/ext 目录下，或被 java.ext.dirs 系统变量所指定的目录下的所有类库；


- 3.**应用程序类加载器**（Application/System ClassLoader）：由 sun.misc.Launcher$AppClassLoader 实现。`它是 ClassLoader.getSystemClassLoader() 方法的默认返回值，所以也称为系统类加载器（System ClassLoader）`。它负责加载 classpath 下所指定的类库，如果应用程序没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

## 4 tomcat容器下类加载器

- Tomcat官方说明：[Class Loader HOW-TO](http://tomcat.apache.org/tomcat-8.0-doc/class-loader-howto.html "Class Loader HOW-TO")

- 跟普通的java程序相比，类加载大体顺序相同。

```
   Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ...
```

- 1.**Bootstrap 启动类加载器**: 加载JVM启动所需的类+系统扩展目录（$JAVA_HOME/jre/lib/ext）里 JAR 文件中的类。

- 2.**System 系统类加载器**：从 CLASSPATH 系统变量指定的目录中加载类库。该加载器加载的类对 tomcat 本身和 web 应用都可见。

`！！！但是，标准的 tomcat 启动脚本`（$CATALINA_HOME/bin/catalina.sh or %CATALINA_HOME%\bin\catalina.bat）`都会忽略系统变量 CLASSPATH 的值，而会使用如下的类库来创建 System 类加载器`：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ecdbd96d797a1eb4de2096d59691cc8.png)


- 3.**Common 通用类加载器**：通过该类加载器加载的类库可被 Tomcat 和所有应用共享。该类加载器的搜索位置是通过 $CATALINA_BASE/conf/catalina.properties 文件中的 common.loader 属性指定的，默认包括如下位置：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10a58289d8d55720abe7eec98ac89c42.png)


- 4.**WebappX 应用类加载器**：每个 Web 应用创建一个自己的类加载器，加载自己项目下的数据： /WEB-INF/classes 和 /WEB-INF/lib 下的类和资源。`并且不使用双亲委派机制，先自己加载，加载不到才使用父类加载器`。

- 本来顺序是1234，但是WebappX不使用委派机制而是先自己加载，加载不了才使用父类，所以**真实的顺序**是：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cbab6a4b8cf54675367085a1ed5a52ed.png)

- tomcat8支持委托：配置允许委派：<Loader delegate="true"/>，顺序变为：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9cd0971e3169fb58192947709b1c82ec.png)
## 5 最终结论

tomcat容器中运行的java程序，使用系统类加载器是不能获取到资源的，必须使用WebappClassLoader。使用getResourceAsStream获取当前类的类加载器，也就是WebappClassLoader，自然可以获取到资源了。