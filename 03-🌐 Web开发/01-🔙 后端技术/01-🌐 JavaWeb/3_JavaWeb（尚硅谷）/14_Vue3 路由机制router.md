---
文章标题: "[[14_Vue3 路由机制router]]" 
文章作者: Dakkk
文章概要: |
  深入讲解Vue3路由机制，涵盖路由配置、编程式导航、参数传递、路由守卫等核心知识点，并通过登录认证案例展示实际应用。
tags:
- "Vue3"
- "Vue Router"
- "路由守卫"
- "编程式路由"
- "SPA"
- "前端路由"
- "组件导航"
- "路由参数"
相关文章:
- "[[13_全局路由拦截：实现页面标题动态设置、后台路由跳转的登录判断]]"
- "[[15_重复登录问题优化、密码框可显示密码]]"
- "[[3_整合 vue-router 路由管理器]]"
- "[[0_项目介绍]]"
- "[[1_搭建管理后台基本布局]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/3_JavaWeb（尚硅谷）/14_Vue3 路由机制router.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-03-28 10:36:00
修改时间: 2025-05-27 23:34:19
---

## 1 路由简介

> 1 什么是路由？

-   定义：`路由就是根据不同的 URL 地址展示不同的内容或页面`。
-   通俗理解：路由就像是一个地图，我们要去不同的地方，需要通过不同的路线进行导航。

> 2 路由的作用

-   `单页应用程序（SPA）中`，路由可以`实现不同视图之间的无刷新切换`，提升用户体验；
-   路由还可以`实现页面的认证和权限控制`，保护用户的隐私和安全；
-   路由还可以`利用浏览器的前进与后退，帮助用户更好地回到之前访问过的页面`
## 2 路由入门案例

### 2.1 总结

- 第一步：创建项目、安装项目依赖，安装全局的vue-router
- 第二步：创建其他vue组件
	- 其中App.vue组件中的template标签
		- `使用router-link标签`，来在页面显示一个链接其他vue组件的超链接![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/198fc38a09f9d0cd4067560e5d556ac3.png)
		- `使用router-view标签`，表示变化的组件（由路由决定）
- 第三步：准备路由配置，在src目录下创建router目录，然后创建router.js文件
	- 编辑router.js文件
		- 从vue-router中导入路由创建的相关方法：createRouter、createWebHashHistory![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7840a06b82ff01b5f701bfef0163c24d.png)
		- 导入第二步创建的组件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c434d29868b7aa5dd80c9d5cdbf2ef1a.png)
		- 创建路由对象![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53f488ae985769110f3001e95b84735f.png)
		- 对外暴露对象 `export default router`
- 第四步：main.js引入router的配置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b10b66a8244109b92cd95ee6de5f7525.png)

### 2.2 案例需求分析

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29431adef19d54e3ad64ea3dbb54ee16.png)

### 2.3 创建项目和导入路由依赖

```shell
npm create vite //创建项目cd 项目文件夹 //进入项目文件夹
npm install //安装项目需求依赖
npm install vue-router@4 --save //安装全局的vue-router 4版本
```

### 2.4 准备页面和组件    

+ components/Home.vue
``` html
<script setup>
</script>

<template>
    <div>
        <h1>Home页面</h1>
    </div>
</template>

<style scoped>
</style>
```

+ components/List.vue
``` html
<script setup>
</script>

<template>
    <div>
        <h1>List页面</h1>
    </div>
</template>

<style scoped>
</style>
```

+ components/Add.vue
``` html
<script setup>
</script>

<template>
    <div>
        <h1>Add页面</h1>
    </div>
</template>

<style scoped>
</style>
```

+ components/Update.vue
``` html
<script setup>
</script>

<template>
    <div>
        <h1>Update页面</h1>
    </div>
</template>

<style scoped>
</style>
```

+ App.vue
``` html
<script setup>
</script>

<template>
  <div>
    App开头的内容 <br>
    <hr/>

      <!-- 路由的连接 -->
      <!-- 可以写在其他标签上，改变的是根路径后的路径 -->
      <router-link to="/">home页</router-link> <br>
      <router-link to="/list">list页</router-link> <br>
      <router-link to="/add">add页</router-link> <br>
      <router-link to="/update">update页</router-link> <br>
    <hr/>

    <!-- 该标签会被替换成任意一个vue组件 -->
    <router-view></router-view>

    <hr>
    App结尾的内容
  </div>
</template>

<style scope>
</style>
```

