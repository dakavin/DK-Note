本小节中，我们来填充一下上小节中定义好的 `AdminHeader.vue` 组件的静态内容。需要完成效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169415636597174)

## 1 边距问题

上小节实现了基本的页面布局，你会发现头部左右有边距，通过 _F12_ 审查元素，发现是 Element Plus 组件样式导致的，如下图所示：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a082c65dfd4f0441badec182a6050da0.png)


编辑 `admin.vue` 布局文件添加 `CSS` 样式，重写 `.el-header` 选择器，将 `padding` 设置为 0：

```
<style scoped>
.el-header {
    padding: 0!important;
}
</style>
```

设置完成后，边距就消失啦：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbd18058bee61ab86bebde8538703375.png)

## 2 分析想要的布局

接着，开始整 `Header` 头里面的内容，分析一波我们想要的布局如下，分为左侧点击收缩菜单栏图标，以及右侧用户头像下拉框、全屏图标：

![](https://img.quanxiaoha.com/quanxiaoha/169415115075381)

## 3 动手实现

Element Plus 官方提供了一些 `Icon` 图标可供使用：[https://element-plus.org/zh-CN/component/icon.html](https://element-plus.org/zh-CN/component/icon.html) ，将页面拉到最下方，有所有图标的列表，通过_点击即可复制图标代码_：

![](https://img.quanxiaoha.com/quanxiaoha/169415964107767)

这里，我们需要的收缩菜单栏 `Icon` , 以及全屏 `Icon` 代码如下：

```html
 <!-- 左边栏收缩、展开 -->
 <el-icon>
	 <Fold />
 </el-icon>
 
<!-- 点击全屏展示 -->
<el-icon>
	<FullScreen />
</el-icon> 
```

除了图标外，还需要一个 Element Plus 提供的下拉菜单组件，该组件官方地址访问地址为：[https://element-plus.org/zh-CN/component/dropdown.html](https://element-plus.org/zh-CN/component/dropdown.html) ，使用格式如下：

```html
<el-dropdown>
    <span class="el-dropdown-link">
      Dropdown List
      <el-icon class="el-icon--right">
        <arrow-down />
      </el-icon>
    </span>
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item>Action 1</el-dropdown-item>
        <el-dropdown-item>Action 2</el-dropdown-item>
        <el-dropdown-item>Action 3</el-dropdown-item>
        <el-dropdown-item disabled>Action 4</el-dropdown-item>
        <el-dropdown-item divided>Action 5</el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
```

效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169415991747030)

将上述的 `Icon` 图标、以及下拉框菜单复制到 `AdminHeader.vue` 组件中，代码如下：

```html
<template>
	<!-- 通过 flex 指定水平布局 -->
    <div class="bg-emerald-700 h-[64px] text-white flex">
        <!-- 左边栏收缩、展开 -->
        <el-icon>
            <Fold />
        </el-icon>

        <!-- 右边容器，通过 ml-auto 让其在父容器的右边 -->
        <div class="ml-auto">
            <!-- 点击全屏展示 -->
            <el-icon>
                <FullScreen />
            </el-icon>

            <!-- 登录用户头像 -->
            <el-dropdown>
                <span class="el-dropdown-link text-white">
                    <!-- 头像 Avatar -->
                    <el-avatar :size="25" src="https://img.quanxiaoha.com/quanxiaoha/f97361c0429d4bb1bc276ab835843065.jpg" />
                    Admin
                    <el-icon class="el-icon--right">
                        <arrow-down />
                    </el-icon>
                </span>
                <template #dropdown>
                    <el-dropdown-menu>
                        <el-dropdown-item>修改密码</el-dropdown-item>
                        <el-dropdown-item>退出登录</el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
        </div>
    </div>
</template>
```

上述代码中，我们将父级 `<div>` 通过 `flex` 实现水平布局，并添加了收缩栏 `Icon` ，然后，定义了一个右边容器 `<div>` , 并通过 `ml-auto` 让其靠右边展示。里面分别又添加了全屏 `Icon` , 以及用户登录头像的下拉菜单，注意，在 `<el-dropdown>` 组件里面，添加了一个 `<el-avatar>` 头像组件，大小为 25, 菜单内容改成了 _修改密码_、_退出登录_，最终效果如下：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d9804cdd2fa6e25e56a81aab4fffd3a.png)


## 4 父容器背景色、高度、边距设置

有了基本的内容，我们来调整一下 `CSS` 样式，让其更好看点：
```html
<!-- 设置背景色为白色、高度为 64px，padding-right 为 4， border-bottom 为 slate 200 -->
<div class="bg-white h-[64px] flex pr-4 border-b border-slate-200">
</div>
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78bba353d8031c93cb8bdd1f244c4859.png)


## 5 图标样式调整

为了不让 Element Plus 内部的样式覆盖 Tailwind CSS 的样式，需要用一层 `<div>` 将包裹起来：
```html
<!-- 左边栏收缩、展开 -->
<div class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 hover:bg-gray-200">
    <el-icon>
        <Fold />
    </el-icon>
</div>

```

解释一下每个样式的作用:
- `w-[42px] h-[64px]` : 设置宽 `42px`, 高为 `64px` ；
- `cursor-pointer` : 鼠标移动上去呈现小手指点击样式；
- `flex items-center justify-center` : 搭配使用，水平、垂直居中；
- `text-gray-700` : 字体颜色；
- `hover:bg-gray-200` : 鼠标移动上去时，背景色为 `bg-gray-200`;

调整完毕后，该图标样式如下：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28d8ffeb7ee82155daed93978751a3c5.png)


同样的思路，我们调整一下全屏图标：
```java
<!-- 点击全屏展示 -->
<div class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">
    <el-icon>
    	<FullScreen />
    </el-icon>
</div>
```

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58d95948c2cd43d7536180c8215e11d5.png)

## 6 用户头像区域调整

图标的 `CSS` 样式是搞定了，现在用户头像的 `dropdown` 明显出问题了，我们来调整一下：

```html
		<!-- 右边容器 -->
        <div class="ml-auto flex">
            // 省略...

            <!-- 登录用户头像 -->
            <el-dropdown>
                <span class="el-dropdown-link text-white">
                    <!-- 头像 Avatar -->
                    <el-avatar :size="25"
                        src="https://img.quanxiaoha.com/quanxiaoha/f97361c0429d4bb1bc276ab835843065.jpg" />
                    Admin
                    <el-icon class="el-icon--right">
                        <arrow-down />
                    </el-icon>
                </span>
                <template #dropdown>
                    <el-dropdown-menu>
                        <el-dropdown-item>修改密码</el-dropdown-item>
                        <el-dropdown-item>退出登录</el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
        </div>
```

上述代码中，我们将 `el-dropdown` 组件的父级 `div` 设置成了 `flex` 水平布局。

![](https://img.quanxiaoha.com/quanxiaoha/169415830663766)

继续调整，代码如下：
```html
<!-- 登录用户头像 -->  
<el-dropdown class="flex items-center justify-center">  
          <span class="el-dropdown-link flex items-center justify-center text-gray-700 text-xs">  
              <!-- 头像 Avatar -->              
              <el-avatar class="mr-2" :size="25" src="./src/assets/default_avatar.webp" />  
              Admin  
              <el-icon class="el-icon--right">  
                  <arrow-down />  
              </el-icon>  
          </span>  
  <template #dropdown>  
    <el-dropdown-menu>  
      <el-dropdown-item>修改密码</el-dropdown-item>  
      <el-dropdown-item>退出登录</el-dropdown-item>  
    </el-dropdown-menu>  
  </template>  
</el-dropdown>
```

上述代码中，我们对 `dropdown` 设置样式如下：
- `flex items-center justify-center` : 水平垂直居中；
- `text-gray-700 text-xs` : 字体为最小号，字体样式为灰色；

最终效果如下：

![](https://img.quanxiaoha.com/quanxiaoha/169415636597174)

## 7 添加 Tooltip 文字提示

![](https://img.quanxiaoha.com/quanxiaoha/169415866662916)

效果如上所示，当鼠标移动到某个区域，也是是 `hover` 时的提示信息，Element Plus 提供了响应的组件以供使用，访问地址：[https://element-plus.org/zh-CN/component/tooltip.html](https://element-plus.org/zh-CN/component/tooltip.html) ， 使用格式如下:

```html
<el-tooltip
	class="box-item"
	effect="dark"
	content="Bottom Center prompts info"
	placement="bottom"
>
	<el-button>bottom</el-button>
</el-tooltip>
```

该组件的属性解释如下：
- 使用 `content` 属性来决定 `hover` 时的提示信息。
- 由 `placement` 属性决定展示效果： `placement`属性值为：`[方向]-[对齐位置]`；四个方向：`top`、`left`、`right`、`bottom`；三种对齐位置：`start`, `end`，默认为空。 如 `placement="left-end"`，则提示信息出现在目标元素的左侧，且提示信息的底部与目标元素的底部对齐。

有了它，接下来，改造一下我们 `Header` 头中的图标：

```
			<!-- 点击全屏展示 -->
            <el-tooltip class="box-item" effect="dark" content="全屏" placement="bottom">
                <div class="w-[42px] h-[64px] cursor-pointer flex items-center justify-center text-gray-700 mr-2 hover:bg-gray-200">
                    <el-icon>
                        <FullScreen />
                    </el-icon>
                </div>
            </el-tooltip>
```

仅需要通过 `<el-tooltip>` 组件包裹住，就可以生成 `hover` 的提示信息了：

![](https://img.quanxiaoha.com/quanxiaoha/169415866662916)
## 9 结语

本小节中，主要带着大家将管理后台的 `Header` 头填充好了，有了基本左右布局，添加了收缩 、全屏 `Icon` , 以及用户下拉菜单，但这些都是静态的，后续小节中我们继续完善它们。