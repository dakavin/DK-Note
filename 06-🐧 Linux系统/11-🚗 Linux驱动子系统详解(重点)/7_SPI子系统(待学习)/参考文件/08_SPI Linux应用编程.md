## 1 ioctl函数

在编写应用程序时需要使用ioctl函数设置spi相关配置，其函数原型如下

```C
 #include <sys/ioctl.h>

 int ioctl(int fd, unsigned long request, ...);
```

当使用ioctl函数进行SPI通信时，常用的request参数主要有以下几种：

- `SPI_IOC_RD_MODE`：用于读取当前SPI通信的模式设置。这个请求参数将模式信息读取到一个整数变量中，以便检查当前SPI通信的极性和相位设置。
    
- `SPI_IOC_WR_MODE`：用于设置SPI通信的模式。您需要提供一个整数值，该值通常由两位二进制数字组成，表示SPI通信的极性和相位。
    
- `SPI_IOC_RD_BITS_PER_WORD`：用于读取每个数据字的位数。这个请求参数将位数信息读取到一个整数变量中。
    
- `SPI_IOC_WR_BITS_PER_WORD`：用于设置每个数据字的位数。您需要提供一个整数值，以指定要发送和接收的每个数据字的位数。
    
- `SPI_IOC_RD_MAX_SPEED_HZ`：用于读取SPI总线的最大速度。这个请求参数将速度信息读取到一个整数变量中，以便检查当前SPI总线的最大传输速度。
    
- `SPI_IOC_WR_MAX_SPEED_HZ`：用于设置SPI总线的最大速度。您需要提供一个整数值，以指定要使用的最大传输速度。
    
- `SPI_IOC_MESSAGE(N)`：用于执行SPI传输的读写操作。这个请求参数需要一个指向 `struct spi_ioc_transfer` 数组的指针，每个元素描述了一个SPI传输操作，可以执行多个操作。
    
- `SPI_IOC_RD_LSB_FIRST`：用于读取LSB（Least Significant Bit）优先设置。这个请求参数将LSB优先信息读取到一个整数变量中。
    
- `SPI_IOC_WR_LSB_FIRST`：用于设置LSB优先设置。您需要提供一个整数值，以指定是否要使用LSB优先模式。
    

  

## 2 示例程序

通过以下程序，可以实现SPI通讯。

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <linux/spi/spidev.h>
#include <sys/ioctl.h>

#define SPI_DEVICE_PATH "/dev/spidev0.0"

int main() {
    int spi_file;
    uint8_t tx_buffer[50] = "hello world!";
    uint8_t rx_buffer[50];

    // Open the SPI device
    if ((spi_file = open(SPI_DEVICE_PATH, O_RDWR)) < 0) {
        perror("Failed to open SPI device");
        return -1;
    }

    // Configure SPI mode and bits per word
    uint8_t mode = SPI_MODE_0;
    uint8_t bits = 8;
    if (ioctl(spi_file, SPI_IOC_WR_MODE, &mode) < 0) {
        perror("Failed to set SPI mode");
        close(spi_file);
        return -1;
    }
    if (ioctl(spi_file, SPI_IOC_WR_BITS_PER_WORD, &bits) < 0) {
        perror("Failed to set SPI bits per word");
        close(spi_file);
        return -1;
    }

    // Perform SPI transfer
    struct spi_ioc_transfer transfer = {
        .tx_buf = (unsigned long)tx_buffer,
        .rx_buf = (unsigned long)rx_buffer,
        .len = sizeof(tx_buffer),
        .delay_usecs = 0,
        .speed_hz = 1000000,  // SPI speed in Hz
        .bits_per_word = 8,
    };

    if (ioctl(spi_file, SPI_IOC_MESSAGE(1), &transfer) < 0) {
        perror("Failed to perform SPI transfer");
        close(spi_file);
        return -1;
    }

     /* Print tx_buffer and rx_buffer*/
    printf("\rtx_buffer: \n %s\n ", tx_buffer);
    printf("\rrx_buffer: \n %s\n ", rx_buffer);

    // Close the SPI device
    close(spi_file);

    return 0;
}
```

  

## 3 文件路径

这行代码定义了一个宏，用于存储 SPI 设备文件的路径。

```C
#define SPI_DEVICE_PATH "/dev/spidev0.0"
```

  

## 4 打开SPI设备

这部分代码尝试打开指定的 SPI 设备文件。

```C
// Open the SPI device
if ((spi_file = open(SPI_DEVICE_PATH, O_RDWR)) < 0) {
    perror("Failed to open SPI device");
    return -1;
}
```

## 5 配置SPI

这段代码用于配置 SPI 通信的模式为 SPI 模式0（时钟极性为0、时钟相位为0）以及每个字的位数为8位，以确保 SPI 通信的正确性和一致性。

```C
// Configure SPI mode and bits per word
uint8_t mode = SPI_MODE_0;
uint8_t bits = 8;
if (ioctl(spi_file, SPI_IOC_WR_MODE, &mode) < 0) {
    perror("Failed to set SPI mode");
    close(spi_file);
    return -1;
}
if (ioctl(spi_file, SPI_IOC_WR_BITS_PER_WORD, &bits) < 0) {
    perror("Failed to set SPI bits per word");
    close(spi_file);
    return -1;
}
```

这段代码定义了一个 spi_ioc_transfer 结构体变量 transfer，用于配置 SPI 传输的参数。它指定了传输的数据缓冲区、传输的长度、延迟时间、SPI 速度（以赫兹为单位）以及每个字的位数。这个结构体将被传递给 `SPI_IOC_MESSAGE ioctl` 函数，以执行 SPI 传输。

```C
// Perform SPI transfer
struct spi_ioc_transfer transfer = {
    .tx_buf = (unsigned long)tx_buffer,
    .rx_buf = (unsigned long)rx_buffer,
    .len = sizeof(tx_buffer),
    .delay_usecs = 0,
    .speed_hz = 1000000,  // SPI speed in Hz
    .bits_per_word = 8,
};
```

## 6 数据收发

这段代码使用 `ioctl` 函数执行 SPI 传输。它通过 SPI_IOC_MESSAGE(1) 宏指定传输的数量为1，第三个参数为上文中配置的结构体变量 transfer 的地址。如果 SPI 传输失败，将会输出错误消息并关闭 SPI 设备文件描述符。

```C
if (ioctl(spi_file, SPI_IOC_MESSAGE(1), &transfer) < 0) {
    perror("Failed to perform SPI transfer");
    close(spi_file);
    return -1;
}
```

## 7 测试

将SPI的MOSI和MISO相接，运行程序：结果如下图
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6b0b29ca0f0b927df1f88f108057cea2.png)
