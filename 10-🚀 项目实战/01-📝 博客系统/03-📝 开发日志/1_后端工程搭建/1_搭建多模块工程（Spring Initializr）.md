
## 1 什么是多模块项目？

多模块项目是项目构建中的概念。拿 Maven 来说，多模块项目（Multi-Module Project）是其一个重要特性，**它允许我们在一个项目中管理多个子模块。**

在一个 Maven 多模块项目中，**每个模块都是一个独立的项目，拥有自己的 POM 文件（Project Object Model，项目对象模型）**。这些模块可以互相依赖，也可以被其他项目依赖。但是，所有的模块都会被统一管理，它们共享同一套构建系统和依赖管理。

Maven 多模块项目的结构大概是下面这样的：
```text
my-app/ (父项目) 
|- pom.xml 
|- my-module1/ (子模块1) 
	|- pom.xml 
|- my-module2/ (子模块2) 
	|- pom.xml 
| ... (实际企业级项目中，会分非常多的模块)
```

在这个例子中，`my-app` 是父项目，`my-module1` 和 `my-module2` 是它的子模块。每个模块都有自己的 `pom.xml` 文件。

## 2 为什么需要多模块项目？

主要有以下几个原因：

- **代码组织**：在大型项目中，我们经常需要把代码分成多个模块，以便更好地组织代码。==每个模块可以聚焦于一个特定的功能或领域==，这样可以提高代码的可读性和可维护性。
    
- **依赖管理**：Maven 多模块项目可以帮助我们更好地管理项目的依赖。在==父项目的 POM 文件==中，我们==可以定义所有模块共享的依赖==，这样可以避免重复的依赖定义，也方便我们管理和升级依赖。
    
- **构建和部署**：Maven 多模块项目的另一个优点是它可以==统一管理项目的构建和部署==。我们只需要在父项目中执行 Maven 命令，就可以对所有模块进行构建和部署。这大大简化了开发者的工作。

## 3 IDEA搭建Springboot多模块工程骨架

### 3.1 开始

首先选择一个位置，新建一个名为 `blog-backend` 的工程目录，用于统一存放后端项目和前端项目，方便后续管理：

### 3.2 创建父项目

打开 IDEA, 依次点击菜单 _File -> New -> Project_, 准备新建**父项目** :
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/09a73f576ab045531e9b68dceda83b98.png)

> **注意**：IDEA 通过 Spring Initializr 来创建 Spring Boot 项目， 突然不支持勾选 Java 8 了 ， 点击小齿轮，将初始化链接换成阿里云的 `http://start.aliyun.com`, 就可以正常选择 Java 8 了

点击 Next ，进入下一步，配置 Spring Boot，不用选择依赖，直接创建
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d3ef62de495516418c19c9a45eeb7fed.png)

因为这是个父项目，专门负责统一管理子模块、依赖版本等，创建完成后，在左边导航栏中，先删除下图中标注的无用文件：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c704758ffda3eab0ef2a61a1374f263.png)

接下来，开始整理一下父项目的 `pom.xml` 文件，整理后内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <parent>        
	    <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <!-- 将 Spring Boot 的版本号切换成 2.6 版本 -->  
        <version>2.6.3</version>  
        <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
  
    <groupId>com.dakkk</groupId>  
    <artifactId>blog-backend</artifactId>  
    <version>${revision}</version>  
    <name>blog-backend</name>  
    <!-- 项目描述 -->  
    <description>前后端分离博客 blog By dakkk</description>  
  
    <!-- 多模块项目父工程打包模式必须指定为 pom -->    <packaging>pom</packaging>  
  
    <!-- 子模块管理 -->  
    <modules>  
    </modules>  
    <!-- 版本号统一管理 -->  
    <properties>  
        <!-- 项目版本号 -->  
        <revision>0.0.1-SNAPSHOT</revision>  
        <java.version>1.8</java.version>  
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
  
        <!-- Maven 相关 -->  
        <maven.compiler.source>${java.version}</maven.compiler.source>  
        <maven.compiler.target>${java.version}</maven.compiler.target>  
  
    </properties>  
  
    <!-- 统一依赖管理 -->  
    <dependencyManagement>  
  
    </dependencyManagement>  
    <build>        
    <!-- 统一插件管理 -->  
        <pluginManagement>  
  
        </pluginManagement>    
	</build>  
  
    <!-- 使用阿里云的 Maven 仓库源，提升包下载速度 -->  
    <repositories>  
        <repository>  
            <id>aliyunmaven</id>  
            <name>aliyun</name>  
            <url>https://maven.aliyun.com/repository/public</url>  
        </repository>  
    </repositories>  
