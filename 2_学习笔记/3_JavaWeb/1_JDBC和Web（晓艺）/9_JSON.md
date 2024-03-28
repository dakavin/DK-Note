## 目录

- [1 JSON介绍](#1%20JSON%E4%BB%8B%E7%BB%8D)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 语法](#1.2%20%E8%AF%AD%E6%B3%95)
	- [1.3 分类](#1.3%20%E5%88%86%E7%B1%BB)
- [2 JSON和Java对象之间的转换](#2%20JSON%E5%92%8CJava%E5%AF%B9%E8%B1%A1%E4%B9%8B%E9%97%B4%E7%9A%84%E8%BD%AC%E6%8D%A2)
	- [2.1 JDK方式——JSONObject、JSONArray](#2.1%20JDK%E6%96%B9%E5%BC%8F%E2%80%94%E2%80%94JSONObject%E3%80%81JSONArray)
	- [2.2 Gson方式](#2.2%20Gson%E6%96%B9%E5%BC%8F)
	- [2.3 fastjson方式](#2.3%20fastjson%E6%96%B9%E5%BC%8F)

## 1 JSON介绍

### 1.1 概念

JSON(JavaScript Object Notation, JS 对象简谱) 是一种轻量级的数据交换格式。

![](file:///C:/Users/mikey/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

 特点：

   JSON现在多用于前后端数据交互

   JSON 比 XML 更小、更快，更易解析。

示例：

{"name":"张三","age":23,"gender":"男"}

### 1.2 语法

1. 基本规则

	- 数据是`由键值对构成`的
	- 多个键值对`由逗号分隔`
	- `{} 花括号表示对象`
	- `[] 中括号表示数组`
	- `"" 双引号内是属性或值`
	- `: 冒号表示后者是前者的值`

### 1.3 分类

`基本类型 : JSONObject`

```json
{
"name": "张三",
"age": 18,
"sex":"男"
} 
```

`数组类型 : JSONArray`

```json
[              
	{
		"name": "张三",
		"age": 18,
		"sex": "男"
	},
	
	{
		"name": "李四",
		"age": 19,
		"sex":"男"
	}
]
```

`对象嵌套 :`

```json
{
    "name": "teacher",
    "computer": {
        "CPU": "intel7",
        "disk": "512G"
    },

    "students": [
        {
            "name": "张三",
            "age": 18,
            "sex": "男"
        },

        {
            "name": "李四",
            "age": 19,
            "sex": "男"
        }
    ]
}
```

## 2 JSON和Java对象之间的转换

### 2.1 JDK方式——JSONObject、JSONArray

转Java 对象

        JSONObject jsonObject = new JSONObject(jsonString);

        ………  详见课堂代码展示

转JSON­  

jsonObject. toString ();

### 2.2 Gson方式

转Java 对象

Gson gson = new Gson();

`gson.fromJson(json, Java类.class)`

转JSON

Gson gson = new Gson();

 `String json = gson.toJson(p);`

### 2.3 fastjson方式

转Java 对象 

`JSON. parseObject (json, Java类.class)`

转JSON

`String json = JSON. toJSONString (p);`

注意：fastjson要求JSON串中属性名一定要是小写