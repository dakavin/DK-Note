
# 一、Maven介绍

## 1、什么是Maven

### 1.1 概念

- Maven 的正确发音是[ˈmevən]，而不是“马瘟”以及其他什么瘟。Maven 在美国是一个口语化的词语，代表专家、内行的意思。

- 一个对 Maven 比较正式的定义是这么说的：`Maven 是一个项目管理工具`，它包含了
- `一个项目对象模型 (POM：Project Object Model)`，
- `一组标准集合`
- `一个项目生命周期(Project Lifecycle)`，
- `一个依赖管理系统(Dependency Management System)`，
- 和用来运行`定义在生命周期阶段(phase)中插件(plugin)目标 (goal)的逻辑`

### 1.2 Maven能解决什么问题

- 可以用更通俗的方式来说明。我们知道，项目开发不仅仅是写写代码而已，期间会伴随着各种必不可少的事情要做，下面列举几个感受一下：

	1. 我们需要引用各种 jar 包，尤其是比较大的工程，引用的 jar 包往往有几十个乃至上百个， 每用 到一种 jar 包，都需要手动引入工程目录，而且经常遇到各种让人抓狂的`jar 包冲突，版本冲突`。 
	2. 我们辛辛苦苦写好了 Java 文件，可是只懂 0 和 1 的白痴电脑却完全读不懂，需要将它`编译`成二进制字节码。好在现在这项工作可以由各种集成开发工具帮我们完成，Eclipse、IDEA 等都可以将代 码即时编译。当然，如果你嫌生命漫长，何不铺张，也可以用记事本来敲代码，然后用 javac 命令一 个个地去编译，逗电脑玩。 
	3. 世界上没有不存在 bug 的代码，计算机喜欢 bug 就和人们总是喜欢美女帅哥一样。为了追求美为了减少 bug，因此写完了代码，我们还要写一些`单元测试`，然后一个个的运行来检验代码质量。 
	4. 再优雅的代码也是要出来卖的。我们后面还需要把代码与各种配置文件、资源整合到一起，定型 打包，如果是 web 项目，还需要将之`发布到服务器`，供人蹂躏。

- 试想，如果现在有一种工具，可以把你从上面的繁琐工作中解放出来，能帮你构建工程，管理 jar 包，编译代码，还能帮你自动运行单元测试，打包，生成报表，甚至能帮你部署项目，生成 Web 站 点，你会心动吗？Maven 就可以解决上面所提到的这些问题。

### 1.3 Maven 的优势举例

- 前面我们通过 Web 阶段项目，要能够将项目运行起来，就必须将该项目所依赖的一些 jar 包添加到 工程中，否则项目就不能运行。试想如果具有相同架构的项目有十个，那么我们就需要将这一份 jar 包复制到十个不同的工程中。我们一起来看一个 CRM项目的工程大小。

- 使用传统 Web 项目构建的 CRM 项目如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075458827.png)


- 原因主要是因为上面的 WEB 程序要运行，我们必须将项目运行所需的 Jar 包复制到工程目录中，从 而导致了工程很大。

- 同样的项目，如果我们使用 Maven 工程来构建，会发现总体上工程的大小会少很多。如下图:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075501851.png)


- 小结：可以初步推断它里面一定没有 jar 包，继续思考，没有 jar 包的项目怎么可能运行呢？

## 2、Maven的两个经典作用

### 2.1 Maven 的依赖管理

- Maven 的一个核心特性就是`依赖管理`。当我们涉及到多模块的项目（包含成百个模块或者子项目），管理依赖就变成 一项困难的任务。Maven 展示出了它对处理这种情形的高度控制。

- 传统的 WEB 项目中，我们必须将工程所依赖的 jar 包复制到工程中，导致了工程的变得很大。那么 maven 工程是如何使得工程变得很少呢？
- 分析如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075505325.png)


- 通过分析发现：maven 工程中不直接将 jar 包导入到工程中，而是通过`在 pom.xml 文件中添加所需 jar 包的坐标`，这样就很好的避免了 jar 直接引入进来，在需要用到 jar 包的时候，只要查找 pom.xml 文 件，再通过 pom.xml 文件中的坐标，到一个专门用于”存放 jar 包的仓库”(maven 仓库)中根据坐标从 而找到这些 jar 包，再把这些 jar 包拿去运行。

