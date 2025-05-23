
为了更高效率的开发 Vue 3 项目，我们需要有个趁手的兵器，也就是开发工具。比较常见的如 VSCode 、Webstorm 等，但是官方推荐使用 VSCode， 那我们就通过 VSCode 来开发 Vue 3。
## 1 VSCode简介

VSCode 全称 Visual Studio Code，是微软出的一款轻量级代码编辑器，它具有如下特点：
- 开源且免费；
- 代码智能提示、自动补全功能；
- 可自定义配置；
- 支持丰富的文件格式；
- 代码调试功能强大；
- 各种方便的快捷键；
- 强大的插件拓展功能；
## 2 下载安装包

前往 VSCode 官网：[https://code.visualstudio.com/](https://code.visualstudio.com/) 下载对应系统的安装包，这里用 Windows 系统来演示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3641e85480e2a29cc4e5b2c0da3d47df.png)

## 3 开始安装

- 下载成功后，双击安装包开始安装 VSCode
- 勾选【我同意此协议】，点击下一步按钮
- 自定义安装路径，这里安装在了 `D` 盘，可自行选择安装位置，继续点击下一步按钮
- 继续点击下一步按钮
- 勾选【创建桌面快捷方式】，点击下一步：
- 点击【安装】
- 等待一分钟左右，即可安装成功，然后点击【完成】按钮
## 4 使用界面

启动成功后，即可看到如下界面，至此，VSCode 就安装成功啦~
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce775a086f2e01139e2cf3af04b952da.png)

## 5 VSCdoe设置中文

> TIP: **汉化是可选项**，针对初学者来说，全英文化的 VSCode 可能不太友好，所以，根据自己的需要来确定是否需要汉化。

在 VSCode 的左侧栏，可以看到插件市场选项，如下图所示：
![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/86f0dbc03808156e42fafa90da9f81da.png)

打开插件市场，搜索关键词【中文】，即可看到中文汉化插件，点击【Install】安装：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20d1581ef07ea0e62ff52c50ec20c16c.png)

安装成功后，右下角会提示是否需要立刻重启 VSCode 来使汉化生效，点击重启：

---

**汉化后的界面**

重启 VSCode 后，你就可以看到所有菜单均已被设置成中文了：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9fd2895af0eca74737d6e6d9d54733f2.png)
## 6 安装Vue相关插件

想要在 VSCode 中开发 Vue 3 项目，还需要安装几个必备插件。

- Vue Language Features (Volar) 插件
	- 一款专用于构建 Vue 的拓展，想要在 VSCode 上开发 Vue 3 应用，这款插件必不可少。
- Vue 3 Snippets插件
	- HTML 代码自动提示和代码补全插件，提升编码效率。
- 别名路径跳转
	- 别名路径跳转插件，支持任何项目，可以自由配置映射规则，自由配置可缺省后缀名列表
- Auto Rename Tag
	- 当我们修改 HTML/XML 某个节点标签时，能够自动重命名结束标签的命名。
- Auto Close Tag
	- 自动添加 HTML/XML 结束标签，比如当输入 `<p>` 标签，则自动添加 `</p>` 结尾标签。
- CSS Navigation
	- 允许你通过点击 `HTML` 中的类名，直接跳转至对应的样式代码。
- Path Intellisense
	- 当你通过 `.` 的方式导入某个文件时，可自动提示文件路径。
- Moonlight主题插件(可选)
	- 原生的 VSCode 主题样式看腻了？不妨安装一下此主题插件，效果图如下：
- IntelliJ IDEA Keybindings 插件 (可选)
	- 将 VSCode 默认的快捷键变更为 IntellJ IDEA 、Webstorm、PyCharm、PHP Storm 等 IDE 的快捷键。


## 7 通过VSCode打开Weblog项目

前面小节中已经通过命令行创建了第一个 Vue 应用，现在，我们将通过 VSCode 来打开它。

---

**打开项目**

点击 VSCode 左上角菜单：_文件 -> 打开文件夹_，导入之前创建好的 `blog-frontend` 项目：

> TIP : 或者你也可以将项目文件夹直接拖入 VSCode 来打开项目。

![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/11aaca9f8abb02bb42d70d0c9316ba57.png)

导入成功后，视图如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2957ab43ee42f3f03937fd8709a6643f.png)

---

**核心文件说明**

在前面创建项目小节中，我们已经了解了各个文件夹，以及文件的大致用途。这里小哈针对最核心的 3 个文件再详细说明一下，分别是：

- `index.html` ： 首页；
- `main.js` ： 主 js 文件；
- `App.vue` : 主组件；

这 3 者之间的关系如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cea3491c5012e4c5569f0087e5a53652.png)

当打开一个 Vue 3 应用，首先先看 `index.html` 文件，它是首页，代码如下，这里已经添加好注释说明：
![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cea6039563abecffa95fc93790caab6a.png)

再来看 `main.js` 文件：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/80cb5c14d139107dd637fb705f9c3463.png)

再看 `app.vue` 组件代码：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c912156da0bca76cf3cfbc09976e614a.png)

为了更方便的学习，我们先将多余的代码删除掉，只保留结构，如下图所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5ef7b8f8bc1da813a303f57569e51089.png)

结构分为 3 个部分：
- `script` : 节点中间用于放置 `javascript` 代码；
- `template` : 节点中间用于放置 `html` 代码；
- `style` : 节点中间用于放置 `css` 样式代码；

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6373848be62f2db471f6ecb55f62a241.png)


然后，我们在 `<template>` 标签下添加一个 `<h1>` 标题：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a0a4acd542422a9664ed251f169a3e7.png)

打开 `main.js` 文件，注释掉模板工程的自带的样式文件 `main.css`：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3c768525061b8b7a3e72e12a4e322d2d.png)

**保存代码**, 并直接通过 WenStrom 打开命令行，来执行启动命令 `npm run dev`：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c161a8cc2642a3b41867949b2709b953.png)

启动成功后，浏览器访问 `http://localhost:5173` ，效果如下:
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ffc7d9fc41b47ef072a3d845305f856.png)
## 8 结语

本小节中，我们安装好了前端的开发工具 VSCode ，并打开了之前创建好的 `weblog-vue3` 项目，同时了解到项目中几个核心文件的作用，最后，在 `App.vue` 中添加了 `<h1>` 标签，重启了项目，看到了修改内容已生效。