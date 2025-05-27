---
文章标题: "[[17_Ajax]]" 
文章作者: Dakkk
文章概要: |
  介绍Ajax异步通信技术，包括原生Ajax和Axios框架的使用方法，展示前后端数据交互和局部页面更新的实现方式。
tags:
- "Ajax"
- "JavaScript"
- "Axios"
- "异步通信"
- "前后端交互"
- "XMLHttpRequest"
- "Vue.js"
- "HTTP请求"
相关文章:
- "[[10_文章分类：删除功能开发]]"
- "[[13_Ajax&JSON]]"
- "[[10_博客设置页：更新设置]]"
- "[[11_标签-文章列表页开发]]"
- "[[4_前台归档页：分页列表功能开发]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/2_JavaWeb(黑马)/17_Ajax.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:05:46
---

## 1 Ajax

### 1.1 Ajax介绍

我们`前端页面中的数据`，如下图所示的表格中的学生信息，`应该来自于后台`，那么我们的后台和前端是互不影响的2个程序，那么我们前端应该如何从后台获取数据呢？因为是2个程序，所以必须涉及到2个程序的交互，所以这就需要用到我们接下来学习的`Ajax技术`。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0f6cb843dde8e288babc59214fe27a7d.png)

Ajax: 全称`Asynchronous JavaScript And XML`，异步的JavaScript和XML。其作用有如下2点：

- `与服务器进行数据交换`：通过Ajax可以给服务器发送请求，并获取服务器响应的数据。
- `异步交互`：可以在**不重新加载整个页面**的情况下，与服务器交换数据并**更新部分网页**的技术，如：搜索联想、用户名是否可用的校验等等。

### 1.2 Ajax的作用

我们详细的解释一下Ajax技术的2个作用

- `与服务器进行数据交互`
    
    如下图所示前端资源被浏览器解析，但是前端页面上缺少数据，`前端可以通过Ajax技术，向后台服务器发起请求`，后台服务器接受到前端的请求`，从数据库中获取前端需要的资源，然后响应给前端`，前端在通过我们学习的vue技术，可以将数据展示到页面上，这样用户就能看到完整的页面了。此处可以对比JavaSE中的网络编程技术来理解。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1478bd2c5b10db3902848da91ea311f7.png)

- `异步交互`：可以在**不重新加载整个页面**的情况下，与服务器交换数据并**更新部分网页**的技术。

  如下图所示，当我们再百度搜索java时，下面的`联想数据是通过Ajax请求从后台服务器得到的`，在整个过程中，我们的Ajax请求不会导致整个百度页面的重新加载，并且`只针对搜索栏这局部模块的数据进行了数据的更新`，不会对整个页面的其他地方进行数据的更新，这样就`大大提升了页面的加载速度，用户体验高`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9c8182429cd46012b107552c0eaa8f90.png)
### 1.3 同步异步

针对于上述Ajax的`局部刷新功能`是因为`Ajax请求是异步的`，与之对应的有同步请求。接下来我们介绍一下异步请求和同步请求的区别。

- 同步请求发送过程如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3eee530c76cba726b219ea41081ef642.png)
  浏览器页面在发送请求给服务器，`在服务器处理请求的过程中，浏览器页面不能做其他的操作`。只能等到服务器响应结束后才能，浏览器页面才能继续做其他的操作。 

- 异步请求发送过程如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c577c087270747d00d18b684c2ebcca0.png)

  浏览器页面发送请求给服务器，在`服务器处理请求的过程中，浏览器页面还可以做其他的操作`。

## 2 原生Ajax

对于Ajax技术有了充分的认知了，我们接下来`通过代码来演示Ajax的效果`。此处我们先采用`原生的Ajax代码`来演示。因为Ajax请求是基于客户端发送请求，服务器响应数据的技术。所以为了完成快速入门案例，我们需要提供服服务器端和编写客户端。

- `服务器端`

  因为我们暂时还没学过服务器端的代码，所以此处已经直接提供好了服务器端的请求地址，我们前端直接通过Ajax请求访问该地址即可。

