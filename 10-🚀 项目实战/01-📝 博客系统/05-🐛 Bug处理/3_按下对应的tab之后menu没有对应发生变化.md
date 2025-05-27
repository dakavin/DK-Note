---
文章标题: "[[3_按下对应的tab之后menu没有对应发生变化]]" 
文章作者: Dakkk
文章概要: |
  本文展示了Vue 3中，如何利用`vue-router`的`useRoute`和`watch`功能，实时监听路由路径变化，并相应更新菜单的选中状态（`defaultActive`），以确保菜单显示与当前页面同步。
文章标签:
- "Vue 3"
- "Vue Router"
- "路由监听"
- "菜单选中"
- "响应式"
- "useRoute"
- "watch"
相关文章:
- "[[13_Vue3语法(视图渲染技术)]]"
- "[[3_整合 vue-router 路由管理器]]"
- "[[8_Vue 整合 Axios 实现登录功能（解决跨域问题）]]"
- "[[12_Vue3 + Vite]]"
- "[[14_Vue3 路由机制router]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/05-🐛 Bug处理/3_按下对应的tab之后menu没有对应发生变化.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-16 23:38:20
修改时间: 2024-04-16 23:39:47
---

```js
import {useRoute, useRouter} from "vue-router"  
import {ref,watch } from 'vue'  

// 用于获取关于当前路由的信息，如当前的路径、查询参数等  
const route = useRoute()  
// 路由对象，使用它可以实现页面的跳转  
const router = useRouter()

// 根据路由地址判断那个菜单被选中，从而保持常亮  
const defaultActive = ref(route.path)  
  
// 监听路由变化，避免按下了tab，但是菜单栏没有变化  
watch(()=>route.path,(newPath)=>{  
    defaultActive.value = newPath  
})
```