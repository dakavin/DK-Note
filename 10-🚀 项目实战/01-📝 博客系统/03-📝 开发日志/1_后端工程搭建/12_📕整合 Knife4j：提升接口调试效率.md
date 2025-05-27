---
文章标题: "[[12_📕整合 Knife4j：提升接口调试效率]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了如何整合Knife4j，一个增强版Swagger-UI，以提升接口调试效率。它涵盖了依赖配置、Spring Boot配置类设置、利用Swagger注解优化文档，以及通过Spring Profile控制环境可见性和实现API分组功能，旨在简化后端接口调试流程。
文章标签:
- "Knife4j"
- "Swagger"
- "API文档"
- "接口调试"
- "Spring Boot"
- "后端开发"
- "Spring Profile"
- "API管理"
相关文章:
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_Java学习路线]]"
- "[[11_📕全局异常处理器+参数校验（最佳实践）]]"
- "[[3_Spring Boot 项目多环境配置]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/1_后端工程搭建/12_📕整合 Knife4j：提升接口调试效率.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-17 00:48:37
修改时间: 2025-05-27 00:52:55
---

## 1 Knife4j是什么？

**Swagger**是一个开源的、用于编写API文档的框架。它可以帮助我们通过Swagger UI对API文档进行在线展示，从而实现API文档的可视化管理。

然而，Swagger在UI界面上的表现并不是很出色。因此，为了增强Swagger的UI交互体验，我们可以使用Knife4j对其进行美化和强化。

**Knife4j是Swagger-UI的增强版**，它是在Swagger-UI的基础上进行了改进和优化，提供了更加完善的交互体验和更加美观的UI设计。同时，它也提供了更多的扩展功能，例如在线调试和多语言支持等。
## 2 Knife4j主要功能

- **美观的 UI**：相比于原生 Swagger UI，Knife4j 提供了更加人性化和美观的界面设计。
- **丰富的文档交互功能**：支持在线调试、请求参数动态输入、接口排序等。
- **个性化配置**：可自定义 API 文档的界面风格，实现文档界面的个性化展示。
## 3 为什么要用它？

小伙伴们可能会疑惑，咱这不是一个人就把前后端包办吗？还搞啥 API 文档，是不是多此一举了。其实，我们主要想用的是它的在线调试功能，在前面小节中，小哈在调试接口这块，都是通过 Postman 来进行的，每当测试一个新的接口时，就不得不手动填写请求路径，组装 JSON 入参格式，其实还是比较繁琐的。

有了 Knife4j，因为它支持在线调试，这块的时间就可以省下来了。本小节中，我们就来看看，如何通过 Knife4j 提升接口调试效率。
## 4 整合Knife4j
### 4.1 添加依赖

在父项目 `weblog-springboot` 中的 `pom.xml` 文件中，添加 Knife4j 依赖版本号：
```xml
<knife4j.version>4.3.0</knife4j.version>

<!-- knife4j（API 文档工具） -->  
<dependency>  
    <groupId>com.github.xiaoymin</groupId>  
    <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>  
    <version>${knife4j.version}</version>  
</dependency>
```

因为 `admin` 后台管理模块和博客前台模块都需要调试接口，所以，我们需要在 `weblog-web` 和 `weblog-module-admin` 两个模块中，都需要引入该依赖：
```xml
<!-- knife4j -->  
<dependency>  
    <groupId>com.github.xiaoymin</groupId>  
    <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>  
</dependency>
```
### 4.2 添加配置类

在 `weblog-web` 模块中添加包 `config` , 用于统一放置配置类。在该包下新建名为 `Knife4jConfig` 配置类

配置类完整代码如下：
```java
@Configuration  
@EnableSwagger2WebMvc  
public class Knife4jConfig {  
    @Bean  
    public Docket createApiDoc(){  
        Docket docket = new Docket(DocumentationType.SWAGGER_2)  
                .apiInfo(buildApiInfo())                .groupName("Web 前台接口")  
                .select()                .apis(RequestHandlerSelectors.basePackage("com.dakkk.blog.web.controller"))  
                .paths(PathSelectors.any())                .build();  
        return docket;  
    }  
  
    /**  
     * 构建 API 信息  
     * @return 返回API信息  
     */  
    private ApiInfo buildApiInfo(){  
        return new ApiInfoBuilder()  
                // 标题  
                .title("DK blog 博客前台接口文档")  
                // 描述  
                .description("DK blog 是一款由 Spring Boot + Vue 3.2 + Vite 4.3 开发的前后端分离博客。")  
                // API 服务条款  
                . termsOfServiceUrl("https://gitee.com/Dakkk_mike")  
                //联系人  
                . contact(new Contact("dakkk","https://gitee.com/Dakkk_mike","mikeylay@126.com"))  
                // 版本号  
                . version("0.0.1")  
                . build();  
    }  
}
```

上述代码中，通过 `@Configuration` 指定了 `Knife4jConfig` 这个类为配置类，同时添加了 `@EnableSwagger2WebMvc` 注解，该注解作用是启用 Swagger2，定义了一个 `Docket` Bean 包含了 Swagger 相关的配置信息。
### 4.3 看看效果

完成上面配置后，重启项目，若控制台出现如下红线标注的日志，则表示 `Knife4j` 配置完成了：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34d29276aec91912299bf84ca16dabc3.png)