- 那么问题来了 
- 第一：”存放 jar 包的仓库”长什么样？ 
- 第二：通过读取 pom.xml 文件中的坐标，再到仓库中找到 jar 包，会不会很慢？从而导致这种方式 不可行！

- `第一个问题`：存放 jar 包的仓库长什么样，这一点我们后期会分析仓库的分类，也会带大家去看我们 的本地的仓库长什么样。

- `第二个问题`：通过 pom.xml 文件配置要引入的 jar 包的坐标，再读取坐标并到仓库中加载 jar 包，这 样我们就可以直接使用 jar 包了，为了解决这个过程中速度慢的问题，maven 中也有索引的概念，通 过建立索引，可以大大提高加载 jar 包的速度，使得我们认为 jar 包基本跟放在本地的工程文件中再 读取出来的速度是一样的。这个过程就好比我们查阅字典时，为了能够加快查找到内容，书前面的 目录就好比是索引，有了这个目录我们就可以方便找到内容了，一样的在 maven 仓库中有了索引我 们就可以认为可以快速找到 jar 包。

### 2.1 项目的一键构建

- 我们的项目，往往都要经历编译、测试、运行、打包、安装 ，部署等一系列过程。
- 什么是构建？
	- 指的是项目从编译、测试、运行、打包、安装 ，部署整个过程都交给 maven 进行管理，这个 过程称为构建。
- `一键构建`指的是整个构建过程，使用 maven 一个命令可以轻松完成整个工作。


- Maven 规范化构建流程如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075508647.png)


- 我们一起来看 Hello-Maven 工程的一键运行的过程。通过 tomcat:run 的这个命令，我们发现现在的 工程编译，测试，运行都变得非常简单。

# 二、Maven的使用

## 1、Maven的安装

### 1.1 Maven 软件的下载

- 为了使用 Maven 管理工具，我们首先要到官网去下载它的安装软件。通过百度搜索“Maven“如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075510989.png)


- 点击 Download 链接，就可以直接进入到 Maven 软件的下载页面：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075529018.png)



### 1.2 Maven 软件的安装

- Maven 下载后，将 Maven 解压到一个没有中文没有空格的路径下，比如 `E:\Maven\apache-maven-3.9.3` 下面。 解压后目录结构如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075531299.png)


- bin:存放了 maven 的命令，比如我们前面用到的 mvn tomcat:run 
- boot:存放了一些 maven 本身的引导程序，如类加载器等 
- conf:存放了 maven 的一些配置文件，如 setting.xml 文件 
- lib:存放了 maven 本身运行所需的一些 jar 包 
- 至此我们的 maven 软件就可以使用了，前提是你的电脑上之前已经安装并配置好了 JDK。

### 1.3 JDK 的准备及统一

- 本次课程我们所使用工具软件的统一，JDK 使用 JDK8版本

### 1.4 Maven 及 JDK 配置

- 电脑上需安装 java 环境，安装 JDK1.7 + 版本 （将JAVA_HOME/bin 配置环境变量 path ），我们使 用的是 JDK8 相关版本

- 配置 MAVEN_HOME ，变量值就是你的 maven 安装 的路径（bin 目录之前一级目录）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075534058.png)
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075539686.png)


- 上面配置了我们的 Maven 软件，注意这个目录就是之前你解压 maven 的压缩文件包在的的目录，最 好不要有中文和空格。

- 再次检查 JDK 的安装目录，如下图:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075554599.png)
### 1.5 Maven 软件版本测试

- 通过 mvn -v命令检查 maven 是否安装成功，看到 maven 的版本为 3.5.2 及 java 版本为 1.8 即为安装 成功。

- 找开 cmd 命令，输入 mvn –v命令，如下图：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075610488.png)
- 我们发现 maven 的版本，及 jdk 的版本符合要求，这样我们的 maven 软件安装就成功了。

## 2、Maven仓库

### 2.1 Maven 仓库的分类

- maven 的工作需要从仓库下载一些 jar 包，如下图所示，本地的项目 A、项目 B 等都会通过 maven 软件从远程仓库（可以理解为互联网上的仓库）下载 jar 包并存在本地仓库，本地仓库 就是本地文 件夹，当`第二次需要此 jar 包时则不再从远程仓库下载，因为本地仓库已经存在了`，可以将`本地仓库理解为缓存`，有了本地仓库就不用每次从远程仓库下载了。

