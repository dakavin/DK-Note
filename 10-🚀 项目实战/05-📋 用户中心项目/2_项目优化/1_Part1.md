---
文章标题: "[[1_Part1]]" 
文章作者: Dakkk
文章概要: |
  文章主要解决了Spring Boot应用中因Jackson默认时区GMT导致前端时间与后端MySQL不一致的问题。提供了在`application.yml`中配置Jackson时区以及在MySQL连接URL中设置服务器时区为`GMT+8`的具体解决方案，以确保时间同步。
文章标签:
- "Spring Boot"
- "Jackson"
- "时区"
- "MySQL"
- "配置"
- "时间同步"
- "后端开发"
相关文章:
- "[[2_文章归档分页接口开发]]"
- "[[9_📕Spring Boot 自定义响应工具类]]"
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_Java学习路线]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/05-📋 用户中心项目/2_项目优化/1_Part1.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 时间问题

问题：后端的数据库中，用户的创建时间等都正常，但是前端的时间却多了8h

分析：由于我们使用的是SpringBoot框架，SpringBoot中对于@RestController或者@Controller+@ResponseBody注解的接口方法的返回值默认是Json格式，所以对于data类型的数据，在返回浏览器段被SpringBoot默认的JackJson框架转换，而JackSon框架默认的时区是GMT，相对于中国少了8个小时

解决：在springboot项目中，添加如下配置
```yml
spring.jackson.time-zone=GMT+8

在mysql的url配置中，后缀加上下面的
serverTimezone=GMT%2B8
```

## 优化加载页面和主页



## 用户头像设置默认


## 管理员增加对用户的增删改查


### 增加用户界面


### 实现用户的
