
<h1>目录</h1>

```table-of-contents
```
## 1 使用阿里云OSS

1.前提是你得有一个[阿里云](https://www.aliyun.com/)账号，没有的直接注册就行，然后选择对象存储OSS，如果找不到直接在搜索栏搜索OSS，点击折扣套餐或者立即购买。
![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280032789.png)

2.进入如下界面，资源包类型和地域不用管，存储包规格正常40GB完全够用了，购买时常自选，半年到5年不等，价格还是比较合适的，一年才9rmb，选择完成之后点击立即购买，然后付款完成。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280032020.png)
3.创建Bucket,还是进入OSS对象存储，如果找不到就搜索，进入管理控制台。名称自定义，后面要用，地域选择离自己近的，读写权限改为公共读，其他不用动，点击确定。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280033135.png)
![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280033148.png)

![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280033446.png)

4.点击Bucket列表就可以看到你刚才创建的Bucket，点击概览![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280033161.png)
点击Bucket名称，进入Bucket管理![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280033818.png)
记住图中所指示的，我的是`vicent-picture-for-typora`和`oss-cn-beijing`。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280034669.png)
点击文件管理，新建一个目录，我的是`img_for_typora/`。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280034115.png)
5.创建AccessKey

鼠标放在右上角的图像上就出来了如下图，点击AccessKey管理![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280034733.png)
先点击继续使用AccessKey，然后再点击创建AccessKey。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280034977.png)
创建完成后，复制AccessKey ID和AccessKey Secret。![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280035986.png)

## 1 使用PicGo

可以去[官方](https://github.com/Molunerfinn/PicGo/releases/tag/v2.3.0)下载，选择对应的版本，我的是windows64,选择如下图。

如果github访问不了，可以自取[安装包](https://www.aliyundrive.com/s/ViQCSDpgcHL)

![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280036091.png)

直接点击安装，成功界面如下
![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280036647.png)

点击图床设置，然后点击阿里云OSS，
![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280036764.png)



这里的设定KeyId将之前的AccessKey ID复制过来

设定KeySecret将之前的AccessKey Secret复制过来

设定存储空间名就是下图中的1，之前2.4的设置

确认存储区域就是下图中的2，之前2.4的设置

存储路径为之前新建目录名

然后点击确定，设置为默认图床![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280036290.png)

## 2 Obsidian设置

下载Image auto upload Plugin插件，安装即可
![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280037830.png)


![image.png](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Skill_Tip/other_skill/202403280037880.png)


