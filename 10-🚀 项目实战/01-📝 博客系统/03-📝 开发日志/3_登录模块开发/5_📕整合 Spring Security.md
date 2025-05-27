---
文章标题: "[[5_📕整合 Spring Security]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了Spring Security框架，阐述其重要性。它指导读者在一个多模块Spring Boot项目中初步整合Spring Security，包括模块配置、依赖添加，并演示了如何通过`WebSecurityConfig`配置基于表单和HTTP Basic的基础认证和URL权限控制。
文章标签:
- "Spring Security"
- "Spring Boot"
- "Web安全"
- "认证"
- "授权"
- "Maven"
- "多模块项目"
相关文章:
- "[[0_参考文章索引]]"
- "[[2_Maven和Springboot下多环境的区别]]"
- "[[7_Springboot]]"
- "[[1_搭建多模块工程（Spring Initializr）]]"
- "[[0_导读]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/3_登录模块开发/5_📕整合 Spring Security.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-22 22:21:31
修改时间: 2025-05-27 00:54:47
---

在现代的 Web 应用程序中，安全性是系统至关重要的一环。Spring Security 是一个功能强大的安全框架，能够帮助您在应用程序中实现认证（Authentication）和授权（Authorization）功能。本小节中，将带着大家在 `blog` 项目中初步整合 Spring Security 框架。

## 1 什么是 Spring Security？

Spring Security 是一个广泛应用于 Java 应用程序的安全框架，旨在保护应用程序免受潜在的安全威胁和攻击。作为 Spring 框架的一部分，Spring Security 提供了强大的功能，帮助开发人员实现身份认证、授权、会话管理以及其他与安全相关的任务。

## 2 为什么要使用 Spring Security？

- **应用程序保护：** 在当今互联网时代，应用程序安全性至关重要。Spring Security 提供了一系列的安全特性，帮助您保护应用程序免受恶意攻击，确保数据和功能不会被未经授权的用户访问。
- **一站式解决方案：** Spring Security 提供了一个集中的框架来处理各种安全性问题。从用户认证、授权到会话管理，Spring Security 提供了一个一站式的解决方案，使开发人员能够更容易地处理安全问题。
- **多样的认证和授权方式：** Spring Security 允许您以多种方式进行用户认证，包括基本认证、表单认证、OAuth 等。此外，您还可以定义精细的授权规则，确保只有经过授权的用户才能访问特定资源。
- **与 Spring 集成：** Spring Security 可与其他 Spring 项目无缝集成，如 Spring Boot、Spring MVC 等。这使得您可以轻松地将安全性集成到现有的 Spring 应用程序中。
- **高度可定制：** Spring Security 允许您根据应用程序的特定需求进行定制。您可以创建自定义的认证提供者、访问决策器、过滤器等，以适应不同的安全性要求。
- **丰富的社区和文档：** Spring Security 拥有活跃的社区和丰富的文档。您可以在社区中找到许多教程、示例代码和解决方案，以帮助您解决各种安全性问题。
- **符合标准：** Spring Security 符合各种安全性标准和最佳实践，确保您的应用程序满足行业和法规的要求。

无论是保护敏感数据、防范潜在的攻击还是确保只有授权用户能够访问特定功能，Spring Security 都是一个可靠的解决方案。通过提供一系列功能强大的工具，Spring Security 帮助您构建安全可靠的应用程序，确保用户数据和应用程序功能得到充分的保护。

## 3 新建 weblog-module-jwt 子模块

在整合 Spring Security 框架之前，我们需要新建一个 `blog-module-jwt` 子模块，它主要用于放置 `jwt` 相关的功能代码。

在父项目 `weblog-springboot` 上右键，新建 `weblog-module-jwt` 子模块，忘记如何创建子模块的小伙伴，可翻阅前面小节[1_搭建多模块工程（Spring Initializr）](../1_后端工程搭建/1_搭建多模块工程（Spring%20Initializr）.md)

创建成功后，删除一些无用的文件、文件夹

修改 `pom.xml` 文件，内容如下：
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
    <artifactId>blog-module-jwt</artifactId>
    <name>blog-module-jwt</name>
    <description>blog-module-jwt (JWT 模块，管理用户认证、鉴权)</description>

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

