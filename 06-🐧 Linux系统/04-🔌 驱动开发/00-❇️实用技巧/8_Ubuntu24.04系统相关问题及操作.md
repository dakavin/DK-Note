## 1 没有python2版本

[安装Ubuntu](https://so.csdn.net/so/search?q=%E5%AE%89%E8%A3%85Ubuntu&spm=1001.2101.3001.7020)24.04之后，安装python2会提示：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/fc21ae0fe34795d012f27b5f242236eb.png)

**解决方式：**
```shell
wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz
sudo tar xzf Python-2.7.18.tgz
cd Python-2.7.18
sudo ./configure --enable-optimizations
sudo make altinstall
```

之后
```shell
python2.7 -V
~ Python 2.7.18
sudo ln -sfn '/usr/local/bin/python2.7' '/usr/bin/python2'
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
```

**可以设置python的版本**
```shell
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
sudo update-alternatives --config python  （选择切换Python版本）
python --version （查看Python版本）
```

**卸载**
```shell
# 删除软链接
sudo rm /usr/bin/python2

# 删除目录
sudo rm -rf Python-2.7.18

# 删除安装文件
sudo rm /usr/local/bin/python2.7
sudo rm /usr/local/bin/python2
sudo rm -rf /usr/local/lib/python2.7
sudo rm -rf /usr/local/include/python2.7

# 查看之前python的方案
sudo update-alternatives --list python
#删除py2的方案
sudo update-alternatives --remove python /usr/bin/python2
#重新配置为python3的方案
sudo update-alternatives --config python
```

## 2 ubuntu系统下deb包安装流程

```shell
# 安装
sudo apt install ./package.deb
# 修复缺失的依赖
sudo apt install -f
# 检查是否安装成功
apt list --installed | grep package-name
# 卸载
sudo apt remove package-name
# 完全卸载
sudo apt purge package-name

dpkg -I package.deb  # 查看包信息
dpkg -c package.deb  # 查看包内文件列表
```
- **推荐方式**，`apt` 会自动处理依赖问题。
- **注意**：路径要写完整（如 `./package.deb` 或 `/path/to/package.deb`）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b761f29c29be9db40a8fa9ca6e849e32.png)

### 2.1 配置snipaste

参考文档：https://blog.csdn.net/qq_44684757/article/details/136062578

Snipaste 官网下载链接: https://www.snipaste.com/download.html

```shell
# 1、进入到 Snipaste-2.8.9-Beta-x86_64.AppImage 所在目录（根据自己的下载目录而定）
cd /home/jack/Downloads/softwares/AppImage

# 2、给 Snipaste-2.8.9-Beta-x86_64.AppImage 添加可执行权限
sudo chmod +x Snipaste-2.8.9-Beta-x86_64.AppImage

# 3、执行即可启动
./Snipaste-2.8.9-Beta-x86_64.AppImage
```

**出现如下报错，需要安装fuse即可**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3a2b63176cf10f434c7e19ae7ed32265.png)
```shell
sudo apt update
sudo apt install fuse libfuse2
```

**配置开机自启动**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/90dcbdb2c0eff79c96f2f9b47b6da4e6.png)

## 3 交换内存没有启用

- **检查当前激活的 SWAP 空间**：
  ```bash
  sudo swapon --show
  free -h
  ```
  如果输出中未显示你的 32GB SWAP 分区，说明它未被启用。

- **手动启用 SWAP**：
  ```bash
  sudo swapon /dev/[swap分区设备名]  # 例如 /dev/nvme0n1p3
  ```
  再运行 `free -h` 查看是否生效。

## 4 使用Win11远程唤醒Ubuntu

**1、检查网卡是否支持WOL**
```shell
sudo ethtool <网卡名>  # 如eth0或enp3s0
sudo ethtool enp4s0
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/345b5d2fba8c88535cd62ba795e52389.png)

**2、开启WOL功能**
```shell
# 启用魔术包唤醒（g表示启用）
sudo ethtool -s enp4s0 wol g

# 验证是否生效（检查Wake-on值是否变为g）
sudo ethtool enp4s0 | grep "Wake-on"
```
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/15430325e235284d7283fa2dcd1d3146.png)

**3、持久化WOL配置**
```shell
# 创建systemd服务确保每次开机自动启用WOL
sudo vi /etc/systemd/system/wol.service
```
- 添加以下内容（替换`enp4s0`为实际网卡名）：
```ini
[Unit]
Description=Enable Wake-on-LAN for enp4s0

[Service]
Type=oneshot
ExecStart=/sbin/ethtool -s enp4s0 wol g

