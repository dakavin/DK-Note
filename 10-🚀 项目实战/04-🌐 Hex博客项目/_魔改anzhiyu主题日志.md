
注意本文内容是在主题文件夹（node_modules/hexo-theme-anzhiyu/）下修改的

`source/js/anzhiyu/ai_abstract.js` 
- 修改 **所有TianliGPT字段** 为  DakkkGPT

`layout/includes/post/reward.pug`
- 修改 赞助字段 为 **你的支持是我前行的动力，感激不尽。**
```pug
.reward-main  
  .reward-all  
    //span.reward-title 感谢你赐予我前进的力量  
    span.reward-title 你的支持是我前行的动力，感激不尽。
```

`source/css/_layout/reward.styl`
- 修改 赞助字段颜色 为下
```styl
.reward-title  
  // color: var(--anzhiyu-red);  
  color: #425aef;
```

`hexo-theme-anzhiyu/scripts/events/welcome.js`
- 修改控制台
```
██████╗ ██╗  ██╗
██╔══██╗██║ ██╔╝
██║  ██║█████╔╝ 
██║  ██║██╔═██╗ 
██████╔╝██║  ██╗
╚═════╝ ╚═╝  ╚═╝
```

`layout/includes/anzhiyu/log-js.pug`
- 修改F12控制台

解决其他页面，上面有空白
- `_config.anzhiyu.yml` 文件，将default_top_img设置为false即可

取消
## 依照装备页面，创建自己的图书页面

`layout/includes/page`
- 找到equipment.pug这个文件，在当前文件夹中复制为book.pug文件
- 然后将book.pug文件中所有的equipment字样 更改为 book字样

`source/css/_page`
- 找到equipment.styl这个文件，在当前文件夹中复制为book.styl文件
- 然后将book.styl文件中所有的equipment字样 更改为 book字样

`layout/page.pug`
- 在这个文件中添加我们需要的book
```pug
block content  
  #page  
	......
      when 'book'  
        include includes/page/book.pug  
	......
```

最后就成功了


即刻说说

|            参数             | 备选值/类型 |                   释义                   |
| :-----------------------: | :----: | :------------------------------------: |
|        class_name         | String |            【可选】标识符，无实际意义，选填            |
|        essay_list         | Array  |              【必选】即刻短文数据列表              |
|    essay_list.content     | String |              【必选】短文 文字内容               |
|      essay_list.date      |  Time  | 【必选】短文发布时间 格式建议为 `2022/10/26 08:00:00` |
|     essay_list.image      | Array  |          【可选】短文图片内容, 可填写多张图片           |
|      essay_list.from      | String |      【可选】短文 来自何处, 当然也可以填任何你想填写的标识      |
|      essay_list.link      | String |                【可选】外部链接                |
|    essay_list.aplayer     | Array  |   【可选】aplayer 播放器的单曲音乐, 需 aplayer 支持   |
| essay_list.aplayer.server | String |  【essay_list.aplayer 后必选】aplayer 服务商   |
|   essay_list.aplayer.id   | String |     【essay_list.aplayer 后必选】单曲 id      |

【主题版本1.6.7以上支持】其中video对于bilibili进行的额外的支持，视频为player.bilibili.com的将显示为bilibili视频，并且可以在链接后面拼接`&autoplay=0`控制不自动播放

