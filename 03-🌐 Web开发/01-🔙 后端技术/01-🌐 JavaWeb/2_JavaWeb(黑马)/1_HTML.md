---
文章标题: "[[1_HTML]]" 
文章作者: Dakkk
文章概要: |
  介绍HTML基础概念、语法结构、常用标签(文本、图片、表格、表单等)及其属性，通过案例演示网页开发入门知识
tags:
- "HTML"
- "网页开发"
- "标签语法"
- "表单"
- "表格"
- "超链接"
- "前端基础"
- "标记语言"
相关文章:
- "[[2_HTML和CSS]]"
- "[[2_关于HTML超链接问题]]"
- "[[12_添加Table组件加载Loading、表单对话框提交按钮Loading动画]]"
- "[[2_后端封装 MD 转换 HTML 工具类]]"
- "[[6_分类管理页面：样式布局]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/1_HTML.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:00:58
---
## 1 概念

- 最基础的网页开发语言
* Hyper Text Markup Language 超文本标记语言
	* `超文本`:
		* 超文本是用`超链接`的方法，将各种不同空间的文字信息组织在一起的网状文本.
	* `标记语言`:
		* 由`标签`构成的语言。<标签名称> 如 html，xml
		* `标记语言不是编程语言`

## 2 快速入门

### 2.1 语法：

1. html文档后缀名 .html 或者 .htm
2. 标签分为
	1. <font color="#d83931">围堵标签</font>：有开始标签和结束标签。如 `<html> </html>`
	2. <font color="#d83931">自闭和标签</font>：开始标签和结束标签在一起。如 `<br/>`
3. 标签可以嵌套：
	需要正确嵌套，不能你中有我，我中有你
	错误：`<a><b></a></b>`
	正确：`<a><b></b></a>`
4. 在`开始标签中可以定义属性。属性是由键值对构成，值需要用引号(单双都可)引起来`
5. html的标签不区分大小写，但是`建议使用小写`。
6. html语法松散
7. html标签`属性值单双引号都可以`

### 2.2 代码

```html
<html>
	<head>
		<title>title</title>
	</head>
		
	<body>
		<FONT color='red'>Hello World</font><br/>
		<font color='green'>Hello World</font>
	</body>
</html>
```

## 3 标签学习

### 3.1 文件标签

- `构成html最基本的标签`

* `html` :    html文档的根标签
* `head`：头标签。<font color="#d83931">用于指定html文档的一些属性。引入外部的资源</font>
* `title`：  标题标签。
* `body`：体标签
* `<!DOCTYPE html>`：html5中定义该文档是html文档

### 3.2 文本标签

- `和文本有关的标签`

* `注释`：`<!-- 注释内容 -->`
* `<h1> to <h6>`：标题标签   h1~h6:字体大小逐渐递减
* `<p>`：段落标签
* `<br>`：换行标签
* `<hr>`：展示一条水平线
	* <font color="#d83931">属性</font>：
		* color：颜色
		* width：宽度
		* size：高度
		* align：对其方式
			* center：居中
			* left：左对齐
			* right：右对齐
* `<b>`：字体加粗
* `<i>`：字体斜体
* `<font>`:字体标签
* `<center>`:文本居中
	* <font color="#d83931">属性</font>：
		* color：颜色
		* size：大小
		* face：字体

* <font color="#d83931">属性定义</font>：
	* color：
		1. 英文单词：red,green,blue
		2. rgb(值1，值2，值3)：值的范围：0~255  如  rgb(0,0,255)
		3. `#值1值2值3`：值的范围：00~FF之间。如： `#FF00FF`
	* width：
		1. 数值：width='20' ,数值的单位，默认是 px(像素)
		2. 数值%：占比相对于父元素的比例

- `案例`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/40294a18a31b60e46b55c8e4bb1b76a5.png)




### 3.3 图片标签

* `img`：展示图片
	* <font color="#d83931">属性</font>：
		* src：指定图片的位置

