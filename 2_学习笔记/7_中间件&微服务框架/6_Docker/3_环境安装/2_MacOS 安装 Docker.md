## 1 使用 Homebrew 安装

通过 [Homebrew](https://brew.sh/) 的 [Cask](https://github.com/Homebrew/homebrew-cask) 可以很方便的安装 Docker, 命令如下：

```docker
brew install --cask docker
```

## 2 手动安装

还可以访问下面链接，下载 Docker 安装包，手动安装 Docker：
[https://docs.docker.com/desktop/mac/install/](https://docs.docker.com/desktop/mac/install/)

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/f2f774a665178fdbf8ee5eb64468931a.png)


> 注意: 请选择对应芯片的安装包。

下载成功后，双击 `.dmg` 安装包，会出现如下界面，然后将 Docker 应用拖拽到 `Applications` 文件夹中（中间可能需要输入用户密码）。

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/9e3330f3ae9e4fa4f646e9f5cee9061c.png)
## 3 运行 Docker

安装成功后，双击 Docker 图标启动它。

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/bb991d946fc5d85dc4375f618426360e.png)

运行成功后，会在菜单栏看到一个鲸鱼的图标：

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/4eedc1ddee9208ded9cd1056e1c69de7.png)


点击 Docker 图标，若显示 `Docker Desktop is running` 则表示 Docker 运行成功啦。

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/7e5353218f941442bf271dc23f90bc5a.png)

## 4 检查当前 Docker 版本

运行成功后，打开命令行，执行 `docker version` 命令则可以成功输出 Docker 版本号了。

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/02a3735dac8fc4a82917dc9a5f51cc0c.png)
