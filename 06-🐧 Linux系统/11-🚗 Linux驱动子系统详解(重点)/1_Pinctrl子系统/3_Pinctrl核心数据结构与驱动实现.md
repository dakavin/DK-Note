
## 1 引言：深入系统内核的数据世界

学习完Pinctrl的基本概念和RK3568的具体配置后，我们现在需要深入了解系统的"钢筋骨架"——核心数据结构，以及系统的"建造过程"——驱动的注册与初始化机制。

Linux中的pinctrl子系统（Pin Control Subsystem）是一个用于管理和配置通用输入/输出（GPIO）引脚的框架。它提供了一种标准化的方法，以在Linux内核中对GPIO引脚进行配置、分配和控制，从而适应不同的硬件平台和设备。

在Linux内核中，一切皆对象。Pinctrl子系统通过一系列精心设计的数据结构来描述和管理引脚资源。**换句话说**，这些数据结构就像是引脚管理系统的"户口本"，记录着每个引脚的身份信息、能力特长和当前状态。

## 2 核心数据结构详解（生成者）

### 2.1 pinctrl_desc - 引脚控制器描述符

`pinctrl_desc`结构体用于**描述引脚控制器（pinctrl）的属性和操作**，而引脚控制器是硬件系统中的一个组件，用于管理和控制引脚的功能和状态。

`pinctrl_desc`结构体的作用是**给Soc厂家提供一个统一的接口，用于配置和管理引脚控制器的行为**。

`pinctrl_desc`结构体定义在内核源码目录的`/include/linux/pinctrl/pinctrl.h`文件中，具体内容如下所示：
```c
// include/linux/pinctrl/pinctrl.h
struct pinctrl_desc {
    const char                            *name;                // 控制器唯一标识名称
    const struct pinctrl_pin_desc         *pins;                // 引脚描述符数组
    unsigned int                           npins;               // 引脚总数
    const struct pinctrl_ops              *pctlops;             // 基础操作函数集
    const struct pinmux_ops               *pmxops;              // 复用操作函数集
    const struct pinconf_ops              *confops;             // 配置操作函数集
    struct module                         *owner;               // 所属内核模块
#ifdef CONFIG_GENERIC_PINCONF
    unsigned int                           num_custom_params;   // 自定义参数数量
    const struct pinconf_generic_params   *custom_params;       // 自定义参数数组
    const struct pin_config_item          *custom_conf_items;   // 自定义配置项数组
#endif
};
```
- `name`：引脚控制器的**唯一标识名称**，用于在系统中区分不同的pinctrl控制器
- `pins`：指向**引脚描述符**数组的指针，用于描述引脚的属性和配置，每个元素包含引脚编号、名称、模式等基本信息
- `npins`：引脚描述符**数组的元素数量**，表示该控制器管理的引脚总数
- `pctlops`：**引脚控制基础操作函数集**，包括引脚枚举、引脚组管理等核心操作接口
- `pmxops`：**引脚复用操作函数集**，提供引脚功能选择和复用管理的操作接口
- `confops`：**引脚配置操作函数集**，提供引脚电气特性配置（上下拉、驱动能力等）的操作接口
- `owner`：拥有该控制器的内核模块指针，用于模块引用计数管理

- `num_custom_params`：自定义配置参数的数量，用于扩展标准配置选项之外的特殊参数
- `custom_params`：指向自定义配置参数数组的指针，定义厂商特有的配置参数类型和属性
- `custom_conf_items`：指向自定义配置项数组的指针，用于设备树解析时识别自定义配置属性

### 2.2 rockchip_pinctrl - RK3568平台封装

瑞芯微为了适应其芯片的特定需求和功能，对`struct pinctrl_desc`进行了再一次封装。封装后的`struct rockchip_pinctrl`结构体在`struct pinctrl_desc`的基础上增加了与瑞芯微芯片相关的字段和指针，这种封装可以提供更好的集成性、易用性和扩展性，同时保持与通用引脚控制器框架的兼容性。

`rockchip_pinctrl`结构体定义在内核源码目录下的`/drivers/pinctrl/pinctrl-rockchip.h`文件中：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6ccfbda34c113b5e4e3dd9991dd3c9eb.png)

具体内容如下所示：
```c
// drivers/pinctrl/pinctrl-rockchip.c
struct rockchip_pinctrl {
    struct regmap                         *regmap_base;         // 基本寄存器映射
    int                                    reg_size;            // 寄存器字节大小
    struct regmap                         *regmap_pull;         // 上下拉寄存器映射
    struct regmap                         *regmap_pmu;          // PMU寄存器映射
    struct device                         *dev;                 // 设备指针
    struct rockchip_pin_ctrl              *ctrl;                // 芯片引脚控制器
    struct pinctrl_desc                    pctl;                // 引脚控制器描述符
    struct pinctrl_dev                    *pctl_dev;            // 引脚控制器设备
    struct rockchip_pin_group             *groups;              // 引脚组数组
    unsigned int                           ngroups;             // 引脚组数量
    struct rockchip_pmx_func              *functions;           // 引脚功能数组
    unsigned int                           nfunctions;          // 引脚功能数量
};
```
- `regmap_base`：基本寄存器映射接口，提供对芯片通用引脚控制寄存器的读写操作
	- 物理地址 → ioremap映射 → 虚拟地址 → regmap封装 → regmap对象
- `reg_size`：寄存器的字节大小，用于确定寄存器地址空间范围
- `regmap_pull`：上下拉电阻控制寄存器映射，专门用于配置引脚的上拉和下拉功能
- `regmap_pmu`：电源管理单元寄存器映射，用于控制PMU域引脚的电源管理功能
- `dev`：指向平台设备的设备指针（pdev->dev），包含设备的物理地址、中断、电源等硬件信息
- `ctrl`：指向Rockchip芯片特定的引脚控制器配置，包含芯片相关的引脚信息和操作接口（如有多少bank、每个bank多少pin等）
- `pctl`：**引脚控制器描述符实例**，用于向pinctrl核心层注册控制器的基本属性和操作函数
- `pctl_dev`：引脚控制器设备实例指针，表示**在pinctrl子系统中注册的设备对象**
- `groups`：Rockchip芯片引脚组数组指针，包含所有已定义的引脚组配置信息（如`i2c3m0_xfer`这样的pin group定义）
- `ngroups`：引脚组数组的元素数量，用于遍历和管理引脚组
- `functions`：Rockchip芯片引脚功能数组指针，定义了引脚可承担的各种功能（如`i2c3`、`uart2`这样的function定义）
- `nfunctions`：引脚功能数组的元素数量，用于功能枚举和管理

**对应的流程图**
```txt
设备树定义
    ↓ 解析
rockchip_pinctrl_probe()
    ↓ 填充
struct rockchip_pinctrl {
    regmap_xxx  ← syscon/ioremap获取
    dev         ← &pdev->dev
    ctrl        ← 芯片特定数据
    groups      ← 解析设备树生成
    functions   ← 解析设备树生成
    pctl        ← 填充操作函数
}
    ↓ 注册
pctl_dev = pinctrl_register(&pctl)
    ↓
系统可以使用pinctrl功能
```

### 2.3 pinctrl_dev - 运行时控制器对象

