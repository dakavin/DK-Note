 ## 目录

- [1 概述](#1%20%E6%A6%82%E8%BF%B0)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 功能](#1.2%20%E5%8A%9F%E8%83%BD)
	- [1.3 JavaScript发展史](#1.3%20JavaScript%E5%8F%91%E5%B1%95%E5%8F%B2)
- [2 ECMAScript基本语法](#2%20ECMAScript%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
	- [2.1 与HTML结合方式](#2.1%20%E4%B8%8EHTML%E7%BB%93%E5%90%88%E6%96%B9%E5%BC%8F)
	- [2.2 注释](#2.2%20%E6%B3%A8%E9%87%8A)
	- [2.3 数据类型](#2.3%20%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
	- [2.4 变量](#2.4%20%E5%8F%98%E9%87%8F)
	- [2.5 运算符](#2.5%20%E8%BF%90%E7%AE%97%E7%AC%A6)
	- [2.6 流程控制语句](#2.6%20%E6%B5%81%E7%A8%8B%E6%8E%A7%E5%88%B6%E8%AF%AD%E5%8F%A5)
	- [2.7 JS特殊语法](#2.7%20JS%E7%89%B9%E6%AE%8A%E8%AF%AD%E6%B3%95)
	- [2.8 类型转换问题](#2.8%20%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E9%97%AE%E9%A2%98)
	- [2.9 练习：99乘法表](#2.9%20%E7%BB%83%E4%B9%A0%EF%BC%9A99%E4%B9%98%E6%B3%95%E8%A1%A8)
- [3 ECMAScript基本对象](#3%20ECMAScript%E5%9F%BA%E6%9C%AC%E5%AF%B9%E8%B1%A1)
	- [3.1 Function：函数对象](#3.1%20Function%EF%BC%9A%E5%87%BD%E6%95%B0%E5%AF%B9%E8%B1%A1)
	- [3.2 Array：数组对象](#3.2%20Array%EF%BC%9A%E6%95%B0%E7%BB%84%E5%AF%B9%E8%B1%A1)
	- [3.3 包装类](#3.3%20%E5%8C%85%E8%A3%85%E7%B1%BB)
	- [3.4 Date：日期对象](#3.4%20Date%EF%BC%9A%E6%97%A5%E6%9C%9F%E5%AF%B9%E8%B1%A1)
	- [3.5 Math：数学对象](#3.5%20Math%EF%BC%9A%E6%95%B0%E5%AD%A6%E5%AF%B9%E8%B1%A1)
	- [3.6 RegExp：正则表达式对象](#3.6%20RegExp%EF%BC%9A%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%AF%B9%E8%B1%A1)
	- [3.7 String对象](#3.7%20String%E5%AF%B9%E8%B1%A1)
	- [3.8 自定义对象&JSON](#3.8%20%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AF%B9%E8%B1%A1&JSON)
	- [3.9 Global对象](#3.9%20Global%E5%AF%B9%E8%B1%A1)
- [4 DOM对象简单学习](#4%20DOM%E5%AF%B9%E8%B1%A1%E7%AE%80%E5%8D%95%E5%AD%A6%E4%B9%A0)
- [5 事件简单学习](#5%20%E4%BA%8B%E4%BB%B6%E7%AE%80%E5%8D%95%E5%AD%A6%E4%B9%A0)
- [6 BOM对象](#6%20BOM%E5%AF%B9%E8%B1%A1)
	- [6.1 概念](#6.1%20%E6%A6%82%E5%BF%B5)
	- [6.2 组成](#6.2%20%E7%BB%84%E6%88%90)
	- [6.3 Window：窗口对象](#6.3%20Window%EF%BC%9A%E7%AA%97%E5%8F%A3%E5%AF%B9%E8%B1%A1)
	- [6.4 Location：地址栏对象](#6.4%20Location%EF%BC%9A%E5%9C%B0%E5%9D%80%E6%A0%8F%E5%AF%B9%E8%B1%A1)
	- [6.5 History：历史记录对象](#6.5%20History%EF%BC%9A%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95%E5%AF%B9%E8%B1%A1)
- [7 DOM](#7%20DOM)
	- [7.1 概念](#7.1%20%E6%A6%82%E5%BF%B5)
	- [7.2 W3C DOM 标准组成部分](#7.2%20W3C%20DOM%20%E6%A0%87%E5%87%86%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86)
	- [7.3 核心DOM模型](#7.3%20%E6%A0%B8%E5%BF%83DOM%E6%A8%A1%E5%9E%8B)
	- [7.4 HTML DOM](#7.4%20HTML%20DOM)
- [8 事件监听机制](#8%20%E4%BA%8B%E4%BB%B6%E7%9B%91%E5%90%AC%E6%9C%BA%E5%88%B6)
	- [8.1 概念](#8.1%20%E6%A6%82%E5%BF%B5)
	- [8.2 事件绑定](#8.2%20%E4%BA%8B%E4%BB%B6%E7%BB%91%E5%AE%9A)
	- [8.3 常见事件](#8.3%20%E5%B8%B8%E8%A7%81%E4%BA%8B%E4%BB%B6)

## 1 概述

### 1.1 概念

- 一门客户端脚本语言
* 运行在客户端浏览器中的。每一个浏览器都有JavaScript的解析引擎
* 脚本语言：不需要编译，直接就可以被浏览器解析执行了

### 1.2 功能

- 可以来增强用户和html页面的交互过程，可以来控制html元素，让页面有一些动态的效果，增强用户的体验。

### 1.3 JavaScript发展史

1. 1992年，Nombase公司，开发出第一门客户端脚本语言，专门用于表单的校验。命名为 ： C--	，后来更名为：ScriptEase
2. 1995年，Netscape(网景)公司，开发了一门客户端脚本语言：LiveScript。后来，请来SUN公司的专家，修改LiveScript，命名为JavaScript
3. 1996年，微软抄袭JavaScript开发出JScript语言
4. 1997年，ECMA(欧洲计算机制造商协会)，制定出客户端脚本语言的标准：ECMAScript，就是统一了所有客户端脚本语言的编码方式。

- `JavaScript = ECMAScript + JavaScript自己特有的东西(BOM+DOM)`

## 2 ECMAScript基本语法

### 2.1 与HTML结合方式

1. 内部JS：
	* 定义\<script>，`标签体内容`就是js代码
2. 外部JS：
	* 定义\<script>，通过`src属性引入外部的js文件`

* 注意：
	1. \<script>可以定义在html页面的任何地方。但是定义的位置会影响执行顺序。
	2. \<script>可以定义多个。

### 2.2 注释

1. 单行注释：//注释内容
2. 多行注释：/\*注释内容*/

### 2.3 数据类型

1. 原始数据类型(基本数据类型)：
	1. `number`：数字。 整数/小数/NaN(not a number 一个不是数字的数字类型)
	2. `string`：字符串。 字符串  "abc" "a" 'abc' ，`使用单双引号都可以`
	3. `boolean`: true和false
	4. `null`：一个对象为空的占位符![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36818fe19d13ab9b9b95d6634c1b330e.png)


	5. `undefined`：未定义。如果一个变量没有给初始化值，则会被默认赋值为undefined
	
2. 引用数据类型：对象

3. typeof 变量名：可以获取该变量的数据类型

### 2.4 变量

书写语法会了，变量是一门编程语言比不可少的，所以接下来我们需要学习js中变量的声明，在js中，变量的声明和java中还是不同的。首先js中主要通过如下3个关键字来声明变量的：

| 关键字 | 解释                                                         |
| ------ | ------------------------------------------------------------ |
| var    | 早期ECMAScript5中用于变量声明的关键字                        |
| let    | ECMAScript6中新增的用于变量声明的关键字，相比较var，let只在代码块内生效 |
| const  | 声明常量的，常量一旦声明，不能修改                           |

在js中声明变量还需要注意如下几点：

- JavaScript 是一门弱类型语言，变量可以存放不同类型的值 。
- 变量名需要遵循如下规则：
  - 组成字符可以是任何字母、数字、下划线（_）或美元符号（$）
  - 数字不能开头
  - 建议使用驼峰命名

* 变量：一小块存储数据的内存空间
* `Java语言是强类型语言，而JavaScript是弱类型语言。`
	* 强类型：在开辟变量存储空间时，定义了空间将来存储的数据的数据类型。只能`存储固定类型的数据`
	* 弱类型：在开辟变量存储空间时，不定义空间将来的存储数据类型，可以`存放任意类型的数据`。
* 语法：
	* `var 变量名 = 初始化值`;


* typeof运算符：获取变量的类型。
	* 注：`null运算后得到的是object`
	* 实际上是JavaScript最初实现中的一个错误，被ECMAScript沿用了，现在，null被认为是对象的占位符，从而解释了这一矛盾。但从技术上来说，它仍是原始值

- 补充：
	- `在IDEA中写JS时，var会出现警告，这是因为高版本的ECMAScript6 建议使用let 或者 const`
	- 点击File→Settings→Language & Frameworks→JavaScript
	- 将JavaScript language version中的ECMAScript 6切换为ECMAScript 5.1即可

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/27298010db50df289c37ab1b8e9ea936.png)



### 2.5 运算符

1. `一元运算符`：只有一个运算数的运算符
	++，-- ， +(正号)  
	* ++ --: 自增(自减)
		* ++(--) 在前，先自增(自减)，再运算
		* ++(--) 在后，先运算，再自增(自减)
	* +(-)：正负号
	* <font color="#d83931">注意：在JS中，如果运算数不是运算符所要求的类型，那么js引擎会自动的将运算数进行类型转换</font>
		* 其他类型转number：`是在其他类型前加+-号后转`
			* string转number：按照字面值转换。如果字面值不是数字，则转为NaN（不是数字的数字）
			* boolean转number：true转为1，false转为0
2. `算数运算符`
	+ - * / % ...

3. `赋值运算符`
	= += -+....

4. `比较运算符`
\> < >= <= `== ===(全等于)`
* 比较方式
  1. <font color="#d83931">类型相同：直接比较</font>
	  * 字符串：按照字典顺序比较。按位逐一比较，直到得出大小为止。
  2. <font color="#d83931">类型不同：先进行类型转换，再比较</font>
	  * \=\==：`全等于`。在比较之前，先判断类型，如果类型不一样，则直接返回false

5. `逻辑运算符`
	&& || !
	* 其他类型转boolean：
	   1. number：0或NaN为假，其他为真
	   2. string：除了空字符串("")，其他都是true
	   3. null&undefined:都是false
	   4. 对象：所有对象都为true
- 应用
```js
<script>  
    var str = "abc";  
    if (str != null && str.length >0)  
        alert(111);  
	//两个if的效果一样，从而简化书写
    if (str)  
        alert(222);  
</script>
```

6. `三元运算符`
	? : 表达式
	var a = 3;
	var b = 4;

	var c = a > b ? 1:0;
	* 语法：
		* 表达式? 值1:值2;
		* 判断表达式的值，如果是true则取值1，如果是false则取值2；

### 2.6 流程控制语句

1. if...else...
2. `switch`:
	* 在java中，switch语句可以接受的数据类型： byte int shor char,枚举(jdk1.5) ,String(jdk1.7)
		* switch(变量):
			case 值:
	* `在JS中,switch语句可以接受任意的原始数据类型`
3. while
4. do...while
5. for

- `switch的case可以是任何类型的`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23f3ce0e042d99956e43c083777d65e1.png)



### 2.7 JS特殊语法

1. 语句以;结尾，如果一行只有一条语句则 ;可以省略 (不建议)
2. 变量的定义使用var关键字，也可以不使用
	* 用： 定义的变量是局部变量
	* 不用：定义的变量是全局变量(不建议)


### 2.8 类型转换问题

- 字符串类型转为数字
	- 将字符串字面值，转为数字；如果字面值不是数字，则转为NaN
- 其他类型转位boolean
	- number：0 和 NaN 为 false；    其他均为 true
	- String： 空字符串为false；    其他均为true
	- Null 和 undefined ：均转为false

- `记住false的情况即可`

### 2.9 练习：99乘法表

```js
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>99乘法表</title>  
    <style >        td{  
            border:1px solid;  
            text-align: center;  
        }  
    </style>  
    <script>        document.write("<table  align='center'>")  
        for (var i = 1; i < 10; i++) {  
            document.write("<tr align='center'>")  
            for (var j = 1; j <=i ; j++) {  
                document.write("<td>")  
                document.write(i + "*" + j + "=" + i*j + "&nbsp;&nbsp;&nbsp;");  
                document.write("</td>")  
            }  
            document.write("<tr>")  
        }  
        document.write("</table>")  
        //完成表格的嵌套  
    </script>  
</head>  
<body>  
  
</body>  
</html>
```

## 3 ECMAScript基本对象

### 3.1 Function：函数对象

1. 创建：
	1. var fun = new Function(形式参数列表,方法体);  //忘掉吧
	2. 
		function 方法名称(形式参数列表){
			方法体
		}
	3. 
	   var 方法名 = function(形式参数列表){
			方法体
	   }
2. 方法： 
 3. 属性：
	`length:代表形参的个数`
4. 特点：
	1. 方法定义是，`形参的类型不用写,返回值类型也不写`。
	2. 方法是一个对象，`如果定义名称相同的方法，会覆盖`
	3. 在JS中，`方法的调用只与方法的名称有关，和参数列表无关`
	4. 在方法声明中有一个隐藏的内置对象（数组），arguments,封装所有的实际参数
5. 调用：
	- 方法名称(实际参数列表);

- `arguments的使用`
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5634a4bf996baa120541e04185b27ab.png)



- `箭头函数（ES6）`：是用来简化函数定义语法的。具体形式为：
```js
(...) => {....}
var xxx = (...) => {....}
```

### 3.2 Array：数组对象

 1. 创建：
	1. var arr = new Array(元素列表);
	2. var arr = new Array(默认长度);
	3. var arr = [元素列表];  `注意和java的区分`
2. 方法
	join(参数): 将数组中的元素按照指定的分隔符拼接为字符串
	push()	:  向数组的末尾添加一个或更多元素，并返回新的长度。
	foreach(): 遍历数组中的每一个`有值的元素`，并调用一次传入的函数![[Pasted image 20230725124330.png]]
	splice()   : 从数组中删除元素![[Pasted image 20230725124650.png]]
1. 属性
	length:数组的长度
4. 特点：
	1. `JS中，数组元素的类型可变的`。
	2. `JS中，数组长度可变的`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f01fce43ec451cb5fe26fcce1a6e9d0a.png)



### 3.3 包装类

- 基本数据类型的包装类
- Boolean、Number、String

### 3.4 Date：日期对象

1. 创建：
	var date = new Date();

2. 方法：
	toLocaleString()：返回当前date对象对应的时间本地字符串格式
	getTime():获取毫秒值。返回当前如期对象描述的时间到1970年1月1日零点的毫秒值差

### 3.5 Math：数学对象

1. 创建：
	* 特点：`Math对象不用创建，直接使用`。  `Math.方法名();`

2. 方法：
	random():返回 0 ~ 1 之间的随机数。` 含0不含1`
	ceil(x)：对数进行上舍入。
	floor(x)：对数进行下舍入。
	round(x)：把数四舍五入为最接近的整数。
3. 属性：
	PI

- 取0-100的随机整数 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42fa412b74c8b2c0d48d4e05509cca73.png)



### 3.6 RegExp：正则表达式对象

1. 正则表达式：`定义字符串的组成规则`。
	1. 单个字符:`[]`
		如： `[a] [ab] [a-zA-Z0-9_ ]`
		* 特殊符号代表特殊含义的单个字符:
			`\d`:单个数字字符 `[0-9]`
			`\w`:单个单词字符`[a-zA-Z0-9 _]`
	2. 量词符号：
		`?`：表示出现0次或1次
		`*`：表示出现0次或多次
		`+`：表示出现1次或多次
		`{m,n}`:表示 `m<= 数量 <= n`
		* m如果缺省： {,n}:最多n次
		* n如果缺省：{m,} 最少m次
	3. 开始结束符号
		* ^:开始
		* $:结束
2. 正则对象：
	1. 创建
		1. `var reg = new RegExp("正则表达式");`
		2. `var reg = /正则表达式/;`
	2. 方法	
		1. `test(参数):验证指定的字符串是否符合正则定义的规范`	

- 案例
```js
<script>  
    var reg1 = new RegExp("\w{6,12}");  
    var reg2 = /^\w{6,12}$/;  
  
   /* alert(reg1);  
    alert(reg2);*/  
    var username = "zhangsan";  
    var flag = reg2.test(username);  
    alert(flag);  
</script>
```

### 3.7 String对象



- 创建方式
```js
var 变量名 = new String("XXXX");
var 变量名 = "XXXX";
var 变量名 = 'XXXX';
```

- 属性
	- length：字符串的长度

- 方法
	- charAt()：返回在指定位置的字符
	- indexOf() ：检索字符串
	- trim()：去除字符串两端的空格
	- substring()：提取字符串中两个指定的索引号之间的字符

### 3.8 自定义对象&JSON

<h4>自定义对象</h4>
- 定义格式
```js
var 对象名 = {
    属性名1: 属性值1, 
    属性名2: 属性值2,
    属性名3: 属性值3,
    函数名称: function(形参列表){}
};
```

- 调用格式
```js
//属性调用格式
对象名.属性名

//方法调用格式
对象名.函数名()
```

- 案例
```js
//自定义对象
var user = {
	name: "Tom",
	age: 10,
	gender: "male",
	eat: function(){
		 console.log("用膳~");
	 }
}

console.log(user.name);
user.eat();


//简化
var user = {
	name: "Tom",
	age: 10,
	gender: "male",
	// eat: function(){
	//      console.log("用膳~");
	//  }
	eat(){
		console.log("用膳~");
	}
}
```

<h4>JSON</h4>

- JSON对象：**J**ava**S**cript **O**bject **N**otation，JavaScript对象标记法。是通过JavaScript标记法书写的文本。其格式如下：

~~~js
{
    "key":value,
    "key":value,
    "key":value
}
~~~

- 其中，**key必须使用引号并且是双引号标记，value可以是任意数据类型。**

- 那么json这种数据格式的文本到底应用在企业开发的什么地方呢？-- `经常用来作为前后台交互的数据载体`

- 如下图所示：前后台交互时，我们需要传输数据，但是java中的对象我们该怎么去描述呢？我们可以使用如图所示的xml格式，可以清晰的描述java中需要传递给前端的java对象。

![1668597000013](assets/1668597000013.png) 

但是xml格式存在如下问题：

- 标签需要编写双份，占用带宽，浪费资源
- 解析繁琐

所以我们可以使用json来替代，如下图所示：

![1668597176685](assets/1668597176685.png) 

- JSON的数据类型
	- 数组（整数或浮点数）
	- 字符串（在双引号中）
	- 逻辑值（true或false）
	- 数组（在方括号中）
	- 对象（在花括号中）
	- null

- `JSON的整体代码`
~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS-对象-JSON</title>
</head>
<body>
    
</body>
<script>
    //自定义对象
    // var user = {
    //     name: "Tom",
    //     age: 10,
    //     gender: "male",
    //     // eat: function(){
    //     //      console.log("用膳~");
    //     //  }
    //     eat(){
    //         console.log("用膳~");
    //     }
    // }

    // console.log(user.name);
    // user.eat();


    // //定义json
    var jsonstr = '{"name":"Tom", "age":18, "addr":["北京","上海","西安"]}';
    //alert(jsonstr.name);

    // //json字符串--js对象
    var obj = JSON.parse(jsonstr);
    //alert(obj.name);

    // //js对象--json字符串
    alert(JSON.stringify(obj));


</script>
</html>
~~~

### 3.9 Global对象

1. 特点：`全局对象，这个Global中封装的方法不需要对象就可以直接调用。  方法名();`
2. 方法：
	encodeURI():url编码
	decodeURI():url解码

	encodeURIComponent():url编码,编码的字符更多
	decodeURIComponent():url解码

	parseInt():将字符串转为数字
		* 逐一判断每一个字符是否是数字，直到不是数字为止，将前边数字部分转为number
	isNaN():判断一个值是否是NaN
		* NaN六亲不认，连自己都不认。`NaN参与的==比较全部问false`

	`eval()`:将 JavaScript 字符串，并把它作为脚本代码来执行。
3. URL编码
   传智播客 =  %E4%BC%A0%E6%99%BA%E6%92%AD%E5%AE%A2

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b42d6f6ee977c35e2a4d971b7d5f73fe.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/960a3b76749061fd2ca83c14c099e6bc.png)



![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1eef4444ee30db69494d2cfe15990ce6.png)

## 4 DOM对象简单学习

* 功能：`控制html文档的内容`
* 获取页面标签(元素)对象：Element
	* `document.getElementById("id值")`:通过元素的id获取元素对象

* 操作Element对象：
	1. 修改属性值：
		1. 明确获取的对象是哪一个？
		2. 查看API文档，找其中有哪些属性可以设置
	2. 修改标签体内容：
		* 属性：innerHTML
		1. 获取元素对象
		2. 使用innerHTML属性修改标签体内容

![[Pasted image 20230707115829.png]]

## 5 事件简单学习

* 功能： `某些组件被执行了某些操作后，触发某些代码的执行`。
	* 造句：  xxx被xxx,我就xxx
		* 我方水晶被摧毁后，我就责备对友。
		* 敌方水晶被摧毁后，我就夸奖自己。

* 如何绑定事件
	1. 直接在html标签上，指定事件的属性(操作)，属性值就是js代码
		1. 事件：onclick--- 单击事件

	2. 通过js获取元素对象，指定事件属性，设置一个函数

- `代码`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>事件绑定操作</title>  
</head>  
<body>  
<!--    方式1：事件在标签上，耦合度比较高-->  
    <img id="light1" src="./img/off.gif" onclick="fun();">  
<!--    方式2：事件转移，标签上写一个id来获取标签，在script中进行操作-->  
    <img id="light2" src="./img/off.gif">  
<script>  
    function fun(){  
        alert('我被点击了');  
        alert('我又被点击了');  
    }  
    function fun1(){  
        alert('咋老点我？');  
    }  
    var light2 = document.getElementById("light2");  
    light2.onclick=fun1;  
</script>  
</body>  
</html>
```

- `演示灯关开关代码`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>模拟灯关开关</title>  
</head>  
<body>  
    <img src="./img/off.gif" id="light">  
</body>  
<script>  
    //1. 获取图片对象  
    var light = document.getElementById("light");  
  
    var flag = false; //表示灯是灭的  
    //2. 绑定单击事件  
    light.onclick=function (){  
        if (flag){  
            light.src="img/off.gif";  
            flag=false;  
        } else{  
            light.src="img/on.gif";  
            flag=true;  
        }  
    }  
</script>  
</html>
```


## 6 BOM对象

### 6.1 概念

- Browser Object Model 浏览器对象模型
- 将浏览器的`各个组成部分封装成对象`

### 6.2 组成

* Navigator：浏览器对象，**包含有关访问者的信息**
* `Window：窗口对象`
	* 它代表浏览器的窗口，所有全局 JavaScript 对象，函数和变量自动成为 window 对象的成员。
- ` Location：地址栏对象` **可用于获取当前页面地址（URL）并把浏览器重定向到新页面。**
* History：历史记录对象，**包含浏览器历史。**
* Screen：显示器屏幕对象，包含用户屏幕的信息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bf3390daebcc483396f26d9187685562.png)



### 6.3 Window：窗口对象

1. 创建
2. 方法
	 1.  `与弹出框有关的方法`：
		- `alert()`	显示带有一段消息和一个确认按钮的警告框。
		- `confirm()`	显示带有一段消息以及确认按钮和取消按钮的对话框。
			* 如果用户点击确定按钮，则方法返回true
			* 如果用户点击取消按钮，则方法返回false
		- `prompt()`	显示可提示用户输入的对话框。
			* 返回值：获取用户输入的值
	 2. `与打开关闭有关的方法`：
		- `close()`	关闭浏览器窗口。
			* `谁调用我 ，我关谁`
		- `open()`	打开一个新的浏览器窗口
			* `返回新的Window对象`
	 3. `与定时器有关的方式`
		- `setTimeout()`	在指定的毫秒数后调用函数或计算表达式。
			* 参数：
				1. js代码或者方法对象
				2. 毫秒值
			* 返回值：唯一标识，用于取消定时器
		- `clearTimeout()`	取消由 setTimeout() 方法设置的 timeout。

		- `setInterval()`	按照指定的周期（以毫秒计）来调用函数或计算表达式。
		- `clearInterval()`	取消由 setInterval() 设置的 timeout。

3. 属性：
	1. 获取其他BOM对象：
		history
		location
		Navigator
		Screen:
	2. 获取DOM对象
		document
4. `特点`
	* `Window对象不需要创建可以直接使用 window使用。` `window.方法名();`
	* `window引用可以省略。  方法名();`

- `图片轮播案例`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>轮播图</title>  
</head>  
<body>  
    <img src="./img/banner_1.jpg" id="banner" width="100%">  
  
    <script>        var i = 1;  
        function fun() {  
            i++;  
            if (i>3)  
                i=1;  
            var banner = document.getElementById("banner");  
            banner.src="img/banner_"+ i +".jpg";  
        }  
        setInterval(fun,1000);  
    </script>  
</body>  
</html>
```

### 6.4 Location：地址栏对象

1. 创建(获取)：
	1. window.location
	2. location

2. 方法：
	* reload()	重新加载当前文档。刷新
3. 属性
	* href	设置或返回完整的 URL。

- `倒计时，跳转回首页`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>自动跳转首页</title>  
    <style>        p{  
            text-align: center;  
        }  
        span{  
            color: red;  
        }  
    </style>  
</head>  
<body>  
    <p><span id="f">5</span>秒之后，自动跳转首页...</p>  
    <script>        
	    var i = 5;  
        var f = document.getElementById("f");  
        function fun1() {  
            if (i<=1){  
                location.href="https://www.baidu.com";  
            }  
            i--;  
            f.innerText=i+"";  
  
        }  
        setInterval(fun1,1000);  
    </script>  
</body>  
</html>
```

### 6.5 History：历史记录对象

1. 创建(获取)：
	1. window.history
	2. history

2. 方法：
	* back()	加载 history 列表中的前一个 URL。
	* forward()	加载 history 列表中的下一个 URL。
	* go(参数)	加载 history 列表中的某个具体页面。
		* 参数：
			* `正数：前进几个历史记录`
			* `负数：后退几个历史记录`
3. 属性：
	* length	返回当前窗口历史列表中的 URL 数量。

## 7 DOM

### 7.1 概念

- Document Object Model 文档对象模型
- 将标记语言文档的各个组成部分，封装成对象。可以使用这些对象，对标记语言文档进行CRUD的动态操作

![[DOM树.bmp]]


那么我们学习DOM技术有什么用呢？主要作用如下：

- 改变 HTML 元素的内容
- 改变 HTML 元素的样式（CSS）
- 对 HTML DOM 事件作出反应
- 添加和删除 HTML 元素


### 7.2 W3C DOM 标准组成部分

* `核心 DOM - 针对任何结构化文档的标准模型`
	* Document：文档对象
	* Element：元素对象，即每一个标签
	* Attribute：属性对象，即每一个标签的属性
	* Text：文本对象，即每一个标签内部的文本
	* Comment:注释对象
	* Node：节点对象，其他5个的父对象

* `XML DOM - 针对 XML 文档的标准模型`
* `HTML DOM - 针对 HTML 文档的标准模型`

### 7.3 核心DOM模型

* `Document：文档对象`
	1. `创建(获取)`：在html dom模型中可以使用window对象来获取
		1. `window.document`
		2. `document`
	2. 方法：
		1. 获取Element对象：
			1. `getElementById()`	： <font color="#d83931">根据id属性值</font>获取元素对象。<font color="#d83931">id属性值一般唯一</font>
			2. `getElementsByTagName()`：<font color="#d83931">根据元素名称</font>获取元素对象们。返回值是一个数组
			3. `getElementsByClassName()`:<font color="#d83931">根据Class属性值</font>获取元素对象们。返回值是一个数组
			4. `getElementsByName()`: <font color="#d83931">根据name 属性值</font>获取元素对象们。返回值是一个数组
		2. 创建其他DOM对象：
			`createAttribute(name)`
			createComment()
			createElement()
			createTextNode()
	3. 属性

* `Element：元素对象`
	1. 获取/创建：通过document来获取和创建
	2. 方法：
		1. removeAttribute()：删除属性，<font color="#d83931">两个字符串为实参（属性名，属性值）</font>
		2. setAttribute()：设置属性，<font color="#d83931">一个字符串为实参（属性名）</font>

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e04b4ead7e7ab082d2ac61ce7564f9d.png)



* `Node：节点对象，其他5个的父对象`
	* 特点：所有dom对象都可以被认为是一个节点
	* 方法：
		* `CRUD dom树`：
			* `appendChild()`：向节点的子节点列表的结尾添加新的子节点。
			* `removeChild()`	：删除（并返回）当前节点的指定子节点。
			* replaceChild()：用新节点替换一个子节点。
	* 属性：
		* parentNode 返回节点的父节点。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e8ff09e636eed6d0582d698e1425bfe.png)



- `动态表格案例`
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>动态表格</title>  
    <style>        input{  
            height: 20px;  
            width: 180px;  
        }  
        #div1{  
            margin-top:50px;  
            text-align: center;  
        }  
        #div2{  
            margin-top:50px;  
            text-align: center;  
        }  
        #btn{  
            width: 60px;  
            height: 30px;  
        }  
        #table{  
            border: 1px solid black;  
            margin: auto;  
            padding: 5px;  
        }  
        td,th{  
            border: 1px solid black;  
            width: 130px;  
        }  
    </style>  
</head>  
<body>  
    <div id="div1">  
        <input type="text" placeholder="请输入编号" id="id">  
        <input type="text" placeholder="请输入姓名" id="name">  
        <input type="text" placeholder="请输入性别" id="gender">  
        <input type="button" id="btn" value="添加">  
    </div>  
    <div id="div2">  
        <table id="table">  
            <thead>学生信息表</thead>  
            <tr>                <th>编号</th>  
                <th>姓名</th>  
                <th>性别</th>  
                <th>操作</th>  
            </tr>  
            <tr>                <td>1</td>  
                <td>令狐冲</td>  
                <td>男</td>  
                <td><a href="javascript:void(0);" onclick="delTr(this)">删除</a></td>  
            </tr>  
            <tr>                <td>2</td>  
                <td>任我行</td>  
                <td>男</td>  
                <td><a href="javascript:void(0);" onclick="delTr(this)">删除</a></td>  
            </tr>  
            <tr>                <td>3</td>  
                <td>岳不群</td>  
                <td>？</td>  
                <td><a href="javascript:void(0);" onclick="delTr(this)">删除</a></td>  
            </tr>  
        </table>  
    </div>  
    <script>        /**  
         * 添加  
         * 1. 给添加按钮，绑定点击事件  
         * 2. 获取文本款的内容  
         * 3. 创建tr、td，设置td的文本为文本框的内容  
         * 4. 获取table，将tr添加到table红  
         */  
  
        var btn = document.getElementById("btn");  
        btn.onclick = function (){  
            //获取填入的值  
            var id = document.getElementById("id").value;  
            var name = document.getElementById("name").value;  
            var gender = document.getElementById("gender").value;  
            //获取table  
			var table = document.getElementById("table");  
			table.innerHTML += "<tr>\n" +  
    "                <td>"+id+"</td>\n" +  
    "                <td>"+name+"</td>\n" +  
    "                <td>"+gender+"</td>\n" +  
    "                <td><a href=\"javascript:void(0);\" onclick=\"delTr(this)\">删除</a></td>\n" +  
    "            </tr>"
        }  
        /**  
         * 删除功能  
         * 1.确定点击的是哪一个超链接  
         */  
        function delTr(obj) {  
            var tr_node = obj.parentNode.parentNode;  
            var table = obj.parentNode.parentNode.parentNode;  
            table.removeChild(tr_node);  
        }  
    </script>  
</body>  
</html>
```

### 7.4 HTML DOM

1. 标签体的设置和获取：`innerHTML`
2. 使用`html元素对象的属性`
3. `控制元素样式`
	1. 使用元素的style属性来设置
		如：
			 //修改样式方式1
			div1.style.border = "1px solid red";
			div1.style.width = "200px";
			//font-size--> fontSize
			div1.style.fontSize = "20px";
	2. 提前定义好类选择器的样式，通过元素的className属性来设置其class属性值。

## 8 事件监听机制

### 8.1 概念

* 概念：某些组件被执行了某些操作后，触发某些代码的执行。	
	* `事件：某些操作`。如： 单击，双击，键盘按下了，鼠标移动了
	* `事件源：组件`。如： 按钮 文本输入框...
	* `监听器：代码`。
	* 注册监听：将事件，事件源，监听器结合在一起。 当事件源上发生了某个事件，则触发执行某个监听器代码。

### 8.2 事件绑定

- 方式一：通过`HTML标签中的事件属性`进行绑定
```html
<input type="button" onclick="on()" value="按钮1">

<script>
	function on(){
		alert('我被点击了');
	}
</script>
```

- 方式二：通过DOM元素属性绑定
```html
<input type="button" id="btn" value="按钮2">

<script>
	document.getElementById('btn').onclick = function(){
		alert('我被点击了');
	}
</script>
```


### 8.3 常见事件

1. 点击事件：
	1. `onclick`：单击事件
	2. ondblclick：双击事件
2. 焦点事件
	1. `onblur`：失去焦点
		- 一般用于表单验证
	2. `onfocus`:元素获得焦点。

3. 加载事件：
	1. `onload`：一张页面或一幅图像完成加载。

4. 鼠标事件：
	1. onmousedown	鼠标按钮被按下。
		- 定义方法时，定义一个形参，接收event对象
		- event对象的button属性可以获取鼠标那个按钮被点击了
	1. onmouseup	鼠标按键被松开。
	2. onmousemove	鼠标被移动。
	3. `onmouseover`	鼠标移到某元素之上。
	4. `onmouseout`	鼠标从某元素移开。

5. 键盘事件：
	1. `onkeydown`	某个键盘按键被按下。	
	2. onkeyup		某个键盘按键被松开。
	3. onkeypress	某个键盘按键被按下并松开。

6. 选择和改变
	1. `onchange`	域的内容被改变。
	2. onselect	文本被选中。

7. 表单事件：
	1. `onsubmit`	确认按钮被点击。
		- 可以阻止表单的提交
		- 方法返回false，则表单被阻止提交
	1. onreset	重置按钮被点击。


- `注意两个阻止提交的写法`

```html
<form action="#" id="form" onclick="return checkForm();">

//方式1：
document.getElementById("form").onsubmit=function(){
	return false;//阻止提交
}

//方式2：
function checkFrom(){
	return false;
}
```