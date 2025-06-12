
## 1 DTB文件基本概念

### 1.1 什么是DTB文件

在设备树技术中，我们会接触到三种文件格式：

- **DTS文件**：Device Tree Source，设备树源文件（`.dts`）
- **DTSI文件**：Device Tree Source Include，设备树包含文件（`.dtsi`）
- **DTB文件**：Device Tree Blob，设备树二进制文件（`.dtb`）

> [!info]+ 简单理解 
> 把DTS想象成我们能看懂的"说明书"，而DTB就是机器能理解的"指令码"。就像C语言源码需要编译成机器码一样，DTS也需要编译成DTB才能被系统使用。

### 1.2 为什么需要DTB格式

**DTS文件的特点**：
- 人类可读的文本格式
- 便于编写和维护
- 占用空间较大，解析速度慢

**DTB文件的特点**：
- 二进制格式，机器可直接识别
- 占用空间小，加载速度快
- U-Boot和Linux内核只能识别DTB格式

> [!note]+ 核心理解 
> DTB是DTS的二进制版本，采用分段式布局，既保持了结构的清晰性，又优化了存储效率和加载速度。

### 1.3 DTC编译工具

**DTC（Device Tree Compiler）** 是设备树编译器，用于在DTS和DTB之间进行转换。

#### 1.3.1 获取DTC工具

**方式一：使用Linux源码中的DTC**
- `内核源码/scripts/dtc/`

**方式二：独立安装DTC**
```bash
sudo apt-get install device-tree-compiler
```

#### 1.3.2 DTC使用方法

**编译：DTS → DTB**
```bash
dtc -I dts -O dtb -o xxx.dtb xxx.dts
```

**反编译：DTB → DTS**
```bash
dtc -I dtb -O dts -o xxx.dts xxx.dtb
```

> [!tip]+ 参数说明
> 
> - `-I`：指定输入格式（Input format）
> - `-O`：指定输出格式（Output format）
> - `-o`：指定输出文件名（Output file）

> [!example]+ 实际应用场景 
> 当我们修改了 DTS 文件后，需要重新编译成 DTB 文件，然后烧录到设备中。反之，当我们想查看设备上实际使用的设备树配置时，可以将 DTB 反编译成 DTS 来分析。

## 2 DTB文件的整体布局

DTB文件采用精心设计的分段式存储结构：

