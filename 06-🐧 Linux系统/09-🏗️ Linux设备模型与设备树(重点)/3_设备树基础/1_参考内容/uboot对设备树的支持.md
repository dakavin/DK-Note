
## 1 传递dtb给内核

### 1.1 uboot启动内核的两种方式

uboot启动内核时，可以选择是否使用设备树：
```shell
# 方式1：不使用设备树
bootm <uImage_addr>           # 例如： bootm 0x300007FC0

# 方式2：使用设备树
bootm <uImage_addr> <initrd_addr> <dtb_addr>  # 完整格式
```
- `bootm`：boot memort的缩写，从内存启动系统的命令
- `uImage`：uboot Image，带有uboot头部信息的内核镜像文件
- `addr`：address，表示内存地址
- `initrd`：initial ramdisk，初始内存盘，包含启动时需要的基本文件系统
- `dtb`：设备树二进制文件，包含硬件信息

### 1.2 实际使用示例

```shell
# 从NAND Flash读取内核到内存
nand read.jffs2 0x30007FC0 kernel

# 从NAND Flash读取设备树到内存
nand read.jffs2 32000000 device_tree

# 启动内核（没有initrd时用“-”占位）
bootm 0x30007FC0 - 0x32000000
```
- `nand`：NAND Flash存储器类型
- `jffs2`：Journalling Flash File System version 2，日志闪存文件系统第2版
- `kernel`：这里指，Flash中存储内核的分区名称
- `device_tree`：这里指，Flash中存储设备树的分区名称
- `-`：占位符，表示该参数为空（这里表示没有initrd）

### 1.3 设备树地址如何传递给内核？

这里涉及ARM程序调用规则（ATPCS）：可以百度搜索一下
```c
// 函数调用传递规则
c_function(p0, p1, p2)   // p0传给r0寄存器, p1 -> r1, p2 -> r2

// uboot内部实现
定义函数指针 the_kernel, 指向内核启动地址
执行：the_kernel(0, machine_id, 0x32000000);
// 这样dtb地址就通过r2寄存器传给了内核
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/98bf5df05c052e7ce55fa2b6550ed26c.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c05989d408c21c752f5896c45aa6b12e.png)

### 1.4 设备树放置位置的注意事项

选择dtb存放地址时需要考虑
1. **不能破坏uboot本身**
2. **不能占用内核需要的空间**（设备树是给内核使用的）
	- 内核本身的存储空间
	- 内核启动时要用的内存区域
	- 内核会在其位置下方创建页表（通常16KB空间）

### 1.5 JZ2440内存布局图

```txt
内存地址分布：
0x33f80000 ┌─────────────────────────────┐
           │         U-Boot              │
           ├─────────────────────────────┤
           │   U-Boot使用的内存(栈等)     │
           ├─────────────────────────────┤
           │                             │
           │                             │
           │         空闲区域             │
           │                             │
           │                             │
           ├─────────────────────────────┤
0x30008000 │        zImage               │
           ├─────────────────────────────┤  ← uImage = 64字节头部 + zImage
0x30007FC0 │      uImage头部             │
           ├─────────────────────────────┤
0x30004000 │     内核创建的页表           │  ← 分析head.S创建即可知道
           ├─────────────────────────────┤
           │                             │
0x30000000 └─────────────────────────────┘  ← 内存基址
```
- `U-Boot`：Universal Boot Loader，通用启动加载程序
- `zImage`：压缩的Linux内核镜像文件
- `uImage`：u-boot Image，带有64字节头部的内核镜像文件
- `head.S`：Linux内核启动汇编代码文件，负责早期初始化工作


**如何知道内核是在0x30008000这里？**
```shell
# 使用mkimage命令可以查看img文件的信息
mkimage -l xxx
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9b2683af6cfa4250c3f6661c02b43ae6.png)

### 1.6 启动命令示例

✅正确示例（可以启动）：
```shell
nand read.jffs2 30000000 device_tree     # DTB放在安全位置
nand read.jffs2 0x30007FC0 kernel
bootm 0x30007FC0 - 30000000
```

❌错误示例（无法启动）
```shell
nand read.jffs2 30004000 device_tree     # DTB放在页表位置，会被破坏
nand read.jffs2 0x30007FC0 kernel
bootm 0x30007FC0 - 30004000
```

## 2 dtb的修改原理

**本质就是`挪`，通过`memmove`函数来移动**

### 2.1 修改已有属性的过程

假设要修改一个属性：
- 老值长度：len
- 新值长度：newlen（假设newlen>len）

**修改步骤**：
1. **扩展空间**：把原属性值占用的空间从len字节拓展为newlen字节
	- 将老值之后的所有内容向后移动（newlen-len）字节
2. **写入新值**：把新值写入扩展后的newlen字节空间
3. **更新DTB头部信息**：
	- 修改sturcture block的长度：`size_dt_struct`
	- 修改sting block的偏移值：`off_dt_strings`
	- 修改DTB总长度：`totalsize`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a163ace6b9864864b511c9a708f46979.png)

### 2.2 添加全新属性的过程

1. **处理属性名**：
	- 如果string block中没有这个属性名，在尾部添加新字符串
	- 更新string block长度：`size_dt_strings`
	- 更新DTB总长度：`totalsize`
2. **在节点中添加属性**：
	- 在属性所在节点尾部扩展空间，添加以下内容：
	```shell
	TAG     // 4字节，值为0x00000003
	len     // 4字节，表示属性值的长度
	nameoff // 4字节，表示属性名在string block中的偏移
	val     // len字节，存放属性值
```
3. **更新DTB头部信息**：
	- 修改sturcture block的长度：`size_dt_struct`
	- 修改sting block的偏移值：`off_dt_strings`
	- 修改DTB总长度：`totalsize`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ce5fb484c30197c572a52d8bd8ceecb3.png)