**struct pinctrl_dev**：运行时引脚控制器设备结构体，表示在pinctrl子系统中注册的引脚控制器实例，管理整个控制器的运行时状态
```c
// drivers/pinctrl/core.h
struct pinctrl_dev {
    struct list_head                       node;                // 全局设备链表节点
    struct pinctrl_desc                   *desc;                // 控制器描述符
    struct radix_tree_root                 pin_desc_tree;       // 引脚描述符树
#ifdef CONFIG_GENERIC_PINCTRL_GROUPS
    struct radix_tree_root                 pin_group_tree;      // 引脚组树
    unsigned int                           num_groups;          // 引脚组数量
#endif
#ifdef CONFIG_GENERIC_PINMUX_FUNCTIONS
    struct radix_tree_root                 pin_function_tree;   // 引脚功能树
    unsigned int                           num_functions;       // 引脚功能数量
#endif
    struct list_head                       gpio_ranges;         // GPIO范围列表
    struct device                         *dev;                 // 设备指针
    struct module                         *owner;               // 所属模块
    void                                  *driver_data;         // 驱动私有数据
    struct pinctrl                        *p;                   // 控制句柄
    struct pinctrl_state                  *hog_default;         // 占用默认状态
    struct pinctrl_state                  *hog_sleep;           // 占用睡眠状态
    struct mutex                           mutex;               // 互斥锁
#ifdef CONFIG_DEBUG_FS
    struct dentry                         *device_root;         // 调试文件系统根目录
#endif
};
```
- `node`：全局链表节点，用于链接到系统pinctrl设备列表
- `desc`：指向控制器描述符，包含控制器的基本属性和操作函数
- `pin_desc_tree`：**引脚描述**符基数树，用于快速查找引脚信息
- `pin_group_tree`：**引脚组描述**基数树，用于快速查找和管理引脚组
- `pin_function_tree`：**引脚功能**基数树，用于快速查找和管理引脚功能
- `gpio_ranges`：GPIO范围列表，描述该控制器管理的GPIO引脚范围
- `dev`：关联的设备结构体指针
- `driver_data`：驱动程序的私有数据指针
- `hog_default`：控制器自身占用的默认状态配置
- `mutex`：保护控制器操作的互斥锁

## 3 Pinctrl数据结构（消费者）

**消费者结构体链路图**
![image.png|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/50c2483cfe3fbef54f63c0319fe1969f.png)

### 3.1 dev_pin_info - 设备引脚信息

`struct device`结构体中如果定义了`CONFIG_PINCTRL`那记录设备引脚信息的结构体`struct dev_pin_info *pins`就会被使能：

**struct dev_pin_info**：设备引脚信息结构体，嵌入在device结构体中，保存设备的引脚控制句柄和常用状态指针
```c
// include/linux/pinctrl/devinfo.h
struct dev_pin_info {
    struct pinctrl                        *p;                   // 引脚控制句柄
    struct pinctrl_state                  *default_state;       // 默认状态
    struct pinctrl_state                  *init_state;          // 初始化状态
#ifdef CONFIG_PM
    struct pinctrl_state                  *sleep_state;         // 睡眠状态
    struct pinctrl_state                  *idle_state;          // 空闲状态
#endif
};
```
- `p`：指向该设备的引脚控制句柄，用于引脚控制和配置操作
- `default_state`：设备默认工作状态的引脚配置，对应pinctrl-names中的"default"
- `init_state`：设备初始化阶段的引脚配置，对应pinctrl-names中的"init"
- `sleep_state`：设备睡眠模式的引脚配置，用于电源管理时的引脚状态切换
- `idle_state`：设备空闲模式的引脚配置，用于节能时的引脚状态管理

**设备树使用示例**：

```c
// arch/arm64/boot/dts/rockchip/rk3568-lubancat-2.dtsi
leds: leds {
    compatible = "gpio-leds";
    
    sys_led: sys-led {
        gpios = <&gpio0 RK_PC7 GPIO_ACTIVE_LOW>;
        pinctrl-names = "default";                    // 状态名称
        pinctrl-0 = <&sys_led_pin>;                  // default状态的pin group
    };
};
```

### 3.2 pinctrl - 引脚控制实例

**struct pinctrl**：设备引脚控制句柄结构体，表示单个设备使用pinctrl子系统的控制实例，管理该设备的所有引脚状态
```c
// drivers/pinctrl/core.h
struct pinctrl {
    struct list_head                       node;                // 全局链表节点
    struct device                         *dev;                 // 使用的设备
    struct list_head                       states;              // 状态列表
    struct pinctrl_state                  *state;               // 当前状态
    struct list_head                       dt_maps;             // 设备树映射表
    struct kref                            users;               // 引用计数
};
```
- `node`：全局链表节点，用于将该控制句柄链接到系统的pinctrl列表中
- `dev`：使用该引脚控制句柄的设备指针
- `states`：该设备定义的所有引脚状态列表头
- `state`：当前激活的引脚状态指针
- `dt_maps`：从设备树动态解析的映射表列表
- `users`：引用计数，用于管理该控制句柄的生命周期

> [!tip]+ 提示
> pinctrl本质就是消费者的“订单”
> - dev：表示是哪个消费者在使用
> - states：这个订单支持什么状态
> - state：当前订单的状态

### 3.3 pinctrl_state - 引脚状态

**struct pinctrl_state**：引脚状态结构体，表示设备在特定工作模式下的引脚配置状态，是pinctrl子系统的基本配置单元
```c
// drivers/pinctrl/core.h
struct pinctrl_state {
    struct list_head                       node;                // 链表节点
    const char                            *name;                // 状态名称
    struct list_head                       settings;            // 设置列表
};
```
- `node`：链表节点，用于将该状态链接到设备的状态列表中
- `name`：状态名称字符串，对应设备树中pinctrl-names定义的名称，如"default"、"sleep"等
- `settings`：该状态包含的具体引脚设置列表，每个设置对应一个pin group的配置

### 3.4 pinctrl_setting - 具体设置

**struct pinctrl_setting**：引脚具体设置结构体，描述单个引脚组的复用功能或配置参数的具体设置内容
```c
// drivers/pinctrl/core.h
struct pinctrl_setting {
    struct list_head                       node;                // 链表节点
    enum pinctrl_map_type                  type;                // 设置类型
    struct pinctrl_dev                    *pctldev;             // 引脚控制器设备
    const char                            *dev_name;            // 设备名称
    union {
        struct pinctrl_setting_mux         mux;                 // 复用设置
        struct pinctrl_setting_configs     configs;             // 配置设置
    } data;
};

struct pinctrl_setting_mux {
    unsigned                               group;               // 引脚组选择器
    unsigned                               func;                // 功能选择器
};

struct pinctrl_setting_configs {
    unsigned                               group_or_pin;        // 组或引脚ID
    unsigned long                         *configs;             // 配置参数数组
    unsigned                               num_configs;         // 配置参数数量
};
```
- `node`：链表节点，用于链接到pinctrl_state的settings列表中
- `type`：设置类型，指示是复用设置还是配置设置
- `pctldev`：处理该设置的引脚控制器设备指针
- `dev_name`：使用该设置的设备名称
- `data.mux`：复用设置数据，包含引脚组和功能选择器
- `data.configs`：配置设置数据，包含配置参数数组和数量

## 4 pinctrl生产者和消费者的关系

### 4.1 整体架构：生产者与消费者模式

pinctrl子系统采用**生产者-消费者**架构，数据结构关系如下：

**生产者侧**：提供引脚配置服务
- pinctrl控制器定义引脚复用和配置能力
- 在系统启动时注册到pinctrl子系统

**消费者侧**：使用引脚配置服务
- 设备通过pinctrl API请求特定的引脚配置
- 在设备probe时自动应用配置

**连接桥梁**：通过设备树phandle引用和pinctrl_dev指针建立关联
- **&引用的是pinctrl节点里面function下的pin group**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/05c805b4ca3529c83557881287b1e3c1.png)

### 4.2 数据结构角色分工

