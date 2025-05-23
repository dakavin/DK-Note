
在进入正文之前，我们先来看看上小节中，更新接口的业务层存在的问题，如下图所示：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a8639b737f5177b7a735c80f0cda4439.png)

_你应该也发现了，需要手动 `copy` 的属性字段值太多了 ！_ 字段值一多，就会导致开发效率比较低，同时也导致了代码也不够美观。本小节中，小哈就带着大家通过 `Mapstruct` 库来美化一下此处的代码。

## 1 什么是 MapStruct？

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/56b00e0a62d16b2202cbf654854dd5c1.png)


MapStruct 是一个用于简化 Java Bean 映射代码的代码生成器。它可以自动生成在不同 Java 对象之间进行映射的代码，而无需手动编写复杂的转换逻辑。MapStruct 使得在 Java 应用程序中进行对象映射变得更加简单、高效、可维护。

## 2 为什么要使用 MapStruct？

- **简化映射过程：** 传统的 Java 对象映射需要编写大量的重复、繁琐的代码。MapStruct 自动生成这些映射代码，节省了开发人员的时间和精力。
    
- **类型安全性：** ==MapStruct 是基于编译时的代码生成==，因此提供了类型安全性。在编译时，它会检查源对象和目标对象的类型，避免了运行时的类型错误。
    
- **高性能：** 由于 MapStruct 生成的代码是高度优化的，所以性能非常好。相比手动编写的映射代码，MapStruct 生成的代码更加高效。
    
- **可维护性：** 当数据模型发生变化时，手动更新映射代码可能会非常麻烦。MapStruct 能够根据最新的数据模型自动生成新的映射代码，确保映射逻辑与数据模型保持同步。
    
- **灵活性：** MapStruct 提供了丰富的配置选项和扩展点，可以满足各种复杂映射场景的需求。你可以定制生成的代码，以适应特定的业务逻辑。
    

总之，MapStruct 是一个强大的工具，可以帮助开发人员简化 Java 对象之间的映射过程，提高代码质量和开发效率。

## 3 添加依赖

接下来，我们就来为 `weblog` 后端服务整合 `Mapstruct` 。编辑父项目的 `pom.xml` 文件，添加 `Mapstruct` 版本管理，代码如下：

```xml
    <!-- 版本号统一管理 -->
    <properties>
        // 省略...
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>
    
    <!-- 统一依赖管理 -->
    <dependencyManagement>
        <dependencies>
			// 省略...

			<!-- Mapstruct 属性映射 -->  
			<dependency>  
			    <groupId>org.mapstruct</groupId>  
			    <artifactId>mapstruct</artifactId>  
			    <version>${mapstruct.version}</version>  
			</dependency>  
			<dependency>  
			    <groupId>org.mapstruct</groupId>  
			    <artifactId>mapstruct-processor</artifactId>  
			    <version>${mapstruct.version}</version>  
			</dependency>

        </dependencies>
    </dependencyManagement>
    
       <build>
        <!-- 统一插件管理 -->
        <pluginManagement>
            <plugins>
				// 省略...

				<plugin>  
				    <groupId>org.apache.maven.plugins</groupId>  
				    <artifactId>maven-compiler-plugin</artifactId>  
				    <configuration>        
					    <source>${java.version}</source> <!-- 根据你的 JDK 版本进行调整 -->  
				        <target>${java.version}</target> <!-- 根据你的 JDK 版本进行调整 -->  
				        <annotationProcessorPaths>  
				            <path>  
				                <groupId>org.projectlombok</groupId>  
				                <artifactId>lombok</artifactId>  
				                <version>${lombok.version}</version>  
				            </path>  
				            <path>                
					            <groupId>org.mapstruct</groupId>  
				                <artifactId>mapstruct-processor</artifactId>  
				                <version>${mapstruct.version}</version> <!-- 使用时请检查最新版本 -->  
				            </path>  
				            <path>                
					            <groupId>org.projectlombok</groupId>  
				                <artifactId>lombok-mapstruct-binding</artifactId>  
				                <version>0.2.0</version>  
				            </path>  
				        </annotationProcessorPaths>  
				    </configuration>  
				</plugin>
            </plugins>
        </pluginManagement>
    </build>
```

然后，在 `weblog-module-admin` 中的 `pom.xml` 文件中，添加 `mapstruct` 依赖：

```
<!-- Mapstruct 属性映射 -->  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct-processor</artifactId>  
</dependency>
```

最后，在 `weblog-web` 入口模块中，添加编译插件：

