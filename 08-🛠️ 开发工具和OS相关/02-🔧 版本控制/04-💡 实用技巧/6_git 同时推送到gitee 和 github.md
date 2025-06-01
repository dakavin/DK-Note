---
文章标题: "[[6_git 同时推送到gitee 和 github]]"
文章作者: Dakkk
文章概要: |
  文章介绍了两种将Git代码同时推送到Gitee和GitHub的方法。一是配置独立远程仓库并分别推送；二是利用Git 2.4+的`pushurl`多值特性，通过修改`.git/config`，实现单条`git push`命令同步更新多个远程仓库，提高效率。
tags:
  - Git
  - 远程仓库
  - Gitee
  - GitHub
  - 代码同步
  - Git配置
  - pushurl
相关文章:
  - "[[1_本章代码及文章获取]]"
  - "[[../../../13-🔧 系统配置/04-🛠️ Obsidian技巧/3_Obsidian实现阿里云同步和Git备份]]"
  - "[[3_Hexo+GitHub+个人域名 博客搭建教程]]"
  - "[[2_gitkralen的破解及汉化]]"
  - "[[7_挂着Clash的时候git操作push失败的问题]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/02-🔧 版本控制/04-💡 实用技巧/6_git 同时推送到gitee 和 github.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 方式一：

```shell
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true

[remote "origin"]
	url = git@gitee.com:Dakkk_mike/java_-study.git
	fetch = +refs/heads/*:refs/remotes/origin/*

[remote "github"]
	url = git@github.com:dakavin/DK-Note.git
	fetch = +refs/heads/*:refs/remotes/github/*

[branch "master"]
	remote = origin
	merge = refs/heads/master

```

- **默认 `git push` 会推送到 `origin`（即 Gitee）**
```shell
git push 
```

- **推送到 Github**
```shell
git push github master
git push github HEAD
```

- **如果想同时推送到两个仓库，可以自己写脚本，或者分别执行两条命令**
```shell
git push origin master
git push github master
```

## 方式二：

让 `git push` 自动推送到两个仓库

Git 本身不支持一条命令同时推送到两个远程仓库，但可以用下面方法之一：
1. **添加一个新的远程，URL 采用 “多个 URL” 形式（但这需要 Git 2.4+）**

示范修改 `origin` 的 URL 支持多个地址：

编辑 `.git/config` 文件，将 Gitee 和 GitHub 的 `url` 和 `pushurl` 合并到 `origin` 远程仓库的配置中。
```shell
[remote "origin"]
    url = git@gitee.com:Dakkk_mike/java_-study.git
    url = git@github.com:dakavin/DK-Note.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

这样当你执行 `git push origin` 时，Git 会分别推送到两个 URL。

```gitconfig
[core]
    repositoryformatversion = 0
    filemode = false
    bare = false
    logallrefupdates = true
    symlinks = false
    ignorecase = true

[remote "origin"]
    url = gitee远程地址
    fetch = +refs/heads/*:refs/remotes/origin/*
    pushurl = gitee远程地址
    pushurl = github远程地址
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

参考截图 ：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/12/6088c55a790f59caa6ee43e96991508f.png)

