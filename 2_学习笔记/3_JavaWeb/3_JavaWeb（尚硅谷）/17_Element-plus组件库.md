## 1 介绍

> Element Plus 是一套基于 Vue 3 的开源 UI 组件库，是由饿了么前端团队开发的升级版本 Element UI。Element Plus 提供了丰富的 UI 组件、易于使用的 API 接口和灵活的主题定制功能，可以帮助开发者快速构建高质量的 Web 应用程序。

- Element Plus 支持按需加载，且不依赖于任何第三方 CSS 库，它可以轻松地集成到任何 Vue.js 项目中。Element Plus 的文档十分清晰，提供了各种组件的使用方法和示例代码，方便开发者快速上手。
    
- Element Plus 目前已经推出了大量的常用 UI 组件，如按钮、表单、表格、对话框、选项卡等，此外还提供了一些高级组件，如日期选择器、时间选择器、级联选择器、滑块、颜色选择器等。这些组件具有一致的设计和可靠的代码质量，可以为开发者提供稳定的使用体验。
    
- 与 Element UI 相比，Element Plus 采用了现代化的技术架构和更加先进的设计理念，同时具备更好的性能和更好的兼容性。Element Plus 的更新迭代也更加频繁，可以为开发者提供更好的使用体验和更多的功能特性。
    
- Element Plus 可以在支持 [ES2018](https://caniuse.com/?feats=mdn-javascript_builtins_regexp_dotall,mdn-javascript_builtins_regexp_lookbehind_assertion,mdn-javascript_builtins_regexp_named_capture_groups,mdn-javascript_builtins_regexp_property_escapes,mdn-javascript_builtins_symbol_asynciterator,mdn-javascript_functions_method_definitions_async_generator_methods,mdn-javascript_grammar_template_literals_template_literal_revision,mdn-javascript_operators_destructuring_rest_in_objects,mdn-javascript_operators_spread_spread_in_destructuring,promise-finally "ES2018") 和 [ResizeObserver](https://caniuse.com/resizeobserver "ResizeObserver") 的浏览器上运行。 如果您确实需要支持旧版本的浏览器，请自行添加 [Babel](https://babeljs.io/ "Babel") 和相应的 Polyfill
    
- 官网[https://element-plus.gitee.io/zh-CN/](https://element-plus.gitee.io/zh-CN/)
    
- 由于 Vue 3 不再支持 IE11，Element Plus 也不再支持 IE 浏览器。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2122da4e4bf534cd6438e2599f8f0acb.png)
## 2 入门案例

>  1 准备vite项目

```shell
npm create vite  // 注意选择 vue+typeScript
npm install 
npm install vue-router@4 --save
npm install pinia
npm install axios
```

- 注意选择TypeScript

>  2 安装element-plus

```shell
npm install element-plus
```

> 3 完整引入element-plus

+ main.js

```javascript
import { createApp } from 'vue'
//导入element-plus相关内容
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)
app.mount('#app')
```

>  4 入门案例

+ App.vue

```html
<script setup>
  import { ref } from 'vue'

  const value = ref(true)
</script>

<template>
  <div>
    <!-- 直接使用element-plus组件即可 -->
    <el-button>按钮</el-button>
    <br>
    <el-switch
      v-model="value"
      size="large"
      active-text="Open"
      inactive-text="Close"
    />
    <br />
    <el-switch v-model="value" active-text="Open" inactive-text="Close" />
    <br />
    <el-switch
      v-model="value"
      size="small"
      active-text="Open"
      inactive-text="Close"
    />
  </div>
</template>

<style scoped>

</style>

```

> 5 启动测试

```shell
npm run dev
```

## 3 常用组件

[Overview 组件总览 | Element Plus (element-plus.org)](https://element-plus.org/zh-CN/component/overview.html)