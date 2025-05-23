本小节中，小哈将带着大家实现 Pinia 数据的持久化。在说数据持久化之前，我们先来看个上小节中存在的问题：

## 问题分析

- 1、仔细看，当用户登录成功后，`AdminHeader` 头中成功渲染了用户名，但是，_右键重新加载_ 页面后，_用户名消失了_ ！

![](https://img.quanxiaoha.com/quanxiaoha/169493902177266)

2、还有，当左边侧边栏被收缩起来后，_右键点击加载_ 页面后，_侧边栏又被展开了_！

![](https://img.quanxiaoha.com/quanxiaoha/169493914236129)

原因都是 Pinina 的数据 _没有被持久化_，当我们重新加载页面后，数据就丢失了，_重置回了初始值_，左边栏宽度被重置到了 `250px` , 登录用户名被重置到了空字符串。

## 什么是 Pinia 数据持久化？

Pinia 数据持久化是一种在 Vue 3 应用程序中的状态管理库 Pinia 中的扩展功能。它允许你将应用程序的某些状态数据保存到浏览器的本地存储中，以便在用户关闭浏览器标签、刷新页面或重新访问应用程序时仍然可以访问到这些数据。

在不使用数据持久化的情况下，当用户刷新页面或关闭浏览器后，应用程序的状态数据通常会被重置，用户需要重新进行登录或重新配置应用程序的设置。==通过使用 Pinia 数据持久化，你可以确保这些关键的状态数据在用户会话之间得以保留，提供更好的用户体验==。

## 为什么需要 Pinia 数据持久化？

1. **保持用户登录状态：** 在许多应用程序中，用户的登录状态是关键数据。通过将用户的身份验证令牌或用户名等信息保存在本地存储中，用户可以在关闭浏览器或刷新页面后保持登录状态，而无需重新登录。
2. **保存用户设置和偏好：** 用户可能会在应用程序中进行各种设置和配置，例如选择应用程序的主题、语言或其他偏好设置。通过将这些设置保存在本地存储中，用户可以在以后的会话中保留其个性化设置。
3. **减少服务器请求：** 如果某些数据需要在多个用户会话之间共享，将这些数据保存在本地存储中可以减少对服务器的请求。这可以提高应用程序的性能，减少服务器负载，并提供更快的用户体验。
4. **离线访问：** 数据持久化还使应用程序具备一定程度的离线访问能力。用户可以在没有互联网连接的情况下访问以前保存的数据。

## 三、使用 Pinia 数据持久化插件

接下来，小哈将介绍如何通过 `pinia-persist` 插件来实现 Pinia 数据持久化。关于此插件更详细的文档，可访问[官网](https://prazdevs.github.io/pinia-plugin-persistedstate/zh/) 。

![](https://img.quanxiaoha.com/quanxiaoha/169494195570061)

### 3.1 安装插件

1、执行如下命令，来安装插件：

```
npm i pinia-plugin-persistedstate
```

2、编辑 `main.js` 文件，将插件添加到 Pinia 实例上:

```js
// 省略...

// 引入全局状态管理 Pinia
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

// 省略...

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

// 省略...
```

### 3.2 用户信息 Store 持久化

安装好插件后，我们来看看要如何使用持久化。先给 `/stores/user.js` 添加持久化功能，修改内容如下：

```js
export const useUserStore = defineStore('user', () => {
  // 用户信息
  const userInfo = ref({})

  // 设置用户信息
  function setUserInfo() {
    // 调用后头获取用户信息接口
    getUserInfo().then(res => {
      if (res.success == true) {
        userInfo.value = res.data
      }
    })
  }

  return { userInfo, setUserInfo }
}, 
{
  // 开启持久化
  persist: true,
}
)
```

使用起来也非常简单，添加 `persist: true` 声明后，这个 `store` 就会把对应的数据存储到 `Local Storage` 中。你可以测试看看，重新登录后，通过 _F12_ 打开控制台，依次点击 _Application | Storage | Local Storage_ 找到存储的数据：

![](https://img.quanxiaoha.com/quanxiaoha/169494365503226)

再次刷新页面，你会发现用户名就不会被重置回去啦~

### 3.3 菜单信息 Store 持久化

菜单 `store` 同样以上述方式，来开启持久化功能，编辑 `menu.js` , 添加持久化声明:

```js
export const useMenuStore = defineStore('menu', () => {
  // 左边栏菜单默认宽度
  const menuWidth = ref("250px")
  
  // 展开或伸缩左边栏菜单
  function handleMenuWidth() {
      menuWidth.value = menuWidth.value == '250px' ? '64px' : '250px'
  }
  
  return { menuWidth, handleMenuWidth }
}, 
{
  // 开启持久化
  persist: true,
}
)
```

开启完成后，当左侧栏被收缩起来后，再次重新加载页面，会看到还是处于收缩状态。

## 四、代码优化

### 4.1 封装 Pinia 相关代码

这里我们可以将 Pinia 实例初始化、持久化插件挂载相关的代码统一抽出来，放到一个文件中来管理。在 `/stores` 文件夹下，新建 `index.js` 文件，将 `main.js` 中和 Pinia 相关的代码抽取出来：

![](https://img.quanxiaoha.com/quanxiaoha/169494449083854)

```js
// 引入全局状态管理 Pinia
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
// 持久化插件
pinia.use(piniaPluginPersistedstate)

// 暴露出去
export default pinia
```

最后，通过 `export` 关键词将 Pinia 实例暴露出去。

### 4.2 main.js 引用 Pinia 实例

修改 `main.js` , 引入刚刚创建的 Pinia 实例：

```
// 引入全局状态管理 Pinia
import pinia from '@/stores'
```

如下图所示，这样 Pinia 初始化，以及挂载持久化插件相关的工作就不用写在 `main.js` 中了，代码看起来更清爽一些：

![](https://img.quanxiaoha.com/quanxiaoha/169494434113761)

## 五、总结

本小节中，我们主要通过 Pinia 数据持久化插件 `pinia-persist` 将全局状态持久化到了 `Local Storage` 中，这样无论用户是重新加载页面，还是关闭浏览器，数据都被缓存起来了，不会导致页面再次打开时，导致数据丢失的问题。