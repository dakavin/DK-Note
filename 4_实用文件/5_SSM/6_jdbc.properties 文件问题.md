
问题情况:
```shell
java.io.FileNotFoundException: Could not open ServletContext resource [/jdbc.properties]
```

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/15d91f36be697131ac2a55ae46597928.png)
解决办法:
- 因为是使用注解开发 , 所以在使用@PropertySource注解的时候 , 需要加上 `classpath:`

![image.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7149685d4660e96bb03dca14772401ef.png)
