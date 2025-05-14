
`ag` 是一个非常快速的代码搜索工具，全名是 **The Silver Searcher**，它的作用跟 `grep` 类似，但是速度更快、对程序员更友好。

## 1 ✅ **`ag` 能干什么？**

简单说，它的功能是：
> **在目录中快速查找包含某个关键词的文件和行内容。**


## 2 🔧 **常用语法：**
```bash
ag 关键词
```

比如你要找包含 `rk3568-lubancat-` 的内容，就可以这样写：
```bash
ag rk3568-lubancat-
```
它会在当前目录（和子目录）里，列出所有包含这个字符串的行和文件路径。


## 3 🔍 **`ag` 的优势：**

|特性|说明|
|---|---|
|✅ 超快|基于 `grep` 和 `ack` 改进，搜索速度极快。|
|✅ 支持正则表达式|可以用正则进行复杂搜索。|
|✅ 忽略 `.gitignore`|默认跳过 `git` 忽略的文件。|
|✅ 高亮搜索词|搜索结果中自动高亮匹配内容。|
|✅ 智能代码项目支持|更适合在代码项目中使用，如 C/C++、Python、DTS、Makefile 等。|

## 4 📦 **安装方法：**

- Ubuntu / Debian：
    ```bash
    sudo apt install silversearcher-ag
    ```

- macOS（使用 Homebrew）：
    ```bash
    brew install the_silver_searcher
    ```


## 5 📌 示例用途：

- 查找某个函数定义：
    ```bash
    ag my_function
    ```

- 查找设备树中的某个芯片名：
    ```bash
    ag rk3568-lubancat-
    ```

- 结合 `vim` 使用（打开搜索结果中的文件）：
    ```bash
    vim $(ag -l rk3568-lubancat-)
    ```

