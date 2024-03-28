
## 1、 web相关概念回顾

1. `软件架构`
	1. C/S：客户端/服务器端
	2. B/S：浏览器/服务器端

2. `资源分类`
	1. 静态资源：所有用户访问后，得到的结果都是一样的，称为静态资源.静态资源可以直接被浏览器解析
		* 如： html,css,JavaScript
	2. 动态资源:每个用户访问相同资源后，得到的结果可能不一样。称为动态资源。动态资源被访问后，需要先转换为静态资源，在返回给浏览器
		* 如：servlet/jsp,php,asp....

3. `网络通信三要素`
	1. IP：电子设备(计算机)在网络中的唯一标识。
	2. 端口：应用程序在计算机中的唯一标识。 0~65536
	3. 传输协议：规定了数据传输的规则，其中基础的协议有
		1. tcp:安全协议，三次握手。 速度稍慢
		2. udp：不安全协议。 速度快

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/%E8%B5%84%E6%BA%90%E5%88%86%E7%B1%BB.bmp)

## 2、web服务器软件

* 服务器：安装了服务器软件的计算机
	* 如：电脑中安装了mysql服务器，通过账户密码可以访问到，就称此电脑为mysql服务器
* 服务器软件：接收用户的请求，处理请求，做出响应
* web服务器软件：接收用户的请求，处理请求，做出响应。
	* `在web服务器软件中，可以部署web项目，让用户通过浏览器来访问这些项目`
	* `web容器：动态资源不能直接运行，需要依赖于web服务器软件`

* 常见的java相关的web服务器软件：
	* `webLogic`：oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
	* `webSphere`：IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
	* `JBOSS`：JBOSS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
	* `Tomcat`：Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范servlet/jsp。开源的，免费的。

>JavaEE：Java语言在企业级开发中使用的技术规范的总和，一共规定了13项大的规范


## 3、IDEA与tomcat的相关配置

### 3.1 tomcat的安装与卸载

* Tomcat：web服务器软件
	1. `下载`：http://tomcat.apache.org/
	2. `安装`：解压压缩包即可。
		* <font color="#d83931">注意：安装目录建议不要有中文和空格</font>
	3. `卸载`：删除目录就行了
	4. `启动`：
		* bin/startup.bat ,双击运行该文件即可
		* 访问：浏览器输入：`http://localhost:8080` 回车访问自己
						  `http://别人的ip:8080` 访问别人
		
		* 可能遇到的问题：
			1. 黑窗口一闪而过：
				* 原因： 没有正确配置JAVA_HOME环境变量
				* 解决方案：正确配置JAVA_HOME环境变量

			2. 启动报错：
				1. 暴力：找到占用的端口号，并且找到对应的进程，杀死该进程
					* 命令行输入：netstat -ano
					* 找到8080进程的PID号
					* 通过任务管理器找到PID对应的进程并关闭
				1. 温柔：修改自身的端口号
					* conf/server.xml
					- `<Connector port="8888" protocol="HTTP/1.1"connectionTimeout="20000"redirectPort="8445" />`
					* <font color="#d83931">一般会将tomcat的默认端口号修改为80。80端口号是http协议的默认端口号。</font>
						* <font color="#d83931">好处：在访问时，就不用输入端口号</font>
	5. `关闭`：
		1. 正常关闭：
			* bin/shutdown.bat
			* ctrl+c
		2. 强制关闭：
			* 点击启动窗口的×
	6. `配置`:
		* 部署项目的方式：
			1. 直接将项目放到webapps目录下即可。
				* /hello：项目的访问路径-->虚拟目录
				* `简化部署：将项目打成一个war包，再将war包放置到webapps目录下，war包会自动解压缩`

			2. 配置conf/server.xml文件
				在\<Host>标签体中配置
				`<Context docBase="D:\baidu" path="/hehe" />`
				* docBase:项目存放的路径
				* path：虚拟目录

			3. 在conf\\Catalina\\localhost创建任意名称的xml文件。在文件中编写
				`<Context docBase="D:\hello" />`
				* 虚拟目录：xml文件的名称
				* `热部署的方式，常用的！！！`
		
* 静态项目和动态项目：
	* 目录结构
		* java动态项目的目录结构：
			- 项目的根目录
				- WEB-INF目录：
					- web.xml：web项目的核心配置文件
					- classes目录：放置字节码文件的目录
					- lib目录：放置依赖的jar包

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074418889.png)



* `将Tomcat集成到IDEA中，并且创建JavaEE的项目，部署项目。`

### 3.2 IDEA与tomcat的相关配置

1. IDEA会为每一个tomcat部署的项目单独建立一份配置文件
	* 查看控制台的log：Using CATALINA_BASE:   `"C:\Users\mikey\AppData\Local\JetBrains\IntelliJIdea2022.1\tomcat\4a3aaad1-ccff-4e12-a2e6-dfe7acf45ceb"`

2. `工作空间项目` 和  `tomcat部署的web项目`
	* tomcat真正访问的是“tomcat部署的web项目”，"tomcat部署的web项目"对应着"工作空间项目" 的web目录下的所有资源
	* WEB-INF目录下的资源不能被浏览器直接访问。
	* `tomcat部署的web项目` 在工作间中的out的artifact中可以找到
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074434153.png)




3. 断点调试：使用"小虫子"启动 dubug 启动


4. 若出现404，可以检查一下组件是否正确
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074443506.png)




![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1_Java%E8%AF%AD%E8%A8%80%E6%A0%B8%E5%BF%83/1_Java%E5%9F%BA%E7%A1%80/1_Java%E5%A4%8D%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230922074452738.png)
