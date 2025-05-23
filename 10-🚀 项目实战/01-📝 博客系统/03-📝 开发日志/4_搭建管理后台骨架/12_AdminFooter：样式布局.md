前面几小节中，我们已经将标签导航栏的功能搞定了。目前，后台管理页面骨架还差最后一块拼图：_Footer 页脚_。因为页脚只涉及样式布局，并没有任何的 `js` 功能代码，所以非常简单。
## 1 最终效果

看看最终效果，如下图所示，可以看到只是一些版权信息的展示：

![](https://img.quanxiaoha.com/quanxiaoha/169467478468696)

## 2 添加页脚样式布局

编辑 `AdminFooter.vue` 组件, 添加样式布局，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169465807609039)

```html
<template>
    <div class="bg-white py-5 flex items-center justify-center text-sm text-gray-500 shadow-none">
        <!-- Copyright 版权信息 -->
        Copyright © 2023. All rights reserved. Provided by&nbsp; <a class="underline" href="https://www.dakkk.top" target="_blank">dakkk</a>
    </div>
</template>
```

上述代码中，通过 `Tailwindcss` 样式库，可以很方便的来设置自己想要的样式布局。我们为最顶层的 `<div>` 容器设置了 `bg-white` 白色背景，`py-5` 纵向轴间隔为 5, `flex items-center justify-center` 让内容水平垂直居中，`text-sm` 文字小号，文字颜色为 `text-gray-500` , 没有阴影 `shadow-none` 。另外，我们通过 `<a>` 标签将 _dakkk_ 包裹起来了，让他能够跳转链接，并给它了一个 `underline` 下划线样式。

## 3 Element Plus 间隔问题

完成上述工作后，回到页面看下效果，你会发现页脚两边有间距：

![](https://img.quanxiaoha.com/quanxiaoha/169465857809629)

这个问题和前面小节中设计 `AdminHeader` 头的间隔问题是一样的。通过 `F12` 审查元素，会发现是 `.el-footer` 这个样式导致的：

![](https://img.quanxiaoha.com/quanxiaoha/169465882972932)

编辑 `admin.vue` 布局文件，重写该样式, 将 `padding` 设置为 0 即可：

```
<style scoped>
// 省略...

.el-footer {
    padding: 0!important;
}
</style>
```
## 4 结语

到这里，本小节的 `AdminFooter` 页脚组件的样式布局就完成啦，再次页面，你就会看到文章开头贴出的最终效果啦~ 是不是非常简单呢？