
<h1>目录</h1>

```table-of-contents
```
## 1 使用阿里云OSS

1.前提是你得有一个[阿里云](https://www.aliyun.com/)账号，没有的直接注册就行，然后选择对象存储OSS，如果找不到直接在搜索栏搜索OSS，点击折扣套餐或者立即购买。
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1999adb69e2a929cd34de584c02e73a0.png)

2.进入如下界面，资源包类型和地域不用管，存储包规格正常40GB完全够用了，购买时常自选，半年到5年不等，价格还是比较合适的，一年才9rmb，选择完成之后点击立即购买，然后付款完成。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1d0401ddd572f762ec5afc4e2bb43fd.png)
3.创建Bucket,还是进入OSS对象存储，如果找不到就搜索，进入管理控制台。名称自定义，后面要用，地域选择离自己近的，读写权限改为公共读，其他不用动，点击确定。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36e56f50d0d2fe1807d02d42845ad6b3.png)
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/418a78d1c49e429eb458c33230e3036a.png)

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bfbe175d2fab70de7e12fab4e5119f6c.png)

4.点击Bucket列表就可以看到你刚才创建的Bucket，点击概览![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/48b1d4317fde6872a50086d9743c207b.png)
点击Bucket名称，进入Bucket管理![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/00316c6b93cba3a08b16235494c16705.png)
记住图中所指示的，我的是`vicent-picture-for-typora`和`oss-cn-beijing`。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4b1f2fcf3dde661e908e10241b7f820d.png)
点击文件管理，新建一个目录，我的是`img_for_typora/`。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/df065e9aa763f8b0c345810e5fe9320a.png)
5.创建AccessKey

鼠标放在右上角的图像上就出来了如下图，点击AccessKey管理![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6907ef67d1a6d8bdb968f09c217026ef.png)
先点击继续使用AccessKey，然后再点击创建AccessKey。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3dc7b1e260a55ea76c8ce0c8976b68ef.png)
创建完成后，复制AccessKey ID和AccessKey Secret。![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/baf063084883fd850debf59806124f1a.png)

## 1 使用PicGo

可以去[官方](https://github.com/Molunerfinn/PicGo/releases/tag/v2.3.0)下载，选择对应的版本，我的是windows64,选择如下图。

如果github访问不了，可以自取[安装包](https://www.aliyundrive.com/s/ViQCSDpgcHL)

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/26fbd7ce542ac527d018ad2fd34c6b59.png)

直接点击安装，成功界面如下
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/de1410e566839cba317148d5f4eaafa9.png)

点击图床设置，然后点击阿里云OSS，
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14f7fcbd78f8d3f229e0e14d9065088c.png)



这里的设定KeyId将之前的AccessKey ID复制过来

设定KeySecret将之前的AccessKey Secret复制过来

设定存储空间名就是下图中的1，之前2.4的设置

确认存储区域就是下图中的2，之前2.4的设置

存储路径为之前新建目录名

然后点击确定，设置为默认图床![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d4dda5975e1114bc24c70a2a479e14cd.png)

## 2 Obsidian设置

下载Image auto upload Plugin插件，安装即可
![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d7fbc0c49aa5c4d7867adf6d2a4ad524.png)


![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9222dc10fe01b6d9950299fceb26ed1.png)


