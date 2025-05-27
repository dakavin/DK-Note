---
文章标题: "[[14_Transition 组件添加全局过渡动画]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了如何使用Vue官方的`<Transition>`组件为路由页面添加全局淡入淡出过渡动画。详细讲解了`<Transition>`的基本用法、6种CSS过渡类名，并通过命名和自定义CSS实现平滑动画效果。文章还强调了组件内容必须为单根元素的注意事项。
文章标签:
- "Vue.js"
- "Transition"
- "CSS过渡"
- "路由动画"
- "前端动画"
- "用户体验"
- "单页应用"
相关文章:
- "[[16_Vue]]"
- "[[10_登录消息提示、回车键监听、按钮加载 Loading]]"
- "[[2_登录页加点盐：通过 Animate.css 添加入场动画]]"
- "[[1_WebStrom优化]]"
- "[[14_Vue3 路由机制router]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/4_搭建管理后台骨架/14_Transition 组件添加全局过渡动画.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-25 20:56:56
修改时间: 2024-04-27 00:02:58
---


本小节中，我们将使用 `vue` 官方提供的 `<Transition>` 动画组件，当内容页跳转时，添加淡入淡出的动画效果。

## 1 最终效果

完成本小节的功能开发，最终效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169469142822428)

## 2 Transition 组件介绍

`<Transition>` 组件是官方提供的内置动画组件，这意味着它在任意的组件中都可以被使用，并且无需注册。通过它，你能在一个元素或组件进入和离开 DOM 时应用动画。

## 3 开始使用

接下来，让我们来看看要如何使用它 ！编辑 `admin.vue` 布局文件，内容如下：

```
<!-- 主内容（根据路由动态展示不同页面） -->  
<router-view v-slot="{ Component }">  
    <Transition>  
        <!-- max 指定最多缓存 10 个组件 -->  
        <KeepAlive :max="10">  
            <component :is="Component"></component>  
        </KeepAlive>  
    </Transition>  
</router-view>
```

可以看到，使用起来非常简单，通过 `<Transition>` 组件将需要添加动画的内容包裹起来即可。再次回到页面查看效果，你会发现默认的动画，有点跳脱，不够平滑。

## 4 6 个应用于进入与离开过渡效果的 CSS

`<Transition>` 组件提供了 6 个 `css` 样式来用于设置组件进入与离开过渡效果：

![](https://img.quanxiaoha.com/quanxiaoha/169469180678615)

解释一下：

1. `v-enter-from`：进入动画的起始状态。在元素插入之前添加，在元素插入完成后的下一帧移除。
2. `v-enter-active`：进入动画的生效状态。应用于整个进入动画阶段。在元素被插入之前添加，在过渡或动画完成之后移除。这个 class 可以被用来定义进入动画的持续时间、延迟与速度曲线类型。
3. `v-enter-to`：进入动画的结束状态。在元素插入完成后的下一帧被添加 (也就是 `v-enter-from` 被移除的同时)，在过渡或动画完成之后移除。
4. `v-leave-from`：离开动画的起始状态。在离开过渡效果被触发时立即添加，在一帧后被移除。
5. `v-leave-active`：离开动画的生效状态。应用于整个离开动画阶段。在离开过渡效果被触发时立即添加，在过渡或动画完成之后移除。这个 class 可以被用来定义离开动画的持续时间、延迟与速度曲线类型。
6. `v-leave-to`：离开动画的结束状态。在一个离开动画被触发后的下一帧被添加 (也就是 `v-leave-from` 被移除的同时)，在过渡或动画完成之后移除。

以上内容，小伙伴们可查看[官网](https://cn.vuejs.org/guide/built-ins/transition.html#css-based-transitions) 来了解详情。

## 5 为过渡效果命名

默认情况下，6 个 `css` 样式都是以 `v` 开头的，但是我们也可以自定义这个前缀名，通过给 `<Transition>` 组件传一个 `name` prop 来声明一个过渡效果名，比如像下面这样：

```
<Transition name="fade">
  ...
</Transition>
```

那么，它被应用的动画 `class` 将会是 `fade-enter-active` 而不是 `v-enter-active`, 格式就像下面这样：

```
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

## 6 自定义淡入淡出动画

有了这 6 个 `css` 样式，我们就可以来实现淡出淡出的效果了，添加 `css` 样式代码如下：

```
<Transition name="fade">
...
</Transition>

<style scoped>
// 省略...

/* 内容区域过渡动画：淡入淡出效果 */
/* 刚开始进入时 */
.fade-enter-from {
    /* 透明度 */
    opacity: 0;
}

/* 刚开始结束 */
.fade-enter-to {
    opacity: 1;
}

/* 刚开始离开 */
.fade-leave-from {
  opacity: 1;
}

/* 离开已结束 */
.fade-leave-to {
  opacity: 0;
}

/* 离开进行中 */
.fade-leave-active {
    transition: all 0.3s;
}

/* 进入进行中 */
.fade-enter-active {
    transition: all 0.3s;
    transition-delay: 0.3s;
}
</style>
```

上述代码中，`.fade-enter-from` 指定了组件刚进入时，透明度为 0， `.fade-enter-to` 指定了进入结束时，透明度为 1。`.fade-leave-from` 指定了组件刚离开时透明度为 1, `.fade-leave-to` 指定了组件完全离开时透明度为 0。另外，还指定了进入和离开时，开启动画，持续时间 `0.3s`，并且为进入设置了一个动画延时，也是 `0.3s`， 也就是说，只有当离开动画完全结束后，才开启进入动画，防止动画衔接不协调。

## 7 注意点

官网有提示下面这句话：

> ⚠️ 注意： `<Transition>` **仅支持单个元素或组件作为其插槽内容**。如果内容是一个组件，这个组件必须仅有一个根元素。

也就是说，我们跳转页面中的布局代码，必须只能是以一个 `<div>` 作为根元素，就像下面这样：

![](https://img.quanxiaoha.com/quanxiaoha/169469816669699)

而不能像下面这样，平级情况下，存在两个 `<div>` 标签：

![](https://img.quanxiaoha.com/quanxiaoha/169469828600544)

如果你真的这么干了，你会在控制台看到下面的警告信息，提示你 `<Transition>` 下面只能有一层根节点，多层节点导致动画不生效：

![](https://img.quanxiaoha.com/quanxiaoha/169469824152121)

## 8 结语

本小节中，我们主要通过 `Vue` 官方提供的 `<Transition>` 组件，实现了页面间跳转时，有了淡入淡出的动画效果，让交互看起来更炫酷一点。