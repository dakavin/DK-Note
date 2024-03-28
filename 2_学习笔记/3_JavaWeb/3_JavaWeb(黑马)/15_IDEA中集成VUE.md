# IDEA中集成VUE

**一、在IDEA中配置vue插件**

点击File-->Settings-->Plugins-->搜索vue.js插件进行安装，下面的图中我已经安装好了。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922184355938.png)
- 配置vue文件

**二、搭建node.js环境**

安装[node](https://so.csdn.net/so/search?q=node&spm=1001.2101.3001.7020).js 可以去[官网下载](https://nodejs.org/zh-cn/)：安装过程就很简单，直接下一步就行![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922184402282.png)

- 测试是否安装成功：要使用管理员方式打开命令行cmd
- 安装完成之后，打开命令行工具,输入node -v如果出现版本号，则说明安装成功，npm包管理器是集成在node中的，所以，直接
- 输入`npm -v` 就会显示npm版本信息![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922184405277.png)
- node环境已经安装成功，由于有些npm有些资源被屏蔽或者是国外资源的原因，经常会导致用npm安装依赖包的时候失败，所以还需要安装npm的国内镜像----cnpm

**三、安装cnpm（注意都是管理员方式运行）**

在命令行中输入`npm install -g cnpm --registry=http://registry.npm.taobao.org`然后等待安装，安装完成之后，我们就可以用cnpm代替npm来安装依赖包了。![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922184409182.png)

**四、安装vue-cli脚手架构建工具（注意都是管理员方式运行）**

在命令行中运行命令`cnpm install -g vue-cli,`然后等待安装完成，通过以上三步，我们的环境和工具都准备好了，接下来就开始使用![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922184412342.png)

vue-cli来构建项目

参考网站：[IDEA部署Vue项目_sky_817的博客-CSDN博客](https://blog.csdn.net/sky_817/article/details/89175223)



