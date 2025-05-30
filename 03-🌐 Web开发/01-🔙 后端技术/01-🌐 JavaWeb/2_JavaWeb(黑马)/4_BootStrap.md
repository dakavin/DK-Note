---
文章标题: "[[4_BootStrap]]" 
文章作者: Dakkk
文章概要: |
  介绍Bootstrap前端框架的基本概念、快速入门方法、响应式布局栅格系统原理，以及CSS样式和JS插件的常用功能。
tags:
- "Bootstrap"
- "前端框架"
- "响应式布局"
- "栅格系统"
- "CSS"
- "JavaScript"
- "HTML"
相关文章:
- "[[8_首页样式布局设计（3）— 侧边栏分类、标签卡片]]"
- "[[2_CSS]]"
- "[[2_HTML和CSS]]"
- "[[1_登录页设计：支持响应式布局]]"
- "[[1_Obsidian小白也能用？AI自动生成笔记元数据 (Templater基础演示)]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/4_BootStrap.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:05:21
---

## 1 概念

- `一个前端开发的框架`，Bootstrap，来自 Twitter，是目前很受欢迎的前端框架。`Bootstrap 是基于 HTML、CSS、JavaScript 的`，它简洁灵活，使得 Web 开发更加快捷。

- 框架:`一个半成品软件`，开发人员可以在框架基础上，在进行开发，简化编码。

- 好处：
	1. 定义了很多的css样式和js插件。我们开发人员直接可以使用这些样式和插件得到丰富的页面效果。
	2. 响应式布局。
		- 同一套页面可以兼容不同分辨率的设备。

## 2 快速入门

1. 下载Bootstrap
2. 在项目中将这三个文件夹复制
3. 创建html页面，引入必要的资源文件

- `bootstrap-3.3.7-dist`
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
	<title>Bootstrap HelloWorld</title>

	<!-- Bootstrap -->
	<link href="css/bootstrap.min.css" rel="stylesheet">
	<!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
	<script src="js/jquery-3.2.1.min.js"></script>
	<!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
	<script src="js/bootstrap.min.js"></script>
</head>
<body>
<h1>你好，世界！</h1>

</body>
</html>
```

- `bootstrap-4.6.2-dist`
```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <!-- 必须的 meta 标签 -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap 的 CSS 文件 -->
    <link rel="stylesheet" href="css/bootstrap.min.css" integrity="sha384-xOolHFLEh07PJGoPkLv1IbcEPTNtaed2xpHsD9ESMhqIYd0nLMwNLD69Npy4HI+N" crossorigin="anonymous">

    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>

    <!-- JavaScript 文件是可选的。从以下两种建议中选择一个即可！ -->

    <!-- 选项 1：jQuery 和 Bootstrap 集成包（集成了 Popper） -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-7ymO4nGrkm372HoSbq1OY2DP4pEZnMiA+E0F3zPr+JQQtQ82gQ1HPY3QIVtztVua" crossorigin="anonymous"></script>

    <!-- 选项 2：Popper 和 Bootstrap 的 JS 插件各自独立 -->
    <!--
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.min.js" integrity="sha384-Lge2E2XotzMiwH69/MXB72yLpwyENMiOKX8zS8Qo7LDCvaBIWGL+GlRQEKIpYR04" crossorigin="anonymous"></script>
    -->
  </body>
