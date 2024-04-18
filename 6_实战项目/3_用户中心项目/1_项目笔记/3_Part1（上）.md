
## 1 项目简介和计划

**用户中心**，即用户管理系统，老生常谈的话题，任何的系统都会有登录的环境，有登录就有用户，我们需要将用户的信息保存下来。

之后我们做的项目也需要用到这个项目中心，然后我们就不需要重新构建用户的一些操作逻辑了！

现在是简单的功能
- 支持用户登录，和用户自己的操作
- 支持管理员登录，和管理员对用户的管理

通过这个项目，我们可以完整了解做项目的思路，接触一些企业级的开发技术，**尽量少写代码，多注重构建项目的思路**

目标：之后可以轻松做出管理系统！！！
## 2 企业做项目完整流程介绍

需求分析 ➡ 设计 ➡ 技术选型 ➡ 初始化项目 ➡ 写一个小Demo ➡ 写代码 ➡ 测试 ➡ 代码提交 ➡ 部署 ➡ 发布

- **需求分析**：需求是否合理，能否带来效益
- **设计**：想一下这个需求如何设计（库表设计，代码如何写）
- **技术选型**：用什么技术完成这个项目？代码语言，语言的版本？
- **初始化项目（引入需要的技术）**
- **写一个小Demo**：测试引入的组件是否可以实现，没有报错
- **写代码**：实现业务逻辑
- **测试**：单元测试等（不要把一些提前可以发现的问题，拖到最后）
- **代码提交**：将代码发布到所有人可以看到的远程仓库，同事看一下代码，**代码评审**
- **部署**：将项目放到服务器、容器等工具上
- **发布**：发布的话，根据服务器或容器选择！
## 3 需求分析

原因：之后我们可能有很多个系统，不可能每个系统都开发独立的用户中心，例如我们一个微信账号可以登录很多个平台。所以我们可以把集中管理的东西，抽取出来。

**那么用户中心需要有那些功能呢？**
1. **登录 / 注册**
2. **用户管理（仅管理员可见**）：对用户的查询和修改等功能
	- 我们项目中就分为用户和管理员即可，因为不像企业那样有那么多的人，分为不同的角色，所以我们==暂时不需要用户权限（后续根据实际情况去优化即可）“先完成再优化”==
3. **用户校验**：仅那些用户可以加入，例如仅星球用户可以加入
## 4 技术选型（各技术作用讲解）

前端：主要运用阿里Ant Design生态：
- HTML+CSS+JavaScript三件套
- React开发框架
- Ant Design Pro项目模板（现成的管理系统）
- Ant Design端组件库
- Umi开发框架（Alibaba）
- Umi Request请求库
- 正向和反向代理

后端：
- Java编程语言
- Spring+SpringMVC+SpringBoot框架
- MyBatis+MyBatis Plus数据访问框架
- MySQL数据库
- JUnit单元测试库

部署：
- Linux单机部署
- Nginx Web服务器
- Docker容器
- 容器托管平台

## 5 前端初始化

### 5.1 nodejs的安装

如果是没接触过前端的伙伴，可以先去nodejs官网，下载一下nodejs

![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b598e16c5b09e541826bf3a1eb9ade80.png)

**注意使用nodejs需要一下操作**
- 设置本地全局依赖目录和本地全局缓存目录，并设置到path去
- 给node和npm管理员权限
### 5.2 Ant Design Pro框架引入

[Ant Design Pro框架](https://pro.ant.design/zh-CN/docs/getting-started/)我们把这个当成一个 **开箱即用的管理系统即可**

我们开始**按照官网文档给出的开始使用**，开始用一下这个框架
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d1d6b4bd49fe29ae08e7da919e70adc1.png)

找一个目录来安装这个前端项目，按照官方说明依次输入指令即可
- `问题一`：详见[问题1](0_bug处理/1_Part1-BUG.md#问题1)

然后使用Wsrstrom软件打开项目，打开项目后使用`npm i`安装项目所需的依赖
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/68304e273af6c019767294e0eee45377.png)

