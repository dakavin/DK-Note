
## 1 什么是设备树插件？

Linux 4.4版本后引入了 **动态设备树（Dynamic DeviceTree）**，也叫 **设备树插件**

简单理解：
- 设备树插件就像是主设备树的`补丁`
- 可以动态加载到系统中，不需要重新编译整个设备树
- eg. 添加RGB驱动，只需要写一个RGB设备树插件，编译后直接加载即可

## 2 设备树插件格式

**格式一：使用fragment结构**
```dts fold
/dts-v1/;
/plugin/;

/ {
	fragment@0 {
		target-path = "/";
		__overlay__ {
			/* 在此添加要插入的节点 */
			......
		};
	};

	fragment@1 {
		target = <&xxx>;
		__overlay__ {
			/* 在此添加要插入的节点 */
			......
		};
	};
	......
};
```
- 第1行：指定dts版本
- 第2行：`/plugin/`表示这是一个插件，可以引用主设备树中的节点
- `target-path = "/"` 或 `target = <&XXXXX>`：制定插件加载到哪个节点下
- `__overlay__`：里面放要插入或修改的节点内容

**格式二：直接引用节点（推荐）**
```dts fold
/dts-v1/;
/plugin/;

&{/} {
	/* 此处在根节点"/"下，添加要插入的节点和属性 */
}

&xxx {
	/* 此处在节点"xxx"下，添加要插入的节点和属性 */
}
```
- 这种格式更简洁，本文实验采用这种方式

## 3 设备树插件加载

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/2b9572a6cb6007577603ecada49dac55.png)


```txt
编写.dts文件 
    ↓
用DTC工具编译生成.dtbo文件
    ↓
存储到boot分区
    ↓
uboot加载并合并dtbo和dtb文件
    ↓
启动内核，传递合并后的设备树地址
```

## 4 设备树插件实验

### 4.1 编写设备树插件文件

> [!tip]+ 提示
> 为了避免冲突，需要删除上一章节在主设备树上添加的led_test节点，改为设备树插件的形式

在内核源码 `/arch/arm64/boot/dts/rockchip/overlays/` 目录下创建 `lubancat-led-overlay.dts`：
```dts
/dts-v1/;
/plugin/;

/ {
    fragment@0 {
        target-path = "/";
        __overlay__ {
            /* 添加led_test节点 */
            led_test{
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "dk,led_test";
                ranges;

                led@0xfdd60004{
                    reg = <0xfdd60004 0x4 0xfdd6000C 0x4>;
                    status = "okay";
                };
            };
        };
    };
};
```
- 第6行：指定设备树插件的加载位置，加载到根节点下
- 第8-21行：插入的设备即节点放在overlay内部

### 4.2 修改Makefile

- 修改内核目录/arch/arm64/boot/dts/rockchip/overlays下的Makefile文件， 添加我们编辑好的设备树插件
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/feb24ba34d387c82b87c16fffbef2054.png)

- 把设备树插件文件放在和Makefile文件同级目录下。 以进行设备树插件的编译

### 4.3 编译设备树插件

在内核目录下编译设备树（推荐，比较快）
```shell
# 先指定配置文件
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 lubancat2_defconfig

# 然后编译设备树
sudo bear -- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=aarch64-linux-gnu-gcc-9 -j8 dtbs
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/83f81dcea5edcf476e57ed073c635962.png)

### 4.4 部署设备树插件

编译出来的设备树插件在 `内核源码/arch/arm64/boot/dts/rockchip/overlay/lubancat-led-overlay.dtbo`

将设备树插件传到板卡的 `/boot/dtb/overlay/` 目录下，并在 `/boot/uEnv/uEnv.txt` 按照格式添加我们的设备树插件，然后重启开发板，那么系统就会加载我们编译的设备树插件。

> [!tip]+ 提示
> 删除了之前dts的描述后，一样需要替换dtb文件

**先scp将dtb和dtbo转移到板卡用户目录下**
```shell
scp arch/arm64/boot/dts/rockchip/rk3568-lubancat-2-v2.dtb arch/arm64/boot/dts/rockchip/overlay/lubancat-led-overlay.dtbo cat@192.168.31.69:/home/cat/
```

**然后转移dtb和dtbo到对应的目录下**
```shell
sudo mv rk3568-lubancat-2-v2.dtb /boot/dtb/rk3568-lubancat-2-v2.dtb

sudo mv lubancat-led-overlay.dtbo /boot/dtb/overlay/
```

**修改uEnv.txt文件**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/9f2a95a5762da03eadc6143c3c7b99d5.png)

**最后重启一下**

## 5 测试验证

> [!tip]+ 提示
> 驱动部分和上一章节一样，不做过多说明

[5 程序运行结果](9_Linux设备树_LED灯实验.md#5%20程序运行结果)


