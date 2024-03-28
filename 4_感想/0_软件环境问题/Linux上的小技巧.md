
## 1、远程访问Linux的mysql

- 设置Window桌面端软件sqlyog的远程访问LInux中的mysql

- 在linux中登陆mysql
```shell
mysql -u root -p
```

- 查看mysql库中的user表的host字段
```mysql
use mysql
select user,host from user
```
- host字段中，localhost表示只允许本机访问，要实现远程连接，可以将root用户的host改为%，%表示允许任意host访问，如果需要设置只允许特定ip访问，则应改为对应的ip。

- 修改root用户的host字段，命令：
```mysql
update user set host="%" where user="root"

flush privileges

```

- 重启mysql服务
```shell
systemctl stop mysqld
systemctl start mysqld
```

- ***注意：在linux中很多软件的端口都被”防火墙”限止，我们需要将防火墙关闭***

## 2、远程访问Linux的配置文件

- 使用软件为EditPlus

1. 选则文件---> FTP ---> FTP上传![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923005755271.png)
2. 选中设置![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923005802361.png)
3. 选中添加，并填入LInux服务器的相关信息![](https://image-for.oss-cn-guangzhou.aliyuncs.com/for-obsidian/Java_Study/2_%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/image-20230923005807841.png)