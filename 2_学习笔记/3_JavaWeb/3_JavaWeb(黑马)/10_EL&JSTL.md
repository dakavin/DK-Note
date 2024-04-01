
## 1 JSP剩余内容

### 1.1 指令

* `作用`：用于配置JSP页面，导入资源文件

* `格式`：
	`<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>`
	
* `分类`：
1. `page		： 配置JSP页面的`
	* contentType：等同于response.setContentType()
		1. 设置响应体的mime类型以及字符集
		2. 设置当前jsp页面的编码（只能是高级的IDE才能生效，如果使用<font color="#00b050">低级工具，则需要设置pageEncoding属性设置当前页面的字符集</font>）
	* import：导包
	* errorPage：当前页面发生异常后，会自动跳转到指定的错误页面
	* isErrorPage：标识当前也是是否是错误页面。
		* true：是，可以使用内置对象exception
		* false：否。默认值。不可以使用内置对象exception
2. `include	： 页面包含的。导入页面的资源文件`
```jsp
<%@include file="top.jsp"%>
```

3. `taglib	： 导入资源` ,  prefix：前缀，自定义的
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```


### 1.2 注释

1. html注释：
	`<!-- -->`：只能注释html代码片段
2. jsp注释：推荐使用
	`<%-- --%>`：可以注释所有

### 1.3 内置对象

* 在jsp页面中不需要创建，直接使用的对象
* 一共有9个：

|变量名|真实类型|作用|
|---|---|--------|
|pageContext|PageContext|当前页面共享数据，还可以获取其他八个内置对象
|request|HttpServletRequest|一次请求访问的多个资源(转发)
|session|HttpSession|一次会话的多个请求间
|application|ServletContext|所有用户间共享数据
|response|HttpServletResponse|响应对象
| page	|Object|当前页面(Servlet)的对象  this
|out|JspWriter|输出对象，数据输出到页面上
|config|ServletConfig|Servlet的配置对象
|exception|Throwable|异常对象




## 2 MVC开发模式

- `注意不是设计模式`

### 2.1 jsp演变历史

1. 早期只有servlet，只能使用response输出标签数据，非常麻烦
2. 后来又jsp，简化了Servlet的开发，如果过度使用jsp，在jsp中即写大量的java代码，有写html表，造成难于维护，难于分工协作
3. 再后来，java的web开发，借鉴mvc开发模式，使得程序的设计更加合理性

### 2.2 MVC

1. `M：Model，模型。JavaBean`
	* 完成具体的业务操作，如：查询数据库，封装对象
2. `V：View，视图。JSP`
	* 展示数据
3. `C：Controller，控制器。Servlet`
	* 获取用户的输入
	* 调用模型
	* 将数据交给视图进行展示
- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c6cfbf862b1c61efe703f36a4c5971f.png)



### 2.3 优缺点

- 优点：
	1. 耦合性低，方便维护，可以利于分工协作
	2. 重用性高

- 缺点：
	1. 使得项目架构变得复杂，对开发人员要求高


## 3 EL表达式

### 3.1 概念：Expression Language 表达式语言

### 3.2 作用：替换和简化jsp页面中java代码的编写

### 3.3 语法：${表达式}

### 3.4 注意

* jsp默认支持el表达式的。如果要忽略el表达式
	1. 设置jsp中page指令中：isELIgnored="true" 忽略当前jsp页面中所有的el表达式
	2. `\${表达式}` ：在$符号前加上一个`“\”`忽略当前这个el表达式

### 3.5 使用

- `运算`
	1. 算数运算符： + - * /(div) %(mod)
	2. 比较运算符： > < >= <= == !=
	3. 逻辑运算符： &&(and) ||(or) !(not)
	4. 空运算符： empty
		* 功能：<font color="#00b050">用于判断字符串、集合、数组对象是否为null或者长度是否为0</font>
		* ${empty list}:判断字符串、集合、数组对象是否为null或者长度为0
		* ${not empty str}:表示判断字符串、集合、数组对象是否不为null 并且 长度>0

- `获取值`
- el表达式只能从域对象中获取值，语法规则如下
	1. `${域名称.键名}`：从指定域中获取指定键的值
		* 域名称：
			1. pageScope		--> pageContext
			2. requestScope 	--> request
			3. sessionScope 	--> session
			4. applicationScope --> application（ServletContext）
		* ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a3a2b825ef7b50056968b664d2b5d460.png)


	2. `${键名}`：表示依次从最小的域中查找是否有该键对应的值，直到找到为止。
	3. `获取对象、List集合、Map集合的值`
		- 对象：${域名称.键名.属性名}，本质上会去调用对象的getter方法![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9e448eef548aadd448f6aa1b206bdc9.png)
		- List集合：${域名称.键名[索引]}
		- Map集合：
			* ${域名称.键名.key名称}
			* ${域名称.键名["key名称"]}

- `隐式对象`：
	* el表达式中有11个隐式对象
	* pageContext：
		* 获取jsp其他八个内置对象
			* ${pageContext.request.contextPath}：动态获取虚拟目录


## 4 JSTL

### 4.1 概念

- `JavaServer Pages Tag Library`  JSP标准标签库
* 是由Apache组织提供的开源的免费的jsp标签    <标签>

### 4.2 作用

- `用于简化和替换jsp页面上的java代码`

### 4.3 使用步骤

1. 导入jstl相关jar包
2. 引入标签库：taglib指令：  `<%@ taglib %>`
3. 使用标签

### 4.4 常用的JSTL标签

1. `if`:相当于java代码的`if语句`
	1. 属性：
		* test 必须属性，接受boolean表达式
			* 如果表达式为true，则显示if标签体内容，如果为false，则不显示标签体内容
			* 一般情况下，test属性值会结合el表达式一起使用
	 2. 注意：
		 * c:if标签没有else情况，想要else情况，则可以再定义一个c:if标签
2. `choose`:相当于java代码的`switch语句`
	1. 使用choose标签声明         			相当于switch声明
	2. 使用when标签做判断         			相当于case
	3. 使用otherwise标签做其他情况的声明    	相当于default

3. `foreach`:相当于java代码的`for语句`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8404cb6c125ac7067823e19c616757c0.png)

### 4.5 练习

* 需求：在request域中有一个存有User对象的List集合。需要使用jstl+el将list集合数据展示到jsp页面的表格table中

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <title>jtsl小练习</title>  
</head>  
<body>  
<%--    在request域中有一个存有User对象的List集合。  
        需要使用jstl+el将list集合数据展示到jsp页面的表格table中  
--%>  
    <%  
        List<User> list = new ArrayList<>();  
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");  
        for (int i = 1; i <=3; i++) {  
            try {  
                list.add(new User("user"+i,i+20,sdf.parse("1999-05-0"+i)));  
            } catch (ParseException e) {  
                throw new RuntimeException(e);  
            }  
        }  
        request.setAttribute("list",list);  
    %>  
  
    <table border="1px" width="500px" align="center">  
        <tr>  
            <th>序号</th>  
            <th>姓名</th>  
            <th>年龄</th>  
            <th>生日</th>  
        </tr>  
        <c:forEach items="${list}" var="list" varStatus="s">  
            <c:if test="${s.count%2==0}">  
                <tr bgcolor="#ffc0cb">  
                    <td>${s.count}</td>  
                    <td>${list.name}</td>  
                    <td>${list.age}</td>  
                    <td>${list.birStr}</td>  
                </tr>  
            </c:if>  
            <c:if test="${s.count%2!=0}">  
                <tr bgcolor="#00ffff">  
                    <td>${s.count}</td>  
                    <td>${list.name}</td>  
                    <td>${list.age}</td>  
                    <td>${list.birStr}</td>  
                </tr>  
            </c:if>  
        </c:forEach>  
    </table>  
  
</body>  
</html>
```