等待依赖安装完毕后，我们启动package.json中的脚本start，使用命令`npm start`（或者直接点击start旁边的按钮）运行一下我们这个前端模版项目，启动成功后如图所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8d9b0890c37edae634f3ddba4bdf19c.png)
- 补充：`start dev`这个启动脚本，`MOCK=none`是无法进入后台的
- `问题二：`详见[问题2](0_bug处理/1_Part1-BUG.md#问题2)

直接登录，账户密码都在填写框内给出提示了
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ddfce54affb43cb20e9ec1ed5c3e5a5f.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/713cd4fd591b5f412ae29e941e1bf320.png)

小米饭插件的构建：安装umi ui ，在命令行输入指令，**保证是在umi@3项目中 （重要的事情说三遍文件夹myapp）**
```text
npm i --save-dev @umijs/preset-ui -D
```
- 可以使用魔法加快速度，不然速度会有点慢
- 此时我们可以看到小米饭插件了
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c681b743cbec57d9eb2132ebdd0d8ead.png)
### 5.3 框架瘦身

**第一步：移除国际化的代码**

- 国际化代码的位置：`src/locales`

- 移除国际化代码的办法：使用脚本中的`i18n-remove`这个脚本
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a2eead39f6211c2dee9f84fe12307898.png)
- 执行脚本的代码展示：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9141b8ea354ddf8c80e3f064bc13e3a.png)

**第二步：快速过一遍目录**
- **注意：删除一个文件跑一遍，看看是否影响主要功能**

- `config`：前端项目的一些配置
	- `oneapi.json`：定义项目用到的接口，==直接删掉！==

- `dist`：部署项目的时候，已经编译好的项目目录

- `mock`：提供前端展示使用的模拟数据

- `node_modules`：项目所需要的依赖

- `public`：项目所需要的一些静态资源

- `src`：写代码的目录
	- `components`：页面使用到的组件
	- `pages`：项目的各个页面
		- 删除页面的时候，在`config/routes.ts`文件中检查一下，是否用到了，然后一块删除
	- `locales`：国际化所使用的各个语言，==执行脚本后删掉！==
	- `e2e`：一个集成测试，定义了一些列的测试流程，==直接删掉！==
	- `services`：一些服务使用的组件
		- `swagger`：一些接口文档根据，==直接删掉！==后续会讲
		- `app.tsx`：前端项目的入口
		- `global.less`：全局页面引用的样式，类似于css，==不要动！==
		- `global.tsx`：全局的脚本文件，类似于js，==不要动！==
		- `service-work.js`：前端的页面缓存，==先不要理解==
		- `typings.d.ts`：定义了一些ts的类型，类似于c++的宏定义，==先不要理解==

- `tests`：测试文件，一般只有在企业很大的项目才会去测试，==直接删掉！==

- `.editorconfig`：代码编辑器的配置，格式化代码
- `.eslintignore`：
- `.eslintrc.js`：检查js语法是否规范
- `.prettierrc.js`：美化代码的工具
- `jest.config.js`：测试工具，==直接删掉！==
- `.stylelintrc.js`：检查css语法是否规范
- `playwright.config.ts`：也是测试模拟根据，==直接删掉！==

- 其实使用WSR软件的时候，删除文件时，会告诉你哪里会有联系，看一下顺带删了就好

- 没有顺带删除的结果如图：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d24d54b57ee283bd3165fa4f40cc1240.png)

- 使用全局搜索`(ctrl + n)`，找一下哪里用到了这个文件

- 最后，重新执行，只要不报错，就完成前端工程瘦身啦！！
## 6 后端初始化

### 6.1 三种方式初始化项目

**方式一：github上搜索SpringBoot Template**

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fef1495bf1d90bc41ec9825b3471a048.png)

- 不建议使用，以防人家代码，不ok！

**方式二：SpringBoot官方的模版生成器**
- 网址：start.spring.io
- 目前不支持SpringBoot2.x 版本的生成了，建议使用阿里云的模版
- 阿里云模版网址：start.aliyun.com

**方式三：IDEA开发工具中直接使用**
- 学过SpringBoot的都知道，就不赘述了
- 主要讲讲需要哪些依赖
	- lombok
	- spring boot devtools：可以帮助我们热更新
	- spring configuration processor：读取注解的时候需要用到
	- mysql driver
	- spring web
	- mybatis
- 还需要添加依赖（因为初始化后端项目的时候没办法选取）
	- mybatis-plus
### 6.2 环境搭建

测试：mybatis-plus给的demo
问题：[问题3](0_bug处理/1_Part1-BUG.md#问题3)
