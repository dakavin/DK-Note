---
文章标题: "[[2_CSS]]" 
文章作者: Dakkk
文章概要: |
  介绍CSS层叠样式表的基本概念，包含与HTML结合的三种方式（内联、内部、外部），CSS语法格式，各种选择器使用方法，以及字体、背景、边框等属性设置
tags:
- "CSS"
- "样式表"
- "选择器"
- "HTML"
- "前端开发"
- "网页布局"
- "盒子模型"
- "样式属性"
相关文章:
- "[[2_HTML和CSS]]"
- "[[8_首页样式布局设计（3）— 侧边栏分类、标签卡片]]"
- "[[12_JQuery]]"
- "[[2_关于HTML超链接问题]]"
- "[[1_登录页设计：支持响应式布局]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/2_CSS.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:05:10
---

## 1 概念

- Cascading Style Sheet 层叠样式表
- 层叠：多个样式可以作用在同一个html的元素(标签)上，同时生效

## 2 好处

1. 功能强大

2. 将内容展示和样式控制分离
	- 降低耦合度，解耦
	- 让分工协作更容易
	- 提高开发效率

## 3 CSS的使用

- CSS与HTML结合方式

1. `内联样式`
	 * 在标签内使用style属性指定css代码
	 * 如：
```html
<div style="color:red;">hello css</div>
```

2. `内部样式`
	* 在head标签内，定义style标签，style标签的标签体内容就是css代码
	* 如：
```html
<style>
	div{
		color:blue;
	}
</style>

<div>hello css</div>
```

3. `外部样式`
	1. 定义css资源文件。
	2. 在head标签内，定义link标签，引入外部的资源文件
	* 如：a.css文件：
```css
div{
	color:green;
}
```
```html
<link rel="stylesheet" href="css/a.css">
<div>hello css</div>
<div>hello css</div>
```

* `注意：`
	* 1,2,3种方式 css作用范围越来越大
	* 1方式不常用，后期常用2,3
	* 3种格式可以写为：
```html
<style>
	@import "css/a.css";
</style>
```

## 4 CSS的语法

- `格式：`
```css
选择器 {
	属性名1:属性值1;
	属性名2:属性值2;
	...
}
```

* 选择器:筛选`具有相似特征的元素`
	* 注意：
		* 每一对属性需要使用；隔开，最后一对属性可以不加；

## 5 选择器

- 筛选具有相似特征的元素(标签、id、类和联合)

1. `基础选择器`
	1. `id选择器`：选择具体的id属性值的元素.建议在一个html页面中id值唯一
		* 语法：`#id属性值{}`
	2. `元素选择器`：选择具有相同标签名称的元素
		* 语法： `标签名称{}`
		* 注意：id选择器优先级高于元素选择器
	3. `类选择器`：选择具有相同的class属性值的元素。
		* 语法：`.class属性值{}
		* 注意：类选择器选择器优`先级高于元素选择器
2. `扩展选择器`：
	1. 选择所有元素：
		* 语法： \*{}
	2. `并集选择器`：
		* 选择器1,选择器2{}
	
	3. `子选择器`：筛选选择器1元素下的选择器2元素
		* 语法：  选择器1 选择器2{}
	4. `父选择器`：筛选选择器2的父元素选择器1
		* 语法：  选择器1 > 选择器2{}

	5. 属性选择器：选择元素名称，属性名=属性值的元素
		* 语法：  元素名称[属性名="属性值"]{}

	6. 伪类选择器：选择一些元素具有的状态
		* 语法： 元素:状态{}
		* 如： \<a>
			* 状态：
				* link：初始化的状态
				* visited：被访问过的状态
				* active：正在访问状态
				* hover：鼠标悬浮状态

## 6 属性

1. `字体、文本`
	* font-size：字体大小
	* color：文本颜色
	* text-align：对其方式
	* line-height：行高 
2. `背景`
	* background：复合属性，可以设置多个值
3. `边框`
	* border：设置边框，符合属性
4. `尺寸`
	* width：宽度
	* height：高度