## 5 三层架构:软件设计架构

1. 界面层(表示层)：用户看的得界面。用户可以通过界面上的组件和服务器进行交互
2. 业务逻辑层：处理业务逻辑的。
3. 数据访问层：操作数据存储文件。
4. ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ffa6ab2bb470ca5bf9a0784d2e773d37.png)



## 6 案例

### 6.1 开发流程

1. `需求`：用户信息的增删改查操作
2. `设计`：
	1. `技术选型`：Servlet+JSP+MySQL+JDBCTempleat+Duird+BeanUtilS+tomcat
	2. `数据库设计`：
```mysql
create database day17; -- 创建数据库
use day17; 			   -- 使用数据库
create table user(   -- 创建表
	id int primary key auto_increment,
	name varchar(20) not null,
	gender varchar(5),
	age int,
	address varchar(32),
	qq	varchar(20),
	email varchar(50)
);
```

3. `开发`：
	- 环境搭建
		- 创建数据库环境
		- 创建项目，导入需要的jar包
			- Servlet ---> Tomcat中的jar包
			- JSP ---> jstl标签所需要的jar包
			- JDBC ---> 数据库连接及连接池，操作工具类，
			- 操作JavaBean ---> BeanUtils
		- 导入前端已经编写好的静态页面

	- 编码
	- 页面层
		- 主界面index.jsp
			- 有一个a标签跳转到list.jsp页面
		- 查询所有用户信息页面list.jsp
			- 导入数据库中的所有用户信息，可以使用EL和JSTL在JSP页面直接获取所有用户的集合，并遍历
			- 其中所有用户的集合通过逻辑层进行获取
			- 可以对用户进行修改和删除操作
		- 修改用户信息页面update.jsp
			- 简单业务逻辑操作即可
			- `注意用户名不用修改的情况，用户名如何获取值`
		- 删除用户信息
			- 简单业务逻辑操作即可
			- `注意删除保护机制`，`还有用户名应该怎么传值问题`
			- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d1c9b5706c7577fa69570275d01404da.png)


		- 增加用户信息页面add.jsp
			- 简单业务逻辑操作即可
		- 管理员登陆页面login.jsp
			- 用户登陆页面
			- 业务逻辑为通过输入信息判断用户是否正确

	- 业务逻辑层（UserService、UserServiceImpl）
		- `简单功能`
		- 获取所有用户的集合（查询多个用户）
		- 修改用户
		- 增加用户
		- 删除用户
		- 用户登陆
		- `复杂功能`
		- 删除选中
		- 分页功能
		- 复杂条件查询

	- 数据访问层（BaseDao、UserDao、UserDaoImpl）
		- 查询单个用户
		- 查询多个用户
		- 增加用户
		- 删除用户
		- 修改用户
		
