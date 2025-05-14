
## 1 配置修改

查看文件 [3_IDEA的安装与使用（上）](../../../1_感想或实用/7_Linux&Git&IDEA/2_IDEA/3_IDEA的安装与使用（上）.md) 和 [3_IDEA的安装与使用（下）](../../../1_感想或实用/7_Linux&Git&IDEA/2_IDEA/3_IDEA的安装与使用（下）.md) 即可

补充：
1. console输出中文乱码
	- 参考链接：[优雅的解决Clion控制台中文乱码(MinGw环境)_clion 控制台乱码-CSDN博客](https://blog.csdn.net/vodka_nice/article/details/120135412)
	- 取消run.processes.with.pty功能，按下Ctrl+shift+alt+/ 调出maintenance界面，点击Registry...进行配置
	- 取消勾选：`run.process.with.pty`
	  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/6cd6c08aa68ee9c19a3fb07c68c2e40a.png)
## 2 HelloWorld的实现

### 2.1 新建Project

- 选择"New Project"：
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/1edc54945a217398d9834f47f9c5c603.png)
- 指定创建C可执行文件、工程目录，图中的“untitled1”需要修改为自己的工程名称。如下所示：
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/27e38ec27d83beab4424878e2012dbdc.png)
- 选择C可执行文件，修改工程名称为demo1
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/b35b2a78fba7cd2abd648c5396a87047.png)
  - 点击“Create”进行下一步，如图所示
	  - 这里Toolset需要改本地的，不然不会有编写提示
    ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/769cb8a559d2d11588fe6b04ea897632.png)
- 此处选择编译器，默认MinGW即可，点击“OK”按钮，如图所示，默认创建了main.c文件。
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/fad08a9f5789e57eea555895c8c9a33a.png)
### 2.2 运行

- 点击执行按钮，如下所示
  
## 3 一个文件夹如何创建多个c文件

参考链接：
- https://www.cnblogs.com/liuyangfirst/p/9559199.html
- https://www.cnblogs.com/liuyangfirst/p/17568673.html
- https://www.cnblogs.com/liuyangfirst/p/16710565.html

参考链接：https://github.com/CarmJos/CWorks/blob/master/.doc/SETTINGS_C.md


**针对初学者使用CLion的注意事项——C语言**

**修改CMakeList.txt**

默认情况下，Clion适用于单独的项目，即一个项目下的文件有且仅有一个main函数。

而针对初学者，我们的作业常常都是单独的一个小程序。如果每个作业都必须新建项目，则太过繁琐且没必要。 因此在使用时，需要修改CMakeList.txt文件，以适应我们的需求。

我们打开根目录下的CMakeList.txt文件，除了第一行、第二行以外，其余内容全部删除，替换为以下内容：

```cmake
# 按照书本要求设定C语言版本
set(CMAKE_C_STANDARD 99)

# 设定构建运行路径，避免污染根目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.archive)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.library)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.runtime)
set(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/.path)

# 遍历项目根目录下所有的 .c 文件，为每一个 .c文件生成一个独立的目标
file(GLOB_RECURSE files *.c **/*.c)
foreach (file ${files})
    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" exe ${file})
    add_executable(${exe} ${file})
    message(\ \ \ \ --\ \[${exe}.c\]\ will\ be\ compiled\ to\ \'.runtime/${exe}.exe\')
endforeach ()
```

拆分为不同的文件夹，文件夹内部的c和h文件有联系，外部没有
```CMake
cmake_minimum_required(VERSION 3.28)  
project(3_C_Algorithm C)  
  
# 按照书本要求设定C语言版本  
set(CMAKE_C_STANDARD 99)  
  
# 设定构建运行路径，避免污染根目录  
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.archive)  
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.library)  
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.runtime)  
set(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/.path)  
  
# 遍历项目根目录下所有的 .c 文件，自动添加  
file(GLOB_RECURSE files *.c **/*.c)  
foreach(dir IN ITEMS a b c)
    file(GLOB sources ${dir}/*.c)
    add_executable(${dir}_exec ${sources})
    target_include_directories(${dir}_exec PRIVATE ${CMAKE_SOURCE_DIR}/${dir})
    message(STATUS "Compile ${dir}/*.c into ${dir}_exec")
endforeach()
```