- `代码`
```html
 <!--展示一张图片 img-->

<img src="image/jingxuan_2.jpg" align="right" alt="古镇" width="500" height="500"/>

<!--
	相对路径
		* 以.开头的路径
			* ./：代表当前目录  ./image/1.jpg
			* ../:代表上一级目录
 -->

<img src="./image/jiangwai_1.jpg">

<img src="../image/jiangwai_1.jpg">
```

### 3.4 列表标签

* 有序列表：
	* ol:
		* li，每一个选项都用此标签，包住
* 无序列表：
	* ul:
		* li，每一个选项都用此标签，包住

### 3.5 链接标签

* `a`:定义一个超链接
	* <font color="#d83931">属性</font>：
		* href：指定访问资源的URL(统一资源定位符)
			* 可以指定网络上的地址
			* 也可以指定当前文件夹中的其他HTML文件
		* target：指定打开资源的方式
			* \_self:默认值，在当前页面打开
			* \_blank：在空白页面打开

- `代码`
```html
 <!--超链接  a-->

<a href="http://www.itcast.cn">点我</a>
<br>

<a href="http://www.itcast.cn" target="_self">点我</a>
<br>

<a href="http://www.itcast.cn" target="_blank">点我</a>
<br>

<a href="./5_列表标签.html">列表标签</a><br>
<a href="mailto:itcast@itcast.cn">联系我们</a>
<br>

<a href="http://www.itcast.cn"><img src="image/jiangwai_1.jpg"></a>
```

### 3.6 div和span

* `div`:每一个div占满一整行。块级标签---不换行的，占据整个行
* `span`：文本信息在一行展示，行内标签 内联标签---换行，不占据整个行

### 3.7 语义化标签

- html4中，一般是如下定义页眉和页脚的
1. `<div id = "header"> </div>`
2. `<div id = "footer> </div>`

- html5中为了提高程序的可读性，提供了一些标签。
3. `<header>`：页眉
4. `<footer>`：页脚


### 3.8 表格标签

* `table`：定义表格
	* `width`：宽度
	* `border`：边框
	* `cellpadding`：定义内容和单元格的距离
	* `cellspacing`：定义单元格之间的距离。如果指定为0，则单元格的线会合为一条
	* `bgcolor`：背景色
	* `align`：对齐方式，相对于外面层次的位置，不是表格中的内容居中
* `tr`：定义行
	* `bgcolor`：背景色
	* `align`：对齐方式，此时是使得表格中某行的内容居中
* `td`：定义单元格
	* `colspan`：合并列
	* `rowspan`：合并行
* `th`：定义表头单元格
* `<caption>`：表格标题
* `<thead>`：表示表格的头部分
* `<tbody>`：表示表格的体部分
* `<tfoot>`：表示表格的脚部分

### 3.9 视频标签

- `src`：规定视频的url
- `controls`：显示播放控件
- `width`：播放器的宽度
- `height`：播放器的高度

## 4 案例：旅游网站首页

- 第一步：确定布局（不使用CSS来布局）
	- 使用table
- 第二步；如果某一行只有一个单元格，则使用 tr和td即可
- 第三步：如果某一行有多个单元格，则在td内嵌套一个table

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/83481fe7a25b1a9a9bff31c2d2aad803.png)



## 5 表单标签

- 概念：用于`采集用户输入的数据`的。用于`和服务器进行交互`

### 5.1 from

* `form`：用于定义表单的。可以定义一个范围，范围代表采集用户数据的范围
	* 属性：
		* `action`：指定`提交数据的URL`
		* `method`:指定提交方式
			* 分类：一共7种，2种比较常用
			   * `get`：
					1. <font color="#d83931">请求参数会在地址栏中显示</font>。会封装到请求行中(HTTP协议后讲解)。
					2. 请求参数大小是有限制的。
					3. 不太安全。
			   * `post`：
						1. <font color="#d83931">请求参数不会再地址栏中显示</font>。会封装在请求体中(HTTP协议后讲解)
					1. 请求参数的大小没有限制。
					2. 较为安全。

