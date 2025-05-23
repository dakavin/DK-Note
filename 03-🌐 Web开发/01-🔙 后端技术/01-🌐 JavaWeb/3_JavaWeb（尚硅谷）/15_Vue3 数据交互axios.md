
## 1 预备知识——Promise

### 1.1 普通函数和回调函数

> `普通函数: 正常调用的函数,一般函数执行完毕后才会继续执行下一行代码`

``` html
<script>
    let fun1 = () =>{
        console.log("fun1 invoked")
    }
    // 调用函数
    fun1()
    // 函数执行完毕,继续执行后续代码
    console.log("other code processon")
</script>
```

> `回调函数: 一些特殊的函数,表示未来才会执行的一些功能,后续代码不会等待该函数执行完毕就开始执行了`

```html
<script>
    // 设置一个2000毫秒后会执行一次的定时任务
    setTimeout(function (){
        console.log("setTimeout invoked")
    },2000)
    console.log("other code processon")
</script>
```

### 1.2 Promise简介

> 前端中的异步编程技术，类似Java中的多线程+线程结果回调！

+ Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

+ 所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

> `Promise`对象有以下两个特点。

- Promise对象代表一个异步操作，有三种状态：
	- `Pending`（进行中）
	- `Resolved`（已完成，又称 Fulfilled）
	- `Rejected`（已失败）。
- 只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从`Pending`变为`Resolved`和从`Pending`变为`Rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。
### 1.3 Promise 基本用法

> `ES6规定，Promise对象是一个构造函数，用来生成Promise实例`。

```js
<script>  
	  
   /*    
	1.实例化promise对象,并且执行(类似Java创建线程对象,并且start)  
	参数: resolve,reject随意命名,但是一般这么叫!  
	参数: resolve,reject分别处理成功和失败的两个函数! 成功resolve(结果)  失败reject(结果)  
	参数: 在function中调用这里两个方法,那么promise会处于两个不同的状态  
	状态: promise有三个状态  
			pending   正在运行  
			resolved  内部调用了resolve方法  
			rejected  内部调用了reject方法  
	参数: 在第二步回调函数中就可以获取对应的结果   
	*/  
	let promise =new Promise(function(resolve,reject){  
		console.log("promise do some code ... ...")  
		//resolve("promise success")  
		reject("promise fail")  
	})  
	console.log('other code1111 invoked')  
	//2.获取回调函数结果  then在这里会等待promise中的运行结果,但是不会阻塞代码继续运行  
	promise.then(  
		function(value){console.log(`promise中执行了resolve:${value}`)},  
		function(error){console.log(`promise中执行了reject:${error}`)}  
	)  
	// 3 其他代码执行     
	console.log('other code2222 invoked')  
</script>
```
### 1.4 Promise catch()

> `Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。

```html
<script>
    let promise =new Promise(function(resolve,reject){
        console.log("promise do some code ... ...")
        // 故意响应一个异常对象
        throw new Error("error message")
    })
    console.log('other code1111 invoked')
    /* 
        then中的reject()的对应方法可以在产生异常时执行,接收到的就是异常中的提示信息
        then中可以只留一个resolve()的对应方法,reject()方法可以用后续的catch替换
        then中的reject对应的回调函数被后续的catch替换后,catch中接收的数据是一个异常对象
        */
    promise.then(
        function(resolveValue){console.log(`promise中执行了resolve:${resolveValue}`)}
        // 后续方法不需要了，因为被catch代替了
        //function(rejectValue){console.log(`promise中执行了reject:${rejectValue}`)}
    ).catch(
        function(error){console.log(error)} 
    )
    console.log('other code2222 invoked')
</script>
```
### 1.5 async 和 await的使用

> `async和await是ES6中用于处理异步操作的新特性`。通常，异步操作会涉及到Promise对象，而async/await则是在Promise基础上提供了更加直观和易于使用的语法。

>  `async 用于标识函数的`

1. async标识函数后,函数的返回值会变成一个promise对象
2. 如果函数内部返回的数据是一个非promise对象,async函数的结果会返回一个成功状态 promise对象
3. 如果函数内部返回的是一个promise对象,则async函数返回的状态与结果由该对象决定
4. 如果函数内部抛出的是一个异常,则async函数返回的是一个失败的promise对象

