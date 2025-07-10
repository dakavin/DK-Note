
## 1 引言：从寄存器混乱到统一管理的演进

在前面章节，我们有过使用寄存器去编写字符设备的经历了。这种直接在驱动代码中，通过寄存器映射来对外设进行使用的编程方式，从驱动开发者的角度可以说是灾难。因为每当芯片的寄存器发生了改动，那么底层的驱动几乎得重写。

让我们回顾一下传统的引脚操作方式：
```c
// 传统方式：直接操作寄存器
#define GPIO_BASE_ADDR    0x12345000
#define GPIO_DIRECTION    (GPIO_BASE_ADDR + 0x04)
#define GPIO_OUTPUT       (GPIO_BASE_ADDR + 0x08)

// 设置引脚为输出模式
writel(readl(GPIO_DIRECTION) | (1 << pin_num), GPIO_DIRECTION);
// 设置输出电平
writel(readl(GPIO_OUTPUT) | (1 << pin_num), GPIO_OUTPUT);
```

> [!example]+ 
> 这种编程方式就像每次想开灯都要去地下室找到对应的保险丝盒，手动连接电线一样繁琐。更糟糕的是缺乏对引脚的统一管理，容易出现引脚的重复定义。

假设我们在I2C的驱动中将`UART_RX_DATA`引脚和`UART_TX_DATA`引脚复用为SCL和SDA，恰好在编写UART驱动驱动时没有注意到UART_RX_DATA引脚和UART_TX_DATA引脚已经被使用，在驱动中又将其初始化为UART_RX和UART_TX，这样IIC驱动将不能正常工作，并且这种错误很难被发现。

那么在这个问题上，我们更进了一步，学会了使用设备树来描述外设的各种信息(比如寄存器地址)，而不是将寄存器的这些内容放在驱动代码里。这样即使设备信息修改了，我们还是可以通过设备树的接口函数，去灵活的获取设备的信息。

那么，在驱动中有没有更通用的方法，可以不涉及到具体的寄存器操作的内容呢？可以使用`驱动框架`（driver framework），内核针对每个种类的驱动设计一套成熟的、标准的、典型的驱动实现，并**把不同厂家的同类硬件驱动中相同的部分抽出来自己实现好，再把不同部分留出接口给具体的驱动开发工程师来实现**。标准化的驱动实现，统一管理系统资源，维护系统稳定。

`Pinctrl子系统`（Pin Control Subsystem）就是**Linux内核为了统一各SoC厂商的引脚管理而提供的标准化驱动框架**。

## 2 Pinctrl子系统简介

### 2.1 核心作用

**引脚枚举与管理**：在系统初始化的时候，枚举所有可以控制的pin，并标识这些pin。
- **换句话说**，系统需要建立一个完整的引脚"户口本"，记录每个引脚的基本信息。

**引脚功能复用设定**：设定引脚的功能复用，比如复用为GPIO还是SPI、I2C、UART等。

**引脚电气特性配置**：引脚的配置，比如上下拉，驱动强度，去抖等。

`pinctrl子系统`主要工作内容如下：
1. 获取设备树中pin信息
2. 根据获取到的pin信息来设置pin的复用功能
3. 根据获取到的pin信息来设置pin的电气特性，如驱动能力

对于我们使用者来讲，**只需要在设备树里面设置好某个pin的相关属性即可，其他的初始化工作均由pinctrl子系统来完成**

### 2.2 架构图解

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/dfa7f65f1d56c58110e38bd3ba3c8197.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9b37fbaf3d8e6c55ca8f39de6d26527b.png)

如上图所示，**pinctrl核心层是内核抽象出来**，向下为SoC pin controler drvier提供底层通信接口的能力，向上为其他驱动提供了控制pin的能力，比如pin复用、配置引脚的电气特性，同时也为GPIO子系统提供pin操作。而pin控制器驱动层，主要提供了操作pin的方法。

**简单来说**，这是一个典型的分层架构设计：
- **上层**：各种外设驱动（I2C、SPI、UART等）通过标准接口请求引脚资源
- **中层**：Pinctrl核心层负责统一管理，提供标准化的引脚管理接口
- **下层**：各厂商的pin控制器驱动实现具体的硬件操作

