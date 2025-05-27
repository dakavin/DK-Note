---
文章标题: "[[12_添加Table组件加载Loading、表单对话框提交按钮Loading动画]]" 
文章作者: Dakkk
文章概要: |
  本文详细介绍了如何在Vue 3和Element Plus应用中，为表格组件和通用表单对话框的提交按钮添加加载Loading动画。通过控制响应式变量，实现数据请求期间的加载状态显示与隐藏，从而显著提升用户交互体验。
文章标签:
- "Vue 3"
- "Element Plus"
- "Loading动画"
- "用户体验"
- "组件交互"
- "前端开发"
- "表格"
- "表单对话框"
相关文章:
- "[[17_Element-plus组件库]]"
- "[[5_AdminMenu：点击收缩、展开功能实现]]"
- "[[6_分类管理页面：样式布局]]"
- "[[7_整合Element Plus 组件库]]"
- "[[9_登录页表单验证]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/5_管理后台：文章分类模块开发/12_添加Table组件加载Loading、表单对话框提交按钮Loading动画.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-30 08:32:22
修改时间: 2024-04-30 10:19:34
---

本小节中，我们将完成文章分类模块的收尾工作。将对应的一些组件交互完善一下，添加 `Table` 表格加载 `Loading` 动画，以及通用表单对话框提交按钮的 `Loading` 动画，让用户体验更加友好。
## 1 添加 Table 组件加载 Loading 动画

![](https://img.quanxiaoha.com/quanxiaoha/169560692327468)

### 1.1 反馈组件：loading 加载

Element Plus 官方提供了相关的反馈组件，其中就包括 `Loading` 加载，其官方文档地址为：[https://element-plus.org/zh-CN/component/loading.html](https://element-plus.org/zh-CN/component/loading.html) 。

### 1.2 添加 loading 加载

接下来，我们实际上手，看看要如何使用它？编辑 `category-list.vue` 分类列表页面，代码如下：

```html
<el-table :data="tableData" border stripe style="width: 100%" v-loading="tableLoading">
				// 省略...
            </el-table>
            
<script setup>
import { ref, reactive } from 'vue'

// 表格加载 Loading
const tableLoading = ref(false)

// 获取分页数据
function getTableData() {
	// 显示表格 loading
	tableLoading.value = true
	
	// 调用后台分页接口，并传入所需参数
	getCategoryPageList({current: current.value, size: size.value, startDate: startDate.value, endDate: endDate.value, name: searchCategoryName.value})
	.then((res) => {
			// 省略...
	})
	.finally(() => tableLoading.value = false) // 隐藏表格 loading
}
getTableData()

// 省略...
</script>
```

上述代码中，我们定义了一个响应式的 `tableLoading` 变量，并绑定到了 `<el-table>` 组件的 `v-loading` 属性上。在请求获取分页接口之前，我们将 `tableLoading` 设置为了 `true` , 显示加载动画，最后，当接口请求结束后，在 `finally` 语句块中，将其设置为了 `false` , 隐藏动画。

## 2 添加表单对话框提交按钮的 Loading 动画

要实现的效果如下，当点击提交按钮后，显示加载动画，请求接口结束后，隐藏该动画：

![](https://img.quanxiaoha.com/quanxiaoha/169560745789035)

### 2.1 为表单对话框添加显示、隐藏 loading 方法

`<el-button>` 组件默认提供了 `loading` 属性用于是否显示加载状态，`true` 为显示，`false` 为不显示。编辑 `FormDialog.vue` 组件，为提交按钮添加 `loading` 属性，代码如下：

```html
<el-button type="primary" @click="submit" :loading="btnLoading">
		{{ confirmText }}
</el-button>

<script setup>
import { ref } from 'vue'

// 省略...

// 确认按钮加载 loading
const btnLoading = ref(false)
// 显示 loading
const showBtnLoading = () => btnLoading.value = true
// 隐藏 loading
const closeBtnLoading = () => btnLoading.value = false

// 省略...

// 对外暴露方法
defineExpose({
		open,
		close,
		showBtnLoading,
		closeBtnLoading
})
```

上述代码中，我们定义了一个响应式的变量 `btnLoading` , 将它绑定到了提交按钮的 `loading` 属性上，用于控制按钮是否显示加载动画。并声明了两个方法：
- `showBtnLoading` : 显示按钮加载；
- `closeBtnLoading` ： 不显示按钮加载；

最终，将这两个方法通过 `defineExpose` 暴露出去，供父组件调用。

### 2.2 展示提交按钮的加载 loading

回到 `category-list.vue` 页面中，为新增分类对话框添加加载 `loading` , 修改代码如下：

```
const onSubmit = () => {
    // 先验证 form 表单字段
    formRef.value.validate((valid) => {
        if (!valid) {
            console.log('表单验证不通过')
            return false
        }
        // 显示提交按钮 loading
        formDialogRef.value.showBtnLoading()
        addCategory(form).then((res) => {
             // 省略...
        }).finally(() => formDialogRef.value.closeBtnLoading()) // 隐藏提交按钮 loading

    })
}
```

主要修改两个地方，在调用添加分类接口之前，显示提交按钮的 `loading` 动画，在请求接口结束后，隐藏提交按钮 `loading` 。

## 3 小作业

布置一个小作业，现在我们已经给通用的表单对话框组件，封装了显示、隐藏提交按钮的 `loading` 方法，将修改用户密码的也改造一下吧，让其能够显示 `loading` 动画。

## 4 结语

本小节中，我们主要完成了文章分类模块的收尾工作，为 `Table` 表格组件，以及表单对话框组件添加了 `Loading` 动画，提升用户交互体验。


