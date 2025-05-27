---
文章标题: "[[16_Vue3 状态管理pinia]]" 
文章作者: Dakkk
文章概要: |
  介绍Vue3状态管理工具Pinia的基本概念和使用方法，包括安装配置、定义store、在组件中操作和展示数据等核心功能。
tags:
- "Vue3"
- "Pinia"
- "状态管理"
- "store"
- "state"
- "getters"
- "actions"
- "前端开发"
相关文章:
- "[[4_整合全局状态管理库 pinia]]"
- "[[17_Pinia 存储用户信息、动态显示登录用户名]]"
- "[[18_Pinia-persist 实现Pinia数据持久化]]"
- "[[5_前台 Header 头组件封装]]"
- "[[5_AdminMenu：点击收缩、展开功能实现]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/3_JavaWeb（尚硅谷）/16_Vue3 状态管理pinia.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-03-28 10:36:00
修改时间: 2025-05-27 23:34:19
---

## 1 介绍

> 如何实现多个组件之间的数据传递?

+ 方式1 组件传参   
	+ 语法复杂，且组件间需要存在父子关系

+ 方式2 路由传参 
	+ 必须使用路由

+ 方式3 通过pinia状态管理定义共享数据

> 当我们有`多个组件共享一个共同的状态(数据源)`时，多个视图可能都依赖于同一份状态。
> 
> 来自不同视图的交互也可能需要更改同一份状态。虽然我们的手动状态管理解决方案（props,组件间通信,模块化）在简单的场景中已经足够了，但是在大规模的生产应用中还有很多其他事项需要考虑：

-   更强的团队协作约定
-   与 Vue DevTools 集成，包括时间轴、组件内部审查和时间旅行调试
-   模块热更新 (HMR)
-   服务端渲染支持

