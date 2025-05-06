
## 1 问题一：

**创建简单的MVC项目的时候，出现以下图片的问题**

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34da4c4402a5773be892a0ff4bb09a85.png)

解决：看自己的sdk版本，jdk版本是8，因此不能用spring6.x开头，只能用spring5以下，修改spring依赖版本后可以正常使用