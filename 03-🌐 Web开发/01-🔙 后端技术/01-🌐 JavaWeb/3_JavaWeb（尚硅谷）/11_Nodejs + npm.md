## 1 nodejs 简介和安装

### 1.1 什么是Nodejs

- `像我们之前学习JS，都是在HTML文件中编写，然后通过浏览器解析HTML文件来运行JS代码的`
- 除了浏览器外，我们还可以使用nodejs软件，运行js代码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3ed52ec0325404f2d6ce061ac3ee13bb.png)

> Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时环境，可以使 JavaScript 运行在服务器端。使用 Node.js，可以方便地开发服务器端应用程序，如 Web 应用、API、后端服务，还可以通过 Node.js 构建命令行工具等。相比于传统的服务器端语言（如 PHP、Java、Python 等），Node.js 具有以下特点：

-   `单线程`，但是采用了事件驱动、异步 I/O 模型，可以处理高并发请求。
-   `轻量级`，使用 C++ 编写的 V8 引擎让 Node.js 的运行速度很快。
-   `模块化`，Node.js 内置了大量模块，同时也可以通过第三方模块扩展功能。
-   `跨平台`，可以在 Windows、Linux、Mac 等多种平台下运行。

> `Node.js 的核心是其管理事件和异步 I/O 的能力`。Node.js 的异步 I/O 使其能够处理大量并发请求，并且能够避免在等待 I/O 资源时造成的阻塞。此外，Node.js 还拥有高性能网络库和文件系统库，可用于搭建 WebSocket 服务器、上传文件等。`在 Node.js 中，我们可以使用 JavaScript 来编写服务器端程序，这也使得前端开发人员可以利用自己已经熟悉的技能来开发服务器端程序，同时也让 JavaScript 成为一种全栈语言。`Node.js 受到了广泛的应用，包括了大型企业级应用、云计算、物联网、游戏开发等领域。常用的 Node.js 框架包括 Express、Koa、Egg.js 等，它们能够显著提高开发效率和代码质量。

![|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/17cbfd49fb1a9df19d3676ba5051f799.png)
### 1.2 如何安装Nodejs

1.  打开官网https://nodejs.org/en下载对应操作系统的 LTS 版本。
2.  双击安装包进行安装，安装过程中遵循默认选项即可(或者参照https://www.runoob.com/nodejs/nodejs-install-setup.html )。安装完成后，可以在命令行终端输入 `node -v` 和 `npm -v` 查看 Node.js 和 npm 的版本号。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c67b8cfc2ec4487b87adb6e1164b413.png)
3. `node.js版本差异比较大，不建议安装 v18之后的版本，本次使用v20.11.1`
4. 定义一个app.js文件,cmd到该文件所在目录,然后在dos上通过`node app.js`命令即可运行
```js
function sum(a,b){
    return a+b;
}
function main(){
    console.log(sum(10,20))
}
main()
```
## 2 npm 配置和使用

### 2.1 npm介绍

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cad094fd728b6124886d31308135c71a.png)

> NPM全称Node Package Manager，是Node.js包管理工具，是全球最大的模块生态系统，里面所有的模块都是开源免费的；也是Node.js的包管理工具，相当于后端的Maven 

- `从功能上对比Maven，npm不能帮我们构建项目，结合vite才能构建项目`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e02af38b0a5f7f6a97a36de0abb038a1.png)

- npm软件
	- 前端框架的下载工具
	- 前端项目的管理工具

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0fbf27215e22894281311d2df56e2a5d.png)
### 2.2 npm 安装和配置

> 1.安装

+ 安装node，自动安装npm包管理工具！


> 2.`配置依赖下载使用阿里镜像`

+ npm 安装依赖包时默认使用的是官方源，由于国内网络环境的原因，有时会出现下载速度过慢的情况。为了解决这个问题，可以配置使用阿里镜像来加速 npm 的下载速度，具体操作如下：
+ 打开命令行终端，执行以下命令，配置使用阿里镜像：
+ `原来的 registry.npm.taobao.org 已替换为 registry.npmmirror.com`

``` shell
npm config set registry https://registry.npmmirror.com
```

+ 确认配置已生效，可以使用以下命令查看当前 registry 的配置：如果输出结果为 `https://registry.npmmirror.com`，说明配置已成功生效。

```shell
npm config get registry
```

+ 如果需要恢复默认的官方源，可以执行以下命令：

```javascript
npm config set registry https://registry.npmjs.org/
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f39bfd91f14b34c1a20844613fe8be07.png)

> 3.`配置全局依赖下载后存储位置`

+ 在 Windows 系统上，npm 的全局依赖默认安装在 `<用户目录>\AppData\Roaming\npm` 目录下。

+ 如果需要修改全局依赖的安装路径，可以按照以下步骤操作：

  1. 创建一个新的全局依赖存储目录，例如 `E:\Nodejs\node_global

  2. 打开命令行终端，执行以下命令来配置新的全局依赖存储路径：