|结构体|角色定位|主要用途|创建/填充时机|
|---|---|---|---|
|`struct pinctrl_desc`|**生产者规格**|描述引脚控制器能力和接口|pinctrl驱动probe时填充|
|`struct rockchip_pinctrl`|**平台封装**|包含pinctrl_desc和平台特定数据|驱动probe时创建并填充|
|`struct pinctrl_dev`|**生产者实例**|运行时的引脚控制器对象|pinctrl_register()时创建|
|`struct dev_pin_info`|**消费者入口**|设备的引脚信息管理入口|设备probe时创建|
|`struct pinctrl`|**消费者句柄**|设备使用pinctrl的控制实例|devm_pinctrl_get()时创建|
|`struct pinctrl_state`|**工作状态**|设备不同模式下的引脚配置|解析pinctrl-names时创建|
|`struct pinctrl_setting`|**具体配置**|单个引脚组的详细设置|解析pinctrl-0/1/2时创建|

### 4.3 数据流向和层次关系

```txt
【设备树定义】                          【设备树定义】
                                   
/* 生产者：pinctrl控制器 */              /* 消费者：使用pinctrl的设备 */
pinctrl: pinctrl {                      i2c3: i2c@fe5c0000 {
  compatible = "rockchip,                 compatible = "rockchip,rk3399-i2c";
              rk3568-pinctrl";            pinctrl-names = "default", "sleep";
  ...                                     pinctrl-0 = <&i2c3m0_xfer>;
                                         pinctrl-1 = <&i2c3_sleep>;
  /* function定义 */                    };
  i2c3 {                               
    i2c3m0_xfer: i2c3m0-xfer {         
      rockchip,pins = <...>;           
    };                                 
  };                                   
};                                     

        ↓ postcore_initcall()                  ↓ 设备probe时
        
【生产者侧初始化流程】                    【消费者侧初始化流程】

rockchip_pinctrl_probe()                device_driver.probe()
    ↓                                       ↓
rockchip_pinctrl_parse_dt()            really_probe()
    ↓                                       ↓
解析设备树中的functions/groups          pinctrl_bind_pins(dev)
    ↓                                       ↓
填充rockchip_pinctrl结构体：            devm_pinctrl_get(dev)
  - groups[] ← 从DT解析                     ↓
  - functions[] ← 从DT解析              create_pinctrl(dev)
  - pctl ← 填充pinctrl_desc                 ↓
    - pins[] ← 硬件定义                of_pinctrl_get(dev)
    - ops ← 操作函数集                      ↓
    ↓                                  解析pinctrl-names → 创建states
pinctrl_register(&info->pctl)          解析pinctrl-0/1 → 创建settings
    ↓                                       ↓
创建pinctrl_dev实例                    对每个phandle调用：
存储到全局链表中                        dt_to_map_one_config()
    ↓                                       ↓
                                      通过phandle找到pinctrl_dev
【运行时数据结构】                           ↓
                                      创建pinctrl_setting并设置：
pinctrl_dev（控制器实例）                 setting->pctldev = 找到的pinctrl_dev
  ├─ desc → &info->pctl                    ↓
  ├─ driver_data → info               应用配置时：
  ├─ pin_desc_tree                    pinctrl_select_state()
  ├─ pin_group_tree                       ↓
  └─ pin_function_tree                通过setting->pctldev->desc->ops
                                      调用具体的硬件操作函数
        ↑                                   ↓
        └───────────────────────────────────┘
              通过setting->pctldev指针连接
```

### 4.4 关键连接点

1. **设备树phandle引用**
    ```dts
    pinctrl-0 = <&i2c3m0_xfer>;  // 通过phandle引用pin group
    ```
2. **运行时指针连接**
    ```c
    pinctrl_setting->pctldev  // 指向提供服务的pinctrl_dev
    pinctrl_dev->desc->ops    // 指向具体的操作函数集
    ```
3. **查找匹配过程**
    - 解析phandle → 找到设备树节点
    - 通过节点找到对应的pinctrl_dev
    - 创建setting并建立连接

### 4.5 理解要点

1. **两条初始化路径**
    - **生产者**：系统启动早期通过postcore_initcall注册
    - **消费者**：设备probe时通过pinctrl_bind_pins自动配置
2. **分层设计的优势**
    - **抽象层次清晰**：从设备树配置到硬件操作，每层职责明确
    - **平台无关性**：通过ops函数指针实现硬件差异的隔离
    - **自动化管理**：设备probe时自动应用默认配置
3. **动态配置能力**
    - 支持多状态切换（default/sleep/idle等）
    - 每个状态可包含多个pin group配置
    - 运行时通过pinctrl_select_state()切换

> [!tip]+ 重要 
> **连接的核心**：消费者通过`pinctrl_setting->pctldev`指针找到生产者的`pinctrl_dev`，然后通过`pinctrl_dev->desc->ops`调用具体的硬件操作函数，实现引脚配置的实际应用。

这种设计实现了**配置与实现分离**、**状态化管理**、**资源共享**的目标，为复杂的引脚管理提供了清晰的软件架构。

## 5 RK3568 Pinctrl驱动具体实现

### 5.1 平台驱动注册

设备树中存放的只是设备的描述信息，而具体的功能实现取决于相应的pinctrl驱动，根据rk3568.dtsi设备树中pinctrl节点的compatible属性进行查找，可以查找到pinctrl的驱动文件是内核源码的`drivers/pinctrl/pinctrl-rockchip.c`，如下所示：

```c
// drivers/pinctrl/pinctrl-rockchip.c 4119-4122
#ifdef CONFIG_CPU_RK3568
	{ .compatible = "rockchip,rk3568-pinctrl",
		.data = &rk3568_pin_ctrl },
#endif
```

首先进入到内核源码目录下的`/drivers/pinctrl/pinctrl-rockchip.c`驱动文件中，找到驱动的入口函数，具体内容如下所示：
```c fold
// drivers/pinctrl/pinctrl-rockchip.c 4126-4145
static struct platform_driver rockchip_pinctrl_driver = {
	.probe		= rockchip_pinctrl_probe,
	.driver = {
		.name	= "rockchip-pinctrl",
		.pm = &rockchip_pinctrl_dev_pm_ops,
		.of_match_table = rockchip_pinctrl_dt_match,
	},
};

static int __init rockchip_pinctrl_drv_register(void)
{
	return platform_driver_register(&rockchip_pinctrl_driver);
}
postcore_initcall(rockchip_pinctrl_drv_register);

static void __exit rockchip_pinctrl_drv_unregister(void)
{
	platform_driver_unregister(&rockchip_pinctrl_driver);
}
module_exit(rockchip_pinctrl_drv_unregister);
```

可以看到pinctrl驱动使用的是platform总线，当设备和驱动匹配成功之后会进入`rockchip_pinctrl_probe`函数进行初始化。

> [!note]+ 背景知识 
> `postcore_initcall`是Linux内核的初始化级别之一，它在`core_initcall`之后执行，确保核心子系统已经初始化完成。Pinctrl作为基础设施，需要在其他驱动之前初始化。

### 5.2 核心初始化函数：rockchip_pinctrl_probe

probe函数的具体内容如下所示：

