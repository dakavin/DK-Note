---
文章标题: "[[7_挂着Clash的时候git操作push失败的问题]]" 
文章作者: Dakkk
文章概要: |
  文章解决了开启Clash代理时Git操作失败的问题，指出原因为Git未配置代理导致端口受阻。解决方案是为Git全局设置http和https代理到Clash的本地地址和端口。文章也提供了取消和查看Git代理配置的命令，帮助用户快速解决网络连接问题。
tags:
- "Git"
- "Clash"
- "代理"
- "网络配置"
- "故障排除"
- "git push"
- "端口443"
相关文章:
- "[[5_Win11开启Rndis usb共享网络驱动]]"
- "[[1_本章代码及文章获取]]"
- "[[1_开发环境搭建]]"
- "[[1_快速开始]]"
- "[[1_使用editing-toolbar插件卡顿]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/02-🔧 版本控制/04-💡 实用技巧/7_挂着Clash的时候git操作push失败的问题.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 16:21:00
修改时间: 2025-05-28 01:03:09
---

当开启[Clash](https://so.csdn.net/so/search?q=Clash&spm=1001.2101.3001.7020)后，本机网络会被代理，此时可以在 设置-网络-代理 中看到：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/87046c6fd5d24f8fe7b974de66047b5d.png)

git push 失败的原因就是本机开启了代理，而git没有设置代理，导致[443端口](https://so.csdn.net/so/search?q=443%E7%AB%AF%E5%8F%A3&spm=1001.2101.3001.7020)转发不过去，此时只需要设置以下git的代理即可解决
```shell
git config --global http.proxy http://127.0.0.1:7890 
git config --global https.proxy http://127.0.0.1:7890
```

取消和查看代理
```shell
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看代理
git config --global --get http.proxy
git config --global --get https.proxy
git config --list

```