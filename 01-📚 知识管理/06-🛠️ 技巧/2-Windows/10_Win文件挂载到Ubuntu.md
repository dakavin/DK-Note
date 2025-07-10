---
创建时间: 2025-06-22 01:09:58
修改时间: 2025-06-22 02:55:05
---
## 1 什么是 Syncthing？

Syncthing 是一个**连续文件同步程序**，可以在多个设备之间安全、私密地同步文件夹，无需云服务器。

**优势：**
- ✅ 真正的双向实时同步
- ✅ 无需中央服务器，点对点同步
- ✅ 加密传输，隐私安全
- ✅ 跨平台支持（Windows、Linux、macOS、Android）
- ✅ 版本控制和冲突处理
- ✅ Web界面管理

## 2 Ubuntu 端安装

### 2.1 安装 Syncthing

```bash
# 方法1：通过官方仓库安装（推荐）
curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
sudo apt update
sudo apt install syncthing

# 方法2：通过Ubuntu仓库安装（版本可能较旧）
sudo apt install syncthing
```

### 2.2 启动 Syncthing 服务

```bash
# 启用用户服务（开机自启）
systemctl --user enable syncthing
systemctl --user start syncthing

# 检查服务状态
systemctl --user status syncthing
```

### 2.3 访问 Web 界面

```bash
# Syncthing 默认在 8384 端口运行
echo "在浏览器中打开：http://127.0.0.1:8384"

# 如果需要外部访问，修改配置
nano ~/.config/syncthing/config.xml
# 找到 <address>127.0.0.1:8384</address>
# 改为 <address>0.0.0.0:8384</address>
# 然后重启服务
systemctl --user restart syncthing
```

## 3 Windows 端安装


1. 访问 https://syncthing.net/downloads/
2. 下载 Windows 版本
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/4fd540599a7eccdc888e90f0bf4a125e.png)
```cmd
# 在解压目录中双击 syncthing.exe
# 或在命令提示符中运行
cd C:\Syncthing
syncthing.exe
```

启动后，浏览器会自动打开：`http://127.0.0.1:8384`


## 4 高级配置

### 4.1 文件夹类型设置

**Folder Type 选项：**
- **Send & Receive**：双向同步（推荐）
- **Send Only**：只发送，不接收
- **Receive Only**：只接收，不发送

### 4.2 版本控制设置

在文件夹设置的 "File Versioning" 中：
- **Simple File Versioning**：保留指定数量的旧版本
- **Staggered File Versioning**：分层保留（1小时、1天、1周、1月）
- **External File Versioning**：使用外部脚本处理版本

**推荐设置：**

```txt
Versioning: Staggered File Versioning
Maximum Age: 365 days
Versions Path: .stversions
```

### 4.3 忽略文件设置

在文件夹设置的 "Ignore Patterns" 中添加：

```txt
# Windows 系统文件
Thumbs.db
.DS_Store
desktop.ini

# 临时文件
*.tmp
*.temp
~*

# Office 临时文件
~$*
*.lnk
```

### 4.4 同步频率优化

在 "Advanced" 设置中：
- **Rescan Interval**: 设为 `60s`（默认）
- **Full Rescan Interval**: 设为 `3600s`（1小时）

---

## 5 使用技巧

### 5.1 查看同步状态

```bash
# Ubuntu 命令行查看状态
syncthing cli show system
syncthing cli show completion
```

### 5.2 手动触发同步

在 Web 界面中点击文件夹旁的 "Rescan" 按钮。

### 5.3 解决同步冲突

当同一文件在两端同时修改时，Syncthing 会创建冲突文件：

```
原文件：document.txt
冲突文件：document.sync-conflict-20240622-142030-DEVICE.txt
```

手动选择要保留的版本，删除冲突文件。

### 5.4 监控同步日志

**Ubuntu 端：**

```bash
# 查看日志
journalctl --user -u syncthing -f

# 或查看 Web 界面的 Logs 部分
```

---

## 6 性能优化

### 6.1 局域网优化

在设备设置中启用：
- **Enable NAT traversal**
- **Local Discovery**
- **Global Discovery**

### 6.2 大文件优化

对于大文件同步，在高级设置中：
- 增加 **Temporary Index File Threshold**
- 启用 **Large Block Sizes**

### 6.3 网络带宽限制

在设备设置的 "Advanced" 中设置：
- **Incoming Rate Limit**：接收速度限制
- **Outgoing Rate Limit**：发送速度限制
