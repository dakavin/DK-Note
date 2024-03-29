一、yarn的简介：
Yarn是facebook发布的一款取代npm的包管理工具。

二、yarn的特点：

速度超快。
Yarn 缓存了每个下载过的包，所以再次使用时无需重复下载。 同时利用并行下载以最大化资源 利用率，因此安装速度更快。
超级安全。
在执行代码之前，Yarn 会通过算法校验每个安装包的完整性。
超级可靠。
使用详细、简洁的锁文件格式和明确的安装算法，Yarn 能够保证在不同系统上无差异的工作。

三、yarn的安装:
```shell
npm install -g yarn
npm uninstall yarn -g  //yarn卸载
```

配置环境变量
复制bin地址：`E:\Nodejs\GlobalNodeModules\node_modules\yarn\bin`

![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/20240329220006.png)

![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/20240329220011.png)

五、yarn的常用命令：

```shell
yarn -v  // 查看yarn 版本
yarn config list  // 查看yarn配置
yarn config get registry   // 查看当前yarn源

// 修改yarn源（此处为淘宝的源）
yarn config set registry https://registry.npm.taobao.org  

// yarn安装依赖
yarn add 包名          // 局部安装
yarn global add 包名   // 全局安装

// yarn 卸载依赖
yarn remove 包名         // 局部卸载
yarn global remove 包名  // 全局卸载（如果安装时安到了全局，那么卸载就要对应卸载全局的）

// yarn 查看全局安装过的包
yarn global list  
```

```shell

npm install -g yarn  // 安装yarn 
yarn --version       // 安装成功后，查看版本号
md yarn   // 创建文件夹 yarn  
cd yarn   // 进入yarn文件夹 

初始化项目 
yarn init // 同npm init，执行输入信息后，会生成package.json文件

yarn的配置项： 
yarn config list // 显示所有配置项
yarn config get <key> //显示某配置项
yarn config delete <key> //删除某配置项
yarn config set <key> <value> [-g|--global] //设置配置项

安装包： 
yarn install         //安装package.json里所有包，并将包及它的所有依赖项保存进yarn.lock
yarn install --flat  //安装一个包的单一版本
yarn install --force         //强制重新下载所有包
yarn install --production    //只安装dependencies里的包
yarn install --no-lockfile   //不读取或生成yarn.lock
yarn install --pure-lockfile //不生成yarn.lock
添加包（会更新package.json和yarn.lock）：

yarn add [package] // 在当前的项目中添加一个依赖包，会自动更新到package.json和yarn.lock文件中
yarn add [package]@[version] // 安装指定版本，这里指的是主要版本，如果需要精确到小版本，使用-E参数
yarn add [package]@[tag] // 安装某个tag（比如beta,next或者latest）

//不指定依赖类型默认安装到dependencies里，你也可以指定依赖类型：
yarn add --dev/-D // 加到 devDependencies
yarn add --peer/-P // 加到 peerDependencies
yarn add --optional/-O // 加到 optionalDependencies

//默认安装包的主要版本里的最新版本，下面两个命令可以指定版本：
yarn add --exact/-E // 安装包的精确版本。例如yarn add foo@1.2.3会接受1.9.1版，但是yarn add foo@1.2.3 --exact只会接受1.2.3版
yarn add --tilde/-T // 安装包的次要版本里的最新版。例如yarn add foo@1.2.3 --tilde会接受1.2.9，但不接受1.3.0

yarn publish // 发布包
yarn remove <packageName>  // 移除一个包，会自动更新package.json和yarn.lock
yarn upgrade // 更新一个依赖: 用于更新包到基于规范范围的最新版本
yarn run   // 运行脚本: 用来执行在 package.json 中 scripts 属性下定义的脚本
yarn info <packageName> 可以用来查看某个模块的最新版本信息

缓存 
yarn cache 
yarn cache list # 列出已缓存的每个包 
yarn cache dir # 返回 全局缓存位置 
yarn cache clean # 清除缓存
```

五、npm 与 yarn命令比较:

比如说你的项目模块依赖是图中描述的，@1.2.1代表这个模块的版本。在你安装A的时候需要安装依赖C和D，很多依赖不会指定版本号，默认会安装最新的版本，这样就会出现问题：比如今天安装模块的时候C和D是某一个版本，而当以后C、D更新的时候，再次安装模块就会安装C和D的最新版本，如果新的版本无法兼容你的项目，你的程序可能就会出BUG，甚至无法运行。这就是npm的弊端，而yarn为了解决这个问题推出了yarn.lock的机制，这是作者项目中的yarn.lock文件

注意：这个文件不要手动修改它，当你使用一些操作如yarn add时，yarn会自动更新yarn.lock。