```c fold
// drivers/pinctrl/pinctrl-rockchip.c 3448-3540
static int rockchip_pinctrl_probe(struct platform_device *pdev)
{
        struct rockchip_pinctrl *info;  // Rockchip GPIO 控制器的信息结构体指针
        struct device *dev = &pdev->dev; // 设备结构体指针
        struct rockchip_pin_ctrl *ctrl;  // Rockchip GPIO 控制器的配置结构体指针
        struct device_node *np = pdev->dev.of_node, *node; // 设备节点指针
        struct resource *res;
        void __iomem *base;
        int ret;

		// 检查设备树节点是否存在，pinctrl驱动必须通过设备树配置
        if (!dev->of_node) {
                dev_err(dev, "device tree node not found\n");
                return -ENODEV;
        }

        // 分配并初始化一个 rockchip_pinctrl 结构体
        // 外部的dev必须存在！使用设备管理的内存分配，设备卸载时自动释放
        info = devm_kzalloc(dev, sizeof(*info), GFP_KERNEL);
        if (!info)
                return -ENOMEM;
		
		// 将dev保存到分配的info中，建立设备与驱动数据的关联
        info->dev = dev;

        // 获取并设置与 pdev 相关的 rockchip_pin_ctrl 结构体
        // 这里会根据compatible字符串匹配对应的SoC配置数据
        ctrl = rockchip_pinctrl_get_soc_data(info, pdev);
        if (!ctrl) {
                dev_err(dev, "driver data not available\n");
                return -EINVAL;
        }
        info->ctrl = ctrl;

        // 解析设备树中的"rockchip,grf"节点， 获取寄存器映射基地址
        // GRF(General Register Files)是Rockchip芯片的通用寄存器文件
        node = of_parse_phandle(np, "rockchip,grf", 0);
        if (node) {
		        // 通过syscon方式获取GRF寄存器映射
                info->regmap_base = syscon_node_to_regmap(node);
                if (IS_ERR(info->regmap_base))
                        return PTR_ERR(info->regmap_base);
        } else {
		        // 如果没有GRF节点，则直接映射pinctrl的寄存器区域
                res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
                base = devm_ioremap_resource(&pdev->dev, res);
                if (IS_ERR(base))
                        return PTR_ERR(base);

				// 配置regmap参数并初始化
                rockchip_regmap_config.max_register = resource_size(res) - 4;
                rockchip_regmap_config.name = "rockchip,pinctrl";
                info->regmap_base = devm_regmap_init_mmio(&pdev->dev, base,
                                                    &rockchip_regmap_config);

                // 检查旧的设备树绑定方式
                info->reg_size = resource_size(res);

                // 兼容旧绑定：RK3188芯片的pull寄存器作为第二个资源
                if (ctrl->type == RK3188 && info->reg_size < 0x200) {
                        res = platform_get_resource(pdev, IORESOURCE_MEM, 1);
                        base = devm_ioremap_resource(&pdev->dev, res);
                        if (IS_ERR(base))
                                return PTR_ERR(base);

                        rockchip_regmap_config.max_register =
                                                        resource_size(res) - 4;
                        rockchip_regmap_config.name = "rockchip,pinctrl-pull";
                        info->regmap_pull = devm_regmap_init_mmio(&pdev->dev,
                                                    base,
                                                    &rockchip_regmap_config);
                }
        }

        // 尝试查找可选的PMU(Power Management Unit)syscon引用 
        // PMU用于低功耗管理和一些特殊的GPIO配置
        node = of_parse_phandle(np, "rockchip,pmu", 0);
        if (node) {
                info->regmap_pmu = syscon_node_to_regmap(node);
                if (IS_ERR(info->regmap_pmu))
                        return PTR_ERR(info->regmap_pmu);
        }

        // 处理某些SoC的特殊初始化
        // 不同的Rockchip芯片可能需要特定的初始化流程
        if (ctrl->soc_data_init) {
                ret = ctrl->soc_data_init(info);
                if (ret)
                        return ret;
        }

		// 注册pinctrl控制器到内核子系统！！！
        // 这会创建pinctrl设备并注册到pinctrl框架
        ret = rockchip_pinctrl_register(pdev, info);
        if (ret)
                return ret;

		// 将info保存为平台设备的驱动数据，供其他函数使用
        platform_set_drvdata(pdev, info);

		// 解析并注册子设备(GPIO bank，gpio0、gpio1等)
        // 会根据设备树创建对应的GPIO控制器设备
        ret = of_platform_populate(np, rockchip_bank_match, NULL, NULL);
        if (ret) {
                dev_err(&pdev->dev, "failed to register gpio device\n");
                return ret;
        }
        dev_info(dev, "probed %s\n", dev_name(dev));

        return 0;
}
```
- **内存分配顺序**：先获取外部的`dev`指针 → 用它分配`info`内存 → 将`dev`保存到`info`中
- **设备树解析**：通过`of_parse_phandle`获取其他节点的引用（如GRF、PMU）
- **regmap机制**：用于统一管理寄存器访问，支持不同的底层访问方式
- **兼容性处理**：代码中处理了旧版本设备树绑定的兼容性
- **子设备注册**：pinctrl控制器注册成功后，再注册GPIO bank子设备

上面Probe函数的作用是初始化和配置Rockchip GPIO控制器，并将相关信息存储在`rockchip_pinctrl`结构体中，最后注册相关设备和GPIO接口。

### 5.3 控制器注册：rockchip_pinctrl_register

```c fold
// drivers/pinctrl/pinctrl-rockchip.c 3164-3210
static int rockchip_pinctrl_register(struct platform_device *pdev,
					struct rockchip_pinctrl *info)
{
	struct pinctrl_desc *ctrldesc = &info->pctl;
	struct pinctrl_pin_desc *pindesc, *pdesc;
	struct rockchip_pin_bank *pin_bank;
	int pin, bank, ret;
	int k;

	// 设置pinctrl控制器描述符的基本信息
	ctrldesc->name = "rockchip-pinctrl";        //控制器名称
	ctrldesc->owner = THIS_MODULE;              // 模块所有者
	ctrldesc->pctlops = &rockchip_pctrl_ops;    //pinctrl基本操作函数集
	ctrldesc->pmxops = &rockchip_pmx_ops;       //pin复用操作函数集
	ctrldesc->confops = &rockchip_pinconf_ops;  //pin配置操作函数集

	// 为所有pin（数组）分配内存
	// nr_pins是该SoC所有pin的数量
	pindesc = devm_kcalloc(&pdev->dev,
			       info->ctrl->nr_pins, sizeof(*pindesc),
			       GFP_KERNEL);
	if (!pindesc)
		return -ENOMEM;

	// 将pin描述符数组和数量设置到控制器描述符中
	ctrldesc->pins = pindesc;
	ctrldesc->npins = info->ctrl->nr_pins;

	// 为每个pin生成描述符和名称，使用另外一个指针，用于处理遍历
	pdesc = pindesc;
	for (bank = 0, k = 0; bank < info->ctrl->nr_banks; bank++) {
		// 获取当前bank
		pin_bank = &info->ctrl->pin_banks[bank];
		// 遍历当前bank中的每个pin
		for (pin = 0; pin < pin_bank->nr_pins; pin++, k++) {
			// 设置pin的全局编号
			pdesc->number = k;
			// 生成pin名称，格式为"bankname-pin编号"，如"gpio0-5"
			pdesc->name = kasprintf(GFP_KERNEL, "%s-%d",
						pin_bank->name, pin);
			// 移动到下一个pin描述符
			pdesc++;
		}
	}

	// 解析设备树中的pinctrl配置信息 
	// 包括pinmux、pinconf等配置
	ret = rockchip_pinctrl_parse_dt(pdev, info);
	if (ret)
		return ret;

	// 向内核pinctrl子系统注册该控制器
	// 注册成功后，其他驱动就可以通过pinctrl接口使用这些pin
	info->pctl_dev = devm_pinctrl_register(&pdev->dev, ctrldesc, info);
	if (IS_ERR(info->pctl_dev)) {
		dev_err(&pdev->dev, "could not register pinctrl driver\n");
		return PTR_ERR(info->pctl_dev);
	}

	return 0;
}
```
- **三种操作函数集**：pctlops-基本操作，pmxops-复用操作，confops-配置操作
- **pin编号系统**：每个pin有一个全局唯一编号`k`，名称为 bank名称-pin号，如gpio0-5
- **内存管理**：`devm_kcalloc`分配pin描述符数组，`kasprintf`为每个pin动态分配名称字符串
- **注册流程**：
	1. 设置控制器基本信息
	2. 生成所有pin的描述符
	3. 解析设备树配置
	4. 最后注册到pinctrl子系统

