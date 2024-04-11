
## 问题1

问题：Java8 + SpringBoot2 + Mybatis-plus3.5.3 ，mapper类加了@Mapper注解，spring配置类加了@MapperScan注解，整合Mybatis-plus出现如下报错
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92db0e4aedc0b3751154224780996b66.png)

原因：Mybatis-plus版本太新

解决：切换至使用比较多的版本 3.5.3.1即可解决

## 问题2

问题：出现下图报错 Access denied for user 'mikey'@'localhost' (using password: YES)
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/316c8a86ee87f248326963893698c9b7.png)

原因：yml文件，数据库用户名，写成了user

解决：应该写为username