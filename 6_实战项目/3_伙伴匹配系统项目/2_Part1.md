
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

看官方文档：[Vant 4 - 快速上手](https://vant-ui.github.io/vant/#/zh-CN/quickstart)

**安装**：我们使用的vite项目，所以选择**按需引入**，直接复制命令安装即可
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b0994fcce95a9b8b88e9c05e75173833.png)


**配置插件**：全选，看文件缺哪里复制哪里，不然样式不起作用
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ebb8143dfd43cec4e38406f88fa9d14c.png)



**使用组件和API**：
- 我们先使用一下Button按钮，复制到App.vue文件中的template标签中即可
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9844353f3ea5247ac194f4e2b275858.png)
  - **注意：**
	  - 教程中，按需引入，还需要导入Button组件，后续使用的组件都需要这样导入
	    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fc2ede85ab7860f808504a71d87534d6.png)
	- 但是在Vnat4版本中，就不需要这么做了；因为在`按需安装vant`的时候，我们导入了==自动导入样式的解析器@vant/auto-import-.resolver==，它可以帮我们自动注入组件
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
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cc825bdc464fb0e3babe7b956c568a79.png)

- 修改一：去掉返回字样，只留下按钮
- 修改二：使用插槽，将右侧变为搜索图标的功能，通过插槽自定义导航栏两侧的内容，注意代码复制的情况
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2db78a871f2ab76e5a5ccc334f289e98.png)

- 完整的代码如下：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/07de120dbed95d21aa63bba429ed8abf.png)



**第二步：设计Tabber标签栏**
- 使用监听切换事件
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c19593febb241e9d79b2441e99c90862.png)
- 修改一：将`tabber-item`中的属性name，设置为我们需要的内容
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8f81042f09080bc63e0809f0016b4e3.png)
- 修改二：通过active关联每个标签中的name属性，当点击某一个tab的时候，就会触发`onChange`方法，并且更改active中的值为该标签的name属性的值
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/663330804373308a6ef70eca0cba9275.png)


  

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