* `表单项中的数据要想被提交：必须指定其name属性`

### 5.2 表单项标签

* `input`：可以`通过type属性值，改变元素展示的样式`
	* `type属性`：
		* `text`：文本输入框，`默认值`
			* `placeholder`：指定输入框的提示信息，当输入框的内容发生变化，会自动清空提示信息
			* `value`：不建议使用这个，不然输入框的内容不会发生变化
		* `password`：密码输入框
		* `radio`:单选框
			* 注意：
				1. 要想<font color="#d83931">让多个单选框实现单选的效果，则多个单选框的name属性值必须一样</font>。
				2. 一般会给<font color="#d83931">每一个单选框提供value属性，指定其被选中后提交的值</font>
				3. checked属性，可以指定默认值
		* `checkbox`：复选框
			* 注意：
				1. 一般会给<font color="#d83931">每一个单选框提供value属性，指定其被选中后提交的值</font>
				2. checked属性，可以指定默认值

		* `file`：文件选择框，如：上传功能
		* `hidden`：隐藏域，用于提交一些信息。
		* `按钮`：
			* submit：提交按钮。可以提交表单
			* button：普通按钮
			* image：图片提交按钮
				* src属性指定图片的路径
		* `color`：取色器
		* `date`/`datetime-local`：日期选择器，一个包括时间，一个不包括时间
		* email

   * `label`：指定输入项的文字描述信息
	   * 注意：
		   * `label的for属性`一般会和 `input 的 id属性值` 对应。如果对应了，则点击label区域，会让input输入框获取焦点。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5bf949cea5743b6bb24c76c5f2e59cb4.png)



* `select`: 下拉列表
	* 子元素：option，指定列表项
	* 记住select加上name属性！！
	* option加上value值，代表默认值
	* 在option加上selected，表示默认选项
	
* `textarea`：文本域
	* cols：指定列数，每一行有多少个字符
	* rows：默认多少行。
	* 注意name属性要加上

### 5.3 补充

在一个表单中，可以存在很多的表单项，而虽然表单项的形式各式各样，但是表单项的标签其实就只有三个，分别是：

- `<input>`: 表单项 , 通过type属性控制输入形式。
    
|type取值|**描述**|
|---|---|
|text|默认值，定义单行的输入字段|
|password|定义密码字段|
|radio|定义单选按钮|
|checkbox|定义复选框|
|file|定义文件上传按钮|
|date/time/datetime-local|定义日期/时间/日期时间|
|number|定义数字输入框|
|email|定义邮件输入框|
|hidden|定义隐藏域|
|submit / reset / button|定义提交按钮 / 重置按钮 / 可点击按钮|
    
- `<select>`: 定义下拉列表, `<option>` 定义列表项
    
- `<textarea>`: 文本域

- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0ef1ea01689356c580030d1272700e64.png)




### 5.4 案例

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
    <form action="#" method="post">  
        <table align="center" width="50%" border="1px">  
            <tr>  
                <td><label for="username">用户名</label></td>  
                <td><input type="text" name="username"  id="username" placeholder="请输入账号"></td>  
<!--                后续依次按上述修改即可-->  
            </tr>  
            <tr>                <td>密码</td>  
                <td><input type="password" placeholder="请输入密码"></td>  
            </tr>  
            <tr>                <td>Email</td>  
                <td><input type="email" placeholder="请输入Email"></td>  
            </tr>  
            <tr>                <td>姓名</td>  
                <td><input type="text" placeholder="请输入真实姓名"></td>  
            </tr>  
            <tr>                <td>手机号</td>  
                <td><input type="number" placeholder="请输入手机号"></td>  
            </tr>  
            <tr>                <td>性别</td>  
                <td>                    <input type="radio" name="gender" checked>&nbsp;男  
                    <input type="radio" name="gender">&nbsp;女  
                </td>  
            </tr>  
            <tr>                <td>出生日期</td>  
                <td><input type="date"></td>  
            </tr>  
            <tr>                <td>验证码</td>  
                <td><input type="text"><img src="img/verify_code.jpg"></td>  
            </tr>  
            <tr>                <td colspan="2" align="center"><input type="button" value="注册"></td>  
            </tr>  
        </table>  
    </form>  
