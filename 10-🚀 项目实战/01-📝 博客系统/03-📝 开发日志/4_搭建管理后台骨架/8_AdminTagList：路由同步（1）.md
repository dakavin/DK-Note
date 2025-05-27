---
文章标题: "[[8_AdminTagList：路由同步（1）]]" 
文章作者: Dakkk
文章概要: |
  文章详细讲解了如何完善 Vue.js 后台管理系统的标签导航栏，实现路由与标签的同步。通过重新定义标签数据结构，并利用 Vue Router 动态绑定当前路径，确保页面刷新时标签栏能正确选中。此外，还演示了如何设置特定标签（如首页）为不可关闭状态，提升用户体验。
文章标签:
- "Vue.js"
- "Element Plus"
- "路由同步"
- "标签导航"
- "后台管理系统"
- "组件开发"
- "Vue Router"
相关文章:
- "[[1_搭建管理后台基本布局]]"
- "[[3_AdminMenu：样式布局]]"
- "[[1_登录页设计：支持响应式布局]]"
- "[[10_登录消息提示、回车键监听、按钮加载 Loading]]"
- "[[15_重复登录问题优化、密码框可显示密码]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/8_AdminTagList：路由同步（1）.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-25 20:56:54
修改时间: 2025-05-27 00:55:22
---

上小节中，我们已经完成了标签导航栏组件的样式布局。但是，当我们点击左侧菜单栏，标签导航栏是没有跟随变动的，比如点击文章管理菜单，跳转页面后，导航栏没有同步到对应 `tab` 标签。后续小节中，我们就来添加路由同步功能。
## 1 重新定义标签数组

先来回顾一下上小节中 `<script>` 的代码，如下图所示：

![](https://img.quanxiaoha.com/quanxiaoha/169448921023551)

这些代码是从 Element Plus 官网复制组件时，一同复制过来的，这里解释一下它们都是干啥的：
- `tabIndex` : 当前被选中的 `tab` 下标；
- `editableTabsValue` : 通过 `ref` 响应式的绑定到了 `<el-tabs>` 组件上，告诉 `<el-tabs>` 组件哪个子标签栏需要被设置为选中状态;
    ![](https://img.quanxiaoha.com/quanxiaoha/169448941280081)
- `editableTags` : 数组，用来定义 `tab` 标签栏内容；

为了实现路由同步功能，需要重新定义一下 `editableTags` 数组内对象的属性，将其变更为下面的代码：
```html
<el-tab-pane v-for="item in tabList" :key="item.path" :label="item.title" :name="item.path">
</el-tab-pane>

const tabList = ref([
    {
        title: '仪表盘',
        path: "/admin/index"
    },
    {
        title: '文章管理',
        path: "/admin/article/list"
    },
    {
        title: '分类管理',
        path: "/admin/category/list"
    },
    {
        title: '标签管理',
        path: "/admin/tag/list"
    },
    {
        title: '博客设置',
        path: "/admin/blog/setting"
    }
])
```

上述代码中，首先我们将数组的命名改成了 `tabList` , 让其命名更具语义化，利于代码维护。然后将数组内对象的只设定了两个属性字段，分别是：
- `title` : 标签栏标题；
- `path` : 对应的路由地址；

对应的，`<el-tab-pane>` 组件内的 `v-for` 循环对象名也要改一下，`key` 和 `name` 属性改为了 `item.path`, 使用的路由地址。

## 2 路由同步

目前虽然我们为每个子标签栏都定义了对应的 `path` 路由，但是还没有办法根据页面当前路由，动态设置被选中状态。继续编辑 `AdminTagList` 组件，完善代码如下：

```html
<el-tabs v-model="activeTab" type="card" class="demo-tabs" closable @tab-remove="removeTab" style="min-width: 10px;">
            // 省略...
</el-tabs>

<script setup>
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

// 当前被选中的 tab
const activeTab = ref(route.path)
// 导航栏 tab 数组
const tabList = ref([
    {
        title: '仪表盘',
        path: "/admin/index"
    },
    {
        title: '文章管理',
        path: "/admin/article/list"
    },
    {
        title: '分类管理',
        path: "/admin/category/list"
    },
    {
        title: '标签管理',
        path: "/admin/tag/list"
    },
    {
        title: '博客设置',
        path: "/admin/blog/setting"
    }
])
// 省略...
</script>
```

上述代码中，我们将 `tabIndex` 、`editableTabsValue` 变量删除掉了，另外定义了一个更具语义化的 `activeTab` , 表示当前被选中的 `tab`, 值为当前路由路径。并响应式的绑定到了 `<el-tabs>` 上。

完善上述代码后，当我们点击左侧栏菜单后，_再刷新页面_，就会动态选中当前路由对应的标签栏啦~

## 3 后台首页标签栏设置不可删除

通常来说，后台标签导航栏中，首页对应的 `tab` 是不应该能够被删除的，不然都能给用户删除了，标签导航栏内就一片空白了，很是奇怪。在 `weblog` 这个项目中，后台首页就是仪表盘，让我们来看看要如何设置。

通过 Element Plus 官网 `Tabs` 标签页：[https://element-plus.org/zh-CN/component/tabs.html#tab-pane-%E5%B1%9E%E6%80%A7](https://element-plus.org/zh-CN/component/tabs.html#tab-pane-%E5%B1%9E%E6%80%A7) 文档，可以看到针对子标签，提供了 `closable` 属性来设置标签是否可关闭：

![](https://img.quanxiaoha.com/quanxiaoha/169449133286630)

有了 `closable` 就好办了，编辑 `AdminTagList` 组件中的 `<el-tabs>` , 修改代码如下：

```html
// 省略...
<el-tabs v-model="activeTab" type="card" class="demo-tabs" @tab-remove="removeTab" style="min-width: 10px;">
            <el-tab-pane v-for="item in tabList" :key="item.path" :label="item.title" :name="item.path" :closable="item.path != '/admin/index'">
            </el-tab-pane>
</el-tabs>
// 省略...
```

上述代码中，首先将 `<el-tabs>` 组件中的 `closable` 属性去掉了，因为我们要在 `<el-tab-pane>` 标签中做更细粒度的控制。通过添加 `:closable="item.path != '/admin/index'"` , 当前标签的路由路径不等于后台首页 `/admin/index` 的时候，则判定为可关闭，否则不可关闭。

## 4 效果图如下

此时，当我们将鼠标移动到仪表盘标签栏时，会发现它始终处于不可删除状态了。

![](https://img.quanxiaoha.com/quanxiaoha/169449208046592)

## 5 结语

本小节中，我们主要对标签导航栏做了更进一步的完善，当刷新页面时，可以根据当前路由动态设置被选中的 `tab` 栏，另外，我们还设置了首页 `tab` 栏为无法关闭状态。