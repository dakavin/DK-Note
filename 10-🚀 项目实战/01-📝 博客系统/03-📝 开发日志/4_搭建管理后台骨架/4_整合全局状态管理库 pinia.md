---
文章标题: "[[4_整合全局状态管理库 pinia]]" 
文章作者: Dakkk
文章概要: |
  文章指导在Vue前端项目集成Pinia进行全局状态管理。它阐述了Pinia的特性、安装、创建Store（以菜单宽度为例）和注册流程，旨在帮助开发者高效地在跨组件或页面间共享数据。
文章标签:
- "Pinia"
- "Vue.js"
- "状态管理"
- "全局状态"
- "前端开发"
- "Store"
- "数据共享"
相关文章:
- "[[16_Vue3 状态管理pinia]]"
- "[[11_存储 Token 到 Cookie 中]]"
- "[[16_Vue]]"
- "[[17_Ajax]]"
- "[[18_日程管理开发]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/4_整合全局状态管理库 pinia.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-25 20:56:53
修改时间: 2024-04-26 20:27:34
---

在 `VUE`前端项目开发中，经常会需要==跨组件或页面来共享数据==。那么就需要用到全局状态管理了。本小节中，将带着大家在 `weblog` 前端项目中整合全局状态管理库 `Pinia`。

## 1 为什么取名 Pinia？

Pinia (发音为 `/piːnjʌ/`，类似英文中的 “peenya”) 是最接近有效包名 piña (西班牙语中的 _pineapple_，即“菠萝”) 的词。 菠萝花实际上是一组各自独立的花朵，它们结合在一起，由此形成一个多重的水果。 与 Store 类似，每一个都是独立诞生的，但最终它们都是相互联系的。 它(菠萝)也是一种原产于南美洲的美味热带水果。

## 2 为什么要使用 Pinia？

Pinia 是 Vue 的专属状态管理库，它允许你跨组件或页面共享状态，更详细文档请访问官网地址：[https://pinia.vuejs.org/zh/](https://pinia.vuejs.org/zh/) 。它具备如下特性：

- 💡 **所见即所得** ：与组件类似的 Store。其 API 的设计旨在让你编写出更易组织的 store；
- 🔑 **类型安全**：类型可自动推断，即使在 JavaScript 中亦可为你提供自动补全功能！
- ⚙️ **开发工具支持**：不管是 Vue 2 还是 Vue 3，支持 Vue devtools 钩子的 Pinia 都能给你更好的开发体验。
- 🔌 **可扩展性** ： 可通过事务、同步本地存储等方式扩展 Pinia，以响应 store 的变更。
- 🏗 **模块化设计**：可构建多个 Store 并允许你的打包工具自动拆分它们。
- 📦 **极致轻量化**: Pinia 大小只有 1kb 左右，你甚至可能忘记它的存在！

## 3 安装 Pinia

在 VSCode 中，打开一个终端，运行以下命令来安装 Pinia：

```shell
npm install pinia
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3c8775f3514b415521cca3cf85326b62.png)


## 4 创建 Store

Pinia 使用 `store` 来管理应用程序的状态。我们可以在项目中创建一个或多个 `store`，每个 `store` 对应一个特定的状态域。接下来，我们在项目的 `/src` 目录下，创建一个 `/stores` 文件夹，用于统一放置状态管理相关功能代码：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2103bb22e6c8d281182a15fc168b8265.png)


在下小节中，我们将通过 Pinia 来实现此功能，点击菜单收缩、展开图标，即可让左侧菜单栏动态的展开、收缩。这就将菜单宽度定义为一个全局状态。所以，在 `/stores` 文件夹中，先新建一个 `menu.js` 文件，用于声明菜单相关的全局状态，代码如下：
```js
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useMenuStore = defineStore('menu', () => {
  // 左边栏菜单默认宽度
  const menuWidth = ref("250px")
  
  // 展开或伸缩左边栏菜单
  function handleMenuWidth() {
      menuWidth.value = menuWidth.value == '250px' ? '64px' : '250px'
  }
  
  return { menuWidth, handleMenuWidth }
})
```

上述代码中，我们先是引入了 `pinia` 的`defineStore()` 函数，它用户声明 `store`。它的第一个参数是个字符串，通过它定义了一个名为 `menu` 的全局 `store`, 注意，名称在应用中 _必须是唯一的_ 。另外，将返回的函数命名为 _use..._ 这个格式，它符合组合式函数风格的约定。

`defineStore()` 的第二个参数可接受两类值：`Setup` 函数或 `Option` 对象。上面的代码中，我们在函数内声明了一个 `menuWidth` 菜单宽度的全局变量，并且是响应式的；然后还定义了一个展开/收缩菜单栏的方法，在该方法中，若当前菜单宽度为 `250px` 则赋值为 `64px`, 表示收缩；反之，则展开。最后，将 `menuWidth` 变量和 `handleMenuWidth` 方法返回，以供外界调用。

## 5 注册 Pinia

想要使用 Pinia, 还需要在 `main.js` 文件中来注册 Pinia，添加代码如下：

```js
// 引入全局状态管理 Pinia
import { createPinia } from 'pinia'

const pinia = createPinia()

// 应用 Pinia
app.use(pinia)
```

## 6 使用 Store

我们前面已经定义好了一个菜单 `Store` ，如何使用它呢？这小节先卖个关子，我们将在下小节实现菜单栏收缩、展开功能时，具体会讲要如何使用 `Store`, 本小节先整合到项目中来。

## 7 结语

Pinia 是一个强大状态管理库，可以帮助你更好地管理和组织应用程序的状态。通过创建 `store`，你可以轻松地将状态逻辑封装在一个地方，然后在组件中使用它。这种集成使得状态管理更加简单和可维护，特别适用于大型应用程序。