</body>  
</html>
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c94beb5be1401d7fcefe70bdae8febcf.png)





## 1 概念

- 最基础的网页开发语言
* Hyper Text Markup Language 超文本标记语言
	* `超文本`:
		* 超文本是用`超链接`的方法，将各种不同空间的文字信息组织在一起的网状文本.
	* `标记语言`:
		* 由`标签`构成的语言。<标签名称> 如 html，xml
		* `标记语言不是编程语言`

## 2 快速入门

### 2.1 语法：

1. html文档后缀名 .html 或者 .htm
2. 标签分为
	1. <font color="#d83931">围堵标签</font>：有开始标签和结束标签。如 `<html> </html>`
	2. <font color="#d83931">自闭和标签</font>：开始标签和结束标签在一起。如 `<br/>`
3. 标签可以嵌套：
	需要正确嵌套，不能你中有我，我中有你
	错误：`<a><b></a></b>`
	正确：`<a><b></b></a>`
4. 在`开始标签中可以定义属性。属性是由键值对构成，值需要用引号(单双都可)引起来`
5. html的标签不区分大小写，但是`建议使用小写`。
6. html语法松散
7. html标签`属性值单双引号都可以`

### 2.2 代码

```html
<html>
	<head>
		<title>title</title>
	</head>
		
	<body>
		<FONT color='red'>Hello World</font><br/>
		<font color='green'>Hello World</font>
	</body>
</html>
```

## 3 标签学习

### 3.1 文件标签

- `构成html最基本的标签`

* `html` :    html文档的根标签
* `head`：头标签。<font color="#d83931">用于指定html文档的一些属性。引入外部的资源</font>
* `title`：  标题标签。
* `body`：体标签
* `<!DOCTYPE html>`：html5中定义该文档是html文档

### 3.2 文本标签

- `和文本有关的标签`

* `注释`：`<!-- 注释内容 -->`
* `<h1> to <h6>`：标题标签   h1~h6:字体大小逐渐递减
* `<p>`：段落标签
* `<br>`：换行标签
* `<hr>`：展示一条水平线
	* <font color="#d83931">属性</font>：
		* color：颜色
		* width：宽度
		* size：高度
		* align：对其方式
			* center：居中
			* left：左对齐
			* right：右对齐
* `<b>`：字体加粗
* `<i>`：字体斜体
* `<font>`:字体标签
* `<center>`:文本居中
	* <font color="#d83931">属性</font>：
		* color：颜色
		* size：大小
		* face：字体

* <font color="#d83931">属性定义</font>：
	* color：
		1. 英文单词：red,green,blue
		2. rgb(值1，值2，值3)：值的范围：0~255  如  rgb(0,0,255)
		3. `#值1值2值3`：值的范围：00~FF之间。如： `#FF00FF`
	* width：
		1. 数值：width='20' ,数值的单位，默认是 px(像素)
		2. 数值%：占比相对于父元素的比例

- `案例`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/40294a18a31b60e46b55c8e4bb1b76a5.png)




### 3.3 图片标签

* `img`：展示图片
	* <font color="#d83931">属性</font>：
		* src：指定图片的位置

- `代码`
```html
 <!--展示一张图片 img-->

<img src="image/jingxuan_2.jpg" align="right" alt="古镇" width="500" height="500"/>

<!--
	相对路径
		* 以.开头的路径
			* ./：代表当前目录  ./image/1.jpg
			* ../:代表上一级目录
 -->

<img src="./image/jiangwai_1.jpg">

<img src="../image/jiangwai_1.jpg">
```

### 3.4 列表标签