</project>
```

### 3.3 创建web访问模块（打包也在这个模块进行）

接下来，我们开始创建父项目下面的子模块。在父项目上**右键**，添加模块 `Module`:

还是和上面创建父项目差不多的步骤，命名一个 `blog-web` 模块，此模块是项目的入口，Maven 打包的打包插件放在这里，同时，和博客前台页面展示相关的功能也统一放在此模块下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7fc2e08d56eeec319f87885334194991.png)

勾选上 `Lombok` 和 `Spring Web` 依赖，点击 `create` 创建模块：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36e85c6becf62ef14627a29d45c4bf68.png)

创建完成后，删除掉无用的一些目录和文件，删完后，大致如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0733de09ad29f84f708ed2f1c3662310.png)

删除完成后，在父项目的 `pom.xml` 中添加该子模块，以及添加 `spring-boot-maven-plugin` ：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9067b144df13f27fc2f45f814cf369c0.png)

```xml
<!-- 子模块管理 -->  
<modules>  
    <!-- 入口模块 -->  
    <module>blog-web</module>  
</modules>

<build>          
<!-- 统一插件管理 -->  
    <pluginManagement>  
        <plugins>  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
                <configuration>                    <excludes>  
                        <exclude>  
                            <groupId>org.projectlombok</groupId>  
                            <artifactId>lombok</artifactId>  
                        </exclude>  
                    </excludes>  
                </configuration>  
            </plugin>  
        </plugins>  
    </pluginManagement>  
</build>
```

添加完成后，开始编辑 `weblog-web` 模块中的 `pom.xml`，只保留了一些需要的配置，最终配置内容如下:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <!-- 指定父项目为 blog-backend -->    <parent>  
        <groupId>com.dakkk</groupId>  
        <artifactId>blog-backend</artifactId>  
        <version>${revision}</version>  
    </parent>  
  
    <groupId>com.dakkk</groupId>  
    <artifactId>blog-web</artifactId>  
    <name>blog-web</name>  
    <description>blog-web (入口项目，负责博客前台展示相关功能，打包也放在这个模块负责)</description>  
  
    <dependencies>        
	    <!-- Web 依赖 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <!-- 免写冗余的 Java 样板式代码 -->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>  
        <!-- 单元测试 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
    </dependencies>  
  
    <build>       
     <plugins>  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>  
        </plugins>  
    </build>  
</project>
```

### 3.4 可能会遇到的问题

**依赖爆红解决方案**
- 若 `pom.xml` 文件中的依赖出现**爆红**等情况，通过点击右侧栏 _Reload_ 图标，重新加载 Maven 项目来解决：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b5a534c48cc0f9a6fcfb20a538e4bd26.png)
---

**控制台出现警告解决方案**
- 在执行 Maven 命令时，若使用 IDEA 默认的 Maven 配置，可能会导致后面打包控制台出现如下警告：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6be9fe52867925d8992ed42f4abf8531.png)
- 可在点击 _File -> Settings_ 找到 Maven 选项，默认是使用 IDEA 自带的 Maven 版本, 这里将 _Maven home path_ 设置为前面小节中，我们手动安装好的 Maven 路径，再次执行 Maven 命令，即警告信息消失了：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/163fb293311db35b7ab2bd9c2fa9af13.png)
---

