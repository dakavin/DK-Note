---
文章标题: "[[2_后端封装 MD 转换 HTML 工具类]]" 
文章作者: Dakkk
文章概要: |
  文章详述了使用 `commonmark-java` 库在后端构建 Markdown 转 HTML 工具类。内容涵盖依赖引入、基础转换，并逐步实现表格、标题ID、图片宽高、任务列表等扩展。还通过自定义渲染器支持超链接 `nofollow` 和图片描述，提升博客文章渲染效果。
文章标签:
- "Java"
- "Markdown"
- "HTML"
- "commonmark-java"
- "后端开发"
- "工具类"
- "文本转换"
- "Maven"
相关文章:
- "[[0_参考文章索引]]"
- "[[1_Arrays工具类]]"
- "[[2_Collections工具类]]"
- "[[5_姿势3：使用 GitHub Action（后端项目）]]"
- "[[12_📕文章管理：获取所有标签 Select 列表接口开发]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/11_博客前台：文章详情页开发/2_后端封装 MD 转换 HTML 工具类.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-05-15 22:38:06
修改时间: 2025-05-27 01:01:19
---

从这小节开始，我们开始进入到前台获取文章详情接口的开发，这个接口中，有一项非常重要的工作 —— _Markdown 文本转换 HTML 代码_， 后端直接将表中的 `markdown` 文本转换为 `html` 代码，以供前端渲染使用。所以，我们先要在后端服务中，封装一个 `Markdown` 文本转换器，方便到时候业务层直接调用。
## 1 **CommonMark** 库

`CommonMark` 是一个严格和标准化的 `Markdown` 规范，该库的实现非常快速和准确。`commonmark-java` 是 `CommonMark`规范的 `Java` 实现库，它的 `GitHub` 地址：[https://github.com/commonmark/commonmark-java](https://github.com/commonmark/commonmark-java) 。 咱们 `Weblog`博客项目中， 将会使用这个库来做 `Markdown` 文本的转换工作。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/8f326d8a1ae37df0ba34331594669886.png)

## 2 添加依赖

编辑 `weblog-springboot` 父项目的 `pom.xml` , 声明 `commonmark` 库的版本号，添加依赖如下：

```
    <!-- 版本号统一管理 -->
    <properties>
		// 省略...
        <commonmark.version>0.20.0</commonmark.version>
    </properties>
    
    <!-- 统一依赖管理 -->
    <dependencyManagement>
        <dependencies>
        
			// 省略...
			<!-- Markdown 解析 -->
            <dependency>
                <groupId>org.commonmark</groupId>
                <artifactId>commonmark</artifactId>
                <version>${commonmark.version}</version>
            </dependency>


        </dependencies>
    </dependencyManagement>
```

接着，编辑 `weblog-web` 子模块的 `pom.xml` 文件，引入该依赖，点击 `reload` 拉一下该库的 `jar` 包到本地：

```
       <!-- Markdown 解析 -->
        <dependency>
            <groupId>org.commonmark</groupId>
            <artifactId>commonmark</artifactId>
        </dependency>
```

## 3 工程结构

本小节中涉及的 `Markdown` 转换功能代码，主要涉及下图中标注的 3 个类，大家先了解一下，然后开始动手：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/1380611d328c03a60391e736928159c4.png)

## 4 创建 Markdown 转换器工具类

开始正式进入工具类的封装工作，我们在 `weblog-web` 模块中，新添加一个 `markdown` 包，统一放置 `markdown` 文本转换相关的代码，然后，新创建一个 `MarkdownHelper` 工具类，代码如下：

```
package com.quanxiaoha.weblog.web.markdown;

import org.commonmark.node.Node;
import org.commonmark.parser.Parser;
import org.commonmark.renderer.html.HtmlRenderer;

public class MarkdownHelper {

    /**
     * Markdown 解析器
     */
    private final static Parser PARSER;
    /**
     * HTML 渲染器
     */
    private final static HtmlRenderer HTML_RENDERER;

    /**
     * 初始化
     */
    static {
        PARSER = Parser.builder().build();
        HTML_RENDERER = HtmlRenderer.builder().build();
    }


    /**
     * 将 Markdown 转换成 HTML
     * @param markdown
     * @return
     */
    public static String convertMarkdown2Html(String markdown) {
        Node document = PARSER.parse(markdown);
        return HTML_RENDERER.render(document);
    }

    public static void main(String[] args) {
        String markdown = "This is *Sparta*";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }

}
```

