## 目录

- [1 JQuery 基础](#1%20JQuery%20%E5%9F%BA%E7%A1%80)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 快速入门](#1.2%20%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)
		- [1.2.1 步骤](#1.2.1%20%E6%AD%A5%E9%AA%A4)
	- [1.3 JQuery对象和JS对象区别与转换](#1.3%20JQuery%E5%AF%B9%E8%B1%A1%E5%92%8CJS%E5%AF%B9%E8%B1%A1%E5%8C%BA%E5%88%AB%E4%B8%8E%E8%BD%AC%E6%8D%A2)
	- [1.4 选择器：筛选具有相似特征的元素(标签)](#1.4%20%E9%80%89%E6%8B%A9%E5%99%A8%EF%BC%9A%E7%AD%9B%E9%80%89%E5%85%B7%E6%9C%89%E7%9B%B8%E4%BC%BC%E7%89%B9%E5%BE%81%E7%9A%84%E5%85%83%E7%B4%A0(%E6%A0%87%E7%AD%BE))
		- [1.4.1 基本操作](#1.4.1%20%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C)
		- [1.4.2 分类](#1.4.2%20%E5%88%86%E7%B1%BB)
	- [1.5 DOM操作](#1.5%20DOM%E6%93%8D%E4%BD%9C)
		- [1.5.1 内容操作](#1.5.1%20%E5%86%85%E5%AE%B9%E6%93%8D%E4%BD%9C)
		- [1.5.2 属性操作](#1.5.2%20%E5%B1%9E%E6%80%A7%E6%93%8D%E4%BD%9C)
		- [1.5.3 CRUD操作](#1.5.3%20CRUD%E6%93%8D%E4%BD%9C)
	- [1.6 案例](#1.6%20%E6%A1%88%E4%BE%8B)
		- [1.6.1 隔行换色和全选](#1.6.1%20%E9%9A%94%E8%A1%8C%E6%8D%A2%E8%89%B2%E5%92%8C%E5%85%A8%E9%80%89)
		- [1.6.2 QQ表情选择](#1.6.2%20QQ%E8%A1%A8%E6%83%85%E9%80%89%E6%8B%A9)
		- [1.6.3 左右移动](#1.6.3%20%E5%B7%A6%E5%8F%B3%E7%A7%BB%E5%8A%A8)
- [2 JQuery高级](#2%20JQuery%E9%AB%98%E7%BA%A7)
	- [2.1 动画](#2.1%20%E5%8A%A8%E7%94%BB)
		- [2.1.1 默认显示和隐藏方式](#2.1.1%20%E9%BB%98%E8%AE%A4%E6%98%BE%E7%A4%BA%E5%92%8C%E9%9A%90%E8%97%8F%E6%96%B9%E5%BC%8F)
		- [2.1.2 滑动显示和隐藏方式](#2.1.2%20%E6%BB%91%E5%8A%A8%E6%98%BE%E7%A4%BA%E5%92%8C%E9%9A%90%E8%97%8F%E6%96%B9%E5%BC%8F)
		- [2.1.3 淡入淡出显示和隐藏方式](#2.1.3%20%E6%B7%A1%E5%85%A5%E6%B7%A1%E5%87%BA%E6%98%BE%E7%A4%BA%E5%92%8C%E9%9A%90%E8%97%8F%E6%96%B9%E5%BC%8F)
	- [2.2 遍历](#2.2%20%E9%81%8D%E5%8E%86)
		- [2.2.1 js的遍历方式](#2.2.1%20js%E7%9A%84%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
		- [2.2.2 jq的遍历方式](#2.2.2%20jq%E7%9A%84%E9%81%8D%E5%8E%86%E6%96%B9%E5%BC%8F)
	- [2.3 事件绑定](#2.3%20%E4%BA%8B%E4%BB%B6%E7%BB%91%E5%AE%9A)
	- [2.4 案例](#2.4%20%E6%A1%88%E4%BE%8B)
		- [2.4.1 广告的自动显示和隐藏](#2.4.1%20%E5%B9%BF%E5%91%8A%E7%9A%84%E8%87%AA%E5%8A%A8%E6%98%BE%E7%A4%BA%E5%92%8C%E9%9A%90%E8%97%8F)
		- [2.4.2 抽奖](#2.4.2%20%E6%8A%BD%E5%A5%96)
	- [2.5 插件：增强JQuery的功能](#2.5%20%E6%8F%92%E4%BB%B6%EF%BC%9A%E5%A2%9E%E5%BC%BAJQuery%E7%9A%84%E5%8A%9F%E8%83%BD)


`技术过旧，建议学习Vue前端开发`
## 1 JQuery 基础

### 1.1 概念

- 一个JavaScript框架。简化JS开发

- jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。

* JavaScript框架：本质上就是一些js文件，封装了js的原生代码而已


### 1.2 快速入门

#### 1.2.1 步骤

1. 下载JQuery
	* 目前jQuery有三个大版本：
		`1.x`：兼容ie678,使用最为广泛的，官方只做BUG维护，
			 功能不再新增。因此`一般项目来说，使用1.x版本就可以了`，
			 最终版本：1.12.4 (2016年5月20日)
		`2.x`：不兼容ie678，很少有人使用，官方只做BUG维护，
			 功能不再新增。如果不考虑兼容低版本的浏览器可以使用2.x，
			 最终版本：2.2.4 (2016年5月20日)
		`3.x`：不兼容ie678，只支持最新的浏览器。除非特殊要求，
			 一般不会使用3.x版本的，很多老的jQuery插件不支持这个版本。
			 目前该版本是官方主要更新维护的版本。最新版本：3.2.1（2017年3月20日）
	* jquery-xxx.js 与 jquery-xxx.min.js区别：
		1. jquery-xxx.js：`开发版本`。给程序员看的，有良好的缩进和注释。体积大一些
		2. jquery-xxx.min.js：`生产版本`。程序中使用，没有缩进。体积小一些。程序加载更快

2. 导入JQuery的js文件：导入min.js文件
3. 使用
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>JQuery快速入门</title>  
    <script src="js/jquery-3.3.1.min.js"></script>  
</head>  
<body>  
    <div id="div1">div1......</div>  
    <div id="div2">div2......</div>  
  
<script>  
    //使用JQuery获取元素对象  
    var div1 = $("#div1");  
    alert(div1.html());  
  
    var div2 = $("#div2");  
    alert(div2.html());  
</script>  
</body>  
</html>
```


### 1.3 JQuery对象和JS对象区别与转换

1. JQuery对象在操作时，更加方便。
2. JQuery对象和js对象方法不通用的.
3. 两者相互转换
	* jq -- > js : jq对象[索引] 或者 jq对象.get(索引)
	* js -- > jq : $(js对象)

- 代码案例：
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>JQuery对象和JS对象的转换</title>  
    <script src="js/jquery-3.3.1.min.js"></script>  
</head>  
<body>  
    <div id="div1">div1......</div>  
    <div id="div2">div2......</div>  
  
<script>  
    //1. 通过js方式来获取名称叫div的所有html元素对象  
    var divs = document.getElementsByTagName("div");  
    alert(divs);    //object HTMLCollection 可以将其当做数组来使用  
    //对divs中所有的div 让其标签体内容变为“aaa”  
    for (var i = 0; i < divs.length; i++) {  
        // divs[i].innerHTML = "aaa";  
        // $(divs[i]).html("ccc");    }  
  
    //2. 通过JQ方式来获取名称叫div的所有html元素对象  
    var $divs = $("div");  
    alert($divs);   //object Object 也可以当做数组来使用  
    //对divs中所有的div 让其标签体内容变为“aaa”  
    $divs.html("bbb");  
  
    $divs[0].innerHTML = "ddd";  
    $divs.get(1).innerHTML = "eee";  
  
  
    /*  
    1. Jquery对象在操作时，更加方便  
    2. JQuery对象和js对象方法不通用的  
    3. 两者相互转换  
        jq ---> js : jq对象[索引] 或者 jq对象.get(索引)  
        js ---> jq : ${js对象}  
     */</script>  
</body>  
</html>
```

### 1.4 选择器：筛选具有相似特征的元素(标签)

#### 1.4.1 基本操作
1. `事件绑定`
	//1.获取b1按钮
	$("#b1").click(function(){
		alert("abc");
	});
2. `入口函数`
	 $(function () {
	   
	 });
	 window.onload  和 $(function) 区别
		 * window.onload 只能定义一次,如果定义多次，后边的会将前边的覆盖掉
		 * $(function)可以定义多次的。
3. `样式控制`：css方法
	 // $("#div1").css("background-color","red");
	$("#div1").css("backgroundColor","pink");

#### 1.4.2 分类

1. 基本选择器
	1. `标签选择器`（元素选择器）
		* 语法： $("html标签名") 获得所有匹配标签名称的元素
	2. `id选择器` 
		* 语法： $("#id的属性值") 获得与指定id属性值匹配的元素
	3. `类选择器`
		* 语法： $(".class的属性值") 获得与指定的class属性值匹配的元素
	4. `并集选择器`：
		* 语法： $("选择器1,选择器2....") 获取多个选择器选中的所有元素
2. 层级选择器
	1. `后代选择器`
		* 语法： $("A B ") 选择A元素内部的所有B元素		
	2. `子选择器`
		* 语法： $("A > B") 选择A元素内部的所有B子元素
		* ![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075217762.png)


3. 属性选择器
	1. 属性名称选择器 
		* 语法： $("A[属性名]") 包含指定属性的选择器
	2. 属性选择器
		* 语法： $("A[属性名='值']") 包含指定属性等于指定值的选择器
	3. 复合属性选择器
		* 语法： $("A[属性名='值'][]...") 包含多个属性条件的选择器
4. 过滤选择器
	1. 首元素选择器 
		* 语法： :first 获得选择的元素中的第一个元素
	2. 尾元素选择器 
		* 语法： :last 获得选择的元素中的最后一个元素
	3. 非元素选择器
		* 语法： :not(selector) 不包括指定内容的元素
	4. 偶数选择器
		* 语法： :even 偶数，从 0 开始计数
	5. 奇数选择器
		* 语法： :odd 奇数，从 0 开始计数
	6. 等于索引选择器
		* 语法： :eq(index) 指定索引元素
	7. 大于索引选择器 
		* 语法： :gt(index) 大于指定索引元素
	8. 小于索引选择器 
		* 语法： :lt(index) 小于指定索引元素
	9. `标题选择器`
		* 语法： :header 获得标题（h1~h6）元素，固定写法
5. `表单过滤选择器`
	1. `可用元素选择器` 
		* 语法： :enabled 获得可用元素
	2. `不可用元素选择器` 
		* 语法： :disabled 获得不可用元素
	3. `选中选择器` 
		* 语法： :checked 获得单选/复选框选中的元素
	4. `选中选择器` 
		* 语法： :selected 获得下拉框选中的元素

### 1.5 DOM操作

#### 1.5.1 内容操作

1. html(): 获取/设置元素的标签体内容   ``<a><font>内容</font></a>``  --> `<font>内容</font>`
2. text(): 获取/设置元素的标签体纯文本内容   `<a><font>内容</font></a>` --> 内容
3. val()： 获取/设置元素的value属性值


#### 1.5.2 属性操作

1. `通用属性操作`
	1. attr(): 获取/设置元素的属性
	2. removeAttr():删除属性
	3. prop():获取/设置元素的属性
	4. removeProp():删除属性

	* attr和prop区别？
		1. 如果操作的是元素的固有属性，则建议使用prop
		2. 如果操作的是元素自定义的属性，则建议使用attr
2. `对class属性操作`
	1. addClass():添加class属性值
	2. removeClass():删除class属性值
	3. toggleClass():切换class属性
		* toggleClass("one"): 
			* 判断如果元素对象上`存在class="one"，则将属性值one删除掉`。  如果元素对象上`不存在class="one"，则添加`
	4. css():

#### 1.5.3 CRUD操作

1. `append()`:父元素将子元素追`加到末尾`
	* 对象1.append(对象2): 将对象2添加到对象1元素内部，并且在末尾
2. `prepend()`:父元素将子元素`追加到开头`
	* 对象1.prepend(对象2):将对象2添加到对象1元素内部，并且在开头
3. `appendTo()`:
	* 对象1.appendTo(对象2):将对象1添加到对象2内部，并且在末尾
4. `prependTo()`：
	* 对象1.prependTo(对象2):将对象1添加到对象2内部，并且在开头
 
 5. `after()`:添加元素到元素后边
        * 对象1.after(对象2)： 将对象2添加到对象1后边。对象1和对象2是兄弟关系
6. `before()`:添加元素到元素前边
	* 对象1.before(对象2)： 将对象2添加到对象1前边。对象1和对象2是兄弟关系
7. `insertAfter()`
	* 对象1.insertAfter(对象2)：将对象2添加到对象1后边。对象1和对象2是兄弟关系
8. `insertBefore()`
	* 对象1.insertBefore(对象2)： 将对象2添加到对象1前边。对象1和对象2是兄弟关系

9. `remove()`:移除元素
	* 对象.remove():将对象删除掉
10. `empty()`:清空元素的所有后代元素。
	* 对象.empty():将对象的后代元素全部清空，但是保留当前对象以及其属性节点

### 1.6 案例

#### 1.6.1 隔行换色和全选

```html
<!DOCTYPE html>  
<html>  
   <head>  
      <meta charset="UTF-8">  
      <title></title>      <script  src="../../js/jquery-3.3.1.min.js"></script>  
      <script>         
      //需求：将数据行的奇数行背景色设置为 pink，偶数行背景色设置为 yellow         
      $(function () {  
            $("table tr:gt(1):even").css("backgroundColor","yellow");  
            $("table tr:gt(1):odd").css("backgroundColor","pink");  
         })  
         //需求：保证下边的选中状态和第一个复选框的选中状态一致即可  
         function selectAll(obj) {  
            //获取下边的复选框  
            $(".itemSelect").prop("checked",obj.checked);  
         }  
      </script>  
   </head>  
   <body>      <table id="tab1" border="1" width="800" align="center" >  
         <tr>  
            <td colspan="5"><input type="button" value="删除"></td>  
         </tr>  
         <tr>            <th><input type="checkbox" onclick="selectAll(this)" ></th>  
            <th>分类ID</th>  
            <th>分类名称</th>  
            <th>分类描述</th>  
            <th>操作</th>  
         </tr>  
         <tr>            <td><input type="checkbox" class="itemSelect"></td>  
            <td>1</td>  
            <td>手机数码</td>  
            <td>手机数码类商品</td>  
            <td><a href="">修改</a>|<a href="">删除</a></td>  
         </tr>  
         <tr>            <td><input type="checkbox" class="itemSelect"></td>  
            <td>2</td>  
            <td>电脑办公</td>  
            <td>电脑办公类商品</td>  
            <td><a href="">修改</a>|<a href="">删除</a></td>  
         </tr>  
         <tr>            <td><input type="checkbox" class="itemSelect"></td>  
            <td>3</td>  
            <td>鞋靴箱包</td>  
            <td>鞋靴箱包类商品</td>  
            <td><a href="">修改</a>|<a href="">删除</a></td>  
         </tr>  
         <tr>            <td><input type="checkbox" class="itemSelect"></td>  
            <td>4</td>  
            <td>家居饰品</td>  
            <td>家居饰品类商品</td>  
            <td><a href="">修改</a>|<a href="">删除</a></td>  
         </tr>  
      </table>  
   </body>  
</html>
```

#### 1.6.2 QQ表情选择

```html
<!DOCTYPE html>  
<html>  
<head>  
    <meta charset="UTF-8" />  
    <title>QQ表情选择</title>  
    <script  src="../../js/jquery-3.3.1.min.js"></script>  
    <style type="text/css">  
    *{margin: 0;padding: 0;list-style: none;}  
  
    .emoji{margin:50px;}  
    ul{overflow: hidden;}  
    li{float: left;width: 48px;height: 48px;cursor: pointer;}  
    .emoji img{ cursor: pointer; }  
    </style>  
   <script>        //需求：点击qq表情，将其追加到发言框中  
        $(function () {  
            //1. 给img图片添加onclick事件  
            $("ul img").click(function () {  
                //2. 追加到p标签中即可  
                $(".word").append($(this).clone())  
            })  
        })  
              
</script>  
   </head>  
<body>  
    <div class="emoji">  
        <ul>  
            <li><img src="img/01.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/02.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/03.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/04.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/05.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/06.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/07.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/08.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/09.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/10.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/11.gif" height="22" width="22" alt="" /></li>  
            <li><img src="img/12.gif" height="22" width="22" alt="" /></li>  
        </ul>  
        <p class="word">  
            <strong>请发言：</strong>  
            <img src="img/12.gif" height="22" width="22" alt="" />  
        </p>  
    </div>  
</body>  
</html>
```

#### 1.6.3 左右移动

```html
<!DOCTYPE html>  
<html>  
   <head>  
      <meta charset="UTF-8">  
      <title></title>      <script  src="../../js/jquery-3.3.1.min.js"></script>  
      <style>         #leftName , #btn,#rightName{  
            float: left;  
            width: 100px;  
            height: 300px;  
         }  
         #toRight,#toLeft{  
            margin-top:100px ;  
            margin-left:30px;  
            width: 50px;  
         }  
         .border{  
            height: 500px;  
            padding: 100px;  
         }  
      </style>  
      <script>         //需求：实现下拉列表选择条目左右选择功能  
         $(function () {  
            $("#toRight").click(function () {  
               //获取右边的下拉列表对象，append左边下拉列表选中的条目  
               $("#rightName").append($("#leftName > option:selected"))  
            })  
            $("#toLeft").click(function () {  
               $("#leftName").append($("#rightName > option:selected"))  
            })  
         })  
      </script>  
   </head>  
   <body>      <div class="border">  
         <select id="leftName" multiple="multiple">  
            <option>张三</option>  
            <option>李四</option>  
            <option>王五</option>  
            <option>赵六</option>  
         </select>  
         <div id="btn">  
            <input type="button" id="toRight" value="-->"><br>  
            <input type="button" id="toLeft" value="<--">  
         </div>  
         <select id="rightName" multiple="multiple">  
            <option>钱七</option>  
         </select>  
      </div>  
   </body>  
</html>
```

## 2 JQuery高级

### 2.1 动画

#### 2.1.1 默认显示和隐藏方式

1. show([speed,[easing],[fn]])
	1. 参数：
		1. speed：动画的速度。三个预定义的值("slow","normal", "fast")或`表示动画时长的毫秒数值`(如：1000)
		2. easing：用来指定`切换效果`，默认是"swing"，可用参数"linear"
			* swing：动画执行时效果是` 先慢，中间快，最后又慢`
			* linear：动画执行时速度是`匀速的`
		3. fn：在`动画完成时执行的函数，每个元素执行一次`。

2. hide([speed],[easing],[fn]])
3. toggle([speed],[easing],[fn])

#### 2.1.2 滑动显示和隐藏方式

1. slideDown([speed],[easing],[fn])
2. slideUp([speed,[easing],[fn]])
3. slideToggle([speed],[easing],[fn])

#### 2.1.3 淡入淡出显示和隐藏方式

1. fadeIn([speed],[easing],[fn])
2. fadeOut([speed],[easing],[fn])
3. fadeToggle([speed,[easing],[fn]])

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075228491.png)



### 2.2 遍历

#### 2.2.1 js的遍历方式

* for(初始化值;循环结束条件;步长)

#### 2.2.2 jq的遍历方式

1. `jq对象.each(callback)`
	1. 语法：
		- jquery对象.each(function(index,element){});
			* index:就是元素在集合中的索引
			* element：就是集合中的每一个元素对象
			* `this：集合中的每一个元素对象`
	2. 回调函数返回值：
		* true:如果当前function返回为false，则结束循环(break)。
		* false:如果当前function返回为true，则结束本次循环，继续下次循环(continue)
2. `$.each(object, [callback])`
3.` for..of`: jquery 3.0 版本之后提供的方式
	for(元素对象 of 容器对象)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075231416.png)



### 2.3 事件绑定

1. jquery标准的绑定方式
	* `jq对象.事件方法(回调函数)`；
	* 注：如果调用事件方法，不传递回调函数，则会触发浏览器默认行为。
		* 表单对象.submit();//让表单提交
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075236659.png)



2. `on绑定事件/off解除绑定`
	* jq对象.on("事件名称",回调函数)
	* jq对象.off("事件名称")
		* 如果off方法不传递任何参数，则将组件上的所有事件全部解绑
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075240331.png)



3. `事件切换：toggle`
	* `jq对象.toggle(fn1,fn2...)`
		* 当单击jq对象对应的组件后，会执行fn1.第二次点击会执行fn2.....
		
	* 注意：1.9版本 .toggle() 方法删除,jQuery Migrate（迁移）插件可以恢复此功能。
		 `<script src="../js/jquery-migrate-1.0.0.js" type="text/javascript" charset="utf-8"></script>`


### 2.4 案例

#### 2.4.1 广告的自动显示和隐藏

```html
<!DOCTYPE html>  
<html>  
<head>  
    <meta charset="UTF-8">  
    <title>广告的自动显示与隐藏</title>  
    <style>        #content{width:100%;height:500px;background:#999}  
    </style>  
  
    <!--引入jquery-->  
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>  
    <script>        /*  
        需求：  
            1. 当页面加载完，3s后，自动显示广告  
            2. 广告显示5s后，自动消失  
         */  
        function adShow() {  
            $("#ad").show("slow");  
        }  
        function adHide() {  
            $("#ad").hide("slow");  
        }  
        $(function () {  
            setTimeout(adShow,3000);  
            setTimeout(adHide,8000);  
        })  
    </script>  
</head>  
<body>  
<!-- 整体的DIV -->  
<div>  
    <!-- 广告DIV -->  
    <div id="ad" style="display: none;">  
        <img style="width:100%" src="../img/adv.jpg" />  
    </div>  
  
    <!-- 下方正文部分 -->  
    <div id="content">  
        正文部分  
    </div>  
</div>  
</body>  
</html>
```


#### 2.4.2 抽奖

```js
<!DOCTYPE html>  
<html>  
<head>  
    <meta charset="UTF-8">  
    <title>jquery案例之抽奖</title>  
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>  
    <script language='javascript' type='text/javascript'>  
        //准备一个一维数组，装用户的像片路径  
        var imgs = [  
            "../img/man00.jpg",  
            "../img/man01.jpg",  
            "../img/man02.jpg",  
            "../img/man03.jpg",  
            "../img/man04.jpg",  
            "../img/man05.jpg",  
            "../img/man06.jpg"  
        ];  
  
        $(function () {  
            //0. 按钮控制  
            $("#startID").prop("disabled",false);  
            $("#stopID").prop("disabled",true);  
        })  
        var index;  
        var interval;  
        //1. 小相框图片刷新  
        function imgStart() {  
            $("#startID").prop("disabled",true);  
            $("#stopID").prop("disabled",false);  
            // 1.1  随机刷新图片  
            interval = setInterval(function () {  
                // 1.2 获取随机数  
                index = Math.floor(Math.random()*7);  
                $("#img1ID").prop("src",imgs[index]);  
            },100);  
        }  
  
        //2. 大相框图片同步  
        function imgStop() {  
            $("#startID").prop("disabled",false);  
            $("#stopID").prop("disabled",true);  
            //2.1 停止刷新  
            clearInterval(interval);  
            //2.2 图片显示  
            $("#img2ID").prop("src",imgs[index]).hide();  
            //2.3 添加动画  
            $("#img2ID").show("slow");  
        }  
    </script>  
</head>  
<body>  
  
<!-- 小像框 -->  
<div style="border-style:dotted;width:160px;height:100px">  
    <img id="img1ID" src="../img/man00.jpg" style="width:160px;height:100px"/>  
</div>  
  
<!-- 大像框 -->  
<div  
        style="border-style:double;width:800px;height:500px;position:absolute;left:500px;top:10px">  
    <img id="img2ID" src="../img/man00.jpg" width="800px" height="500px"/>  
</div>  
  
<!-- 开始按钮 -->  
<input  
        id="startID"  
        type="button"  
        value="点击开始"  
        style="width:150px;height:150px;font-size:22px"  
        onclick="imgStart()">  
  
<!-- 停止按钮 -->  
<input  
        id="stopID"  
        type="button"  
        value="点击停止"  
        style="width:150px;height:150px;font-size:22px"  
        onclick="imgStop()">  
  
</body>  
</html>
```



### 2.5 插件：增强JQuery的功能

- `等同于自定义方法`

1. $.fn.extend(object) 
	* 增强通过Jquery获取的对象的功能  $("#id")
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075246719.png)



2. $.extend(object)
	* 增强JQeury对象自身的功能  $/jQuery
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075249707.png)