- 下图描述了 maven 中仓库的类型：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075626083.png)

- `本地仓库` ：用来存储从远程仓库或中央仓库下载的插件和 jar 包，项目使用一些插件或 jar 包， 优先从本地仓库查找 默认本地仓库位置在`${user.dir}/.m2/repository，${user.dir}`表示 windows 用户目录。
- ![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075631093.png)



- `远程仓库`：如果本地需要插件或者 jar 包，本地仓库没有，默认去远程仓库下载。 远程仓库可以在互联网内也可以在局域网内。***在maven高级课中会有讲解*** [6_Maven高级](../../5_SSM/2_SSM（黑马）/6_Maven高级.md)

- `中央仓库` ：在 maven 软件中内置一个远程仓库地址 http://repo1.maven.org/maven2 ，它是中 央仓库，服务于整个互联网，它是由 Maven 团队自己维护，里面存储了非常全的 jar 包，它包 含了世界上大部分流行的开源项目构件。

### 2.2 Maven 本地仓库的配置

- 本课程是在无网的状态下学习，需要配置老师提供的本地仓库，将 “repository.rar”解压至自己的 电脑上，我们解压在 `D:\repository` 目录下（可以放在没有中文及空格的目录下）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075642287.png)


- 在 MAVE_HOME/conf/settings.xml 文件中配置本地仓库位置（maven 的安装目录下）：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075649985.png)


- 打开 settings.xml文件，配置如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075657296.png)

### 2.3 全局 setting 与用户 setting

- `maven 仓库地址、私服等配置信息需要在 setting.xml 文件中配置`，分为全局配置和用户配置。
- 在 maven 安装目录下的有 conf/setting.xml 文件，此 setting.xml 文件用于 maven 的所有 project 项目，它作为 maven 的全局配置。
- 如需要个性配置则需要在用户配置中设置，用户配置的 setting.xml 文件默认的位置在：`${user.dir} /.m2/settings.xml 目录中,${user.dir} `指 windows 中的用户目录。
- maven 会先找用户配置，如果找到则以用户配置文件为准，否则使用全局配置文件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075722295.png)



## 3、Maven工程的认识

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075800016.png)



### 3.1 Maven 工程的目录结构

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075731386.png)



- 作为一个 maven 工程，它的 `src 目录和 pom.xml 是必备的`。 进入 src 目录后，我们发现它里面的目录结构如下：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075734354.png)


- src/main/java —— 存放项目的.java 文件 
- src/main/resources —— 存放项目资源文件，如 spring, hibernate 配置文件 
- src/test/java —— 存放所有单元测试.java 文件，如 JUnit 测试类
- src/test/resources —— 测试资源文件 target —— 项目输出位置，编译后的 class 文件会输出到此目录 
- pom.xml——maven 项目核心配置文件

- `注意：如果是普通的 java 项目，那么就没有 webapp 目录。`

### 3.2 Maven 工程的运行

- 进入 maven 工程目录（当前目录有 pom.xml 文件），运行 tomcat:run 命令![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075740528.png)


- 根据上边的提示信息，通过浏览器访问：http://localhost:8080/maven-helloworld/![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075743519.png)



### 3.3 问题处理

- 如果本地仓库配置错误会报下边的错误![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075746136.png)

- 分析：maven 工程运行先从本地仓库找 jar 包，本地仓库没有再从中央仓库找，上边提示 downloading… 表示 从中央仓库下载 jar，由于本地没有联网，报错。

- 解决： 在 maven 安装目录的 conf/setting.xml 文件中配置本地仓库，参考：[2.2 Maven 本地仓库的配置](#2.2%20Maven%20本地仓库的配置)


# 三、Maven常用命令

## 1、Maven常用命令

- 我们可以在 cmd 中通过一系列的 maven 命令来对我们的 maven-helloworld 工程进行编译、测试、运 行、打包、安装、部署。

### 1.1 compile

- `compile 是 maven 工程的编译命令`，作用是将 src/main/java 下的文件编译为 class 文件输出到 target 目录下。

- cmd 进入命令状态，执行 mvn compile，如下图提示成功：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075811778.png)
- 查看 `target 目录，class 文件已生成，编译完成`。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075813927.png)
### 1.2 test