### 5.4 设备树解析：rockchip_pinctrl_parse_dt

```c fold
/ 解析设备树中的pinctrl配置信息
// 主要解析pin的功能配置(functions)和引脚组配置(groups)
static int rockchip_pinctrl_parse_dt(struct platform_device *pdev,
					      struct rockchip_pinctrl *info)
{
	struct device *dev = &pdev->dev;
	// 获取当前设备的设备树节点
	struct device_node *np = dev->of_node;
	struct device_node *child;
	int ret;
	int i;

	// 统计设备树中function和group的数量
	// 遍历所有子节点，计算需要多少个function和group结构体
	rockchip_pinctrl_child_count(info, np);

	// 打印调试信息，显示统计到的function和group数量
	dev_dbg(&pdev->dev, "nfunctions = %d\n", info->nfunctions);
	dev_dbg(&pdev->dev, "ngroups = %d\n", info->ngroups);

	// 为function数组分配内存
	// 每个function代表一个功能配置，如uart、spi、i2c等
	info->functions = devm_kcalloc(dev,
					      info->nfunctions,
					      sizeof(struct rockchip_pmx_func),
					      GFP_KERNEL);
	if (!info->functions)
		return -ENOMEM;

	// 为group数组分配内存  
	// 每个group代表一组相关的pin配置
	info->groups = devm_kcalloc(dev,
					    info->ngroups,
					    sizeof(struct rockchip_pin_group),
					    GFP_KERNEL);
	if (!info->groups)
		return -ENOMEM;

	i = 0;

	// 遍历设备树的所有子节点，解析每个function配置
	for_each_child_of_node(np, child) {
		// 检查当前子节点是否是function节点
		// 只有function节点才包含pinctrl配置信息
		if (!is_function_node(child))
			continue;

		// 解析当前function节点的配置信息
		// 包括该function包含的所有group配置
		ret = rockchip_pinctrl_parse_functions(child, info, i++);
		if (ret) {
			dev_err(&pdev->dev, "failed to parse function\n");
			of_node_put(child); // 释放节点引用
			return ret;
		}
	}

	return 0;
}
```
- **Function层**：代表一个功能模块（如uart、spi、i2c等）
- **Group层**：代表该功能模块中的一组pin配置
- **内存分配策略**：先统计数量，再一次性分配所需的内存数组，避免动态扩展，提高效率
- **解析流程**：
	1. 遍历所有子节点
	2. 识别function子节点
	3. 对每个function进行详细解析
	4. 解析过程中田中 info->functions 和 info->groups 两个数组

### 5.5 内核注册：devm_pinctrl_register

该函数的作用是注册一个引脚控制设备，用`pinctrl_dev`来表示，上面提到的私有数据会传输到`pinctrl_init_controller`函数进行初始化和构建`pinctrl_dev`结构体，然后跳转到`pinctrl_init_controller`的函数定义：

```c fold
// drivers/pinctrl/core.c 2190-2211
// 设备管理的pinctrl控制器注册函数
// 使用devres机制，设备卸载时自动注销pinctrl控制器
struct pinctrl_dev *devm_pinctrl_register(struct device *dev,
					  struct pinctrl_desc *pctldesc,
					  void *driver_data)
{
	struct pinctrl_dev **ptr, *pctldev;

	// 分配devres资源管理结构体，用于存储pinctrl_dev指针
	// 当设备被移除时，会自动调用devm_pinctrl_dev_release清理函数
	ptr = devres_alloc(devm_pinctrl_dev_release, sizeof(*ptr), GFP_KERNEL);
	if (!ptr)
		return ERR_PTR(-ENOMEM);

	// 调用核心注册函数，创建并注册pinctrl控制器
	pctldev = pinctrl_register(pctldesc, dev, driver_data);
	if (IS_ERR(pctldev)) {
		// 注册失败，释放之前分配的devres资源
		devres_free(ptr);
		return pctldev;
	}

	// 注册成功，保存pinctrl_dev指针到devres结构中
	*ptr = pctldev;
	// 将devres资源添加到设备的资源管理列表中
	// 设备卸载时会自动清理
	devres_add(dev, ptr);

	return pctldev;
}
EXPORT_SYMBOL_GPL(devm_pinctrl_register);
```

```c fold
// drivers/pinctrl/core.c 2071-2088
// pinctrl控制器注册的核心函数
// 完成控制器的初始化和启用
struct pinctrl_dev *pinctrl_register(struct pinctrl_desc *pctldesc,
				    struct device *dev, void *driver_data)
{
	struct pinctrl_dev *pctldev;
	int error;

	// 初始化pinctrl控制器
	// 创建pinctrl_dev结构体，设置基本信息，注册到全局列表
	pctldev = pinctrl_init_controller(pctldesc, dev, driver_data);
	if (IS_ERR(pctldev))
		return pctldev;

	// 启用pinctrl控制器
	// 创建debugfs接口，注册到pinctrl子系统
	error = pinctrl_enable(pctldev);
	if (error)
		return ERR_PTR(error);

	return pctldev;

}
EXPORT_SYMBOL_GPL(pinctrl_register);
```

```txt
devm_pinctrl_register
├── devres_alloc         // 分配资源管理结构
├── pinctrl_register     // 调用核心注册函数
│   ├── pinctrl_init_controller  // 初始化控制器
│   └── pinctrl_enable          // 启用控制器
└── devres_add          // 添加到设备资源管理
```

### 5.6 pinctrl_init_controller实现

drivers/pinctrl/core.c

