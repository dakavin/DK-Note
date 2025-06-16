---
文章标题: "[[8_GitHub Desktop客户端]]"
文章作者: Dakkk
文章概要: |
  本文提供了GitHub Desktop客户端的详细图文教程，涵盖了软件的下载、安装、基本配置，以及本地仓库的创建、文件操作、分支管理、标签使用，最后演示了远程仓库的克隆、拉取与推送等核心功能。
tags:
  - GitHub Desktop
  - Git
  - 版本控制
  - GUI客户端
  - 教程
  - 仓库管理
  - 分支操作
相关文章:
  - "[[../04-💡 实用技巧/5_Commit 提交规范]]"
  - "[[1_本章代码及文章获取]]"
  - "[[14_RESTful规范]]"
  - "[[3_Hexo+GitHub+个人域名 博客搭建教程]]"
  - "[[../../../01-📚 知识管理/06-🛠️ 技巧/1_obsidian技巧/3_Obsidian实现阿里云同步和Git备份]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/02-🔧 版本控制/03-🌐 GitHub&GitLab/8_GitHub Desktop客户端.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 1 下载 和 安装

### 1.1 下载

Git官网提供对应得下载链接页面：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce79e2dd74a5589fc278f1f5d4447cbf.png)

下载地址：[https://central.github.com/deployments/desktop/desktop/latest/win32s](https://central.github.com/deployments/desktop/desktop/latest/win32s)
### 1.2 安装

无安装过程，安装完成后，弹出应用界面
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c595b930cf45f2cf6502ed65d4b150c6.png)
### 1.3 配置

点击软件得File菜单后，选择Options, 设定软件得操作用户名称及对应得邮箱地址
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d7470000c8ad5166494ba095ed012122.png)

### 1.4 主题样式

可以根据自己得偏好设定软件主题样式

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e90ce3e511650c616d23e6fc6665296a.png)

### 1.5 全屏

如果觉得软件界面比较小，可以适当进行调整或全屏
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ecde0394779c33d130b90d3da0f0713e.png)

## 2 创建本地仓库

### 2.1 创建仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/226774f90580dcfdc8816a5a9c2550ab.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c00453448340d379edc10fe9b5b80726.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d22ea0c5c4f30a885e63b912acbf27eb.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/84c00486137ef6227ae84886b5a276ee.png)

### 2.2 切换仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cedf1a382f942373f235d4a73312f519.png)

### 2.3 删除仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b8be4be285fdc46ac5e5d5dacf944c1a.png)

## 3 文件操作

### 3.1 新增文件

当工作区域创建了一份新文件，工具可以自动识别并进行对应得显示
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d8b4e997636413a2c0f41a8ad8470b14.png)

此时Git仓库中并没有这份文件，所以需要执行commit操作，将文件保存到Git仓库中。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/315517b753d2e3dcd678b98b1cab342f.png)
### 3.2 忽略文件

如果某一个文件或某一类得文件，不想被Git软件进行管理。可以在忽略文件中进行设定
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e9fa741a6831124f1462c2de756edb8.png)

### 3.3 修改文件

修改文件只是将工作区域得文件进行修改，但是对于Git软件来讲，其实本质上还是提交，因为底层会生成新得文件
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce6b563f7a3a52c34610d14b4a357269.png)
### 3.4 删除文件

删除文件对于Git软件来讲，依然是一个提交
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/38aa77e5c6ec95106bb01d46084b35f9.png)
提交后，最新版本得文件也会被“删除”

### 3.5 历史记录

如果存在多次得提交操作得话，可以查看提交得历史记录
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4020317346ceab7d67eea48615a9c0d7.png)

## 4 分支操作

### 4.1 默认分支

软件创建仓库时，默认创建得分支为main
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/122545a9289172fdb9c572caca508064.png)

点击右键可以改名
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/530e97d15abda038d5a2697bd2e669bc.png)

### 4.2 创建分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75fe23645ba03a9b541a60aa03eca285.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9f185765dcbb5363fe8805ae91d1c614.png)

### 4.3 切换分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7b955ec1ac0164beeabcae64acf107e.png)

### 4.4 删除分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/637e4836d944780c0fcc3e4820afcee9.png)

### 4.5 合并分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b7049652f326dae04a48be465efd4e1.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8320a2c0f8cb67d8e7a9832aca0baa9.png)

## 5 标签操作

### 5.1 创建标签

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c30c82e72f070074c2c57af4a38691be.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/66c2086848cec548b2dae39fa00255e3.png)

### 5.2 删除标签

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ffeec8ed1c9fd10078979cb4c97c46a.png)

## 6 远程仓库

### 6.1 克隆仓库

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8dfaaace38308fa3d5393b39810d6555.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eeb32b41cb45175a80d8262fb40d7aa5.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fd6862935d878be0a946cb16f33381a1.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7aa445c17ecd6160233b182b8b5dc126.png)

### 6.2 拉取文件

远程仓库更新了文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71387caf4f3d47bd40e8d5796a882061.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3b665cdf8c3e19d1f393fb5d39b20761.png)

### 6.3 推送文件

本地创建新文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b703a07b0cb9493fbeac23d1ac869f3.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb489ed791bf54c7d21c2b91bec0c002.png)