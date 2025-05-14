
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