以上代码中，主要涉及两个实例，一个是_解析器_， 另一个是 _`HTML` 渲染器_ ， 在静态代码块中，我们首先对它们进行了实例化。并对外提供了一个静态方法 `convertMarkdown2Html()` 方法，用来将 `markdown` 文本转换为 `HTML` 代码，入参是 `markdown` 文本，出参是 `HTML` 代码。最后，通过一个 `main` 方法测试了一下，看看该工具类方法能够正常工作，效果如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/10037d5a49c35a15120e56b2d1559cb1.png)

可以看到，能够正常将 `markdown` 文本转换为 `HTML` 代码的。
## 5 支持表格转换

默认情况下，该库是不支持 `markdown` 表格转换的，这里我们可以来测试一下：

```
    public static void main(String[] args) {
        String markdown = "| First Header  | Second Header |\n" +
                "| ------------- | ------------- |\n" +
                "| Content Cell  | Content Cell  |\n" +
                "| Content Cell  | Content Cell  |";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }
```

运行 `main` 方法后，控制台打印如下，并没有将其转换为 `<table>` 代码：

```
<p>| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |</p>
```

### 5.1 添加拓展

为了支持表格装换，官方提供了对应的拓展，我们需要手动添加该拓展。首先，在父 `pom.xml` 文件中，声明该扩展的依赖，代码如下：

```
            <dependency>
                <groupId>org.commonmark</groupId>
                <artifactId>commonmark-ext-gfm-tables</artifactId>
                <version>${commonmark.version}</version>
            </dependency>
```

然后，在 `weblog-web` 字模块中的 `pom.xml` 文件中，引入该依赖：

```
        <dependency>
            <groupId>org.commonmark</groupId>
            <artifactId>commonmark-ext-gfm-tables</artifactId>
        </dependency>
```

接着，在工具类中添加表格拓展，并给解析器和渲染器都设置上，代码如下：

```
    /**
     * 初始化
     */
    static {
        // Markdown 拓展
        List<Extension> extensions = Arrays.asList(
                TablesExtension.create() // 表格拓展
        );
        
        PARSER = Parser.builder().extensions(extensions).build();
        HTML_RENDERER = HtmlRenderer.builder().extensions(extensions).build();
    }
```

再次测试效果，你会发现已经能够正常转换为 `<table>` 表格代码了：

```
<table>
<thead>
<tr>
<th>First Header</th>
<th>Second Header</th>
</tr>
</thead>
<tbody>
<tr>
<td>Content Cell</td>
<td>Content Cell</td>
</tr>
<tr>
<td>Content Cell</td>
<td>Content Cell</td>
</tr>
</tbody>
</table>
```

## 6 自动添加标题 ID

文章详情页中，通常来说都会给每个标题，如 `<h1>` 、`<h2>` 等标题自动添加 `id` 属性，如下图所示，后续如果给博客添加目录功能时，可以通过该 `id` 快速跳转到指定标题位置：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/1fbac9dd3856dbe09c64359595a005b8.png)

`commonmark` 同样提供了相关拓展。在父 `pom.xml` 中声明该扩展的版本，代码如下：

```
            <dependency>
                <groupId>org.commonmark</groupId>
                <artifactId>commonmark-ext-heading-anchor</artifactId>
                <version>${commonmark.version}</version>
            </dependency>
```

然后，在 `weblog-web` 子模块中引入该拓展：

```
        <dependency>
            <groupId>org.commonmark</groupId>
            <artifactId>commonmark-ext-heading-anchor</artifactId>
        </dependency>
```

回到 `MarkdownHelper` 工具类中，添加该拓展：

```
    /**
     * 初始化
     */
    static {
        // Markdown 拓展
        List<Extension> extensions = Arrays.asList(
                TablesExtension.create(), // 表格拓展
                HeadingAnchorExtension.create() // 标题锚定项
        );

        PARSER = Parser.builder().extensions(extensions).build();
        HTML_RENDERER = HtmlRenderer.builder().extensions(extensions).build();
    }
    
    // 省略...
    
   public static void main(String[] args) {
        String markdown = "# 一级标题\n" +
                "## 二级标题\n";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }
```

运行 `main` 方法来测试一下，效果如下，正确为每个标题添加了 `id` 属性：

```
<h1 id="一级标题">一级标题</h1>
<h2 id="二级标题">二级标题</h2>
```

## 7 支持自定义图片宽高

`markdown` 语法无法对图片指定宽高，有时候上传的图片太大，文章里面想要手动指定一下宽高，就会体会到不方便的地方。如果能够像下面这样：

```
![text](/url.png){width=640 height=480}
```

能够支持通过 `{width=640 height=480}` 来指定宽高，转换器能够动态识别，并添加宽高属性就好了，就像下面这样：

