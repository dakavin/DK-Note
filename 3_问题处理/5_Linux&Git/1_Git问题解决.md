
## 1、SSH公钥错误

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220854.png)

一般出现如上错误，就是Git远程仓库的SSH免密公钥和推送用户提供的公钥不一致导致的。

## 2、IDEA集成Gitee失败

如果IDEA集成Gitee时，向远程仓库push代码失败，且没有弹出账号窗口，可以尝试修改IDEA得相关配置

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230923220909.png)

## 3、Git Clone过慢解决

使用git的clone下载[github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)上面的仓库速度特别慢，由第一个红框能够看出，但是从gitee上下载同样的东西速度就快很多。

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230925143902.png)

网上给的解决办法都是将github上面的仓库转存到gitee中进行下载，输入url’以后自动给了一个已公开的仓库，直接点进去选择克隆地址再使用clone命令就能达到下面那个速度了。
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Pasted%20image%2020230925143916.png)