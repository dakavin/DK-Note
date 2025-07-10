
## 1 通用学习框架

**1、相关概念（理论基础）**
- 子系统的作用和定位
- 硬件工作原理
- Linux内核中的抽象层次

**2、数据结构体和API（编程接口）**
- 核心数据结构
- 注册/注销API
- 操作API（读写、配置等）
- 回调函数接口

**3、控制器和设备驱动流程（架构核心）**
- **控制器驱动**：管理硬件控制器
- **设备驱动**：操作具体设备
- **子系统核心**：提供统一接口

**4、实践应用（动手实现）**
- 设备树配置
- 驱动代码编写
- 用户空间接口

**5、调试手段（问题解决）**
- 内核调试工具
- sysfs接口查看
- 常见问题排查

## 2 Linux驱动的经典分层架构

```txt
用户空间应用
     ↕
系统调用接口
     ↕
子系统核心层 (I2C Core/SPI Core/IRQ Core)
     ↕
控制器驱动 (Controller Driver)
     ↕
硬件控制器 (I2C Controller/SPI Controller/GIC)
     ↕
具体设备 (EEPROM/Flash/外设)
```

## 3 以I2C子系统为例预览

### 3.1 相关概念

```txt
- I2C总线协议（主从、时钟、数据线）
- 适配器(Adapter)和客户端(Client)概念
- 设备地址和寻址方式
```

### 3.2 核心数据结构

```c
struct i2c_adapter    // I2C控制器
struct i2c_client     // I2C设备
struct i2c_driver     // I2C设备驱动
struct i2c_msg        // I2C消息
```

### 3.3 驱动架构

```txt
I2C控制器驱动 ←→ I2C Core ←→ I2C设备驱动
(platform driver)     (子系统核心)   (i2c_driver)
```

### 3.4 实践应用

```dts
// 设备树
i2c1: i2c@40005400 {
    eeprom@50 {
        compatible = "atmel,24c02";
        reg = <0x50>;
    };
};
```

### 3.5 调试手段

```bash
# sysfs接口
ls /sys/bus/i2c/devices/
i2cdetect -y 1
```

## 4 SPI子系统预览

### 4.1 架构对比

```txt
I2C: Adapter ←→ Client
SPI: Master ←→ Device
中断: Controller ←→ IRQ Domain
```

### 4.2 共同特点

- 都有**控制器驱动**和**设备驱动**分离
- 都有**子系统核心**提供统一接口
- 都支持**设备树自动匹配**
- 都有**标准的注册注销流程**

## 5 学习顺序建议

```txt
1. GPIO (基础) → 理解pinctrl配合使用
2. 中断 (核心) → 理解硬件中断处理
3. I2C (通用) → 理解总线子系统
4. SPI (类似) → 巩固总线概念  
5. 高级主题 → DMA、时钟、电源管理
```

## 6 每个子系统的核心问题

|子系统|核心问题|关键概念|
|---|---|---|
|**GPIO**|如何控制引脚？|pinctrl配合、方向控制|
|**中断**|如何响应事件？|中断控制器、中断域|
|**I2C**|如何总线通信？|适配器、客户端|
|**SPI**|如何高速传输？|主从模式、时钟相位|

## 7 通用代码模板

```c
// 每个子系统都有类似的结构
static int xxx_probe(struct platform_device *pdev)
{
    // 1. 获取资源 (内存、中断、时钟等)
    // 2. 初始化硬件
    // 3. 注册到子系统
    // 4. 创建设备节点
}

static int xxx_remove(struct platform_device *pdev)
{
    // 1. 注销设备
    // 2. 释放资源  
    // 3. 清理工作
}
```
