
当前文章步骤在Ubuntu 24.04.01 LTS中进行了实测。它应该适用于所有适用的Ubuntu版本
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9ef3a3d03a575808cfce512af6c5ac14.png)

**步骤一：查看Snap软件包列表**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5eb74b40d52c34a47e73f29b3b8b5568.png)

**步骤二：按顺序删除snap包**
```
snap remove --purge xxx
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e4a731ba5f70a2d898982a85445c07a6.png)


软件移除顺序一般为：一般应用>美化应用>底层应用>核心应用>本应用，于是先移除桌面和主题应用，然后移除bare和core，最后移除snapd。由于系统原来Ubuntu24默认就装的firefox和thunderbird已经删除，它们应该第一批可以删除的应用。

**步骤三：停用snap守护进程snapd**
```shell
apt remove --autoremove snapd
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c8d2bd6a8c44351bc2509afd662dbb7f.png)

**步骤四：防止snap恢复**
- 由于一些软件依赖snapd包，就会导致在更新软件时snapd会被重新安装，例如：ubuntu-desktop就会依赖snapd
```shell
# 删除snapd残留目录
rm -rf ~/snap /snap /var/snap /var/lib/snapd

# 阻止snapd被重新安装
apt-mark manual snapd

# 移除元包依赖的升级或者更新，亦或者删除元包
apt remove ubuntu-desktop

# 配置禁止APT对snapd自动安装
echo "Package: snapd Pin: release a=* Pin-Priority: -10" | tee /etc/apt/preferences.d/nosnap.pref apt update

# 使用dpkg绕过依赖（谨慎操作）
dpkg --remove --force-depends snapd
```

- 上面办法不行的话，使用如下办法
```shell
# 创建并打开配置文件
sudo vi /etc/apt/preferences.d/nosnap.pref

# 粘贴以下几行以告知拒绝`snapd`任何存储库：
# 为防止存储库包触发 snap 的安装，  
# 此文件禁止 APT 安装 snapd。

Package: snapd  
Pin: release a=*  
Pin-Priority: -10
```
- 保存文件后，通过命令刷新包缓存：
```shell
sudo apt update
```

**步骤五：恢复snap**
```shell
# 解除对守护进程的阻止
sudo rm /etc/apt/preferences.d/nosnap.pref

# 安装Ubuntu软件
sudo snap install snap-store

# 如果需要，可以通过运行命令将 Firefox 安装为 snap
sudo apt install Firefox
```