---
文章标题: "[[6_整合 Tailwind CSS组件库：Flowbite]]" 
文章作者: Dakkk
文章概要: |
  本文详细指导如何在Web项目整合基于Tailwind CSS的Flowbite组件库。涵盖npm安装、`tailwind.config.js`配置，并以Vue组件中响应式导航栏为例，演示使用及JS初始化，助力开发者快速构建现代UI。
文章标签:
- "Tailwind CSS"
- "Flowbite"
- "组件库"
- "Vue.js"
- "前端开发"
- "UI框架"
- "npm"
- "Web开发"
相关文章:
- "[[1_Vue 3环境安装 和 blog前端项目搭建]]"
- "[[11_Nodejs + npm]]"
- "[[15_IDEA中集成VUE]]"
- "[[16_Vue]]"
- "[[17_Ajax]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/2_前端工程搭建/6_整合 Tailwind CSS组件库：Flowbite.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-18 14:22:10
修改时间: 2025-05-27 00:54:20
---


市面上有很多基于 Tailwind CSS 开源的组件库，综合选择下来，觉得 Flowbite 的 UI 更好看些，组件也较为丰富，所以在 `blog` 的前台展示中决定再整个框架。

## 1 什么是 Flowbite？

Flowbite 是一个基于 Tailwind CSS 创建的组件库，旨在帮助开发者快速构建现代、响应式的 Web 应用界面。在本小节中，将的带着大家在 `blog` 项目中集成和使用 Flowbite。

## 2 开始整合

Flowbite 是基于 Tailwind CSS 构建，因此我们需要先安装 Tailwind CSS、PostCSS 和 Autoprefixer，这项工作在上一小节已经完成了。这里，我们只需要安装 Flowbite 本身就行了。

### 2.1 安装 Flowbite

执行如下命令安装 Flowbit :

```bash
npm install flowbite
```

在 `tailwind.config.js` 文件添加 Flowbit 插件：

```
module.exports = {
	省略...
    plugins: [
        require('flowbite/plugin')
    ]

}
```

另外, 还需要在 `tailwind.config.js` 文件中，添加 `js` 相关的文件，因为页面交互需要 `js`：

```
module.exports = {

    content: [
        "./node_modules/flowbite/**/*.js"
    ]

}
```

重新执行 `npm run dev` 命令使之生效。

## 3 使用 Flowbite

Flowbit 支持的组件列表，大家可以去官网：[https://flowbite.com/docs/components/accordion/](https://flowbite.com/docs/components/accordion/) 查看:
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/daf5a89ab236e42621b36e5d06070bb7.png)

