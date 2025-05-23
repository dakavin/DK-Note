
## 1 什么是 Vue Router？

Vue Router 是 Vue.js 官方提供路由管理器。它与 Vue.js 核心深度集成，让构建单页面应用（Single Page Applications，SPA）变得轻而易举。

## 2 为什么需要 Vue Router？

在一个标准的单页面应用中，仅有一个 HTML 页面被服务器发送到客户端。随后的页面内容都是通过 JavaScript 动态替换生成的。这时候，就需要 Vue Router 来管理这些页面的导航和组织。

## 3 开始设置

### 3.1 安装 vue-router

在命令行中，执行如下命令，安装 `vue-router`:

```shell
npm install vue-router
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/74b337facb41e2c9c5f9a7e5630d8658.png)

### 3.2 配置 Router

**创建首页**

在 `src` 目录下，创建 `/pages` 文件夹，此文件夹中统一存放_页面_相关代码，然后 `/pages` 文件夹下，再创建两个文件夹, 分别是：

- `/admin` : 存放管理后台相关页面；
- `/frontend` ：存放前台展示相关页面，如首页、博客详情页等；
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c30a6abf5719cf9dac7f7eeaafd61866.png)
在 `/frontend` 文件夹下，创建 `index.vue` 首页文件，代码如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f85efcf642c6371b001fb6170e6dfaff.png)

---

**设置路由**

在 `/src` 目录下，新建 `/router` 路由文件夹，用于存放路由相关代码，并在此文件夹下，新建 `index.js` 文件, 代码如下：
```java
import Index from '../pages/frontend/index.vue'  
import {createRouter,createWebHashHistory} from "vue-router";  
  
// 统一在这里声明所有路由  
const routes = [  
    {        // 路由路径  
        path: '/',  
        // 对应组件  
        component: Index,  
        // meta信息  
        meta: {  
            // 页面标题  
            title: 'blog首页'  
        }  
    }  
]  
  
// 创建路由  
const router = createRouter({  
    // 指定路由的历史管理方式，hash 模式指的是 URL 的路径是通过 hash 符号（#） 进行标识  
    history: createWebHashHistory(),  
    // routes： routes 的缩写  
    routes,  
})  
  
// ES6模块导出语句，它用于将 router 对象导出，以便其他文件可以导入和使用这个对象  
export default router;
```

上述代码中，我们先通过 `import` 关键词导入了 `index.vue` 组件，以及 `vue-router` 路由中的相关方法。然后，我们定义一个 `routes` 数组，用于统一声明所有路由。最后，我们创建了路由，并使用 `export default` 关键字导出了 `router` 对象。

---

**使用 router-view 组件**

`<router-view>` 是 Vue Router 的一个核心组件，它是一个功能性组件（functional component）。其主要作用是根据当前的路由（URL）动态地渲染对应的组件，相当于是一个“占位符”，它会自动渲染与当前路径匹配的组件。

作用：
- 1、**动态渲染组件**： `<router-view>` 根据当前的 URL，自动渲染与当前路径匹配的组件。
- 2、**嵌套路由**： 在有嵌套路由的情况下，`<router-view>` 可以用于渲染嵌套的子路由组件。
    
接下来，我们需要在 `app.vue` 中添加该组件，**用于动态渲染路由对应的组件**：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f492c83808bc25fa361556de9f0cdef5.png)

---

**将Router添加到Vue实例**

最后，要想路由生效，还需要打开 `src/main.js` 文件，将 `router` 导入并添加到 Vue `app` 实例中，应用刚刚声明的路由
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/00a9407b1557782d259414f31892977e.png)

---

**启动项目看看效果**

如果你的项目已经启动了，则直接浏览器访问 `http://localhost:5173/`，无需重启，前端所有新修改的代码都会实时的被动态加载。如果未启动项目，则控制台执行 `npm run dev` 启动项目后再查看，效果图如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/73e57f2c84cacb63ab8f17f26d6ab165.png)


可以看到，路由生效了，且能够正常渲染出 `index.vue` 组件。


## 4 小结

本小节中，带着大家在 `weblog-vue3` 前端项目中，添加了路由管理器，并创建了一个首页文件，使得它能够根据 URL 动态被渲染出来。
