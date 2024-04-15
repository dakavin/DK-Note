
## 问题1

问题：前端点击登录按钮后，无法自动跳转、报错 status undefined
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ad00b745599e5248f4692e29dc85657.png)

原因：修改了全局返回参数，这个setUserLoginState无法读取信息

解决：打开登录页面，直接删除这两个地方
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23e949578461dc95eb561592818f11eb.png)
