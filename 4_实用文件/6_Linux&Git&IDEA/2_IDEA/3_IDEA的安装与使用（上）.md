
推荐参考网站： 
[IDEA Tab 页设置多行显示（图文讲解） - 犬小哈教程 (quanxiaoha.com)](https://www.quanxiaoha.com/idea/idea-enable-tabs-in-one-row.html)
## 1 卸载与安装

### 1.1 卸载过程

这里以卸载2022.1.2版本为例说明。在【控制面板】找到【卸载程序】

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5eca0f35067284dd4769e6f1d4424450.png)

右键点击或左键双击IntelliJ IDEA 2022.1.2进行卸载：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ebdb477e3af2857dfb8e4e52ae76cc9.png)


如果需要保留下述数据，就不要打√。如果想彻底删除IDEA所有数据，那就打上√。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b80850303d4d38c0f9915a36e2f14148.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/039d1336fe0861f268e9a737871ffa5a.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dd8320a824339742768210343e76f49e.png)



软件卸载完以后，还需要删除其它几个位置的残留：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d48f3c4893f50b0f8560fa428d0e8303.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/993b76aca1ad24862d2f23c1c1048f52.png)

### 1.2 安装前的准备

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d725cdf4d99e99c0c8bc47fa88ab141a.png)

* 64 位 Microsoft Windows 11、10、8
* 最低 2 GB 可用 RAM，推荐 8 GB 系统总 RAM
* 2.5 GB 硬盘空间，推荐 SSD
* 最低屏幕分辨率 1024x768

从安装上来看，IntelliJ IDEA 对硬件的要求`似乎不是很高`。可是在实际开发中并不是这样的，因为 IntelliJ IDEA 执行时会有大量的缓存、索引文件，所以如果你正在使用 Eclipse / MyEclipse，想通过 IntelliJ IDEA 来解决计算机的卡、慢等问题，这基本上是不可能的，本质上你应该对自己的硬件设备进行升级。

### 1.3 安装过程

1、下载完安装包，双击直接安装

2、欢迎安装
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6808daf7d16e16a165c719a654ef4a69.png)


3、是否删除电脑上低版本的IDEA（如果有，可以选择忽略）

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e1fa359e08dfa8d79221c73ad3ac54b.png)

- 如果电脑上有低版本的IDEA，可以选择删除或保留。

- 这里没有卸载旧版本，如果需要卸载，记得勾选下面的保留旧的设置和配置。

4、选择安装目录

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/423e37abddb34cdc421dd6f78bfedda3.png)



选择安装目录，目录中要避免中文和空格。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/978f359b649a53fc481cbaa34ad99480.png)



5、创建桌面快捷图标等

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3be5a7b88037efce94f94f53e89b1a39.png)

确认是否与.java、.groovy、.kt 格式文件进行关联。这里建议不关联。

6、在【开始】菜单新建一个文件夹（这里需要确认文件夹的名称），来管理IDEA的相关内容。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8d30f27ca1acbfda2e32ae7a7a392bd2.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbca6e003d406039a273174ed43e5860.png)

7、完成安装

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/72b9d3519a7b1356c115dd8cc6dd939e.png)

重启以后，单击登录

### 1.4 注册

首先，需要通过用户协议：
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/daf1acdea965c43f47d16c14c5c53491.png)


是否同意发送用户数据（特性、使用的插件、硬件与软件配置等），建议选择：不发送。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b501117de9a108126e087d8bc5aaa4c3.png)


接着，会提示我们进行注册。

- 选择1：试用30天。在IDEA2022.1.2版本中，需要先登录，才能开启试用。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e08923e374b3f19b40885baa46015116.png)

- 选择2：付费购买旗舰版

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a04c7fc1776ca75d917542a32f2adef.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aa4907447c133c02f383309adda803f1.png)


- 选择3：（推荐）
  - 可以自行搜索注册方式即可。

### 1.5 闪退问题

问题描述：2022.1启动不了，双击桌面图标，没有响应。

解决办法：

打开`C:\Users\songhk\AppData\Roaming\JetBrains\IntelliJIdea2022.1\idea64.exe.vmoptions` 这个文件。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4e43abe1f504f1cbf387c93999e4e58f.png)



