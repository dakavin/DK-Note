---
文章标题: "[[4_Hexo博客开发（一）]]" 
文章作者: Dakkk
文章概要: |
  文章详细指导Hexo博客用户进行多项优化。内容涵盖使用`hexo-abbrlink`插件实现永久链接、设置全局背景、通过`hexo-magnet`插件定制顶部磁吸分类展示，并提供该插件的bug修复方案。文章侧重实际操作，旨在提升Hexo博客的功能性和用户体验。
文章标签:
- "Hexo"
- "博客优化"
- "永久链接"
- "hexo-abbrlink"
- "hexo-magnet"
- "插件配置"
- "博客定制"
- "CSS"
相关文章:
- "[[0_JavaWeb概述]]"
- "[[2_CSS]]"
- "[[2_HTML和CSS]]"
- "[[_魔改anzhiyu主题日志]]"
- "[[_Hexo博客开发（弃置）]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/04-🌐 Hex博客项目/4_Hexo博客开发（一）.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 01:03:34
---

## 1 博客生成永久链接

### 1.1 前言

Hexo 文章链接默认的生成规则是：`:year/:month/:day/:title`，是按照年、月、日、标题来生成的。

这样的话，生成的链接非常长长长长长，而且如果我们的 Markdown 使用中文标题，那就更惨了，URL 一转码，将是一场灾难。

更难受的是如果我们修改了文章的日期或者标题，那么将导致链接改变，别人或者你分享出去的文章就会 404，这非常的蛋疼啊，所以就有了这种插件，不论你如何修改文章的日期、名称，只要不改变 footer-matter 中的 id 值，那么文章链接永远不会变。

### 1.2 安装插件

```shell
npm install hexo-abbrlink --save
```

### 1.3 使用

修改 `_config.yml` ：
```yml
## permalink: :year/:month/:day/:title/  
permalink: posts/:abbrlink.html  ## 此处可以自己设置
```

增加以下配置：（还是在  `_config.yml` ）
```yml
## abbrlink config  
abbrlink:  
  alg: crc32      #support crc16(default) and crc32 进制  
  rep: hex        #support dec(default) and hex  算法  
  drafts: false   #(true)Process draft,(false)Do not process draft. false(default)   
  ## Generate categories from directory-tree  
  ## depth: the max_depth of directory-tree you want to generate, should > 0  
  auto_category:  
     enable: true  #true(default)  
     depth:        #3(default)  
     over_write: false   
  auto_title: false #enable auto title, it can auto fill the title by path  
  auto_date: false #enable auto date, it can auto fill the date by time today  
  force: false #enable force mode,in this mode, the plugin will ignore the cache, and calc the abbrlink for every post even it already had abbrlink.
```

其他配置意义可查看插件文档哦，链接在顶部。

记着要先 `hexo clean` 再 `hexo g` 哦～～～
## 2 全局背景

### 2.1 具体操作

你可以选择在主题配置文件内更改背景图片的url
```yaml
# The formal of image: url(http://xxxxxx.com/xxx.jpg)  
background: url( )
```

也可以选择通过引入css更改
```css
#web_bg {  
    background: linear-gradient(180deg,#fff1eb,#ace0f9)  
}
```

### 2.2 引入

1. 先在博客文件中创建css、js和img的文件夹，用于存放相关资源![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9cbe90164c8fa84375dacea4f4f926fa.png)
2. 复制上面的css代码，然后在css文件夹创建一个css文件，粘贴代码
3. 在主题配置文件中，如下，引入css文件![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1a15724d49569cd650fcaa7092c64138.png)
4. **注意：** 主题配置文件中，此选项不能为空，随便选一个图片链接就可以了![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce0053ad0b416449fa26366fbb0033df.png)


## 3 顶部磁吸分类

### 3.1 前言

这个插件主要实现了以下功能： 
1. 自定义 tags 或 categories 的排列和展示 
2. 自定义 tags 或 categories 的展示图标，名称 
3. 自定义排列的行数，默认 2 行
### 3.2 安装

```shell
npm i hexo-magnet --save  
  
# 或者  
  
cnpm i hexo-magnet --save
```
### 3.3 配置

