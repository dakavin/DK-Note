
参考文章：https://blog.csdn.net/m0_63558278/article/details/136440100?spm=1001.2101.3001.10796

## 1 完全卸载fcitx4和搜狗输入法

```bash
# 停止fcitx进程
killall fcitx

# 卸载fcitx4相关包
sudo apt remove --purge fcitx fcitx-* libfcitx-* sogoupinyin

# 清理用户配置
rm -rf ~/.config/fcitx
rm -rf ~/.config/sogoupinyin
rm -rf ~/.local/share/fcitx
```

## 2 安装fcitx5完整套件

```bash
# 安装fcitx5核心组件
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5

# 安装中文输入法
sudo apt install fcitx5-pinyin

# 安装配置工具
sudo apt install fcitx5-config-qt

# 可选：安装其他输入法
sudo apt install fcitx5-rime fcitx5-table-wubi
```

## 3 设置环境变量

```bash
# 清理旧的环境变量，重新设置
vi ~/.profile
```

确保内容为：
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

## 4 设置fcitx5开机自启动

```bash
# 创建自启动配置
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/fcitx5.desktop << EOF
[Desktop Entry]
Name=Fcitx 5
GenericName=Input Method
Comment=Start Input Method
Exec=fcitx5
Icon=fcitx
Terminal=false
Type=Application
Categories=System;Utility;
StartupNotify=false
NoDisplay=true
EOF
```

## 5 重启系统
```bash
sudo reboot
```

## 6 重启后配置fcitx5
```bash
# 启动配置工具
fcitx5-configtool
```

在配置界面中添加中文输入法：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d3c12a39be1b53fccbc64fff1e5dafa4.png)

修改切换输入法
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1410b6569b8dee99920e132e73caaaa2.png)

## 7 添加词库

在 GitHub 打开[维基百科中文拼音词库](https://link.zhihu.com/?target=https%3A//github.com/felixonmars/fcitx5-pinyin-zhwiki "维基百科中文拼音词库")的 [Releases](https://link.zhihu.com/?target=https%3A//github.com/felixonmars/fcitx5-pinyin-zhwiki/releases "Releases") 界面，下载最新版的 `.dict` 文件。按照 README 的指导，将其复制到 `~/.local/share/fcitx5/pinyin/dictionaries/` 文件夹下即可。

```c
# 下载词库文件
wget https://github.com/felixonmars/fcitx5-pinyin-zhwiki/releases/download/0.2.5/zhwiki-20240509.dict

# 创建存储目录
mkdir -p ~/.local/share/fcitx5/pinyin/dictionaries/

# 移动词库文件至该目录
mv zhwiki-20220416.dict ~/.local/share/fcitx5/pinyin/dictionaries/

# 重启fcitx5使词库生效 
fcitx5 -r
```

## 8 自定义主题
Fcitx 5 默认的外观比较朴素，用户可以根据喜好使用自定义主题。

第一种方式为使用[经典用户界面](https://link.zhihu.com/?target=https%3A//fcitx-im.org/wiki/Theme_Customization/zh-cn%23%25E7%25BB%258F%25E5%2585%25B8%25E7%2594%25A8%25E6%2588%25B7%25E7%2595%258C%25E9%259D%25A2 "经典用户界面")，可以[在 GitHub 搜索主题](https://link.zhihu.com/?target=https%3A//github.com/search%3Fq%3Dfcitx5%2Btheme%26type%3DRepositories "在 GitHub 搜索主题")，然后在 [Fcitx5 configtool](https://zhuanlan.zhihu.com/write#fcitx-%E9%85%8D%E7%BD%AE "Fcitx5 configtool") —— 「附加组件」 —— 「经典用户界面」中设置即可

第二种方式为使用 [Kim面板](https://link.zhihu.com/?target=https%3A//fcitx-im.org/wiki/Theme_Customization/zh-cn%23kim%25E9%259D%25A2%25E6%259D%25BF "Kim面板")，一种基于 DBus 接口的用户界面。此处安装了 [Input Method Panel](https://link.zhihu.com/?target=https%3A//extensions.gnome.org/extension/261/kimpanel/ "Input Method Panel") 这个 GNOME 扩展，黑色的风格与正在使用的 GNOME 主题 [Orchis-dark](https://link.zhihu.com/?target=https%3A//www.gnome-look.org/p/1357889 "Orchis-dark") 非常搭配


推荐主题项目：https://github.com/thep0y/fcitx5-themes-candlelight?tab=readme-ov-file

```c
# 垂直候选列表
Vertical Candidate List=False

# 按屏幕 DPI 使用
PerScreenDPI=True

# Font (设置成你喜欢的字体)
Font="JetBrains Mono"

# 主题(这里要改成你想要使用的主题名，主题名就在下面)
Theme=macOS-dark
```