内容如下所示：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c93abc5587106447933e949200c34b73.png)



删除红框的数据以后，再登录即可正常进入。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9414dc91f748e6ff688bb77553674b13.png)


原因：之前使用过的比如2021.2.2版本，pojie了。新版IEDA太智能了，把现有的启运参数也都复制过去了。又因为最新的IDEA，不兼容pojie程序-javaagent:D:\develop_tools\IDEA\IntelliJ IDEA 2021.2.2\bin\jetbrains-agent.jar了，所以报错了，所以JVM结束了，所以没有启动画面，凉凉了。

## 2 HelloWorld的实现

### 2.1 新建Project - Class

选择"New Project"：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f593f7350d60cb990ecac12ea8ab575.png)



指名工程名、使用的JDK版本等信息。如下所示：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0434004e45952531a5139bc56e7107ca.png)



接着创建Java类：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/091afd114e7373830f67ff614b672e7c.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d78ce8d618c5cebe754e01ed8e1768d.png)

### 2.2 编写代码

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello,World!");
    }
}
```

### 2.3 运行

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/236787b5d6c2f7687d1b020813766045.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a1c62f07f641babc3d171837d2b9b5dc.png)
## 3 JDK相关设置

### 3.1 项目的JDK设置

`File-->Project Structure...-->Platform Settings -->SDKs`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d1c87fb18cc0264f514a417a1b2d0a5.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b0ab894473b1cb793ffd0fb39580e34d.png)

- 注1：SDKs全称是Software Development Kit ，这里一定是选择JDK的安装根目录，不是JRE的目录。
- 注2：这里还可以从本地添加多个JDK。使用“+”即可实现。

### 3.2 out目录和编译版本

`File-->Project Structure...-->Project Settings -->Project`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ea9f019371ec24796590b54a5bd5324.png)


## 4 详细设置

### 4.1 如何打开详细配置界面

1、显示工具栏

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d59d9dea1da08263a312a224dd272206.png)



2、选择详细配置菜单或按钮

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/21011e89eeb4cb9e9258ee2751fa650a.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e180670418425676902d92e1acf2530a.png)

### 4.2 系统设置

#### 4.2.1 1、默认启动项目配置

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1a7dc4dea452286256b7b7964bb8474.png)


启动IDEA时，默认自动打开上次开发的项目？还是自己选择？

如果去掉Reopen projects on startup前面的对勾，每次启动IDEA就会出现如下界面：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/94d1ffeb72337355d96982b7c4e9005f.png)

#### 4.2.2 2、取消自动更新

Settings-->Appearance & Behavior->System Settings -> Updates

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/97afef2a9c399114782a4303a35a1d88.png)


默认都打√了，建议检查IDE更新的√去掉，检查插件更新的√选上。

### 4.3 设置整体主题

#### 4.3.1 1、选择主题

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e70de0639aec97357b95d54b66f57524.png)

#### 4.3.2 2、设置菜单和窗口字体和大小

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/45267433f8a1fbb616c22cdedf1333e2.png)

#### 4.3.3 3、设置IDEA背景图

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cc76ad1256b4f2d115b961e3ce3f0873.png)

选择一张合适的图片作为背景，即可。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1a0dede45bebe9fa4fee81ab5057d200.png)

### 4.4 设置编辑器主题样式

#### 4.4.1 1、编辑器主题

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/38bb65084c68b38b66e2306805af2b8d.png)

#### 4.4.2 2、字体大小

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2cdd3dc0899d3d53a83864ed5569776.png)



更详细的字体与颜色如下：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d2bb0c8a55fbff5650e9fd1cb75db539.png)

> 温馨提示：如果选择某个font字体，中文乱码，可以在fallback font（备选字体）中选择一个支持中文的字体。
>

#### 4.4.3 3、注释的字体颜色

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e09ade8e58bbb92ae5a9eb4118d0beb6.png)

- Block comment：修改多行注释的字体颜色
- Doc Comment –> Text：修改文档注释的字体颜色
- Line comment：修改单行注释的字体颜色

### 4.5 显示行号与方法分隔符

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33b2a10dfe6df9ffec3e92b4b21618fc.png)



### 4.6 代码智能提示功能

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dd49f38e96c0d320bd555de56055da94.png)



IntelliJ IDEA 的代码提示和补充功能有一个特性：`区分大小写`。 如果想不区分大小写的话，就把这个对勾去掉。`建议去掉勾选`。
### 4.7 自动导包配置

* 默认需要自己手动导包，Alt+Enter快捷键

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/611a063a3c663340eef5a09052c50145.png)



* 自动导包设置
  * 动态导入明确的包：Add unambiguous imports on the fly，该设置具有全局性；
  * 优化动态导入的包：Optimize imports on the fly，该设置只对当前项目有效；

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a232f4ad6fd64dfa2f100307b0a77349.png)



### 4.8 设置项目文件编码（一定要改）

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f8a857f053e0944e8adee0d5bbc777f9.png)



说明： Transparent native-to-ascii conversion主要用于转换ascii，显式原生内容。一般都要勾选。

### 4.9 设置控制台的字符编码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7ae22c62254ba763ad29c2a0f6cf9b37.png)



### 4.10 修改类头的文档注释信息

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce38aa4695041f5a674ed69e075c42e6.png)



比如：

```java
/**
* ClassName: ${NAME}
* Package: ${PACKAGE_NAME}
* Description: 
* @Author 尚硅谷-宋红康
* @Create ${DATE} ${TIME} 
* @Version 1.0   
*/
```

常用的预设的变量，这里直接贴出官网给的：

```java
${PACKAGE_NAME} - the name of the target package where the new class or interface will be created. 
${PROJECT_NAME} - the name of the current project. 
${FILE_NAME} - the name of the PHP file that will be created. 
${NAME} - the name of the new file which you specify in the New File dialog box during the file creation. 
${USER} - the login name of the current user. 
${DATE} - the current system date. 
${TIME} - the current system time. 
${YEAR} - the current year. 
${MONTH} - the current month. 
${DAY} - the current day of the month. 
${HOUR} - the current hour. 
${MINUTE} - the current minute. 
${PRODUCT_NAME} - the name of the IDE in which the file will be created. 
${MONTH_NAME_SHORT} - the first 3 letters of the month name. Example: Jan, Feb, etc. 
${MONTH_NAME_FULL} - full name of a month. Example: January, February, etc.

