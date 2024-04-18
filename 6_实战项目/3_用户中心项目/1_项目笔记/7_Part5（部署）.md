
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

**问题：怎么把这个配置传输传进去呢？**

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

想一下我们后端做了什么事情，是不是就是提供了用户的增删改查、登录注册。

那我们用户的增删改查的数据存在本地的mysql数据库，上线的时候，是不是要把这个数据库地址改成一个远程地址，起码要公网可以访问，或者说自己的前后端项目部署的那个服务器可以访问

所以，**后端多环境主要修改的是**：
- 依赖的环境地址
	- 数据库地址
	- 缓存地址
	- 消息队列地址
	- 项目端口号
- 服务器配置

---

**后端怎么去区分不同的环境?**

和前端一样，基本上所有的项目都是这种模式：**通过配置文件后面加一个后缀来区分环境**

我们项目是Springboot框架的，所以配置文件就是`applicaiton.yml文件`
- 这个文件是==公共的配置==，公共配置就是==任何环境都会加载这个配置==，所以像一些MP这种框架层面的配置就写在这里就好了

那么我们直接复制这个文件，再创建一个`application-prod.yml文件`
- 这个文件就是，用于==生产环境的配置==，所以我们要删除掉一些公共的配置，如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12e2069ec04ffbb3c08c0da4726bc7dd.png)
> 提示：我们可以将我们创建的user表的sql文件保存在项目的sql目录下，==后期我们就可以将这个创建表的sql语句，放在服务器上的mysql执行了==
> ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e070c8c49cea650bc710013bf170a74.png)

---

**修改application-prod.yml中数据库的配置**

前提：
- 有一个云端的数据库
- 或者，自己的服务器上已经安装了mysql数据库，并且正常启动
- 没有的话，先看后面[2 项目的部署上线](#2%20项目的部署上线)  以及 [2.5 服务器安装](../../../2_学习笔记/6_Linux&Git/1_Linux/7_Linux系统软件安装-1.md#2.5%20服务器安装)

这个文件是生产环境使用的，那么数据库肯定也是连接生成环境的，所以我们需要将项目数据库的相关配置进行修改，修改如下
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/57f54e1812991e2261b8fd368bce52c4.png)

我们现在使用maven工具，将本地项目打一个jar包，会在文件中生成一个target目录，目录下有一个jar包就是我们后端项目打包形成的
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/462f05bbe71c9a05ec67514249a057be.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0abfefe7a3751a55a039e9fbdced069b.png)

现在我们进入终端，运行jar包，并传入一个参数
```shell
java -jar ./usercenter-backend-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

如果报错（没有清单主属性），说明是使用阿里云镜像创建的springboot项目，需要在pom.xml文件中注释掉这一行
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0691e24f7b378dcf4ab075268847b8ce.png)


运行后端项目成功后，我们运行前端项目，看一看前端访问那一个数据库。
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/97a23ff5859d69457190a536c79c267c.png)

我们也可以直接注册，然后发现注册的数据在服务器上的数据库，说明成功了
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a98381e78798e1d193b5f04c868926d2.png)
## 2 项目的部署上线

### 2.1 原始部署

**什么的软件都自己安装！**
#### 2.1.1 部署前端

前端部署需要的软件（web服务器）
- ==nginx==：主要使用的
- apache
- tomcat

---

**安装nginx服务器**

安装方法：
- 用系统自带的软件包管理器快速安装，centos系统中是yum
- 自己到[官网](http://nginx.org/en/linux_packages.html)，下载安装包，来安装（我们用这种方式来进行）

下载nginx服务器，复制选择款的链接
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75b1d32dea5d1fd1364f08d3fb2bc86c.png)
安装前，先做一点准备工作
```shell
//查看当前目录
pwd

// 在usr目录下创建services目录，用来存放所有项目的依赖和安装包
mkdir services /usr/services

//列出目前目录所包含的文件和文件夹
ls

//进入services目录中
cd services
ls
```

在当前services目录下，下载和解压nginx
```shell
//下载
curl -o nginx-1.24.0.tar.gz http://nginx.org/download/nginx-1.24.0.tar.gz

//解压
tar -zxvf nginx-1.24.0.tar.gz
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f69f8a9e430d20cd50eed8a5127a9c8.png)

进入到nginx目录中，并查看内容
```shelll
cd nginx-1.24.0.tar.gz
ls
```

