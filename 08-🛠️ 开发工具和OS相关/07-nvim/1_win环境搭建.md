
字体包：[Nerd Fonts - Iconic font aggregator, glyphs/icons collection, & fonts patcher](https://www.nerdfonts.com/font-downloads)

nvim下载安装连接：[Releases · neovim/neovim](https://github.com/neovim/neovim/releases)
- 下载msi包，不然appdata没有对应的数据

lazyvim安装连接：[🛠️ Installation | LazyVim](https://www.lazyvim.org/installation)
- 打开powershell进行如下操作

**备份原有的数据：**
```shell
# required
mv ~/.config/nvim{,.bak}

# optional but recommended
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

**克隆lazyvim到appdata中的nvim**
```shell
git clone https://github.com/LazyVim/starter ~/.config/nvim
```

**移出git目录，这样可以将自己的配置保存在git上**
```shell
rm -rf ~/.config/nvim/.git
```

**开始nvim：** 打开nvim安装所在文件夹中的bin目录，执行nvim即可

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/0cf5fd6f9859bd15be1a2f7c38ab79a5.png)
