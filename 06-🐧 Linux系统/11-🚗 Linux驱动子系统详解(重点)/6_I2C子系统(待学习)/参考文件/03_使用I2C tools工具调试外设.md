
当你拿到开发板或者是从公司的硬件同事拿到一个带有`I2C`外设的板子，我们应该如何最快速的使用起来这个`I2C`设备呢？既然我们总是说这个`I2C`总线在嵌入式开发中被广泛的使用，那么是否有现成的测试工具帮我们完成这个快速使用板子的`I2C`设备呢？

答案是有的，而且这个测试工具的代码还是开源的，它被广泛的应用在`linux`应用层来快速验证`I2C`外设是否可用，为我们测试`I2C`设备提供了很好的捷径。


## I2C tools概述：

`I2C tools`包含一套用于`Linux`应用层测试各种各样`I2C`功能的工具。它的主要功能包括：总线探测工具、`SMBus`访问帮助程序、`EEPROM`解码脚本、`EEPROM`编程工具和用于`SMBus`访问的`python`模块。只要你所使用的内核中包含`I2C`设备驱动，那么就可以在你的板子中正常使用这个测试工具。

## 下载I2C tools源码：

前面我们已经说过了这个`I2C tools`工具是开源的，那么这个源码在哪里可以找到呢？

- **下载方法一**：直接在内核的网站`[i2c-tools](https://mirrors.edge.kernel.org/pub/software/utils/i2c-tools/)`下载`I2C tools`代码的压缩包
    
- **下载方法二**：利用`git`管理工具下载这个`I2C tools`的源代码，命令为`git clone` `git://git.kernel.org/pub/scm/utils/i2c-tools/i2c-tools.git`
    

强烈建议读者采用第二种方法下载这个代码，因为你可以通过`git`快速地了解这个开源代码的不同版本的功能改进及`bug`修复，而且使用`git`开发也是作为一名优秀的开发人员必备的一项技能。


## 编译I2C tools源码：

进入刚才利用`git`下载好的`i2c-tools`源码目录，修改编译工具为你当前使用的交叉编译工具：

```C
26         CC  ?= aarch64-linux-gnu-gcc
27         AR  ?= aarch64-linux-gnu-ar
```

编译源码：如果你想编译静态版本，你可以输入命令：`make USE_STATIC_LIB=1`；如果使用动态库的话，可以直接输入`make`进行编译。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f4f8a610d16f2ad3901653723859df32.png)


1. 将`tools`目录下的`5`个可执行文件`i2cdetect`，`i2cdump`，`i2cget`，`i2cset`和`i2ctransfer`复制到板子的`/usr/sbin/`中；
    
2. 若`adb push` 传输失败：`Read-only file system`，说明系统是只读权限，需要修改权限：`mount -o remount,rw /`
    
3. `chmod 777 i2cdetect`…文件修改为可执行的权限。
    
4. 将`lib`目录下的`libi2c.so.0.1.1`文件复制到板子的`/usr/lib/libi2c.so.0`
    
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/77638256ca7210f7e613d2281447b267.png)

## i2cdetect

`i2cdetect`的主要功能就是`I2C`设备查询，它用于扫描`I2C`总线上的设备。它输出一个表，其中包含指定总线上检测到的设备的列表。 该命令的常用格式为：`i2cdetect [-y] [-a] [-q|-r] i2cbus [first last]`。具体参数的含义如下：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/45a9bee32bf07292249019aaa464b7c5.png)


|   |   |
|---|---|
|参数|含义|
|-y|取消交互模式。默认情况下，i2cdetect将等待用户的确认，当使用此标志时，它将直接执行操作。|
|-a|强制扫描非规则地址。一般不推荐。|
|-q|使用SMBus“快速写入”命令进行探测。一般不推荐。|
|-r|使用SMBus“接收字节”命令进行探测。一般不推荐。|
|-F|显示适配器实现的功能列表并退出。|
|-V|显示I2C工具的版本并推出。|
|-l|显示已经在系统中使用的I2C总线。|
|i2cbus|表示要扫描的I2C总线的编号或名称。|
|first last|表示要扫描的从设备地址范围。|
|该功能的常用方式：||

- **第一**，先通过`i2cdetect -l`查看当前系统中的`I2C`的总线情况：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/47b91b95295fedbe2dab4d4ad327bb3c.png)


- **第二**，若总线上挂载`I2C`从设备，可通过`i2cdetect`扫描某个`I2C`总线上的所有设备。可通过控制台输入`i2cdetect -r -y 1`：（其中"`--`"表示地址被探测到了，但没有芯片应答； "`UU`"因为这个地址目前正在被一个驱动程序使用，探测被省略；
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6df0e087419406d6f99f67c66715329a.png)


- **第三**，查询`I2C`总线`2` (`I2C -2`)的功能，命令为`i2cdetect -F 2`：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/cb0cea81ebc9ac015b98700ad9b27e13.png)

## i2cget

`i2cget`的主要功能是获取`I2C`外设某一寄存器的内容。 该命令的常用格式为：`i2cget [-f] [-y] [-a] i2cbus chip-address [data-address [mode]]`。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7bb31096423ee724b30086b2aaee7fe6.png)


