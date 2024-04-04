
引用大佬文章：[reCaptcha人机验证无法显示和CSP问题解决方案 – Azure Zeng Blog](https://blog.azurezeng.com/recaptcha-use-in-china/)

## 问题

Your reCAPTCHA response did not validate. Please try again
## 科普

reCaptcha是Google公司的验证码服务，方便快捷，改变了传统验证码需要输入n位失真字符的特点。reCaptcha在使用的时候是这样的：
![20201017201846476.gif|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4bf5c8127eb31a159394c912c0cd6767.gif)


只需要点一下复选框，Google会收集一些鼠标轨迹、网络信息、浏览器信息等等，依靠后端的神经网络判断是机器还是人，绝大多数验证会一键通过，无需像传统验证码一样。但是reCaptcha使用了google.com的域名，这个域名在国内是被墙的。所以，也就导致了官网上无法注册/登录的问题，挡住了不少人入正的脚步。

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/045c851ede94f0504c8ff97b415ac087.png)

## 解决办法

### 插件安装

由于基于 Chromium 的浏览器太多了，此处只提 Microsoft Edge (Chromium) / Chrome 的安装方法。

Microsoft Edge (Chromium)：[Microsft Edge Add-ons](https://microsoftedge.microsoft.com/addons/detail/header-editor/afopnekiinpekooejpchnkgfffaeceko)
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2042ca6ebfd43db66b3f17f894e5571b.png)

- Edge直接安装即可

Chrome (Microsoft Edge 也可使用此安装途径)：[Chrome Web Store (需要魔法工具)](https://chrome.google.com/webstore/detail/header-editor/eningockdidmgiojffjmkdblpjocbhgh)

Chorme 通过本文提供的离线文件手动安装:

将你下载到的离线安装文件解压出来。解压后，你应该可以看到一个名字为 Header Editor.crx 的文件。

之后，打开 Chrome，进入扩展程序管理页面。
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a423c63158042d5521b5a9a9b11f76f0.png)

将你解压的 Header Editor.crx 拖到里面来。记得在拖动之前打开右上角的 “开发者模式”。
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a1af1b89f14a104b5abe5502782f3784.png)

若出现这个对话框即代表可以正常安装。点击 “添加扩展程序” 即可。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2ffeabad0e76ed4d666ed016319048e.png)
### 配置插件

打开 Header Editor 插件的配置页面，选择 “导入和导出” 选项。`点击即可`
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d10f847fe7cca5c422e0ef7a542452b5.png)

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f110719e520bfeee5bc49c83da97711.png)

导入在线配置：

在下载规则中，填入下面的地址 (任选其一，推荐使用 GitHub 版本):

- (GitHub，**推荐**) [https://github.azurezeng.com/static/HE-GoogleRedirect.json](https://github.azurezeng.com/static/HE-GoogleRedirect.json)
- (本站服务器) [https://www.azurezeng.com/static/HE-GoogleRedirect.json](https://www.azurezeng.com/static/HE-GoogleRedirect.json)

**_重要提醒：建议使用 GitHub 地址。本站服务器地址在站点维护时可能无法使用。_**

然后点击下载按钮。

如果先前导入过，你应该可以在下载规则中直接找到这个地址，直接点击旁边的下载按钮即可。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/91a66b002c7e7c21749b129d7a8b3668.png)
接下来你应该会在 “导入” 看到相关规则 (如果之前导入过，“操作” 中的 “添加” 会显示为 “覆盖已有”)。选择 “保存” 即可。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59fb13393a1a2680d1a8220028b820d5.png)
最后你的规则列表应该是这样的:![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d1519759867dce9da191b470c555b1e.png)
好了，关闭这个页面。然后就可以了，现在 reCaptcha 应该可以正常显示了。
