
关于部署前后端项目的docker镜像到服务器中，建议参考文章：[1_SpringBoot项目部署（Docker）](../../../4_实用文件/8_docker&nginx/1_SpringBoot项目部署（Docker）.md)

使用IDEA部署docker镜像到服务器上后，就不需要我们做如下工作
- 使用git拉取代码
- 然后使用docker build构建镜像
- 然后使用docker run命令来启动镜像（容器）

## 1 Docker平台部署

我们之前的docker的部署方式，有以下两种
- 将前端项目和后端项目通过git的方式，拉取到服务器中，然后在服务器中使用docker来构建镜像和运行镜像
- 前端项目和后端项目，依靠本地IDE工具通过tcp连接服务器的docker，然后再本地对项目进行构建镜像并推送到服务器中

上述的过程都是**在自己的服务器中**进行的，我们来讲讲如何**使用别人的服务器（Docker平台）**，来部署项目

---

**有哪些容器平台，这里列举一些**
- 云服务商的容器平台（腾讯云、阿里云）
- 面向某个领域的容器平台（前端webify 、前端/后端微信云托管） 要花钱！
	- webify（web应用托管）：比容器更傻瓜式，不需要自己写构建应用的命令，就能启动前端项目；缺点就是需要将代码放到代码托管平台上
	- 微信云托管：傻瓜式操作，自己看一看怎么操作就好了！

---

**容器平台的好处**
- 不需要输入命令来操作，更方便省事
- 不用在控制台操作，更傻瓜式、更简单
- 大厂运维，比如监控、告警、其他（存储、负载均衡、自动扩缩容、流水线）


## 2 域名

### 2.1 域名解析流程➡

**前端项目访问：**

用户输入网址 ➡ 域名解析服务器（把网址解析为ip地址/其他的域名解析服务）➡ 服务器 ➡（防火墙） ➡ nginx接收请求，找到对应的文件，返回文件给前端 ➡ 前端加载文件（html、js、css等）到浏览器中 ➡浏览器渲染页面

**后端项目访问：**

用户输入网址 ➡ 域名解析服务器 ➡ 服务器 ➡ nginx接收请求 ➡ 后端项目（比如8080端口）
### 2.2 绑定域名

我们这里不使用docker的方式运行项目，并且绑定域名，回到最初的状态，上传文件，让本地nginx和tomcat来运行我们的项目

先把docker运行的两个前后端项目停止， 然后打开nginx

到所在的域名购买商哪里先设置域名解析，给域名添加解析记录
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fe278b556ca771d62aee8342b0fc3338.png)

记录类型的选择，一般默认选择A就好了
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71e1c6a0112109d965d9638f3ec262ed.png)

---

**前端绑定域名**

我们回到宝塔前端项目，进行设置一下域名，将之前ip地址访问的，添加上域名
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/033db85582e2fafcca5d101f66da7d1a.png)

添加上这个域名后，宝塔也会在配置文件中，给我们加上这个域名
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2a0e6395ca77a2fc29db8c27ad6447a0.png)

然后我们把之前ip访问，禁用或者删除掉
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f0b6cdfba76d968d9ac4a0937cfa6611.png)


再次访问 user.yupi666.com 就可以访问到了，访问公网ip地址就会返回404（多访问几次）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cdccaa3c480bf7d06e7d56fc45d72870.png)

不过第一次访问官网ip地址还是会访问到登录页面的
- 因为在默认情况下，当访问到nginx这个端口时，如果没有匹配上的域名，就会试着匹配这个主页（之前默认的文件），从而就匹配到默认的index.html
- 所以访问公网ip地址时，会重定向到前端页面，

---

**后端绑定域名**

继续去域名解析的地方，添加一个后端域名的记录，然后点击保存
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c310a0ebafe52a43d7855c9d53f9b906.png)

==记住在宝塔界面设置后端文件的域名==

最后访问一下 `user-backend.yupi666.com:8080` 和 `user-backend.yupi666.com:8080/api/user/search`
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75251f41ace014958572e025101b643a.png)

我们发现这样很丑，难道每次用户访问都需要加一个端口吗？

虽然用户不可能直接访问后端，但是对于开发者来说，加个端口号也是不舒服的，那能不能去掉这个端口号呢？
- 如果直接去掉端口号，就默认访问80端口，80端口的请求转发到nginx
- 可以让nginx接收到请求，分析这个请求，然后把请求转发到后端项目去
- 即将该没有端口的域名，转发到8080端口去

--- 

**配置nginx转发我们的后端请求**

在PHP项目中，继续按照我们这个域名，创建一个站点
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/910b84b41670a65d377cbd104b1eb4b7.png)

进入这个站点的设置，点击反向代理
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eddf9140b2bf15aa0004c19b504eb82e.png)

写一下代理名称 和 目标URL，然后点击提交
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a05986bbfc1c0e3f9610598d42f50d3a.png)

试一下不带8080访问后端，成功了
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f8c38a8fa3e6a8de9cba6e9361b96b50.png)