还需要安装nginx的相关依赖
```shell
yum install pcre pcre-devel -y
yum install openssl openssl-devel -y
```

设置系统配置参数（注意这个时候是在nginx的目录下执行的）
```shell
./configure --with-http_ssl_module --with-http_v2_module --with-stream
```

此时直接编译
```shell
make
```

**如果编译报错，还需要安装下面的依赖**
```shell
yum -y install make zlib-devel gcc-c++ libtool openssl openssl-devel

// 然后重新configure 和 make
```

再来安装
```shell
make install
```

配置nginx的环境变量，加入到系统环境变量中去
```shell
// 打开使用 /etc/profile这个文件
vim /etc/profile

// 按下shigt+g 将光标定位到最后一行，新增如下内容：
export PATH=$PATH:/usr/local/nginx/sbin

// 使用上述文件生效
source /ect/profile
```

测试nginx命令能否使用，如果没有报错，就说明成功了
```shell
nginx
```

使用命令`netstat -ntlp`，继续查看当前所有tcp端口情况，可以看到nginx启动成功
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/058c6269e555df8b3661953187824820.png)

复制nginx.conf文件，并重命名为nginx.default.conf
```shell
cd /usr/local/nginx/conf
cp nginx.conf nginx.default.conf
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0835e4a7310f328a561514e352266343.png)

回到services目录
```shell
cd /usr/services
```

---

**前端开始部署**

前端build一下，打包后，将dist包拖入到我们的services目录下，并改名
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d31ee9ae382c3882e2dc7bf7c7085990.png)

```shell
// 改名
mv dist usercenter-frontend
```

修改nginx配置文件，设置==启动用户==和==前端项目所在路径==
```shell
// 注意不是nginx目录下的嗷

vim /usr/local/nginx/conf/nginx.conf
```

修改的内容如图所示
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f6e05f2bd110bcdabdf77c4e1d89f66.png)

补充：上述图片不需要`/root`这个目录

更新nginx.conf文件
```shell
nginx -s reload
```

访问我们公网的地址，出现页面
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/086b24b40f3bf10a2e32fb866a556569.png)

#### 2.1.2 部署后端

后端的部署需要两个依赖 java  和 maven

---

**安装java**

```shell
yum install -y java-1.8.0-openjdk*
// yum安装不用配置环境变量
// 查看java版本，安装成功
java -version
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f0090856272c79e1eb7612dc47667a1f.png)

--- 

**安装maven**

```shell
cd /usr/services

curl -o apache-maven-3.8.8-bin.tar.gz https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz

tar -zxvf apache-maven-3.8.8-bin.tar.gz
```

修改镜像为阿里云
```shell
vim apache-maven-3.8.8/conf/settings.xml

//找到<mirrors>标签在标签里面添加阿里云镜像配置
    <mirror>  
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>

    </mirror>

  </mirrors>

```

进入到maven
```shell
cd apache-maven-3.8.8
cd bin
ls
// ls打开后看到的mvnmvn是maven的可执行文件，咱们就是用这个文件去构建项目

pwd
//获取当前目录路径后，复制一下
```

添加环境变量
```shell
vim /etc/profile

// 把乖乖复制的路径，放在nginx环境变量后面

//使文件生效
source /ect/profile

//检查一些环境变量是否设置成功
mvn -v
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35aa18f3c13d8ddff7e2292e134d2550.png)

---

**将后端项目部署到服务器**

使用git直接把拖到services目录中，先下载git
```shell
yum install -y git
```

使用git拉取我们的项目，这里代码省略，学过git就知道啦

进入到我们下载好的后端项目，使用如下命令，来打包构建我们的后端项目
```shell
cd usercenter-backend

//打包构建，并跳过测试 或者 直接将我们window中打包好的项目放入也行
mvn package -DskipTests
```

打包完成之后会在这个后端项目生成一个target目录，目录下保存着我们的jar包
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a78588eb544081872f2060656a9c9109.png)

执行这个jar包
```shell
//注意目录
java -jar ./usercenter-backend-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod

//之后如果显示没有权限，给这个文件添加可执行权限
chmod a+x usercenter-backend-0.0.1-SNAPSHOT.jar

//再次执行上述命令

//这样的话这个，窗口就一直被这个jar包执行的内容所占满了，我们ctrl+c终止

// 使用如下命令，让他在后台运行
nohup java -jar ./usercenter-backend-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod & 

