前面小节中，我们已经将首页所需的几个后台接口都开发完毕了，这小节中将把前台 `Header` 头封装成一个独立的公共组件，这样在后续页面开发中，直接引入即可。另外，关于 `Header` 头中的博客信息，也不再是前台写死的，而是是通过后台接口来获取。最终实现效果如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e2086e0837767371c77f7d580417dad6.png)

## 1 登录头像样式布局

通常来说，当我们登录系统后，前台的 `Header` 头中应该能够展示用户的登录头像，而不是没什么变化，只显示一个登录按钮。同时，当点击头像时，会弹出一个下拉菜单，可以让用户点击直接 _进入后台_，或者 _退出登录_。`Flowbit` 组件库提供了现成的下拉菜单以供使用，官方地址：[https://flowbite.com/docs/components/dropdowns/](https://flowbite.com/docs/components/dropdowns/) ， 可以通过点击 _Copy_ 按钮，直接将组件的示例源码复制出来，添加到 `/pages/frontend/index.vue` 首页中：

![](https://img.quanxiaoha.com/quanxiaoha/169789274623527)

示例代码有个大致的架子，但是还需要稍微定制一下，在按钮中添加一个用户头像的图片，同时，下拉菜单也要改一改，代码如下：

```
                    <!-- 已经登录，展示用户头像 -->
                    <button id="dropdownDefaultButton" data-dropdown-toggle="dropdown"
                        class="text-white ml-2 focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
                        type="button">
                        <!-- 用户登录头像 -->
                        <img class="w-8 h-8 rounded-full" src="https://img.quanxiaoha.com/quanxiaoha/f97361c0429d4bb1bc276ab835843065.jpg" alt="user photo">
                    </button>

                    <!-- Dropdown menu -->
                    <div id="dropdown"
                        class="z-10 hidden bg-white divide-y divide-gray-100 rounded-lg shadow dark:bg-gray-700">
                        <ul class="py-2 text-sm text-gray-700 dark:text-gray-200" aria-labelledby="dropdownDefaultButton">
                            <li>
                                <a href="#"
                                    class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">进入后台</a>
                            </li>
                            <li>
                                <a href="#"
                                    class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">退出登录</a>
                            </li>
                        </ul>
                    </div>
                    
<script setup>
import { onMounted, ref } from 'vue'
import { initCollapses, initDropdowns } from 'flowbite'


// 初始化 flowbit 相关组件
onMounted(() => {
    initCollapses();
    initDropdowns();
})
</script>
```

上述代码中，除了要将组件的示例代码进行个性化定制一下外，还需要引入 `initDropdowns` 方法，在 `onMounted` 周期方法中执行该初始化下拉菜单方法，如果你不这样做，会导致点击头像不会弹出下拉菜单。最终效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169788971403837)

## 2 添加下拉菜单图标

为了让下拉菜单更加好看点，可以在它的左侧添加一个 `icon` 图标，`Flowbit` 组件库官方就提供了一些 `svg` 格式的 `icon` 图标以供使用，官方地址：[https://flowbite.com/icons/](https://flowbite.com/icons/) ，挑选自己中意的，直接复制即可，这里小哈挑选了两个，加入到 `<a>` 标签中，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169789008957553)

```
                        <ul class="py-2 text-sm text-gray-700 dark:text-gray-200" aria-labelledby="dropdownDefaultButton">
                            <li>
                                <a href="#"
                                    class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">
                                    <svg class="inline w-3 h-3 mb-[2px] mr-1 text-gray-700 dark:text-white" aria-hidden="true"
                                        xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">
                                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                            stroke-width="2"
                                            d="M10 14v4m-4 1h8M1 10h18M2 1h16a1 1 0 0 1 1 1v11a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1Z" />
                                    </svg>
                                    进入后台
                                </a>
                            </li>
                            <li>
                                <a href="#"
                                    class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">
                                    <svg class="inline w-3 h-3 mb-[2px] mr-1 text-gray-700 dark:text-white" aria-hidden="true"
                                        xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 16 16">
                                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                            stroke-width="2"
                                            d="M4 8h11m0 0-4-4m4 4-4 4m-5 3H3a2 2 0 0 1-2-2V3a2 2 0 0 1 2-2h3" />
                                    </svg>
                                    退出登录
                                </a>
                            </li>
                        </ul>
```

## 3 动态判断用户是否登录

从上面的效果图可以看到，这个时候 _登录按钮_ 和 _用户头像_ 是同时存在的，我们需要的效果是，当用户已登录就显示头像，未登录则显示登录按钮。为了达到这个效果，我们可以对之前封装的 `pinia` 用户 `store` 中的 `userInfo` 对象进行判断，因为只有用户登录后，才会设置它的值，若对象存在字段值，则表明用户已登录。编辑 `/index.vue` 首页，添加代码如下：

```
<script setup>
// 省略...
import { useUserStore } from '@/stores/user'

// 是否登录，通过 userStore 中的 userInfo 对象是否有数据来判断
const userStore = useUserStore()
// 获取 userInfo 对象所有属性名称的数组
const keys = Object.keys(userStore.userInfo)
// 若大于零，则表示用户已登录
const isLogined = ref(keys.length > 0)
</script>
```

上述代码中，我们首先引入 `useUserStore` , 通过 `Object.keys()` 拿到 `userInfo` 用户信息变量中的所有属性字段数组，若数组长度大于 0, 则表示用户已登录，否则未登录。

### 3.1 重构 token 过期跳转登录页逻辑

因为我们是对 `userInfo` 对象进行判断，从而得知用户是否登录。 然而，之前我们在 `axios.js` 文件中，当服务端返回 `401` 状态事件时，仅仅删除了 `cookie` 中的令牌，并没有将全局状态 `store` 中的 `userInfo` 变量置为空，所以还需完善一下，修改代码如下：

```
import { useUserStore } from '@/stores/user'

// 添加响应拦截器
instance.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response.data
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    let status = error.response.status

    // 状态码 401
    if (status == 401) {
        // 退出登录
        let userStore = useUserStore()
        userStore.logout()
        // 刷新页面
        location.reload()
    }

    // 省略...
})
```

上述代码中，当 `status` 为 `401` 时，则直接调用 `userStore` 中封装好的 `logout` 退出登录方法，该方法不但删除了 `Token` 令牌，还将用户信息置空了：

![](https://img.quanxiaoha.com/quanxiaoha/169789034188371)

### 3.2 添加动态判断

回到 `index.vue` 首页中，为登录按钮添加 `v-if="!isLogined"` 指令，表示只有用户非登录状态时才显示，另外，为头像按钮添加 `v-else` 指令，表示登录后才显示。

![](https://img.quanxiaoha.com/quanxiaoha/169789054173317)

可以看到，当我登录了后台系统后，首页现在只展示用户头像，不会显示登录按钮了：

![](https://img.quanxiaoha.com/quanxiaoha/169789081245597)

## 4 渲染博客设置信息

目前来说，`Header` 头中博客标题、`Logo` , 以及登录头像都还是写死的，我们需要调用后台接口先获取这些数据，然后再动态渲染到页面中。

### 4.1 封装博客设置 pinia 全局状态

博客设置信息除了 `Header` 头中会使用到，左侧栏的博主信息也会用到。我们可以将这些信息单独存储到一个博客设置 `store` 中。在 `/store` 文件夹下，创建 `blogsettings.js` 文件，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169789066104752)

```js
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { getBlogSettingsDetail } from '@/api/frontend/blogsettings'

export const useBlogSettingsStore = defineStore('blogsettings', () => {
  // 博客设置信息
  const blogSettings = ref({})

  // 获取博客设置信息
  function getBlogSettings() {
    // 调用后台获取博客设置信息接口
    console.log('获取博客设置信息')
    getBlogSettingsDetail().then(res => {
      if (res.success) {
        blogSettings.value = res.data
      }
    })
  }

  return { blogSettings, getBlogSettings }
})
```

上述代码中，创建了一个名为 `blogsettings` 的全局 `store` , 在其中添加了一个 `blogSettings` 变量，用于存储博客设置信息。另外，还对外提供了一个 `getBlogSettings()` 方法，用于调用后台接口并保存数据。

还需要再api/front文件夹中，添加一个blogsettings.js文件，使用前台获取文章信息的接口
```js
import axios from "@/axios";  
  
// 获取博客设置详情  
export function getBlogSettingsDetail() {  
    return axios.post("/blog/settings/detail")  
}
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/b7d11e46074dec8d6f5b2cc53af08ab2.png)

### 4.2 全局前置守卫

那么，什么时候调用 `useBlogSettingsStore` 中的 `getBlogSettings()` 方法获取博客设置信息呢？我们可以在全局前置守卫中，当想要跳转的页面路由不是以 `/admin` 开头，也就是想跳转的是前台页面的时候，来调用该方法，代码如下：

```
import { useBlogSettingsStore } from '@/stores/blogsettings'

// 全局路由前置守卫
router.beforeEach((to, from, next) => {
    console.log('==> 全局路由前置守卫')

    // 展示页面加载 Loading
    showPageLoading()
    
    let token = getToken()

    if (!token && to.path.startsWith('/admin')) { 
        // 省略...
    } else if (token && to.path == '/login') {
       // 省略...
    } else if (!to.path.startsWith('/admin')) {
        // 如果访问的非 /admin 前缀路由
        // 引入博客设置 store
        let blogSettingsStore = useBlogSettingsStore()
        // 获取博客设置信息并保存到全局状态中
        blogSettingsStore.getBlogSettings()
        next()
    } else {
        next()
    }
})
```

回到浏览器中，刷新页面，可以在 `Vue` 插件中看到存储成功的博客设置信息，如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/fa6e8e677d9794ee8a09291d67c2483f.png)

### 4.3 渲染数据

存储到全局状态中后，我们就可以直接去取了，将 `Logo` 地址、博客名称，以及头像地址都改成从 `store` 中获取，代码如下：

```
				// 省略..
				<!-- 博客 LOGO 、博客名称 -->
                <a href="/" class="flex items-center">
                    <img :src="blogSettingsStore.blogSettings.logo" class="h-8 mr-3 rounded-full" alt="Flowbite Logo" />
                    <span class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white">{{
                        blogSettingsStore.blogSettings.name }}</span>
                </a>
                // 省略..
                
                    <!-- 已经登录，展示用户头像 -->
                    <button id="dropdownDefaultButton" data-dropdown-toggle="dropdown" v-else
                        class="text-white ml-2 focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
                        type="button">
                        <!-- 用户登录头像 -->
                        <img class="w-8 h-8 rounded-full" :src="blogSettingsStore.blogSettings.avatar" alt="user photo">
                    </button>
                    // 省略...

<script setup>
// 省略...
import { useBlogSettingsStore } from '@/stores/blogsettings'

// 引入博客设置信息 store
const blogSettingsStore = useBlogSettingsStore()
</script>
```

## 5 封装 Header 组件

`Header` 头由于前台每个页面都会使用到，如首页、文章详情页等，所以完全可以封装成一个组件。在 `/layout/` 文件夹下，创建 `/frontend/components` 文件夹，用于统一放置前台相关的组件，并创建 `Header.vue` 组件，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169789155560445)

```html
<script setup>  
import {onMounted,ref} from "vue";  
import {initCollapses, initDropdowns} from "flowbite";  
import {useUserStore} from "@/stores/user.js";  
import {useBlogSettingsStore} from "@/stores/blogsettings.js";  
  
  
// 初始化 flowbite 相关组件  
onMounted(() => {  
    initCollapses();  
    initDropdowns();  
})  
  
// 通过 pinia 中的 user仓库中的 userInfo是否有值，来判断用户是否登录  
const useStore = useUserStore()  
// 获取 userInfo 对象所有属性名称的数组  
// 接收一个对象作为参数，并返回一个包含对象所有可枚举属性键的数组  
const keys = Object.keys(useStore.userInfo)  
// 若大于0，表示用户已登录  
const isLogin = ref(keys.length>0)  
  
// 引入pinia保存的博客设置信息  
const blogSettingsStore = useBlogSettingsStore()  
</script>  
  
<template>  
    <!--顶部Navbar，使用Flowbite组件-->  
    <nav class="bg-white border-gray-200 border-b dark:bg-gray-900">  
        <div class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4">  
            <!-- 博客 LOGO 、博客名称 -->  
            <a href="/" class="flex items-center">  
                <img :src="blogSettingsStore.blogSettings.logo" class="h-8 mr-3" alt="Flowbite Logo"/>  
                <span class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white">  
                    {{ blogSettingsStore.blogSettings.name }}</span>  
            </a>  
            <div class="flex md:order-2 items-center">  
                <button type="button" data-collapse-toggle="navbar-search" aria-controls="navbar-search"  
                        aria-expanded="false"  
                        class="md:hidden text-gray-500 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-sm p-2.5 mr-1">  
                    <svg class="w-5 h-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"  
                         viewBox="0 0 20 20">  
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                              d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z"/>  
                    </svg>  
                    <span class="sr-only">Search</span>  
                </button>  
                <div class="relative hidden md:block">  
                    <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">  
                        <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true"  
                             xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                                  d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z"/>  
                        </svg>  
                        <span class="sr-only">Search icon</span>  
                    </div>  
                    <input type="text" id="search-navbar"  
                           class="block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"  
                           placeholder="📝找点啥呢...">  
                </div>  
                <!-- 登录 -->  
                <div class="text-gray-900 ml-3 mr-1 hover:text-blue-700" @click="$router.push('/login')" v-if="!isLogin">登录</div>  
                <button data-collapse-toggle="navbar-search" type="button" v-else  
                        class="inline-flex items-center p-2 w-10 h-10 justify-center text-sm text-gray-500 rounded-lg md:hidden hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-gray-200 dark:text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-600"  
                        aria-controls="navbar-search" aria-expanded="false">  
                    <span class="sr-only">Open main menu</span>  
                    <svg class="w-5 h-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"  
                         viewBox="0 0 17 14">  
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                              d="M1 1h15M1 7h15M1 13h15"/>  
                    </svg>  
                </button>  
                <!-- 已经登录，展示用户头像 -->  
                <button id="dropdownDefaultButton" data-dropdown-toggle="dropdown"  
                        class="text-white ml-2 focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"  
                        type="button">  
                    <!-- 用户登录头像 -->  
                    <img class="w-8 h-8 rounded-full" :src="blogSettingsStore.blogSettings.avatar" alt="user photo">  
                </button>  
                <!-- 点击头像显示下拉菜单 -->  
                <div id="dropdown"  
                     class="z-10 hidden bg-white divide-y divide-gray-100 rounded-lg shadow dark:bg-gray-700">  
                    <ul class="py-2 text-sm text-gray-700 dark:text-gray-200" aria-labelledby="dropdownDefaultButton">  
                        <li>  
                            <a href="#"  
                               class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">  
                                <!--svg引入图标-->  
                                <svg class="inline w-3 h-3 mb-[2px] mr-1 text-gray-700 dark:text-white" aria-hidden="true"  
                                     xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"  
                                          stroke-width="2"  
                                          d="M10 14v4m-4 1h8M1 10h18M2 1h16a1 1 0 0 1 1 1v11a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1Z" />  
                                </svg>  
                                进入后台  
                            </a>  
                        </li>  
                        <li>                            <a href="#"  
                               class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">  
                                <!--svg引入图标-->  
                                <svg class="inline w-3 h-3 mb-[2px] mr-1 text-gray-700 dark:text-white" aria-hidden="true"  
                                     xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 16 16">  
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"  
                                          stroke-width="2"  
                                          d="M4 8h11m0 0-4-4m4 4-4 4m-5 3H3a2 2 0 0 1-2-2V3a2 2 0 0 1 2-2h3" />  
                                </svg>  
                                退出登录  
                            </a>  
                        </li>  
                    </ul>  
                </div>  
            </div>  
            <div class="items-center justify-between hidden w-full md:flex md:w-auto md:order-1" id="navbar-search">  
                <div class="relative mt-3 md:hidden">  
                    <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">  
                        <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true"  
                             xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                                  d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z"/>  
                        </svg>  
                    </div>  
                    <input type="text" id="search-navbar"  
                           class="block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"  
                           placeholder="📝找点啥呢...">  
                </div>  
                <ul                    class="flex flex-col p-4 md:p-0 mt-4 font-medium border border-gray-100 rounded-lg bg-gray-50 md:flex-row md:space-x-8 md:mt-0 md:border-0 md:bg-white dark:bg-gray-800 md:dark:bg-gray-900 dark:border-gray-700">  
                    <li>  
                        <a href="#"  
                           class="block py-2 pl-3 pr-4 text-white bg-blue-700 rounded md:bg-transparent md:text-blue-700 md:p-0 md:dark:text-blue-500"  
                           aria-current="page">首页</a>  
                    </li>  
                    <li>                        <a href="#"  
                           class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 md:dark:hover:text-blue-500 dark:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">分类</a>  
                    </li>  
                    <li>                        <a href="#"  
                           class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 dark:text-white md:dark:hover:text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">标签</a>  
                    </li>  
                    <li>                        <a href="#"  
                           class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 dark:text-white md:dark:hover:text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">归档</a>  
                    </li>  
                </ul>  
            </div>  
        </div>  
    </nav>  
  
</template>  
  
<style scoped>  
  
</style>

```

然后，重构 `index.vue` 首页，直接引入封装好的 `Header.vue` 即可。

```
<template>
    <Header></Header>
</template>

<script setup>
import Header from '@/layouts/frontend/components/Header.vue'
</script>
```