这里我们在 `/frontend/index.vue` 首页文件中添加 `Navbar` 导航栏头的源码，并将几处跳转的地方改为中文，代码如下：
```java
<template>  
  <!--顶部Navbar，使用Flowbite组件-->  
  <nav class="bg-white border-gray-200 border-b dark:bg-gray-900">  
    <div class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4">  
      <a href="/" class="flex items-center">  
        <img src="https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e822bf9c8372a73cc51bc55f70262841.png" class="h-8 mr-3" alt="DKBlog Logo" />  
        <span class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white">DK Blog</span>  
      </a>  
      <div class="flex md:order-2">  
        <button type="button" data-collapse-toggle="navbar-search" aria-controls="navbar-search"  
                aria-expanded="false"  
                class="md:hidden text-gray-500 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-sm p-2.5 mr-1">  
          <svg class="w-5 h-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"  
               viewBox="0 0 20 20">  
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                  d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />  
          </svg>  
          <span class="sr-only">Search</span>  
        </button>  
        <div class="relative hidden md:block">  
          <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">  
            <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true"  
                 xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
              <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                    d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />  
            </svg>  
            <span class="sr-only">Search icon</span>  
          </div>  
          <input type="text" id="search-navbar"  
                 class="block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"  
                 placeholder="📝找点啥呢...">  
        </div>  
        <button data-collapse-toggle="navbar-search" type="button"  
                class="inline-flex items-center p-2 w-10 h-10 justify-center text-sm text-gray-500 rounded-lg md:hidden hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-gray-200 dark:text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-600"  
                aria-controls="navbar-search" aria-expanded="false">  
          <span class="sr-only">Open main menu</span>  
          <svg class="w-5 h-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"  
               viewBox="0 0 17 14">  
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                  d="M1 1h15M1 7h15M1 13h15" />  
          </svg>  
        </button>  
      </div>  
      <div class="items-center justify-between hidden w-full md:flex md:w-auto md:order-1" id="navbar-search">  
        <div class="relative mt-3 md:hidden">  
          <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">  
            <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true"  
                 xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
              <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  
                    d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />  
            </svg>  
          </div>  
          <input type="text" id="search-navbar"  
                 class="block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"  
                 placeholder="📝找点啥呢...">  
        </div>  
        <ul            class="flex flex-col p-4 md:p-0 mt-4 font-medium border border-gray-100 rounded-lg bg-gray-50 md:flex-row md:space-x-8 md:mt-0 md:border-0 md:bg-white dark:bg-gray-800 md:dark:bg-gray-900 dark:border-gray-700">  
          <li>  
            <a href="#"  
               class="block py-2 pl-3 pr-4 text-white bg-blue-700 rounded md:bg-transparent md:text-blue-700 md:p-0 md:dark:text-blue-500"  
               aria-current="page">首页</a>  
          </li>  
          <li>            <a href="#"  
               class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 md:dark:hover:text-blue-500 dark:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">分类</a>  
          </li>  
          <li>            <a href="#"  
               class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 dark:text-white md:dark:hover:text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">标签</a>  
          </li>  
          <li>            <a href="#"  
               class="block py-2 pl-3 pr-4 text-gray-900 rounded hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 dark:text-white md:dark:hover:text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">归档</a>  
          </li>  
        </ul>  
      </div>  
    </div>  
  </nav>  
</template>

<script setup>  
import {onMounted} from "vue";  
import {initCollapses} from "flowbite";  
  
// 初始化 flowbite 相关组件  
onMounted(()=>{  
  initCollapses();  
})  
</script>  
```

解释一下 `<script>` 中的代码：

- `onMounted` 是一个生命周期钩子（lifecycle hook）。生命周期钩子是 Vue 组件在其生命周期的不同阶段可以调用的函数。`onMounted` 钩子在组件被挂载到 DOM 之后立即调用。这意味着，当此钩子被触发时，你可以安全地访问和操作组件的 DOM 元素；
    
- `initCollapses()` 用于初始化 `flowbite` 的 `collapse` 组件，有了它，当页面在移动端展示时，点击菜单收缩按钮，可查看隐藏的菜单选项，如下图所示：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c8ad4331f2ab4ab283f8a64571d8d99.png)
然后，在 `main.css` 中添加 `<body>` 标签的基本样式：
```css
body { font-family: -apple-system-font,BlinkMacSystemFont,Helvetica Neue,PingFang SC,Hiragino Sans GB,Microsoft YaHei UI,Microsoft YaHei,Arial,sans-serif;
color: #4c4e4d;
font-size: 16px;
background: #f4f4f4;
line-height: 1.6;
}
```

保存后，查看效果，如下图所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2b9aff08536ab9c4d32bd2964f88cd3c.png)

## 4 添加【登录】超链接

接下来，我们要完成的第一个完整页面就是 _登录页_，这就需要在导航栏中的搜索框后面，添加一个跳转超链接，代码如下：

```
<!-- 登录 -->
<div class="text-gray-900 ml-2 mr-1 hover:text-blue-700">登录</div>
```

## 5 结语

本小节中，带着大家在 `blog` 项目中整合了 `Flowbite` 组件库，并将首页的头部导航栏完成了。注意，我们暂时只完成首页的导航栏部分，下小节我们先将登录页搞定，在后续小节中，再回来将首页开发完成。