//直接回车，不要管，就回到命令界面了

// 使用如下命令，查看后台运行的情况
jobs
netstat -ntlp
//或者
jps
```

关闭
```shell
ps -ef | grep 你的jar包文件名
kill [-9] 找到其运行的进程
```

**友情提示：到这里最好镜像一下嗷！**

### 2.2 宝塔


宝塔只是一个Linux运维面板，方便管理服务器，方便服务器中安装软件，也可以自行去官网安装，[官网地址](https://www.bt.cn/new/download.html)

目前（2024-4-15）购买的腾讯云服务器，已经自带了宝塔Linux面板，就不需要在服务器中安装宝塔了

---

**开启宝塔界面**

放行端口一下就好，就是让自己的电脑（ip）可以访问服务器
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6ea5d05ebdcdec8c2eb2aa52d9df3f27.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/34b3d2aa5ae7b2ddcf65e3fd905a361d.png)

回到应用管理，可以看到面板首页地址，本机使用这个地址到浏览器中就可以访问
不过先需要点击下面的登录，然后输入命令获取用户名和密码
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2941109707f4504b84dc6e9b4cfbc6cd.png)

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ad70279abe74950aded044972e1941e7.png)

进入宝塔界面之后，点击面板设置，修改一下安全入口、用户名、密码登资料

---

**回到服务器最初的镜像文件中（啥都没有），我们之前保存了**

开始使用宝塔安装剩余的软件：nginx、tomcat、docker

---

**部署前端页面**

域名部署参考链接：[宝塔Linux面板Java项目部署域名访问 (SpringBoot项目)_宝塔 java项目 配置域名-CSDN博客](https://blog.csdn.net/weixin_43652507/article/details/132076705)

回到宝塔面板，点击添加站点![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/85cd1142c94ce1a7a0cdfebba023b962.png)
进入到站点的目录
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6eb64da817953ecd506ae9696031bcc2.png)

全选文件删除掉！
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56568b1882ffa1dada80511ccd20fdaa.png)

然后直接把前端打包好的项目dist，扔进去！
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/69853f05b4b550794a1d9bff199817eb.png)

直接在公网访问一下，`如果是域名就访问域名，如果是服务器ip就访问服务器ip`

---

**部署后端部分**

找到侧边栏的文件，来到wwwroot目录下，创建一个usercenter-backend目录，用于存放我们后端项目
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a31961742c4773fcb51fdac5187551c.png)

进入这个项目，把之前打包好的jar包，放进去
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d719ca5b7c0f23e19404f9256d41336a.png)

复制该jar包所在的路径，然后去网站哪里创建Java项目
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/55fe403876493b43a69a42be53dfc74d.png)
```shell
/www/server/java/jdk1.8.0_371/bin/java  -jar -Xmx1024M -Xms256M  /www/wwwroot/usercenter-backend/usercenter-backend-0.0.1-SNAPSHOT.jar --server.port=8080 --spring.profiles.active=prod
```


可以发现项目运行中，但是端口还没有放行![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cee8f76b84d00a27065c76b9334edeb0.png)
先把tomcat停掉，否则tomcat占用8080端口
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a17e9afe5b67f2818e176d721a63fa03.png)


去防火墙，把8080端口放行
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb8be2b7170adf61486e8cd661f3aaa1.png)

访问一下`服务器ip：8080`，发现无法访问
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5154020362139ab30870ea253cdffb69.png)

本地修改一些配置，重新打个包，传到宝塔上面（下面错了，直接写0.0.0.0即可）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/87ffca32dda6c0a435dc9b4745721e0a.png)

再次访问，出现如下界面，随便加一个路径，成功了
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/54bac77bdedca732f45281be7e3d5ea7.png)

---

前后端联调

### 2.3 Docker

Docker 是容器，可以将项目的环境（java、nginx）和项目的代码一起打包成镜像，这样有权限的用户能下载镜像，更容易分发和移植

再启动项目时，不需要敲一大堆命令，而是直接下载docker镜像，启动镜像就启动项目了

**docker可以理解为软件安装包**

--- 

**安装**
- 官网安装
- 宝塔安装（我们用这个）

---

**什么是Dockerfile文件**
怎么把我们的前后端项目制作成镜像，肯定是需要一个文件来定义，所以我们需要把我们的前端项目的依赖、启动流程，全部写到一个文件里面，这个文件就是Dockerfile

所以，Dockerfile用于指定构建Docker镜像的方法：
- 一般情况下不需要完全从0自己写，建议去github，gitee等托管平台参考同类项目即可

Dockfile文件编写：
- FROM：依赖的基础镜像
- WORKDIR：工作目录
- COPY：从本机复制文件
- RUN：执行命令
- CMD/ENTRYPOINT：附加额外参数，指定运行容器时默认执行的命令

---

**构建后端**

**推荐使用使用IDEA连接服务器的docker，然后本地构建镜像到服务器上去**
- [1_SpringBoot项目部署（Docker）](../../../5_实用文件/8_部署上线相关/1_SpringBoot项目部署（Docker）.md)

我们这里直接复制鱼皮的后端Dockerfile
```dockerfile
# Docker 镜像构建
FROM maven:3.5-jdk-8-alpine as builder

