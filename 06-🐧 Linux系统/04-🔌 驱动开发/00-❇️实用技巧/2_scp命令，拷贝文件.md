
scp 命令用于 Linux 之间复制文件和目录，该命令基于ssh，需要搭建好ssh环境，scp命令格式如下：
```shell
scp local_file remote_username@remote_ip:remote_folder
```

**转移文件如下**
```shell
scp hellomodule.ko cat@192.168.31.69:/home/cat/
```

将hellomodule.ko发送到192.168.31.69的/home/cat/目录下，192.168.31.69是开发板ip，需根据实际而定，开发板用户名为cat， 输入yes，然后输入密码进行验证，等待传输完成，这个时候我们开发板就有了hellomodule.ko 这个文件。如果是在开发板进行本地编译则不需要再进行传输

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d057f8c9aa9fd6148a847b7335de0450.png)
