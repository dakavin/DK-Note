
```js
import {useRoute, useRouter} from "vue-router"  
import {ref,watch } from 'vue'  

// 用于获取关于当前路由的信息，如当前的路径、查询参数等  
const route = useRoute()  
// 路由对象，使用它可以实现页面的跳转  
const router = useRouter()

// 根据路由地址判断那个菜单被选中，从而保持常亮  
const defaultActive = ref(route.path)  
  
// 监听路由变化，避免按下了tab，但是菜单栏没有变化  
watch(()=>route.path,(newPath)=>{  
    defaultActive.value = newPath  
})
```