具体参数的含义如下：

|   |   |
|---|---|
|参数|含义|
|-f|强制访问设备，即使它已经很忙。默认情况下，i2cget将拒绝访问已经在内核驱动程序控制下的设备。|
|-y|取消交互模式。默认情况下，i2cdetect将等待用户的确认，当使用此标志时，它将直接执行操作。|
|-a|取消交互模式。默认情况下，i2cdetect将等待用户的确认，当使用此标志时，它将直接执行操作。|
|i2cbus|表示要扫描的I2C总线的编号或名称。这个数字应该与i2cdetect -l列出的总线之一相对应。|
|chip-address|要操作的外设从地址。|
|data-address|被查看外设的寄存器地址。|
|mode|显示数据的方式：b (read byte data, default)、w (read word data)、c (write byte/read byte)|

下面是完成读取0总线上从地址为0x50的外设的0x10寄存器的数据，命令为：

```C
i2cget -y -f 2 0x18 0x10
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bad335146075621b39c1db201a621d7b.png)


## i2cdump

`i2cdump`的主要功能查看`I2C`从设备器件所有寄存器的值。 该命令的常用格式为：`i2cdump [-f] [-r first-last] [-y] [-a] i2cbus address [mode [bank [bankreg]]]`。

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/aef697da727afb67a7c84534614a61a0.png)


具体参数的含义如下：

|   |   |
|---|---|
|参数|含义|
|-f|强制访问设备，即使它已经很忙。默认情况下，i2cget将拒绝访问已经在内核驱动程序控制下的设备。|
|-r|限制正在访问的寄存器范围。 此选项仅在模式b，w，c和W中可用。对于模式W，first必须是偶数，last必须是奇数。|
|-y|取消交互模式。默认情况下，i2cdetect将等待用户的确认，当使用此标志时，它将直接执行操作。|
|-V|显示I2C工具的版本并推出。|
|i2cbus|表示要扫描的I2C总线的编号或名称。这个数字应该对应于i2cdetect -l列出的总线之一。|
|first last|表示要扫描的从设备地址范围。|
|mode|b: 单个字节、w：16位字、s：SMBus模块、i：I2C模块的读取大小、c: 连续读取所有字节，对于具有地址自动递增功能的芯片（如EEP​​ROM）非常有用、W与w类似，只是读命令只能在偶数寄存器地址上发出;这也是主要用于EEPROM的。|

下面是完成读取`2`总线上从地址为`0x18`的数据，命令为：`i2cdump -f -y 2 0x18`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/99e3bfc77993f03232b7f75cbaabe77d.png)

### 7、i2cset

`i2cset`的主要功能是通过`I2C`总线设置设备中某寄存器的值。该命令的常用格式为： `i2cset [-f] [-y] [-m mask] [-r] i2cbus chip-address data-address [value] ...[mode]`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/313d0d58a9f558ee51e2f088a8754cc6.png)


具体参数的含义同`i2cdump` 下面是完成向`2`总线上从地址为`0x18`的`0x10`寄存器写入`0x55`，然后用`i2cget`读取确认。
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/990880e868b177abca8ab5bda650d8e8.png)


## i2ctransfer

`i2ctransfer`的主要功能是在一次传输中发送用户定义的`I2C`消息。`i2ctransfer`是一个创建`I2C`消息并将其合并为一个传输发送的程序。对于读消息，接收缓冲区的内容被打印到`stdout`，每个读消息一行。 该命令的常用格式为：`i2ctransfer [-f] [-y] [-v] [-a] i2cbus desc [data] [desc [data]]`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/62001e5e0aa9900b6caabb47d42e7fb8.png)


具体参数的含义如下：

|   |   |
|---|---|
|参数|含义|
|-f|强制访问设备，即使它已经很忙。|
|默认情况下，i2cget将拒绝访问已经在内核驱动程序控制下的设备。||
|-y|取消交互模式。默认情况下，i2cdetect将等待用户的确认，当使用此标志时，它将直接执行操作。|
|-v|启用详细输出。它将打印所有信息发送，即不仅为读消息，也为写消息。|
|-V|显示I2C工具的版本并推出。|
|-a|允许在0x00 - 0x02和0x78 - 0x7f之间使用地址。一般不推荐。|
|i2cbus|表示要扫描的I2C总线的编号或名称。这个数字应该对应于i2cdetect -l列出的总线之一。|

下面是完成向`2`总线上从地址为`0x18`的`0x10`开始的`4`个寄存器写入`0x01，0x02，0x03，0x04` 命令为：`i2ctransfer -f -y 2 w5@0x18 0x10 0x01 0x02 0x03 0x04`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6b86d4e2cef3e58399b6e11aa427bd60.png)


然后再通过命令`i2ctransfer -f -y 2 w1@0x18 0x10 r4`将`0x10`地址的`4`个寄存器数据读出来，见下图：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9abdc5ce57cd8bd22c4f3dd4ba84f039.png)
