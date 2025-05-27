---
文章标题: "[[1_彻底搞懂使用MyBatis时为什么Dao层不需要@Repository]]" 
文章作者: Dakkk
文章概要: |
  分析MyBatis框架中Dao层接口不需要@Repository注解的原因，通过源码解析ClassPathMapperScanner如何扫描接口并注册为MapperFactoryBean类型的Bean
tags:
- "MyBatis"
- "Spring"
- "Bean注册"
- "MapperScan"
- "动态代理"
- "源码分析"
- "Dao层"
- "框架原理"
相关文章:
- "[[11_反射]]"
- "[[0_课程介绍]]"
- "[[4_Java反射机制详解]]"
- "[[1_ArrayList源码分析]]"
- "[[1_Java反射]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/1_彻底搞懂使用MyBatis时为什么Dao层不需要@Repository.md"
文章难度: 高级 🔥
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---


参考文章 : [彻底搞懂使用MyBatis时为什么Dao层不需要@Repository_mybatis中不需要用@respository-CSDN博客](https://blog.csdn.net/tianshan2010/article/details/105889133)
## 1 问题

Service层注入Dao时， Intellij 总会以红色波浪线提示我们
```java
@Autowired
private UserDao userDao;
```

Could not autowire. No beans of ‘UserDao’ type found.
Checks autowiring problems in a bean class.

尽管我们都知道 Dao 层的 Bean 实际上都是有的，并且可以设置关闭这恼人的提示，但是我们有没有想过为什么 Intellij 就找不到这个 Bean 呢？甚至有人有这种做法
```java
@Repository
public interface UserDao {
}
```

来避免提示，但是这种做法正确么？

所以今天我们的疑问就是
1. 为什么 Dao 层不需要加 @Repository 注解，源码里到底做了什么？
2. 加了 @Repository 注解有什么影响？

## 2 答案

1. 关键在于 ClassPathMapperScanner 对指定包的扫描，并且扫描过程对 Spring 原本的扫描 Bean 的步骤 “加了料” ，Spring 本身只扫实现类，但 MyBatis 的扫描器扫了接口 。并且扫完接口之后，为接口配了个 BeanDefinition ,并且这个 bd 的 BeanClass 是 MapperFactoryBean 。

> 对于 BeanDefinition 和 MapperFactoryBean 不了解的同学请查询相关资料和源码

2. 仅仅只能解决 Intellij 静态查找 bean 的问题，没有实际作用。即使加了注解，比如@Controller，@Service 等等，也会被 Spring 的扫描器给忽略掉，因为**扫描器会过滤掉接口**。

## 3 源码探索

> 下面的源码部分如果读者提前有 MyBatis 的 Bean 的执行流程，和 Spring 的 Bean加载的相关知识就更好理解
### 3.1 分析问题

关于为什么不需要注解就能获取到 Dao 层的 Bean，看似答案很简单，因为配置了扫描指定这个包里的 xxxDao.class 啊，比如使用注解 @MapperScan(“com.example.dao”)。

这个答案太过表面，觉得问题简单只是因为对 Spring 的 Bean 不熟悉。

我们何时见过 @Component 及其衍生的3个注解 @Controller、@Service、@Repository 加在接口上面的？

自己测试新建个接口，上面加注解，然后找个 Controller 里 @Autowired 注入一下，项目立马会报错 NoSuchBeanDefinitionException 。
### 3.2 切入源码

切入点

既然使用注解 @MapperScan 就好使，那么我们就从这个点切入源码看一下，先找出源码中何处用了此注解，非常幸运的是，只有一处用到了此注解 ：MapperScannerRegistrar.registerBeanDefinitions() 。

并且从类名和方法名就可以很清楚的看出这个类的功能是扫描 Mapper 并注册，方法的功能就是注册 BeanDefinitions 到 Spring 中。方法的源码我就不贴了，很容易看出来是创建一个扫描器 ClassPathMapperScanner ，设置好一系列属性比如 Spring 的注册表之后，执行 doScan() 方法去扫描 @MapperScan 提供的包。

doScan() 扫描资源，转换为 BeanDefinition

doScan() 方法也很简单，就是两步：
- 第一步 : 调用父类 ClassPathBeanDefinitionScanner 的doScan()方法，也就是 Spring 扫描BeanDefinition 的方法。过程不是很重要，我们需要知道这个扫描方法的一个关键就是
```java
Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
```
- 在其中对所有的候选者使用 isCandidateComponent() 方法判断是否为符合要求的 BeanDefinition。
```java
for (TypeFilter tf : this.excludeFilters) {
	if (tf.match(metadataReader, this.metadataReaderFactory)) {
		return false;
	}
}
for (TypeFilter tf : this.includeFilters) {
	if (tf.match(metadataReader, this.metadataReaderFactory)) {
		return isConditionMatch(metadataReader);
	}
}
```

这有两组过滤器来过滤扫描到的资源。**Spring 默认的过滤器是排除掉抽象类/接口的。而MyBatis 的扫描器重新注册了过滤器，默认对接口放行。**

> 其实还有一些其它的过滤要求，但是不影响我们本问题的探究，所以不深入解读了。

源码读到这里，我们先找到了本文的第二个问题的答案。也就是 Spring 会忽略掉接口上面的注解，不会添加它进入 BeanDefiniiton ,也就难怪测试的时候会抛出 NoSuchBeanDefinitionException 的异常了。而 MyBatis 则会把这些接口拉过来注册BD 。

对 BeanDefinition 的加工

读到这里我们可能有了更大的疑问，拿接口注册 BeanDefinition ，那获取 Bean 的时候如何去实例化这个对象啊？接口可是不能实例化出对象的啊，而且我们也没有做实现。

原来是 MyBatis 的扫描器在调用完父类的扫描方法后，对 BeanDefinition 进行了加工 processBeanDefinitions() 。其中最关键的两行代码是

```java
definition.getConstructorArgumentValues().addGenericArgumentValue(definition.getBeanClassName());

definition.setBeanClass(this.mapperFactoryBean.getClass());
```

第一行，我们发现把这个**接口的类名塞到了构造器参数中**。
- 这里塞的是 String ，而我们的构造器参数其实要的是 Class 。但是 Spring 的 ConstructorResolver.autowireConstructor 中用到了 Object[] argsToUse 去做了个转换 。
第二行，beanDefinition 的 **BeanClass 被设置成了 MapperFactoryBean** !

熟悉 Spring 和 MyBatis 的读者肯定一下就明白了，就是这个地方进行了”偷梁换柱”！
```java
@Autowired
private UserDao userDao;
```

还是拿 UserDao 为例，我们向 Spring 容器说 “给我来个 UserDao 的实例”，而 Spring 根据注册时候的 BeanDefinition ，去工厂( MapperFactoryBean )里面扔了个 UserDao.class 的参数进去，工厂的 getObject() 方法给我们返回了它制造的 userDao 。

就这样，我们没有去写实现类，轻轻松松拿到了我们需要的 userDao 。

至于 MapperFactoryBean 里做了什么返回了 userDao 出来？其实就是它的 getObject 方法返回的是 DefaultSqlSession.getMapper(Class type)方法，返回的是 MapperProxy 代理的类，而这个代理类的 invoke 方法并不像我们平时见到的代理中的 invoke 方法一样调用原始目标的 method.invoke ，而是去找 MapperMethod 执行了。