# Copy local code to the container image.
WORKDIR /app
COPY pom.xml .
COPY src ./src

# Build a release artifact.
RUN mvn package -DskipTests

# Run the web service on container startup.
CMD ["java","-jar","/app/target/usercenter-backend-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]
```

在后端项目根目录下，新建一个Dockerfile，并复制代码
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7c11b009250c2d852737d8f64086939.png)

然后再shell中，进入 /www/wwwroot下 使用git拉取后端项目
```shell
git clone 后端项目的地址
```

拉取完成后，进入项目的目录中，可以发现Dockerfile已经存在了,我们使用它来构建后端镜像
```shell
sudo docker build -t usercenter-backend:v0.0.1 .
```

然后我们可以发现，这里命令执行的很慢，为什么呢？
- 因为我们把我们把maevn构建打包项目放在了dockefile里面了
- 优化，我们可以把maven现在本地构建，然后打包放过去

---

**构建前端**

还是使用鱼皮的Dockerfile，这个复制到根目录下的Dockerfile文件中
```dockerfile
// docker镜像依赖的web服务器
FROM nginx
// 指定nginx的工作目录
WORKDIR /usr/share/nginx/html/
// 指定一下用户，可以不写
USER root
// 将docker中的nginx文件，覆盖前端项目的nginx文件
COPY ./docker/nginx.conf /etc/nginx/conf.d/default.conf
// 将我们前端的代码，转移到nginx默认可以识别的html目录下
COPY ./dist  /usr/share/nginx/html/
// 项目端口
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

还需要粘贴一个nginx.conf的配置，这个复制到docker目录（在根目录）下的nginx.conf配置中
```conf
server {
    listen 80;
    
    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";
    root /usr/share/nginx/html;
    include /etc/nginx/mime.types;
    location / {
        try_files $uri /index.html;
    }
}
```
- try_files：表示如果用户找不到某个页面的话，直接到index主页去

同理，使用git拉取我们的前端代码，然后使用dockerfile构建前端镜像
```shell
sudo docker build -t usercenter-frontent:v0.0.1 .

// 可以看一下安装的镜像
sudo docker images
```

- Docker 构建优化可以从这些方面入手：减少尺寸、减少构建时间等，感兴趣可以试试

---

**启动项目**
```shell
docker run -p 80:80 -d usercenter-frontend:v0.0.1

docker run -p 8080:8080 usercenter-backend:v0.0.1
```

虚拟化：
- 端口映射：把本机的资源（实际访问地址）和容器内部的资源（应用启动端口）进行关联
- 目录映射：把本机的端口和容器应用端口进行关联

---

[Docker 命令大全 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-command-manual.html)

**查看日志**
```shell
//查看进程(可以查看已经启动的容器ID，没有权限用sudo)
//容器ID不是固定的，不要盲目复制命令哦
docker ps

//查看日志 docker logs 容器ID
//这些都是咱们的访问记录,可以看到哪个用户,哪个IP,在什么时间访问了你的网页的哪些文件
docker logs 61444cbea251
```

这样看日志有点麻烦，怎么样让这个日志保持始终的持续不断的输出
```shell
//-f跟踪日志输出(没有权限用sudo)
docker logs 容器ID -f

//ctrl + c退出 杀死掉这个容器 docker kill 容器ID
docker kill 61444cbea251
```

举例：如果说我们觉得这些镜像太占用空间了，想把它删掉，怎么删？
```shell
//查看镜像
docker images
//强制删除镜像 docker rmi -f 镜像id
```




