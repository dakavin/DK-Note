## 1 什么是vue?

Vue (发音为 /vjuː/，类似 **view**) 是一款用于构建用户界面的 JavaScript **渐进式框架**。它基于标准 HTML、CSS 和 JavaScript 构建，并提供了一套**声明式**的、**响应式**的、**组件化**的编程模型，帮助你高效地开发用户界面。无论是简单还是复杂的界面，Vue 都可以轻松搞定。
## 2 安装Node.js

要想学习 Vue ，首先需要安装好 Vue 的环境。

### 2.1 第一步：安装 Node.js 环境

访问 Node.js 官网：[https://nodejs.org/en](https://nodejs.org/en) ，点击左侧的下载按钮，下载 Node.js LTS 版本的安装包：

> ⚠️ 注意：**学习 Vue 3, 你需要安装 Node.js 16.0 版本或者更高**, LTS 表示该安装包是一个被长期支持的版本，可以理解成是一个稳定版本。

**下载安装：**
- 下载完成后，双击开始安装
- 无脑点击下一步【Next】按钮即可，其中，需要勾选接受协议，以及自选选择安装路径
- 继续点击【Next】按钮, 然后进入安装
- 等待其安装成功
- 然后点击【Finish】按钮，到这里 Node.js 就安装成功了
### 2.2 第二步：验证是否真的安装成功了

按住快捷键 `win + R` 输入 `cmd` 打开命令行，或者使用 PowerShell 等其他命令行工具，执行如下命令：
```shell
node -v
npm -v
```

如果能够正确输出版本号，则表示 Vue 的环境搭建成功：
![20240418_142733_097_copy.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/994dac2ef220a6bd353ecde0f7a8847c.png)

## 3 创建 blog前端工程

打开 `cmd` 命令工具，进入到 `blog` 目录中**或者** 直接window文件管理器打开该目录，然后再目录路径上输出 `cmd` 打开管理工具

然后，执行创建vue项目命令：
```shell
npm create vue@latest
```

> TIPS : 该命令会安装并执行 `create-vue`, 它是 Vue 官方的项目脚手架工具。
> TIPS : 后面的@latest可以去掉，然后选择你想要的vue版本

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/993656ea80ffce5ef55b3b386360ffe5.png)

执行过程中，会提示你命名新项目，我们命名为 `blog-vue3`，以及是否需要开启一些诸如 TypeScript 和测试支持之类的可选功能，这里统一敲击回车键选择 `No` 即可。当你看到命令行中提示 `Done` ， 表示你已经创建好了 `blog` 前端项目。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/04a74cd57fd767c69918137df1e9dfe3.png)

---

**项目目录说明**
创建好了 `weblog` 前端工程后，进入该文件夹，看下目录结构：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/02e0bd3a434493804327bd34d42f8a0a.png)
解释一下目录结构以及相关文件的作用：
- `node_modules` : 项目依赖包文件夹，比如通过 `npm install 包名` 安装的包都会放在这个目录下，_因为现在还没有执行 npm install 命令，所以还未生成该文件夹_；
- `public` : 公共资源目录，用于存放公共资源，如 `favicon.ico` 图标等；
- `index.html` : 首页；
- `package.json` : 项目描述以及依赖；
- `package-lock.json` : 版本管理使用的文件；
- `README.md` : 用于项目描述的 markdown 文档；
- `src` : 核心文件目录，源码都放在这里面；

进入 `src` 文件夹，目录如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac5ee5542b4940b70dc9774f7ed7c42b.png)

- `assets` : 静态资源目录，用于存放样式、图片、字体等；
- `components`: 组件文件夹，通用的组件存放目录；
- `App.vue`: 主组件，也是页面的入口文件，所有页面都是在 App.vue 下进行路由切换的；
- `main.js` : 入口 Javascript 文件；

## 4 启动项目

进入到想要启动的项目文件夹中，执行如下命令，为项目安装依赖并启动项目：
```shell
# 进入项目文件夹
cd weblog-vue3
# 安装项目所需依赖
npm install
# 启动项目
npm run dev
```

启动成功后，会提示项目的访问地址，如 `http://localhost:5173/`:

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29f082994cf97d0ed570649487516a8e.png)

在浏览器地址栏中访问该地址，即可访问此 Vue 项目啦，整个过程还是非常简单的。
![20240418_151946_398_copy.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a2d7572ba7e9982b53dd1530dd2138ee.png)

## 5 结语

本小节中，我们正式进入了 `weblog` 的前端开发领域，了解了 什么是 `Vue`, 以及安装好了 `Vue 3` 所需的环境，最后，我们搭建了 `weblog` 前端项目，并启动了它，也能够在浏览器中正常访问该项目。