**执行 mvn clean package 命令报错**
- 执行 `mvn clean package` 等相关命令时, 控制台提示如下图所示的错误，这个问题好多人反馈都有犯：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/918ff3ff05a58a34916a2b7f886859bb.png)
- 问题原因是，在子模块中的 `<parent></parent>` 节点下添加了 `<relativePath/>` ，注意，它只需要在父级项目中添加一次即可，子模块中是无需添加这个的，一定要去掉：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea3d46f9f2aa09593638c831e5e1b4fc.png)
### 3.5 创建 Admin 管理后台功能模块

再次新建一个负责 Admin 管理后台功能的子模块，命名为 `blog-module-admin` ，此模块用于统一放置和 Admin 管理后台相关的功能，依赖仅需勾选 `Lombok` 即可，后面需要什么再添加：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b225ef370c3adba01998dbbb36669c0.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5bfbb2308993f2a26b5797a40b23847a.png)

创建成功后，在父项目的 `pom.xml` 文件中添加该子模块：
```xml
<!-- 子模块管理 -->  
<modules>  
    <!-- 入口模块 -->  
    <module>blog-web</module>  
    <!-- 管理后台 -->  
    <module>blog-module-admin</module>  
</modules>
```

然后，和前面一样，删除掉哪些无用的文件夹、文件，

📕另外，还需要将 `resource` 目录下的配置文件，和 `Application` 启动类删除掉。配置文件统一放在 `blog-web` 入口模块中来管理：![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6f1d767897af856a07a15860cf67e1e9.png)
最后，再整理一下此模块的 `pom.xml` , 内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <!-- 指定父项目为 weblog-springboot --><parent>  
        <groupId>com.dakkk</groupId>  
        <artifactId>blog-backend</artifactId>  
        <version>${revision}</version>  
    </parent>  
  
    <groupId>com.dakkk</groupId>  
    <artifactId>blog-module-admin</artifactId>  
    <name>blog-module-admin</name>  
    <description>blog-admin (负责管理后台相关功能)</description>  
  
    <dependencies>        
	    <!-- 免写冗余的 Java 样板式代码 -->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>  
  
        <!-- 单元测试 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
    </dependencies>  
  
</project>
```

### 3.6 创建 common 通用功能子模块

依葫芦画猫，按照上面的步骤，再次创建 `blog-module-common` 子模块，此模块专门用于存放一些通用的功能，如接口的日志切面、全局异常管理等等。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/003fd964f38a95934077231eaabecbc3.png)

创建成功后，在父项目的 `pom.xml` 文件中添加该子模块：
```xml
<!-- 子模块管理 -->  
<modules>  
    <!-- 入口模块 -->  
    <module>blog-web</module>  
    <!-- 管理后台 -->  
    <module>blog-module-admin</module>  
    <!-- 通用模块 -->  
    <module>blog-module-common</module>  
</modules>
```

和 `weblog-mudule-admin` 子模块一样，再删除掉无用的文件，最终其 `pom.xml` 内容如下:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>        
	    <groupId>com.dakkk</groupId>  
        <artifactId>blog-backend</artifactId>  
        <version>${revision}</version>  
    </parent>  
  
    <groupId>com.dakkk</groupId>  
    <artifactId>blog-module-common</artifactId>  
    <name>blog-module-common</name>  
    <description>blog-module-common (此模块用于存放一些通用的功能)</description>  
  
    <dependencies>        
    <!-- 免写冗余的 Java 样板式代码 -->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>  
  
        <!-- 常用工具库 -->  
        <dependency>  
            <groupId>com.google.guava</groupId>  
            <artifactId>guava</artifactId>  
        </dependency>  
  
        <!-- 单元测试 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
    </dependencies>  
  
</project>
```

问题：如果guava版本为31.1-jre下载不了，直接去maven仓库下载，然后拷贝到本地的maven仓库中

### 3.7 父项目统一版本管理

