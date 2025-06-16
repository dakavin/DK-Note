---
文章标题: "[[4_Vite 配置路径别名：方便的引用文件]]"
文章作者: Dakkk
文章概要: |
  文章介绍了Vite中`alias`配置的作用。它通过定义路径别名（如`@`指向`src`），将复杂的相对文件引用路径简化为简洁的别名路径，极大提高代码可读性、可维护性，对于大型前端项目管理尤为重要。
文章标签:
  - Vite
  - alias
  - 路径别名
  - 配置
  - 文件引用
  - 前端开发
  - 代码优化
  - 项目结构
相关文章:
  - "[[../../../../08-🛠️ 开发工具/01-💻 IDE工具/03-🌐 WebStorm/1_WebStrom优化]]"
  - "[[16_Vue3 状态管理pinia]]"
  - "[[17_Element-plus组件库]]"
  - "[[0_vscode的安装与使用]]"
  - "[[11_Nodejs + npm]]"
文章分类: 🚀 项目实战
文章路径: 10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/2_前端工程搭建/4_Vite 配置路径别名：方便的引用文件.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-18 14:22:10
修改时间: 2024-04-18 20:58:28
---


在本小节中，我们来说说 Vite 的一个重要配置选项：`alias`。
## 1 什么是 alias？

`alias` 是一个用于定义路径别名的配置选项。当你的项目结构变得复杂时，路径别名可以帮助你简化 `import` 或 `require` 语句中的路径，让代码更干净、更可维护。

## 2 为什么需要 Alias？

上小节中，我们在 `/router/index.js` 文件中引入了 `index.vue` 组件，格式是下面这样的：
```
import Index from '../pages/frontend/index.vue'
```

通过 `..` 来指定上一级目录，然后再定位具体路径下。考虑一个大型项目中，我们经常需要这样的引用，当文件层级很深，那么代码可能会像下面这样：
```
import Button from '../../../components/Button.vue';
```

这种相对路径不易于管理和阅读。使用 `alias`，我们可以将路径简化为：
```
import Button from '@/components/Button.vue';
```

这样一来，不仅路径更短、更明确，而且移动文件时无需改动 `import` 路径。

## 3 如何在 Vite 中配置 Alias

以下是如何在 Vite 项目中设置 `alias` 的步骤:

---

**修改 Vite 配置**

在项目的根目录中，找到 Vite 的配置文件 `vite.config.js`，编辑它，添加`alias` 配置，代码如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56d0f928d01ec7b5e289c5c4b4087efa.png)

上述代码中，我们定义一个别名 `@`，该别名对应于当前 JavaScript 模块文件所在目录下的 `src` 目录的绝对文件路径。

## 4 如何使用别名 Alias？

定义好了别名以后，我们将之前的 `/router/index.js` 文件中的路径引用改进一下，如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24523d2c85007d7d41b72207e82e4b12.png)

另外，`main.js` 中的路径引用同样也可以改进一波：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/65c2e2f81a5cc0a3bedf73e113f60fcc.png)

## 5 小结

`alias` 是一个非常有用的 Vite 配置选项，它可以让你的导入语句更加清晰和易于管理。在大型项目中，使用 `alias` 可以极大地提高代码的可维护性。