### 2.5 准备路由配置
+ src/routers/router.js
```javascript
// 导入路由创建的相关方法
import {createRouter,createWebHashHistory} from 'vue-router'

// 导入vue组件
import Home from '../components/Home.vue'
import List from '../components/List.vue'
import Add from '../components/Add.vue'
import Update from '../components/Update.vue'

//创建路由对象，声明路由规则
const router = createRouter({

    // history属性用于记录路由切换的历史
    history:createWebHashHistory(),

    // 用于定义多个不同的路径和组件之间的对应关系
    routes:[

        //component指定组件在默认的路由视图位置展示 
        //定义 / 路径下，启用那个组件，否则/路径下没有显示
        {path:'/',component:Home},
        {path:'/home',component:Home},
        {path:'/list',component:List},
        {path:'/add',component:Add},
        {path:'/update',component:Update}
    ]
})

//对外暴露路由对象
export default router
```

### 2.6 main.js引入router配置
+ 修改文件：main.js (入口文件)
```javascript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
//导入router模块
import router from './routers/router.js'
let app = createApp(App)
//绑定路由对象（这个先执行！）
app.use(router)
//挂在试图
app.mount("#app")
```

### 2.7 启动测试
``` shell
npm run dev
```

### 2.8 补充：

- `一般在APP.vue组件中，我们只用一个<router-view>标签`

- 如果需要使用多个`<router-view>`标签，需要在标签内添加name属性
	- 如图所示![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c42d242f192e94815187a380d18cabc4.png)
	- 对应的route数组中的`component`也应该变为`components`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3f9b205f59275821dfd2740b454eef02.png)
## 3 路由重定向

> 重定向的作用：将一个路由重定向到另一个路由上

+ 修改案例：访问/list和/showAll都定向到List.vue
+ router.js
``` javascript
routes:[
	//重定向
	{path:'/showAll',redirect:'/list'}
]
```

+ App.vue
```html
<!-- 路由的连接 -->
<router-link to="/showAll">showAll页</router-link> <br>
```
## 4 编程式路由(useRouter)

> 普通路由

+ `<router-link to="/list">list页</router-link>  `这种路由，to中的内容目前是固定的，点击后只能切换/list对象组件(声明式路由)

> 编程式路由

+ 通过useRouter，动态决定向那个组件切换的路由
+ 在 Vue 3 和 Vue Router 4 中，你可以使用 `useRouter` 来实现动态路由(编程式路由)
+ 这里的 `useRouter` 方法返回的是一个 router 对象，你可以用它来做如导航到新页面、返回上一页面等操作。

> 案例需求: 通过普通按钮配合事件绑定实现路由页面跳转，不直接使用router-link标签

+ App.vue
``` html
<script setup>
  import {useRouter} from 'vue-router'
  import {ref} from 'vue'

  //创建动态路由对象
  let router = useRouter()

  //创建一个路由参数
  let routePath = ref('')

  //创建一个函数，用于修改路由参数，并传递
  let show = ()=>{

    //编程式路由
    //直接push一个路径
    router.push(routePath.value)
    //push一个带有path属性的对象
    // router.push({path:'/list'})
  }
</script>

<template>
    <div>
      <h1>App页面</h1>
      <hr/>
        <!-- 动态输入路径,点击按钮,触发单击事件的函数,在函数中通过编程式，完成路由切换页面 -->
        <button @click="show()">show</button> <br>
      <hr/>
      <!-- 路由连接对应视图的展示位置 -->
      <hr>
      默认展示位置:<router-view></router-view>
      <hr>
      Home视图展示:<router-view name="homeView"></router-view>
      <hr>
      List视图展示:<button @click="show()"></button>
    </div>
</template>

<style scoped>
</style>
```
## 5 路由传参(useRoute)

> 路径参数

+ 在路径中使用一个动态字段来实现，我们称之为 **路径参数**
  + 例如： 查看数据详情  `/showDetail/1`  ,`1`就是要查看详情的id,可以动态添值！

> 键值对参数

+ 类似与get请求通过url传参,数据是键值对形式的

+ 例如:  查看数据详情`/showDetail?hid=1`,`hid=1`就是要传递的键值对参数

+ 在 Vue 3 和 Vue Router 4 中，你可以使用  `useRoute` 这个函数从 Vue 的组合式 API 中获取路由对象。
+ `useRoute` 方法返回的是当前的 route 对象，你可以用它来`获取关于当前路由的信息，如当前的路径、查询参数等`。

> 案例需求 : 切换到ShowDetail.vue组件时,向该组件通过路由传递参数

### 5.1 路径参数

