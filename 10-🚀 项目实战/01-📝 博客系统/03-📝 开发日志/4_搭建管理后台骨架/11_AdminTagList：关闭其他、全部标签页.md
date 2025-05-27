---
文章标题: "[[11_AdminTagList：关闭其他、全部标签页]]" 
文章作者: Dakkk
文章概要: |
  文章详细演示了如何在Vue.js应用中，利用Element Plus的Dropdown组件实现管理员标签页的“关闭其他”和“关闭全部”功能。它通过监听命令事件，结合Vue 3 Composition API和Composables，管理和更新标签列表，并封装了Cookie操作。
文章标签:
- "Vue.js"
- "Element Plus"
- "标签页管理"
- "前端开发"
- "Composition API"
- "Composables"
- "UI组件"
- "状态管理"
相关文章:
- "[[1_登录页设计：支持响应式布局]]"
- "[[7_AdminTagList：样式布局]]"
- "[[11_存储 Token 到 Cookie 中]]"
- "[[2_AdminHeader：样式布局]]"
- "[[3_AdminMenu：样式布局]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/11_AdminTagList：关闭其他、全部标签页.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-25 20:56:55
修改时间: 2025-05-27 00:55:26
---

本小节中，我们将搞定标签导航栏组件中右侧下拉菜单的关闭其他、关闭全部标签页功能。

