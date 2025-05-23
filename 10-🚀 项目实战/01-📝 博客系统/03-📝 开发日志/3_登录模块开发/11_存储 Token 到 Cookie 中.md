
前面小节中，我们已经成功调用了后台登录接口，当用户名、密码正确时，也拿到了 `Token` 令牌。但是登录成功后，并没有对 `Token` 做任何存储操作，正确的做法一般是存储到 `Cookie` 中，以方便后续调用受保护接口时，从 `Cookie` 中获取它，并设置到请求头中去。

本小节中，就带着大家通过 `VueUse` 搞定如何在 `vue` 中实现添加、删除、获取 `Cookie` 值。

## 1 什么是 Cookie ?

Cookie（或者称为 HTTP Cookie 或 Web Cookie）是一种用于在客户端和服务器之间传递信息的小型文本文件。它通常由服务器发送给用户的浏览器，然后浏览器将这些信息存储在用户的计算机上，以备将来的使用。

以下是 Cookie 的一些关键特点和用途：

1. **持久性和会话性**: Cookie 可以分为两种类型：==会话 Cookie== 和 ==持久 Cookie==。会话 Cookie 存储在用户的计算机上，直到用户关闭浏览器，而持久 Cookie 可以在指定的时间段内保留在用户的计算机上。
2. **用于状态管理**: 最常见的用途之一是在 Web 应用程序中管理用户的状态信息。例如，当用户登录时，服务器可以创建一个包含用户标识的 Cookie，以便在后续请求中识别用户。
3. **跟踪和分析**: 网站可以使用 Cookie 跟踪用户的活动，例如浏览历史、购物车内容等。这可以用于分析用户行为和提供个性化的体验。
4. **认证和授权**: Cookie 也可以用于认证和授权用户。一些网站使用 Cookie 来确定用户是否已登录，并根据其权限控制其访问。
5. **个性化内容**: Cookie 可以用于存储用户的首选项和设置，以便提供个性化的内容和用户体验。
6. **广告和跟踪**: 一些广告商和分析工具使用 Cookie 来跟踪用户的浏览活动，以便投放相关广告和收集统计数据。
7. **安全性**: 虽然 Cookie 主要用于传递无害的文本信息，但在某些情况下，它们可能被滥用以进行安全攻击，例如跨站脚本（XSS）和跨站请求伪造（CSRF）。因此，开发人员需要小心处理 Cookie，确保它们不会导致安全问题。

在 `weblog` 项目中，用户登录成功后获取的 `Token` 令牌就是存储在 `Cookie` 中。

## 2 VueUse 是什么？

VueUse 是一个**用于 Vue.js 应用程序的工具库**，旨在提供一组可重复使用的 Vue Composition API 函数，以简化和增强 Vue.js 组件的开发，其官方访问地址为：[https://vueuse.org/](https://vueuse.org/)
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/878ca3d829b4ad7c97c3cd2ec5b0768a.png)

VueUse 提供了一些常见功能的封装，以便你可以更轻松地处理状态管理、副作用、计算属性等等。

以下是一些 VueUse 提供的功能和用途：
1. **状态管理**: VueUse 提供了一些用于处理状态管理的函数，包括 `useLocalStorage`、`useSessionStorage` 等，可以让你轻松地在==本地存储中保存和检索数据==。
2. **副作用和生命周期管理**: VueUse 提供了用于处理副作用和生命周期的函数，例如 `useDebounce` 和 `useThrottle`，可以帮助你==控制函数的执行频率==。
3. **计算属性**: 它包含了一些用于处理计算属性的函数，如 `useComputed`，可以方便地==定义计算属性并自动进行依赖追踪==。
4. **事件处理**: VueUse 提供了一些处理事件的函数，如 `useEventListener`，可以方便地==添加和移除事件监听器==。
5. **浏览器 API 封装**: 它还提供了一些封装浏览器 API 的函数，如 `useMediaQuery` 和 `useOnline`，可以让你更轻松地==访问和响应浏览器的一些特性==。

## 3 为什么要使用 VueUse？

