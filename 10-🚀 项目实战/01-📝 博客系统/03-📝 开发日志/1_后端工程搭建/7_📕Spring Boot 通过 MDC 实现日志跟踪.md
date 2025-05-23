
## 1 什么是 MDC？为什么需要它？

MDC（Mapped Diagnostic Context）是 SLF4J 和 log4j 等日志框架提供的一种方案，它允许开发者将一些特定的数据（如用户ID、请求ID等）存储到当前线程的上下文中，使得这些数据可以在日志消息中使用。这对于跟踪多线程或高并发应用中的单个请求非常有用。

**在高并发环境中，由于多个请求可能同时处理，日志消息可能会交错在一起**。使用MDC，我们可以 _为每个请求分配一个唯一的标识，并将该标识添加到每条日志消息中，从而方便地区分和跟踪每个请求的日志_。
## 2 实战

### 2.1 日志切面中设置 MDC 值

编辑 `ApiOperationLogAspect` 日志切面类，在 `doAround()` 方法中处理请求开始的时候, 将请求的跟踪标识放入MDC 中：
```java
// traceId 表示跟踪 ID， 值这里直接用的 UUIDMDC.put("traceID", UUID.randomUUID().toString());
```
### 2.2 配置日志框架

在 `logback-weblog.xml` 配置文件中，可以使用 `%X` 来引用MDC中的值。例如，要引用上述的 `traceId`，你可以这样配置：
```xml
[TraceId: %X{traceId}] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c63ee9f69b37c08721828939d91f93d0.png)

### 2.3 清除 MDC 值

在请求结束时，为了避免污染其他请求，还需要清除 MDC 中的值：
```java
MDC.clear();
```
### 2.4 与AOP切面结合

所以，`ApiOperationLogAspect` 切面类中需添加的代码如下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ef1a2917a432e7b57e8119efa8bc45b2.png)

## 3 测试

我们将 `application.yml` 中的 `profile` 改为 `prod` 环境，来测试一波日志文件是否含有 `traceId`。 重启项目，再次请求 `/test` 接口，然后查看日志：

**！！！注意：使用postman是无法测试的，只有在服务器中开启服务才会获取到**
- 否则就卡在在springboot图标上
- traceId 是切面中设置进去的，请求会过切面，所以会打印，其他任何不过切面的，都不会打印具体 traceId，而postman的请求是不过切面的

## 4 小结