* 有序列表：
	* ol:
		* li，每一个选项都用此标签，包住
* 无序列表：
	* ul:
		* li，每一个选项都用此标签，包住

### 3.5 链接标签

* `a`:定义一个超链接
	* <font color="#d83931">属性</font>：
		* href：指定访问资源的URL(统一资源定位符)
			* 可以指定网络上的地址
			* 也可以指定当前文件夹中的其他HTML文件
		* target：指定打开资源的方式
			* \_self:默认值，在当前页面打开
			* \_blank：在空白页面打开

- `代码`
```html
 <!--超链接  a-->

<a href="http://www.itcast.cn">点我</a>
<br>

<a href="http://www.itcast.cn" target="_self">点我</a>
<br>

<a href="http://www.itcast.cn" target="_blank">点我</a>
<br>

<a href="./5_列表标签.html">列表标签</a><br>
<a href="mailto:itcast@itcast.cn">联系我们</a>
<br>

<a href="http://www.itcast.cn"><img src="image/jiangwai_1.jpg"></a>
```

### 3.6 div和span

* `div`:每一个div占满一整行。块级标签---不换行的，占据整个行
* `span`：文本信息在一行展示，行内标签 内联标签---换行，不占据整个行

### 3.7 语义化标签

- html4中，一般是如下定义页眉和页脚的
1. `<div id = "header"> </div>`
2. `<div id = "footer> </div>`

- html5中为了提高程序的可读性，提供了一些标签。
3. `<header>`：页眉
4. `<footer>`：页脚


### 3.8 表格标签

* `table`：定义表格
	* `width`：宽度
	* `border`：边框
	* `cellpadding`：定义内容和单元格的距离
	* `cellspacing`：定义单元格之间的距离。如果指定为0，则单元格的线会合为一条
	* `bgcolor`：背景色
	* `align`：对齐方式，相对于外面层次的位置，不是表格中的内容居中
* `tr`：定义行
	* `bgcolor`：背景色
	* `align`：对齐方式，此时是使得表格中某行的内容居中
* `td`：定义单元格
	* `colspan`：合并列
	* `rowspan`：合并行
* `th`：定义表头单元格
* `<caption>`：表格标题
* `<thead>`：表示表格的头部分
* `<tbody>`：表示表格的体部分
* `<tfoot>`：表示表格的脚部分

### 3.9 视频标签

- `src`：规定视频的url
- `controls`：显示播放控件
- `width`：播放器的宽度
- `height`：播放器的高度

## 4 案例：旅游网站首页

- 第一步：确定布局（不使用CSS来布局）
	- 使用table
- 第二步；如果某一行只有一个单元格，则使用 tr和td即可
- 第三步：如果某一行有多个单元格，则在td内嵌套一个table

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/83481fe7a25b1a9a9bff31c2d2aad803.png)



## 5 表单标签

- 概念：用于`采集用户输入的数据`的。用于`和服务器进行交互`

### 5.1 from

* `form`：用于定义表单的。可以定义一个范围，范围代表采集用户数据的范围
	* 属性：
		* `action`：指定`提交数据的URL`
		* `method`:指定提交方式
			* 分类：一共7种，2种比较常用
			   * `get`：
					1. <font color="#d83931">请求参数会在地址栏中显示</font>。会封装到请求行中(HTTP协议后讲解)。
					2. 请求参数大小是有限制的。
					3. 不太安全。
			   * `post`：
						1. <font color="#d83931">请求参数不会再地址栏中显示</font>。会封装在请求体中(HTTP协议后讲解)
					1. 请求参数的大小没有限制。
					2. 较为安全。

* `表单项中的数据要想被提交：必须指定其name属性`

### 5.2 表单项标签

