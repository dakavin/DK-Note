📢 在 `Linux` 系统上，“`spidev_test`” 是一个用于测试和配置 `SPI`（`Serial Peripheral Interface`）设备的命令行工具。`SPI`是一种串行通信协议，通常用于连接微控制器、传感器和其他外部设备。“`spidev_test`” 工具通常在`Linux`系统上使用，用于检查`SPI`接口的正确性以及与`SPI`设备的通信是否正常工作。

## 环境

1. 编译并传输spidev测试程序到目标板。
    
2. spidev设备节点。

初始化SDK环境，进入内核的源码目录，然后进入tools/spi目录，make.

[https://elixir.bootlin.com/linux/latest/source/tools/spi](https://elixir.bootlin.com/linux/latest/source/tools/spi)

得到 spidev_test 和 spidev_fdx两个程序。 我们可以通过ADB 进行上传。

## 执行测试

spidev_test的帮助：参考下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3da5a4c3a1e28fd9b21091bba7b14947.png)


以下是一些常见的 “spidev_test” 命令行选项和说明：

- -D 或 --device：用于指定SPI设备的路径。通常，SPI设备的路径类似于 “/dev/spidevX.Y”，其中X是SPI控制器的编号，Y是SPI设备的编号。
    
- -s 或 --speed：用于设置SPI通信的时钟速度，以Hz为单位。例如，-s 1000000 将设置SPI时钟速度为1 MHz。
    
- -b 或 --bits：用于设置每个字节的位数。SPI通信通常使用8位，但可以根据需要进行配置。
    
- -v 或 --verbose：用于启用详细的输出，以便查看SPI通信的细节。
    
- -w 或 --write：用于发送数据到SPI设备。可以使用此选项将数据发送到设备。
    
- -r 或 --read：用于从SPI设备读取数据。可以使用此选项读取来自设备的数据。
    
- -p 或 --loop：用于设置循环模式。如果启用了循环模式，“spidev_test” 将持续运行，不断发送和接收数据。
    
- -h 或 --help：用于显示帮助信息，列出可用的命令行选项和其说明。
    

## 回环测试

- 内循环：
```txt
spidev_test -v -l -I 1 -S 256
```
- l：内循环模型
- -I xx：发送数据的次数
- -S xx：每次发送的字节数

- 外循环：
	- 将SPI_DI和SPI_DO短接进行外循环测试，执行如下命令
```txt
spidev_test -v -I 1 -S 256
```


## 字节发送测试

在/dev/spidev2.0上发送0x00 0x47，然后读取两个字节：，结果如下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b6cf29e6786b186f80560ec042a39bd0.png)


其中的CPHA，CPOL配置请参考下图，来自 [http://www.gammon.com.au/spi](http://www.gammon.com.au/spi)

## 32位数据发送测试
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d1d4f49a77a4842f7cb19d08f77af4b0.png)


```txt
如果要发送32位/16位的数据，则需要先生成二进制文件，如生成32字节的随机数据：

dd if=/dev/urandom of=test_data bs=16 count=2

用hexdump来查看这个二进制文件：

# hexdump -v test_data -C
00000000  74 6a 59 3e 1e 81 73 fb  5a 3f 94 c7 d8 20 ca e9  |tjY>..s.Z?... ..|
00000010  24 2e a5 68 75 ab f7 12  af e6 c1 3d e2 d8 9a ba  |$..hu......=....|
00000020

发送：

# ./spidev_test -D /dev/spidev2.0 -b 32 -v -i test_data
spi mode: 0x0
bits per word: 32
max speed: 500000 Hz (500 kHz)
TX | 74 6A 59 3E 1E 81 73 FB 5A 3F 94 C7 D8 20 CA E9 24 2E A5 68 75 AB F7 12 AF E6 C1 3D E2 D8 9A BA | tjY>.s.Z?... .$..hu.....=...|
RX | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 | ................................|
```