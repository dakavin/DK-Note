## 1、超链接概念

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

## 2、href属性探讨

- 跳转页面的情况
	- 情况1：填写外部服务器地址
	- 情况2：填写本地服务器地址
		- 若是Web包下的html页面，注意`相对路径和绝对路径`
		- 若是Servlet的实现类，要写上`虚拟目录`

- 特殊情况
	- href = “javascript:;”  或  href = "javascript:void(0);"
	- 表示不跳转其他页面，只在本页面点击链接，实现链接功能

	- href = "#"   或 href = “”
	- 表示以当前页面为锚点，实现刷新页面的功能

	- href=”#top"
	- 表示回到顶部,如果当前页面需要滚动的话,就可以通过这种方式直接回到顶部
