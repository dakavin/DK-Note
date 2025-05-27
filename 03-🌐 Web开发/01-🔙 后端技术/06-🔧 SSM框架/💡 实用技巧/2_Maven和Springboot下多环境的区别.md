---
文章标题: "[[2_Maven和Springboot下多环境的区别]]" 
文章作者: Dakkk
文章概要: |
  文章对比了Maven和SpringBoot多环境配置的异同，介绍了两者在配置位置（pom.xml vs application.yml）和配置内容（数据库连接 vs 服务端口）上的差异。
tags:
- "Maven"
- "SpringBoot"
- "多环境配置"
- "profiles"
- "配置管理"
- "开发环境"
- "测试环境"
- "生产环境"
相关文章:
- "[[0_主要内容和学习目标]]"
- "[[1_搭建多模块工程（Spring Initializr）]]"
- "[[1_C语言入门]]"
- "[[1_Java语言概述]]"
- "[[1_mapstruct 和 lombok 的冲突问题]]"
文章分类: "🌐 Web开发"
文章路径: "03-🌐 Web开发/01-🔙 后端技术/06-🔧 SSM框架/💡 实用技巧/2_Maven和Springboot下多环境的区别.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 23:34:19
---

## 1 共同点

为项目构建开发、测试和发布三种情境下不同的属性值，比如mysql的配置信息url、username、password等。
## 2 不同点

### 2.1 书写位置

maven配置多环境在项目的pom.xml中，常见用在maven多模块继承中，在父模块的pom.xml中通过配置如下代码
```xml
<!--配置多环境-->
<profiles>
	<!--开发环境-->
	<profile>
			<id>env_dep</id>
			<properties>
					<jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>
			</properties>

			<!--设定是否为默认启动环境-->
			<activation>
					<activeByDefault>true</activeByDefault>
			</activation>
	</profile>
	<!--生产环境-->
	<profile>
			<id>env_pro</id>
			<properties>
				<jdbc.url>jdbc:mysql://127.2.2.2:3306/ssm_db</jdbc.url>
			</properties>

	</profile>
	<!--测试环境-->
	<profile>
			<id>env_test</id>
			<properties>
				<jdbc.url>jdbc:mysql://127.3.3.3:3306/ssm_db</jdbc.url>
			</properties>

	</profile>
</profiles>

<build>  
    <resources>  
        <!--  
            ${project.basedir}: 当前项目所在目录,子项目继承了父项目，  
            相当于所有的子项目都添加了资源目录的过滤  
        -->  
        <resource>  
            <directory>${project.basedir}/src/main/resources</directory>  
            <filtering>true</filtering>  
        </resource>  
    </resources>  
</build>
```

springboot配置多环境常在项目中resources下的配置文件中（application.yml、application.yml或者application.properties），在其中通过"---"来划分不同环境的配置，通过activate来启动环境
```yml
#设置启用的环境
spring:
  profiles:
    active: dev
 
---
spring:
  config:
    activate:
      on-profile: dev
 
server:
  port: 80
---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 81
---
spring:
  config:
    activate:
      on-profile: test
server:
  port: 82
```
### 2.2 书写内容

maven：maven中书写的内容常常是**一些变量值，如jdbc的连接信息**等

springboot：springboot中书写内容常常是**连接tomcat服务器的连接端口**


## springboot 集成了打包和运行环境的切换

https://cloud.tencent.com/developer/article/2317841