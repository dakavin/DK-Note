
## 1 前言

Hexo是基于[Node.js](https://nodejs.org/en)编写的，所以需要安装一下nodejs和npm工具（**安装了nodejs就会附带安装npm**），还需要Git工具

不确定是否安装了nodejs、npm、Git？打开cmd输入一下指令即可
```shell
node -v
npm -v
```

安装了的效果如下![41367c96f4257f636254c4b446e8a0d0.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/41367c96f4257f636254c4b446e8a0d0.png)

关于nodejs、npm和Git的介绍及安装，伙伴们可以在网上寻找，博主也会在[本站](https://dakkk.top)后续发布，敬请关注。

如果你已经安装了上述工具，请往下看。
## 2 Hexo

[Hexo](https://hexo.io/zh-cn/)

![e33ba19f7a9e3a8ab2ece224af8b35e1.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e33ba19f7a9e3a8ab2ece224af8b35e1.png)


主要内容来自 [Hexo官方文档](https://hexo.io/zh-cn/docs/)，附上实际安装过程中的一些插图，以便于完全不懂编程的伙伴也能开发出自己的网站，如果你懂编程那就更好啦。

也推荐参考这一份文档 [Easy Hexo 👨‍💻](https://easyhexo.com/)
### 2.1 安装Hexo

在cmd中输入如下指令，在nodejs全局依赖中安装Hexo
```shell
npm install -g hexo-cli
```

成功后效果![fe1846bd7832bdbd9c0f8f16d95ae640.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fe1846bd7832bdbd9c0f8f16d95ae640.png)

也可以输入一下指令，检查是否安装hexo成功
```shell
hexo -v
```

如图所示表示安装成功![864d71508bf819d1372847cc562a7851.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/864d71508bf819d1372847cc562a7851.png)

### 2.2 建立hexo静态网站

在本地创建一个文件夹，作为博客的原始路径（就是从这里出发的）

进入博客文件夹，右键鼠标，选择GitBashHere![fa63bd902ec461571ae59d2559556e42.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fa63bd902ec461571ae59d2559556e42.png)

**一定要进入创建的目录下执行命令，除非你会cd**

执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件
```shell
hexo init     
npm install
```

效果如下：![a825db170c152dddf181e4dd63f47a56.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a825db170c152dddf181e4dd63f47a56.png)
![f3ddcb760a6cf649b501a4dfd3f908be.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3ddcb760a6cf649b501a4dfd3f908be.png)


新建完成后，指定文件夹的目录如下：
```plaintxt
.  
├── _config.yml  
├── package.json  
├── scaffolds  
├── source  
|   ├── _drafts  
|   └── _posts  
└── themes
```

`_config.yml`：网站的 **配置** 信息，可以在此配置大部分的参数；（**重要！**）
`package.json`：应用程序的信息，主要包含了hexo的相关依赖；（新手忽略吧！）
`scaffolds`：模版文件夹，当您新建文章时，Hexo 会根据 scaffold 来创建文件；（新手忽略吧！）
`source`：资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去；（和原理相关的，后面命令的时候会知道的）
`themes`：主题文件夹，Hexo 会根据主题来生成静态页面。

我们来看看静态网站是怎么样的吧，在博客目录下，依次输入下面指令
```shell
hexo clean
hexo generate
hexo server
```

然后在浏览器上输入
```http
https://localhost:4000
```

效果如下：![37bdcee8283ae669ab7e4374715ec5d2.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/37bdcee8283ae669ab7e4374715ec5d2.png)

注意：看完后在命令行中按 `ctrl+c`取消`hexo server`命令
### 2.3 Hexo配置(**_config.yml**)

下面罗列一下主要会用到的配置！

其余的可以参考官方文档：[配置 | Hexo](https://hexo.io/zh-cn/docs/configuration)
#### 2.3.1 网站

|参数|描述|
|---|---|
|`title`|网站标题|
|`subtitle`|网站副标题|
|`description`|网站描述|
|`keywords`|网站的关键词。支持多个关键词。|
|`author`|您的名字|
|`language`|网站使用的语言。对于简体中文用户来说，使用不同的主题可能需要设置成不同的值，请参考你的主题的文档自行设置，常见的有 `zh-Hans`和 `zh-CN`。|
|`timezone`|网站时区。Hexo 默认使用您电脑的时区。请参考 [时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 进行设置，如 `America/New_York`, `Japan`, 和 `UTC` 。一般的，对于中国大陆地区可以使用 `Asia/Shanghai`。|

其中，`description` 主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author` 参数用于主题显示文章的作者。
#### 2.3.2 网址

|参数|描述|默认值|
|---|---|---|
|`url`|网址, 必须以 `http://` 或 `https://` 开头||
|`root`|网站根目录|`url's pathname`|
|`permalink`|文章的 [永久链接](https://hexo.io/zh-cn/docs/permalinks) 格式|`:year/:month/:day/:title/`|
|`permalink_defaults`|永久链接中各部分的默认值||
|`pretty_urls`|改写 [`permalink`](https://hexo.io/zh-cn/docs/variables) 的值来美化 URL||
|`pretty_urls.trailing_index`|是否在永久链接中保留尾部的 `index.html`，设置为 `false` 时去除|`true`|
|`pretty_urls.trailing_html`|是否在永久链接中保留尾部的 `.html`, 设置为 `false` 时去除 (_对尾部的 `index.html`无效_)|`true`|

> **网站存放在子目录**
> 
> 如果您的网站存放在子目录中，例如 `http://example.com/blog`，则请将您的 `url` 设为 `http://example.com/blog` 并把 `root` 设为 `/blog/`。

|                                                                                                                                                      |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| # 比如，一个页面的永久链接是 http://example.com/foo/bar/index.html  <br>pretty_urls:  <br>  trailing_index: false  <br># 此时页面的永久链接会变为 http://example.com/foo/bar/ |

### 2.4 hexo命令(可以不看)

主要罗列后续会用到了hexo命令，其余的参考官方文档[指令 | Hexo](https://hexo.io/zh-cn/docs/commands)

#### 2.4.1 init

```shell
hexo init [folder]
```

新建一个网站。如果没有设置 `folder` ，Hexo 默认在目前的文件夹建立网站。

#### 2.4.2 generate

```shell
hexo generate
hexo g
```

生成静态文件。
#### 2.4.3 server

```shell
hexo server
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

#### 2.4.4 deploy

```shell
hexo deploy
```

部署网站，会在博客目录下的source文件夹的内容生成一个public文件夹，静态网站主要是部署public文件夹的内容
#### 2.4.5 clean

```shell
hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。
## 3 GitHub

如果你还没有github账户，请去注册吧，此文暂不赘述这过程啦
### 3.1 GitHub操作

在你的github账户上，新建一个仓库`new repository`![a5d76150a28a47d94950df410c720e3e.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a5d76150a28a47d94950df410c720e3e.png)

仓库填写如下![ba75774d3ab98fbcd945932bcfe0d90f.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ba75774d3ab98fbcd945932bcfe0d90f.png)


- **注意选择public**
- **注意仓库的取名格式：用户名.github.io(这将是以后的访问域名)**

到这里代表我们Github账号以及仓库都已经创建完毕，可以把下面这段仓库的地址复制下来留着后面配置时会用到![56aad5efa7d7ec625d454079eb2514ca.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/56aad5efa7d7ec625d454079eb2514ca.png)

推荐使用ssh，https容易后期出现bug（ssh需要配置，详情百度哈）
### 3.2 Hexo部署到GitHub

第一步：hexo安装`hexo-deployer-git`插件，在**博客目录**右键鼠标，选择gitbash，输入如下指令
```shell
npm install hexo-deployer-git --save
```

效果如图：![7c49588afccedea9a2a84933f6aadb64.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7c49588afccedea9a2a84933f6aadb64.png)



第二步：打开博客目录下的`_config.yml`文件，设置如下
```yml
deploy:  
  type: git # 类型填git  
  repo: https://github.com/xxx/xxx # 你的Github仓库地址  
  branch: master # 分支名称。默认填写 master 如果您使用的是 GitHub ，程序会尝试自动检测。
```

效果如图：![524348637affee0b8a8f6e9143bc086e.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/524348637affee0b8a8f6e9143bc086e.png)


第三步：还是在gitbash命令中，**一键三连**
```shell
hexo clean
hexo g
hexo d
```

第四步：就可以看到，github仓库中，上传了我们的文件了![d3e97471b61e0cd74467839bc0b9e450.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d3e97471b61e0cd74467839bc0b9e450.png)

第五步：检查仓库设置中，pages页面，是否生成了 `仓库名.github.io`的路径![2689a1b9530d98924f51becd3a0b8e88.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2689a1b9530d98924f51becd3a0b8e88.png)

第六步：在浏览器中，直接输入 `仓库名.github.io`，即可访问你的网站啦

**注意：每次更新本地博客目录的内容后，都需要一键三连**
### 3.3 写文章并发布文章

在博客目录中使用gitbash，输入命令
```shell
hexo new “my article/我的文章”
```

然后，再次一键三连，就可以看到这篇文章在网站上了

**刚刚部署后需要一定时间加载，出现未更新等待一段时间即可**
### 3.4 备份博客目录

由于使用**hexo中的git插件**，我们部署在github仓库中的只是`hexo generate`出来的`public文件`（即静态网站资源），而其他文件是没有保存备份的。

所以，我们可以直接给整个目录进行备份，流程如下

1. 还是在github上创建仓库，此时仓库注意是私有的![92c66b69cd828e231846e0f7e0c205db.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92c66b69cd828e231846e0f7e0c205db.png)

2. 创建成功后如图![ed7c7861c413e9040799e741f1a5d0b0.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ed7c7861c413e9040799e741f1a5d0b0.png)

3. 在我们的博客目录下，创建一个`.gitignore文件`。**注意文件名**，文件内容如下
```git
.DS_Store  
Thumbs.db  
db.json  
*.log  
node_modules/  
public/  
.deploy*/  
.vscode/  
.idea/  
/.idea/  
.deploy_git*/  
.idea  
themes/
```

5. 在我们的博客目录下，输入如下命令，即可完成备份
```git
git init
git add .
git cmomit -m "first commit"
git remote add origin 你的仓库地址（HTTPS/SSH）
git push -u origin main/master（看清楚分支）
```

5. 最后，可以看到我们刚刚创建的仓库，就有我们整个博客（除了忽略的）的内容啦![05ea0c21cf88c7e080ae1dfc8f81b8b0.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/05ea0c21cf88c7e080ae1dfc8f81b8b0.png)

`补充，每次更新完博客后，需要备份则重复一下指令`
```git
git add *  
git commit -m "本次提交的内容"
git push -u origin main/master（看清楚分支）
```

也可以使用本地可视化工具，具体见文章[3_gitkralen的破解及汉化](../../../../1_感想或实用/7_Linux&Git&IDEA/3_Git/3_gitkralen的破解及汉化.md)

## 4 个人域名

### 4.1 前言
国内主要的购买域名网站无非就是腾讯云、阿里云、华为云，随便找一个注册即可。

tips：如果你使用的是Hexo等静态网页技术搭建的框架是可以不用买域名的，GitHub提供了一个二级域名，格式为：https://用户名.github.io。

![833cf0be0a58d65b11fab82859c757e2.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/833cf0be0a58d65b11fab82859c757e2.png)


进入购买网站，在上图箭头指向处输入你想要注册的域名，加入购物车后付款即可。

这里注意的是顶级域名的选择，.com域名价格较贵而且极有可能已经被注册，可以选择.cn或者.net这类也还算常见的顶级域名。

### 4.2 域名注册

博主使用的是阿里云，伙伴们需要注册域名的话，可以在[域名注册](https://domain.aliyun.com/)先查询一下嗷

域名的价格不一，com后缀的偏贵，cn后缀其次，其他域名一般都比较便宜，博主目前的域名`dakkk.top` 花了78r，10年试用期😃😃😃

### 4.3 域名使用

当你购买域名后，在阿里控制台中，找到域名选项，会出现域名列表!![08abace87b8634efb8b825c5a8032021.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/08abace87b8634efb8b825c5a8032021.png)
我们点击管理，选择右侧的域名解析，然后添加记录![713de6301adcf8a636aa05d2e3e71474.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/713de6301adcf8a636aa05d2e3e71474.png)

- 两条记录如上图所示，主要是记录值填写`xxxx.github.io`即可

接下来，我们来到github仓库的页面进行配置

同样是，setting中的Pages页面![254c3429ed5128d4d296d0d834c6395a.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/254c3429ed5128d4d296d0d834c6395a.png)

- 在Custom domain选项中，将你购买的域名填入即可
- 注意：开启下面的Enforce HTTPS功能哈

## 5 Hexo主题

博主开发个人博客主题，使用的是Jerry大佬的butterfly主题，有需要的可以参考大佬的文档[Butterfly 安裝文檔(一) 快速開始 | Butterfly](https://butterfly.js.org/posts/21cfbf15/)

我们主要介绍安装主题的通用设置

### 5.1 安装

主题的安装一般有2种方法：Git安装 和 npm安装

Git安装：使用clone指令拉取主题文件到博客目录中，主题存放的路径是博客目录中的themes这个目录下

npm安装：使用install指令，直接安装主题，**通过 npm 安装并不会在 themes 里生成主题文件夹，而是在 node_modules 里生成**

### 5.2 应用

修改 博客目录下的 `_config.yml`，把主题改为` butterfly`
```yml
theme: butterfly
```

### 5.3 安装插件

一般来说，很多主题都需要两个插件：pug和stylus，所以我们可以提前安装，避免后面出现问题

在博客目录下安装，安装指令如下
```shell
npm install hexo-renderer-pug hexo-renderer-stylus --save
```