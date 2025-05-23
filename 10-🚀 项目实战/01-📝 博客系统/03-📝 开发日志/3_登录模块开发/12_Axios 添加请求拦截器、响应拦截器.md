
本小节中，我们将为 `axios` 请求库**添加请求拦截器、响应拦截器**，通过它们，就可以==在请求之前，或者请求结束以后定制一些个性化需求==。

## 1 拦截器

关于拦截器的详细文档，可访问官方地址：[https://www.axios-http.cn/docs/interceptors](https://www.axios-http.cn/docs/interceptors) ，官方对它的解释是：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48e3017d9b4f6242c86cef1b1abc1e30.png)


> 在请求或响应被 then 或 catch 处理前拦截它们。

使用示例如下：
```javascript
// 添加请求拦截器 请求发送之前
axios.interceptors.request.use(
  function (config) {
    // 在发送请求之前做些什么
    return config;
  }, 
  function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  }
);

// 添加响应拦截器 数据响应回来
axios.interceptors.response.use(
  function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response;
  }, 
  function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
  }
);
```

## 2 整合到项目中

编辑 `axios.js` 文件，添加上述代码，**注意，需要将实例名改成之前定义好的 `instance`**, 代码如下：
```js
import axios from "axios";

// 创建 Axios 实例
const instance = axios.create({
    baseURL: "/api", // 你的 API 基础 URL
    timeout: 7000, // 请求超时时间
})

// 添加请求拦截器
instance.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error)
});

// 添加响应拦截器
instance.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error)
})

// 暴露出去
export default instance;
```

## 3 定制请求拦截器

上小节中，在用户登录成功后，我们已经将 `Token` 令牌存储到了 `Cookie` 中，但是，==光存储还不够，当我们请求受保护的接口时，需要将它动态添加到请求头中才行==。这个工作，就可以在请求拦截器中来完成。

### 3.1 给请求头添加 Token 令牌

编辑 `axios.js` 中请求拦截器的逻辑，代码如下：

```js
import { getToken } from "@/composables/auth";
// 省略...

// 添加请求拦截器
instance.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    const token = getToken()
    console.log('统一添加请求头中的 Token:' + token)

    // 当 token 不为空时
    if (token) {
        // 添加请求头, key 为 Authorization，value 值的前缀为 'Bearer '
        config.headers['Authorization'] = 'Bearer ' + token
    }

    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error)
});

// 省略...
```

上述代码中，我们在请求拦截器中先是获取了 `Cookie` 中存储的 `Token` 令牌，在不为空的情况下，将其添加到请求头中，按后端的规范，`key` 为 `Authorization`, 值为 `Bearer + 令牌` 的格式。

#### 3.1.1 测试看看

目前只有一个请求登录的功能，我们就拿它来测试，点击页面登录按钮，看下请求头是否带上了 `Token` 令牌：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab73ddf840dda565c7b6c50ce2222704.png)

提前按 _F12_ 打开前端控制台，点击登录，在 _Network_ 选项卡中，选择登录接口，可以看到请求头中成功添加了存储在 `Cookie` 中的 `Token` 令牌了。

## 4 定制响应拦截器

接下来，我们来修改一下响应拦截器，对取参的问题，做一下优化。

### 4.1 取参不方便问题优化

还记得前面请求接口后，获取 `JSON` 返参是如何获取的吗？通过打印 `res` 对象，我们知道想要的参数都在 `data` 字段中：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ba03aa980e9f72b496a5790fa1b86b9a.png)

所以，我们就得以如下方式来获取对应的字段值：

```js
res.data.success
res.data.data
res.data.message
```

你会发现总得跟着一个 `.data` 才能拿到自己想要的，如果能以如下方式, 就方便多了：

```js
res.success
res.data
res.message
```

#### 4.1.1 如何优化？

我们可以对 `axios` 添加响参拦截器来实现以上功能，修改 `axios.js` 文件，修改代码如下：

```js
// 省略...

// 添加响应拦截器
instance.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response.data
}, function (error) {
    // 省略...
})
// 省略...
```

上述代码中，在响应拦截器里，当响应正确的情况下，直接返回了 `response.data` , 这样在后续调用接口时，就可以直接使用 `res.success` 这样的格式来获取想要的字段啦~

修改完成后，改造 `login.vue` 中的请求接口相关代码：
```java
// 调用登录接口
	login(form.username, form.password).then((res) => {
		console.log(res)
		// 判断是否成功
		if (res.success == true) {
			// 省略...
			// 存储 Token 到 Cookie 中
			let token = res.data.token
			// 省略...
		} else {
			// 获取服务端返回的错误消息
			let message = res.message
			// 提示消息
			showMessage(message, 'error')
		}
	})
```

此时登录一下，查看数据，可以发现返回的JSON数据就是我们的Response对象
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aba8d58279ff62df869bd18c2ae3e449.png)

### 4.2 请求失败统一处理（独立404页面）

#### 4.2.1 页面无反馈现象

手动将 `Spring Boot` 服务停掉，当我们点击登录按钮时，你会发现什么也没有发生，用户就会一脸懵逼了，啥情况这是:

![](https://img.quanxiaoha.com/quanxiaoha/169381359515338)

#### 4.2.2 优化它

当通过 `axios` 给后台服务发送请求时，若服务挂了，或者返回的状态码不在 `2xx` 范围内，比如 `401` 未授权等。我们可以统一在响应拦截器中来处理，给用户弹一个消息提示，这样交互上也更加友好。

修改响应拦截器的错误响应模块，代码如下：

```js
import { showMessage} from '@/composables/util'
// 省略...

// 添加响应拦截器
instance.interceptors.response.use(function (response) {
    // 省略...
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么

	// 若后台有错误提示就用提示文字，默认提示为 ‘请求失败’  
	let errorMsg = error.response.data.msg || '请求失败了哈😰'  
	showMessage(errorMsg,'error')  
	  
	return Promise.reject(error)

    return Promise.reject(error)
})

// 省略...
```

再次测试登录，会发现现在能够正确提示用户 _请求失败_ 消息了：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fbebb94712b42661994ebe505915cfed.png)

## 6 结语

本小节中，我们主要给 `axios` 添加了请求拦截器和响应拦截器，在请求拦截器中，我们统一给每次请求的请求头添加了 `Token` 令牌；在响应拦截器中，先是优化了获取参数的格式，然后当请求失败时，统一弹出了消息提示，让交互更加友好。