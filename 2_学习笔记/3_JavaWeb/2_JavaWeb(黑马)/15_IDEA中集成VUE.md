

## 1 在IDEA中配置vue插件 

点击File-->Settings-->Plugins-->搜索vue.js插件进行安装，下面的图中我已经安装好了。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f79a7512e8c265c395e4c653ad950ab.png)
- 配置vue文件

## 2 搭建node.js环境

安装[node](https://so.csdn.net/so/search?q=node&spm=1001.2101.3001.7020).js 可以去[官网下载](https://nodejs.org/zh-cn/)：安装过程就很简单，直接下一步就行![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d569bcf58d31efa87033da9aa1289dc5.png)

- 测试是否安装成功：要使用管理员方式打开命令行cmd
- 安装完成之后，打开命令行工具,输入node -v如果出现版本号，则说明安装成功，npm包管理器是集成在node中的，所以，直接
- 输入`npm -v` 就会显示npm版本信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb63b0e71b5b508dd254fd0762604e74.png)
- node环境已经安装成功，由于有些npm有些资源被屏蔽或者是国外资源的原因，经常会导致用npm安装依赖包的时候失败，所以还需要安装npm的国内镜像----cnpm

## 3 安装cnpm（注意都是管理员方式运行）

在命令行中输入`npm install -g cnpm --registry=http://registry.npm.taobao.org`然后等待安装，安装完成之后，我们就可以用cnpm代替npm来安装依赖包了。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4132fdc9a4ea726f93d38732870ee2c.png)

## 4 安装vue-cli脚手架构建工具（注意都是管理员方式运行）

在命令行中运行命令`cnpm install -g vue-cli,`然后等待安装完成，通过以上三步，我们的环境和工具都准备好了，接下来就开始使用![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f2f6cd33288ec5d00deb59bba60de805.png)

vue-cli来构建项目

参考网站：[IDEA部署Vue项目_sky_817的博客-CSDN博客](https://blog.csdn.net/sky_817/article/details/89175223)



