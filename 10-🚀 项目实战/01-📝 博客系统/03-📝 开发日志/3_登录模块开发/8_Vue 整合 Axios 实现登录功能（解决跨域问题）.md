---
文章标题: "[[8_Vue 整合 Axios 实现登录功能（解决跨域问题）]]" 
文章作者: Dakkk
文章概要: |
  文章详细讲解了Vue 3整合Axios实现用户登录功能，涵盖Axios安装、实例创建、API封装及表单绑定。重点通过Vite代理解决开发环境跨域问题，最终实现登录成功后页面跳转，是前端基础开发实践的典型案例。
文章标签:
- "Vue 3"
- "Axios"
- "登录功能"
- "跨域"
- "Vite代理"
- "API封装"
- "HTTP请求"
- "Vue Router"
相关文章:
- "[[15_Vue3 数据交互axios]]"
- "[[17_Ajax]]"
- "[[3_整合 vue-router 路由管理器]]"
- "[[1_Web乱码和路径问题总结]]"
- "[[12_Vue3 + Vite]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/3_登录模块开发/8_Vue 整合 Axios 实现登录功能（解决跨域问题）.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-24 16:47:15
修改时间: 2024-04-24 20:02:55
---

## 1 前言

Axios 是一个流行的用于发起 HTTP 请求的 JavaScript 库。它可以在浏览器和 Node.js 环境中使用，提供了一种简洁且强大的方式来处理异步网络请求。在项目中整合 Axios 可以帮助你轻松地与后端服务器进行数据交互。

本小节中，我们将**为登录按钮绑定点击事件，并调用后台登录接口，最终实现跳转后台首页的功能**。

## 2 安装 Axios

首先，在项目中安装 Axios。打开控制台，运行以下命令安装 Axios：

```shell
npm install axios
```

## 3 创建 Axios 实例

在开始使用 Axios 发起请求之前，通常需要创建一个 Axios 实例。这样可以为整个应用程序配置默认的请求配置和拦截器。在 `/src` 目录下创建 `axios.js` 文件，内容如下：
```js
import axios from "axios";  
  
// 创建 axios 实例  
const instance = axios.create({  
    // API（接口）的前缀 URL    baseURL:"http://localhost:8080",  
    // 请求超时时间  
    timeout: 7000  
})  
  
// 暴露出去  
export default instance;
```

## 4 封装请求 API

为了更好的维护前端 API 请求，推荐在 `/src` 目录下创建一个 `/api` 文件夹，专门用于放置接口请求相关代码：
- `/admin` : 后台相关接口；
- `/frontend` : 前台相关接口；
  ![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57d2f3880c104a64e9d0ec4c59947116.png)
在 `/admin` 文件夹下创建 `user.js` ，用于统一放置用户相关的接口，如登录接口等，内容如下：
```js
import axios from "@/axios.js";  
  
// 登录接口  
export function login(username,password){  
    return axios.post("/login",{username,password})  
}
```

## 5 完善登录页面

回到登录页，修改内容如下：
```html
<template>
	// 省略...
		<el-form class="w-5/6 md:w-2/5">
			<el-form-item>
				<!-- 输入框组件 -->
				<el-input size="large" v-model="form.username" placeholder="请输入用户名" :prefix-icon="User" clearable />
			</el-form-item>
			<el-form-item>
				<!-- 密码框组件 -->
				<el-input size="large" type="password" v-model="form.password" placeholder="请输入密码" :prefix-icon="Lock" clearable />
		</el-form-item>
			<el-form-item>
				<!-- 登录按钮，宽度设置为 100% -->
				<el-button class="w-full mt-2" size="large" type="primary" @click="onSubmit">登录</el-button>
			</el-form-item>
		</el-form>
	// 省略...
</template>

<script setup>
// 省略...
import { login } from '@/api/admin/user'
import { reactive } from 'vue' 

// 定义响应式的表单对象
const form = reactive({
    username: '',
    password: ''
})

// 登录
const onSubmit = () => {
    console.log('登录')
    login(form.username, form.password).then((res) => {
        console.log(res)
    })
}

</script>
```