- `客户端`

- 客户端的Ajax请求代码如下有如下4步，接下来我们跟着步骤一起操作一下。
	1. 第一步：首先我们再VS Code中创建AJAX的文件夹，并且创建名为01. Ajax-原生方式.html的文件，提供如下代码，主要是按钮绑定单击事件，我们希望点击按钮，来发送ajax请求
	2. 第二步：创建XMLHttpRequest对象，用于和服务器交换数据，也是原生Ajax请求的核心对象，提供了各种方法。代码如下：
	3. 第三步：调用对象的open()方法设置请求的参数信息，例如请求地址，请求方式。然后调用send()方法向服务器发送请求，代码如下：
	4. 第四步：我们通过绑定事件的方式，来获取服务器响应的数据。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>原生Ajax</title>
</head>
<body>
    <input type="button" value="获取数据" onclick="getData()">
    <div id="div1"></div>
</body>
<script>
    function getData(){
        //1. 创建XMLHttpRequest
        var xmlHttpRequest  = new XMLHttpRequest();
        //2. 发送异步请求
        xmlHttpRequest.open('GET','http://yapi.smart-xwork.cn/mock/169327/emp/list');
        xmlHttpRequest.send();//发送请求
        //3. 获取服务响应数据
        xmlHttpRequest.onreadystatechange = function(){
            if(xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200){
                document.getElementById('div1').innerHTML = xmlHttpRequest.responseText;
            }
        }
    }
</script>
</html>
```


最后我们通过浏览器打开页面，请求点击按钮，发送Ajax请求，最终显示结果如下图所示：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/199862e599fc5422f4a52ec17821161c.png)



并且通过浏览器的f12抓包，我们点击网络中的XHR请求，发现可以抓包到我们发送的Ajax请求。XHR代表的就是异步请求

## 3 Axios

上述`原生的Ajax请求的代码编写起来还是比较繁琐的`，所以接下来我们学习一门更加简单的发送Ajax请求的技术Axios 。Axios是对原生的AJAX进行封装，简化书写。Axios官网是：[https://www.axios-http.cn]

### 3.1 Axios的基本使用

Axios的使用比较简单，主要分为2步：

- 引入Axios文件
```html
<script src="js/axios-0.18.0.js"></script>
```
    
- 使用Axios发送请求，并获取响应结果，官方提供的api很多，此处给出2种，如下
    - 发送 get 请求
```js
axios({  
	method:"get",  
	url:"http://localhost:8080/ajax-demo1/aJAXDemo1?username=zhangsan"  
}).then(function (resp){  
	alert(resp.data);  
})
```
        
- 发送 post 请求
```js
axios({  
	method:"post",  
	url:"http://localhost:8080/ajax-demo1/aJAXDemo1",  
	data:"username=zhangsan"  
}).then(function (resp){  
	alert(resp.data);  
});
```


axios()是用来发送异步请求的，小括号中`使用 js的JSON对象`传递请求相关的参数：

- `method属性`：用来设置请求方式的。取值为 get 或者 post。
- `url属性`：用来书写请求的资源路径。如果是 get 请求，需要将请求参数拼接到路径的后面，格式为： url?参数名=参数值&参数名2=参数值2。
- `data属性`：作为请求体被发送的数据。也就是说<font color="#00b050">如果是 post 请求的话，数据需要作为 data 属性的值</font>。

`then() 需要传递一个匿名函数`。我们将 then()中传递的匿名函数称为 **回调函数**，意思是该匿名函数在发送请求时不会被调用，而是在成功响应后调用的函数。而该`回调函数中的 resp 参数是对响应的数据进行封装的对象`，通过 resp.data 可以获取到响应的数据。

### 3.2 Axios快速入门

- 后端实现
    查询所有员工信息服务器
    根据员工id删除员工信息服务器
    
- 前端实现
	1. 首先在VS Code中创建js文件夹，与html同级，然后将**资料/axios-0.18.0.js** 文件拷贝到js目录下，然后创建名为02. Ajax-Axios.html的文件，
	2. 然后在html中引入axios所依赖的js文件，并且提供2个按钮，绑定单击事件，分别用于点击时发送ajax请求
	3. 然后分别使用Axios的方法，完整get请求和post请求的发送 
	4. 浏览器打开，f12抓包，然后分别点击2个按钮，查看控制台效果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3f3d43853de6f2448763ef43b0430d21.png)

`完整代码如下`：
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
	<meta charset="UTF-8">  
	<meta http-equiv="X-UA-Compatible" content="IE=edge">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0">  
	<title>Ajax-Axios</title>  
	<script src="js/axios-0.18.0.js"></script>  
</head>  
<body>  
	<input type="button" value="获取数据GET" onclick="get()"> 
	<input type="button" value="删除数据POST" onclick="post()">  
</body>  
<script>  
	function get(){  
		//通过axios发送异步请求-get  
		axios({  
			method: "get",  
			url: "http://yapi.smart-xwork.cn/mock/169327/emp/list"  
		}).then(result) => {  
			console.log(result.data);  
		})  
	}  
	function post(){  
	   // 通过axios发送异步请求-post  
		axios({  
			method: "post",  
			url: "http://yapi.smart-xwork.cn/mock/169327/emp/deleteById",  
			data: "id=1"  
		}).then(result) => {  
			console.log(result.data);  
		})  
	}  
</script>  
</html>
```