### 2.3 源码结构

pinctrl子系统的源文件在内核源码`/drivers/pinctrl`目录下，主要包括
- 核心文件
- 其他内核驱动的接口头文件
- 底层的pin控制器驱动接口
- 其他厂商Pinctrl驱动文件

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/79c22869b64c6fa68138a40b37516814.png)

从目录结构可以看出，pinctrl子系统包含以下重要文件：
- **Makefile**：在内核配置中开启CONFIG_DEBUG_PINCTRL的的话，就会启用Pinctrl下所有与该宏有关的代码，主要用于调试功能
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1a6706acee828c170b002f6f22c8570b.png)

- **core.c**：pinctrl子系统的核心实现文件
- **pinconf.c**：对驱动能力、上下拉等电气特性进行操作（底层就是SoC物理寄存器的操作）
- **pinmux.c**：对复用功能进行操作（底层就是SoC物理寄存器的操作）
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/81d1c5a4f399a71fce46ef41710c83f1.png)
这些截图展示了drivers/pinctrl目录的完整结构，包含了各厂商的具体实现文件，如rockchip、qualcomm、sunxi等平台的pinctrl驱动。

## 3 核心概念详解

**系统中到底有多少个引脚，如何描述这些引脚？** 
pinctrl核心层提供`struct pinctrl_desc`，具体**soc厂商根据这个结构体抽象出一个pin controller实例**， 把系统中所有的pin描述出来（实际是struct pinctrl_desc结构中pins和npins来完成），并建立索引。 相应的就有pin controller driver。

Pinctrl驱动对引脚的管理，通过抽象出`pin groups`和`function`实现，具体以rk3568-lubancat2.dts设备树举例：
```d fold
&pinctrl {
   /*
    * FUNCTION: pmic
    * 功能定义：PMIC（电源管理芯片）相关的引脚功能
    * 说明：一个function代表一组引脚功能，这里定义了所有与PMIC相关的引脚组
    */
   pmic {
      /*
       * PIN GROUP: pmic_int
       * 引脚组名称：PMIC中断引脚组
       * 包含引脚：bank0的PA3引脚
       * 用途：用于PMIC向SoC发送中断信号
       */
      pmic_int: pmic_int {
            rockchip,pins =
               <0 RK_PA3 RK_FUNC_GPIO &pcfg_pull_up>;
      };
      
      /*
       * PIN GROUP: soc_slppin_gpio  
       * 引脚组名称：SoC睡眠引脚GPIO模式组
       * 包含引脚：bank0的PA2引脚
       * 用途：PA2引脚作为普通GPIO使用时的配置
       * 注意：同一个物理引脚PA2在不同pin group中有不同function
       */
      soc_slppin_gpio: soc_slppin_gpio {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_GPIO &pcfg_output_low_pull_down>;
      };
      
      /*
       * PIN GROUP: soc_slppin_slp
       * 引脚组名称：SoC睡眠引脚睡眠模式组  
       * 包含引脚：bank0的PA2引脚（与上面是同一个物理引脚）
       * 用途：PA2引脚作为睡眠控制功能使用时的配置
       * Function冲突：PA2只能选择GPIO或睡眠功能中的一种
       */
      soc_slppin_slp: soc_slppin_slp {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_1 &pcfg_pull_up>;
      };
      
      /*
       * PIN GROUP: soc_slppin_rst
       * 引脚组名称：SoC睡眠引脚复位模式组
       * 包含引脚：bank0的PA2引脚（与上面是同一个物理引脚）  
       * 用途：PA2引脚作为复位功能使用时的配置
       * Function冲突：PA2只能选择GPIO、睡眠或复位功能中的一种
       */
      soc_slppin_rst: soc_slppin_rst {
            rockchip,pins =
               <0 RK_PA2 RK_FUNC_2 &pcfg_pull_none>;
      };
      
      /*
       * PIN GROUP: spk_ctl_gpio
       * 引脚组名称：扬声器控制GPIO引脚组
       * 包含引脚：bank3的PC5引脚
       * 用途：控制扬声器的开关
       */
      spk_ctl_gpio: spk_ctl_gpio {
            rockchip,pins = <3 RK_PC5 RK_FUNC_GPIO &pcfg_pull_up>;
      };
   };
   
   /*........*/
   
   /*
    * FUNCTION: spi  
    * 功能定义：SPI总线相关的引脚功能
    * 说明：定义了SPI总线需要的所有引脚组，如片选信号等
    */
   spi {
      /*
       * PIN GROUP: spi3_cs0
       * 引脚组名称：SPI3片选0引脚组
       * 包含引脚：bank4的PC6引脚  
       * 用途：SPI3总线的第一个片选信号
       * 协同工作：与spi3_cs1共同提供SPI3的片选功能
       */
      spi3_cs0: spi3-cs0 {
            rockchip,pins = <4 RK_PC6 RK_FUNC_GPIO &pcfg_pull_up_drv_level_1>;
      };
      
      /*
       * PIN GROUP: spi3_cs1
       * 引脚组名称：SPI3片选1引脚组  
       * 包含引脚：bank4的PC4引脚
       * 用途：SPI3总线的第二个片选信号
       * 协同工作：与spi3_cs0共同提供SPI3的片选功能
       */
      spi3_cs1: spi3-cs1 {
            rockchip,pins = <4 RK_PC4 RK_FUNC_GPIO &pcfg_pull_up_drv_level_1>;
      };
   };
   /*........*/
}
```