+ 修改App.vue文件
```html
<script setup>
    import { useRouter } from 'vue-router';
    let router = useRouter()

    let show1 = (id,language)=>{
        //拼接字符串的方式
        router.push(`/show1/${id}/${language}`)

    }
</script>

<template>
  <div>
    <router-link to="/show1/1/Java">show1声明式路径传参</router-link> <br>

    <button @click="show1(2,'Http')">show1编程式路径传参</button>
    <hr>
    showDetail视图展示:<router-view></router-view>
  </div>
</template>

<style scope>
</style>
```

+ 修改router.js文件
``` javascript
// 导入路由创建的相关方法// 导入路由创建的相关方法
import {createRouter,createWebHashHistory} from 'vue-router'
// 导入vue组件
import Show1 from '../components/Show1.vue'
import Show2 from '../components/Show2.vue'

const router = createRouter({
    history:createWebHashHistory(),
    routes:[
        {path:'/show1/:id/:language',component:Show1},
    ]
})

export default router
```

+ 添加Show1.vue文件
``` html
<script setup>
  import { useRoute } from 'vue-router';
  import { onUpdated,ref } from 'vue';

  // 使用useRoute函数，来创建一个函数对象，用于接受参数
  let route = useRoute()

  // 创建接收参数的变量
  let languageId = ref(0)
  let languageName = ref('')

  // 借助更新时生命周期,将数据实时更新进入响应式对象
  onUpdated(()=>{
    //获取参数
    languageId.value = route.params.id
    languageName.value = route.params.language
  })

</script>

<template>
  <div>
    <h1>Show1接受路径参数</h1><br>
    <h3>编号{{languageId}}:{{languageName}}是世界上最好的语言</h3><br>
    <h3>编号{{route.params.id}}:{{route.params.language}}是世界上最好的语言</h3><br>
  </div>
</template>

<style scoped>
</style>

```
### 5.2 键值对参数

+ 修改App.vue文件
```html
<script setup>
    import { useRouter } from 'vue-router';
    let router = useRouter()

    let show2 = (id,language)=>{
        //拼接字符串的方式
        // router.push(`/show2?id=${id}&language=${language}`)
        // 使用push 对象
        router.push({path:'/show2',query:{id:id,language:language}})
    }
</script>

<template>
  <div>
    <router-link to="/show2?id=1&language=PHP">show2声明式键值对传参</router-link> <br>
    <router-link :to="{path:'/show2',query:{id:2,language:'PHP'}}">show2声明式键值对传参</router-link> <br>
    <button @click="show2(3,'Go')">show2编程式键值对传参</button>
    <hr>
    showDetail视图展示:<router-view></router-view>
  </div>
</template>

<style scope>
</style>
```

+ 修改router.js文件
```js
// 导入路由创建的相关方法
import {createRouter,createWebHashHistory} from 'vue-router'

// 导入vue组件
import Show2 from '../components/Show2.vue'

const router = createRouter({
    history:createWebHashHistory(),
    routes:[
        {path:'/show2',component:Show2},
    ]
})

export default router
```

+ 添加Show2.vue文件
```html
<script setup>
  import { useRoute } from 'vue-router';
  import { onUpdated,ref } from 'vue';

  // 使用useRoute函数，来创建一个函数对象，用于接受参数
  let route = useRoute()

  // 创建接收参数的变量
  let languageId = ref(0)
  let languageName = ref('')

  // 借助更新时生命周期,将数据实时更新进入响应式对象
  onUpdated(()=>{
    //获取参数
    languageId.value = route.query.id
    languageName.value = route.query.language
  })
</script>

<template>
  <div>
    <h1>Show2接受键值对参数</h1><br>
    <h3>编号{{languageId}}:{{languageName}}是世界上最好的语言</h3><br>
    <h3>编号{{route.query.id}}:{{route.query.language}}是世界上最好的语言</h3><br>
  </div>
</template>

<style scope>
</style>

```
## 6 路由守卫(开发用的多！)

> 在 Vue 3 中，`路由守卫是用于在路由切换期间进行一些特定任务的回调函数`。路由守卫可以用于许多任务，例如`验证用户是否已登录`、在`路由切换前提供确认提示、请求数据`等。Vue 3 为路由守卫提供了全面的支持，并提供了以下几种类型的路由守卫：

1. **全局前置守卫**：在路由切换前被调用，可以用于验证用户是否已登录、中断导航、请求数据等。
    
2. **全局后置守卫**：在路由切换之后被调用，可以用于处理数据、操作 DOM 、记录日志等。
    
