
`pinctrl` 节点是设备树（Device Tree）中用来描述硬件平台**引脚控制（Pin Control）** 相关信息的节点。
## 1 DTS介绍

RK芯片的设备树⼀般把`pinctrl`节点放在`soc.dtsi`，例如`rk3588s.dtsi`，⼀般位于最后⼀个节点，例如在 `rk3568.dtsi` 文件的最后

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6f4afd076e6caf61bff95e31368d13eb.png)

如上图，`pinctrl`节点没有`reg`，它不是⼀个标准`platform device`，寄存器基地值是通过`rockchip,grf=<&grf>` 传⼊；

驱动内部根据这个基地值，加偏移地址，完成`IOMUX`、`PINCONF`的配置；`GPIO`是使⽤`gpio`节点的`reg`地址
- IOMUX：引脚多功能复用，即切换引脚功能
- PINCONF：引脚的电气属性及行为配置，常见配置如下
	- 上下拉电阻
	- 驱动能力
	- 输出类型（推挽/开漏）
	- 输入/输出是能
	- 输入延迟、滤波
	- 中断触发类型
	- Schmitt出发
	- 保留模式（HIZ）

而在 `rk3568-pinctrl.dtsi` 文件中还存在引用的方式配置 `pinctrl`

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8493d1260d0ef63767574ebcab3d58db.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e6067bf4638c7bb820905a99053b0a75.png)

## 2 新建 pinctrl

`rk3568-pinctrl.dtsi`⽂件已经枚举了rk3568芯⽚所有`iomux`的实例，各模块⼀般不再需要创建`iomux`实例； 创建`iomux`实例需要遵循如下规则：
1. 必须在`pinctrl`节点下
2. 必须以`function`+`group`的形式添加
3. `function`+`group`的格式如下
```C
function {
        group {
                rockchip,pin = <bank gpio func &ref>;
        };
};
```
4. 遵循其他dts的基本规则
## 3 引用 pinctrl

模块引⽤`pinctrl`是通过 `pinctrl-names` 和 `pinctrl-0` 来连接模块 和`pinctrl`驱动。 

**举例： uart2：**
```C
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

`uart2m1_xfer` 是⼀个`pinctrl group`；

模块可以同时引⽤多组`group`，例如：

```C
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
	pinctrl-0 = <&pdm1m0_clk&pdm1m0_clk1&pdm1m0_sdi0&pdm1m0_sdi1&pdm1m0_sdi2&pdm1m0_sdi3>;#sound-dai-cells = <0>;
	status = "disabled";};
```

`pinctrl-names`可以支持多个实例（`state`），`pinctrl`默认提供了4种实例：
```C
#define PINCTRL_STATE_DEFAULT "default"
#define PINCTRL_STATE_INIT "init"
#define PINCTRL_STATE_IDLE "idle"
#define PINCTRL_STATE_SLEEP "sleep"
```

`"init"` 在`driver probe`期间⽣效，`probe done`之后可能会切换回 `"default"`（如果`probe`中切换到其他`state`，就不会切换回 `"init"`）。 `pinctrl-names` 是可以⾃定义的，可以在`driver`中去匹配解析。

```C
pwm4: pwm@febd0000 {
	compatible = "rockchip,rk3588-pwm", "rockchip,rk3328-pwm";
	reg = <0x0 0xfebd0000 0x0 0x10>;#pwm-cells = <3>;
	pinctrl-names = "active";
	pinctrl-0 = <&pwm4m0_pins>;
	clocks = <&cru CLK_PWM1>, <&cru PCLK_PWM1>;
	clock-names = "pwm", "pclk";
	status = "disabled";};
```