```js
<script>

	async function fun1(){
		//内部返回的数据是一个非promise对象,async函数的结果会返回一个成功状态 promise对象
		//return 10
		
		//内部抛出的是一个异常,则async函数返回的是一个失败的promise对象
		//throw new Error("something wrong")
		
		//内部返回的是一个promise对象,则async函数返回的状态与结果由该对象决定
		let promise = Promise.reject("heihei")
		return promise
	}
	
	//async标识函数后,async函数的返回值会变成一个promise对象
	let promise =fun1()

	promise.then(
		function(value){
			console.log("success:"+value)
		}
	).catch(
		function(value){
			console.log("fail:"+value)
		}
	)
</script>
```

> `await 用于标识promise对象/其他值`

1. await右侧的表达式一般为一个promise对象,但是也可以是一个其他值
2. `如果表达式是promise对象,await返回的是promise成功的值`
3. await会等右边的promise对象执行结束,然后再获取结果,后续代码也会等待await的执行
4. 如果表达式是其他值,则直接返回该值
5. `await必须在async函数中,但是async函数中可以没有await`
6. 如果await右边的promise失败了,就会抛出异常,需要通过 try ... catch捕获处理

```js
<script>
	async function fun1(){
		return 10
	}

	async function fun2(){
		try{
			
			// await右侧的表达式是一个promise对象
			// 表达式是promise对象,await返回的是promise成功的值
			//await会等右边的promise对象执行结束,然后再获取结果,后续代码也会等待await的执行
			//如果表达式是其他值,则直接返回该值
			let res = await fun1()
			
			//如果await右边的promise失败了,就会抛出异常,可以通过 try ... catch捕获处理
			//let res = await Promise.reject("something wrong")
		}catch(e){
			console.log("catch got:"+e)   
		}
		
		console.log("await got:"+res)
	}

	fun2()
</script>
```
## 2 Axios介绍

> `ajax`

- AJAX = Asynchronous(`eɪˈsɪŋkrənəs`) JavaScript and XML（`异步的 JavaScript 和 XML`）。
- AJAX 不是新的编程语言，而是一种使用现有标准的新方法。
- AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。
- AJAX 不需要任何浏览器插件，但需要用户允许 JavaScript 在浏览器上执行。
- XMLHttpRequest 只是实现 Ajax 的一种方式。

**ajax工作原理：**![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2089e8c49dca533fadbc6bba5e8fac4f.png)
原生**javascript方式进行ajax(了解):**
```js
<script>
  function loadXMLDoc(){
    var xmlhttp;
    if (window.XMLHttpRequest){
      //  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
      xmlhttp=new XMLHttpRequest();
    }
    else{
      // IE6, IE5 浏览器执行代码
      xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
    }
      // 设置回调函数处理响应结果
    xmlhttp.onreadystatechange=function(){
      if (xmlhttp.readyState==4 && xmlhttp.status==200)
      {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
      }
    }
      // 设置请求方式和请求的资源路径
    xmlhttp.open("GET","/try/ajax/ajax_info.txt",true);
      // 发送请求
    xmlhttp.send();
  }
</script>  
```

>  什么是axios  官网介绍:https://axios-http.com/zh/docs/intro