3. **守卫代码的位置**: 在router.js中，或者单独提取出来export，然后import

```js
//全局前置路由守卫
router.beforeEach( (to,from,next) => {
    //to 是目标地包装对象  .path属性可以获取地址
    //from 是来源地包装对象 .path属性可以获取地址
    console.log(from.path,to.path)

    //需要判断，注意避免无限重定向
    if(to.path == '/index'){
	    //next是方法，不调用默认拦截！ next() 放行,直接到达目标组件
        next()
    }else{
	    //next('/地址')可以转发到其他地址,到达目标组件前会再次经过前置路由守卫
        next('/index')
    }
    
} )

//全局后置路由守卫
router.afterEach((to, from) => {
    console.log(`Navigate from ${from.path} to ${to.path}`);
});
```
### 6.1 登录案例

`登录以后才可以进入home,否则必须进入login`

+ 定义Login.vue
	+ 使用ref保存数据，v-model双向绑定
	+ 使用router编程式路由，并键值对传参
	+ 使用前端存储机制
```html
<script setup>
    import {ref} from 'vue'
    import {useRouter} from 'vue-router'
    let username =ref('')
    let password =ref('')
    let router = useRouter();
    let login = () =>{
        console.log(username.value,password.value)
        if(username.value == 'root' & password.value == '123456'){
            router.push({path:'/home',query:{'username':username.value}})
            //登录成功利用前端存储机制，存储账号！
            localStorage.setItem('username',username.value)
            //sessionStorage.setItem('username',username)
        }else{
            alert('登录失败，账号或者密码错误！');
        }
    }

</script>

<template>
    
    <div>
        账号： <input type="text" v-model="username" placeholder="请输入账号！"><br>
        密码： <input type="password" v-model="password" placeholder="请输入密码！"><br>
        <button @click="login()">登录</button>
    </div>

</template>

<style scoped>
</style>

```

+ 定义Home.vue
	+ 使用ref保存数据，{{}}单向获取数据
	+ 使用router编程式路由
	+ 使用前端存储机制
```html
<script setup>
 import {ref} from 'vue'
 import {useRouter} from 'vue-router'

 let router = useRouter()
 //  并不是每次进入home页时,都有用户名参数传入
 //let username = route.query.username
 let username =window.localStorage.getItem('username'); 

 let logout= ()=>{
    // 清除localStorge中的username
    //window.sessionStorage.removeItem('username')
    window.localStorage.removeItem('username')
    // 动态路由到登录页
    router.push("/login")

 }

</script>

<template>
    <div>
        <h1>Home页面</h1>
        <h3>欢迎{{username}}登录</h3>
        <button @click="logout">退出登录</button>
    </div>
</template>

<style scoped>

</style>

```

+ App.vue
```java
<script setup type="module">
  
</script>

<template>
    <div>
      
      <router-view></router-view>
      
    </div>
</template>

<style scoped>
</style>
```

+ 定义routers.js
	+ 使用路由全局前置守卫
``` javascript
// 导入路由创建的相关方法
import {createRouter,createWebHashHistory} from 'vue-router'

// 导入vue组件

import Home from '../components/Home.vue'
import Login from '../components/login.vue'
// 创建路由对象,声明路由规则
const router = createRouter({
    history: createWebHashHistory(),
    routes:[
        {
            path:'/home',
            component:Home
        },
        {
            path:'/',
            redirect:"/home"
        },
        {
            path:'/login',
            component:Login
        },
    ]

})

// 设置路由的全局前置守卫
router.beforeEach((to,from,next)=>{
    /* 
    to 要去那
    from 从哪里来
    next 放行路由时需要调用的方法,不调用则不放行
    */
    console.log(`从哪里来:${from.path},到哪里去:${to.path}`)

    if(to.path == '/login'){
        //放行路由  注意放行不要形成循环  
        next()
    }else{
        //let username =window.sessionStorage.getItem('username'); 
        let username =window.localStorage.getItem('username'); 
        if(null != username){
            next()
        }else{
            next('/login')
        }

    }
})
// 设置路由的全局后置守卫
router.afterEach((to,from)=>{
    console.log(`从哪里来:${from.path},到哪里去:${to.path}`)
})

// 对外暴露路由对象
export default router;
```

- main.js引入router配置
```javascript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
//导入router模块
import router from './routers/router.js'
let app = createApp(App)
//绑定路由对象（这个先执行！）
app.use(router)
//挂在试图
app.mount("#app")
```

+ 启动测试
```shell
npm run dev
```

