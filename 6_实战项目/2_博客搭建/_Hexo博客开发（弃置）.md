
内容来源：[安知鱼主题官方文档 | 一个简洁、美丽的静态hexo主题 (anheyu.com)](https://docs.anheyu.com/)
## 1 主题配置

### 1.1 前言

- **站点配置** ：指的 Hexo 博客目录下的` _config.yml`

- **主题配置** ：指的是 `theme/anzhiyu/_config.yml` 或者`_config.anzhiyu.yml` ，注意区分

-  **source 目录**：指的是博客目录下的 `source` 文件夹，不推荐修改主题内 `source` 目录；
  
- 每次无论 `hexo g` 或 `hexo s`，都最好先使用 `hexo clean` 清除本地缓存；
  
- 页面结果以本地 `hexo s` 为准，部署后的异常大部分是线上缓存原因，在确认没有报错的情况下，等待若干时间后即可正常

### 1.2 主题安装与应用

**方式一(Github 推荐):**
```bash
git clone -b main https://github.com/anzhiyu-c/hexo-theme-anzhiyu.git themes/anzhiyu
```

效果如图：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8518b140de4568d06d1290a1b2c99bd5.png)
如遇安装不上可以使用以下url代理安装
```bash
git clone -b main https://ghproxy.com/https://github.com/anzhiyu-c/hexo-theme-anzhiyu.git themes/anzhiyu
```

**方式二(Release 推荐):**
下载 anzhiyu主题 [最新的release版本](https://github.com/anzhiyu-c/hexo-theme-anzhiyu/releases) 解压到 `themes` 目录，并将解压出的文件夹重命名为 `anzhiyu`。

**方式三(npm安装):**
```bash
npm i hexo-theme-anzhiyu
```
此方法只支持 Hexo 5.0.0 以上版本 **通过 npm 安装并不会在 themes 里生成主题文件夹，而是在 node_modules 里生成**

### 1.3 应用主题

**第一步：** 应用主题

打开 **Hexo** 根目录下的 `config.yml`, 找到以下配置项，把主题改为`anzhiyu`
```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: anzhiyu
```

效果如图：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e5ee5853045b3244dbdfa7688a7cd6aa.png)


