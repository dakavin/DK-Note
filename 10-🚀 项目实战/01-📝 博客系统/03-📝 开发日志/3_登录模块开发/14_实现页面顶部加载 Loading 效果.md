
本小节中，将带着大家实现一个页面顶部加载 Loading 效果，让交互看起来更炫一点！

## 1 效果图如下

![image.png|300|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71c94c8bfba4a7435cf9ff29d8b31465.png)
## 2 安装 nprogress

实际上有现成的库可以使用，访问 `npm` 官网：[https://www.npmjs.com/](https://www.npmjs.com/) ，在搜索框内搜索关键词 _nprogress_，或者直接访问贴出来的地址：[https://www.npmjs.com/package/nprogress](https://www.npmjs.com/package/nprogress) ：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3856965a5229ec6759f4551e03665663.png)


复制安装命令，在 VSCode 命令行窗口中执行它：

```shell
npm i nprogress
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2ccd122c092794d3e3c2dcea2d973c43.png)


## 3 封装工具类

安装成功后，我们来封装两个 `nprogress` 的方法，分别是显示、隐藏加载 Loading 的工具方法。修改 `/composables/util.js` , 添加代码：

```js
import nprogress from "nprogress"

// 显示页面加载 Loading
export function showPageLoading() {
    nprogress.start()
}

// 隐藏页面加载 Loading
export function hidePageLoading() {
    nprogress.done()
}
```

## 4 如何使用？

### 4.1 引入 css 文件

因为在安装好了 `nprogress` 后，其 `css` 文件也一并下载下来了，可以在 `/node_modules/nprogress` 中找到它，直接引用就行：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d586473a563a3dd93862e54833360f7f.png)


我们需要在 `main.js` 文件中引入该库的 `css` 文件：

```
import 'nprogress/nprogress.css'
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80534a2017b55f278637e677bfa87f32.png)


### 4.2 显示加载 Loading

_何时显示加载 Loading 呢？_ 应该是在每次请求之前来做这个工作，那么，全局路由前置守卫又要上场了，修改 `permission.js` 代码如下：

```js
import { showPageLoading } from '@/composables/util'

// 全局路由前置守卫
router.beforeEach((to, from, next) => {
    console.log('==> 全局路由前置守卫')

    // 展示页面加载 Loading
    showPageLoading()

    // 省略
})
```

再次加载页面，你就会看到加载 Loading 效果啦：

![image.png|300|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71c94c8bfba4a7435cf9ff29d8b31465.png)


### 4.3 自定义加载 Loading 的颜色

如何你觉得默认的 Loading 加载的颜色，和页面整体的色调不大协调的话，可以通过重写 `nprogress` 的样式类来自定义。页面右键 _检查_，即可审查页面元素，追踪到该样式类，如下图所示：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/481d856a975ea0dc04a3ab81aca4851d.png)


可以看到是通过 `#nprogress .bar` 选择器，通过 `background` 属性，来指定加载 Loading 颜色的，我们只要重写这个属性值即可。

编辑 `App.vue` 文件，添加如下代码：

```css
<style>
/* 自定义顶部加载 Loading 颜色 */
#nprogress .bar {
   background: #409eff!important;
}
</style>
```

这里，我们让加载 Loading 的颜色和页面中的登录按钮的颜色保持一致，使用的 `#409eff`。

### 4.4 隐藏加载 Loading

除了请求开始显示外，还需要请求结束的时候隐藏起来，才是一个完整的逻辑。如何实现呢? 很显然在路由的后置钩子中，来做个这个工作最合适，添加代码如下：

```js
import { showPageLoading, hidePageLoading } from '@/composables/util'
// 省略...

// 全局路由后置守卫
router.afterEach((to, from) => {
    // 省略...

    // 隐藏页面加载 Loading
    hidePageLoading()
})
```


## 6 结语

至此，本小节的 UI 优化，实现一个页面顶部加载 Loading 效果就完成了，赶快上手试试吧~