然后分别使用Axios的方法，完整get请求和post请求的发送

get请求代码如下：
```js
//通过axios发送异步请求-get
 axios({
     method: "get",
     url: "http://yapi.smart-xwork.cn/mock/169327/emp/list"
 }).then(result => {
     console.log(result.data);
 })
```

post请求代码如下：
```js
//通过axios发送异步请求-post
 axios({
     method: "post",
     url: "http://yapi.smart-xwork.cn/mock/169327/emp/deleteById",
     data: "id=1"
 }).then(result => {
     console.log(result.data);
 })
```

浏览器打开，f12抓包，然后分别点击2个按钮，查看控制台效果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/071bc01c38dc03a5375c4f1c171e9a8e.png)
### 3.3 请求方法的别名

Axios还针对不同的请求，提供了别名方式的api,具体如下：

|方法|描述|
|---|---|
|axios.get(url [, config])|发送get请求|
|axios.delete(url [, config])|发送delete请求|
|axios.post(url [, data[, config]])|发送post请求|
|axios.put(url [, data[, config]])|发送put请求|

我们目前只关注get和post请求，所以在上述的入门案例中，我们可以将get请求代码改写成如下：
```js
axios.get("http://yapi.smart-xwork.cn/mock/169327/emp/list").then(result) => {  
    console.log(result.data);  
})
```

post请求改写成如下：
```js
axios.post("http://yapi.smart-xwork.cn/mock/169327/emp/deleteById","id=1").then(result) => {  
    console.log(result.data);  
})
```

### 3.4 案例

- 需求：基于Vue及Axios完成数据的动态加载展示，如下图所示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1578d386bb116154ccd7bab0bff7100.png)
- 分析：
    
    前端首先是`一张表格`，我们`缺少数据`，而`提供数据的地址`已经有了，所以意味这我们需要`使用Ajax请求获取后台的数据`。但是Ajax请求什么时候发送呢？页面的数据应该是页面加载完成，自动发送请求，展示数据，所以我们`需要借助vue的mounted钩子函数`。那么拿到数据了，我们该怎么将数据显示表格中呢？这里就得`借助v-for指令来遍历数据`，展示数据。
    
- 步骤：
    1. 首先创建文件，提前准备基础代码，包括表格以及vue.js和axios.js文件的引入
    2. 我们需要在vue的mounted钩子函数中发送ajax请求，获取数据
    3. 拿到数据，数据需要绑定给vue的data属性
    4. 在\<tr>标签上通过v-for指令遍历数据，展示数据