1. **提高开发效率**: VueUse 提供了一组常见功能的抽象和封装，可以帮助你更快地编写 Vue.js 组件，减少重复工作。
2. **代码重用**: VueUse 的函数可以在多个组件中重复使用，提高了代码的可维护性和可复用性。
3. **Vue Composition API 的增强**: VueUse 扩展了 Vue Composition API，提供了更多的功能和工具，使开发更加便捷。
4. **社区支持**: VueUse 是一个活跃的开源项目，有一个强大的社区支持和贡献，你可以从社区中获得帮助和反馈。

## 4 useCookies

前面我们说到了 VueUse 提供了一些常见功能的封装，其中就有对 `Cookie` 操作的相关封装。在搜索框中，输入关键词 `cookie`, 就可以看到 `useCookies` 这个内部封装库了：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/02edf69de4a9b0cf912ae1e6269a11a3.png)

### 4.1 安装

在 `vscode` 中开启一个新的终端，执行以下两行安装命令：
```shell
# useCookies 依赖 integrations，需要先安装它
npm i @vueuse/integrations
# 安装 useCookies
npm i universal-cookie
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4dcd65fc6b27cb123f6ccc59244df661.png)

### 4.2 支持的方法

在官网上可以看到，`useCookies` 提供了一些 `API` ，用于操作 `Cookie` :
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5fc0e37ff8948ad8fe8796de0934a822.png)

解释一下：
- `get` : 获取某个 `cookie`;
- `getAll` : 获取所有 `cookie`；
- `set` : 设置 `cookie`;
- `remove` : 删除某个 `cookie`;
- 以及一些监听相关的 `API` 

## 5 封装一个工具类

我们在 `/src/composables` 文件夹下新建一个 `auth.js` 文件，用于封装和令牌相关的 `cookie` 操作，代码如下：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6069ab9110ee594da9b8f6f3f02ec9ba.png)

```js
import { useCookies } from '@vueuse/integrations/useCookies'

// 存储在 Cookie 中的 Token 的 key
const TOKEN_KEY = 'Authorization'
const cookie = useCookies()

// 获取 Token 值
export function getToken() {
    return cookie.get(TOKEN_KEY)
}

// 设置 Token 到 Cookie 中
export function setToken(token) {
    return cookie.set(TOKEN_KEY, token)
}

// 删除 Token
export function removeToken() {
    return cookie.remove(TOKEN_KEY)
}
```
上述工具类中，我们总共封装了三个常用的和 `Token` 相关的方法，分别是获取 `Token` 值、设置 `Token` 值、删除 `Token` 值。

## 6 登录成功后存储 Token

封装好了工具类后，再来改造一下登录的逻辑，将 `Token` 存储到 `Cookie` 中，以方便后续读取，主要修改的代码如下：
```js
<script setup>
// 省略...
import { setToken } from '@/composables/auth'

// 省略...

const onSubmit = () => {
    formRef.value.validate((valid) => {
        
        // 省略...

        // 调用登录接口
        login(form.username, form.password).then((res) => {
            // 判断是否成功
            if (res.data.success == true) {
                // 省略...

                // 存储 Token 到 Cookie 中
                let token = res.data.data.token
                setToken(token)

                // 省略...
            }
        })
    })
}

</script>
```

## 7 测试一波

重新==登录==一下账号，看看是否能够正确设置 `Token` 令牌到 `cookie` 中。查看步骤如下：
- 右键，点击 _检查_，或者按 `F12`;
- 点击 `Applicaiotn(应用)` 选项卡；
- 左边栏有个 `Cookies`, 如下图所示，选择对应的访问链接，即可看到已经存储成功的 `Token` 令牌啦~

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/989c1f027e3b134e1d2b007b5df5445b.png)

## 8 结语

本小节中，我们在 `weblog` 项目中引入了 VueUse， 通过 `useCookies` 库封装了操作 `Cookie` 的 3 个常用方法，最后改造了登录逻辑，将 `Token` 令牌成功存储到 `Cookie` 中。