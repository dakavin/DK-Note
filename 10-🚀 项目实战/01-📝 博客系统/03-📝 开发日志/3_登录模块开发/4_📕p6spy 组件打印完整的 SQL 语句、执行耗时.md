
## 1 前言

前面小节中，我们已经在 `blog` 项目中整合了 Mybatis Plus 。本小节中，我们来配置一下 Mybatis Plus 打印 SQL 功能（包括执行耗时），一方面可以了解到每个操作都具体执行的什么 SQL 语句， 另一方面通过打印执行耗时，也可以提前发现一些慢 SQL，提前做好优化， 省得 DBA 公开处刑。**注意，生产环境不推荐打印执行 SQL，会有数据泄漏风险，仅推荐本地开发使用。**

> TIP : 此种方式为官方推荐，通过 `p6spy` 组件来实现完整的 SQL 打印。请使用 Mybatis Plus 3.1.0 以上版本。

## 2 添加依赖

在父项目 `blog-springboot` 的 `pom.xml` 文件中，声明 `p6spy` 依赖的版本号：

```xml
<properties>
	// 省略...
	<p6spy.version>3.9.1</p6spy.version>
</properties>

<dependencyManagement>
	<dependencies>
		// 省略...
		<dependency>
			<groupId>p6spy</groupId>
			<artifactId>p6spy</artifactId>
			<version>${p6spy.version}</version>
		</dependency>
	</dependencies>
</dependencyManagement>
```

然后在 `weblog-module-common` 模块中的 `pom.xml` 文件中，引入该依赖：

```xml
<dependency>  
    <groupId>p6spy</groupId>  
    <artifactId>p6spy</artifactId>  
</dependency>
```

## 3 添加配置

### 3.1 第一步：修改 `application-dev.yml` 配置文件

`application-dev.yml` 配置文件：

```yml
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://127.0.0.1:3306/weblog?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&zeroDateTimeBehavior=convertToNull
    ...
```

> **注意：**
> - `driver-class-name` : 修改为 `p6spy` 提供的驱动类;
> - `url` ： 修改为前缀为 `jdbc:p6spy` 跟着冒号，后面对应数据库连接地址；

### 3.2 第二步：添加 `p6spy` 配置文件

然后在 `weblog-web` 模块中的 `resources` 目录下添加 `spy.properties` 配置文件：

![](https://img.quanxiaoha.com/quanxiaoha/169275255938724)

配置文件内容如下：

```properties
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```

## 4 看看最终效果

配置添加完成后，再次执行上小节中的单元测试方法 `insertTest()` ，观察控制台输出，效果图如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dad880a99a3a234b8c3fa97e7d41e2ac.png)
可以看到完整的打印了执行语句，以及执行耗时为 17 ms：

## 5 生产环境不要启用 p6spy

`p6spy` 组件**请勿在生产环境使用，因为有性能损耗，推荐仅本地 `dev` 环境开发开启使用。** 最终做法是，在 `applcation-prod.yml` 文件不启用 `p6spy` 组件，我们将 `application-dev.yml` 的数据库配置复制一份到 `application-prod.yml` 文件中，注意，要将驱动、前缀恢复正常，内容如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78d277db95b3be19e3936404efc1f6b8.png)

## 6 结语

本文给大家介绍了在项目用使用 Mybatis Plus 持久层框架时，如何通过 `p6spy` 组件完整的打印 SQL 语句，以及执行耗时，此功能对于本地开发定位问题、杜绝慢查询都非常有帮助。