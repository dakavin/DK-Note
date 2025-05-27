---
文章标题: "[[13_公共 Header 头：跳转后台、退出登录功能开发]]" 
文章作者: Dakkk
文章概要: |
  文章详细阐述了Vue.js前端项目如何开发公共Header头中的“跳转后台”和“退出登录”功能。利用Vue Router实现后台页面跳转，并整合Flowbit模态框组件提供退出确认，最后通过调用用户状态管理中的方法完成安全退出登录的业务逻辑。
文章标签:
- "Vue.js"
- "前端开发"
- "Header组件"
- "路由跳转"
- "退出登录"
- "模态框"
- "Flowbit"
- "用户认证"
相关文章:
- "[[19_📕用户修改密码、退出功能开发]]"
- "[[17_Pinia 存储用户信息、动态显示登录用户名]]"
- "[[5_前台 Header 头组件封装]]"
- "[[16_Vue]]"
- "[[17_Ajax]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/9_博客前台：首页开发/13_公共 Header 头：跳转后台、退出登录功能开发.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-05-10 09:50:55
修改时间: 2025-05-27 01:00:21
---

本小节中，我们将把公共 `Header` 头中最后一点功能开发完成。也就是当用户处于已登录状态，点击下拉菜单中的 _跳转后台、退出登录_ 的功能开发完成。
## 1. 跳转后台

跳转后台实现比较简单，编辑 `/frontend/components/Header.vue` 组件，引入路由，并为进入后台 `<a>` 标签添加点击事件，跳转到后台首页地址即可，代码如下：

```html
<a @click="router.push('/admin/index')"  
   class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">  
    <!--svg引入图标-->  
    <svg class="inline w-3 h-3 mb-[2px] mr-1 text-gray-700 dark:text-white" aria-hidden="true"  
         xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">  
        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"  
              stroke-width="2"  
              d="M10 14v4m-4 1h8M1 10h18M2 1h16a1 1 0 0 1 1 1v11a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1Z" />  
    </svg>  
    进入后台  
</a>
// 省略..
                                
<script setup>
// 省略..
import { useRouter } from 'vue-router'

const router = useRouter()
// 省略..
</script>
```

## 2. 退出登录

### 2.1 弹出确认框

当用户点击退出登录时，可弹出一个确认对话框，提示用户是否确认退出登录。`Flowbit` 官方提供了模态框组件：[https://flowbite.com/docs/components/modal/](https://flowbite.com/docs/components/modal/) ， 并有多种样式可供选择，我们挑选一个合适的，复制其源码，并添加到 `Header.vue` 组件中：

![](https://img.quanxiaoha.com/quanxiaoha/169829317673771)

```
    <header>
    	// 省略...
    	<li>
<a data-modal-target="popup-modal" data-modal-toggle="popup-modal"
      class="block px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-600 dark:hover:text-white">
       // 省略...
       退出登录                                    
</a>
        </li>
        // 省略...
    </header>
    <!-- 退出登录 -->
    <div id="popup-modal" tabindex="-1"
        class="fixed top-0 left-0 right-0 z-50 hidden p-4 overflow-x-hidden overflow-y-auto md:inset-0 h-[calc(100%-1rem)] max-h-full">
        <div class="relative w-full max-w-md max-h-full">
            <div class="relative bg-white rounded-lg shadow dark:bg-gray-700">
                <button type="button"
                    class="absolute top-3 right-2.5 text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ml-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
                    data-modal-hide="popup-modal">
                    <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                        viewBox="0 0 14 14">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                            d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6" />
                    </svg>
                    <span class="sr-only">Close modal</span>
                </button>
                <div class="p-6 text-center">
                    <svg class="mx-auto mb-4 text-gray-400 w-12 h-12 dark:text-gray-200" aria-hidden="true"
                        xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                            d="M10 11V6m0 8h.01M19 10a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
                    </svg>
                    <h3 class="mb-5 text-lg font-normal text-gray-500 dark:text-gray-400">是否确定退出登录?
                        </h3>
                    <button data-modal-hide="popup-modal" type="button"
                        class="text-white bg-red-600 hover:bg-red-800 focus:ring-4 focus:outline-none focus:ring-red-300 dark:focus:ring-red-800 font-medium rounded-lg text-sm inline-flex items-center px-5 py-2.5 text-center mr-2">
                        确定
                    </button>
                    <button data-modal-hide="popup-modal" type="button"
                        class="text-gray-500 bg-white hover:bg-gray-100 focus:ring-4 focus:outline-none focus:ring-gray-200 rounded-lg border border-gray-200 text-sm font-medium px-5 py-2.5 hover:text-gray-900 focus:z-10 dark:bg-gray-700 dark:text-gray-300 dark:border-gray-500 dark:hover:text-white dark:hover:bg-gray-600 dark:focus:ring-gray-600">
                        取消</button>
                </div>
            </div>
        </div>
    </div>
    
    
    <script setup>
    
    import { initCollapses, initDropdowns, initModals } from 'flowbite'
    
    // 初始化 flowbit 相关组件
    onMounted(() => {
        // 省略...
        initModals();
    })
    </script>
```

> _注意：复制的组件代码需要和 `<header>` 标签同级，否则会出现样式问题。_

上述代码中，模态框组件通过 `id="popup-modal"` 标识其唯一性，当想要点击退出登录菜单时才显示它，只需要为退出登录 `<a>` 标签添加 `data-modal-target="popup-modal" data-modal-toggle="popup-modal"` 两个属性即可，值需要和模态框组件的 `id` 保持一致。

![](https://img.quanxiaoha.com/quanxiaoha/169829163079611)

### 2.2 退出登录逻辑

当用户点击确认按钮后，才会真正执行退出登录逻辑。只需要为确定按钮添加 `@click` 点击事件，并直接执行 `userStore` 中封装好的 `logout` 退出登录方法即可。注意还需要将 `isLogined` 用户是否登录变量设置为 `false`, 最后弹出退出登录成功的提示消息，代码如下：

```
<button @click="logout" data-modal-hide="popup-modal" type="button"
                        class="text-white bg-red-600 hover:bg-red-800 focus:ring-4 focus:outline-none focus:ring-red-300 dark:focus:ring-red-800 font-medium rounded-lg text-sm inline-flex items-center px-5 py-2.5 text-center mr-2">
                        确定
                    </button>
                    // 省略...
                    
                    <script setup>
                    import { showMessage } from '@/composables/util'
                    
                    // 省略...
                    // 退出登录
                    const logout = () => {
                        userStore.logout()
                        // 标记为未登录
                        isLogined.value = false
                        showMessage('退出登录成功')
                    }
                    </script>
```