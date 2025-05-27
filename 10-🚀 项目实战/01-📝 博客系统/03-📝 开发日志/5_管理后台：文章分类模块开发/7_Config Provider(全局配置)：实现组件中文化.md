---
文章标题: "[[7_Config Provider(全局配置)：实现组件中文化]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了如何利用 Element Plus 的 `Config Provider` 全局配置功能，解决组件默认英文显示问题。通过引入中文语言包并将其绑定至 `el-config-provider`，可快速实现 Element Plus 组件的汉化，提升用户体验。
文章标签:
- "Element Plus"
- "Config Provider"
- "组件国际化"
- "组件本地化"
- "Vue.js"
- "语言配置"
- "前端开发"
相关文章:
- "[[1_登录页设计：支持响应式布局]]"
- "[[11_AdminTagList：关闭其他、全部标签页]]"
- "[[19_📕用户修改密码、退出功能开发]]"
- "[[3_AdminMenu：样式布局]]"
- "[[6_分类管理页面：样式布局]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/5_管理后台：文章分类模块开发/7_Config Provider(全局配置)：实现组件中文化.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-04-27 13:58:54
修改时间: 2024-04-27 22:53:55
---

在说 `Config Provider` 全局配置之前，我们先来看看上小节完成的分类页面中，已知的一些问题。

![](https://img.quanxiaoha.com/quanxiaoha/169528628014137)

![](https://img.quanxiaoha.com/quanxiaoha/169528636270125)

细心的小伙伴应该都发现了，_Element Plus 组件默认是英文的，看着非常不爽，如何将其变成中文呢？_

## 1 Config Provider 全局配置

Element Plus 提供了 `Config Provider` ，它可以用来提供全局的配置选项，让你的配置能够在全局都能够被访问到。这其中，就包括语言的配置。

> 💡 TIP : 更多详情信息，可访问官网：[https://element-plus.org/zh-CN/component/config-provider.html](https://element-plus.org/zh-CN/component/config-provider.html)

## 2 实现中文配置

接下来，就带着大家，用 Config Provider 实现组件支持中文。编辑 `App.vue` 文件，修改代码如下：

```html
<template>
   <!-- 设置语言为中文 -->
   <el-config-provider :locale="locale">
      <router-view></router-view>
   </el-config-provider>
</template>

<script setup>
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'
const locale = zhCn
</script>

// 省略...
```

我们将 `<router-view>` 通过 `<el-config-provider>` 包裹了起来，通过属性值 `:locale` 将其和变量 `locale` 绑定在了一起，引用的则是中文汉化文件 `/locale/zh-cn.mjs` 。

## 3 看看效果

完成上述简单的配置后，我们就实现了组件的汉化啦。非常简单，再次回到页面中，可以看到相关组件都显示的是中文了：

![](https://img.quanxiaoha.com/quanxiaoha/169528699424637)

![](https://img.quanxiaoha.com/quanxiaoha/169528704470544)

## 5 结语

本小节涉及代码量不多，主要针对 Element Plus 组件默认是英文的问题，通过 Config Provider 全局配置对语言设置为中文，让后台管理使用上更加人性化。