### 3.1 Pin Groups（引脚组）

**Pin Groups**是具有相似功能或需要协同工作的引脚集合。就像一套工具中的螺丝刀套装，不同规格的螺丝刀组成一个工具组，共同完成拧螺丝的任务。

group，就像设备树中的`spi3_cs0: spi3-cs0`，表示一组pins，这组pins统一表示了一种功能，比如，spi需要两个cs，而pmic需要用五个引脚。在定义pins的同时，还会提供对于每个pin的电气特性的配置，如，上下拉电阻、驱动能力等。在内核中用结构体`struct group_desc`描述。
```c
struct group_desc {
    const char *name;    // 引脚组名称
    int *pins;          // 引脚编号数组
    int num_pins;       // 引脚数量
    void *data;         // 私有数据
};
```

### 3.2 Function（功能）

**Function**定义了引脚可以承担的具体功能类别。function，就是一组引脚功能，表示当前这个pin所代表的的功能。只有一个group所引用的pin的function有效，否则会引起pin的function冲突。比如，一个pin既可以作为普通的gpio，也可以作为SPI的cs引脚，那么，这个pin只能代表一个function，即，要么作为普通的gpio，要么作为SPI的cs引脚。

### 3.3 Pin State（引脚状态）

**Pin State**描述设备在特定工作状态下所需的完整引脚配置。一个设备工作在某个状态，所使用的pin和function是唯一确认的，**一个固定的组合确认一个固定的状态，这固定的状态在pinctrl子系统中称为"pin state"**，枚举了当下该设备的可能。在其他设备驱动使用时，pinctrl子系统只需设置相应pin state即可。在设备树中描述就是使用pinctrl-names指定状态名字，pinctrl-x索引对应的引脚状态。

## 4 主要数据结构介绍

### 4.1 struct pinctrl_desc

从面向对象的思想来看，内核将**pinctrl驱动**抽象为`pinctrl_desc`对象，具体到soc厂商的pinctrl驱动便是该对象一个实例，在驱动所有的pin信息以及对于pin的控制接口实例化成pinctrl_desc，并将pinctrl_desc注册到内核中

```c
// include/linux/pinctrl/pinctrl.h
struct pinctrl_desc {
    const char *name;                           // 引脚控制器的名称
    const struct pinctrl_pin_desc *pins;        // 描述一个pin控制器的引脚
    unsigned int npins;                         // 描述该控制器有多少个引脚
    const struct pinctrl_ops *pctlops;          // 引脚操作函数，有描述引脚，获取引脚等，全局控制函数
    const struct pinmux_ops *pmxops;            // 引脚复用相关的操作函数
    const struct pinconf_ops *confops;          // 引脚配置相关
    struct module *owner;                       // 拥有该结构体的模块
#ifdef CONFIG_GENERIC_PINCONF
    unsigned int num_custom_params;             // 自定义参数数量
    const struct pinconf_generic_params *custom_params; // 自定义参数数组
    const struct pin_config_item *custom_conf_items;    // 自定义配置项数组
#endif
};
```

