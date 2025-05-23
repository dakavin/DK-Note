
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
