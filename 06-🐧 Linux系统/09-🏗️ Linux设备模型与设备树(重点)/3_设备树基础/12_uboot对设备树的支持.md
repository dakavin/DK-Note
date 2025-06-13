
## 1 U-Boot如何传递设备树给内核

### 1.1 两种启动方式对比

U-Boot启动Linux内核时，可以选择是否使用设备树：
```bash
# 方式1：传统方式（不使用设备树）
bootm <uImage_addr>

# 方式2：设备树方式（推荐）
bootm <uImage_addr> <initrd_addr> <dtb_addr>
```

> [!info]+ 参数说明
> 
> - **bootm**：boot from memory，从内存启动系统
> - **uImage_addr**：内核镜像在内存中的地址
> - **initrd_addr**：初始内存盘地址（可选，没有时用"-"占位）
> - **dtb_addr**：设备树文件在内存中的地址

### 1.2 实际操作示例

让我们看一个完整的启动流程：
```bash
# 1. 从NAND Flash读取内核到内存
nand read.jffs2 0x30007FC0 kernel

# 2. 从NAND Flash读取设备树到内存
nand read.jffs2 0x32000000 device_tree

# 3. 启动内核（使用设备树）
bootm 0x30007FC0 - 0x32000000
```

> [!note]+ 命令解释
> 
> - `nand read.jffs2`：从NAND Flash读取数据
> - `kernel`：Flash中存储内核的分区名
> - `device_tree`：Flash中存储设备树的分区名
> - `-`：表示没有initrd（初始内存盘）

### 1.3 设备树地址传递机制

U-Boot是如何把设备树地址告诉内核的呢？这涉及到ARM的函数调用规则（ATPCS）。

> [!abstract]+ ARM函数调用规则 在ARM架构中，函数调用时：
> 
> - 第1个参数通过r0寄存器传递
> - 第2个参数通过r1寄存器传递
> - 第3个参数通过r2寄存器传递

**U-Boot的实现原理**：
```c
// U-Boot内部定义内核启动函数指针
void (*theKernel)(int zero, int arch, uint params);
theKernel = (void (*)(int, int, uint))images->ep;

// 调用内核，DTB地址通过r2传递
theKernel(0, machid, (unsigned long)images->ft_addr);
```

流程图解：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/98bf5df05c052e7ce55fa2b6550ed26c.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c05989d408c21c752f5896c45aa6b12e.png)

## 2 设备树放置位置的注意事项

### 2.1 内存布局规划

选择DTB存放地址时，必须避免以下问题：

> [!warning]+ 放置原则
> 
> 1. **不能破坏U-Boot**：避开U-Boot代码和数据区
> 2. **不能影响内核**：
>     - 避开内核镜像存放区
>     - 避开内核解压缩区域
>     - 避开内核页表区域（通常16KB）
> 3. **保证安全距离**：与其他数据保持足够间隔

### 2.2 JZ2440开发板内存布局

```txt
内存地址分布：
0x33f80000 ┌─────────────────────────────┐
           │        U-Boot代码           │
           ├─────────────────────────────┤
           │    U-Boot栈和堆            │
           ├─────────────────────────────┤
           │                             │
           │      安全的空闲区域         │ ← DTB可以放这里
           │                             │
           ├─────────────────────────────┤
0x32000000 │     设备树存放位置（推荐）   │ ← 推荐的DTB位置
           ├─────────────────────────────┤
           │                             │
           │        空闲区域             │
           │                             │
           ├─────────────────────────────┤
0x30008000 │    zImage（压缩内核）       │
           ├─────────────────────────────┤
0x30007FC0 │    uImage头部（64字节）     │
           ├─────────────────────────────┤
0x30004000 │    内核页表（16KB）         │ ← 绝对不能放DTB！
           ├─────────────────────────────┤
           │                             │
0x30000000 └─────────────────────────────┘ ← 内存起始地址
```

### 2.3 如何确定内核加载地址

使用`mkimage`命令查看内核镜像信息：

```bash
mkimage -l uImage
```

输出示例： ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9b2683af6cfa4250c3f6661c02b43ae6.png)

