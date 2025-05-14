## 1 查看系统当前使用的文件使用

```shell
#在主机或板卡上执行如下命令
df -T
```

## 2 定位头文件

```shell
#我们需要安装该软件
sudo apt install locate
#然后需要更新一下数据
sudo updatedb
#然后就可以使用了

locate sys/stat.h
```