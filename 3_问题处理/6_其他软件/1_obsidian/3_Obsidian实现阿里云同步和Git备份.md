
<h1>目录</h1>

```table-of-contents
```

## 1 阿里云OSS实现同步
### 1.1 Obsidian软件安装Remotely Save插件

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223707541.png)



### 1.2 创建一个阿里云的OSS服务

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223740410.png)



### 1.3 开始同步

#### 1.3.1 在OSS平台上创建一个Bucket

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223751895.png)



Bucket名称：skill-tip
Endpoint：oss-cn-guangzhou.aliyuncs.com

#### 1.3.2 开始填写Remotely Save信息

##### 1.3.2.1 填写信息

- 先将上面的两个信息进行填写![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223932225.png)


- Region，可以查询文档[访问域名和数据中心_对象存储 OSS-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/oss/user-guide/regions-and-endpoints)

- 选择华南3（广州）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223940955.png)


- oss-cn-guangzhou，填写进去![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223944192.png)


- 查看自己阿里云账户所在的key id 和 key![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223947070.png)



```shell
AccessKey ID

LTAI5t9yLGp9bdSG17s7pgA9

AccessKey Secret

S4FxfwoZ1pFCpphpWFjR1C23ynJw8d
```

- 完成填写![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921223959053.png)


- 检查连接是否正常
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224002309.png)



- 开启三项基本配置![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224005009.png)


##### 1.3.2.2 测试

- 点击右边按钮开始同步![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224008452.png)


- 检测Bucket是否同步完成![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224011318.png)



### 1.4 移动端设置

- 下载Obsdian
- 创建一个名字相同的仓库![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224021743.png)


- 安装Remotely Save插件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224025154.png)



- 电脑端打开插件的QR码，手机开启扫描![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224035974.png)


- 手机扫描成功后自行导入配置，并完成同步![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224045250.png)



## 2 Git实现仓库备份

- `这里的远程仓库我们使用Gitee` [工作台 - Gitee.com](https://gitee.com/)

### 2.1 安装Git

- 自行搜索教程即可

### 2.2 Git初始化

- 加入仓库所在的文件夹，点击鼠标右键，选择GitBash![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224054617.png)


- 按照命令依次输入
	- git init  （初始化本地Git仓库）
	- git config --global user.name dakkk   (用户注册)
	- git config --global user.email mikeylay@126.com    (用户注册)
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224058440.png)



- 在当前文件夹，创建 .gitignore文件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224101033.png)


### 2.3 注册Gitee账户，创建仓库

- 注册Gitee账户，自行注册即可

- 创建Gitee仓库![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224103244.png)


- 复制仓库的Http链接![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224105708.png)


- 开始git同步，按照下述命令执行
	- git remote add origin https://gitee.com/Dakkk_mike/skill-tip.git
	- git add --all
	- git commit -m "commit all"
	- git push -u origin master

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224112935.png)



- 查看远程仓库，发现本地仓库内容已完成上传![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224115740.png)
- 解除一个项目的git配置
	- rm -rf .git
	- 删除远程仓库


### 2.4 使用Obsidian Git插件实现自动同步

- 下载Obsidian Git插件![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224120068.png)


- 配置本地Git的路径（没有配置系统的环境变量）![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224123701.png)


- 设置![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/image-20230921224130756.png)



