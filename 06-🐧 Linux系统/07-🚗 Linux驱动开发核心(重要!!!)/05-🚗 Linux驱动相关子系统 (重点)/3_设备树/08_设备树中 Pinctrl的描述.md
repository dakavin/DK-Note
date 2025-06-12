---
文章标题: "[[08_设备树中 Pinctrl的描述]]"
文章作者: Dakkk
文章概要: |
  介绍Linux设备树中Pinctrl节点的作用、配置方法和引用机制，详细说明了IOMUX、PINCONF等配置类型以及多状态支持的实现方式。
tags:
  - 设备树
  - Pinctrl
  - 引脚控制
  - IOMUX
  - PINCONF
  - GPIO
  - RK芯片
  - 嵌入式系统
相关文章:
  - "[[07_设备树中 GPIO 相关属性 (定义)]]"
  - "[[02_设备树（device Tree）的由来]]"
  - "[[01_Linux系统构成简单介绍]]"
  - "[[03_图解Kernel Device Tree(设备树)的使用]]"
  - "[[04_设备树中CPU描述 (不需要改)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/3_设备树/1_Linux之设备树详解（训练营）/07_设备树中 Pinctrl的描述.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-11 14:47:16
---

## 1 什么是 Pinctrl 节点

**pinctrl 节点**：设备树（Device Tree）中专门用来描述硬件平台**引脚控制（Pin Control）** 相关信息的特殊节点。

> [!info]+ 通俗理解 
> 把 pinctrl 想象成一个"引脚管理员"，它负责告诉系统每个引脚应该做什么工作，比如某个引脚是用来发送串口数据，还是用来控制 GPIO。

## 2 DTS 中的 Pinctrl 介绍

### 2.1 Pinctrl 节点的位置

在 RK 芯片的设备树中，**pinctrl 节点**通常放在 `soc.dtsi` 文件里（比如 `rk3588s.dtsi` 或 `rk3568.dtsi`），一般位于文件的最后一个节点
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6f4afd076e6caf61bff95e31368d13eb.png)

### 2.2 Pinctrl 节点的特殊性

从上图可以看出，**pinctrl 节点有几个特点**：
1. **没有 reg 属性**：它不是一个标准的 platform device
2. **通过 rockchip,grf=<&grf> 传入寄存器基地址**：换句话说，它不直接管理寄存器，而是通过引用其他节点获取地址
3. **驱动内部计算实际地址**：基地址 + 偏移地址 = 最终的寄存器地址

> [!note]+ 三种配置类型
> 
> - **IOMUX**：引脚多功能复用，简单来说就是切换引脚功能（比如同一个引脚既可以做 UART 也可以做 GPIO）
> - **PINCONF**：引脚的电气属性及行为配置
> - **GPIO**：使用单独的 gpio 节点的 reg 地址

### 2.3 PINCONF 的常见配置项

**PINCONF** 主要包括以下配置：
- **上下拉电阻**：控制引脚的默认电平状态
- **驱动能力**：控制输出信号的强弱
- **输出类型**：推挽模式或开漏模式
- **输入/输出使能**：决定引脚是输入还是输出
- **输入延迟、滤波**：消除信号干扰
- **中断触发类型**：边沿触发或电平触发
- **Schmitt 触发**：增强信号稳定性
- **保留模式（HIZ）**：高阻态模式

### 2.4 引用方式的 Pinctrl 配置

除了直接在主 dtsi 文件中定义，还可以通过引用的方式配置 pinctrl

在 `rk3568-pinctrl.dtsi` 文件中可以看到这种用法：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8493d1260d0ef63767574ebcab3d58db.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e6067bf4638c7bb820905a99053b0a75.png)

> [!tip]+ 为什么要这样设计 
> 这种分离式设计让代码更清晰：主 dtsi 定义基本框架，pinctrl.dtsi 定义具体的引脚配置，便于维护和复用。

## 3 新建 pinctrl 配置

### 3.1 是否需要新建

一般情况下，**不需要手动创建新的 pinctrl 配置**。

因为 `rk3568-pinctrl.dtsi` 文件已经枚举了 rk3568 芯片所有 `iomux` 的实例，各个模块通常直接引用即可。

> [!warning]+ 注意事项 
> 只有在现有配置无法满足需求时，才需要新建 pinctrl 配置。

### 3.2 新建规则

如果确实需要创建新的 `iomux` 实例，必须遵循以下规则：

**规则 1：位置要求**
- 必须在 **pinctrl 节点下** 创建

**规则 2：命名格式**
- 必须以 **function + group** 的形式添加

**规则 3：代码格式**
```d
function {
	group {
		rockchip,pin = <bank gpio func &ref>;
	};
};
```

