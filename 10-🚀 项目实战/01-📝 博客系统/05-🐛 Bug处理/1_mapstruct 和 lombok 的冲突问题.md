
## 前言

我们经常会使用到mapstruct来进行do和vo之间的转换

DO代码如下：
```java
@TableName(value ="t_article")  
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Builder  
public class ArticleDO implements Serializable {  
    /**  
     * 文章id  
     */    @TableId(type = IdType.AUTO)  
    private Long id;  
  
    /**  
     * 文章标题  
     */  
    private String title;  
  
    /**  
     * 文章封面  
     */  
    private String cover;  
  
    /**  
     * 文章摘要  
     */  
    private String summary;  
  
    /**  
     * 创建时间  
     */  
    private LocalDateTime createTime;  
  
    /**  
     * 最后一次更新时间  
     */  
    private LocalDateTime updateTime;  
  
    /**  
     * 删除标志位：0：未删除 1：已删除  
     */  
    private Integer isDeleted;  
  
    /**  
     * 被阅读次数  
     */  
    private Integer readNum;  
  
    @TableField(exist = false)  
    private static final long serialVersionUID = 1L;  
}
```

很显然，我们使用了lombok提供的一些注解，免去了一些set和get的方法书写

VO代码如下：
```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Builder  
public class FindArticleDetailRspVO {  
    /**  
     * 文章 ID  
     */    private Long id;  
  
    /**  
     * 文章标题  
     */  
    private String title;  
  
    /**  
     * 文章封面  
     */  
    private String cover;  
  
    /**  
     * 文章内容  
     */  
    private String content;  
  
    /**  
     * 分类 ID  
     */    private Long categoryId;  
  
    /**  
     * 标签 ID 集合  
     */  
    private List<Long> tagIds;  
  
    /**  
     * 摘要  
     */  
    private String summary;  
}
```

使用mapstruct代码如下：
```java
@Mapper  
public interface ArticleDetailConvert {  
    /**  
     * 初始化 convert 实例(获取该类自动生成的实现类的实例)  
     */    
     ArticleDetailConvert INSTANCE = Mappers.getMapper(ArticleDetailConvert.class);  
  
    /**  
     * 将 DO 转 VO  
     */    
     FindArticleDetailRspVO convertDO2VO(ArticleDO bean);  
}
```

注意点1：@Mapper注解是 mapstruct依赖提供的

可以看到我们有一个将DO转VO的方法

## 问题

如果我们只是单纯的在项目中 添加了 lombok 和 mapstruct 依赖，就会出现java.lang.ExceptionInInitializerError异常

简单分析：
- debug发现，INSTANCE这个静态常量无法初始化
- 查询一些文档发现可能是lombok 提供的get set方法没有在 mapstruct之前实现，导致无法初始化等问题

## 解决

在父项目的pom文件添加如下依赖
```xml
<!-- 版本号统一管理 -->  
<properties>  
    <!-- 依赖包版本 -->  
    <lombok.version>1.18.28</lombok.version>   
    <mapstruct.version>1.5.5.Final</mapstruct.version>  
</properties>

<!-- 统一依赖管理 -->  
<dependencyManagement>  
    <dependencies>
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

在使用到mapstruct依赖的模块添加对应的依赖
```xml

<!-- 免写冗余的 Java 样板式代码 -->  
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <optional>true</optional>  
</dependency>

<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct-processor</artifactId>  
</dependency>
```

即可完成！！