```c fold
// drivers/pinctrl/core.c
// pinctrl控制器初始化函数
// 创建pinctrl_dev结构体并进行基本的初始化工作
static struct pinctrl_dev *
pinctrl_init_controller(struct pinctrl_desc *pctldesc, struct device *dev,
                        void *driver_data)
{
        struct pinctrl_dev *pctldev;
        int ret;

		// 参数有效性检查
        if (!pctldesc)
                return ERR_PTR(-EINVAL);
        if (!pctldesc->name)
                return ERR_PTR(-EINVAL);

		// 分配pinctrl设备结构体内存
        pctldev = kzalloc(sizeof(*pctldev), GFP_KERNEL);
        if (!pctldev)
                return ERR_PTR(-ENOMEM);

        /* 初始化引脚控制设备结构体 */
        pctldev->owner = pctldesc->owner;   
        pctldev->desc = pctldesc;
        pctldev->driver_data = driver_data;

		// 初始化基数树，用于快速查找pin描述符
		// 基数树比链表查找更高效，时间复杂度为O(log n)
        INIT_RADIX_TREE(&pctldev->pin_desc_tree, GFP_KERNEL);
        
#ifdef CONFIG_GENERIC_PINCTRL_GROUPS
		// 初始化pin组基数树（如果启用了通用pin groups支持）
        INIT_RADIX_TREE(&pctldev->pin_group_tree, GFP_KERNEL);
#endif
#ifdef CONFIG_GENERIC_PINMUX_FUNCTIONS
		// 初始化pin功能基数树（如果启用了通用pinmux功能支持）
        INIT_RADIX_TREE(&pctldev->pin_function_tree, GFP_KERNEL);
#endif

		// 初始化GPIO范围链表，用于管理GPIO和pinctrl的映射关系
        INIT_LIST_HEAD(&pctldev->gpio_ranges);
        // 初始化控制器节点链表，用于加入全局pinctrl控制器列表
        INIT_LIST_HEAD(&pctldev->node);
        // 关联设备指针
        pctldev->dev = dev;
        // 初始化互斥锁，保护pinctrl操作的并发访问
        mutex_init(&pctldev->mutex);

        /* 检查核心操作函数的完整性 */ 
        // 验证必须的基本操作函数是否都已实现
        ret = pinctrl_check_ops(pctldev);
        if (ret) {
                dev_err(dev, "pinctrl ops lacks necessary functions\n");
                goto out_err;
        }

        /* 如果实现了pinmux功能，检查pinmux操作函数的完整性 */
        if (pctldesc->pmxops) {
                ret = pinmux_check_ops(pctldev);
                if (ret)
                        goto out_err;
        }

        /* 如果实现了pinconfig功能，检查pinconfig操作函数的完整性 */
        if (pctldesc->confops) {
                ret = pinconf_check_ops(pctldev);
                if (ret)
                        goto out_err;
        }

        /* 注册所有的引脚 */
        dev_dbg(dev, "try to register %d pins ...\n",  pctldesc->npins);
        // 将所有pin注册到pinctrl子系统中
        // 为每个pin创建描述符并添加到基数树中
        ret = pinctrl_register_pins(pctldev, pctldesc->pins, pctldesc->npins);
        if (ret) {
                dev_err(dev, "error during pin registration\n");
                // 注册失败，清理已注册的pin描述符
                pinctrl_free_pindescs(pctldev, pctldesc->pins,
                                      pctldesc->npins);
                goto out_err;
        }

        return pctldev;

out_err:
		// 错误清理：销毁互斥锁和释放内存
        mutex_destroy(&pctldev->mutex);
        kfree(pctldev);
        return ERR_PTR(ret);
}
```
- **数据结构体**
	- **基数树**：用于快速查找pin、group、function
	- **链表**：用于管理GPIO范围和控制器节点
	- **互斥锁**：保护并发访问
- **操作函数验证**：验证基本操作、pinmux操作、配置操作的函数
- **pin注册流程**
	1. 调用`pinctrl_register_pins`注册所有的pin
	2. 为每个pin 创建描述符
	3. 失败时自动清理已注册的pin

> [!note]+ 背景知识 
> `基数树`（Radix Tree）是一种压缩的前缀树，Linux内核大量使用它来实现高效的键值查找。相比普通的链表或数组，基数树在查找、插入和删除操作上都有更好的性能。

## 6 三大操作函数集

> [!tip]+ 开发人员涉及
> - 芯片驱动开发：实现完整的ops
> - BSP开发：主要配置设备树
> -  复杂驱动开发：可能需要调试pinctrl问题
> - 一般驱动开发：调用pinctrl高层API
> - 应用开发：使用标准接口(/dev, /sys)

在`pinctrl_desc`结构体中总共有三个函数操作集，具体内容如下所示：

```c
const struct pinctrl_ops *pctlops; // 引脚控制操作函数指针
const struct pinmux_ops *pmxops; // 引脚复用操作函数指针
const struct pinconf_ops *confops; // 引脚配置操作函数指针
```

### 6.1 pinctrl_ops - 通用引脚控制操作

`pinctrl_ops`结构体定义在内核源码的`include/linux/pinctrl/pinctrl.h`文件中，具体内容如下所示：

```c fold
// include/linux/pinctrl/pinctrl.h
struct pinctrl_ops {
	/* 获取引脚组总数 */
	int (*get_groups_count) (struct pinctrl_dev *pctldev);
	
	/* 根据选择器获取引脚组名称 */
	const char *(*get_group_name) (struct pinctrl_dev *pctldev,
				       unsigned selector);
	
	/* 获取指定引脚组中包含的引脚列表和数量 */
	int (*get_group_pins) (struct pinctrl_dev *pctldev,
			       unsigned selector,
			       const unsigned **pins,
			       unsigned *num_pins);
	
	/* 显示引脚调试信息到seq_file */
	void (*pin_dbg_show) (struct pinctrl_dev *pctldev, struct seq_file *s,
			  unsigned offset);
	
	/* 将设备树节点转换为pinctrl映射表 */
	int (*dt_node_to_map) (struct pinctrl_dev *pctldev,
			       struct device_node *np_config,
			       struct pinctrl_map **map, unsigned *num_maps);
	
	/* 释放由dt_node_to_map分配的pinctrl映射表 */
	void (*dt_free_map) (struct pinctrl_dev *pctldev,
			     struct pinctrl_map *map, unsigned num_maps);
};
```


| 函数名                  | 作用描述                      | 入参                                                                                      | 返回值                                |
| -------------------- | ------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------- |
| **get_groups_count** | 获取pinctrl控制器中定义的**引脚组总数** | `pctldev`: pinctrl设备指针                                                                  | 成功: 引脚组总数(≥0)<br>失败: 负数错误码         |
| **get_group_name**   | 根据选择器获取对应**引脚组的名称**       | `pctldev`: pinctrl设备指针<br>`selector`: 引脚组索引号                                            | 成功: 引脚组名称字符串指针<br>失败: NULL         |
| **get_group_pins**   | 获取指定**引脚组中包含的引脚列表和数量**    | `pctldev`: pinctrl设备指针<br>`selector`: 引脚组索引号<br>`pins`: 输出引脚列表指针<br>`num_pins`: 输出引脚数量  | 成功: 0，pins和num_pins有效<br>失败: 负数错误码 |
| **pin_dbg_show**     | 将指定引脚的调试信息输出到seq_file     | `pctldev`: pinctrl设备指针<br>`s`: seq_file输出接口<br>`offset`: 引脚偏移量                          | 无返回值(void)                         |
| **dt_node_to_map**   | 将设备树节点转换为pinctrl映射表       | `pctldev`: pinctrl设备指针<br>`np_config`: 设备树配置节点<br>`map`: 输出映射表指针<br>`num_maps`: 输出映射表数量 | 成功: 0，map和num_maps有效<br>失败: 负数错误码  |
| **dt_free_map**      | 释放由dt_node_to_map分配的映射表内存 | `pctldev`: pinctrl设备指针<br>`map`: 要释放的映射表<br>`num_maps`: 映射表数量                           | 无返回值(void)                         |

### 6.2 pinmux_ops - 引脚复用操作

`pinmux_ops`结构体定义在内核源码的`include/linux/pinctrl/pinmux.h`文件中，具体内容如下所示：
```c fold
// include/linux/pinctrl/pinmux.h
/**
 * struct pinmux_ops - 引脚复用操作函数集合
 * 定义了引脚复用控制器驱动需要实现的操作接口
 */
struct pinmux_ops {
	/* 请求使用指定的引脚 */
	int (*request) (struct pinctrl_dev *pctldev, unsigned offset);
	
	/* 释放指定的引脚 */
	int (*free) (struct pinctrl_dev *pctldev, unsigned offset);
	
	/* 获取支持的复用功能总数 */
	int (*get_functions_count) (struct pinctrl_dev *pctldev);
	
	/* 根据选择器获取复用功能名称 */
	const char *(*get_function_name) (struct pinctrl_dev *pctldev,
					  unsigned selector);
	
	/* 获取指定功能对应的引脚组列表 */
	int (*get_function_groups) (struct pinctrl_dev *pctldev,
				  unsigned selector,
				  const char * const **groups,
				  unsigned *num_groups);
	
	/* 设置引脚复用功能：将指定引脚组配置为指定功能 */
	int (*set_mux) (struct pinctrl_dev *pctldev, unsigned func_selector,
			unsigned group_selector);
	
	/* 请求并启用GPIO功能 */
	int (*gpio_request_enable) (struct pinctrl_dev *pctldev,
				    struct pinctrl_gpio_range *range,
				    unsigned offset);
	
	/* 禁用并释放GPIO功能 */
	void (*gpio_disable_free) (struct pinctrl_dev *pctldev,
				   struct pinctrl_gpio_range *range,
				   unsigned offset);
	
	/* 设置GPIO方向（输入/输出） */
	int (*gpio_set_direction) (struct pinctrl_dev *pctldev,
				   struct pinctrl_gpio_range *range,
				   unsigned offset,
				   bool input);
	
	bool strict;	/* 严格模式标志：是否严格检查引脚冲突 */
};
```


