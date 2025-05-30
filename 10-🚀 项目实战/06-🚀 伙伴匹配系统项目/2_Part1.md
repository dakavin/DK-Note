---
文章标题: "[[2_Part1]]" 
文章作者: Dakkk
文章概要: |
  文章详细阐述了一个移动端H5用户匹配与组队应用的项目启动过程。内容涵盖了需求分析、前后端技术栈选型（Vue3, SpringBoot, Redis等）、前端项目初始化与组件整合、页面布局设计、以及核心的后端数据库表结构设计（用户表与标签表）及初步的API开发。
文章标签:
- "Vue3"
- "SpringBoot"
- "Redis"
- "数据库设计"
- "H5开发"
- "用户匹配"
- "全栈开发"
- "项目实战"
相关文章:
- "[[1_优化分类管理模块]]"
- "[[0_导读]]"
- "[[0_导读]]"
- "[[0_导论]]"
- "[[0_课程介绍]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/06-🚀 伙伴匹配系统项目/2_Part1.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-24 00:31:55
---

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


**配置插件**：全选，看文件缺哪里复制哪里，不然样式不起作用（已经不需要了）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ebb8143dfd43cec4e38406f88fa9d14c.png)



**使用组件和API**：
- 我们先使用一下Button按钮，复制到App.vue文件中的template标签中即可
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9844353f3ea5247ac194f4e2b275858.png)
  - **注意：**
	  - 教程中，按需引入，还需要导入Button组件，后续使用的组件都需要这样导入
	    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fc2ede85ab7860f808504a71d87534d6.png)
	- 但是在Vnat4版本中，就不需要这么做了；因为在`按需安装vant`的时候，我们导入了==自动导入样式的解析器@vant/auto-import-.resolver==，它可以帮我们自动注入组件
## 5 前端页面构建

### 5.1 设计

开发页面的经验：
- 多参考人家的网页
- 从整体到局部
- 先想清楚页面要做成什么样子，再写代码

**移动端不需要侧边栏**

补充使用vue模版，一键创建vue文件：[webstrom创建vue常用模板-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1834320)

**设计内容：**
- 导航条：展示当前页面名称
- 主页搜索框 =》 搜索页 =》搜索结果页（标签筛选页）
- 内容
- tab栏：
	- 主页（推荐页 + **广告**）
		- banner
		- 搜索款
		- 推荐信息流
	- 队伍页
	- 用户页（消息）

### 5.2 layout设计

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
  **第三步：使用函数切换页面**
  - 代码如下：
    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79ab1dc5fae518e6876690bdc7914a95.png)
### 5.3 创建其他页面

先在src目录下创建一个pages目录，用于存放常规页面。创建index和team页面，效果如图所示：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f861c9187dc56bc310a2967f5a1bbc40.png)


## 6 后端库表设计

标签的分类（要有那些标签，怎么把标签进行分类）

### 6.1 标签表（分类表）

建议用标签，不要用分类，标签更加灵活
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b6c69c844a8a50b55dfa2bdb6e39f1d8.png)

标签有那些组？
- 性别：男、女
- 方向：Java、C++、Go、前端
- 目标：考研、春招、秋招、社招、考公、竞赛、转行、跳槽
- 段位：初级、中级、高级、码神
- 身份：小学、初中、高中、大一、大二、大三、大四、学生、待业、就业、研一、研二、研三
- 状态：乐观、有点丧、一般、单身、已婚、有对象
- 【用户自定义标签】

使用MySQL数据库把这些标签存下来，那字段怎么设计呢？
- id：int、primary key
- 标签名：varchar、not null
- 上传标签的用户 userId：int
- 父标签id，parentId：int（分类）
- 是否为父标签 isParent：tinyint（0不是父标签 1是父标签） ==boolean类型不够灵活==
- 创建时间 createTime：
- 更新时间 updateTime：
- 是否删除 isDeleted：tinyint（0 1）

如何查询所有标签，并且把标签分好组？ 通过父标签id，可以实现√

根据父标签查询子标签？根据id查询，也能实现√

**开始创建标签表**
```mysql
CREATE DATABASE IF NOT EXISTS rainbow;

USE rainbow;

CREATE TABLE tag
(
	id BIGINT auto_increment COMMENT 'id'            PRIMARY KEY,
	tagName VARCHAR(256)                            NULL COMMENT '标签名称',
	userId BIGINT                                    NULL COMMENT '用户id',
	parentId BIGINT                                 NULL COMMENT '父标签id',
	isParent TINYINT                                NULL COMMENT '0-不是，1-父标签',
	createTime DATETIME DEFAULT CURRENT_TIMESTAMP   NULL COMMENT '创建时间',
	updateTime DATETIME DEFAULT CURRENT_TIMESTAMP   NULL ON UPDATE CURRENT_TIMESTAMP,
	isDelete TINYINT DEFAULT 0                      NOT NULL COMMENT '是否删除'
) 
	COMMENT '标签';
```
### 6.2 创建用户表

