
移动端的H5网页（尽量兼容PC端）
## 1 需求分析

如何去匹配用户
1. 用户添加标签
	- 标签的分类，如：学习方向、性格等
2. 用户主动搜索：允许用户根据标签去搜索其他用户
	- Redis 缓存 + 本地（想清楚值不值得！）
3. 用户组队（房间）
	- 创建队伍
	- 加入队伍
	- 根据标签查询队伍
	- 邀请其他人
	- 额外需求：允许超过房间的人数（房间限定3人，最多5人，前3人有人不在，后2人补位）
4. 用户推荐功能
	- 相似度计算算法 + 本地分布式计算
5. 用户修改标签


## 2 技术栈

### 2.1 前端

1. Vue3 开发框架（提高页面开发的效率）
2. Vant UI（基于Vue的移动端组件库）（也支持React版 Zent）
3. Vite（前端工程的打包工具，速度快！）
4. Nginx 来单机部署

### 2.2 后端

1. Java + SpringBoot
2. SpringMVC + MyBatis + MyBatis Plus
3. MySQL 数据库
4. Redis 缓存（提高数据的查询性能，降低MySQL数据库的压力）
5.  Swagger + Knife4j 接口文档
6. 持续补充

## 3 本期内容计划

1. 前端项目初始化
2. 前端页面 + 组件预览
3. 数据库表设计
	- 标签表
	- 用户表
4. 后端 - 根据标签搜索用户
5. 前端 - 根据标签搜索用户

## 4 前端项目初始化

### 4.1 用脚手架初始化项目

- Vue CLI：官方不支持使用该工具快速构建项目，我们换一种方式[工具链 | Vue.js (vuejs.org)](https://cn.vuejs.org/guide/scaling-up/tooling.html#project-scaffolding)
- Vite：快速构建Vite + Vue项目
	- 在前端项目所在目录下CMD，然后输入命令`npm create vite`（**需要安装了Nodejs**）
	- 分别创建前端项目名称 `rainbow-front`，使用`Vue`框架，选择`TS`语言
	  ![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3019b96b68a256d2732d533fea76b80b.png)
使用WSR打开前端项目
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/772cb1b5cfaf7118428ccb2a7dfeae42.png)

`package.json`文件中定义了前端所需要的依赖和项目的构建等过程，具体内容见下图：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dcf0e2a42e2de714c919ac64b083eb33.png)
**注意：json是不能注释的！！！**

开启项目后，需要安装项目所需的依赖，在命令行中使用`npm i` 命令
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1bce9d301828aab5347c2d9323e4e37a.png)
### 4.2 整合组件库Vant

看官方文档：[Vant 3 - 轻量、可靠的移动端组件库 (gitee.io)](https://vant-contrib.gitee.io/vant/v3/#/zh-CN)

**安装**：我们使用的vite项目，所以选择**按需引入**，直接复制命令安装即可
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7709eed41837d62f95a27d6c1981cc2.png)

**配置插件**：全选，覆盖原有文件即可
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b98512b5bdd0efa776d44bc9fbf7186c.png)


**使用组件和API**：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c568353a05291780fc145526b45e70a.png)

## 5 前端页面构建

开发页面的经验：
- 多参考人家的网页
- 从整体到局部
- 先想清楚页面要做成什么样子，再写代码

**移动端不需要侧边栏**

补充使用vue模版，一键创建vue文件：[webstrom创建vue常用模板-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1834320)
### 5.1 layout设计

每一个页面都要复用一些组件（vue），重复写很麻烦，不利于维护，所以抽象一个通用的布局（layout），可以使用layouts文件夹存放一些我们需要复用的vue组件，layouts目录（在src目录）下创建`BasicLayout.vue`

**第一步：设计NavBar导航栏 **
- 导入UI模板
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/340d6e0e57d83fc7a228c6df2513ca73.png)
- 修改一：去掉返回字样，只留下按钮
- 修改而：通过插槽自定义导航栏两侧的内容，注意代码复制的情况
		![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e31ad8befdd78072bb396f0b3273559.png)

**第二步：设计tabber标签栏**
- 使用监听切换事件
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c19593febb241e9d79b2441e99c90862.png)
- 修改`tabber-item`中的属性，设置为我们需要的内容

  

### 5.2 导航栏

### 5.3 主页搜索框

搜索页

搜索结果页（标签筛选页）

### 5.4 内容

### 5.5 tab栏（tabbar）

主页（推荐页 + **广告** ）
- 搜索框
- banner
- 推荐信息流

队伍


用户（消息——暂时考虑发邮件）