- 代码实现：
	1. 首先创建文件，提前准备基础代码，包括表格以及vue.js和axios.js文件的引入
	2. 在vue的mounted钩子函数，编写Ajax请求，请求数据，代码如下： 
	3. ajax请求的数据我们应该绑定给vue的data属性，之后才能进行数据绑定到视图；并且浏览器打开后台地址，数据返回格式如下图所示：
		- 因为服务器响应的json中的data属性才是我们需要展示的信息，所以我们应该将员工列表信息赋值给vue的data属性，代码如下： 
	4. 在\<tr>标签上通过v-for指令遍历数据，展示数据，其中需要注意的是图片的值，需要使用vue的属性绑定，男女的展示需要使用条件判断，其代码如下：

`完整代码如下`：

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta http-equiv="X-UA-Compatible" content="IE=edge">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Ajax-Axios-案例</title>  
    <script src="js/axios-0.18.0.js"></script>  
    <script src="js/vue.js"></script>  
</head>  
<body>  
    <div id="app">  
        <table border="1" cellspacing="0" width="60%">  
            <tr>  
                <th>编号</th>  
                <th>姓名</th>  
                <th>图像</th>  
                <th>性别</th>  
                <th>职位</th>  
                <th>入职日期</th>  
                <th>最后操作时间</th>  
            </tr>  
​  
            <tr align="center" v-for="(emp,index) in emps">  
                <td>{{index + 1}}</td>  
                <td>{{emp.name}}</td>  
                <td>  
                    <img :src="emp.image" width="70px" height="50px">  
                </td>  
                <td>  
                    <span v-if="emp.gender == 1">男</span>  
                    <span v-if="emp.gender == 2">女</span>  
                </td>  
                <td>{{emp.job}}</td>  
                <td>{{emp.entrydate}}</td>  
                <td>{{emp.updatetime}}</td>  
            </tr>  
        </table>  
    </div>  
</body>  
<script>  
    new Vue({  
       el: "#app",  
       data: {  
         emps:[]  
       },  
       mounted () {  
          //发送异步请求,加载数据  
          axios.get("http://yapi.smart-xwork.cn/mock/169327/emp/list").then(result => {  
            console.log(result.data);  
            this.emps = result.data.data;  
          })  
       }  
    });  
