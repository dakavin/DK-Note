## 1 简介

`Device Tree`文件的格式为`dts`，包含的头文件格式为`dtsi`，`dts`文件是一种人可以看懂的编码格式。但是`uboot`和`linux`不能直接识别，他们只能识别二进制文件，所以需要把`dts`文件编译成`dtb`文件。

`dtb`文件是一种可以被`kernel`和`uboot`识别的二进制文件。把`dts`编译成`dtb`文件的工具是`dtc`。`Linux`源码目录下`scripts/dtc`目录包含`dtc`工具的源码。

在`Linux`的`scripts/dtc`目录下除了提供`dtc`工具外，也可以自己安装`dtc`工具，`linux`下执行：`sudo apt-get install device-tree-compiler`安装`dtc`工具。

`dtc`工具的使用方法是：
```C
dtc –I dts –O dtb –o xxx.dtb xxx.dts
```

反过来即可生成`dts`文件
```C
dtc –I dtb –O dts –o xxx.dts xxx.dtb
```
## 2 Device Tree 头信息

我们可以使用`fdtdump`的工具，用于`dump dtb`文件,方便查看dtb信息

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/279d8bb1d1ee9d87bb39876a98bc7550.png)

以上信息便是`Device Tree`文件头信息，存储在`dtb`文件的开头部分。在`Linux`内核中使用`struct fdt_header`结构体描述。`struct fdt_header`结构体定义在`scripts\dtc\libfdt\fdt.h`文件中
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/44c1ba8d64089dd743c3740d20c1e1f2.png)

## 3 Device Tree文件结构

`Device Tree`源文件的结构分为`header`、`fill_area`、`dt_struct`及`dt_string`四个区域。

`fill_area`区域填充数值`0`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/b8c137a71cd369d381d98b252a336c36.png)

节点（`node`）信息使用`struct fdt_node_header`结构体描述。属性信息使用`struct fdt_property`结构体描述。各个结构体信息如下:
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c200e5bce1939c9fcdc5ce642853d011.png)

`struct fdt_node_header`描述节点信息，`tag`是标识`node`的起始结束等信息的标志位，`name`指向`node`名称的首地址。`tag`的取值如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/668871cb8a400eaed200d9981cafa0c5.png)

`FDT_BEGIN_NODE`和`FDT_END_NODE`标识`node`节点的起始和结束，`FDT_PROP`标识`node`节点下面的属性起始符，`FDT_END`标识`Device Tree`的结束标识符。因此，对于每个`node`节点的`tag`标识符一般为`FDT_BEGIN_NODE`，对于每个`node`节点下面的属性的`tag`标识符一般是`FDT_PROP`。

描述属性采用`struct fdt_property`描述，tag标识是属性，取值为`FDT_PROP`；`len`为属性值的长度（包括`‘\0’`，单位：字节）；`nameoff`为属性名称存储位置相对于`off_dt_strings`的偏移：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ff89d9fcb68a9ee7242f6c1984e3709d.png)