* `input`：可以`通过type属性值，改变元素展示的样式`
	* `type属性`：
		* `text`：文本输入框，`默认值`
			* `placeholder`：指定输入框的提示信息，当输入框的内容发生变化，会自动清空提示信息
			* `value`：不建议使用这个，不然输入框的内容不会发生变化
		* `password`：密码输入框
		* `radio`:单选框
			* 注意：
				1. 要想<font color="#d83931">让多个单选框实现单选的效果，则多个单选框的name属性值必须一样</font>。
				2. 一般会给<font color="#d83931">每一个单选框提供value属性，指定其被选中后提交的值</font>
				3. checked属性，可以指定默认值
		* `checkbox`：复选框
			* 注意：
				1. 一般会给<font color="#d83931">每一个单选框提供value属性，指定其被选中后提交的值</font>
				2. checked属性，可以指定默认值

		* `file`：文件选择框，如：上传功能
		* `hidden`：隐藏域，用于提交一些信息。
		* `按钮`：
			* submit：提交按钮。可以提交表单
			* button：普通按钮
			* image：图片提交按钮
				* src属性指定图片的路径
		* `color`：取色器
		* `date`/`datetime-local`：日期选择器，一个包括时间，一个不包括时间
		* email

   * `label`：指定输入项的文字描述信息
	   * 注意：
		   * `label的for属性`一般会和 `input 的 id属性值` 对应。如果对应了，则点击label区域，会让input输入框获取焦点。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5bf949cea5743b6bb24c76c5f2e59cb4.png)



* `select`: 下拉列表
	* 子元素：option，指定列表项
	* 记住select加上name属性！！
	* option加上value值，代表默认值
	* 在option加上selected，表示默认选项
	
* `textarea`：文本域
	* cols：指定列数，每一行有多少个字符
	* rows：默认多少行。
	* 注意name属性要加上

### 5.3 补充

在一个表单中，可以存在很多的表单项，而虽然表单项的形式各式各样，但是表单项的标签其实就只有三个，分别是：

- `<input>`: 表单项 , 通过type属性控制输入形式。
    
|type取值|**描述**|
|---|---|
|text|默认值，定义单行的输入字段|
|password|定义密码字段|
|radio|定义单选按钮|
|checkbox|定义复选框|
|file|定义文件上传按钮|
|date/time/datetime-local|定义日期/时间/日期时间|
|number|定义数字输入框|
|email|定义邮件输入框|
|hidden|定义隐藏域|
|submit / reset / button|定义提交按钮 / 重置按钮 / 可点击按钮|
    
- `<select>`: 定义下拉列表, `<option>` 定义列表项
    
- `<textarea>`: 文本域

- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0ef1ea01689356c580030d1272700e64.png)




### 5.4 案例

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
    <form action="#" method="post">  
        <table align="center" width="50%" border="1px">  
            <tr>  
                <td><label for="username">用户名</label></td>  
                <td><input type="text" name="username"  id="username" placeholder="请输入账号"></td>  
<!--                后续依次按上述修改即可-->  
            </tr>  
            <tr>                <td>密码</td>  
                <td><input type="password" placeholder="请输入密码"></td>  
            </tr>  
            <tr>                <td>Email</td>  
                <td><input type="email" placeholder="请输入Email"></td>  
            </tr>  
            <tr>                <td>姓名</td>  
                <td><input type="text" placeholder="请输入真实姓名"></td>  
            </tr>  
            <tr>                <td>手机号</td>  
                <td><input type="number" placeholder="请输入手机号"></td>  
            </tr>  
            <tr>                <td>性别</td>  
                <td>                    <input type="radio" name="gender" checked>&nbsp;男  
                    <input type="radio" name="gender">&nbsp;女  
                </td>  
            </tr>  
            <tr>                <td>出生日期</td>  
                <td><input type="date"></td>  
            </tr>  
            <tr>                <td>验证码</td>  
                <td><input type="text"><img src="img/verify_code.jpg"></td>  
            </tr>  
            <tr>                <td colspan="2" align="center"><input type="button" value="注册"></td>  
            </tr>  
        </table>  
    </form>  
</body>  
</html>
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c94beb5be1401d7fcefe70bdae8febcf.png)




