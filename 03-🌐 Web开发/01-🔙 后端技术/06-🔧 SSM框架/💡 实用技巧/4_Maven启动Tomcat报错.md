
```txt
Illegal access: this web application instance has been stopped already. Could not load [org.mybatis.spring$SqlSessionFactoryBean]. The following stack trace is thrown for debugging purposes as well as to attempt to terminate the thread which caused the ...
```

报错信息如上

**原因**  
因为在tomcat重启的时候，因为之前的tomcat中的线程还没有完全关闭，新启动tomcat就会报这个异常，不过这个不影响正常使用，只是跳个异常挺烦人的。

**解决方法**  
把tomcat的server.xml 中的reloadable=“true” 改成false

**其他办法**

如果找到不这个字段,在该文件中直接添加如下标签

```xml
<Context path="/expert" docBase="expert" debug="0" reloadable="false"></Context>
```