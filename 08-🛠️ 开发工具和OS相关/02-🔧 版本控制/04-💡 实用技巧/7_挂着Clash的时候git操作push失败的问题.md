
当开启[Clash](https://so.csdn.net/so/search?q=Clash&spm=1001.2101.3001.7020)后，本机网络会被代理，此时可以在 设置-网络-代理 中看到：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/87046c6fd5d24f8fe7b974de66047b5d.png)

git push 失败的原因就是本机开启了代理，而git没有设置代理，导致[443端口](https://so.csdn.net/so/search?q=443%E7%AB%AF%E5%8F%A3&spm=1001.2101.3001.7020)转发不过去，此时只需要设置以下git的代理即可解决
```shell
git config --global http.proxy http://127.0.0.1:7890 
git config --global https.proxy http://127.0.0.1:7890
```

取消和查看代理
```shell
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看代理
git config --global --get http.proxy
git config --global --get https.proxy
git config --list

```