- `test 是 maven 工程的测试命令 mvn test`，会执行 src/test/java 下的单元测试类。
- cmd 执行 mvn test 执行 src/test/java 下单元测试类，下图为测试结果，运行 1 个测试用例，全部成功![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075824606.png)
### 1.3 clean

- clean 是 maven 工程的清理命令，`执行 clean 会删除 target 目录及内容`
### 1.4 package

- package 是 maven 工程的打包命令，对于` java 工程执行 package 打成 jar 包`，`对于 web 工程打成 war 包`。
- 会先进行编译操作，再执行打包操作
### 1.5 install

- install 是 maven 工程的安装命令，`执行 install 将 maven 打成 jar 包或 war 包发布到本地仓库中的cn文件夹中`。 
- 从运行结果中，可以看出： 
- 当后面的命令执行时，前面的操作过程也都会自动执行，
## 2、Maven指令的生命周期

### 2.1 介绍

- Maven的生命周期就是为了对所有的构建过程进行抽象和统一。 描述了一次项目构建，经历哪些阶段。

- 在Maven出现之前，项目构建的生命周期就已经存在，软件开发人员每天都在对项目进行清理，编译，测试及部署。虽然大家都在不停地做构建工作，但公司和公司间、项目和项目间，往往使用不同的方式做类似的工作。

- Maven从大量项目和构建工具中学习和反思，然后总结了一套高度完美的，易扩展的项目构建生命周期。这个生命周期包含了项目的清理，初始化，编译，测试，打包，集成测试，验证，部署和站点生成等几乎所有构建步骤。

- Maven对项目构建的生命周期划分为3套（相互独立）：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075917715.png)

- `clean：清理工作。`
    
- `default：核心工作。如：编译、测试、打包、安装、部署等。`
    
- `site：生成报告、发布站点等。`

- 三套生命周期又包含哪些具体的阶段呢, 我们来看下面这幅图:![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075922376.png)


- 我们看到这三套生命周期，里面有很多很多的阶段，这么多生命周期阶段，其实我们常用的并不多，主要关注以下几个：
	• clean：移除上一次构建生成的文件
	• compile：编译项目源代码
	• test：使用合适的单元测试框架运行测试(junit)
	• package：将编译后的文件打包，如：jar、war等
	• install：安装项目到本地仓库

- Maven的生命周期是抽象的，这意味着生命周期本身不做任何实际工作。***在Maven的设计中，实际任务（如源代码编译）都交由插件来完成。**

- IDEA工具为了方便程序员使用maven生命周期，在右侧的maven工具栏中，已给出快速访问通道![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080038605.png)

- 生命周期的顺序是：clean --> validate --> compile --> test --> package --> verify --> install --> site --> deploy

- 我们需要关注的就是：`clean --> compile --> test --> package --> install`

- `说明：在同一套生命周期中，我们在执行后面的生命周期时，前面的生命周期都会执行。`

## 3、Maven的概念模型

- Maven 包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段 (phase)中插件(plugin)目标(goal)的逻辑![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080053608.png)


- 项目对象模型 (Project Object Model)
	- 一个 maven 工程都有一个 pom.xml 文件，通过 pom.xml 文件定义项目的坐标、项目依赖、项目信息、 插件目标等。
- 依赖管理系统(Dependency Management System)
	- 通过 maven 的依赖管理对项目所依赖的 jar 包进行统一管理。
	- 比如：项目依赖 junit4.9，通过在 pom.xml 中定义 junit4.9 的依赖即使用 junit4.9，如下所示是 junit4.9 的依赖定义：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080110408.png)


- 一个项目生命周期(Project Lifecycle)
	- 使用 maven 完成项目的构建，项目构建包括：清理、编译、测试、部署等过程，maven 将这些 过程规范为一个生命周期，如下所示是生命周期的各各阶段：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080113835.png)


	- maven 通过执行一些简单命令即可实现上边生命周期的各各过程，比如执行 mvn compile 执行编译、 执行 mvn clean 执行清理。

- 一组标准集合 
	- maven 将整个项目管理过程定义一组标准，比如：通过 maven 构建工程有标准的目录结构，有 标准的生命周期阶段、依赖管理有标准的坐标定义等。

- 插件(plugin)目标(goal)
	- `maven 管理项目生命周期过程都是基于插件完成的`。

# 四、 IDEA集成Maven

