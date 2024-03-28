## 1、下载 和 安装

### 1.1 下载

Git官网提供对应得下载链接页面：
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220124.png)

下载地址：[https://central.github.com/deployments/desktop/desktop/latest/win32s](https://central.github.com/deployments/desktop/desktop/latest/win32s)
### 1.2 安装

无安装过程，安装完成后，弹出应用界面
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220140.png)
### 1.3 配置

点击软件得File菜单后，选择Options, 设定软件得操作用户名称及对应得邮箱地址
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220314.png)

### 1.4 主题样式

可以根据自己得偏好设定软件主题样式

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220320.png)

### 1.5 全屏

如果觉得软件界面比较小，可以适当进行调整或全屏
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220342.png)

## 2、创建本地仓库

### 2.1 创建仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220354.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220403.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220409.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220415.png)

### 2.2 切换仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220420.png)

### 2.3 删除仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220426.png)

## 3、文件操作

### 3.1 新增文件

当工作区域创建了一份新文件，工具可以自动识别并进行对应得显示
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220437.png)

此时Git仓库中并没有这份文件，所以需要执行commit操作，将文件保存到Git仓库中。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220446.png)
### 3.2 忽略文件

如果某一个文件或某一类得文件，不想被Git软件进行管理。可以在忽略文件中进行设定
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220457.png)

### 3.3 修改文件

修改文件只是将工作区域得文件进行修改，但是对于Git软件来讲，其实本质上还是提交，因为底层会生成新得文件
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220543.png)
### 3.4 删除文件

删除文件对于Git软件来讲，依然是一个提交
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220554.png)
提交后，最新版本得文件也会被“删除”

### 3.5 历史记录

如果存在多次得提交操作得话，可以查看提交得历史记录
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220610.png)

## 4、分支操作

### 4.1 默认分支

软件创建仓库时，默认创建得分支为main
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220621.png)

点击右键可以改名
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220629.png)

### 4.2 创建分支

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220636.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220641.png)

### 4.3 切换分支

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220645.png)

### 4.4 删除分支

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220651.png)

### 4.5 合并分支

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220700.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220706.png)

## 5、标签操作

### 5.1 创建标签

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220712.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220719.png)

### 5.2 删除标签

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220724.png)

## 6、远程仓库

### 6.1 克隆仓库

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220733.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220745.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220801.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220809.png)

### 6.2 拉取文件

远程仓库更新了文件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220819.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220825.png)

### 6.3 推送文件

本地创建新文件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220838.png)

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220845.png)