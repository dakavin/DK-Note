
## 1 配置修改

查看文件 [3_IDEA的安装与使用（上）](../../../1_JavaStudy/4_实用文件/6_Linux&Git&IDEA/2_IDEA/3_IDEA的安装与使用（上）.md) 和 [3_IDEA的安装与使用（下）](../../../1_JavaStudy/4_实用文件/6_Linux&Git&IDEA/2_IDEA/3_IDEA的安装与使用（下）.md) 即可

补充：
1. console输出中文乱码
	- 参考链接：[优雅的解决Clion控制台中文乱码(MinGw环境)_clion 控制台乱码-CSDN博客](https://blog.csdn.net/vodka_nice/article/details/120135412)
	- 取消run.processes.with.pty功能，按下Ctrl+shift+alt+/ 调出maintenance界面，点击Registry...进行配置
	- 取消勾选：`run.process.with.pty`
	  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/6cd6c08aa68ee9c19a3fb07c68c2e40a.png)

## 2 插件推荐

参考链接：[2023年Clion插件推荐 - 北极的大企鹅 - 博客园 (cnblogs.com)](https://www.cnblogs.com/liuyangfirst/p/17569046.html)
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/07/4079bf2454fd9211ed141dd9b9700d18.png)

## 3 一个文件夹如何创建多个c文件

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

# 遍历项目根目录下所有的 .c 文件，自动添加
file(GLOB_RECURSE files *.c **/*.c)
foreach (file ${files})
    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" exe ${file})
    add_executable(${exe} ${file})
    message(\ \ \ \ --\ \[${exe}.c\]\ will\ be\ compiled\ to\ \'.runtime/${exe}.exe\')
endforeach ()
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