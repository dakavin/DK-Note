
## 1 Element Plus 是什么？

Element Plus 是一个专为 Vue 3 设计的 UI 组件库。与 Vue 2 的 Element UI 相似，它提供了一整套丰富且高质量的组件，从基本的按钮和输入框到复杂的日期选择器和数据表格。这些组件不仅样式美观，而且具有广泛的定制选项。_同时，它也是目前企业级 Admin 管理后台中，使用广度非常高的一个组件框架，所以，针对 `blog` 后台相关页面，决定使用它作为技术栈，更加贴合实际企业场景。_

## 2 为什么使用 Element Plus？

- **为 Vue 3 量身打造**：尽管存在许多 UI 组件库，但 Element Plus 是专门为 Vue 3 设计的，确保最大的兼容性和性能。
- **高效开发**：使用预制的 UI 组件可以大大提高开发速度，减少从零开始编写 UI 代码的时间。
- **一致性和专业性**：Element Plus 提供的组件在设计上保持了一致性，使得你的应用看起来更为专业。
- **广泛的社区支持**：由于 Element UI 和 Vue 的广泛使用，Element Plus 也有一个日益增长的社区，这意味着丰富的资源、插件和持续的更新。
- **企业项目中使用频次非常多**：它是目前国内企业级后台项目中，使用频率非常高的一个组件库，为了贴合企业级开发，也是小哈将它作为技术选型的重要依据。

## 3 整合 Element Plus

### 3.1 安装

打开命令行，通过 `npm` 执行如下命令来安装 Element Plus:

```
npm install element-plus --save
```

### 3.2 自动导入

Element Plus 有很多组件，而我们实际项目中，并不是每个组件都会被用到。所以，在生产级项目中，比较**推荐按需导入组件**，而不是在打包的时候，一次性将所以组件都打包进去，导致相关包非常大，页面初次加载很慢的情况发生。

要想实现按需导入组件，你需要安装`unplugin-vue-components` 和 `unplugin-auto-import`这两款插件：

```shell
npm install -D unplugin-vue-components unplugin-auto-import
```

然后把下列代码插入到你的 `Vite` 配置文件 `vite.config.js` 中:

```js
import { defineConfig } from 'vite'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  // ...
  plugins: [
    // ...
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
})
```

完成上面工作后，在控制台 执行 `npm run dev` 重新构建一下项目。

## 4 开始使用

### 4.1 新建一个登录页

为了测试一下目前项目是否正确整合了 `Elment Plus` 组件库，先来新建一个登录页，等下在该页面中，添加一些 Element Plus 组件，看看能够正确被展示。另外，刚好下面一章进入登录模块的开发，提前建好。在 `/pages/admin/` 中新建一个 `login.vue` 登录页面。

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d60c6ea839ee47d1f3cc8d1749e028c.png)

### 添加登录页路由

页面新建好了后，还需要在 `/router/index.js` 中添加对应的路由：

```js
import Login from '@/pages/admin/login.vue'

// 统一在这里声明所有路由
const routes = [
    // 省略...
    {
        path: '/login', // 登录页
        component: Login,
        meta: {
            title: 'Weblog 登录页'
        }
    },
]
    // 省略...
```

路由添加好了后，编辑 `/pages/frontend/index.vue` 首页文件，为登录按钮添加点击事件，实现跳转功能：

```vue
<!-- 登录 -->
<div class="text-gray-900 ml-2 mr-1 hover:text-blue-700" @click="$router.push('/login')"
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e1da55bcfdd934deaa69d2c79c0afd0.png)


### 在登录页中引入组件

Element Plus 提供了非常多的组件以供选择，小伙们可访问官网查看：[https://element-plus.org/zh-CN/component/button.html](https://element-plus.org/zh-CN/component/button.html)

我们将按钮相关的组件代码，通过_右下角查看源码_， 即可看到每个组件对应的源码。复制它们到登录页 `/pages/admin/login.vue` 中：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59e2c1cb2231fa8e5182458d2989863b.png)

访问登录页，看下效果：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a942056643958645fb249b363075fe5.png)

可以看到，按钮样式正确展示，这说明我们整合 Element Plus 成功了，后面按自己需要，在官网选择自己想要的组件代码，贴到项目中，按需修改就行了。

## 结语

在日益繁忙和要求即时交付的前端开发领域中，有效利用现有的工具和资源显得尤为重要。Element Plus 为 Vue 3 开发者提供了这样一个强大的工具，不仅节省了手动编写和调整 UI 的时间，还确保了应用在各种设备和场景下都能提供一致和高质量的用户体验。另外一个重要的原因就是，国内的 `Admin` 前端管理后台大部分都是选择的它作为的技术栈，选择它，也是为了更好的贴合实际企业开发场景。