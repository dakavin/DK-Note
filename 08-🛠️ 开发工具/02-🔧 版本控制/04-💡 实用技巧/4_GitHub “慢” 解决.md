---
文章标题: "[[4_GitHub “慢” 解决]]"
文章作者: Dakkk
文章概要: |
  这篇文章详细介绍了通过修改本地 hosts 文件来解决 GitHub 访问速度慢和图片加载困难的问题。它提供了多系统下的手动配置步骤，并推荐使用 SwitchHosts 和 AdGuard Home 等工具实现自动化管理和更新，旨在显著提升 GitHub 使用体验。
tags:
  - GitHub加速
  - hosts文件
  - DNS解析
  - 网络优化
  - SwitchHosts
  - AdGuard Home
  - 访问速度
  - 教程
相关文章:
  - "[[2_Pipeline]]"
  - "[[../../../01-📚 知识管理/06-🛠️ 技巧/3_特殊软件技巧/2_gitkralen的破解及汉化]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/02-🔧 版本控制/04-💡 实用技巧/4_GitHub “慢” 解决.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 1 介绍

对 GitHub 说"爱"太难了：访问慢、图片加载不出来。

_注：_ 本项目还处于测试阶段，仅在本机测试通过，如有问题欢迎提 [issues](https://github.com/521xueweihan/GitHub520/issues/new)

---

本项目无需安装任何程序，通过修改本地 hosts 文件，试图解决：

- GitHub 访问速度慢的问题
- GitHub 项目中的图片显示不出的问题

花 5 分钟时间，让你"爱"上 GitHub。
## 2 使用方法

### 2.1 手动方式

**复制下面的代码**
```
# GitHub520 Host Start 185.199.108.154 github.githubassets.com 140.82.113.22 central.github.com 185.199.108.133 desktop.githubusercontent.com 185.199.108.153 assets-cdn.github.com 185.199.108.133 camo.githubusercontent.com 185.199.108.133 github.map.fastly.net 199.232.69.194 github.global.ssl.fastly.net 140.82.113.3 gist.github.com 185.199.108.153 github.io 140.82.114.4 github.com 140.82.112.6 api.github.com 185.199.108.133 raw.githubusercontent.com 185.199.108.133 user-images.githubusercontent.com 185.199.108.133 favicons.githubusercontent.com 185.199.108.133 avatars5.githubusercontent.com 185.199.108.133 avatars4.githubusercontent.com 185.199.108.133 avatars3.githubusercontent.com 185.199.108.133 avatars2.githubusercontent.com 185.199.108.133 avatars1.githubusercontent.com 185.199.108.133 avatars0.githubusercontent.com 185.199.108.133 avatars.githubusercontent.com 140.82.113.9 codeload.github.com 52.217.88.28 github-cloud.s3.amazonaws.com 52.216.238.99 github-com.s3.amazonaws.com 52.216.26.252 github-production-release-asset-2e65be.s3.amazonaws.com 52.217.101.68 github-production-user-asset-6210df.s3.amazonaws.com 52.217.48.84 github-production-repository-file-5c1aeb.s3.amazonaws.com 185.199.108.153 githubstatus.com 64.71.168.201 github.community 185.199.108.133 media.githubusercontent.com # Update time: 2021-03-16T12:24:16+08:00 # Star me GitHub url: https://github.com/521xueweihan/GitHub520 # GitHub520 Host End
```

上面内容会自动定时更新，保证最新有效。

也可以自行手动搜索这三个网站的dns，然后保存在hosts文件中
- 搜索网站：[IP Tracer & Tracker - IP Address Lookup Made Easy](https://www.ipaddress.com/ip-lookup)
- 搜索网站：https://tool.chinaz.com/
- github.com
- github.global.ssl.fastly.net
- assets-cdn.github.com

**修改hosts文件**

hosts 文件在每个系统的位置不一，详情如下：

- Windows 系统：`C:\Windows\System32\drivers\etc\hosts`
- Linux 系统：`/etc/hosts`
- Mac（苹果电脑）系统：`/etc/hosts`
- Android（安卓）系统：`/system/etc/hosts`
- iPhone（iOS）系统：`/etc/hosts`

修改方法，把第一步的内容复制到文本末尾：

1. Windows 使用记事本。
2. Linux、Mac 使用 Root 权限：`sudo vi /etc/hosts`。
3. iPhone、iPad 须越狱、Android 必须要 root。

**激活生效**

大部分情况下是直接生效，如未生效可尝试下面的办法，刷新 DNS：

1. Windows：在 CMD 窗口输入：`ipconfig /flushdns`
    
2. Linux 命令：`sudo rcnscd restart`
    
3. Mac 命令：`sudo killall -HUP mDNSResponder`
    

**Tips：** 上述方法无效可以尝试重启机器。
### 2.2 自动方式

**Tip**：推荐 [SwitchHosts](https://github.com/oldj/SwitchHosts) 工具管理 hosts

以 SwitchHosts 为例，看一下怎么使用的，配置参考下面：

- Title: 随意
    
- Type: `Remote`
    
- URL: `https://cdn.jsdelivr.net/gh/521xueweihan/GitHub520@main/hosts`
    
- Auto Refresh: 最好选 `1 hour`
    

如图：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8dfea66fdc4959b347884c2671d1e495.png)
这样每次 hosts 有更新都能及时进行更新，免去手动更新。

### 2.3 AdGuard Home 用户（自动方式）

在 **过滤器>DNS 封锁清单>添加阻止列表>添加一个自定义列表**，配置如下：

- 名称: 随意
    
- URL: `https://cdn.jsdelivr.net/gh/521xueweihan/GitHub520@main/hosts`（和上面 SwitchHosts 使用的一样）
    

如图：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5426a6cdc92bdff1228770e48237c677.png)
更新间隔在 **设置>常规设置>过滤器更新间隔（设置一小时一次即可）**，记得勾选上 **使用过滤器和 Hosts 文件以拦截指定域名**![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e15caa7aaf63d6e842d99af762e7da78.png)
**Tip**：不要添加在 **DNS 允许清单** 内，只能添加在 **DNS 封锁清单** 才管用。另外，AdGuard for Mac、AdGuard for Windows、AdGuard for Android、AdGuard for IOS 等等 **AdGuard 家族软件** 添加方法均类似。
## 3 效果对比

之前的样子：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8344e75dc2634dc88538542c61d9fd78.png)
修改完 hosts 的样子：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8483b59385f59aacd7dde8a3dbe4d20d.png)

## 4 补充

如果代理出现问题，导致无法`push`

请在gitbash写上如下代码：
```shell
git config --global --unset http.proxy 
git config --global --unset https.proxy
```

注意把自己的代理软件也都关闭，然后重新打开下项目（不重新打开项目有时候会出现编辑器配置修改了没有生效的问题）。  
如果不行的话尝试下刷新DNS缓存，用cmd打开[windows命令](https://so.csdn.net/so/search?q=windows%E5%91%BD%E4%BB%A4&spm=1001.2101.3001.7020)窗口，输入
```shell
ipconfig/flushdns
```