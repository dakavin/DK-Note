
上小节中，我们已经在项目中整合好了全局状态管理 Pinia 。本小节中，我们将通过它，来实现左边菜单栏点击收缩、展开的功能。
## 1 效果图如下

完成本小节的功能开发，应该达到的效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169430600284952)

## 2 开始

### 2.1 添加 Icon 的点击事件

首先，我们需要为 `AdminHeader` 组件中的收缩 `Icon` 添加点击事件，代码如下：

```html
<!-- 左边栏收缩、展开 -->  
<div @click="handleMenuWidth"  
     class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 hover:bg-gray-200">  
  <el-icon>  
    <Fold/>  
  </el-icon>  
</div>
        
<script setup>  
import {useMenuStore} from '@/stores/menu.js'  
  
// 引入pinia中的菜单store  
const menuStore = useMenuStore()  
// icon点击事件  
const handleMenuWidth = () => {  
  // 动态设置菜单的宽度大小  
  menuStore.handleMenuWidth()  
}  
</script>
```

上述代码中，我们通过 `@click` 为 `Icon` 添加了一个 `handleMenuWidth()` 监听事件。另外，我们还引入了上小节中定义好的 `menu` 菜单的 `store` , 然后，在该事件方法中，调用了 `menuStore.handleMenuWidth()` 方法，用来动态的设置之前定义好的、全局的 `menuWidth` 变量，也就是菜单的宽度。

### 2.2 如何查看 store 中的值呢？

在页面中，通过点击收缩 `Icon` 后，如何查看 `store` 中的变量值呢？大家可以去 `chrome` 应用商店下载安装一个 `Vue.js devtools` 插件，它可以帮助我们更方便的开发与调试 `Vue` 应用，安装地址：[https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd/related?hl=zh-CN](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd/related?hl=zh-CN) ：

![](https://img.quanxiaoha.com/quanxiaoha/169430683571553)

安装成功后，按 `F12` 就可以看到，就可以看到浏览器中多了一个 `Vue` 的选项卡，点击进去，我们就看到 `store` 中的`menuWidth` 的全局状态值，通过点击收缩图标，你会发现它是动态变化的，这说明通过 `handleMenuWidth()` 方法设置菜单宽度是 OK 的：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13fc77ae2d3c7d9a814277be853fca80.png)


### 2.3 动态设置菜单栏的宽度

光动态设置全局状态中的 `menuWidth` 值还不行，我们还需要拿到它，并动态将其应用到左侧菜单的 `css` 样式中才行。编辑 `AdminMenu.vue` 组件，添加如下代码：

```html
<template>
<div class="bg-slate-800 h-screen text-white menu-container" :style="{ width: menuStore.menuWidth }">
	<!-- 顶部 Logo, 指定高度为 64px, 和右边的 Header 头保持一样高 -->
	<div class="flex items-center justify-center h-[64px]">
			// 省略...
	</div>

	// 省略...
</div></template>

<script setup>
// 省略...
import { useMenuStore } from '@/stores/menu'
// 引入 useMenuStore
const menuStore = useMenuStore()
// 省略...
</script>
```

上述代码中，首先我们引入了 `useMenuStore` , 有了它就可以拿到全局状态中的菜单宽度了。然后，为菜单最顶层的 `<div>` 容器绑定了动态样式 `:style="{ width: menuStore.menuWidth }"` , 这样就可以响应式的设置菜单的 `CSS` 宽度值了。

### 2.4 Element Plus 菜单组件设置可折叠

测试一下效果，如下，可以看到左侧菜单的宽度的确可以被动态设置了，但是内部的 Elment Plus 菜单组件的文字并没有被折叠起来。