>  [Pinia](https://pinia.vuejs.org/zh/ "Pinia") 就是一个实现了上述需求的状态管理库，由 Vue 核心团队维护，对 Vue 2 和 Vue 3 都可用。https://pinia.vuejs.org/zh/introduction.html
## 2 基本用法

### 2.1 准备vite项目

```javascript
npm create vite
npm install 
npm install vue-router@4 --save
```

### 2.2 安装pinia

```javascript
npm install pinia
```

### 2.3 定义pinia store对象 

src/store/store.js \[推荐这么命名不是强制]


- 定义数据并且对外暴露
- store就是定义共享状态的包装对象
- 内部包含四个属性：
	- id 唯一标识 
	- state 完整类型推理，推荐使用箭头函数 存放的数据 
	- getters 类似属性计算，存储放对数据
	- 操作的方法  actions 存储数据的复杂业务逻辑方法

- 理解： store类似Java中的实体类， id就是类名， state 就是装数据值的属性  getters就是get方法，actions就是对数据操作的其他方法

```javascript
import {defineStore } from 'pinia'

export const definedPerson = defineStore(
    {
        id: 'personPinia', //必须唯一
        state:()=>{ // state中用于定义数据
            return {
                username:'张三',
                age:0,
                hobbies:['唱歌','跳舞']
            }
        },
        getters:{// 用于定义一些通过数据计算而得到结果的一些方法 一般在此处不做对数据的修改操作
                 // getters中的方法可以当做属性值方式使用
            getHobbiesCount(){
                return this.hobbies.length
            },
            getAge(){
                return this.age
            }
        },
        actions:{ // 用于定义一些对数据修改的方法
            doubleAge(){
                this.age=this.age*2
            }
        }
    }
)
```

### 2.4 在main.js配置pinia组件到vue中 

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './routers/router.js'
// 导pinia
import { createPinia } from 'pinia'
// 创建pinia对象
let pinia= createPinia()

let app =createApp(App)
app.use(router)
// app中使用pinia功能
app.use(pinia) 
app.mount('#app')
```

### 2.5 Operate.vue 中操作Pinia数据

```html
<script setup type="module">
    import { ref} from 'vue';
    import { definedPerson} from '../store/store';
    // 读取存储的数据
    let person= definedPerson()

    let hobby = ref('')
   
</script>

<template>
    <div>
        <h1>operate视图,用户操作Pinia中的数据</h1>
        请输入姓名:<input type="text" v-model="person.username"> <br>
        请输入年龄:<input type="text" v-model="person.age"> <br>
        请增加爱好:
        <input type="checkbox" value="吃饭"  v-model="person.hobbies"> 吃饭
        <input type="checkbox" value="睡觉"  v-model="person.hobbies"> 睡觉
        <input type="checkbox" value="打豆豆"  v-model="person.hobbies"> 打豆豆 <br>
        
        <!-- 事件中调用person的doubleAge()方法 -->
        <button @click="person.doubleAge()">年龄加倍</button> <br>
        <!-- 事件中调用pinia提供的$reset()方法恢复数据的默认值 -->
        <button @click="person.$reset()">恢复默认值</button> <br>
        <!-- 事件中调用$patch方法一次性修改多个属性值 -->
        <button @click="person.$patch({username:'奥特曼',age:100,hobbies:['晒太阳','打怪兽']})">变身奥特曼</button> <br>
		显示pinia中的person数据:{{person}}
    </div>
</template>
<style scoped>
</style>
```

### 2.6 st.vue中展示Pinia数据

``` html
<script setup type="module">
    import { definedPerson} from '../store/store';
    // 读取存储的数据
    let person= definedPerson()
</script>

<template>
    <div>
        <h1>List页面,展示Pinia中的数据</h1>
        读取姓名:{{person.username}} <br>
        读取年龄:{{person.age}} <br>
        通过get年龄:{{person.getAge}} <br>
        爱好数量:{{person.getHobbiesCount}} <br>
        所有的爱好:
        <ul>
            <li v-for='(hobby,index) in person.hobbies' :key="index" v-text="hobby"></li>
        </ul>
    </div>
</template>

<style scoped>
</style>

```

### 2.7 定义组件路由router.js

``` javascript
// 导入路由创建的相关方法
import {createRouter,createWebHashHistory} from 'vue-router'

// 导入vue组件
import List  from '../components/List.vue'
import Operate  from '../components/Operate.vue'
// 创建路由对象,声明路由规则
const router = createRouter({
    history: createWebHashHistory(),
    routes:[
        {
            path:'/opearte',
            component:Operate
        },
        
        {
            path:'/list',
            component:List
        },
    ]

})

// 对外暴露路由对象
export default router;
```

### 2.8 App.vue中通过路由切换组件

``` html
<script setup type="module">

  
</script>

<template>
    <div>
      <hr>
      <router-link to="/opearte">显示操作页</router-link> <br>
      <router-link to="/list">显示展示页</router-link> <br>
      <hr>
      <router-view></router-view>
    </div>
</template>

<style scoped>
</style>
```

- 启动测试

```javascript
npm run dev
```

## 3 其他细节

>  State (状态) 在大多数情况下，state 都是你的 store 的核心。人们通常会先定义能代表他们 APP 的 state。在 Pinia 中，state 被定义为一个返回初始状态的函数。

+ store.js

```javascript
import {defineStore} from 'pinia'

//将id属性值移出来
export const definedPerson = defineStore('personPinia',
    {
        state:()=>{
            return {
                username:'',
                age:0,
                hobbies:['唱歌','跳舞']
            }
        },
        getters:{
            getHobbiesCount(){
                return this.hobbies.length
            },
            getAge(){
                return this.age
            }
        },
        actions:{
            doubleAge(){
                this.age=this.age*2
            }
        }
    }
)
```

+ Operate.vue

``` html
<script setup type="module">
    import { ref} from 'vue';
    import { definedPerson} from '../store/store';
    // 读取存储的数据
    let person= definedPerson()

    let hobby = ref('')
    let addHobby= ()=> {
        console.log(hobby.value)
        person.hobbies.push(hobby.value)
    }
    // 监听状态
    person.$subscribe((mutation,state)=>{
        console.log('---subscribe---')
        /* 
        mutation.storeId
            person.$id一样
        mutation.payload
            传递给 cartStore.$patch() 的补丁对象。
        state 数据状态,其实是一个代理
        */
        console.log(mutation)
        console.log(mutation.type)
        console.log(mutation.payload)
        console.log(mutation.storeId)
        console.log(person.$id)
        // 数据 其实是一个代理对象
        console.log(state)
    })


</script>

<template>
    <div>
        <h1>operate视图,用户操作Pinia中的数据</h1>
        请输入姓名:<input type="text" v-model="person.username"> <br>
        请输入年龄:<input type="text" v-model="person.age"> <br>
        请增加爱好:
        <input type="checkbox" value="吃饭"  v-model="person.hobbies"> 吃饭
        <input type="checkbox" value="睡觉"  v-model="person.hobbies"> 睡觉
        <input type="checkbox" value="打豆豆"  v-model="person.hobbies"> 打豆豆 <br>
        <input type="text" @change="addHobby" v-model="hobby"> <br>  
        <!-- 事件中调用person的doubleAge()方法 -->
        <button @click="person.doubleAge()">年龄加倍</button> <br>
        <!-- 事件中调用pinia提供的$reset()方法恢复数据的默认值 -->
        <button @click="person.$reset()">恢复默认值</button> <br>
        <!-- 事件中调用$patch方法一次性修改多个属性值 -->
        <button @click="person.$patch({username:'奥特曼',age:100,hobbies:['晒太阳','打怪兽']})">变身奥特曼</button> <br>
		person:{{person}}
    </div>
</template>
<style scoped>
</style>

```

>  2 Getter 完全等同于 store 的 state 的[计算值](https://cn.vuejs.org/guide/essentials/computed.html "计算值")。可以通过 `defineStore()` 中的 `getters` 属性来定义它们。**推荐**使用箭头函数，并且它将接收 `state` 作为第一个参数：

```javascript
export const useStore = defineStore('main', {
  state: () => ({
    count: 0,
  }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
})
```

>  3  Action 相当于组件中的 [method](https://v3.vuejs.org/guide/data-methods.html#methods "method")。它们可以通过 `defineStore()` 中的 `actions` 属性来定义，**并且它们也是定义业务逻辑的完美选择。** 类似 [getter](https://pinia.vuejs.org/zh/core-concepts/getters.html "getter")，action 也可通过 `this` 访问**整个 store 实例**，并支持**完整的类型标注(以及自动补全)**。不同的是，`action` 可以是异步的，你可以在它们里面 `await` 调用任何 API，以及其他 action！

```javascript
export const useCounterStore = defineStore('main', {
  state: () => ({
    count: 0,
  }),
  actions: {
    increment() {
      this.count++
    },
    randomizeCounter() {
      this.count = Math.round(100 * Math.random())
    },
  },
})
```
