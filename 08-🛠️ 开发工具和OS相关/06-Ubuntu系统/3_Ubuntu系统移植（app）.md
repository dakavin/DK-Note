## 1 下载

参考链接：
- [ubuntu24.04安装systemback2.0 解决ISO不能大于4G问题_systemback ubuntu24.04-CSDN博客](https://blog.csdn.net/yaozhixing_sz/article/details/141254327)
- [systemback2.0 安装ISO 踩过的坑以及软件本身BUG-CSDN博客](https://blog.csdn.net/yaozhixing_sz/article/details/142099129)


 ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8d66aa5d7ad4bae28d3f1861f4820189.png)

## 2 使用Systemback构建ISO

安装完systemback后，就可以使用systemback来备份当前系统
- 打开Systemback软件，输入密码，点击创建Live系统 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3b2128e0f4bca969891e6c125eed555c.png)
- 设置存放sblive文件的目录以及文件名，根据需要勾选“包含用户数据文件”，最后点击“创建新的”，等待创建完成后，会在右上角的框中显示创建的文件 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/2fcf58604cd0e4f90635d5f297aa2ae4.png)
- 镜像文件大小区分
	- 小于4GB，直接插入u盘，然后点击下方转存为光盘镜像即可完成镜像制作 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/f5e93b14a839af127d5749598bbfd53d.png)
- 大于4GB可以选择其他方法，不过这个最新版本的systemback好像生成了iso文件，地址在 `/home/`目录下面，可以用来试一下 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/10d31d0687e4a5913aea8aef863f6c85.png)
## 3 制作启动U盘

制作工具如下：
- 制作启动U盘，参考：https://www.ventoy.net/cn/index.html
- 使用说明：https://www.ventoy.net/cn/doc_start.html
- Ventoy下载地址：https://github.com/ventoy/Ventoy/releases

下载之后解压，然后进入ventory的目录，打开x86_64结尾的GUI，输入密码，即可打开软件 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3d97043d6f1420aa192bb6487b1c6dba.png)
- 插入U盘，选择安装即可

**安装完之后，将iso文件复制到U盘中**
- 输入 `sudo fdisk -l` 命令查看U盘所在路径  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/83a04d5c2a860fb2ddfe79f83ec434d6.png)

- 输入以下命令，挂载U盘，并拷贝iso文件到U盘中
```shell
# 创建u盘挂载目录
sudo mkdir -p /mnt/upan
# 挂载u盘
sudo mount /dev/sdd1 /mnt/upan
# 拷贝文件
cp /home/dakkk.iso /mnt/upan/
# 检查是否拷贝成功
ls -l /mnt/upan/
# 卸载u盘
umount /mnt/upan

32,768
131,072
286,720
```

## 4 安装系统

开机按键F11（我是微星，其他的可以百度查一下）进入bios，选择u盘启动
- 注意这个时候应该只有u盘和需要移植的新盘

进入u盘启动后，界面如下，选择ISO文件然后回车进入
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d4c3f591984c1c6858908816a23d518d.png)

选择Boot in normal mode 进入，也可以选择第二个 grub2的方式
- 最后安装系统的时候提示硬盘sdd3不可用，可以尝试从第二项进入
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/3db11a2e728b497ea6d4f9278ac96ab1.png)

使用gparts格式化分区，格式分区大小和格式如下图![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/a37875f3af21b3424da9e360feb99653.png)
然后搜索打开systemback软件，点击系统安装或者系统复制（我用的复制），然后设置挂载点到三个分区
- 找到对应的新盘，然后进行挂载，注意的是，/boot/efi分区需要手写，并且取消下面的格式对钩，自己进行格式化即可
```shell
# 格式化 efi分区，根据具体的挂载写
mkfs.vfat -F 32 /dev/sda1
```

最后开始安装
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7407e72cc6d44ea0a78b617b2142da01.png)

等待完成，然后重启电脑，把系统盘拔了，即可进入备份的系统