上述代码中：（知识回顾reactive、router、v-model）
- 定义了一个响应式的表单对象 `form`, 里面声明了用户名、密码字段，同时通过 `v-model` 将输入框与 `form` 绑定在了一起
- 为登录按钮添加了点击事件, 对应的方法为 `onSubmit()` , 方法中调用了刚刚声明的 `login()` 接口，入参为输入框中填写的用户名、密码。最终，打印了 `res` 对象，用于查看接口能够正常调通。

## 6 跨域问题

开启前端页面，进入登录页面，然后再开发者模式下，进行登录

`F12` 查看 `Console` 控制台日志，当我们点击登录按钮后，会看到如下异常：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a825bc16b6677fbb37514bfd194d2a00.png)

跨域是指在同一浏览器中运行的前端应用程序试图与不同域（==协议、域名、端口号中有任何一个不同==）的服务器进行通信。由于浏览器的同源策略，这种情况下会受到限制。

## 7 跨域解决方案

在开发环境中，你可以使用代理来绕过跨域问题。Vite 提供了一个配置项来设置代理。在项目根目录的 `vite.config.js` 文件中，添加以下内容：
```js
// 设置正向代理，使用 /api 代替后端完整的请求地址  
// 从而解决dev环境下的跨域问题  
server: {  
    proxy: {  
        // 对于前端的url地址中有 /api 的，就会被此代理规则处理  
        '/api': {  
            // 实际要转发的服务器（后端服务器）  
            target: 'http://localhost:8080',  
            // 在转发请求时，修改请求头中的 Origin 和 Host 字段，使其与后端服务器地址相匹配  
            changeOrigin: true,  
            // 正则：对于以 /api 为开始的url，将其替换  
            // 如：访问前端 localhost:3000/api/user 就会正向代理到后端 localhost:8080/user            rewrite: path => path.replace(/^\/api/,'')  
        }  
    }  
}
```

再修改axios.js文件中的baseUrl
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7aa9cd72c33ae7e1a1236ab7959297f9.png)

再次点击登录按钮，查看控制台日志，即可看到正常打印 `res` 对象信息啦：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20b3e56b20a3a3ba58884977c7678a09.png)

> 可以看到这里有两个data
> `第一个data`：返参Promise这个对象中的一个属性，表示后端服务器发送过来的数据，也就是我们之前定义全局返参对象Response
> `第二个data`：Response中的data属性，这个data保存的是我们定义的LoginRspVO对象

OK, 到这里说明调用接口的流程通了。接下来，我们要完成当登陆成功后，跳转后台首页功能。

## 8 后台首页

在 `/pages/admin/` 文件夹下，新建 `index.vue` , 表示登录后台的首页，也就是仪表盘页，内容如下，暂时就写个骨架页，等后续再完善：

```vue
<template>
    <div>
        后台首页
    </div>
</template>
```

### 8.1 注册路由

编辑 `/router/index.js` 路由文件，添加后台首页路由：

```js
import AdminIndex from '@/pages/admin/index.vue'

// 统一在这里声明所有路由
const routes = [
    // 省略...
    {
        path: "/admin/index", // 后台首页
        component: AdminIndex,
        meta: {
            title: 'Admin 后台首页'
        }
    }
]
```

然后，修改登录按钮对应的点击事件函数，当登录成功后，通过 `router` 跳转 `index.vue` 页:

```js
<script setup>
// 省略...
import { useRouter } from 'vue-router';

const router = useRouter()

// 登录
const onSubmit = () => {
    console.log('登录')
    login(form.username, form.password).then((res) => {
        console.log(res)
        // 判断是否成功
        if (res.data.success == true) {
            // 跳转到后台首页
            router.push('/admin/index')
        }
    })
}

</script>
```

当输入正确的用户名和密码后，再次点击登录按钮，效果如下：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/084dfa74c47fc9120acf6c4df8babcbd.png)

## 9 结语

至此，本小节目标：整合 Axios 并调用登录接口，当登录成功后，跳转后台首页的功能就完成了，但是还有很多细节没有完善，比如前端的参数校验、登录按钮加载 Loading 小图标、登录失败、成功的消息提示等，后面小节中，我们将一一整上。