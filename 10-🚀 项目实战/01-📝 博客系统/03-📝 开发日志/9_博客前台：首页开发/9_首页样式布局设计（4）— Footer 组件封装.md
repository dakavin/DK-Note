---
文章标题: "[[9_首页样式布局设计（4）— Footer 组件封装]]" 
文章作者: Dakkk
文章概要: |
  本文详细指导了如何封装网站首页的Footer页脚组件。文章利用Flowbite提供的现成组件，演示了创建Footer.vue文件、进行内容定制（如版权、备案号），并使用Tailwind CSS样式布局，最后将其成功引用到主页面布局中。
文章标签:
- "Footer"
- "组件封装"
- "Vue"
- "前端开发"
- "Flowbite"
- "布局设计"
- "Tailwind CSS"
相关文章:
- "[[6_整合 Tailwind CSS组件库：Flowbite]]"
- "[[1_登录页设计：支持响应式布局]]"
- "[[20_小结]]"
- "[[5_整合 Tailwind CSS]]"
- "[[5_AdminMenu：点击收缩、展开功能实现]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/9_博客前台：首页开发/9_首页样式布局设计（4）— Footer 组件封装.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-05-10 09:50:54
修改时间: 2025-05-27 00:59:18
---

本小节中，我们将完成首页样式布局的最后一块拼图 —— Footer 页脚，效果如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/aee6f891c537467a422fa979b04583fc.png)
## 1. 添加页脚组件

`Flowbit` 官方就提供了现成的页脚组件：[https://flowbite.com/docs/components/footer/](https://flowbite.com/docs/components/footer/) 可供使用，在多种样式中，挑一个自己想要的进行复制：

![](https://img.quanxiaoha.com/quanxiaoha/169811372411589)

## 2. 添加 Footer 组件

![](https://img.quanxiaoha.com/quanxiaoha/169811357298587)

在 `/frontend/components` 文件夹下创建 `Footer.vue` 组件，粘贴刚刚复制的组件源码，稍作修改，代码如下：

```
<template>
    <footer class="bg-white mt-5 dark:bg-gray-800">
        <div class="w-full mx-auto max-w-screen-xl py-6 md:flex md:items-center md:justify-between">
            <span class="text-sm text-gray-500 sm:text-center dark:text-gray-400">© 2023 <a href="https://www.quanxiaoha.com/"
                    class="hover:underline">犬小哈</a>. All Rights Reserved.
            </span>
            <ul class="flex flex-wrap items-center mt-3 text-sm font-medium text-gray-500 dark:text-gray-400 sm:mt-0">
                <li>
                    备案号：<a href="#" class="mr-4 hover:underline md:mr-6 ">京ICP备2020016734号</a>
                </li>
            </ul>
        </div>
    </footer>
</template>
```

上述左右布局的代码中，我们去除了一些不必要的元素，左侧改为了个人博主名，右侧则是网站都需要添加的备案号。
## 3. 引用 Footer 组件

编辑 `/frontend/index.vue` 首页，添加刚刚封装的 `Footer.vue` 组件即可：

![](https://img.quanxiaoha.com/quanxiaoha/169811412616015)

```
<template>
    <Header></Header>

    <!-- 主内容区域 -->
    <main class="container max-w-screen-xl mx-auto p-4">
		// 省略...
    </main>

    <Footer></Footer>
</template>

<script setup>
// 省略...
import Footer from '@/layouts/frontend/components/Footer.vue'
// 省略...
</script>
```