
## 1 多环境

[多环境参考文章：原文链接](https://blog.csdn.net/weixin_41701290/article/details/120173283)
### 1.1 概述


**为什么**：
- 每个环境互不影响
- 区分不同阶段：本地 / 开发 / 测试 /  预发布 / 正式上线 / 沙箱（实验）
- 对项目进行优化：
	- 本地日志级别
	- 精简依赖，节省项目体积
	- 项目的环境 或 参数调整，比如JVM参数
---

**概念**：
- 根据实际需要，将同一个项目（代码）按照一定方法进行区分，并将所需资源和项目本身部署到不同的机器上。**不同环境的项目可以有不同的行为**，且能够 **同时存在、互不影响**
- 换个说法：同一个项目（代码）在不同的阶段需要根据实际情况来调整配置并且部署到不同的机器上。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e6c3a70a932e111b8e5cc3a8cc03136.png)

这样一来，本地和线上的项目就完全隔离开了，开发者在本地想怎么折腾就怎么折腾！这便是多环境的好处。

### 1.2 常用环境

多环境听起来虽然挺爽的，但事实上，环境不是区分的越多越好！

一方面是搭建多环境需要额外的工作量；另一方面是项目依赖的资源越多，成本就越高，而且维护起来也更麻烦。

因此，企业中常用的环境也就那么几种，都快成为一种约定俗成的规范了，下面给大家介绍一下。

> 不同团队区分环境的方式可能不同，仅供参考。

---

**本地环境**
- 一般用 `local` 标识，是指前端或后端独立开发、自主测试的环境。通常就是让项目和依赖在我们自己的电脑上运行，比如数据库、缓存、队列等各种服务，可能需要自己在本地搭建。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1a528c5c08eaa027db51d0367aac5098.png)
---

**开发环境**
- 一般用 `dev` 标识，是指前端和后端（或者多个程序员）一起协作开发、联调的环境。通常将项目和依赖放在员工电脑可以直接访问的开发机上，不用自己搭建，直接跑起来项目，提高开发和协作效率。
- 对规模不大的团队来说，开发和本地环境其实有一套就够了，毕竟本地也可以连接公用的数据库等服务。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/37682fa5a1f0369ed1bc480989df85f1.png)
---

**测试环境**
- 一般用 `test` 标识，是指前端和后端开发和联调完成，做出完整的新功能后，交给测试同学去找 Bug 的环境。
- 通常在测试环境需要有==独立的测试数据库和其他服务==，让测试同学大显身手。每次修改完 Bug 后，也都要再次发布项目到测试环境，让测试同学重新验证。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8f59184120cfa87d4c31366e50279bf7.png)
---

**预发布环境（体验服）**
- 一般用 `pre` 标识，这是==和线上项目最接近的环境==，一般是测试验证通过、产品经理体验过后，才能将项目发布到这个环境。
- 实际上，预发布环境的项目调用的后端接口、连接的数据库、服务等都 ==和线上项目一致== ，和线上==唯一的区别就是前端访问的域名不同==。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/81b110d567185da08df59c55365b8b7a.png)
---

**生产/线上环境**
- 一般用 `prod` 标识，又叫线上环境，是给所有真实用户使用的环境。
- 因此不能随意修改，且发布项目到该环境时必须格外小心。线上的数据库、机器等资源一般也是由专业的运维来负责，想要登录机器、修改配置，都需要经过严格审批。
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/672c9a585cb4716ea467523ce11c90db.png)
---

**沙箱环境（实验环境）**
- 为了某个功能进行实验的环境
### 1.3 如何实现？

多环境的实现方式，其实大同小异，遵循 3 个步骤：抽象配置类 + 配置文件化 + 注入环境参数，就能轻松实现~

---

**抽象配置类**
- 将项目代码中==需要根据环境的变化而更改的变量==整理到一个或多个配置类中，集中管理。
- 例如：连接数据库时，我们需要数据库 IP、端口、配置等信息，我们可以将==这些变量放入到一个类中==，然后从这个类读取变量的值
- 这样的好处是，如果代码中还有其他地方用到了这些变量，也都可以从同一个类去获取，而不是把 ==死值== 重复写多次，难以维护。

---

**配置文件化**
- 可以==用专门的配置文件来维护配置==，从而让用户修改配置更方便，不用再去找代码、改代码
- 常见的配置文件格式有 `properties`、`yaml`、`yml`、`json` 等，比如新建一个数据库配置文件 `db.properties`
- 接下来在初始化数据库时，就可以==将配置文件中的值加载到上一步写好的配置类==中，然后读取
- ==只不过是把配置的值从代码中移到了文件中而已~==
- 无论是前端还是后端，大部分的多环境实现都是这个原理 —— 搞多套配置，所以总能在项目中看到类似的配置文件
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa0e3fae639ce18dc08463e6008f1b11.png)
---

**注入环境参数**
- 到目前为止，其实我们还是在代码中写了 ==死值== ，来告诉程序应该加载哪个名称的配置文件
- 比如在本地开发时，加载 `db-dev.properties` ，开发完成后、正式上线前，再改代码为加载 `db-prod.properties` ➡样不仅麻烦，而且可能忘了修改，把开发环境的项目发布到了线上
- 最理想的效果应该是：无论项目要切换到哪个环境，整个项目都完全不用修改
- 因此，我们可以==将 指定环境 这件事放到最后==，在==通过命令去打包或者启动项目时，将环境参数写进去==

