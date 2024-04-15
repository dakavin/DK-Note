
## 问题1


问题：前端完成获取后端用户信息的方法后，开始用户登录，显示登录成功了，但是页面不跳转到主页

原因：返回的currentUser包装在data数据里面，导致全局的initialState，读取不了currentUser，就给判空，所以走进那个重定向到登录页的逻辑
![img_v3_029t_3152223f-ad5e-49c6-928e-a5ec1b20c00g.jpg|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f4912b3177b7dae69e1ea444074c7cf.jpg)

解决：取消封装
![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59eda99e7f4f012f11e7911d210ceefc.png)