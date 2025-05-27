---
文章标题: "[[4_apt命令及问题]]" 
文章作者: Dakkk
文章概要: |
  介绍Linux系统中apt包管理命令的使用方法，包括软件源配置、常用指令操作以及依赖缺失等问题的解决方案
tags:
- "apt"
- "Linux"
- "包管理"
- "软件源"
- "Ubuntu"
- "依赖关系"
- "系统管理"
- "命令行"
相关文章:
- "[[4_Ubuntu 安装 Docker]]"
- "[[3_CentOS 安装 Docker]]"
- "[[5_Linux相关命令]]"
- "[[1_开发环境搭建]]"
- "[[2_Linux入门基础]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/00-❇️实用技巧/4_apt命令及问题.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-13 19:51:27
修改时间: 2025-05-28 00:19:51
---

当我们烧录了新的镜像或者遇到部分软件不能安装，我们建议使用apt进行更新和升级

**apt命令使用的时候需要连接网络**
**如果在升级的时候遇到update报错，我们重新执行命令**

```shell
#更新软件包数据库
sudo apt update

#升级已安装的软件包
sudo apt upgrade
```

## 1 修改apt软件源

在改写软件源前，可以备份一下软件源，防止设置错误的软件源
```shell
#备份软件源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup

#编辑你要使用的软件源
sudo vi /etc/apt/sources.list

#更新软件源
sudo apt update

#升级
sudo apt upgrade
```

修改软件源回中科大的软件源
```shell
#修改软件源为中科大软件源
sudo cp /etc/apt/sources.list.backup /etc/apt/sources.list

#更新软件源
sudo apt update

#升级
sudo apt upgrade
```

## 2 apt常用指令

```shell
# 更新apt数据库
sudo apt update
# 升级所有有可用更新的软件包
sudo apt upgrade
# 安装软件包
sudo apt install package_name
# 安装软件包（只安装不升级）
sudo apt install <package_name> --no-upgrade
# 安装软件包（只升级不安装）
sudo apt install <package_name> --only-upgrade
# 安装特点版本的软件包
sudo apt install <package_name>=<version_number>

# 移出已安装的软件包
sudo apt remove package_name
sudo apt remove package1 package2
# 删除包含所有配置文件的软件包
sudo apt purge
# 删除未使用的软件包
sudo apt autoremove

# 生成软件包列表
sudo apt list
sudo apt list | grep package_name
sudo apt list --installed
sudo apt list --upgradeable

# 搜索软件包
sudo apt search package_name
# 显示软件包信息
sudo apt show package_name
# 清理下载文件的存档
sudo apt-get clean
# 下载软件源代码
sudo apt-get source <packages>
# 了解软件依赖关系
sudo apt-cache depends <packages>
# 检查软件依赖关系
sudo apt-get check
```

### 2.1 常见问题

#### 2.1.1 依赖缺失

问题现象：
```shell
下列软件包有未满足的依赖关系：
libc6-dev : 依赖: libc6 (= 2.27-3ubuntu1.6) 但是 2.35-0ubuntu3 正要被安装
依赖: libc-dev-bin (= 2.27-3ubuntu1.6)
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

一般都是镜像源和发现版本不匹配导致的

每个发行版本都有自己的代号，可以查看
```shell
lsb_release -a
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9366960ec2c1d84345869fe962df2520.png)

那么镜像源也要对应修改
```shell
vi /etc/apt/sources.list
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1fdda9da3164470d320308b16a1bfcf2.png)