```
<img src="/url.png" alt="text" width="640" height="480" />
```

要想实现上面的效果，我们可以添加如下拓展。在父 `pom.xml` 文件中，声明该拓展版本号：

```
            <dependency>
                <groupId>org.commonmark</groupId>
                <artifactId>commonmark-ext-image-attributes</artifactId>
                <version>${commonmark.version}</version>
            </dependency>
```

然后在 `weblog-web` 子模块中，引入该依赖：

```
        <dependency>
            <groupId>org.commonmark</groupId>
            <artifactId>commonmark-ext-image-attributes</artifactId>
        </dependency>
```

最后在工具类中添加该拓展：

```
    /**
     * 初始化
     */
    static {
        // Markdown 拓展
        List<Extension> extensions = Arrays.asList(
                TablesExtension.create(), // 表格拓展
                HeadingAnchorExtension.create(), // 标题锚定项
                ImageAttributesExtension.create() // 图片宽高
        );

        PARSER = Parser.builder().extensions(extensions).build();
        HTML_RENDERER = HtmlRenderer.builder().extensions(extensions).build();
    }
    
    // 省略...
    public static void main(String[] args) {
        String markdown = "![text](/url.png){width=640 height=480}";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }
```

我们来测试一下，运行 `main` 方法，控制台输入如下，达到了我们想要的效果：

```
<p><img src="/url.png" alt="text" width="640" height="480" /></p>
```

## 8 任务列表

偶尔呢，也会需要任务列表的效果，能够展示哪些任务完成了，哪些任务未完成。如果 `markdown` 能够支持如下输入：

```
- [ ] task #1
- [x] task #2
```

并动态将 `[ ]` 、 `[x]` 转换为 `checkbox` ，来标识该任务是否已完成，如下所示：

```
<ul>
<li><input type="checkbox" disabled=""> task #1</li>
<li><input type="checkbox" disabled="" checked=""> task #2</li>
</ul>
```

同样有对应的拓展。编辑父 `pom.xml` 文件声明该拓展：

```
            <dependency>
                <groupId>org.commonmark</groupId>
                <artifactId>commonmark-ext-task-list-items</artifactId>
                <version>${commonmark.version}</version>
            </dependency>
```

并在 `weblog-web` 子模块中引入该拓展：

```
        <dependency>
            <groupId>org.commonmark</groupId>
            <artifactId>commonmark-ext-task-list-items</artifactId>
        </dependency>
```

最后在工具类中添加该拓展：

```
    /**
     * 初始化
     */
    static {
        // Markdown 拓展
        List<Extension> extensions = Arrays.asList(
                TablesExtension.create(), // 表格拓展
                HeadingAnchorExtension.create(), // 标题锚定项
                ImageAttributesExtension.create(), // 图片宽高
                TaskListItemsExtension.create() // 任务列表
        );

        PARSER = Parser.builder().extensions(extensions).build();
        HTML_RENDERER = HtmlRenderer.builder().extensions(extensions).build();
    }
    // 省略...
```

## 9 支持超链接中动态添加 nofollow

### 9.1 什么是 nofollow ? 为什么需要它 ？

`nofollow` 是 HTML 中 `<a>` 标签的一个属性值，它的主要作用是告诉搜索引擎“**不要追踪此链接**”。当一个网页中的链接被设置为 `nofollow` 时，搜索引擎会忽略这个链接，即不会从这个链接获取权重或跟踪链接的内容。

`nofollow` 的作用主要表现在以下几个方面：

1. 避免权重分散：当一个网页上存在多个链接时，搜索引擎会根据链接的重要性进行权重分配。但是，如果其中一些链接被设置为`nofollow`，那么这些链接就不会被搜索引擎赋予权重，从而避免了权重的分散。
2. 提高网站排名：通过将一些不重要的链接设置为 `nofollow`，可以将权重集中到重要的链接上，从而提高网站的重要页面排名。
3. 避免垃圾链接：有些网站为了增加外链数量，会发布一些毫无价值的垃圾链接。如果这些链接被设置为`nofollow`，那么搜索引擎就不会跟踪这些链接，从而避免了垃圾链接的影响。
4. 控制页面权重：通过将`nofollow`属性添加到不重要的页面或特定链接上，可以控制页面的权重分配，使重要的页面获得更多的权重，从而提高网站的整体权重。

### 9.2 自定义 AttributeProvider

通常来说，我们只需要给非本网站域名的超链接添加 `nofollow` 即可，比如你在你的博文中，引用了 `redis` 的官方网址，则可以添加 `nofollow` , 防止将博客的权重传递过去。为了实现该效果，我们在 `/markdown` 包下新建 `/provider` 包，然后，创建一个 `NofollowLinkAttributeProvider` 类，代码如下：