+ Axios 是一个基于 [*promise*](https://javascript.info/promise-basics "promise") 网络请求库，作用于[node.js](https://nodejs.org/ "node.js") 和浏览器中。 它是 [*isomorphic*](https://www.lullabot.com/articles/what-is-an-isomorphic-application "isomorphic") 的(即同一套代码可以运行在浏览器和node.js中)。
+ `在服务端它使用原生 node.js http 模块, 而在客户端 (浏览端) 则使用 XMLHttpRequests`。

+ 特性
	- 从浏览器创建 [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest "XMLHttpRequests")
	+ 从 node.js 创建 [http](http://nodejs.org/api/http.html "http") 请求
	+ 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise "Promise") API
	+ 拦截请求和响应
	+ 转换请求和响应数据
	+ 取消请求
	+ 自动转换JSON数据
	+ 客户端支持防御[XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery "XSRF")
## 3 Axios 入门案例

### 3.1 案例需求

`请求后台获取随机土味情话`

+ 请求的url
``` http
https://api.uomg.com/api/rand.qinghua?format=json    
或者使用
http://forum.atguigu.cn/api/rand.qinghua?format=json
```

+ 请求的方式
``` http
GET/POST
```

+ 数据返回的格式
```json
{"code":1,"content":"我努力不是为了你而是因为你。"}
```

### 3.2 准备项目


```javascript
npm create vite
npm install 
/*npm install vue-router@4 --save
npm install pinia */
```

### 3.3 安装axios

```shell
npm install axios
```

### 3.4 设计页面（App.Vue）

```vue
<script setup>
  import axios from 'axios'
  import { onMounted,reactive } from 'vue';
  
  let jsonData = reactive({code:1,content:'我的努力不是为了你而是因为你'})

  let getLoveMsg = ()=>{
    axios({
      //请求方式
      //post，并且使用data，数据在请求体中
      //get，并且使用params，数据在url中拼接
      method:'post',
      //请求url
      url:"https://api.uomg.com/api/rand.qinghua?format=json",
      data:{
	      //请求的方式为post，data下的数据以JSON字符串放入请求体，，否则以k-v形式放入url后（data需要改为params）
	      username:'123456'
	  }
  
      //then表示响应成功时要执行的函数
    }).then(
      (response)=>{
        console.log('response:',response)
        Object.assign(jsonData,response.data)
      }

      //catch表示响应失败要执行的函数
    ).catch(
      (error)=>{console.log(error)}
    )
  }

  //通过onMounted生命周期函数，自动加载方法
  onMounted(()=>{getLoveMsg()})
</script>

<template>
    <div>
      <h1>今日土味情话:{{jsonData.content}}</h1>
      <button  @click="getLoveMessage">获取今日土味情话</button>
    </div>
</template>

<style scoped>
</style>
```

- 启动测试

```shell
npm run dev
```

### 3.5 异步响应的数据结构

+ `响应的数据是经过包装返回的！` 一个请求的响应包含以下信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/08fbfb32b33ed31f3e5a2ad1c574ad40.png)
```json
{
  // `data` 由服务器提供的响应
  data: {},
  
  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,
  
  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',
  
  // `headers` 是服务器响应头
  // 所有的 header 名称都是小写，而且可以使用方括号语法访问
  // 例如: `response.headers['content-type']`
  headers: {},
  
  // `config` 是 `axios` 请求的配置信息
  config: {},
  
  // `request` 是生成此响应的请求
  // 在node.js中它是最后一个ClientRequest实例 (in redirects)，
  // 在浏览器中则是 XMLHttpRequest 实例
  request: {}
}
```

+ then取值
```javascript
then(function (response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
});
```


### 3.6 通过async和await处理异步请求

- `此处不建议看，直接看后面的axios的get和post方法即可`

```html
<script setup type="module">
  import axios from 'axios'
  import { onMounted,reactive } from 'vue';
    
  let jsonData =reactive({code:1,content:'我努力不是为了你而是因为你'})

  let getLoveWords = async ()=>{
    //返回的是一个promise对象
    return await axios({
      method:"post",
      url:"https://api.uomg.com/api/rand.qinghua?format=json",
      data:{
        username:"123456"
      }
    })
  }

  let getLoveMessage = async ()=>{
   	 let {data}  = await getLoveWords()
     Object.assign(jsonData,data)
  }

  /* 通过onMounted生命周期,自动加载一次 */
  onMounted(()=>{
    getLoveMessage()
  })
</script>

<template>
    <div>
      <h1>今日土味情话:{{jsonData.content}}</h1>
      <button  @click="getLoveMessage">获取今日土味情话</button>
    </div>
</template>

<style scoped>
</style>

```

### 3.7 axios在发送异步请求时的可选配置