[Install]
WantedBy=multi-user.target
```
- 保存后启用服务：
```shell
sudo systemctl enable wol.service
sudo systemctl start wol.service
```

**4、检查BIOS/UEFI设置
- 进入主板BIOS，确保以下选项已启用：
	- **Wake-on-LAN** 或 **PCI-E Power On**
	- **ErP Ready** 或 **EuP 2013**（需禁用，否则可能限制WOL功能）

**5、验证网络环境**
- **路由器/交换机**：确保局域网内允许广播包（UDP端口9）传输，部分路由器需关闭“节能模式”或“ARP绑定限制”
- **防火墙**：Ubuntu端开放UDP端口9
```shell
sudo ufw allow 9/udp
```

**6、从Win11发包**
- Ubuntu设备mac地址：10:ff:e0:0c:76:c5
- ip：192.168.31.147
- 编写Win11的Powershell脚本用于测试，命名`wake-on-lan.ps1`
```shell
# 配置参数
$macAddress = "10:ff:e0:0c:76:c5"  # 替换为Ubuntu的MAC地址
$port = 9                           # WOL默认端口
$broadcastIP = "255.255.255.255"    # 广播地址

# 构建魔术包（6字节0xFF + 16次重复MAC地址）
$magicPacket = [byte[]](,0xFF * 6) + ($macAddress -replace '[:-]','' -split '(..)' | Where-Object { $_ } | ForEach-Object { [byte]('0x' + $_) }) * 16

# 发送UDP广播包
$udpClient = New-Object System.Net.Sockets.UdpClient
$udpClient.Connect([System.Net.IPAddress]::Parse($broadcastIP), $port)
$bytesSent = $udpClient.Send($magicPacket, $magicPacket.Length)
$udpClient.Close()

# 输出结果
Write-Host "魔术包已发送（${bytesSent}字节）！"
```
- 或者编写bat脚本，命名为 `wake-on-lan.bat`，粘贴以下内容：
```bat
@echo off
chcp 65001 >nul

:: 设置 MAC 地址、端口、广播地址
set "MAC=10:ff:e0:0c:76:c5"
set "PORT=9"
set "BROADCAST=255.255.255.255"

:: 调用 PowerShell 执行唤醒功能
powershell -NoProfile -Command ^
    "$mac='%MAC%'; $port=%PORT%; $bip='%BROADCAST%';" ^
    "$bytes = ($mac -replace '[:-]','' -split '(..)' | Where-Object { $_ } | ForEach-Object { [byte]('0x' + $_) });" ^
    "$packet = [byte[]](,0xFF * 6) + ($bytes * 16);" ^
    "$client = New-Object Net.Sockets.UdpClient;" ^
    "$client.Connect([Net.IPAddress]::Parse($bip), $port);" ^
    "$sent = $client.Send($packet, $packet.Length);" ^
    "$client.Close();" ^
    "Write-Host \"魔术包已发送（${sent}字节）！\""

:: 保持窗口
pause
```

## 5 开机界面黑屏，无bios界面

显卡为GTX1060 6GB，查询资料发现是N卡的DP有bug，更换HDMI就好了，不过188HZ的显示器只能用144Hz了，不甘

查询后可以发现如下解决方式
- 启用 CSM（兼容性支持模块）：
	- 进入 BIOS 设置（开机时按 Del/F2），找到 启动模式 或 CSM 选项，设置为 Enabled。
	- CSM 允许 UEFI 主板兼容旧硬件，部分显卡的 DP 接口在纯 UEFI 模式下无法初始化，需开启 CSM 以显示 BIOS 界面。
- 调整显卡初始化顺序：
	- 在 BIOS 的 显示设置 中，将 首选显卡 设为 独立显卡（如 PCI-E） 而非集成显卡。
- 更新主板 BIOS：
	- 访问主板厂商官网，下载最新 BIOS 固件并升级，可能修复 DP 接口的兼容性问题

## 6 系统无法进入，出现`[ok]`

报错现象如下：

原因：gdm3图形显示界面和nvidia显卡驱动冲突

参考链接：[【问题解决】Ubuntu无法进入图形页面，全屏出现OK，而且屏幕不停闪烁_ubuntu打开后一堆ok-CSDN博客](https://blog.csdn.net/qyuqing/article/details/124193139)

解决办法：
- **按 ctrl + alt +F2 进入命令行,输入用户名和密码。（可能各种电脑换版本不同，大家可以从F1-F7，全都是试一个遍，总有一个适合**
- **输入以下命令：**
```shell
sudo apt-get remove --p
 
sudo apt purge gdm3
 
sudo reboot
 
sudo apt install gdm3
 
sudo service gdm start
```

## 7 nvidia显卡驱动适配问题汇总

参考链接：[记ubuntu20.04显卡驱动安装_sudo service gdm3 stop-CSDN博客](https://blog.csdn.net/baidu_41816106/article/details/121184844)
