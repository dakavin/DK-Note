
[IDEA中使用Docker插件构建镜像并推送至私服Harbor_idea中springboot项目使用dockerfile+harbor部署-CSDN博客](https://blog.csdn.net/An1090239782/article/details/111316025)
## 1 Dockefile初识

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/35c814877cca499740c0448ed80211a5.png)

Docker是一个开源的应用容器引擎，开发者可以打包自己的应用到容器里面，然后迁移到其他机器的docker应用中，可以实现快速部署。Docker利用容器（Container）来运行应用。容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。**Docker的主要概念包括镜像和容器**。

**Images（镜像）** 是一个只读模板，含创建Docker容器的说明，它与操作系统的安装光盘有点像

**Containers（容器）** 是镜像的运行实例，镜像与容器的关系类比面向对象中的类和对象

**Docker镜像通过Dockefile文件构建**，文件中包含指定的基础镜像、运行命令等

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d823cdc0415514f40b0f621fd096968c.png)

![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5ecd98de233ba15548112a60f624a398.png)

## 2 在IDEA中配置Docker

### 2.1 在Linux中配置docker remote api

在linux系统中，查询docker的service配置文件的位置
```shell
systemctl show —property=FragmentPath docker
```

宝塔界面安装的docker下配置文件都在下面这个路径，我们cat一下
```shell
cat /usr/lib/systemd/system/docker.service
```

使用vim编辑这个文件，修改如图所示
```conf
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/afe14d8dbf91c70138d775a353855560.png)

重载配置文件，并且重启docker
```shell
systemctl daemon-reload

systemctl restart docker
```

测试一下2375端口
```shell
netstat -nplt|grep 2375
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/014e8e456b9718e8e71d38fb6d329410.png)

测试一下配置有没有问题
```shell
curl localhost:2375/info
```

宝塔界面配合linux防火墙，开放一下2375端口
```shell
firewall-cmd --zone==public --add-port=2375/tcp --permanent
//如果显示这个FirewallD is not running，就开启防火墙
systemctl start firewalld.service

//如果添加失败，直接在配置文件中添加
vi /etc/firewalld/zones/public.xml

//开放3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
//防火墙重新加载配置
firewall-cmd --reload
//检查防火墙开放的端口
firewall-cmd --list-ports
```

**注意：云服务管理界面也需要开放一下docker端口！**

**docker添加国内镜像仓库**
- 添加或修改镜像仓库
```text
vim /etc/docker/daemon.json
```

- 文件内增加或修改一下内容
```text
{
  "registry-mirrors": [
    "https://ccr.ccs.tencentyun.com"
  ]
}
```

- 重启并检查配置是否生效
```shell
systemctl restart docker

docker info
```

- 成功如图所示
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/018734f34805d175590ceeed6e0bb2ab.png)
- 测试镜像源是否有效
```text
docker pull ccr.ccs.tencentyun.com/library/nginx:latest
```



### 2.2 配置idea docker插件

打开settings -> docker ，然后再docker中点击 + 号
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a0b400d89da70dc217fe443191c1e7e.png)

配置docker插件
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f4be65f2ea74ef833c9c56c7c794120b.png)

在IDEA底部service中就会出现Docker选项，然后选择这个选项，会显示容器中的内容
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e6da63d54ec40a3a64748d358671cdf1.png)
## 3 整合项目Dockerfile文件


使用maven将项目打包成jar包

创建Dockerfile文件
```java
#使用java和maven环境  
FROM openjdk:8  

# 项目的端口，内部服务端口  
EXPOSE 8080  

# 切换到容器内部的 
/appWORKDIR /app  

# 添加要运行的jar文件  
COPY target/usercenter-backend-0.0.1-SNAPSHOT.jar /app/App.jar  

# 容器启动后运行的命令  
CMD ["java","-jar","/app/App.jar","--spring.profiles.active=prod"]
```

参数设置
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cacce16bfe5883c26de798d882cf785e.png)

点击运行Dockerfile文件，发现创建镜像成功，并成功启动
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ce3d52787a4231cfa787611ff951632.png)

从外界访问容器，成功
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ee316962c78b9836fb09cbcb24a6205f.png)
