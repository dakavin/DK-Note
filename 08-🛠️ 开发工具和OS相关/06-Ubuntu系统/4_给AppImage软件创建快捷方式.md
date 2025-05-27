---
文章标题: "[[4_给AppImage软件创建快捷方式]]" 
文章作者: Dakkk
文章概要: |
  本文详细介绍了在Linux系统上为AppImage应用创建快捷方式的两种方法：一是手动创建`.desktop`文件，包括权限设置、文件内容编辑和桌面集成；二是利用AppImageLauncher工具实现自动化集成。文章还包括了AppImage文件的初步准备工作。
tags:
- "AppImage"
- "Linux"
- "快捷方式"
- ".desktop文件"
- "AppImageLauncher"
- "桌面集成"
- "应用程序管理"
相关文章:
- "[[0_介绍]]"
- "[[0_课程介绍]]"
- "[[07_Linux的介绍]]"
- "[[1_本章代码及文章获取]]"
- "[[1_常用文件夹]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/06-Ubuntu系统/4_给AppImage软件创建快捷方式.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-27 06:58:11
修改时间: 2025-05-28 01:03:09
---

## 1 准备工作

**第一步：赋予执行权限**
- 可以看到是没有权限的
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1f608310bbc911f17587316de442d3e0.png)
- 使用`chmod +x 对应的文件`即可赋予执行权限了

**第二步：移动到合适的位置**
- 一般是home目录下的 app目录，这里已经移动了
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8e9e2c358c4947c369f794fa5bc8efd6.png)
- 命令为 `mv AppImage所在路径 移动到的路径`

## 方式一：手动创建 .desktop文件

**第一步：创建.desktop文件**
- 桌面条目文件通常存放在
	- `~/.local/share/applications/` (仅对当前用户可见) 
	- `/usr/share/applications/` (对所有用户可见，需要sudo权限)
- 这里创建一个文本文件名为 `obsidian.desktop`，并保存在当前用户目录下
```shell
touch ~/.local/share/applications/obsidian.desktop
```

**第二步：编辑文件内容**
- 我这里使用的是vim
```shell
vi touch ~/.local/share/applications/obsidian.desktop
```
- 然后填入一下内容，根据实际情况进行修改
```txt
[Desktop Entry]
Version=1.0
Type=Application
Name=My Awesome App                  # 应用程序的显示名称
Comment=A brief description of my app # 应用程序的描述 (可选)
Exec=/home/yourusername/Applications/YourApp.AppImage %U  # 替换为AppImage的完整路径
Icon=/path/to/your/icon.png        # 替换为图标的完整路径 (可选)
Terminal=false                       # 如果是GUI应用，设为false；如果是命令行应用，设为true
Categories=Utility;Development;    # 应用程序分类 (可选, 多个用分号隔开)
```

> [!warning]+ 注意：
> 如果打开不了文件，出现sandbox的错误，就需要在Exec后面加上 --no-sandbox %U
> 
> 也就是Exec=/home/dakkk/app/obsidian/Obsidian-1.8.10.AppImage --no-sandbox %U

**第三步：保存文件并使其可以被系统识别**
- 保存文件后，通常桌面环境会自动检测到新的 `.desktop` 文件
- 如果快捷方式没有立即出现在应用程序菜单中，你可能需要：
	- 注销再登录
	- 运行命令`update-desktop-database ~/.local/share/applications`更新桌面数据库

**第四步：创建桌面图标**
如果你希望在桌面上也看到这个快捷方式，可以在文件管理器中导航到 ~/.local/share/applications/，找到你创建的 YourApp.desktop 文件，然后：
- 右键点击它，选择 "Allow Launching" (允许启动) 或类似选项（具体取决于你的桌面环境和文件管理器）。
- 然后，你可以复制这个 .desktop 文件到你的桌面 (~/Desktop/)

## 2 方式二：使用AppImageLauncher 

AppImageLauncher 是一个辅助工具，它可以将AppImage文件集成到你的系统中，自动创建菜单项，并在你第一次运行AppImage时提示你是否要集成。

