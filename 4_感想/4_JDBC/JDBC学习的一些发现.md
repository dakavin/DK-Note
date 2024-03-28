
## 1. 关于一个方法的结束

- 一个方法中，定义了事务不是默认提交的
- 并且在方法的最后，没有进行事务的提交或者回滚
- `那么方法结束时，该事务会自动回滚到最初状态！！！`

## 2. junit的批量测试操作

1. 在src或其他包下定义一个test source root类型的包
	- 右键该包，在mark directory as 中选择
	- 或者在project structure中按下图进行选择

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923010312241.png)



![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923010313748.png)



2. 在测试所在类中alt + ins 

![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923010315167.png)



## 3. 修改Date的做法

```java
LocalDate localDate = LocalDate.of(1999,9,30);

//转化为sql下的date
java.sql.Date.valueOf(localDate);
```

## 4. 导入jar包的操作

1. 在项目下，新建一个名为lib文件夹
![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923010321481.png)


1. 将jar拷贝到该文件夹中，右键jar，点击add as library

1. 在project structure中，给每个模块添加该jar文件

## 添加XML文件的方法