> [!example]+ 格式说明
> 
> - **function**：功能名称（如 uart、spi、i2c 等）
> - **group**：组名称（如 m0、m1 表示不同的复用组合）
> - **bank**：GPIO 组编号
> - **gpio**：具体的 GPIO 编号
> - **func**：功能编号
> - **&ref**：引用的其他节点

**规则 4：基本规范**
- 遵循设备树的其他基本规则（如命名规范、语法要求等）

## 4 引用 pinctrl

### 4.1 引用机制

模块引用 **pinctrl** 是通过两个关键属性实现的：
- **pinctrl-names**：定义状态名称
- **pinctrl-0**：指定具体的 pinctrl 组

> [!info]+ 工作原理 
> 这就像给每个设备分配一个"引脚使用说明书"，告诉系统这个设备**需要用哪些引脚**，以及**如何配置这些引脚**。

### 4.2 单组引用示例

**UART2 的配置示例**：
```c
uart2: serial@fe660000 {
	compatible = "rockchip,rk3568-uart", "snps,dw-apb-uart";
	reg = <0x0 0xfe660000 0x0 0x100>;
	interrupts = <GIC_SPI 118 IRQ_TYPE_LEVEL_HIGH>;
	clocks = <&cru SCLK_UART2>, <&cru PCLK_UART2>;
	clock-names = "baudclk", "apb_pclk";
	reg-shift = <2>;
	reg-io-width = <4>;
	dmas = <&dmac0 4>, <&dmac0 5>;
	pinctrl-names = "default";
	pinctrl-0 = <&uart2m0_xfer>;
	status = "disabled";
};
```

在这个例子中，`uart2m0_xfer` 是一个 **pinctrl group**，它定义了 UART2 需要使用的引脚配置。

### 4.3 多组引用示例

模块可以同时引用多组 **group**，比如 PDM（脉冲密度调制）模块：
```c
pdm1: pdm@fe4c0000 {
	compatible = "rockchip,rk3588-pdm";
	reg = <0x0 0xfe4c0000 0x0 0x1000>;
	clocks = <&cru MCLK_PDM1>, <&cru HCLK_PDM1>;
	clock-names = "pdm_clk", "pdm_hclk";
	assigned-clocks = <&cru MCLK_PDM1>;
	assigned-clock-parents = <&cru PLL_AUPLL>;
	dmas = <&dmac1 4>;
	dma-names = "rx";
	power-domains = <&power RK3588_PD_AUDIO>;
	pinctrl-names = "default";
	pinctrl-0 = <&pdm1m0_clk&pdm1m0_clk1&pdm1m0_sdi0&pdm1m0_sdi1&pdm1m0_sdi2&pdm1m0_sdi3>;
	#sound-dai-cells = <0>;
	status = "disabled";
};
```

> [!note]+ 多组引用的优势 
> 当一个功能模块需要使用多个引脚时（如 PDM 需要时钟引脚和多个数据引脚），可以将不同类型的引脚分组管理，使配置更加清晰。

### 4.4 多状态支持

**pinctrl-names** 可以支持多个实例（也叫 **state**），系统默认提供了 4 种状态：
```c
#define PINCTRL_STATE_DEFAULT "default"
#define PINCTRL_STATE_INIT "init"
#define PINCTRL_STATE_IDLE "idle"
#define PINCTRL_STATE_SLEEP "sleep"
```

#### 4.4.1 状态切换机制

- **"init" 状态**：在 driver probe 期间生效
- **状态切换**：probe 完成后可能会切换回 "default"（如果 probe 中切换到其他 state，就不会自动切换回 "init"）

> [!tip]+ 自定义状态 
> **pinctrl-names** 是可以自定义的，驱动程序中可以根据需要匹配和解析自定义的状态名称。

### 4.5 自定义状态示例

**PWM4 的自定义状态配置**：
```c
pwm4: pwm@febd0000 {
	compatible = "rockchip,rk3588-pwm", "rockchip,rk3328-pwm";
	reg = <0x0 0xfebd0000 0x0 0x10>;
	#pwm-cells = <3>;
	pinctrl-names = "active";
	pinctrl-0 = <&pwm4m0_pins>;
	clocks = <&cru CLK_PWM1>, <&cru PCLK_PWM1>;
	clock-names = "pwm", "pclk";
	status = "disabled";
};
```

在这个例子中，使用了自定义的 **"active"** 状态，而不是默认的 "default" 状态。

> [!abstract]+ 本章总结 
> 设备树中的 pinctrl 配置本质上是一套引脚管理机制：
> 
> 1. **系统定义**：在 pinctrl.dtsi 中定义所有可能的引脚配置
> 2. **模块引用**：各个功能模块通过 pinctrl-names 和 pinctrl-x 引用需要的配置
> 3. **状态管理**：支持多种工作状态，满足不同场景下的引脚需求
> 4. **灵活配置**：既支持标准状态，也支持自定义状态，适应各种应用场景

这种设计让引脚配置既规范又灵活，大大简化了硬件驱动的开发工作。