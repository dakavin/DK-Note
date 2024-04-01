
在之前的操作中，所有的操作都是基于一条主线完成的。就好比，咱们学习的时候，记学习笔记，今天学点，那么就写一点，明天学点，再写一点，最后，完全学完了，这个笔记也就记全了。

但实际上，有些文件可能再不同的场合需要同时使用不同的内容，而且还不能冲突，比如项目的配置文件，我需要本地进行测试，同时还要部署到服务器上进行测试。

本地和服务器上的环境是不一样的，所以同一个配置文件就需要根据环境的不同，进行不同的修改。本地环境没问题了，修改配置文件，提交到服务器上进行测试，如果测试有问题，再修改为本地环境，重新测试，没问题了，再修改为服务器配置，然后提交到服务器上进行测试。依次类推，形成迭代式开发测试。

从上面的描述上看，就会显得非常繁琐，而且本质上并没有太重要的内容，`仅仅是因为环境上的变化`，就需要重新修改，所以如果`将本地测试环境和服务器测试环境区分开，分别进行文件版本维护`，是不是就会显得更合理一些。这个操作，在Git软件中，我们称之为branch分支。

这里的分支感觉上就是树上的分叉一样，会按照不同的路线生长下去。有可能以后不再相交，当然，也可能以后会不断地纠缠下去，都是有可能的。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6336eca1c44d6fc3656a4216e310309a.png)
## 1、Git分支

### 1.1 主干分支

默认情况下，Git软件就存在分支的概念，而且就是一个分支，称之为master分支，也称之为主干分支。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24f8fd2aaf489984bcb23cc26408f8f1.png)

这就意味着，所有文件的版本管理操作都是在master这一个分支路线上进行完成的。

不过奇怪的是，为什么之前的操作没有体现这个概念呢，那是因为，默认的所有操作本身就都是基于master分支完成的。而master主干分支在创建版本库时，也就是git init时默认就会创建。

### 1.2 其他分支

就像之前说的，如果仅仅是一个分支，在某些情况并不能满足实际的需求，那么就需要创建多个不同的分支。
#### 1.2.1 创建分支

```shell
# git branch** **分支名称**

git branch b1

git branch b2
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1645c87a29531e6f3547fdb66fe6ab7f.png)

现在我们创建了2个分支，不过这两个分支都是基于master主干分支为基础的。
#### 1.2.2 查看分支

```shell
git branch -v
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/854051c8f79e0ead66cc13ccf615e98c.png)
#### 1.2.3 切换分支

我们将工作线路切换到b1
```shell
# git checkout 分支名称

git checkout b1
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8d87e7307aec8037e10d3fb84e9f388.png)

此时我们添加新的文件b1.txt
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4cb2c5e605d1828af4ab360d4fe3959f.png)

然后提交到版本库
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/456731f336177fc638b6a68d0806129c.png)

此时，查看分支信息，会发现不同分支的版本进度信息发生了改变
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9473b888eb6b5da95cef24bddf27b64c.png)

如果此时切换回到主干分支的话，那么b1.txt文件就不存在了，因为对应版本信息不一样。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6027ba2e91942b71d6252cc26539fa70.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/813f9569b6e690637fb40d88e64b821b.png)
#### 1.2.4 删除分支

如果觉得某一个分支建立的不太理想或已经没有必要在使用了，那么是可以将这个分支删除的。
```shell
# git branch -d 分支名称

Git branch -d b2
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e2718ccf4bf4a83023875fec54ed6b4.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e06ca90de06fa5bb69a81de98e4670f1.png)

## 2、Git合并

无论我们创建多少个分支，都是因为我们需要在不同的工作环境中进行工作，但是，最后都应该将所有的分支合在一起。形成一个整体。作为项目的最终结果。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a276193e387c85097a03c2bb35db82f.png)

### 2.1 主干分支

首先我们先将主干分支的所有文件清空掉

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/64767e3cac47b2dede4b5d5e9d2a4591.png)

在当前主干分支中创建一份文件master.txt，并提交

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b4062ec11f16e63b80604dec9acf51ad.png)
### 2.2 其他分支