### 2.3 FDT命令的实现原理

查看uboot源码中的 `cmd/fdt.c` 文件，了解命令调用过程：
```shell
fdt set <path> <prop> [<val>]  # 设置属性值命令
```

![fdt help.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/83048a157d87d0808dfaddf8cc5fe1af.png)

**set prop内部实现流程**:
1. **根据path找到节点**
2. **处理新值**：根据val确定新值长度newlen，并转换为字节流
3. **调用fdt_setprop**：
	- `fdt_setprop_placeholderr`：为新值在DTB中腾出空间
		- `fdt_get_property_w`：获取老值长度
		- `fdt_splice_struct_`：腾出空间
			- `fdt_splice_`：使用memmove移动DTB数据，移动（newlen-oldlen）字节
			- `fdt_set_size_dt_struct`：修改DTB头部的size_dt_struct
			- `fdt_set_off_dt_strings`：修改DTB头部的off_dt_strings
		- `memcpy(prop_data, val, len)`：在DTB中写入新值

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dbd8c4e12a6bc3813bb7a27c5f36bef1.png)

## 3 dtb的修改命令fdt移植

### 3.1 为什么需要移植

我们使用的是uboot 1.1.6版本，这个版本中实现了很多实用功能：
- USB下载
- 菜单操作
- 网卡永远使能等
不想丢弃这些功能，所以需要从新版本uboot中移植fdt命令

> [!tip]+ 提示
> fdt命令是在U-Boot 2007年版本中引入的，从1.3.0版本开始，高于这个版本都不需要移植fdt

### 3.2 移植步骤

**1. 获取源码和补丁**
- **uboot官网**：ftp://ftp.denx.de/pub/u-boot/
- **补丁位置**：`doc_and_sources_for_device_tree\source_and_images\u-boot\u-boot-1.1.6_device_tree_for_jz2440_add_fdt_20181022.patch`

**2. 使用补丁的方法** 
```c
# 设置交叉编译环境
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/work/system/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabi/bin

# 解压源码
tar xjf u-boot-1.1.6.tar.bz2

# 进入目录
cd u-boot-1.1.6

# 打补丁
patch -p1 < ../u-boot-1.1.6_device_tree_for_jz2440_add_fdt_20181022.patch

# 配置
make 100ask24x0_config

# 编译
make    # 编译完成后得到u-boot.bin
```

**3. 手动移植fdt命令**：
**需要移植的文件**
- 新uboot中的 `cmd/fdt.c` -> 重命名为 `cmd_fdt.c`
- `lib/libfdt/*` 目录下的文件
- `scripts/dtc/libfdt/*` 目录下的文件

**移植步骤**
1. 复制文件
	- 将上述文件复制到老uboot的 `common/fdt` 目录
2. 修改Makefile：
	- 修改老uboot根目录的Makefile，添加：`LIBS += common/fdt/libfdt.a`
	- 修改 `common/fdt/Makefile`，参考 `drivers/nand/Makefile` 的格式
3. 解决编译错误：根据编译错误信息修改源码

### 3.3 移植常见问题及解决办法

#### 3.3.1 问题1：No such file or directory

**原因分析：**
```c
#include "xxx.h"  // 在当前目录下查找xxx.h  
#include <xxx.h>  // 在指定目录下查找xxx.h（通常是include目录）
```

**解决方法：**
- 确定头文件位置
- 将头文件移到include目录或源码当前目录

#### 3.3.2 问题2：xxx undeclared

**可能原因：**
- 宏未定义
- 变量未声明/未定义
- 函数未声明/未定义

**解决方法：**
- 对于宏：添加宏定义
- 对于变量：定义变量或声明为外部变量
- 对于函数：实现函数或声明为外部函数

#### 3.3.3 问题3：undefined reference to 'xxx'

**原因：**
- 代码中使用了xxx函数，但函数没有实现（链接时出现）

**解决方法：**
- 实现该函数
- 找到函数所在文件，将文件加入工程

### 3.4 FDT命令使用示例

**在 U-Boot 环境中使用 `fdt` 命令，在Linux启动后的环境是无法使用的**

#### 3.4.1 基本操作流程

```shell
# 1. 从Flash读取DTB文件到内存
nand read.jffs2 32000000 device_tree

# 2. 告诉fdt命令DTB文件的位置
fdt addr 32000000

# 3. 查看属性值
fdt print /led pin                # 打印/led节点的pin属性

# 4. 读取属性值到环境变量
fdt get value xxx /led pin        # 取/led节点的pin属性，赋给环境变量XXX
print xxx                         # 打印环境变量XXX的值

# 5. 修改属性值
fdt set /led pin <0x00050005>     # 设置/led节点的pin属性

# 6. 验证修改结果
fdt print /len pin                # 打印/led节点的pin属性

# 7. 保存修改到Flash
nand erase device_tree            # 擦除flash分区
nand write.jffs2 32000000 device_tree # 把修改后的DTB文件写入flash分区
```

#### 3.4.2 实用技巧

1. **备份原始DTB**：修改前先备份原始设备树文件
2. **验证修改**：使用`fdt_print`命令验证修改是否成功
3. **测试启动**：修改后先测试启动，确认无误后再写入Flash

这样，我们就可以在U-Boot中灵活地查看和修改设备树，为内核传递正确的硬件信息