使用此CMakeList时，若要新建C语言文件，请按照以下步骤：
1. 右键根目录——新建——C/C++源文件
2. 在弹出的对话框中，输入文件名(英文小写及下划线)，后缀为 `.c`，不要勾选“添加到目标”，点击确定。
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/575ebd736018f8fa4685e00271419034.png)

3. 点击 左上角横线——文件——重新加载CMake项目 。
   ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/132bdda037f071636a89579d3e175271.png)
---

**针对初学者使用CLion的注意事项——C++**

**修改CMakeList.txt**

默认情况下，Clion适用于单独的项目，即一个项目下的文件有且仅有一个main函数。

而针对初学者，我们的作业常常都是单独的一个小程序。如果每个作业都必须新建项目，则太过繁琐且没必要。 因此在使用时，需要修改CMakeList.txt文件，以适应我们的需求。

我们打开根目录下的CMakeList.txt文件，除了第一行、第二行以外，其余内容全部删除，替换为以下内容：

```cmake
# 按照书本要求设定C++版本
set(CMAKE_CXX_STANDARD 20)

# 设定构建运行路径，避免污染根目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.archive)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.library)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/.runtime)
set(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/.path)

# 遍历项目根目录下所有的 .c 文件，自动添加
file(GLOB_RECURSE files *.cpp **/*.cpp)
foreach (file ${files})
    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" exe ${file})
    add_executable(${exe} ${file})
    message(\ \ \ \ --\ \[${exe}.cpp\]\ will\ be\ compiled\ to\ \'.runtime/${exe}.exe\')
endforeach ()
```

使用此CMakeList时，若要新建C语言文件，请按照以下步骤：

1. 右键根目录——新建——C/C++源文件
2. 在弹出的对话框中，输入文件名(英文小写及下划线)，后缀为 `.cpp`，不要勾选“添加到目标”，点击确定。
3. 点击 左上角横线——文件——重新加载CMake项目 。

## 4 文件头信息

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/08/f4bad34899e3141c2e11efe19be46f0c.png)


```template
/**
 * @file ${FILE_NAME}
 * @author dakkk
 * @brief 
 * @version 0.1
 * @date ${DATE}
 * 
 * @copyright Copyright (c) ${YEAR}
 * 
 */

#include "${FILE_NAME}.h"

${BODY}
```

```template
/**
 * @file ${FILE_NAME}
 * @author dakkk
 * @brief 
 * @version 0.1
 * @date ${DATE}
 * 
 * @copyright Copyright (c) ${YEAR}
 * 
 */
```

## 5 ssh连接远端服务器

需要提前**安装cmake、gcc、g++和gdb**
- cmake：跨平台的自动化构建系统生成工具，用于管理编译过程
- gcc：c编译器
- g++：c++编译器
- gdb：gnu调试器，调试
```shell
sudo apt install cmake -y
sudo apt install gcc -y
sudo apt install g++ -y
sudo apt install gdb -y

#验证
cmake --version
gcc -v
g++ -v
gdb -v

#查看安装路径
which命令即可
```

打开设置添加远程主机，并点击凭据后的设置按钮
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c09548be4f7ef65ec7a83e52bd921d84.png)


输入ssh相关信息，然后测试连接，成功即可
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9709f74da3bf2f6dc9aa45ca571c3a0d.png)

指定远程linux的相关编译工具的路径，一般都是在 /bin 或者 /usr/bin 目录下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/1c82c4ff08a1b13f9a8d31308bd5df2d.png)

接下来开始部署，来到映射，配置部署路径，这个部署路径就是你Clion项目放到Linux上面的位置，然后点击应用
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/9c528c43d144bddc3ef71ade13eb5979.png)

接着是CMake，将工具链换成远程主机的，生成器推荐使用Ninja(要在Linux上面安装了并写到环境变量才能使用，不然会报错)，点击应用
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c8aba9584384211fd83c47c5107e11f1.png)

接着回到项目页面，你就发现Clion自动将项目文件上传到远程主机了。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/70d5036743501515bef91d60e68686da.png)

## 6 插件推荐

参考链接：[2023年Clion插件推荐 - 北极的大企鹅 - 博客园 (cnblogs.com)](https://www.cnblogs.com/liuyangfirst/p/17569046.html)
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/04/4079bf2454fd9211ed141dd9b9700d18.png)
