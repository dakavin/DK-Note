---
文章标题: "[[5_Commit 提交规范]]"
文章作者: Dakkk
文章概要: |
  文章详细阐述了基于Angular的Git提交规范，包括类型、范围、主题、正文和脚注的编写指南。强调标准化提交对代码审查、生成发布日志和项目维护的重要性，并介绍了Commitizen等工具辅助实践。
tags:
  - Git
  - Commit 规范
  - Angular
  - Commitizen
  - conventional-changelog
  - 代码管理
  - 版本控制
  - 团队协作
相关文章:
  - "[[1_本章代码及文章获取]]"
  - "[[14_RESTful规范]]"
  - "[[3_Hexo+GitHub+个人域名 博客搭建教程]]"
  - "[[../../../13-🔧 系统配置/04-🛠️ Obsidian技巧/3_Obsidian实现阿里云同步和Git备份]]"
  - "[[2_gitkralen的破解及汉化]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/02-🔧 版本控制/04-💡 实用技巧/5_Commit 提交规范.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

参考文章：https://mp.weixin.qq.com/s/mvYkeuY2P6vRZro4vj5g9g

Git commit message规范指提交代码时编写的规范注释，编写良好的Commit messages可以达到3个重要的目的：

- 加快代码review的流程
    
- 帮助我们编写良好的版本发布日志
    
- 让之后的维护者了解代码里出现特定变化和feature被添加的原因
    
## 1 Angular Git Commit Guidelines

业界应用的比较广泛的是Angular Git Commit Guidelines：
```text
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- `type`：提交类型
- `scope`：可选项，本次 commit 波及的范围
- `subject`：简明扼要的阐述下本次 commit 的主旨，在`Angular Git Commit Guidelines`中强调了三点。使用祈使句，首字母不要大写，结尾无需添加标点
- `body`: 同样使用祈使句，在主体内容中我们需要把本次 commit 详细的描述一下，比如此次变更的动机
- `footer`: 描述下与之关联的 issue 或 break change

**简易版**
项目中实际可以采用简易版规范：

```text
<type>(<scope>):<subject>
```

### 1.1 type规范

`Angular Git Commit Guidelines`中推荐的type类型如下：

- feat: 新增功能
- fix: 修复bug
- docs: 仅文档更改
- style: 不影响代码含义的更改（空白、格式设置、缺失 分号等）
- refactor: 既不修复bug也不添加特性的代码更改
- perf: 改进性能的代码更改
- test: 添加缺少的测试或更正现有测试
- chore: 对构建过程或辅助工具和库（如文档）的更改

除此之外，还有一些常用的类型：
- delete：删除功能或文件
- modify：修改功能
- build：改变构建流程，新增依赖库、工具等（例如webpack、gulp、npm修改）
- test：测试用例的新增、修改
- ci：自动化流程配置修改
- revert：回滚到上一个版本

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e913b16945baaa09b36ae5ad89799899.png)

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a83255e411ba45e410c2039fe9e9f24.png)

### 1.2 scope

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在`Angular`，可以是`$location`, `$browser`, `$compile`, `$rootScope`, `ngHref`, `ngClick`, `ngView`等。

如果你的修改影响了不止一个`scope`，你可以使用`*`代替。

### 1.3 subject

`subject`是 commit 目的的简短描述，不超过50个字符。

其他注意事项：

- 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
- 第一个字母小写
- 结尾不加句号（.）

### 1.4 Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```text
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点:

- 使用第一人称现在时，比如使用change而不是changed或changes。
- 永远别忘了第2行是空行
- 应该说明代码变动的动机，以及与以前行为的对比。

### 1.5 Footer

Footer 部分只用于以下两种情况：

#### 1.5.1 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法。

```text
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

#### 1.5.2 关闭 Issue

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

```text
Closes #234
```

#### 1.5.3 Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header。

```text
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit &lt;hash>`.，其中的hash是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。
### 1.6 单次提交注意事项

- 提交问题必须为同一类别
- 提交问题不要超过3个
- 提交的commit发现不符合规范，`git commit --amend -m "新的提交信息"`或 `git reset --hard HEAD` 重新提交一次

## 2 Commitizen

可以使用典型的git工作流程或通过使用CLI向导[Commitizen](https://link.zhihu.com/?target=https%3A//github.com/commitizen/cz-cli)来添加提交消息格式。
### 2.1 安装

全局安装commitizen插件
```text
npm install -g commitizen cz-con
```

然后，在项目根目录里，运行下面的命令，使其支持 Angular 的 Commit message 格式。
```text
# 适用于npm
commitizen init cz-conventional-changelog --save --save-exact

# 适用于pnpm
 pnpx commitizen init cz-conventional-changelog --pnpm -Dw

# 移除操作
npm uninstall cz-conventional-changelog commitizen
# 或
yarn remove cz-conventional-changelog commitizen
# 移除Husky钩子
rm .husky/commit-msg