在hexo的配置文件中，增加如下代码：
```yaml
magnet:  
  # 是否开启插件  
  enable: true  
  # 插件的叠放顺序，数字越大，叠放约靠前。如果你安装了 hexo-githubcalendar，请将 hexo-githubcalendarnpm 插件更新至 @1.2.3 版本。然后给 hexo-githubcalendar 添加 priority 参数。  
  priority: 1  
  # 路由地址，如 / 代表主页。/me/ 代表自我介绍页等等  
  enable_page: /  
  # 选择筛选分类还是标签  
  type: categories  
  # 表示分隔的列数，2 表示分为两列展示  
  devide: 2  
  # 配置项，可自行设置，按照设置的顺序展示  
  display:  
    - name: 教程  
      display_name: dakkkの魔改教程  
      icon: 📚  
    - name: 游戏评测  
      display_name: dakkkの游戏评测  
      icon: 🎮  
    - name: 生活杂记  
      display_name: dakkkの生活趣闻  
      icon: 🐱‍👓  
    - name: java  
      display_name: dakkkの编程学习  
      icon: 👩‍💻  
    - name: 学习  
      display_name: dakkkの读书笔记  
      icon: 📒  
    - name: 随想  
      display_name: dakkkの胡思乱想  
      icon: 💡  
  color_setting:  
    text_color: black # 文字默认颜色  
    text_hover_color: white # 文字鼠标悬浮颜色  
    background_color: "#f2f2f2"  # 文字背景默认颜色  
    background_hover_color: "#b30070" # 文字背景悬浮颜色  
  # 如果说 magnet 是一幅画，那么这个 layout 就是指定了哪面墙来挂画  
  layout:  
    type: id  
    name: recent-posts  
    index: 0  
  #包含挂载容器  
  temple_html: '<div class="recent-post-item" style="width:100%;height: auto"><div id="catalog_magnet">${temple_html_item}</div></div>'  
  #提供可自定义的 style，如加入黑夜模式。  
  plus_style: ""
```

`注意：你的文章font中的分类，要和此处分类名称一致`