一般控制器驱动匹配设备成功后会调用probe，最后会调用`pinctrl_register`函数，向内核注册pinctrl，产生pinctrl_dev，该函数如下：

### 4.2 pinctrl_register

**pinctrl_register**：用于向内核注册一个引脚控制器设备，创建对应的pinctrl_dev实例。
```c
// include/linux/pinctrl/pinctrl.h
struct pinctrl_dev *pinctrl_register(struct pinctrl_desc *pctldesc,
                                 struct device *dev, void *driver_data);
```
- `pctldesc`：指向pinctrl_desc结构体的指针，描述控制器的属性和操作
- `dev`：关联的设备对象指针
- `driver_data`：驱动私有数据指针
- **返回值**：
	- 成功：返回指向`struct pinctrl_dev`的指针，表示注册成功的控制器设备
	- 失败：返回ERR_PTR类型的错误指针

### 4.3 struct pinctrl_pin_desc

**struct pinctrl_pin_desc**：`单个引脚描述符结构体`，用于描述引脚控制器中每个引脚的基本属性和标识信息
```c
// include/linux/pinctrl/pinctrl.h
struct pinctrl_pin_desc {
    unsigned                               number;              // 引脚编号
    const char                            *name;                // 引脚名称
    void                                  *drv_data;            // 驱动私有数据
};
```
- `number`：引脚在控制器中的唯一编号，用于硬件寄存器访问和引脚索引
- `name`：引脚的字符串名称，用于调试和用户接口显示，如"PA0"、"GPIO_1"等
- `drv_data`：驱动程序的私有数据指针，可存储引脚相关的特定硬件信息或配置

### 4.4 struct group_desc

**struct group_desc**：引脚组描述符结构体，用于描述具有相同功能或需要协同工作的引脚集合
```c
// drivers/pinctrl/core.h
struct group_desc {
    const char                            *name;                // 引脚组名称
    int                                   *pins;                // 引脚编号数组
    int                                    num_pins;            // 引脚数量
    void                                  *data;                // 私有数据
};
```
- `name`：引脚组的标识名称，对应设备树中定义的pin group名称，如"spi3_cs0"、"uart0_pins"
- `pins`：指向引脚编号数组的指针，包含该组中所有引脚的number值
- `num_pins`：引脚组中包含的引脚数量，用于确定pins数组的长度
- `data`：引脚组的私有数据指针，可存储组相关的配置信息或硬件特性参数

## 5 总结

> [!note]+ 背景知识 
> Pinctrl子系统在Linux内核2.6.21版本引入，统一了不同平台的GPIO操作接口，解决了早期各厂商自定义引脚管理方式带来的混乱问题。

> [!tip]+ 实用技巧 
> 在实际开发中，建议先查看芯片的数据手册了解引脚的复用能力，然后参考厂商提供的设备树示例来配置pinctrl节点。

> [!warning]+ 重要提醒 
> 引脚配置错误可能导致硬件损坏，在修改pinctrl配置时要特别小心，建议先在测试板上验证配置的正确性。


Pinctrl子系统通过分层架构设计，实现了引脚管理的标准化：
1. **统一接口**：为上层驱动提供了标准化的引脚操作接口
2. **自动管理**：通过设备树配置实现引脚的自动初始化
3. **冲突检测**：系统级的引脚使用冲突检测和管理
4. **平台抽象**：抽象了不同SoC平台的硬件差异

**换句话说**，Pinctrl子系统让驱动开发者从复杂的寄存器操作中解放出来，只需要在设备树中声明需要什么样的引脚配置，系统会自动完成所有底层工作。这种设计理念体现了现代操作系统追求的"简单性"和"可维护性"原则，是Linux内核设计的典型体现。