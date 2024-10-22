**Mingw**

打开 [WinLibs](https://winlibs.com/)。个人建议一般情况下选择 UCRT runtime，但如果需要用 [EasyX](https://easyx.cn/)，就选择 MSVCRT runtime（我更建议去用 [VS](https://visualstudio.microsoft.com/downloads/)）。

可以选最新版本，目前是 GCC 13.2.0；也可以选自己需要的其他版本。在 POSIX 和 MCF 之间，个人建议选择 POSIX。

如果在用 64 位机，就选择 Win64 的 [Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip)，若安装了 [7-Zip](https://www.7-zip.org/) 等压缩软件也可以下载 [7-Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.7z)，能比 zip 小不少；

如果在用 32 位机，就相应地选择 Win32 的 [Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-i686-posix-dwarf-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip) 或 [7-Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-i686-posix-dwarf-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.7z)。

这里以下载 [UCRT runtime GCC 13.2.0 (with POSIX threads) Win64 Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip) 为例。**友情提示**：压缩包从 [GitHub](https://github.com/) 下载，如果无法访问 GitHub


- `x86_64`是指64位的操作系统，i686是指32位的操作系统。现在系统都是64位操作系统，所以选择x86_64。

- `win32`是开发`windows`系统程序的协议，posix是其他系统的协议（例如Linux、Unix、Mac OS）。

- 异常处理模型 **seh（新的，仅支持64位系统）**，**sjlj （稳定的，64位和32位都支持）**， **dwarf （优于sjlj的，仅支持32位系统）**


![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/09/794fee498f2d714ee005cc82228ba449.png)

## 3 安装

### 3.1 Mingw

参考地址：https://blog.iw17.cc/vscode-c-cpp/

创建一个文件夹，然后将下载的压缩包所有文件解压过去即可
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/09/73d43c2f21be3ec0b6dc566ecf97062a.png)

设置环境变量

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/09/bae41c2398fb04703344445d5248f1e7.png)

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/09/765502805219d0fb7aa61412de4fae45.png)

使用cmd 测试一下
```shell
gcc --version
gcc -v
```

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/09/c4ad23789485489d473e3ff99cd4eefa.png)

编译命令
```c
// c
gcc <filename>.c -o <filename>.exe
// c++
gg++ <filename>.cpp -o <filename>.exe
```