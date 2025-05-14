
参考文档如下：
- [0_vscode的安装与使用](../../../../2_JavaStudy/2_学习笔记/3_JavaWeb/3_JavaWeb（尚硅谷）/0_vscode的安装与使用.md)
- [2_VsCode使用](../../../../3_C&C++Study/2_学习笔记/0_环境搭建/2_VsCode使用.md)
- [1.5 vscode通过ssh连接](../../../0_学习笔记汇总/1_开发环境/1_开发环境搭建.md#1.5%20vscode通过ssh连接)

补充一个 **ssh免密登录的办法**
- 本文部分参考自：[https://blog.csdn.net/qq_44571245/article/details/123031276](https://blog.csdn.net/qq_44571245/article/details/123031276)

如果你觉得使用vscode每次都需要登陆很麻烦，你可以使用以下方法来解决
```shell
#打开PC的命令提示符或者power shell
#输入
ssh-keygen -t rsa

#然后连续回车直到结束

#找到.ssh这个文件夹，打开id_rsa_pub这个文件
```

如下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/da0ae3a18479e2a0020e4f6a724c84f6.png)


可以用记事本打开，打开后把里面的内容复制好。

```shell
#连接板卡
#以cat用户登陆
#在命令行上输入

ssh-keygen -t rsa

#然后连续回车直到结束
#可以看到/home/cat目录下会有.ssh文件夹
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d29a15abd99242b165aaeec9e8d09c0e.png)

然后进入该文件夹，就可以看到生成的公钥和私钥
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/5007824e528d36f784e2abc90126aad8.png)

```shell
#然后我们可以创建一个文件authorized_keys
vi authorized_keys

#把刚刚从记事本上复制的内容粘贴到里面，然后保存退出
```

以上就是免登录的操作了。

**失败参考文章**
- [VSCode SSH免密登录失败原因 原因分析及解决_vscode设置密钥登陆没有效果-CSDN博客](https://blog.csdn.net/sinat_16489689/article/details/127192214)
- [解决Linux普通用户配置SSH免密登录不生效 - 秦一觉 - 博客园](https://www.cnblogs.com/qyjzjw/p/16173016.html)