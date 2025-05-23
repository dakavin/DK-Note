
## 1 支持全屏展示

想要实现页面的全屏展示，其实非常简单。之前小节中说到的 `VueUse` 这个工具集合中就包含了这个功能库。我们可以在官网：[https://vueuse.org/](https://vueuse.org/) 搜索关键词 _useFullscreen_, 就能找到这个库了：

![](https://img.quanxiaoha.com/quanxiaoha/169439740012354)

接下来，让我们通过它来实现全屏功能。

### 1.1 安装 VueUse 核心库

首先，终端中执行如下命令，安装 `VueUse` 核心库：
```shell
npm i @vueuse/core
```

### 1.2 整合 useFullscreen

安装完成后，编辑 `AdminHeader` 组件，修改代码如下：

```html
<!-- 点击全屏展示 -->  
<el-tooltip class="box-item" effect="dark" content="全屏" placement="bottom">  
	<div @click="toggle"  
			class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">  
		<el-icon>  
			<FullScreen/>  
		</el-icon>  
	</div>  
</el-tooltip>
            

<script setup>
    // 引入 useFullscreen
    import { useFullscreen } from '@vueuse/core'

	// isFullscreen 表示当前是否处于全屏；toggle 用于动态切换全屏、非全屏
    const { isFullscreen, toggle } = useFullscreen()
</script>
```

上述代码中，我们首先引入核心包中的 `useFullscreen` ，然后拿到了它提供的 `isFullscreen` 变量，它被用于记录当前是否处于全屏；还有一个 `toggle` 方法，它用于动态切换全屏、非全屏。最后，为全屏图标添加了 `@click` 点击事件，直接调用的 `toggle` 方法。

完成上述工作，就搞定页面的全屏展示功能了，是不是非常简单，效果图如下：

![](https://img.quanxiaoha.com/quanxiaoha/169439820664255)

### 1.3 图标美化

目前不管是已经处于全屏状态、还是非全屏状态，展示的都是下面这个图标：

![](https://img.quanxiaoha.com/quanxiaoha/169439834430428)

我们希望，当处于全屏状态时，展示的是一个退出全屏的图标，如下：

![](https://img.quanxiaoha.com/quanxiaoha/169439841604946)

> 💡 TIP : 图标可以访问 Elment Plus 官网来获取：[https://element-plus.org/zh-CN/component/icon.html#icon-collection](https://element-plus.org/zh-CN/component/icon.html#icon-collection)

编辑 `AdminHeader` 组件，添加展示不同图标的逻辑判断：

```html
<el-icon>
	<FullScreen v-if="!isFullscreen"/>
	<Aim v-else/>
</el-icon>


// 方法二

<!-- 点击全屏展示 -->  
<el-tooltip v-if="!isFullscreen" class="box-item" effect="dark" content="全屏" placement="bottom">  
  <div @click="toggle"  
      class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">  
    <el-icon>  
      <FullScreen/>  
    </el-icon>  
  </div>  
</el-tooltip>  
<!-- 点击退出全屏展示 -->  
<el-tooltip v-else class="box-item" effect="dark" content="退出全屏" placement="bottom">  
  <div @click="toggle"  
       class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">  
    <el-icon>  
      <Aim/>  
    </el-icon>  
  </div>  
</el-tooltip>
```

上述代码中，我们对 `isFullscreen` 状态进行了 `if-else` 判断，若非全屏状态，则展示全屏图标，否则，展示退出全屏图标。

## 2 手动更新页面

继续编辑 `AdminHeader` 组件。将之前已经设计好的全屏`Icon` 的整体代码重新复制一份，位置放到全屏前面，然后，将中间的 `Icon` 图标替换成刷新图标：

> 💡 TIP : 刷新图标可从 Element Plus 官网来获取。

```html
<el-tooltip class="box-item" effect="dark" content="刷新当前页面" placement="bottom">  
  <div @click="handleRefresh"  
       class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">  
    <el-icon>  
      <Refresh/>  
    </el-icon>  
  </div>  
</el-tooltip>

<script setup>
// 省略..
// 刷新页面  
const handleRefresh = ()=>location.reload()
</script>  
```

上述代码中，我们为刷新图标添加了 `@click` 点击事件，直接执行 `location.reload()` 方法，即可实现页面刷新啦~

## 4 结语

本小节功能代码实现比较简单，主要就是完善了 `AdminHeader` 组件的两个细节功能，支持了页面全屏展示，以及手动刷新页面。