
## 1、解释原因
spring2.X版本在2023年11月24日停止维护了，因此创建spring项目时不再有2.X版本的选项，只能从3.1.X版本开始选择

而Spring3.X版本不支持JDK8，最低支持JDK17，因此JDK8也无法主动选择了

当然，停止维护只代表我们无法用idea主动创建spring2.X版本的项目了，不代表我们无法使用，该使用依然能使用，丝毫不受影响

![](assets/Pasted%20image%2020240326010007.png)

## 2、解决方案

### 2.1、阿里云创建Spring2.X版本的项目

修改Server URL为https://start.aliyun.com  

- 原来的URL为 start.spring.io

目前阿里云还是支持创建Spring2.X版本的项目的

![](assets/Pasted%20image%2020240326010030.png)

然后就可以愉快的创建项目了

需要注意的是，通过阿里云创建的项目，初始结构与通过Spring官方创建的项目有所不同，但完全不影响使用，放心

### 2.2、阿里云官网创建Spring2.X版本的项目

打开阿里云官网 [https://start.aliyun.com](https://start.aliyun.com/ "https://start.aliyun.com")

![](assets/Pasted%20image%2020240326010049.png)

创建过程很简单，此处不再展示，记得选择依赖，创建完毕后保存本地：

- 先点击获取代码，后点击下载代码包，下载代码包即下载该项目的压缩包
- 会git操作的也可以用git命令下载该项目文件，只是操作不同罢了，结果都是得到一个Spring2.X版本的初始项目文件

后续解压缩后直接用idea打开此项目即可

![](assets/Pasted%20image%2020240326010101.png)

可以将此压缩包**保存**，每次新建项目时复制出一个新的项目文件，idea直接打开即可，压缩包可以当一个**永久的备份**，毕竟说不定哪天阿里云也创建不了spring2.X版本的项目了呢

### 2.3、下载JDK17，创建Spring3.X版本

#### 1. 修改pom.xml

![|](assets/Pasted%20image%2020240326010235.png)

修改完毕后启动一下项目看能否启动成功，若启动成功说明该修改的都修改好了，若报错，报错内容为JDK17/8不是国内源之类的问题，则继续修改，总共需要修改5个地方

#### 2. 修改IDEA设置

![](assets/Pasted%20image%2020240326010308.png)

![](assets/Pasted%20image%2020240326010314.png)

Modules这里也修改成JDK8![](assets/Pasted%20image%2020240326010323.png)
这里也要修改![](assets/Pasted%20image%2020240326010332.png)

设置这里也要进行修改![](assets/Pasted%20image%2020240326010352.png)