### 2.4 正确与错误示例

> [!success]+ ✅ 正确示例
> 
> ```bash
> # DTB放在0x32000000（安全位置）
> nand read.jffs2 0x32000000 device_tree
> nand read.jffs2 0x30007FC0 kernel
> bootm 0x30007FC0 - 0x32000000
> ```

> [!danger]+ ❌ 错误示例
> 
> ```bash
> # DTB放在0x30004000（页表位置）- 会被内核覆盖！
> nand read.jffs2 0x30004000 device_tree
> nand read.jffs2 0x30007FC0 kernel
> bootm 0x30007FC0 - 0x30004000  # 启动失败！
> ```

## 3 U-Boot中修改设备树的原理

### 3.1 修改的本质：数据搬移

U-Boot修改DTB的核心操作是**数据搬移**，主要通过`memmove`函数实现。

### 3.2 修改已有属性

假设要修改一个属性值：

- 原值长度：`len`字节
- 新值长度：`newlen`字节

**当newlen > len时的修改步骤**：
1. **腾出空间**
    - 计算需要的额外空间：`delta = newlen - len`
    - 将属性值后面的所有数据向后移动`delta`字节
2. **写入新值**
    - 在腾出的空间中写入新的属性值
3. **更新文件头**
    - 更新structure block大小：`size_dt_struct`
    - 更新string block偏移：`off_dt_strings`
    - 更新文件总大小：`totalsize`

图解过程： ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a163ace6b9864864b511c9a708f46979.png)

### 3.3 添加新属性

添加全新属性需要处理两个部分：

**步骤1：处理属性名**
- 检查string block中是否已有该属性名
- 如果没有，在string block末尾添加
- 更新`size_dt_strings`和`totalsize`

**步骤2：添加属性数据**
在目标节点的末尾添加以下结构：

```txt
┌─────────────┬──────────┐
│ TAG (4字节) │ 0x00000003│  ← FDT_PROP标志
├─────────────┼──────────┤
│ len (4字节) │ 属性值长度 │
├─────────────┼──────────┤
│nameoff(4字节)│ 名称偏移  │
├─────────────┼──────────┤
│ value       │ 属性值数据 │
└─────────────┴──────────┘
```

图解过程： ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/ce5fb484c30197c572a52d8bd8ceecb3.png)

### 3.4 FDT命令的内部实现

查看U-Boot源码`cmd/fdt.c`，了解`fdt set`命令的实现流程：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dbd8c4e12a6bc3813bb7a27c5f36bef1.png)

> [!note]+ 核心函数调用链
> 
> ```txt
> do_fdt()
> └── fdt_setprop()
>     ├── fdt_setprop_placeholder()  // 腾出空间
>     │   ├── fdt_get_property_w()   // 获取原值
>     │   └── fdt_splice_struct_()   // 调整结构
>     │       └── fdt_splice_()      // memmove搬移数据
>     └── memcpy()                   // 写入新值
> ```

## 4 FDT命令的移植与使用

### 4.1 为什么需要移植

> [!info]+ 版本说明
> 
> - U-Boot 1.1.6（2006年）没有FDT命令
> - FDT命令从U-Boot 1.3.0（2007年）开始引入
> - 我们使用的JZ2440版本基于1.1.6，包含了许多定制功能

为了保留原有功能并支持设备树，我们需要从新版本移植FDT命令。

### 4.2 移植步骤详解

#### 4.2.1 步骤1：准备工作

```bash
# 设置交叉编译环境
export PATH=$PATH:/work/system/gcc-linaro-4.9.4/bin

# 解压源码
tar xjf u-boot-1.1.6.tar.bz2
cd u-boot-1.1.6

# 应用补丁（可选）
patch -p1 < ../u-boot-1.1.6_device_tree_for_jz2440.patch
```

#### 4.2.2 步骤2：移植文件清单

需要从新版U-Boot移植的文件：

- `cmd/fdt.c` → 重命名为 `cmd_fdt.c`
- `lib/libfdt/` 目录下所有文件
- `scripts/dtc/libfdt/` 目录下的必要文件

#### 4.2.3 步骤3：修改Makefile

