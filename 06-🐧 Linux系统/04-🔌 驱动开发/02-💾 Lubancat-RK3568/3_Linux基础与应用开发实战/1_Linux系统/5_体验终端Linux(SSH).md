---
文章标题: "[[5_体验终端Linux(SSH)]]" 
文章作者: Dakkk
文章概要: |
  介绍LubanCat板卡通过SSH终端连接Linux系统的完整操作流程，包括硬件准备、网络配置、基础命令使用和软件包管理等内容。
tags:
- "Linux"
- "SSH"
- "终端"
- "嵌入式"
- "RNDIS"
- "MobaXterm"
- "apt包管理"
- "命令行"
相关文章:
- "[[7_Linux命令行]]"
- "[[3_CentOS 安装 Docker]]"
- "[[4_Ubuntu 安装 Docker]]"
- "[[1_Mobaxterm软件的使用]]"
- "[[2_scp命令，拷贝文件]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/3_Linux基础与应用开发实战/1_Linux系统/5_体验终端Linux(SSH).md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-05-06 17:04:28
修改时间: 2025-05-28 00:19:51
---


**❗️注意：** 本章内容仅支持局域网内只有一个LubanCat板卡的情况，因为是根据主机名连接板卡的，如果超过会可能导致连接到同一板卡上，不适用本方法，如果是多台设备可以使用ip进行连接。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/92f370ce212edb5a2a5856b95c2bd843.png)

**❗️注意：** 如果接上电脑USB电源后，过一会儿出现绿灯(系统心跳灯)一直在闪烁，证明供电足够，如果电脑的供电能力不足，可能会导致板卡不断重启(绿色的灯很暗或者两秒内没有连续跳动)，此时需要使用额外电源供电，或者找电脑USB3.0接口，供电能力更强。

## 1 前言

如果您想体验本章节的内容，你需要具备以下条件
1. 镜像，带emmc的板卡在出厂前就会烧录好镜像，如果使用的sd卡或者没带桌面镜像需要烧录镜像，可以前往 [2 系统镜像烧录](../../1_快速使用手册/1_快速开始.md#2%20系统镜像烧录) 烧录镜像
2. 通信软件软件,我们推荐使用 `MobaXterm`
3. 电脑需要支持RNDIS功能（win10默认支持，win11需要更换微软自带驱动，其他系统可能需要安装RNDIS驱动）
4. 一根带通信功能的type-c线。
5. 如果使用电脑usb单独供电，电脑的供电接口的供电能力大于5V@1A，建议直连电脑主板的USB接口，拓展坞的接口可能会供电不足。
## 2 开机前准备

### 2.1 软件准备

MobaXterm安装及使用
- 下载地址： [https://mobaxterm.mobatek.net/download.html](https://mobaxterm.mobatek.net/download.html)

`MobaXterm` 终端软件的详细使用可以参考文档：
- [1_Mobaxterm软件的使用](../4_补充部分/1_Mobaxterm软件的使用.md)
- [3.4 串口终端登录](../../1_快速使用手册/1_快速开始.md#3.4%20串口终端登录)
### 2.2 上电

使用Type-C线连接板卡上写着 `pwr` 或者 `OTG` 的Type-C接口， 如果是使用LubanCat-2或者LubanCat-2N，也可以使用DC接口给他们供电使用。
## 3 开机

- 板卡烧录后的第一次启动需要等待大概一分钟再观察板卡状态灯的情况
- 如果不是初次启动只需等待20秒即可

1. 绿灯像心跳一样跳动 —–> 正常开机
2. 绿灯很暗或者在两秒内没能连续跳动 —-> 可能在重启(您的电脑的usb接口可能不适合大电流供电，可以减少外设再试)

### 3.1 USB共享方式，配置网络（用不了）

**❗️注意：** 电脑需要支持RNDIS功能（win10默认支持，其他系统可能需要安装RNDIS驱动）

- 板卡烧录后的第一次启动需要等待大概一分钟完成初始化操作
- 搜索打开网络设置，查看当前网络状态
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/75ad844ea657d616c7450a90deb37159.png)
- 查看当前所有的网络连接（标注着“Remote NDIS based Internet”就是我们的板卡）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5792f145563693cf1f63003d7813db9f.png)
- 配置网络共享
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f6d0f452b38f60efe7b0411fa8dfcb18.png)
- 完成后就可以连接我们的板卡了，打开MobaXterm软件，点击“Session”创建连接
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1249a5cb872f61d5659fd1e06fa3bd4c.png)
- 然后按照以下配置输入
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f38e40603b923839cc723cfb8c1edce8.png)
## 4 网络连接

此时板卡上已经有网络了，可以不用连接网口和wifi，如果有需要的话可以前往[4 网络连接及静态配置](../../1_快速使用手册/1_快速开始.md#4%20网络连接及静态配置)配置您的网络
## 5 介绍功能

### 简单命令使用

```shell
#列出当前目录的文件
ls

#列出其他文件的目录
ls + 其他目录

#列出当前目录的位置
pwd

#切换目录
cd + 目录位置

#切换上次切换的目录
cd -

#切换家目录
cd
cd ~

#切换上一层目录
cd ..
```

更多命令，可以查看[7_Linux命令行](7_Linux命令行.md)
## 6 软件更新（apt）

Debian和Ubuntu的软件大部分都是通过网络获取的，而且这些软件会自动更新， 我们的镜像可能使用的是旧的软件，这些旧的软件可能会存在漏洞或者bug， 影响我们的正常使用。所以我们可以在适当时间更新自家的软件。

**常见apt指令汇总**

|功能|指令|说明|
|---|---|---|
|🔄 更新软件包列表|`sudo apt update`|从软件源获取最新的可用包信息，但不安装|
|⬇️ 安装软件包|`sudo apt install <包名>`|安装一个或多个包，例如：`sudo apt install curl`|
|♻️ 升级所有已安装包|`sudo apt upgrade`|将所有已安装的软件包升级到最新版|
|🚀 系统全面升级（含核心包变更）|`sudo apt full-upgrade`|类似于 `dist-upgrade`，自动处理依赖关系和删除旧包|
|🔍 搜索包|`apt search <关键词>`|查找可安装的包，例如：`apt search nginx`|
|🧾 查看包信息|`apt show <包名>`|查看软件包的详细信息（版本、依赖等）|
|❌ 卸载包（保留配置）|`sudo apt remove <包名>`|卸载程序但保留配置文件|
|🧹 卸载包（连配置一起）|`sudo apt purge <包名>`|完全卸载，包括配置文件|
|🔥 自动清理不再需要的依赖|`sudo apt autoremove`|删除安装后不再使用的依赖包|
|🧼 清理已下载的缓存|`sudo apt clean`|清除所有已下载的软件包文件|
|📂 清理部分缓存|`sudo apt autoclean`|清除旧版本但保留当前可用的包文件|

**❗️注意：** 烧录镜像后的第一次启动后，需要更新自己的软件源，才可以从网络上获取软件

```shell
#更新软件
sudo apt update

#安装更新的软件
sudo apt upgrade

#软件安装
sudo apt install xxx

#软件升级
sudo apt upgrade xxx

#软件卸载
sudo apt remove xxx

#查看软件清单
sudo apt list
```

这里以tree软件为例,tree是以树状结构查看文件夹
```shell
#安装软件
sudo apt install tree

#使用tree查看文件夹
tree
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/608eeb029e718222809d20c5da444d56.png)
