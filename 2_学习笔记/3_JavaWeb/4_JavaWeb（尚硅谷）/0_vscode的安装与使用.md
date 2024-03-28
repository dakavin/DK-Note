
+ 前端工程师“Front-End-Developer”源自于美国。

+ 最初所有的开发工作都是由后端工程师完成的，随着业务越来越繁杂，工作量变大，于是我们将项目中的可视化部分和一部分交互功能的开发工作剥离出来，形成了前端开发。

+ PRD（产品原型-产品经理） - PSD（视觉设计-UI工程师） - HTML/CSS/JavaScript（PC/移动端网页，实现网页端的视觉展示和交互-前端工程师）

+ 前端工程师比较推崇的一款开发工具就是visual  studio code,下载地址为:https://code.visualstudio.com/

> `1 安装过程`

安装过程比较简单,一路next,注意安装路径不要有中文,空格和特殊符号即可

> `2 安装插件`

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240313142333.png)

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
+ `直接用vscode打开某个目录即可`直接将某个目录作为项目代码存放的根目录![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240313142912.png)
> `4 在工作空间下创建目录和文件`
+ 点击带有"+"号的按钮即可创建文件或者目录![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240313142942.png)
+ 在html中,输入"  !  " 并回车即可快速出现html的基本结构![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/newhtml.gif)
> `5 通过live Server 小型服务器运行项目`
+ 点击右下角Go Live , 或者在html编辑视图上右击 open with live Server  ,会自动启动小型服务器,并自动打开浏览器访问当前资源![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240313143201.png)
+ Live Server 可实现实时加载功能
+ Live Server使用完毕后,要记得关闭![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020240313143230.png)
> `6 其他常见设置`

+ 设置字体:    齿轮>search>搜索    "字体大小"
+ 设置字体大小可以用滚轮控制:  齿轮>设置>搜索 "Mouse Wheel Zoom"
+ 设置左侧树缩进: 齿轮>设置>搜索 "树缩进"
+ 设置文件夹折叠:  齿轮>设置>搜索 "compact" 取消第一个勾选
+ 设置编码自动保存: 齿轮> 设置> 搜索 "Auto Save" ,选择为"afterDelay"