![](https://img.quanxiaoha.com/quanxiaoha/169459935904043)

## 1 下拉菜单监听事件

从 Element Plus 官网查看 [Dropdown 下拉菜单](https://element-plus.org/zh-CN/component/dropdown) 文档，可以看到 `<el-dropdown>`组件提供了 `command` 事件，当用户点击菜单项时，会触发该回调事件。同时，它需要搭配 `<el-dropdown-item>` 子元素的 `command` 属性一起使用，通过手动指定指令参数，当用户点击不同子菜单时，该指令参数会作为入参传入 `command` 事件函数中。

![](https://img.quanxiaoha.com/quanxiaoha/169459984127430)

## 2 添加监听事件

接下来，我们就来看看要如何使用？编辑 `AdminTagList` 组件， 为 `<el-dropdown>` 添加 `command` 菜单点击事件，代码如下：

```html
// 省略...
<el-dropdown @command="handleCloseTab">
	<span class="el-dropdown-link">
		<el-icon>
			<arrow-down />
		</el-icon>
	</span>
	<template #dropdown>
		<el-dropdown-menu>
			<el-dropdown-item command="closeOthers">关闭其他</el-dropdown-item>
			<el-dropdown-item command="closeAll">关闭全部</el-dropdown-item>
		</el-dropdown-menu>
	</template>
</el-dropdown>
// 省略...

<script setup>
// 处理关闭标签菜单事件
	const handleCloseTab = (command) => {
		console.log(command)
	}
</script>
```

上述代码中，我们首先为 `<el-dropdown-item>` 子组件添加了 `command` 属性，并分别指定了 _关闭其他、关闭全部_ 子菜单的指令参数，分别为 `closeOthers` 、 `closeAll` 。然后，为 `<el-dropdown>` 组件添加了 `@command` 监听事件，在事件方法中打印了参数 `command` 到控制台日志。

重回到页面中，当点击不同菜单时，查看控制台日志，你会发现传入进来的指令参数如下，正是对应菜单指定的 `command` 属性值：

![](https://img.quanxiaoha.com/quanxiaoha/169460071707463)

## 3 添加关闭其他、关闭全部标签逻辑代码

通过传入的指令参数，就知道了用户点击了哪个菜单，完善 `handleCloseTab` 监听方法代码，如下：

```js
// 省略...

// 处理关闭标签菜单事件
const handleCloseTab = (command) => {
    // 首页路由
    let indexPath = '/admin/index'
    // 处理关闭其他
    if (command == 'closeOthers') {
        // 仅过滤出首页和当前页
        tabList.value = tabList.value.filter((tab) => tab.path == indexPath || tab.path == activeTab.value)
    } else if (command == 'closeAll') { // 处理关闭全部
        // 切换回首页
        activeTab.value = indexPath
        // 只保留首页
        tabList.value = tabList.value.filter((tab) => tab.path == indexPath)
		// 切换标签页
        tabChange(activeTab.value)
    }
    
    // 设置到 cookie 中
    setTabList(tabList.value)
}

// 省略...
```

上述代码中，通过对 `command` 参数进行判断，当关闭其他时，则仅过滤出首页和当前页；当关闭全部时，则只保留首页。然后重新设置回 `tabList` 中，最后，再存储到 `cookie` 中。

## 4 查看效果

到这里，关闭其他和关闭全部标签功能就搞定啦，效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169460193197163)

## 5 代码封装

为了组件代码的美观性，我们可以将 `<script>` 标签下的代码统一封装起来，放置到单独一个文件中，最后再在 `AdminTagList` 引用它们。在 `/src/composables` 目录下，新建 `useTagList.js` 文件，将 `AdminTabList` 组件中的 `<script>` 标签下的代码全部剪切过去：

![](https://img.quanxiaoha.com/quanxiaoha/169460268367596)

```js
import { ref } from 'vue'
import { useMenuStore } from '@/stores/menu'
import { useRoute, useRouter, onBeforeRouteUpdate } from 'vue-router'
import { setTabList, getTabList } from '@/composables/tag-list'

export function useTabList() {
    const menuStore = useMenuStore()
    const route = useRoute()
    const router = useRouter()

    // 当前被选中的 tab
    const activeTab = ref(route.path)
    // 导航栏 tab 数组
    const tabList = ref([
        {
            title: '仪表盘',
            path: "/admin/index"
        },
    ])

    // 添加 Tab 标签页
    function addTab(tab) {
        // 标签是否不存在
        let isTabNotExisted = tabList.value.findIndex(item => item.path == tab.path) == -1
        // 如果不存在
        if (isTabNotExisted) {
            // 添加标签
            tabList.value.push(tab)
        }
        // 存储 tabList 到 cookie 中
        setTabList(tabList.value)
    }

    function initTabList() {
        // 从 cookie 中获取缓存起来的标签导航栏数据
        let tabs = getTabList()
        // 若不为空，则赋值
        if (tabs) {
            tabList.value = tabs
        }
    }
    // 初始化标签导航栏
    initTabList()

    // 在路由切换前被调用
    onBeforeRouteUpdate((to, from) => {
        // 设置被激活的 Tab 标签
        activeTab.value = to.path
        // 添加 Tab 标签页
        addTab({
            title: to.meta.title,
            path: to.path
        })
    })

    // 标签页切换事件
    const tabChange = (path) => {
        // 设置被激活的 Tab 标签
        activeTab.value = path
        // 路由跳转
        router.push(path)
    }

    // 删除 Tab 标签
    const removeTab = (path) => {
        let tabs = tabList.value
        // 当前被选中的 tab 标签
        let actTab = activeTab.value

        // 如果要删除的是当前被选中的标签页，则需要判断其被删除后，需要激活哪个 tab 标签页
        if (actTab == path) {
            // 循环 tabList
            tabs.forEach((tab, index) => {
                // 获取被选中的 tab 元素
                if (tab.path == path) {
                    // 拿到被选中的标签页下标，如果它后面还有标签页，则取下一个标签页，否则取上一个
                    let nextTab = tabs[index + 1] || tabs[index - 1]
                    if (nextTab) {
                        actTab = nextTab.path
                    }
                }
            })
        }

        // 需要被激活的标签页
        activeTab.value = actTab

        // 过滤掉被删除的标签页, 重新设置回去
        tabList.value = tabList.value.filter((tab) => tab.path != path)

        // 存储到 cookie 中
        setTabList(tabList.value)
    }

    // 处理关闭标签菜单事件
    const handleCloseTab = (command) => {
        // 首页路由
        let indexPath = '/admin/index'
        // 处理关闭其他
        if (command == 'closeOthers') {
            // 仅过滤出首页和当前页
            tabList.value = tabList.value.filter((tab) => tab.path == indexPath || tab.path == activeTab.value)
        } else if (command == 'closeAll') { // 处理关闭全部
            // 切换回首页
            activeTab.value = indexPath
            // 只保留首页
            tabList.value = tabList.value.filter((tab) => tab.path == indexPath)
			// 切换标签页
            tabChange(activeTab.value)
        }

        // 设置到 cookie 中
        setTabList(tabList.value)
    }

    return {
        menuStore,
        activeTab,
        tabList,
        tabChange,
        removeTab,
        handleCloseTab
    }
}
```

复制过去后，将主要代码封装到 `export function useTabList() {}` 语句块中，最终将需要供 `AdminTabList` 组件使用的变量和方法全部 `return` 出去。

完成上述工作后，回到 `AdminTabList` 组件中，引用 `useTagList.js` 文件 , `<script>` 标签下代码如下：

```js
<script setup>
import { useTabList } from '@/composables/useTagList.js'

const { menuStore, activeTab, tabList, tabChange, removeTab, handleCloseTab } = useTabList()
</script>
```

上述代码中，我们将标签导航栏组件中需要用到的变量和方法全部声明出来。至此，封装工作就完成了，回到页面重新测试一波，不出意外的话，一切功能正常！

## 6 Cookie 操作相关代码统一封装

还有个问题需要优化一下，观察 `/composables` 目录下文件，你会发现命名格式不一致，出现了 _驼峰式_ 和   `-` _连接符_ 两种形式的风格，非常难看，这里统一保持驼峰式风格就好。另外，`auth.js` 和 `tag-list.js` 两个工具类都是用于操作 `cookie` 的，完全可以统一封装到一个工具内：

![](https://img.quanxiaoha.com/quanxiaoha/169460321353228)

在 `/composables` 文件夹下新建 `cookie.js` , 内容如下:

```
import { useCookies } from '@vueuse/integrations/useCookies'

const cookie = useCookies()

// ============================== Token 令牌 ==============================

// 存储在 Cookie 中的 Token 的 key
const TOKEN_KEY = 'Authorization'

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

// ============================== 标签页 ==============================

// 存储在 Cookie 中的标签页数据的 key
const TAB_LIST_KEY = 'tabList'

// 获取 TabList
export function getTabList() {
    return cookie.get(TAB_LIST_KEY)
}

// 存储 TabList 到 Cookie 中
export function setTabList(tabList) {
    return cookie.set(TAB_LIST_KEY, tabList)
}
```

将 `auth.js` 和 `tab-list.js` 两个文件全部_删除_掉。

![](https://img.quanxiaoha.com/quanxiaoha/169460375358010)

最后注意，之前登录模块、和标签导航组件中的相关使用到 `cookie` 代码也需要改造一下，统一使用 `cookie.js` 这个工具类。这里就不贴出了，小伙伴们自己尝试自己动手改一下，然后自己测试一下对应的模块功能，要善于通过控制台错误提示，找到遗漏的、忘记改的地方。若有问题，也可下载小节源码对比看看。

## 8 结语

本小节中，小哈主要带着大家将标签导航栏右边下拉菜单中的，关闭其他、关闭全部标签栏功能开发完成了。最后，我们将相关代码封装了一波，让代码更利于维护一些。