**补充：如果没有域名，使用的是IP地址，就不需要创建站点，直接在原来的站点添加反向代理即可**
- 好像就把主页也给反向了，没有域名就不要设置这个啦！！！不然主页都访问不了！

### 2.3 跨域问题


**什么是跨域？**

浏览器为了用户的安全，仅允许向==同协议、同域名、同端口==的服务器发送请求。

例如我们前端的域名是user，后端的域名是user-backend，域名不一样就出现了跨域，跨域就会报错

---

**我们看一下网络请求**

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b631db80cd831f0b7841acde5f3b6d92.png)

为了防止跨域，或者说检测跨域，浏览器会在它正式发送请求之前，发送一个预检请求，预检的请求方式的OPTIONS

预检请求经常用来检查是否跨域，即当我们请求的域名和当前网页的域名不同的时候，他就会发送预检请求

总结：预检就是提前探路

****

**如何解决跨域？**

有三种方式：
- 把域名、端口改成相同的
- 网关支持（nginx）
- 修改后端服务

---
**把域名、端口改成相同的**

由于我们的前端框架是ant design pro的，所以关于跨域问题我们去看官方文档[同源策略 跨域与代理 - Ant Design Pro](https://pro.ant.design/zh-CN/blog/proxy/)

我们在前端文件中，找到这两个文件，proxy.ts 和 globalRequest.ts

先查看globalRequest.ts这个文件，发现它指定了我们生产环境中，前端如何调用后端接口的
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/adb7a6fb713c4dfe5d67841b677ae989.png)

我们继续看proxy.ts文件，是不是发现我们在开发环境dev下，配置过跨域的问题
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2e2bf2725866636493c942ba61c5ef71.png)

那么我们照抄，并参考官方文档，添加如下代码
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d8f588598cb880cc133ceb7b490f7503.png)

打包dist，重新发布，测试一下，注册能否跨域

==研究半天，还是存在跨域问题，放弃！==

---

**网关支持**

让服务器告诉浏览器：允许跨域（返回cross-origin-allow响应头）

这个时候我们就上升一个层面了，因为我们的前后端服务，都是用nginx网关的，所以我们可以修改nginx配置，来实现跨域


**需要将下面这个配置，放在后端项目的站点中！！**
- 不是前端项目嗷！

```js
location ^~ /api {  
	proxy_pass http://127.0.0.1:8080/api/;  
	add_header 'Access-Control-Allow-Origin' $http_origin;  
	add_header 'Access-Control-Allow-Credentials' $http_origin;  
	add_header Access-Control-Allow-Methods 'GET,POST,OPTIONS';  
	add_header Access-Control-Allow-Headers '*';  
	// 注意if后面有一个空格
	// 注意 = 前后也要有空格
	if ($request_method = 'OPTIONS'){  
		add_header 'Access-Control-Allow-Credentials' 'true';  
		add_header 'Access-Control-Allow-Origin' $http_origin;  
		add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';  
		add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';  
		add_header 'Access-Control-Max-age' 172800;  
		add_header 'Content-Type' 'text/plain; charset=utf-8';  
		add_header 'COntent-Length' 0;  
		return 204;  
	}  
}
```


![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/75135a7f123c9d9a4449bd29801acb37.png)

然后去访问站点，就可以成功打通前后端了！！

---

**修改后端服务**

https://www.jianshu.com/p/b02099a435bd

第一种方式：
- 在UserController中添加@CrossOrigin注解
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/89ee65f42435b18f61b3be5fbb0a44c2.png)
第二种方式：（推荐）
- 在config目录下，添加web全局请求拦截器
```java
@Configuration  
public class WebMvcConfig implements WebMvcConfigurer {   
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        //设置运行跨域的路径  
        registry.addMapping("/**")  
                //设置允许跨域请求的域名  
                //当 Credentials为true时，Origin不能为*号，需要为具体的ip地址 【如果接口不带Cookies,ip无需设置成具体的ip】  
                .allowedOrigins("http://localhost:9527","http://127.0.0.1:9527","http://127.0.0.1:8082","http://127.0.0.1:8083")  
                // 是否允许证书 不在默认开启  
                .allowCredentials(true)  
                // 设置允许的方法  
                .allowedMethods("*")  
                //跨域允许时间  
                .maxAge(3600);  
    }  
}
```

第三种方式：定义性的corsFilter Bean，参考上述文件


**修改上述方式后，都要重启一下后端的服务**


### 项目的优化点

1. 功能扩展
	- 管理员创建用户、修改用户信息、删除用户
	- 上传头像
	- 按照更多的条件去查询用户
	- 更改权限

2. 修改Bug

3. 项目登录改为分布式session，甚至有可能改为redis单点登录

4. 通用性
	- set-cookie domain域名更通用，比如改为 `*.xxx.com`这个通用的域名
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e8a4d05526794b0cd45acba79176faaa.png)
	  - 把用户管理系统 =》 用户中心（之后所有的服务都请求这个后端）

5. 后台添加全局请求拦截器（统一去判断用户权限，统一记录请求日志）