1. **安装 AppImageLauncher：**
    - 访问 AppImageLauncher 的 GitHub Releases 页面：[https://github.com/TheAssassin/AppImageLauncher/releases](https://www.google.com/url?sa=E&q=https%3A%2F%2Fgithub.com%2FTheAssassin%2FAppImageLauncher%2Freleases)
    - 下载适合你系统架构（通常是 amd64）的 .deb 安装包 (例如 `appimagelauncher_*.x86_64.deb`)。
    - 安装它：
        ```
        sudo apt update
        sudo apt install ./appimagelauncher_*.x86_64.deb
        ```
	- (将 appimagelauncher_*.x86_64.deb 替换为你下载的实际文件名)
        
2. **使用：**
    - 安装完成后，当你第一次双击运行一个AppImage文件时，AppImageLauncher会弹出一个对话框，询问你是否要 "Integrate and run" (集成并运行) 或 "Run once" (运行一次)。
    - 选择 "Integrate and run"。它会自动将AppImage文件移动到 ~/Applications 目录（如果还不在那里），并为你创建一个相应的桌面条目（快捷方式）。
    - 之后，你就可以在应用程序菜单中找到并启动它了。
    - 如果你已经将AppImage文件放在了 ~/Applications，AppImageLauncher 也会检测到并提供集成选项


## 1 准备工作

**第一步：赋予执行权限**
- 可以看到是没有权限的
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1f608310bbc911f17587316de442d3e0.png)
- 使用`chmod +x 对应的文件`即可赋予执行权限了

**第二步：移动到合适的位置**
- 一般是home目录下的 app目录，这里已经移动了
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8e9e2c358c4947c369f794fa5bc8efd6.png)
- 命令为 `mv AppImage所在路径 移动到的路径`

## 方式一：手动创建 .desktop文件

**第一步：创建.desktop文件**
- 桌面条目文件通常存放在
	- `~/.local/share/applications/` (仅对当前用户可见) 
	- `/usr/share/applications/` (对所有用户可见，需要sudo权限)
- 这里创建一个文本文件名为 `obsidian.desktop`，并保存在当前用户目录下
```shell
touch ~/.local/share/applications/obsidian.desktop
```

**第二步：编辑文件内容**
- 我这里使用的是vim
```shell
vi touch ~/.local/share/applications/obsidian.desktop
```
- 然后填入一下内容，根据实际情况进行修改
```txt
[Desktop Entry]
Version=1.0
Type=Application
Name=My Awesome App                  # 应用程序的显示名称
Comment=A brief description of my app # 应用程序的描述 (可选)
Exec=/home/yourusername/Applications/YourApp.AppImage %U  # 替换为AppImage的完整路径
Icon=/path/to/your/icon.png        # 替换为图标的完整路径 (可选)
Terminal=false                       # 如果是GUI应用，设为false；如果是命令行应用，设为true
Categories=Utility;Development;    # 应用程序分类 (可选, 多个用分号隔开)
```

> [!warning]+ 注意：
> 如果打开不了文件，出现sandbox的错误，就需要在Exec后面加上 --no-sandbox %U
> 
> 也就是Exec=/home/dakkk/app/obsidian/Obsidian-1.8.10.AppImage --no-sandbox %U

**第三步：保存文件并使其可以被系统识别**
- 保存文件后，通常桌面环境会自动检测到新的 `.desktop` 文件
- 如果快捷方式没有立即出现在应用程序菜单中，你可能需要：
	- 注销再登录
	- 运行命令`update-desktop-database ~/.local/share/applications`更新桌面数据库

**第四步：创建桌面图标**
如果你希望在桌面上也看到这个快捷方式，可以在文件管理器中导航到 ~/.local/share/applications/，找到你创建的 YourApp.desktop 文件，然后：
- 右键点击它，选择 "Allow Launching" (允许启动) 或类似选项（具体取决于你的桌面环境和文件管理器）。
- 然后，你可以复制这个 .desktop 文件到你的桌面 (~/Desktop/)

## 2 方式二：使用AppImageLauncher 

AppImageLauncher 是一个辅助工具，它可以将AppImage文件集成到你的系统中，自动创建菜单项，并在你第一次运行AppImage时提示你是否要集成。

1. **安装 AppImageLauncher：**
    - 访问 AppImageLauncher 的 GitHub Releases 页面：[https://github.com/TheAssassin/AppImageLauncher/releases](https://www.google.com/url?sa=E&q=https%3A%2F%2Fgithub.com%2FTheAssassin%2FAppImageLauncher%2Freleases)
    - 下载适合你系统架构（通常是 amd64）的 .deb 安装包 (例如 `appimagelauncher_*.x86_64.deb`)。
    - 安装它：
        ```
        sudo apt update
        sudo apt install ./appimagelauncher_*.x86_64.deb
        ```
	- (将 appimagelauncher_*.x86_64.deb 替换为你下载的实际文件名)
        
2. **使用：**
    - 安装完成后，当你第一次双击运行一个AppImage文件时，AppImageLauncher会弹出一个对话框，询问你是否要 "Integrate and run" (集成并运行) 或 "Run once" (运行一次)。
    - 选择 "Integrate and run"。它会自动将AppImage文件移动到 ~/Applications 目录（如果还不在那里），并为你创建一个相应的桌面条目（快捷方式）。
    - 之后，你就可以在应用程序菜单中找到并启动它了。
    - 如果你已经将AppImage文件放在了 ~/Applications，AppImageLauncher 也会检测到并提供集成选项

