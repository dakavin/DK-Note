---
文章标题: "[[13_KeepAlive 缓存组件]]" 
文章作者: Dakkk
文章概要: |
  文章详细介绍了Vue 3的`<KeepAlive>`组件，阐述其通过缓存组件实例来显著提升多标签页应用中页面切换性能和响应速度的机制。文章演示了`<KeepAlive>`与`<router-view>`结合使用的具体方法，并提及了`max`属性限制缓存数量。
文章标签:
- "Vue 3"
- "KeepAlive"
- "组件缓存"
- "性能优化"
- "路由"
- "router-view"
- "前端开发"
- "响应速度"
相关文章:
- "[[17_Element-plus组件库]]"
- "[[1_Vue 3环境安装 和 blog前端项目搭建]]"
- "[[3_整合 vue-router 路由管理器]]"
- "[[5_AdminMenu：点击收缩、展开功能实现]]"
- "[[7_整合Element Plus 组件库]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/13_KeepAlive 缓存组件.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-25 20:56:56
修改时间: 2025-05-27 00:55:33
---


本小节中，我们将引入 `vue` 官方提供的 `<KeepAlive>` 缓存组件，达到当用户打开多个标签页时，重新回到之前已打开的标签页时，无需再次加载，而是使用缓存，提高页面切换性能和响应速度。

## 1 什么是 `<KeepAlive>` 组件？

`VUE` 官方对其的解释是：_缓存包裹在其中的动态切换组件，_ 具体可查阅官方文档：[https://cn.vuejs.org/api/built-in-components.html#keepalive](https://cn.vuejs.org/api/built-in-components.html#keepalive "https://cn.vuejs.org/api/built-in-components.html#keepalive") 。

什么意思呢？ 就是说组件被`<KeepAlive>` 包裹时，会缓存不活跃的组件实例，而不是销毁它们。

> ⚠️ 注意：任何时候都只能有一个活跃组件实例作为 `<KeepAlive>` 的直接子节点。

## 2 基本使用示例

```
<KeepAlive>
  <component :is="view"></component>
</KeepAlive>
```

更多使用示例，请查阅官网。

## 3 整合`<KeepAlive>` 组件

接下来，我们就来为内容区域包裹一层 `<KeepAlive>` 组件，让其能够缓存其中已经打开的组件，而不是每次跳转都创建新的组件，发送新的请求给后端：

![](https://img.quanxiaoha.com/quanxiaoha/169467592866261)

编辑 `admin.vue` 布局文件，为了能够像上面基础示例那样，以包裹 `<component>` 的格式来使用它，我们需要对 `<router-view>` 进行解构，代码如下：

```html
<!-- 主内容（根据路由动态展示不同页面） -->
<router-view v-slot="{ Component }">
	<!-- max 指定最多缓存 10 个组件 -->
    <KeepAlive :max="10">
   		 <component :is="Component"></component>
    </KeepAlive>
</router-view>
```

解释一下上面的代码：
- `v-slot="{ Component }"` 是 Vue 3 中的插槽语法。它表示为`<router-view>`组件定义了一个插槽，并将当前路由的组件赋值给了 `Component` 变量。这意味着在这个插槽内，你可以通过 `Component` 访问当前路由匹配的组件。
- `<KeepAlive :max="10">` 是 Vue 3 中的 `Keep-Alive` 组件。它用于缓存动态加载的组件，以提高页面切换时的性能和响应速度。`:max="10"` 表示最多可以缓存 10 个组件实例。
- `<component :is="Component"></component>` 是一个动态组件，它的 `:is` 属性设置为 `Component`，也就是当前路由匹配的组件。这将导致在页面切换时，路由视图中的组件会被动态加载和替换。

此时，再回到页面中，点击各个菜单测试一波，不出意外，页面依然能够正常跳转，同时 `<KeepAlive>` 组件还帮助我们提高了页面切换性能和响应速度。

## 4 结语

本小节中，我们主要为 `<router-view>` 主内容区域添加了 `<KeepAlive>` 组件, 它可以使页面的切换性能和响应速度得到显著提高。