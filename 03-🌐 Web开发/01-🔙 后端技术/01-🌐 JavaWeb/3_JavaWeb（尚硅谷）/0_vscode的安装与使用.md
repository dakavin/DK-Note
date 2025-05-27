---
文章标题: "[[0_vscode的安装与使用]]" 
文章作者: Dakkk
文章概要: |
  介绍VSCode的安装配置流程，包括前端开发常用插件推荐、工作空间创建、Live Server使用及常见设置调整，适合前端开发入门者快速搭建开发环境。
tags:
- "VSCode"
- "前端开发"
- "开发环境"
- "IDE配置"
- "Live Server"
- "开发工具"
- "编辑器插件"
- "前端入门"
相关文章:
- "[[1_VsCode & Mingw]]"
- "[[2_安装VsCode 开发工具]]"
- "[[2_环境搭建]]"
- "[[4_vscode整合ds和配置linux内核源码]]"
- "[[0_快捷键大全]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/01-🌐 JavaWeb/3_JavaWeb（尚硅谷）/0_vscode的安装与使用.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-03-28 10:36:00
修改时间: 2025-05-27 23:34:19
---

+ 前端工程师“Front-End-Developer”源自于美国。

+ 最初所有的开发工作都是由后端工程师完成的，随着业务越来越繁杂，工作量变大，于是我们将项目中的可视化部分和一部分交互功能的开发工作剥离出来，形成了前端开发。

+ PRD（产品原型-产品经理） - PSD（视觉设计-UI工程师） - HTML/CSS/JavaScript（PC/移动端网页，实现网页端的视觉展示和交互-前端工程师）

+ 前端工程师比较推崇的一款开发工具就是visual  studio code,下载地址为:https://code.visualstudio.com/

> `1 安装过程`

安装过程比较简单,一路next,注意安装路径不要有中文,空格和特殊符号即可

> `2 安装插件`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7de0ee6dc2f6c5a10a4a91d4596fa052.png)

- Auto Rename Tag 自动修改标签对插件
- Chinese Language Pack 汉化包
- HTML CSS Support HTML CSS 支持
- Intellij IDEA Keybindings IDEA快捷键支持
- Live Server 实时加载功能的小型服务器
- open in browser 通过浏览器打开当前文件的插件
- Prettier-Code formatter 代码美化格式化插件
- Vetur VScode中的Vue工具插件
- vscode-icons 文件显示图标插件
- Vue 3 snipptes 生成VUE模板插件
- Vue language Features Vue3语言特征插件

> `3 准备工作空间 ` 
+ `直接用vscode打开某个目录即可`直接将某个目录作为项目代码存放的根目录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/002c12542fcb91dac679fd06cfa2f9a0.png)
> `4 在工作空间下创建目录和文件`
+ 点击带有"+"号的按钮即可创建文件或者目录![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/540c2f5104eda1387210dd9b5f16535f.png)
+ 在html中,输入"  !  " 并回车即可快速出现html的基本结构![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79a141daf200ba5a5738ff7286e67b4c.gif)
> `5 通过live Server 小型服务器运行项目`
+ 点击右下角Go Live , 或者在html编辑视图上右击 open with live Server  ,会自动启动小型服务器,并自动打开浏览器访问当前资源![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4a602d5f8cc4ba901b793a44b99ff104.png)
+ Live Server 可实现实时加载功能
+ Live Server使用完毕后,要记得关闭![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce9756238e3ca5e0866b923b583b2c84.png)
> `6 其他常见设置`

+ 设置字体:    齿轮>search>搜索    "字体大小"
+ 设置字体大小可以用滚轮控制:  齿轮>设置>搜索 "Mouse Wheel Zoom"
+ 设置左侧树缩进: 齿轮>设置>搜索 "树缩进"
+ 设置文件夹折叠:  齿轮>设置>搜索 "compact" 取消第一个勾选
+ 设置编码自动保存: 齿轮> 设置> 搜索 "Auto Save" ,选择为"afterDelay"