在父项目的 `pom.xml` 中，将这几个子模块统一声明一下，另外添加相关常用的工具包，如 `commons-lang3` 、`guava` ：
```xml
<!-- 版本号统一管理 -->  
<properties>  
	
	...
	
    <!-- Maven 相关 -->  
    <maven.compiler.source>${java.version}</maven.compiler.source>  
    <maven.compiler.target>${java.version}</maven.compiler.target>  
  
    <!-- 依赖包版本 -->  
    <lombok.version>1.18.28</lombok.version>  
    <guava.version>31.1-jre</guava.version>  
    <commons-lang3.version>3.12.0</commons-lang3.version>  
  
</properties>


<!-- 统一依赖管理 -->  
<dependencyManagement>  
    <dependencies>  
        <dependency>  
            <groupId>com.dakkk</groupId>  
            <artifactId>blog-module-admin</artifactId>  
            <version>${revision}</version>  
        </dependency>  
  
        <dependency>            
        <groupId>com.dakkk</groupId>  
            <artifactId>blog-module-common</artifactId>  
            <version>${revision}</version>  
        </dependency>  
  
        <!-- 常用工具库 -->  
        <dependency>  
            <groupId>com.google.guava</groupId>  
            <artifactId>guava</artifactId>  
            <version>${guava.version}</version>  
        </dependency>  
  
        <dependency>            <groupId>org.apache.commons</groupId>  
            <artifactId>commons-lang3</artifactId>  
            <version>${commons-lang3.version}</version>  
        </dependency>  
    </dependencies>  
</dependencyManagement>
```

### 3.8 子模块之间的依赖关系

这几个子模块之间，互相还存在依赖关系，我们也需要引入一下。如入口模块 `weblog-web` 依赖于 `weblog-module-admin` 和 `weblog-module-common`，在其 `pom.xml` 添加如下：
```xml
<dependencies>        <!-- Web 依赖 -->  
    <dependency>  
        <groupId>com.dakkk</groupId>  
        <artifactId>blog-module-common</artifactId>  
    </dependency>  
  
    <dependency>        
	    <groupId>com.dakkk</groupId>  
        <artifactId>blog-module-admin</artifactId>  
    </dependency>  

	...

</dependencies>
```

以及 `weblog-module-admin` 依赖于 `weblog-module-common`，在其 `pom.xml` 添加如下：
```xml
<dependencies>       
    <dependency>  
        <groupId>com.dakkk</groupId>  
        <artifactId>blog-module-common</artifactId>  
    </dependency>  

	...

</dependencies>
```
### 3.9 测试！！

在 IDEA 中, 对 `blog-backedn` 父项目执行 Maven 的 `clean package` 打包命令，看看是否能够正常给项目打包：

⚠️ 注意：企业开发中，一般都比较追求敏捷开发，可能不会写太多的单元测试。本实战项目同样为了追求进度，不会编写单元测试，所以多模块的单元测试并未配置完成，这里，需要如下图所示，将_跳过单元测试勾选上_，否则，你打包的时候会报错。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13e18a69881491234c27928209fb01c4.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4a84bf67923a3f6f5c6cc7cdb7f8134c.png)

如果没有问题，控制台会提示 _BUILD SUCCESS_ 表示构建成功：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a0566b473bddea190886c22a50f1e332.png)

打包成功后的 `Jar` 包，可以在项目目录的 `weblog-web` 入口模块中的 `/target` 目录下找到：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f758062ce4a09893c764096a3782a822.png)

## 4 启动Springboot项目

进入 `weblog-web` 入口项目，找到 `WeblogWebApplication` 启动类，**点击启动图标**，运行此项目：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6453e0d2b70bfe95cd70756a3a54fc4c.png)

若控制台能够正确打印如下日志，则表示 Spring Boot 启动成功：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d60f83ce0a342b3c45452a99022bb62.png)

至此，搭建 Spring Boot 多模块工程就成功啦，是不是还挺简单的。