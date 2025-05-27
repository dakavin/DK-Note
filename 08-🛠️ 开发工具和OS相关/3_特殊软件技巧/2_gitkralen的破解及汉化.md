---
文章标题: "[[2_gitkralen的破解及汉化]]"
文章作者: Dakkk
文章概要: |
  文章详细介绍了GitKraken 8.2.0-9.x版本的破解及更新屏蔽方法，包括Node.js、Yarn环境配置，使用GitHub上的破解工具进行激活。此外，还提供了GitKraken汉化参考链接。
tags:
  - GitKraken
  - 破解
  - 软件激活
  - Yarn
  - Git
  - hosts文件
相关文章:
  - "[[3_Hexo+GitHub+个人域名 博客搭建教程]]"
  - "[[1_本章代码及文章获取]]"
  - "[[1_克隆项目]]"
  - "[[1_Part1-BUG]]"
  - "[[1_Vue 3环境安装 和 blog前端项目搭建]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/3_特殊软件技巧/2_gitkralen的破解及汉化.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐ 基础知识
创建时间: 2025-05-06 16:48:05
修改时间: 2025-05-28 01:03:09
---

## 1 破解要求环境

- Node.js Version>=12 LTS
- yarn
- GitKraKen v8.2.0-9.x
## 2 gitkarlen下载地址

[gitkraken](https://www.gitkraken.com/git-client/try-free)
## 3 破解步骤

下载之后需要先运行安装下载的GitKraken（它会自动安装到AppData/Local/gitkraken9.x中）。

安装完毕后将会自动打开gitkraken，此时直接将其关闭即可。
### 3.1 从npm安装yarn

- 使用本机的cmd/powershell，需要管理员运行，否则不安装yran
- 注意nodejs的path也设置过了（一般不用设置）
- 运行如下指令
```shell
npm install --global yarn
npm uninstall yarn -g  //yarn卸载
```
### 3.2 配置yarn的环境及使用

- 找到yarn的安装目录
- 配置环境变量
- 复制bin地址：`E:\Nodejs\GlobalNodeModules\node_modules\yarn\bin`

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7272b1db18d34bd65c3cd9ca2efc3ecd.png)

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ac92d3c8e24af904f069a25e60e987b4.png)

- 直接在cmd中，打开`yarn.cmd`
```shell
yarn.cmd //因为已经配置过path了，在任何路径可以直接打开
```

## 4 克隆破解软件

- 找到一个放破解软件的目录，使用git bash
```shell
git clone https://github.com/qsshs/GitKraKen-Crack.git
```

- 进入软件所在目录
```shell
cd GitKraKen-Crack

yarn install   //注意要先开启yarn.cmd

yarn build

yarn gitcracken patcher  //输入完成后，等待几十秒就好了
```

## 5 屏蔽更新

### 5.1 方式一：修改Host

在`C: \ Windows \ System32 \ drivers \ etc`下的hosts文件中最下面添加一行`127.0.0.1 release.gitkraken.com`即可。
### 5.2 方式二：直接删除update.exe文件

直接在AppData/Local/gitkraken目录下，将update.exe删掉，也可禁止gitkraken自动更新。

## 6 启动

- 注意：要在AppData/Local/gitkraken/app-9.x/目录下的gitkraken.exe启动
- 可以创建快捷方式等

- 展示一张图![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7e31bd35a97ece29349157248a403cf5.png)
## 汉化

参考地址：https://github.com/yk47g/gitkraken-chinese