```json
{
  // `url` 是用于请求的服务器 URL
  url: '/user',
  
  // `method` 是创建请求时使用的方法
  method: 'get', // 默认值
  
  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',
  
  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 它只能用于 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 数组中最后一个函数必须返回一个字符串， 一个Buffer实例，ArrayBuffer，FormData，或 Stream
  // 你可以修改请求头。
  transformRequest: [function (data, headers) {
    // 对发送的 data 进行任意转换处理
    return data;
  }],
  
  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对接收的 data 进行任意转换处理
    return data;
  }],
  
  // 自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},
  
  // `params` 是与请求一起发送的 URL 参数
  // 必须是一个简单对象或 URLSearchParams 对象
  params: {
    ID: 12345
  },
  
  // `paramsSerializer`是可选方法，主要用于序列化`params`
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },
  
  // `data` 是作为请求体被发送的数据
  // 仅适用 'PUT', 'POST', 'DELETE 和 'PATCH' 请求方法
  // 在没有设置 `transformRequest` 时，则必须是以下类型之一:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属: FormData, File, Blob
  // - Node 专属: Stream, Buffer
  data: {
    firstName: 'Fred'
  },
  
  // 发送请求体数据的可选语法
  // 请求方式 post
  // 只有 value 会被发送，key 则不会
  data: 'Country=Brasil&City=Belo Horizonte',
  
  // `timeout` 指定请求超时的毫秒数。
  // 如果请求时间超过 `timeout` 的值，则请求会被中断
  timeout: 1000, // 默认值是 `0` (永不超时)
  
  // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // default
  
  // `adapter` 允许自定义处理请求，这使测试更加容易。
  // 返回一个 promise 并提供一个有效的响应 （参见 lib/adapters/README.md）。
  adapter: function (config) {
    /* ... */
  },
  
  // `auth` HTTP Basic Auth
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },
  
  // `responseType` 表示浏览器将要响应的数据类型
  // 选项包括: 'arraybuffer', 'document', 'json', 'text', 'stream'
  // 浏览器专属：'blob'
  responseType: 'json', // 默认值
  
  // `responseEncoding` 表示用于解码响应的编码 (Node.js 专属)
  // 注意：忽略 `responseType` 的值为 'stream'，或者是客户端请求
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // 默认值
  
  // `xsrfCookieName` 是 xsrf token 的值，被用作 cookie 的名称
  xsrfCookieName: 'XSRF-TOKEN', // 默认值
  
  // `xsrfHeaderName` 是带有 xsrf token 值的http 请求头名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认值
  
  // `onUploadProgress` 允许为上传处理进度事件
  // 浏览器专属
  onUploadProgress: function (progressEvent) {
    // 处理原生进度事件
  },
  
  // `onDownloadProgress` 允许为下载处理进度事件
  // 浏览器专属
  onDownloadProgress: function (progressEvent) {
    // 处理原生进度事件
  },
  
  // `maxContentLength` 定义了node.js中允许的HTTP响应内容的最大字节数
  maxContentLength: 2000,
  
  // `maxBodyLength`（仅Node）定义允许的http请求内容的最大字节数
  maxBodyLength: 2000,
  
  // `validateStatus` 定义了对于给定的 HTTP状态码是 resolve 还是 reject promise。
  // 如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，
  // 则promise 将会 resolved，否则是 rejected。
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认值
  },
  
  // `maxRedirects` 定义了在node.js中要遵循的最大重定向数。
  // 如果设置为0，则不会进行重定向
  maxRedirects: 5, // 默认值
  
  // `socketPath` 定义了在node.js中使用的UNIX套接字。
  // e.g. '/var/run/docker.sock' 发送请求到 docker 守护进程。
  // 只能指定 `socketPath` 或 `proxy` 。
  // 若都指定，这使用 `socketPath` 。
  socketPath: null, // default
  
  // `httpAgent` and `httpsAgent` define a custom agent to be used when performing http
  // and https requests, respectively, in node.js. This allows options to be added like
  // `keepAlive` that are not enabled by default.
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),
  
  // `proxy` 定义了代理服务器的主机名，端口和协议。
  // 您可以使用常规的`http_proxy` 和 `https_proxy` 环境变量。
  // 使用 `false` 可以禁用代理功能，同时环境变量也会被忽略。
  // `auth`表示应使用HTTP Basic auth连接到代理，并且提供凭据。
  // 这将设置一个 `Proxy-Authorization` 请求头，它会覆盖 `headers` 中已存在的自定义 `Proxy-Authorization` 请求头。
  // 如果代理服务器使用 HTTPS，则必须设置 protocol 为`https`
  proxy: {
    protocol: 'https',
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },
  
  // see https://axios-http.com/zh/docs/cancellation
  cancelToken: new CancelToken(function (cancel) {
  }),
  
  // `decompress` indicates whether or not the response body should be decompressed 
  // automatically. If set to `true` will also remove the 'content-encoding' header 
  // from the responses objects of all decompressed responses
  // - Node only (XHR cannot turn off decompression)
  decompress: true // 默认值
}
```
## 4 Axios get和post方法！！

> 配置添加语法

``` javascript
axios.get(url[, config])

axios.get(url,{
   上面指定配置key:配置值,
   上面指定配置key:配置值
})

axios.post(url[, data[, config]])

axios.post(url,{key:value //此位置数据，没有空对象即可{}},{
   上面指定配置key:配置值,
   上面指定配置key:配置值
})
```

> 测试get参数