浏览器访问路径 `http://localhost:8080/doc.html` , 就可以看到 `api` 管理界面了，如下图所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/688860ff2f0d7d4d942537fb535a2013.png)

大概解释一下 UI 界面的功能点：

- **主页**： 文档整体信息一览，包括文档标题、简介、作者、接口统计信息等，也就是 `Knife4jConfig` 类中 `buildApiInfo()` 方法配置的相关信息；

- **分组**：实际企业级项目中，可能会分为多个模块、或者很多微服务，分组用于对接口进行分类。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b79d8e0220d98d3b989bdcb66fe05419.png)
- **Swagger Models** : 服务端接口相关的领域模型；

- **文档管理**： 这个模块里面可以设置全局参数，比如每次请求在 header 头中统一添加某个参数等；导出 `API` 文档到本地，以及个性化设置，有兴趣的小伙伴可以自己点进去体验一波：

- **项目接口**：左侧导航栏中会列举出本项目中的所有 `controller` 类，以及其中的接口。因为当前 `weblog` 项目只有一个供测试的 `TestController`，效果如下:
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5aef94cd07bb0781c7e05b4411121fbc.png)

可以看到 Knife4j 自动根据我们的 `controller` 代码，将所有的接口列出来了，包括请求示例、请求参数等。
### 4.4 如何调试接口

假设说，你刚刚开发好了一个新的接口，需要本地调试一下接口是否正常。只需点击左侧 _调试_ 选项，然后填写字段值，在点击发送按钮即可：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/98e3da188081ca74709416450f60a6a7.png)

怎么样，是不是相较于使用 Postman, 调试接口的效率大大地提升了~
### 4.5 给controller添加Swagger相关注解

痛点：目前在左侧显示的模块类名，以及接口名称都是代码中本身的英文，也许你当前调试中，能够清晰的知道哪个 `controller` 类、哪个接口都是干啥的，但是时间一长，接口多了起来，你就会一脸懵逼了，挨，这个 `controller` 是管理那块的功能的？

**为了解决这个问题，Swagger 提供了相关的注解，可以标识模块名称，以及接口名称**，并方便的展示在管理界面中。添加方式如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4d189c933fd4de06639805e94f9a3087.png)

- `@Api` : 此注解作用于 `controller` 之上，用于描述相关职责；
- `@ApiOperation` : 此注解作用于接口上，用于描述接口干啥的；

添加完上面两个注解后，管理界面的效果图如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d0ac8d656d0e31a0f57eea3716af2cac.png)


明显感觉到管理上更加清晰明了了。

除上面两个注解外，我们还可以给入参对象添加 Swagger 相关注解，如下所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c70c907822906dc393bcd9391ba3ef2.png)

- `@ApiModel` : 此注解作用于实体类上，用于描述类；
- `@ApiModelProperty` : 此注解作用于字段上，用于描述字段；

最终效果如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04c2b1bd45d60a2ca2593b57bb941525.png)
每个字段对应的说明也有了。
## 5 生产环境如何屏蔽Knife4j

很多时候，我们不想项目在生产环境中暴露出所有接口信息，只想在开发/测试环境中联调使用，那么要如何屏蔽该功能呢?
### 5.1 Spring Boot Profile特性

Profile 是 Spring Boot 中的一项特性，允许你在不同环境中使用不同的配置。这种机制使得我们可以轻松地在不同环境（如开发、测试和生产环境）中使用不同的配置参数。
### 5.2 @Profile注解

你可以在配置类上添加 `@Profile` 注解，来控制 Knife4j 是否生效 。只有当指定的 Profile 处于激活状态时，该配置类才会被创建和被使用。代码如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/722a5a440b45cf8c5c1ff4c702ffef02.png)

## 6 分组功能

前面小节中说到，`weblog` 项目接口分为前台和 Admin 后台，所以，除了在 `weblog-web` 模块中配置 Knife4j 外，还需要在 `web-module-admin` 也配置一份，并使用 Knife4j 分组功能将各自的接口隔离开来。
### 6.1 添加依赖、配置类

`weblog-module-admin` 模块和前面的步骤一样，在 `pom.xml` 添加依赖

建包 `config`, 并创建 `Knife4jAdminConfig` 配置类，将刚刚在web模块下创建的配置及config包复制过去，然后修改

**修改注意事项：**
注意`类名`，和 `@Bean` 的名称不能和 `weblog-web` 中的一样，否则会冲突。然后，改写分组名称，以及包扫描路径，还有 API 相关信息。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78859e69379bad72b696706248485a32.png)



**可以提前在admin目录下，创建controller包**

完成上述工作后，重启项目，再次访问 `API` 管理界面，即可看到分组勾选栏中，有 _前台接口_ 和 _Admin 后台接口_ 了，后面查找相关接口，只需先确定是属于前台，还是后台，然后再对应的去查找相关接口，也方便很多：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e3b951299327396b30fd97cb80647cd7.png)
## 8 结语

本小节中，带着大家了解了什么是 knife4j， 它的主要功能，以及相较于 Swagger 它的优势在哪里？最后，在 `blog` 项目中整合了 knife4j ，可以在后续的功能开发中，很方便的进行本地接口的调试，提升调试效率。