</html>
```

## 3 响应式布局

* 同一套页面可以兼容不同分辨率的设备。
* 实现：`依赖于栅格系统`
	* 将一行平均分成12个格子，可以指定元素占几个格子
* 步骤：
	1. `定义容器`。相当于之前的table、
		* 容器分类：
			1. container：`两边留白`
			2. container-fluid：`每一种设备都是100%宽度`
	2. `定义行`。相当于之前的tr   样式：row
	3. `定义元素`。指定该元素在不同的设备上，所占的格子数目。样式：col-设备代号-格子数目
		* 设备代号：
			1. xs：超小屏幕 手机 (<768px)：`col-xs-12`
			2. sm：小屏幕 平板 (≥768px)
			3. md：中等屏幕 桌面显示器 (≥992px)
			4. lg：大屏幕 大桌面显示器 (≥1200px)

	* 注意：
		1. 一行中如果格子数目超过12，则`超出部分自动换行`。
		2. `栅格类属性可以向上兼容`。栅格类适用于与屏幕宽度大于或等于分界点大小的设备。
		3. 如果真实设备宽度小于了设置栅格类属性的设备代码的最小值，`会一个元素沾满一整行`。

## 4 CSS样式和JS插件

1. 全局CSS样式：
	* 按钮：class="btn btn-default"
	* 图片：
		*  class="img-responsive"：`图片在任意尺寸都占100%`
		*  图片形状
			* ` <img src="..." alt="..." class="img-rounded">`：方形
			*  `<img src="..." alt="..." class="img-circle">` ： 圆形
			* ` <img src="..." alt="..." class="img-thumbnail">` ：相框
	* 表格
		* table
		* table-bordered
		* table-hover
	* 表单
		* 给表单项添加：class="form-control" 
2. 组件：
	* 导航条
	* 分页条
3. 插件：
	* 轮播图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/334229293e1ffda194336da685449c13.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5a06c426f54f3274e2fa58c05a5b880a.png)





<h2>一句话，查文档，复制后修改，哪里不懂删哪里</h2>


## 1 概念

- `一个前端开发的框架`，Bootstrap，来自 Twitter，是目前很受欢迎的前端框架。`Bootstrap 是基于 HTML、CSS、JavaScript 的`，它简洁灵活，使得 Web 开发更加快捷。

- 框架:`一个半成品软件`，开发人员可以在框架基础上，在进行开发，简化编码。

- 好处：
	1. 定义了很多的css样式和js插件。我们开发人员直接可以使用这些样式和插件得到丰富的页面效果。
	2. 响应式布局。
		- 同一套页面可以兼容不同分辨率的设备。

## 2 快速入门

1. 下载Bootstrap
2. 在项目中将这三个文件夹复制
3. 创建html页面，引入必要的资源文件

- `bootstrap-3.3.7-dist`
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
	<title>Bootstrap HelloWorld</title>

	<!-- Bootstrap -->
	<link href="css/bootstrap.min.css" rel="stylesheet">
	<!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
	<script src="js/jquery-3.2.1.min.js"></script>
	<!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
	<script src="js/bootstrap.min.js"></script>
</head>
<body>
<h1>你好，世界！</h1>

</body>
</html>
```

- `bootstrap-4.6.2-dist`
```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <!-- 必须的 meta 标签 -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap 的 CSS 文件 -->
    <link rel="stylesheet" href="css/bootstrap.min.css" integrity="sha384-xOolHFLEh07PJGoPkLv1IbcEPTNtaed2xpHsD9ESMhqIYd0nLMwNLD69Npy4HI+N" crossorigin="anonymous">

    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>

    <!-- JavaScript 文件是可选的。从以下两种建议中选择一个即可！ -->

    <!-- 选项 1：jQuery 和 Bootstrap 集成包（集成了 Popper） -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-7ymO4nGrkm372HoSbq1OY2DP4pEZnMiA+E0F3zPr+JQQtQ82gQ1HPY3QIVtztVua" crossorigin="anonymous"></script>

    <!-- 选项 2：Popper 和 Bootstrap 的 JS 插件各自独立 -->
    <!--
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.min.js" integrity="sha384-Lge2E2XotzMiwH69/MXB72yLpwyENMiOKX8zS8Qo7LDCvaBIWGL+GlRQEKIpYR04" crossorigin="anonymous"></script>
    -->
  </body>
</html>
```

## 3 响应式布局

* 同一套页面可以兼容不同分辨率的设备。
* 实现：`依赖于栅格系统`
	* 将一行平均分成12个格子，可以指定元素占几个格子
* 步骤：
	1. `定义容器`。相当于之前的table、
		* 容器分类：
			1. container：`两边留白`
			2. container-fluid：`每一种设备都是100%宽度`
	2. `定义行`。相当于之前的tr   样式：row
	3. `定义元素`。指定该元素在不同的设备上，所占的格子数目。样式：col-设备代号-格子数目
		* 设备代号：
			1. xs：超小屏幕 手机 (<768px)：`col-xs-12`
			2. sm：小屏幕 平板 (≥768px)
			3. md：中等屏幕 桌面显示器 (≥992px)
			4. lg：大屏幕 大桌面显示器 (≥1200px)

	* 注意：
		1. 一行中如果格子数目超过12，则`超出部分自动换行`。
		2. `栅格类属性可以向上兼容`。栅格类适用于与屏幕宽度大于或等于分界点大小的设备。
		3. 如果真实设备宽度小于了设置栅格类属性的设备代码的最小值，`会一个元素沾满一整行`。

## 4 CSS样式和JS插件

1. 全局CSS样式：
	* 按钮：class="btn btn-default"
	* 图片：
		*  class="img-responsive"：`图片在任意尺寸都占100%`
		*  图片形状
			* ` <img src="..." alt="..." class="img-rounded">`：方形
			*  `<img src="..." alt="..." class="img-circle">` ： 圆形
			* ` <img src="..." alt="..." class="img-thumbnail">` ：相框
	* 表格
		* table
		* table-bordered
		* table-hover
	* 表单
		* 给表单项添加：class="form-control" 
2. 组件：
	* 导航条
	* 分页条
3. 插件：
	* 轮播图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/334229293e1ffda194336da685449c13.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5a06c426f54f3274e2fa58c05a5b880a.png)





<h2>一句话，查文档，复制后修改，哪里不懂删哪里</h2>