``` shell
npm config set prefix "E:\Nodejs\node_global"
```

  3. 确认配置已生效，可以使用以下命令查看当前的全局依赖存储路径：

``` shell
npm config get prefix
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ee4ccbcce1ff7f3e5d5aa651788d35a.png)


> 4.`配置全局缓存位置`

+ 如果需要修改全局缓存路径，可以按照以下步骤操作：

  1. 创建一个新的全局依赖存储目录，例如 `E:\Nodejs\node_cache

  2. 打开命令行终端，执行以下命令来配置新的全局依赖存储路径：

``` shell
npm config set cache "E:\Nodejs\node_cache"
```

  3. 确认配置已生效，可以使用以下命令查看当前的全局依赖存储路径：

``` shell
npm config get cache
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/107103179cd19ff2ae6f4851768410aa.png)

> 4. `设置node_global全局变量`

+ “此电脑”-->右键“属性”-->“高级系统设置”-->“高级”-->“环境变量”![image.png|400|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c11c4c72d901860e11f46640fdb00c08.png)

- 在用户变量中的PATH中（`重要！！！`）：
	
	- 将我们创建的`node_global`文件夹添加即可![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/45b63ff824ca9e51650964e697cf7bf5.png)

> 5.`升级npm版本`

+ cmd 输入npm -v 查看版本（`管理员运行`）

+ 如果node中自带的npm版本过低！则需要升级至10.2.4！

```shell
npm install -g npm@10.2.4
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5988f613a6afa179299f65cd878df1f.png)

### 2.3 npm 常用命令

> 1.`项目初始化`

+ `npm init`
	+ 进入一个vscode创建好的项目中, 执行 npm init 命令后，npm 会引导您在命令行界面上回答一些问题,例如项目名称、版本号、作者、许可证等信息，并`最终生成一个package.json 文件。package.json信息会包含项目基本信息！类似maven的pom.xml`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a44e5727ee2f42e44438eb4a4b9aee99.png)

+ `npm init -y`
	+ 执行，-y yes的意思，所有信息使用当前文件夹的默认值！不用挨个填写！

- 如果删除了创建的json文件，可以重新init

> 2.`安装依赖  (查看所有依赖地址  https://www.npmjs.com )`

+ `npm install` 包名 或者 `npm install 包名@版本号`
	+ 安装包或者指定版本的依赖包(安装到当前项目中)![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/51020d5017fb9161dd2fc95c25ba8fff.png)

+ `npm install -g 包名`
	+ 安装全局依赖包(安装到d:/GlobalNodeModules)则可以在任何项目中使用它，而无需在每个项目中独立安装该包。
	+ `建议不要使用全局依赖，给项目单独放依赖就好了`

+ `npm install` 或 `npm i`
	+ 安装package.json中的所有记录的依赖

> 3.`升级依赖`

+ `npm update 包名`
	+ 将依赖升级到最新版本

> 4.`卸载依赖`

+ `npm uninstall 包名`

> 5.`查看依赖`

+ `npm ls`
	+ 查看项目依赖

+ `npm list -g`
	+ 查看全局依赖

> 6.`运行命令`

+ `npm run` 命令是在执行 npm 脚本时使用的命令。npm 脚本是一组在 package.json 文件中定义的可执行命令。npm 脚本可用于启动应用程序，运行测试，生成文档等，还可以自定义命令以及配置需要运行的脚本。

+ 在 package.json 文件中，scripts 字段是一个对象，其中包含一组键值对，键是要运行的脚本的名称，值是要执行的命令。例如，以下是一个简单的 package.json 文件：

```json
{
	"name": "my-app",
  	"version": "1.0.0",
    "scripts": {
        "start": "node index.js",
        "test": "jest",
        "build": "webpack"
    },
    "dependencies": {
        "express": "^4.17.1",
        "jest": "^27.1.0",
        "webpack": "^5.39.0"
    }
}
```

+ scripts 对象包含 start、test 和 build 三个脚本。
	+ 当您运行 npm run start 时，将运行 node index.js，并启动应用程序。
	+ 同样，运行 npm run test 时，将运行 Jest 测试套件
	+ npm run build 将运行 webpack 命令以生成最终的构建输出。

+ 总之，npm run 命令为您提供了一种在 package.json 文件中定义和管理一组指令的方法，可以在项目中快速且灵活地运行各种操作。

- 补充：
- 我们可以在vscode，进入文件的终端![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c5aa5b0a61e5438553034c5ad1f04f0.png)
- 如果代码运行错误，可以先管理员运行vscode，然后再正常打开vscode即可