基于主干分支的内容，我们创建其他分支，并直接切换到新的分支
```shell
# git checkout -b 分支名称

git checkout -b new_branch
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/61cc05360af8e813aa789b188463e693.png)

在新的分支中添加新文件branch.txt
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/625fb0cbfe22caae9f3e4a3fb58408fe.png)

此时切换回主干分支，只有master.txt文件。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c4895ffff309b56b810a364bc0b7969.png)

再切换回new_branch分支，branch文件就又回来了。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5d0f868780056b0fcbef8c00db6b7e26.png)
### 2.3 合并分支

这里我们将new_branch分支的文件内容合并到主干分支中。首先先切换回主干分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/94fc51b844ac7eb092107331247a798b.png)

然后执行分支合并指令
```shell
# git merge 分支名称

git merge new_branch
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c52190f0aaa9c9927908edce8bce9168.png)

此时再次查看文件，就会发现branch.txt文件已经可以看到了。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dfd1e73f4b060e4b3dcb1b2501064a5d.png)
## 3、Git冲突

在多分支并行处理时，每一个分支可能是基于不同版本的主干分支创建的。如果每隔分支都独立运行而不进行合并，就没有问题，但是如果在后续操作过程中进行合并的话，就有可能产生冲突。比如B1, B2的两个分支都是基于master分支创建出来的。B1分支如果和B2分支修改了同一份文件的话，那么在合并时，以哪一个文件为准呢，这就是所谓的冲突。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e5465f554374472dbc64652e0071efb.png)
### 3.1 主干分支

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/909fccd96c191afe09d059c00f195083.png)

首先我们先将主干分支的所有文件清空掉

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fdee765b6fc353256e459f80657a8db3.png)

主干分支添加文件test.txt，文件内容为空

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bcb1fec743648c7447a2e4fbecda195c.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2889646789e711d3bfb6fb63212d575f.png)

### 3.2 其他分支

基于主干分支，创建两个分支B1, B2

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/23f193cbddf74952d20173b0760b56da.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88685c09b54dc2112094b720ce4dda02.png)

### 3.3 切换分支-B1

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d369637ef79e86c43a2cb587ba331b15.png)

切换到B1分支，修改文件内容

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eeebb18c4d9bc4c22c9c01cc4bdf7078.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4b5aea3e0f91d5f1f73b54bae88a4a03.png)

提交修改后的文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71d5a4e487849cf91d50e0ac3fb8fbcc.png)

### 3.4 切换分支-B2

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e14ef95088542b236a8e0e78a93552a.png)

切换到B2分支，查看文件内容

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79d368a1f4027fc582f406a98ccf8219.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d4c44e5f33f7c24e5278782e0178baf.png)

修改文件内容：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fd4a4ecda8ad6c3f6f433828e40d7cb5.png)

提交文件

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2126dd272838994ae3cb712a8c5627c2.png)

### 3.5 合并分支-B1

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/566924c929e9504314662c2b65cb7730.png)

切换到master主干分支，此时test.txt文件内容为空

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae179c0bc46be1838364c9f0b81c339b.png)

将B1分支合并到主干分支中

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24f6e92d507e60284db993388514e653.png)

### 3.6 合并分支-B2

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f7b324d47cf2720eccf465bddfd2d4c3.png)

因为B2分支也对文件进行了修改，所以如果此时合并B2分支,就会提示冲突

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e1028d6a3dfa3b47b700dacd892c082.png)

查看文件内容差异

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/585145511936dbdcc5520c687416dfee.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9821bf4c32fac37b9bddc80baaefd04.png)

这里的冲突，软件是无法判断该如何出来处理的，所以需要人工进行判断，将冲突的文件内容进行修正。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aee12f218e7b1bcd210fb77314bf1b25.png)

重新提交到master主干分支中。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/85922a50edbebb8e3218125592e367a7.png)

```shell
# git commit 文件名称 -i -m 注释
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/937d0cae3e1f9f6af29da2f8859408dc.png)

再查看一下Git软件的操作日志
```shell
# git log --graph
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/551846e1625e5dd441b49cfd52a8606b.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d264ea58920536fe174ed35868683456.png)



