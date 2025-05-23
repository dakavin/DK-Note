
上小节中，我们已经将 _获取当前登录用户信息接口_ 开发好了。本小节中，我们回到前端，先把 `AdminHeader` 头组件中的用户名显示问题弄好，目前的 `Admin` 用户名是写死在代码中的，现在，后台已经提供了接口，就来把这块改造一下，从后台接口中拿数据并显示到 `AdminHeader` 组件中。

![](https://img.quanxiaoha.com/quanxiaoha/169484819572739)

## 添加请求接口 api

编辑 `/api/user.js` 文件，封装一个请求获取登录用户信息接口的方法，方便等会直接调用：

```js
// 获取登录用户信息
export function getUserInfo() {
    return axios.post("/admin/user/info")
}
```

## 二、添加用户 Store

在 `/stores` 目录下，新建一个 `user.js` 用户存放用户相关的全局状态，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169485787987643)

```js
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { getUserInfo } from '@/api/admin/user'

export const useUserStore = defineStore('user', () => {
  // 用户信息
  const userInfo = ref({})

  // 设置用户信息
  function setUserInfo() {
    // 调用后头获取用户信息接口
    getUserInfo().then(res => {
      if (res.success == true) {
        userInfo.value = res.data
      }
    })
  }

  return { userInfo, setUserInfo }
})
```

上述代码中，我们定义了一个用户 `Store`, 在其中，定义了一个 `userInfo` 对象，用于存储后台返回的用户信息。同时，还定义了一个 `setUserInfo()` 方法，方法中调用刚刚写的 `getUserInfo()` 方法，来调用后台接口，获取用户信息，当接口调用成功后，再将用户信息存储到了全局状态中。

## 三、登录成功后，调用设置用户信息到全局状态方法

`Store` 已经开发完成了，那么时候来主动存储用户信息呢？当用户登录成功后，这个时机是比较合适的。编辑 `login.vue` 文件，在登录方法中，添加如下逻辑：

```js
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

const onSubmit = () => {
    console.log('登录')
    // 先验证 form 表单字段
    formRef.value.validate((valid) => {
        // 省略...

        // 调用登录接口
        login(form.username, form.password).then((res) => {
            console.log(res)
            // 判断是否成功
            if (res.success == true) {
                // 省略...
                
                // 存储 Token 到 Cookie 中
                let token = res.data.token
                setToken(token)

                // 获取用户信息，并存储到全局状态中
                userStore.setUserInfo()

                // 省略...
            }
        })
    })
}

```

上述代码中，先是引入了 `useUserStore` , 然后登录成功后，在存储 `Token` 令牌到 `cookie` 代码的后面，主动调用了 `setUserInfo()` 方法，以存储用户信息到全局状态中。

## 四、测试看看

回到页面，重新登录，通过 `vue` 插件，看看全局状态中是否正确存储了用户信息：

![](https://img.quanxiaoha.com/quanxiaoha/169485872405963)

- ①：选择 Pinia ;
- ②：查看 `store` 列表，选择 `user` ；
- ③：可以看到当前登录用户名已经成功被存储了；

## 五、从全局状态中获取登录用户名

现在，我们已经存储了用户信息，只需要从用户 `store` 中拿到它，渲染到页面上即可。编辑 `AdminHeader` 组件，将原先写死的用户名改造一下：

```html
<!-- 登录用户头像 -->  
<el-dropdown class="flex items-center justify-center">  
    <span class="el-dropdown-link flex items-center justify-center text-gray-700 text-xs">  
        <!-- 头像 Avatar -->        <el-avatar class="mr-2" :size="25" src="./src/assets/default_avatar.webp"/>  
        {{userStore.userInfo.username}}  
        <el-icon class="el-icon--right">  
            <arrow-down/>  
        </el-icon>  
    </span>
    // 。。。 省略
                
<script setup>
import { useUserStore } from '@/stores/user'

// 引入了用户 Store
const userStore = useUserStore()

// 省略...
</script>
```

上述代码中，我们先是引入了 `useUserStore` , 然后通过插值语法将用户名渲染到了页面上。

## 七、结语

本小节中，我们将调用获取用户信息接口拿到的用户信息，在用户登录成功后，通过 Pinia 存储到了全局状态中，并在 `AdminHeader` 组件中，获取到用户信息并渲染到页面上。