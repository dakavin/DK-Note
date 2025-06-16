---
文章标题: "[[1_win环境搭建]]" 
文章作者: Dakkk
文章概要: |
  本文详细指导用户如何在Windows环境下安装Neovim文本编辑器，并集成LazyVim配置框架。内容涵盖字体包下载、Neovim安装、原有配置备份及LazyVim克隆等关键步骤，旨在提供高效的开发环境搭建方案。
tags:
- "Neovim"
- "LazyVim"
- "Windows环境搭建"
- "文本编辑器"
- "开发环境"
- "配置指南"
- "命令行"
相关文章:
- "[[10_文本编辑器（Vim）]]"
- "[[3_VIM使用场景]]"
- "[[0_快捷键大全]]"
- "[[0_vscode的安装与使用]]"
- "[[1_常用文件夹]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/07-nvim/1_win环境搭建.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-15 22:53:56
修改时间: 2025-05-28 01:03:09
---


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