1. `测试`
2. `部署运维`


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/96cf6afea0cd4b38935efaef17af6028.png)




### 6.2 简单功能对比分析及优化
1. 关于登陆功能
	- 应该使用Beanutils，通过map集合和User的对象，给该对象的对应的属性进行赋值，即可！！！
	- 后续涉及到User的属性赋值，都可以使用这个！！！
	- 如：修改用户，删除用户和添加用户等


2. 关于添加功能
	- 使用BeanUtlis
	- 添加一个转行页面，使得添加成功后，`页面自动定时跳转`会list页面，详见[[3_JavaScript#6.4 Location：地址栏对象||定时自动跳转案例]]

3. 关于删除功能
	- bug --- > 通过用户名删除用户，这个方法不可靠，因为用户名不是唯一的，容易同时删除多个用户，应该使用唯一的，即数据库中的id属性！！
	- `删除要增加保护功能`

4. 关于修改功能
	- 需要一个`数据回显的功能`，而不是不能修改用户名
		- 先根据list中的用户id，查询到该用户的信息
		- 然后跳转到修改联系人的页面，并显示原有数据
	- 修改完成后，将数据存入数据库中！！
	- 用户的id，在updata.jsp中使用`隐藏域`进行传递
	- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6ace66a66459d99b49d85d280134195a.png)


### 6.3 复杂功能分析
1. 删除选中
	- 逻辑分析![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a3668dba2806b83dbcc2da54f7fa51d4.png)
	- 难点主要在获取选中的用户的id信息
		1. 可以在table标签外层嵌套一个提交表单，用于获取所有选中的用户id信息；
		2. 将该表单的提交事件，绑定在删除选中的按钮处；
		3. 当删除按钮点击时，就出发该表单的提交；
	- 优化
		- 添加全选功能![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6991b79210fbcb935e9f9e730c061aec.png)


		- 删除保护功能，没有选中就删除的bug处理![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e42ab06fe730ab4a77b09cf99554d890.png)






2. `分页查询`
	- 好处：减轻服务内存的开销，并提升用户的体验![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0fea78bbf610335ded89e31f32299cef.png)
	![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3080f23b31d1eb5ca7dabaee1673d5d8.png)


	- 代码优化：
		1. 默认访问页面的时候，设置为第1页；
		2. 跳转页面都跳转到FindUserByPageServlet去
		3. 页码处高亮![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bd082d947fd14096c008bf0883e3bd7e.png)


		4. 页码功能上下页恢复
		5. 会有-1页超出页面，通过前端改变图片，然后通过后端限制currentPage![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ca1793418b555ea6cdc5d610af041c6.png)



4. 复杂条件查询
	- ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f8bc210a924af8fa412a8eb4e4db1c96.png)


	- 直接在FindUserByPageServlet中修改即可


### 6.4 补充小工具
- 补充功能：在mysql中添加多条数据
	- 使用了循环和case功能
```mysql
delimiter //
create procedure add_user(in count int)
begin
	declare i int default 1;
	while i<=count do
		insert  into `user1`(`name`,`gender`,`age`,`address`,`qq`,`email`) 
		values (
		concat('lisi-',i),
		case  i%2 when 0 Then '女' when 1 Then '男' end,
		i,
		case i%3 when 0 then '北京'  when 1 then '陕西' when 2 then '湖南' end,
		CONCAT(i,i,i,i,i),
		concat(i,'@itcast.cn')
		);
		set i = i +1;
	END While;
end //
delimiter ;

		 
drop procedure add_user
set @count=5;
call add_user(@count);
```