```
package com.quanxiaoha.weblog.web.markdown.provider;

import org.commonmark.node.Link;
import org.commonmark.node.Node;
import org.commonmark.renderer.html.AttributeProvider;

import java.util.Map;

public class NofollowLinkAttributeProvider implements AttributeProvider {

    /**
     * 网站域名（上线后需要改成自己的域名）
     */
    private final static String DOMAIN = "www.quanxiaoha.com";

    @Override
    public void setAttributes(Node node, String tagName, Map<String, String> attributes) {
        if (node instanceof Link) {
            Link linkNode = (Link) node;
            // 获取链接地址
            String href = linkNode.getDestination();
            // 如果链接不是自己域名，则添加 rel="nofollow" 属性
            if (!href.contains(DOMAIN)) {
                attributes.put("rel", "nofollow");
            }
        }
    }
}
```

上述类实现了 `AttributeProvider` 接口，通过实现其 `setAttributes` 方法，并对 `node` 进行判断，若类型为 `Link` 链接，并且链接不包含我们网站的域名时，则为其添加 `rel="nofollow"` 属性。

定义好了 `NofollowLinkAttributeProvider` 后，再将其添加到解析器中：

```
    /**
     * 初始化
     */
    static {
       	// 省略...
        HTML_RENDERER = HtmlRenderer.builder()
                .extensions(extensions)
                .attributeProviderFactory(context -> new NofollowLinkAttributeProvider())
                .build();
    }
    
    // 省略...
    public static void main(String[] args) {
        String markdown = "[个人网站域名](http://www.quanxiaoha.com \"个人网站域名\")\n" +
                "\n" +
                "[第三方网站域名](http://www.baidu.com \"第三方网站域名\")";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }
```

接下来，我们来测试一下，`markdown` 文本中有两个链接，一个是本网站的链接 `http://www.quanxiaoha.com` , 一本非本网站的链接，对其进行转换，控制台打印如下，功能正常：

```
<p><a href="http://www.quanxiaoha.com" title="个人网站域名">个人网站域名</a></p>
<p><a href="http://www.baidu.com" title="第三方网站域名" rel="nofollow">第三方网站域名</a></p>
```

## 10 图片下方描述文字

如果你的 `markdown` 图片格式如下，添加了图片描述：

```
![图片描述](https://img.quanxiaoha.com/quanxiaoha/169883410438258 "图片描述")
```

很多做的比较好的博客中，都会将图片的描述文字放到图片底部，交互更加友好：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/7a0158b398d44de5f8705e74bc07cb09.png)


