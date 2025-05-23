## 1 最终效果

完成本小节的功能开发，最终管理后台的标签导航栏组件效果如下图：

![](https://img.quanxiaoha.com/quanxiaoha/169442997792760)

## 2 布局分析

先来分析一波布局，再动手敲代码：

![](https://img.quanxiaoha.com/quanxiaoha/169443008443456)

如上图所示，我们需要的是一个左右布局：

- ①：左侧栏是标签导航栏；
    
- ②：右侧是一个下拉菜单，鼠标悬浮其上，会弹出菜单，以供用户关闭标签页；
    
    ![](https://img.quanxiaoha.com/quanxiaoha/169443019531649)
    

## 3 开始动手

### 3.1 实现基本布局

#### 3.1.1 Element Plus Tabs 标签页

关于左侧的标签导航栏，我们可以使用 Element Plus 官方提供了 `Tabs` 标签页来实现，其访问地址为：[https://element-plus.org/zh-CN/component/tabs.html](https://element-plus.org/zh-CN/component/tabs.html) ，提供的类型也比较多，我们需要的是 _自定义增加标签页触发器_：

![](https://img.quanxiaoha.com/quanxiaoha/169443043597866)

将其核心代码复制到 `AdminTagList` 组件中，最终代码如下：

```html
<template>
	<!-- 左边：标签导航栏 -->  
	<div>  
	  <el-tabs v-model="editableTabsValue" type="card" class="demo-tabs" closable @tab-remove="removeTab">  
	    <el-tab-pane v-for="item in editableTabs" :key="item.name" :label="item.title" :name="item.name">  
	      {{ item.content }}  
	    </el-tab-pane>  
	  </el-tabs>  
	</div>
</template>

<script setup>  
import { ref } from 'vue'  
  
let tabIndex = 2  
const editableTabsValue = ref('2')  
const editableTabs = ref([  
  {    title: 'Tab 1',  
    name: '1',  
    content: 'Tab 1 content',  
  },  
  {  
    title: 'Tab 2',  
    name: '2',  
    content: 'Tab 2 content',  
  },  
])  
  
const addTab = (targetName) => {  
  const newTabName = `${++tabIndex}`  
  editableTabs.value.push({  
    title: 'New Tab',  
    name: newTabName,  
    content: 'New Tab content',  
  })  
  editableTabsValue.value = newTabName  
}  
const removeTab = (targetName) => {  
  const tabs = editableTabs.value  
  let activeName = editableTabsValue.value  
  if (activeName === targetName) {  
    tabs.forEach((tab, index) => {  
      if (tab.name === targetName) {  
        const nextTab = tabs[index + 1] || tabs[index - 1]  
        if (nextTab) {  
          activeName = nextTab.name  
        }  
      }  
    })  
  }  
  editableTabsValue.value = activeName  
  editableTabs.value = tabs.filter((tab) => tab.name !== targetName)  
}  
</script>

<style>
</style>
```

> ⚠️ 注意：官方提供的代码中， `<script>` 使用的 `TypeScript` , 本项目未使用 `TypeScript`, 所以需要将和 `TypeScript` 相关的内容需要都删除掉。

修改代码完成后，效果图如下：

![](https://img.quanxiaoha.com/quanxiaoha/169443231082474)

#### 3.1.2 添加右侧下拉菜单

从官网 [https://element-plus.org/zh-CN/component/dropdown.html#%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95](https://element-plus.org/zh-CN/component/dropdown.html#%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95) 获取下拉菜单组件代码，稍作修改，代码如下：

```
        // 省略...
		<el-tabs v-model="editableTabsValue" type="card" class="demo-tabs" closable @tab-remove="removeTab">
            <el-tab-pane v-for="item in editableTabs" :key="item.name" :label="item.title" :name="item.name">
                {{ item.content }}
            </el-tab-pane>
        </el-tabs>


        <!-- 右侧下拉菜单 -->
        <span>
            <el-dropdown>
                <span class="el-dropdown-link">
                    <el-icon class="el-icon--right">
                        <arrow-down />
                    </el-icon>
                </span>
                <template #dropdown>
                    <el-dropdown-menu>
                        <el-dropdown-item>关闭其他</el-dropdown-item>
                        <el-dropdown-item>关闭全部</el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
        </span>
        // 省略...
```

添加完成后，效果如下:

![](https://img.quanxiaoha.com/quanxiaoha/169443259859720)

可以看到现在的布局还不是左右布局，需要继续调整：

```html
<div class="fixed top-[64px] h-[44px] px-2 right-0 z-50 flex items-center bg-white" style="left: 250px;">  
  <!-- 左边：标签导航栏 -->  
  <el-tabs v-model="editableTabsValue" type="card" class="demo-tabs" closable @tab-remove="removeTab">  
    <el-tab-pane v-for="item in editableTabs" :key="item.name" :label="item.title" :name="item.name">  
      {{ item.content }}  
    </el-tab-pane>  
  </el-tabs>  
  
  <!-- 右侧下拉菜单 -->  
  <span class="ml-auto">  
    <el-dropdown>
```

上述代码中，我们给最顶层的 `<div>` 容器设置了 `fixed` 固定位置，高 `44px`, 距离顶部 `64px` , 这个高度和 `AdminHeader` 组件高度一致，水平间距为 `px-2` , 距离右边为 `0` , 垂直高度为 `z-50`, 方向通过 `flex items-center` 将 `<div>` 设置为水平布局，且内容垂直居中，`bg-white` 背景色为白色。另外，单独通过 `style` 设置了距离左边 `250px`, 这个宽度是左边菜单栏的宽度。最后，将右侧下拉菜单通过 `ml-auto` 设置为了靠最右边。效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169443334725564)

基本的布局实现了，但是还要继续调整细节样式。

### 3.2 样式美化

#### 3.2.1 内容区域文字消失了

从上图也看到了，内容区域中，选中左侧不同的菜单，是应该显示对应页面中的文字的，现在 _不见了_。原因是导航栏为 `fixed` 布局，且背景色为白色，导致内容被遮挡住了。修改代码如下：

```html
    // 省略。。。
</div>  
<div class="h-[44px]"></div>
```

我的解决方案是在最顶层 `<div>` 下面再放置一个高度为 `44px` 的容器，里面没有任何内容，目的就是把被隐藏的内容顶下去。注意，`<el-tabs>` 内的 `{{ item.content }}` 同时要删除掉，效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169443410213194)

OK , 现在被隐藏的文字显示出来了。

#### 3.2.2 标签 Item 样式美化

原生的标签 `Item` 比较丑陋，与我们现在的主题也不搭。需要我们 `F12` 审查元素，慢慢找到相关 `CSS` 选择器重写自己想要的样式，慢工出细活，过程也比较繁琐。这里小哈就不一一讲解了，直接贴出来我调整好的 `CSS` 代码，小伙伴们直接复制到 `<style>` 标签内即可：

```css
<style>  
.el-tabs__item {  
    font-size: 12px;  
    border: 1px solid #d8dce5!important;  
    border-radius: 3px!important;  
}  
  
.el-tabs--card>.el-tabs__header .el-tabs__item {  
    margin-left: 0.1rem!important;  
    margin-right: 0.1rem!important;  
}  
  
.el-tabs__item.is-active {  
    background-color: var(--el-color-primary)!important;  
    color: #fff;  
}  
  
.el-tabs__item.is-active::before {  
    content: "";  
    background-color: #fff;  
    display: inline-block;  
    width: 8px;  
    height: 8px;  
    border-radius: 50%;  
    position: relative;  
    margin-right: 4px;  
}  
  
.el-tabs {  
    height: 32px;  
}  
  
.el-tabs__header {  
    margin-bottom: 0;  
}  
  
.el-tabs--card>.el-tabs__header .el-tabs__nav {  
    border: 0;  
}  
  
.el-tabs--card>.el-tabs__header .el-tabs__item {  
    height: 32px;  
    line-height: 32px;  
    border: 0;  
    background: #fff;  
}  
  
.el-tabs--card>.el-tabs__header {  
    border: 0;  
}  
  
.el-tabs__nav-prev, .el-tabs__nav-next {  
    line-height: 35px;  
}  
  
.is-disabled {  
    cursor: not-allowed;  
    color: #d1d5db;  
}  
  
</style>
```

效果如下，现在效果就舒服多了：

![](https://img.quanxiaoha.com/quanxiaoha/169443446036284)

#### 3.2.3 右边下拉菜单样式美化

上图中可以看到，右边的下拉菜单内容不是垂直居中的，

```html
<!-- 右侧下拉菜单 -->  
<span class="ml-auto flex items-center justify-center h-[32px] w-[32px]">  
  <el-dropdown>  
    <span class="el-dropdown-link">  
      <el-icon class="el-icon--right">  
        <arrow-down />  
      </el-icon>  
    </span>  
    <template #dropdown>  
      <el-dropdown-menu>  
        <el-dropdown-item>关闭其他</el-dropdown-item>  
        <el-dropdown-item>关闭全部</el-dropdown-item>  
      </el-dropdown-menu>  
    </template>  
  </el-dropdown>  
</span>
```

上述代码中，我们为下拉菜单设置了 `flex items-center justify-center h-[32px] w-[32px]`, 让其内容水平垂直居中，同时宽高均为 `32px`。最后，将 `<el-icon>` 自带的样式也给删除了，官方给它添加了一个右边距，这里我们不需要。完成后，效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169443477337973)

现在内容居中显示了。

### 3.3 标签无法滚动问题

还有个问题，当子标签比较多，且超过页面宽度时，是 _无法滚动的！_

![](https://img.quanxiaoha.com/quanxiaoha/169443501601456)

要解决这个问题，也比较简单，为 `<el-tabs>` 组件添加一个 `style="min-width: 10px;"` 即可。

![](https://img.quanxiaoha.com/quanxiaoha/169443514022046)

设置完成后，我们再来看效果，标签能够正常滚动了：

![](https://img.quanxiaoha.com/quanxiaoha/169443522546780)

### 3.4 收缩左侧菜单，标签导航栏未跟随

当我们点击收缩左侧菜单栏，标签栏由于设定了 `left: 250px` , 这个值是写死的，所以会像下面这样：

![](https://img.quanxiaoha.com/quanxiaoha/169443537756271)

解决这个问题也比较简单，Pinia 的全局状态又要上场了, 修改代码如下：

```html
<!-- 左边：标签导航栏 -->
<div class="fixed top-[64px] h-[44px] px-2 right-0 z-50 flex items-center bg-white transition-all duration-300 shadow" :style="{left: menuStore.menuWidth}">
	//省略...
</div>

	//省略...

<script setup>
import { useMenuStore } from '@/stores/menu'

const menuStore = useMenuStore()
</script>
```

上述代码中，通过 `:style` 为导航栏动态设置了 `left` 值，同时添加了 `transition-all duration-300 shadow` Tailwind CSS 样式，为其添加了过渡动画，以及阴影效果。

### 3.5 Header 头中的用户头像下拉菜单位置靠下问题

这里顺手解决一个 `Bug`，`AdminHeader` 头组件中的用户头像下拉菜单位置比较靠下的问题，如下图所示:

![](https://img.quanxiaoha.com/quanxiaoha/169443586692817)

编辑 `AdminHeader` 组件, 为 `<el-dropdown>` 组件添加 `flex items-center justify-center` 样式，即可搞定：

```html
<!-- 登录用户头像 -->
<el-dropdown class="flex items-center justify-center">
		// 省略...
</el-dropdown>
```

添加完成后，可以看到下拉菜单位置上移了：

![](https://img.quanxiaoha.com/quanxiaoha/169443605710350)

### 3.6 Header 头的 border 值调整

因为加了标签导航栏，为了让 `AdminHeader` 组件与标签导航栏组件看上去像一个整体，我们需要将 `border` 的颜色深度调的更浅一点，改为 `border-slate-100` :

![](https://img.quanxiaoha.com/quanxiaoha/169443679157127)

## 4 最终效果图

![](https://img.quanxiaoha.com/quanxiaoha/169442997792760)

## 6 结语

本小节中，小哈主要带着大家将后台管理的标签导航栏的静态样式布局搞定了，风格上也比较好看，很贴合现有的主题。