
<h1>目录</h1>

```table-of-contents
```

## 1 阿里云OSS实现同步
### 1.1 Obsidian软件安装Remotely Save插件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/03dd168272e80cf776e882e73218f3ce.png)



### 1.2 创建一个阿里云的OSS服务

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/64c6ea4259e74e6d34c764f9793cd66e.png)



### 1.3 开始同步

#### 1.3.1 在OSS平台上创建一个Bucket

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c873ba92bdde867ec214b23ccdde5f7f.png)



Bucket名称：skill-tip
Endpoint：oss-cn-guangzhou.aliyuncs.com

#### 1.3.2 开始填写Remotely Save信息

##### 1.3.2.1 填写信息

- 先将上面的两个信息进行填写![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5a4c93a37d0d6e6415e8f889664c1fc.png)


- Region，可以查询文档[访问域名和数据中心_对象存储 OSS-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/oss/user-guide/regions-and-endpoints)

- 选择华南3（广州）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2c899ddc15c21d03465a42199fee6317.png)


- oss-cn-guangzhou，填写进去![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/627d9fc4b890b1b40d422bec092915ae.png)


- 查看自己阿里云账户所在的key id 和 key![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c11284d91bced52f65b08020c3b6663f.png)



```shell
AccessKey ID

LTAI5t9yLGp9bdSG17s7pgA9

AccessKey Secret

S4FxfwoZ1pFCpphpWFjR1C23ynJw8d
```

- 完成填写![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/882493310d7e1318660289f025f8b937.png)


- 检查连接是否正常
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/db52700a67e221b14b163e3f5e1e0c3a.png)



- 开启三项基本配置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c9bf4b2196a1f61dcc1218f40f9dbf30.png)


##### 1.3.2.2 测试

- 点击右边按钮开始同步![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/10e4d7881e3e44bc5c26ff61c8d98651.png)


- 检测Bucket是否同步完成![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5710c4f32881384e89622babc00addf9.png)



### 1.4 移动端设置

- 下载Obsdian
- 创建一个名字相同的仓库![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/528bebf0384a2bd0d639de9af0e3d313.png)


- 安装Remotely Save插件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/89d44843b85e6575147ac00c1f8fddff.png)



- 电脑端打开插件的QR码，手机开启扫描![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/269be7dab94a247c89c2593ce37eaf40.png)


- 手机扫描成功后自行导入配置，并完成同步![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2dcf11bf350d95dd6eb50684246a1b02.png)



## 2 Git实现仓库备份

- `这里的远程仓库我们使用Gitee` [工作台 - Gitee.com](https://gitee.com/)

### 2.1 安装Git

- 自行搜索教程即可

### 2.2 Git初始化

- 加入仓库所在的文件夹，点击鼠标右键，选择GitBash![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36a85c43ce9328dcd6c41ec4a557228a.png)


- 按照命令依次输入
	- git init  （初始化本地Git仓库）
	- git config --global user.name dakkk   (用户注册)
	- git config --global user.email mikeylay@126.com    (用户注册)
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/698ed08ba88a23dcbb189b82d322e9b1.png)



- 在当前文件夹，创建 .gitignore文件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4e6a039618c285dd61a9c85115467c8.png)


### 2.3 注册Gitee账户，创建仓库

- 注册Gitee账户，自行注册即可

- 创建Gitee仓库![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/966e67d16083fe58d08c11e86ad69b5e.png)


- 复制仓库的Http链接![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2ca35557c7218201000b34590cd599e6.png)


- 开始git同步，按照下述命令执行
	- git remote add origin https://gitee.com/Dakkk_mike/skill-tip.git
	- git add --all
	- git commit -m "commit all"
	- git push -u origin master

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6ea7a5fd18a1a5e71ce8b4b77cc40701.png)



- 查看远程仓库，发现本地仓库内容已完成上传![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/421614118ff454235105d603ef1d3fa8.png)
- 解除一个项目的git配置
	- rm -rf .git
	- 删除远程仓库


### 2.4 使用Obsidian Git插件实现自动同步

- 下载Obsidian Git插件![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7adf6f725586bed09e5eb51e69b217f6.png)


- 配置本地Git的路径（没有配置系统的环境变量）![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/695e3667b67fe7fb452a6c34928f1ea0.png)


- 设置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23a059fc70d0024bc6c97b3f5ff7a846.png)