</script>  
</html>
```
## 1 Ajax

### 1.1 Ajax介绍

我们`前端页面中的数据`，如下图所示的表格中的学生信息，`应该来自于后台`，那么我们的后台和前端是互不影响的2个程序，那么我们前端应该如何从后台获取数据呢？因为是2个程序，所以必须涉及到2个程序的交互，所以这就需要用到我们接下来学习的`Ajax技术`。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0f6cb843dde8e288babc59214fe27a7d.png)

Ajax: 全称`Asynchronous JavaScript And XML`，异步的JavaScript和XML。其作用有如下2点：

- `与服务器进行数据交换`：通过Ajax可以给服务器发送请求，并获取服务器响应的数据。
- `异步交互`：可以在**不重新加载整个页面**的情况下，与服务器交换数据并**更新部分网页**的技术，如：搜索联想、用户名是否可用的校验等等。

### 1.2 Ajax的作用

我们详细的解释一下Ajax技术的2个作用

- `与服务器进行数据交互`
    
    如下图所示前端资源被浏览器解析，但是前端页面上缺少数据，`前端可以通过Ajax技术，向后台服务器发起请求`，后台服务器接受到前端的请求`，从数据库中获取前端需要的资源，然后响应给前端`，前端在通过我们学习的vue技术，可以将数据展示到页面上，这样用户就能看到完整的页面了。此处可以对比JavaSE中的网络编程技术来理解。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1478bd2c5b10db3902848da91ea311f7.png)

- `异步交互`：可以在**不重新加载整个页面**的情况下，与服务器交换数据并**更新部分网页**的技术。

  如下图所示，当我们再百度搜索java时，下面的`联想数据是通过Ajax请求从后台服务器得到的`，在整个过程中，我们的Ajax请求不会导致整个百度页面的重新加载，并且`只针对搜索栏这局部模块的数据进行了数据的更新`，不会对整个页面的其他地方进行数据的更新，这样就`大大提升了页面的加载速度，用户体验高`。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9c8182429cd46012b107552c0eaa8f90.png)
### 1.3 同步异步

针对于上述Ajax的`局部刷新功能`是因为`Ajax请求是异步的`，与之对应的有同步请求。接下来我们介绍一下异步请求和同步请求的区别。

- 同步请求发送过程如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3eee530c76cba726b219ea41081ef642.png)
  浏览器页面在发送请求给服务器，`在服务器处理请求的过程中，浏览器页面不能做其他的操作`。只能等到服务器响应结束后才能，浏览器页面才能继续做其他的操作。 

- 异步请求发送过程如下图所示：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c577c087270747d00d18b684c2ebcca0.png)

  浏览器页面发送请求给服务器，在`服务器处理请求的过程中，浏览器页面还可以做其他的操作`。

## 2 原生Ajax

对于Ajax技术有了充分的认知了，我们接下来`通过代码来演示Ajax的效果`。此处我们先采用`原生的Ajax代码`来演示。因为Ajax请求是基于客户端发送请求，服务器响应数据的技术。所以为了完成快速入门案例，我们需要提供服服务器端和编写客户端。

- `服务器端`

  因为我们暂时还没学过服务器端的代码，所以此处已经直接提供好了服务器端的请求地址，我们前端直接通过Ajax请求访问该地址即可。

- `客户端`

- 客户端的Ajax请求代码如下有如下4步，接下来我们跟着步骤一起操作一下。
	1. 第一步：首先我们再VS Code中创建AJAX的文件夹，并且创建名为01. Ajax-原生方式.html的文件，提供如下代码，主要是按钮绑定单击事件，我们希望点击按钮，来发送ajax请求
	2. 第二步：创建XMLHttpRequest对象，用于和服务器交换数据，也是原生Ajax请求的核心对象，提供了各种方法。代码如下：
	3. 第三步：调用对象的open()方法设置请求的参数信息，例如请求地址，请求方式。然后调用send()方法向服务器发送请求，代码如下：
	4. 第四步：我们通过绑定事件的方式，来获取服务器响应的数据。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>原生Ajax</title>
</head>
<body>
    <input type="button" value="获取数据" onclick="getData()">
    <div id="div1"></div>
</body>
<script>
    function getData(){
        //1. 创建XMLHttpRequest
        var xmlHttpRequest  = new XMLHttpRequest();
        //2. 发送异步请求
        xmlHttpRequest.open('GET','http://yapi.smart-xwork.cn/mock/169327/emp/list');
        xmlHttpRequest.send();//发送请求
        //3. 获取服务响应数据
        xmlHttpRequest.onreadystatechange = function(){
            if(xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200){
                document.getElementById('div1').innerHTML = xmlHttpRequest.responseText;
            }
        }
    }
</script>
</html>
```


最后我们通过浏览器打开页面，请求点击按钮，发送Ajax请求，最终显示结果如下图所示：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/199862e599fc5422f4a52ec17821161c.png)



并且通过浏览器的f12抓包，我们点击网络中的XHR请求，发现可以抓包到我们发送的Ajax请求。XHR代表的就是异步请求

## 3 Axios