| 函数名                     | 作用描述                      | 入参                                                                                                 | 返回值                                    |
| ----------------------- | ------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **request**             | **请求使用指定的引脚**，进行资源申请和冲突检查 | `pctldev`: pinctrl设备指针<br>`offset`: 引脚偏移量                                                          | 成功: 0<br>失败: 负数错误码(如-EBUSY)            |
| **free**                | **释放指定的引脚**，释放之前申请的引脚资源   | `pctldev`: pinctrl设备指针<br>`offset`: 引脚偏移量                                                          | 成功: 0<br>失败: 负数错误码                     |
| **get_functions_count** | 获取pinctrl控制器支持的复用功能总数     | `pctldev`: pinctrl设备指针                                                                             | 成功: 复用功能总数(≥0)<br>失败: 负数错误码            |
| **get_function_name**   | 根据选择器获取复用功能的名称            | `pctldev`: pinctrl设备指针<br>`selector`: 功能选择器索引                                                      | 成功: 功能名称字符串指针<br>失败: NULL              |
| **get_function_groups** | 获取指定功能对应的引脚组列表            | `pctldev`: pinctrl设备指针<br>`selector`: 功能选择器索引<br>`groups`: 输出引脚组名称列表<br>`num_groups`: 输出引脚组数量      | 成功: 0，groups和num_groups有效<br>失败: 负数错误码 |
| **set_mux**             | 设置引脚复用功能，将指定引脚组配置为指定功能    | `pctldev`: pinctrl设备指针<br>`func_selector`: 功能选择器<br>`group_selector`: 引脚组选择器                       | 成功: 0<br>失败: 负数错误码                     |
| **gpio_request_enable** | 请求并启用GPIO功能               | `pctldev`: pinctrl设备指针<br>`range`: GPIO范围结构体<br>`offset`: 引脚偏移量                                    | 成功: 0<br>失败: 负数错误码                     |
| **gpio_disable_free**   | 禁用并释放GPIO功能               | `pctldev`: pinctrl设备指针<br>`range`: GPIO范围结构体<br>`offset`: 引脚偏移量                                    | 无返回值(void)                             |
| **gpio_set_direction**  | 设置GPIO方向（输入/输出）           | `pctldev`: pinctrl设备指针<br>`range`: GPIO范围结构体<br>`offset`: 引脚偏移量<br>`input`: 方向标志(true=输入,false=输出) | 成功: 0<br>失败: 负数错误码                     |

**附加成员变量：**
- `strict`: bool类型，严格模式标志，是否严格检查引脚冲突

### 6.3 pinconf_ops - 引脚配置操作

`pinconf_ops`结构体定义在内核源码的`include/linux/pinctrl/pinconf.h`文件中，具体内容如下所示：

```c fold
// include/linux/pinctrl/pinconf.h
/**
 * struct pinconf_ops - 引脚配置操作函数集合
 * 定义了引脚配置控制器驱动需要实现的操作接口
 */
struct pinconf_ops {
#ifdef CONFIG_GENERIC_PINCONF
	bool is_generic;	/* 是否使用通用引脚配置接口 */
#endif
	/* 获取单个引脚的配置参数 */
	int (*pin_config_get) (struct pinctrl_dev *pctldev,
			       unsigned pin,
			       unsigned long *config);
	
	/* 设置单个引脚的配置参数 */
	int (*pin_config_set) (struct pinctrl_dev *pctldev,
			       unsigned pin,
			       unsigned long *configs,
			       unsigned num_configs);
	
	/* 获取引脚组的配置参数 */
	int (*pin_config_group_get) (struct pinctrl_dev *pctldev,
				     unsigned selector,
				     unsigned long *config);
	
	/* 设置引脚组的配置参数 */
	int (*pin_config_group_set) (struct pinctrl_dev *pctldev,
				     unsigned selector,
				     unsigned long *configs,
				     unsigned num_configs);
	
	/* 调试接口：解析并修改配置参数 */
	int (*pin_config_dbg_parse_modify) (struct pinctrl_dev *pctldev,
					   const char *arg,
					   unsigned long *config);
	
	/* 显示单个引脚配置的调试信息 */
	void (*pin_config_dbg_show) (struct pinctrl_dev *pctldev,
				     struct seq_file *s,
				     unsigned offset);
	
	/* 显示引脚组配置的调试信息 */
	void (*pin_config_group_dbg_show) (struct pinctrl_dev *pctldev,
					   struct seq_file *s,
					   unsigned selector);
	
	/* 显示配置参数的调试信息 */
	void (*pin_config_config_dbg_show) (struct pinctrl_dev *pctldev,
					    struct seq_file *s,
					    unsigned long config);
};
```


| 函数名                             | 作用描述           | 入参                                                                                         | 返回值                             |
| ------------------------------- | -------------- | ------------------------------------------------------------------------------------------ | ------------------------------- |
| **pin_config_get**              | 获取单个引脚的配置参数    | `pctldev`: pinctrl设备指针<br>`pin`: 引脚编号<br>`config`: 输出配置参数                                  | 成功: 0，config包含有效配置<br>失败: 负数错误码 |
| **pin_config_set**              | 设置单个引脚的配置参数    | `pctldev`: pinctrl设备指针<br>`pin`: 引脚编号<br>`configs`: 配置参数数组<br>`num_configs`: 配置参数数量        | 成功: 0<br>失败: 负数错误码              |
| **pin_config_group_get**        | 获取引脚组的配置参数     | `pctldev`: pinctrl设备指针<br>`selector`: 引脚组选择器<br>`config`: 输出配置参数                           | 成功: 0，config包含有效配置<br>失败: 负数错误码 |
| **pin_config_group_set**        | 设置引脚组的配置参数     | `pctldev`: pinctrl设备指针<br>`selector`: 引脚组选择器<br>`configs`: 配置参数数组<br>`num_configs`: 配置参数数量 | 成功: 0<br>失败: 负数错误码              |
| **pin_config_dbg_parse_modify** | 调试接口：解析并修改配置参数 | `pctldev`: pinctrl设备指针<br>`arg`: 命令行参数字符串<br>`config`: 输出解析后的配置                            | 成功: 0，config包含解析结果<br>失败: 负数错误码 |
| **pin_config_dbg_show**         | 显示单个引脚配置的调试信息  | `pctldev`: pinctrl设备指针<br>`s`: seq_file输出接口<br>`offset`: 引脚偏移量                             | 无返回值(void)                      |
| **pin_config_group_dbg_show**   | 显示引脚组配置的调试信息   | `pctldev`: pinctrl设备指针<br>`s`: seq_file输出接口<br>`selector`: 引脚组选择器                          | 无返回值(void)                      |
| **pin_config_config_dbg_show**  | 显示配置参数的调试信息    | `pctldev`: pinctrl设备指针<br>`s`: seq_file输出接口<br>`config`: 配置参数值                             | 无返回值(void)                      |

