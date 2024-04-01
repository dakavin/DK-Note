
## 1、SSH公钥错误

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d75784e5023bff35fda261fb67ed52f.png)

一般出现如上错误，就是Git远程仓库的SSH免密公钥和推送用户提供的公钥不一致导致的。

## 2、IDEA集成Gitee失败

如果IDEA集成Gitee时，向远程仓库push代码失败，且没有弹出账号窗口，可以尝试修改IDEA得相关配置

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d3fd25838ef365e94c0c150d3acd6049.png)

## 3、Git Clone过慢解决

使用git的clone下载[github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)上面的仓库速度特别慢，由第一个红框能够看出，但是从gitee上下载同样的东西速度就快很多。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ed84d93a2db2ce802c3a0ed88b3fc1b0.png)

网上给的解决办法都是将github上面的仓库转存到gitee中进行下载，输入url’以后自动给了一个已公开的仓库，直接点进去选择克隆地址再使用clone命令就能达到下面那个速度了。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ab1b1bdcdf969ccb141feb5972eede73.png)