上述`原生的Ajax请求的代码编写起来还是比较繁琐的`，所以接下来我们学习一门更加简单的发送Ajax请求的技术Axios 。Axios是对原生的AJAX进行封装，简化书写。Axios官网是：[https://www.axios-http.cn]

### 3.1 Axios的基本使用

Axios的使用比较简单，主要分为2步：

- 引入Axios文件
```html
<script src="js/axios-0.18.0.js"></script>
```
    
- 使用Axios发送请求，并获取响应结果，官方提供的api很多，此处给出2种，如下
    - 发送 get 请求
```js
axios({  
	method:"get",  
	url:"http://localhost:8080/ajax-demo1/aJAXDemo1?username=zhangsan"  
}).then(function (resp){  
	alert(resp.data);  
})
```
        
- 发送 post 请求
```js
axios({  
	method:"post",  
	url:"http://localhost:8080/ajax-demo1/aJAXDemo1",  
	data:"username=zhangsan"  
}).then(function (resp){  
	alert(resp.data);  
});
```


axios()是用来发送异步请求的，小括号中`使用 js的JSON对象`传递请求相关的参数：

- `method属性`：用来设置请求方式的。取值为 get 或者 post。
- `url属性`：用来书写请求的资源路径。如果是 get 请求，需要将请求参数拼接到路径的后面，格式为： url?参数名=参数值&参数名2=参数值2。
- `data属性`：作为请求体被发送的数据。也就是说<font color="#00b050">如果是 post 请求的话，数据需要作为 data 属性的值</font>。

`then() 需要传递一个匿名函数`。我们将 then()中传递的匿名函数称为 **回调函数**，意思是该匿名函数在发送请求时不会被调用，而是在成功响应后调用的函数。而该`回调函数中的 resp 参数是对响应的数据进行封装的对象`，通过 resp.data 可以获取到响应的数据。

### 3.2 Axios快速入门

- 后端实现
    查询所有员工信息服务器
    根据员工id删除员工信息服务器
    
- 前端实现
	1. 首先在VS Code中创建js文件夹，与html同级，然后将**资料/axios-0.18.0.js** 文件拷贝到js目录下，然后创建名为02. Ajax-Axios.html的文件，
	2. 然后在html中引入axios所依赖的js文件，并且提供2个按钮，绑定单击事件，分别用于点击时发送ajax请求
	3. 然后分别使用Axios的方法，完整get请求和post请求的发送 
	4. 浏览器打开，f12抓包，然后分别点击2个按钮，查看控制台效果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3f3d43853de6f2448763ef43b0430d21.png)

`完整代码如下`：
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
	<meta charset="UTF-8">  
	<meta http-equiv="X-UA-Compatible" content="IE=edge">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0">  
	<title>Ajax-Axios</title>  
	<script src="js/axios-0.18.0.js"></script>  
</head>  
<body>  
	<input type="button" value="获取数据GET" onclick="get()"> 
	<input type="button" value="删除数据POST" onclick="post()">  
</body>  
<script>  
	function get(){  
		//通过axios发送异步请求-get  
		axios({  
			method: "get",  
			url: "http://yapi.smart-xwork.cn/mock/169327/emp/list"  
		}).then(result) => {  
			console.log(result.data);  
		})  
	}  
	function post(){  
	   // 通过axios发送异步请求-post  
		axios({  
			method: "post",  
			url: "http://yapi.smart-xwork.cn/mock/169327/emp/deleteById",  
			data: "id=1"  
		}).then(result) => {  
			console.log(result.data);  
		})  
	}  
</script>  
</html>
```

然后分别使用Axios的方法，完整get请求和post请求的发送

get请求代码如下：
```js
//通过axios发送异步请求-get
 axios({
     method: "get",
     url: "http://yapi.smart-xwork.cn/mock/169327/emp/list"
 }).then(result => {
     console.log(result.data);
 })
```

post请求代码如下：
```js
//通过axios发送异步请求-post
 axios({
     method: "post",
     url: "http://yapi.smart-xwork.cn/mock/169327/emp/deleteById",
     data: "id=1"
 }).then(result => {
     console.log(result.data);
 })