```
    <build>
        <plugins>
			// 省略...
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

## 4 IDEA 配置 Mapstruct

添加好了 MapStruct 依赖后，已经可以确保项目中能够正常使用 MapStruct 了。但是，为了在开发过程中获得更好的体验，我们可以为 IDE 进行一些配置，从而支持 MapStruct 的自动代码生成、代码提示等功能。

### 4.1 启用注解处理器

1. 打开 IntelliJ IDEA，并加载你的项目。
2. 打开 `File -> Settings (或 Preferences)`。
3. 在左侧导航栏中选择 `Build, Execution, Deployment -> Compiler -> Annotation Processors`。
4. 勾选 `Enable annotation processing`。
5. 在 `Store generated sources relative to` 下拉列表中选择 `Module content root`。
6. 点击右下角 `Apply` 按钮应用设置，然后，点击 `ok` 按钮关闭弹框。

图示如下：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a97915f6eb7b68f98dfff15bc992eb08.png)


### 4.2 添加 MapStruct 插件

虽然这不是必需的，但 MapStruct 插件可以为你提供一些很有用的功能，例如代码提示和自动补全。

1. 打开 `File -> Settings (或 Preferences)`。
2. 在左侧选择 `Plugins`。
3. 在市场 (Marketplace) 中搜索 “MapStruct” 并安装它。
4. 重启 IntelliJ IDEA。

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/9c70a5fb1bd47e6680b5d93a046db23c.png)


## 5 添加 convert 接口

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/b2df8567cea73fbbb6cd95655c5ab43b.png)


相关配置完成后，我们在 `weblog-module-admin` 模块中，新建一个 `/convert` 包，并创建一个 `BlogSettingsConvert` 转换接口，代码如下：

```java
@Mapper
public interface BlogSettingsConvert {
    /**
     * 初始化 convert 实例
     */
    BlogSettingsConvert INSTANCE = Mappers.getMapper(BlogSettingsConvert.class);

    /**
     * 将 VO 转化为 DO
     * @param bean
     * @return
     */
    BlogSettingsDO convertVO2DO(UpdateBlogSettingsReqVO bean);

}
```

> 注意，接口类上添加的 `@Mapper` 注解，导入自 `org.mapstruct.Mapper`, 别搞错了哦。

上述代码中，我们首先了创建了一个 `INSTANCE` 接口实例，该实例可以用于在不同类型（例如 VO 到 DO 或者 DO 到 DTO）之间进行转换。然后，定义了 `convertVO2DO()` 接口方法，用于将类型为 `UpdateBlogSettingsReqVO` 的对象转换为类型为 `BlogSettingsDO` 对象。**在 MapStruct 中，您不需要手动编写转换逻辑，MapStruct 会根据这个方法的签名自动生成转换的代码。**

## 6 使用 Mapstruct 来转换

`convert` 接口添加完毕后，重构一下上小节中的 `AdminBlogSettingsServiceImpl` 业务层中 `VO` 转换 `DO` 逻辑代码，内容如下：

```java
@Service
public class AdminBlogSettingsServiceImpl extends ServiceImpl<BlogSettingsMapper, BlogSettingsDO> implements AdminBlogSettingsService {

    @Override
    public Response updateBlogSettings(UpdateBlogSettingsReqVO updateBlogSettingsReqVO) {
        // VO 转 DO
        BlogSettingsDO blogSettingsDO = BlogSettingsConvert.INSTANCE.convertVO2DO(updateBlogSettingsReqVO);
        blogSettingsDO.setId(1L);

        // 保存或更新（当数据库中存在 ID 为 1 的记录时，则执行更新操作，否则执行插入操作）
        saveOrUpdate(blogSettingsDO);
        return Response.success();
    }
}
```

上述代码中，我们直接通过 `BlogSettingsConvert.INSTANCE.convertVO2DO()` 方法，将 `VO` 类转换为了 `DO` 类，仅需要对 `VO` 中没有值的字段，如 `ID` 字段设置一下值即可。代码相较于之前，变得精简了很多，有木有~

## 7 重新测试一下

重启项目，我们再来测试一下该接口，看看功能是否依然正常

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d2f922a343cdd9d04609fcccc54cbf8a.png)


可以看到，修改成功了，这表明字段值也映射成功了。
## 8 结语

本小节中，我们在 `Spring Boot` 工程中整合了 `Mapstruct` 对象映射工具，之所以选择它，因为它是目前 Java 中，性能最强的对象映射工具，它的映射代码都是在代码编译期间自动生成的，相比于其他对象映射工具，通过反射来完成，自然效率高出太多。然后，我们自定义了转换 `convert` 接口，将 `VO` 类转换为了 `DO` 类，最后，重构了上小节的对象转换代码，让代码变得更加清晰，增加了可维护性。