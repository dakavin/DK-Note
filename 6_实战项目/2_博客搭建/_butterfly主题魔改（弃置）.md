
## 1 全局背景

### 1.1 具体操作

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

### 1.2 引入

1. 先在博客文件中创建css、js和img的文件夹，用于存放相关资源![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9cbe90164c8fa84375dacea4f4f926fa.png)
2. 复制上面的css代码，然后在css文件夹创建一个css文件，粘贴代码
3. 在主题配置文件中，如下，引入css文件![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1a15724d49569cd650fcaa7092c64138.png)
4. **注意：** 主题配置文件中，此选项不能为空，随便选一个图片链接就可以了![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce0053ad0b416449fa26366fbb0033df.png)


## 2 顶部磁吸分类

### 2.1 前言

这个插件主要实现了以下功能： 
1. 自定义 tags 或 categories 的排列和展示 
2. 自定义 tags 或 categories 的展示图标，名称 
3. 自定义排列的行数，默认 2 行
### 2.2 安装

```shell
npm i hexo-magnet --save  
  
# 或者  
  
cnpm i hexo-magnet --save
```
### 2.3 配置

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

### 2.4 问题

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