- 举个例子，我们在启动 java 项目时，给 `env` 系统变量传递不同参数：
```shell
# 测试环境
java -jar -Denv=test dist.jar
# 生产环境
java -jar -Dend=prod dist.jar
```

- 然后在程序中读取该参数，加载对应的配置即可：
```java
// 读取 env 参数
String env = System.getProperty("env");
new DBConfig("db-" + env + ".properties");
```

- 同理，对于前端项目，可以在打包构建时传入环境变量，然后自己在代码中读取，或者交给 Webpack 之类的打包工具处理：
```json
{  "scripts": 
	{    
		"serve": "env=dev serve",    
		"build:test": "env=test build"    "build": "env=prod build"  
	} 
}
```

### 1.4 前端多环境实战

首先，我们启动前端项目，进入登录页面，随便输入账户密码，可以看到请求的地址是`localhost:8000`
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf27d92edde96cd87fa3bd735df9c02e.png)

之后要把整个服务上线，那我们自己的电脑，肯定不能24h开着，所以这个地址肯定要改一下，那么问题来了，

---

**问题：怎么让前端知道什么时候该请求localhost，什么时候该请求远程地址呢？**

**答案：** 可以在启动项目（或构建项目）的时候，把这个环境变量给它写进去，通过传入一个变量来区分环境

---

**问题：怎么把这个传输传进去呢？**

**分析：**
- 我们启动项目的时候，是在package.json文件里启动的，这里定义了一系列的命令，那么我们就可以在执行这些命令的时候给这个前端项目传递一些参数！
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0335406bc9aab6902f016600fbfc5753.png)

- （umi框架特性）项目使用的是umi框架，build的时候会自动传入 NODE_ENV == production 参数，start的时候 NODE_ENV == development
- 那么我们直接看看看全局文件，app.tsx文件，发现有一个 process.env.NODE_ENV，那么我们就知道了，前端项目就是拿到这个参数（development）去打包的
	- 可以在下面加一个`alert(process.env.NODE_ENV)`来验证一下
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d3490c617b1aa0bedcd4f75e4a39be76.png)
- 我们现在build一下，会发现项目已经编译出来一个dist目录，这就是build出来的目录（即将我们所有文件打包出来的一个目录）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/088b21faf78461f3401478a846739265.png)

	- **注意**：如果nodejs版本很高的话，需要在build前面加上`set NODE_OPTIONS=--openssl-legacy-provider &&`

- 那我们试一下运行这个dist文件夹吧！
	- 先全局安装 serve插件 `npm i -g serve`，无所谓使用哪里的命令行
	- 注意：安装完成后使用`serve -v` 检查是否安装成功
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9194b6bce666eb5986541b61bc40374.png)
	- 进入dist目录，使用`serve`指令，运行这个目录下的程序，发现启动成功
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8c3abe3200bbd3e0d8bf50845d0c3b9.png)
	- 意思是在本地给你启动了一个外部服务器，可以在本地把这个前端项目，放到这个外部服务器上
	- 用serve 启动相当于我们在自己的电脑运行了这个正式环境，现在自己的电脑就是这个前端项目的服务器，只不过别人访问不了
	- 进入这个网站，发现弹出来的是production 而不是 development，说明我们用build命令的时候，umi框架已经帮我们把NODE_ENV修改为production
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/67436876aa0790264a64451142119761.png)
- 根据上面的理论，我们现在要求线上的前端要请求到线上的后端，我们修改全局请求类`globalRequest.ts`中umi请求库（request）的prefix，代码如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/638fc2e6973edff5815a5ac6f1dade8a.png)
	- prefix就是我们每次请求的前缀，通过前缀可以改变每次请求url前面的部分（域名+端口等）
	- 重新build命令打包一下，然后serve运行一下dist文件，然后打开页面进行登录，发现接入的域名发生了变化，如下图
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b6bc0831b3befae6302649da71d007c1.png)
	- 也就是说，通过了一个环境变量就成功区分出了本地以及线上项目，如果要加入测试环境，就如法炮制

---

上面我们将了启动方式（dev / build-serve）和 请求地址的修改，但是开发环境和线上环境还是有一些区别的，比如说项目的配置

**项目的配置**
- 不同的项目（框架）都有不同的配置文件，umi的配置文件是config，可以在配置文件后添加对应的环境名称后缀来区分开发环境和线上环境
- 例如：[参考文档](https://umijs.org/docs/guides/env-variables)
	- 开发环境：config.dev.ts
	- 线上环境：config.prod.ts
	- 公共环境：config.ts（所有环境都适用的配置）

我们现在本地上运行的这些代码，没有做一些压缩，加密混淆的工作
因为在本地运行我们不太需要考虑性能，但是发布到线上的时候，可能要对网站做一些按需加载，或者静态化

### 1.5 后端多环境实战

## 2 项目的部署上线

### 2.1 原始部署

#### 2.1.1 部署前端

#### 2.1.2 部署后端

### 2.2 宝塔

### 2.3 Docker