- 我们要想在IDEA中使用Maven进行项目构建，就需要在IDEA中集成Maven

## 1、配置Maven环境 

### 1.1 当前工程设置

1、选择 IDEA中 File  =>  Settings  =>  Build,Execution,Deployment  =>  Build Tools  =>  Maven

2、设置IDEA使用本地安装的Maven，并修改配置文件及本地仓库路径
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080131715.png)




- `Maven home path` ：指定当前Maven的安装目录
- `User settings file` ：指定当前Maven的settings.xml配置文件的存放路径
- `Local repository` ：指定Maven的本地仓库的路径 (如果指定了settings.xml, 这个目录会自动读取出来, 可以不用手动指定)

3、配置工程的编译版本为8

- Maven默认使用的编译版本为5（版本过低）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080134827.png)


- 上述配置的maven环境，只是针对于当前工程的，如果我们再创建一个project，又恢复成默认的配置了。 要解决这个问题， 我们就需要配置全局的maven环境。

### 1.2 全局设置

1、进入到IDEA欢迎页面

- 选择 IDEA中 File => close project![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080138134.png)



2、打开 All settings , 选择 Build,Execution,Deployment  =>  Build Tools  =>  Maven![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080141108.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080149600.png)



3、配置工程的编译版本为8![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080155212.png)


- 这里所设置的maven的环境信息，并未指定任何一个project，`此时设置的信息就属于全局配置信息`。 以后，我们再创建project，默认就是使用我们全局配置的信息。

## 2、Maven项目

### 2.1 创建Maven-Java项目（无骨架）

- 在Project中新建一个Maven项目![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080158296.png)


- 此时test文件夹中缺少resource文件夹，需要补齐![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080201897.png)



- 在pom.xml文件中补齐打包方式![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080205588.png)



- 在Maven工程下，创建HelloWorld类![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080208695.png)



### 2.2 创建Maven-Java项目（有骨架）

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080211633.png)



- 缺少resource文件夹，需要补齐![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080215633.png)


- 自定义maven命令运行配置![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080218534.png)


### 2.3 创建Maven-Web项目（无骨架）

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080243742.png)



- 补齐webapp目录![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080246448.png)


- 创建一个index.html页面，使用tomcat插件（`可以使用alt+ins快捷键`，快速创建plugins模版）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080248246.png)


- 启动tomcat![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080249955.png)


- 修改端口号和虚拟目录![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080252534.png)


- 创建一个servlet，需要导入jar包（即在pom文件中添加依赖）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080253908.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080256815.png)

### 2.4 创建Maven-web项目（有骨架）

