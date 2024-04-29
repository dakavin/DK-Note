
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