```

### 4.11 设置自动编译

`Settings-->Build,Execution,Deployment-->Compiler`

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f78a2777e988f01000bfe56e996a3e16.png)



### 4.12 设置为省电模式 (可忽略)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/27ffb619c28ac6e38905c2bc0fbb3868.png)



IntelliJ IDEA 有一种叫做`省电模式`的状态，开启这种模式之后 IntelliJ IDEA 会`关掉代码检查`和`代码提示`等功能。所以一般也可认为这是一种`阅读模式`，如果你在开发过程中遇到突然代码文件不能进行检查和提示，可以来看看这里是否有开启该功能。

### 4.13 取消双击shift搜索

因为我们按shift切换中英文输入方式，经常被按到，总是弹出搜索框，太麻烦了。可以取消它。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cb689fe6ac11dde48df161cb2cfb6450.png)



- 方式1：适用于IDEA 2022.1.2版本

在2022.1版本中，采用如下方式消双击shift出现搜索框：搜索double即可，勾选Disable double modifier key shortcuts，禁用这个选项。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ea0bd850da8373c2ca5e98e19cf1c02.png)



- 方式2：适用于IDEA 2022.1.2之前版本

双击shift 或 ctrl + shift + a，打开如下搜索窗口：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6627823120d73fcf5d94e50438cf6203.png)



选择registry...，找到"ide.suppress.double.click.handler"，把复选框打上勾就可以取消双击shift出现搜索框了。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/45b351c2e83cbcd8e805729359a84804.png)



## 5 工程与模块管理

### 5.1 IDEA项目结构

**层级关系：**

```
project(工程) - module(模块) - package(包) - class(类)
```

**具体的：**

```
一个project中可以创建多个module

一个module中可以创建多个package