### 6.4 RK3568平台的具体实现

首先找到内核源码目录下的`/drivers/pinctrl/pinctrl-rockchip.c`文件，在`rockchip_pinctrl_register`函数中对三个函数操作集进行了填充，具体内容如下所示：

```c
ctrldesc->pctlops = &rockchip_pctrl_ops;
ctrldesc->pmxops = &rockchip_pmx_ops;
ctrldesc->confops = &rockchip_pinconf_ops;
```

接下来对每一个函数操作集的具体实现进行讲解。

**1、pinctrl_ops的填充**
```c
static const struct pinctrl_ops rockchip_pctrl_ops = {
	.get_groups_count	= rockchip_get_groups_count,
	.get_group_name		= rockchip_get_group_name,
	.get_group_pins		= rockchip_get_group_pins,
	.dt_node_to_map		= rockchip_dt_node_to_map,
	.dt_free_map		= rockchip_dt_free_map,
};
```

**2、pinmux_ops填充**
```c
static const struct pinmux_ops rockchip_pmx_ops = {
	.get_functions_count	= rockchip_pmx_get_funcs_count,
	.get_function_name	= rockchip_pmx_get_func_name,
	.get_function_groups	= rockchip_pmx_get_groups,
	.set_mux		= rockchip_pmx_set,
};
```

**3、pinconf_ops填充**
```c
static const struct pinconf_ops rockchip_pinconf_ops = {
	.pin_config_get			= rockchip_pinconf_get,
	.pin_config_set			= rockchip_pinconf_set,
	.is_generic			= true,
};
```

> [!tip]+ 实用技巧 
> 这三个操作函数集体现了面向对象的设计思想，**通过函数指针实现了多态，不同平台可以提供不同的实现，但对上层提供统一的接口**。

### 6.5 rockchip_dt_node_to_map函数实现

设备树（Device Tree）中存放的是对硬件设备信息的描述，包含了硬件设备的配置和连接信息，例如在pinctrl节点中的引脚的配置和映射关系。而`rockchip_dt_node_to_map`函数的作用就是根据设备树中的节点信息，生成对应的引脚映射数组。这个映射数组将描述硬件功能（如复用功能和配置信息）与设备树中的引脚信息进行绑定。

在讲解该函数之前需要先对`struct pinctrl_map`结构体进行介绍，该结构体定义在内核源码的`include/linux/pinctrl/machine.h`目录下，具体内容如下所示：
```c
struct pinctrl_map {
    const char *dev_name;           // 设备名称
    const char *name;               // 映射名称
    enum pinctrl_map_type type;     // 映射类型
    const char *ctrl_dev_name;      // 控制设备名称
    union {
        struct pinctrl_map_mux mux;         // 复用映射数据
        struct pinctrl_map_configs configs;  // 配置映射数据
    } data;
};
```
该结构体用于在引脚控制器中定义引脚的映射关系。通过映射类型的不同，可以将引脚与具体的复用功能或配置信息关联起来，从而实现引脚的配置和控制。

**rockchip_dt_node_to_map函数实现**
- 该函数的具体实现在内核源码的`drivers/pinctrl/pinctrl-rockchip.c`文件中，具体内容如下所示：
```c fold
static int rockchip_dt_node_to_map(struct pinctrl_dev *pctldev,
                                 struct device_node *np,
                                 struct pinctrl_map **map, unsigned *num_maps)
{
    struct rockchip_pinctrl *info = pinctrl_dev_get_drvdata(pctldev);
    const struct rockchip_pin_group *grp;
    struct pinctrl_map *new_map;
    struct device_node *parent;
    int map_num = 1;    // 映射数量，默认为1
    int i;
    
    /*
     * 首先查找此节点的引脚组，并检查是否需要为引脚创建配置映射
     */
    /* 查找引脚组 */
    grp = pinctrl_name_to_group(info, np->name);  // 根据节点名称查找对应的引脚组
    if (!grp) {
        dev_err(info->dev, "unable to find group for node %pOFn\n",
                np);
        return -EINVAL;
    }
    
    map_num += grp->npins;  // 计算映射数量，包括复用映射和配置映射
    new_map = kcalloc(map_num, sizeof(*new_map), GFP_KERNEL);  // 分配内存空间用于存储映射数组
    if (!new_map)
        return -ENOMEM;
        
    *map = new_map;         // 将分配的映射数组赋值给输出参数
    *num_maps = map_num;    // 将映射数量赋值给输出参数
    
    /* 创建复用映射 */
    parent = of_get_parent(np);
    if (!parent) {
        kfree(new_map);
        return -EINVAL;
    }
    new_map[0].type = PIN_MAP_TYPE_MUX_GROUP;
    new_map[0].data.mux.function = parent->name;
    new_map[0].data.mux.group = np->name;
    of_node_put(parent);
    
    /* 创建配置映射 */
    new_map++;
    for (i = 0; i < grp->npins; i++) {
        new_map[i].type = PIN_MAP_TYPE_CONFIGS_PIN;
        new_map[i].data.configs.group_or_pin =
                        pin_get_name(pctldev, grp->pins[i]);
        new_map[i].data.configs.configs = grp->data[i].configs;
        new_map[i].data.configs.num_configs = grp->data[i].nconfigs;
    }
    
    dev_dbg(pctldev->dev, "maps: function %s group %s num %d\n",
            (*map)->data.mux.function, (*map)->data.mux.group, map_num);
    
    return 0;
}
```

**函数执行流程分析**
1. **获取pinctrl控制器数据**：通过`pinctrl_dev_get_drvdata`获取RockChip平台特定的pinctrl信息。
2. **查找引脚组**：根据设备树节点名称查找对应的引脚组，如果找不到则返回错误。
3. **计算映射数量**：映射数量包括1个复用映射加上引脚组中所有引脚的配置映射。
4. **分配映射数组内存**：使用`kcalloc`分配足够的内存来存储所有映射。
5. **创建复用映射**：
    - 映射类型设置为`PIN_MAP_TYPE_MUX_GROUP`
    - function名称来自父节点名称
    - group名称来自当前节点名称
6. **创建配置映射**：为引脚组中的每个引脚创建配置映射：
    - 映射类型设置为`PIN_MAP_TYPE_CONFIGS_PIN`
    - 设置每个引脚的名称和配置参数

> [!important]+ 关键点
> 
> - **映射表的作用**：将设备树中的引脚描述转换为内核pinctrl子系统能够理解的数据结构
> - **两种映射类型**：
>     - 复用映射（MUX）：定义引脚的功能选择
>     - 配置映射（CONFIG）：定义引脚的电气特性（如上下拉、驱动强度等）
> - **内存管理**：映射表的内存由`dt_node_to_map`分配，由`dt_free_map`释放

## 7 总结

Pinctrl子系统的数据结构设计体现了优秀的软件工程思想：
1. **分层设计**：从通用的`pinctrl_desc`到平台特定的`rockchip_pinctrl`，层次清晰
2. **组合模式**：通过组合而不是继承来扩展功能
3. **面向对象**：通过函数指针实现多态，支持不同平台的实现
4. **自动化管理**：引脚配置的应用完全自动化，驱动开发者无需手动干预

**简单来说**，这套数据结构就像一个精密的齿轮系统，每个组件都有自己明确的职责，组合在一起就能高效地管理复杂的引脚资源。理解了这些数据结构的设计思想，就掌握了现代内核驱动开发的核心理念。

> [!warning]+ 重要提醒 
> 理解这些数据结构的关系对于调试pinctrl相关问题非常重要，当遇到引脚配置问题时，可以从这些数据结构入手分析问题根源。