**根目录Makefile**：
```makefile
LIBS += common/fdt/libfdt.a
```

**common/fdt/Makefile**：
```makefile
include $(TOPDIR)/config.mk

LIB = $(obj)libfdt.a
COBJS := fdt.o fdt_ro.o fdt_rw.o fdt_strerror.o fdt_sw.o fdt_wip.o
COBJS += cmd_fdt.o

SRCS := $(COBJS:.o=.c)
OBJS := $(addprefix $(obj),$(COBJS))

all: $(LIB)

$(LIB): $(obj).depend $(OBJS)
    $(AR) $(ARFLAGS) $@ $(OBJS)

include $(SRCTREE)/rules.mk
```

### 4.3 常见移植问题及解决

> [!warning]+ 问题1：
> 头文件找不到 **错误**：`No such file or directory` **解决**：
> 
> - 将头文件复制到`include`目录
> - 或修改源码中的`#include <xxx.h>`为`#include "xxx.h"`

> [!warning]+ 问题2：
> 未定义的标识符 **错误**：`xxx undeclared` **解决**：
> 
> - 宏定义：在配置文件中添加
> - 变量/函数：找到定义文件一起移植

> [!warning]+ 问题3：
> 链接错误 **错误**：`undefined reference to 'xxx'` **解决**：找到函数实现文件，加入编译列表

### 4.4 FDT命令使用实战

#### 4.4.1 基本命令

```bash
# 查看帮助
fdt help

# 常用命令
fdt addr <addr>              # 设置DTB地址
fdt print [path] [prop]      # 打印节点或属性
fdt list [path] [prop]       # 列出节点或属性
fdt set <path> <prop> [val]  # 设置属性值
fdt mknode <path> <node>     # 创建节点
fdt rm <path> [prop]         # 删除节点或属性
```

#### 4.4.2 实际操作流程

```bash
# 1. 加载设备树到内存
nand read.jffs2 0x32000000 device_tree

# 2. 设置FDT操作地址
fdt addr 0x32000000

# 3. 查看设备树结构
fdt print /                  # 打印整个设备树
fdt list /soc               # 列出soc节点

# 4. 查看具体属性
fdt print /led pin          # 查看LED引脚配置

# 5. 修改属性值
fdt set /led pin <0x00050005>  # 修改LED引脚

# 6. 创建新节点
fdt mknode / mydevice       # 在根节点下创建mydevice
fdt set /mydevice compatible "my,device"

# 7. 保存修改
nand erase device_tree
nand write.jffs2 0x32000000 device_tree
```

#### 4.4.3 高级技巧

> [!tip]+ 使用环境变量
> 
> ```bash
> # 读取属性到环境变量
> fdt get value ledpin /led pin
> print ledpin
> 
> # 使用环境变量设置属性
> setenv newpin "<0x00060006>"
> fdt set /led pin ${newpin}
> ```

> [!tip]+ 批量修改
> 
> ```bash
> # 创建修改脚本
> setenv update_fdt 'fdt addr 0x32000000; fdt set /led pin <5 5>; fdt set /led compatible "gpio-leds"'
> 
> # 执行批量修改
> run update_fdt
> ```

### 4.5 调试技巧

1. **验证修改**：每次修改后使用`fdt print`确认
2. **备份原始DTB**：修改前先备份到另一个地址
3. **测试启动**：修改后先测试能否正常启动
4. **查看启动日志**：通过串口查看内核是否正确解析设备树

## 5 总结

> [!abstract]+ 本章要点
> 
> 1. **传递机制**：U-Boot通过r2寄存器将DTB地址传给内核
> 2. **内存布局**：合理规划DTB存放位置，避免被覆盖
> 3. **修改原理**：通过数据搬移实现动态修改
> 4. **FDT命令**：提供了强大的设备树操作能力
> 5. **实际应用**：可以在U-Boot阶段灵活调整硬件配置
> 
> 掌握U-Boot的设备树支持，让我们能够：
> 
> - 在启动阶段动态调整硬件配置
> - 调试设备树相关问题
> - 适配不同的硬件版本
> - 实现更灵活的系统启动流程