```txt
             ------------------------------
     base -> |  struct boot_param_header  |  ← 文件头（目录信息）
             ------------------------------
             |      (alignment gap)       |  ← 对齐填充
             ------------------------------
             |    memory reserve map      |  ← 内存保留映射表
             ------------------------------
             |      (alignment gap)       |  ← 对齐填充
             ------------------------------
             |                            |
             |  device-tree structure     |  ← 设备树结构数据（核心内容）
             |                            |
             ------------------------------
             |      (alignment gap)       |  ← 对齐填充
             ------------------------------
             |                            |
             |   device-tree strings      |  ← 字符串存储区
             |                            |
      -----> ------------------------------
      |
      |
      --- (base + totalsize)
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6067f6cd4732cdddb960e8b329e0ffe4.png)

> [!abstract]+ 四个主要区域
> 
> 1. **header**：文件头，包含基本信息和各区域的偏移地址
> 2. **memory reserve map**：内存保留映射表
> 3. **device-tree structure**：设备树结构数据，存储节点和属性
> 4. **device-tree strings**：字符串存储区，集中存储所有字符串

## 3 字节序说明

> [!warning]+ 重要：DTB使用大端序 
> DTB文件采用**大端序（Big Endian）** 存储数据：
> 
> - 大端序：高位字节存储在低地址（0x12345678 → 12 34 56 78）
> - 小端序：低位字节存储在低地址（0x12345678 → 78 56 34 12）

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d3f7fb1d21385a72fabd1730d80a669b.png)

**注意**：大小端序只影响数字的存储，对字符串没有影响。字符串"abc"中，'a'永远存储在低地址。

## 4 文件头结构（boot_param_header）

### 4.1 文件头的作用

文件头就像DTB文件的"目录"，包含了整个文件的基本信息。

使用`fdtdump`工具可以查看文件头信息：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0c4663069f857c9c1d1d308a882105e9.png)

### 4.2 文件头结构体定义

文件头使用`struct fdt_header`结构体描述（定义在`scripts/dtc/libfdt/fdt.h`）：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/44c1ba8d64089dd743c3740d20c1e1f2.png)

主要字段说明：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0e23b82b37005acbe591e4ce5c63eb29.png)

> [!note]+ 文件头的重要性 文件头告诉系统：
> 
> - 文件的总大小
> - 各个区域在文件中的位置
> - 文件的版本信息
> - 数据的完整性（通过magic number验证）

## 5 内存保留映射表

**memory reserve map** 记录了哪些内存区域需要被保留，不能被内核随意使用。

常见的保留内存包括：
- Bootloader占用的内存
- 设备固件占用的内存
- 其他重要数据占用的内存

## 6 设备树结构数据区

### 6.1 数据组织方式

这是DTB文件的**核心部分**，包含了所有的设备节点和属性信息。数据通过特定的标志位（Tag）来组织：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/23d03d538d12106b51323f559d2de7bf.png)

### 6.2 标志位定义

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/668871cb8a400eaed200d9981cafa0c5.png)

**标志位含义**：
- `0x00000001`（FDT_BEGIN_NODE）：节点开始，后接节点名称
- `0x00000002`（FDT_END_NODE）：节点结束
- `0x00000003`（FDT_PROP）：属性开始
- `0x00000009`（FDT_END）：整个设备树结束

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ff89d9fcb68a9ee7242f6c1984e3709d.png)

> [!example]+ 解析过程示例 系统解析DTB时，就像读书一样：
> 
> - 遇到`FDT_BEGIN_NODE`："开始读一个新设备"
> - 遇到`FDT_PROP`："开始读设备属性"
> - 遇到`FDT_END_NODE`："这个设备读完了"
> - 遇到`FDT_END`："整个文件读完了"

### 6.3 节点信息结构

节点使用`struct fdt_node_header`描述：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c200e5bce1939c9fcdc5ce642853d011.png)

- **tag**：标识节点类型（FDT_BEGIN_NODE）
- **name**：节点名称的首地址

### 6.4 属性信息结构

属性使用`struct fdt_property`描述：
- **tag**：标识这是属性（FDT_PROP）
- **len**：属性值的长度（包括'\0'）
- **nameoff**：属性名在字符串区的偏移量

> [!tip]+ 存储优化设计 属性名不直接存储在属性结构体中，而是通过偏移量指向字符串区。这样设计的好处：
> 
> 1. **节省空间**：相同的属性名只存储一次
> 2. **提高效率**：通过偏移量快速定位
> 3. **便于管理**：所有字符串集中存储

## 7 字符串存储区

为了节省空间，DTB把所有的字符串（属性名、节点名等）集中存放在字符串区，其他地方只需要引用这些字符串的偏移位置。

这种设计类似于"字符串池"的概念，避免了重复存储相同的字符串。

## 8 对齐填充

各个区域之间可能会有对齐填充（alignment gap），目的是：
- 确保数据按照特定字节边界对齐（通常是4或8字节）
- 提高内存访问效率
- 符合处理器的内存访问要求

## 9 DTB文件的解析流程

### 9.1 系统启动时的解析过程

1. **读取文件头**：获取magic number验证文件有效性，读取各区域位置
2. **验证完整性**：检查版本信息、文件大小等
3. **解析结构区**：按照标志位逐个解析节点和属性
4. **建立内存结构**：将DTB信息转换为内核可用的数据结构

### 9.2 解析优化

> [!success]+ 高效的解析设计
> 
> - 通过标志位引导，可以快速跳过不需要的信息
> - 字符串集中存储，减少内存碎片
> - 二进制格式，无需复杂的文本解析

## 10 实用工具

### 10.1 查看DTB内容

```bash
# 使用fdtdump查看DTB信息
fdtdump xxx.dtb

# 反编译DTB查看源码
dtc -I dtb -O dts -o output.dts input.dtb
```

### 10.2 验证DTB文件

```bash
# 编译时进行语法检查
dtc -I dts -O dtb -o test.dtb test.dts

# 查看文件头信息验证
hexdump -C test.dtb | head -20
```

## 11 常见问题

> [!question]+ 为什么采用大端序？ 
> 设备树标准源于PowerPC架构，该架构使用大端序。为了保持兼容性，DTB沿用了大端序格式。

> [!question]+ DTB文件损坏的后果？
> 
> - 轻则导致某些设备无法识别
> - 重则导致系统无法启动
> - 文件头的magic number可以帮助检测文件是否损坏

> [!question]+ 如何优化DTB文件大小？
> 
> - 合理使用DTSI文件，避免重复定义
> - 删除未使用的节点和属性
> - 使用引用（&label）而不是重复定义

## 12 总结

> [!abstract]+ DTB格式核心要点
> 
> 1. **二进制格式**：相比DTS文本格式，加载速度快，占用空间小
> 2. **分段存储**：文件头、内存保留表、结构数据、字符串区各司其职
> 3. **大端字节序**：数值采用大端序存储，需要注意转换
> 4. **标志位导航**：通过Tag标志位组织数据，便于快速解析
> 5. **字符串池优化**：集中存储字符串，通过偏移引用，节省空间
> 
> 理解DTB文件格式有助于：
> 
> - 调试设备树相关问题
> - 优化系统启动性能
> - 开发设备树相关工具