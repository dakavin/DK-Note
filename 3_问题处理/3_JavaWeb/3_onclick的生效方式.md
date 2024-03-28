## 1、在Window.onload内部

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
    <script>        
    window.onload = function () {  
		//给图片绑定，单击事件  
		var img = document.getElementById("checkCode"); 
		img.onclick=function () {  
			//加时间戳，防止本地缓存URL相同的资源  
			var time = new Date().getTime();  
			img.src="/C15/CheckCodeServlet?"+time;  
		}  
		
		// 给超链接绑定单击事件  
		var a = document.getElementById("change");  
		a.onclick=function (){  
			//加时间戳  
			var time = new Date().getTime();  
			img.src="/C15/CheckCodeServlet?"+time;  
		}  
	};  
  
    </script>  
</head>  
<body>  
    <img src="/C15/CheckCodeServlet" id="checkCode">  
    <a href="javascript:void(0);" id="change" >看不清，点击换一张</a>  
</body>  
</html>
```

## 2、在Window.onload外部

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
    <script>         
        function change1() {  
            var img = document.getElementById("checkCode");  
            var time = new Date().getTime();  
            img.src="/C15/CheckCodeServlet?"+time;  
        };  
    </script>  
    
</head>  
<body>  
    <img src="/C15/CheckCodeServlet" id="checkCode" onclick="change1()">  
    <a href="javascript:void(0);" id="change" onclick="change1()">看不清，点击换一张</a>  
</body>  
</html>
```

## 3、原因分析

- 浏览器加载是从上至下按顺序加载，而通过onload可以使代码在其他内容加载完毕后再加载。

- 若function change1写在load内部

- 这里加载到“onclick”这行代码时， change1函数还未被加载到，即还未定义

- 此时无法将change1函数绑定到onclick事件上，所以，当我们点击按钮时，也就不会执行change1函数里的内容