5. `盒子模型`：控制布局
	* margin：外边距
	* padding：内边距
		* `默认情况下内边距会影响整个盒子的大小`
		* box-sizing: border-box;  设置盒子的属性，让width和height就是最终盒子的大小
	* float：浮动
		* left
		* right

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/500fedd867e4768af69c08632ed1cb4e.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b17e0d225c4056eb9dd61164e29681f0.png)



## 7 案例


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35e85e2c01ce48f7a77381b351c79500.png)




## 1 概念

- Cascading Style Sheet 层叠样式表
- 层叠：多个样式可以作用在同一个html的元素(标签)上，同时生效

## 2 好处

1. 功能强大

2. 将内容展示和样式控制分离
	- 降低耦合度，解耦
	- 让分工协作更容易
	- 提高开发效率

## 3 CSS的使用

- CSS与HTML结合方式

1. `内联样式`
	 * 在标签内使用style属性指定css代码
	 * 如：
```html
<div style="color:red;">hello css</div>
```

2. `内部样式`
	* 在head标签内，定义style标签，style标签的标签体内容就是css代码
	* 如：
```html
<style>
	div{
		color:blue;
	}
</style>

<div>hello css</div>
```

3. `外部样式`
	1. 定义css资源文件。
	2. 在head标签内，定义link标签，引入外部的资源文件
	* 如：a.css文件：
```css
div{
	color:green;
}
```
```html
<link rel="stylesheet" href="css/a.css">
<div>hello css</div>
<div>hello css</div>
```

* `注意：`
	* 1,2,3种方式 css作用范围越来越大
	* 1方式不常用，后期常用2,3
	* 3种格式可以写为：
```html
<style>
	@import "css/a.css";
</style>
```

## 4 CSS的语法

- `格式：`
```css
选择器 {
	属性名1:属性值1;
	属性名2:属性值2;
	...
}
```

* 选择器:筛选`具有相似特征的元素`
	* 注意：
		* 每一对属性需要使用；隔开，最后一对属性可以不加；

## 5 选择器

- 筛选具有相似特征的元素(标签、id、类和联合)

1. `基础选择器`
	1. `id选择器`：选择具体的id属性值的元素.建议在一个html页面中id值唯一
		* 语法：`#id属性值{}`
	2. `元素选择器`：选择具有相同标签名称的元素
		* 语法： `标签名称{}`
		* 注意：id选择器优先级高于元素选择器
	3. `类选择器`：选择具有相同的class属性值的元素。
		* 语法：`.class属性值{}
		* 注意：类选择器选择器优`先级高于元素选择器
2. `扩展选择器`：
	1. 选择所有元素：
		* 语法： \*{}
	2. `并集选择器`：
		* 选择器1,选择器2{}
	
	3. `子选择器`：筛选选择器1元素下的选择器2元素
		* 语法：  选择器1 选择器2{}
	4. `父选择器`：筛选选择器2的父元素选择器1
		* 语法：  选择器1 > 选择器2{}

	5. 属性选择器：选择元素名称，属性名=属性值的元素
		* 语法：  元素名称[属性名="属性值"]{}

	6. 伪类选择器：选择一些元素具有的状态
		* 语法： 元素:状态{}
		* 如： \<a>
			* 状态：
				* link：初始化的状态
				* visited：被访问过的状态
				* active：正在访问状态
				* hover：鼠标悬浮状态

## 6 属性

1. `字体、文本`
	* font-size：字体大小
	* color：文本颜色
	* text-align：对其方式
	* line-height：行高 
2. `背景`
	* background：复合属性，可以设置多个值
3. `边框`
	* border：设置边框，符合属性
4. `尺寸`
	* width：宽度
	* height：高度
5. `盒子模型`：控制布局
	* margin：外边距
	* padding：内边距
		* `默认情况下内边距会影响整个盒子的大小`
		* box-sizing: border-box;  设置盒子的属性，让width和height就是最终盒子的大小
	* float：浮动
		* left
		* right

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/500fedd867e4768af69c08632ed1cb4e.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b17e0d225c4056eb9dd61164e29681f0.png)



## 7 案例


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35e85e2c01ce48f7a77381b351c79500.png)