一个package中可以创建多个class
```

> 这些结构的划分，是为了方便管理功能代码。

### 5.2 Project和Module的概念

在 IntelliJ IDEA 中，提出了Project和Module这两个概念。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f98af8f7cb2a184e5bfd0987f1c3fc6.png)



在 IntelliJ IDEA 中Project是`最顶级的结构单元`，然后就是Module。目前，主流的大型项目结构基本都是多Module的结构，这类项目一般是`按功能划分`的，比如：user-core-module、user-facade-module和user-hessian-module等等，模块之间彼此可以`相互依赖`，有着不可分割的业务关系。因此，对于一个Project来说：

- 当为单Module项目的时候，这个单独的Module实际上就是一个Project。
- 当为多Module项目的时候，多个模块处于同一个Project之中，此时彼此之间具有`互相依赖`的关联关系。
- 当然多个模块没有建立依赖关系的话，也可以作为单独一个“小项目”运行。

### 5.3 Module和Package

在一个module下，可以声明多个包（package），一般命名规范如下：

```
1.不要有中文
2.不要以数字开头
3.给包取名时一般都是公司域名倒着写,而且都是小写
  比如：尚硅谷网址是www.atguigu.com
  那么我们的package包名应该写成：com.atguigu.子名字。
```

### 5.4 创建Module

建议创建“Empty空工程”，然后创建多模块，每一个模块可以独立运行，相当于一个小项目。JavaSE阶段不涉及到模块之间的依赖。后期再学习模块之间的依赖。

步骤：

（1）选择创建模块

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8c5a7c060eeffa37f3b6fcd365873ea8.png)



（2）选择模块类型：这里选择创建Java模块，给模块命名，确定存放位置

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/495013acecf8a3a2bd973294e39445be.png)



（4）模块声明在工程下面

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29a69df7ceaad04417043fce75b73c9a.png)



### 5.5 删除模块

（1）移除模块

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/26dbe9690ea45bbbede1b42ab8574af2.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ec1ff8f4296a5d4d3c1f7f2cf684a405.png)


（2）彻底删除模块

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a02b6e5bcf11ad00ee2e311d5a43dbf0.png)


### 5.6 导入老师的模块

（1）将老师的模块`teacher_chapter04`整个的复制到自己IDEA项目的路径下

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f632f711b955966eede535e03e09d87a.png)


接着打开自己IDEA的项目，会在项目目录下看到拷贝过来的module，只不过不是以模块的方式呈现。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/375c2f79f9f6f18fea4964727658c918.png)


（2）查看Project Structure，选择import module

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8ccf700db27007214e6d3c7114d1ebc9.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/88520c8c8f773e85e48abc6e03effe37.png)

（3）选择要导入的module：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a5b8e71b19ed643d140abe873ed7ee12.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/003447bc3e144456dfd561e53f460aff.png)

（4）接着可以一路Next下去，最后选择Overwrite

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13da057c1d7eb6c5ff453e8f218ae9d9.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/580ea3f61c442c9bb277828f5d27eb83.png)


最后点击OK即可了。

### 5.7 同时打开两个IDEA项目工程

#### 5.7.1 1、两个IDEA项目工程效果

有些同学想要把上课练习代码和作业代码分开两个IDEA项目工程。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c04e77e88cc06d482f0719c0ec353dad.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/64c6cd8449b7fa0b73996f05ed922cdf.png)

#### 5.7.2 2、新建一个IDEA项目

注意：第一次需要新建，之后直接打开项目工程即可

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cce05a7868570687850beb979589d9f5.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b97261a0eff9c8f73170884c2a6480f.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5082319f867c1d2a5ed483fb547e6a5b.png)

#### 5.7.3 3、打开两个IDEA项目

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e665cc931e70ec7e1d92f8edcc9305b0.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fce8fa0b068aedb67a4a458cdc785c6d.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7d6e2f2859aac3559b67a0ce7823447.png)

### 5.8 导入前几章非IDEA工程代码

**1、创建chapter01、chapter02、chapter03等章节的module**

将相应章节的源文件粘贴到module的src下。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7060264a23f710670c404830bc651fa2.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4cc7ace39951e916bc82739a52ec5005.png)

打开其中各个源文件，会发现有乱码。比如：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6562dab630daacc4c869c87bcabb3e3a.png)


**2、设置编码**

当前项目是UTF-8。如果原来的.java文件都是GBK的（如果原来.java文件有的是GBK，有的是UTF-8就比较麻烦了）。

可以单独把这两个模块设置为GBK编码的。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d084d27510b8fec21d210cdab086db7e.png)


改为GBK，确认即可。如图：

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fd19e3f0f2d8e70bb12c618d53f2b94d.png)


## 6 代码模板的使用

### 6.1 查看Postfix Completion模板(后缀补全)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/383b6479afda0c11a7ea3c8105ff3735.png)


### 6.2 查看Live Templates模板(实时模板)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d496066ea22bedbe1578f5518a163eed.png)

### 6.3 常用代码模板

#### 6.3.1 1、非空判断

* 变量.null：if(变量 == null)
* 变量.nn：if(变量 != null) 
* 变量.notnull：if(变量 != null) 
* ifn：if(xx  == null)
* inn：if(xx  != null)

#### 6.3.2 2、遍历数组和集合

* 数组或集合变量.fori：for循环
* 数组或集合变量.for：增强for循环
* 数组或集合变量.forr：反向for循环
* 数组或集合变量.iter：增强for循环遍历数组或集合

#### 6.3.3 3、输出语句

- sout：相当于System.out.println
- soutm：打印当前方法的名称
- soutp：打印当前方法的形参及形参对应的实参值
- soutv：打印方法中声明的最近的变量的值
- 变量.sout：打印当前变量值
- 变量.soutv：打印当前变量名及变量值

#### 6.3.4 4、对象操作

- 创建对象
  - Xxx.new  .var ：创建Xxx类的对象，并赋给相应的变量
  - Xxx.new  .field：会将方法内刚创建的Xxx对象抽取为一个属性
- 强转
  - 对象.cast：将对象进行强转
  - 对象.castvar：将对象强转后，并赋给一个变量

#### 6.3.5 5、静态常量声明

* psf：public static final
* psfi：public static final int
* psfs：public static final String
* prsf：private static final

### 6.4 自定义代码模板

#### 6.4.1 自定义后缀补全模板

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9c2c5e7ecdaef872ad89d8ae548829eb.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2597aafe387b397f09f7c293fc9af899.png)
#### 6.4.2 自定义Live Templates

例如：定义sop代表System.out.print();语句

①在Live Templates中增加模板

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71b088cdf25ca75a59875a354c258965.png)


②先定义一个模板的组，这样方便管理所有自定义的代码模板
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cfc1d9935ab76ea8f8a937b488de3adc.png)


③在模板组里新建模板
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c5c1b6a94e835afd9035a303cc990d43.png)


④定义模板（以输出语句为例）
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b3855b3aa11161e18d67a5d59960dfde.png)


- Abbreviation：模板的缩略名称
- Description：模板的描述
- Template text：模板的代码片段
- 模板应用范围。比如点击Define。选择如下：应用在java代码中。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c92d252819cc09941ef253e3e0129b6.png)


**其它模板1：单元测试模板：**

```java
@Test
public void test$var1$(){
    $var2$
}
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c0ababac0567dac8ad4f3da2ff2f7a5.png)


**其它模板2：创建多线程**

```java
new Thread(){
    public void run(){
        $var$
    }
};
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a0b71a8a779dc592cd966e5739e8e0f1.png)


**其它模板3：冒泡排序**

```java
for(int $INDEX$ = 1; $INDEX$ < $ARRAY$.length; $INDEX$++) {
    for(int $INDEX2$ = 0; $INDEX2$ < $ARRAY$.length-$INDEX$; $INDEX2$++) {
        if($ARRAY$[$INDEX2$] > $ARRAY$[$INDEX2$+1]){
            $ELEMENT_TYPE$ temp = $ARRAY$[$INDEX2$];
            $ARRAY$[$INDEX2$] = $ARRAY$[$INDEX2$+1];
            $ARRAY$[$INDEX2$+1] = temp;
        }
    }
}
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/074eb3e550bebb0499a80dc7d4194a31.png)