```

浏览器打开，f12抓包，然后分别点击2个按钮，查看控制台效果如下：![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/071bc01c38dc03a5375c4f1c171e9a8e.png)
### 3.3 请求方法的别名

Axios还针对不同的请求，提供了别名方式的api,具体如下：

|方法|描述|
|---|---|
|axios.get(url [, config])|发送get请求|
|axios.delete(url [, config])|发送delete请求|
|axios.post(url [, data[, config]])|发送post请求|
|axios.put(url [, data[, config]])|发送put请求|

我们目前只关注get和post请求，所以在上述的入门案例中，我们可以将get请求代码改写成如下：
```js
axios.get("http://yapi.smart-xwork.cn/mock/169327/emp/list").then(result) => {  
    console.log(result.data);  
})
```

post请求改写成如下：
```js
axios.post("http://yapi.smart-xwork.cn/mock/169327/emp/deleteById","id=1").then(result) => {  
    console.log(result.data);  
})
```

### 3.4 案例

- 需求：基于Vue及Axios完成数据的动态加载展示，如下图所示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c1578d386bb116154ccd7bab0bff7100.png)
- 分析：
    
    前端首先是`一张表格`，我们`缺少数据`，而`提供数据的地址`已经有了，所以意味这我们需要`使用Ajax请求获取后台的数据`。但是Ajax请求什么时候发送呢？页面的数据应该是页面加载完成，自动发送请求，展示数据，所以我们`需要借助vue的mounted钩子函数`。那么拿到数据了，我们该怎么将数据显示表格中呢？这里就得`借助v-for指令来遍历数据`，展示数据。
    
- 步骤：
    1. 首先创建文件，提前准备基础代码，包括表格以及vue.js和axios.js文件的引入
    2. 我们需要在vue的mounted钩子函数中发送ajax请求，获取数据
    3. 拿到数据，数据需要绑定给vue的data属性
    4. 在\<tr>标签上通过v-for指令遍历数据，展示数据
- 代码实现：
	1. 首先创建文件，提前准备基础代码，包括表格以及vue.js和axios.js文件的引入
	2. 在vue的mounted钩子函数，编写Ajax请求，请求数据，代码如下： 
	3. ajax请求的数据我们应该绑定给vue的data属性，之后才能进行数据绑定到视图；并且浏览器打开后台地址，数据返回格式如下图所示：
		- 因为服务器响应的json中的data属性才是我们需要展示的信息，所以我们应该将员工列表信息赋值给vue的data属性，代码如下： 
	4. 在\<tr>标签上通过v-for指令遍历数据，展示数据，其中需要注意的是图片的值，需要使用vue的属性绑定，男女的展示需要使用条件判断，其代码如下：

`完整代码如下`：

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta http-equiv="X-UA-Compatible" content="IE=edge">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Ajax-Axios-案例</title>  
    <script src="js/axios-0.18.0.js"></script>  
    <script src="js/vue.js"></script>  
</head>  
<body>  
    <div id="app">  
        <table border="1" cellspacing="0" width="60%">  
            <tr>  
                <th>编号</th>  
                <th>姓名</th>  
                <th>图像</th>  
                <th>性别</th>  
                <th>职位</th>  
                <th>入职日期</th>  
                <th>最后操作时间</th>  
            </tr>  
​  
            <tr align="center" v-for="(emp,index) in emps">  
                <td>{{index + 1}}</td>  
                <td>{{emp.name}}</td>  
                <td>  
                    <img :src="emp.image" width="70px" height="50px">  
                </td>  
                <td>  
                    <span v-if="emp.gender == 1">男</span>  
                    <span v-if="emp.gender == 2">女</span>  
                </td>  
                <td>{{emp.job}}</td>  
                <td>{{emp.entrydate}}</td>  
                <td>{{emp.updatetime}}</td>  
            </tr>  
        </table>  
    </div>  
</body>  
<script>  
    new Vue({  
       el: "#app",  
       data: {  
         emps:[]  
       },  
       mounted () {  
          //发送异步请求,加载数据  
          axios.get("http://yapi.smart-xwork.cn/mock/169327/emp/list").then(result => {  
            console.log(result.data);  
            this.emps = result.data.data;  
          })  
       }  
    });  
</script>  
</html>
```