**第二步：** 安装 `pug` 和 `stylus` 渲染插件
```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

无法安装可以使用cnpm进行安装
```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save --registry=http://registry.npmmirror.com
```

效果如图：
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5aa69683893f4e5fffc7d828c7b67c75.png)


**第三步：** 覆盖配置

覆盖配置可以使`主题配置`放置在 anzhiyu 目录之外，避免在更新主题时丢失自定义的配置。

即将 `themes/anzhiyu/_config.yml` 这个文件复制到hexo 根目录，然后重命名为 `./_config.anzhiyu.yml`


效果如图：![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a21502c397316ff8100e9319ec111da.png)
**注意：**
- 只要存在于 `_config.anzhiyu.yml` 的配置都是高优先级，修改原 `_config.yml` 是无效的。
- 每次更新主题可能存在配置变更，请注意更新说明，可能需要手动对 `_config.anzhiyu.yml` 同步修改。
- 想查看覆盖配置有没有生效，可以通过 `hexo g --debug` 查看命令行输出。
- 如果想将某些配置覆盖为空，注意不要把主键删掉，不然是无法覆盖的


**第五步：** 一键三连（本地启动hexo）
```bash
hexo cl
hexo g
hexo s
```

**提示：**
- 如果你也使用的是WebStrom或者是VsCode软件开发博客内容，可以修改 `package.json` 文件，直接一键三连![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e5c4b8c9ed0cffc0881ec34e64231585.png)

- 修改server中的value值，如下代码即可
```json
"server": "hexo clean && hexo generate && hexo server"
```

效果如图：![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/112f05c247630365d1668375972f5734.png)
点击链接，即可进入网站静态页面看到效果了，按 `ctrl+c` 关闭端口（资源）

## 2 页面配置

### 2.1 Front-matter 的基本认识

**现在不了解也可以，后续使用到的时候，再回来看看**

`Front-matter` 是 `markdown` 文件最上方以 `---` 分隔的区域，用于指定个别档案的变数。其中又分为两种 markdown 里

**一定要明确，明确，明确！**
1. `Page Front-matter` 用于**页面**配置
2. `Post Front-matter` 用于**文章页**配置

> 如果标注可选的参数，可根据自己需要添加，不用全部都写在 markdown 里

- Page Front-matter
```markdown
title:
date:
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
type:
top_single_background:
```

|写法|解释|
|---|---|
|title|【必需】页面标题|
|date|【必需】页面创建日期|
|type|【必需】标签、分类、关于、音乐馆、友情链接、相册、相册详情、朋友圈、即刻页面需要配置|
|updated|【可选】页面更新日期|
|description|【可选】页面描述|
|keywords|【可选】页面关键字|
|comments|【可选】显示页面评论模块(默认 true)|
|top_img|【可选】页面顶部图片|
|mathjax|【可选】显示 mathjax(当设置 mathjax 的 per_page: false 时，才需要配置，默认 false)|
|katex|【可选】显示 katex(当设置 katex 的 per_page: false 时，才需要配置，默认 false)|
|aside|【可选】显示侧边栏 (默认 true)|
|aplayer|【可选】在需要的页面加载 aplayer 的 js 和 css,请参考文章下面的音乐 配置|
|highlight_shrink|【可选】配置代码框是否展开(true/false)(默认为设置中 highlight_shrink 的配置)|
|top_single_background|【可选】部分页面的顶部模块背景图片|
- Post Front-matter
```markdown
title:
date:
updated:
tags:
categories:
keywords:
description:
top_img:
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
swiper_index: 1
top_group_index: 1
background: "#fff"
```

|写法|解释|
|---|---|
|title|【必需】文章标题|
|date|【必需】文章创建日期|
|updated|【可选】文章更新日期|
|tags|【可选】文章标签|
|categories|【可选】文章分类|
|keywords|【可选】文章关键字|
|description|【可选】文章描述|
|top_img|【可选】文章顶部图片|
|cover|【可选】文章缩略图(如果没有设置 top_img,文章页顶部将显示缩略图，可设为 false/图片地址/留空)|
|comments|【可选】显示文章评论模块(默认 true)|
|toc|【可选】显示文章 TOC(默认为设置中 toc 的 enable 配置)|
|toc_number|【可选】显示 toc_number(默认为设置中 toc 的 number 配置)|
|toc_style_simple|【可选】显示 toc 简洁模式|
|copyright|【可选】显示文章版权模块(默认为设置中 post_copyright 的 enable 配置)|
|copyright_author|【可选】文章版权模块的`文章作者`|
|copyright_author_href|【可选】文章版权模块的`文章作者`链接|
|copyright_url|【可选】文章版权模块的`文章链接`链接|
|copyright_info|【可选】文章版权模块的版权声明文字|
|mathjax|【可选】显示 mathjax(当设置 mathjax 的 per_page: false 时，才需要配置，默认 false)|
|katex|【可选】显示 katex(当设置 katex 的 per_page: false 时，才需要配置，默认 false)|
|aplayer|【可选】在需要的页面加载 aplayer 的 js 和 css,请参考文章下面的`音乐` 配置|
|highlight_shrink|【可选】配置代码框是否展开(true/false)(默认为设置中 highlight_shrink 的配置)|
|aside|【可选】显示侧边栏 (默认 true)|
|swiper_index|【可选】首页轮播图配置 index 索引，数字越小越靠前|
|top_group_index|【可选】首页右侧卡片组配置, 数字越小越靠前|
|ai|【可选】文章ai摘要|
|main_color|【可选】文章主色，必须是16进制颜色且有6位，不可缩减，例如#ffffff 不可写成#fff|

1. 首页轮播图配置: `swiper_index`, 数字越小越靠前
2. 首页卡片配置: `top_group_index`, 数字越小越靠前
3. page 中`top_single_background`, 可配置部分页面的顶部背景图片

> 只需要在你的文章顶部的`Front-matter`配置这`swiper_index`和`top_group_index`两个字段即可显示轮播图和推荐卡片

### 2.2 标签页配置

**第一步：** 前往你的 Hexo 博客的根目录

**第二步：** 在 Hexo 博客根目录 `[blog]`下打开终端，输入
```bash
hexo new page tags
```

**第三步：** 你会找到 `source/tags/index.md` 这个文件
![image.png|380|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2b35560792974c4f15ada088affc511f.png)

**第四步：** 修改这个文件： 记得添加 `type: "tags"`
```markdown
---
title: 标签
date: 2021-04-06 12:01:51
type: "tags"
comments: false
top_img: false
---
```

|参数|解释|
|---|---|
|type|【必须】页面类型，必须为 tags|
|comments|【可选】是否显示评论|
|top_img|【可选】是否显示顶部图|
|orderby|【可选】排序方式 ：random/name/length|
|order|【可选】排序次序： 1, asc for ascending; -1, desc for descending|
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b64b1ebc505dcbd1d39c95207290e2d.png)

### 2.3 分类页配置

**第一步：** 前往你的 Hexo 博客的根目录

**第二步：** 在 Hexo 博客根目录 `[blog]`下打开终端，输入
```bash
hexo new page categories
```

**第三步：** 你会找到 `source/tags/index.md` 这个文件
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e6421bb9dec21b3c9b41967ea44530a.png)


**第四步：** 修改这个文件： 记得添加 `type: "tags"`
```markdown
---
title: 标签
date: 2021-04-06 12:01:51
type: "tags"
comments: false
top_img: false
---
```

### 2.4 404页面配置

主题内置了一个简单的 404 页面，可在设置中开启 本地预览时，访问出错的网站是不会跳到 404 页面的。

如需本地预览，请访问 `http://localhost:4000/404.html`

```yml
# A simple 404 page
error_404:
  enable: true
  subtitle: "页面没有找到"
  background:
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/633fe4937a41c23ae06c7231ca46cd4a.png)