``` html
<script setup type="module">
  import axios from 'axios'
  import { onMounted,ref,reactive,toRaw } from 'vue';
  let jsonData =reactive({code:1,content:'我努力不是为了你而是因为你'})

  let getLoveWords= async ()=>{
    try{
      return await axios.get(
        'https://api.uomg.com/api/rand.qinghua',
        {
			//向url后添加键值对参数
			params:{format:'json',username:'zhangsan'},

			//设置请求头
			headers:{'Accept':'application/json, text/plain, text/html,*/*'}
        }
      )
    }catch (e){
      return await e
    }
  }

  let getLoveMessage =()=>{
     let {data}  = await getLoveWords()
     Object.assign(message,data)
  }
  /* 通过onMounted生命周期,自动加载一次 */
  onMounted(()=>{
    getLoveMessage()
  })
</script>

<template>
    <div>
      <h1>今日土味情话:{{jsonData.content}}</h1>
      <button  @click="getLoveMessage">获取今日土味情话</button>
    </div>
</template>

<style scoped>
</style>

```

> 测试post参数

```html
<script setup type="module">
  import axios from 'axios'
  import { onMounted,ref,reactive,toRaw } from 'vue';
  let jsonData =reactive({code:1,content:'我努力不是为了你而是因为你'})

  let getLoveWords= async ()=>{
    try{
      return await axios.post(
        'https://api.uomg.com/api/rand.qinghua',
        //请求体中的JSON数据
        {
            username:'zhangsan',
            password:'123456'
        },
        
        // 其他参数
        {
         	params:{// url上拼接的键值对参数
            	format:'json',
          	},
          	headers:{// 请求头
            	'Accept' : 'application/json, text/plain, text/html,*/*',
            	'X-Requested-With': 'XMLHttpRequest'
          	}
        }
      )
    }catch (e){
      return await e
    }
  }

  let getLoveMessage =()=>{
     let {data}  = await getLoveWords()
     Object.assign(message,data)
  }
  /* 通过onMounted生命周期,自动加载一次 */
  onMounted(()=>{
    getLoveMessage()
  })
</script>

<template>
    <div>
      <h1>今日土味情话:{{jsonData.content}}</h1>
      <button  @click="getLoveMessage">获取今日土味情话</button>
    </div>
</template>

<style scoped>
</style>

```
## 5 Axios 拦截器

> 如果想在axios发送请求之前,或者是数据响应回来在执行then方法之前做一些额外的工作,可以通过拦截器完成

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

+ `定义src/axios.js提取拦截器和配置语法`
```javascript
import axios from 'axios'

//  创建instance实例
const instance = axios.create({
	// 使用instance发任何请求，都会在前面拼接baseURL
    baseURL:'https://api.uomg.com',
    timeout:10000
})

//  添加请求拦截
instance.interceptors.request.use(
    // 设置请求头配置信息
    config=>{
        //处理指定的请求头
        console.log("before request")
        config.headers.Accept = 'application/json, text/plain, text/html,*/*'
    
        //设置完毕之后，必须返回
        return config
    },
    // 设置请求错误处理函数
    error=>{
        console.log("request error")
        return Promise.reject(error)
    }
)
// 添加响应拦截器
instance.interceptors.response.use(
    // 设置响应正确时的处理函数
    response=>{
        console.log("after success response")
        console.log(response)
        return response
    },
    // 设置响应异常时的处理函数
    error=>{
        console.log("after fail response")
        console.log(error)
        return Promise.reject(error)
    }
)
// 默认导出
export default instance
```

+ App.vue
```html
<script setup type="module">
  // 导入我们自己定义的axios.js文件,而不是导入axios依赖  
  import axios from './axios.js'
  
  import { onMounted,ref,reactive,toRaw } from 'vue';
  
  let jsonData =reactive({code:1,content:'我努力不是为了你而是因为你'})

  let getLoveWords= ()=>{
    try{
      return axios.post('api/rand.qinghua',
        //请求体中的JSON数据
        {},
        // 其他键值对参数
        {params:{format:'json'}}
      )
    }catch (e){
      return await e
    }
  }

  let getLoveMessage = async ()=>{
    // 这里返回的是一个fullfilled状态的promise
     await getLoveWords().then(
        (response) =>{
          console.log("after getloveWords")
          console.log(response)
          Object.assign(jsonData,response.data)
        }
    )
  }
  /* 通过onMounted生命周期,自动加载一次 */
  onMounted(()=>{
    getLoveMessage()
  })
</script>

<template>
    <div>
      <h1>今日土味情话:{{jsonData.content}}</h1>
      <button  @click="getLoveMessage">获取今日土味情话</button>
    </div>
   
</template>

<style scoped>
</style>
```