另外，还需在父项目 `weblog-springboot` 的 `pom.xml` 文件中添加 `jwt` 子模块 :
```xml
<!-- 子模块管理 -->
<modules>
	<!-- JWT 模块 -->
	<module>blog-module-jwt</module>
</modules>

<!-- 统一依赖管理 -->
<dependencyManagement>
	<dependencies>
		<dependency>
				<groupId>com.dakkk</groupId>
				<artifactId>blog-module-jwt</artifactId>
				<version>${revision}</version>
		</dependency>
	</dependencies>
</dependencyManagement>
```

最后，在 `weblog-module-admin` 模块中，引入 `weblog-module-jwt` 模块，因为认证、鉴权功能只有 `admin` 后台需要：
```xml
<dependency>
	<groupId>com.dakkk</groupId>
	<artifactId>blog-module-jwt</artifactId>
</dependency>
```

## 4 开始整合 Spring Security

### 4.1 添加 Spring Security 依赖

分别在 `weblog-module-admin` 和 `weblog-module-jwt` 子模块中，添加 `security` 依赖，因为这两个模块都会使用到 `spring security`框架中的功能：
```xml
<!-- Spring Security -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 4.2 自定义 Spring Security 配置

要更好地控制 Spring Security 的行为，你可以创建一个自定义的 `SecurityConfig` 类，继承自 `WebSecurityConfigurerAdapter`。通过覆盖方法，您可以配置认证、授权规则、自定义登录页面、注销等。

我们在 `weblog-module-admin` 模块中的 `config` 包下，创建一个 `WebSecurityConfig` 配置类，代码如下：
```java
@Configuration  
@EnableWebSecurity  
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {  
    @Override  
    protected void configure(HttpSecurity http) throws Exception {  
        http.authorizeHttpRequests()  
                // 认证所有以 /admin  为前缀的url资源  
                .mvcMatchers("admin/**").authenticated()  
                // 其他就放行，无需认证  
                .anyRequest().permitAll().and()  
                // 使用表单登录的  
                .formLogin().and()  
                // 使用 HTTP Basic 认证  
                .httpBasic();  
    }  
}
```

上述代码中，我们重写了 `configure()` 方法，在该方法中，指定了对所有以 `/admin` 为前缀的请求需要进行安全认证，另外，对其他请求均放行，无需认证，因为博客前台的页面是任何人都可以访问的。最后，通过 `formLogin()` 方法指定使用表单登录，并使用 使用 HTTP Basic 认证。

> _什么是 HTTP Basic 认证？_
> 
> HTTP Basic 认证是 Spring Security 中的一种认证方式，它基于 HTTP 基本认证协议。这种认证方式是一种简单的认证方式，适用于简单的应用场景。当客户端发送请求时，会将用户名和密码使用 Base64 编码的形式放在请求头中，服务器接收到请求后会解码并验证这些信息。


### 4.3 自定义登录用户名、密码

通过编辑 `applicaiton-dev.yml` 文件，添加如下配置，可以自定义测试环境下的登录用户名、密码：

```
spring:
  // 省略...
  security:
    user:
      name: admin # 登录用户名
      password: 123456 # 登录密码
```

## 5 试试效果

自定义好了 Spring Security 的配置后，我们将之前的 `/test` 接口改为 `/admin/test` , 看看 Spring security 框架是否能够正确拦截，并跳转表单认证。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e4ea4015a585273d2305025c5a92161.png)


可以看到，正常被拦截了，并跳转到了登录表单页。这里我们输入之前自定义的用户名、密码，看看认证成功后，能够正常访问接口：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/637a177a27f55ecdd1637648a9a9e685.png)

提示的是全局异常返回的 JSON 提示信息。这是正常的，应为浏览器是通过 `GET` 请求访问的该接口，而我们定义的是 `POST` 请求:
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b048a9913fb90a49996f2caf10d891e1.png)


无伤大雅，但是说明了认证成功后，接口的确能被正常访问。测试成功。

## 结语

本小节中，我们初步整合了 Spring Security 框架，并自定义了表单认证来对 `/admin/**` 路径在未登录的情况下，进行拦截。但是，在实际的前后端分离项目中，往往不是这样玩的，通常的做法是通过 Spring Security + JWT（JSON Web Token）来实现用户的安全身份验证和授权机制。我们将在后续小节中，一步一步实现它。