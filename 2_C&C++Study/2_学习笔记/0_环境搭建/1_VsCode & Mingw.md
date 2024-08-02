
## 1 卸载

没安装过，就不用卸载了

## 2 下载

**VsCode地址**：[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)

**Mingw**

打开 [WinLibs](https://winlibs.com/)。个人建议一般情况下选择 UCRT runtime，但如果需要用 [EasyX](https://easyx.cn/)，就选择 MSVCRT runtime（我更建议去用 [VS](https://visualstudio.microsoft.com/downloads/)）。

可以选最新版本，目前是 GCC 13.2.0；也可以选自己需要的其他版本。在 POSIX 和 MCF 之间，个人建议选择 POSIX。

如果在用 64 位机，就选择 Win64 的 [Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip)，若安装了 [7-Zip](https://www.7-zip.org/) 等压缩软件也可以下载 [7-Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.7z)，能比 zip 小不少；

如果在用 32 位机，就相应地选择 Win32 的 [Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-i686-posix-dwarf-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip) 或 [7-Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-i686-posix-dwarf-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.7z)。

这里以下载 [UCRT runtime GCC 13.2.0 (with POSIX threads) Win64 Zip archive](https://github.com/brechtsanders/winlibs_mingw/releases/download/13.2.0posix-17.0.6-11.0.1-ucrt-r5/winlibs-x86_64-posix-seh-gcc-13.2.0-llvm-17.0.6-mingw-w64ucrt-11.0.1-r5.zip) 为例。**友情提示**：压缩包从 [GitHub](https://github.com/) 下载，如果无法访问 GitHub

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/794fee498f2d714ee005cc82228ba449.png)

## 3 安装

### 3.1 Mingw

参考地址：https://blog.iw17.cc/vscode-c-cpp/

创建一个文件夹，然后将下载的压缩包所有文件解压过去即可
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/73d43c2f21be3ec0b6dc566ecf97062a.png)

设置环境变量

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/bae41c2398fb04703344445d5248f1e7.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/765502805219d0fb7aa61412de4fae45.png)

使用cmd 测试一下
```shell
gcc --version
gcc -v
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/c4ad23789485489d473e3ff99cd4eefa.png)

编译命令
```c
// c
gcc <filename>.c -o <filename>.exe
// c++
gg++ <filename>.cpp -o <filename>.exe
```

### 3.2 VsCode

直接安装，疯狂下一步即可

#### 3.2.1 设置编译环境

打开 VS Code，按 Ctrl+Shift+X 键，搜索 “c/c++” 插件并点击 “Install” 安装
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/27639453585012310e1860f73955150a.png)

按 Ctrl+K Ctrl+O “打开文件夹”（Open Folder），选择或新建一个文件夹，作为写 C 语言程序的工作区（workspace）
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/304f0ec752a6d6cf66552343bc522880.png)


#### 3.2.2 复制json文件

在`.vscode` 文件夹里新建一个文件，命名为 `launch.json`，代码如下：
```json
{
    "version": "0.2.0",
    "configurations": [
        { //‘调试(Debug)
            "name": "Debug",
            "type": "cppdbg",
            // cppdbg对应cpptools提供的调试功能；只能是cppdbg
            "request": "launch",
            //这里program指编译好的exe可执行文件的路径,与tasks中要对应
            "program": "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe", //(单文件调试)
            //"program": "${workspaceFolder}\\${workspaceRootFolderName}.exe", //(多文件调试)
            "args": [],
            "stopAtEntry": false, // 这里改为true作用等同于在main处打断点
            "cwd": "${fileDirname}", // 调试程序时的工作目录,即为源代码所在目录,不用改
            "environment": [],
            "externalConsole": false, // 改为true时为使用cmd终端,推荐使用vscode内部终端
            "internalConsoleOptions": "neverOpen", // 设为true为调试时聚焦调试控制台,新手用不到
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\mingw64\\bin\\gdb.exe",
            // 指定调试器所在路径，注意间隔是\\,请修改为你的路径
            // 指定调试器所在路径，注意间隔是\\,请修改为你的路径
            // 指定调试器所在路径，注意间隔是\\,请修改为你的路径
            "preLaunchTask": "build" // 调试开始前执行的任务(任务依赖),与tasks.json的label相对应
        }
    ]
}

```

再在`.vscode` 文件夹里新建一个 `tasks.json` 文件，写入如下代码
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            //这里构建build任务
            "label": "build",
            "type": "shell",
            "command": "gcc",
            "args": [
                //此处为编译选项
                "${file}",//该(单文件编译)
                //"${workspaceFolder}\\*.c",//(多文件编译)
                "-o",
                //承接上述,把源代码编译为对应exe文件,
                "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",//(单文件编译)
                //"${workspaceFolder}\\${workspaceRootFolderName}.exe",//(多文件编译)
                "-g",
                "-Wall",//获取警告
                "-static-libgcc",
                "-fexec-charset=GBK",//按GBK编码
                "-std=c11"//选择C标准,这里按照你需要的换
            ],
            "group": {
                //把该任务放在build组中
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                //配置build任务的终端相关
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "new"//为了方便每次都重新开启一个终端
            },
            "problemMatcher": "$gcc"
        },
        {
            //这里配置run任务
            "label": "run",
            "type": "shell",
            "dependsOn": "build",
            "command": "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",//(单文件编译)
            //"command":"${workspaceFolder}\\${workspaceRootFolderName}.exe",//(多文件编译)
            //这里command与前面build中的编译输出对应
            "group": {
                //这里把run任务放在test组中,方便我们使用快捷键来执行程序
                //请人为修改"设置","键盘快捷方式"中的"运行测试任务"为"你喜欢的键位"
                //推荐为"ALT+某个字母键",使用该键来运行程序
                "kind": "test",
                "isDefault": true
            },
            "presentation": {
                //同理配置终端
                "echo": true,
                "reveal": "always",
                "focus": true,
                "panel": "new"
            }
        }
    ]
}

```
#### 3.2.3 编译运行

选择右上角的播放按钮
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/b02b7e2383d72cd93141490f7371c7ee.png)

在下方调试控制台（Debug Console）看到程序正常结束，输出（黄字）以 `exited with code 0 (0x00000000)` 结尾。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/ea8c6620c144972451b44a7ae31f1817.png)

点击下方 “终端”（Terminal），看到输出 `hello, world`，则环境配置成功。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/1cf627a969fa19010bc2df984299fc84.png)


然后就会在当前的c文件的同级目录下生成一个exe文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/55b1d287361de3eeb84760b57c3ee6c5.png)

进入当前c文件所在目录，输入 `.\main.exe` 也可运行当前文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/29102b429cbc83c6debdff0ae608c55f.png)

#### 3.2.4 小提示

1. 如果想在外部控制台（窗口）显示输出，可以把上面 `launch.json` 第 12 行的 `"externalConsole": false` 改为 `"externalConsole": true`。如果控制台闪退，可以在最后加 `getchar();` 或 `system("pause");` 等语句，不让程序直接退出。
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/43de4b70c16d5a149ca77272143837e5.png)


2. 有关路径和文件名最好没有非 ASCII 字符（比如中文）和空格。