这里仅仅展示 butterfly 配置，更多主题配置请前往：[https://github.com/Zfour/hexo-magnet/issues](https://github.com/Zfour/hexo-magnet/issues)  
也欢迎共享自己的配置和进行修改。

**然后hexo一键三连即可**
### 3.4 问题

出现data_list is not iterable问题

将如下代码，完全复制到node_modules中magnet插件目录下的index.js即可
```js
function priority_magnet(){
    var priority = 0
    if(hexo.config.magnet.priority){
        priority = hexo.config.magnet.priority
    }
    return priority
}

hexo.extend.filter.register('after_generate',function() {
    var magnet = hexo.config.magnet.enable;
    if(magnet){
        var temple = hexo.config.magnet.temple_html.split("${temple_html_item}")
        var layout_name ='';
        var layout_type ='';
        var layout_index =0;
        layout_name = hexo.config.magnet.layout.name;
        layout_type = hexo.config.magnet.layout.type;
        layout_index =  hexo.config.magnet.layout.index;
        var get_layout = ''
        if (layout_type == 'class'){
            get_layout =  `document.getElementsByClassName('${layout_name}')[${layout_index}]`
        }else if (layout_type == 'id'){
            get_layout =  `document.getElementById('${layout_name}')`
        }else {
            get_layout =  `document.getElementById('${layout_name}')`
        }
        var data_list =[]
        var load_more_href = ''
        //只修改了按categories展示-----
        //获取categories种类数
        var raw_categories_list = hexo.locals.get('categories').data;
        var categories_list= Object.values(raw_categories_list).map(item => ({
            name: item.name,
            _id: item._id
          }));
        //获取每个种类的文章数
        const categories = {};
        hexo.locals.get('posts').forEach(post => {
        const postCategories = post.categories.toArray();
        postCategories.forEach(category => {
            if (!categories[category.name]) {
            categories[category.name] = 1;
            } else {
            categories[category.name]++;
            }
        });
        });
        //按tags展示同理修改，
        var tag_list = hexo.locals.get('tags').data;
        if(hexo.config.magnet.type ==='categories'){
            data_list = categories_list
            load_more_href = hexo.config.url + '/categories'
        }else if (hexo.config.magnet.type ==='tags'){
            data_list = tag_list
            load_more_href = hexo.config.url+ '/tags'
        }
        var categories_new_list = [];
        var temple_html_item = '';
        if (hexo.config.magnet.devide>3){
            br_devide = '<br>'
        }else{
            br_devide =' '
        }
        var devide = 100 / hexo.config.magnet.devide;
        if(hexo.config.magnet.display){
            for (j of hexo.config.magnet.display){
                for (item of data_list){
                    if(j.name == item.name){
                        var item_group = {}
                        item_group[item.name]=categories[item.name]
                        categories_new_list.push(item_group)
                        temple_html_item += `<div class="magnet_item"><a class="magnet_link" href="${hexo.config.url}/categories/${item.name}/"><div class="magnet_link_context" style=""><span style="font-weight:500;flex:1">${j.icon} ${j.display_name}${br_devide}(${categories[item.name]})</span><span style="padding:0px 4px;border-radius: 8px;"><i class="fas fa-arrow-circle-right"></i></span></div></a></div>`;
                    }
                }
            }
        } //当display没元素时，感觉没必要，但是留下来了
        else{
            for (item of data_list){
                var item_group = {}
                item_group[item.name]=categories[item.name]
                categories_new_list.push(item_group)
                temple_html_item += `<div class="magnet_item"><div style="display:flex;padding: 20px;font-size:16px;"><span style="font-weight:500;flex:1">${item.name} (${categories[item.name]})</span><span style="padding:0px 4px;border-radius: 8px;"><i class="fas fa-arrow-circle-right"></i></span></div></div>`;
            }
        }
        if((categories_new_list.length % hexo.config.magnet.devide)>0){
            for(var i=0;i<(hexo.config.magnet.devide-categories_new_list.length % hexo.config.magnet.devide);i++){
                temple_html_item += `<div class="magnet_item" style="visibility: hidden"></div>`
            }
        }
        var load_more = `<a class="magnet_link_more"  href="${load_more_href}" style="flex:1;text-align: center;margin-bottom: 10px;">查看更多...</a>`
        temple_html_item += load_more
        var temple_html =`${temple[0]}${temple_html_item}${temple[1]}`;
        var script_text = ` <script data-pjax>if(${get_layout} && (location.pathname ==='${hexo.config.magnet.enable_page}'|| '${hexo.config.magnet.enable_page}' ==='all')){
    var parent = ${get_layout};
    var child = '${temple_html}';
    console.log('已挂载magnet')
    parent.insertAdjacentHTML("afterbegin",child)}
     </script><style>#catalog_magnet{flex-wrap: wrap;display: flex;width:100%;justify-content:space-between;padding: 10px 10px 0 10px;align-content: flex-start;}.magnet_item{flex-basis: calc(${devide}% - 5px);background: ${hexo.config.magnet.color_setting.background_color};margin-bottom: 10px;border-radius: 8px;transition: all 0.2s ease-in-out;}.magnet_item:hover{background: ${hexo.config.magnet.color_setting.background_hover_color}}.magnet_link_more{color:#555}.magnet_link{color:${hexo.config.magnet.color_setting.text_color}}.magnet_link:hover{color:${hexo.config.magnet.color_setting.text_hover_color}}@media screen and (max-width: 600px) {.magnet_item {flex-basis: 100%;}}.magnet_link_context{display:flex;padding: 10px;font-size:16px;transition: all 0.2s ease-in-out;}.magnet_link_context:hover{padding: 10px 20px;}</style>
    <style>${hexo.config.magnet.plus_style}</style>`;
        hexo.extend.injector.register('body_end',script_text, "default");
    }

},priority_magnet())
```

## 4 创建个人图标库

[Butterfly 安裝文檔(六) 進階教程 | Butterfly](https://butterfly.js.org/posts/4073eda/#%E6%B7%BB%E5%8A%A0%E9%8F%88%E6%8E%A5%E9%80%B2%E4%B8%BB%E9%A1%8C%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

## 5 评论Twikoo部署

[云函数部署 | Twikoo 文档](https://twikoo.js.org/backend.html#hugging-face-%E9%83%A8%E7%BD%B2)

[进阶配置 | 安知鱼主题官方文档 (anheyu.com)](https://docs.anheyu.com/advanced/#%E8%AF%84%E8%AE%BA)

## 6 个人css调整


### 6.1 布置ai

## 7 douban 及 douban-card的使用

