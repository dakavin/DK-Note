
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

1. 选则文件---> FTP ---> FTP上传![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4e0a9a0cbc32838b1d21d8f9c2b234d6.png)
2. 选中设置![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b63969a95c854d949b0264a5675fe20.png)
3. 选中添加，并填入LInux服务器的相关信息![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce859f1dc0fe3c63d23be556d3cdac48.png)