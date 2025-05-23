
## 1 什么是 Tailwind CSS ？

Tailwind CSS 是一个高度可定制的、实用工具优先的 **CSS 框架**，它使你能够通过组合小型、单一用途的类来构建现代网站界面，而无需写任何 CSS。

## 2 为什么选择 Tailwind CSS？

- 1、 **高度可定制**：Tailwind CSS 提供了一整套预设样式，但所有这些都是完全可配置的。
- 2、 **实用工具优先**：它允许你通过在 HTML 中组合类，而不是自定义 CSS，来迅速构建响应式页面。
- 3、 **优化生产环境**：Tailwind CSS 在生产环境中会自动移除未使用的样式，这有助于保持 CSS 文件大小最小。

## 3 开始安装

执行如下命令，开始安装 Tailwind CSS :
```shell
npm install -D tailwindcss postcss autoprefixer
```

此命令会在你的项目中安装三个依赖，它们分别是：

1、`tailwindcss`：Tailwind CSS 框架本身
2、`postcss`：一个用于转换 CSS 的工具
3、`autoprefixer`：一个 PostCSS 插件，用于自动添加浏览器供应商前缀到 CSS 规则中，确保跨浏览器的兼容性。

然后，再执行如下命令：
```shell
npx tailwindcss init -p
```

此命令用于生成 `tailwind.config.js` 和 `postcss.config.js` 配置文件
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eee6a51fe3993975b385758eb1e50295.png)

## 4 配置模板路径

在 `tailwind.config.js` 文件中添加所有模板文件的路径:
```java
/** @type {import('tailwindcss').Config} */  
export default {  
  content: [  
      "./index/html",  
      "./src/**/*.{vue,js,ts,jsx,tsx}",  
  ],  
  theme: {  
    extend: {},  
  },  
  plugins: [],  
}
```

## 5 将 Tailwind 指令添加到 CSS 文件中

修改 `main.js` 文件，引入 `main.css` :


然后编辑 `main.css` 文件，_清空_ 里面初始化项目时自动生成的 `css` 代码，添加如下代码：
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## 6 重新构建工程

执行 `npm run dev` 命令，重新构建工程。

## 7 在项目中使用 Tailwind CSS

编辑 `/frontend/index.vue` 文件，内容如下：

```
<template>
    <div class="bg-green-300 inline">绿色</div>
    <div class="bg-yellow-300 ml-2 inline">黄色</div>
    <div class="bg-blue-300 ml-2 inline">蓝色</div>
</template>
```

我们在 `div` 标签上添加了一些 Tailwind CSS 库中的样式类：

- `bg-green-300` : 绿色背景；
- `bg-yellow-300` : 黄色背景；
- `bg-blue-300` : 蓝色背景；
- `inline` : 相当于 `display: inline;`
- `ml-2` : 相当于 `margin-left`;

效果如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92c1efe64679d907ca9c794d30210634.png)

### 7.1 响应式设计

通过在 Tailwind 类名前添加前缀，你可以轻松地应用不同的样式于不同屏幕尺寸：

```
<div class="text-xs sm:text-lg md:text-xl lg:text-2xl">
  响应式字体
</div>
```

更多样式类，小伙伴们可查询官网：[https://tailwindcss.com/docs/installation](https://tailwindcss.com/docs/installation) ，应有尽有。

## 结语

本小节中，带着大家在 `blog` 前端项目中安装了 Tailwind CSS 库，它在后面设计页面时，会被频繁被用到。这里只是简单介绍下，等涉及到具体功能开发时，我们再慢慢体会。