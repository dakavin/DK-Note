
本小节中，将带着大家将**登录页的细节交互优化一下**，比如登录成功、失败的消息提示；输入完成用户名、密码后，按回车键直接登录，无需点击登录按钮，以及登录按钮的 Loading 加载提示功能。

## 1 添加消息提示

Element Plus 提供了消息提示组件，直接拿来用就行，具体可参考官方文档： [https://element-plus.org/zh-CN/component/message.html](https://element-plus.org/zh-CN/component/message.html) 。

### 1.1 提示组件样式

失败红框提示，如下图所示：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8d6888cbd5d25590d45f4171298a559.png)

成功绿框提示，如下图所示：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/44f9cf2acab6cf8e66cf4dec6e3fe85d.png)

除了以上两种「成功、错误」类消息以外，官方还提供了「警告、消息」类的操作反馈，可自行去官网查看。

> **提示**：部分小伙伴反馈说，本小节中弹出 ElMessage 组件的样式失效了，排查原因是因为安装的 Element Plus 版本较高，高版本可能对于自动引入插件可能支持上有 BUG, 提供以下两种解决方案：
> 
> - **解决方式1**：将项目中的 `package.json` 中的 Element Plus 版本修改为和小节源码的版本一致： `2.3.9` 版本，然后重新执行 `npm install` 命令。
> - **解决方式2** ： 若上述方法样式未能解决问题，也可以查看官方 issue 的解决方案：[https://github.com/element-plus/element-plus/issues/9214](https://github.com/element-plus/element-plus/issues/9214) ，手动引入 `ElMessage` 组件的样式文件，编辑 `/composable/util.js` 文件，在文件头部添加如下引入：

```js
// 导入这两行信息，实现按需引入ElMessage组件
import 'element-plus/es/components/message/style/css'  
import {ElMessage} from "element-plus";
```

### 1.2 修改代码

想要添加消息组件，大致格式如下：

```js
ElMessage({
    message: '提示内容',
    type: 'success',
})
```
- `message` : 填写想要提示的内容；
- `type`: 消息类型，如 `success` 、`error`、`warning`、`error`, 对应的样式也不一样;

以下是登录页的修改内容：
```
		// 省略...
		// 调用登录接口
        login(form.username, form.password).then((res) => {
            console.log(res)
            // 判断是否成功
            if (res.data.success == true) {
            	// 提示登录成功
                ElMessage({
                    message: '登录成功',
                    type: 'success',
                })

                // 跳转到后台首页
                router.push('/admin/index')
            } else {
                // 获取服务端返回的错误消息
                let message = res.data.message
                // 提示消息
                ElMessage({
                    message: message,
                    type: 'error',
                })
            }
        })
       // 省略...
```

以上代码中，我们在请求成功后，弹出了_登录成功_的消息提示，若登录失败，则获取服务端返回的错误信息，并动态用 `error` 级别的消息弹出。

## 2 封装消息提示工具类

其实，上面的消息提示代码，我们可以再封装成一个工具类，这样使用起来就会更加方便。在 `/src` 目录下创建 `/composables` 文件夹，专门用于==放置工具类相关代码：==
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/446a5d0b942b1be4738f66057578a055.png)

新建 `util.js` , 添加一个 `showMessage()` 弹出提示框方法：
```js
// 消息提示
export function showMessage(message = '提示内容', type = 'success', customClass = '') {
    return ElMessage({
        type: type,
        message,
        customClass,
    })
}
```
入参有三个字段：
- `message` : 提示内容；
- `type` : 默认为成功消息提示；
- `customClass` : 用于传入自定义样式，默认为空；

## 3 直接使用 util.js

封装好了工具类后，我们来改造一下代码，先==引入刚刚封装好的方法==，如下：
```js
import { showMessage} from '@/composables/util'
	// 省略...
	// 调用登录接口
	login(form.username, form.password).then((res) => {
		console.log(res)
		// 判断是否成功
		if (res.data.success == true) {
			// 提示登录成功
			showMessage('登录成功')

			// 跳转到后台首页
			router.push('/admin/index')
		} else {
			let message = res.data.message
			// 提示消息
			showMessage(message, 'error')
		}
	})
	// 省略...
```

怎么样，是不是清爽了很多~

## 4 回车键直接登录

很多交互做的好一点的网站，都会有这样的功能，当输入完用户名、密码后，==直接敲击回车键就能完成登录了==。我们也来实现该功能，改造代码如下：
```js
<script setup>
// 省略...
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'

// 省略...

// 按回车键后，执行登录事件
function onKeyUp(e) {
    console.log(e)
    if (e.key == 'Enter') {
        onSubmit()
    }
}

// 添加键盘监听
onMounted(() => {
    console.log('添加键盘监听')
    document.addEventListener('keyup', onKeyUp)
})

// 移除键盘监听
onBeforeUnmount(() => {
    document.removeEventListener('keyup', onKeyUp)
})
</script>
```

首先，我们引入了 `onMounted` 、`onBeforeUnmount` 生命周期方法，然后在 `onMounted()` 方法中添加了键盘监听事件，当键盘的 `key` 值为 `Enter` 时，也就是回车键时，直接调用 `onSubmit()` 登录。然后在 `onBeforeUnmount()` 生命周期方法中，移除了键盘监听事件。

## 5 按钮加载

按钮加载样式如下，当触发登录后，会调用后台接口，网络 IO 可能会消耗一点时间，为了给用户一个友好的提示，显示加载 Loading 非常有必要：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/504255bbc8a04e204208e4118ac963f5.png)

代码修改如下：

```html
// 省略...
<el-button class="w-full mt-2" size="large" :loading="loading" type="primary" @click="onSubmit">登录</el-button>

<script setup>
// 登录按钮加载
const loading = ref(false)

const onSubmit = () => {
	// 省略...
    // 先验证 form 表单字段
    formRef.value.validate((valid) => {
	    // 省略...
        
        // 开始加载
        loading.value = true

        // 调用登录接口
        login(form.username, form.password).then((res) => {
            // 省略...
        })
        .finally(() => {
            // 结束加载
            loading.value = false
        })
    })
}
</script>
```

Element Plus 提供的按钮组件提供了加载属性 `loading`, 通过设置为 `true` 或者 `false` 来决定是否要显示加载状态。上述代码中，我们声明了一个响应式的变量 `loading`, 在请求登录接口的开始，将其设置为了 `true`, 然后请求结束后，将其设置为 `false`, 恢复正常样式。

## 6 结语

本小节中，主要带着大家将登录页的一些交互问题优化了一下，另外，还封装了消息提示工具类，可以让后续的功能开发中，更加方便快捷的使用消息提示。