首先我们需要知道，用户有哪些标签？
- 方法一：直接在用户表补充tagList字段，如字段上直接使用 `['Java','男']`这样的JSON字符串
	- 优点：
		- 查询方便，查询用户的时候，直接将标签查出来了
		- 不需要新建关联表
		- ==标签是用户的固有属性==（以后其他系统也会用到）
		- 节省开发成本
	- 缺点：
		- 用户表多了一个列
		- 数量量大的时候，查询会比较慢（`可以使用缓存优化`）
- 方法二：加一个关联表，记录用户表和标签表的关系
	- 思考一下什么时候需要用关联表？
		- 查询某个标签下有多少个用户
		- 某个用户有多少个标签，不过这种操作比较麻烦
	- 优点：查询灵活，可以正反查询
	- 缺点：多建/维护一个表
	- 重点：企业大项目开发中尽量减少关联查询，很影响拓展性，并影响查询性能。

场景分析：查询用户列表，先查关系表拿到这100个用户有的所有标签，再根据标签id，查询标签表，很麻烦！

**根据我们目前系统的业务需求，我们使用方法一**

创建用户表，代码如下：
```mysql
CREATE TABLE user
(
	username VARCHAR(256)                           NULL COMMENT '用户昵称',  
	id BIGINT auto_increment COMMENT 'id'           PRIMARY KEY,
	userAccount VARCHAR(256)                        NULL COMMENT '账号',
	avatarUrl VARCHAR(1024)                         NULL COMMENT '用户头像',
	gender    TINYINT                               NULL COMMENT '性别',
	userPassword VARCHAR(512)                       NOT NULL COMMENT '密码',
	phone VARCHAR(128)                              NULL COMMENT '电话',
	email VARCHAR(512)                              NULL COMMENT '邮箱',
	userStatus INT DEFAULT 0                        NOT NULL COMMENT '状态0-正常',
	
	createTime DATETIME DEFAULT CURRENT_TIMESTAMP   NULL COMMENT '创建时间',
	updateTime DATETIME DEFAULT CURRENT_TIMESTAMP   NULL ON UPDATE CURRENT_TIMESTAMP,
	isDelete TINYINT DEFAULT 0                      NOT NULL COMMENT '是否删除',
	
	userRole INT DEFAULT 0                          NOT NULL COMMENT '用户角色 0-普通用户 1-管理员',
	userCode VARCHAR(512)                           NULL COMMENT '用户编号',
	tags VARCHAR(512)                               NULL COMMENT '用户标签列表'
) 
	COMMENT '用户';
```

### 6.3 索引处理

标签表的处理：
- 标签名tagName：必须加一个唯一索引
- 上传标签的用户userId：目前可加可不加，如果要根据userId查已上传标签的话，则需要加索引，加一个普通索引
- 父标签id：最好是对于那些有很多值的列，再加上父标签

使用navicat软件，设计tag表，增加索引如下
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8bc111d2679ff50eca18884a30ef5c52.png)

## 7 开发

### 7.1 创建后端项目

**导入所需的依赖：**
- `spring-boot-starter-web`：
- `mybatis-spring-boot-starter`：3.5.1
- mybatis-plus-boot-starter：3.5.1
- commons-lang3：3.12.0
- `spring-boot-devtools`：scope---runtime optional---true
- `mysql-connector-java`：scope---runtime
- `spring-boot-configuration-processor`：optional---true
- `lombok`：optional---true
- `spring-boot-starter-test`：scope---test
- junit：4.13.2 scope---test
- 其中标颜色的地方，可以在初始化springboot项目的时候选择（test自动生成，不用选择）

**导入我们的数据库**
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14f4d86cc4402bb0c7db2b2145936aca.png)

**使用MybatisX一键生成代码**
- 我们没有用驼峰，注意选项
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0123b7cdb2b60a14de0bdc7effd8d527.png)

- 下一步的选项，options中选择实际的列
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a8537e189cf5d5a0fb11b41edc619695.png)
- 检查生成的文件是否有问题，没有问题将生成的文件移动到对应的文件夹中
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/18d78ede072e9041faedc03e86fcb45c.png)
- 注意我们有一个逻辑删除的列，需要备注上注解@TableLogic
  ![image.png|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fdcf411813f822320c8017118e81e1b8.png)
- 用户表同理

### 7.2 开发后端接口

#### 7.2.1 搜索标签

用户根据标签进行搜索，两种方式

**方式一：SQL查询**
- 允许用户传入多个标签，多个标签都存在才能搜索出来 and
	- `LIKE '%java%' AND LIKE'%C++%'`
- 允许用户传入多个标签，有任何一个标签存在都能搜索出来 or
	- `LIKE '%java%' OR LIKE'%C++%'`

**方式二：内存查询**
- 先将用户信息拉取到内存中，在内存中解析用户的标签为一个List之类的结构
- 然后查询的时候，再在内存中判断有没有我们需要的值


直接在UserServiceImpl中写一个 **通过tag查找用户的方法**
```java

```



**方式比较**
- 

### 7.3 根据标签搜索用户（前端）