比如小哈的教程站：[www.quanxiaoha.com](https://www.quanxiaoha.com/column/www.quanxiaoha.com) 就实现了该功能，通过 _F12_ 审查元素，我们可以看到描述文字其实就是一个 `<span>` 标签，如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/53a966f7c182ea08d708f80bc108267b.png)


但是，如果不做定制化，默认情况系下， `markdown` 转换库都不会给你额外添加描述文字。下面就来演示一下，如何定制 `commonmark` 库实现该功能，我们在 `/markdown` 包下，再新建一个 `/renderer` 包，并创建 `ImageNodeRenderer` 图片节点渲染类，代码如下：

```java
package com.quanxiaoha.weblog.web.markdown.renderer;

import org.apache.commons.lang3.StringUtils;
import org.commonmark.ext.image.attributes.ImageAttributes;
import org.commonmark.node.Image;
import org.commonmark.node.Node;
import org.commonmark.renderer.NodeRenderer;
import org.commonmark.renderer.html.HtmlNodeRendererContext;
import org.commonmark.renderer.html.HtmlWriter;
import org.springframework.util.CollectionUtils;

import java.util.Collections;
import java.util.Objects;
import java.util.Set;

public class ImageNodeRenderer implements NodeRenderer {

    private final HtmlWriter html;

    /**
     * 图片宽
     */
    private final static String KEY_WIDTH = "width";
    /**
     * 图片高
     */
    private final static String KEY_HEIGHT = "height";


    public ImageNodeRenderer(HtmlNodeRendererContext context) {
        this.html = context.getWriter();
    }

    @Override
    public Set<Class<? extends Node>> getNodeTypes() {
        // 指定想要自定义渲染的节点，这里指定为图片 Image
        return Collections.singleton(Image.class);
    }

    @Override
    public void render(Node node) {
        // We only handle one type as per getNodeTypes, so we can just cast it here.
        Image img = (Image) node;
        html.line();
        // 图片链接
        String imgUrl = img.getDestination();
        // 图片描述
        String imgTitle = img.getTitle();

        // 拼接 HTML 结构
        StringBuilder sb = new StringBuilder("<img src=\"");
        sb.append(imgUrl);
        sb.append("\"");

        // 添加 title="图片标题" alt="图片标题" 属性
        if (StringUtils.isNotBlank(imgTitle)) {
            sb.append(String.format(" title=\"%s\" alt=\"%s\"", imgTitle, imgTitle));
        }

        // 图片宽高
        Node lastChild = node.getLastChild();
        if (Objects.nonNull(lastChild) && lastChild instanceof ImageAttributes) {
            ImageAttributes imageAttributes = (ImageAttributes) lastChild;
            if (Objects.nonNull(imageAttributes) && !CollectionUtils.isEmpty(imageAttributes.getAttributes())) {
                String width = imageAttributes.getAttributes().get(KEY_WIDTH);
                String height = imageAttributes.getAttributes().get(KEY_HEIGHT);
                sb.append(StringUtils.isBlank(width) ? "" : (" " + KEY_WIDTH + "=" + width + "\""));
                sb.append(StringUtils.isBlank(height) ? "" : (" " + KEY_HEIGHT + "=" + height + "\""));
            }
        }
        sb.append(">");

        if (StringUtils.isNotBlank(imgTitle)) {
            // 图文下方文字
            sb.append(String.format("<span>%s</span>", imgTitle));
        }
		
		// 设置 HTML 内容
        html.raw(sb.toString());
        html.line();
    }
}

```

主要看 `render()` 方法，该方法需要用于渲染 `HTML` 代码，首先，我们将 `node` 节点转换为图片 `Image` , 该类是 `commonmark` 库内部提供的。然后拿到图片的链接与描述，接着开始拼接 `HTML` 代码，如果 `markdown` 图片指定了描述文字时，则添加 `title` 和 `alt` 属性值， 然后是对图片宽高的支持，因为这里我们自定义了图片节点的渲染逻辑，所以前面添加的 `ImageAttributesExtension` 拓展不生效了, 所以需要自行支持一波宽高功能。

接着，就是图片下方的提示文字了，同样的是对图片描述进行判空，若不为空则添加，否则不添加。最终，通过 `raw()` 方法将拼接好的 `HTML` 代码设置好。

> TIP : 图文下方文字，这里并没有设置任何样式，仅仅添加了 `HTML` 结构, 样式等到后续写页面的时候，再回过头来完善。

自定义节点渲染类开发完成后，我们在解析器中添加一下，代码如下：

```'
    /**
     * 初始化
     */
    static {
        // Markdown 拓展
        List<Extension> extensions = Arrays.asList(
                TablesExtension.create(), // 表格拓展
                HeadingAnchorExtension.create(), // 标题锚定项
                ImageAttributesExtension.create(), // 图片宽高
                TaskListItemsExtension.create() // 任务列表
        );

        PARSER = Parser.builder().extensions(extensions).build();
        HTML_RENDERER = HtmlRenderer.builder()
                .extensions(extensions)
                .attributeProviderFactory(context -> new NofollowLinkAttributeProvider())
                .nodeRendererFactory(context -> new ImageNodeRenderer(context))
                .build();
    }
    
    // 省略...
    public static void main(String[] args) {
        String markdown = "![图 1-1 技术栈](https://img.quanxiaoha.com/quanxiaoha/169560181378937 \"图 1-1 技术栈\")";
        System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
    }
```

然后来测试一下，看看是否能够正确转换出图片下方的文字，效果如下，没有任何问题：

```
<p>
<img src="https://img.quanxiaoha.com/quanxiaoha/169560181378937" title="图 1-1 技术栈" alt="图 1-1 技术栈"><span>图 1-1 技术栈</span>
</p>
```

再来测试一下宽高功能是否支持：

```
String markdown = "![图 1-1 技术栈](https://img.quanxiaoha.com/quanxiaoha/169560181378937 \"图 1-1 技术栈\"){width=100 height=100}";
System.out.println(MarkdownHelper.convertMarkdown2Html(markdown));
```

运行 `main` 方法，控制台输入如下，宽高属性的设置也是没问题的：

```
<p>
<img src="https://img.quanxiaoha.com/quanxiaoha/169560181378937" title="图 1-1 技术栈" alt="图 1-1 技术栈" width=100" height=100"><span>图 1-1 技术栈</span>
</p>
```

至此，本小结对 `Markdown` 转换 `HTML` 的工具类就封装完成了。下小节中，我们正式来开发获取文章详情接口。