![](https://img.quanxiaoha.com/quanxiaoha/169430774920686)

继续来优化，编辑 `AdminMenu` 组件，修改代码如下：

```html
<!-- 下方菜单 -->
	<el-menu :default-active="defaultActive" @select="handleSelect" :collapse="isCollapse" :collapse-transition="false">
	// 省略...
</el-menu>
        
<script setup>
	// 需要引入 computed 否则无法使用
	import {ref,computed} from 'vue'
	// 获取菜单是否折叠，避免折叠的时候，文字没有折叠  
const isCollapse = computed(() => !(menuStore.menuWidth === '250px'))
</script>
```

上述代码中，我们为 `<el-menu>` 组件添加了 `collapse` 属性，这个属性是官方提供的，用于指定是否折叠菜单。然后，我们将其绑定到了 `isCollapse` 变量上，并通过 `computed` 计算属性来动态计算是否需要折叠。`:collapse-transition="false"` 用于设置折叠时，取消过渡动画。再来看下效果：

![](https://img.quanxiaoha.com/quanxiaoha/169430824062718)

这次，文字也被折叠起来了。

### 2.5 左侧内容区域问题

你应该注意到了！左侧的内容区域，当菜单被折叠起来后，空了很大一块，太难看了！编辑 `admin.vue` 布局文件，同样需要为最顶层的 `<el-aside>` 动态设置宽度，只是设置内部的 `AdminMenu` 还不够，修改代码如下：

```html
<!-- 左边侧边栏 -->  
<el-aside :width="menuStore.menuWidth">  
  <AdminMenu/>  
</el-aside>
        
<script setup>
	import {useMenuStore} from "@/stores/menu.js";  
	// 同步菜单store中menu的宽度  
	const menuStore = useMenuStore()
</script>
```

现在，内容区域的空白也没有了：

![](https://img.quanxiaoha.com/quanxiaoha/169430851806788)

### 2.6 动态变化 Icon 图标

目前我们只有一个收缩的 `Icon` :

![](https://img.quanxiaoha.com/quanxiaoha/169430866222454)

当左侧菜单栏真正被收缩起来后，应该展示的是展开 `Icon` , 如下图所示:

![](https://img.quanxiaoha.com/quanxiaoha/169430876655765)

该图标可以在 Element Plus 提供的图标库中找到，地址为: [https://element-plus.org/zh-CN/component/icon.html](https://element-plus.org/zh-CN/component/icon.html) 。接下来，我们就来实现动态变化收缩、展开图标。

编辑 `AdminHeader` 组件，修改代码如下：

```html
<!-- 左边栏收缩、展开 -->  
<div @click="handleMenuWidth"  
     class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 hover:bg-gray-200">  
  <el-icon>  
    <!-- 实现收缩和展开时，显示不同的图标 -->  
    <Fold v-if="menuStore.menuWidth === '250px'"/>  
    <Expand v-else/>  
  </el-icon>  
</div>
```
上述代码中，通过添加 `<v-if>` 条件，来判断 `menuStore` 中的菜单宽度，若等于 `250px` , 则显示 `<Fold/>` 收缩图标，否则，显示 `<Expand/>` 展开图标。

### 2.7 Logo 图片

目前的 `Logo` 图片在菜单栏收缩起来的时候，会显得非常拥挤，不专业，不好看！

![](https://img.quanxiaoha.com/quanxiaoha/169430911702379)

我们需要将之前制作的 `Logo` 图片剪裁一下，高度保持不变，去掉 `Weblog` 文字部分，用于在收缩后来显示没有文字的 `Logo` 图片。大家可以用在线 PS 工具来制作一下，制作完成后，是下面这个样子：

![](https://img.quanxiaoha.com/quanxiaoha/169430931457773)

将其拖入 `/asserts` 文件夹下，命名为 `weblog-logo-mini.png` , 然后在 `AdminMenu` 组件中，添加动态展示不同 `logo` 图片的逻辑：

```
<!-- 顶部 Logo, 指定高度为 64px, 和右边的 Header 头保持一样高 -->  
<div class="flex items-center justify-center h-[64px]">  
  <img v-if="menuStore.menuWidth === '250px'"  
      src="/src/assets/AdminLogoBig.png" class="h-[70px]">  
  <img v-else src="/src/assets/AdminLogoMini.png" class="h-[70px]">  
</div>
```

判断逻辑与上述展示 `Icon` 是一致的，这里就不再赘述了。最终效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169430952066246)

嗯，看着舒服多了。

### 2.8 添加过渡动画

目前，收缩、展开左侧栏菜单的功能就基本搞定了，但是，你会发现菜单收缩、展开速度非常快，显得不够平滑。可以再为其添加一下过渡动画，让交互看起来更友好一些。

编辑 `AdminMenu` 组件，为最顶层的 `<div>` 添加 Tailwind CSS 提供的动画 `transition-all` ，它等同于设置了:

```css
transition-property: all;
transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
transition-duration: 150ms;
```

> 💡 TIP : 如果想单独设置过渡动画的时间，可通过添加 `duration` ，比如 `duration-200` , `duration-300` 等，可访问 Tailwind CSS 官方文档：[https://tailwindcss.com/docs/transition-duration](https://tailwindcss.com/docs/transition-duration)

![](https://img.quanxiaoha.com/quanxiaoha/169430979125627)

同样的，也需要为布局文件 `admin.vue` 中的 `<el-aside>` 组件添加一下：

![](https://img.quanxiaoha.com/quanxiaoha/169430985963112)

## 3 最终效果

完成上述设置工作后，你就可以看到一个当被点击后，能够平滑的收缩、展开的左侧菜单栏效果了：

![](https://img.quanxiaoha.com/quanxiaoha/169430600284952)

## 5 结语

本小节接着上小节的内容，通过全局状态管理 Pinia 实现了点击收缩、展开菜单栏的功能，主要是页面交互的一些小细节需要注意，比如展示不同图标 `Icon`, 菜单栏动态折叠判断，平滑过渡动画等等。