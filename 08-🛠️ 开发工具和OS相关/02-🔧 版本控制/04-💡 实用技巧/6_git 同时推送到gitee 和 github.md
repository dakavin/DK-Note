
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

