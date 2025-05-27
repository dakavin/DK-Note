---
文章标题: "[[3_AdminMenu：样式布局]]" 
文章作者: Dakkk
文章概要: |
  文章详述Vue项目结合Element Plus构建后台管理系统左侧菜单。内容涵盖组件布局、样式定制、Logo集成，以及通过Vue Router实现菜单动态渲染、选中状态管理和点击跳转功能。
文章标签:
- "Element Plus"
- "Vue.js"
- "菜单组件"
- "后台管理"
- "Vue Router"
- "UI样式"
- "前端开发"
相关文章:
- "[[1_搭建管理后台基本布局]]"
- "[[1_登录页设计：支持响应式布局]]"
- "[[3_整合 vue-router 路由管理器]]"
- "[[16_Vue]]"
- "[[17_Ajax]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/3_AdminMenu：样式布局.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-25 20:56:53
修改时间: 2024-04-26 20:27:34
---

## 1 本小节目标

完成本小节的功能开发，你应该达到下面的效果：除了菜单布局样式，还能够点击不同的菜单，内容区域渲染菜单对应的页面：

![](https://img.quanxiaoha.com/quanxiaoha/169422557336260)

## 2 布局分析

分析一波左侧栏的布局结构，分为两个部分：

- ①：上面的 Logo 图片，高度应该和右边的 `AdminHeader` 高度保持一致；
- ②：下面的菜单栏；

![](https://img.quanxiaoha.com/quanxiaoha/169422086039135)

## 3 开始动手

搞清楚布局结构后，我们开始动手。

### 3.1 Element Plus Menu 菜单组件

一个可以为网站提供导航功能的菜单组件。官方提供的类型也有很多，可访问官网地址 [https://element-plus.org/zh-CN/component/menu.html#%E4%BE%A7%E6%A0%8F](https://element-plus.org/zh-CN/component/menu.html#%E4%BE%A7%E6%A0%8F) 来查看详情，这里我们的场景需求是侧边栏菜单，如下图这种:

![](https://img.quanxiaoha.com/quanxiaoha/169422115600719)

_点击右下角查看源代码_，复制白色的那个菜单组件代码，格式如下：

```html
<el-menu
	default-active="2"
	class="el-menu-vertical-demo"
	@open="handleOpen"
	@close="handleClose"
>
	<el-sub-menu index="1">
		<template #title>
			<el-icon><location /></el-icon>
			<span>Navigator One</span>
		</template>
		<el-menu-item-group title="Group One">
			<el-menu-item index="1-1">item one</el-menu-item>
			<el-menu-item index="1-2">item two</el-menu-item>
		</el-menu-item-group>
		<el-menu-item-group title="Group Two">
			<el-menu-item index="1-3">item three</el-menu-item>
		</el-menu-item-group>
		<el-sub-menu index="1-4">
			<template #title>item four</template>
			<el-menu-item index="1-4-1">item one</el-menu-item>
		</el-sub-menu>
	</el-sub-menu>
	<el-menu-item index="2">
		<el-icon><icon-menu /></el-icon>
		<span>Navigator Two</span>
	</el-menu-item>
	<el-menu-item index="3" disabled>
		<el-icon><document /></el-icon>
		<span>Navigator Three</span>
	</el-menu-item>
	<el-menu-item index="4">
		<el-icon><setting /></el-icon>
		<span>Navigator Four</span>
	</el-menu-item>
</el-menu>
```

解释一下事件部分：
- `@open="handleOpen"` : 监听菜单打开事件；
- `@close="handleClose"` : 监听菜单关闭事件；

### 3.2 布局实现

上面这两个事件，我们都用不到，删除掉就行。然后整合到 `AdminMenu.vue` 组件中，代码如下：

```html
<div class="bg-slate-800 h-screen text-white">  
  <!-- 顶部 Logo, 指定高度为 64px, 和右边的 Header 头保持一样高 -->  
  <div class="flex items-center justify-center h-[64px]">  
    Logo  
  </div>  
  
  <!-- 下方菜单 -->  
  <el-menu  
      default-active="2"  
      class="el-menu-vertical-demo"  
  >  
    <el-sub-menu index="1">  
      <template #title>  
        <el-icon><location /></el-icon>  
        <span>Navigator One</span>  
      </template>  
      <el-menu-item-group title="Group One">  
        <el-menu-item index="1-1">item one</el-menu-item>  
        <el-menu-item index="1-2">item two</el-menu-item>  
      </el-menu-item-group>  
      <el-menu-item-group title="Group Two">  
        <el-menu-item index="1-3">item three</el-menu-item>  
      </el-menu-item-group>  
      <el-sub-menu index="1-4">  
        <template #title>item four</template>  
        <el-menu-item index="1-4-1">item one</el-menu-item>  
      </el-sub-menu>  
    </el-sub-menu>  
    <el-menu-item index="2">  
      <el-icon><icon-menu /></el-icon>  
      <span>Navigator Two</span>  
    </el-menu-item>  
    <el-menu-item index="3" disabled>  
      <el-icon><document /></el-icon>  
      <span>Navigator Three</span>  
    </el-menu-item>  
    <el-menu-item index="4">  
      <el-icon><setting /></el-icon>  
      <span>Navigator Four</span>  
    </el-menu-item>  
  </el-menu>  
</div>
```

上述代码中，我们首先定义了一个 `<div>` 来存放 Logo ，然后将刚刚的菜单组件代码复制了进来。完成上面工作后，效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169422157498089)

### 3.3 样式问题

可以看到基本布局有了，但是菜单栏的背景色、字体色和我们定义的 `bg-slate-800` 背景非常不搭，原因是菜单组件本身的样式导致，需要重写一下。通过 _F12_ 审查元素，找到需要重写的样式选择器，在 `AdminMenu.vue` 组件中添加 `<style>` 样式代码, 代码如下：
```html
<style>
.el-menu {
    background-color: rgb(30 41 59 / 1);
    border-right: 0;
}

.el-sub-menu__title {
    color: #fff;
}

.el-sub-menu__title:hover {
    background-color: #ffffff10;
}

.el-menu-item.is-active {
    background-color: var(--el-color-primary);
    color: #fff;
}

.el-menu-item.is-active:hover {
    background-color: var(--el-color-primary);
}

.el-menu-item {
    color: #fff;
}

.el-menu-item:hover {
    background-color: #ffffff10;
}

</style>
```

再来看下效果，如下图，这样看着就舒服多了：

![](https://img.quanxiaoha.com/quanxiaoha/169422202365087)

### 3.4 结构调整

因为我们是博客系统，没有复杂的二级菜单、三级菜单，所以 _只需保留一级菜单即可_。精简一下菜单代码，如下：

```html
<!-- 下方菜单 -->  
<el-menu default-active="2" class="el-menu-vertical-demo">  
  <el-menu-item index="1-1">item one</el-menu-item>  
  <el-menu-item index="1-2">item two</el-menu-item>  
  <el-menu-item index="1-3">item three</el-menu-item>  
</el-menu>
```

效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169422229590437)

### 3.5 Logo 制作

为了好看，这里小哈用[稿定设计在线工具](https://www.gaoding.com/) ，制作了一张背景透明的 Logo 图片，格式为 _PNG_，宽高为 _195 x 84 px_：

![](https://img.quanxiaoha.com/quanxiaoha/169422248086356)

制作完成后，将其复制到 `/assets` 文件夹下，命名为 `weblog-log.png` ：

![](https://img.quanxiaoha.com/quanxiaoha/169422267573593)

在代码中，将顶部 Logo 栏 `<div>` 标签内引入刚刚制作的图片，高度设置为了 `60px`：
```html
<template>  
  <div class="bg-slate-800 h-screen text-white">  
    <!-- 顶部 Logo, 指定高度为 64px, 和右边的 Header 头保持一样高 -->  
    <div class="flex items-center justify-center h-[64px]">  
      <img src="/src/assets/AdminLogo.png" class="h-[70px]">  
    </div>  
  
    <!-- 下方菜单 -->  
    <el-menu default-active="2" class="el-menu-vertical-demo">  
      <el-menu-item index="1-1">item one</el-menu-item>  
      <el-menu-item index="1-2">item two</el-menu-item>  
      <el-menu-item index="1-3">item three</el-menu-item>  
    </el-menu>  
  </div>  
</template>
```

效果如下：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0532b26831da5d14cb646eee2eabc922.png)


### 3.6 菜单栏数据

接下来，我们来完善菜单栏数据。在 `<script>` 标签中定义个 `menus` 数组，用于存放菜单需要的数据。数组内是个对象，包含以下属性：

- `name` : 菜单名称；
- `icon` : 菜单图标；
- `path` : 菜单对应的路由；

代码如下：

```js
<script setup>  
const menus = [  
  {    'name':'监控台',  
    'icon':'Monitor',  
    'path':'/admin/index'  
  },  
  {  
    'name': '文章管理',  
    'icon': 'Document',  
    'path': '/admin/article/list',  
  },  
  {  
    'name': '分类管理',  
    'icon': 'FolderOpened',  
    'path': '/admin/category/list',  
  },  
  {  
    'name': '标签管理',  
    'icon': 'PriceTag',  
    'path': '/admin/tag/list',  
  },  
  {  
    'name': '博客设置',  
    'icon': 'Setting',  
    'path': '/admin/blog/setting',  
  },  
]  
</script>
```

有了菜单数据后，编辑菜单代码，改写为通过 `<v-for>` 来循环渲染出菜单节点：

```html
<!-- 下方菜单 -->  
<el-menu>  
  <template v-for="(item,index) in menus" :key="index">  
    <el-menu-item :index="item.path">  
      <el-icon>  
        <component :is="item.icon"></component>  
      </el-icon>  
      <span>{{item.name}}</span>  
    </el-menu-item>  
  </template>  
</el-menu>
```

注意，上述代码中，`<el-menu-item>` 节点的 `index` 属性值为路由路径，目的是，等会可以通过路由地址来指定默认选中的菜单，也可以方便的获取到每个菜单的路由地址。然后，图标部分是通过 `<component>` 动态渲染出来的。看下现在的效果：

![](https://img.quanxiaoha.com/quanxiaoha/169422345253956)

### 3.7 默认选中

`<el-menu>` 组件提供了属性 `default-active` ，可以用来指定菜单默认选中哪个菜单项。我们可以通过当前的路由地址，来动态指定它，代码如下：
```html

// 省略...
<el-menu :default-active="defaultActive">
	// 省略...
</el-menu>

<script setup>
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

// 根据路由地址判断哪个菜单被选中
const defaultActive = ref(route.path)
</script>
```

上述代码中，通过 `route.path` 获取了当前的路由地址，并通过 `ref` 响应式的绑定到了 `defaultActive` 变量上。再次查看效果，你会发现根据路由地址动态选中菜单的效果有了：

![](https://img.quanxiaoha.com/quanxiaoha/169422398366178)

### 3.8 路由跳转

点击不同的菜单，内容区域应该能渲染对应路由的页面，我们来实现一下这个效果，为 `<el-menu>` 组件添加 `@select` 选择事件，代码如下。

```html
// 省略...
<el-menu :default-active="defaultActive" @select="handleSelect">
	// 省略...
</el-menu>

<script setup>

// 省略...

// 菜单选择事件
const handleSelect = (path) => {
    console.log(path)
}
</script>
```

我们在选择事件中，打印了传递进来的 `path` 参数，控制台日志如下：

![](https://img.quanxiaoha.com/quanxiaoha/169422438728536)

可以看到，打印的就是菜单对应的路由地址，这就简单了，直接路由跳转就行：

```js
<script setup>
import { ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const route = useRoute()
const router = useRouter()

// 根据路由地址判断哪个菜单被选中
const defaultActive = ref(route.path)

// 菜单选择事件
const handleSelect = (path) => {
    router.push(path)
}
// 省略...
</script>
```

当真正点击菜单去跳转的时候，你会发现出现了空白页面，控制台提示 `[Vue Router warn]: No match found for location with path "/admin/article/list"` 警告信息：

![](https://img.quanxiaoha.com/quanxiaoha/169422453901550)

原因是，我们还没创建对应路由的页面，`Vue Router` 提示找不到匹配的路由。

### 3.9 创建菜单对应页面

在 `/pages/admin` 目录下分别创建 _文章管理页、分类管理页、标签管理页、博客设置页_ 4个页面：

![](https://img.quanxiaoha.com/quanxiaoha/169422513632838)

#### 3.9.1 文章管理页 article-list.vue

内容给个提示文字就行，后续再完善。代码如下：
```html
<template>
    <div>
        文章管理页
    </div>
</template>
```

#### 3.9.2 分类管理页 category-list.vue

```html
<template>
    <div>
        分类管理页
    </div>
</template>
```

#### 3.9.3 标签管理页 tag-list.vue

```html
<template>
    <div>
        标签管理页
    </div>
</template>
```

#### 3.9.4 博客设置 blog-setting.vue

```html
<template>
    <div>
        博客设置页
    </div>
</template>
```

### 3.10 注册对应路由

页面创建好了以后，还需在 `/router/index.js` 中声明好对应的路由，代码如下：

```
import AdminArticleList from '@/pages/admin/article-list.vue'
import AdminCategoryList from '@/pages/admin/category-list.vue'
import AdminTagList from '@/pages/admin/tag-list.vue'
import AdminBlogSetting from '@/pages/admin/blog-setting.vue'

// 统一在这里声明所有路由
const routes = [
    // 省略...
    {
        path: "/admin", // 后台首页
        component: Admin,
        // 使用到 admin.vue 布局的，都需要放置在其子路由下面
        children: [
            {
                path: "/admin/index",
                component: AdminIndex,
                meta: {
                    title: '仪表盘'
                }
            },
            {
                path: "/admin/article/list",
                component: AdminArticleList,
                meta: {
                    title: '文章管理'
                }
            },
            {
                path: "/admin/category/list",
                component: AdminCategoryList,
                meta: {
                    title: '分类管理'
                }
            },
            {
                path: "/admin/tag/list",
                component: AdminTagList,
                meta: {
                    title: '标签管理'
                }
            },
            {
                path: "/admin/blog/setting",
                component: AdminBlogSetting,
                meta: {
                    title: '博客设置'
                }
            },
        ]
        
    }
]

```

> ⚠️ 注意：声明的路由地址需要和菜单组件定义的路由保持一致。

## 4 看看效果

路由声明完成后，我们来看看最终效果：

![](https://img.quanxiaoha.com/quanxiaoha/169422557336260)

可以看到，当我们点击不同的菜单，内容区域能够动态渲染对应的页面啦~

## 5 结语

本小节中，主要带着大家将左侧栏的菜单布局搞定了，重写了 Element Plus 菜单组件的相关样式，使之与目前的黑色背景保持一致。然后，能够根据菜单对应的路由地址，实现点击不同菜单，内容区域渲染各自对应的页面。又搞定一个功能，冲冲冲！