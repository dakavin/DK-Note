---
文章标题: "[[4_JavaEE的13项规范]]" 
文章作者: Dakkk
文章概要: |
  文章概述了JavaEE的13项核心规范，包括JDBC、JNDI、EJB、JSP、Servlet等，涵盖数据访问、分布式、Web开发、消息服务和事务管理等关键领域。它简洁介绍了各规范的功能和作用，为理解JavaEE技术栈提供了基础概览。
tags:
- "JavaEE"
- "JDBC"
- "EJB"
- "Servlet"
- "JSP"
- "JMS"
- "JTA"
- "分布式系统"
相关文章:
- "[[10_EL&JSTL]]"
- "[[18_日程管理开发]]"
- "[[7_Servlet、HTTP和Request]]"
- "[[9_Cookie&Session]]"
- "[[0_项目介绍]]"
文章分类: "📚 知识管理"
文章路径: "01-📚 知识管理/03-💡 思考感悟/4_JavaEE的13项规范.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 20:52:58
---

## 1 JDBC(Java DataBase Connectivity)

java数据连接，是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问。有了JDBC就不用因为不同的数据库而要写个不同的应用程序，开发人员只需要使用JDBC API写一个程序就够了。

## 2 JNDI(Java Naming and Directory Interface)

Java命名和目录接口，提供了一种统一的方式可以在网络上查找和访问服务，通过指定一个资源名称，该名称对应于数据库或命名服务中的一个记录，同时返回数据库连接建立所必需的信息。

在DataSource中事先建立多个数据库连接，保存在数据库连接池中，当程序访问数据库时，只用从连接池中取空闲状态的数据库连接即可，访问结束，销毁资源，数据库链接重新回到连接池。



## 3 EJB(Enterprise JavaBean)

EJB是sun的JavaEE服务器端组件模型，设计目标与核心应用是部署分布式应用程序。简单来说就是把已经编写好的程序（即：类）打包放在服务器上执行。凭借java跨平台的优势，用EJB技术部署的分布式系统可以不限于特定的平台。包括四种对象类型：无状态会话bean（提供独立服务）,有状态会话bean（提供会话交互），实体bean（持久性数据在内存中的体现，服务器崩溃后可恢复），消息驱动bean。

## 4 RMI(Remote Method Invoke)

远程方法调用，能够让在某个java虚拟机上的对象调用本地对象一样的调用另一个java虚拟机中高的对象上的方法。

## 5 JSP(Java Server Pages)

java服务器页面，是一个动态内容模板，实现了Html语法中的java扩展。

## 6 Servlet

Servlet是一种小型的Java程序，它扩展了Web服务器的功能。作为一种服务器端的应用，当被请求时开始执行，这和CGI Perl脚本很相似。Servlet提供的功能大多与JSP类似，不过实现的方式不同。JSP通常是大多数HTML代码中嵌入少量的Java代码，而servlets全部由Java写成并且生成HTML。

## 7 XML(Extensible Markup Language)

是一种可扩展的标记语言，被用来在不同的商务过程中共享数据，其目标是平台独立性，记得在学习XML的时候，可以自己写标签，只要有结束标签就可以识别，还是相当强大的。

## 8 JMS(Java Message Service)

是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。消息通信可以是点对点的，也可以是发布订阅型的。

## 9 Java IDL

Java IDL支持的是一个瞬间的CORBA对象，即在对象服务器处理过程中有效。实际上，Java IDL的ORB是一个类库而已，并不是一个完整的平台软件，但它对Java IDL应用系统和其他CORBA应用系统之间提供了很好的底层通信支持，实现了OMG定义的ORB基本功能。

## 10 JTS(object transaction monitor)

组件事务监视器，TPM 是一个程序，它代表应用程序协调分布式事务的执行。TPM 与数据库出现的时间长短差不多；在 60 年代后期，IBM 首先开发了 CICS，至今人们仍在使用。经典的（或者说 程序化）TPM 管理被程序化定义为针对事务性资源（比如数据库）的操作序列的事务。随着分布式对象协议，如 CORBA、DCOM 和 RMI 的出现，人们希望看到事务更面向对象的前景。将事务性语义告知面向对象的组件要求对 TPM 模型进行扩展 ― 在这个模型中事务是按照事务性对象的调用方法定义的。JTS 只是一个组件事务监视器（有时也称为 对象事务监视器（object transaction monitor）），或称为 CTM。

## 11 JTA(Java Transaction API)

JTA允许应用程序执行分布式事务处理——在两个或多个网络计算机资源上访问并且更新数据。JDBC驱动程序的JTA支持极大地增强了数据访问能力。

## 12 JavaMail

提供给开发者处理电子邮件相关的编程接口。

## 13 JAF(JavaBeans Activation Framework)

JAF是一个专用的数据处理框架，它用于封装数据，并为应用程序提供访问和操作数据的接口。
