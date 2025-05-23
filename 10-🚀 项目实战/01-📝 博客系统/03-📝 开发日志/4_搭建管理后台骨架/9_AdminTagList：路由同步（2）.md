本小节中，我们继续完善标签导航栏组件中的路由同步功能。上小节中，虽然实现了当页面刷新后，能根据当前路由地址动态激活被选中的 `tab` 标签，但是还是存在一些问题。问题如下：
## 1 点击左侧栏菜单，无法动态添加 tab 标签页

这里我们将 `tabList` 数组中的元素删除掉，只保留 _仪表盘_ 标签，来复现一下这个问题：

```js
// 导航栏 tab 数组
const tabList = ref([
    {
        title: '仪表盘',
        path: "/admin/index"
    },
])
```

![](https://img.quanxiaoha.com/quanxiaoha/169450388724385)

可以看到，正常的功能应该是，当我们点击左侧栏菜单，如果标签导航栏中没有对应的标签页，应该新建一个才对。

### 1.1 功能完善

#### 1.1.1 删除无用代码

上小节代码中，`AdminTagList` 组件中，还有些无用代码没有删除干净：

![](https://img.quanxiaoha.com/quanxiaoha/169450520608639)

将 `addTab` 这个方法删除掉，然后将 `removeTab` 方法块中的代码删除掉，只保留方法声明即可，后面开发删除`tab` 标签, 还需要用此方法：

```js
// 删除 Tab 标签
const removeTab = (targetName) => {
}
```

#### 1.1.2 添加 onBeforeRouteUpdate 生命周期钩子

编辑 `AdminTagList` 组件，添加如下代码：

```js
<script setup>
import { useRoute, onBeforeRouteUpdate } from 'vue-router'

// 在路由切换前被调用
onBeforeRouteUpdate((to, from) => {
	// 打印控制台日志
    console.log({
        title: to.meta.title,
        path: to.path
    }) 
})
</script>
```

上述代码中，我们引入了 `vue-router` 中的 `onBeforeRouteUpdate` 路由生命周期钩子，它会在路由切换前被调用。添加完成后，回到页面中，点击左侧菜单栏分类管理菜单，看看是否能够打印目前页面的标题以及路由地址：

![](https://img.quanxiaoha.com/quanxiaoha/169450430074023)

OK , 正确打印了相关数据。

#### 1.1.3 添加 Tab 标签

添加如下代码:

```js
// 省略...

// 添加 Tab 标签页
function addTab(tab) {
    // 标签是否不存在
    let isTabNotExisted = tabList.value.findIndex(item => item.path == tab.path) == -1
    // 如果不存在
    if (isTabNotExisted) {
        // 添加标签
        tabList.value.push(tab)
    }
}

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

// 省略...
```

上述代码中，我们定义了一个 `addTab()` 添加标签方法。在该方法中，我们首先判断了传入的 `tab` 参数是否在 `tabList` 数组中已经存在，若不存在，则添加，否则不添加。最后在 `onBeforeRouteUpdate` 路由生命周期钩子中，先是设置了需要被激活的 `tab` 标签，然后调用了 `addTab()` 方法来动态添加标签页。

代码添加完毕后，当点击左侧栏菜单后，就能够动态添加了对应的标签页了，同时，如果目标标签页已经存在于导航栏中，则不会添加：

![](https://img.quanxiaoha.com/quanxiaoha/169450639639360)

## 2 点击标签页，内容区域没有反应

目前，当我们点击导航栏中的标签时，内容区域是毫无反应的，这个问题要怎么解决呢？思路其实也非常简单，只需监听 `<el-tabs>` 组件的切换事件，并获取标签对应的路由，然后调用 `push()` 方法跳转路由即可。

查看 [Element Plus 官方文档](https://element-plus.org/zh-CN/component/tabs.html#tabs-%E4%BA%8B%E4%BB%B6) ， 可以看到标签页组件提供了相应的事件方法：

![](https://img.quanxiaoha.com/quanxiaoha/169450671622516)

编辑 `AdminTagList` 组件，修改代码如下：

```js
<el-tabs v-model="activeTab" type="card" class="demo-tabs" @tab-remove="removeTab" @tab-change="tabChange" style="min-width: 10px;">
          // 省略...
</el-tabs>
        
<script setup>
// 标签页切换事件
const tabChange = (path) => {
    console.log(path)
}
</script>        
```

上述代码中，首先给 `<el-tabs>` 组件添加了 `@tab-change="tabChange"` 监听方法, 同时，定义了对应的 `tabChange()` 方法，并打印了入参 `path`, 你会发现，当点击左侧栏菜单时，浏览器控制台能够打印对应的路由地址：

![](https://img.quanxiaoha.com/quanxiaoha/169450711995735)

有了目标路由地址，一切就好办了, 修改代码如下：

```js
<script setup>
// 省略...
import { useRoute, useRouter, onBeforeRouteUpdate } from 'vue-router'
// 省略...

const router = useRouter()

// 标签页切换事件
const tabChange = (path) => {
    // 设置被激活的 Tab 标签
    activeTab.value = path
    // 路由跳转
    router.push(path)
}
// 省略...
</script>
```

上述代码中，先引入了 `router` 路由，然后改写了 `tabChange` 监听事件，里面的逻辑为：先激活目标路由对应的标签页，然后进行路由跳转。效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169450743218166)

现在当我们点击对应标签后，可以正常跳转对应页面了。

## 3 刷新页面，标签页消失了？

_好像标签导航栏都优化完毕了？ No , No, No !_

当我们打开了几个标签页后，再刷新页面，你会发现标签页都消失了。原因是标签页的数据没有被存储起来，就好像登录模块中，登录后需要存储 `token` 令牌一样，为了不丢失标签数据，同样需要将已添加的标签数据存储到 `cookie` 中，当页面重新加载时，再从 `cookie` 中获取，重新渲染到页面。

### 3.1 功能完善

#### 3.1.1 定义工具类

首先，我们在 `/src/composables` 文件夹下，创建一个 `tag-list.js` 工具类，内容如下：

```js
import { useCookies } from '@vueuse/integrations/useCookies'

// 存储在 Cookie 中的标签页数据的 key
const TAB_LIST_KEY = 'tabList'
const cookie = useCookies()

// 获取 TabList
export function getTabList() {
    return cookie.get(TAB_LIST_KEY)
}

// 存储 TabList 到 Cookie 中
export function setTabList(tabList) {
    return cookie.set(TAB_LIST_KEY, tabList)
}
```

和之前小节定义的 `auth.js` 内容相差无几，分别定义了两个方法：
- `setTabList()` : 存储 `TabList` 到 `Cookie` 中;
- `getTabList()` : 从 `cookie` 中获取 `TabList` ；

#### 3.1.2 初始化 cookie 中的标签导航栏数据

定义好工具类后，修改 `AdminTagList` 组件中的代码，内容如下:

```js
// 省略...
import { setTabList, getTabList } from '@/composables/tag-list'

// 省略...
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

// 省略...
```

上述代码中，我们首先引入了刚刚定义好的 `tag-list` 工具类。在 `addTab` 方法的最后面，调用了 `setTabList` 方法，用以每次创建新的标签页时，能够将 `tabList` 数据存储到 `cookie` 中。光是存储还不够，最后，我们定义了一个 `initTabList()` 初始化标签栏数据方法，当每次页面加载完毕后调用它，以重新渲染存储在 `cookie` 中的标签导航栏数据。

#### 3.1.3 效果如下

搞定上面的工作后，再次刷新页面，你会看到标签栏不会消失啦：

![](https://img.quanxiaoha.com/quanxiaoha/169451204197312)

## 5 结语

本小节中，我们主要完善了标签导航栏的路由同步功能，主要包括当点击左侧栏菜单时，能够动态添加 `tab` 标签页、点击标签跳转对应页面，以及存储标签页数据到 `cookie` 中，保证每次刷新页面后，已经被打开的标签页不会莫名其妙的消失。