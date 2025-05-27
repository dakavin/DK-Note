---
文章标题: "[[2_token失效，没有退出到登录页面]]" 
文章作者: Dakkk
文章概要: |
  文章解决API token失效未自动退出问题。通过在axios响应拦截器中判断后端返回的特定错误（如'token失效'或401），清除本地token并重定向到登录页，提升用户体验。
文章标签:
- "token失效"
- "axios"
- "响应拦截器"
- "错误处理"
- "用户认证"
- "前端开发"
- "登录退出"
相关文章:
- "[[15_Vue3 数据交互axios]]"
- "[[17_Ajax]]"
- "[[10_博客设置页：更新设置]]"
- "[[10_文章分类：删除功能开发]]"
- "[[11_标签-文章列表页开发]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/05-🐛 Bug处理/2_token失效，没有退出到登录页面.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-16 23:38:20
修改时间: 2024-04-16 23:39:47
---

## 问题

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/bc2986cea41048aa8f997a491c3f6348.png)

token已经失效了，不过没有重置，导致用户需要手动退出

## 思路

一般在axios的请求拦截器中，没有对上面的逻辑进行处理

分析：
![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/efd18610471c871cafc29f5d2bc0e350.png)

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/5e3b0d73133e824e554f93ca3860fe5f.png)

可以看到token失效的时候，返回的内容，我们根据error中的内容进行判断，从而完成退出的功能实现

## 解决


![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d2dbb99ef542c69386a57dc5ada345c6.png)
