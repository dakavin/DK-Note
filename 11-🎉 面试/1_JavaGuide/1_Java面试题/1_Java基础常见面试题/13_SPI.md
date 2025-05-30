---
文章标题: "[[13_SPI]]" 
文章作者: Dakkk
文章概要: |
  本文详细解释了Java SPI机制，强调其作为服务提供者接口如何实现服务接口与实现的解耦，从而提升系统扩展性与可维护性。文章对比了SPI与API的区别，并分析了SPI的优缺点。
文章标签:
- "Java SPI"
- "Service Provider Interface"
- "API"
- "解耦"
- "扩展性"
- "Java核心"
- "设计模式"
相关文章:
- "[[1_面向对象]]"
- "[[1_牛客网错题集]]"
- "[[1_JDBC概述]]"
- "[[11_单例模式的6种实现方式]]"
- "[[2_静态代理的继承性理解]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/1_JavaGuide/1_Java面试题/1_Java基础常见面试题/13_SPI.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 01:06:00
---

## 1 何谓SPI？

SPI 即 Service Provider Interface ，字面意思就是：“服务提供者的接口”，我的理解是：专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口。

SPI 将服务接口和具体的服务实现分离开来，将服务调用方和服务实现者解耦，能够提升程序的扩展性、可维护性。修改或者替换服务实现并不需要修改调用方。

很多框架都使用了 Java 的 SPI 机制，比如：Spring 框架、数据库加载驱动、日志接口、以及 Dubbo 的扩展实现等等。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0129b49bfecf62489cc0ca7046ce2fc.png)

`回答思路：`
	1. 顾名思义，SPI（Server Provider Interface）指的是服务提供者的接口，即专门给服务提供者（框架开发者）去使用的接口；
	2. 将 服务接口 和 服务实现 分离，将 服务调用方 和 服务实现者 解耦，提高程序的扩展性、可维护性；
	3. 例如：Spring框架，JDBC、日志接口以及Dubbo的扩展
## 2 SPI和API有什么区别？

**那 SPI 和 API 有啥区别？**

说到 SPI 就不得不说一下 API 了，从广义上来说它们都属于接口，而且很容易混淆。下面先用一张图说明一下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/96a2bc63b09c10d02c75d8c12cf71226.png)
一般模块之间都是通过接口进行通讯，那我们在服务调用方和服务实现方（也称服务提供者）之间引入一个“接口”。

当实现方提供了接口和实现，我们可以通过调用实现方的接口从而拥有实现方给我们提供的能力，这就是 API ，这种接口和实现都是放在实现方的。

当接口存在于调用方这边时，就是 SPI ，由接口调用方确定接口规则，然后由不同的厂商去根据这个规则对这个接口进行实现，从而提供服务。

举个通俗易懂的例子：公司 H 是一家科技公司，新设计了一款芯片，然后现在需要量产了，而市面上有好几家芯片制造业公司，这个时候，只要 H 公司指定好了这芯片生产的标准（定义好了接口标准），那么这些合作的芯片公司（服务提供者）就按照标准交付自家特色的芯片（提供不同方案的实现，但是给出来的结果是一样的）。

`回答思路：`
	1. API：接口靠近实现方，我们调用实现方的接口来处理业务；
	2. SPI：接口靠近调用方，调用方确定接口的规则，然后不同的厂商根据规则对该接口进行实现；
	3. 总结：API即现成的接口实现类，可以直接用；SPI是编写接口的规则，实现方需要按照接口的规则来编码实现类；
## 3 SPI的优缺点？

通过 SPI 机制能够大大地提高接口设计的灵活性，但是 SPI 机制也存在一些缺点，比如：

- 需要遍历加载所有的实现类，不能做到按需加载，这样效率还是相对较低的。
- 当多个 `ServiceLoader` 同时 `load` 时，会有并发问题。

`回答思路：`
	1. 优点：提高接口设计的灵活性，便于接口调用者和实现者之间的解耦和分离；
	2. 缺点：见上（还是不明白！）
