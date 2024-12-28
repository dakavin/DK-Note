
编辑 `.git/config` 文件，将 Gitee 和 GitHub 的 `url` 和 `pushurl` 合并到 `origin` 远程仓库的配置中。

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

参考截图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/12/6088c55a790f59caa6ee43e96991508f.png)
