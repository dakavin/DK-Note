## 目录

- [1 Ajax](#1%20Ajax)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 实现方式](#1.2%20%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F)
		- [1.2.1 原生的JS实现方式（了解）](#1.2.1%20%E5%8E%9F%E7%94%9F%E7%9A%84JS%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F%EF%BC%88%E4%BA%86%E8%A7%A3%EF%BC%89)
		- [1.2.2 JQeury实现方式](#1.2.2%20JQeury%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F)
		- [1.2.3 aixos原生方式](#1.2.3%20aixos%E5%8E%9F%E7%94%9F%E6%96%B9%E5%BC%8F)
- [2 JSON](#2%20JSON)
	- [2.1 概念](#2.1%20%E6%A6%82%E5%BF%B5)
	- [2.2 语法](#2.2%20%E8%AF%AD%E6%B3%95)
		- [2.2.1 基本规则](#2.2.1%20%E5%9F%BA%E6%9C%AC%E8%A7%84%E5%88%99)
		- [2.2.2 获取数据](#2.2.2%20%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE)
	- [2.3 JSON数据和Java对象的相互转换](#2.3%20JSON%E6%95%B0%E6%8D%AE%E5%92%8CJava%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%9B%B8%E4%BA%92%E8%BD%AC%E6%8D%A2)
- [3 案例](#3%20%E6%A1%88%E4%BE%8B)
	- [3.1 校验用户名是否存在](#3.1%20%E6%A0%A1%E9%AA%8C%E7%94%A8%E6%88%B7%E5%90%8D%E6%98%AF%E5%90%A6%E5%AD%98%E5%9C%A8)

## 1 Ajax

### 1.1 概念

- ASynchronous JavaScript And XML	异步的JavaScript 和 XML

- 异步和同步：客户端和服务器端相互通信的基础上![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075255253.png)


	* 客户端必须等待服务器端的响应。`在等待的期间客户端不能做其他操作`。
	* 客户端不需要等待服务器端的响应。`在服务器处理请求的过程中，客户端可以进行其他的操作`。

- Ajax 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。
- 通过在后台与服务器进行少量数据交换，Ajax 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。
- 传统的网页（不使用 Ajax）如果需要更新内容，必须重载整个网页页面。

- `优点：提升用户的体验`

### 1.2 实现方式

#### 1.2.1 原生的JS实现方式（了解）

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075257367.png)



#### 1.2.2 JQeury实现方式

- $.ajax()
* 语法：$.ajax({键值对});
```js
//使用$.ajax()发送异步请求
$.ajax({
	url:"ajaxServlet1111" , // 请求路径
	type:"POST" , //请求方式
	//data: "username=jack&age=23",//请求参数
	data:{"username":"jack","age":23},
	success:function (data) {
		alert(data);
	},//响应成功后的回调函数
	error:function () {
		alert("出错啦...")
	},//表示如果请求响应出现错误，会执行的回调函数

	dataType:"text"//设置接受到的响应数据的格式
});
```

-` $.get()：发送get请求`
- 语法：`$.get(url, [data], [callback], [type])`
	- url：请求路径
	* data：请求参数
	* callback：回调函数
	* type：响应结果的类型

- `$.post()：发送post请求`
* 语法：`$.post(url, [data], [callback], [type])`
	* url：请求路径
	* data：请求参数
	* callback：回调函数
	* type：响应结果的类型

#### 1.2.3 aixos原生方式

- 第一步：导入axios的js文件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075321283.png)


- 第二步：使用axios发送请求，并获取响应结果![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075323096.png)



- `axios请求方式别名`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075324957.png)
	
	![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075326946.png)

## 2 JSON

### 2.1 概念

- JavaScript Object Notation		JavaScript对象表示法
```js
Person p = new Person();
p.setName("张三");
p.setAge(23);
p.setGender("男");

var p = {"name":"张三","age":23,"gender":"男"};
```

* json现在多用于存储和交换文本信息的语法
* 进行数据的传输
* JSON 比 XML 更小、更快，更易解析。

### 2.2 语法

#### 2.2.1 基本规则

* 数据在名称/值对中：`json数据是由键值对构成的`
	* 键`用引号(单双都行)引起来，也可以不使用引号`
	* 值的取值类型：
		1. 数字（整数或浮点数）
		2. 字符串（在双引号中）
		3. 逻辑值（true 或 false）
		4. 数组（在方括号中）	{"persons":[{},{}]}
		5. 对象（在花括号中） {"address":{"province"："陕西"....}}
		6. null
* 数据由逗号分隔：`多个键值对由逗号分隔`
* `花括号保存对象`：`使用{}定义json 格式`
* `方括号保存数组`：[]

#### 2.2.2 获取数据

1. json对象.键名
2. json对象["键名"]  `在遍历的时候使用这个方法`
3. 数组对象[索引]
4. 遍历

```js
 //1.定义基本格式
var person = {"name": "张三", age: 23, 'gender': true};

var ps = [{"name": "张三", "age": 23, "gender": true},
	{"name": "李四", "age": 24, "gender": true},
	{"name": "王五", "age": 25, "gender": false}];
//获取person对象中所有的键和值
//for in 循环
/* for(var key in person){
	//这样的方式获取不行。因为相当于  person."name"
	//alert(key + ":" + person.key);
	alert(key+":"+person[key]);
}*/

//获取ps中的所有值
for (var i = 0; i < ps.length; i++) {
	var p = ps[i];
	for(var key in p){
		alert(key+":"+p[key]);
	}
}
```


### 2.3 JSON数据和Java对象的相互转换

* JSON解析器：
	* 常见的解析器：Jsonlib，Gson，fastjson，jackson

1. `JSON转为Java对象`
	- 导入jackson的相关jar包
	- 创建Jackson核心对象 ObjectMapper
	- 调用ObjectMapper的相关方法进行转换
		- readValue(json字符串数据,Class)
2. `Java对象转换JSON`
	- 使用步骤：
		- 导入jackson的相关jar包
		- 创建Jackson核心对象 ObjectMapper
		- 调用ObjectMapper的相关方法进行转换
			- 转换方法：
				* writeValue(参数1，obj):
					参数1：
						File：将obj对象转换为JSON字符串，并保存到指定的文件中
						Writer：将obj对象转换为JSON字符串，并将json数据填充到字符输出流中
						OutputStream：将obj对象转换为JSON字符串，并将json数据填充到字节输出流中
				* writeValueAsString(obj):将对象转为json字符串

			- 注解：
				- @JsonIgnore：`排除属性`。
				- @JsonFormat：`属性值得格式化`
					* @JsonFormat(pattern = "yyyy-MM-dd")

			- 复杂java对象转换
				- List：数组
				- Map：`对象格式一致`

- `使用FastJson`![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075358963.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922075401571.png)





## 3 案例

### 3.1 校验用户名是否存在

1. 服务器响应的数据，在客户端使用时，要想当做json数据格式使用。有两种解决方案：
	- $.get(type):将最后一个参数type指定为"json"
	- 在服务器端设置MIME类型
- response.setContentType("application/json;charset=utf-8");

- html代码
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>注册页面</title>  
    <script src="./js/jquery-3.3.1.min.js"></script>  
    <script>        //在页面加载完成后  
        $(function () {  
            //给username绑定一个离焦事件  
            $("#username").blur(function () {  
                //获取文本输入框的值  
                var username = $(this).val();  
                //发生Ajax请求  
                //期望服务器响应会的数据格式：  
                /**  
                 * {"userExist":true,"msg":"此用户名太受欢迎，请更换一个"}  
                 * {"userExist":false,"msg":"用户名可用"}  
                 */  
                $.get("findUserServlet",{username:username},function (data) {  
                    //判断userExist键的值是否为true  
                    var span = $("#s_username");  
                    if (data.userExist){  
                        span.css("color","red")  
                        span.html(data.msg);  
                    }else{  
                        span.css("color","green")  
                        span.html(data.msg);  
                    }  
                },"json")  
            })  
        })  
    </script>  
</head>  
<body>  
    <form>  
        <input type="text" id="username" name="username" placeholder="请输入用户名">  
        <span id="s_username"></span>  
        <br>        <input type="password" name="password" placeholder="请输入密码"><br>  
        <input type="submit" value="注册"><br>  
    </form>  
</body>  
</html>
```

- servlet代码
```java
@WebServlet("/findUserServlet")  
public class FindUserServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        //0.设置编码  
        req.setCharacterEncoding("utf-8");  
        resp.setContentType("text/html;charset=utf-8");  
        //1. 获取用户名  
        String username = req.getParameter("username");  
        //2. 调用service层判断用户名是否存在  
        //期望服务器响应会的数据格式：  
        /**  
         * {"userExist":true,"msg":"此用户名太受欢迎，请更换一个"}  
         * {"userExist":false,"msg":"用户名可用"}  
         */        Map<String,Object> map = new HashMap<>();  
        if ("tom".equals(username)){  
            map.put("userExist",true);  
            map.put("msg","此用户名太受欢迎，请更换一个");  
        }else{  
            map.put("userExist",false);  
            map.put("msg","用户名可用");  
        }  
        ObjectMapper mapper = new ObjectMapper();  
        //并传递给客户端  
        mapper.writeValue(resp.getWriter(),map);  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        doGet(req, resp);  
    }  
}
```