```

[pnpm中使用commitizen init](https://juejin.cn/post/7215608036394827836)

这将在项目中安装 `cz-conventional-changelog` 适配器，并在 `package.json` 中添加相应的配置


以后，凡是用到`git commit`命令，一律改为使用`git cz`。这时，就会出现选项，用来生成符合格式的 Commit message。  

下面git cz 的具体解释
```shell
# 命令解析
## 本次提交你修改的类型是什么？使用方向键进行选择
Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature # 实现新功能
  fix:      A bug fix # bug修复
  docs:     Documentation only changes # md文件修改
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)  # 样式修改
  refactor: A code change that neither fixes a bug nor adds a feature # 功能重构
  perf:     A code change that improves performance # 性能提升
  test:     Adding missing tests or correcting existing tests # 与测试有关的更改
  build:    Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm) # 影响到项目构建的相关修改
  ci:       Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs) # 对CI配置文件和脚本的修改
  chore:    Other changes that don't modify src or test files # 不会修改src或测试文件的更改
  revert:   Reverts a previous commit # 恢复上一次提交

## 本次提交更改的范围
What is the scope of this change (e.g. component or file name): (press enter to skip) # 输入更改的内容后按回车，此处我的内容是：设计图

## 本次提交修改内容的简短概括，最多89个字
 Write a short, imperative tense description of the change (max 89 chars):
 (0)   # 输入本次更改的内容后按回车，此处我的内容是：设计模块添加设计图
 
## 本次提交修改内容的详细描述，用1，2，3..数字来描述，每一点之间用空格隔开
Provide a longer description of the change: (press enter to skip) # 此处我的内容是：1.添加反馈设计图 2.添加文件列表设计图 3.添加账户信息设计图

## 是否有重大变化
Are there any breaking changes? (y/N) # 此处我输入的是N

## 此更改是否会影响到未解决的问题
Does this change affect any open issues? (y/N) # 此处我输入的是N
```

内容填写参考指引：
- 选择提交类型：
  ![|300|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/02f4764ab10fd4e861f287fe7e99deb5.webp)
	- feat:     新功能
	- fix:      修复 Bug
	- docs:     文档更新
	- style:    代码样式调整
	- refactor: 代码重构
	- perf:     性能优化
	- test:     添加或更新测试
	- chore:    构建过程或辅助工具的变动
- 这次提交的改动所影响的范围? (按回车键跳过)
	- 如果我们的更改影响到特定的模块或组件，可以在这里输入相应的范围，否则直接按回车键跳过。
- 输入一个简短的描述：
	- 写一个简短的变化描述，使用命令式语气，尽量包含主语（50个字符以内）
	- 输入一个简洁明了的描述，说明这次提交的主要变化，例如：添加用户注册功能
- 输入一个更详细的描述（可选）：
	- 如果需要提供更多关于这次变更的信息，可以在这里输入多行描述，每行以 `|` 符号开头。如果不需要详细描述，直接按回车键跳过。
- 是否有任何破坏性变更：
	- 如果这次提交包含了破坏性变更，即可能影响到其他部分的功能或者与之前的版本不兼容，需要输入 `y` 并按回车键。否则，直接按回车键选择默认的 `N`。
- 是否关闭了某个 Issue：
	- 如果这次提交解决了某个 Issue，可以输入 `y` 并在提示中输入 Issue 的编号，多个 Issue 编号以逗号分隔。如果不关闭任何 Issue，直接按回车键选择默认的 `N`。



git cz的具体过程参考图片
![image.png|300|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c7d942c8acd0cb8ec35107286950f9c.png)
### 2.2 参考示例

## 3 validate-commit-msg

[validate-commit-msg](https://link.zhihu.com/?target=https%3A//github.com/kentcdodds/validate-commit-msg) 用于检查项目的 Commit message 是否符合Angular规范。

该包提供了使用githooks来校验`commit message`的一些二进制文件。在这里，我推荐使用[husky](https://link.zhihu.com/?target=http%3A//npm.im/husky)，只需要添加 `"commitmsg": "validate-commit-msg"` 到你的`package.json`中的`nam scripts`即可.

当然，你还可以通过定义配置文件`.vcmrc`来自定义校验格式。详细使用请见文档 [validate-commit-msg](https://link.zhihu.com/?target=https%3A//github.com/kentcdodds/validate-commit-msg)

## 4 生成 Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成。生成的文档包括以下三个部分：

- New features
- Bug fixes
- Breaking changes.

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

[conventional-changelog](https://link.zhihu.com/?target=https%3A//github.com/ajoslin/conventional-changelog) 就是生成 Change log 的工具，运行下面的命令即可。

```text
$ npm install -g conventional-changelog
$ cd my-project
$ conventional-changelog -p angular -i CHANGELOG.md -w
```