参考文章：[IDEA2022版本创建maven web项目（两种方式）最全图文教学_idea创建web+maven项目-CSDN博客](https://blog.csdn.net/weixin_45384457/article/details/128532296)

## 3、POM配置详解

- POM (Project Object Model) ：指的是项目对象模型，用来描述当前的maven项目。

- 使用pom.xml文件来实现
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- POM模型版本 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 当前项目坐标 -->
    <groupId>com.dakkk</groupId>
    <artifactId>maven_project1</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <!-- 打包方式 -->
    <packaging>jar</packaging>
 
</project>
```

- `pom文件详解`：
	- `<project>` ：pom文件的根标签，表示当前maven项目
	- `<modelVersion>` ：声明项目描述遵循哪一个POM模型版本
		- 虽然模型本身的版本很少改变，但它仍然是必不可少的。目前POM模型版本是4.0.0
	- `坐标` ：`<groupId>`、`<artifactId>`、`<version>`
		- 定位项目在本地仓库中的位置，由以上三个标签组成一个坐标
	- `<packaging>` ：maven项目的打包方式，通常设置为jar或war（默认值：jar）

## 4、Maven坐标详解

- 什么是坐标？
	- Maven中的坐标是`资源的唯一标识` , 通过该坐标可以唯一定位资源位置
	- 使用坐标来定义项目或引入项目中需要的依赖

- Maven坐标主要组成
	- `groupId`：定义当前Maven项目隶属组织名称（通常是域名反写，例如：com.itheima）
	- `artifactId`：定义当前Maven项目名称（通常是模块名称，例如 order-service、goods-service）
	- `version`：定义当前项目版本号

- 如下图就是使用坐标表示一个项目：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080311322.png)


**注意：**
- 上面所说的资源可以是`插件、依赖、当前项目`。
- 我们的项目如果被其他的项目依赖时，也是需要坐标来引入的。

## 5、导入maven项目

### 5.1 使用Maven面板，快速导入项目

- 打开IDEA，选择右侧Maven面板，点击 + 号，选中对应项目的pom.xml文件，双击即可![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080320899.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080324447.png)


- 说明：如果没有Maven面板，选择 View => Appearance => Tool Window Bars![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080326769.png)



### 5.2 使用idea导入模块项目

- File => Project Structure => Modules => + => Import Module![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080329768.png)


- 找到要导入工程的pom.xml![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080331809.png)



# 五、依赖管理

## 1、依赖配置

### 1.1 配置

- 依赖：指当前项目运行所需要的jar包。一个项目中可以引入多个依赖：
- 例如：在当前工程中，我们需要用到logback来记录日志，此时就可以在maven工程的pom.xml文件中，引入logback的依赖。具体步骤如下：
	1. 在pom.xml中编写`<dependencies>`标签
	2. 在`<dependencies>`标签中使用`<dependency>`引入坐标
	3.  定义坐标的` groupId、artifactId、version`
```xml
<dependencies>
    <!-- 第1个依赖 : logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.11</version>
    </dependency>
    <!-- 第2个依赖 : junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

4. 点击刷新按钮，引入最新加入的坐标
	- 刷新依赖：保证每一次引入新的依赖，或者修改现有的依赖配置，都可以加入最新的坐标![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080335659.png)



- 注意事项：
1. 如果引入的依赖，在本地仓库中不存在，将会连接远程仓库 / 中央仓库，然后下载依赖（这个过程会比较耗时，耐心等待）
2. 如果不知道依赖的坐标信息，可以到mvn的中央仓库（[https://mvnrepository.com/](https://mvnrepository.com/)）中搜索

### 1.2 添加依赖的几种方式

- 利用中央仓库搜索的依赖坐标
- 利用IDEA工具搜索依赖
	- 下载Maven-search插件，重启IDEA后，可以在Tools中看到
- 熟练上手maven后，快速导入依赖
## 2、依赖传递

### 2.1 依赖具有传递性

- 早期我们没有使用maven时，向项目中添加依赖的jar包，需要把所有的jar包都复制到项目工程下。如下图所示，需要logback-classic时，由于logback-classic又依赖了logback-core和slf4j，所以必须把这3个jar包全部复制到项目工程下

- 我们现在使用了maven，当项目中需要使用logback-classic时，`只需要在pom.xml配置文件中，添加logback-classic的依赖坐标即可`。

- 在pom.xml文件中只添加了logback-classic依赖，但由于maven的依赖具有传递性，所以会自动把所依赖的其他jar包也一起导入。

- 依赖传递可以分为：
1. 直接依赖：在当前项目中通过依赖配置建立的依赖关系
2. 间接依赖：被依赖的资源如果依赖其他资源，当前项目间接依赖其他资源
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080354089.png)



- 比如以上图中：
	- projectA依赖了projectB。对于projectA 来说，projectB 就是直接依赖。
	- 而projectB依赖了projectC及其他jar包。 那么此时，在projectA中也会将projectC的依赖传递下来。对于projectA 来说，projectC就是间接依赖。

### 2.2 排除依赖

- 问题：之前我们讲了依赖具有传递性。那么A依赖B，B依赖C，如果A不想将C依赖进来，是否可以做到？ 

- 答案：在maven项目中，我们可以通过排除依赖来实现。

- 排除依赖：指主动断开依赖的资源。（被排除的资源无需指定版本）

```xml
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>maven-projectB</artifactId>
    <version>1.0-SNAPSHOT</version>
   
    <!--排除依赖, 主动断开依赖的资源-->
    <exclusions>
    	<exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 3、 依赖范围

- 在项目中导入依赖的jar包后，默认情况下，可以在任何地方使用。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080404444.png)


- 如果希望限制依赖的使用范围，可以通过`<scope>`标签设置其作用范围。
	1. 主程序范围有效（main文件夹范围内）
	2. 测试程序范围有效（test文件夹范围内）
	3. 是否参与打包运行（package指令范围内）
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080408752.png)



- 如上图所示，给junit依赖通过scope标签指定依赖的作用范围。 那么这个依赖就只能作用在测试环境，其他环境下不能使用。

- scope标签的取值范围：

| **scope**值     | **主程序** | **测试程序** | **打包（运行）** | **范例**    |
| --------------- | ---------- | ------------ | ---------------- | ----------- |
| compile（默认） | Y          | Y            | Y                | log4j       |
| test            | -          | Y            | -                | junit       |
| provided        | Y          | Y            | -                | servlet-api |
| runtime         | -          | Y            | Y                | jdbc驱动    |

- `servlet-api要使用provided，因为tomcat本身jar包中有这个api，会发生冲突`

## 5、 更新依赖索引

- 有时候给idea配置完maven仓库信息后，在idea中依然搜索不到仓库中的jar包。这是因为仓库中的jar包索引尚未更新到idea中。这个时候我们就需要更新idea中maven的索引了，具体做法如下：

- 打开设置----搜索maven----Repositories----选中本地仓库-----点击Update![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080419730.png)



## 6、 清理maven仓库

- 初始情况下，我们的本地仓库是没有任何jar包的，此时会从私服去下载（如果没有配置，就直接从中央仓库去下载），可能由于网络的原因，jar包下载不完全，这些不完整的jar包都是以lastUpdated结尾。此时，maven不会再重新帮你下载，需要你删除这些以lastUpdated结尾的文件，然后maven才会再次自动下载这些jar包。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080429821.png)



- 如果本地仓库中有很多这样的以lastUpadted结尾的文件，可以定义一个批处理文件，在其中编写如下脚本来删除：
```bat
set REPOSITORY_PATH=E:\develop\apache-maven-3.6.1\mvn_repo
rem 正在搜索...

del /s /q %REPOSITORY_PATH%\*.lastUpdated

rem 搜索完毕
pause
```

操作步骤如下：

1). 定义批处理文件del_lastUpdated.bat  (直接创建一个文本文件，命名为del_lastUpdated，后缀名直接改为bat即可 )


2). 在上面的bat文件上**右键---》编辑** 。修改文件：



修改完毕后，双击运行即可删除maven仓库中的残留文件。

# 六、Maven工程运行调试

## 1、 端口占用处理

- 重新执行 tomcat:run 命令重启工程，重启之前需手动停止 tomcat，否则报下边的错误：![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080453027.png)



## 2、 断点调试

- 点击如图所示选项![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080456386.png)


- 在弹出框中点击如图加号按钮找到 maven 选项![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080458963.png)


- 在弹出窗口中填写如下信息![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080501059.png)


- 完成后先 Apply 再 OK 结束配置后，可以在主界面找到我们刚才配置的操作名称。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922080519842.png)

- 如上图红框选中的两个按钮，左侧是正常启动，右侧是 debug 启动。

# 五、总结

## 1、Maven仓库

1、maven 仓库的类型有哪些？ 
2、maven 工程查找仓库的流程是什么？ 
3、本地仓库如何配置？

## 2、常用的Maven命令

常用 的 maven 命令包括： 
compile：编译 
clean：清理
test：测试
package：打包
install：安装

## 3、坐标定义

在 pom.xml 中定义坐标，内容包括：`groupId`、artifactId、version，详细内容如下：
```xml
<!--项目名称，定义为组织名+项目名，类似包名-->
<groupId>com.itheima</groupId>
<!-- 模块名称 -->
<artifactId>hello_maven</artifactId>
<!-- 当前项目版本号，snapshot 为快照版本即非正式版本，release 为正式发布版本 -->
<version>0.0.1-SNAPSHOT</version>
<packaging>：打包类型
	jar：执行 package 会打成 jar 包 
	war：执行 package 会打成 war 包
```

## 4、pom基本配置

`pom.xml 是 Maven 项目的核心配置文件`，位于每个工程的根目录，基本配置如下：

```xml
<project>：文件的根节点 . 
<modelversion>： pom.xml 使用的对象模型版本 
<groupId>：项目名称，一般写项目的域名 
<artifactId>：模块名称，子项目名或模块名称 
<version>：产品的版本号 . 
<packaging>：打包类型，一般有 jar、war、pom 等 
<name>：项目的显示名，常用于 Maven 生成的文档。 
<description>：项目描述，常用于 Maven 生成的文档 
<dependencies>：项目依赖构件配置，配置项目依赖构件的坐标 
<build>：项目构建配置，配置编译、运行插件等。
```